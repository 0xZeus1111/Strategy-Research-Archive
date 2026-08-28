# Curve有毒报价池：eth_call默认值、USDC折扣oracle与quote执行错配

## 原文信息

- 作者：`wavey0x`
- 原文链接：<https://gist.wavey.info/B4lu8GXEyJQvlu4E5nJuyugF>
- 原文标题：`The Curve Pool That Lies To Quotes`
- 创建时间：`2026-06-24 21:57`（Asia/Shanghai）
- 最新编辑：`2026-06-25 00:48`（Asia/Shanghai）
- 内容类型：`DeFi市场微结构 / Curve rate oracle / toxic liquidity / 聚合器报价安全`
- 是否有配图：无正文配图，仅页面作者头像
- 原文归档：[`sources/wavey0x-b4lu8gxeyjqvlu4e5njuyugf-curve-toxic-quote-oracle/`](sources/wavey0x-b4lu8gxeyjqvlu4e5njuyugf-curve-toxic-quote-oracle/)
- 相关本地分析：
  - [`Uniswap_v4_Hook动态费陷阱：报价模拟、路由盲区与LP收益掠夺.md`](Uniswap_v4_Hook动态费陷阱：报价模拟、路由盲区与LP收益掠夺.md)
  - [`有毒Uniswap_v4_Hook：gas分支、quote执行错配与聚合器防御.md`](有毒Uniswap_v4_Hook：gas分支、quote执行错配与聚合器防御.md)

## 主题

这篇 gist 分析的是一个 Curve `USDC/USDT` 池 `PB USDC USDT v4a100`。表面上它像普通稳定币池，但它的 rate oracle 会根据调用上下文返回不同价格：常见离线报价模拟使用的 `eth_call` 默认环境，会让 USDC rate 进入折扣分支；真实交易上下文则看到正常市场价。

作者的核心判断是：

**这个池不是靠普通滑点赚钱，而是让聚合器和离线模拟在 quote 阶段看到更便宜的 USDC，从而在 `USDT -> USDC` 方向给出过高输出预期；真实执行时 oracle 回到正常 rate，用户收到的 USDC 少于模拟支持的数量。**

它和最近的 Uniswap v4 Hook 案例同属一类问题：

```text
quote 环境看到的规则 != execution 环境实际执行的规则
```

区别在于，Uniswap v4 案例把差异藏在 Hook 动态费率里；这篇 Curve 案例把差异藏在外部 rate oracle 里。

## 机制拆解

### 1. Curve 池为什么会受 rate oracle 影响

Curve Stableswap-NG 支持带 rate oracle 的资产。池子在计算内部标准化余额和报价时，会读取每个资产对应的 rate。

这对正常资产是必要功能，例如：

- 封装资产；
- 生息资产；
- 需要汇率换算的资产；
- 价格或兑换率不是固定 `1:1` 的代币。

问题是，如果池子依赖的外部 rate oracle 本身带有上下文分支，那么 Curve 池的报价也会被这个 oracle 带偏。

因此，这不是“Curve 公式坏了”，而是：

```text
Curve 池允许读取外部 rate oracle
  -> 外部 oracle 根据调用上下文撒谎
  -> 池子的 get_dy / stored_rates 被污染
  -> 聚合器得到错误 quote
```

### 2. `eth_call` 默认值成了可识别信号

作者指出，这个 oracle 利用了常见 `eth_call` 默认环境：

- `gasPrice = 0`
- `tx.origin = address(0)` 或没有真实交易发起上下文

当调用落入这种环境时，USDC oracle 会返回带 `48 bps` 折扣的 rate。真实交易上下文则返回正常 market rate。

可以把它理解成：

```text
模拟报价：USDC 被 oracle 报得更便宜
真实执行：USDC 恢复正常价格
```

在 `USDT -> USDC` 方向，这会让离线 quote 显示“同样 USDT 可以换到更多 USDC”。但真实交易执行时，池子不再按折扣 rate 计算，实际能拿到的 USDC 更少。

这就是 toxic direction。

### 3. 反方向不是主要受害方向

作者强调，`USDC -> USDT` 方向符号会反过来。

