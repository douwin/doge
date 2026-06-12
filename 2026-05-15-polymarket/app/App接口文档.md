# App接口文档

## 通用说明

### 接口前缀
`/v1/oracle`

### 请求方式
除特殊说明外，统一使用 `POST`

### 请求头
| 名称 | 是否必填 | 类型 | 说明 |
|---|---|---|---|
| `Language` | 否 | String | 语言，默认 `en_US` |
| `Token` | 按接口说明 | String | 登录凭证 |

### 通用响应结构
| 字段 | 类型 | 说明 |
|---|---|---|
| `success` | Boolean | 是否成功 |
| `ts` | String | 服务端当前时间戳 |
| `data` | Object | 响应数据 |
| `code` | Integer | 响应码，成功为 `200` |
| `msg` | String | 响应描述 |

### 分页响应结构
`data` 为分页对象时，结构如下：

| 字段 | 类型 | 说明 |
|---|---|---|
| `pageNo` | Integer | 当前页码 |
| `pageSize` | Integer | 每页数量 |
| `totalSize` | String | 总记录数 |
| `totalPages` | Integer | 总页数 |
| `prevPage` | Integer | 上一页 |
| `nextPage` | Integer | 下一页 |
| `list` | Array | 列表数据 |

### 字段说明
| 规则 | 说明 |
|---|---|
| 时间字段 | 返回时间戳字符串 |
| `Long` 字段 | 返回 String |
| `BigDecimal` 字段 | 返回 String |
| 请求参数 | 除 `tinyint`、`Int`、`BigDecimal` 外，其他统一使用 String |
| 命名风格 | 请求参数、响应字段统一使用驼峰命名 |

---

## 1. 查询交易对

### 接口信息
| 项 | 内容 |
|---|---|
| 接口名称 | 查询交易对 |
| 请求地址 | `/v1/oracle/symbol/select` |
| 请求方式 | `GET` |
| Token | 否 |

### 请求参数
无

### 响应数据
`data` 为 `List<SelectRes>`

| 字段 | 类型 | 说明 |
|---|---|---|
| `value` | String | 交易对编码 |
| `label` | String | 基础币名称 |

### 返回值说明
当前根据 `OraclePairSymbolEnum` 返回。

---

## 2. 查询推荐期次

### 接口信息
| 项 | 内容 |
|---|---|
| 接口名称 | 查询推荐期次 |
| 请求地址 | `/v1/oracle/issue/recommend/list` |
| 请求方式 | `GET` |
| Token | 否 |

### 业务说明
查询 `oracle_issue` 表中 `isRecommended=1` 且 `status=1` 的数据，按倒序返回，不分页。

### 请求参数
| 字段 | 类型 | 是否必填 | 说明 |
|---|---|---|---|
| `symbol` | String | 否 | 币对 |

### 响应数据
`data` 为 `List<OracleRecommendIssueRes>`

| 字段 | 类型 | 说明 |
|---|---|---|
| `id` | String | 期次ID |
| `issueCode` | String | 期次编码 |
| `symbol` | String | 币对 |
| `baseCoin` | String | 基础币 |
| `beginTime` | String | 开始时间 |
| `endTime` | String | 结束时间 |
| `openPrice` | String | 基准价 |
| `closePrice` | String | 结束价(当前价) |
| `startAdvanceMinutes` | Integer | 参与开始提前分钟数 |
| `cutoffAdvanceMinutes` | Integer | 参与截止提前分钟数 |
| `marketType` | Integer | 市场类型 |
| `status` | Integer | 状态：`1=进行中`，`2=待开始`，`3=待开奖`，`4=已开奖` |
| `statusName` | String | 状态名称 |
| `isRecommended` | Integer | 是否推荐：`1=是`，`2=否` |
| `totalEffectiveAmount` | String | 有效参与金额 |
| `positiveAmount` | String | 正方(涨方)金额 |
| `negativeAmount` | String | 反方(跌方)金额 |

---

## 3. 查询所有期次

### 接口信息
| 项 | 内容 |
|---|---|
| 接口名称 | 查询所有期次 |
| 请求地址 | `/v1/oracle/issue/list` |
| 请求方式 | `GET` |
| Token | 否 |

### 请求参数
| 字段 | 类型 | 是否必填 | 说明 |
|---|---|---|---|
| `pageNo` | Integer | 否 | 页码，默认 `1` |
| `pageSize` | Integer | 否 | 每页数量，默认 `10` |
| `status` | Integer | 否 | 状态：`1=进行中`，`2=待开始`，`3=待开奖`，`4=已开奖` |
| `symbol` | String | 否 | 币对 |

### 响应数据
`data` 为 `PageResult<OracleIssueListRes>`

