# LLM-Powered Detection of Price Manipulation in DeFi

LU LIU, The Hong Kong University of Science and Technology, China

WUQI ZHANG, MegaETH, China

LILI WEI, McGill University, Canada

HAO GUAN, Nankai University, China

YONGQIANG TIAN, Monash University, Australia

YEPANG LIU, Southern University of Science and Technology, China

SHING-CHI CHEUNG, The Hong Kong University of Science and Technology, China

Decentralized Finance (DeFi) smart contracts manage billions of dollars, making them a prime target for exploits. Price manipulation vulnerabilities, often via flash loans, are a devastating class of attacks causing significant financial losses. Existing detection methods are limited. Reactive approaches analyze attacks only after they occur, while proactive static analysis tools rely on rigid, predefined heuristics, limiting adaptability. Both depend on known attack patterns, failing to identify novel variants or comprehend complex economic logic. We propose PMDetector, a hybrid framework combining static analysis with Large Language Model (LLM)-based reasoning to proactively detect price manipulation vulnerabilities. Our approach uses a formal attack model and a three-stage pipeline First, static taint analysis identifies potentially vulnerable code paths. Second, a two-stage LLM process filters paths by analyzing defenses and then simulates attacks to evaluate exploitability. Finally, a static analysis checker validates LLM results, retaining only high-risk paths and generating comprehensive vulnerability reports. To evaluate its efectiveness, we built a dataset of 73 real-world vulnerable and 288 benign DeFi protocols. Results show PMDetector achieves 88% precision and 90% recall with Gemini 2.5-flash, significantly outperforming state-of-the-art static analysis and LLM-based approaches. Auditing a vulnerability with PMDetector costs just \$0.03 and takes 4.0 seconds with GPT-4.1, ofering an eficient and cost-efective alternative to manual audits.

Additional Key Words and Phrases: smart contract vulnerability, price manipulation, static analysis, large language model

## ACM Reference Format:

Lu Liu, Wuqi Zhang, Lili Wei, Hao Guan, Yongqiang Tian, Yepang Liu, and Shing-Chi Cheung. 2026. LLM-Powered Detection of Price Manipulation in DeFi. 1, 1 (April 2026), 21 pages. https://doi.org/10.1145/nnnnnnn.nnnnnnn

## 1 Introduction

Decentralized Finance (DeFi) [10] has emerged as a significant force on the blockchain, with automated smart contracts managing over a hundred billion dollars in value [27]. These contracts enable novel financial services such as decentralized exchanges (DEX) and lending protocols [44]. However, core blockchain principles such as immutability and

Authors’ Contact Information: Lu Liu, lliubf@connect.ust.hk, The Hong Kong University of Science and Technology, Computer Science and Engineering Department, Hong Kong, China; Wuqi Zhang, wuqi.zhang@connect.ust.hk, MegaETH, Hong Kong, China; Lili Wei, lili.wei@mcgill.ca, McGill University Montreal, Canada; Hao Guan, hguandl@icloud.com, Nankai University, Tian Jin, China; Yongqiang Tian, yongqiang.tian@monash.edu, Monash University Melbourne, Australia; Yepang Liu, liuyp1@sustech.edu.cn, Southern University of Science and Technology, Shen Zhen, China; Shing-Chi Cheung, scc@cse.ust.hk, The Hong Kong University of Science and Technology, Hong Kong, China.

autonomous execution make it crucial to ensure the security of smart contracts. Once deployed, a flawed contract cannot be easily patched. A single vulnerability can be exploited systematically, leading to catastrophic financial losses [22, 26].

Among diverse security threats, price manipulation vulnerabilities are particularly critical and financially devastating. These attacks occur when an adversary artificially distorts an asset’s price as reported by on-chain price oracles [74]. Victim protocols that blindly trust the price information for critical operations, such as determining collateral value o calculating swap rates, become vulnerable to exploitation. Attacks recorded on DefiHacklabs [11] between Jan 2023 and May 2025 show that price manipulation was the root cause of 17.3% of all major exploits, resulting in losses exceeding \$165.8 million [20]. BonqDao, one of the highest-impact price manipulation attacks, caused a loss of \$88.0 million [1]. A common attack vector involves an attacker using a flash loan to execute a large trade, which temporarily unbalances liquidity pools on decentralized exchanges. A victim protocol using the spot price from pools as an oracle can then be drained.

The research community has developed automated detection techniques, broadly categorized into post-mortem and pre-mortem analysis approaches [33, 48, 68, 69, 71]. Post-mortem analysis tools like DeFiRanger [69] and DeFort [71] analyze on-chain transactions for already occurring attacks. While efective for monitoring attacks, they are not applicable for pre-deployment vulnerability prevention. Conversely, static analysis frameworks like SMARTCAT [33] and DeFiTainter [48] ofer a pre-mortem solution by proactively analyzing source code or bytecode. However, their eficacy is constrained by reliance on predefined, manually-crafted heuristics. This rigidity restricts their ability to reason about semantic nuances. They struggle to identify novel price manipulation vulnerabilities that deviate from known patterns, leading to critical false negatives (FNs). This rigidity can also produce false positives (FPs) when encountering legitimate but unconventional defense mechanisms. Thus, a critical gap exists: the need for proactive detection that transcends fixed patterns by combining systematic code analysis with deeper, contextual understanding of economic exploit logic.

The need for proactive and context-aware detection motivates exploring new paradigms beyond traditional program analysis. The recent success of Large Language Models (LLMs) in code understanding and bug detection [47, 49, 52, 55] presents a new opportunity for smart contract security analysis. Recent studies [38, 41, 66] have demonstrated tha models like GPT-4 could identify logic flaws in smart contracts. However, these approaches often sufer from high false positive rates, limiting practical applicability. Subsequent research has focused on improving LLM-based detection reliability through combining LLM analysis with static analysis [43, 59, 70], implementing multi-agent pipelines [65, 66], and developing fine-tuned domain-specific LLMs [34, 53, 75]. Despite demonstrated success on common vulnerabilities, these general-purpose LLM approaches often prove inefective at identifying sophisticated economic exploits like price manipulation attacks. For example, GPTScan [59] uses LLMs to match vulnerability patterns within individual functions; this function-centric view is inadequate for detecting price manipulation, which is a path-dependent exploit spanning multiple contract interactions.

We summarize three main challenges for applying LLMs to the price manipulation detection task: C1: Lack of domain-specific attack knowledge. Price manipulation is fundamentally an economic exploit, not a conventional code bug. Detection requires a deep understanding of DeFi primitives such as AMM invariants and oracle mechanics. General-purpose LLMs, trained primarily on vast code corpora [46], lack this specialized financial knowledge and cannot distinguish legitimate trades from manipulative ones. C2: Inherent hallucination in code reasoning. LLMs are probabilistic systems rather than logical execution engines [61]. They generate explanations by predicting likely word sequences rather than simulating code execution. This makes them frequently “hallucinate” execution paths that are syntactically plausible but semantically infeasible [58], undermining reliability for security analysis. C3: Limited Manuscript submitted to ACM

generalization beyond seen patterns. Current models exhibit over-reliance on syntactic patterns from training data. They often fail to identify novel attack vectors that are semantically equivalent to known exploits but syntactically distinct. Unlike well-documented vulnerabilities such as reentrancy [4], price manipulation vulnerabilities sufer from limited representation in existing datasets, further constraining LLM understanding.

To address these challenges, we present PMDetector, a novel framework that integrates taint analysis, LLM-based reasoning, and rule-based static checking. First, to address the lack of domain knowledge (C1) and poor generalization (C3), we introduce a formal attack model that defines price manipulation semantics. The model guides static taint analysis to identify data flows where external price input could influence critical financial operations, and guides LLM prompt design for path filtering and attack simulation. By constraining the problem space, the model focuses LLM analysis on plausible vulnerabilities and relevant DeFi exploit context, reducing false negatives. Next, we leverage a two-stage LLM pipeline as a semantic analysis engine. In the path filtering stage, the LLM evaluates defense mechanism eficacy, pruning safe execution flows. For remaining high-risk candidates, the attack simulation prompts the LLM to construct viable exploit scenarios. This structured pipeline concentrates LLM reasoning capabilities and lowers the risk of false positives. Finally, to mitigate LLM hallucination (C2), a rule-based static checker validates remaining high-risk paths. This module acts as a lightweight verifier, cross-referencing paths against established defense mechanisms, eliminating false positives from LLM hallucinations

To evaluate PMDetector, we construct a benchmark of real-world DeFi protocols with confirmed price manipulation vulnerabilities. The benchmark comprises 73 contracts from prior benchmarks [36, 68, 69, 71] and the public security dataset DeFiHackLabs [20]. The set spans diverse protocol categories and covers incidents from 2020 to 2025, providing broad coverage of price-manipulation patterns. Financial losses from these 73 contracts exceed \$258 million [11]. We also include a balanced set of 288 benign contracts from prior studies [48]. Results demonstrate that PMDetector achieves 88% precision and 90% recall using Gemini 2.5-flash [14], outperforming state-of-the-art static analysis and LLM-based baselines. Ablation studies confirm the contribution ofeach component and removing any component leads to substantial drops in precision and recall, validating our architectural design. The cost to audit a price manipulation vulnerability with PMDetector is \$0.03 for GPT 4.1, substantially lower than typical human-audit costs (\$5,000-\$15,000) [30].