因为 `eth_call` 下 USDC 被低估，所以从 USDC 换 USDT 时，离线模拟反而会低估输出。真实交易上下文下，输出可能看起来比 quote 更好。

所以这个池的核心受害方向不是所有交易，而是：

```text
USDT -> USDC
```

这点很重要。很多毒池不是“所有方向都坑”，而是只有某个方向会系统性让 quote 高估成交结果。

## 直接证据

作者列出两个 rate-oracle adapter：

- Curve 池：<https://etherscan.io/address/0xe60B5E323D72a914b089f137eC9b3aB91ae24A65>
- USDC oracle：<https://etherscan.io/address/0x09e889BE79da10ABe97116Ef1aFCF5A25509FC16>
- USDT oracle：<https://etherscan.io/address/0xA7Bf5E68d96fc20C4FC95420fe20d69bb1301Da9>

USDC oracle 在不同调用上下文中的 `getRate()` 返回值：

| Call context | `getRate()` |
| --- | ---: |
| `from` omitted, `gasPrice = 1` | `0.994871584` |
| nonzero `from`, `gasPrice = 0` | `0.994871584` |
| nonzero `from`, `gasPrice = 1` | `0.999670000` |

这里能看出，关键不只是有没有 `from`，而是 `gasPrice = 0` 也足以把调用送进折扣分支。

Curve pool 的 `stored_rates()` 对比：

| Context | Stored rates |
| --- | --- |
| Via `eth_call` | `[0.994871584, 0.99872873]` |
| Via transaction context | `[0.99967, 0.99872873]` |

`get_dy` 示例：

| Input | Direction | Via `eth_call` | Via transaction context | Difference |
| --- | --- | ---: | ---: | ---: |
| `1,000` USDT | USDT -> USDC | `1002.988043` USDC | `998.222250` USDC | `-47.52 bps` |
| `1,000` USDC | USDC -> USDT | `995.322982` USDT | `1000.074945` USDT | `+47.74 bps` |

第一行是风险核心：模拟报价比交易上下文支持的输出多了约 `47.52 bps`。

对稳定币池来说，几十个 bps 已经足够大。它不会像 `12.8%` Hook fee 那样刺眼，但因为池子看起来是稳定币换汇，用户和聚合器更容易默认它是低风险路径。

## 损害估算

作者扫描了该池历史 swap，并给出以下估计：

| Metric | Result |
| --- | ---: |
| Total pool swaps | `124,172` |
| USDT -> USDC swaps compared | `68,699` |
| USDT sold volume | `68.058098M` USDT |
| Actual USDC bought volume | `67.926090M` USDC |
| Sum quoted via `eth_call` | `67.980535M` USDC |
| Sum quoted via transaction context | `67.863652M` USDC |
| Gross adverse quote delta | `225,476.476233` USDC |
| Signed quote-minus-transaction delta | `116,882.591250` USDC |

这里最容易误读的是 `225,476 USDC`。

它不是攻击者钱包最终利润，也不是用户实际已经被直接转走的净额，而是 toxic direction 上“离线 quote 比交易上下文多承诺了多少”的 gross adverse quote delta。

换句话说，它衡量的是欺骗面：

```text
模拟环境承诺的输出 - 真实交易上下文支持的输出
```

作者还指出，反方向 `USDC -> USDT` 的 gross basis 上，交易上下文输出比 `eth_call` 模拟高出 `223,957.960803 USDT`。这会抵消一部分 signed delta，也说明不能把单向损害数字直接等同于攻击者收益。

## 攻击者利润与 LP 证据

作者把攻击者收益拆成 wallet accounting，而不是直接拿 quote delta 当利润。

### 1. deployer / oracle owner

作者标记的 deployer / oracle owner：

<https://etherscan.io/address/0x215cB1AECdd7D392F793680017960748174b5Fd6>

该地址：

- 部署了池子；
- 拥有两个 oracle adapter；
- 直接 LP；
- 后续退出了直接 LP 仓位。

直接 LP 记账：

| Token | Deposited | Withdrawn | Net |
| --- | ---: | ---: | ---: |
| USDC | `46,801.856700` | `54,801.271263` | `+7,999.414563` |
| USDT | `40,444.440486` | `62,390.861719` | `+21,946.421233` |
| Combined stable units | `87,246.297186` | `117,192.132982` | `+29,945.835796` |

