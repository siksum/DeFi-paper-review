# Ormer: A Manipulation-resistant and Gas-efficient Blockchain Pricing Oracle for DeFi

Dongbin Bai<sup>†</sup>, Jiannong Cao<sup>†</sup>, Fellow, IEEE, Yinfeng Cao<sup>†\*</sup>, Long Wen<sup>†‡\*\*</sup>, Milos Stojmenovic<sup>§</sup>

<sup>†</sup>The Hong Kong Polytechnic University, Hong Kong, China

<sup>‡</sup>Derivation Technology Limited, Hong Kong, China

<sup>§</sup>Singidunum University, Belgrade, Serbia

dong-bin.bai@connect.polyu.hk, {csjcao, csyfcao}@comp.polyu.edu.hk,

long.wen@derivation.info, mstojmenovic@singidunum.ac.rs

Abstract—Price feeds of cryptocurrencies are essential for Decentralized Finance (DeFi) applications to realize fundamental trading and exchanging functionalities, which are retrieved from external price data sources such as exchanges and input to onchain smart contracts in real-time. Currently, arithmetic mean based time-weighted average price (TWAP) oracles are widely used to process price feeds by averaging asset price with short time frame to achieve reliable and gas-efficient pricing. However, recent research indicates that TWAP is vulnerable to price manipulation attacks, resulting in abnormal price fluctuations and severe financial loss. Even worse, TWAP oracles usually set a relatively long time frame setting to prevent such attack. However, it would further introduce long delays and high price deviation errors from the market asset price.

To address this issue, we propose a novel on-chain gasefficient pricing algorithm (ORMER) that heuristically estimates the median of asset price within an observation window based on a piecewise-parabolic formula, while the time delay is suppressed by fusing estimations with different window sizes. Our evaluation based on multiple pairs of token swapping price feed across different chains show that ORMER reduces the mean absolute price error by 15.3% and the time delay by 49.3% compared to TWAP. For gas efficiency, regardless of the number of price observations, an encoding mechanism with constant storage requirement is employed without saving all the historical data for median estimation. Surprisingly, the lowest gas consumption of ORMER is even 15.2% less than TWAP, and the oracle querying fee would be saved up to ∼20K USD per day for DeFi participants.

Index Terms—Blockchain Data Processing, Decentralized Finance, Smart Contract, Oracle

## I. INTRODUCTION

In recent years, blockchain-based Decentralized Finance (DeFi) applications have made significant advances in reshaping traditional financial services. Empowered by on-chain smart contracts, these applications have provided a fair and open financial platform for different stakeholders [2]. For example, various novel financial applications and services are proposed, such as token swapping (exchange) [3], [4], lending [5], [6], and derivatives[7], allowing users and institutions to conduct diverse financial businesses flexibly with tokens. By 2025, there are over \$108 billion Total Value Locked (TVL) in over 3, 800 DeFi applications on various blockchains like

![](images/090b4990504dd913b13652d0ab5091e7c1c0a43e53be7788e3c62a5920ca8502.jpg)  
Fig. 1: A real-world flash loan-based price manipulation attack [1] that makes DEX price feed fluctuates abnormally. TWAP price feed achieves fair manipulation resistance to price fluctuation but fails to reflect the latest market price. ORMER reflects the market price better by following closely to the trend of DEX and CEX price feeds at the same time, while introducing fair fluctuation and quick recovery speed after the attack.

Ethereum, Binance Smart Chain, and Solana, indicating that a new global decentralized finance ecosystem is rapidly forming [8].

Among which, pricing data is essential for DeFi applications. It reflects continuous and accurate cryptocurrency/fiat money market information as important reference for on-chain applications like trading, swapping, loaning, and depositing, etc. For example, the exchange rate between ETH and USD is frequently utilized by on-chain token swapping applications for reference to trade various tokens on Ethereum.

Price Feed Processing. Due to the decentralized nature of blockchains and smart contracts (automated on-chain program), processing such pricing data needs multiple complex procedures. Specifically, in practice, such price data are originally generated from Price Data Sources, such as off-chain Centralized Exchanges (CEXs), Decentralized Exchanges (DEXs), and Automated Market Makers (AMMs), where trading and arbitraging activities happen and decide the market price [9], [10]. To this end, blockchain pricing oracle is developed as a crucial on-chain contract to process these original market price data into a correct and continuous form (feed), which is further fed to downstream DeFi applications.

At current stage, time-weighted average price (TWAP)- based pricing oracle is the most popular pricing oracle developed by Uniswap [3], which simply takes arithmetic mean of the token quantities collected from DEXs with a pre-set fixed time frame for updating price data. As only a minimal storage space in smart contract is required, TWAP demonstrates a high gas-efficiency in practical deployment. Compared with centralized solutions like Chainlink, TWAP oracle is fully on-chain and thus considered more trustworthy. As a result, TWAP oracles have been adopted as the mainstream price references (around 60% market share for DEX token pricing) for DeFi applications.

Problem. However, many recent studies have found that TWAP oracles severely suffer from manipulation attacks such as flash loan attacks and arbitrage attacks [11], [12]. Specifically, for a short time frame setting of a TWAP oracle, attackers could make the oracle output price data deviates by injecting extreme price data points through multiple malicious transactions (e.g., use an extremely low or high rate to swap tokens). Even worse, for a TWAP oracle with a long time frame for mitigation of manipulations, the time delay and price deviation error to market price would be increased because the oracle fails to follow the up-to-date price, while a higher price deviation error would create additional opportunities for arbitrage attacks by malicious parties [13], [14]. According to the top 200 costliest attacks recorded in Rekt Database, the financial losses caused by only 36 flash loan attacks have exceeded 418 million USD [15], [16], [17].

Challenges. Different from traditional financial services, on-chain pricing data can not be directly used by DeFi applications because they are (1) Discrete: price data points are usually generated and changed block-by-block (e.g., the price from on-chain DEX or AMM); (2) Deviated: price data points are only updated by public invocations from users or contracts; (3) Expensive: smart contract invocations are costly in public blockchain, making DeFi participants less willing to update the pricing data. These factors make data points scattered or out-dated, and can hardly reflect the market price correctly in time (§III).

In this paper, we propose ORMER, the first manipulationresistant and gas-efficient on-chain pricing oracle for DeFi. In particular, inspired by traditional financial stock pricing services, ORMER novelly adopts the median-based method to process on-chain price data from a market source, which is effective on resisting manipulations as the price median is hard to be affected by few extreme prices. Moreover, we overcome the expensive gas consumption issues of calculating streaming median in smart contracts by a carefully designed algorithm, which only requires a constant storage space by heuristically constructing the median estimations of a past sliding window. We also conduct extensive experiments and practical deployment to demonstrate the improved security, performance, and effectiveness in practice. To emphasize, the algorithm is a general alternative to TWAP, which can also be used in traditional financial data management services.

Contributions. The contributions of ORMER can be summarized as follows:

• ORMER: in this paper, we propose the first on-chain pricing algorithm for blockchain pricing oracle with SotA performance, providing the following properties:

– Streaming Median: ORMER could estimate the median of the last sliding window of streaming price feed in a gas-efficient manner without storing all the historical data in a fully on-chain setting (§IV-B).

– Manipulation Resistant: ORMER estimates the median of a given input price feed window, which significantly raises the cost on price manipulation attack, since more corrupted prices are needed to affect the feed (§IV-B). According to the metrics defined in §V-B, decentralized ORMER MedDS (one derivative of ORMER requires more gas for a low delay) achieves 90.0% performance of True Median and 65.7% performance of a centralized oracle regarding manipulation resistance, with 75.4% improvement to TWAP.

– Delay Reduction: with the method introduced in §IV-C, price feed constructed by ORMER MedDS could achieve significant lower time delay than other smoothing based methods. The delay is reduced by 49.3% compared to TWAP. Statistical results indicate that purely on-chain ORMER MedDS could reach only 4% more delay in time in fully decentralized manner compared to centralized offchain oracle (§V).

– Gas Efficient: ORMER MED (another derivative of ORMER requires less gas for accurate estimation of median) requires a fixed 256 bits of contract storage, while 512 bits of fixed storage is required for ORMER MedDS with delay suppression (§IV-D). Both modes consume less gas on oracle price querying compared to TWAP, which would ultimately save up to \$19 USD per querying. In real-world environment, ORMER MED could help DeFi participants save up to \$26k USD per day (§V).

• Implementation and Compatibility: we implement ORMER as an example oracle contract in Solidity language, which is deployed in real-world environment<sup>1</sup>. ORMER is compatible to scale on most of the mainstream blockchains (e.g., Ethereum Virtual Machine compatible blockchains like Ethereum, Tron, Polygon) without needing significant modifications. All the source codes and experimental artifacts are made publicly available<sup>2</sup>.

• Experimental Evaluation: we have collected 4, 130, 000 spot prices from real-world swapping pools (DEXs) from 2022-01-13 to 2023-09-13 as workloads. To the best of our knowledge, we are the first to conduct quantitative experiment on blockchain oracles. We comprehensively compare ORMER MED and ORMER MedDS with Chainlink, TWAP,

Exponential Moving Average (EMA), and True Median with practical implementations. We also firstly define several metrics for evaluating the pricing oracles. Results indicate that both modes of ORMER outperform existing widely deployed on-chain pricing oracles (§V).

## II. SYSTEM MODEL

In most DeFi applications requiring price feed for trading, exchanging, etc, there are three main components over time t as shown in Figure 2:

• A price data source (DEX, CEX, AMM, etc.) SC that actually produces a source price feed $\begin{array} { r l } { \mathcal { P } _ { s c } } & { { } = } \end{array}$ $\left\{ ( t _ { 1 } , p _ { 1 } ^ { s c } ) , . . . , ( t _ { n } , p _ { n } ^ { s c } ) \right\}$ of spot price (i.e., the latest price for successful orders) for the rate between Token B and Token A (e.g, USDT and ETH). At t, the market price is $\mathcal { P } _ { m a r k e t } = \{ ( t _ { 1 } , p _ { 1 } ^ { m a r k e t } ) , . . . , ( t _ { n } , p _ { n } ^ { m a r k e t } ) \}$

