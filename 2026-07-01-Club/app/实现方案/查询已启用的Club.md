# 查询已启用的Club

## 接口信息
| 项 | 内容 |
|---|---|
| 请求地址 | `/api/v1/club/enable/list` |
| 请求方式 | `GET` |
| Token | 选填 |

## 业务说明
1. 当前实现只返回一条已启用的 Club 配置，取 `club_config` 中 `status=ENABLE` 且 `id` 最大的一条。
2. 未登录时可访问，未登录或当前账号没有对应会员时，`isBuy=false`、`memberInfo=null`。
3. 若当前没有任何已启用 Club，接口成功返回，`data=null`。

## 请求参数
无

## 请求示例
```http
GET /api/v1/club/enable/list
Language: zh_CN
Token: eyJhbGciOiJIUzI1NiJ9.demo
```

## 响应数据
`data` 为对象

| 字段 | 类型 | 说明 |
|---|---|---|
| `clubConfigId` | String | Club配置ID |
| `name` | String | Club名称 |
| `description` | String | Club描述，多语言内容已按 `Language` 返回 |
| `benefitList` | Array | 权益集合 |
| `isBuy` | Boolean | 是否已购买 |
| `hasDiscount` | Boolean | 是否有可用优惠 |
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
| `discountQuota` | String | 折扣总名额，取 `club_config_discount.discount_quota` |
| `remainingQuota` | String | 剩余优惠名额，计算公式：`discountQuota - useQuota`，最小返回 `0` |
| `discountRate` | String | 折扣原始值，例如 `8` 表示 `8折` |

`memberInfo` 结构：

| 字段 | 类型 | 说明 |
|---|---|---|
| `memberId` | String | 会员ID |
| `memberNo` | String | 会员卡号 |
| `validStart` | String | 生效开始时间戳(毫秒) |
| `validEnd` | String | 生效结束时间戳(毫秒) |
| `createTime` | String | 开通时间戳(毫秒) |

## 响应示例
```json
{
  "success": true,
  "ts": "1784592000123",
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

## 校验说明
1. 本接口无入参校验。
2. 折扣信息只读取 `club_config_discount` 中 `status=ENABLE` 且 `id` 最大的一条。

## 复杂业务说明
1. `benefitList` 数据来源于 `club_config_benefit`，当前实现不再按 `status` 过滤，排序规则为 `benefitType asc, id asc`。
2. `benefitList.num` 返回原始配置数量，`benefitList.displayName` 返回带单位/文案的展示值。
3. `isBuy` 表示“当前账号是否买过该 Club”，只要当前账号存在对应 `club_member` 记录，就返回 `true`；是否允许再次新购属于其他业务判断，不再混用到该字段中。
4. `memberInfo` 与 `isBuy` 保持一致：只要存在会员记录就返回，不区分当前会员状态是否已失效。
5. `hasDiscount=true` 需同时满足以下条件：
   - 存在启用中的折扣配置。
   - `discountQuota > useQuota`。
   - 折扣换算后小于原价，当前换算公式为 `discountRate / 10`。

## 失败码
本接口当前无主动抛出的业务失败码；未命中数据时返回 `success=true, data=null`。

# BUG（已完成）
1. `benefitList.num` 返回值，现改为返回原始配置数量。已完成
2. `benefitList.displayName` 字段，取值规则沿用原 `num` 的展示逻辑。已完成
3. 购买后会员状态为已失效,为什么接口中返回的isBuy是false? 已完成
   - 当前实现已调整为：只要存在对应会员记录，`isBuy=true`，不再使用“是否允许再次新购”来判断是否买过。已完成