### 2. linked LP

作者还把当前最大 LP 地址纳入攻击者相关账户：

<https://etherscan.io/address/0x159a7a9c58860bce0ea567d7371fb46ef2fdd9de>

归因理由包括：

- 入场前由 owner 提供资金；
- 共享同样的 `23-byte delegation code`；
- 后续部署过多个 toxic `PB USDC/USDT` 变体；
- 仍是当前最大 LP。

这一行不是完全 realized PnL，而是把已提取金额和当前 LP 可平衡退出价值一起计入。

综合作者的口径：

| Bucket | Net |
| --- | ---: |
| Deployer/oracle owner, realized | `+29,945.835796` |
| linked LP, marked current | `+4,647.031453` |
| Combined attacker wallets | `+34,592.867249` |

所以标题里的 `$34,592.87` 是攻击者相关钱包的记账净收益，不是全体用户损失，也不是 quote delta。

## 为什么损害大于利润

这篇的一个关键优点，是作者没有把“被误导的 quote 差额”直接说成攻击者利润。

两者口径不同：

- `225,476 USDC` 是 gross adverse quote delta，衡量 toxic direction 的报价欺骗空间；
- `34,592.87` 是攻击者相关钱包的资产增值，考虑了存入、取出、当前 LP 价值、库存、反方向 flow 和 Curve 价格动态；
- 即使攻击者是主要 LP，也不代表能吃掉全部 quote delta；
- 稳定币池里双向流量、库存变化和真实成交价格会把毛损害转化为更小的可归因收益。

这对研究链上事件很重要：损害面、用户损失、LP 收益、攻击者净利润是四个不同概念，不能混用。

## 和 Uniswap v4 Hook 案例的关系

最近两篇本地文档分析的是 Uniswap v4 Hook 动态费陷阱。

这篇 Curve 案例与它们的共同点是：

- 都利用模拟环境和真实执行环境的差异；
- 都让聚合器在 quote 阶段看到更优结果；
- 都把“交易成功但到手变差”作为隐蔽损害；
- 都需要聚合器做多环境模拟和执行后对账；
- 都不能只靠用户肉眼判断。

区别在于：

| 维度 | Uniswap v4 Hook 案例 | Curve rate oracle 案例 |
| --- | --- | --- |
| 差异来源 | Hook 动态 fee override | 外部 rate oracle |
| 识别信号 | gas envelope / gasleft 等 | `tx.origin` / `gasPrice` 默认值 |
| 表面形态 | LP fee 突然变高 | USDC rate 在 quote 中被折扣 |
| 风险方向 | 取决于 Hook 费率逻辑 | 主要是 `USDT -> USDC` |
| 结果特征 | 高费更显眼 | 几十 bps 更像普通稳定币价差 |

这说明 toxic liquidity 不一定只发生在新协议。只要一个交易池允许外部组件影响报价，而且外部组件能识别模拟环境，就可能出现 quote / execution mismatch。

## 风险与限制

### 1. 本文未独立重跑 124,172 笔 swap

原文给出了完整方向、关键地址、上下文对比和损害估算，但本文没有独立扫描所有交易。

因此：

- 数字按原文归档和分析；
- 钱包关联证据按原文口径记录；
- 当前最大 LP 状态可能随时间变化；
- 若要用于实盘风控，应重新拉链上数据复核。

### 2. 不能泛化为“所有 Curve 池都有问题”

这个案例依赖特定 pool 的 rate oracle adapter。

Curve 支持 rate oracle 是正常功能。风险来自：

- oracle 由谁控制；
- oracle 是否有上下文分支；
- `stored_rates()` 在 quote 和真实交易中是否一致；
- 聚合器有没有把默认 `eth_call` 当作最终真相；
- 该池是否被 allowlist 或历史监控覆盖。

所以结论不是“Curve 不能用”，而是“带外部 rate oracle 的池必须检查 oracle 行为”。

### 3. 几十 bps 在稳定币池里已经很危险

