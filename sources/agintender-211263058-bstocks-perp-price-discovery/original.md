# bStocks 的 B 面：不做更好的 Nasdaq，用 Perp 绕出一条定价权之路

- Author: Danny (`@agintender`)
- Published: 2026-08-15 12:17:40 +08:00
- URL: https://agintender.substack.com/p/bstocks-b-nasdaq-perp
- Subtitle: 与其在正股市场正面硬刚，不如把战场搬到传统市场关门以后
- Source Type: Substack 付费长文（由用户提供的本地 MHTML 归档）
- Capture Note: 正文与 25 张有效配图已归档；其中 19–23、25 号图因 MHTML 未内嵌原图，按正文中的原始 Substack CDN 地址补充下载。页面导航、头像、交互控件和评论区未纳入正文。

![图01](assets/%E5%9B%BE01.webp)

1848 年，Chicago Board of Trade （CBOT）成立的时候，它还不是后来那个巨大的期货市场。芝加哥正在变成美国中西部谷物的集散地，铁路、运河把一船一船、一车一车的小麦和玉米送进城市。问题是，小麦不是股票。同样叫 wheat，不同农场、不同年份、不同质量，根本不是同一种东西。买家如果连自己买到什么都不知道，就很难形成一个统一的市场价格。CBOT 最早做的事情之一，就是把这种混乱整理成可以交易的标准。1859 年获得伊利诺伊州授权后，CBOT 可以制定谷物等级，并由指定检验员判定质量；到了 1865 年，保证金和交割规则也开始制度化。

这件事听起来不像金融创新，更像仓库管理，但这个仓库改变了后来的市场结构。

谷物进入大型 elevator 以后，不再需要追问这一袋麦子来自哪一个农场。市场开始按照统一等级交易，例如某一种标准等级的小麦。美国联储在回顾这段历史时提到，这种 grading 和标准化仓储让买家知道自己买的是什么，也降低了交易成本，给流动性市场创造了条件。

然后才轮到期货。

农民可以提前卖未来的收成，粮商可以买未来的货，投机者不用把几千蒲式耳小麦搬回家，也能围绕未来价格下注。CBOT 从早期的 forward、“to-arrive”交易逐渐发展出标准化期货，到 19 世纪后半叶，芝加哥的谷物期货开始承担价格发现、风险管理和公开报价的功能。

这里有一个容易被忽略的地方。

**最后跑得最快的是期货，但让期货不至于变成一场脱离实物的赌博的，是后面的粮仓。**

2026 年 8 月的一个周末，Binance 上 NVDA、TSLA、SNDK、SKHY 四只股票永续合计成交约 **4.61 亿美元**。星期天，四只 Perp 的价格都高于前一个现金市场收盘，星期一美国现金市场开门以后，四只却全部低开。单看 SNDK，那个周末 Perp 成交约 **3.38 亿美元**，Friday Cash Close 是 1,212.21 美元，Sunday 交易到 1,223.98 美元，Monday Cash Open 却落在 1,203.41 美元。

一个月前的美国独立日长周末，同一只 SNDK 给出了另一种答案。现金市场从 1,745 美元休市，但在周日 Binance 交易到 1,841.88 美元，周一 Cash Open 是 1,828.68 美元。现金市场重新开门后出现约 4.8% 的跳空，Binance 周末价格提前覆盖了这次上涨的大部分幅度，只是价格走过了头。

同一个产品，一次把周一的大跳空提前交易出来，一次在几亿美元成交以后把方向看反。这件事把 24/7 股票的问题从“交易时间延长”推到了市场结构：**成交量不是定价权。**

如果 Binance 的目标只是让用户星期六也可以买股票，延长交易时间就够了。但把 Direct Stock、TradFi Perp 和 bStock 放在一起看，可以看到一套不同的结构。现金市场闭市以后，Perp 先承接交易方向、杠杆和高频交易，bStock 提供可以持有、搬运、抵押和对冲的股票库存，链上协议把中心化交易所之外的资本接进来。等传统股票市场重新开门，再看这套 24/7 市场形成的价格有没有被美股市场接受。

这篇文章讨论的不是 bStock 是不是另一种 tokenized stock，而是另一件事：**如果 Binance 要争传统股票闭市后的第一版价格，为什么发动机要用 Perp，而 bStock 又为什么会成为库存层和纠错层。**

## **一、Binance 要争的不是多两个交易日，而是股市关门以后的价格**

股票市场有一个时间上的缺口。纽约星期五下午 4 点收盘以后，公司还会发布消息，宏观政策会变化，产业链会发生新的事情，战争和政治也不会等 Nasdaq 上班。资金仍然在重新估值这些股票，只是这些判断暂时进不了美国现金股票的正式订单簿。

所以问题不是没有信息，而是信息出来以后应该去哪一个市场（用仓位）表达。如果某个市场可以在 NYSE、Nasdaq、KRX 、港交所休市的时候承接股票风险，它就有机会形成下一次正股市场开门之前的价格发现。

Binance 的三种产品刚好分成三个角色。Direct Stock 负责真实证券、公司行动和传统市场接口；bStock 把股票变成一件可以持有、转换、转链和抵押的资产；TradFi Perp 负责多空、杠杆和连续交易。

![图02](assets/%E5%9B%BE02.webp)

如果要成为价格发现的源头，最适合冲在前面的不是 bStock，而是 Perp。原因也不复杂：价格发现的第一步是让观点进入市场，而衍生品表达观点的成本是低于现货的。

## **二、为什么Perp更容易做到价格发现？**

以 NVDA 为例，Binance 内部两种产品的成交结构差距不小。7 月 17 日至 22 日的一段样本里，NVDAB Spot 名义成交约 **421 万美元**，同期 NVDAUSDT Perp 约 **4.18 亿美元**，相差约 **99 倍**。只看周末，7 月 18 日 Perp 约为 bStock 的 37 倍，7 月 19 日约为 68 倍。

![图03](assets/%E5%9B%BE03.webp)

这里比较的是 Binance 内部两种产品，不是 Binance Perp 和 Nasdaq NVDA 正股。美国现金股票的成交规模仍高于 Binance 股票 Perp。这个比较的意义，是看现金市场休息以后，新的方向订单更容易流向哪里。