In summary, this paper makes the following contributions.

• A comprehensive benchmark for price manipulation. We construct a comprehensive benchmark comprising 73 real-world vulnerable contracts responsible for over \$258 million in losses. Systematically curated from prior research and public incident reports, the benchmark ofers broad coverage of attack patterns across diverse DeFi protocols from 2020 to 2025.

• A novel hybrid detection framework. We present PMDetector, a hybrid framework combining static analysis and LLM-based reasoning to overcome individual limitations. It employs static taint analysis for candidate identification and performs a two-step LLM pipeline for context-aware semantic validation, followed by static validation checking

• Superior detection performance. We evaluate PMDetector on our benchmark and safe real-world DeFi protocols. Results demonstrate state-of-the-art accuracy, significantly outperforming existing static analysis and LLM-based baselines.

## 2 Background

## 2.1 Automated Market Makers and Price Discovery in DeFi Ecosystem

Automated Market Makers (AMMs) [8] emerged as a solution to address the fundamental limitations of traditiona Central Limit Order Books (CLOBs) [6] in on-chain transaction environments. Traditional CLOBs operate through a centralized matching mechanism where market participants submit buy and sell orders at specified price levels which are subsequently matched by a central authority. This approach faces significant scalability challenges when deployed in decentralized environments, particularly due to high transaction costs and substantial network latency. AMMs restructure decentralized trading by replacing the order book mechanism with liquidity pools, which are smar contracts holding reserves of two or more tokens. Liquidity providers (LPs) [19] deposit paired assets into these pools and receive LP tokens proportional to their contribution to the pool’s total liquidity. These pooled assets are then available for other users to trade against.

The core of an AMM lies in its pricing algorithm, which determines the exchange rate between the assets in the pool based solely on their respective quantities. The most influential and widely adopted model is the constant product market maker (CPMM) [8]. A CPMM maintains the invariant $x \cdot y = k $ , where � and � represent the quantities of two tokens (e.g., ETH [12] and USDC [28]) in the pool, and � is a constant. When users initiate a trade, they deposit a certain amount of one token (Δ�) into the pool and receive a corresponding amount of the other token (Δ�), ensuring that the invariant $( x + \Delta x ) \cdot ( y - \Delta y ) = k$ remains satisfied (excluding transaction fees)

The spot price in such a pool is not explicitly stored but rather derived implicitly as the ratio of the reserves: $P = x / y$ Consequently, any transaction that alters the reserve quantities � and � simultaneously modifies the instantaneous price. This mechanism enables on-chain price discovery and serves as a critical primitive in the decentralized finance (DeFi) ecosystem [10]. The price information reported by prominent AMM pools is often used by other decentralized applications (DApps) [9], such as lending protocols, derivatives platforms, and yield aggregators. These DApps rely on AMM-derived prices as price oracles to determine collateral values, calculate liquidation thresholds, and settle financia contracts. However, the reliance on AMM-derived pricing creates a significant attack vector that can be exploited by malicious actors.

## 2.2 Price Manipulation Vulnerabilities in DeFi Protocols

Since AMM prices are direct functions of token reserves within liquidity pools, actors with suficient capital can execute large trades that significantly alter reserve ratios, creating temporary but substantial price dislocation. The goal of such an attack is not to profit from the trade itself, but to exploit a separate, victim protocol that uses the manipulated price from the AMM as a trusted oracle. Historically, executing such an attack required an immense upfront capita commitment, making it impractical for all but the wealthiest actors. However, the introduction of flash loans [10] fundamentally transformed this attack landscape. A flash loan is an uncollateralized loan that must be borrowed and repaid within the same atomic transaction [29]. If the borrower cannot repay the full amount plus a small fee by the end of the transaction, the entire transaction, including all actions performed, is reverted. This mechanism allows an attacker to borrow millions of dollars’ worth of cryptocurrency for a few seconds, use it to manipulate marke conditions, and repay the loan using the proceeds of their exploit, all with zero initial capital.

In this work, we focus on flash loan-based vulnerabilities, excluding front-running and sandwich attacks, narrowing the scope to manipulations occurring within one transaction, leveraging the atomic nature of blockchain transactions.

Manuscript submitted to ACM

![](images/2b9ae12dc722598b869c7b3994a475a1f5fe1073564bcbd2d2d1c9cb4d98da30.jpg)  
Fig. 1. Vulnerable logic in the ZZF protocol.

## 3 Motivation and Atack Model

This section presents an example of real-world price manipulation vulnerabilities, followed by the definition of an attack model that captures the essential characteristics and attack vectors of this vulnerability class.

## 3.1 Illustrating Example

Figure 1 shows a real-world price manipulation vulnerability found in the ZZF protocol [3], a BEP-20 token contract deployed on the BNB Smart Chain [5]. The exploit resulted in a loss of approximately \$223,000 [2]. The vulnerability resides within the ZZF smart contract’s reward distribution mechanism. The protocol rewards users with ZZF tokens for burning ZongZi tokens (\_burnToken at line 3), with rewards proportional to the burned tokens’ value. The flaw lies in how the contract determines this value. The burnToHolder function uses the spot price from a PancakeSwap liquidity pool [23] as its price oracle via uniswapRouter.getAmountsOut (line 6). The variable deserved, representing the burned tokens’ value in WBNB, is calculated based on current reserves of the ZongZi/WBNB liquidity pool. In automated market makers (AMMs), token spot prices are determined by reserve ratios within liquidity pools. The spot price of WBNB is �/�, where � and � represent WBNB and ZongZi token reserves, subject to the constant product constraint � · = �. This mechanism makes both prices highly susceptible to manipulation through large trades that alter the reserve ratio. The manipulated deserved value is passed to burnFeeRewards, which calculates and distributes the ZZF reward (line 9). This function uses the manipulated increase value to transfer disproportionately large amounts of ZZF tokens to the attacker (line 14). The attacker then calls receiveRewards to swap these tokens for WBNB, draining the contract’s funds.

Static analysis excels at detecting data flows but does not interpret business logic or economic intent. Consider the ZZF contract: a static analyzer can trace that burnToHolder invokes burnFeeRewards, which calls \_transfer, showing data flow from getAmountsOut to token transfer. However, this appears as disconnected operations, not a coherent economic mechanism. In contrast, an LLM can infer intent from function names like burnToHolder and burnFeeRewards, suggesting a reward mechanism, and variable names like deserved, implying fair token valuation. The LLM recovers the core business logic: rewarding users with ZZF tokens proportional to the burned ZongZi token value. Once this intent is clear, the security implication follows: manipulating the price oracle undermines the contract’s economic premise, enabling attackers to claim inflated, unearned rewards.

Lu Liu, Wuqi Zhang, Lili Wei, Hao Guan, Yongqiang Tian, Yepang Liu, and Shing-Chi Cheung

![](images/e4579c9094ee8bd69393ff87df663e01e327eaf320b48822f65c771d7e89aa85.jpg)  
Fig. 2. Demonstration of the Atack on the ZZF Protocol

## 3.2 Atack Model

Following existing practices [62, 67, 71, 75], our attack model formalizes flash-loan-based price manipulation as a multi-stage process, which begins with a flash loan and proceeds through manipulation, exploitation, value extraction, and cleanup. We develop taint analysis and LLM reasoning prompts based on this model. We use the ZZF protocol (Figure 1) and its attack demonstration (Figure 2) for illustration.

1) Phase 0: Setup. Focusing on flash-loan-based price manipulation, the attacker first acquires exploitable capital via a flash loan. For example, in Figure 2, an attacker initiates a flash loan to borrow large amounts of WBNB without collateral, serving as capital for subsequent manipulation.

2) Phase 1: Taint Introduction. The attacker introduces a tainted price into the DeFi ecosystem as the taint source, performing actions that cause the protocol’s price-reporting mechanism to reflect abnormal values through large DEX swaps or direct oracle manipulation. In the ZZF protocol, the attacker invokes swap to exchange WBNB for ZongZi tokens within the ZongZi/WBNB pool. This dramatically reduces WBNB reserves while inflating ZongZi reserves, artificially elevating ZongZi price relative to WBNB, resulting in abnormal ZongZi pricing when uniswapRouter.getAmountsOut is called.

3) Phase 2: Propagation and Exploitation. The attacker propagates the manipulated oracle price to subvert protocol logic. In Figure 2, the attacker invokes burnToHolder within the ZZF protocol. This function queries the ZongZi/WBNB pool for ZongZi token price, receiving the inflated price from Phase 1 manipulation. This manipulated price passes to burnFeeRewards, subverting core contract logic by using the malicious value to calculate rewards, transferring illegitimately large quantities of ZZF tokens to the attacker and corrupting internal accounting