• A pricing oracle P O that takes one $\mathcal { P } _ { s c }$ as input and output oracle price feed $\mathcal { P } _ { o } = \{ ( t _ { 1 } , p _ { 1 } ^ { o } ) , . . . , ( t _ { n } , p _ { n } ^ { o } ) \}$

• A DeFi application APP that receives $\mathcal { P } _ { o }$ as reference.

## A. Price Data Source

A price data source SC can be an AMM, DEX, or CEX, which provides a source price feed $\begin{array} { r l } { \mathcal { P } _ { s c } } & { { } = } \end{array}$ $\left\{ ( t _ { 1 } , p _ { 1 } ^ { s c } ) , . . . , ( t _ { n } , p _ { n } ^ { s c } ) \right\}$ according to the successful orders. Therefore, price feeds from all price data sources in market should reflect a logical market price feed $\mathcal { P } _ { m a r k e t } ~ =$ $\{ ( t _ { 1 } , p _ { 1 } ^ { m a r k e t } ) , . . . , ( t _ { n } , p _ { n } ^ { m a r k e t } ) \}$

For example, AMM is typically implemented by a smart contract that provides token swapping service between Token A and Token B (e.g., USDT and ETH) according to the liquidity pool, and the price feed $\mathcal { P } _ { s c }$ from AMM can be accessed directly by DeFi applications. Similarly, DEXs can be composed by AMMs, and a CEX’s price data can be relayed to blockchains block-by-block, they all have identical features with general DEXs under this system model.

However, due to the limited liquidity of any single price data source, the price feed $\mathcal { P } _ { s c }$ may deviate from the logical market price feed $\mathcal { P } _ { m a r k e t }$ , especially when manipulation attacks happen.

## B. Pricing Oracle

Therefore, to provide a more stable and continuous price feed based on price data feed from price data source, Pricing Oracle is proposed and deployed on-chain between the price data source $S C$ and the DeFi application $A P P ,$

A pricing oracle P O takes source price feed $\begin{array} { r l } { \mathcal { P } _ { s c } } & { { } = } \end{array}$ $\left\{ ( t _ { 1 } , p _ { 1 } ^ { s c } ) , . . . , ( t _ { n } , p _ { n } ^ { s c } ) \right\}$ to construct a new price feed $\mathcal { P } _ { o } =$ $\left\{ ( t _ { 1 } , p _ { 1 } ^ { o } ) , . . . , ( t _ { n } , p _ { n } ^ { o } ) \right\}$ by running a pricing algorithm.

Ideally, $\mathcal { P } _ { s c }$ should have no deviation from the market price feed $\mathcal { P } _ { m a r k e t }$ if the smart contract smoothly updates in realtime. However, as the $\mathcal { P } _ { s c }$ is produced discretly block-byblock (several seconds on blockchains) instead of real-time (milliseconds in traditional exchanges), it may be significantly deviated from $\mathcal { P } _ { m a r k e t }$ across different blocks. Moreover, as PO is typically implemented by a smart contract, it is extremely costly to update price feed in real-time. Therefore, the design of pricing algorithm inside PO is the key, which should be designed to be robust (even against the manipulation attacks) and gas-efficient.

## C. DeFi Application

Therefore, for safety considerations, DeFi applications typically read price feed $\mathcal { P } _ { o }$ from PO instead of directly reading $\mathcal { P } _ { s c }$ from SC to get the latest and robust market price.

For example, loaning application is a mainstream type of DeFi application that accepts cryptocurrency as collateralization. It is important for such application to get the latest market price of the collateralized cryptocurrency for its own profit (i.e., liquidize the collateralized cryptocurrency when its value drops). As shown in Figure 2, if the pricing oracle feed is influenced by the manipulated Price Data Source, the liquidation process designed to protect the application profit will be exploited against the application owner by false triggering or not triggering [16]. Similarly, such pricing algorithm is also essential and desired by other DeFi applications including token swapping, derivatives trading, yield farming (deposit cryptocurrency to a platform with investing strategy for better revenue), etc [2].

## D. Threat Model

Price Manipulation Attack. In practice, price data sources with low liquidation are vulnerable to price manipulation attacks [15], where attacker aims to affect a DEX’s spot price by artificially determine the spot price of a token with one large volume transaction. As results, these attacks would lead to severe financial losses in real-world scenarios [11], [18], [19].

Example. As shown in Figure 2, a typical construction of flash loan-based price manipulation attack aims to cheat the downstream DeFi applications with false pricing data to make profit. Here we provide an example of flash loan based price manipulation attack, which involves manipulating the price between two DeFi applications:

• Flash Loan Provider: smart contract $A P P _ { f p }$ that offers token liquidity without any prior collateral requirement.

• Collateralization-based DeFi application: smart contract $A P P _ { C o l }$ that offers Token A loaning service by collateralizing Token B.

The attacker first borrows 10, 000 Token A from $A P P _ { f p }$ with no prior collateralized assets, which is made possible based on the atomicity property of blockchain transactions (⃝1 ). Then the attacker would use the borrowed Token A to swap for 1, 000 Token B in $A P P _ { f p } ,$ which would significantly raises Token $B ^ { * } { \mathrm { s } }$ spot price 10 $A / B$ into 100 $A / B _ { i }$ , since the saved quantity of Token A in the exchange is pushed up to an abnormally high position (⃝2 -a). Under this situation, if PO fails to smooth the abnormal surge of spot price, the oracle referencing price 9.8 A/B would deviates into 97 $A / B$ (⃝2 -b). At this moment, the attacker invokes service of borrowing Token A by collateralizing Token B in a downstream application $A P P _ { \mathrm { C o l } } ~ ( \textcircled { 3 } { - } \mathrm { { a } ) }$ . To determine the price of Token B, the application $A P P _ { \mathrm { C o l } }$ would query PO for latest oracle referencing price, resulting in being cheated by the attacker that the price of Token B has dramatically increased (⃝3 -b). Thus, $A P P _ { \mathrm { C o l } }$ lends 12, 000 Token A to the attacker, as it is cheated to have an over-collateralization of Token B (⃝4 ). In the end, the attacker pays back borrowed 10, 000 Token A to $A P P _ { f p }$ and default the loan from $A P P _ { \mathrm { C o l } } \left( \mathfrak { H } \mathrm { - a } \right)$ . As a result, the attacker earns 2, 000 Token A (⃝5 -b).

![](images/4df1f1d80b462e9a65e4ef8b2b739cd5a4d6dd5fc6207c21b49d8d62a0684cf3.jpg)  
Fig. 2: A typical flash loan based price manipulation attack. An attacker manipulates the Price Data Source with flash loan (⃝1 ⃝2 ); A downstream application (i.e., Collateralizationbased DeFi application) takes manipulated Oracle Price Feed (constructed by a Pricing Oracle) to determine financial service fee (⃝3 ⃝4 ); The attacker profits from untruthful service fee: here, undercollateralized loan (⃝5 ).

In this example, the Collateralization-based DeFi application $A P P _ { \mathrm { C o l } }$ suffers from an under-collateralization situation with financial loss, which is caused by a pricing oracle that fails to smooth the abnormal surge of spot price (⃝5 -c) [15].

## III. MOTIVATIONS AND OBJECTIVES

To counter price manipulation and abnormal price fluctuations, TWAP oracles are widely adopted in practice, which is designed to smooth the price feed by averaging the incoming price data over a fixed time window. However, many recent studies have found that TWAP oracles actually failed [11], [12]. Specially, as illustrated in Figure 2, for a short time frame setting of TWAP oracle, it is ineffective to filter out the extreme price data points injected by attackers. For a TWAP oracle with a long time frame, the time delay and price deviation error would be also increased because the oracle fails to follow the up-to-date delay, while a higher price deviation error would create additional opportunities for arbitrage attacks by malicious parties [13], [14].

Other price feed smoothing methods, such as median-based algorithms that are widely used in traditional trading systems, are proved to be effective for smoothing the price feed and filtering out the extreme price data points. However, they require a large amount of storage space to store the historical price data, which is impractical for on-chain implementation due to the high gas consumption for storage. For example, querying a price data point from a median-based oracle would require around three times more gas (∼310k gas) than a TWAP oracle (∼100k gas).

Therefore, we have following objectives when designing ORMER:

• Robust Deviation Resistance: The pricing oracle should be robust against manipulation attacks, which means the deviation between the pricing oracle feed and the market price feed should be minimized.

• Low Delay: The pricing oracle should be able to provide the latest market price in real-time.

• Low On-chain Cost: The pricing oracle should be gasefficient, which means the gas consumption of the pricing oracle should be minimized.

## A. Robust Deviation Resistance

The primary objective of pricing oracle is to provide a price feed $\mathcal { P } _ { o }$ according to delayed $\mathcal { P } _ { s c }$ that has minimal deviation from $\mathcal { P } _ { m a r k e t }$ in a fully on-chain manner. Formally, given the time $t \in \tau$ , true external price feed $\mathcal { P } _ { m a r k e t } =$ $\bar { \{ ( t , p _ { t } ^ { m a r k e t } ) | t \in \mathcal { T } \} }$ , exchange price feed $\mathcal { P } _ { s c } = \{ ( t , p _ { t } ^ { s c } ) | t \in$ T}, the deviation with observation window L can be defined as:

$$
\mathcal{D}(\mathcal{P}_{sc},L):= \min_{\substack{t\in \mathcal{T},\\ n_{0} = |\mathcal{T}| - L}}\sum_{n_{0}}^{\mathcal{N}}\sum_{t_{0}}^{\mathcal{T}_{L}}Distance(Oracle(p_{t}^{sc})^{2} - p_{t}^{market2})\tag{1}
$$

In this paper, we assume that only upper bound of the manipulated prices is known. Since the oracle contract do not have direct access to $\mathcal { P } _ { m a r k e t }$ , it could be impossible for the contract to construct $\mathcal { P } _ { o }$ when all the data of $\mathcal { P } _ { s c }$ are manipulated, so an upper bound $\beta$ is assumed.