现货买完以后可以放几个月，Perp 会不断开仓、平仓、反手、调整杠杆、做 basis、收 funding，做市商也会不断重新 hedge。同样一块钱的资本，在 Perp 里可以贡献多次 gross turnover。做空的区别更大：星期六出现 NVDA 利空，没有 NVDAB 库存的人想在现货做空，需要先处理 borrow；Perp 只需要 Sell 就能表达方向。

因此，休市后的第一条价格路径可以写成 **Information → Perp → Candidate Price**。Perp 的作用是生产候选价格，而不是保证候选价格正确。这个区别决定了 bStock 后面为什么会出现。

## **三、研究口径：25 次“闭市—重开”的数据比较**

为了观察这口价格是怎么形成的，我们收集并整理了 NVDAUSDT、TSLAUSDT、SNDKUSDT、SPCXUSDT、SKHYUSDT 和 SKHYNIXUSDT 的交易数据。美国市场能够完整对应下一次 Cash Open 的样本有 25 次，其中 NVDA、TSLA、SNDK、SPCX 各 6 次，SKHY 1 次；SKHYNIX 对应韩国 KRX，因此另外处理。

一次样本从前一个 Cash Close 开始，经过 Pure Weekend、Sunday Price、Monday Premarket、Opening Auction，再到下一次 Cash Open。Sunday 19:59 ET 用来观察常规美股 Premarket 尚未开始时的价格，Monday 04:00 以后 Premarket 回来，09:25 之后 Opening Auction 信息进入，最后拿正式现金开盘作为验证点。

这批数据分别回答四种问题：

> **1.** 价格部分看周末报价离下一次现金开盘有多远；
>
> **2.** 流动性部分看屏幕价格可以承受多大的实际订单；
>
> **3.** 成交量部分看市场转了多少钱；
>
> **4.** 交易频次部分则看这些成交更像投资者的独立观点，还是程序化交易、做市和 HFT 形成的高周转市场。

一个市场有价格，不代表价格能执行；有成交，不代表成交来自独立判断；有流动性，也不代表它拥有价格发现能力。

## **四、价格调查：从 181.9bp 收敛至 11.3bp**

这里的“误差”是某一时点 Binance Perp 价格与下一次 Cash Open 的绝对距离，统一换算成 basis points （bp）。

![图04](assets/%E5%9B%BE04.webp)![图05](assets/%E5%9B%BE05.webp)

周日的结果是 13/25，也就是 52%。25 个样本里 13 次方向一致、12 次不一致，差不多 50% ，所以这批数据不能证明 Binance 存在稳定的 Weekend Discovery。

09:29:59 的 11.3bp 也不能被简单地认为是 Binance“预测周一开盘”的能力。到了这个时间，美国 Premarket 已经交易几个小时，Opening Auction 的信息也进入市场。Perp 如果仍然大幅偏离股票盘前报价，本身就会形成交易机会。

因此，从 181.9bp 收到 11.3bp 能说明 Binance Perp 在传统市场重新上线的过程中逐步接近下一次 Cash Open，却不能说明是 Binance 带着现金市场走。要回答这个问题，还需要把 Binance Perp、bStock 和同一秒的美国 Premarket last、bid、ask 放到一起做 lead-lag。

如果 Premarket 先从 100 走到 105，Perp 后来才到 105，这是 follow；如果 Perp 先到 105，Premarket 后面向它靠，才有价格发现和价格领导的意义。

![图06](assets/%E5%9B%BE06.webp)

**图1｜从周末到周一开盘：误差收敛与方向一致率**

> **图表说明：** 这张图要读的不是“越靠近开盘越会预测”，而是两条曲线一起变化：平均绝对误差从 181.9bp 收到 11.3bp，方向一致率从 52% 升到 96%。

## **五、开盘前的价差不是一条直线：SNDK 在 09:25 只差 2bp，五分钟后反而偏离了**

2026 年独立日长周末，NVDA、TSLA、SNDK 三只 Perp 从 Sunday 到 Opening Cross 前的路径就不同。

![图07](assets/%E5%9B%BE07.webp)

这么看起来，这时候 NVDA 更像一条纠错路径。周日在 198.35 美元，而 Cash Open 最后是 194.42 美元，差约 202bp；09:25 回到 194.97，09:29:59 是 194.55，误差缩到 6.7bp。

TSLA 的方向从 Sunday 开始就是对的。Cash Close 是 393.45，Sunday 是 399.40，Cash Open 是 397.50。市场提前交易出上涨方向，只是 Sunday 的幅度超过了最终 Opening Gap。

SNDK 的路径更说明问题。09:25 时 Perp 在 1,828.97 美元，Cash Open 最后是 1,828.68 美元，差不到 2bp；到了 09:29:59，Perp 又掉到 1,825 美元，误差扩大到约 20bp。

这说明 Opening 前价格不会沿一条直线朝最终开盘价移动。Premarket 成交、NOII、做市商库存和最后几分钟的订单都会改变价格。研究谁拥有定价权，不能只看周日和周一09:29:59 两个时间段，而要看整段 lead-lag path。

## **六、bStock 自己的价格有没有信息：Perp 不一定比 bStock 更接近下一次 Open**

前面的 25 个样本研究的是 Perp，现在我们来看 bStock 的价格变化。

另一组低频样本使用 NVDAB、TSLAB、MUB、COINB 的 UTC 日末价格，与下一次美股 Cash Open 做 handoff 比较，并对有数据的标的加入 Perp。这个口径使用 UTC close，不是严格的 09:29 ET，因此只能作为 handoff proxy，不能和前面的逐秒样本混在同一个统计池里。

![图08](assets/%E5%9B%BE08.webp)

八个 bStock 观察值的 absolute handoff error 中位数约 **117.2bp**，六个可比 Perp 观察值约 **118.9bp**。

7 月 23→24 日的 NVDA 是一个好例子。NVDAB 是 207.97，Perp 是 207.99，下一次 Cash Open 是 207.45，两边都高约 25–26bp。这里 bStock 的价值不是给出一个比 Perp 更接近正股市场的价格，而是把休市期间形成的股票价格保存成一份可以持有、转移、转换和融资的现货型库存。

Perp 负责让价格移动，bStock 负责把价格变成资产。

## **七、股票拆开以后：NVDA 是纠错，TSLA 是 Overshoot，SNDK 是压力测试，SPCX 是产品实验**

