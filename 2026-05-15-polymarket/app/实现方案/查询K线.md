# 开发
1. 数据库:mysql
2. 开发语言:java

# 要求
1. 执行“连接数据库”加载skill,通过提供的表读取表结构
2. 一定要遵循现有代码的规范
3. 关键位置和方法需要中文注释
4. SQL需要有一个很好的可读性格式
5. 写代码时一直执行，无需我确认
6. 禁止修改表结构,如果必须修改表结构只给出sql语句
7. 返回的时间、Long 、BigDecimal 都必须转化成String类型,时间必须转化成时间戳返回
8. 非特殊说明全部使用post请求
9. 不能修改现有业务逻辑
10. 所有参数和响应字段都需要使用驼峰命名
11. 请求中除tinyint、Int、BigDecimal 外其他全部用string类型
12. 项目入口:doge-app-api
13. 完成后标记为已完成,下次自动读取未完成的

# 接口
## 查询K线（已完成）
* 该接口使用Get请求
* 查询表: `oracle_kline_snapshot_${symbol.toLowerCase()}`
### 请求参数
* issueId 期次ID(必填)
* limit 返回数量(必填, 仅限制数据库查询条数)
* time 上一页最后一条K线时间戳(毫秒, 非必填)
### 响应字段
* symbol 币对
* cycle K线周期
* data K线数据
* price 价格
* time 时间(返回快照表中的 lastTime)

# 实现口径
1. 先查询 `oracle_issue`，取 `symbol`、`beginTime`、`klinePeriod`
2. 动态表名按 `oracle_kline_snapshot_${symbol.toLowerCase()}` 生成
3. 查询范围在业务层计算，不在 SQL 中做时间换算
4. `startTime = issue.beginTime`
5. `endTime = issue.beginTime + klinePeriod 对应毫秒数`
6. SQL 查询条件：
   * `kline_interval = klinePeriod`
   * `open_time >= startTime`
   * `closeTime < endTime`
7. SQL 按 `lastTime desc` 查询，并结合 `limit` 只取最新数据
8. service 层将结果反转成 `lastTime asc` 后返回前端，方便图表直接绘制

# 调整内容
1. 查询表调整为 `oracle_kline_snapshot_${symbol.toLowerCase()}`（已完成）
2. 查询条件增加 `kline_interval` 过滤（已完成）
3. 查询范围改为 `open_time >= startTime` 且 `closeTime < endTime`（已完成）
4. 时间范围计算放在业务层，不在 SQL 中处理（已完成）
5. 返回时间字段使用 `lastTime`，并按时间正序返回前端（已完成）
6. 查询异常,xml中字段名字使用了驼峰命名（已完成）
7. price 去除末尾0（已完成）
8. 获取K线时,需通过查询数据库的数据和缓存数据融合,缓存key(`market:kline:<交易对小写>:<K线周期小写>`),缓存值为 `List` 且最多参与融合 30 条数据,拿到数据后通过 `lastTime` 去重,最终按 `lastTime` 升序返回,limit 仅限制数据库查询条数（已完成）
9. 判断当前时间小于期次开始时间则返回空集合（已完成）
10. 从redis拿到数据后,需通过time过滤出在期次的开始和结束时间范围内的数据(包含开始和结束时间)（已完成）
11. limit修改为必填,后台不做默认值及最大值和最小值校验
    增加time字段(时间戳,毫秒),如果为空才从redis中加载数据+数据库数据返回,如果time不为空,通过数据库查询lastTime小于time的limit数据,并保留`kline_interval = klinePeriod`、`open_time >= startTime`、`close_time < endTime`条件（已完成）