Assumption 1: An adversary $e _ { t }$ at time t arbitrarily manipulates at most $\beta$ number of $p _ { t ^ { \prime } }$ in observation window with size L of $\mathcal { P } _ { e }$

Given a back-testing evaluation function V, we say V is $( \beta , \epsilon )$ -secure if it satisfies:

$$
\mathcal {V} (\mathcal {P} _ {o}, \mathcal {P} _ {\text {market}}, \beta , L) \leq \epsilon\tag{2}
$$

## B. Low Delay

While robust estimations can effectively resist manipulation attack, they may fail to accurately reflect the latest market dynamics of the external centralized exchanges and the market price. The underlying principle behind a pricing oracle is to sample a subset of historical data for generating a statistical result as the current price. This approach inherently hinders the timeliness of the output. Specifically, a naive way to convert the discrete DEX price feed into a continuous and more dense and stable feed is by employing a moving average filter, which is the underlying principle used by TWAP. Oracle price calculated according to such filter would falls behind DEX in time, varies by the choice of averaging window size. In order to keep pace with delay, we empower ORMER with prediction functionality that fuses two parallel updated estimations as the final delay suppressed output.

![](images/5ecbf16fb2a05b7fa397189abd35d297633651b7981afda512e340f8b2383de1.jpg)  
Fig. 3: Overview of ORMER system model workflow with three main components: Market Price Source, Pricing Oracle, and DeFi Protocol. Instead of relying potentially vulnerable Spot Price Feed, DeFi Protocol would use Pricing Oracle Feed that is constructed upon weighted historical data with delay suppression techniques.

## C. Low On-chain Cost

The primary challenge ORMER faces is to design a purely on-chain pricing oracle. Although there are many robust streaming data estimation methods studied in data science (e.g., t-digest, Quantile Regression, Gaussian Processes, etc.), the implementation of such methods are gas-consuming in smart contract due to their complex matrix computations, sophisticated data structures, or large storage space requirements for historical data. Since the pricing oracle is updated upon contract invocations, directly migrating such methods to blockchain is impractical. For example, about \$6 USD for accessing a 256 bits value in Ethereum (estimated according to: 3, 000 USD/ETH, 100 Gas Price). Thus, it is desired to have a pricing oracle with less smart contract storage requirement and computation steps. Formally, given the time $t \in \tau$ , true external price feed $\mathcal { P } _ { m a r k e t } = \{ ( t , p _ { t } ^ { m a r k e t } ) | t \in \mathcal { T } \}$ , exchange price feed $\mathcal { P } _ { s c } = \{ ( t , p _ { t } ^ { s c } ) | t \in \mathcal { T } \}$ , the gas consumption with observation window L can be defined as:

$$
\mathcal {G} (\mathcal {P} _ {s c}, L) := \underset {n _ {0} = | \mathcal {T} | - L} {\min} \sum_ {n _ {0}} ^ {\mathcal {N}} \operatorname{Gas} (\operatorname{Oracle} (\mathcal {P} _ {s c} | t \in [ n, n + L - 1 ]))\left. \right)\tag{3}
$$

## D. Decentralization

Centralized solutions such as Chainlink can provide a lowdeviation price feed with low latency and at a low cost by using high-end servers to periodically fetch data from several of the largest CEXs. However, for DeFi applications built on such centralized solutions, a connection failure with the servers would result in business failure, thereby exposing severe attack surfaces to malicious parties. Furthermore, it is hard to ensure or verify whether the servers have been corrupted by external attackers or internal operators. Therefore, they are not utilized by the most of DeFi applications in practice [10], [16], [20].

## IV. ORMER PRICING ORACLE

## A. Overview

In order to provide $\mathcal { P } _ { o }$ with security insurance while remaining gas consumption comparable to State-of-the-Art TWAP (one cold slot read per update), we propose a novel streaming median estimation algorithm: ORMER. The estimation feed $\mathcal { P } _ { o }$ is produced dynamically, which could follow the trend of input source with less time delay as the spot price $\mathcal { P } _ { e }$ changes, and the algorithm has a constant storage requirement regardless of the number of price observations. The goals are achieved by three main components:

• Median Estimator (§IV-B): we design a sliding window median estimation algorithm that accepts data feed input in a streaming manner. The algorithm does not require to store all the historical data for median output by using a piecewise-parabolic formula and an auxiliary sliding storage for median estimation.

• Delay Suppression (§IV-C): we propose a novel price feed delay suppression method to better reflect market price is achieved by utilizing the projection angle of the markers.

• Slot Encoding (§IV-D): we design a novel slot encoding template for smart contract implementation that would reduce the gas consumption.

## B. Median Estimator

Inspired by [21], [22], we have developed a novel sliding window estimator that could deal with the streaming input data with only five markers: the minimum, the 25th percentile, the 50th percentile, the 75th percentile and the maximum price in the observation window. A marker is a vertical stick that contains $( n _ { i } , h _ { i } )$ , where $n _ { i }$ is the horizontal position of stick and $h _ { i }$ is the vertical height of stick. With five sticks sorted from the lowest height to the highest height, stick with index 2 (start from 0) is regarded as the estimation of median based on the recorded states. When a new $\mathbf { \nabla } _ { p _ { t } ^ { e } }$ observation comes in, it is compared with stored markers. As a result, all the markers with greater heights than observation are moved one position to the right (denoted as $n _ { i } + 1 )$ . After n observations, the desired position n<sup>′</sup> of markers should be: $n _ { 0 } ^ { \prime } = 0 , \ n _ { 1 } ^ { \prime } =$ $\begin{array} { r } { \frac { n } { 4 } , \ n _ { 2 } ^ { \prime } = \frac { n } { 2 } , \ n _ { 3 } ^ { \prime } = \frac { 3 n } { 4 } } \end{array}$ , and $n _ { 4 } ^ { \prime } = n$

```txt
Marker Position (n) vs. Marker Height (h)
(n_{i-1}, h_{i-1}) (n_{i-1}, h_{i-1})
(n_i, h_i) (n_i, h_i')
(n'_{i}, h'_{i})
(n_{i+1}, h_{i+1}) (n_{i+1}, h_{i+1})
```  
Fig. 4: Marker update process of $n _ { i } ^ { \prime }$ with parabolic formula.

If the moved marker position $n _ { i }$ deviates its ideal position $n _ { i } ^ { \prime }$ more than one, it means the distribution of observation may have shifted from the states stored in smart contract. To follow up the current distribution, both the position and height should be adjusted. For the position, we would prefer to move it either one to the right (+1) or one to the left (−1) determined by the sign of $d _ { i } ~ = ~ n _ { i } ^ { \prime } - n _ { i }$ . For the height adjustment, as shown in Figure4, markers with position $[ n _ { 0 } , n _ { 1 } , n _ { 2 } ] , \ [ n _ { 1 } , n _ { 2 } , n _ { 3 } ]$ , and $[ n _ { 2 } , n _ { 3 } , n _ { 4 } ]$ would be updated sequentially when a new observation arrives according to parabola with form $h _ { i } = a n _ { i } ^ { 2 } + b n _ { i } + c .$ . It is necessary to mention that $a , b ,$ and c are coefficients determined by three position-height pairs: $( n _ { i - 1 } , h _ { i - 1 } ) , ( n _ { i } , h _ { i } )$ , and $( n _ { i + 1 } , h _ { i + 1 } )$ where $i \in \{ 1 , 2 , 3 \}$ . According to the position index $i - 1 , i ,$ and $i + 1 ,$ given $n _ { i } ^ { \prime } = n _ { i } + d _ { i } ,$ it is straightforward to get $h ^ { \prime } { } _ { i }$ with ${ h ^ { \prime } } _ { i } = a { n ^ { \prime } } _ { i } ^ { 2 } + b { n ^ { \prime } } _ { i } + c \mathrm { : }$ $\begin{array} { l } { { h _ { i } ^ { \prime } = a ( n _ { i } + d _ { i } ) ^ { 2 } + b ( n _ { i } + d _ { i } ) + c } } \\ { { \quad = h _ { i } + \displaystyle \frac { d _ { i } } { n _ { i + 1 } - n _ { i - 1 } } } } \\ { { \quad \quad \cdot \left[ ( n _ { i } - n _ { i - 1 } + d _ { i } ) \displaystyle \frac { h _ { i + 1 } - h _ { i } } { n _ { i + 1 } - n _ { i } } + ( n _ { i + 1 } - n _ { i } - d _ { i } ) \displaystyle \frac { h _ { i } - h _ { i - 1 } } { n _ { i } - n _ { i - 1 } } \right] } } \end{array}$ (4)

For the algorithm to work correctly, the heights of markers should remain in an ascending order $( h _ { i + 1 } ~ \geq ~ h _ { i } , ~ i ~ \in$ {0, 1, 2, 3}). So if an unlawful estimation $h _ { i } ^ { \prime }$ that locates outside desired range $\left( h _ { i - 1 } ~ < ~ h _ { i } ^ { \prime } ~ < ~ h _ { i + 1 } \right)$ , the result is ignored and $h _ { i } ^ { \prime }$ is recalculated according to linear formula where $d _ { i } = \mathrm { s i g n } ( n _ { i } ^ { \prime } - n _ { i } )$

$$
h _ {i} ^ {\prime} = h _ {i} + d _ {i} \frac {h _ {i + d _ {i}} - h _ {i}}{n _ {i + d _ {i}} - n _ {i}}, \quad d _ {i} \in \{1, - 1 \}\tag{5}
$$

ORMER-Median. With the marker updating process described above, we are able to construct ORMER-Median that constantly takes input $\mathcal { P } _ { e }$ from time 0 to t and estimates the median of all the received data as $p _ { 0 , i } ^ { o }$ . However, ORMER-Median is not capable of estimating the median of given window size in an online streaming data setting (e.g., window time from $t _ { 1 }$ to $t _ { 2 } ,$ expect $p _ { t _ { 1 } , t _ { 2 } } ^ { o } )$

