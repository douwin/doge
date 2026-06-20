# 开发
1. 数据库: mysql
2. 开发语言: java
3. 项目入口:
   * `doge-app-api`
   * `doge-ws`

# 要求
1. 一定要遵循现有代码规范
2. 关键位置和方法需要中文注释
3. 不能修改现有业务逻辑
4. 本次推送为公共盘口数据,不做用户鉴权
5. 推送维度按期次处理,不推送用户私有信息
6. 通过 Redis 发布订阅实现,不采用定时扫描 Redis 的方式
7. 支付主流程不能因 WS 推送失败而回滚
8. `doge-app-api` 发布事件不能影响支付流程,发布动作必须异步并吞掉异常

# 目标
1. `doge-app-api` 在 Oracle 支付成功后,向 `doge-ws` 广播期次金额变更
2. 推送数据必须包含:
   * 期次ID `issueId`
   * 期次编码 `issueCode`
   * 交易对 `symbol`
   * 有效总金额 `totalEffectiveAmount`
   * 涨方有效金额 `positiveAmount`
   * 跌方有效金额 `negativeAmount`
3. `doge-ws` 不查询数据库,只消费 Redis 消息并广播给所有在线连接
4. 前端收到广播后,按 `issueId` 自行过滤并刷新页面展示

# 现状分析
1. `OracleController.pay` 当前支付成功后直接返回 `true`,没有推送链路
2. `OracleIssueServiceImpl.pay` 是事务方法,在支付成功后会:
   * 写入 `oracle_order`
   * 扣减用户资产
   * 写钱包账单
   * 回写 `oracle_issue` 聚合字段
3. `doge-ws` 当前已有广播能力,但只有“全量广播”
4. `doge-ws` 当前 K 线广播是“定时任务 + Redis 拉取 + 全量广播”
5. 本需求不适合照搬 K 线的定时扫描模式,更适合走“支付成功事件驱动”

# 方案边界
1. 本次广播只负责同步公共盘口金额变化
2. 不推送账户ID、订单号、支付金额、余额等用户私有数据
3. 不需要前端订阅某一期次,所有在线连接都接收广播
4. 不要求消息可靠补偿,页面首次进入和断线重连后仍以 HTTP 查询结果为准

# 总体方案
## 整体链路
1. 前端连接 `doge-ws` 的 `/ws`
2. 用户在 `doge-app-api` 调用 `/v1/oracle/pay`
3. `OracleIssueServiceImpl.pay` 完成支付事务,更新 `oracle_issue.total_effective_amount/positive_amount/negative_amount`
4. `OracleIssueServiceImpl.pay` 返回后,说明支付事务已提交
5. `OracleController.pay` 以“异步 + try/catch”的方式触发 Redis Pub/Sub 发布
6. `doge-ws` 订阅 Redis 频道,收到消息后组装 WS 报文
7. `doge-ws` 通过现有 `WebSocketFrameHandler.broadcast(...)` 全量广播给所有在线连接
8. 前端收到消息后,按 `issueId` 判断是否为当前页面关注的期次,命中则刷新展示

## 为什么采用“service返回后异步发布”
1. `OracleIssueServiceImpl.pay(...)` 是事务方法,返回时数据库事务已经提交
2. 若在事务中同步发 Redis,会出现:
   * Redis 慢或异常拖慢支付接口
   * 发布异常直接影响接口返回
3. 若使用同步 `AFTER_COMMIT` 监听器,虽然不会回滚数据库,但仍可能拖慢请求线程
4. 因此这里最稳妥的做法是:
   * `service` 只返回推送所需数据
   * `controller` 在收到成功结果后异步提交发布任务
   * 提交任务失败和 Redis 发布失败都只记日志,不影响接口返回

# Redis 发布订阅设计
## 频道名称
```text
ws:oracle:issue:amount:update
```

## 发布消息体
```json
{
  "event": "oracle_issue_amount_update",
  "data": {
    "issueId": "1001",
    "issueCode": "DOGE-1H-2026061509",
    "symbol": "DOGEUSDT",
    "totalEffectiveAmount": "1250.36",
    "positiveAmount": "760.12",
    "negativeAmount": "490.24",
    "ts": "1750460000000"
  }
}
```