在 NVDA 的数据样本里，Sunday 与下一次 Cash Open 的平均绝对偏差约 113bp，方向 3/6；TSLA 约 81bp，方向 4/6；SPCX 约 149bp，方向 3/6；SNDK 达到约 351bp，方向 3/6。到了 09:29:59，四者的描述性平均偏差分别降到约 11.8bp、8.6bp、9.4bp 和 16.0bp。

![图09](assets/%E5%9B%BE09.webp)

NVDA 更适合看错误价格怎么修正。值得注意的是在独立日样本里，Sunday 198.35，Cash Open 194.42，偏差 202bp，之后价格一路向 194 美元附近回归。

![图10](assets/%E5%9B%BE10.webp)

TSLA 适合看 Overshoot。Cash Close 393.45，Sunday 399.40，Cash Open 397.50，方向对了，幅度超了。

SNDK 把两种状态放在同一只股票里。独立日长周末，它提前交易出大部分 +4.8% 的 Opening Gap；8 月另一个周末，Friday Close 是 1,212.21，Sunday 是 1,223.98，Monday Cash Open 却是 1,203.41，方向反了。

SPCX 的位置不一样。它的 Sunday MAE 约 148.8bp，方向也是 3/6，但这只股票从 Pre-IPO Perp 一路走到公开股票、TradFi Perp 和 SPCXB，同一个风险在 Binance 里经历了“先有衍生品价格、后有现金股票”的产品迁移。它更适合拿来观察一套 24/7 价格体系怎样和后来出现的现金市场接轨。

所以闭市价格不适合只分“对”和“错”，而是可以进一步分为：**Direction Discovery、Magnitude Discovery、Correction Dependency**。如果最终 Opening Gap 足够大，可以讨论周日提前交易了最终 Gap 的多少；如果 Opening Gap 只有几十 bp，就不应该使用比例指标，因为分母太小会放大结果，这种样本直接看绝对 bp 偏离更合适。

![图11](assets/%E5%9B%BE11.webp)

**图2｜周末定价的三种状态：方向发现、过冲、纠错**

> **图表说明：** 同一个“周末价格接近周一开盘”的结果，背后可能是三种不同路径。TSLA、SNDK 可以先判断对方向、再出现 Overshoot；NVDA 则可以先报错，再在 Premarket 和 Opening Auction 回来以后纠错。

## **八、SPCX：从 Pre-IPO Perp 到公开股票，从 Perp 到股票的定价实验**

SPCX 和 NVDA、TSLA、SNDK 有一个结构上的差别。Binance Futures 在 2026 年 5 月推出 Pre-IPO Perpetual 时，第一只合约就是 SPCXUSDT，用来交易 SpaceX 未来公开市场估值。SpaceX 上市以后，SPCXUSDT 转为标准 TradFi Perp；同时也增加了 SPCX Direct Stock 和 SPCXB。也就是说，这只股票先有衍生品价格，后有可以验证它的公开现金市场，再补上可以持有和上链的 bStock。

这给了一个少见的实验环境。对 NVDA 和 TSLA 来说，Binance 是在一个成熟市场外额外增加的 24/7 风险层；SPCX 则经历了“Pre-IPO Perp → Public Stock → TradFi Perp → bStock”的迁移。等现金股票出现以后，原来那个不停交易的衍生品价格才第一次有了固定的 Monday Cash Open 可以反复校验。

六个完整周末的 SPCXUSDT 数据如下：

![图12](assets/%E5%9B%BE12.webp)

六次 Sunday Price 的方向是 3/6，距离下一次 Cash Open 的平均绝对偏差约 148.8bp，中位数约 134.4bp。到了 Monday 04:00，平均偏差降到 114.1bp；08:00 是 102.0bp；09:00 是 53.8bp；09:25 是 29.3bp；09:29:59 则降到 9.4bp。08:00 以后六次方向都与最终 Opening Gap 一致，但这时候 Premarket 已经回来，所以这条曲线证明的是价格接力和收敛，不是 Binance 单独完成了价格发现。

![图13](assets/%E5%9B%BE13.webp)

SPCX 还把 Overshoot 这件事表现得很集中。在 Sunday 方向看对、同时 Opening Gap 超过 50bp 的三个周末里，7 月 20 日 Sunday Move 约为最终 Gap 的 2.79 倍，7 月 27 日约 1.60 倍，8 月 10 日约 1.34 倍。它不是只会“猜对或猜错”，而是经常先把方向交易出来，但幅度一般都超过了。

8 月 10 日的路径更说明问题。Friday Close 是 133.11 美元，Sunday 已经到 135.57，Monday Cash Open 是 134.95。这个 Sunday Price 只比最终 Open 高约 46bp，但 Monday 08:00 SPCXUSDT 一度冲到 138.80，离最终 Cash Open 扩大到约 285bp；09:29:59 又回到 134.86，只差约 6.7bp。价格可以靠近，也可以在 Premarket 回来以后重新走开，再收回来。

此外，SPCX 的成交结构也不是一个安静的现货市场。六个周末 SPCXUSDT 合计成交约 15.46 亿美元，约为 NVDA 同口径的 10 倍；底层 fills 约 358 万笔，平均 3.45 笔/秒。它的规模处在 SNDK 和 NVDA 之间，更像一台全天运行的程序化风险机器。对 bStock 来说，这个案例的意义很直接：Perp 转得越快，只要存在客户净订单流，做市商就越需要 SPCXB 或 SPCX 这样的库存腿去承接 delta。

所以 SPCX 不是“Binance 已经拿到周末定价权”的证明。它更像整套产品栈的缩影：Perp 先生产候选价格，SPCXB 把价格保存成资产，Stock 提供现金市场出口，borrow 和跨市场套利决定错误价格能不能被双向攻击。它让我们看见 Binance 如果要争闭市定价权，这套基础设施可能长什么样。

## **九、SKHY：一个周末 8,500 万美元成交，也能偏离下一次开盘近 4%**

SKHY 周五 Cash Close 是 **137.91 美元**，Sunday Binance 是 **140.24 美元**，Monday Cash Open 是 **135.03 美元**。Sunday 与下一次开盘相差约 **386bp**。到了 09:25，误差缩到约 15.6bp；09:29:59，SKHYUSDT 在 135.15 左右，只比 Cash Open 高约 **8.9bp**。那个纯周末窗口的 Perp 成交约 **8,530 万美元**。

