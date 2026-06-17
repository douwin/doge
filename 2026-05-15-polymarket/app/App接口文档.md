# App接口文档

## 通用说明

### 接口前缀
`/v1/oracle`

### 请求方式
除特殊说明外，统一使用 `POST`

### 请求头
| 名称 | 是否必填 | 类型 | 说明 |
|---|---|---|---|
| `Language` | 否 | String | 语言，支持 `zh-CN`、`en-US`、`vi-VN`、`id-ID` |
| `Token` | 按接口说明 | String | 登录凭证 |

### 通用响应结构
| 字段 | 类型 | 说明 |
|---|---|---|
| `success` | Boolean | 是否成功 |
| `ts` | String | 服务端当前秒级时间戳 |
| `data` | Object | 响应数据 |
| `code` | Integer | 响应码，成功为 `200` |
| `msg` | String | 响应描述，成功为 `成功` |

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
| 时间字段 | 返回毫秒时间戳字符串 |
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

### 请求示例
```http
GET /v1/oracle/symbol/select
Language: zh-CN
```

### 响应数据
`data` 为 `List<SelectRes>`

| 字段 | 类型 | 说明 |
|---|---|---|
| `value` | String | 交易对编码 |
| `label` | String | 基础币名称 |

### 响应示例
```json
{
  "success": true,
  "ts": "1750062000",
  "data": [
    {
      "value": "DOGEUSDT",
      "label": "DOGE"
    },
    {
      "value": "BTCUSDT",
      "label": "BTC"
    }
  ],
  "code": 200,
  "msg": "成功"
}
```

---

## 2. 查询推荐期次

### 接口信息
| 项 | 内容 |
|---|---|
| 接口名称 | 查询推荐期次 |
| 请求地址 | `/v1/oracle/issue/recommend/list` |
| 请求方式 | `GET` |
| Token | 否 |

### 请求参数
| 字段 | 类型 | 是否必填 | 说明 |
|---|---|---|---|
| `symbol` | String | 否 | 币对 |

### 请求示例
```http
GET /v1/oracle/issue/recommend/list?symbol=DOGEUSDT
Language: zh-CN
```

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
| `buyBeginTime` | String | 购买开始时间 |
| `buyEndTime` | String | 购买结束时间 |
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

### 响应示例
```json
{
  "success": true,
  "ts": "1750062000",
  "data": [
    {
      "id": "1001",
      "issueCode": "DOGE-1H-2026061509",
      "symbol": "DOGEUSDT",
      "baseCoin": "DOGE",
      "beginTime": "1750045200000",
      "endTime": "1750048800000",
      "buyBeginTime": "1750044300000",
      "buyEndTime": "1750048500000",
      "openPrice": "0.186200000000000000",
      "closePrice": "0.186900000000000000",
      "startAdvanceMinutes": 15,
      "cutoffAdvanceMinutes": 5,
      "marketType": 1,
      "status": 1,
      "statusName": "进行中",
      "isRecommended": 1,
      "totalEffectiveAmount": "1250.360000000000000000",
      "positiveAmount": "760.120000000000000000",
      "negativeAmount": "490.240000000000000000"
    }
  ],
  "code": 200,
  "msg": "成功"
}
```

---

## 3. 查询所有期次

### 接口信息
| 项 | 内容 |
|---|---|
| 接口名称 | 查询所有期次 |
| 请求地址 | `/v1/oracle/issue/list` |
| 请求方式 | `GET` |
| Token | 选填 |

### 请求参数
| 字段 | 类型 | 是否必填 | 说明 |
|---|---|---|---|
| `pageNo` | Integer | 否 | 页码，默认 `1` |
| `pageSize` | Integer | 否 | 每页数量，默认 `10` |
| `status` | Integer | 否 | 状态：`1=进行中`，`2=待开始`，`3=待开奖`，`4=已开奖` |
| `symbol` | String | 否 | 币对 |

### 请求示例
```http
GET /v1/oracle/issue/list?pageNo=1&pageSize=10&status=4&symbol=DOGEUSDT
Language: zh-CN
Token: xxxxx
```

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
| `buyBeginTime` | String | 购买开始时间 |
| `buyEndTime` | String | 购买结束时间 |
| `openPrice` | String | 基准价 |
| `closePrice` | String | 结束价(当前价) |
| `startAdvanceMinutes` | Integer | 参与开始提前分钟数 |
| `cutoffAdvanceMinutes` | Integer | 参与截止提前分钟数 |
| `marketType` | Integer | 市场类型 |
| `status` | Integer | 状态：`1=进行中`，`2=待开始`，`3=待开奖`，`4=已开奖` |
| `statusName` | String | 状态名称 |
| `drawResult` | Integer | 开奖结果：`1=正方胜`，`2=反方胜`，`status=4` 时有效，否则返回空 |
| `incomeAmount` | String | 收益金额，已登录且 `status=4` 时有效，否则返回空 |

