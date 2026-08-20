# 原文归档：beiyue66 Robinhood链 Ponds v1 流水庄

## 主帖信息

- 作者：`@beiyue66`（beiyue1）
- 原文链接：<https://x.com/beiyue66/status/2089288325790122322?s=20>
- 发布时间：`2026-08-17 17:48`（Asia/Shanghai）
- 抓取工具：`twitter-cli`
- 抓取时互动数据：`53` likes，`7` retweets，`47` replies，`0` quotes，`10539` views，`56` bookmarks
- 是否有配图：有
- 配图文件：[`assets/HP6jfBEaMAAPIJH.png`](assets/HP6jfBEaMAAPIJH.png)

## 主帖结构化摘要

作者观察到 Robinhood 链上存在一类围绕 FOMO APP `Graduated` 榜单的流水庄玩法。

主帖核心信息：

- 流水庄依靠抢 FOMO APP 上的 Robinhood `Graduated` 排行榜获取注意力；
- 早期使用的是 `Ponds v1` 池；
- `Ponds v1` 直接在 `Uniswap v4` 上加池，不需要额外锁定流动性，因此低成本适合批量发射；
- `Ponds v1` 被官方关闭后，新发射统一走 `Ponds v2`；
- `Ponds v2` 更接近 `pump` 内盘模式，需要毕业迁移，并在毕业后锁定部分流动性，因此发射成本提高；
- 有一批人在十几天前已经批量创建了几千个 `Ponds v1` 老池；
- 虽然 v1 新池入口关闭，但这些老池仍可被用来刷交易量、冲排行榜；
- 作者提前扫出这批代币并埋伏了一段时间，直到这类流水暂时消停。

## 配图内容

配图是 FOMO APP 的 `Tokens -> Graduated` 榜单界面。

截图中可见多个代币在短时间内上榜，界面展示字段包括：

- 代币名称；
- 上榜或毕业时间；
- 交易量；
- 市值；
- 涨幅。

截图示例包括 `BULLS`、`GOAT`、`钱来`、`MARIJUANA`、`bALL`、`Stonks`、`Starbucks`、`YOLOer` 等条目，其中部分代币显示极高短期涨幅和较高成交量。

## 有信息量的回复

### 主网和BSC的历史类比

有评论提到，以前主网和 `BSC` 上也有不少类似流水庄：

- 发币；
- 加池；
- 用费用或刷量加速进入 `Dexscreener` 排行榜；
- 后续争取 `CMC`、交易所或其他注意力入口。

作者回复称现在也有类似结构，但很多已经变成捆绑发射，不好薅。

### SOL榜单的差异

有评论让作者观察 `FOMO` 榜单上的 `SOL` 流水。

作者回复说已经研究过，认为那边多是捆绑发射：

- 发射成本约 `3 SOL`；
- 开盘拉升后用很小金额不断买卖；
- 机器人难以抢到筹码；
- 因此外部参与价值不高，更适合屏蔽。

### 机器人层

有评论认为 Robinhood 上夹子机器人也有收益空间，说明该市场里除了流水庄和扫池者，还有围绕新买盘与低流动性成交的 MEV/夹子层。

## 归档说明

`twitter status --yaml` 在当前环境返回本地 Keychain cookie 权限错误，但直接 `twitter tweet` 能成功读取该推文详情。尝试将完整 JSON 重定向到文件时仍触发认证错误，因此本归档保存为结构化摘要、元数据和配图文件。
