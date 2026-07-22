# App接口文档

## 通用说明

### 接口前缀
`/api/v1/club`

### 请求方式
查询接口统一使用 `GET`，提交类接口统一使用 `POST`

### 请求头
| 名称 | 是否必填 | 类型 | 说明 |
|---|---|---|---|
| `Language` | 否 | String | 语言，默认 `zh_CN` |
| `Token` | 按接口说明 | String | 登录凭证 |

### 通用响应结构
| 字段 | 类型 | 说明 |
|---|---|---|
| `success` | Boolean | 是否成功 |
| `ts` | Long | 服务端当前时间戳 |
| `data` | Object | 响应数据 |
| `code` | Integer | 响应码，成功为 `200` |
| `msg` | String | 响应描述 |

### 分页响应结构
`data` 为分页对象时，结构如下：

| 字段 | 类型 | 说明 |
|---|---|---|
| `pageNo` | Integer | 当前页码 |
| `pageSize` | Integer | 每页数量 |
| `totalSize` | Long | 总记录数 |
| `totalPages` | Integer | 总页数 |
| `prevPage` | Integer | 上一页 |
| `nextPage` | Integer | 下一页 |
| `list` | Array | 列表数据 |

### 字段说明
| 规则 | 说明 |
|---|---|
| 时间字段 | 返回时间戳(毫秒) |
| `Long` 字段 | 返回 String |
| `BigDecimal` 字段 | 返回 String |
| 请求参数 | 除 `pageNo`、`pageSize`、状态码等整型字段外，其他统一使用 String |
| 命名风格 | 请求参数、响应字段统一使用驼峰命名 |

---

## 1. 查询已启用的Club

### 接口信息
| 项 | 内容 |
|---|---|
| 接口名称 | 查询已启用的Club |
| 请求地址 | `/api/v1/club/enable/list` |
| 请求方式 | `GET` |
| Token | 选填 |

### 业务说明
当前仅返回一个已启用的 Club 对象。未登录时 `isBuy=false`、`memberInfo=null`。有优惠时返回 `hasDiscount=true` 且携带 `discountInfo`。

### 请求参数
无

### 请求示例
```http
GET /api/v1/club/enable/list
Language: zh_CN
Token: eyJhbGciOiJIUzI1NiJ9.demo
```

### 响应数据
`data` 为 `ClubEnabledRes`

| 字段 | 类型 | 说明 |
|---|---|---|
| `clubConfigId` | String | Club配置ID |
| `name` | String | Club名称 |
| `description` | String | Club描述 |
| `benefitList` | Array | 权益集合 |
| `isBuy` | Boolean | 是否已购买 |
| `hasDiscount` | Boolean | 是否有优惠 |
| `discountInfo` | Object | 折扣信息，无优惠时返回 `null` |
| `memberInfo` | Object | 会员信息，未购买时返回 `null` |

`benefitList` 元素结构：

| 字段 | 类型 | 说明 |
|---|---|---|
| `name` | String | 权益名称 |
| `unit` | String | 单位 |
| `num` | String | 原始数量，取 `club_config_benefit.config_num` |
| `displayName` | String | 展示文案，按权益类型格式化后的数量 |

`discountInfo` 结构：

| 字段 | 类型 | 说明 |
|---|---|---|
| `discountQuota` | String | 折扣名额 |
| `remainingQuota` | String | 剩余优惠名额 |
| `discountRate` | String | 折扣，返回原始配置值 |

`memberInfo` 结构：

| 字段 | 类型 | 说明 |
|---|---|---|
| `memberId` | String | 会员ID |
| `memberNo` | String | 会员卡号 |
| `validStart` | String | 有效期开始时间戳(毫秒) |
| `validEnd` | String | 有效期结束时间戳(毫秒) |
| `createTime` | String | 开通日期时间戳(毫秒) |