### 响应示例
```json
{
  "success": true,
  "ts": "1750062000",
  "data": {
    "pageNo": 1,
    "pageSize": 10,
    "totalSize": "2",
    "totalPages": 1,
    "prevPage": 1,
    "nextPage": 1,
    "list": [
      {
        "id": "1001",
        "issueCode": "DOGE-1H-2026061509",
        "symbol": "DOGEUSDT",
        "baseCoin": "DOGE",
        "beginTime": "1750045200000",
        "endTime": "1750048800000",
        "buyBeginTime": "1750044300000",
        "buyEndTime": "1750048500000",
        "openPrice": "0.186200000000000000",
        "closePrice": "0.187500000000000000",
        "startAdvanceMinutes": 15,
        "cutoffAdvanceMinutes": 5,
        "marketType": 1,
        "status": 4,
        "statusName": "已开奖",
        "drawResult": 1,
        "incomeAmount": "32.560000000000000000"
      }
    ]
  },
  "code": 200,
  "msg": "成功"
}
```

---

## 4. 查询期次详情

### 接口信息
| 项 | 内容 |
|---|---|
| 接口名称 | 查询期次详情 |
| 请求地址 | `/v1/oracle/issue/info` |
| 请求方式 | `GET` |
| Token | 选填 |

### 请求参数
| 字段 | 类型 | 是否必填 | 说明 |
|---|---|---|---|
| `id` | String | 是 | 期次ID |

### 请求示例
```http
GET /v1/oracle/issue/info?id=1001
Language: zh-CN
Token: xxxxx
```

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
| `allocType` | Integer | 净清算池类型 |
| `allocTypeName` | String | 净清算池名称 |
| `buyBeginTime` | String | 购买开始时间 |
| `buyEndTime` | String | 购买结束时间 |
| `buyInfo` | List | 购买信息 |
| `buyInfo[].choiceValue` | Integer | 涨/跌：`1=涨`，`2=跌` |
| `buyInfo[].amount` | String | 数量 |
| `balance` | String | 用户资产余额(USDT)，未登录时返回空 |
| `isParticipate` | Boolean | 是否参与：`true=是`，`false=否` |

### 响应示例
```json
{
  "success": true,
  "ts": "1750062000",
  "data": {
    "id": "1001",
    "issueCode": "DOGE-1H-2026061509",
    "symbol": "DOGEUSDT",
    "baseCoin": "DOGE",
    "beginTime": "1750045200000",
    "endTime": "1750048800000",
    "openPrice": "0.186200000000000000",
    "closePrice": "0.186900000000000000",
    "startAdvanceMinutes": 15,
    "cutoffAdvanceMinutes": 5,
    "marketType": 1,
    "status": 1,
    "statusName": "进行中",
    "feeRate": "0.050000000000000000",
    "totalEffectiveAmount": "1250.360000000000000000",
    "positiveAmount": "760.120000000000000000",
    "negativeAmount": "490.240000000000000000",
    "limitMode": 2,
    "allocType": 1,
    "allocTypeName": "胜方按有效金额占比分配",
    "buyBeginTime": "1750044300000",
    "buyEndTime": "1750048500000",
    "buyInfo": [
      {
        "choiceValue": 1,
        "amount": "120.000000000000000000"
      }
    ],
    "balance": "568.230000000000000000",
    "isParticipate": true
  },
  "code": 200,
  "msg": "成功"
}
```

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
| `amount` | BigDecimal | 是 | 金额，最多 `2` 位小数 |
| `safetyPassword` | String | 是 | 资金密码，加密传输，处理方式参考 `MineController.buyElect` |

### 请求示例
```json
POST /v1/oracle/pay
Language: zh-CN
Token: xxxxx

{
  "id": "1001",
  "choiceValue": 1,
  "amount": 100.00,
  "safetyPassword": "encrypted-password"
}
```

### 响应数据
`data` 为 `Boolean`

| 字段 | 类型 | 说明 |
|---|---|---|
| `data` | Boolean | 成功返回 `true` |

### 响应示例
```json
{
  "success": true,
  "ts": "1750062000",
  "data": true,
  "code": 200,
  "msg": "成功"
}
```

---

## 6. 查询收益统计

### 接口信息
| 项 | 内容 |
|---|---|
| 接口名称 | 查询收益统计 |
| 请求地址 | `/v1/oracle/income/total` |
| 请求方式 | `GET` |
| Token | 是 |

### 请求参数
无

### 请求示例
```http
GET /v1/oracle/income/total
Language: zh-CN
Token: xxxxx
```

### 响应数据
`data` 为 `OracleIncomeTotalRes`