4) Phase 3: Value Extraction. The attacker capitalizes on the inflated token balance to drain protocol funds through direct mechanisms (transferring borrowed assets) or indirect methods (updating the attacker’s balance in internal accounting for subsequent withdrawal). In ZZF protocol, the attacker calls burnFeeRewards, transferring large balances of unfairly minted ZZF tokens.

5) Phase 4: Cleanup. The final step cleans up positions and repays flash loans. The attacker unwinds positions, repays loans, restores oracles if necessary, and finalizes profit. In Figure 2, the attacker swaps remaining ZongZi tokens back to WBNB, restoring the liquidity pool’s price.

Manuscript submitted to ACM

![](images/3c84cafc58f737b68c0b8c94447bbebf7ad50c3bac22600532800bd82e387940.jpg)  
Fig. 3. The workflow of PMDetector.

Attackers may loop through these phases, e.g., by re-manipulating oracles after initial value extraction (circular attacks) or chaining exploits across protocols. Our model efectively captures these complex scenarios by focusing on end-to-end flow from manipulated source to profitable sink, regardless of intermediate complexity.

## 4 Methodology

We present PMDetector, a hybrid framework integrating inter-procedural taint analysis, LLM-based reasoning, and semantic sanity checking to detect price manipulation vulnerabilities in DeFi smart contracts. Our approach operates in three phases: (1) formal attack model-guided static taint analysis with path grouping to reduce token consumption, (2) a two-step LLM pipeline for defense validation and high-risk path selection, and (3) a rule-based semantic checker for false positive reduction. Figure 3 illustrates PMDetector’s framework

## 4.1 Static Taint Analysis

Based on the attack model in Section 3.2, we propose an inter-procedural, flow-sensitive static taint analysis framework for identifying vulnerable paths in smart contracts using Slither Intermediate Representation (SlithIR) [39]. The target DeFi protocol is converted into an inter-procedural control-flow graph (ICFG), then we initiate taint analysis from identified sources and track taints by traversing the ICFG. To overcome limitations of existing heuristic-based approaches [33, 48, 68], rather than relying on pattern matching, we analyze semantic and economic intent in code patterns for more accurate vulnerability detection. We define semantic primitives extracted from the smart contract’s IR (Table 1) as foundational facts for higher-level semantic analysis and establish inference rules based on these primitives. The notation � ↓ represents that variable � is tainted. A ���� (��) predicate is derived if a tainted value reaches operation ��.

4.1.1 Taint Sources. Taint sources are entry points for attacker manipulation, aligning with phase 1 of the attack model. We consider any external information potentially controlled by the attacker as a taint source. Specifically, there are two kinds of taint sources, i.e., data directly provided by the attacker and data indirectly loaded from external contracts that may be controlled by the attacker. The former kind includes inputs of public or external functions and transaction property variables (e.g. msg.data, msg.value). The latter are typically external price oracle contracts. We conservatively identify such contracts by marking external function calls as taint sources if they are view or Manuscript submitted to ACM pure and their return values are used in multiplication or division operations. This pattern indicates price/ratio calculations and discovers custom-wrapped oracle calls. We also mark return values from known DEX functions (e.g., getAmountsOut, getReserves) as taint sources. For example, in Figure 1, deserved is identified as a taint source tainted by uniswapRouter.getAmountsOut().

Table 1. Definition of Semantic Primitives.

<table><tr><td>Predicate</td><td>Description</td></tr><tr><td>Call(cs, f, args, ret)</td><td>At call site cs, function f is called with arguments args, and its result is assigned to ret.</td></tr><tr><td>EC(cs, f, ...)</td><td>A call to a function f in an external contract.</td></tr><tr><td>SSTORE(σ, v)</td><td>The value of variable v is written to storage slot σ (SSTORE).</td></tr><tr><td>IsPublic(f)</td><td>Function f has public or external visibility.</td></tr><tr><td>Transfer(r, a)</td><td>r and a are the recipient and the transfer amount for a transfer function.</td></tr><tr><td>Arg(cs, i, v)</td><td>At call site cs, the variable v is passed as the i-th argument.</td></tr><tr><td>IsMappingSlot(σ)</td><td>A mapping from address addr to a uint type unit at base slot σ.</td></tr></table>

4.1.2 Taint Propagation Rules. Taint propagation rules define how taint spreads through data and control dependencies, modeling Phase 2 of our attack model. We consider four types of rules:

1) Intra-procedural Data Flow: Taint propagates through direct data dependencies within a function. If a variable’s value is computed using one or more tainted operands (e.g., in an assignment or binary operation), this variable also becomes tainted.

2) Inter-procedural Data Flow: Taint flows across function call boundaries. When a tainted value is passed as an argument to a function, the corresponding parameter in the callee is tainted. Conversely, a tainted value returned from a function taints the variable that receives the result at the call site.

3) Persistent State Tainting: Taint persists across transactions via contract storage. Writing tainted values to storage marks slots as tainted; reads from tainted slots propagate taint. The analysis handles simple variables, packed struct members, mappings (keccak256(key, p)), and arrays (keccak256(p) + I).

4) Implicit Control-Flow Tainting: Taint propagates based on control dependencies. When tainted variables appear in conditionals (textitif, textitrequire, textitassert), all control-dependent statements are conservatively tainted to maximize detection capability. Note that it may lead to over-tainting as it is based on the principle of minimizing false negatives at the taint analysis phase.

4.1.3 Taint Sinks. Taint sinks, corresponding to Phase 3 of our model, represent operations that confer economic benefit. When tainted data reaches a sink, a taint path is reported. We consider two fundamental pathways for illegitimate profit: direct and indirect value extraction. Direct value extraction occurs when attackers manipulate price oracles and immediately exploit incorrect calculations to transfer assets directly to their address. Indirect value extraction involves a two-step process. First, the attacker corrupts the protocol’s internal state to credit unearned value. Second, they withdraw the value through a subsequent, seemingly legitimate transaction. Our analysis focuses on the critica first step, state corruption. Figure 4 shows the three types of taint sinks defined by us.

1) Tainted Ether and Token Transfer. We define sinks for low-level value transfer calls (.call, .send, .transfer) where the amount derives from a tainted source. Additionally, we flag calls to an ERC20 token’s transfer() or transferFrom() function as sinks when the amount argument is tainted. As formalized in Formula Ether and token Manuscript submitted to ACM

$$
\begin{array}{l l} \frac {\text {Call} (c s , f , \_ , \_) \wedge \text {Transfer} (r , a) \wedge \text {Arg} (c s , \_ , a) \wedge a \downarrow}{\text {Sink} (c s)} & \text {(Ether and token transfer)} \\ \frac {\neg \text {IsPublic} (f) \wedge \exists \sigma , v . s . t . [ \text {SSTORE} (\sigma , v) \in f \wedge \text {IsMappingSlot} (\sigma) ]}{\text {IsLedgerUpdate} (f)} & \\ \frac {\text {Call} (c s , f , \arg s , \_) \wedge \text {IsLedgerUpdate} (f) \wedge \arg s \downarrow}{\text {Sink} (c s)} & \text {(Internal ledger update)} \\ \frac {\text {SSTORE} (\sigma , v) \wedge \text {IsMappingSlot} (\sigma) \wedge v \downarrow}{\text {Sink} (\text {SSTORE} (\sigma , v))} & \text {(Economic state write)} \end{array}
$$

Fig. 4. Definition of taint sinks.  
```txt
Algorithm 1: Taint Analysis Algorithm
Input :C, the smart contract to be analyze
Output :TaintPaths, a set of taint paths
Function TaintAnalysis(C):
    CFGs ← PreProcess (C)
    TaintMap ← IdentifySources (C)
    repeat
        TaintMapold ← TaintMap
        TaintMap ← Propagate (TaintMap, CFGs)
    until TaintMap == TaintMapold
    TaintPaths ← ∅
    foreach instruction Inst in C do
        if IsSink (Inst, TaintMap) then
            Path ← ReconstructPath (Inst, TaintMap)
            TaintPaths ← TaintPaths ∪{Path}
    return TaintPaths
```

transfer, this captures direct fund drainage. In Figure 1, the burnFeeRewards function is marked as a taint sink as it is a value-transfer function.

2) Tainted Internal Ledger Update. Beyond native currency, DeFi protocols manage value through internal accounting systems or ledgers. The most common pattern is mapping(address =⇒ uint), which maintains critical financial data such as balances and shares. Attackers can manipulate oracles to inflate balances for later withdrawal. We flag calls to ledger-update functions with tainted values (Formula Internal ledger update). Functions are classified as ledger-update if they are internal or private and modify mapping state variables.

3) Tainted Economic State Write. While many ledger updates are abstracted and captured by Sink 2, some may be performed via direct, low-level state writes. For direct state writes, we flag SSTORE operations writing tainted values to economic balance storage locations (Formula Economic state write). This detects two-step exploits where attackers corrupt internal state without immediate fund extraction.

