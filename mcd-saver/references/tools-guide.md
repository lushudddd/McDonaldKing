# 麦当劳 MCP 省钱工具速查表

本文档是 mcd-saver Skill 的工具参考手册，基于麦当劳 MCP Server（`https://mcp.mcd.cn`，Streamable HTTP + Bearer Token）的真实工具定义整理。

## 通用规则（先读这一节）

### 就餐方式编码表

| beType | 场景 | orderType | beCode 要求 |
|--------|------|-----------|-------------|
| 1 | 到店自取 | 1 | 不传 |
| 5 | 得来速（DT） | 1 | 必传（来自 query-nearby-stores） |
| 2 | 麦乐送到家 | 2 | 必传（来自 delivery-query-stores） |
| 6 | 企业团餐 | 2 | 必传（来自 delivery-query-stores） |

- `beCode` 与 `storeCode` 配对使用，从门店查询工具的返回结果中获取，后续所有工具调用必须原样透传。
- 预约场景：所有点餐链路工具需额外传 `reservationDate`，格式 `yyyy-MM-dd HH:mm`。
- 金额单位：`calculate-price` 返回的价格字段单位为**分**，展示时除以 100 转为元。

### 限流与错误

| 错误码 | 原因 | 处理 |
|--------|------|------|
| 401 | Token 无效/过期/未提供 | 提示用户检查 Authorization 请求头和 Token |
| 429 | 超过 600 次/分钟 | 降低调用频率，稍后重试 |

---

## 一、券务工具（省钱核心）

### available-coupons — 麦麦省可领券列表

- **无参数。** 返回当前可领取的麦麦省优惠券（券名称、图片、状态、促销标签）。
- 调用时机：用户问"有什么优惠 / 可以领什么券"。

### auto-bind-coupons — 麦麦省一键领券

- **无参数。** 自动领取全部当前可领的券，无需指定 couponId。
- 调用时机：用户说"帮我领券 / 一键领券"，或工作流 A 中领券动作。
- 通常与 `available-coupons`（先看）→ `auto-bind-coupons`（再领）→ `query-my-coupons`（确认到账）串联使用。

### query-my-coupons — 我的卡包券

- 参数：`page`（默认 1，最多 5 页）、`pageSize`（默认 200，最大 200），均为字符串型。
- 返回用户**拥有的全部券**，⚠️ 不做门店/渠道/配送方式校验，**不承诺当前订单可用**。
- 判断"当前订单可用券"必须改用 `query-store-coupons`。

### query-store-coupons — 门店可用券（比价数据源）

- 必填：`storeCode`（纯数字门店编码）、`orderType`（1/2）、`beType`（1/2/5/6）；beType=5/2/6 时必传 `beCode`。
- 选填：`reservationDate`（预约场景）。
- 调用时机：已确定门店后，用户问"我有什么券能用 / 下单前看可用优惠 / 选了商品看能用哪些券"。

---

## 二、点餐比价工具

### query-nearby-stores — 查附近门店（到店/得来速）

- 必填：`beType`（1 到店自提 / 5 得来速）、`searchType`（1 收藏餐厅 / 2 按位置搜索）。
- searchType=2 时必填：`city`、`keyword`。
- 返回门店 storeCode；beType=5 时同时返回 beCode（后续必传）。
- 用户无收藏记录时改用 searchType=2 搜索。
- reservation=true 的门店要完整展示 reservationTimeOptions，today 标记"(今天)"。

### delivery-query-addresses / delivery-create-address — 配送地址管理

- 外送链路前置：先查地址列表；用户无可配送地址或需新增时再新建。

### delivery-query-stores — 可配送门店（麦乐送/团餐）

- 基于配送地址查询附近可配送门店，返回 storeCode + beCode（后续必传）。

### query-meals — 餐品菜单

- 必填：`storeCode`、`orderType`、`beType`（+ beCode 视场景）。
- 返回门店可售餐品的分类、商品编码（productCode）、标签、随单购商品等。**下单选品前必须先查本工具拿商品 code。**