| 字段 | 类型 | 说明 |
|---|---|---|
| `totalIncomeAmount` | String | 累计收益 |
| `totalProfitAmount` | String | 累计盈利 |
| `waitDrawNum` | String | 待开奖期次数量，同一期多笔订单算 `1` |

### 响应示例
```json
{
  "success": true,
  "ts": "1750062000",
  "data": {
    "totalIncomeAmount": "286.520000000000000000",
    "totalProfitAmount": "120.180000000000000000",
    "waitDrawNum": "3"
  },
  "code": 200,
  "msg": "成功"
}
```

---

## 7. 查询期次状态

### 接口信息
| 项 | 内容 |
|---|---|
| 接口名称 | 查询期次状态 |
| 请求地址 | `/v1/oracle/issue/status/select` |
| 请求方式 | `GET` |
| Token | 否 |

### 请求参数
无

### 请求示例
```http
GET /v1/oracle/issue/status/select
Language: zh-CN
```

### 响应数据
`data` 为 `List<SelectRes>`

| 字段 | 类型 | 说明 |
|---|---|---|
| `value` | String | 状态值 |
| `label` | String | 状态名称 |

### 响应示例
```json
{
  "success": true,
  "ts": "1750062000",
  "data": [
    {
      "value": "1",
      "label": "进行中"
    },
    {
      "value": "3",
      "label": "待开奖"
    },
    {
      "value": "4",
      "label": "已开奖"
    }
  ],
  "code": 200,
  "msg": "成功"
}
```

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

### 请求示例
```http
GET /v1/oracle/income/list?pageNo=1&pageSize=10&status=4
Language: zh-CN
Token: xxxxx
```

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
| `statusName` | String | 状态名称（新增，按语言国际化返回） |
| `positiveAmount` | String | 正方(涨方)金额 |
| `negativeAmount` | String | 反方(跌方)金额 |
| `positiveRate` | String | 正方(涨方)比例（新增） |
| `negativeRate` | String | 反方(跌方)比例（新增） |
| `winnerRate` | String | 胜方比例（新增，仅已开奖时返回） |
| `buyInfo` | List | 购买信息 |
| `buyInfo[].choiceValue` | Integer | 涨/跌：`1=涨`，`2=跌` |
| `buyInfo[].amount` | String | 数量 |
| `totalProfitAmount` | String | 当前期次盈利总额金额 |
| `incomeAmount` | String | 当前期次收益总额金额 |

### 响应示例
```json
{
  "success": true,
  "ts": "1750062000",
  "data": {
    "pageNo": 1,
    "pageSize": 10,
    "totalSize": "1",
    "totalPages": 1,
    "prevPage": 1,
    "nextPage": 1,
    "list": [
      {
        "id": "1001",
        "issueCode": "DOGE-1H-2026061509",
        "symbol": "DOGEUSDT",
        "baseCoin": "DOGE",
        "marketType": 1,
        "openPrice": "0.1862",
        "closePrice": "0.1875",
        "totalEffectiveAmount": "1250.36",
        "beginTime": "1750045200000",
        "endTime": "1750048800000",
        "status": 4,
        "statusName": "已开奖",
        "positiveAmount": "760.12",
        "negativeAmount": "490.24",
        "positiveRate": "0.60792091",
        "negativeRate": "0.39207908",
        "winnerRate": "0.60792091",
        "buyInfo": [
          {
            "choiceValue": 1,
            "amount": "120"
          }
        ],
        "totalProfitAmount": "32.56",
        "incomeAmount": "152.56"
      }
    ]
  },
  "code": 200,
  "msg": "成功"
}
```

---

## 9. 查询订单状态

### 接口信息
| 项 | 内容 |
|---|---|
| 接口名称 | 查询订单状态 |
| 请求地址 | `/v1/oracle/order/status/select` |
| 请求方式 | `GET` |
| Token | 否 |

### 请求参数
无

### 请求示例
```http
GET /v1/oracle/order/status/select
Language: zh-CN
```

### 响应数据
`data` 为 `List<SelectRes>`

| 字段 | 类型 | 说明 |
|---|---|---|
| `value` | String | 状态值 |
| `label` | String | 状态名称 |

### 响应示例
```json
{
  "success": true,
  "ts": "1750062000",
  "data": [
    {
      "value": "1",
      "label": "待开奖"
    },
    {
      "value": "2",
      "label": "已结算"
    },
    {
      "value": "3",
      "label": "未中奖"
    }
  ],
  "code": 200,
  "msg": "成功"
}
```

---

## 10. 查询付款记录

### 接口信息
| 项 | 内容 |
|---|---|
| 接口名称 | 查询付款记录 |
| 请求地址 | `/v1/oracle/order/list` |
| 请求方式 | `GET` |
| Token | 是 |