这个案例不说明 SKHY 通常会偏 4%，它说明另一件事：一个休市市场可以有数千万美元成交，Sunday Price 仍然可能偏离下一次现金开盘数百 bp。价格在周一重新靠近，也不能直接证明 Binance 在领先，因为 Premarket 同时在恢复。

有流动性，不等于有价格发现。

## **十、SKHYNIX：如果这套结构从 Nasdaq 延伸到 KRX，故事就不只是美股**

SKHYUSDT 和 SKHYNIX 的周末成交规模也值得观察分析。

六个 Saturday+Sunday UTC 窗口的 gross notional 合计约 **24.06 亿美元**。

它提出的问题却比美股更大。美国开着的时候韩国可能已经关门，韩国开门的时候美国还没醒，日本和欧洲又有各自的交易时间，周末大多数传统市场一起关闭，而 Crypto 那层没有这个边界。

如果这套产品往日本股票、欧洲股票、ETF、商品扩展，Binance 所争的位置就不只是“24/7 美股”，而可能是一层 **24/7 Global Risk Layer**：原生市场沉默时，风险继续在另一个市场交易。

## **十一、周末的交易量调查**

从六个周末的数据统计来看，NVDAUSDT 累计成交约 1.54 亿美元，TSLAUSDT 约 1.07 亿美元，SPCXUSDT 约 15.46 亿美元，SNDKUSDT 达到 29.22 亿美元。

![图14](assets/%E5%9B%BE14.webp)

从上表来看，成交量出现了清楚的梯队：SNDK 六个周末 29.22 亿美元，SPCX 15.46 亿美元，NVDA 1.54 亿美元，TSLA 1.07 亿美元。SPCX 约为 NVDA 的 10 倍、TSLA 的 14 倍，但只有 SNDK 的一半左右。这个排序和公司市值、传统股票知名度并不是线性关系，所以 gross volume 不能直接翻译成自然投资需求。

还有一个数据值得注意：开盘前最后五分钟，SNDK 六次平均仍有约 2,664 万美元成交，SPCX 约 1,888 万美元，NVDA 约 249 万美元，TSLA 约 201 万美元。Opening Cross 临近时，Perp 不是停在那里等结果，而是在继续重新交易价格。

Volume 这个数据的核心是：这些成交是谁做的、以什么频率发生、每单多大。

![图15](assets/%E5%9B%BE15.webp)

**图3｜六个周末累计成交额：SNDK 与 SPCX 形成单独梯队**

> **图表说明：** SNDK 六个周末累计 29.22 亿美元，SPCX 15.46 亿美元，远高于 NVDA 和 TSLA。这个排序与传统股票市值和知名度并不线性，因此 gross volume 更适合用来描述市场转速，不能直接解释成同等规模的自然投资需求。

## **十二、交易频率调查：SNDK 每秒 7.3 笔，SPCX 每秒 3.45 笔**

把 SNDK 的 29.22 亿美元和 SPCX 的 15.46 亿美元拆到逐笔成交以后，市场形态发生了变化。

![图16](assets/%E5%9B%BE16.webp)

SNDK 六个周末一共有约 757 万个 fills，平均每秒约 7.3 笔；SPCX 约 358 万个 fills，平均每秒约 3.45 笔；NVDA 约 64 万个 fills，平均每秒约 0.62 笔。三只股票的差距首先体现在交易频率。

单笔金额的差距没有成交频率那么大。SNDK 平均 raw fill 约 386 美元，SPCX 约 432 美元，NVDA 约 241 美元。SPCX 的中位金额约 171 美元，也和 SNDK 的 173 美元接近。SNDK 和 SPCX 的大额 turnover **不是**靠少数超级大单堆出来的，而是靠更密集的成交。

由于我们无从得知交易账户的信息、我们也不知道这些 fills 来自多少独立参与者，也不知道有多少来自做市商、HFT、程序化 basis 或重复回转等等，但从交易的体量和频率观察，本文倾向于，“这几十亿美元的成交并不代表同等规模独立投资观点”的论点。（aka 不是机构在下注）

我们认为，SNDK 和 SPCX 都形成了高转速股票衍生品市场，只是 SNDK 的转速更高。

