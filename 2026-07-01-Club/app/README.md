# Club App 接口需求文档

## 通用约定

### 接口前缀
`/api/v1/club`

### 请求头
| 名称 | 是否必填 | 类型 | 说明 |
|---|---|---|---|
| `Language` | 否 | String | 语言，默认 `zh_CN` |
| `Token` | 按接口说明 | String | 登录凭证 |

### 通用响应结构
| 字段 | 类型 | 说明 |
|---|---|---|
| `success` | Boolean | 是否成功 |
| `ts` | String | 服务端当前时间戳(毫秒) |
| `code` | Integer | 响应码，成功为 `200` |
| `msg` | String | 响应描述 |
| `data` | Object | 响应数据 |

### 分页响应结构
| 字段 | 类型 | 说明 |
|---|---|---|
| `pageNo` | Integer | 当前页码 |
| `pageSize` | Integer | 每页数量 |
| `totalSize` | String | 总记录数 |
| `totalPages` | Integer | 总页数 |
| `prevPage` | Integer | 上一页 |
| `nextPage` | Integer | 下一页 |
| `list` | Array | 列表数据 |

### 字段规则
| 规则 | 说明 |
|---|---|
| 时间字段 | 接口返回时间戳(毫秒) |
| `Long` 字段 | 接口返回 String |
| `BigDecimal` 字段 | 接口返回 String |
| 请求参数 | 除 `pageNo`、`pageSize`、状态码等整型字段外，其余统一按 String 传入 |
| 接口风格 | 查询接口使用 `GET`，提交接口使用 `POST` |

## 文档列表
- [查询已启用的Club](/Users/roy/workspace/dog-elon/doge-app-api/prd/查询已启用的Club.md)
- [查询Club详情](/Users/roy/workspace/dog-elon/doge-app-api/prd/查询Club详情.md)
- [查询会员信息](/Users/roy/workspace/dog-elon/doge-app-api/prd/查询会员信息.md)
- [查询权益详情](/Users/roy/workspace/dog-elon/doge-app-api/prd/查询权益详情.md)
- [购买](/Users/roy/workspace/dog-elon/doge-app-api/prd/购买.md)
- [续费](/Users/roy/workspace/dog-elon/doge-app-api/prd/续费.md)
- [分页查询权益记录](/Users/roy/workspace/dog-elon/doge-app-api/prd/分页查询权益记录.md)
- [分页查询积分记录](/Users/roy/workspace/dog-elon/doge-app-api/prd/分页查询积分记录.md)
- [分页查询权益票证记录](/Users/roy/workspace/dog-elon/doge-app-api/prd/分页查询权益票证记录.md)
