# bStock与股票永续的定价权：Perp价格发现、库存层与Borrow双向纠错

## 原文信息

- 作者：Danny（`@agintender`）
- 原文标题：`bStocks 的 B 面：不做更好的 Nasdaq，用 Perp 绕出一条定价权之路`
- 原文链接：`https://agintender.substack.com/p/bstocks-b-nasdaq-perp`
- 发布时间：`2026-08-15 12:17:40 +08:00`
- 内容类型：Substack 付费长文；股票永续与代币化股票的市场结构、样本统计和策略框架
- 原文归档：[`sources/agintender-211263058-bstocks-perp-price-discovery/`](sources/agintender-211263058-bstocks-perp-price-discovery/)
- 配图：正文 25 张，已归档

## 主题

这篇文章的核心不是比较 `bStock` 与股票永续谁的成交量更大，而是给两类产品重新分工：

```text
Perp：低成本表达多空与杠杆，快速生产休市后的候选价格
bStock：把股票风险变成可持有、转换、抵押和对冲的库存
Borrow：让库存能被卖空与做市，补齐双向纠错
Cash Market：重新开市后提供验证与最终交割约束
```

作者的论点是：Binance 若要争夺传统股票闭市后的定价权，冲在前面的会是 Perp，因为它周转快、做空方便、资本效率高；但候选价格能否变成可信市场价格，取决于 bStock、真实股票转换、借券、做市和跨市场套利能否持续证伪错误报价。

这可以压缩成一条价格链：

```text
新信息
→ Perp 形成第一版价格
→ bStock / Stock 提供库存与终端约束
→ Borrow / Conversion / Arbitrage 双向纠错
→ Cash Open 验证
→ 外部行情、OTC与风控系统是否采用
```

## 核心判断

### 1. Perp更适合先报价，但先报价不等于报价正确

Perp 具备三个价格发现优势：

- 多空表达对称，不需要先解决现金股票借券；
- 杠杆降低表达观点的资本门槛；
- 开仓、平仓、反手、Basis、Funding 和做市对冲让同一资本反复周转。

作者给出的 Binance 内部样本中，`2026-07-17` 至 `07-22` 的 `NVDAUSDT Perp` 名义成交约 `4.18 亿美元`，`NVDAB Spot` 约 `421 万美元`，相差约 `99 倍`。这个比较只能说明休市后的方向订单更容易流入哪种产品，不能拿来证明 Binance Perp 已超过 Nasdaq 正股的定价能力。

因此更准确的关系是：

```text
Information → Perp → Candidate Price
```

Perp 负责生产候选价格，不负责保证候选价格正确。

### 2. 25次“闭市—重开”样本没有证明稳定周末定价权

作者收集 `NVDAUSDT`、`TSLAUSDT`、`SNDKUSDT`、`SPCXUSDT` 和 `SKHYUSDT` 共 `25` 次可对应下一次美股 Cash Open 的样本，并观察 Sunday、Premarket、Opening Auction 前和 Cash Open 的价格路径。

主要结果是：

- Sunday 到下一次 Cash Open 的平均绝对误差约 `181.9 bp`；
- `25` 次中方向一致 `13` 次，约 `52%`，接近随机；
- 到 `09:29:59 ET`，绝对误差降至约 `11.3 bp`，方向一致率升至 `96%`。

最后一组数字不能解释为 Binance 成功预测开盘。到 `09:29:59`，美国 Premarket 已运行数小时，Opening Auction 信息也已进入；Perp 与现金市场接近，可能只是跨市场套利和跟随的结果。

真正要证明价格领导，必须比较逐秒 lead-lag：

```text
Premarket先动、Perp后到 → follow
Perp先动、Premarket后靠拢 → discovery / leadership
```

只比较周日价格和下次开盘，最多说明结果接近程度，不能识别信息传播方向。

### 3. 不同股票对应不同的休市定价状态

作者没有把所有偏差都归为“预测成功或失败”，而是拆成三类：

- `Direction Discovery`：提前交易出开盘跳空的方向；
- `Magnitude Discovery`：提前交易出的幅度与最终跳空接近；
- `Correction Dependency`：周末先报错，依赖 Premarket 和 Opening Auction 修正。

样本中：

- `NVDA` 更像纠错路径，独立日样本 Sunday 高于最终 Cash Open 约 `202 bp`，随后回归；
- `TSLA` 更像方向正确但幅度过冲；
- `SNDK` 同时展示过提前覆盖大部分 `+4.8%` Opening Gap，也展示过周末几亿美元成交后方向看反；
- `SPCX` 是产品迁移实验：先有 Pre-IPO Perp，后有公开股票、标准 Perp 和 bStock；
- `SKHY` 单个周末约 `8,530 万美元`成交，Sunday 仍可偏离下一次开盘约 `386 bp`。