## 字段口径
1. `totalEffectiveAmount`
   * 支付成功并更新聚合后的最新有效总金额
2. `positiveAmount`
   * 支付成功并更新聚合后的最新涨方有效金额
3. `negativeAmount`
   * 支付成功并更新聚合后的最新跌方有效金额
4. `ts`
   * 服务端当前毫秒时间戳字符串

# doge-app-api 实现方案
## 设计原则
1. 不在事务中直接发布 Redis 消息
2. 不让 WS 推送异常影响支付主流程
3. 支付成功返回前,最多只做“异步任务提交”; 任务提交失败也只记日志
4. Redis 真正发布在异步线程中执行,失败只记日志

## 推荐实现结构
1. 调整 `IOracleIssueService.pay(...)` 返回值:
   * `OracleIssueAmountNoticeDto`
2. 新增发布器:
   * `OracleIssueWsNotifyPublisher`
3. 新增线程池配置:
   * `OracleWsNotifyExecutorConfig`

## 触发点
1. `OracleIssueServiceImpl.pay(...)` 在完成以下动作后构建返回 DTO:
   * `oracle_order` 落库成功
   * `oracle_issue` 聚合字段更新成功
   * `account_oracle` 汇总更新成功
2. `OracleController.pay(...)` 在 `oracleIssueService.pay(req)` 成功返回后:
   * 调用 `oracleIssueWsNotifyPublisher.publishAsync(dto)`
   * 发布器内部自行切到线程池执行 Redis 发布
3. DTO 内容使用当前事务中已知的最新值,不额外查库

## 最新金额的获取方式
由于 `pay(...)` 内已锁定期次记录并已知本次 `effectiveAmount` 与 `choiceValue`,可以直接用“旧值 + 本次值”得到新值:

1. `newTotalEffectiveAmount = issue.totalEffectiveAmount + effectiveAmount`
2. 若 `choiceValue = 1`
   * `newPositiveAmount = issue.positiveAmount + effectiveAmount`
   * `newNegativeAmount = issue.negativeAmount`
3. 若 `choiceValue = 2`
   * `newPositiveAmount = issue.positiveAmount`
   * `newNegativeAmount = issue.negativeAmount + effectiveAmount`

其中空值统一按 `0` 处理。

## 为什么在 Controller 中只触发“异步发布”
1. 当前 `OracleController.pay` 在 service 返回后,事务已经提交,此时读取到的数据口径是安全的
2. 但 Controller 不能直接同步 `convertAndSend(...)`,否则 Redis 慢时会拖长支付接口耗时
3. 因此 Controller 只做一件事:
   * try-catch 包裹 `publishAsync(...)`
4. `publishAsync(...)` 内部再把真正的 Redis 发布切到线程池中处理
5. 即使线程池拒绝、Redis 故障、序列化异常,都不会影响 `JsonResult.success(true)` 返回

## 推荐代码位置
1. `doge-app-api/src/main/java/com/doge/app/api/model/dto/OracleIssueAmountNoticeDto.java`
2. `doge-app-api/src/main/java/com/doge/app/api/service/OracleIssueWsNotifyPublisher.java`
3. `doge-app-api/src/main/java/com/doge/app/api/service/impl/OracleIssueWsNotifyPublisherImpl.java`
4. `doge-app-api/src/main/java/com/doge/app/api/common/config/OracleWsNotifyExecutorConfig.java`
5. `doge-app-api/src/main/java/com/doge/app/api/controller/OracleController.java`
6. `doge-app-api/src/main/java/com/doge/app/api/service/IOracleIssueService.java`
7. `doge-app-api/src/main/java/com/doge/app/api/service/impl/OracleIssueServiceImpl.java`

# doge-ws 实现方案
## 设计原则
1. 沿用现有 `WebSocketFrameHandler.broadcast(...)` 做全量广播
2. 不查询数据库
3. 只订阅 Redis Pub/Sub 并转发消息
4. 客户端发来的普通文本消息不再全站转发,避免被滥用

