# Morpho_PT-reUSD清算事件：Pendle_YT拉升、TWAP预言机与Loop风险

## 原文信息

- 作者：`@MengLayer`（陈小萌）
- 原文链接：<https://x.com/MengLayer/status/2092149550282891485?s=20>
- 发布时间：`2026-08-25 15:18`（Asia/Shanghai）
- 内容类型：`DeFi借贷风险 / 预言机操纵 / PT-YT结构 / 循环贷清算`
- 是否有配图：有，Morpho `PT-reUSD-10DEC2026` 清算列表截图
- 主帖配图：[`sources/menglayer-2092149550282891485-morpho-pt-reusd-oracle-liquidation/assets/HQjO7M1bAAAKSyp.jpg`](sources/menglayer-2092149550282891485-morpho-pt-reusd-oracle-liquidation/assets/HQjO7M1bAAAKSyp.jpg)
- 官方文档参考：
  - Morpho 清算机制：<https://docs.morpho.org/learn/concepts/liquidation/>
  - Pendle PT 说明：<https://docs.pendle.finance/pendle-v2/ProtocolMechanics/YieldTokenization/PT>
  - Pendle YT 说明：<https://docs.pendle.finance/pendle-v2/ProtocolMechanics/YieldTokenization/YT>
  - Pendle `PYLpOracle`：<https://docs.pendle.finance/pendle-v2-dev/Contracts/Oracle/PYLpOracle>
  - Pendle PT / LP oracle 集成说明：<https://docs.pendle.finance/pendle-v2-dev/Oracles/HowToIntegratePtAndLpOracle>
- 原文归档：[`sources/menglayer-2092149550282891485-morpho-pt-reusd-oracle-liquidation/`](sources/menglayer-2092149550282891485-morpho-pt-reusd-oracle-liquidation/)

## 主题

这条推文讲的是一次围绕 `PT-reUSD` 抵押品的 Morpho 清算事件。

作者给出的核心叙述是：攻击者先在 Pendle 市场里买入 `YT`，短时间把市场隐含收益率 `IY` 推到 `20%+`，导致与 `PT-reUSD` 相关的 TWAP 预言机价格下跌；Morpho 借贷市场使用该价格评估抵押品后，一批原本处在清算边缘的循环贷仓位被打到不健康状态，随后发生 `30m+` 规模清算，攻击者通过清算获利约 `25wU`。

这不是一篇单纯的“清算机会”推文。更准确地说，它是一个 DeFi 组合风险案例：

- Pendle 的 `PT / YT` 市场价格可以影响 PT 估值；
- Morpho 借贷市场依赖 oracle 价格判断 LTV 和清算；
- Loop 仓位把小幅 oracle 波动放大成强制清算；
- 市场驱动型 TWAP oracle 如果窗口、深度和资产结构不匹配，会被短时交易冲击利用。

## 事件机制

### 1. PT 与 YT 的关系

Pendle 把生息资产拆成两部分：

- `PT`：Principal Token，代表到期可赎回的本金部分，类似零息债；
- `YT`：Yield Token，代表到期前的收益权。

在简化理解里，同一底层资产被拆成：

```text
SY ≈ PT + YT
```

当市场愿意高价买 `YT`，意味着市场在给未来收益权更高估值。对应地，`PT` 作为本金部分的折价会扩大，隐含收益率 `IY` 上升，`PT` 当前价格下降。

所以作者说的“市价买了 YT，把市场 IY 短时间干到 20%+”，传导方向是：

```text
买入 YT
  -> YT 价格上升
  -> 隐含收益率上升
  -> PT 价格下降
  -> 以 PT 为抵押品的借贷仓位 LTV 上升
```

### 2. Morpho 为什么会触发清算

Morpho 的借贷市场会按 oracle 价格计算抵押品价值。

如果借款人的 LTV 超过该市场的 `LLTV`，仓位就可以被清算。Morpho 官方文档说明，清算人可以偿还借款人的债务，并按 oracle 价格拿走相应抵押品和清算奖励。

这次事件里，抵押品是 `PT-reUSD-10DEC2026`。当相关 oracle 价格下跌时：

- 抵押品估值下降；
- 债务规模不变；
- LTV 被动上升；
- 原本靠近清算线的 Loop 仓位先被击穿；
- 清算人偿还债务并拿走 PT 抵押品。

主帖截图显示的是 `All Liquidations` 列表，列名包括：

- `Date & Time`
- `Liquidated Wallet`
- `Collateral Seized (PT-reUSD-10DEC2026)`
- `Loan Repaid`

截图第一页可见多笔清算，时间集中在 `2026-08-25 12:41:35` 到 `12:48:35`，可见页的 `Loan Repaid` 合计约 `10.10m USDC`。主帖称总清算规模为 `30m+`，说明截图只是其中一页或一个截面。

### 3. 为什么 Loop 仓位最脆弱