### 响应示例
```json
{
  "success": true,
  "ts": 1784592000123,
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
        "num": "10000",
        "displayName": "10000 ELON"
      },
      {
        "name": "高级权益票证",
        "unit": "张",
        "num": "1",
        "displayName": "1 张高级权益票证"
      }
    ],
    "isBuy": true,
    "hasDiscount": true,
    "discountInfo": {
      "discountQuota": "100",
      "remainingQuota": "12",
      "discountRate": "8"
    },
    "memberInfo": {
      "memberId": "20001",
      "memberNo": "DGC100001",
      "validStart": "1782864000000",
      "validEnd": "1814400000000",
      "createTime": "1782864000000"
    }
  }
}
```

### 业务说明
分页记录按 `benefitType asc, createTime desc, id desc` 排序返回。

---

## 2. 查询Club详情

### 接口信息
| 项 | 内容 |
|---|---|
| 接口名称 | 查询Club详情 |
| 请求地址 | `/api/v1/club/info` |
| 请求方式 | `GET` |
| Token | 选填 |

### 业务说明
返回 Club 商品信息、优惠信息、余额信息、购买状态和会员状态。未登录时 `balance=0`、`isBuy=false`、`memberInfo=null`。

### 请求参数
| 字段 | 类型 | 是否必填 | 说明 |
|---|---|---|---|
| `clubConfigId` | String | 是 | Club配置ID |

### 请求示例
```http
GET /api/v1/club/info?clubConfigId=10001
Language: zh_CN
Token: eyJhbGciOiJIUzI1NiJ9.demo
```

### 响应数据
`data` 为 `ClubDetailRes`

| 字段 | 类型 | 说明 |
|---|---|---|
| `clubConfigId` | String | Club配置ID |
| `name` | String | Club名称 |
| `description` | String | Club描述 |
| `benefitList` | Array | 权益集合 |
| `originalFee` | String | 原价 |
| `discountFee` | String | 优惠价 |
| `hasDiscount` | Boolean | 是否有优惠 |
| `discountInfo` | Object | 折扣信息，无优惠时返回 `null` |
| `annualFee` | String | 年费 |
| `payCoinId` | String | 支付币种ID |
| `payCoinName` | String | 支付币种名称 |
| `balance` | String | 当前用户可用余额 |
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
| `num` | String | 原始数量，取 `club_config_benefit.config_num` |
| `displayName` | String | 展示文案，按权益类型格式化后的数量 |
| `remark` | String | 权益备注 |

`discountInfo` 结构：

| 字段 | 类型 | 说明 |
|---|---|---|
| `discountQuota` | String | 折扣名额 |
| `remainingQuota` | String | 剩余优惠名额 |
| `discountRate` | String | 折扣，返回原始配置值 |

`memberInfo` 结构：

| 字段 | 类型 | 说明 |
|---|---|---|
| `memberId` | String | 会员ID |
| `memberNo` | String | 会员卡号 |
| `validStart` | String | 有效期开始时间戳(毫秒) |
| `validEnd` | String | 有效期结束时间戳(毫秒) |
| `status` | Integer | 会员状态 |
| `statusName` | String | 会员状态名称 |