`47 bps` 在波动资产交易里可能被滑点噪音掩盖，但在 `USDC/USDT` 稳定币池里，这是非常大的系统性差异。

它可能造成：

- 聚合器错误选路；
- 用户以为只是普通稳定币兑换；
- 小额交易不明显，大额流量累计损害很大；
- 攻击者以 LP 形式慢慢承接收益，而不是一次性大额转走。

### 4. 技术附录是重构，不是官方源码

原文附了 oracle adapter 的人类可读重构。它对理解行为有价值，但仍应区分：

- 链上可直接验证的返回值和地址关系；
- 作者根据 selector 与行为恢复出的代码结构；
- 对 owner、linked LP 和部署变体的归因推断。

防御上不需要依赖完整源码名，只需要发现同一个 oracle 在不同上下文下返回不同 rate。

## 防御检查表

### 1. 聚合器 / 路由器

- 不要用默认 `eth_call` 结果直接作为唯一 quote；
- 对同一 `get_dy` 同时跑默认 `eth_call` 和 transaction-like `eth_call`；
- 显式设置真实 `from`、非零 gas price 或等价交易字段；
- 比较 `stored_rates()` 在不同上下文下是否一致；
- 单独标记带外部 rate oracle 的 Curve 池；
- 对 owner 可改参数的 oracle 降权或隔离；
- 对 quote 与 receipt 的实际 output 偏差做方向性统计；
- 发现某方向长期过度承诺时，加入 toxic pool 列表。

### 2. 钱包 / 前端

- 对稳定币池也展示最低到手数量，而不是只展示“低滑点”；
- 对外部 oracle 池标记额外风险；
- 大额稳定币兑换不要隐藏路径细节；
- 对新池、奇怪名称池、非主流 LP 地址集中池做提示；
- 如果 quote 与成交长期偏离，应自动换路或提示用户。

### 3. 研究者

- 分开统计 `USDT -> USDC` 和 `USDC -> USDT`，不要把双向流量直接合并；
- 分开记录 gross quote delta、signed delta、LP 收益和攻击者钱包 PnL；
- 对 rate oracle 做多上下文调用；
- 关注 owner 是否仍可修改 `discountBps`；
- 追踪 LP 地址之间的 funding、delegation code 和后续部署行为；
- 对同一模式的新池变体做地址簇识别。

## 扩散分析 / 延展思路

### 1. 稳定币池的风险不只来自 depeg

很多人看稳定币池，只关注：

- 两个稳定币是否都可靠；
- TVL 是否够大；
- 费率是否低；
- 历史价格是否接近 `1:1`。

这篇说明，还要看池子如何获得 rate。

如果 rate oracle 会根据模拟环境返回不同值，那么即使 USDC 和 USDT 本身没有 depeg，池子也能在报价层制造系统性误差。

### 2. 聚合器需要从“价格搜索”升级为“上下文一致性检查”

过去路由器主要问：

```text
哪条路径输出最多？
```

现在还要问：

```text
这条路径在真实交易环境下还是这个输出吗？
```

这要求聚合器把所有外部可调用组件都当成潜在状态机：

- rate oracle；
- hook；
- ERC4626 `convertToAssets`；
- rebasing token；
- fee-on-transfer token；
- router callback；
- 任何能读取调用上下文的合约。

只要 quote 和 execution 的上下文不同，攻击者就可能把差异变成收费通道。

### 3. 链上风控要记录“方向性”

这个案例里，`USDT -> USDC` 是 toxic direction，反方向并不是同样损害。

因此风控不能只给 pool 一个统一标签，还要记录：

- 哪个方向 quote 被高估；
- 哪个方向 quote 被低估；
- 哪种调用上下文触发；
- 该方向的历史成交量；
- 实际成交偏差是否长期同号。

否则很容易把风险平均掉，看不到真正受害路径。

## 一句话结论

这篇 gist 的核心不是 Curve 稳定币池普通滑点，而是外部 rate oracle 利用 `eth_call` 默认上下文给 quote 喂入折扣 USDC rate，导致 `USDT -> USDC` 离线报价系统性高估输出；对聚合器来说，真正要防的是所有能让 quote 与 execution 看到不同规则的上下文敏感组件。
