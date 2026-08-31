# Nasdaq关门以后，谁来报价：Stock Perp、Oracle和一场24/7的股票定价战争（一）

- Author: Danny (`@agintender`)
- Published: 2026-08-26 19:02:46 +08:00
- URL: https://agintender.substack.com/p/nasdaqstock-perporacle247
- Subtitle: Stock Perp的合约算法镰刀（一）：Binance的股票休市算法有什么漏洞？
- Source Type: Substack 付费长文（由用户提供的本地 MHTML 归档）
- Capture Note: 正文与 16 张有效配图已从 MHTML 提取；页面导航、头像、交互控件和评论区未纳入正文。

## **从《华尔街》到《繁花》，价格总要有个地方先醒过来**

1987年的《华尔街》里，一笔股票order从电话那头传进纽约证券交易所，clerk写ticket，runner拿着单子穿过交易大厅，交易员在一片叫喊声里完成成交，新的价格再通过ticker传出去。今天手机上按一下按钮就结束的事情，当时是真的要”人肉“在交易大厅里面奔跑。

![图01](assets/%E5%9B%BE01.webp)

《繁花》里的上海也差不多。九十年代初，浦江饭店孔雀厅里坐着红马甲，大屏幕不停跳报价；再早一点，阿宝要买股票，还得拿着地址去西康路柜台找人办手续。纽约和上海隔着一个太平洋，但有一件事是一样的：**价格不是从天上掉下来的，总要先有一个地方真的发生交易。**

![图02](assets/%E5%9B%BE02.webp)

以前金融市场忙着解决一个问题：价格出来以后，怎样快一点传出去。交易大厅里面先有人喊出新的报价，外面的ticker才会跟着动；上海柜台有人买卖，行情板上才会出现新的数字。

![图03](assets/%E5%9B%BE03.webp)

