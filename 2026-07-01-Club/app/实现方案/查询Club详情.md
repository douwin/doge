# 查询Club详情

## 接口信息
| 项 | 内容 |
|---|---|
| 请求地址 | `/api/v1/club/info` |
| 请求方式 | `GET` |
| Token | 选填 |

## 业务说明
1. 返回 Club 商品信息、权益配置、优惠信息、余额信息以及当前用户的购买/续费状态。
2. 未登录时可访问，此时 `balance=0`、`isBuy=false`、`memberInfo=null`。

## 请求参数
| 字段 | 类型 | 是否必填 | 说明 |
|---|---|---|---|
| `clubConfigId` | String | 是 | Club配置ID |

## 请求示例
```http
GET /api/v1/club/info?clubConfigId=10001
Language: zh_CN
Token: eyJhbGciOiJIUzI1NiJ9.demo
```

## 响应数据
`data` 为对象

| 字段 | 类型 | 说明 |
|---|---|---|
| `clubConfigId` | String | Club配置ID |
| `name` | String | Club名称 |
| `description` | String | Club描述 |
| `benefitList` | Array | 权益集合 |
| `originalFee` | String | 原价，取 `club_config.admission_fee` |
| `discountFee` | String | 优惠价，当前实现按 `originalFee * (discountRate / 10)` 计算 |
| `hasDiscount` | Boolean | 是否有可用优惠 |
| `discountInfo` | Object | 折扣信息，无优惠时返回 `null` |
| `annualFee` | String | 年费，取 `club_config.annual_fee` |
| `payCoinId` | String | 支付币种ID |
| `payCoinName` | String | 支付币种名称 |
| `balance` | String | 当前用户可用余额，未登录返回 `0` |
| `purchaseAvailable` | Boolean | 是否可购买 |
| `purchaseUnavailableReason` | String | 不可购买原因 |
| `isPreRenew` | Boolean | 是否进入提前续费窗口 |
| `preRenewDays` | Integer | 提前续费天数 |
| `preRenewDate` | String | 提前续费开始时间戳(毫秒) |
| `isGracePeriod` | Boolean | 是否处于宽限期 |
| `gracePeriodDays` | Integer | 宽限期天数 |
| `gracePeriodDate` | String | 宽限期结束时间戳(毫秒) |
| `deadline` | Integer | 会员期限，单位：年 |
| `isBuy` | Boolean | 是否已购买 |
| `memberInfo` | Object | 会员信息，未购买时返回 `null` |

`benefitList` 元素结构：

| 字段 | 类型 | 说明 |
|---|---|---|
| `name` | String | 权益名称 |
| `unit` | String | 单位 |
| `num` | String | 展示文案，按权益类型格式化后的数量 |
| `remark` | String | 权益备注 |

`discountInfo` 结构：

| 字段 | 类型 | 说明 |
|---|---|---|
| `discountQuota` | String | 折扣总名额 |
| `remainingQuota` | String | 剩余优惠名额 |
| `discountRate` | String | 折扣原始值，例如 `8` 表示 `8折` |

`memberInfo` 结构：

| 字段 | 类型 | 说明 |
|---|---|---|
| `memberId` | String | 会员ID |
| `memberNo` | String | 会员卡号 |
| `validStart` | String | 生效开始时间戳(毫秒) |
| `validEnd` | String | 生效结束时间戳(毫秒) |
| `status` | Integer | 会员状态：`1=正常`，`2=已超期`，`3=已冻结`，`4=已停用`，`5=已失效` |
| `statusName` | String | 会员状态名称 |

## 响应示例
```json
{
  "success": true,
  "ts": "1784592001123",
  "code": 200,
  "msg": "success",
  "data": {
    "clubConfigId": "10001",
    "name": "DOGElon Club",
    "description": "加入 DOGElon Club，获得多项会员权益。",
    "benefitList": [
      {
        "name": "ELON代币",
        "unit": "ELON",
        "num": "10000 ELON",
        "remark": "支付成功后发放"
      },
      {
        "name": "高级权益票证",
        "unit": "张",
        "num": "1 张高级权益票证",
        "remark": "具体活动规则以通知为准"
      }
    ],
    "originalFee": "1000",
    "discountFee": "800",
    "hasDiscount": true,
    "discountInfo": {
      "discountQuota": "100",
      "remainingQuota": "12",
      "discountRate": "8"
    },
    "annualFee": "100",
    "payCoinId": "1",
    "payCoinName": "USDT",
    "balance": "2500.35",
    "purchaseAvailable": false,
    "purchaseUnavailableReason": "一人一份",
    "isPreRenew": true,
    "preRenewDays": 30,
    "preRenewDate": "1811808000000",
    "isGracePeriod": false,
    "gracePeriodDays": 15,
    "gracePeriodDate": "1815696000000",
    "deadline": 1,
    "isBuy": true,
    "memberInfo": {
      "memberId": "20001",
      "memberNo": "DGC100001",
      "validStart": "1782864000000",
      "validEnd": "1814400000000",
      "status": 1,
      "statusName": "正常"
    }
  }
}
```

## 校验说明
1. `clubConfigId` 不能为空。
2. `clubConfigId` 必须可转为数值，否则返回参数校验失败。
3. `clubConfig` 不存在，或状态不是启用时，返回 `CLUB_NOT_EXISTS`。

## 复杂业务说明
1. `benefitList` 来源于 `club_config_benefit`，当前实现不按 `status` 过滤，排序规则为 `benefitType asc, id asc`。
2. `purchaseAvailable` 当前有两种明确的不可购买原因：
   - 当前账号已有一张按规则不可再次新购的会员卡，返回 `false`，原因固定为 `一人一份`。
   - `club_config.used_quota >= club_config.max_quota`，返回 `false`，原因固定为 `当前Club已售罄`。
3. `isBuy` 的判断规则与“查询已启用的Club”一致，不是简单判断是否存在会员记录。
4. `isPreRenew` 计算逻辑：
   - 仅在当前账号已购且 `validEnd` 不为空时计算。
   - `preRenewDate = validEnd - preRenewDays`。
   - 当 `当前时间 >= preRenewDate` 且 `当前时间 <= validEnd` 时返回 `true`。
5. `isGracePeriod` 计算逻辑：
   - 仅在当前账号已购且 `validEnd` 不为空时计算。
   - `gracePeriodDate = validEnd + gracePeriodDays`。
   - 当 `当前时间 > validEnd` 且 `当前时间 <= gracePeriodDate` 时返回 `true`。
6. 折扣相关规则：
   - 折扣换算公式：`discountRate / 10`，例如 `8` 表示 `8折`。
   - `hasDiscount=true` 需满足存在启用折扣、优惠名额未用尽、并且优惠价小于原价。
   - 当前实现中 `discountFee` 只要存在启用折扣配置就会按折扣值计算；前端如需判断是否展示优惠，应以 `hasDiscount` 为准。
7. `balance` 仅在已登录且能查询到支付币种资产时返回实际余额，否则返回 `0`。

## 失败码
| 错误码 | 说明 |
|---|---|
| `10001` | 参数校验失败 |
| `10049` | Club不存在 |
