# 原文归档：Cody_DeFi 链上股票 AMM 做市与相关资产 LP

## 主帖信息

- 作者：`@Cody_DeFi`（Cody）
- 原文链接：<https://x.com/Cody_DeFi/status/2090373390993633505?s=20>
- 发布时间：`2026-08-20 17:40`（Asia/Shanghai）
- 抓取工具：`twitter-cli`
- 抓取时互动数据：`41` likes，`8` retweets，`14` replies，`0` quotes，`7722` views，`43` bookmarks
- 是否有配图：无
- 引用推文：`@haydenzadams` 发布 X Article `Correlated Pairs: How AMMs Win the Biggest Markets`

## Cody主帖结构化摘要

Cody 讨论 Hayden Adams 关于用 AMM 做链上股票做市的文章。

主帖核心观点：

- 如果一个人本来就愿意持有某个资产，那么他做市的库存成本可能低于传统 market maker；
- 传统 market maker 通常需要保持 `delta neutral`，因此需要对冲价格波动，对冲成本较高；
- AMM 可以被理解成一种非对称收费型网格交易；
- 如果做 `NVDA/USDC`，`NVDA` 上涨时 AMM 会逐步把 `NVDA` 卖成 `USDC`，产生卖飞风险，也就是常说的无常损失；
- Hayden 提出“相关资产配对”思路，例如不做 `NVDA/USDC`，而做 `SPY/NVDA`，用相关资产降低无常损失，同时保留股票多头敞口并赚取手续费；
- 这个思路类似 crypto 中生态资产经常与 `ETH`、`SOL` 等 base asset 配对。

Cody 自己的简单回测：

- 用过去三年 `NVDA` 与 `SPY` 数据粗测；
- `NVDA/SPY` LP 的期末无常损失约 `-10.8%`；
- 在手续费不复投情况下，大约需要 `11.5%` 年化手续费才能追平单纯持有；
- 以当前链上股票代币的流动性和交易量看，这个手续费门槛暂时偏高；
- 股票代币非交易时段基本没有足够手续费收入，`SPY/NVDA` 对应交易对少且 APY 低。

Cody 当前实盘方向：

- 相比配对 LP，他自己测试的是 `USDC/NVDA`、`USDC/CRCL`；
- 核心是把 AMM 当成收费网格；
- 结合传统金融估值模型设置 LP 区间；
- 目标是把持股收益转化成 `持股 + 手续费 + 网格高抛低吸` 的混合收益；
- 该策略仍在实盘验证，需要更长周期。

## Hayden文章要点

Hayden Adams 的 X Article 标题是：

```text
Correlated Pairs: How AMMs Win the Biggest Markets
```

文章主要观点：

- 代币化不只是让传统市场更快、更便宜、更 24/7，而是改变市场本身的可编程性；
- AMM 已经在长尾资产和稳定币配对中证明了优势；
- 传统金融做市商把资本、策略、执行、结算和分发捆在一起；
- 区块链把这些模块拆开，让资本和库存偏好成为新的做市优势来源；
- 专业做市商要对冲非美元敞口，而愿意持有底层资产的 LP 可以用更低成本承担库存；
- 流动性会自然流向相关资产配对，例如 `NVDA/SPY`，再通过 `SPY/USD` 这类 bridge pair 回到美元；
- crypto 中已经出现类似结构：生态资产对 `ETH` 或 `SOL`，稳定币互相配对，少数深度桥接对连接资产簇；
- Robinhood Chain 上已经出现 tokenized stocks 对 `SPY` 的 Uniswap pools；
- Uniswap v4 hooks 可以继续提升 LP 回报，例如把未用于 swap 的资金用于 lending yield。

## 有信息量的回复

- Cody 回复读者：无常损失本质上就是网格卖飞的收益差，但期间可以赚手续费。
- 有读者指出：`IL` 能否被手续费覆盖是核心变量，股票 token 还有非交易时段流动性断层。
- Hayden 回复半导体 pair 问题：ETF 可以成为半导体单股的 base pair。
- 有评论追问周末/夜间传统市场关闭时，谁给相关股票簇定锚；Hayden 回复称午夜看起来运行尚可，并附图。
- 有批评者指出：Uniswap LP 长期暴露于 LP 未必愿意或充分理解的 delta risk；tokenized stock 属于成熟资产，价格发现主要在链下，相关资产池仍有风险。

## 归档说明

`twitter article` 对 `x.com/i/article/2089523718137466880` 返回 `Article not found`，因此本归档未保存 Hayden Article 全文文件。本文依据 `twitter tweet` 返回的 Cody 主帖、Hayden articleText 摘要和有信息量回复做结构化归档。
