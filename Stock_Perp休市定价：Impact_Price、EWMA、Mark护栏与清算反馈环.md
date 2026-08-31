# Stock Perp休市定价：Impact Price、EWMA、Mark护栏与清算反馈环

## 原文信息

- 作者：Danny（`@agintender`）
- 原文标题：`Nasdaq关门以后，谁来报价：Stock Perp、Oracle和一场24/7的股票定价战争（一）`
- 原文链接：`https://agintender.substack.com/p/nasdaqstock-perporacle247`
- 发布时间：`2026-08-26 19:02:46 +08:00`
- 内容类型：Substack 付费长文；股票永续休市定价算法与市场结构分析
- 原文归档：[`sources/agintender-212825757-stock-perp-oracle-offhours-pricing/`](sources/agintender-212825757-stock-perp-oracle-offhours-pricing/)
- 配图：正文 16 张，已归档
- 范围说明：这是系列第一篇，重点覆盖 Binance 与 Bitget；作者预告的其他交易所比较、策略和数据实证不在本篇内。

## 主题

这篇文章研究的不是简单的“股票永续周末还能交易”，而是一个更基础的问题：**当 Nasdaq、NYSE 等现金市场休市，股票永续的参考价由谁产生，交易所如何把自己的订单簿逐层变成 Index 和 Mark，以及这套内部定价如何反过来影响保证金与清算。**

传统股票开市时，价格链条大致是：

```text
现金股票成交 → 外部数据商 / Oracle → 交易所 Index → Mark → 风控与清算
```

休市时，如果外部报价不可用，而平台改用自己的永续盘口生成参考价，链条会变成：

```text
本地 Orderbook → Impact Price → EWMA / EMA → Internal Index
                                      ↓
                            Basis / Last → Mark → 清算
```

这使股票永续从“跟踪股票价格的合约”变成了一个阶段性的价格生产者。真正的结构变化不是交易时间从 `5×8` 变成 `24/7`，而是**价格源、价格使用者和被价格清算的仓位开始集中在同一个系统里。**

## 核心机制

### 1. Oracle、Index、Last、Mark回答的是四个问题

- `Oracle / Reference Price`：外部世界最近提供了什么股票报价，是交易所加工价格的原材料。
- `Index Price`：交易所选择多个来源后，经加权、异常值处理、平滑和保护带得到的内部参考价。
- `Last Price`：本平台最后一笔真实成交，最及时，也最容易被薄盘口或单笔大单拉远。
- `Mark Price`：风险系统用于未实现盈亏、保证金和强平的价格，通常综合 Index、资金费率基差、本地盘口和 Last。

正常时主链条是 `TradFi / Oracle → Index → Mark`；永续盘口则通过 `BBO / Basis / Last → Mark` 进入风控。休市模式最特别的地方，是平台可能再把本地盘口接回 Index：

```text
Orderbook → Internal Index / Oracle
```

于是同一个 `NVDA` 永续可以同时出现 `Last = 108`、`Index = 102`、`Mark = 104`。这不一定是系统错误，而可能是不同过滤器分别在处理成交即时性、参考价稳定性和清算连续性。

### 2. Binance：空间过滤、时间过滤、Mark快通道与动态护栏

作者整理的 Binance 休市定价链条可以拆成五层。

第一层是 `Impact Price`。系统不是直接相信 Last，而是用规定的 `Impact Margin Notional` 模拟吃掉订单簿两侧，得到 `Impact Bid` 和 `Impact Ask`：

```text
Impact Mid = (Impact Bid + Impact Ask) / 2

Impact Margin Notional
= 200 USDT / 最高杠杆对应的 Initial Margin Rate
```

若最高杠杆为 `10x`、初始保证金率约 `10%`，测量名义金额约为 `2,000 USDT`。它过滤的是空间深度：一笔孤立成交不足以移动参考价，盘口必须在一定可执行规模上整体迁移。

第二层是 `EWMA`。Impact Mid 即使已经移动，Index 也不会立即全部采纳，而会根据持续时间逐步靠近。Binance 没有在作者引用的规则中公开具体衰减参数；文中约 `300 秒` 是作者测算，不应当当成官方固定值。