4.1.4 Taint Analysis Algorithm. Next, we explore all possible execution paths that could lead to price manipulation. The algorithm consists of three main phases, as detailed in Algorithm 1.

The analysis begins with the PreProcess procedure (Line 2), which parses the input smart contract C to construct Control Flow Graphs (CFGs), which serve as the foundational data structures for applying propagation rules. Next, the algorithm performs a fixed-point computation to identify all tainted variables. The IdentifySources procedure (Line 3) initializes the TaintMap by identifying all initial taint sources according to our definitions. Then, the algorithm enters a repeat-until loop (Line 4-7). In each iteration, the Propagate function is called to apply our data-flow and control-flow propagation rules, expanding the TaintMap with newly discovered taints. The loop continues until no new taints are introduced in an iteration, signifying that a fixed point has been reached. Finally, with the complete TaintMap, the IsSink procedure (Line 10) inspects every instruction Inst to determine if it represents an economic sink. If a sink is found, ReconstructPath (Line 11) traces the taint dependencies from the sink back to its source, and the resulting taint path is added to the output set for further analysis (Line 12). For example, the ZZF protocol (Figure 1) exhibits a taint path that propagates data directly from the external price oracle uniswapRouter.getAmountsOut() to the value-transfer function burnFeeRewards.

4.1.5 Path Grouping and Abstraction. The taint analysis phase may generate numerous paths that are syntactically diferent but semantically redundant. To address this computational expense and reduce noise, we introduce a path grouping and abstraction phase. Our approach is based on the insight that paths sharing the same fundamental vulnerability characteristic can be treated as a single analysis unit. We transform each raw taint path into an enriched data structure by parsing IR instructions to extract high-level semantic features. As shown in Formula 1, we identify security-critical operations: external calls and state variable writes

$$
\frac {o p = \mathrm{EC} (c s , f , \dots) \vee o p = \mathrm{SSTORE} (\sigma , v)}{\text {IsCritical} (o p)}\tag{1}
$$

For each path $P ,$ we generate a unique group key Key(�) (Formula 2) comprising the source function, sink function, and critical operations ��. Paths sharing the same key are clustered into group $G _ { k }$ (Formula 3).

$$
\text {Key} (P) := \langle \text {Source} (P), \text {Sink} (P), \{o p \mid o p \in P \land \text {IsCritical} (o p) \} \rangle\tag{2}
$$

$$
G _ {k} := \{P \mid \operatorname{Key} (P) = k \}\tag{3}
$$

Each group is consolidated into a representative case $P _ { r } ( G _ { k } )$ (Formula 4) by selecting the longest path. The rationale is that the longest path is more likely to capture the most complex control flow and include the logic present in the shorter, subsumed paths. The structured summary serves as input to the LLM phase.

$$
P _ {r} (G _ {k}) := \underset {P \in G _ {k}} {\operatorname{argmax}} | P |\tag{4}
$$

## 4.2 LLM-based Reasoning

To efectively leverage LLMs for identifying price manipulation vulnerabilities from suspicious paths collected during taint analysis, a sophisticated prompting strategy is needed. We observe that a single, static prompt is insuficient, as i fails to guide the LLM’s reasoning process in a context-aware manner, frequently leading to inaccurate conclusions. To address this, we propose a two-stage, template-driven prompting strategy that dynamically customizes LLM inpu based on the specific details of each taint path. Our methodology operates on the principle of providing each LLM agent with only the information most relevant to its specific objective. The process begins with the path filtering stage, which filters a large volume of taint paths to isolate a small set of plausible candidates. Filtered paths are then passed to the attack simulation stage, which attempts to construct a concrete, step-by-step exploit scenario using these filtered paths. For each stage, we design prompt templates containing placeholders. A pre-processing script parses the raw taint analysis output and populates these placeholders, generating customized and detailed prompts for the LLM. This Manuscript submitted to ACM

![](images/ed9ab0cc95e320ae79cd439b5076068c48b8e6aff0ec8bb63d5ed3216b23d58f.jpg)  
Fig. 5. Prompt Templates of the Path Filtering Stage (left) and the Atack Simulation Stage (right).

ensures that the model’s analysis is tailored and dynamically adapts to the specific characteristics of the DeFi protocol under review.

4.2.1 Path Filtering Stage. The path filtering stage serves as a heuristic filter. It examines taint paths, eliminating those incompatible with the attack model’s premises to focus on plausible candidates. We create a benign behavior checklist derived from the attack model, comprising validation rules selected based on price manipulation vulnerability characteristics and existing research findings [62, 67, 71]. Each rule evaluates whether a code path meets necessary conditions for successful attacks.

The left of Figure 5 presents the prompt template for path filtering, which configures the LLM as a smart contract auditor applying rule-based checks. Each rule contains placeholders for specific taint information, including <source function>, <sink function>, <affected states>, and <critical operations>. The <source function> represents the entry point where an external attacker initiates the manipulation attack. The <sink function> is the terminal point where the manipulated state is consumed to generate profit for the attacker. The <critical operations> includes all low-level external call functions that participate in the execution path between the source and sink functions. These operations represent the core mechanisms through which state manipulation is propagated throughout the system. The <affected states> refers to state variables whose value is tainted by the source function and subsequently used by the sink function. The template includes guided analysis questions: (1) Access Control: Does the exploit require compromised privileged accounts? (2) Economic Intent: Is this an intended economic feature of the protocol, not an unintended flaw? Manuscript submitted to ACM (3) Mitigation: Does the code have mitigations like oracle checks or cooldowns to prevent manipulation? For each path group, we populate this template and concatenate the instances into a complete LLM prompt.

4.2.2 Atack Simulation Stage. After path filtering, this stage examines filtered paths to determine if they can create viable economic exploits. We use the template-and-populate approach to guide the LLM in building plausible exploit scenarios.

The right half of Figure 5 outlines the prompt template positioning the LLM as a security auditor analyzing highpotential paths. The template structures the task as step-by-step flash loan attack analysis, with a chain-of-thought reasoning guiding the LLM through four steps. First, the LLM identifies the specific on-chain price source being manipulated. Second, it constructs a detailed attack scenario starting from the entry point, using plausible paths and explaining the attacker’s profit mechanism. Third, it examines the cash-out method by analyzing how an attacker would leverage the profit function. Fourth, it performs a final defense check, re-evaluating all involved functions for subtle, overlooked mitigation like TWAP or reentrancy guards. The LLM outputs structured JSON containing verdict vulnerable functions/paths, and attack explanation. By populating this template with key functions, operations, and state variables from plausible taint paths, we guide the LLM through the logical exploit sequence. This structured approach enhances the LLM’s ability to synthesize taint data into plausible attack narratives.

## 4.3 Semantic Sanity Checker

While the two-step LLM approach efectively reasons about attack feasibility on taint paths, it may still generate false negatives when failing to recognize subtle mitigation controls within the code. To address this, we introduce a semantic sanity checker as the final filtering stage to validate LLM results. This checker serves as a high-precision semantic filter verifying whether high-risk paths are neutralized by established defense mechanisms [69, 71].

Based on the structured Intermediate Representation (IR) of the contract, the checker enables control flow and data dependency analysis. We leverage Slither [39] to parse Solidity code into structured IR format. The checker executes heuristic-based filtering to prune false positives based on defensive code patterns. The most common cause of LLM false positives is failure to recognize security controls that block exploits. Our checker analyzes the call graph of vulnerable functions flagged by the LLM, searching for defense mechanisms that protect high-risk paths:

1) Privilege-based defense: we check if external function calls, especially those interacting with common, wellestablished DEXs (e.g., Uniswap V2) or manipulating core contract business logic, are protected by access control modifiers. This is achieved by identifying common modifiers (e.g., onlyOwner, onlyAdmin) and require(msg.sender==owner) patterns that restrict execution to privileged accounts.

2) Temporal defense: to counter economic exploits like flash loan attacks, developers often introduce time-based controls. Our checker identifies these patterns by searching for require and revert statements that enforce a delay between actions (e.g., require(block.timestamp >= lastActionTime + cooldownPeriod)).

3) Fee-on-transfer pattern: we recognize the fee-on-transfer token mechanism [13] as contextually benign. This pattern involves swap operations within transfer functions where balance updates are finalized before external calls adhering to Checks-Efects-Interactions [7]. Our tool verifies balance updates precede external interactions and calls target known DEX routers, distinguishing legitimate fee collection from malicious reentrancy patterns.

## 5 Evaluation

We aim to evaluate the following research questions:

Manuscript submitted to ACM

• RQ1 (Efectiveness): How efective is PMDetector at detecting price manipulation vulnerabilities in real-world smart contracts?

• RQ2 (Comparison with SOTA): How does PMDetector perform compared with the state-of-the-art tools on price manipulation vulnerability detection?

• RQ3 (Ablation Study): How does each component of PMDetector contributes to its overall performance?

• RQ4 (Execution Cost): What are the execution time and token usage costs of PMDetector?

## 5.1 Experiment Setup