这些案例共同说明：**成交活跃、方向正确、幅度准确和价格领导是四个不同问题。**

### 4. 巨额成交更像高转速风险市场，不等于独立观点资金

六个周末的作者统计中：

- `SNDKUSDT` 累计成交约 `29.22 亿美元`、约 `757 万` fills，平均 `7.3 fills/秒`；
- `SPCXUSDT` 累计约 `15.46 亿美元`、约 `358 万` fills，平均 `3.45 fills/秒`；
- `NVDAUSDT` 累计约 `1.54 亿美元`、平均约 `0.62 fills/秒`。

SNDK 与 SPCX 的单笔成交金额没有大到足以解释几十倍总量，差距主要来自更高的成交频率。由于没有账户身份和策略标签，无法知道多少成交来自独立投资观点，多少来自做市、HFT、Basis 和重复回转。

因此 `gross notional` 更适合衡量市场转速，不能直接当作同等规模的净资金流入或价格共识。

## bStock为什么不是多余的现货复制品

### 1. bStock的价值不是比Perp更会预测，而是把价格保存成资产

作者另一组低频 handoff proxy 中，`8` 个 bStock 观察值相对下一次 Cash Open 的绝对误差中位数约 `117.2 bp`，`6` 个可比 Perp 约 `118.9 bp`。样本很小且口径不同，但至少没有显示 Perp 在所有时点都明显更接近下一次开盘。

bStock 的作用不是抢在 Perp 前面报出更准的价格，而是把休市期间的股票风险变成一份可以持有、转移、转换、抵押和做市的现货型库存：

```text
Perp让价格移动
bStock把价格变成资产
```

### 2. 1:1 backing是终端约束，不是周末即时锚

现金市场关闭时，把 NVDAB 转成 NVDA Stock，不代表同一秒能在 Nasdaq 完成另一条套利腿。`1:1 backing` 更像 `terminal constraint`：市场知道 bStock 最终能接回真实股票、现金市场也会重开，因此大偏差会吸引承担库存、资金和事件风险的套利者，但它不是 USDT 式随时可执行的即时赎回锚。

所以 bStock 更准确的定位是：

- Perp 做市商的对冲库存；
- Basis trader 的现货腿；
- 长期持有者的股票敞口；
- Lending 与 Margin 的抵押品；
- DEX LP 的股票资产；
- 跨时段 Stock Conversion 的库存载体。

### 3. Borrow决定市场能否双向纠错

两类 Basis 的可执行性不对称：

```text
Perp贵、bStock便宜：Long bStock + Short Perp
bStock贵、Perp便宜：Short bStock + Long Perp
```

第一条通常只需现金和 Perp 保证金；第二条必须先获得可借 bStock。没有 Borrow 时，便宜资产可以被买入，高估 bStock 却未必能被卖空，错误价格只能单向修正。

Borrow 对做市也同样关键。客户持续买 bStock 时，做市商的卖盘会消耗库存；库存用完后，只能缩小 Ask、抬价或退出。有 Borrow 后，库存从硬上限变成有价格的资源，做市商可借入 bStock 卖给客户，再用 Long Perp 或真实股票控制 Delta。

因此原文最精炼的产品分工是：

> **Perp负责产生订单，bStock负责提供股票库存，Borrow负责让库存流动。**

### 4. 上链的关键不是TVL，而是库存周转率

bStock 进入 DEX、Lending、Collateral 和 Basket 并不自动增加可执行流动性。把 NVDAB 锁作抵押品只增加了用途；只有库存能被做市商、套利者或空头借出，并通过 Perp、CEX、DEX 和 Stock Conversion 循环，才真正增加双边报价能力。

理想路径是：

```text
bStock
→ DEX / Lending / Credit Pool
→ Market Maker / Arbitrageur
→ Perp Hedge
→ CEX
→ Stock Conversion
```

因此比 TVL 更值得观察的是同一份库存能被调用多少次，以及能否在事件周末持续借出。

## 定价权的四重验证

### 1. Lead-Lag

把 Perp、bStock 与同一秒的现金市场 Premarket `last / bid / ask` 放在一起，判断谁先吸收信息。只看两个静态时点无法证明价格领导。

### 2. Executability

领先几十秒但只能成交几千美元的价格，很难成为机构参考。应持续保存 L2，测算 `10万 / 50万 / 100万美元` 订单的真实 impact，而不是只看屏幕中间价。

### 3. Two-way Correction

Perp 高估时能否 `Long bStock + Short Perp`，bStock 高估时又能否 `Short bStock + Long Perp`。这取决于 Borrow Depth、Borrow Rate、Conversion Capacity、保证金共用和双边现货深度。

### 4. Reference Adoption

平台内部有大量用户交叉使用 Perp、bStock 和 Equity，只能说明生态内部连接正在形成。真正的定价权证据来自外部：其他 venue、Oracle、OTC desk、行情终端或第三方风控系统是否开始引用这套休市价格。