Loop 的基本结构是：

```text
存入抵押品
  -> 借出稳定币
  -> 买入更多抵押品
  -> 再存入
  -> 继续借
```

这能放大收益，也会压缩安全垫。

对于 `PT-reUSD` 这类看似到期收敛、波动较低的抵押品，用户容易误以为主要风险是底层信用或到期赎回风险。但如果借贷市场使用的 oracle 跟 Pendle 市场价格联动，那么短时 `IY` 冲击也会直接打到抵押品估值。

Loop 仓位的问题是：

- 债务端通常是稳定币，短时间不下降；
- 抵押品 oracle 一跌，LTV 立刻上升；
- 杠杆越高，安全垫越薄；
- 一旦价格跌破 LLTV，清算是机械执行，不会等待价格恢复；
- 清算发生后，即使 oracle 之后回升，原仓位也已经被处理。

这就是作者提醒“开循环之前，要看好 Oracle”的核心含义。

## 攻击路径拆解

按主帖信息，攻击路径可以整理为：

```text
从 Gate 提取资金
  -> 在 Pendle 市场买入 YT
  -> 推高短时隐含收益率 IY
  -> PT 相关 TWAP 价格下跌
  -> Morpho 上 PT-reUSD 抵押品估值下降
  -> 边缘 Loop 仓位 LTV 超过 LLTV
  -> 攻击者或关联清算地址执行清算
  -> 获得清算奖励与价格错配收益
```

这里的关键不是“买 YT 能赚钱”，而是买 YT 被用作 oracle 输入操纵。

攻击收益来自两部分的组合：

- 用有限资金冲击一个相对可推动的 YT / PT 市场；
- 在另一个依赖该价格的借贷市场里收割被动清算。

这种结构的危险在于，风险不是单个协议内部闭环，而是跨协议传导。

Pendle 市场本身只是提供 PT/YT 交易，Morpho 本身只是按 oracle 和 LLTV 执行清算。问题发生在“把市场驱动价格接入高杠杆借贷市场”这条组合链上。

## 评论区补充

### 1. PT oracle 并不都一样

有读者说，以为现在 `PT` 都已经 hardcode 成线性调整函数。

作者回复：不同 pool 不一样。

这条补充很重要。Pendle 官方文档也提到，对多数新集成，推荐使用 `Linear Discount Oracle` 这类更抗操纵、与 AMM 无关的设计；但 TWAP oracle 仍然支持需要市场驱动价格的集成。

所以不能把“PT 到期趋近 1”理解成所有借贷市场都使用线性、不可操纵的定价。具体风险取决于：

- 该 Morpho 市场实际使用哪个 oracle；
- oracle 是线性折价、Pendle AMM TWAP，还是其他封装；
- TWAP 窗口多长；
- 市场深度能否承受冲击；
- 是否有 sanity check 或价格上限/下限。

### 2. Morpho 清算奖励解释

评论区有一条长回复指出，清算发生在 Morpho，而不是 Pendle。

这个区分是对的：

- Pendle 提供 `PT / YT` 交易市场；
- 攻击者在 Pendle 上改变了价格输入；
- Morpho 根据 oracle 价格判断仓位健康；
- 清算逻辑发生在 Morpho 借贷市场。

该回复还引用了 `91.5% LLTV` 和 Morpho 的 `LIF` 公式。按 Morpho 官方文档中的 Blue 市场公式：

```text
LIF = min(1.15, 1 / (0.3 * LLTV + 0.7))
```

如果 `LLTV = 91.5%`，则：

```text
LIF ≈ 1.0262
清算奖励 ≈ 2.62%
```

这与评论里提到的 `2.61%` 接近。

但需要注意：主帖没有给出具体市场参数页面、清算交易哈希或攻击地址，所以这部分应当作为评论区机制解释，而不是完整链上复盘结论。

### 3. “清算机会”不是普通人手动可捡

评论区也提到，Morpho 清算是 permissionless，任何人都可以清算。

机制上这是对的，但实际执行不等于普通人能稳定参与：

- 需要实时监控 oracle 和仓位健康度；
- 需要准备还款资金或闪电贷；
- 需要处理被清算 PT 的退出或再抵押；
- 需要和专业清算机器人、MEV 搜索者竞争；
- 交易路径要足够快，否则清算机会会被抢走；
- 如果 oracle 或 PT 市场很快恢复，处理抵押品还会有价格风险。

所以这篇材料的重点不是“哪里点清算”，而是“为什么循环贷仓位会被 oracle 输入击穿”。

## 风险与限制

### 1. 主帖是事件说明，不是完整链上审计

作者给出了机制、规模和利润估计，但主帖没有列出：

- 攻击地址；
- 清算交易哈希；
- Pendle 买入 YT 的交易哈希；
- Morpho market ID；
- oracle 合约地址；
- TWAP 窗口；
- 完整清算列表。