### 响应示例
```json
{
  "success": true,
  "ts": 1784592001123,
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
        "num": "10000",
        "displayName": "10000 ELON",
        "remark": "支付成功后发放"
      },
      {
        "name": "高级权益票证",
        "unit": "张",
        "num": "1",
        "displayName": "1 张高级权益票证",
        "remark": "具体活动规则以通知为准"
      }
    ],
    "originalFee": "100.00",
    "discountFee": "80.00",
    "hasDiscount": true,
    "discountInfo": {
      "discountQuota": "100",
      "remainingQuota": "12",
      "discountRate": "8"
    },
    "annualFee": "50.00",
    "payCoinId": "1",
    "payCoinName": "USDT",
    "balance": "560.23",
    "purchaseAvailable": false,
    "purchaseUnavailableReason": "一人一份",
    "isPreRenew": true,
    "preRenewDays": 15,
    "preRenewDate": "1813104000000",
    "isGracePeriod": false,
    "gracePeriodDays": 7,
    "gracePeriodDate": "1815004800000",
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

---

## 3. 查询会员信息

### 接口信息
| 项 | 内容 |
|---|---|
| 接口名称 | 查询会员信息 |
| 请求地址 | `/api/v1/club/member/info` |
| 请求方式 | `GET` |
| Token | 必填 |

### 业务说明
返回当前登录用户最新一条 Club 会员信息。`benefitList` 仅取会员权益快照表，无快照时返回空数组。

### 请求参数
无

### 请求示例
```http
GET /api/v1/club/member/info
Language: zh_CN
Token: eyJhbGciOiJIUzI1NiJ9.demo
```

### 响应数据
`data` 为 `ClubMemberInfoRes`

| 字段 | 类型 | 说明 |
|---|---|---|
| `clubConfigId` | String | Club配置ID |
| `clubConfigName` | String | Club配置名称 |
| `memberId` | String | 会员ID |
| `memberNo` | String | 会员号 |
| `validStart` | String | 有效期开始时间戳(毫秒) |
| `validEnd` | String | 有效期结束时间戳(毫秒) |
| `status` | Integer | 会员状态码 |
| `statusName` | String | 会员状态名 |
| `admissionFee` | String | 入会费 |
| `annualFee` | String | 年费 |
| `createTime` | String | 开通日期时间戳(毫秒) |
| `availablePoints` | String | 可用积分 |
| `availableTickets` | String | 可用票据 |
| `benefitList` | Array | 权益集合 |

`benefitList` 元素结构：

| 字段 | 类型 | 说明 |
|---|---|---|
| `id` | String | 权益ID |
| `benefitType` | Integer | 权益类型 |
| `name` | String | 名称 |
| `unit` | String | 单位 |
| `num` | String | 数量 |
| `status` | Integer | 状态 |
| `statusName` | String | 状态名称 |

### 响应示例
```json
{
  "success": true,
  "ts": 1784592002123,
  "code": 200,
  "msg": "success",
  "data": {
    "clubConfigId": "10001",
    "clubConfigName": "DOGElon Club",
    "memberId": "20001",
    "memberNo": "DGC100001",
    "validStart": "1782864000000",
    "validEnd": "1814400000000",
    "status": 1,
    "statusName": "正常",
    "admissionFee": "100.00",
    "annualFee": "50.00",
    "createTime": "1782864000000",
    "availablePoints": "8200",
    "availableTickets": "1",
    "benefitList": [
      {
        "id": "90001",
        "benefitType": 1,
        "name": "ELON代币",
        "unit": "ELON",
        "num": "10000",
        "status": 2,
        "statusName": "已生效"
      },
      {
        "id": "90002",
        "benefitType": 3,
        "name": "俱乐部积分",
        "unit": "积分",
        "num": "10000",
        "status": 2,
        "statusName": "已生效"
      }
    ]
  }
}
```

---

## 4. 查询权益详情

### 接口信息
| 项 | 内容 |
|---|---|
| 接口名称 | 查询权益详情 |
| 请求地址 | `/api/v1/club/member/benefit/info` |
| 请求方式 | `GET` |
| Token | 必填 |

### 业务说明
根据权益ID查询会员权益详情。接口会根据 `benefitType` 返回不同结构：
- `benefitType=1` 返回 `elonInfo`
- `benefitType=2` 返回 `voucherList`
- `benefitType=3` 返回 `pointsInfo`

### 请求参数
| 字段 | 类型 | 是否必填 | 说明 |
|---|---|---|---|
| `benefitId` | String | 是 | 权益ID |

### 请求示例
```http
GET /api/v1/club/member/benefit/info?benefitId=90002
Language: zh_CN
Token: eyJhbGciOiJIUzI1NiJ9.demo
```

### 响应数据
`data` 为 `ClubBenefitInfoRes`

| 字段 | 类型 | 说明 |
|---|---|---|
| `id` | String | 权益ID |
| `benefitType` | Integer | 权益类型 |
| `benefitTypeName` | String | 权益类型名称 |
| `elonInfo` | Object | ELON权益详情，仅 `benefitType=1` 返回 |
| `voucherList` | Array | 票证列表，仅 `benefitType=2` 返回，其他类型返回空数组 |
| `pointsInfo` | Object | 积分权益详情，仅 `benefitType=3` 返回 |

`elonInfo` 结构：

| 字段 | 类型 | 说明 |
|---|---|---|
| `totalNum` | String | 发放总数 |
| `unit` | String | 单位 |

`voucherList` 元素结构：

| 字段 | 类型 | 说明 |
|---|---|---|
| `id` | String | 票证ID |
| `voucherCode` | String | 票证码 |
| `status` | Integer | 状态 |
| `statusName` | String | 状态名称 |

`pointsInfo` 结构：

| 字段 | 类型 | 说明 |
|---|---|---|
| `totalPoints` | String | 总积分 |
| `usedPoints` | String | 已核销 |
| `remainingPoints` | String | 剩余积分 |

### 响应示例
```json
{
  "success": true,
  "ts": 1784592003123,
  "code": 200,
  "msg": "success",
  "data": {
    "id": "90002",
    "benefitType": 3,
    "benefitTypeName": "俱乐部积分",
    "elonInfo": null,
    "voucherList": [],
    "pointsInfo": {
      "totalPoints": "10000",
      "usedPoints": "1800",
      "remainingPoints": "8200"
    }
  }
}
```

---

## 5. 购买Club

### 接口信息
| 项 | 内容 |
|---|---|
| 接口名称 | 购买Club |
| 请求地址 | `/api/v1/club/purchase` |
| 请求方式 | `POST` |
| Token | 必填 |

### 业务说明
按当前价格购买 Club。接口内部会做当前账号的购买串行锁控制；成功后会落库会员、订单、订单权益、库存和账单，其中会员表会同步写入当前账号的 `accountNo`，订单和订单权益编号统一使用 `IdNoUtils.createOrderNo()` 生成。

### 请求参数
| 字段 | 类型 | 是否必填 | 说明 |
|---|---|---|---|
| `clubConfigId` | String | 是 | Club配置ID |
| `amount` | String | 是 | 支付金额 |
| `safetyPassword` | String | 是 | 安全密码 |

### 请求示例
```json
POST /api/v1/club/purchase
Language: zh_CN
Token: eyJhbGciOiJIUzI1NiJ9.demo

