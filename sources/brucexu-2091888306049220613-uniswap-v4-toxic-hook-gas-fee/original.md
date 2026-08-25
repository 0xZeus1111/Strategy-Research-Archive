# 原文归档：brucexu.eth Uniswap v4 有毒 Hook 与聚合器防御

## 主帖信息

- 作者：`@brucexu_eth`（brucexu.eth）
- 原文链接：<https://x.com/brucexu_eth/status/2091888306049220613?s=20>
- 发布时间：`2026-08-24 22:00`（Asia/Shanghai）
- 抓取工具：`twitter-cli`
- 抓取时互动数据：`140` likes，`17` retweets，`34` replies，`3` quotes，`21160` views，`143` bookmarks
- 是否有配图：无
- 引用推文：`@BlockBloomer` 关于 BNB Chain `WBNB/USDC` Uniswap v4 Hook 池动态费率陷阱的分析

## 外部链接

- 代表交易：<https://bscscan.com/tx/0x7615dec2ce80506ec461a47739eb8533ac2bc7c605ca56c255964481b76363d0>
- `@EnsoBuild` 旧案例讨论：<https://x.com/EnsoBuild/status/2077770173793370556>
- `@ruyasuihe258` 提到的 6 月相关旧帖：<https://x.com/ruyasuihe258/status/2092026093763240423>
- 已归档的引用推文分析：[`../../Uniswap_v4_Hook动态费陷阱：报价模拟、路由盲区与LP收益掠夺.md`](../../Uniswap_v4_Hook动态费陷阱：报价模拟、路由盲区与LP收益掠夺.md)

## 主帖结构化摘要

作者从聚合器工作视角解释同一 `WBNB/USDC` v4 Hook 池的更深层问题。

主帖核心信息：

- 这类行为更接近利用系统盲区进行的恶意攻击；
- 用户表面上看到交易成功，但最终收到的资产数量变少；
- 聚合器会用 `eth_call` 模拟不同路径；
- 为避免模拟因 gas 不足失败，报价系统常给 `eth_call` 很高的 gas ceiling；
- 恶意 Hook 可以读取执行时的 gas 环境；
- 高 gas 模拟落入低费分支，真实交易则可能落入高费分支；
- 同样的 `beforeSwap` calldata，只改变 call gas，就能得到不同 fee override；
- 作者调查时的配置显示：`gas = 1,000,000` 对应约 `10%`，`gas ≈ 16,000,000` 对应约 `0.5%`，`gas = 30,000,000` 对应 `0%`；
- 历史代表交易中的 `12.8%` 来自 `PoolManager` 的 `Swap` event；
- `PoolKey` 里的 `0x800000` 只是 dynamic-fee flag，不代表 `800%` 手续费；
- 真实费率应看每笔 `Swap` event 的 `fee` 字段；
- `fee = 128000` 按百万分比单位换算为 `12.8%`；
- `amountOutMin` 只检查最终输出，不检查中间池 fee 是否和 quote 一致；
- 如果用户滑点或上层系统 buffer 足够宽，最终输出仍可能高于 `amountOutMin`，交易就会成功。

## 代表交易摘要

代表交易链接：

<https://bscscan.com/tx/0x7615dec2ce80506ec461a47739eb8533ac2bc7c605ca56c255964481b76363d0>

作者对这笔交易的路径拆解：

- 顶层是 Relay 流程；
- source-side swap 由 `0x BnbSettler` 执行；
- 调用层级包括 `ERC-4337 EntryPoint`、Relay v3 Router / ApprovalProxy / Depository、`0x BnbSettler`；
- `FOMO action` 完成 `USDT -> WBNB`；
- 随后的 Uniswap v4 action 完成 `WBNB -> USDC`；
- 问题出在后一步进入目标动态费率池和恶意 Hook。

作者给出的关键数值：

- 问题步骤前价值约 `5,926.9 USDT`；
- FOMO 输出约 `8.536 WBNB`；
- 进入目标 v4 pool 后得到约 `5,169.5 USDC`；
- 稳定币价值差约 `757.4` 美元；
- 差额比例约 `12.779%`，与 `12.8%` LP fee 基本一致；
- 该交易的 `minAmountOut` 约 `4,742.5 USDC`；
- 实际输出仍比 minimum 高约 `9%`，因此交易没有回滚。

作者说明，无法仅凭链上数据确认约 `20%` 的 buffer 来自用户手动设置，还是 Relay、0x 或上层集成为 meme、tax、低流动性、多跳路线自动加入。

## 反编译与链上验证信息

作者提到，该 Hook 没有 verified source，也没有 Sourcify metadata。

调查中取得约 `7,191-byte` runtime bytecode，并结合反编译与 live `eth_call` 验证。

可以归档的高层结论：

- `beforeSwap` 只接受 `PoolManager` 调用；
- 合约会读取池子配置和执行环境；
- fee tier 选择与 gas 环境有关；
- 相同输入在不同 gas envelope 下可能返回不同 fee override；
- 历史 `Swap` event 记录了 `12.8%` fee。

归档时不保存完整伪代码，原因是反编译结果不是 verified source，且这份材料的研究价值在防御和识别，不在复现攻击。

## 有信息量的回复

### 旧案例与 EnsoBuild

`@hin_myy` 提到，`@EnsoBuild` 此前也讨论过类似 toxic pool，指出一些 DeFi 流动性会在模拟阶段表现良好、执行阶段改变行为。

作者回复认为“受害者现身了”，并判断这个漏洞已经被利用很久。

### slippage 不能完全解决

有读者问，如果聚合器支持 JIT 或者用户把 slippage 设置到 `10%`，是否可以杜绝问题。

作者回复：不能完全杜绝。很低的 `minOut` 仍可能让部分量走到毒池，只是损失小一点。

### 攻防不对称

`@strconvAtoi` 提到，从聚合器角度看 toxic pool 很难应对，因为攻击者不只可以看 gas，还可能看 `tx.origin`、余额、`gasLimit` 等上下文来动态调整 fee。

### 核心攻击面

`@wwardenn` 总结：`quote execution mismatch is the real attack surface`。

这句话概括了主帖的防御重点：真正的问题不是单一费率，而是报价与执行之间的规则错配。

### 其他补充

- `@buzhidaoqishala` 提到，自己发交易前通常会在 Geth 上先模拟；
- `@luke_bit` 判断，这类高集中度币对的 Hook 可能带有白名单机制；
- `@0xCrabRust` 调侃“学到了吗家人们”，作者回复强调不是教人作恶。

## 归档说明

`twitter status --yaml` 在当前环境返回本地 Keychain cookie 权限错误，但直接 `twitter tweet` 成功读取主帖、引用帖和主要回复上下文。主帖里的 `t.co` 链接已用 `curl` 解析为上方外部链接。本归档保存结构化摘要和元数据，不保存反编译伪代码全文。