第三层是 Mark 的本地快通道。作者将常规定价概括为三项取中位数：

```text
Mark = Median(资金费率调整后的 Index,
              Index + 约30秒盘口 Basis 均值,
              Last / Contract Price)
```

因此 Index 仍在慢慢移动时，持续变化的本地买卖盘和 Last 已经可以先推动 Mark。`Index 走得慢` 不等于 `清算价完全不动`。

第四层是 Mark 相对 Index 的偏离护栏。作者引用的规则是常规、盘前、盘后和夜盘阶段通常为 `±5%`，周末与节假日收紧为 `±3%`。这里限制的是**当时 Mark 相对当时 Index 的距离**，不是整个周末最多涨跌 `3%`。Index 自己移动后，护栏也随之移动。

第五层是模式交接。进入或退出 Orderbook EWMA Mode 时，旧模式与新模式的 Index 按窗口加权混合；作者引用的 RCH 默认窗口为 `30 秒`。这避免瞬时硬切，却把价格跳跃改写成一段较可预测的重新定价斜坡。

### 3. Bitget：两台平滑器与两道围栏

Bitget 与 Binance 的方向相同：外部市场开着时使用 `Pyth / dxFeed / Massive / Intrinio` 等外部来源，休市时由本地股票永续盘口生成 Internal Index。不同点是 Bitget 公开了更多 Index 参数。

作者整理的 Internal Index 链条是：

```text
Orderbook
→ 2,000 USDT Impact Bid / Ask
→ IPD
→ τ = 300秒的 EMA（约每200ms更新）
→ Internal Index
→ 最后有效 External Price ±10% 保护带
```

其中：

```text
IPD = max(P_impactBid - S, 0) - max(S - P_impactAsk, 0)
```

`S` 是当前 Internal Index。IPD 不直接给出“新价格是多少”，而是表达当前盘口想把 Index 往哪个方向拖、拖多远。

Mark 又有另一层 EMA / EWMA 平滑，并在许多股票永续上受 `Index ±5%` 的最终偏离限制。因此 Bitget 的休市结构可以概括成：

```text
Index：Impact → 300秒 EMA → ±10% 外部收盘价围栏
Mark：本地成交 / 报价 → 未完全公开的 EWMA → Index ±5% 围栏
```

需要保留的不确定点是：Bitget 不同官方页面对休市 Mark 的输入描述并不完全一致，一处偏向“最新成交”，另一处偏向“订单簿报价”；Mark EWMA 的时间常数、更新频率和单步限制也没有像 Internal Index 那样完整公开。

## 关键风险与观察框架

### 1. Impact Price防插针，但不等于只受真实成交影响

Impact Price 是基于当前盘口进行的模拟成交。它确实比 Last 更难被单笔成交推走，但盘口可以撤单、重挂和分层报价。操纵者不一定要把所有价格层真实吃完，仍可能通过改变可见深度影响测量结果。

因此不能把 `2,000 USDT Impact Notional` 误读成“花 2,000 美元即可操纵 Index”，也不能反过来把 Impact 视为完全免疫 spoofing。真正的成本取决于要移动多大一段盘口、维持多久、竞争做市商是否补单，以及 EWMA 会吸收多少。

### 2. 最关键的安全比率不是测量尺大小，而是清算放大倍数

作者提出的研究方向比单看 Impact Notional 更有用：

```text
Liquidatable Open Interest / Oracle Manipulation Cost
```

如果把 Internal Index 推进 `1%` 需要承担的持续盘口风险很小，而穿过这一价格区间会释放数十倍的被动清算单，那么主动资本可能借助风险系统放大自己的影响。经济安全边界取决于“推动价格的成本”与“价格移动后释放的强制订单”之间的比例。

### 3. Oracle可能观察到自己制造的订单流

休市模式下最危险的反馈环是：

```text
卖压 → Impact Mid下降 → Index下降 → Mark下降
→ 多仓清算 → 强制卖单进入同一本订单簿
→ Impact Mid继续下降 → 下一轮清算
```

到第二轮以后，Oracle 看到的已不只是外部信息导致的重新估值，还包含了它自己先前移动后触发的清算订单。价格源与风险系统失去部分独立性，温度计开始变成发热器。