{
  "clubConfigId": "10001",
  "amount": "80.00",
  "safetyPassword": "123456"
}
```

### 响应数据
`data` 为 `Boolean`

| 字段 | 类型 | 说明 |
|---|---|---|
| `data` | Boolean | 是否购买成功，成功返回 `true` |

### 响应示例
```json
{
  "success": true,
  "ts": 1784592004123,
  "code": 200,
  "msg": "success",
  "data": true
}
```

---

## 6. 续费Club

### 接口信息
| 项 | 内容 |
|---|---|
| 接口名称 | 续费Club |
| 请求地址 | `/api/v1/club/renew` |
| 请求方式 | `POST` |
| Token | 必填 |

### 业务说明
按当前年费续费 Club。接口内部会校验安全密码，并做当前账号的续费串行锁控制。
续费成功后会更新原会员信息；若历史会员数据缺失，会同步补齐 `accountNo`，并在 `validStart` 为空时按会员创建时间兜底。订单和订单权益编号统一使用 `IdNoUtils.createOrderNo()` 生成。

### 请求参数
| 字段 | 类型 | 是否必填 | 说明 |
|---|---|---|---|
| `clubConfigId` | String | 是 | Club配置ID |
| `memberId` | String | 是 | 会员ID |
| `amount` | String | 是 | 支付金额 |
| `safetyPassword` | String | 是 | 安全密码 |

### 请求示例
```json
POST /api/v1/club/renew
Language: zh_CN
Token: eyJhbGciOiJIUzI1NiJ9.demo

