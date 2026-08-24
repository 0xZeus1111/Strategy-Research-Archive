# 原文归档：BlockBloomer Uniswap v4 Hook 动态费率与路由盲区

## 主帖信息

- 作者：`@BlockBloomer`（青蛙）
- 原文链接：<https://x.com/BlockBloomer/status/2091547470950244771>
- 发布时间：`2026-08-23 23:25`（Asia/Shanghai）
- 抓取工具：`twitter-cli`
- 抓取时互动数据：`211` likes，`16` retweets，`29` replies，`3` quotes，`37300` views，`221` bookmarks
- 是否有配图：有
- 主帖配图：[`assets/HQanJUiagAA9wWJ.png`](assets/HQanJUiagAA9wWJ.png)
- 引用推文配图：[`assets/HQX7v06bUAAq7vv.jpg`](assets/HQX7v06bUAAq7vv.jpg)
- 引用推文：`@w3_888` 关于“10万本金，一天狂赚几万U”的截图分享

## 外部链接

- Uniswap v4 动态费率说明：<https://developers.uniswap.org/docs/protocols/v4/concepts/dynamic-fees>
- 代表交易：<https://bscscan.com/tx/0x7615dec2ce80506ec461a47739eb8533ac2bc7c605ca56c255964481b76363d0>
- 实时池数据 API：<https://api.geckoterminal.com/api/v2/networks/bsc/pools/0x36e5540e9dedc02229fe8a82aa5b10c0bf07d1fa74e4f2ffe0efd00fa1a36aea>

## 主帖结构化摘要

作者解读 BNB Chain 上一个 `WBNB/USDC` Uniswap v4 Hook 池的异常收益来源。

主帖核心信息：

- 该池在聚合器模拟报价时可以显示 `0%` 手续费；
- 真实成交时却可能出现高额 LP fee；
- 作者复核某笔真实交易的 `PoolManager` `Swap` 事件，`fee` 字段为 `128000`；
- 在 v4 口径中 `1000000` 代表 `100%`，因此该笔交易实际 LP fee 为 `12.8%`；
- 该机制不是预测 `BNB` 涨跌，而是利用聚合器选路逻辑，把订单引入高费池；
- 路由器只是送单员，真正承担高费的是最终交易者或执行策略；
- 如果滑点保护严格，交易会回滚并浪费 Gas；
- 如果滑点足够宽或套利空间覆盖费用，交易会成功，手续费进入 LP；
- 截至 `2026-08-23 22:51`，该池滚动 `24` 小时仍有约 `1082万美元` 成交量、`16671` 笔交易，但最近一小时成交已归零；
- 作者不建议复制，认为这是流氓与掠夺行为；
- 案例真正值得科普的是 Uniswap v4 把 AMM 从固定公式变成了可编程市场。

## 配图内容

### 主帖配图

主帖配图是聊天截图，内容包括：

- “下午一直换池子”；
- “我吃了一万”；
- “老哥你都赚了1万刀？”；
- “本金5w”。

这张图用于说明被引用事件里有人宣称通过不断换池获得高收益。

### 引用推文配图

引用推文配图是 `WBNB/USDC` v4 池界面截图。

可见信息包括：

- BNB Chain；
- v4 池；
- `WBNB/USDC`；
- 过去 `1` 天交易量约 `US$1173.7万`；
- 总 APR 约 `507,342.22%`；
- 总锁仓量约 `US$10.5万`；
- `24` 小时交易量约 `US$1167.8万`。

## 有信息量的回复

### Hook白名单

有读者提到，这个 Hook 似乎设置了白名单，一般人无法向该池添加流动性。

作者回复确认，并提醒即使可以添加也不要添加，危险。

### 盲盒费率

作者补充，这个 Hook 设计像盲盒，过路费按盲盒收，有倒霉交易随机付了 `12%`。

### 聚合器识别风险

有读者反馈，自己曾在钱包聚合器里遇到 v4 池识别问题：走其他路径正常，但某聚合器路径会导致到手金额大幅减少。

作者回复“这个真吓人”。

### 用户是否能看Hook机制

有读者问，不懂技术的用户能否看到 v4 池 Hook 机制。

作者回复“能”。

### 类似掠夺操作

有评论提到，原始操作者此前也做过一些掠夺性质操作，例如周末操控价格打出其他人 LP 区间、自己独占奖励等。

作者回复称自己也给 Hook 池抢过流量。

## 归档说明

`twitter status --yaml` 在当前环境仍返回本地 Keychain cookie 权限错误，但直接 `twitter tweet` 成功读取主帖和回复上下文。主帖里的 `t.co` 链接已用 `curl` 解析为上方外部链接。未保存完整 `tweet.json`，本归档保存结构化摘要、元数据和图片资产。