### 4. 平滑只能在抗操纵和真实价格响应之间取舍

较慢的 EWMA 能过滤短暂异常，但遇到真实的重大新闻也会落后；较快的 EWMA 能及时反映新信息，却降低了维持异常盘口所需的时间。不存在同时满足“真消息一分钟完全反映”和“攻击者必须维持半小时才被采纳”的单一参数。

### 5. 动态偏离限制不是熔断

周末 `Mark = Index ±3%` 只限制相对距离。Index 从 `100` 逐步移动到 `103`、`106` 后，Mark 上限也可依次移动到 `103`、`106.09`、`109.18`。它能把瀑布摊平、给仓位更多反应时间，却不能阻止 Index 持续移动时发生分批清算。

### 6. 周末缺少最干净的现金股票套利腿

BTC 永续偏离现货时，套利者可同步买现货、卖永续；股票休市时，`Short Stock Perp + Long Cash Equity` 这条腿暂时不可执行。bStock、其他股票永续、ETF、期权和相关股票只能提供代理对冲，基差、相关性与库存风险都会削弱纠错速度。

### 7. 30秒交接斜坡是研究假设，不是现成利润

若内部 Index 与重新上线的外部 Index 相差很大，线性混合会产生可计算的方向路径。但能否交易取决于真实更新时间、订单延迟、Mark 与成交价的关系、清算触发方式、盘口深度、手续费和竞争者。原文第一篇没有提供这一路径的系统回测，因此应将其视为待验证的执行假设。

## 扩散分析 / 延展思路

### 1. 建立“模式感知”的价格矩阵

普通价差监控只比较 Last 不够。对每个股票永续至少应记录：

- 当前现金市场 session 与交易所定价模式；
- 外部 reference、Internal Index、Mark、Last、BBO 与 Impact Price；
- Mark 相对 Index 的护栏使用率；
- 资金费率、OI、估计清算密度；
- 模式切换时间和实际 blend 路径；
- bStock、其他 venue 永续与可用代理对冲价格。

这样才能区分“可成交价差”“Index 平滑滞后”“Mark 风控滞后”和“即将发生的模式切换”。

### 2. 把系统安全拆成三张曲线

更完整的实证研究不应只反推 EWMA 参数，而应同时估计：

1. 移动 Impact Mid `x bp` 所需的持续盘口资本；
2. Index 与 Mark 穿过该区间的时间；
3. 该区间可能触发的清算名义金额。

三张曲线合在一起，才接近平台休市定价的真实脆弱性。

### 3. 与库内已有材料的关系

本文是对 [`RWA永续套利复盘`](RWA永续套利复盘：周末指数错配、资金费率与海力士ADR溢价.md) 的底层补充：后者从交易者角度讨论周末 Index 恢复和跨所价差，本文解释错价为何会由 Impact、EWMA、Mark 护栏和模式交接机械地产生。它也延续了 [`永续合约十年进化史`](永续合约十年进化史：从312、ADL与保险基金到数据盘的结构性漏洞.md) 的主题：结构性风险已从单一合约公式迁移到价格源、订单簿与清算系统之间的耦合。

## 风险与限制

- 本文基于作者对交易所规则文件的整理，库内未对每个参数做独立在线复核；交易所可随时调整公式、窗口和保护带。
- Binance EWMA 约 `300 秒` 是作者实测估计，不是原文所称的官方公开参数。
- Bitget 休市 Mark 的输入与平滑参数存在官方描述歧义，不能据此精确复刻风险引擎。
- `Impact Notional`、清算密度和操纵成本之间不是线性关系；没有 L2、账户与清算逐笔数据，无法直接证明存在可盈利攻击。
- 原文是三部曲第一部分，对其他交易所、策略与数据实证的预告不能当成本篇已经完成的结论。
- 文中例子主要用于解释机制，不构成交易建议。

## 一句话结论

**股票永续休市后真正的变化不是多了周末交易，而是本地订单簿开始生产 Index，Index 再驱动 Mark 与清算；这带来价格发现能力，也把 Oracle、杠杆和强制订单连成了可能自我放大的反馈环。**