| 字段 | 类型 | 说明 |
|---|---|---|
| `id` | String | 期次ID |
| `issueCode` | String | 期次编码 |
| `symbol` | String | 币对 |
| `baseCoin` | String | 基础币 |
| `beginTime` | String | 开始时间 |
| `endTime` | String | 结束时间 |
| `openPrice` | String | 基准价 |
| `closePrice` | String | 结束价(当前价) |
| `startAdvanceMinutes` | Integer | 参与开始提前分钟数 |
| `cutoffAdvanceMinutes` | Integer | 参与截止提前分钟数 |
| `marketType` | Integer | 市场类型 |
| `status` | Integer | 状态：`1=进行中`，`2=待开始`，`3=待开奖`，`4=已开奖` |
| `statusName` | String | 状态名称 |
| `drawResult` | Integer | 开奖结果：`1=正方胜`，`2=反方胜` |
| `incomeAmount` | String | 收益金额，`status=4` 时有效 |

---

## 4. 查询期次详情

### 接口信息
| 项 | 内容 |
|---|---|
| 接口名称 | 查询期次详情 |
| 请求地址 | `/v1/oracle/issue/info` |
| 请求方式 | `GET` |
| Token | 选填 |

### 业务说明
未登录时 `isParticipate` 默认返回 `false`，`balance` 默认返回 `0`。

### 请求参数
| 字段 | 类型 | 是否必填 | 说明 |
|---|---|---|---|
| `id` | String | 是 | 期次ID |

### 响应数据
`data` 为 `OracleIssueInfoRes`

| 字段 | 类型 | 说明 |
|---|---|---|
| `id` | String | 期次ID |
| `issueCode` | String | 期次编码 |
| `symbol` | String | 币对 |
| `baseCoin` | String | 基础币 |
| `beginTime` | String | 开始时间 |
| `endTime` | String | 结束时间 |
| `openPrice` | String | 基准价 |
| `closePrice` | String | 结束价(当前价) |
| `startAdvanceMinutes` | Integer | 参与开始提前分钟数 |
| `cutoffAdvanceMinutes` | Integer | 参与截止提前分钟数 |
| `marketType` | Integer | 市场类型 |
| `status` | Integer | 状态：`1=进行中`，`2=待开始`，`3=待开奖`，`4=已开奖` |
| `statusName` | String | 状态名称 |
| `feeRate` | String | 手续费率 |
| `totalEffectiveAmount` | String | 有效参与金额 |
| `positiveAmount` | String | 正方(涨方)金额 |
| `negativeAmount` | String | 反方(跌方)金额 |
| `limitMode` | Integer | 限制模式：`1=仅1次`，`2=不限制` |
| `balance` | String | 用户资产余额(USDT) |
| `isParticipate` | Boolean | 是否参与：`true=是`，`false=否` |

---

## 5. 支付

### 接口信息
| 项 | 内容 |
|---|---|
| 接口名称 | 支付 |
| 请求地址 | `/v1/oracle/pay` |
| 请求方式 | `POST` |
| Token | 是 |

### 请求参数
| 字段 | 类型 | 是否必填 | 说明 |
|---|---|---|---|
| `id` | String | 是 | 期次ID |
| `choiceValue` | Integer | 是 | 选择值：`1=涨`，`2=跌` |
| `amount` | BigDecimal | 是 | 金额 |
| `safetyPassword` | String | 是 | 资金密码，加密传输，处理方式参考 `MineController.buyElect` |

### 响应数据
`data` 为 `Boolean`

| 字段 | 类型 | 说明 |
|---|---|---|
| `data` | Boolean | 成功返回 `true` |

### 业务说明
支付失败直接返回业务错误码和错误信息。
支付币种按 `USDT` 处理。
支付前需校验资金密码，处理方式参考 `MineController.buyElect`。
仅 `status=1(进行中)`、`status=2(待开始)` 的期次允许支付。
支付时需校验 `minPayAmount/maxPayAmount`、`limitMode`、参与时间窗口以及 `maxHoldRatio`。
`maxHoldRatio` 按本次提交后的 `effectiveAmount` 校验，不按当前用户整期期次累计占比处理。
当期次当前 `totalEffectiveAmount <= 0` 时，本次不做 `maxHoldRatio` 拦截，避免首笔订单无法参与。
支付成功后除落库 `oracle_order` 外，还需同步回写 `oracle_issue` 聚合字段。
支付成功后还需同步维护 `account_oracle` 用户汇总。
若当前用户不存在汇总记录，则新增一条。
`totalPayAmount += amount`
`totalFeeAmount += fee`
`totalIncomeAmount` 支付时不变，待开奖结算后再处理。
`totalProfitAmount` 支付时不变，待开奖结算后再处理。
钱包账单和资产流水使用 Oracle 专用分类，错误场景使用 Oracle 专用错误码。

---

## 6. 查询收益统计

### 接口信息
| 项 | 内容 |
|---|---|
| 接口名称 | 查询收益统计 |
| 请求地址 | `/v1/oracle/income/total` |
| 请求方式 | `POST` |
| Token | 是 |

### 请求参数
无

### 响应数据
`data` 为 `OracleIncomeTotalRes`