<div class="mineru-algorithm" style="white-space: pre-wrap; font-family:monospace;">
Algorithm 1: ORMER MED
Input : External price feed
    $\mathcal{P}_e = \{(t_1, p_1^e), ..., (t_n, p_n^e)\}$
Output: Oracle price feed $\mathcal{P}_o = \{(t_1, p_1^o), ..., (t_n, p_n^o)\}$
count ← 0; last estimation ← 0;
$n[5] \leftarrow [1, 2, 3, 4, 5] \ h[5] \leftarrow [0, 0, 0, 0, 0]$;
$dn[5] = [0, 0.25, 0.5, 0.75, 1.0]$
while $\mathcal{P}_e \neq \phi$ do
    $p_t \leftarrow \text{latest } p_t^e$ at time $t$; count ← count + 1;
    if $count == Observation Window Size$ then
        Update last window estimation, initialize markers;
    if $count &lt; 6$ then
        Latest non-initiated marker height ← $p_t$;
        if $count == 5$ then
            Latest non-initiated marker height ← $p_t$;
            Sort ascending marker heights $h[5]$;
        else
            continue;
    // Find cell $k$ that $h_k \leq p_j \leq h_{k+1}$ and adjust $h_0$ and $h_4$ if selected
    switch $p_t$ do
        case $p_t &lt; h_0$ do $h_0 \leftarrow p_t$; $k \leftarrow 0$;
        case $h_0 \leq p_t \leq h_1$ do $k \leftarrow 0$;
        case $h_1 \leq p_t \leq h_2$ do $k \leftarrow 1$;
        case $h_2 \leq p_t \leq h_3$ do $k \leftarrow 2$;
        case $h_3 \leq p_t \leq h_4$ do $k \leftarrow 3$;
        case $h_4 \leq p_t$ do $h_4 \leftarrow p_t$; $k \leftarrow 3$;
    $n_i \leftarrow n_i + 1 \quad i = k, ..., 4; n'_i \leftarrow count \cdot dn[i] \quad i = 0, ..., 4;$
    // Marker height adjustment
    for $i = 1; i &lt; 4; i = i + 1$ do
        $d_i \leftarrow n'_i - n_i;$
        Determine sign of $d_i$, position of $n_i$ would either move +1 or -1;
        Try $h'_i \leftarrow h_i$ from parabolic formula;
        if not $h_{i-1} &lt; h'_i &lt; h_{i+1}$ then
            $h_i \leftarrow h'_i;$
        else
            $h'_i \leftarrow h_i$ from linear formula;
        $n_i \leftarrow n_i + d_i;$ $p_t^o \leftarrow Estimates according to h[2] and last window estimation;$
</div>

ORMER-SlidingWindow. To resolve the flaw, we need to add an additional storage requirement for recording the median estimation result of the last window $E _ { \mathrm { l a s t } }$ . ORMER-SlidingWindow is designed to receive and deal with streaming data using sliding window. First, an ORMER-Median is initialized to receive coming $\mathbf { \nabla } _ { p _ { t } ^ { e } }$ . Given the manually defined window size $L ,$ once the number of L is fulfilled, $E _ { \mathrm { l a s t } }$ is estimated as $h _ { 2 } ^ { \mathrm { l a s t } }$ ORMER-Median according to historical data collected in last window. Then ORMER-Median is reset to initial state. For each subsequent ORMER-SlidingWindow estimation $E _ { t }$ it could be approximated with weighted sum:

![](images/3eef189ef9aa3b9d2e355d647f93340970cca03ddd1c10e6c7f990ae3132438d.jpg)

![](images/490075b39ae38b57473f05b235e4ef83085df892e086b848f18cfc8754ce7bba.jpg)  
(a) Estimate $\hat { p }$ when $p _ { T / 2 }$ marker is longer than $p _ { T }$ marker.  
(b) Estimate $\hat { p }$ when $p _ { T }$ marker is longer than $p _ { T / 2 }$ marker.  
Fig. 5: Estimate non-delayed data $\hat { p }$ with $p _ { T / 2 }$ and $p _ { T }$ .

$$
E _ {t} = \frac {\left(L - c _ {t}\right) \cdot E _ {\text {last}} + c _ {t} \cdot E _ {\text {current}}}{L}\tag{6}
$$

where $c _ { t }$ is the number of observation received of current window, and $E _ { c u r r e n t }$ is given by $h _ { 2 } ^ { \mathrm { c u r r e n t } }$ . In the first window, the $E _ { t }$ would be simply given by $h _ { 2 }$ of ORMER-Median. ORMER MED is the combination of ORMER-Median and ORMER-SlidingWindow, which use $E _ { t }$ as $p _ { t } ^ { o }$ , and the pseudocode of our proposed algorithm is summarized in Algorithm 1.

## C. Delay Suppression

It is unavoidable estimating current time data point based on the historical observations. In this paper, we propose to fuse additional information from a smaller observation window $L = T / 2$ besides updating ORMER MED according to manually settled $L = T$ . Intuitively, estimation of $p _ { T / 2 }$ would provide more up-to-date information since it is constructed based on historical data ranging from $t - ( T / 2 )$ to t compared to $t - T$ to t. Simply averaging these two information with $\bar { p } = ( p _ { T / 2 } + p _ { T } ) / 2$ would let estimated $\hat { p }$ always fall between $p _ { T / 2 }$ and p . As shown in Figure 5, in order to better estimate non-delayed data $\hat { p } ,$ we rotate $p _ { T }$ marker to x-axis, forming a new projection line $\tilde { p _ { T } } p _ { T / 2 }$ with slope tan $\theta = p _ { T / 2 } / p _ { T }$ . For estimation of ${ \hat { p } } ,$ we assume if $\bar { p }$ marker is rotated to x-axis, it would share the same slope θ. Thus, estimation of $\hat { p } _ { t }$ at time t is given by:

$$
\hat {p _ {t}} = \frac {p _ {T / 2} + p _ {T}}{2} \cdot \frac {p _ {T / 2}}{p _ {T}}\tag{7}
$$

As demonstrated in Figure 5a and Figure 5b, pˆ would aggressively follow the trend (either increase or decrease) of $p _ { T / 2 }$ while fusing the more smoothed $p _ { T }$ to ensure the property of manipulation resistance. With the design of Eq.

<div class="mineru-algorithm" style="white-space: pre-wrap; font-family:monospace;">
Algorithm 2: ORMER MedDS
Input : Observation window $L$,
Incoming data point $p_t$
Output: Ormer estimation $\hat{p}_t^o$
if not Initialized then
    $O_{\text{full}} \leftarrow new \text{ORMERMED}(T)$;
    $O_{\text{half}} \leftarrow new \text{ORMERMED}(\lfloor T/2 \rfloor)$;
$O_{\text{full}}.\text{Update}(p_t)$;
$O_{\text{half}}.\text{Update}(p_t)$;
$\hat{p}_{\text{full}} \leftarrow O_{\text{full}}.\text{getMedian()}$;
$\hat{p}_{\text{half}} \leftarrow O_{\text{half}}.\text{getMedian()}$;
$\hat{p}_t^o \leftarrow (\hat{p}_{\text{half}}/\hat{p}_{\text{full}}) \cdot (\hat{p}_{\text{half}} + \hat{p}_{\text{full}})/2$;
</div>

TABLE I: Bits allocation details for one slot.

<table><tr><td>Category</td><td>Information</td><td>Signed</td><td>Bits Allocated</td></tr><tr><td rowspan="3">Window Info</td><td>Window Size</td><td>✗</td><td>16</td></tr><tr><td>Observation Count</td><td>✗</td><td>16</td></tr><tr><td>Last Median Estimation</td><td>✓</td><td>24</td></tr><tr><td rowspan="2">Price Marker</td><td>Marker Positions</td><td>✗</td><td> $16 \times 5$ </td></tr><tr><td>Marker Heights</td><td>✓</td><td> $24 \times 5$ </td></tr></table>

7, $\mathcal { D } ( \mathcal { P } _ { s c } , L )$ in Eq. 1 is fulfilled. Delay suppressed ORMER MedDS is given as Algorithm 2.

## D. Slot Encoding

Slot is the minimum storage unit of smart contract. In Ethereum, one slot has 256 bits of space. As default, one variable would take one whole slot to be stored in smart contract, so encoding bits inside one slot is important for gas saving. Tick introduced in [3] could encode a $p _ { t }$ as a signed 24-bits number with reasonable precision:

$$
\tau = \log_{1.0001}p_t\tag{8}
$$

According to equation 8, the price movements of one tick can still be detected as a precision of 1 basis point (0.01%), which would suit most of financial applications. With tick, we are able to store the five marker height information into 120 bits. Another 24 bits is allocated for storing the last estimation tick. Based on remaining bits in a slot, five marker positions, observation windows size, and observation count are limited as unsigned 16-bits integers, which are 65, 535 in maximum in digit. With slot encoding and assistance of bit computation tricks, $\mathcal { G } ( \mathcal { P } _ { s c } , L )$ in Eq. 3 is fulfilled. Additionally, for ORMER MedDS, there are two slots need to be encoded as illustrated in Algorithm 2.

Detailed bits allocation for EVM compatible smart contract slot is shown in Table I. It is worth mentioning that in EVM blockchains, writing pure zero on blockchain slot would cost extra gas fee. Thus, for state initialization in ORMER-SlidingWindow, stored values should be carefully manipulated in the smart contract slot. Frequently used values are also hard coded as constants in the contract, so the gas would only be consumed once during the contract publish process.

## A. Experiment Setup

Dataset. We have collected 4, 130, 000 spot prices from real-world swapping pools (DEXs) from 2022-01-13 to 2023- 09-13 [23], and the original price feed is sampled according to Poisson distribution corresponds to random trading activities. 839, 728 of real-world spot prices of DEX are obtained after sampling. Besides, we have also collected 42, 082, 934 (kline in seconds) DEXs’ corresponding market prices ranging from 2022-01-01 to 2023-08-31 from Binance (CEX), and 134, 669 off-chain oracle prices ranging from 2020-01-15 to 2023-07-27 from Chainlink (off-chain pricing oracle) [24]. In our evaluations, we assume the CEX price feed (e.g., from Binance) reflects accurately about market price feed $\mathcal { P } _ { \mathrm { t r u e } }$