### 请求参数
| 字段 | 类型 | 是否必填 | 说明 |
|---|---|---|---|
| `pageNo` | Integer | 否 | 页码，默认 `1` |
| `pageSize` | Integer | 否 | 每页数量，默认 `10` |
| `status` | Integer | 否 | 状态：`1=待开奖`，`2=已结算`，`3=未中奖` |
| `issueId` | String | 否 | 期次ID |

### 请求示例
```http
GET /v1/oracle/order/list?pageNo=1&pageSize=10&status=1&issueId=1001
Language: zh-CN
Token: xxxxx
```

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
| `profitAmount` | String | 盈利金额（新增） |
| `incomeAmount` | String | 收益金额（新增） |

### 响应示例
```json
{
  "success": true,
  "ts": "1750062000",
  "data": {
    "pageNo": 1,
    "pageSize": 10,
    "totalSize": "1",
    "totalPages": 1,
    "prevPage": 1,
    "nextPage": 1,
    "list": [
      {
        "id": "9001",
        "orderNo": "PM202606150001",
        "symbol": "DOGEUSDT",
        "baseCoin": "DOGE",
        "marketType": 1,
        "status": 1,
        "statusName": "待开奖",
        "choiceValue": 1,
        "amount": "100",
        "effectiveAmount": "95",
        "createdTime": "1750046100000",
        "issueId": "1001",
        "issueCode": "DOGE-1H-2026061509",
        "profitAmount": "0",
        "incomeAmount": "0"
      }
    ]
  },
  "code": 200,
  "msg": "成功"
}
```

---

## 11. 查询K线

### 接口信息
| 项 | 内容 |
|---|---|
| 接口名称 | 查询K线 |
| 请求地址 | `/v1/oracle/kline` |
| 请求方式 | `GET` |
| Token | 否 |

### 请求参数
| 字段 | 类型 | 是否必填 | 说明 |
|---|---|---|---|
| `issueId` | String | 是 | 期次ID |
| `limit` | Integer | 是 | 返回数量 |
| `time` | String | 否 | 上一页最后一条K线时间戳(毫秒)（新增，不为空时查询 `lastTime < time` 的历史数据） |

### 请求示例
```http
GET /v1/oracle/kline?issueId=1001&limit=20&time=1750048800000
Language: zh-CN
```

### 响应数据
`data` 为 `OracleKlineRes`

| 字段 | 类型 | 说明 |
|---|---|---|
| `symbol` | String | 币对 |
| `cycle` | String | K线周期 |
| `data` | List | K线数据 |
| `data[].price` | String | 价格 |
| `data[].time` | String | 时间，返回 `lastTime` |

### 响应示例
```json
{
  "success": true,
  "ts": "1750062000",
  "data": {
    "symbol": "DOGEUSDT",
    "cycle": "1h",
    "data": [
      {
        "price": "0.1862",
        "time": "1750045200000"
      },
      {
        "price": "0.1869",
        "time": "1750047000000"
      },
      {
        "price": "0.1875",
        "time": "1750048800000"
      }
    ]
  },
  "code": 200,
  "msg": "成功"
}
```

---

## 12. 查询系统配置

### 接口信息
| 项 | 内容 |
|---|---|
| 接口名称 | 查询系统配置 |
| 请求地址 | `/v1/oracle/config` |
| 请求方式 | `POST` |
| Token | 否 |

### 请求参数
| 字段 | 类型 | 是否必填 | 说明 |
|---|---|---|---|
| `configKey` | String | 是 | 配置KEY，盘口背景=`oracle_market_depth_background`，盘口规则=`oracle_market_depth_rule` |

### 请求示例
```json
POST /v1/oracle/config
Language: zh-CN

{
  "configKey": "oracle_market_depth_background"
}
```

### 响应数据
`data` 为 `Object`

| 字段 | 类型 | 说明 |
|---|---|---|
| `data` | Object | 配置内容，按 `sys_config.cfg_value` 的 JSON 结构直接返回 |

### 响应示例
```json
{
  "success": true,
  "ts": "1750062000",
  "data": {
    "title": "盘口背景：本期是自动生成的外部行情事件。用戶只选择涨或跌，平台不作为对手方，不承诺固定赔率，也不参与开奖结果。"
  },
  "code": 200,
  "msg": "成功"
}
```

```json
{
  "success": true,
  "ts": "1750062000",
  "data": {
    "title": "盘口背景：本期是自动生成的外部行情事件。用戶只选择涨或跌，平台不作为对手方，不承诺固定赔率，也不参与开奖结果。",
    "items": [
      {
        "key": "evidenceCaliber",
        "value": "K线 open/close"
      },
      {
        "key": "closingTime",
        "value": "观察前{0}分钟"
      },
      {
        "key": "netClearingPool",
        "value": "{1}"
      }
    ]
  },
  "code": 200,
  "msg": "成功"
}
```