Dataset. To evaluate the efectiveness of PMDetector, we constructed a comprehensive ground-truth dataset with two components: a vulnerability set $( D _ { v u l } )$ and a non-vulnerability set $( D _ { s a f e } )$ . The $D _ { v u l }$ set consolidates cases from four academic benchmarks [36, 68, 69, 71] and the widely adopted industry-maintained DeFiHackLabs incident repository [20] From DeFiHackLabs, we collected all confirmed price manipulation incidents reported between January 2023 and May 2025. We constructed the $D _ { v u l }$ set using the following criteria:

• Open-source and peer-reviewed: The selected vulnerability cases must be open-source and widely used in academic papers or real-world industries.

• Clear vulnerability classification: Each case must be explicitly labeled with price manipulation vulnerability identifiers, ensuring taxonomic clarity and consistency.

• Verifiable exploit impact: The security implications of each vulnerability must be clearly demonstrated through Proof of Concept (PoC) or documented in formal security audit reports.

Based on the above criteria, we examined these five sources to establish our evaluation benchmark for PMDetector. Two of the authors participated in this process and manually verified the datasets. We identified 73 vulnerable contracts with price manipulation vulnerabilities across these datasets, with Solidity versions ranging from 0.4 to 0.8, and established our analysis benchmark. The average lines of code (LoC) for contracts in the benchmark is 301.3. For the non-vulnerable set $D _ { s a f e } ,$ we sampled 288 of the 1,195 protocols from the DeFiTainter dataset [48]. All involved protocols have been audited by security agencies and are reported to be free of price manipulation vulnerabilities. The sample size corresponds to a 95% confidence level with a 5% margin of error [31].

Implementation. We implemented the PMDetector prototype in Python with approximately 3,000 lines of code. For taint analysis and sanity checker, we utilized the Slither Intermediate Representation (SlithIR) [39]. For the LLM evaluation, we selected three state-of-the-art Large Language Models (LLMs) based on their code understanding capabilities and cost-efectiveness: GPT-4.1 [16], Gemini-2.5-Flash [14], and Qwen3-235B-A22B [24].

Evaluation Metrics. For the experiments, we employed five metrics, including true positive (TP), false negative (FN), false positive (FP), precision, recall, and F1-score, to evaluate their performance. For the RQ4 (Execution Cost), we evaluated the execution time, the input and output tokens consumed, and the financial costs of LLMs.

## 5.2 RQ1: Efectiveness

We evaluate PMDetector’s efectiveness by measuring its true positive(TP), false negative (FN), false positive (FP), precision, recall, and F1-score. Table 2 shows the performance. Out of the 73 vulnerable cases, we successfully converted 68 of them into SlithIR and constructed the taint graph. The result is based on these 68 valid cases.

Overall Results. As detailed in Table 2, PMDetector demonstrates high efectiveness across all three LLMs. It achieves a recall of up to 88% (with Gemini2.5-flash) and a precision of up to 100% (with GPT-4.1). The results indicate that our approach is highly capable of accurately identifying vulnerable code paths while maintaining a low false positive rate

Table 2. Performance comparison of PMDetector across diferent models.

<table><tr><td>Model</td><td>#TP</td><td>#FN</td><td>Recall</td><td>#FP</td><td>Precision</td><td>F1-Score</td></tr><tr><td>Gemini2.5-flash</td><td>60</td><td>8</td><td>0.88</td><td>6</td><td>0.90</td><td>0.90</td></tr><tr><td>GPT-4.1</td><td>57</td><td>11</td><td>0.84</td><td>0</td><td>1.00</td><td>0.91</td></tr><tr><td>Qwen3-235B-A22B</td><td>59</td><td>9</td><td>0.87</td><td>10</td><td>0.86</td><td>0.86</td></tr></table>

The results also highlight a classic trade-of. GPT-4.1 achieves perfect precision by being more conservative, while Gemini2.5-flash and Qwen3-235B-A22B achieve higher recall at the cost of a small number of false positives. Even with models that produce FPs, the precision remains high (90% for Gemini and 86% for Qwen). This demonstrates the robustness of our methodology across diferent language model architectures.

False Negatives. False Negatives occur at both the static analysis and LLM-based reasoning stages. In taint analysis, false negatives arise when highly modular architectures with complex inter-contract call chains break taint propagation. For instance, in Mahalend [21], taint paths originating in Pool.sol cannot be traced to sinks deep within library contracts accessed via delegatecall. The LLM-based reasoning stage introduces additional false negatives by incorrectly discarding valid vulnerability paths for two primary reasons. First, overly strict filtering based on access control leads the LLM to dismiss paths originating from privileged functions, such as \_setComptroller or \_reduceReserves in GoodCompound [15]. Second, insuficient reasoning about complex economic logic causes the LLM to default to assuming that syntactically correct financial formulas are economically sound, failing to identify subtle vulnerabilities like ImperVexV3 [17].

False Positives. The majority of false positives generated by PMDetector stem from the LLM-based reasoning stages, mainly due to two reasons. First, the framework misclassifies non-price-manipulation bugs as its target vulnerability class. For instance, in Inverse [18], PMDetector flags a critical typo (add996 instead of add96) that causes transfers to fail. This is a Denial of Service (DoS) vulnerability; however, the LLM-generated explanation attempts to frame it as a price manipulation attack. Second, the LLM inadequately assesses the role of access control mechanisms. The Path Filtering stage is designed to evaluate defensive measures, yet it can still fail to recognize that restricting a function to a trusted owner mitigates public attacks. The Attack Simulation stage then proceeds to construct narratives that assume the attacker has gained privileged access, thus leading to false positives.

Answer to RQ1: PMDetector is efective at detecting price manipulation vulnerabilities, achieving a recall of up to 88% (with Gemini2.5-flash) and a precision of up to 100% (with GPT-4.1). GPT-4.1 achieves the best overall performance with an F1-score of 0.91.

## 5.3 RQ2: Comparison with SOTA

Baselines. We evaluate our method against two state-of-the-art tools: DeFiTainter [48] (static analysis) and GPTScan [59] (LLM-based analysis). We also implement Pure-CoT, which provides LLMs with only contract source code, task description, and chain-of-thought prompts without additional domain-specific enhancements. We exclude on-chain approaches [62, 67, 69, 71] requiring blockchain transaction data, attack synthesis approaches [33] targeting attack contracts, and closed-source tools [40, 63, 68] to maintain evaluation fairness. DeFiTainter analyzed 60 cases (13 failed due to compilation errors). GPTScan requires single consolidated files; we successfully flattened 68 out of 73 contract Manuscript submitted to ACM

Table 3. Performance comparison with existing tools

<table><tr><td>Tool</td><td>#TP</td><td>#FN</td><td>Recall</td><td>#FP</td><td>Precision</td><td>F1-Score</td></tr><tr><td>DeFiTainter [48]</td><td>4</td><td>56</td><td>0.06</td><td>0</td><td>1.00</td><td>0.13</td></tr><tr><td>GPTScan [59] $^{1}$ </td><td>34</td><td>34</td><td>0.50</td><td>44</td><td>0.44</td><td>0.47</td></tr><tr><td>Pure-CoT $^{1}$ </td><td>40</td><td>28</td><td>0.59</td><td>10</td><td>0.80</td><td>0.68</td></tr><tr><td>PMDetector $^{1}$ </td><td>57</td><td>11</td><td>0.84</td><td>0</td><td>1.00</td><td>0.91</td></tr></table>

<sup>1</sup> We use GPT-4.1 [16] as the base model for GPTScan, Pure-CoT, and PMDetector.

![](images/e6847de1a5bb8c8e1aadcfef9f88580d048f8f675271351770077ed23d9a6f47.jpg)  
Fig. 6. Ablation result of PMDetector.

projects to meet this requirement. To ensure fair comparison across all LLM-based approaches, we use GPT-4.1 [16] as the base language model for GPTScan, Pure-CoT, and PMDetector.

Overall Results. Table 3 presents the comparison results with all baseline tools. DeFiTainter achieves zero false positives with pre-defined patterns but low recall (0.06) due to rigidity, resulting in an F1-score of 0.13. GPTScan improves significantly with 0.50 recall and 0.47 F1-score, but low precision (0.44) yields 44 false positives. This suggests that its predefined scenarios are too general and its static confirmation phase is insuficient to filter out the LLM’s incorrect reasoning. Pure-CoT surpasses GPTScan with 0.59 recall and 0.80 precision through step-by-step reasoning, but generates 10 false positives likely due to “hallucinating” infeasible attack paths. Besides, it generates 28 false negatives, which suggests that without domain-specific guidance, the LLM can still fail to comprehend highly complex or subtle vulnerability logic. The results demonstrate that PMDetector significantly outperforms both traditional static tools and LLM-based methods, demonstrating its efectiveness in detecting price manipulation vulnerabilities.

Answer to RQ2: PMDetector demonstrates superior performance compared to existing static analysis tools and LLM-based methods.

## 5.4 RQ3: Ablation Study