## 推荐实现结构
1. 新增 Redis 订阅配置:
   * `RedisMessageListenerContainer`
2. 新增订阅监听器:
   * `OracleIssueAmountRedisSubscriber`
3. 新增广播服务:
   * `OracleIssueBroadcastService`

## 消费流程
1. 监听 Redis 频道 `ws:oracle:issue:amount:update`
2. 收到消息后进行 JSON 基础校验:
   * `event` 是否存在
   * `data.issueId` 是否存在
   * `data.totalEffectiveAmount/positiveAmount/negativeAmount` 是否存在
3. 校验通过后直接调用:
   * `webSocketFrameHandler.broadcast(payload)`
4. 若当前没有在线连接,直接记录 debug 日志并跳过

## 对现有 WebSocketFrameHandler 的建议调整
当前 `channelRead0(...)` 中,除 `ping` 外会把客户端任意文本消息广播给所有连接。该行为不适合线上公共广播场景。

建议改为:
1. 保留 `ping -> pong`
2. 其他客户端消息返回固定响应或直接忽略
3. 真正的业务广播只允许服务端内部调用 `broadcast(...)`

## 推荐代码位置
1. `doge-ws/src/main/java/com/doge/ws/config/RedisSubscriberConfig.java`
2. `doge-ws/src/main/java/com/doge/ws/service/OracleIssueBroadcastService.java`
3. `doge-ws/src/main/java/com/doge/ws/service/OracleIssueBroadcastServiceImpl.java`
4. `doge-ws/src/main/java/com/doge/ws/redis/OracleIssueAmountRedisSubscriber.java`
5. `doge-ws/src/main/java/com/doge/ws/netty/WebSocketFrameHandler.java`

# 前端接入约定
1. 连接地址保持不变:
   * `/ws`
2. 不需要 Token
3. 不需要订阅某一期次
4. 收到如下事件时处理:
   * `oracle_issue_amount_update`
5. 前端处理规则:
   * 若当前页面展示的是该 `issueId`,则更新金额
   * 否则忽略

# 日志建议
## doge-app-api
1. 支付成功后准备发布 WS 事件:
   * `issueId`
   * `issueCode`
   * `symbol`
   * `totalEffectiveAmount`
   * `positiveAmount`
   * `negativeAmount`
2. Redis 发布成功/失败摘要日志

## doge-ws
1. Redis 订阅启动成功日志
2. 收到 Redis 消息日志
3. 广播成功日志:
   * `event`
   * `issueId`
   * 当前在线连接数
4. 广播失败异常日志

# 异常处理
1. `doge-app-api` 异步任务提交失败:
   * 记录错误日志
   * 不影响支付接口返回成功
2. `doge-app-api` Redis 发布失败:
   * 在异步线程中记录错误日志
   * 不重试,不向上抛异常
3. `doge-ws` 收到脏数据:
   * 记录 warn 日志
   * 丢弃该消息
4. `doge-ws` 当前无在线连接:
   * 记录 debug 日志
   * 不报错

# 方案优点
1. 不需要鉴权,实现简单
2. 不需要 `doge-ws` 访问数据库
3. 不需要前端维护订阅关系
4. `doge-app-api` 发布失败不会影响支付成功返回
5. 完全基于支付成功事件驱动,时效性优于定时轮询
6. 能复用现有 `doge-ws` 的广播能力

# 方案限制
1. 广播是全量的,所有在线连接都会收到消息
2. Redis Pub/Sub 不做消息持久化,断线期间的广播不会补发
3. 前端必须按 `issueId` 自行过滤

# 结论
本需求最合理的做法是:
1. `doge-app-api` 在 Oracle 支付成功后,于事务提交完成后异步发布 Redis Pub/Sub 消息
2. `doge-ws` 订阅该事件并全量广播
3. 前端按 `issueId` 自行过滤和刷新

该方案与当前“不鉴权、按期次维度推送、Redis 发布订阅、广播模式”的要求完全一致,同时改动面最小、实现成本最低。