### query-meal-detail — 餐品详情

- 根据餐品编码查套餐组成与可替换选项（modification），构造特制参数前必须先查。

### calculate-price — 价格试算（比价核心）

- 必填：`storeCode`、`orderType`、`beType`；视场景传 `beCode`、`reservationDate`。
- `items[]` 每项：`productCode`（必填）、`quantity`（必填 ≥1）、`couponId` / `couponCode`（挂券时传）、`modification`（特制）、`roundList`（套餐子项）。
- 选填：`needTableware`（餐具费）、`withOrder`（随单购，可享随单购优惠）、`gmServiceCode`（团餐助餐服务）。
- **返回价格单位为分**，除以 100 转元；到店下单所需的 `takeWayCode` 取自返回的 `data.takeWayList[].code`。

#### 特制（modification）传参规则 ⚠️

对每个特制项：用户选中的项 `key` = query-meal-detail 返回的 `selectedKey`；未选中的项若 `unselectedKey` 不为空，也必须传（key = unselectedKey）。即：**含 unselectedKey 的特制组内所有项都要传入**，不要遗漏。

### create-order — 创建订单

- 必填：`storeCode`、`orderType`、`beType`（+ 视场景 beCode）。
- 到店（orderType=1）：必传 `takeWayCode`（来自 calculate-price 的 takeWayList）。
- 外送（orderType=2）：必传 `addressId`；`remark` 选填（≤50 字，如"无接触配送"）。
- items 结构与 calculate-price 一致，透传最优比价方案。
- 成功后返回订单详情与支付链接，引导用户支付后回复"支付完成/失败"，再用 `query-order` 查状态。

### query-order / order-list / cancel-order — 订单管理

- `query-order`：查单笔订单状态、内容、配送信息。
- `order-list`：查近期到店/外送历史订单（商城订单用 mall-order-list）。
- `cancel-order`：取消订单，用户说"取消订单 / 帮我取消"时使用。

---

## 三、积分与商城工具

### query-my-account — 积分账户

- **无参数。** 返回可用/累计/冻结/即将过期积分。资产盘点与积分变现的入口工具。

### mall-points-products — 积分商城商品列表

- 选填 `catRuleIds`（逗号分隔多选）：`1>4` 商品券、`2` 实物商品、`2>8` 周边产品、`2>9` 实物礼品卡、`1>6>20` 生日类派对、`1>6>21` 主题类派对、`1>6>22` 麦麦体验营、`1>6>25` 品鉴会、`1>6>34` 读书会、`1>6>40` 积分兑换活动。不传默认查全部。

### mall-product-detail / mall-create-order — 商城详情与积分兑换下单

- 兑换流程：列表 → 详情（图片、积分、有效期、说明）→ 下单（完成积分校验与扣减，返回兑换订单号和券码）。

### mall-order-list / mall-order-detail — 商城订单

- 查近一年商城兑换/购买记录与订单详情（支付积分、支付金额、订单状态）。

### query-lottery-info / draw-lottery / query-my-prizes — 积分抽奖

- `query-lottery-info`：活动状态、奖品列表、消耗规则、可用资源；`draw-lottery`：执行一次抽奖（先向用户披露消耗规则再执行）；`query-my-prizes`：奖品记录（分页，按中奖时间倒序）。

---

## 四、辅助工具

### now-time-info — 当前时间

- **无参数。** 返回时间戳、格式化日期时间、时区。涉及"今天/本月/最新"语义或预约场景前先调用。

### campaign-calendar — 活动日历

- 选填 `specifiedDate`（`yyyy-MM-dd`）：传入则以该日为锚点返回前后各一个有活动日子（共三天）；不传返回当月全部活动。
- 返回进行中、往期和未来活动及用户订阅状态。用户问"最近有什么活动/优惠活动"时使用；发现可领券活动时衔接工作流 A。