To investigate the contribution of the core components of PMDetector, we compare the performance of PMDetector and its variants. We create four variants of PMDetector, i.e., (a) PMDetector-NoFP, which omits the LLM-based path filtering stage; (b) PMDetector-NoAS, which omits the LLM-based attack simulation stage; (c) PMDetector-NoLLM, which removes all LLM modules, including both path filtering and attack simulation; (d) PMDetector-NoSC, which skips the final semantic sanity checker that validates high-risk taint paths. Gemini2.5-flash [14] is the base model in this RQ.

Table 4. Running time and financial cost of PMDetector.

<table><tr><td>Model</td><td>Time (s)</td><td>Avg. Time (s)</td><td>Avg. In</td><td>Avg. Out</td><td>Avg. Cost (USD)</td></tr><tr><td>Gemini2.5-flash</td><td>3067.6</td><td>9.7</td><td>11,325.0</td><td>1342.0</td><td>0.016</td></tr><tr><td>GPT-4.1</td><td>1398.4</td><td>4.0</td><td>10,834.8</td><td>1044.3</td><td>0.030</td></tr><tr><td>Qwen3-235B-A22B</td><td>1734.9</td><td>4.9</td><td>10,829.1</td><td>847.6</td><td>0.002</td></tr></table>

Overall Results. The performance of the variants is shown in Figure 6. Removing the LLM path filtering stage causes precision to drop from 0.90 to 0.40 due to an over-approximate static taint analysis that generates many false positives. Without the LLM’s pruning ability, the system is overwhelmed by these paths. Removing the LLM attack simulation stage leads to a recall drop from 0.88 to 0.59, highlighting its importance in confirming true vulnerabilities by simulating attacks on high-risk paths. Removing both LLM modules results in the worst performance, with the F1-score dropping to 0.52. Omitting the sanity checker causes a moderate drop in precision (from 0.90 to 0.83) and F1-score (from 0.90 to 0.86), but recall remains unchanged at 0.88, showing that the sanity checker filters false positives without discarding true positives. These results confirm that both LLM modules and the sanity checker are crucial for efective vulnerability detection.

Answer to RQ3: The ablation study confirms that all components of PMDetector are essential for optimal performance.

## 5.5 RQ4: Execution Cost

In this RQ, we evaluate the execution cost of PMDetector across diferent LLMs. Following existing practices [59], we use tiktoken [25], a tokenization tool from OpenAI, to estimate the token counts for all models. We measure several key metrics: (a) Time: total execution time between issuing a request and receiving the result, (b) Avg. Time, average execution time per contract, (c) Avg. In and Avg.Out, average number of input and output tokens consumed per contract, respectively, and (d) Avg. Cost, average financial cost to identify one price manipulation vulnerability .

Overall Results. As shown in Table 4, there are significant diferences in performance and cost. In terms of speed, PMDetector powered by GPT-4.1 is the most time-eficient, requiring only 4.0 seconds per contract on average. Qwen3- 235B-A22B is also eficient, with an average time of 4.9 seconds. In contrast, Gemini2.5-flash is the slowest, taking 9.7 seconds per contract. Regarding financial cost, Qwen3-235B-A22B is the most economical, costing a mere \$0.002 pe vulnerability. This is substantially lower than both Gemini2.5-flash (\$0.016) and GPT-4.1 (\$0.030). We observe that token consumption (Avg. In and Avg. Out) is relatively stable across models, indicating that the cost diference stems primarily from the pricing structures of the API services rather than the verbosity of the models’ outputs. Compared with human audits, which typically cost \$5,000–\$15,000 [30], all LLMs are dramatically more afordable. These findings highlight a clear trade-of between speed and cost. While GPT-4.1 ofers the fastest analysis, Qwen3-235B-A22B provides a practica and scalable solution, achieving comparable speed at a fraction of the financial cost.

Answer to RQ4: Compared to traditional human audits, which average roughly \$10,000, PMDetector ofers a substantially more eficient and cost-efective alternative (\$0.030 per vulnerability using GPT-4.1). Overall performance varies with the selected LLM back end.

![](images/e37372f0dca88876b5c4df1fec1e5c796da60df2451ca3eac88950c0508caafb.jpg)  
Fig. 7. A zero-day price manipulation vulnerability.

## 6 Discussion

## 6.1 Case Study

We demonstrate practical usage of PMDetector by analyzing a real-world vulnerable contract (Listing 7, with identifiers anonymized for privacy). The contract facilitates token swaps using an external AMM as a price oracle. The vulnerability stems from the contract’s reliance on the AMM’s spot price, which is calculated from live, manipulable token reserves (ORACLE.getReserves() in the \_cal\_swap\_out function). An attacker can exploit this by: (1) securing a flash loan to artificially skew the AMM’s reserves, manipulating the price, and (2) immediately calling the executeSwap function, which reads the distorted price and executes a swap at an extremely favorable rate. We have submitted this case to an auditing platform for validation.

## 6.2 Threats To Validity

Reproducibility. LLM behavior is not fully deterministic. Although we employ structured prompts and chain-ofthought reasoning to guide the model, variations in version, internal state, or prompt changes could afect reproducibility. To support reproducibility, we have released our dataset, source code, and relevant materials in our GitHub repository. Training Data Contamination. The LLM may have encountered vulnerabilities from our evaluation dataset during training, potentially inflating performance metrics. To mitigate this, we tested a baseline Pure-COT that provided a naive CoT prompt without our framework’s structural guidance. This baseline achieved only 0.59 recall and 0.68 F1-score, demonstrating that even with potential prior exposure, the LLM cannot reliably identify vulnerabilities. We further validated our approach’s efectiveness by using PMDetector to discover previously unknown vulnerabilities, several of which were submitted to auditing platforms for confirmation.

Scope and Extensibility. Our framework targets flash-loan-based price manipulation attacks, focusing on the most prevalent and financially devastating attack vectors in DeFi. However, the core workflow of PMDetector is extensible. The key idea is the hybrid methodology, which leverages static taint analysis to identify potentially vulnerable data flows and employs LLMs to reason about their complex economic exploitability, followed by a final validation step. This pipeline can be extended to other smart contract vulnerabilities requiring deep semantic understanding.

Detection Limitations. Our taint analysis relies on common smart contract patterns. Unconventional or obfuscated contracts may evade detection. Our analysis focuses on direct profit extraction through asset transfers and ledger corruption, potentially missing subtle exploitation mechanisms. The semantic sanity checker and the checklist in the LLM path filtering stage also rely on heuristics for defensive patterns, potentially missing novel defenses (false Manuscript submitted to ACM positives) or approving flawed implementations. We mitigate this through rules based on established security practices like Checks-Efects-Interactions [7].

Exploit Generation. PMDetector provides detailed vulnerability reports but does not generate runnable exploit scripts. This limitation stems from the complexity of price manipulation attacks, which depend heavily on the dynamic on-chain state that is dificult to model statically. Exploiting such vulnerabilities requires prerequisites like precise knowledge of external liquidity pool reserves and real-time oracle prices. Future work could integrate PMDetector with mainne forking environments to provide a realistic on-chain context (token prices, pool balances) needed for symbolic execution or fuzzing of the transaction sequences.

## 7 Related Work

## 7.1 Smart Contract Vulnerability Detection

The automated detection of vulnerabilities in smart contracts has been extensively studied, with approaches broadly categorized into static, dynamic, and machine learning-based methods. Static analysis techniques [39, 51, 57, 60, 73] inspect source code or bytecode without execution, examining code structure and semantics to identify potentia vulnerabilities. Dynamic analysis techniques [37, 45, 54, 56] execute smart contracts to observe runtime behavior, monitoring execution paths and state changes to detect vulnerabilities. Machine learning-based methods [35, 42, 50, 64, 72] leverage various feature representations, such as bytecode and data-flow dependencies, to train models that can generalize across diferent contract implementations. The recent success of LLMs in code understanding has opened new avenues for smart contract security analysis [32, 53, 59, 66]. GPTScan [59] combines LLM code analysis capabilities with traditional static analysis to confirm potential vulnerabilities. LLM-SmartAudit [66] introduces a conversationa framework with specialized agents that collaborate to analyze contracts. iAudit [53] employs a two-stage fine-tuning process with separate Detector and Reasoner models.

## 7.2 Price Manipulation Vulnerability Detection

Research in detecting price manipulation vulnerabilities can be categorized into three categories: static analysis, dynamic analysis, and LLM-based analysis. Static analysis tools [33, 48, 68] inspect contract code to find vulnerabilities before exploitation. DeFiTainter [48] uses inter-contract taint analysis to track manipulated price inputs. SMARTCAT [33] analyzes newly deployed bytecode in real-time to identify malicious contracts. Dynamic analysis techniques [62, 67 69, 71] analyze transaction data to detect malicious financial activity. DeFiRanger [69] constructs cash flow trees from transactions and detects attacks using predefined patterns. FORAY [67] uses a Domain-Specific Language to mode financial operations and generate attack sketches. LLM-based analysis [40, 59, 63, 75] has emerged as a promising direction. GPTScan [59] breaks down vulnerability types into code-level scenarios and properties and using an LLM to match them, followed by a static confirmation step. SmartInv [63] employs a Tier of Thought (ToT) prompting strategy to infer crucial security invariants.