Baselines and Comparisons. There are two baselines selected respectively for quantifying the metrics and comparing the performances:

• CEX Price Feed: it is selected as the baseline for calculating the metrics in §V-B. The improved metrics indicate that the oracle price feed will be kept very close to the market price, so attackers need to spend more funds to deviate the price. In other words, there is less opportunity to initiate a successful attack on a price feed with low delay and deviation from market.

• True Median: it is selected as the baseline for comparing the performances of pricing oracles.

We consider four (Chainlink, TWAP, EMA, and True Median) existing pricing oracles and implement three of them (TWAP, EMA, and True Median) as there are no available implementations can be tested<sup>3</sup>. We record and compare their performances by feeding the above dataset in a streaming way.

Implementations. ORMER MED and ORMER MedDS are implemented as one ORMER Contract with Solidity language that is Ethereum Virtual Machine (EVM) compatible for public blockchains such as Ethereum, Polygon, and Avalanche. In order to further reduce gas consumption, considering processing ticks on-chain would involves complex exponent computation, our contract implementation employs the tick library developed by Uniswap V3 with magic number tricks that significantly reduce on-chain tick-related computations [3]. Moreover, since float number is not originally supported on EVM compatible blockchain, a gas efficient 64x64-bits fixed point number computing framework is used [25].

For simplicity on blockchain wallet management, besides evaluating on a public blockchain (Ethereum Sepolia), all the implemented pricing oracles are also evaluated on locally deployed EVM compatible blockchain network [26]. The blockchain full nodes are run on a workstation (Intel i9- 13900K CPU, 96GB Memory) and several Nvidia Jetson Orins in a local network.

## B. Metrics

According to our knowledge, metrics evaluating the performance of a blockchain pricing oracle is missing in the literature. In this paper, we evaluate the performance of price feed constructed by blockchain pricing oracles from three aspects: deviation resistance, delay and on-chain cost. Among which, evaluation on deviation resistance and delay corresponds to $\mathcal { D } ( \mathcal { P } _ { s c } , L )$ described in Eq. 1 (i.e., manipulation resistance), while evaluation on on-chain cost corresponds to $\mathcal { G } ( \mathcal { P } _ { s c } , L )$ described in Eq. 3 (i.e., gas efficiency).

Based on these metrics, we further calculate three scores: Stationary Score, Delay Score, and Gas Score, which use True Median as the baseline to eventually compare their effectiveness a unified way. True Median is sleeted as it is the most promising manipulation-resistant solution besides ORMER. In the end, weighted average based Resistance Efficiency Scores, that jointly consider three scores, are first proposed for overall evaluation for on-chain pricing oracles. The calculation of Resistance Efficiency Scores can be used in future researches, filling the absence of a comparison metric.

## Deviation Resistance.

Besides the commonly used sub-metrics like MAE, MSE, we also consider Tweedie deviance error. Tweedie deviance error is a regression result evaluation metric that elicits predicted expectation values of regression targets. We employed special cases of Tweedie deviance error formula as our evaluation metrics. The cases of listed $T D _ { p } ( y , \hat { y } )$ are equivalent to Mean Square Error (MSE), Mean Poisson Deviance, and Mean Gamma Deviance respectively, which would give us comprehensive intuitions regarding the correctness of the constructed feed with potential flexible variance structure.

Particularly, if $\hat { y } _ { i }$ from price feed $\hat { y }$ is the predicted value of the i-th sample in observation window with number of n samples, and $y _ { i }$ from price feed y is the corresponding baseline value, then the Tweedie deviance error formula $T D ( y , \hat { y } )$ is given by [27]:

$$
T D _ {p} (y, \hat {y}) = \frac {1}{n} \sum_ {i = 0} ^ {n - 1} \left\{ \begin{array}{l l} (y _ {i} - \hat {y} _ {i}) ^ {2}, & (p = 0, \text {Normal}) \\ 2 (y _ {i} \log (y _ {i} / \hat {y} _ {i}) + \hat {y} _ {i} - y _ {i}), & (p = 1, \text {Poisson}) \\ 2 (\log (\hat {y} _ {i} / y _ {i}) + y _ {i} / \hat {y} _ {i} - 1), & (p = 2, \text {Gamma}) \end{array} \right.\tag{9}
$$

With this, we further define Stationary Score $\mathrm { S c r } _ { X } ^ { \mathrm { S t } }$ to comprehensively reflects the resistance of a price feed by considering three cases of Tweedie deviance at the same time:

$$
3 (\frac {1}{\mathrm{Scr} _ {X} ^ {\mathrm{St}}} - 1) = \sum_ {p = 0} ^ {2} [ T D _ {p} (\mathrm{CEX}, X) - T D _ {p} (\mathrm{CEX}, \mathrm{MED}) ] ^ {2}\tag{10}
$$

where $X$ represents the price feed under evaluation, CEX refers to ${ \mathcal { P } } _ { \mathrm { t r u e } } ,$ and MED refers to price feed provided by True Median. $\mathrm { S c r } _ { X } ^ { \mathrm { S t } }$ reflects how similar the X performs like True Median, and a higher score indicates a more stationary price feed. There are overlaps among the regression metrics we have evaluated, so only three cases of Tweedie deviance are selected as the score definition, since they could cover the most concerned aspects of the regression evaluation. Notably, a low $\mathrm { S c r } _ { X } ^ { \mathrm { S t } }$ does not mean the price feed is vulnerable in practice (e.g., Chainlink). It need to be further balanced with the delay metrics to represent our manipulation-resistant goal.

TABLE II: Evaluation results on different price feeds with window size $L = 2 5$ blocks compared to CEX price feed.

<table><tr><td rowspan="2">Category</td><td colspan="9">Deviation Resistance</td><td colspan="3">Market Delay</td><td colspan="2">On-chain Cost</td><td rowspan="2">Overall Resistance Efficiency Score</td></tr><tr><td>Price Feed</td><td>MAE</td><td>MSE</td><td>MedAE</td><td>MaxErr</td><td>TDP</td><td>TDG</td><td>MAPE</td><td>Stationary Score</td><td>Delay (Window)</td><td>Delay (All)</td><td>Delay Score</td><td>Gas Consumption</td><td>Gas Score</td></tr><tr><td>Off-chain Oracle (Centralized)</td><td>Chainlink</td><td>3.342</td><td>24.605</td><td>2.276</td><td>135.589</td><td>1.166</td><td>0.006</td><td>0.178</td><td>0.003</td><td>313.2</td><td>249</td><td>2.736</td><td>N/A</td><td>N/A</td><td>N/A</td></tr><tr><td rowspan="5">On-chain Oracle (Decentralized)</td><td>TWAP</td><td>4.651</td><td>50.484</td><td>3.122</td><td>202.341</td><td>2.540</td><td>0.014</td><td>0.253</td><td>0.109</td><td>627.8</td><td>1,049</td><td>0.917</td><td>199,474</td><td>1.794</td><td>1.106 ▷</td></tr><tr><td>EMA</td><td>4.255</td><td>40.545</td><td>2.948</td><td>184.079</td><td>2.028</td><td>0.011</td><td>0.231</td><td>0.013</td><td>404.7</td><td>830</td><td>1.246</td><td>213,981</td><td>1.673</td><td>1.170 ▷</td></tr><tr><td>True Median</td><td>4.735</td><td>55.437</td><td>3.091</td><td>204.251</td><td>2.798</td><td>0.016</td><td>0.257</td><td>1.000</td><td>555.0</td><td>983</td><td>1.000</td><td>357,909</td><td>1.000</td><td>1.000 ▷</td></tr><tr><td>ORMER MED</td><td>4.867</td><td>57.334</td><td>3.192</td><td>225.891</td><td>2.883</td><td>0.016</td><td>0.264</td><td>0.454</td><td>555.2</td><td>1,162</td><td>0.896</td><td>169,062</td><td>2.117</td><td>1.296 ▷</td></tr><tr><td>ORMER MedDS</td><td>3.940</td><td>33.412</td><td>2.801</td><td>168.874</td><td>1.677</td><td>0.009</td><td>0.214</td><td>0.006</td><td>325.7</td><td>532</td><td>1.793</td><td>284,317</td><td>1.259</td><td>1.222 ▷</td></tr></table>

Note: All the metrics with white background are regarded as better if they are closed to 0. All the scores with gray background are regarded as better if it is higher. MAE: Mean Absolute Error, MSE: Mean Squared Error, MedAE: Median Absolute Error. MaxErr: Max Error TDP: Tweedie Deviance Error with power of 1 (Poisson distribution). TDG: Tweedie Deviance Error with power of 2 (Gamma distribution). MAPE: Mean Absolute Percentage Error. The unit of MAPE is percentage, the unit of TDP is in 10<sup>−2</sup> USD (\$), the unit of TDG is in 10<sup>−3</sup> USD (\$), the unit of Delays are second (s), the unit of Gas Consumption is in GWei, Scores (Stationary, Delay, Gas, Resistance Efficiency) do not have units, and all the other metrics are in USD (\$). Calculation of Stationary Score discards the units of TDP and TDG.

Delay. We use cross correlation to evaluate the time delay in our experiments. Time delay of price feed Y comparing to price feed X is measured according to:

$$
\text {Delay} (X, Y, \mathcal {T}) = \underset {t \in \mathcal {T}} {\max} \frac {\mathbb {E} [ (X - \mu_ {X}) (Z - \mu_ {Z}) ]}{\sigma_ {X} \sigma_ {Z}}\tag{11}
$$

where $\mu$ and σ are corresponding mean and variance to price feeds, $Z ~ = ~ \{ ( t _ { Z } , p _ { Z } ) | t _ { Z } ~ \in ~ \{ t _ { Y 1 } - t , \ldots , t _ { Y n } - t \} , p _ { Z } ~ \in$ $\{ p _ { Y _ { 1 } } , \dotsc , p _ { Y _ { n } } \} \}$ . Price feed X is fixed to CEX price feed in our experiment setup on overall delay, while $\tau$ is capped with 1800s.

We also define the Delay Score $\mathrm { S c r } _ { X } ^ { \mathrm { D e } }$ to represent how good the price feed reflects the delay more intuitively:

$$
\operatorname{Scr} _ {X} ^ {\mathrm{De}} = \frac {\text {Delay} (\mathrm{CEX} , \mathrm{MED} , \mathcal {T} _ {\text {all}}) + \text {Delay} (\mathrm{CEX} , \mathrm{MED} , \mathcal {T} _ {\text {win}})}{\text {Delay} (\mathrm{CEX} , X , \mathcal {T} _ {\text {all}}) + \text {Delay} (\mathrm{CEX} , X , \mathcal {T} _ {\text {win}})} \tag {12}\tag{12}
$$

On-chain Cost. The on-chain cost is given by the gas consumption of an on-chain smart contract invocation, which is measured by recording the blockchain transaction commit. To invoke smart contract functions through blockchain transactions, the DeFi participants need to pay the fee for code executions (i.e., Gas Fee). The fee paid to the block builder can be directly obtained from the receipts of transactions, and can be further calculated as gas scores. We define the Gas Score $\mathrm { S c r } _ { X } ^ { \mathrm { G a s } }$ by comparing to True Median that needs to record full historical data (i.e., high gas consumption):

$$
\mathrm{Scr} _ {X} ^ {\mathrm{Gas}} = \frac {\mathrm{Gas} _ {\mathrm{MED}}}{\mathrm{Gas} _ {X}}\tag{13}
$$

Overall Resistance Efficiency. In order to evaluate ORMER Contract by jointly considering deviation resistance, delay, and on-chain cost, we define Resistance Efficiency Score by taking weighted average of the Stationary Score, Delay Score, and Gas Score:

$$
\mathrm{Scr} _ {X} ^ {\mathrm{RE}} = (\omega_ {0} \mathrm{Scr} _ {X} ^ {\mathrm{St}} + \omega_ {1} \mathrm{Scr} _ {X} ^ {\mathrm{De}} + \omega_ {2} \mathrm{Scr} _ {X} ^ {\mathrm{Gas}}) / \sum_ {i = 0} ^ {2} \omega_ {i}\tag{14}
$$

where weights vector $[ \omega _ { 0 } , \omega _ { 1 } , \omega _ { 2 } ]$ is given by [1, 2, 2] for paying more attention to the challenges (i.e., delay and onchain cost) we mainly focus on in this work.

## C. Effectiveness on Manipulation Resistance

The manipulation resistances of price feeds constructed by existing oracles and ORMER are evaluated by jointly considering three aspects: regression metrics comparing to CEX, stationary scores, and delay score.

The regression metrics are used to quantify the similarity of the price feed comparing to the external market price (e.g., CEX). As we mentioned above, if an on-chain price feed is constructed more similar to the external source (i.e., regression metrics are smaller), the cost on deviating the price will be raised. Thus, there is less opportunity to proceed a price manipulation attack on a price feed with low delay and deviation from market. The results in Table II illustrate that ORMER MedDS outperforms all the implemented on-chain oracle in our setup with real-world trading data. Surprisingly, decentralized on-chain ORMER MedDS only shares 17.9% more MAE comparing to a centralized off-chain oracle, which is well-known as a reliable external data source. While ORMER MED trades for more stationary price feed with 4.6% more MAE comparing to the SotA TWAP oracle.

The stationary score is used to reflect the similarity between a price feed and the True Median, which is derived from regression metrics to the external market price. A higher score indicates that the price feed is less sensitive to an outlier price, which is potentially manipulated by malicious parties. The stationary scores in Table II clearly distinguish ORMER MED from other price feeds (e.g., 416.5% to TWAP), indicating ORMER MED’s significant performance in estimating the True Median in a streaming way with low gas consumption. ORMER MedDS’s score is lower because it is translated to fit the latest market price after estimating the True Median. However, this high idleness to fluctuation would result in high market delay, which may expose extra attack interfaces [13], [14]. So the stationary scores are further balanced with the following delay scores to measure the effectiveness on manipulation resistances.

TABLE III: Statistical results of gas consumption and the partitioning percentages of $U p d a t e ( )$ and $Q u e r y ( )$

<table><tr><td rowspan="2">Price Feed</td><td colspan="4">Invocation Gas Cost (GWei)</td></tr><tr><td>Update()</td><td>Percentage</td><td>Query()</td><td>Percentage</td></tr><tr><td>TWAP</td><td>99,614±1</td><td>49.94%</td><td>99,860±7,629</td><td>50.06%</td></tr><tr><td>EMA</td><td>120,733±1,600</td><td>56.42%</td><td>93,248±7,740</td><td>43.58%</td></tr><tr><td>True Median</td><td>45,700±4,433</td><td>12.77%</td><td>312,209±29,175</td><td>87.23%</td></tr><tr><td>Ormer MED</td><td>124,183±34,121</td><td>73.45%</td><td>44,879±589</td><td>26.55%</td></tr><tr><td>Ormer MedDS</td><td>214,000±54,980</td><td>75.27%</td><td>70,317±1,075</td><td>24.73%</td></tr></table>

The delay score reflects the lag between oracle price feed and external market price comparing to True Median. A higher score indicates a better tracing to the truthful market dynamics, and an abnormal fluctuation caused by outlier prices would recover faster. Surprisingly, with the window size set to 300s, the delay measured for ORMER MedDS reaches 532s, only 50.7% of the SotA TWAP. The result indicates that ORMER MedDS may have the potential for dealing with price feed prediction tasks. If the delay is estimated in 1h-window, the statistical result of ORMER MedDS reaches (325.7 ± 891.7)s, even comparable to off-chain oracle Chainlink with (313.2 ± 872.1)s. The results of $\mathrm { S c r } _ { X } ^ { \mathrm { D e } }$ illustrate that ORMER MedDS outperforms TWAP and True Median for 95.5% and 79.3% respectively. Based on the results, we can conclude that the delay suppression method introduced in §IV-C works well on real-world dataset. For ORMER MED, although less sensitive to market dynamics by design, still achieves more than 97.7% performance of TWAP according to Delay Score.

Jointly considering the aspects mentioned by taking average of Station Scores and Delay Scores, ORMER MedDS achieves 90.0% performance of True Median and 65.7% performance of Chainlink regarding deviation resistance, with 75.4% improvement to TWAP; ORMER MED achieves 67.5% performance of True Median and 49.3% performance of Chainlink, with 31.6% improvement to TWAP. Therefore, ORMER is manipulation resistant.

## D. Effectiveness on Gas Efficiency

In our experiment setup, the gas is measured on 10, 000 contract invocation transactions from two separate blockchain transaction commits: invocation of Update() function that triggers state update of oracle contract, and invocation of Query() function that simply returns current p<sup>o</sup>. In real-world scenario, U pdate() would be triggered by traders who swap cryptocurrencies in the exchanges, while Query() would be invoked by downstream DeFi applications requiring reliable price feed.

Gas score results in Table II indicate that ORMER MED outperforms True Median for 111.7%. It even outperforms TWAP for 18.0%, which is widely deployed in production as its low gas consumption. As for ORMER MedDS, since it requires one more auxiliary storage slot for delay suppression (i.e., more manipualtion resistance), it only outperforms True Median for 25.9%. But it still reaches 70.2% performance of SotA TWAP regarding gas efficiency.

As shown in Table III, comparing to other existing onchain oracles, ORMER Contract spends more cost on U pdate() invocations, while True Median spends most of the cost on Query(). This is because ORMER algorithms require complex state updates by writing more information on blockchain (writing data on-chain is expensive), but querying estimated median would require less computations. For True Median, to reduce gas cost, we maintain a pre-allocated space in our implementation. That is to say, updating states involves minimum slot storage change, but querying would require more reading-extensive operations to rebuild price feed with given window size.

As for detailed comparisons to TWAP, ORMER MedDS with higher gas consumption would cost (70, 317 ± 1, 075) GWei (around 21.1 USD, estimated according to: 3, 000 USD/ETH, 100 Gas Price) for querying oracle price, even less that TWAP oracle querying historical data in ring buffer with (99, 860 ± 7, 629) GWei (around 30.0 USD) [3]. This would significantly help to reduce trading cost and elevate liquidity efficiency to the DeFi market. ORMER MED even reaches 44.9% gas consumption on Query() compared to TWAP. For U pdate() that requires more complex smart contract storage changing, ORMER MED and ORMER MedDS cost 24.7% and 114.8% more gas respectively for more manipulation resistances.

To conclude with Resistance Efficiency Score that jointly considers manipulation resistance and gas efficiency, as shown in Table II, both modes of ORMER Contract outperform price feeds constructed by existing pricing oracles with up to 29.6% improvement comparing to True Median. Based on the obtained evaluation results, we can conclude that the ORMER Contract implementation is manipulation-resistant and gas-efficient.

## E. Ablation Study

To better illustrate the effectiveness of ORMER Contract, ablation studies on window size choices are conducted on five on-chain oracles (i.e., TWAP, EMA, True Median, ORMER MED, and ORMER MedDS).

As shown in Figure 6(a) and Figure 6(b), MSE and Delay (All) would increase lineally when window size grows. For gas consumption, there are no significant changes in updating oracles except ORMER MedDS as shown in Figure 6(c), since ORMER algorithms require to gather sufficient markers to boot. To be more specific, cost of ORMER MedDS would surge up 30% when window size is changed from 23 to 24; for ORMER MED, a smaller surge occurs at window size 9. For comparison fairness, we choose to set the number larger than 23 in Table II (25 is randomly selected, which is also widely used in DeFi according to the literature and our observations). For querying cost in Figure 6(d), there are no significant changes, except

![](images/5d6c9f6de9e4ea37f9c5a2152c89e51e472eb22c9edac155c59660f7f3fe7f27.jpg)  
(a) Window size versus MSE.

![](images/23454246bf6aa95decde18de7e4fa5a2237709eee74cc0df89f0f5c417a031e7.jpg)  
(b) Window size versus Delay (All).

![](images/a1695c62c7ed476521f3551baf889c45b8167087bff14dcf05adebdc11f1751e.jpg)

![](images/6b11294617ab647b369ffac9d06107dbe679394ef0486a10fff715f95454d5e0.jpg)  
(c) Window size versus oracle Update gas consumption. (d) Window size versus oracle Query gas consumption.

![](images/7c684ea2108ece7b580d97349bcf54e45f0cb362f9a99c3ffef29517556d1659.jpg)  
(e) Window size versus Stationary Score.

![](images/104922a4def3429e7ae0c7a91ac156f854dd2d14943450eca16ead795490ccaa.jpg)  
(f) Window size versus Delay Score.

![](images/26f3136f66546e8e3123aa6b8a2028a22b2f4e29c2c39b83eb905653240f33ca.jpg)  
(g) Window size versus Gas Score.

![](images/0554adcd3b36509a5fd9e02e2e4cefed7bbc1c564f7750d0a357b3f212c4fa72.jpg)  
(h) Window size versus Resistance Eficiency Score.  
Fig. 6: Ablation studies on influences of window size on evaluation metrics.

True Median as it would require more slot readings to find the accurate median with a bigger window size.

Stationary scores of TWAP, EMA, and ORMER MedDS would decrease exponentially as window size grows, as illustrated in Figure 6(e), since the shape would be distorted significantly from CEX compared to True Median. Meanwhile, ORMER MED demonstrates its great estimation of True Median with best Stationary Score among all the evaluated oracles except window size with 10. Figure 6(f) demonstrates the effectiveness of the Delay Suppression method proposed in §IV-C, while Figure 6(g) illustrates the effectiveness of the Slot Encoding method proposed in §IV-D. In the end, Figure 6(h) demonstrates the overall resistance efficiency of ORMER.

As the only tunable parameter, in this paper, we yield downstream applications to choose the number according to their practical businesses, so the optimal number is not pursued in our work (e.g., smaller for low-delay derivative trading, greater for long-term trend analysis). Dynamically tuning the number based on market conditions would be a great direction for future works. For this work, we would recommend using a fixed small window size to track up-to-date market price.

## F. Case Study on Manipulation Attacks

We have shown the significances of both ORMER MED and ORMER MedDS in a real-world flash loan attack in Figure 1. To make manipulation-resistant property more clear, we provide a zoom-in case study on clipped real-world trading data as shown in Figure 7. The original DEX price datapoints in the clipped period of time remains small fluctuations around 407 USD. We replayed a manipulation attack on 3-th, 4-th, 5-th, 6-th, and 7-th datapoints. TWAP, True Median, EMA, ORMER MED, and ORMER MedDS are then all employed with window size of 12 for pricing oracle feed constructions. The results intuitively confirms the stationary property of ORMER MED and True Median as they stay inactive for the peak, while both ORMER MedDS and EMA are sensitive to the changes. However, EMA is misled by the 4-th peak in the following prices, while ORMER MedDS stays stationary at 4-th peak and self-corrects the price quickly after 01:07:00 based on the latest market dynamics. Both ORMER MED and EMA demonstrates some sort of self-correction property after manipulation, while TWAP behaves lagged in recovering. To conclude, all the pricing oracles shown in Figure 7 demonstrates different levels of manipulation resistance, but only ORMER MedDS follows latest DEX price feed actively.

![](images/ac78b4d460aaf843859487a97ce5baf95955cf737271423195aec515f4a3bdc5.jpg)  
Fig. 7: Case Study of manipulation resistance on real-world token swapping historical data [1].

To conclude, ORMER MED suits for use cases that are more sensitive to gas (e.g., on-chain order book [7]), while ORMER

MedDS would be a promising alternative to the SotA TWAP as a new pricing oracle infrastructure. Both oracles provide a more cost-efficient option by balancing all the metrics.

## VI. DISCUSSION

Potential Risks. According to Figure 4, we would prefer to update current marker at index i with its two neighbor markers (i − 1 and i + 1) due to the gas consideration. This constrains the number of coefficients to limited 3: $a , b ,$ and $c .$ In fact, estimating median through piecewise-parabolic formula with order of 2 would also deviates when dealing with tremendous extreme cases, saying 10, 000% of price deviation. To better accommodate such cases, higher order polynomial formula is needed $( \mathrm { e . g . } , \ h _ { i } = a n _ { i } ^ { 4 } + b n _ { i } ^ { 3 } + c n _ { i } ^ { 2 } + d n _ { i } + e )$ . However, to solve the higher order equations, more linear combination computations would be needed for the estimation of single update of $h _ { i } ^ { \prime } .$ A possible solution is to fix the coefficient limitation while trying to solve formula like $h _ { i } = a n _ { i } ^ { 6 } { + } b n _ { i } ^ { 2 } { + } c .$ But our evaluation results indicate that the algorithm would also be impractical for on-chain pricing oracle regarding gas consumption. That is to say, the reason that piecewiseparabolic formula is selected as our core estimation formula in ORMER is based on the trade-off between robustness and gas consumption.

The Delay Suppression introduced in §IV-C may introduce new risks of inaccurate price predictions. However, the threats can be negligible in practice under the assumptions in this work. First, according to large-scale real-would evaluation results, which potentially contains unknown attacks, the overall evaluations on such dataset proves the soundness of ORMER’s manipulation-resistant property. Second, the cost-of-attacks on SotA oracles are significantly lower than ORMER theoretically. As an example, a manipulation on a single block (i.e., time step) is sufficient to influence the output of TWAP, while an observable attack on ORMER requires successive multi-block manipulations, which is possible but very hard to achieve and shares negligible possibility in real-world scenarios.

Scalability. As ORMER Contract is implemented with Solidity, which could be directly migrated to EVM compatible blockchains without any modification (e.g., Tron, Polygon). For other chains (e.g., Solana), it would require rewriting smart contracts with different programming languages (e.g., Rust). Therefore, the deployment could be trivial.

## VII. RELATED WORK

Attack Detection. One possible countermeasure is to detect and deny transactions containing unlawful operations online (e.g., flash loan based manipulations) before they are confirmed on-chain. Researches have developed automated pattern detection methods for attack identification [28], [29], [30], [18]. However, detecting such patterns according to semantic analysis requires additional domain knowledge and complicated execution logic on smart contracts (e.g., smart contract fuzzing [31]), which is thereby hard to be implemented and deployed into DeFi applications. There are also studies focusing on analyzing offline transaction histories to find out fraudulent users, which can be further blocked by the oracle maintainers [32], [33]. But maintaining a blacklist on-chain would cost a huge amount of money, and querying a growing list would introduce new problems for on-chain smart contracts.

Blockchain Pricing Oracle. Another countermeasure is mitigating the manipulation impacts through a pricing oracle. Currently, TWAP is widely used both in Decentralized Finance (DeFi) and Traditional Finance (TradFi) providing reference price data feed that smooth the original price data feed incoming from an upstream Price Data Source (e.g., DEX, Stock Exchange). The goal of such mechanism is to provide a relatively stable pricing data feed in an ever changing market, while providing fair resistance to the manipulation attack, even in TradFi. The underlying principle of TWAP is a sliding-window based moving average filter that smooths the price feed from the source. For the on-chain TWAP Pricing Oracle, in order to be more cost effective (i.e., gas efficient), only the accumulative price $\begin{array} { r } { \boldsymbol { a } _ { t } = \sum _ { i = 1 } ^ { t } p _ { i } } \end{array}$ of spot prices is updated and saved in the oracle contract once a new block is proposed at time t, and the current oracle reference price $p _ { t } ^ { o }$ with observation window L is estimated according to $\begin{array} { r } { p _ { t } ^ { o } = \frac { \sum _ { i = t - t _ { L } } ^ { t } p _ { i } } { t - t _ { L } } = \frac { a _ { t } - a _ { t _ { L } } } { t - t _ { L } } } \end{array}$

However, recent researches indicate that TWAP is not manipulation-resistant as expected [11], [34], [19]. Park et al. proposed a Kalman Filter based conformal prediction oracle contract that could give an uncertain price interval of current spot price [35]. But the reliability of the results constructed by their method relies on combining information from multiple data sources, while the pick of exact numerical current price estimation is not discussed in the paper. Another off-chain Price Data Source called Chainlink periodically queries data from multiple off-chain CEXs while constructing price feeds based on them, which are further relayed to on-chain contracts [24]. Despite a price feed provided by Chainlink is efficient and reliable, it works in a fully off-chain manner, which is infeasible to be directly invoked by on-chain smart contracts, and centralized managed relaying contracts may pose potential trust issues. There are also some researches focus on outsourcing complex on-chain contract computation to trusted off-chain compute nodes, such as: SMART [36], POSE [37], and Arbitrum [38]. However, these methods would require introducing additional security assumptions to the system (e.g., Trusted Execution Environment) [39], [40].

## VIII. CONCLUSION

In this paper, we propose an on-chain streaming median estimation algorithm called ORMER, which is implemented as ORMER Contract with two price feed mode: ORMER MED and ORMER MedDS. The evaluation on large amount of realworld data illustrates that ORMER Contract, as a pricing oracle, is manipulation-resistant and gas-efficient, which reaches significant result compared to existing methods. It is worth highlighting that querying on ORMER Contract consumes comparable gas to TWAP, while reaches same level of time delay in 1h-window setting to an off-chain oracle in a purely on-chain manner.

[1] Etherscan, “Eminence attack transaction,” https://etherscan.io/tx/0x35f8 d2f572fceaac9288e5d462117850ef2694786992a8c3f6d02612277b0877, 2020.

[2] S. Werner, D. Perez, L. Gudgeon, A. Klages-Mundt, D. Harz, and W. Knottenbelt, “Sok: Decentralized finance (defi),” in Proceedings of the 4th ACM Conference on Advances in Financial Technologies, 2022, pp. 30–46.

[3] H. Adams, N. Zinsmeister, M. Salem, R. Keefer, and D. Robinson, “Uniswap v3 core,” Tech. rep., Uniswap, Tech. Rep., 2021.

[4] L. Heimbach, E. Schertenleib, and R. Wattenhofer, “Exploring price accuracy on uniswap v3 in times of distress,” in Proceedings of the 2022 ACM CCS Workshop on Decentralized Finance and Security, 2022, pp. 47–53.

[5] A. Labs, “Aave protocol,” https://github.com/aave/aave-protocol/, 2020.

[6] G. H. Robert Leshner, “Compound: The money market protocol,” https: //compound.finance/documents/Compound.Whitepaper.pdf, 2019.

[7] S. Team, “Synfutures v3: The oyster amm model for next-gen defi derivatives [draft],” https://www.synfutures.com/v3-whitepaper.pdf, 2024.

[8] DefiLlama, “Defillama,” https://defillama.com/, 2024.

[9] R. Fritsch, “Concentrated liquidity in automated market makers,” in Proceedings of the 2021 ACM CCS Workshop on Decentralized Finance and Security, 2021, pp. 15–20.

[10] W. Ni, Z. Yiwei, W. Sun, L. Chen, P. Cheng, C. J. Zhang, and X. Lin, “Money never sleeps: Maximizing liquidity mining yields in decentralized finance,” in Proceedings of the 30th ACM SIGKDD Conference on Knowledge Discovery and Data Mining, 2024, pp. 2248–2259.

[11] T. Mackinga, T. Nadahalli, and R. Wattenhofer, “Twap oracle attacks: Easier done than said?” in 2022 IEEE International Conference on Blockchain and Cryptocurrency (ICBC). IEEE, 2022, pp. 1–8.

[12] E. Koutsoupias, P. Lazos, F. Ogunlana, and P. Serafino, “Blockchain mining games with pay forward,” in The World Wide Web Conference, 2019, pp. 917–927.

[13] L. Heimbach, V. Pahari, and E. Schertenleib, “Non-atomic arbitrage in decentralized finance,” in 2024 IEEE Symposium on Security and Privacy (SP). IEEE Computer Society, 2024, pp. 224–224.

[14] Y. Wang, Y. Chen, H. Wu, L. Zhou, S. Deng, and R. Wattenhofer, “Cyclic arbitrage in decentralized exchanges,” in Companion Proceedings of the Web Conference 2022, 2022, pp. 12–19.

[15] J. McKay, “Defi-ing cyber attacks,” https://tellingstorieswithdata.com/i nputs/pdfs/final paper-2022-jack mckay.pdf, 2022.

[16] L. Zhou, X. Xiong, J. Ernstberger, S. Chaliasos, Z. Wang, Y. Wang, K. Qin, R. Wattenhofer, D. Song, and A. Gervais, “Sok: Decentralized finance (defi) attacks,” in 2023 IEEE Symposium on Security and Privacy (SP). IEEE, 2023, pp. 2444–2461.

[17] K. Kulkarni, T. Diamandis, and T. Chitra, “Routing mev in constant function market makers,” in International Conference on Web and Internet Economics. Springer, 2023, pp. 456–473.

[18] R. Xi, Z. Wang, and K. Pattabiraman, “Pomabuster: Detecting price oracle manipulation attacks in decentralized finance,” in 2024 IEEE Symposium on Security and Privacy (SP). IEEE, 2024, pp. 3923–3942.

[19] K. Qin, L. Zhou, B. Livshits, and A. Gervais, “Attacking the defi ecosystem with flash loans for fun and profit,” in International conference on financial cryptography and data security. Springer, 2021, pp. 3–32.

[20] X. T. Lee, A. Khan, S. Sen Gupta, Y. H. Ong, and X. Liu, “Measurements, analyses, and insights on the entire ethereum blockchain network,” in Proceedings of The Web Conference 2020, 2020, pp. 155– 166.

[21] A. Akinshin, “Trimmed harrell-davis quantile estimator based on the highest density interval of the given width,” Communications in Statistics-Simulation and Computation, vol. 53, no. 3, pp. 1565–1575, 2024.

[22] R. Jain and I. Chlamtac, “The p2 algorithm for dynamic calculation of quantiles and histograms without storing observations,” Communications of the ACM, vol. 28, no. 10, pp. 1076–1085, 1985.

[23] Etherscan, “Uniswap v2 usdt-weth swapping pool,” https://etherscan.io /address/0x0d4a11d5eeaac28ec3f61d100daf4d40471f1852, 2023.

[24] L. Breidenbach, C. Cachin, B. Chan, A. Coventry, S. Ellis, A. Juels, F. Koushanfar, A. Miller, B. Magauran, D. Moroz et al., “Chainlink 2.0: Next steps in the evolution of decentralized oracle networks,” Chainlink Labs, vol. 1, pp. 1–136, 2021.

[25] ABDK, “Abdk libraries for solidity,” https://github.com/abdk-consultin g/abdk-libraries-solidity, 2019.

[26] NomicFoundation, “Hardhat,” https://github.com/NomicFoundation/har dhat, 2024.

[27] F. Pedregosa, G. Varoquaux, A. Gramfort, V. Michel, B. Thirion, O. Grisel, M. Blondel, P. Prettenhofer, R. Weiss, V. Dubourg, J. Vanderplas, A. Passos, D. Cournapeau, M. Brucher, M. Perrot, and E. Duchesnay, “Scikit-learn: Machine learning in Python,” Journal of Machine Learning Research, vol. 12, pp. 2825–2830, 2011.

[28] Q. Xia, Z. Huang, W. Dou, Y. Zhang, F. Zhang, G. Liang, and C. Zuo, “Detecting flash loan based attacks in ethereum,” in 2023 IEEE 43rd International Conference on Distributed Computing Systems (ICDCS). IEEE, 2023, pp. 154–165.

[29] Z. Chen, S. M. Beillahi, and F. Long, “Flashsyn: Flash loan attack synthesis via counter example driven approximation,” in Proceedings of the IEEE/ACM 46th International Conference on Software Engineering, 2024, pp. 1–13.

[30] X. Deng, S. M. Beillahi, C. Minwalla, H. Du, A. Veneris, and F. Long, “Safeguarding defi smart contracts against oracle deviations,” in Proceedings of the IEEE/ACM 46th International Conference on Software Engineering, 2024, pp. 1–12.

[31] P. Qian, H. Wu, Z. Du, T. Vural, D. Rong, Z. Cao, L. Zhang, Y. Wang, J. Chen, and Q. He, “Mufuzz: Sequence-aware mutation and seed mask guidance for blockchain smart contract fuzzing,” in 2024 IEEE 40th International Conference on Data Engineering (ICDE). IEEE, 2024, pp. 1972–1985.

[32] S. Zhong and A. Mueen, “Bitlink: Temporal linkage of address clusters in bitcoin blockchain,” in Proceedings of the 30th ACM SIGKDD Conference on Knowledge Discovery and Data Mining, 2024, pp. 4583– 4594.

[33] Y. Elmougy and L. Liu, “Demystifying fraudulent transactions and illicit nodes in the bitcoin network for financial forensics,” in Proceedings of the 29th ACM SIGKDD Conference on Knowledge Discovery and Data Mining, 2023, pp. 3979–3990.

[34] M. Bentley, “Manipulating uniswap v3 twap oracles,” https://github.c om/euler-xyz/uni-v3-twap-manipulation/tree/master, 2022.

[35] S. Park, O. Bastani, and T. Kim, “Acon2: Adaptive conformal consensus for provable blockchain oracles,” in Proceedings of the 32nd USENIX Conference on Security Symposium, 2023, pp. 3313–3330.

[36] J. Huang, L. Kong, G. Cheng, Q. Xiang, G. Chen, G. Huang, and X. Liu, “Advancing web 3.0: Making smart contracts smarter on blockchain,” in Proceedings of the ACM on Web Conference 2024, 2024, pp. 1549–1560.

[37] T. Frassetto, P. Jauernig, D. Koisser, D. Kretzler, B. Schlosser, S. Faust, and A.-R. Sadeghi, “Pose: Practical off-chain smart contract execution,” in Network and Distributed System Security (NDSS) Symposium 2023, 2023, pp. 1–18.

[38] H. Kalodner, S. Goldfeder, X. Chen, S. M. Weinberg, and E. W. Felten, “Arbitrum: Scalable, private smart contracts,” in 27th USENIX Security Symposium (USENIX Security 18), 2018, pp. 1353–1370.

[39] M. Fang, X. Zhou, Z. Zhang, C. Jin, and A. Zhou, “Seframe: An sgx-enhanced smart contract execution framework for permissioned blockchain,” in 2022 IEEE 38th International Conference on Data Engineering (ICDE). IEEE, 2022, pp. 3166–3169.

[40] C. Li, B. Palanisamy, and R. Xu, “Scalable and privacy-preserving design of on/off-chain smart contracts,” in 2019 IEEE 35th International Conference on Data Engineering Workshops (ICDEW). IEEE, 2019, pp. 7–12.

[41] Etherscan, “Ethereum gas tracker,” https://etherscan.io/gastracker, 2024.

[42] Binance, “Bnb smart chain white paper,” https://github.com/bnb-chain /whitepaper, 2020.

[43] R. McLaughlin, C. Kruegel, and G. Vigna, “A large scale study of the ethereum arbitrage ecosystem,” in 32nd USENIX Security Symposium (USENIX Security 23), 2023, pp. 3295–3312.

[44] Y. Cao, C. Zou, and X. Cheng, “Flashot: a snapshot of flash loan attack on defi ecosystem,” arXiv preprint arXiv:2102.00626, 2021.

[45] G. Angeris, A. Agrawal, A. Evans, T. Chitra, and S. Boyd, “Constant function market makers: Multi-asset trades via convex optimization,” in Handbook on Blockchain. Springer, 2022, pp. 415–444.

[46] I. Bordino, N. Kourtellis, N. Laptev, and Y. Billawala, “Stock trade volume prediction with yahoo finance user browsing behavior,” in 2014 IEEE 30th International Conference on Data Engineering. IEEE, 2014, pp. 1168–1173.