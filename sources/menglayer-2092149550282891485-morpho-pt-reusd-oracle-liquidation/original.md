# 原文归档：MengLayer Morpho PT-reUSD 预言机清算事件

## 主帖信息

- 作者：`@MengLayer`（陈小萌）
- 原文链接：<https://x.com/MengLayer/status/2092149550282891485?s=20>
- 发布时间：`2026-08-25 15:18`（Asia/Shanghai）
- 抓取工具：`twitter-cli`
- 抓取时互动数据：`30` likes，`1` retweet，`17` replies，`0` quotes，`6166` views，`15` bookmarks
- 是否有配图：有
- 主帖配图：[`assets/HQjO7M1bAAAKSyp.jpg`](assets/HQjO7M1bAAAKSyp.jpg)

## 外部链接

- Morpho 清算机制：<https://docs.morpho.org/learn/concepts/liquidation/>
- Pendle PT 说明：<https://docs.pendle.finance/pendle-v2/ProtocolMechanics/YieldTokenization/PT>
- Pendle YT 说明：<https://docs.pendle.finance/pendle-v2/ProtocolMechanics/YieldTokenization/YT>
- Pendle `PYLpOracle`：<https://docs.pendle.finance/pendle-v2-dev/Contracts/Oracle/PYLpOracle>
- Pendle PT / LP oracle 集成说明：<https://docs.pendle.finance/pendle-v2-dev/Oracles/HowToIntegratePtAndLpOracle>

## 主帖结构化摘要

作者称 Morpho 刚刚清算了 `30m+` 的 `PT REUSD`。

主帖核心信息：

- 作者将事件归因为预言机攻击；
- 攻击者从 Gate 提取部分资金；
- 攻击者在市场上买入 `YT`；
- 该操作短时间把市场隐含收益率 `IY` 推到 `20%+`；
- Morpho 池子的 TWAP 价格随后暴跌；
- 一批接近清算线的循环贷仓位被清算；
- 作者称攻击者通过清算赚了约 `25wU`；
- 作者提醒 Loop 有风险，开循环前要看清楚 oracle。

## 配图内容

主帖配图是一张 `All Liquidations` 列表截图。

截图字段包括：

- `Date & Time`
- `Liquidated Wallet`
- `Collateral Seized (PT-reUSD-10DEC2026)`
- `Loan Repaid`

截图第一页可见清算时间集中在 `2026-08-25 12:41:35` 到 `12:48:35`。

可见行包括：

- `0xf991...2691` 被清算约 `7,350,330.45 PT-reUSD-10DEC2026`，对应约 `$7.11m` 抵押品，偿还约 `6,834,782.19 USDC`；
- `0x9f09...f363` 被清算约 `1,233,380.7 PT-reUSD-10DEC2026`，对应约 `$1.19m` 抵押品，偿还约 `1,145,155.71 USDC`；
- `0x6554...f438` 被清算约 `855,192.27 PT-reUSD-10DEC2026`，对应约 `$828.11k` 抵押品，偿还约 `791,807.54 USDC`；
- 其他可见行还包括几十万到几万美元级别的抵押品清算。

截图第一页可见 `Loan Repaid` 合计约 `10.10m USDC`。主帖称总清算规模为 `30m+`，因此截图应只是部分清算列表。

## 有信息量的回复

### PT oracle 不同 pool 不一样

`@trader_alvin` 表示，以为现在 `PT` 都 hardcode 成线性调整函数。

作者回复：不同 pool 不一样。

这个补充说明，不能把 PT 的到期收敛特性直接等同于借贷市场使用的 oracle 方式。不同市场可能采用线性折价、Pendle AMM TWAP 或其他封装。

### Loop 用户大意

有读者说一直不敢玩 loop，总觉得很脆弱。

作者回复：这波很多人大意了。

### Morpho 与 Pendle 的角色区分

`@w3bD4nny` 发表长回复，指出清算发生在 Morpho，不是 Pendle。Pendle 只是 `PT / YT` 交易市场，本身不提供借贷、抵押或清算机制。攻击者是在 Pendle 上影响价格输入，再让 Morpho 上的仓位触发清算。

该回复还提到：

- 某市场 `LLTV` 为 `91.5%`；
- 按 Morpho 的 `LIF` 公式，清算奖励约 `2.61%`；
- 若总清算债务约 `3613 万美元`，理论奖励上限接近 `90 多万美元`；
- 实际攻击者净赚约 `25 万 U`，可能受竞争、价格恢复、部分清算等因素影响；
- 普通人机制上可以清算，但实际多由专业清算机器人和 MEV 搜索者竞争。

这条回复提供了有用的机制解释，但主帖没有给出完整链上参数和交易哈希，因此应作为评论区补充归档。

## 官方文档背景

Morpho 官方文档说明：

- 借款仓位的 LTV 超过市场 `LLTV` 时，仓位可被清算；
- 清算人可以偿还债务，并拿走相应抵押品和清算奖励；
- 清算奖励由 `Liquidation Incentive Factor` 决定；
- 清算使用当前 oracle price 判断和计价；
- 清算是 permissionless，竞争者通常包括清算机器人和 MEV 搜索者。

Pendle 官方文档说明：

- `PT` 代表生息资产的本金部分，到期可按 accounting asset 赎回；
- `YT` 代表收益部分，提供对未来收益的杠杆化敞口；
- Pendle `PYLpOracle` 可以按 TWAP implied rate 推导 `PT`、`YT` 和 LP 价格；
- Pendle 文档也提示多数新集成推荐使用 `Linear Discount Oracle`，因其更抗操纵且与 AMM 无关；TWAP oracle 仍适用于需要市场驱动价格的集成。

## 归档说明

`twitter status --yaml` 在当前环境返回本地 Keychain cookie 权限错误，但直接 `twitter tweet` 成功读取主帖、主要回复和配图 URL。配图已下载到 `assets/`。本归档保存主帖结构化摘要、截图信息、关键回复和官方文档背景；未进行独立链上交易级复核。