## 8 Conclusion

In this paper, we introduced PMDetector, a novel hybrid framework that addresses the limitations of existing tools in detecting DeFi price manipulation vulnerabilities. Our approach combines static taint analysis for comprehensive path identification, a two-stage LLM pipeline for semantic reasoning and attack simulation, and a final static checker to Manuscript submitted to ACM

validate results and mitigate hallucinations. Evaluated on a comprehensive benchmark of 73 real-world vulnerable contracts, PMDetector achieves state-of-the-art performance with 88% precision and 90% recall, significantly outperforming previous static analysis and LLM-based methods. At just \$0.03 and 4.0 seconds per audit, our work demonstrates that the structured integration of static analysis techniques and large-scale language models provides a powerful, scalable, and economically viable paradigm for securing smart contracts against complex economic attacks.

## References

[1] 2023. BonqDAO Protocol Attack Incident. https://github.com/SunWeb3Sec/DeFiHackLabs/blob/main/past/2023/README.md#20230202--- bonqdao---price-oracle-manipulation

[2] 2024. The attack loss of the ZZF protocol. https://immunebytes.com/blog/list-of-crypto-hacks-in-the-month-of-march/

[3] 2024. ZZF protocol. https://bscscan.com/address/0xb7a254237e05ccca0a756f75fb78ab2df222911b

[4] 2025. 5 Common Smart Contract Vulnerabilities. https://www.hydnsec.com/blog-posts/5-common-smart-contract-vulnerabilities

[5] 2025. Binance Smart Chain (BSC). https://www.bnbchain.org/en/bnb-smart-chain

[6] 2025. Central limit order book (CLOB). https://en.wikipedia.org/wiki/Central\_limit\_order\_book

[7] 2025. Checks Efects Interactions. https://fravoll.github.io/solidity-patterns/checks\_efects\_interactions.html

[8] 2025. Constant function market maker. https://en.wikipedia.org/wiki/Constant\_function\_market\_maker

[9] 2025. Decentralized Application (DApp). https://en.wikipedia.org/wiki/Decentralized\_application

[10] 2025. Decentralized Finance. https://en.wikipedia.org/wiki/Decentralized\_finance

[11] 2025. Defihacklabs. https://defihacklabs.io

[12] 2025. Ether. https://en.wikipedia.org/wiki/Ethereum

[13] 2025. Fee on Transfer Mechanism. https://help.1inch.io/en/articles/5651059-what-is-a-fee-on-transfer-token

[14] 2025. Gemini 2.5 Flash. https://cloud.google.com/vertex-ai/generative-ai/docs/models/gemini/2-5-flash

[15] 2025. GoodCompound. https://etherscan.io/address/0x3d9819210a31b4961b30ef54be2aed79b9c9cd3b

[16] 2025. GPT-4.1. https://platform.openai.com/docs/models/gpt-4.1

[17] 2025. ImpermaxV3. https://basescan.org/address/0x5d93f216f17c225a8B5fFA34e74B7133436281eE

[18] 2025. Inverse Finance FiRM. https://etherscan.io/address/0x41d5d79431a913c4ae7d69a668ecdfe5f9dfb68

[19] 2025. Liquidity Provider. https://en.wikipedia.org/wiki/Market\_maker

[20] 2025. List of Past DeFi Incidents. https://github.com/SunWeb3Sec/DeFiHackLabs#list-of-past-defi-incidents

[21] 2025. Mahalend Protocol. https://etherscan.io/address/0xfd11aba71c06061f446ade4eec057179f19c23c4

[22] 2025. OWASP Smart Contract Top 10. https://owasp.org/www-project-smart-contract-top-10/

[23] 2025. PancakeSwap. https://pancakeswap.finance/

[25] 2025. tiktoken. https://github.com/openai/tiktoken

[24] 2025. Qwen3-235B-A22B. https://huggingface.co/Qwen/Qwen3-235B-A22B

[26] 2025. The Top 100 DeFi Hacks. https://www.halborn.com/reports/top-100-defi-hacks-2025

[27] 2025. Total Value Locked in DeFi. https://defillama.com/

[28] 2025. USDC. https://en.wikipedia.org/wiki/USDC\_(cryptocurrency)

[29] 2025. What is a flash loan? https://www.coinbase.com/learn/advanced-trading/what-is-a-flash-loan

[30] 2025. What Is a Smart Contract Audit? https://hedera.com/learning/smart-contracts/smart-contract-audit#:\~:text=How%20much%20does%20a% 20smart,and%20complexity%20of%20the%20contract.

[31] J.E. Barlett, J. Kotrlik, and C. Higgins. 2001. Organizational Research: Determining Appropriate Sample Size in Survey Research. Information Technology, Learning, and Performance Journal 19 (01 2001).

[32] Biagio Boi, Christian Esposito, and Sokjoon Lee. 2024. Smart contract vulnerability detection: The role of large language model (llm). ACM SIGAPP applied computing review 24, 2 (2024), 19–29.

[33] Ningyu He Bosi Zhang, Xiaohui Hu, Kai Ma, and Haoyu Wang. 2025. Following Devils’ Footprint: Towards Real-time Detection of Price Manipulation Attacks. (2025).

[34] Jiuyang Bu, Wenkai Li, Zongwei Li, Zeng Zhang, and Xiaoqi Li. 2025. Enhancing smart contract vulnerability detection in dapps leveraging fine-tuned llm. arXiv preprint arXiv:2504.05006 (2025).

[35] Jie Cai, Bin Li, Jiale Zhang, Xiaobing Sun, and Bing Chen. 2023. Combine sliced joint graph with graph neural networks for smart contract vulnerability detection. Journal ofSystems and Software 195 (2023), 111550

[36] Zhiyang Chen, Sidi Mohamed Beillahi, and Fan Long. 2024. Flashsyn: Flash loan attack synthesis via counter example driven approximation. In Proceedings of the IEEE/ACM 46th International Conference on Software Engineering. 1–13.

[37] Jaeseung Choi, Doyeon Kim, Soomin Kim, Gustavo Grieco, Alex Groce, and Sang Kil Cha. 2021. Smartian: Enhancing smart contract fuzzing with static and dynamic data-flow analyses. In 2021 36th IEEE/ACM International Conference on Automated Software Engineering (ASE). IEEE, 227–239.

Manuscript submitted to ACM

[38] Isaac David, Liyi Zhou, Kaihua Qin, Dawn Song, Lorenzo Cavallaro, and Arthur Gervais. 2023. Do you still need a manual smart contract audit? arXiv:2306.12338 [cs.CR] https://arxiv.org/abs/2306.12338

[39] Josselin Feist, Gustavo Grieco, and Alex Groce. 2019. Slither: a static analysis framework for smart contracts. In 2019 IEEE/ACM 2nd International Workshop on Emerging Trends in Software Engineering for Blockchain (WETSEB). IEEE, 8–15.

[40] Bo Gao, Yuan Wang, Qingsong Wei, Yong Liu, Rick Siow Mong Goh, and David Lo. 2025. AiRacleX: Automated Detection of Price Oracle Manipulations via LLM-Driven Knowledge Mining and Prompt Generation. arXiv:2502.06348 [cs.CR] https://arxiv.org/abs/2502.06348

[41] Bo Gao, Qingsong Wei, Yong Liu, and Rick Siow Mong Goh. 2024. Unveiling the potential of chatgpt in detecting machine unauditable bugs in smar contracts: A preliminary evaluation and categorization. In 2024 IEEE Conference on Artificial Intelligence (CAI). IEEE, 1481–1486

[42] Zhipeng Gao, Vinoj Jayasundara, Lingxiao Jiang, Xin Xia, David Lo, and John Grundy. 2019. Smartembed: A tool for clone and bug detection in smart contracts through structural code embedding. In 2019 IEEE International Conference on Software Maintenance and Evolution (ICSME). IEEE, 394–397.

[43] Sihao Hu, Tiansheng Huang, Fatih İlhan, Selim Furkan Tekin, and Ling Liu. 2023. Large language model-powered smart contract vulnerability detection: New perspectives. In 2023 5th IEEE International Conference on Trust, Privacy and Security in Intelligent Systems and Applications (TPS-ISA). IEEE, 297–306.

[44] Johannes Rude Jensen, Victor von Wachter, and Omri Ross. 2021. An introduction to decentralized finance (defi). Complex Systems Informatics and Modeling Quarterly 26 (2021), 46–54.

[45] Bo Jiang, Ye Liu, and Wing Kwong Chan. 2018. Contractfuzzer: Fuzzing smart contracts for vulnerability detection. In Proceedings of the 33rd ACM/IEEE international conference on automated software engineering. 259–269.