因此本文只能做结构化分析，不能把 `30m+` 和 `25wU` 当作已经由本文独立复核的最终数字。

### 2. 不能把所有 PT 抵押品都视为同一风险

PT 的风险取决于具体 oracle。

如果 oracle 是线性折价、与 AMM 价格脱钩，并且有充分 sanity check，那么单次 YT 买盘未必能影响借贷抵押品价格。

如果 oracle 使用 Pendle AMM 的短窗口 TWAP，且市场深度不足，那么短时 IY 冲击就可能变成清算触发器。

所以风险判断必须落到具体池子，而不是泛化成“PT 都危险”或“PT 都安全”。

### 3. Loop 收益经常低估 oracle 风险

循环贷界面通常会突出：

- 年化收益；
- 借贷利差；
- points；
- 资金利用率；
- 清算线。

但真正决定尾部风险的是：

- oracle 来源；
- oracle 可操纵成本；
- TWAP 窗口；
- 抵押品二级市场深度；
- YT / PT 的交叉价格传导；
- 清算机器人竞争强度；
- 价格恢复前清算是否已经完成。

如果这些条件没看清，所谓低波动抵押品也会在 oracle 层出现瞬时高波动。

### 4. 清算奖励不是无风险利润

按 `91.5% LLTV` 估算，`LIF` 大约提供 `2.62%` 清算奖励。

但清算人还要承担：

- gas；
- 竞争失败；
- 闪电贷成本；
- 被清算 PT 的退出流动性；
- PT 价格继续下跌；
- oracle 恢复后路径失效；
- 与操纵交易之间的成本配比。

所以“攻击者赚了 25wU”应理解为该事件中的估计结果，不应直接外推成清算策略的稳定收益率。

## 防御检查表

### 1. 对 Loop 用户

- 不只看 APY，要看 oracle；
- 不只看当前 LTV，要看 oracle 被打穿时的 LTV；
- 远离贴近 LLTV 的高倍循环；
- 对 PT 抵押品确认 oracle 是线性折价还是市场 TWAP；
- 看清 TWAP 窗口和底层市场深度；
- 如果 YT 市场很浅，不要默认 PT 抵押品稳定；
- 对新池、长尾底层、points 驱动资产降低杠杆。

### 2. 对借贷市场和 curator

- PT 抵押品优先使用更抗操纵的 oracle；
- market-derived TWAP 要配合足够长窗口和深度要求；
- 对 IY 的短时跳变设置 sanity check；
- 对单池流动性不足的 PT 降低 LLTV；
- 对可被 YT 小额推动的市场设置供应上限；
- 监控 YT 买盘、IY、PT oracle、Morpho LTV 的联动；
- 清算集中发生时及时暂停前端新增 loop 或提高风险提示。

### 3. 对研究者

需要把风险图画成跨协议链条，而不是单点协议问题：

```text
Pendle YT 市场深度
  -> IY
  -> PT oracle
  -> Morpho collateral value
  -> Loop LTV
  -> Liquidation bot / MEV
```

这条链上任何一个环节变薄，都可能让“固定收益仓位”突然变成可清算仓位。

## 扩散分析 / 延展思路

### 1. PT 的“到期确定性”不等于“借贷估值稳定”

PT 到期趋近 1:1，是到期赎回逻辑。

借贷市场里的清算，看的却是当前 oracle 价格。

如果当前 oracle 跟市场 IY 绑定，那么即使资产最终到期能收敛，用户也可能在到期前因为短时价格输入被清算。

这类似债券：到期还本不代表中途按市价抵押融资不会爆仓。

### 2. Points 和固定收益产品会诱导过度 Loop

PT / Pendle / Morpho 组合经常被包装成：

- 固定收益；
- points farming；
- 稳定币资产；
- 到期收敛；
- 低波动抵押。

这些叙事会鼓励用户加杠杆。

但越多人做同质化 Loop，清算边缘仓位越集中，攻击者越容易通过一次 oracle 冲击获得规模化清算池。

### 3. Oracle 是 DeFi 组合策略的第一风险变量

这篇最可迁移的教训是：任何跨协议收益策略，第一步不是算 APY，而是拆 oracle。

要问：

- 价格来自哪里；
- 谁可以推动这个价格；
- 推动成本多少；
- TWAP 多长；
- 是否有备用价格源；
- 是否有上下限保护；
- 价格变化会影响哪些借贷仓位；
- 清算是否 FCFS、是否被机器人垄断。

如果这些问题没有答案，Loop 收益再高也只是把 oracle 风险卖给自己。

## 一句话结论

这次 `PT-reUSD` 清算事件的核心不是 Morpho 或 Pendle 单点失效，而是市场驱动的 PT/YT 价格、借贷 oracle、Loop 杠杆和开放清算机制连成了一条可被攻击的传导链；做循环贷前真正要看的不是年化收益，而是 oracle 被短时冲击时仓位还能不能活下来。