Stock perp把这个问题倒了过来。（延伸阅读：[bStocks 的 B 面](https://agintender.substack.com/p/bstocks-b-nasdaq-perp)）假设星期五NVDA收100美元，Nasdaq关门。星期六英伟达突然出了一条大新闻，有人觉得星期一应该105，有人觉得108。股票市场没开，没有新的NBBO，但Binance、Bitget、OKX、Hyperliquid上面的NVDA perp还在交易，107有人买，108有人卖。

于是问题来了：**原来的交易大厅关灯以后，谁来接过报价板？**

过去是：

**股票成交 → 报价 → 全市场知道**

现在周末可能变成：

**Perp成交 → Oracle/Index → 其他市场**

等Nasdaq重新开门，大家再回来对答案。

这里最关键的角色，就是Oracle。Binance后面可能接着Pyth、dxFeed、Kaiko和自己的orderbook；OKX可以同时参考Pyth、Ondo和其他交易所perp；DEX又可能用Chainlink或者Pyth。到了真正没有外部股票价格的时候，有的平台干脆让自己的perp盘口顶上来。

所以这篇文章不是只想讲“Mark price怎么算”，而是深扒整个信息传播链条：**股市休市以后，价格从哪里产生，谁负责把它传出去，最后谁又敢拿这个价格去交易清算。**

——————————————————————————————————————

写在前面：本文将会分为上中下三部曲（分三次发）：

第一部分是前言、背景介绍、Binance和Bitget的Oracle/Index算法分析；

第二部分是Trade.xyz、OKX/MEXC/Gate/Bybitd算法分析；

第三部分是算法分析、策略分析和数据实证分析

准备好之后，就请大家准备好纸巾和咖啡，跟我一起深入算法的无尽海洋吧～

## **二、啥子是Oracle、Index、Last和Mark？**

假设星期五美股收盘，NVDA真实股票最后成交在100美元。你星期六打开某个crypto交易所，可能同时看到四个数字：Oracle/reference price是100，Index Price是100.20，Last Price已经被人扫到108，Mark Price却只有101.10。

![图04](assets/%E5%9B%BE04.webp)

第一次看的人很容易问：为什么都是“NVDA”，可以同时有四个价格？因为这四个数字根本不是在回答同一个问题。

**Oracle**比较像原材料。它告诉平台：“外面的股票世界最近给我的信息是什么？” 这个信息可能来自Pyth、Chainlink、dxFeed、Kaiko、Ondo，也可能来自其他交易venue。Oracle不是上帝价格，也不一定是一家公司，它只是交易所拿来做Index的输入。

**Index Price**是交易所自己加工后的参考价。假设平台同时看到Pyth 100、dxFeed 100.1、另一家股票perp 100.5，再加上自己的一些reference，经过加权、EMA、异常source剔除、band保护以后，最后可能得到100.20。Index其实是在说：“我把我信的几个source揉在一起以后，觉得现在合理参考价在这里。”

**Last Price**最好懂，就是这家perp刚刚最后一笔真实成交。星期六一个买家很急，直接把卖盘扫到108，那Last就是108。它最“真”，因为真的有人在108成交；它也最容易骗人，因为一笔单就可以把它打得很远。交易所的官方文档也提醒，Last最即时，但流动性薄的时候最容易被大单影响。

**Mark Price**才是最接近风控系统的那个价格。交易所拿Index，再加funding basis、自己盘口的mid-price basis、Last等东西过滤一次，最后得到Mark。很多平台算未实现盈亏、保证金和强平，看的是Mark，不是Last。

所以Last突然从100打到108，而Mark还在101，不代表交易所没有跟着市场行情了，反而说明风控到位，因为外面的世界还没下雨。你如果刚好在108买进去，成交价当然是108；但你的仓位会不会被强平，很多平台看的是Mark。

可以把四个价格记成下面这张表：

![图05](assets/%E5%9B%BE05.webp)

正常时候，价格大概沿着这条线走：

**TradFi/Oracle → Index → Mark**

但perp自己的orderbook又有另一条线：

**Orderbook → Last/BBO/Basis → Mark**

到了休市的时候，有些平台干脆把第二条线接回第一条：

**Orderbook → InternalIndex/Oracle**

这就是stock perp最有意思的地方。它不是单纯跟股票，而是在某些时段自己开始生产价格。

## **三、Oracle也有分工：Chainlink和Pyth看起来都在报股票价，做的生意其实不太一样**

现在stock perp里面最值得看的两个Oracle体系，一个是Chainlink，一个是Pyth。

Pyth强调first-party data。它让exchange、market maker、trading firm、bank这些本来就在产生价格的人直接做publisher，再把大家的报价聚合成price和confidence interval。Pyth Pro/Pro X又把低延迟、跨资产覆盖、API/WebSocket、same-day listing这些交易所很在意的东西做得相对丝滑。对CEX来说：matching engine和risk engine都在自己的服务器里，最关心的是快不快、稳不稳、coverage够不够，拿到数据以后直接塞进系统就可以，至于数据是不是经过多层验证反而不是这么重要。

Chainlink Data Streams的出发点不太一样。DEX最后要在smart contract里面执行、结算、清算，所以它不能只说“后台API告诉我NVDA是100，大家信我”。Chainlink会先在offchain聚合和签名report，协议需要的时候再pull onchain验证，而且股票Streams不只给一个价格，还给market status、staleness、bid/ask、volume、last trade这些上下文。对DEX来说，价值不只是“NVDA=100”，还有“这个100的价格是谁认证的、现在是什么session、数据是不是stale、智能合约能不能验证”。

所以Pyth的设计逻辑更多是从“市场数据基础设施”的角度出发，而Chainlink更侧重在“可验证执行基础设施”。最后两个都会碰到stock perp，只是卖点不一样。

在进入硬核的算法正文之前，请大家先看下，对perp的几个“价格”有个大致的了解：

![图06](assets/%E5%9B%BE06.webp)

## **四、Binance：美股休市以后，自己的盘口开始接手报价**

Binance这套东西最关键的一次变化，发生在2026年5月16日（有懂的人知道发生了什么事吗？哈哈哈）。从这一天开始，equity TradFi perp在外部股票报价不可用、Index进入休市定价模式以后，不再像旧的Fixed Mode那样拿着休市前的股票价格等美股重新开门，而是切进 **Orderbook EWMA Mode**，让Binance自己的perp orderbook参与生成新的Index。

![图07](assets/%E5%9B%BE07.webp)

这是一次看起来common sense，但是并不common的算法升级，逻辑和性质其实变了。以前是Nasdaq最后告诉Binance：“NVDA现在100。”Nasdaq下班以后，Binance还是按照100的价格来做交易。现在的逻辑变成：外面的股票市场暂时不给答案，那我先看看自己这里的人愿意多少钱买、多少钱卖，再从自己的盘口里算一个新的参考价格出来 —— **当外部股票报价不可用、对应合约进入Orderbook EWMA Mode时，Binance自己的盘口开始接班。** 对美股来说，最典型的就是周末和节假日。

### **4.1 第一步，不要急着看Last，先看看盘口的深度**

Binance并不会因为最后一笔NVDA perp突然成交在108，就马上认定NVDA现在值108。Last太容易被薄盘口以及场内的成交搞出一根针，所以它先算 **Impact Bid Price** 和 **Impact Ask Price**，再取两边的中间：

**Impact Mid = (Impact Bid + Impact Ask) / 2**

这里的Impact Price不是“订单簿深度”本身，它更像是**把订单簿深度折算成一个可执行（冲击）的价格**。

![图08](assets/%E5%9B%BE08.webp)

Binance RCH对Impact Bid和Impact Ask的定义，是按照一笔指定的 **Impact Margin Notional** 去吃bid或者ask那一侧的盘口，看平均会成交在哪里。你可以理解为假设一笔市价单的单子能够打到哪一层的order？这个Impact Margin Notional又跟合约最高杠杆对应的Initial Margin Rate有关：

**Impact Margin Notional = 200 USDT / Initial Margin Rate at Maximum Leverage**

例如某只股票perp最高杠杆是10倍，对应Initial Margin Rate大约10%，那这笔Impact Notional就是大约2,000 USDT。

![图09](assets/%E5%9B%BE09.webp)

所以Binance问的不是：“刚才有没有一笔交易是在108买过？”

它问的是：“**如果按规则拿2,000 USDT真的进去吃盘口，平均到底要在哪里成交？**”

这个差别很大。假设NVDA星期五收100，星期六流动性很薄，有人用一笔不大的市价单把Last扫到108，但100附近还有一堆真实买卖盘，那108更像一根针，Impact Price未必会跟着移动。

反过来，如果真出了重大新闻，原本100附近的卖单不断被吃掉，新的bid堆到107，ask也搬到108，而且这个价格后面有真实深度，那么Impact Bid、Impact Ask和Impact Mid才会一起往上走。

所以Impact这一层做的事情，就是：**别拿一个成交价来骗我，你得把一段真正能成交的盘口一起搬过去。这个无形中就增加了“操纵”成本。**这也是它比Last更抗插针的地方。

### **4.2 第二步，盘口搬过去了，Index也不会马上采纳**

即使Impact Mid已经从100跑到108，Binance也不会啪一下把Index从100改成108，中间还隔着一层EWMA。

休市时这条链可以写成：

**Binance Orderbook → Impact Bid / Ask → Impact Mid → EWMA → Index**

EWMA可以理解成一个很简单的态度：“108我看到了，但你先在那里站一阵子。”

如果Impact Mid只是几秒钟跳到108，很快又回来，EWMA不会全部吃进去；如果整个盘口持续在107、108附近成交，Index才会一点一点被拉过去。Binance还明确写了，Orderbook EWMA Mode下Index本身的movement也有限制。

![图10](assets/%E5%9B%BE10.webp)

所以前面Impact过滤的是**空间上的深度**，后面EWMA过滤的是**时间上的持续性**。

这里有一个很关键的参数，到现在Binance还是没有公开。一般EWMA可以写成：

**EWMA(t) = α × P(t) + (1 − α) × EWMA(t−1)**

真正决定它多快相信新价格的是α，也可以换成half-life、decay constant或者τ来表达。Bitget的官方文档写的是τ是300秒，trade\[XYZ\]现在equity Internal Oracle的τ是30分钟，但Binance最新RCH文件依旧只写 **“EWMA determined by Binance RCH”**，没有把具体α、half-life或者τ摆出来。（本文实际测算下来的数据大约是在300秒左右）

### **4.3 第三步，Index只是参考价，真正拿去管仓位的Mark还有另一套算法**

到这里很多人很容易产生一个误会：Impact Mid经过EWMA变成Index，那Mark是不是就等于Index？

也不是。

按照Binance当前更具体的TradFi产品规则，当Index处于Orderbook EWMA Mode时，Mark继续走regular Mark Price calculation，大体上是三个价格取中位数：

**Mark = Median(Price 1, Price 2, Contract Price)**

Price 1主要是Index再加上funding basis。Price 2则很有意思，因为Binance自己的盘口会从这里**第二次**进入Mark：

**Price 2 = Index + Moving Average Basis**

这里的Basis来自：

**Basis = (Best Bid + Best Ask) / 2 − Index**

而Binance RCH当前规则对这一层使用过去 **30秒** 左右的orderbook basis moving average。第三项Contract Price，就是Last Traded Price。

于是Binance自己的盘口其实有三条路往风险系统里面传。

第一条是比较慢的：

**Orderbook → Impact Price → EWMA → Index**

第二条是：

**Best Bid / Ask → 30秒Basis → Price 2 → Mark**

第三条则最直接：

**Last → Contract Price → Mark**

这就解释了一个很重要的现象：**Index还没完全追上场内价格，不代表Mark一定原地不动。**

假设周末NVDA的EWMA Index还在102，但Binance自己的perp已经连续一段时间在105附近交易。Best Bid和Best Ask都移动过去以后，Price 2里面的30秒basis会开始变大，Last也可能在105。Mark没有必要等Index逐步爬到105以后才反应，它也可以先往场内方向移动。

所以以后看stock perp，不能只盯Index。

**Index走得慢，Mark未必慢。**

这里还有一个官方文档留下的一个模糊的点：Binance当前针对Orderbook EWMA的TradFi产品页明确写，这种模式下继续采用regular Mark calculation；但当前RCH Clearing Procedures里还有一条更宽泛的描述，说TradFi perp在regular session之外会对recent execution prices做EWMA。

### **4.4 第四步，Mark会听场内交易，但周末最多只能离Index跑3%**

Binance现在还给equity TradFi perp的Mark套了一道很重要的护栏。

在regular、pre-market、after-hours和overnight这些阶段，Mark相对Index的最大偏离通常是 **±5%**；到了weekend和holiday，这个范围会收紧成 **±3%**。

这个±3%很容易被误会。

它不是说“Binance周末NVDA最多只能涨3%”，而是说：

**Mark相对于当前Index，最多先跑3%。**

假设周末Index还在100，但是因为场内买盘很疯狂，按照Last和30秒basis算出来的Calculated Mark已经到了106。最终Mark不不会一下就跳到106，而会先被卡在这个3%的上限：

**100 × 1.03 = 103**

但如果Impact Mid持续往上，EWMA Index后来自己从100走到了102，Mark的上限也会跟着变成：

**102 × 1.03 = 105.06**

如果Index继续来到105，上限又会搬到：

**105 × 1.03 = 108.15**

所以这里的关系不是“Mark能不能变动”，而是**Index往前走一步，Mark的限制也跟着往前逐步移动。**

![图11](assets/%E5%9B%BE11.webp)

这套设计蛮聪明。它允许周末市场自己做price discovery，又不希望清算价格因为场内一阵情绪就比参考Index快跑太远。至于为什么设计成这个样子，是因为Mark是决定了一个仓位是否会被清算的关键。

### **4.5 第五步，连“换班”不能硬切，而是用30秒慢慢交接**

两套定价机制怎么交接换班？总不可能直接切过去，打出一根天地针。（你别笑，还真有交易所是这么干的～）

假设星期五股票市场还开着，Index主要跟着外部vendor。后来外部市场关掉，Binance要从原来的Index算法切去Orderbook EWMA；又或者周末结束，external price重新回来，要从Orderbook EWMA切回正常模式。

Binance现在不是直接在某一秒直接把旧Index换一个当天的开盘价上去，而是做weighted blend：

**Price Index = (1 − t / Window Length) × Old-Mode Index + (t / Window Length) × New-Mode Index**

当前RCH Clearing Procedures把这个预设的 **Window Length写成30秒**。

而且这个transition不只是external重新回来时才有，官方写的是：**进入和退出Orderbook EWMA Mode都会做blend。**

也就是说，周末开始时，从TradFi世界切去Binance自己的orderbook，要慢慢迁移roll；周末结束后，从自己的orderbook重新切回external price，也要慢慢迁移。

假设周末Binance自己的Index已经交易到108，external重新回来以后，新外部Index是104。如果按照当前30秒window，大概会经历这样的过程：

10秒左右，Index约106.67；

15秒左右，Index约106；

30秒以后，才全部切到104。

所以它不是：**108 → 104**

一秒钟砸下去。

而是：**108 → 106.x → 105.x → 104**

在一小段时间里完成交接。

![图12](assets/%E5%9B%BE12.webp)

Binance面向用户的TradFi页面写得比较宽，说transition会在 **1分钟以内** 完成，同时保留调整window length的权利；当前RCH Clearing Procedures则把预设参数明确写成30秒。**现在的基准值是30秒，但交易所保留调整空间。**

### **4.6 小结**

现在回头看，Binance在股市休市期间的定价算法可以概括为：

> **Orderbook先提出价格 → Impact Price检查后面有没有可执行深度 → EWMA决定Index多快相信 → 30秒basis和Last让Mark也听到自己场内 → 周末±3%限制Mark不要离Index跑得太快 → 进入和退出休市模式再用30秒weighted blend交接。**

每一步都用它的作用，看似漫不经心，实则都是一环扣一环的巧思：

1.  Impact防的是针：你不能只打一笔108给我看，你得真的用真金白银把盘口填上。

2.  EWMA防的是阵风：你吹过去可以，但你还需要吹好一阵才行，我才慢慢认可这个价格。

3.  Mark里面的30秒basis解决的是另外一个问题：自己的perp市场都交易到105了，清算系统也不能掩耳盗铃。

4.  周末±3%的护栏则是在说：看到归看到，但不要一下拿一个远离Index的价格去爆一大片仓位。

5.  最后30秒transition处理的是两套价格在交接的时候不要火星撞地球，造成太大的波动。

如果你单看其中一条，其实真的也就这样，但是连起来真的就不一样了。这也是Binance这次改算法最有意思的地方。以前股票perp的逻辑很简单：

**Nasdaq先产生价格，Binance负责跟。** 现在到了真正没有外部股票报价的时段，关系开始反过来：

> **Binance自己的交易者先出价，订单簿先把信息交易出来，然后算法决定这份价格有没有足够的深度、持续时间和可信度，够不够资格一步一步变成Index和Mark。**

好处是周末如果NVDA、TSLA、SNDK真的出了大新闻，Binance不需要一直参照周五收盘价。自己的市场可以先把新信息交易出来，而一笔孤立的Last又比较难直接穿透Impact、EWMA、Median和Mark deviation一路变成清算价。

代价也很清楚。当自己的orderbook成为休市Index的重要输入以后，Binance就不再只是“追踪美股”的crypto venue。

**它自己的orderflow，开始拥有一部分生产股票参考价格的能力。**

而真正的问题也从这里开始：如果Binance先生产出这个价格，OKX、MEXC、Gate随后把它吸进自己的Composite Index，其他market maker和套利机器人又跟着调整报价，那么到了最后，我们看到的是七家交易所都报108，还是一开始只有Binance这一个市场真正先说出了108？

## **五、Bitget：跟Binance是同一派，但它把参数都摆在桌面**

Bitget跟Binance走的是同一条路：传统股票市场开着的时候，Index主要听外面的价格；等传统市场关门，外部报价停下来以后，自己的perp orderbook开始接班。

两家的方向很像，只是Bitget把很多关键参数直接写出来了。Binance告诉你自己会看Impact Price、会做EWMA，但EWMA到底多快没有公开；Bitget连Impact Notional、时间常数、更新频率、单步变化限制都摆在官方文档里。

![图13](assets/%E5%9B%BE13.webp)

### **外面有股票报价的时候，还是先听TradFi**

传统市场正常交易时，Bitget的Index处于External Mode。官方的做法是从多个外部数据源取得股票报价，再做weighted average，Index每200ms更新一次。

目前Bitget针对stock perp的说明里，列出的传统股票数据供应商包括：Pyth、dxFeed、Massive、Intrinio。

正常美股交易时段，大致还是这条线：

Pyth / dxFeed / Massive / Intrinio → External Index → Bitget Stock Perp

这个阶段，Bitget主要还是价格接受者。Nasdaq、NYSE和数据vendor告诉它股票在哪里，它再把这个价格带进perp。

有意思的是外部股票市场关门以后。

到了daily maintenance、weekend或者holiday这类传统市场休市时段，Bitget会让Index进入Internal Mode，由自己的perp orderbook继续生成新的参考价格。

### **5.1 第一步：先拿2,000 USDT试一下，这个盘口是不是托儿？**

Bitget休市以后也不会直接拿Last做Index。

它先设定一笔Impact Notional，当前官方默认参数是：

Impact Notional = 2,000 USDT

系统拿这2,000 USDT去模拟吃bid和ask两边的订单簿，分别算出：

P_impactBid 和 P_impactAsk

这里跟Binance的Impact逻辑很像。它是在问：

“如果现在真的拿2,000 USDT进去成交，平均到底能成交在哪里？”

假设NVDA星期五收100。星期六市场很薄，有人用一笔不大的订单把Last打到108，但2,000 USDT真正进去，大部分仓位还是只能在100、101附近成交，那这个108基本不会形成Internal Index。

反过来，如果真出了大新闻，100附近的卖盘被一路吃掉，新的bid已经堆到107，ask也搬到108，拿2,000 USDT进去也只能在107、108附近成交，Bitget才开始承认：Okay，这次不是一根针，整个盘口真的在移动了。

所以Impact Price的作用，不是单纯看最后成交，而是拿一笔规定规模的钱去“验牌”。

### **5.2 第二步：盘口想把Index往哪里拖，先算一个IPD**

算完Impact Bid和Impact Ask以后，Bitget不会直接拿两者平均值当Index，而是先算一个IPD：

IPD = max(P_impactBid − S, 0) − max(S − P_impactAsk, 0)

这里的S，就是当前Internal Index。

各位看到公式别慌，你就想成如果整个orderbook已经搬到当前Index上面，P_impactBid会高过S，IPD就变成正数 aka “盘口现在在往上拉Index。”； 如果整个orderbook掉到Index下面，P_impactAsk会低过S，IPD变成负数 aka“盘口现在在往下拖Index。”

如果bid和ask还围绕着原来的Index，IPD就会很小，甚至接近0。

所以Bitget不是直接问：

“新的价格应该是多少？”

它反而会问：

“现在这张orderbook想把Index往哪个方向拖，而且拖多远？”

这个IPD才是后面EMA真正需要考虑的参数。

### **5.3 第三步：五分钟EMA，决定Bitget多快相信自己的盘口**

有了IPD以后，Bitget每200ms推进一次Internal Index：

S_new = S_old + (1 − β) × IPD

其中：

β = exp(−Δt\* / τ)

而：

Δt\* = min(Δt, c × τ)

现在最重要的参数，Bitget都公开了：

**τ = 300秒**

**c = 0.1**

**Impact Notional = 2,000 USDT**

300秒就是5分钟。（Binance也告诉你自己用EWMA，但具体τ不公开。Bitget则直接告诉你，Internal Index的默认时间常数是300秒。）

所以假设NVDA周末出了大利好，perp盘口从100整体搬到108，Bitget不会啪一下把Internal Index改成108。新的盘口需要持续在那里“说服”系统，EMA才会慢慢把Index往上拉。一根108的针不会打穿index。

你得让2,000 USDT Impact Price持续停在107、108附近，Index才会一点点过去。

所以如果以后真要做stock perp的lead-lag研究，Bitget是一个很好用的样本。因为这里我们不只是知道“它会平滑”，还知道它大概用什么时间尺度去做平滑调整。

### **5.4 Bitget并没有把钥匙全部交给orderbook：外面还有一道±10%的院墙**

Bitget的Internal Index并不是可以无限跟着自己的perp市场跑。

在传统市场日内休市、weekend或者holiday期间，Bitget会拿External Mode最后一个有效的closing price作为reference，并给Internal Index加一道Index Price Protection。

当前默认范围是：

最后External Price ±10%

假设NVDA外部市场最后的参考价是100，那么Internal Index默认的活动范围大致就是：

90 ≤ Internal Index ≤ 110

这是一种有限的price discovery。

如果星期六出了一个超级大利好，市场觉得NVDA应该值125，Bitget自己的Last当然可能有人在115、120甚至更高的地方成交，但Internal Index默认不会一下跟过去，它会需要一步步地破开这些“枷锁”。

这个细节对后面比较Binance、Bitget和trade\[XYZ\]很重要。

三家都允许自己的orderbook在传统市场休市时参与价格发现，但“能走多快”和“能走多远”不是一回事。Bitget把两件事都写出来了：

> 多快——τ = 300秒。
>
> 多远——默认离最后External参考价±10%。

### **5.5 Mark的算法结构**

Bitget普通perp的Standard Mark公式是比较清楚的，Mark每200ms更新一次，核心是三个价格取中位数：

Mark = Median(Price 1, Price 2, Price 3)

其中：

Price 1 = Last Price

Price 2 = Funding-adjusted Index

Price 3 = Index + 30秒 Orderbook Basis Moving Average

Price 3 里面的basis来自：

Basis = (Bid1 + Ask1) / 2 − Index

官方规则是每秒采样一次，取过去30个点，做30秒的moving average。

所以在Standard Mark模型下，Bitget自己的perp市场也有几条路进入Mark：

Last → Price 1

Index + Funding → Price 2

Best Bid / Ask → 30秒Basis → Price 3

最后取median。

问题出在休市以后。

Bitget关于stock-perp资料和FAQ都指向休市Mark采用EMA / EWMA smoothing，并且休市Mark有平滑机制和最终deviation protection。

### **5.6 休市以后，Bitget其实同时在平滑两个东西：Index和Mark**

Bitget这里最容易让人混淆的地方，是官方一会儿写EMA，一会儿写EWMA。不过好在数学上这两个属于同一个家族：都是让“最近的数据权重大，越老的数据权重越小”，所以在理解和算法上的影响不大。

Bitget这套stock perp里面，它们用在了两个不同的地方。

第一套，是前面已经讲过的 **Internal Index EMA**。这一套公式公开得很完整：

S_new = S_old + (1 − β) × IPD

β = exp(−Δt\* / τ)

τ = 300秒

它喂进去的参数并不是Last，而是通过2,000 USDT Impact Notional算出来的orderbook偏差信号IPD。也就是说，这套EMA在回答：

**“自己的整张盘口，持续多久以后，我才愿意把Index往那边搬？”**

这一套我们知道得比较清楚：200ms更新、τ默认300秒、c=0.1、Impact Notional默认2,000 USDT。

但到了 **Mark**，Bitget又加了一层EMA / EWMA smoothing。

这层解决的是另外一个问题：

**“即使自己场内突然交易到108，我要不要马上拿108去算PnL、保证金和强平？”**

答案显然是不想马上全信。

Bitget当前的Traditional Asset FAQ直接写，传统市场休市期间，Mark进入 **EWMA smoothing mode**，把最新的市场成交价格经过Exponentially Weighted Moving Average处理以后，再得到Mark，目的就是避免休市期间出现sharp price swings，让价格保持连续

它大致可以抽象成：

Mark_EWMA(t) = β × Mark_EWMA(t−1) + (1 − β) × X(t)

这里的X(t)，就是这一刻Bitget拿来喂给Mark平滑器的场内价格输入。

如果上一刻Mark是100，新输入突然变成108，那么Mark不会直接：

100 → 108

而是先变成：

100 + (1 − β) × 8

只要β大于0，它第一步就不会把这8美元全部吃进去。

如果108只存在一两下，很快又掉回100，EWMA Mark可能只往上挪一点，然后又回来。

如果场内价格持续很久都在108，那么新的108会不断获得权重，老的100权重越来越低，Mark才会慢慢向108靠近。

这就是EWMA在这里真正干的事情：

**如果就是一根针打上来的价格，我不予采纳；但你如果一直在这里成交，我才开始相信。**

### **5.7 这里有一个问题：Bitget没有把休市Mark的EWMA参数像Index那样全部公开**

Bitget明确公开的是：

Internal Index EMA：

τ = 300秒

但是针对off-hours Mark的EWMA，目前官方没有在我们查到的页面里公开：

- β到底是多少；

- τ是多少；

- half-life是多少；

- 多久更新一次；

- 有没有单步change cap；

- EWMA具体吃Last、mid，还是某种quote-derived price。

5分钟这个参数，目前确认的是 **Internal Index**，不是Mark。

### **5.8 Bitget对Mark的参数输入条件是啥？官方自己也有两种说法**

这是现在这套规则最值得写出来的地方。

Bitget的Traditional Asset FAQ写的是：

**latest traded market prices → EWMA → Mark**

也就是听起来比较像：

Last / recent trades → EWMA smoothing → Calculated Mark。

但Bitget 2026年4月28日那篇专门介绍stock perp的Quick Guide又写：

**Outside traditional market hours: calculated from order book quote data using EMA.**

也就是说这里描述的输入更像是：

Orderbook quotes → EMA → Mark。

这两个说法并不完全一样。

一个说“最新成交价格”。

一个说“盘口报价数据”。

本文认为：**休市时Mark会从Bitget自己场内的成交/盘口报价中取输入，再做EMA/EWMA平滑**

### **5. 9 为什么Mark还要再做一次EWMA？因为Index和Mark是在防两种不同的风险**

这点可以用一个NVDA例子看得很清楚。

假设星期五NVDA最后External Price是100，星期六出了一个新闻，Bitget的perp orderbook开始往108走。

Internal Index先走第一套算法：

2,000 USDT Impact检查盘口\
→ IPD\
→ 300秒EMA\
→ Internal Index慢慢从100往108走

假设过了一小段时间，Index才走到102。但这个时候Bitget自己的perp已经有人不断在107、108附近成交。

如果Mark完全等Index，那清算系统可能太迟钝；如果Mark直接等于Last=108，又太敏感；所以EWMA这个平滑机制的出现就应运而生了。

它看到场内108了，但不是直接全信108，而是：

100/102附近的旧Mark\
→ 不断吸收107、108的新成交/盘口信息\
→ EWMA Mark逐渐上移

这就是为什么Bitget可能出现这种一物多价的状态。：

Last = 108\
Internal Index = 102\
Calculated Mark = 104.x

### **5. 10 最后还有一道deviation clamp，所以EWMA出来的Mark也不是最终Mark**

就算EWMA算出来一个Calculated Mark，Bitget后面还有最后一道“防脱”程序：

Actual Mark = clamp( Calculated Mark, Index × (1 − d), Index × (1 + d))

对于NVDA、TSLA、AAPL、MSFT、GOOGL等很多stock perp，目前官方表里d是5%。

于是整个休市Mark机制可以理解成两步。

第一步是：

场内成交 / orderbook quote\
→ EMA / EWMA\
→ Calculated Mark

第二步：

Calculated Mark\
→ Index ± deviation constraint\
→ Actual Mark

假设：

Internal Index = 102

场内价格 = 108

EWMA算出来Calculated Mark = 107

而NVDA当前deviation constraint = 5%。

那Mark最高允许到： 102 × 1.05 = 107.10

所以107可以通过，但如果EWMA算出来108.5，那么最终Mark还是只能先被卡在107.10附近。

换句话说，EWMA负责： **“别因为一两笔交易让价格跳得太快。”；**

deviation clamp负责： **“就算你持续交易得很远，也别让清算价离Index太远。”**

### **5.11 Bitget休市期间，其实是“两台平滑器 + 两道围栏”**

把整套东西画出来会更清楚：

**第一条，Index：**

Orderbook\
→ 2,000 USDT Impact Price\
→ IPD\
→ 300秒EMA\
→ Internal Index\
→ 最后External Price ±10% Band

**第二条，Mark：**

Local trades / orderbook quotes\
→ EMA / EWMA smoothing\
→ Calculated Mark\
→ Current Index ±5%（很多stock perp）\
→ Actual Mark

它其实有两个时间尺度。

一个是**Index多快相信orderbook**，这个我们知道：默认τ=300秒。

另一个是**Mark多快相信场内交易价格**。

而这恰恰会影响后面的lead-lag分析：如果周末真出新闻，我们不只要问“Bitget Index多久从100走到105”，还要问：

**Last先到108以后，Mark到底领先Index多少、领先多久？**

这个delta，才是Bitget休市算法真正值得验证的地方。

## **六、贪吃蛇困境：当Oracle开始吃自己的尾巴，Binance休市算法的风险与代价**

前面把Binance这套休市算法拆开来看，会觉得它设计得很有巧思：Last容易插针，那就看Impact Price；盘口突然跳一下被操纵和干扰，那就再过一层EWMA；Index走得慢，Mark又可以通过场内价格做调整；Mark怕跑太远，那就再套一个周末±3%的护栏；最后外部股票市场重新开门的时候，两套价格用30秒慢慢交接。

![图14](assets/%E5%9B%BE14.webp)

每一层单独看都有道理。

问题是，金融系统里面最麻烦的东西，往往不是某一个公式写错，而是几个正确的模块连起来以后，形成了一条新的反馈链，这个时候影响不是相加而是相乘。

Binance这套算法最大的变化，就是休市以后，原本来自外面的股票参考价，开始由Binance自己的perp orderbook生产。于是产生价格的人、使用价格的人、被这个价格清算的人，第一次集中到了同一个市场里面。

正常股票市场开门的时候，链条大概是：

**Nasdaq成交 → 外部Data Vendor → Binance Index → Mark → Liquidation**

外面的股票市场负责生产价格，Binance拿回来做风险管理。就算Binance自己的perp突然抽风，也还有一个外部参考系在那里。

但周末进入Orderbook EWMA Mode以后，链条变成：

**Binance Orderbook → Impact Price → EWMA → Index → Mark → Liquidation**

如果再把清算发生以后会发生什么补进去，整条线就不再是一条直线，而是一个圆：

**Orderbook → Index → Mark → Liquidation → Orderbook （贪吃蛇既视感有木有）**

![图15](assets/%E5%9B%BE15.webp)

这就是这套设计真正需要小心的地方。

### **6.1 Impact Price防得了插针，但不代表操纵者一定要拿真钱成交**

这里先纠正一个很容易产生的误解。

前面为了方便理解，可以说Impact Price是在问：“假设拿2,000 USDT进去吃盘口，平均会成交在哪里？”这个说法没问题。但如果进一步说成“所以你想操纵Index，就必须真的拿2,000 USDT把盘口打过去”，就不准确了。

Impact Price本质上是根据当时orderbook计算出来的一笔**模拟成交价格**。

它看的不是刚才真实发生了多少成交，而是：如果现在有这样一笔市价单进来，按照眼前挂出来的bid和ask，它理论上会成交在哪里。

这个区别很重要。

因为订单簿上的流动性，不等于已经成交的流动性。

一个做市商完全可以挂出一批订单，又在市场快碰到的时候撤掉；多个账户也可以通过改变报价位置、撤单和重新挂单，改变算法看到的orderbook形状。传统市场里面的spoofing和layering，玩的就是这个东西。

所以Impact Price确实比Last强很多。

Last只要打一笔成交就能动。

Impact Price要求你影响一整段盘口。

但它真正提高的是：**操纵订单簿的难度。**

不是把系统变成：**只有真实成交才能影响Oracle。**

这两者不是一个事情。

如果星期六NVDA在100美元，有人想把Impact Ask往105推，他未必需要一路把100、101、102、103、104全部真金白银买光。只要100到104之间的卖盘大量撤掉，新的ask集中到105以上，Impact Ask本身就可能往上走。当然，这样做有风险。

因为一旦其他人真的来扫货/fat finger，你挂出来的单子可能成交；而且你还得同时面对其他做市商重新补流动性。所以这不是说“Impact很好操纵”，更多的是提高了操纵的成本。

### **6.2 更值得问的不是Impact Notional多少，而是它和整个清算盘子的比例**

假设某张股票perp最高杠杆是10倍，Initial Margin Rate大概10%，按照Binance的公式，Impact Margin Notional大约就是2,000 USDT。

乍看之下，一个2,000美元的测量尺拿去决定一张股票perp的Index，好像有点小。

但单独看2,000这个数字其实没有太大意义。

因为你不能说：“Impact Notional只有2,000美元，所以花2,000美元就可以操纵Binance。”这不成立。想持续改变Impact Mid，你可能需要影响更大一段盘口，还要维持足够长的时间去推动EWMA，也要面对做市商和套利者不断回来补单。

真正值得算的是另外一个比例。

假设把Index有效推低1%，需要你持续承担5万美元的盘口风险。

而这个1%的变化，刚好会穿过一大片高杠杆多仓的清算区域，触发300万美元强平。

那问题就来了。

市场为了改变Oracle需要付出的资本，和Oracle改变以后能够释放出来的强制交易量，已经不是一个量级。

这时候最值得看的指标不是：

**Impact Margin Notional是多少？**

而是：

**把Index移动1%需要多少资本 ÷ 这1%会触发多少Liquidation Notional？**

或者反过来看：

**Liquidatable Open Interest / Oracle Manipulation Cost**

这个比例如果很高，系统就可能产生一种很危险的经济激励。

因为攻击者不一定非得靠操纵那一张NVDA perp本身赚钱。

他可以在一个账户影响orderbook，在另一个账户提前做空，甚至在其他venue建立受益仓位。一旦Index穿过清算区，真正帮他继续往下卖的，可能就不是他自己的钱，而是别人被强制平掉的仓位。

到这一步，Oracle安全就不只是一个技术问题，而变成了一个经济学问题：

> **让价格往某个方向走的成本，和把价格推过去以后能释放出来的被动订单，到底差多少？**

### **6.3 最大的结构风险：Oracle会看到自己制造出来的后果**

假设星期六NVDA从100开始往下跌。

最初可能只是因为出了一条坏消息，Binance场内的买家撤单，卖家往下压。Impact Mid从100掉到97，EWMA Index开始缓慢往下移动。

当Index来到98、97以后，Mark也开始跟着变化。

第一批高杠杆多仓触发清算。

这些清算仓位最终需要在Binance自己的市场里面卖掉。

卖单重新打进orderbook以后，原本已经变薄的bid继续被吃掉，Impact Bid进一步往下走，Impact Mid也跟着下降。

于是：

> **坏消息\
> → Orderbook下降\
> → Impact Mid下降\
> → Index下降\
> → Mark下降\
> → 多仓强平\
> → 强平卖单重新进入Orderbook\
> → Impact Mid继续下降。**

这时候Oracle看到的已经不只是“市场怎么看NVDA”。

它看到的里面，还混入了：

**因为Oracle自己之前下跌，而产生出来的清算订单。**

这就是一个典型的内生反馈。

最开始，Index是在观察市场； 到后面，市场的一部分变化，却是Index自己制造出来的。

于是会出现一个很有意思的问题：

假设没有这些强平单，NVDA周末市场也许只会从100跌到96。

但因为96附近刚好存在大量杠杆仓位，第一轮下跌触发清算，清算把盘口继续打到93，新的Index又开始确认93附近的价格。

最后你看到的是：

**“市场交易到了93。”**

但这个93里面，有多少是新闻带来的重新定价，有多少是风险系统自己造成的强制卖出？

到了极端行情，这两样东西很难分开。

这就是我觉得Binance这套算法最深的一层风险：

> **价格源和使用这个价格的风险系统，不再完全独立。Oracle开始吃自己的尾巴，开始引用自己的价格。**

### **6.4 EWMA能过滤假动作，也会延迟真消息**

EWMA解决的是时间问题。

一秒钟跳到108，我不相信；你在那里待五分钟，我才慢慢相信。

防操纵的时候，这很合理。但算法没办法判断新闻是真是假。

假设星期六英伟达真的突然出了一个足以让公司估值重估15%的消息。股票市场又不开，Binance perp成了少数可以马上交易NVDA风险的地方。

新的信息出来以后，市场可能很快形成：

**Bid 114，Ask 115。**

如果这个价格是真的，那么最“正确”的Index理论上应该尽快承认世界变了。

但EWMA不会。

它还是按照同样的规则慢慢往上爬。

也就是说，EWMA面对两种完全不同的情况，做的是同一件事。

有人短暂操纵价格：**慢一点相信，是优点。**

公司真的发生重大变化：**慢一点相信，就变成缺点。**

这是一套平滑Oracle永远绕不开的矛盾。

你把α调大，Index跟得快，真实消息反映得漂亮，但攻击者只需要更短时间维持异常盘口。

你把α调小，操纵成本上去了，但真正的价格变化来了以后，Index又会长时间落后。

所以不存在一个神奇的τ，可以同时做到：

**重大消息一分钟全部反映，又要求攻击者维持半小时才算数。**

这两件事在逻辑上就是冲突的，不过算法只能选一个折中点。

### **6.5 周末±3%不是熔断，只是把大行情拆成很多个3%**

Mark相对Index的±3%护栏，很容易被误读成：**“Binance周末最多只能涨跌3%。”**

实际上不是。

限制的是：**Mark相对当时Index最多偏离3%。**

Index自己还是可以继续移动。

假设开始：**Index = 100**

那么Mark上限是：**103**

过了一会，Impact Mid持续上涨，EWMA把Index推到了103。

新的Mark上限马上变成：

**106.09**

如果Index再来到106：

**Mark上限 = 109.18**

于是整个过程完全可能是：

**100 → 103 → 106 → 109 → 112**

每一个时刻，Mark都没有违反±3%。

所以这个护栏更像一根跟着主人一起走的狗绳，而不是一道固定的墙。

这也意味着，它不能真正阻止清算cascade。

它能做的是：

**不让Mark一次跑得比Index快太多。**

但如果Index自己持续移动，清算还是会一批一批发生。

甚至可能出现一种很特别的情况。

100的时候，第一批多仓爆掉。

Index慢慢来到102，第二批爆掉。

104的时候，第三批再爆。

于是原本可能在一分钟里面发生的瀑布，被平滑算法摊成十分钟、二十分钟。

看起来没有一根巨大的针。

但仓位还是一层一层被送上清算线。

换句话说，平滑机制未必消灭cascade —— 但是会让整个清算更有序一些，不至于形成踩踏，给你跑路的机会。

> 周末的踩踏从这个角度出发是可以被预测的。

### **6.6 Stock perp还有一个BTC没有的问题：周末套利者少了一条腿**

BTC perpetual如果星期六突然比BTC spot贵5%，套利者很好处理。

买BTC spot，卖BTC perp。价格一偏移，套利资金就进来。

Stock perp周末却完全不是同一个情况。

假设NVDA星期五收100，星期六Binance perpetual涨到108。

理论上最漂亮的套利是：

**Short NVDA perp + Buy NVDA cash equity。**

问题是：

**Nasdaq没开。**

真正那一条NVDA现货腿暂时不存在。

做市商当然还有办法。

可以买其他股票代币，可以用其他perp，可以用相关ETF，可以拿半导体指数、SOXL、QQQ、期权甚至相关股票去proxy hedge。

这也是为什么这篇文章里：<https://agintender.substack.com/p/bstocks-b-nasdaq-perp?r=8gs9xi> 强调股票借贷的重要性。

### **6.7 套利机会：30秒交接避免了一根断层，却制造了一条可以提前计算的斜坡**

Binance用30秒weighted blend处理两套Index的交接，这个设计比硬切聪明。

但它也有代价。

假设周末Binance自己的Index是108。

星期一external price重新出现以后，外部Index是100。

如果按照30秒线性blend，那么在外部价格不继续变化的情况下，接下来这30秒Index的大致路径其实是可以计算出来的：

开始是108。

10秒以后大约105.33。

20秒以后大约102.67。

30秒以后回到100。

这就意味着，原本：

**108 → 100**

的一根大台阶，被换成了：

**108 → 105 → 102 → 100** 的一条斜坡。

对于普通用户，这可能是一种保护。

对于套利机器人，却可能变成一段可以提前计算的repricing path。

尤其是如果市场知道某一批仓位的清算区域大概在哪里，那么这条斜坡走到什么地方，会开始对哪些仓位产生压力，也变得更容易预测。

所以套利者们，你们还在等什么？

## **6.8 Binance解决了一个外部价格问题，又换来了一个内部反馈问题**

**生产价格的人，使用这个价格的人，被这个价格清算的人，都在同一个系统里面。**

Binance把一个：

**“外部股票市场关门以后，我没有新价格。”**

的问题，变成了：

**“我可以自己生产新价格，但我要防止我的价格、杠杆和清算互相喂数据。”**

这两套方案没有哪一个是绝对正确的。

它只是把风险从一个地方搬到了另一个地方。

真正应该担心的，也不是某个人拿一笔小单就可以把NVDA从100操纵到120。

更值得研究的是极端情况下的反馈：

**NVDA 100\
→ 坏消息\
→ Orderbook 96\
→ Impact Mid下降\
→ EWMA Index 98\
→ 第一批多仓清算\
→ Liquidation重新打进Orderbook\
→ Bid继续消失\
→ Impact Mid 94\
→ Index继续下降\
→ 第二批清算。**

当这个连锁反应跑起来以后，Oracle已经不只是一个观察市场的温度计。

**相反地，温度计变成了发热器。**

![图16](assets/%E5%9B%BE16.webp)

这才是Binance休市Oracle算法最值得警惕的地方。

所以后面如果真的要用数据检验这套机制，我觉得最重要的也不是只去估EWMA到底是300秒还是500秒，而是另外算一个指标：

**某个价格区间可能触发的Liquidation Notional\
÷\
把Impact Mid和EWMA Index推进这个区间所需要的持续资本。**

这个比例越大，意味着越少的主动资本，可能释放越多的被动清算订单。

如果有一天真的出现：

**5万美元的持续盘口压力，可以把Index推过一个对应500万美元强平仓位的区间，**

那时候问题就不再是“Oracle准不准”。

而是：

> **这套Oracle有没有可能把市场原本的一次价格变化，放大成由自己的清算机制继续推动的第二轮行情。**

这也是为什么，Binance休市算法最有意思的部分，反而不在算法写完的地方。

而是在算法开始和杠杆发生关系以后。

知其然，且知其所以然

**后记**

假设你能从头看到这里，真的已经很不错了 - 基本上击败了99%的人了🤣

下一期，我们将接着讲解trade.xyz 的股市休市期间的oracle算法，以及OKX/BYBIT/GATE/MEXC的算法特殊性

你们将会知道为什么BINANCE的策略会吊打其他交易所？为什么Bitget的stock perp流动性如此之丰沛？

欲知后事如何，请听下回分解