[46] Juyong Jiang, Fan Wang, Jiasi Shen, Sungju Kim, and Sunghun Kim. 2024. A survey on large language models for code generation. arXiv preprint arXiv:2406.00515 (2024).

[47] Haolin Jin, Linghan Huang, Haipeng Cai, Jun Yan, Bo Li, and Huaming Chen. 2024. From llms to llm-based agents for software engineering: A survey of current, challenges and future. arXiv preprint arXiv:2408.02479 (2024).

[48] Queping Kong, Jiachi Chen, Yanlin Wang, Zigui Jiang, and Zibin Zheng. 2023. Defitainter: Detecting price manipulation vulnerabilities in defi protocols. In Proceedings ofthe 32nd ACM SIGSOFT International Symposium on Software Testing and Analysis. 1144–1156.

[49] Haonan Li, Yu Hao, Yizhuo Zhai, and Zhiyun Qian. 2024. Enhancing static analysis for practical bug detection: An llm-integrated approach Proceedings ofthe ACM on Programming Languages 8, OOPSLA1 (2024), 474–499.

[50] Zhenguang Liu, Peng Qian, Xiaoyang Wang, Yuan Zhuang, Lin Qiu, and Xun Wang. 2021. Combining graph neural networks with expert knowledg for smart contract vulnerability detection. IEEE Transactions on Knowledge and Data Engineering 35, 2 (2021), 1296–1310.

[51] Loi Luu, Duc-Hiep Chu, Hrishi Olickel, Prateek Saxena, and Aquinas Hobor. 2016. Making smart contracts smarter. In Proceedings of the 2016 ACM SIGSAC conference on computer and communications security. 254–269.

[52] Wei Ma, Shangqing Liu, Zhihao Lin, Wenhan Wang, Qiang Hu, Ye Liu, Cen Zhang, Liming Nie, Li Li, and Yang Liu. 2023. Lms: Understanding cod syntax and semantics for code analysis. arXiv preprint arXiv:2305.12138.

[53] Wei Ma, Daoyuan Wu, Yuqiang Sun, Tianwen Wang, Shangqing Liu, Jian Zhang, Yue Xue, and Yang Liu. 2024. Combining fine-tuning and llm-based agents for intuitive smart contract auditing with justifications. arXiv preprint arXiv:2403.16073 (2024)

[54] Mark Mossberg, Felipe Manzano, Eric Hennenfent, Alex Groce, Gustavo Grieco, Josselin Feist, Trent Brunson, and Artem Dinaburg. 2019. Manticore: A user-friendly symbolic execution framework for binaries and smart contracts. In 2019 34th IEEE/ACM International Conference on Automated Software Engineering (ASE). IEEE, 1186–1189.

[55] Daye Nam, Andrew Macvean, Vincent Hellendoorn, Bogdan Vasilescu, and Brad Myers. 2024. Using an llm to help with code understanding. In Proceedings of the IEEE/ACM 46th International Conference on Software Engineering. 1–13.

[56] Tai D Nguyen, Long H Pham, Jun Sun, Yun Lin, and Quang Tran Minh. 2020. sfuzz: An eficient adaptive fuzzer for solidity smart contracts. In Proceedings of the ACM/IEEE 42nd international conference on software engineering. 778–788.

[57] Sunbeom So, Myungho Lee, Jisu Park, Heejo Lee, and Hakjoo Oh. 2020. Verismart: A highly precise safety verifier for ethereum smart contracts. In 2020 IEEE Symposium on Security and Privacy (SP). IEEE, 1678–1694.

[58] Gaurang Sriramanan, Siddhant Bharti, Vinu Sankar Sadasivan, Shoumik Saha, Priyatham Kattakinda, and Soheil Feizi. 2024. Llm-check: Investigating detection of hallucinations in large language models. Advances in Neural Information Processing Systems 37 (2024), 34188–34216.

[59] Yuqiang Sun, Daoyuan Wu, Yue Xue, Han Liu, Haijun Wang, Zhengzi Xu, Xiaofei Xie, and Yang Liu. 2024. Gptscan: Detecting logic vulnerabilitie in smart contracts by combining gpt with program analysis. In Proceedings of the IEEE/ACM 46th International Conference on Software Engineering. 1–13.

[60] Sergei Tikhomirov, Ekaterina Voskresenskaya, Ivan Ivanitskiy, Ramil Takhaviev, Evgeny Marchenko, and Yaroslav Alexandrov. 2018. Smartcheck: Static analysis of ethereum smart contracts. In Proceedings ofthe 1st international workshop on emerging trends in software engineering for blockchain. 9–16.

[61] Ashish Vaswani, Noam Shazeer, Niki Parmar, Jakob Uszkoreit, Llion Jones, Aidan N Gomez, Łukasz Kaiser, and Illia Polosukhin. 2017. Attention i all you need. Advances in neural information processing systems 30 (2017)

[62] Dabao Wang, Bang Wu, Xingliang Yuan, Lei Wu, Yajin Zhou, and Helei Cui. 2024. Defiguard: A price manipulation detection service in defi using graph neural networks. IEEE Transactions on Services Computing (2024).

[63] Sally Junsong Wang, Kexin Pei, and Junfeng Yang. 2024. Smartinv: Multimodal learning for smart contract invariant inference. In 2024 IEEE Symposium on Security and Privacy (SP). IEEE, 2217–2235.

Manuscript submitted to ACM

[64] Wei Wang, Jingjing Song, Guangquan Xu, Yidong Li, Hao Wang, and Chunhua Su. 2020. Contractward: Automated vulnerability detection models for ethereum smart contracts. IEEE Transactions on Network Science and Engineering 8, 2 (2020), 1133–1144.

[65] Zhiyuan Wei, Jing Sun, Yuqiang Sun, Ye Liu, Daoyuan Wu, Zijian Zhang, Xianhao Zhang, Meng Li, Yang Liu, Chunmiao Li, et al. 2025. Advanced Smart Contract Vulnerability Detection via LLM-Powered Multi-Agent Systems. IEEE Transactions on Software Engineering

[66] Zhiyuan Wei, Jing Sun, Zijiang Zhang, Xianhao Zhang, Meng Li, and Zhe Hou. 2024. Llm-smartaudit: Advanced smart contract vulnerability detection. arXiv preprint arXiv:2410.09381 (2024)

[67] Hongbo Wen, Hanzhi Liu, Jiaxin Song, Yanju Chen, Wenbo Guo, and Yu Feng. 2024. FORAY: Towards Efective Attack Synthesis against Deep Logical Vulnerabilities in DeFi Protocols. In Proceedings ofthe 2024 on ACM SIGSAC Conference on Computer and Communications Security. 1001–1015.

[68] Ka Wai Wu. 2024. Strengthening DeFi Security: A Static Analysis Approach to Flash Loan Vulnerabilities. arXiv preprint arXiv:2411.01230 (2024)

[69] Siwei Wu, Zhou Yu, Dabao Wang, Yajin Zhou, Lei Wu, Haoyu Wang, and Xingliang Yuan. 2023. Defiranger: detecting DeFI price manipulation attacks. IEEE Transactions on Dependable and Secure Computing 21, 4 (2023), 4147–4161.

[70] Yin Wu, Xiaofei Xie, Chenyang Peng, Dijun Liu, Hao Wu, Ming Fan, Ting Liu, and Haijun Wang. 2024. Advscanner: Generating adversarial smart contracts to exploit reentrancy vulnerabilities using llm and static analysis. In Proceedings of the 39th IEEE/ACM International Conference on Automated Software Engineering. 1019–1031.

[71] Maoyi Xie, Ming Hu, Ziqiao Kong, Cen Zhang, Yebo Feng, Haijun Wang, Yue Xue, Hao Zhang, Ye Liu, and Yang Liu. 2024. DeFort: Automatic Detection and Analysis of Price Manipulation Attacks in DeFi Applications. In Proceedings ofthe 33rd ACM SIGSOFT International Symposium on Software Testing and Analysis. 402–414

[72] Yingjie Xu, Gengran Hu, Lin You, and Chengtang Cao. 2021. A novel machine learning-based analysis model for smart contract vulnerability Security and Communication Networks 2021, 1 (2021), 5798033.

[73] Yinxing Xue, Mingliang Ma, Yun Lin, Yulei Sui, Jiaming Ye, and Tianyong Peng. 2020. Cross-contract static analysis for detecting practical reentrancy vulnerabilities in smart contracts. In Proceedings ofthe 35th IEEE/ACM International Conference on Automated Software Engineering. 1029–1040.

[74] Zhuo Zhang, Brian Zhang, Wen Xu, and Zhiqiang Lin. 2023. Demystifying exploitable bugs in smart contracts. In 2023 IEEE/ACM 45th International Conference on Software Engineering (ICSE). IEEE, 615–627.

[75] Juantao Zhong, Daoyuan Wu, Ye Liu, Maoyi Xie, Yang Liu, Yi Li, and Ning Liu. 2025. DeFiScope: Detecting Various DeFi Price Manipulations with LLM Reasoning. arXiv preprint arXiv:2502.11521 (2025).