延伸阅读：[为什么偏偏是 SNDK 的合约交易量最大——永续合约怎么变成了它的第二主场](https://substack.com/@agintender/note/c-311284509)

SPCX 交易量的 24 小时分布还能再进一步分析：

美东时间 00:00–06:00 仍贡献约 20.8% 的周末 gross notional 和约 20.9% 的 raw fills，对应这段深夜时段平均约 2.9 fills/秒。六个周末的 taker buy/sell notional imbalance 约为 -1.16%，整体接近双向平衡。这个形态更像做市、程序化交易和跨市场套利共同形成的全天候市场。

![图17](assets/%E5%9B%BE17.webp)

**图4｜纯周末平均底层成交频率**

> **图表说明：** SNDK 的 7.30 fills/秒、SPCX 的 3.45 fills/秒和 NVDA 的 0.62 fills/秒，把总成交额的差异拆开了。SNDK、SPCX 的单笔金额没有大到足以解释几十倍 turnover，差距来自更高的成交频率。这也是为什么文章把它们理解成高转速的风险市场，而不是把 gross notional 当成独立资金流入。

## **十三、每单大小也提示，Perp 和 bStock 可能承接的是两类订单**

SNDK Perp 的平均 raw fill 约 386 美元，SPCX 约 432 美元，NVDA 约 241 美元；SPCX 和 SNDK 的 aggTrade 中位金额分别约 171 美元和 173 美元。Binance Research 对 bStock 的报告称约 93% 的 bStock 交易属于 fractional trade，中位成交额只有 18.81 美元，约 80% 的 tokenized-stock 交易来自新兴市场用户。（[Binance Research 报告](https://www.binance.com/en/research/analysis/opportunity-only-tokenized-stocks-unlock)）

需要注意的是，本文 Binance Research的数据口定义不同，所以不能直接用 386 除以 18.81，然后推出 Perp 用户的平均仓位是 bStock 用户的 20 倍。不过如果把它们放在一起，可以看出一种产品分工：bStock 更容易承接小额现货、长期库存、跨时区 retail 和链上用户，Perp 更容易集中杠杆、basis、做市、HFT 和高周转资金。

一个市场负责形成可持有的股票库存，另一个市场负责让同一份风险高速换手。只有做市商和套利者把两边连接起来，才可能形成一套价格网络，而目前这两个不同的产品还没有联动起来，这是一个可以发力的点。

## **十四、Perp 成交越大，bStock 的库存作用越突出**

既然 Perp 成交量可以是 bStock 的几十倍，为什么 Binance 还需要 bStock？答案在做市商的账本里。

客户和 Market Maker 的 Perp 仓位是镜像关系。客户净 Long，MM 就是 Short Perp，需要买入正 delta 的 bStock 或 Stock；客户净 Short，MM 就是 Long Perp，需要卖掉 bStock 库存，或者借 bStock 以后卖。

![图18](assets/%E5%9B%BE18.webp)

Perp 解决的是交易速度，bStock 解决的是风险放在哪一层资产负债表里。如果客户净订单持续向一个方向流，MM 的 delta 就会积累，不能永远留在 Perp 内部转手，总要有一条库存腿承担风险。

所以 Perp 的高成交量不是 bStock 失败的证据，反而是 bStock 存在的理由的之一。衍生品转速越高，对库存、融资和借券的需求就越大，对散户可能没什么感觉，但是这对于做市商来说则是一个必需品。

## **十五、bStock 不是“支点”，1:1 backing 更像 terminal constraint**

说到这里，大家可能会把 bStock 叫作 Perp 的现货锚定点，容易把它想成 USDT 对美元的那一套。但是在不同场景下作用是不同的，至少在股票周末不是这样的。

星期六把 NVDAB 转成 NVDA Stock，不代表 Nasdaq 同一秒有现金市场让另一条套利腿完成。你拿到的是一份能够接回真实股票的库存，不是一个可以马上完成的 Cash Arbitrage。

所以 1:1 backing 更像 **terminal constraint**。市场知道 bStock 最后能够接回真实股票，也知道 Cash Market 会再次打开，因此价格可以在休市时偏离，但偏差超过资金成本、库存成本、event risk 和执行摩擦以后，会产生等待传统市场重新开放再收掉价差的激励。

因此 bStock 的工作不是告诉 Perp“正确价格是什么”，而是给 Perp 的候选价格配上一份可以持有、对冲和套利的库存。

这就是为什么把 bstock 看成是“库存层”和“纠错层”更准确。

## **十六、bStock 上链以后，重点是让库存转起来**

如果 NVDAB 只能留在 Binance Spot 账户里，它只是 CEX 内部的一层现货库存。bStock 上链以后，它可以进入 DEX、Lending、Collateral 和 Margin 系统，市场结构就改变了。

PancakeSwap 上线了包括 TSLAB、NVDAB、MUB、SNDKB 在内的 bStock 池；Lista 支持部分 bStocks 抵押借 Stablecoin；Aster 允许符合条件的 bStocks 进入 Multi-Assets Mode；[Youcanshortit.com](http://youcanshortit.com/) 让用户可以杠杆借出bStock；Pundi X Basket 让用户自由定制自己的投资组合和指数ETF，让其他散户可以跟单。

它们解决的不是同一件事，但方向是一致的：让 NVDAB、SNDKB、SPCXB 不只躺在钱包里，而是可以被交易、抵押、融资、做市和对冲。

这里要区分“增加用途”和“增加流动性”。一枚 NVDAB 被锁进 Lending Protocol 做 collateral，utility 增加了，但它未必给市场增加新的买卖盘。只有当这份库存能被做市商、套利者或空头借出来，再进入 CEX、DEX 或 Perp 对冲，它才开始增加市场里的可用库存。

所以理想的链上循环不是“Binance → Wallet → DEX”，而是

> **bStock → DEX / Lending / Credit Pool → Market Maker → Perp Hedge → CEX → Stock Conversion**。

同一份 bStock 被不同账户重复使用，库存才会产生周转。

对 bStock 来说，TVL 不是最关键的指标。更值得看的是一份股票库存能被使用多少次。100 万美元 NVDAB 如果只是锁在协议里，它仍然是 100 万美元静态资产；如果其中一部分能够被 MM 借出来报价，再用 Perp 对冲，成交后重新回到库存池，它才开始变成市场基础设施。

Binance 自身研究还报告过 2,806 名用户参与 bStock、Perp 和 Equity 之间的近似套利式交易，涉及约 2.16 亿美元，约 58.5% 的 bStock 用户同时使用 Perp 和/或 Equity。

上链以后，一条路径可以是 NVDAB 抵押以后借 Stablecoin，Stablecoin 再回 CEX 做 NVDAUSDT Perp；另一条路径可以是 NVDAB 进入 DEX LP，再用 NVDAUSDT Perp 对冲 LP 的股票 delta。

于是 bStock 把 CEX Perp、CEX Spot、真实 Stock、DEX、Lending 和 Stablecoin 放进同一个风险网络。

这也是 bStock 和普通“股票代币”之间的差别。

![图19](assets/%E5%9B%BE19.jpg)

**图5｜让 bStock 转起来的协议地图**

> **图表说明：** PancakeSwap、Lista、Venus、Aster、YouCanShortIt、Pundi X Basket 各自解决的不是同一件事。交易、抵押、做空、对冲和组合分发只有接成一条库存循环，bStock 才会从“被持有的资产”变成“能被不同资产负债表调用的库存”。

![图20](assets/%E5%9B%BE20.jpg)

**图6｜Pundi X Basket：从单只 bStock 到可分发的组合**

> **图表说明：** Basket 的意义不只是多一个产品外壳。用户可以按权重组合 NVDAB、SPCXB、SNDKB、SKHYB 等成分，其他用户直接买入组合；如果底层还能连接 Borrow、Perp Hedge 和链上调仓，组合本身会成为 bStock 流动性的一个新入口。

## **十七、这套系统缺的不是更多 ticker，而是 borrow**

拿 NVDAB 抵押后借 USDT，是融资。借 NVDAB 出来，再把 NVDAB 卖掉，才是 stock borrow。两件事都会出现“借”，但对价格发现的作用不同。

如果 Perp 贵、bStock 便宜，套利者可以 Long bStock + Short Perp。这条交易是相对来说容易执行，买便宜的现货、卖贵的衍生品，会压缩 Perp 溢价。

如果 bStock 贵、Perp 便宜，交易应当反过来做：Short bStock + Long Perp。问题是，没有可借库存就做不了这个交易。

于是纠错会出现不对称。折价时有现金就可以买，高估时却不是所有人都有货可以卖。

![图21](assets/%E5%9B%BE21.jpg)

所以 bStock 的成熟度不应该只看 listing 数量。Borrow Depth、Borrow Rate、可借库存，以及事件周末 borrow 是否稳定，会决定这套市场有没有双向纠错能力。

没有 borrow，bStock 只是一个资产层，没办法在平台上发挥出应有的优势。

borrow 补上以后，它才接近证券库存市场。

> 做市需求带动 borrow → borrow需求带动利率上升 → 利率上升吸引越来越多的deposit → bStocks的交易量和需求带起来 ，如此反复

## **十八、CEX 的 bStock 要做深，核心还是 Borrow**

回到 Binance CEX，bStock 的流动性瓶颈更清楚。客户大量卖 NVDAB 时，做市商可以买入 NVDAB，同时 Short NVDAUSDT 对冲，Bid Side 主要消耗现金；客户大量买 NVDAB 时，MM 必须不断把股票卖给客户，库存卖完以后，如果没有 Borrow，只能缩小 Ask Size、提高报价，或者退出市场。

所以 bStock 的现货盘口天然有一个库存问题。没有 Borrow，做市商手上有多少股票，就只能围绕多少股票报价；有了 Borrow 以后，库存从一个硬上限变成一个有价格的资源。MM 可以借 NVDAB 卖给客户，同时 Long Perp 或用 Stock 对冲。借券需求上升，Borrow Rate 上升，又可以吸引持有人把更多 bStock 放进库存池。

Borrow 还解决了前文的反向 Basis 问题。Perp 贵、bStock 便宜时，Long bStock + Short Perp 谁都能做；bStock 贵、Perp 便宜时，需要 Short bStock + Long Perp，如果此时没有可借库存，那么这条套利链就无法起到作用。

当然，Borrow 不是单独工作的。CEX 流动性要做深，还需要 Stock ↔ bStock Conversion 补充库存，Portfolio Margin 降低 bStock 与 Perp 对冲的资本占用，Market Maker Program 再把这些库存转成 Bid 和 Ask。Maker Rebate 可以让做市商愿意开机器，但 Borrow 决定机器手里有没有货。

所以总结起来：

> **Perp 负责产生订单，bStocks 负责提供股票库存，Borrow 负责让库存流动。**

如果 Borrow 做起来，bStocks 才会从“可以交易的股票代币”，往“可以被做市、融资和双向套利的证券库存”再走一步。

![图22](assets/%E5%9B%BE22.jpg)

**图7｜Borrow 如何把静态库存变成 CEX 流动性**

> **图表说明：** Borrow 的作用不是只增加“做空功能”，而是给 Ask Side 增加弹性库存。持有人把 bStock 放入库存池，做市商借出后在 CEX 报卖盘，再用 Perp 控制 delta。库存能借出来，卖盘才有机会变厚，Spread 和大额订单冲击才有下降空间。

## **十九、终局不是 bStock 对 Perp，而是四个市场能联动起来**

假设未来 borrow、conversion 和链上深度都成熟，同一个 NVIDIA 风险可能同时有四个价格。

![图23](assets/%E5%9B%BE23.png)

这时候 cross-venue trader 最先关心的不会是 NVIDIA 明年 EPS，而是哪个市场贵、哪个市场便宜。

DEX NVDAB 在 201，Perp 在 199.60，可以卖贵的一腿、买便宜的一腿；没有 bStock 库存就 borrow，没有现金就拿资产做 collateral，不想承担 NVDA 总体方向，就在另一条腿锁住 delta。现金市场重新开放以后，再根据 Stock、bStock 和 Perp 的 basis 调整库存。

这时候 bStock 不再是一种独立交易产品，而是一种让 **Stock ⇄ CEX Spot ⇄ DEX ⇄ Perp ⇄ Credit** 之间能够迁移风险的资产格式。

价格也不是某一家交易所喊出来的，而是不同市场的套利者拿资产负债表不断压价差以后形成的。

## **二十、以后判断 bStock 成熟度，不该只看 Volume 和上线数量**

如果只用 AUM、24h Volume 和 ticker 数量衡量 bStock，就埋没了它在市场结构里发挥的作用。

本文整理了对应的能力及其对应的指标：

![图24](assets/%E5%9B%BE24.webp)

Volume 和流动性只是其中一项，而且 gross volume 还会受到程序化回转和高频交易影响。

如果 Binance 想把 bStock 做成周末股票市场的库存层，Borrow Depth、可执行现货深度、Conversion Capacity 和 Perp-bStock basis 的稳定性，这可能比多上几百只 ticker 更重要。

## **二十一、定价权怎么验证：先看谁领先，再看价格能不能被大钱执行**

判断 Binance 有没有争到股票定价权，第一关应该是 **Lead-Lag**。必须把 Binance Perp、bStock 和同一秒的 Premarket last、bid、ask 放在一张图上。如果 Perp 全程跟随传统市场，那么它可以是一个好的 24/7 交易工具，却不能因为成交量大就获得价格领导地位。

第二关是 **Executability**。一个价格领先几十秒，但只能成交几千美元，专业资金也不会拿它作为参考。要知道 10 万、50 万、100 万美元会产生多少 impact，需要持续采集历史 L2 数据。

第三关是 **Two-way Correction**。Perp 贵时能不能压得住价格，bStock 贵时也能不能 Short，这取决于 inventory、conversion 和 borrow。没有双向套利，就没有价格发现。

最后一关是 **Reference Adoption**。这个指标是不能拿 Binance 自己的用户数据来证明。平台内部的跨产品用户和套利成交只能说明生态内部连接在形成。定价权的证据应该来自外面，比如说Bloomberg 有没有展示，OTC desk 有没有引用这个价格，第三方风控系统是不是参考了？其他 venue、oracle 或清算系统有没有引用。

因为定价权的体现：

> 不是 Binance 用户都在看 Binance，而是华尔街也开始拿着这个价格在周末去清算客户的风险敞口。

## **结语：Perp 抢定价权，bStock 决定这个价能不能变成市场价格**

如果我们把价格、成交量、交易频次、盘口和股票等等数据放在一起比较后，bStock 的定位将远超“tokenized stock”。

Perp 以后将会成为新的衍生品宠儿，除了交易量、操作之外，还是因为华尔街最终会知道 OI \> 股票的意义。OI 就是钱，而且还无需稀释控制权、无需对外披露、无税收管制（目前尚未）的钱。

Binance 如果要争传统股票闭市后的定价权，前面冲出去的是 Perp。NVDA 的一段样本里，Perp 成交量达到 bStock 的 99 倍；SNDK 六个纯周末成交 29.22 亿美元；SPCX 六个纯周末成交 15.46 亿美元；SKHYNIX 六个 UTC 周末窗口累计超过 24 亿美元。这些数字说明，传统股票关门以后，有一部分风险交易正在迁移到 24/7 衍生品市场。

同一批数据也说明，Volume 和 Price Discovery 不能画等号。SNDK 一个周末几亿美元可以把方向看反，四只股票合计 4.61 亿美元也可以一起错。SPCX 加入后，25 个美国闭市—重开样本的 Sunday 方向是 13/25，仍然只有 52%。SNDK 的 29.22 亿拆开以后是每秒约 7.3 个 fills，SPCX 的 15.46 亿则是每秒约 3.45 个 fills；这些高周转数字不能直接解释成同等规模的独立投资意见。

Perp 解决了“市场可以继续报价”的问题，但一个价格要获得可信度，还需要解决“报价错了以后谁来修”的问题。

bStock 就站在这里。它可以成为做市商 hedge Perp 的库存腿，成为 basis trader 的现货腿，成为长期投资者的底仓，成为 Lending 的 collateral，成为 DEX LP 的股票资产，也可以通过 Stock Conversion 管理跨时段 inventory。如果 borrow 补齐，它还会成为空头可以使用的证券库存，让高估的一边也能被攻击。

所以 Perp 和 bStock 的关系应该是：

**Perp 负责生产第一版价格，bStock 负责把第一版价格变成一件可以被持有、融资、搬运和证伪的资产。**

有一天，如果星期六 NVIDIA 出现重大消息，NVDAUSDT 的价格先动了，NVDAB 跟着重新定价，Perp-bStock basis 拉开，borrow rate 变化，DEX LP 开始被搬，做市商调整库存，套利者压缩价差；随后美国 Premarket 上线，再到 09:30 Opening Cross，那时候值得观察的已经不是 Binance 周末有没有猜对，而是 **谁的价格向谁靠拢**。

如果 Binance 一直在 Premarket 出现以后追着传统市场跑，它是一家 24/7 股票交易所。如果 Cash Market 重新开门以后，开始向过去几十小时在 Perp、bStock、DEX、Lending 和库存市场里反复交易出来的价格靠拢，而且外部 OTC、风控和行情系统也开始引用这套价格，那么 Binance 拿到的就不只是多几个交易小时，而是：

**股票市场的定价权。**

**这才是这套结构有可能具有划时代意义的地方。**

## **番外篇：Perp 和 bStock 的联动策略——从看盘到执行**

下面这些结构用于说明 bStock、Perp、Borrow、DEX 和 Collateral 怎么配合，不是针对某一只股票的交易建议。实操时最先做的不是下单，而是把几个价格和库存指标放进同一套交易体系内：Cash Close、bStock、Perp、Perp-bStock basis、funding、可借 bStock 数量、Borrow Rate、bStock 的 bid/ask 与可执行深度、Stock ⇄ bStock Conversion 状态，以及股票市场此刻是开着还是关着的。

最简单的 basis 公式是：

**Basis（bp）=（Perp - bStock）÷ bStock × 10,000**

如果 bStock 是 1,220，Perp 是 1,250，basis 约 246bp。这个 246bp 不是利润，真正能拿走的是 basis 扣掉两条腿的 spread、slippage、funding、borrow、资本占用、链上 gas 和转换成本以后剩下的部分。做每一笔交易前，先把这些成本列出来，避免看到价差就把它当成免费套利。

![图25](assets/%E5%9B%BE25.jpg)

**图8｜bStock 的组合拳：对冲、Basis、Borrow、DEX 与 Basket**

## **第一招：底仓不动，让 Perp 管短期 delta**

bStock 和 Perp 最基础的组合，是把长期持仓和短期方向拆开。假设持有 10 万美元 NVDAB，对 NVIDIA 半年的判断没有变化，但周末要把组合 delta 从 1 降到 0.4，那么需要减少的方向敞口是 0.6，对应可以 Short 约 6 万美元的 NVDAUSDT。这里的 6 万美元是名义金额示例，实际下单还要按合约乘数、保证金模式和账户风险限额换算。

这个结构最需要盯两件事。第一是 funding，如果短期对冲拖成几天甚至几周，funding 会从小成本变成主要成本；第二是 Perp-bStock basis，如果 Perp 在事件时段偏离 bStock 很多，Short Perp 虽然降了 delta，却多了一层 basis 风险。事件过去以后，先看 basis 是否恢复，再决定一次性平掉还是分批减掉 hedge。

## **第二招：Long bStock + Short Perp，专门打 Overshoot**

假设 SNDKB 在 1,220，SNDKUSDT 在 1,250，basis 约 246bp，同时 Short Perp 一侧还能收到 funding。执行前先做一个“成本预算”：假设两条腿合计 spread 和 slippage 60bp，持仓期间 funding 和资金成本 30bp，转账和其他摩擦再留 20bp，那么 246bp 的表面价差只剩约 136bp 的空间。

这类交易的入场条件不是“Perp 比 bStock 贵”，而是**价差大于全成本，并且两条腿都有足够深度**。离场也不需要等 basis 回到 0，可以预先设一个收敛区间。例如 basis 从 246bp 收到 80bp，剩下的 80bp 已经不足以覆盖继续持仓的风险，就可以退出。另一个退出信号是 funding 反向，或者 bStock 盘口变薄，导致现货腿无法按预期价格平仓。

## **第三招：Long Perp + Short bStock，先算 Borrow 再谈套利**

反向 Basis 更能检验市场结构。假设 bStock 比 Perp 贵 120bp，理论上可以 Short bStock、Long Perp，但 Short bStock 之前要先确认库存。这里最有用的不是“能不能借”，而是三个数字：**Available Borrow、Borrow APR、Borrow 是否可能被收回或触发额外保证金要求**。

Borrow 成本可以先换算成持仓期成本。假设借 10 万美元 bStock，APR 是 30%，只借两天，利息约为：

**100,000 × 30% × 2 / 365 ≈ 164 美元，约 16.4bp。**

如果 APR 变成 300%，两天成本就接近 164bp。此时一个 120bp 的 premium 从一开始就不够覆盖借券。YouCanShortIt 这类做空/借券路径如果用于反向 Basis，最先查的也是库存、APR、清算和召回规则，而不是只看屏幕上的 premium。

## **第四招：Stock ⇄ bStock，按时间管理库存**

Stock ⇄ bStock 更像库存迁移，不是周末提款机。一个实用的做法是把时间拆成三个窗口。现金市场开着时，做市商准备 Stock 或 bStock 库存；现金市场关闭后，主要靠 bStock、Borrow Pool 和 Perp 管理订单；到 Premarket 和 Cash Open 回来以后，再把临时库存迁回成本低的一侧。

因此周五收盘前要准备的不是一个方向观点，而是一张库存表：手里有多少 bStock、还有多少可以 borrow、Stablecoin 余额够不够、Perp 还有多少保证金空间、Conversion 是否可用。周末出现单边买盘时，MM 可以先用 Borrow 补 Ask，再用 Perp hedge；周一传统市场回来后，再用 Stock 和 Conversion 把借来的库存补回去。这个流程里，basis 是时间和资产负债表成本的价格。

## **第五招：bStock 抵押 → Stablecoin → Perp，先算清算缓冲**

拿 bStock 抵押借 Stablecoin，关键不是能借到多少，而是跌多少会进入危险区。实操可以先做一个压力测试：如果协议允许的最高 LTV 假设是 60%，账户不一定要借到 55%。如果把实际 LTV 控制在 30%–40%，再假设 bStock 一夜跌 20%，看看新的 LTV、健康度和 Perp 保证金是否还能承受。

这条链里至少有三层价格在动：bStock 本身、Stablecoin、Perp。股票下跌会推高 LTV，Perp basis 扩大可能增加 hedge 的保证金要求，Stablecoin 流动性又影响还款成本。把借出来的钱全部拿去加杠杆，会让几个风险共用一个触发器。更稳的做法是保留一部分 Stablecoin 作为追加保证金和还款缓冲。

## **第六招：DEX LP + CEX Perp，先算 LP 的股票 delta**

给 NVDAB/USDT 做 LP，不是放进去以后就只收手续费。随着价格移动，池子里 NVDAB 和 USDT 的数量会变化，LP 的股票 delta 也会变。实操时可以定时读取自己在 LP 里的 NVDAB 数量，用：

**LP 股票 Delta Notional ≈ LP 中 NVDAB 数量 × NVDAB 价格**

再用 NVDAUSDT Perp 做相反方向的名义对冲。LP 里的 NVDAB 增加，Perp 空头就要增加；NVDAB 减少，Perp 空头也要减。Concentrated Liquidity 的区间越窄，资金效率越高，但价格跑出区间后，LP 会失去一侧报价能力，所以周末事件风险必须放进区间设计里。

## **第七招：bStock → Collateral → Stablecoin → Perp，做一张多层清算表**

这条跨层循环容易让资本利用率变高，也容易让风险路径变长。实操前可以把风险拆成四栏：bStock 跌 10%/20% 会怎样，Borrow Rate 上升会怎样，Perp funding 反向会怎样，Stablecoin 脱离 1 美元会怎样。四种情况单独发生时账户是否安全，同时发生两种时是否还安全。

如果同一份 bStock 既是长期股票 exposure，又是借款 collateral，借出来的 Stablecoin 又拿去做 Perp 保证金，那么一次股票下跌可能同时降低抵押品价值和提高衍生品保证金压力。这种结构的重点不是把资本用到满，而是给每一层留退出通道。

## **第八招：四个市场一起盯，先锁住最稀缺的一条腿**

如果同一个股票风险同时出现在 Stock、CEX bStock、DEX bStock 和 Perp，最实用的工具是一张四市场价差表。每隔几秒或几分钟记录四个价格、bid/ask、可执行深度、Borrow 库存和 funding，再把每个市场相对参考价格的偏差换成 bp。

假设 NVDA Stock 是 200.00，Binance NVDAB 是 200.20，DEX NVDAB 是 201.00，NVDAUSDT Perp 是 199.60。表面上最贵的是 DEX，最便宜的是 Perp，但执行顺序要看哪一条腿最稀缺。需要 Short DEX bStock，就先确认 borrow 和 DEX 可成交深度；否则先 Long Perp，等到想卖 bStock 时才发现没有库存，会留下裸露方向仓位。跨市场套利最怕的不是判断错，而是四条腿只能成交三条。

## **第九招：Pundi X Basket，把单只 bStock 变成组合流动性**

Pundi X Basket 适合把 bStock 从单只股票扩展到主题组合。一个简单例子是建立四只成分的 Basket：NVDAB 40%、MUB 25%、SNDKB 20%、SPCXB 15%。这里的权重只是演示，Basket 的净值可以按各成分价格乘以权重加总，其他用户买入 Basket 后，底层执行需要把资金分配到各只 bStock。

真正需要设计的是再平衡和流动性。某只成分涨得太快，权重偏离目标，Basket 要不要按固定时间调仓，还是在偏离超过某个阈值以后调仓；某只 bStock 周末深度不足，是不是用对应 Perp 暂时承接 delta，再等现货流动性回来以后补库存；某只成分 borrow 枯竭，Basket 的做市商还能不能维持双边报价。Basket 做大以后，组合本身会把订单分发到底层 bStock，也会把底层的流动性问题放大出来。

## **最后一张实操清单**

一笔 bStock / Perp 联动交易开始前，可以先回答八个问题：**现在 Cash Market 开没开？Perp-bStock basis 是多少？两边 10万/50万美元的可执行深度够不够？funding 是付还是收？bStock 有多少可以 borrow？Borrow APR 折算到持仓期是多少 bp？Conversion 是否可用？最坏情况下哪一条腿最难退出？**

这八个问题如果有一项没有答案，就先把它当成未定价成本，而不是当成 0。bStock 这套体系的机会来自市场之间的连接，风险也来自这些连接。做得好的时候，Perp、bStock、Borrow、DEX 和 Stock 会互相压缩价差；做得差的时候，其中一条腿断掉，就会把原本的相对价值交易变成方向交易。

注：文中交易结构与数值示例用于说明市场机制；图表用于辅助读图，数据口径以正文表格和原始样本定义为准。