如果现金市场重开后始终是 Perp 追着传统市场跑，Binance 只是一个好的 `24/7` 交易场所；只有外部市场和风险系统开始向它的价格靠拢或引用，它才拥有价格领导地位。

## 机会来源 / 执行框架

最基础的价差是：

```text
Basis(bp) = (Perp - bStock) / bStock × 10,000
```

但屏幕 Basis 不是利润。净收益还需要扣除：

- 两条腿的 spread 与 slippage；
- Funding、Borrow APR 与资金成本；
- 链上 gas、跨链和转换费用；
- 资本占用与保证金缓冲；
- Borrow 召回、Conversion 暂停与一条腿无法退出的尾部风险。

可迁移的组合包括：

- 长期持有 bStock，用 Short Perp 临时降低事件 Delta；
- `Long bStock + Short Perp` 交易 Perp 过冲；
- `Short bStock + Long Perp` 交易反向 Basis，但必须先锁定库存与 Borrow 成本；
- bStock 抵押借 Stablecoin，再用 Perp 管理方向或 Basis；
- DEX bStock LP 配合 CEX Perp 动态对冲 LP 股票 Delta；
- Stock、CEX bStock、DEX bStock、Perp 四市场价差交易，先锁定最稀缺的一腿。

这些结构一旦缺失 Borrow、现货深度、保证金或 Conversion，就会从相对价值交易退化成方向仓位。

## 风险与限制

- `25` 次美股样本规模较小，标的集中在活跃且具有产品特殊性的股票，不能外推所有股票永续。
- Sunday 与下一次 Cash Open 的误差只衡量结果距离，不识别信息传播方向；Cash Open 本身也受 Opening Auction、隔夜订单和库存影响，不等于唯一“公允价值”。
- bStock 低频 handoff 使用 UTC 日末代理，而 Perp 主样本使用美东逐时点，两组口径不能混合统计。
- 没有账户身份、策略标签和净头寸数据，无法判断高频成交中有多少来自独立信息、做市回转或关联交易。
- 原文使用的交易量、用户交叉和产品数据部分来自 Binance 自身研究，平台口径存在选择性展示风险。
- 作者曾公开披露自己是 `youcanshortit.com` builder；文中将该产品列入 bStock Borrow 生态时，需要考虑利益相关立场。
- Borrow、Conversion、托管、公司行动、证券合规、链上预言机和跨层清算都可能成为单点故障。
- bStock 抵押、Stablecoin 借款和 Perp 保证金如果共享同一股票下跌触发器，会把资本效率变成多层同步清算风险。
- 文中策略示例用于说明机制，不构成交易建议；所有数字和规则都需在执行前重新核验。

## 扩散分析 / 延展思路

### 1. bStock成熟度应改用“库存型KPI”

相比上线数量和总 TVL，更有解释力的指标包括：

- 可执行现货深度与大单 impact；
- Borrow Depth、Borrow APR、利用率与召回稳定性；
- Stock ↔ bStock 的 Conversion 容量、时延和暂停频率；
- Perp-bStock Basis 的分布、半衰期与双向收敛能力；
- 单位 bStock 库存支持的成交额和做市周转率；
- 事件周末的库存缺口、Funding 与保证金占用；
- 外部 Oracle、OTC 和风控系统的引用率。

### 2. 把价格发现拆成三层，而不是只算预测准确率

更完整的研究可以依次检验：

1. `Information Share`：谁先反映新闻；
2. `Executable Leadership`：领先价格能承载多大资金；
3. `Balance-Sheet Adoption`：外部机构是否愿意用该价格对冲、报价和清算。

第一层可以用高频 lead-lag、Hasbrouck information share 或 VECM 研究；第二层依赖历史 L2；第三层需要观察外部报价采纳，而不是平台内部成交量。

### 3. 与库内已有材料的关系

本文与 [`RWA永续套利复盘`](RWA永续套利复盘：周末指数错配、资金费率与海力士ADR溢价.md) 形成上下游关系：后者关注周末 Index 错配和跨所价差，本文进一步解释为什么 Perp 先报价、bStock 提供库存、Borrow 决定纠错方向。它也补充了 [`链上股票AMM做市`](链上股票AMM做市：相关资产配对、库存偏好与无常损失门槛.md)：AMM 能否成为有效股票流动性，不只取决于手续费覆盖无常损失，还取决于 LP 库存能否被 Perp 动态对冲、能否借出，以及 Conversion 是否稳定。

## 一句话结论

**Perp可以抢到股票休市后的第一版价格，但只有 bStock、Borrow、Conversion 和跨市场套利把这份价格变成可持有、可做空、可执行且能被外部采用的库存网络，成交量才可能升级为真正的定价权。**