| 字段 | 类型 | 说明 |
|---|---|---|
| `totalIncomeAmount` | String | 累计收益 |
| `totalProfitAmount` | String | 累计盈利 |
| `waitDrawNum` | String | 待开奖数量 |

---

## 7. 查询期次状态

### 接口信息
| 项 | 内容 |
|---|---|
| 接口名称 | 查询期次状态 |
| 请求地址 | `/v1/oracle/issue/status/select` |
| 请求方式 | `GET` |
| Token | 否 |

### 业务说明
根据 `OracleIssueStatusEnum` 返回期次状态选择列表。
`label` 按请求头 `Language` 对应语言国际化返回。
按需求约定，该接口不返回 `2=待开始`。

### 请求参数
无

### 响应数据
`data` 为 `List<SelectRes>`

| 字段 | 类型 | 说明 |
|---|---|---|
| `value` | String | 状态值 |
| `label` | String | 状态名称 |

---

## 8. 查询收益记录

### 接口信息
| 项 | 内容 |
|---|---|
| 接口名称 | 查询收益记录 |
| 请求地址 | `/v1/oracle/income/list` |
| 请求方式 | `GET` |
| Token | 是 |

### 请求参数
| 字段 | 类型 | 是否必填 | 说明 |
|---|---|---|---|
| `pageNo` | Integer | 否 | 页码，默认 `1` |
| `pageSize` | Integer | 否 | 每页数量，默认 `10` |
| `issueId` | String | 否 | 期次ID |
| `status` | Integer | 否 | 状态，取值通过“查询期次状态”接口获取 |

### 响应数据
`data` 为 `PageResult<OracleIncomeListRes>`

| 字段 | 类型 | 说明 |
|---|---|---|
| `id` | String | 期次ID |
| `issueCode` | String | 期次编码 |
| `symbol` | String | 交易对 |
| `baseCoin` | String | 基础币 |
| `marketType` | Integer | 市场类型 |
| `openPrice` | String | 基准价 |
| `closePrice` | String | 结束价 |
| `totalEffectiveAmount` | String | 净清算池 |
| `beginTime` | String | 期次开始时间 |
| `endTime` | String | 期次结束时间 |
| `status` | Integer | 状态：`1=进行中`，`2=待开始`，`3=待开奖`，`4=已开奖` |
| `upAmount` | String | 当前用户参与的 `UP/YES` 有效金额 |
| `downAmount` | String | 当前用户参与的 `DOWN/NO` 有效金额 |
| `totalProfitAmount` | String | 当前期次盈利总额金额 |
| `incomeAmount` | String | 当前期次收益总额金额 |

---

## 9. 查询订单状态

### 接口信息
| 项 | 内容 |
|---|---|
| 接口名称 | 查询订单状态 |
| 请求地址 | `/v1/oracle/order/status/select` |
| 请求方式 | `GET` |
| Token | 否 |

### 业务说明
根据 `OracleOrderStatusEnum` 返回订单状态选择列表。
`label` 按请求头 `Language` 对应语言国际化返回。
当前订单状态枚举仅包含 `1=待开奖`、`2=已结算`、`3=未中奖`。

### 请求参数
无

### 响应数据
`data` 为 `List<SelectRes>`

| 字段 | 类型 | 说明 |
|---|---|---|
| `value` | String | 状态值 |
| `label` | String | 状态名称 |

---

## 10. 查询付款记录

### 接口信息
| 项 | 内容 |
|---|---|
| 接口名称 | 查询付款记录 |
| 请求地址 | `/v1/oracle/order/list` |
| 请求方式 | `GET` |
| Token | 是 |

### 业务说明
查询当前登录用户的付款记录。
主表为 `oracle_order`，用于订单状态筛选时请配合“查询订单状态”接口。
SQL 表连接不超过 3 层，且不在 SQL 中处理业务数据。

### 请求参数
| 字段 | 类型 | 是否必填 | 说明 |
|---|---|---|---|
| `pageNo` | Integer | 否 | 页码，默认 `1` |
| `pageSize` | Integer | 否 | 每页数量，默认 `10` |
| `status` | Integer | 否 | 状态：`1=待开奖`，`2=已结算`，`3=未中奖` |
| `issueId` | String | 否 | 期次ID |

### 响应数据
`data` 为 `PageResult<OracleOrderListRes>`

| 字段 | 类型 | 说明 |
|---|---|---|
| `id` | String | 订单ID |
| `orderNo` | String | 订单号 |
| `symbol` | String | 交易对 |
| `baseCoin` | String | 基础币 |
| `marketType` | Integer | 市场类型 |
| `status` | Integer | 状态 |
| `statusName` | String | 状态名称 |
| `choiceValue` | Integer | 用户选择：`1=UP/YES`，`2=DOWN/NO` |
| `amount` | String | 订单金额 |
| `effectiveAmount` | String | 有效参与金额 |
| `createdTime` | String | 创建时间(参与时间) |
| `issueId` | String | 期次ID |
| `issueCode` | String | 期次编码 |