{
  "clubConfigId": "10001",
  "memberId": "20001",
  "amount": "50.00",
  "safetyPassword": "123456"
}
```

### 响应数据
`data` 为 `ClubRenewRes`

| 字段 | 类型 | 说明 |
|---|---|---|
| `orderNo` | String | 订单号 |
| `renewBeforeExpireDate` | String | 续费前到期时间戳(毫秒) |
| `renewAfterExpireDate` | String | 续费后到期时间戳(毫秒) |
| `createTime` | String | 成交时间戳(毫秒) |
| `payCoinId` | String | 支付币种ID |
| `payCoinName` | String | 支付币种名称 |
| `payAmount` | String | 支付金额 |
| `memberInfo` | Object | 续费后会员信息 |

`memberInfo` 结构：

| 字段 | 类型 | 说明 |
|---|---|---|
| `memberId` | String | 会员ID |
| `memberNo` | String | 会员卡号 |
| `validStart` | String | 有效期开始时间戳(毫秒) |
| `validEnd` | String | 有效期结束时间戳(毫秒) |
| `status` | Integer | 会员状态 |
| `statusName` | String | 会员状态名称 |

### 响应示例
```json
{
  "success": true,
  "ts": 1784592005123,
  "code": 200,
  "msg": "success",
  "data": {
    "orderNo": "CLUB20260722ABC123",
    "renewBeforeExpireDate": "1814400000000",
    "renewAfterExpireDate": "1845936000000",
    "createTime": "1784592005000",
    "payCoinId": "1",
    "payCoinName": "USDT",
    "payAmount": "50.00",
    "memberInfo": {
      "memberId": "20001",
      "memberNo": "DGC100001",
      "validStart": "1782864000000",
      "validEnd": "1845936000000",
      "status": 1,
      "statusName": "正常"
    }
  }
}
```

---

## 7. 分页查询权益记录

### 接口信息
| 项 | 内容 |
|---|---|
| 接口名称 | 分页查询权益记录 |
| 请求地址 | `/api/v1/club/member/benefit/list` |
| 请求方式 | `GET` |
| Token | 必填 |

### 请求参数
| 字段 | 类型 | 是否必填 | 说明 |
|---|---|---|---|
| `clubConfigId` | String | 是 | Club配置ID |
| `memberId` | String | 是 | 会员ID |
| `pageNo` | Integer | 否 | 页码，默认 `1`，继承自 `PageReq` |
| `pageSize` | Integer | 否 | 每页数量，默认 `10`，继承自 `PageReq` |

### 请求示例
```http
GET /api/v1/club/member/benefit/list?clubConfigId=10001&memberId=20001&pageNo=1&pageSize=10
Language: zh_CN
Token: eyJhbGciOiJIUzI1NiJ9.demo
```

### 响应数据
`data` 为分页对象，`list` 元素为 `ClubBenefitRecordRes`

| 字段 | 类型 | 说明 |
|---|---|---|
| `id` | String | 权益记录ID |
| `benefitType` | Integer | 权益类型 |
| `benefitTypeName` | String | 权益类型名称 |
| `amount` | String | 数量 |
| `unit` | String | 单位 |
| `status` | Integer | 状态 |
| `statusName` | String | 状态名称 |
| `createTime` | String | 创建时间戳(毫秒) |

### 响应示例
```json
{
  "success": true,
  "ts": 1784592006123,
  "code": 200,
  "msg": "success",
  "data": {
    "pageNo": 1,
    "pageSize": 10,
    "totalSize": 2,
    "totalPages": 1,
    "prevPage": 1,
    "nextPage": 1,
    "list": [
      {
        "id": "80001",
        "benefitType": 1,
        "benefitTypeName": "ELON代币",
        "amount": "10000",
        "unit": "ELON",
        "status": 2,
        "statusName": "已发放",
        "createTime": "1782864000000"
      }
    ]
  }
}
```

---

## 8. 分页查询积分记录

### 接口信息
| 项 | 内容 |
|---|---|
| 接口名称 | 分页查询积分记录 |
| 请求地址 | `/api/v1/club/member/points/list` |
| 请求方式 | `GET` |
| Token | 必填 |

### 请求参数
| 字段 | 类型 | 是否必填 | 说明 |
|---|---|---|---|
| `clubConfigId` | String | 是 | Club配置ID |
| `memberId` | String | 是 | 会员ID |
| `pageNo` | Integer | 否 | 页码，默认 `1`，继承自 `PageReq` |
| `pageSize` | Integer | 否 | 每页数量，默认 `10`，继承自 `PageReq` |

### 请求示例
```http
GET /api/v1/club/member/points/list?clubConfigId=10001&memberId=20001&pageNo=1&pageSize=10
Language: zh_CN
Token: eyJhbGciOiJIUzI1NiJ9.demo
```

### 响应数据
`data` 为分页对象，`list` 元素为 `ClubPointsRecordRes`

| 字段 | 类型 | 说明 |
|---|---|---|
| `id` | String | 积分记录ID |
| `sourceType` | Integer | 来源类型 |
| `sourceTypeName` | String | 来源类型名称 |
| `beforePoints` | String | 变动前积分 |
| `points` | String | 变动积分 |
| `afterPoints` | String | 变动后积分 |
| `createTime` | String | 创建时间戳(毫秒) |

### 响应示例
```json
{
  "success": true,
  "ts": 1784592007123,
  "code": 200,
  "msg": "success",
  "data": {
    "pageNo": 1,
    "pageSize": 10,
    "totalSize": 2,
    "totalPages": 1,
    "prevPage": 1,
    "nextPage": 1,
    "list": [
      {
        "id": "70001",
        "sourceType": 1,
        "sourceTypeName": "Club购买",
        "beforePoints": "0",
        "points": "10000",
        "afterPoints": "10000",
        "createTime": "1782864000000"
      }
    ]
  }
}
```

---

## 9. 分页查询权益票证记录

### 接口信息
| 项 | 内容 |
|---|---|
| 接口名称 | 分页查询权益票证记录 |
| 请求地址 | `/api/v1/club/member/voucher/list` |
| 请求方式 | `GET` |
| Token | 必填 |

### 请求参数
| 字段 | 类型 | 是否必填 | 说明 |
|---|---|---|---|
| `clubConfigId` | String | 是 | Club配置ID |
| `memberId` | String | 是 | 会员ID |
| `pageNo` | Integer | 否 | 页码，默认 `1` |
| `pageSize` | Integer | 否 | 每页数量，默认 `10` |

### 请求示例
```http
GET /api/v1/club/member/voucher/list?clubConfigId=10001&memberId=20001&pageNo=1&pageSize=10
Language: zh_CN
Token: eyJhbGciOiJIUzI1NiJ9.demo
```

### 响应数据
`data` 为分页对象，`list` 元素为 `ClubVoucherRecordRes`

| 字段 | 类型 | 说明 |
|---|---|---|
| `id` | String | 票证记录ID |
| `voucherCode` | String | 票证码 |
| `status` | Integer | 核销状态 |
| `statusName` | String | 核销状态名称 |
| `createTime` | String | 创建时间戳(毫秒) |

### 响应示例
```json
{
  "success": true,
  "ts": 1784592008123,
  "code": 200,
  "msg": "success",
  "data": {
    "pageNo": 1,
    "pageSize": 10,
    "totalSize": 1,
    "totalPages": 1,
    "prevPage": 1,
    "nextPage": 1,
    "list": [
      {
        "id": "60001",
        "voucherCode": "VIP-202607220001",
        "status": 1,
        "statusName": "未核销",
        "createTime": "1782864000000"
      }
    ]
  }
}
```
