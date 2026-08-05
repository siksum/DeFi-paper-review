# Revealing Adversarial Smart Contracts through Semantic Interpretation and Uncertainty Estimation

Yating Liu, Xing Su, Hao Wu, Sijin Li, Yuxi Cheng, Fengyuan Xu<sup>∗</sup>, Sheng Zhong National Key Lab for Novel Software Technology, Nanjing University, Nanjing, Jiangsu, China Emails: {yatingliu, xingsu, sijinli, yuxicheng}@smail.nju.edu.cn, {hao.wu, fengyuan.xu, zhongsheng}@nju.edu.cn

Abstract—Adversarial smart contracts, mostly on EVMcompatible chains like Ethereum and BSC, are deployed as EVM bytecode to exploit vulnerable smart contracts for financial gain. Detecting such malicious contracts at the time of deployment is an important proactive strategy to prevent losses from victim contracts. It offers a better cost-benefit ratio than detecting vulnerabilities on diverse potential victims. However, existing works are not generic with limited detection types and effectiveness due to imbalanced samples, while the emerging LLM technologies, which show their potential in generalization, have two key problems impeding its application in this task: hard digestion of compiled-code inputs, especially those with task-specific logic, and hard assessment of LLM’s certainty in its binary (yes-or-no) answers. Therefore, we propose a generic adversarial smart contracts detection framework FinDet, which leverages LLM with two enhancements addressing the above two problems. FinDet takes as input only the EVM bytecode contracts and identifies adversarial ones among them with high balanced accuracy. The first enhancement extracts concise semantic intentions and highlevel behavioral logic from the low-level bytecode inputs, unleashing the LLM reasoning capability restricted by the task input. The second enhancement probes and measures the LLM uncertainty to its multi-round answering to the same query, improving the LLM answering robustness for binary classifications required by the task output. Our comprehensive evaluation shows that FinDet achieves a BAC of 0.9374 and a TPR of 0.9231, significantly outperforming existing baselines. It remains robust under challenging conditions including unseen attack patterns, low-data settings, and feature obfuscation. FinDet detects all 5 public and 20+ unreported adversarial contracts in a 10-day real-world test, confirmed manually.

## 1. Introduction

While blockchain technology is advancing rapidly, its decentralized and anonymous nature also creates opportunities for malicious activities. In 2024, security incidents led to losses exceeding \$2.01 billion [1]. To protect the ecosystem, prior research mainly focused on detecting vulnerable smart contracts. However, many reported vulnerabilities are ultimately unexploitable [2]. Moreover, defenders face an inherent asymmetry, as they must secure against all threats, whereas attackers need only a single successful exploit. Other approaches monitor adversarial transactions in public mempools, but they cannot observe transactions in private pools and can only analyze malicious behavior after it has been broadcast or included on-chain.

![](images/0db956655ec4e8c61330812abc5ec44168cc50fadcac6bddfc0c928288418cba.jpg)  
Figure 1: Comparison of adversarial contract detection methods: (a) Rule-based methods rely on fixed patterns and fail on unexpected attacks. (b) ML-based methods leverage low-level features but struggle with few-shot or unseen cases, resulting in high FNR. (c) FinDet employs semantic reasoning for robust adversarial detection.

To address these limitations, recent research efforts have shifted toward adversarial contract detection. Adversarial contracts are deliberately crafted smart contracts that exploit known vulnerabilities or manipulate protocol logic for illicit gain. These approaches focus on contracts that are both exploitable and likely to be deployed in real attacks. By identifying these high-impact threats ahead of execution, defenders can intervene before damage occurs and significantly reduce the burden of manual auditing.

Most adversarial contracts exist only as bytecode to conceal their malicious intent. As shown in Figure 1, rule-based methods [3], [4] detect adversarial behaviors by disassembling bytecode and extracting control and data flows, which are then analyzed using manually defined heuristics and rule-based logic. However, these methods are effective only for single attack types and lack the flexibility to generalize beyond predefined patterns. To enable detection of multiple attack types, Machine learning (ML)-based methods [5], [6] leverage low-level intermediate representation (IR) and onchain features to train detection models. However, these methods suffer from high false negative rates, reaching 35.16% for Lookahead [5] and 18.68% for Skyeye [6].

Moreover, they face two fundamental limitations: limited generalization ability, which impedes the detection of unseen attack patterns as adversarial behaviors continuously evolve beyond the training data, and a strong dependence on large amounts of labeled data, which is often scarce and costly, resulting in significant performance degradation when training samples are insufficient.

![](images/f85b50aa04baabc5ef3fa93eaa4495bcd421be1cd8c41856c1aad0f8707cc7e8.jpg)

![](images/f552108146539fbecd57b05b08024ce36b31f144d1e4daf65623220b3dc2cd6e.jpg)  
Figure 2: Three-phase adver- Figure 3: Token distribution sarial lifecycle. of TAC on DeepSeek-V3.

Encouragingly, large language models (LLMs) have demonstrated tremendous potential in understanding advanced code representations (e.g., source code), exhibiting strong capabilities in capturing behavioral semantics and interpreting smart contract logic. These strengths make LLMs a promising alternative for detecting adversarial smart contracts, especially in scenarios where labeled training data is scarce and costly to obtain. However, this approach faces several key challenges:

• C1: Limited Understanding of Low-Level Intermediate Representations. LLMs are primarily pretrained on high-level source code, and thus lack the capability to accurately understand the low-level semantics of EVM bytecode. One alternative approach is to lift bytecode into IR, such as using Elipmoc [7] to transform bytecode into three-address code (TAC) for improved interpretability. However, as shown in Figure 3, 87.02% of TAC representations exceed typical LLM input limits, leading to frequent detection failures. Fine-tuning LLMs directly on bytecode is possible but costly and limited by scarce labeled data.

• C2: Diverse and Evolving Attack Strategies. Adversarial contracts employ a wide range of attack methods that continuously evolve over time. Relying solely on superficial features of previously known attacks for decision-making limits the ability to detect unseen and emerging attack types effectively.

• C3: Hallucination and Challenging Uncertainty Assessment. LLMs tend to lose focus when processing complex logic, often resulting in high false positives due to conservative decisions under uncertainty [8]; since most mainstream LLMs are closed-source and operate as black-box systems, detecting and quantifying their hallucinations is particularly challenging and reduces detection reliability.

In this work, to leverage the capabilities of LLMs while addressing the above challenges, we propose FinDet, a generic adversarial smart contract detection method that enhances high-level behavioral semantics and enables quantified assessment of LLM uncertainty. FinDet relies solely on bytecode thus enabling early detection during the predeployment phase, and is also capable of robustly identifying previously unseen attack patterns. It operates in two stages. In Stage I, the bytecode is lifted into a semistructured natural language (NL) description, based on which we perform general-purpose analysis from the perspective of the contract’s overall semantics, and attackspecific analysis grounded in the semantics of operational logic. In Stage II, we conduct fine-grained uncertainty probing and assessment. By leveraging the multi-view semantic information obtained in Stage I through targeted questions and performing reliable fusion, FinDet achieves robust detection results.

To address C1, FinDet adopts a bytecode lifting mechanism that translates raw bytecode into semi-structured NL descriptions. This process preserves critical semantic information while enhancing interpretability. Unlike traditional intermediate-representation-level static analyses that struggle to capture subtle contract logic, this semantic elevation enables more accurate and robust detection of adversarial behaviors. C2 stems from the complexity and rapid evolution of adversarial attack strategies, making generic detection difficult. To tackle this, we analyzed many adversarial cases and identified a fundamental three-stage operational pattern for attacker-deployed adversarial contracts aimed at profit extraction: ① Preparation, ② Exploitation, and ③ Exit (see Figure 2 and Obs2 in §3.3). FinDet systematically analyzes fund-flow reachability across these stages, capturing core operational logic that differentiates such adversarial contracts from benign ones. As to C3, we first transform the yes-or-no detection query posed to LLMs into a task of assigning probabilities across four levels of uncertainty. We then repeat the same task with disturbance prompts to assess how confident the LLMs are. The uncertainty in their responses is quantified by the entropy of the resulting probability distributions [9], which is further used to derive the final yes-or-no answer with high confidence.

In our experiments, FinDet achieved a state-of-the-art balanced accuracy (BAC) of 0.9374, surpassing the baselines Lookahead and Skyeye by 13.83% and 3.59%, respectively. It also attained a recall of 0.9231 on the adversarial class, outperforming Lookahead (42.37%) and Skyeye (13.51%), while maintaining an average cost of \$0.003 per contract. Notably, FinDet consistently demonstrated robust perfor mance under challenging scenarios, including unseen attack patterns, insufficient training data, and on-chain feature obfuscation, where baseline methods suffered significant performance degradation. During a 10-day real-world detection, FinDet successfully identified 29 adversarial contracts, among which 5 were previously undisclosed and exhibited clear adversarial intent. To support systematic evaluation and future research, we further curated the first and the largest dataset of adversarial contracts with normalized categories, including 455 adversarial and 20,000 benign samples, of which 200 adversarial contracts are labeled with normalized attack types.

We summarize our contributions as follows:

• We propose FinDet, the first generic and training-free detection framework that leverages the semantic understanding capabilities of LLMs to identify adversarial smart contracts directly from EVM bytecode, enabling detection in the pre-deployment phase. FinDet demonstrates robust performance under challenging scenarios such as unseen attack patterns and insufficient training data, and can seamlessly work with different LLMs without requiring fine-tuning.

• We enhance semantic understanding by integrating fund-flow reachability analysis and multi-view semantic reasoning to capture fundamental adversarial patterns, both based on the lifted semi-structured NL descriptions derived from bytecode.

• We design a novel quantitative uncertainty assessment of LLM outputs by extending labels to a fine-grained scale, incorporating multiple probing prompts and applying entropy-based fusion to produce reliable uncertainty estimates.

• FinDet achieved state-of-the-art performance with a BAC of 0.9374 (surpassing the baselines by 13.83% and 3.59%) and an adversarial recall (i.e., TPR) of 0.9231 (exceeding the baselines by 42.37% and 13.51%). Furthermore, FinDet identified 29 previously undisclosed adversarial contracts in real-world.

## 2. Background

## 2.1. Decentralized Finance

Blockchain provides a decentralized, peer-to-peer, and verifiable infrastructure that records and secures transactions through distributed consensus while preserving user anonymity. Built atop this foundation, Decentralized Finance (DeFi) enables permissionless and composable financial services through smart contracts. Its core elements are accounts and transactions, which define asset ownership and drive contract execution.

Accounts. Ethereum supports two account types: Externally Owned Accounts (EOAs) controlled by private keys, and smart contract accounts governed by deployed code.

Transactions. Transactions underpin DeFi execution. EOAs initiate external transactions, while smart contracts generate internal transactions to implement protocol logic.

## 2.2. Vulnerabilities and Adversarial Contracts

Smart contracts are autonomous programs that manage assets and enforce logic in DeFi protocols, featuring immutability, composability, and permissionless interaction. Vulnerable contracts [10] contain unintended flaws that can be exploited to cause incorrect behavior or asset loss, such as reentrancy or price manipulation. In contrast, adversarial contracts [3] are deliberately crafted malicious contracts designed to exploit known vulnerabilities or manipulate protocol logic for illicit gain.

2.3. Large Language Models for Blockchain Security

The emergence of LLMs marks a major advance in language understanding and generation. Built on Transformer architectures with billions of parameters, they exhibit strong reasoning and semantic comprehension. In blockchain, LLMs have gained attention for tasks like smart contract auditing [8], [11], on-chain transaction analysis [12], and dynamic contract analysis [13]. These applications highlight LLMs’ potential as powerful tools to enhance blockchain security.

## 3. Threat Model, Motivation and Observations

## 3.1. Threat Model, Scope, and Assumptions

FinDet targets adversarial contracts crafted to exploit on-chain vulnerabilities: attackers deploy such contracts to interact with vulnerable victim contracts, exploit their flaws, and ultimately extract illicit proceeds. We assume attackers have full access to public information including victim contract source or decompiled code and can deploy arbitrary contracts, submit transactions, and leverage private submis sion channels (e.g., Flashbots) to conceal their activity.

Attacks unrelated to contract-level vulnerabilities (e.g., private-key leaks or off-chain compromises) and those carried out by privileged users (e.g., rug pulls) are out of scope. We also exclude attacks executed solely via crafted transactions without a deployed adversarial contract.

Detection occurs pre-deployment (mempool stage), relying only on bytecode without runtime blockchain data. Our approach generalizes across variants by analyzing semantic intent rather than relying on brittle signatures or handcrafted datasets. A detection is considered successful if malicious behavioral indicators are identified prior to deployment, allowing the contract to be correctly classified as adversarial.

## 3.2. Motivation Example

Figure 4 illustrates a typical scenario of an attack via flashloans.

The blue arrows indicate the attack steps: (1) The attacker calls the exploit function, obtaining a flash loan from the DPP contract to secure initial capital; (2) A callback is triggered during the loan, invoking DPPFlashLoanCall, a function controlled by the attacker; (3-4) The attacker performs repeated token buy/sell operations to exploit price discrepancies; (5) The borrowed funds are then repaid to the lending protocol, and (6) the extracted profit is transferred to the attacker.

The yellow arrows illustrate the fund-flow throughout the attack: (a) a flash loan is issued from the DPP contract to the adversarial contract; (b) the adversary initiates a buy operation with a small input to artificially inflate the token price; (c) this is followed by a sell operation to extract profit based on the manipulated price; (d) the borrowed funds are then repaid to the DPP contract; and finally, (e) the remaining profit is transferred from the adversarial contract to the attacker’s address. This motivating example reveals typical behavioral patterns of adversarial contracts. We elaborate on these observations in the following section.

![](images/e10c76a2bed8740f3bf3d3152b6d55d64e84c46c0be9c3c57ca212b2dd57d4d4.jpg)  
Figure 4: Decompiled Solidity snippet of an adversarial contract (see BscScan link) involved in a flash loan attack on April 26, 2025. This case illustrates a typical flash loan exploit, following the three-phase fund-flow pattern. The right part visualizes the corresponding fund transfers across different addresses during the attack.

## 3.3. Observations

After identifying the core challenges, we further make the following key observations from our empirical analysis, which directly inform the design of our system.

Obs1: Long-Form Inputs Cause Loss of Contextual Focus. We find that LLMs struggle to maintain semantic coherence when presented with entire contracts as monolithic inputs. Instead of capturing the overarching behavioral intent, the model often fixates on isolated or syntactically unusual fragments, leading to erroneous reasoning. For example, as shown in Figure 16 in Appendix C.2, a benign contract was misclassified due to its intricate logic structure and dense permission checks. This observation motivates our use of structured prompting and hierarchical summarization to preserve contextual clarity and guide the model’s attention more effectively.

Obs2: Adversarial Fund-Flows Capture the Fundamental Three-Stage Attack Lifecycle. From empirical analysis of adversarial contracts, we observe that, within our scope (i.e., attacks where an attacker deploys a contract to exploit a victim contract for financial gain), the driving force is illicit fund acquisition, and the behavior can be abstracted into three essential stages (see Figure 2) that are fundamentally required to achieve the attacker’s goal: ① Preparation, where the attacker first deploys the adversarial contract and completes necessary setup (e.g., obtaining authorizations, funding, or initializing state); ② Exploitation, where the contract executes logic to leverage the victim’s vulnerabilities; and ③ Exit, where, since adversarial contracts are often deployed for one-off attacks, the illicit proceeds are typically transferred to attacker-controlled addresses .

We emphasize that these stages do not constitute a general abstraction of all blockchain attacks, but instead represent the necessary elements to achieve financial gain within the threat model of attacker-deployed adversarial contracts. Under this abstraction, fund-flow analysis, tracing incoming funds, internal transfers, and outgoing flows, naturally captures the chain-of-attack reasoning required to distinguish adversarial contracts from benign ones.

Obs3: Ambiguous Patterns Trigger Conservative Reasoning. When encountering gray-area behaviors such as redundant access checks, complex fallback logic, or reentrant fund management, LLMs tend to default to conservative classifications due to uncertainty, often leading to false positives. These behaviors may be essential for security or robustness in benign contracts but are misinterpreted as obfuscation or adversarial intent. This insight underscores the need to quantify model uncertainty, leading to our entropy-based scoring and fusion mechanism, which interprets model outputs as graded confidence signals rather than binary decisions.

## 4. Overview of FinDet

This section presents the overall design of FinDet, a general framework that leverages the strong semantic understanding capabilities of LLMs to detect adversarial smart contracts directly from EVM bytecode. FinDet supports generic detection and is capable of identifying previously unseen attack patterns. As illustrated in Figure 5, the framework consists of two main stages: high-level behavioral semantic analysis (Stage I) and uncertainty quantification via probing and fusion (Stage II). We parallelize LLM queries within Stage I and Stage II separately to improve efficiency.

In Stage I, we begin by lifting raw EVM bytecode into semantic units and transforming them into semi-structured NL descriptions through templates (§4.1) that apply tailored grammatical rules to express different semantic unit types, addressing the limited understanding of low-level intermediate representations (C1). We perform general-purpose analysis (§4.2) to understand the overall purpose of the contract and extract sensitive semantics. We also conduct attackspecific analysis (§4.3) by examining each function individually to summarize its intent, suspicious behaviors, and supporting evidence. In addition, a key component of this analysis is the fund-flow reachability analysis (§5), which targets diverse and evolving attack strategies (C2) based on the three-phase adversarial pattern shown in Figure 2. This approach traces attacker-controlled fund-related operations and uncovers potential exploitation paths.

![](images/8e6e1e3101c93ae6fbd5c2ff85097ea5c614c357ed40c87c2c4f63c664f5d85f.jpg)  
Figure 5: A high-level workflow of FinDet.

In Stage II, to address LLM hallucination and the difficulty of assessing uncertainty, especially in black-box scenarios (C3), we introduce a probing- and fusion-based method that extends binary classification into fine-grained risk assessment (§6). Leveraging normal and misleading probes with entropy-based fusion, we quantify and rank LLM uncertainty to achieve reliable detection.

## 4.1. Natural Language Description Generation

As shown in Figure 3, converting bytecode into TAC via Elipmoc [7] makes over 87% of contracts exceed typical LLM token limits, highlighting the need for more compact, high-level inputs.

Smart contracts automatically execute agreements when certain conditions are met and can thus be seen as a set of conditional behaviors, referred to as semantic units, representing behaviors triggered under specific conditions. We lift bytecode into semantically faithful intermediate representations, the semantic units, by disassembling it into a controlflow graph and extracting semantically relevant operations along each execution path. For each condition and behavior, we employ predefined templates to transform them into semantically equivalent NL descriptions. Condition templates $( { \mathrm { e . g . , } } ^ { \infty } { \mathrm { w h e n . } } \ldots { } ^ { \infty } { \mathrm { , } } ^ { \infty } \Sigma { \mathrm { o r } } \ldots { } ^ { \mathrm { , 9 } } )$ express the triggering logic, while behavior templates summarized in TABLE 8 describe the resulting actions.

Unlike LLM-based decompilation approaches [14], [15], our method adopts a precise and interpretable NL representation that does not rely on LLMs for its generation. We retain the NL representation for three reasons. First, regenerating code through LLMs may introduce hallucinations and semantic inconsistencies. As our objective is to extract and reason about semantics rather than reproduce the original source code, we instead preserve a faithful NL description. Second, code generation substantially increases token consumption and inference latency, while low-level alternatives such as TAC remain excessively verbose for long bytecode sequences. Third, NL provides a concise, highlevel abstraction that enables efficient downstream analyses. All subsequent components in our framework directly operate on this representation.

## 4.2. General-Purpose Analysis

General-purpose analysis strategy begins with understanding the contract’s overall intent and core functionality. By capturing this high-level perspective, we can more effectively interpret the contract’s design objectives and evaluate its potential security risks. To achieve this, we prompt a LLM to synthesize information across all functions and generate a comprehensive, contract-level summary, as illustrated in Task: contract\_summary in Figure 8. An example summary for the contract depicted in Figure 4 is provided in Figure 14. Furthermore, we extract several contract-level indicators from the NL descriptions, including the number and proportion of external calls, unknown functions, and botrelated functions, as well as the presence of fund-transfer behaviors (e.g., operations indicative of attempts to drain the contract balance). These metrics contribute to a holistic understanding of the contract’s structural characteristics and its potential security implications.

## 4.3. Attack-Specific Analysis

A central component of our attack-specific analysis is the fund-flow reachability analysis, which traces how attackercontrolled inputs propagate toward sensitive or high-risk operations. Given its importance, we provide a dedicated and detailed discussion in §5.

Beyond fund-flow reasoning, we further adopt a finegrained, function-level behavioral analysis that guides the LLM to examine each function’s intended purpose, suspicious behaviors, and supporting evidence, as illustrated in Task: function\_summary in Figure 8. This approach reveals subtle signs of malicious logic that may be overlooked in global-level summaries. While benign contracts typically consist of modular functions with well-defined objectives and appropriate access control, adversarial contracts often embed sensitive operations or privilege escalation within ostensibly benign functions. Accurately identifying each function’s intent is thus essential for distinguishing malicious behavior from legitimate functionality. Moreover, detecting suspicious patterns, such as unsafe external calls or attacker-controlled inputs, and extracting concrete evidence helps preserve critical indicators. As shown in Figure 13, this function-centric analysis is based on the contract presented in Figure 4 and guides the LLM to develop a detailed understanding of the contract’s behavior. We also retain the NL descriptions of functions without identified names, as these are more likely to contain malicious logic.

![](images/66984fb6c2121167ce9bf1f90bb62869bd8de5fe0425890330cb3ee37da016c5.jpg)  
Figure 6: Workflow of fund-flow reachability analysis consisting of six steps: ❶ chunking, ❷ contract forest construction, ❸ behavior node entity resolution, ❹ graph transformation & ingress/egress identification, ❺ fund-flow reachability analysis, and ❻ reverse pruning & output.

## 5. Fund-Flow Reachability Analysis

Building on the three-stage lifecycle described in Obs2, we focus on fund-flow reachability to capture the “chainof-attack” logic inherent to attacker-deployed adversarial contracts, as part of our analysis in §4.3. Adversarial contracts are fundamentally driven by illicit financial gain, which leads to deliberate manipulation of fund movements and underscores the necessity of analyzing fund-flows.

At the same time, they differ from benign contracts in how funds are routed: whereas benign contracts enforce transfers through rigid, auditable logic (e.g., owner-only withdrawals or strict conditional checks), adversarial contracts introduce flexible or hidden paths that enable unauthorized movement. These structural differences indicate the feasibility of extracting fund-flow information from the contract’s semantic representations, as such manipulations are encoded in the operations within the contract.

We construct a control dependency forest from the NL representation using grammatical rules to get a structured view of the contract and decomposes complex code into manageable structures. We then trace data flow from attacker-controlled ingress to egress that may trigger critical financial operations.

## 5.1. Control Dependency Forest Construction

As shown in Figure 6 (a-d), we propose a control dependency forest to systematically analyze the semantic structure of smart contracts. Given a NL description of a contract (a), we first perform ❶ chunking to segment it into a set of function-level descriptions (b). For each function, we ❷ construct a control dependency tree by capturing the dependencies expressed in the description. Specifically, in each tree, every sentence is treated as a node, with the function signature serving as the root. Each node is heuristically classified into one of four categories:

• Function: root node representing a function signature;

• Behavior: actions such as external calls, assignments, and delegate calls;

• Condition: control logic including if-else, require, and loops;

• Unknown: fallback nodes for unparsed sentences as a safety mechanism.

This classification is feasible because the NL descriptions are generated using a predefined template, ensuring consistent structure and phrasing. As a result, the entire contract is transformed into a control dependency forest (c).

We further ❸ refine the behavior node set by assigning each node a fine-grained behavior type (e.g., external call, assignment) according to its semantic role, as summarized in TABLE 8. For each behavior node $B _ { i } \ \in$ $\mathcal { N } _ { \mathrm { b e h a v i o r } }$ , we extract the related entities including variables v and function operations $o p ,$ , and define a propagation tuple:

$$
B _ {i} = (\mathcal {X} _ {\mathrm{src}}, x _ {\mathrm{dst}})\tag{1}
$$

where $\mathcal { X } _ { \mathrm { s r c } }$ is the set of source entities (inputs) and $x _ { \mathrm { d s t } }$ is the destination entity (output). We exclude constants such as numeric literals and hardcoded addresses. To distinguish homonymous entities across functions, we prepend the function name as a namespace to local entities (e.g., pancakecall:param1), while global entities retain their original names (e.g., msg.caller). Depending on the behavior type, either the source set or destination field in the tuple may be empty. For the other three node types, we keep their original NL descriptions to preserve full semantic context. This yields the refined forest of behavior nodes illustrated in Figure 6(d).

## 5.2. Graph Transformation

As illustrated in Figure 6 (d-e), we first perform ❹ graph transformation to convert the contract forest into a structure suitable for fund-flow reachability analysis. For each function tree, we conduct a depth-first traversal from the root node $F _ { i }$ , maintaining a set of visited entities with their associated condition sets $( e , \mathcal { C } )$ , initialized by the formal parameters of $F _ { i }$ and global variables. When visiting a behavior node $B _ { i } ,$ we extract its source entities $\chi _ { \mathrm { s r c } } ^ { ( i ) }$ and destination entity $x _ { \mathrm { d s t } } ^ { ( i ) }$ . If a source $x _ { j } \in \mathcal { X } _ { \mathrm { s r c } } ^ { ( i ) }$ has been visited and $x _ { \mathrm { d s t } } ^ { ( i ) }$ is new, we connect them with an edge annotated by the combined conditions accumulated along the traversal path:

$$
x _ {j} \xrightarrow {\mathcal {C}} x _ {\mathrm{dst}} ^ {(i)}
$$

We then mark $x _ { \mathrm { d s t } } ^ { ( i ) }$ as visited with the corresponding C.

To ensure cross-function consistency, local variables are prefixed with their function names, while global variables retain their original identifiers, as described before. Entities sharing the same normalized name across functions are linked to form inter-function edges (e.g., variable v3 in F1 and F2 in Figure 6 (e)). Behavior nodes without resolvable entities are discarded.

This process yields a transformed graph where each node (except the root) represents an entity, either a variable or an operation, and all condition nodes are encoded as edge annotations, as shown in Figure 6 (e).

## 5.3. Ingress and Egress Identification

To enable fund-flow reasoning in adversarial contract analysis, we first identify potential entry and exit points of value-related information within contracts, referred to as ingress and egress. These anchors, shown in Figure 6 (eg), enable us to capture how valuable attacker-controlled inputs propagate toward risky operations, forming the foundation for modeling a contract’s fund-flow logic and driving subsequent reachability analysis. We construct initial sets of ingress and egress reflecting real-world attack semantics, corresponding to the three phases in Figure 2 and detailed in Listing 1.

Ingress points represent attacker-controllable or valuable data that may trigger value or permission flows within a contract. Typical examples include the function caller (msg.sender), the amount of Ether attached to a call (msg.value), the transaction initiator (tx.origin), and the contract’s current balance (address(this).balance). We also treat function parameters as contextual ingress points, as they frequently carry user-supplied inputs that affect internal logic and state. Together, these constructs capture the primary channels through which external influence can enter a contract’s execution. Semantically, they correspond to the ① Preparation phase of an attack, where adversaries gain capital, trigger flash loans, or manipulate inputs to set up subsequent exploitation.

Egress points, in contrast, denote critical financial or permission-related operations whose execution under attacker influence may lead to fund extraction, unauthorized transfers, or system compromise. They encompass both standard and low-level behaviors. On the standard side, functions such as transfer, transferFrom, and approve are key ERC-20 primitives for asset transfer and allowance control, while deposit and withdraw govern asset custody in DeFi protocols. On the low-level side, delegatecall enables arbitrary code execution, and selfdestruct can irreversibly drain contract balances. These operations align with the ② Exploitation phase, where internal states are manipulated for gain, and the ③ Exit phase, where profits are withdrawn or laundered. By tracing propagation paths from ingress to egress, we capture the semantic essence of adversarial fund-flows and ensure that our definition of these anchors is grounded in real-world attack semantics rather than arbitrary heuristics.

By matching behavior-node variables against these predefined patterns, we extract ingress and egress node sets $\scriptstyle { S _ { \mathrm { i n g r e s s } } }$ and $ { S _ { \mathrm { e g r e s s } } }$ for each function, serving as semantic anchors for downstream fund-flow and intent analysis.

```txt
INITIAL_INGRESS = {
    "msg.sender",            # function caller
    "msg.value",            # incoming ETH value
    "tx.origin",            # transaction origin
    "address(this).balance", # contract's balance
    "<function parameters>"  # function parameters
}
INITIAL_EGRESS = {
    "transfer",            # token/native transfer
    "transferFrom",          # delegated transfer
    "approve",            # grant allowance
    "deposit",            # asset deposit
    "withdraw",          # asset withdrawal
    "flashLoan",          # flash loan
    "swapExactTokensForTokens",  # token swap
    "selfdestruct",          # destroy & transfer
    "delegatecall"      # external logic call
}
```  
Listing 1: Initial ingress and egress for fund-flow-oriented adversarial analysis

## 5.4. Reachability Analysis

Given a set of ingress points $S _ { \mathrm { i n g r e s s } } ,$ , we perform fundflow propagation using a depth-first search (DFS) strategy, as outlined in Algorithm 1. Starting from each ingress node, the traversal recursively explores downstream variables; each newly encountered variable is added to the reachable set until no new nodes appear. Inter-function propagation is enabled by the normalized naming scheme, which identifies equivalent entities across functions and connects them through inter-function edges. To avoid infinite recursion from cycles, a visited set is maintained throughout the process.

After forward propagation, a backward traversal is conducted from the egress points $\scriptstyle { \mathcal { S } } _ { \mathrm { e g r e s s } }$ to retain only semantically valid paths that originate from an ingress and terminate at an egress, pruning irrelevant branches. For example, in Figure 6 (f), the path from v to op is removed since $o p _ { 1 } \notin S _ { \mathrm { e g r e s s } }$ , whereas $o p _ { 2 } \in \mathcal { S } _ { \mathrm { e g r e s s } }$ yields a valid crossfunction path $v _ { 2 } \xrightarrow { c _ { 3 } } v _ { 3 } \xrightarrow { c _ { 1 } }$ op<sub>2</sub>, with c<sub>3</sub> and c<sub>1</sub> preserved as edge annotations encoding control-flow context.

The analysis results in a set of fund-flow paths, each represented as a variable-level dependency chain annotated with its associated conditions. These conditions, expressed in NL, are recorded for subsequent reasoning but not enforced during traversal, balancing semantic interpretability and scalability for downstream large language model-based analysis.

## 6. Uncertainty Quantification via Probing and Fusion

To better quantify uncertainty, we probe the model from a general-purpose view capturing global semantics and an attack-specific view focusing on adversarial behaviors. In addition, we introduce misleading prompts as controlled perturbations to examine confidence stability.

In borderline scenarios, a single direct query often misclassifies benign contracts as adversarial (see Baseline ’s result in §7.3), as LLMs tend to be overconfident [16], [17]. In black-box senario, internal confidence signals are inaccessible, making it impossible to directly calibrate the model’s confidence. When the LLM outputs verbalized confidence scores, the distribution reflects its certainty. A uniform distribution (each of n labels assigned 1/n) indicates high uncertainty, while a highly skewed one (e.g., 95% versus 5%) implies strong confidence. This aligns with the concept of entropy in information theory, which quantifies uncertainty that lower entropy indicates higher reliability.

By integrating multi-dimensional information about the contract and entropy-based uncertainty quantification, we fuse predictions into more stable and robust results. All queries are executed in parallel, and the fusion step maintains an acceptable overall cost of roughly \$0.003 per contract.

## 6.1. Binary Label Expansion

In binary adversarial detection, LLMs often produce overconfident predictions, particularly for borderline cases that exhibit partial suspicious behaviors. This overconfidence can obscure the model’s uncertainty and lead to unreliable decisions. To capture more fine-grained confidence information, we expand the binary classification task into a four-level label spectrum:

<div class="mineru-algorithm" style="white-space: pre-wrap; font-family:monospace;">
$\mathcal{L} = \{\text{adversarial, suspicion, uncertain, benign}\}$.
</div>

Directly using verbalized confidence scores has been shown to be unreliable. In contrast, comparative judgments based on relative rankings often yield more consistent and trustworthy results [16]. Therefore, we prompt the LLM to rank the four label options by likelihood and assign confidence scores that sum to 100%, ensuring both interpretability and comparability. An illustrative example of this prompt is shown in Figure 7. This structured output not only enables precise quantification of suspicion levels, but also provides the necessary data foundation for the subsequent entropybased multi-view fusion.

## 6.2. Confidence Probing

In Stage I, we obtained both general-purpose and attackspecific information regarding the contract. Due to their abstract nature, general-purpose information often fail to capture concrete behavioral patterns, thereby biasing the model toward benign classifications. In contrast, attackspecific information provides detailed contextual cues more commonly associated with malicious behavior. However, such information may not always clearly indicate attacking behavior due to its specificity and may bias the model towards malicious judgment in some cases. Hence, incorporating the both perspectives is essential for a more comprehensive behavioral analysis.

![](images/f1147c7ab2243ace801c5ed69a8681003a9aef702dd24268eda5b885a3e41345.jpg)  
Figure 7: Prompt Template of Stage II. The contract information of general-purpose analysis is provided in Figure 9, and the attack-specific information is provided in Figure 10.

Furthermore, prior research has shown that misleading or weakly suggestive prompts can influence the model’s expressed confidence, even when the final prediction remains unchanged. For example, if the LLM is highly confident that a contract is adversarial, introducing a misleading cue implying benign behavior has minimal effect. Conversely, when the model is uncertain, a misleading cue suggesting adversarial intent can increase its confidence in classifying the contract as adversarial. Building on this insight, we propose the use of misleading prompts to probe the LLM’s confidence expression.

Specifically, we design six types of prompts to evaluate the model’s confidence across diverse perspectives, defined as:

$$
\begin{array}{l} \mathcal {P} = \{\text {g\_normal}, \text {s\_normal}, \text {g\_mislead\_adv}, \\ \quad \text {g\_mislead\_be}, \text {s\_mislead\_adv}, \text {s\_mislead\_be} \}, \end{array}
$$

where g\_normal and s\_normal correspond to generalpurpose and attack-specific information, respectively; g\_mislead\_adv and g\_mislead\_be denote adversarial and benign misleading prompts based on general-purpose information; while s\_mislead\_adv and s\_mislead\_be apply similar misleading cues to attackspecific information. Each misleading prompt contains a soft hint such as: “Hint: I think the answer should be (A) adversarial / (D) benign”, as illustrated in Figure 7.

## 6.3. Entropy-Based Fusion and Decision

Confidence entropy provides a principled measure of uncertainty in a prediction. Higher entropy indicates greater ambiguity in the model’s output, while lower entropy suggests more stable and reliable predictions, consistent with Shannon’s information theory [9]. Based on this principle, we adopt an entropy-based fusion strategy to aggregate multi-view outputs obtained from six different prompts. Specifically, each view produces a ranked confidence distribution over the label options, and we compute the entropy of each distribution. During fusion, predictions from views with lower entropy are assigned higher weights, reflecting their higher reliability. This approach effectively leverages the consistency across multiple prompts to improve classification robustness without incurring significant computational overhead.

Based on this principle, we adopt an entropy-based fusion strategy to aggregate multi-view outputs from six prompts. Each prompt instructs the LLM to assign confidence scores to four label options, forming a normalized confidence distribution. The entropy of this distribution measures view-level uncertainty: a lower entropy indicates more decisive predictions. During fusion, views with lower entropy receive higher weights, thereby emphasizing more reliable judgments while maintaining negligible computational overhead.

Each of the 6 prompts generates a confidence distribution over 4 labels, denoted as:

$$
\left\{\left(G _ {i} ^ {(p)}, P _ {i} ^ {(p)}\right) \mid p \in \mathcal {P}, i \in \mathcal {L}, \sum_ {i \in \mathcal {L}} P _ {i} ^ {(p)} = 1 0 0 \% \right\},\tag{2}
$$

where $G _ { i } ^ { ( p ) }$ is the label i under the prompt $p ,$ and $P _ { i } ^ { ( p ) }$ is the corresponding confidence score, which sums to 100% across all labels for each prompt. To quantify the uncertainty of each prompt’s output, we compute the entropy as

$$
H ^ {(p)} = - \sum_ {i \in \mathcal {L}} p _ {i} ^ {(p)} \log p _ {i} ^ {(p)},\tag{3}
$$

where $H ^ { ( p ) }$ denotes the entropy score corresponding to the prompt $p . \mathrm { ~ A ~ }$ lower entropy implies a more confident prediction. Hence, we define the weight of prompt p as:

$$
w ^ {(p)} = \frac {1}{H ^ {(p)} + \varepsilon}, \quad \varepsilon = 1 0 ^ {- 6}.\tag{4}
$$

Each prompt also produces a ranked list over the 4 labels. We assign discrete scores to the labels based on their ranks, with the highest-ranked label receiving 3 points, followed by 2, 1, and 0. Let $s _ { i } ^ { ( p ) }$ denote the score assigned to label $i \in \mathcal { L }$ under prompt $p \in \mathcal P$ . The final weighted score for each label is computed as:

$$
S _ {i} = \sum_ {p \in \mathcal {P}} w ^ {(p)} \cdot s _ {i} ^ {(p)}.\tag{5}
$$

We then normalize these scores to a probability distribution:

$$
\hat {S} _ {i} = \frac {S _ {i}}{\sum_ {j \in \mathcal {L}} S _ {j}}.\tag{6}
$$

Finally, to make a binary decision, we merge the four labels into two groups. We compute:

$$
\text {adv\_score} = \hat {S} _ {\text {adversarial}} + \hat {S} _ {\text {suspicion}},\tag{7}
$$

$$
\mathrm {be\_score} = \hat {S} _ {\text {benign}} + \hat {S} _ {\text {uncertain}}.\tag{8}
$$

If $\mathsf { a d v \_ s c o r e } > \mathsf { b e \_ s c o r e }$ , the contract is classified as adversarial; otherwise, it is classified as benign.

## 6.4. Adaptive Detection Trade-offs

Thanks to our flexible aggregation strategy, we obtain a continuous adversarial score (adv\_score) rather than a binary label, enabling finer perception of the LLM’s confidence. Users can select thresholds according to their needs: a lower threshold favors sensitivity (lower FNR), while a higher one reduces false alarms (lower FPR). Contracts with scores above the threshold are classified as adversarial, and others as benign. The corresponding trade-offs between sensitivity and specificity under different thresholds are illustrated in §7.5.1, with further quantitative details provided in the experimental section.

## 7. Evaluation

## 7.1. Experimental Setup

7.1.1. Dataset. To comprehensively evaluate FinDet, we constructed two datasets: $\mathcal { D } _ { G T }$ , a ground-truth-labeled dataset, and $\mathcal { D } _ { R W }$ , a collection of contracts from the wild.

For the adversarial contracts in $\mathcal { D } _ { G T }$ , due to the limited and fragmented availability of labeled adversarial contract data, we made our best effort to comprehensively collect relevant information and annotate it through a combination of original source labels and expert refinement. Specifically, we gathered data from six sources [3], [4], [5], [6], [18], [19], resulting in a total of 455 adversarial contracts across multiple chains, including Ethereum, BSC, Optimism and Polygon. Among these, 200 contracts contained initial type labels that could be traced to reported attack incidents. We further refined and standardized these labels by referencing incident reports (e.g., tweets, blogs, and disclosures) and analyzing the decompiled pseudocode of the corresponding adversarial contracts. The normalized labels were aligned with the OWASP Smart Contract Top 10 (2025)<sup>1</sup> , a widely recognized guideline that categorizes the most critical smart contract vulnerabilities. We focus on attacks actively initiated by adversarial contracts, excluding cases such as rug pulls and private key compromises. As shown in TABLE 9, the labeled subset of $\mathcal { D } _ { G T }$ , consisting of 200 adversarial contracts with normalized vulnerability categories, represents the only publicly available and the largest multi-type labeled dataset of adversarial contracts to date. The complete set of 455 adversarial contracts further serves as the largest and most comprehensive dataset covering diverse attack types. For the benign subset of $\mathcal { D } _ { G T }$ , we utilized the benign contracts provided by Lookahead and randomly sampled 20,000 contracts.

As for $\mathcal { D } _ { R W }$ , we collected contract data from BscScan over a 10-day period from April 18 to April 27 (UTC time), 2025. We extracted transaction information and contract creation records to obtain contract addresses, and subsequently crawled the corresponding runtime bytecode. Contracts that had already self-destructed were excluded from the dataset. As a result, we obtained 13,210 valid contracts. This dataset was only used in §7.6.

7.1.2. Metrics. We use the Balanced Accuracy (BAC)<sup>2</sup> to evaluate detection performance. Balanced accuracy is particularly suitable for binary and multiclass classification problems [20], [21], [22] involving imbalanced datasets. It is defined as the average of recall values across all classes, and is particularly suitable for binary and multiclass classification problems involving imbalanced datasets and has been widely adopted in detection tasks on imbalanced datasets [23]. To provide a more comprehensive evaluation, we also report the False Positive Rate (FPR), False Negative Rate (FNR), True Positive Rate (TPR), and True Negative Rate (TNR).

7.1.3. Baseline Selection. To our knowledge, only two prior works perform adversarial contract detection directly from bytecode without targeting specific attack types:

• Lookahead [5]: Uses 40 handcrafted features (onchain transactions and static bytecode features) and a PSCFT module to extract semantic information from bytecode. Both are inputs to ML classifiers.

• Skyeye [6]: Selects 12 features from Lookahead and applies control flow graph (CFG) segmentation for semantic extraction. These representations are also used with ML models. We reproduced their method from partially released code.

For fairness, we followed their setups by oversampling adversarial contracts, using an 80/20 train–test split, and training for 10 epochs (Lookahead) and 20 epochs (Skyeye). We also add Baselin $\mathbf { e _ { N L } }$ in §7.3 to compare our method against directly feeding NL descriptions to an LLM.

7.1.4. Implementation. Experiments were conducted on an Ubuntu 24.04 server with a 64-core Intel Xeon Gold 6426Y CPU, eight NVIDIA RTX 4090 GPUs (24GB each), and 256GB RAM. FinDet uses deepseek-v3 [24] as the default LLM, and we further evaluate generalizability using claude-3-5-haiku-20241022 [25] and gpt-4o-2024-08-06 [26].

## 7.1.5. Research Questions.

• RQ1: How does FinDet compare to existing baseline methods in detecting adversarial contracts under a training-free paradigm? (Baseline Comparison)

TABLE 1: Overall performance of different methods. Training time is reported in minutes.

<table><tr><td>Method</td><td>FPR ↓</td><td>FNR ↓</td><td>TPR ↑</td><td>TNR ↑</td><td>BAC ↑</td><td>Training Time ↓</td></tr><tr><td>Lookahead</td><td>0.0014</td><td>0.3516</td><td>0.6484</td><td>0.9987</td><td>0.8235</td><td>16.58</td></tr><tr><td>Skyeye</td><td>0.0034</td><td>0.1868</td><td>0.8132</td><td>0.9966</td><td>0.9049</td><td>1440.06</td></tr><tr><td>FinDet</td><td>0.0483</td><td>0.0769</td><td>0.9231</td><td>0.9517</td><td>0.9374</td><td>0.00</td></tr></table>

• RQ2: What is the impact of individual components in the Stage I probing module on detection performance? (Ablation Study of Stage I)

• RQ3: What is the impact of each component in the Stage II fusion and reasoning module? (Ablation Study of Stage II)

• RQ4: How efficient and adaptable is FinDet in terms of inference time, token cost, cross-LLM generalization, and threshold flexibility? (Scalability and Efficiency)

• RQ5: How effective is FinDet in detecting adversarial contracts in real-world settings, and what is its potential impact in practical deployments? (Real-World Impact)

## 7.2. RQ1: Baseline Comparison

To ensure fair comparison and reliable results, we use the same training and testing splits across all experiments. Specifically, we randomly select 20% of adversarial and benign contracts from $\mathcal { D } _ { G T }$ as the testing set, with the remaining 80% used for training. This process is repeated five times, and the average performance is reported. All baseline methods are trained and evaluated using these five data splits, except for the “unseen pattern” experiment in §7.2.2, which requires special consideration of adversarial contract types. As our method does not require training, we directly evaluate it on the five testing sets.

7.2.1. Overall Performance. TABLE 1 summarizes the detection performance of different methods. We observe that FinDet achieves a state-of-the-art balanced accuracy (BAC) of 0.9374, outperforming Lookahead (0.8235) and Skyeye (0.9049). The false negative rates (FNR) of Lookahead and Skyeye are 0.3516 and 0.1868, respectively, indicating their high miss rates in detecting adversarial contracts. In contrast, FinDet significantly reduces the FNR to 0.0769, demonstrating its superior capability in identifying attack behaviors. The “Training Time” column in TABLE 1 represents the total time required for model training after all necessary data preparation has been completed. Specifically, Lookahead requires up to 16.58 minutes, while Skyeye takes approximately 1,440 minutes (about 24 hours). In comparison, FinDet is a training-free approach that requires no training time, making it significantly more efficient.

7.2.2. Unseen Pattern Evaluation. In this experiment, we use the subset of 200 adversarial contracts with normalized type labels in $\mathcal { D } _ { G T }$ , which covers nine types of adversarial contract. In each experiment, one type $t y p e _ { i }$ is used as the test set. To ensure fairness, any contracts that are labeled with multiple types including type<sub>i</sub> are excluded from the test set. If $t y p e _ { i }$ has over 10 contracts, 10 are randomly selected for testing; otherwise, all are used. The training set is drawn from the remaining types, capped at 100 contracts if available. This maintains a roughly 100:10 training-totesting ratio. We report per-type results along with microand macro-averaged metrics to provide a comprehensive evaluation.

TABLE 2: Performance under unseen attack types. “micro” and “macro” denote different aggregation methods over the nine attack categories.

<table><tr><td>Method</td><td>FPR ↓</td><td>FNR ↓</td><td>TPR ↑</td><td>TNR ↑</td><td>BAC ↑</td></tr><tr><td>Lookahead (micro)</td><td>0.0006</td><td>0.6610</td><td>0.3390</td><td>0.9994</td><td>0.6692</td></tr><tr><td>Lookahead (macro)</td><td>0.0004</td><td>0.7043</td><td>0.2957</td><td>0.9996</td><td>0.6476</td></tr><tr><td>Skyeye (micro)</td><td>0.0022</td><td>0.5085</td><td>0.4915</td><td>0.9978</td><td>0.7447</td></tr><tr><td>Skyeye (macro)</td><td>0.0028</td><td>0.4914</td><td>0.5086</td><td>0.9972</td><td>0.7529</td></tr><tr><td>FinDet</td><td>0.0484</td><td>0.1050</td><td>0.8950</td><td>0.9517</td><td>0.9233</td></tr></table>

TABLE 2 summarizes the results of baseline methods and FinDet under the new attack type evaluation. Lookahead’s FNR rises sharply to 0.6610 (micro) and 0.7043 (macro), reducing BAC from 0.8235 to 0.6692 and 0.6476, respectively. Skyeye shows a similar drop, with BAC decreasing from 0.9049 to 0.7447 (micro) and 0.7529 (macro), and FNR rising to over 0.49. In contrast, FinDet achieves the best BAC of 0.9233, with a low FNR of 0.1050, indicating strong generalization to unseen attack types. For example, when testing on Type 5 (reentrancy), Lookahead detects only 1 of 10 cases, and Skyeye detects 3, while FinDet performs significantly better.

7.2.3. Training vs. Training-Free Performance. To evaluate the impact of training set size on method performance, we conducted stratified sampling on the pre-partitioned training dataset, keeping the train-test split fixed for fair comparison.

TABLE 3 presents results across various sampling ratios. Even at 10% (1,636 contracts in training set), the training set remains larger than many datasets used by existing ML methods [23]. As training data decreases from 100% to 10%, Lookahead’s BAC drops from 0.8235 to 0.7604, Skyeye’s from 0.9049 to 0.8488, while FinDet consistently maintains a BAC of 0.9374, demonstrating robustness as a trainingfree method.

Notably, Lookahead and Skyeye require 40 and 12 handcrafted features respectively, and the process of obtaining these features incurs substantial extra cost that is not included in the training time reported in TABLE 3. In contrast, FinDet only requires the contract bytecode as input, eliminating the need for any extra data preparation.

7.2.4. Obfuscation Evaluation. Lookahead and Skyeye depend on on-chain transaction features like Nonce, Value, InputDataLength, GasUsed, and Verified, which are easily manipulated through simple obfuscation. Ma et al. [27] revealed that verified Ethereum contracts’ bytecode and source code may mismatch, exposing verification-breaking vulnerabilities with real exploitable cases. Inspired by this, we simulate an attacker submitting benign-appearing source code that doesn’t match deployed bytecode, setting the Verified feature to “True” for adversarial contracts to evade detection.

TABLE 3: Training vs. training-free performance.

<table><tr><td>Method</td><td>Sample Ratio</td><td>FPR ↓</td><td>FNR ↓</td><td>TPR ↑</td><td>TNR ↑</td><td>BAC ↑</td></tr><tr><td rowspan="6">Lookahead</td><td>100%</td><td>0.0014</td><td>0.3516</td><td>0.6484</td><td>0.9987</td><td>0.8235</td></tr><tr><td>90%</td><td>0.0015</td><td>0.3626</td><td>0.6374</td><td>0.9986</td><td>0.8180</td></tr><tr><td>70%</td><td>0.0008</td><td>0.3736</td><td>0.6264</td><td>0.9992</td><td>0.8128</td></tr><tr><td>50%</td><td>0.0021</td><td>0.3758</td><td>0.6242</td><td>0.9980</td><td>0.8111</td></tr><tr><td>30%</td><td>0.0021</td><td>0.4352</td><td>0.5648</td><td>0.9980</td><td>0.7814</td></tr><tr><td>10%</td><td>0.0023</td><td>0.4769</td><td>0.5231</td><td>0.9978</td><td>0.7604</td></tr><tr><td rowspan="6">Skyeye</td><td>100%</td><td>0.0034</td><td>0.1868</td><td>0.8132</td><td>0.9966</td><td>0.9049</td></tr><tr><td>90%</td><td>0.0043</td><td>0.1868</td><td>0.8132</td><td>0.9957</td><td>0.9044</td></tr><tr><td>70%</td><td>0.0034</td><td>0.2000</td><td>0.8000</td><td>0.9967</td><td>0.8983</td></tr><tr><td>50%</td><td>0.0035</td><td>0.2022</td><td>0.7978</td><td>0.9965</td><td>0.8972</td></tr><tr><td>30%</td><td>0.0039</td><td>0.2615</td><td>0.7385</td><td>0.9961</td><td>0.8673</td></tr><tr><td>10%</td><td>0.0057</td><td>0.2967</td><td>0.7033</td><td>0.9943</td><td>0.8488</td></tr><tr><td>FinDet</td><td>0%</td><td>0.0483</td><td>0.0769</td><td>0.9231</td><td>0.9517</td><td>0.9374</td></tr></table>

TABLE 4 shows that Lookahead and Skyeye’s FNRs increase (from 0.3516 to 0.5956 and from 0.1868 to 0.2132), and BACs drop to 0.7017 and 0.8917. In contrast, FinDet, which doesn’t rely on on-chain features, remains unaffected. Other features like Nonce can also be obfuscated by sending benign transactions before deploying the real adversarial contract, disguising deployment patterns.

TABLE 4: Performance under obfuscation scenarios.

<table><tr><td>Method</td><td>FPR ↓</td><td>FNR ↓</td><td>TPR ↑</td><td>TNR ↑</td><td>BAC ↑</td></tr><tr><td>Lookahead</td><td>0.0011</td><td>0.5956</td><td>0.4044</td><td>0.9990</td><td>0.7017</td></tr><tr><td>Skyeye</td><td>0.0034</td><td>0.2132</td><td>0.7868</td><td>0.9966</td><td>0.8917</td></tr><tr><td>FinDet</td><td>0.0483</td><td>0.0769</td><td>0.9231</td><td>0.9517</td><td>0.9374</td></tr></table>

Summary to RQ1: FinDet achieves a SOTA BAC of 0.9374 and low FNR of 0.0769, significantly better than baselines. FinDet remains robust under challenging scenarios where baseline performance degrades rapidly.

## 7.3. RQ2: Impact of Behavioral Semantics

To evaluate the effectiveness of behavioral semantics, we introduce an additional baseline:

• Baseline : NL descriptions are fed directly into the LLM with a task-specific prompt (see Figure 11).

We also conduct an ablation study by removing the generalpurpose view (§4.2, FinDet w/o G) and the attack-specific view (§4.3, FinDet w/o S) to assess their individual contributions.

As shown in TABLE 5, removing the general-purpose view increases FPR to 0.2341, while removing the attack-specific view raises FNR to 0.1473. The corresponding BAC drops to 0.8532 and 0.9010, respectively.

Baseline performs worst, with an FPR of 0.5712 and BAC of 0.7067. These results highlight the importance of both views for accurate detection.

TABLE 5: Performance comparison of different input information for Stage I.

<table><tr><td>Method</td><td>FPR ↓</td><td>FNR ↓</td><td>TPR ↑</td><td>TNR ↑</td><td>BAC ↑</td></tr><tr><td>FinDet w/o G</td><td>0.2341</td><td>0.0595</td><td>0.9405</td><td>0.7659</td><td>0.8532</td></tr><tr><td>FinDet w/o S</td><td>0.0508</td><td>0.1473</td><td>0.8527</td><td>0.9493</td><td>0.9010</td></tr><tr><td> $Baseline_{NL}$ </td><td>0.5712</td><td>0.0154</td><td>0.9846</td><td>0.4289</td><td>0.7067</td></tr><tr><td>FinDet</td><td>0.0483</td><td>0.0769</td><td>0.9231</td><td>0.9517</td><td>0.9374</td></tr></table>

## 7.4. RQ3: Impact of Confidence Probing and Fusion Strategy

In this section, we evaluate the effectiveness of the confidence probing and fusion strategies in Stage II. We compare our method with some alternatives: (1) g\_normal and $( 2 ) \mathsf { s \_ n o r m a l }$ , which use a single prompt respectively; (3) no\_misleading, which fuses the two non-misleading prompts in FinDet; and (4) borda count, which ranks and aggregates confidence scores using the Borda count method [28].

FinDet achieves the highest BAC and the lowest FPR among all methods. Results are summarized in TABLE 6. Methods (1)-(3) show the necessity of using all six prompts, while (4) demonstrate the effectiveness of our fusion strategy. This confirms that our approach better integrates multiperspective information and fuses predictions with varying confidence levels to enhance overall detection performance.

TABLE 6: Performance under different probing and fusion strategies.

<table><tr><td>Method</td><td>FPR ↓</td><td>FNR ↓</td><td>TPR ↑</td><td>TNR ↑</td><td>BAC ↑</td></tr><tr><td>g_normal</td><td>0.0643</td><td>0.1275</td><td>0.8725</td><td>0.9358</td><td>0.9041</td></tr><tr><td>s_normal</td><td>0.3002</td><td>0.0573</td><td>0.9427</td><td>0.6998</td><td>0.8213</td></tr><tr><td>no_misleading</td><td>0.0912</td><td>0.0703</td><td>0.9297</td><td>0.9089</td><td>0.9193</td></tr><tr><td>borda_count</td><td>0.1290</td><td>0.0637</td><td>0.9363</td><td>0.8710</td><td>0.9036</td></tr><tr><td>FinDet</td><td>0.0483</td><td>0.0769</td><td>0.9231</td><td>0.9517</td><td>0.9374</td></tr></table>

## 7.5. RQ4: Scalability and Efficiency

7.5.1. Optional Threshold Selection. As discussed in §6.4, users may have different preferences regarding the balance between FPR and FNR depending on specific application scenarios. To accommodate this, we provide an optional threshold selection mechanism, allowing users to adjust the adversarial score threshold based on their desired trade-off. Generally, a higher threshold yields lower FPR but higher FNR, while a lower threshold results in the opposite. As shown in Figure 12, setting the threshold to 0.18 results in a very low FNR of 0.0088, whereas increasing the threshold to 0.7 reduces the FPR to 0.0003. This illustrates the inherent trade-off between sensitivity and specificity in thresholdbased decisions.

7.5.2. Evaluation under Different LLM Backbones. To evaluate the generalizability of our approach, we conduct experiments using different LLM backbones, as shown in TABLE 7. FinDet achieves consistently strong performance across models, with peak BAC scores of 0.9064 and 0.8461 on Claude 3.5 and ${ \mathrm { G P T } } - 4 0 ,$ , and correspondingly low FNRs of 0.1429 and 0.1231 with an optimized threshold, outperforming the baselines by up to 59.36% and 64.99% in FNR. Notably, the best performance is observed on DeepSeek-V3, with a BAC of 0.9374 and an FNR of 0.0769. These results demonstrate that FinDet maintains high detection accuracy across a range of LLMs, indicating good generalizability. In particular, Claude 3.5 surpasses all baselines in both BAC and FNR, and GPT-4o delivers comparable performance.

TABLE 7: Performance under different LLMs.

<table><tr><td>Model</td><td>FPR ↓</td><td>FNR ↓</td><td>TPR ↑</td><td>TNR ↑</td><td>BAC ↑</td></tr><tr><td>DeepSeek-V3</td><td>0.0483</td><td>0.0769</td><td>0.9231</td><td>0.9517</td><td>0.9374</td></tr><tr><td>Claude 3.5</td><td>0.0444</td><td>0.1429</td><td>0.8571</td><td>0.9557</td><td>0.9064</td></tr><tr><td>GPT-4o</td><td>0.1848</td><td>0.1231</td><td>0.8769</td><td>0.8153</td><td>0.8461</td></tr></table>

7.5.3. Efficiency Analysis. We compare average token and time costs across models to assess efficiency. For Deepseek-V3, the average token usage is 17,085.48, with a per-contract cost of \$0.0030, including \$0.0018 from Stage I and \$0.0012 from Stage II. In comparison, Claude-3.5 costs \$0.0021, while GPT-4o costs \$0.0664 per contract. Since our method operates solely on bytecode, it supports pre-deployment detection and reduces overall overhead.

## 7.6. RQ5: Real-World Impact

To evaluate the real-world effectiveness of FinDet, we collected five publicly reported adversarial contracts (TA-BLE 11) from reliable sources, all of which were correctly identified based on bytecode, achieving a 100% TPR. Additionally, by randomly sampling contracts from $\mathcal { D } _ { R W }$ and validating them with expert assistance, we uncovered 29 previously undocumented adversarial contracts showing clear malicious intent or stealthy behavior.

7.6.1. Case Study. As part of our case study, we analyze an adversarial contract (see BscScan link) flagged by FinDet and independently confirmed by TenArmorAlert [29]. The attack resulted in a loss of approximately \$26K. As the source code is unavailable, we integrate two state-of-the-art tools, Disco [14] for source code recovery and Elipmoc [7] for pseudocode lifting, together with manual validation to ensure that the recovered logic is semantically consistent with the on-chain runtime bytecode, as shown in Figure 4.

As analyzed in Obs2, this contract executes its attack in typical three stages: preparation (injecting funds via exploit()), exploitation (arbitrage via DPPFlashLoanCall()), and exit (extracting illicit gains in DPPFlashLoanCall()). FinDet extracted key fundflow paths (Figure 15). TABLE 13 shows prediction scores from 6 prompts. After entropy-based fusion, the adversarial, suspicion, uncertain, and benign scores are 0.288, 0.327, 0.221, and 0.163, respectively. The combined adversarial score (adversarial + suspicion) is 0.615, leading to a final prediction of adversarial.

7.6.2. Real World Findings. Notably, FinDet identified instances of price manipulation and arbitrage facilitated by flash loans, as illustrated in Figure 4. These cases represent a rapidly emerging and increasingly prevalent threat in recent years, distinctly different from conventional flash loan attacks that exploit protocol vulnerabilities, as well as from benign arbitrage strategies that leverage price inefficiencies without adversarial manipulation. The detected behaviors involve adversaries borrowing large amounts of assets via flash loans to actively distort price dynamics across protocols and extract profits through carefully orchestrated arbitrage. These cases conform to the three-stage adversarial pattern abstracted in our model, highlighting FinDet’s capability to capture emerging yet previously unseen attack patterns.

## 8. Discussion

Limitations. Our method detects adversarial behavior exhibited directly by contracts. It may miss proxy contracts that delegate malicious logic elsewhere, as these proxies show no adversarial actions themselves. For example, some false negatives occur when the actual exploit lies in a downstream callee, not the proxy.

Future Work. Currently, we analyze adversarial contracts independently, focusing on internal logic and fund flows. However, many attacks involve subtle interactions with victim contracts. Future work could integrate victim contracts into the analysis, enabling joint reasoning over attacker and target behaviors. Since most adversarial contracts hardcode victim addresses or store them in fixed slots [3], cross-contract analysis is both feasible and promising. This would improve detection of coordinated attacks (e.g., reentrancy, storage collisions), track fund flows across contracts, and enhance accuracy in complex scenarios.

## 9. Related Work

## 9.1. Vulnerability Detection in Smart Contracts

Vulnerability detection is a long-standing and critical task in the blockchain ecosystem. Traditional methods can be broadly categorized into four types: static analysis-based [30], symbolic execution-based [31], fuzzingbased [32], and machine learning-based approaches [33]. LLMs have recently been explored for vulnerability detection [34], with hybrid methods combining them with static analysis [8], multi-agent reasoning [35], or fuzzing [36].

Despite recent progress, existing approaches face three key limitations. First, most detected vulnerabilities are not practically exploitable: only 2.68% [3] of flagged contracts are actually exploited. Second, high false positives remain common [37], such as misclassifying safe anti-reentrancy contracts as vulnerable. Third, there remains an inherent asymmetry: defenders must address all threats, while attackers only need to exploit one. These issues highlight the need for proactive, adversary-aware detection focused on real-world exploits rather than theoretical flaws.

## 9.2. Adversarial Detection in Transactions

Some works aim to detect attacks in real time by monitoring unconfirmed transactions in the public mempool. They typically follow two approaches: pattern-based [38], [39], [40], using manually defined heuristics; and learningbased [41], [42], [43], leveraging ML models with graphbased features.

While effective in public settings, these methods fail to detect transactions submitted via private mempools, which bypass public monitoring. Moreover, their reliance on fixed rules or models limits adaptability to novel or evolving attacks.

## 9.3. Adversarial Detection in Smart Contracts

Beyond vulnerability identification, recent research has focused on detecting adversarial smart contracts deliberately designed to exploit vulnerabilities or manipulate protocol logic. For example, Yang et al. [3] propose a method to identify contracts capable of launching exploitable reentrancy attacks. Smartcat [4] proposes an efficient static analyzer that detects price manipulation adversarial contracts via cross-function call graphs and token flow graphs. However, these approaches target only a specific type of adversarial contracts.

In contrast to pattern-specific detection, two works aim to develop ML-based systems that generalize across various types of adversarial contracts. Lookahead [5] lifts bytecode into TAC using Elipmoc [7], and combines Pruned Semantic-Control Flow Tokenization with handcrafted features using a dual-branch architecture for final classification. Skyeye [6] extracts static features and performs control flow graph segmentation to build a unified representation for training a binary classifier. However, ML-based approaches heavily rely on labeled training data, limiting their ability to detect novel or unseen attack variants. They are trained on low-level opcode features, lacking the capacity to capture high-level semantic intent. Furthermore, their poor interpretability hinders security analysts from understanding model decisions, which is critical in high-stakes security contexts.

## 10. Conclusion

In this work, we presented FinDet, a novel and trainingfree framework for detecting adversarial smart contracts directly from EVM bytecode. By lifting low-level bytecode into semi-structured natural language description, FinDet enhances semantic understanding and enables multi-view analysis of contract behavior. Additionally, FinDet incorporates fund-flow reachability analysis to capture the distinct stages of attacks, thereby strengthening semantic precision. To further improve robustness, we introduced a fine-grained uncertainty quantification mechanism that mitigates hallucinations and enhances detection reliability. Our method demonstrates strong generalization to unseen attack patterns, resilience in low-data settings, and state-of-the-art performance across multiple metrics. These results highlight the potential of LLM-based semantic reasoning for proactive and effective smart contract security. We believe FinDet offers a practical step toward securing blockchain ecosystems against emerging adversarial threats.

## References

[1] SlowMist, “Slowmist hacked statistics 2024,” https://hacked.slowmist. io/statistics/?c=all&d=2024, SlowMist, 2024, accessed: 2025-07-04.

[2] D. Perez and B. Livshits, “Smart contract vulnerabilities: Vulnerable does not imply exploited,” in 30th USENIX Security Symposium (USENIX Security 21), 2021, pp. 1325–1341.

[3] S. Yang, J. Chen, M. Huang, Z. Zheng, and Y. Huang, “Uncover the premeditated attacks: Detecting exploitable reentrancy vulnerabilities by identifying attacker contracts,” in Proceedings of the IEEE/ACM 46th International Conference on Software Engineering, 2024, pp. 1–12.

[4] B. Zhang, N. He, X. Hu, K. Ma, and H. Wang, “Following devils footprint: Towards real-time detection of price manipulation attacks,” arXiv preprint arXiv:2502.03718, 2025.

[5] S. Ren, L. He, T. Tu, D. Wu, J. Liu, K. Ren, and C. Chen, “Lookahead: Preventing defi attacks via unveiling adversarial contracts,” Proceedings of the ACM on Software Engineering, vol. 2, no. FSE, pp. 1847–1869, 2025.

[6] H. Wang, Y. Hu, H. Wu, D. Liu, C. Peng, Y. Wu, M. Fan, and T. Liu, “Skyeye: Detecting imminent attacks via analyzing adversarial smart contracts,” in Proceedings of the 39th IEEE/ACM International Conference on Automated Software Engineering, 2024, pp. 1570– 1582.

[7] N. Grech, S. Lagouvardos, I. Tsatiris, and Y. Smaragdakis, “Elipmoc: Advanced decompilation of ethereum smart contracts,” Proceedings of the ACM on Programming Languages, vol. 6, no. OOPSLA1, pp. 1–27, 2022.

[8] Y. Sun, D. Wu, Y. Xue, H. Liu, H. Wang, Z. Xu, X. Xie, and Y. Liu, “Gptscan: Detecting logic vulnerabilities in smart contracts by combining gpt with program analysis,” in Proceedings of the IEEE/ACM 46th International Conference on Software Engineering, 2024, pp. 1–13.

[9] C. E. Shannon, “A mathematical theory of communication,” The Bell system technical journal, vol. 27, no. 3, pp. 379–423, 1948.

[10] C. Sendner, L. Petzi, J. Stang, and A. Dmitrienko, “Large-scale study of vulnerability scanners for ethereum smart contracts,” in 2024 IEEE Symposium on Security and Privacy (SP). IEEE, 2024, pp. 2273– 2290.

[11] Y. Liu, Y. Xue, D. Wu, Y. Sun, Y. Li, M. Shi, and Y. Liu, “Propertygpt: Llm-driven formal verification of smart contracts through retrievalaugmented property generation,” arXiv preprint arXiv:2405.02580, 2024.

[12] S. Hu, T. Huang, K.-H. Chow, W. Wei, Y. Wu, and L. Liu, “Zipzap: Efficient training of language models for large-scale fraud detection on blockchain,” in Proceedings of the ACM Web Conference 2024, 2024, pp. 2807–2816.

[13] J. Sun, Z. Yin, H. Zhang, X. Chen, and W. Zheng, “Adversarial generation method for smart contract fuzz testing seeds guided by chain-based llm,” Automated Software Engineering, vol. 32, no. 1, pp. 1–28, 2025.

[14] X. Su, H. Liang, H. Wu, B. Niu, F. Xu, and S. Zhong, “Disco: Towards decompiling evm bytecode to source code using large language models,” Proceedings of the ACM on Software Engineering, vol. 2, no. FSE, pp. 2311–2334, 2025.

[15] I. David, L. Zhou, D. Song, A. Gervais, and K. Qin, “Decompiling smart contracts with a large language model,” arXiv preprint arXiv:2506.19624, 2025.

[16] M. Xiong, Z. Hu, X. Lu, Y. Li, J. Fu, J. He, and B. Hooi, “Can llms express their uncertainty? an empirical evaluation of confidence elicitation in llms,” in ICLR, 2024.

[17] H. Zhao, H. Chen, F. Yang, N. Liu, H. Deng, H. Cai, S. Wang, D. Yin, and M. Du, “Explainability for large language models: A survey,” ACM Transactions on Intelligent Systems and Technology, vol. 15, no. 2, pp. 1–38, 2024.

[18] SunWeb3Sec, “DeFiHackLabs github repository,” https://github.com/ SunWeb3Sec/DeFiHackLabs, 2025, accessed: 2025-06-26.

[19] P. C. Aversaccio, “Reentrancy attacks github repository,” https: //github.com/pcaversaccio/reentrancy-attacks, 2025, accessed: 2025- 06-26.

[20] K. H. Brodersen, C. S. Ong, K. E. Stephan, and J. M. Buhmann, “The balanced accuracy and its posterior distribution,” in 2010 20th international conference on pattern recognition. IEEE, 2010, pp. 3121–3124.

[21] L. Mosley, “A balanced approach to the multi-class imbalance problem doctoral dissertation,” Iowa State University of Science and Technology, USA, 2013.

[22] A. Amin, S. Anwar, A. Adnan, M. Nawaz, N. Howard, J. Qadir, A. Hawalah, and A. Hussain, “Comparing oversampling techniques to handle the class imbalance problem: A customer churn prediction case study,” Ieee Access, vol. 4, pp. 7940–7957, 2016.

[23] C. Wu, J. Chen, Z. Wang, R. Liang, and R. Du, “Semantic sleuth: Identifying ponzi contracts via large language models,” in Proceedings of the 39th IEEE/ACM International Conference on Automated Software Engineering, 2024, pp. 582–593.

[24] A. Liu, B. Feng, B. Xue, B. Wang, B. Wu, C. Lu, C. Zhao, C. Deng, C. Zhang, C. Ruan et al., “Deepseek-v3 technical report,” arXiv preprint arXiv:2412.19437, 2024.

[25] Anthropic, “Claude: Haiku,” https://www.anthropic.com/claude/ haiku, 2023, accessed: 2025-07-30.

[26] OpenAI, “GPT-4o Model,” https://platform.openai.com/docs/models/ gpt-4o, accessed: 2025-06-30.

[27] P. Ma, N. He, Y. Huang, H. Wang, and X. Luo, “Abusing the ethereum smart contract verification services for fun and profit,” arXiv preprint arXiv:2307.00549, 2023.

[28] Wikipedia contributors, “Borda count,” https://en.wikipedia.org/wiki/ Borda\_count, 2025, accessed: 2025-11-11.

[29] TenArmorAlert, “Tenarmor security alert,” https://x.com/ TenArmorAlert/status/1916312483792408688, 2024, accessed: 2024-11-12.

[30] J. Feist, G. Grieco, and A. Groce, “Slither: a static analysis framework for smart contracts,” in 2019 IEEE/ACM 2nd International Workshop on Emerging Trends in Software Engineering for Blockchain (WET-SEB). IEEE, 2019, pp. 8–15.

[31] A. Wang, H. Wang, B. Jiang, and W. K. Chan, “Artemis: An improved smart contract verification tool for vulnerability detection,” in 2020 7th International Conference on Dependable Systems and Their Applications (DSA). IEEE, 2020, pp. 173–181.

[32] C. F. Torres, A. K. Iannillo, A. Gervais, and R. State, “Confuzzius: A data dependency-aware hybrid fuzzer for smart contracts,” in 2021 IEEE European Symposium on Security and Privacy (EuroS&P). IEEE, 2021, pp. 103–119.

[33] H. H. Nguyen, N.-M. Nguyen, H.-P. Doan, Z. Ahmadi, T.-N. Doan, and L. Jiang, “Mando-guru: Vulnerability detection for smart contract source code by heterogeneous graph embeddings,” in Proceedings of the 30th ACM Joint European Software Engineering Conference and Symposium on the Foundations of Software Engineering, 2022, pp. 1736–1740.

[34] J. Lin and D. Mohaisen, “From large to mammoth: A comparative evaluation of large language models in vulnerability detection,” in Proceedings of the 2025 Network and Distributed System Security Symposium (NDSS), 2025.

[35] W. Ma, D. Wu, Y. Sun, T. Wang, S. Liu, J. Zhang, Y. Xue, and Y. Liu, “Combining fine-tuning and llm-based agents for intuitive smart contract auditing with justifications,” arXiv preprint arXiv:2403.16073, 2024.

[36] C. Shou, J. Liu, D. Lu, and K. Sen, “Llm4fuzz: Guided fuzzing of smart contracts with large language models,” arXiv preprint arXiv:2401.11108, 2024.

[37] Q. Song, H. Huang, X. Jia, Y. Xie, and J. Cao, “Silence false alarms: Identifying anti-reentrancy patterns on ethereum to refine smart contract reentrancy detection,” in The Network and Distributed System Security (NDSS) Symposium 2025, 2025.

[38] D. Wang, S. Wu, Z. Lin, L. Wu, X. Yuan, Y. Zhou, H. Wang, and K. Ren, “Towards a first step to understand flash loan and its applications in defi ecosystem,” in Proceedings of the Ninth International Workshop on Security in Blockchain and Cloud Computing, 2021, pp. 23–28.

[39] S. Wu, Z. Yu, D. Wang, Y. Zhou, L. Wu, H. Wang, and X. Yuan, “Defiranger: detecting defi price manipulation attacks,” IEEE Transactions on Dependable and Secure Computing, vol. 21, no. 4, pp. 4147–4161, 2023.

[40] M. Xie, M. Hu, Z. Kong, C. Zhang, Y. Feng, H. Wang, Y. Xue, H. Zhang, Y. Liu, and Y. Liu, “Defort: Automatic detection and analysis of price manipulation attacks in defi applications,” in Proceedings of the 33rd ACM SIGSOFT International Symposium on Software Testing and Analysis, 2024, pp. 402–414.

[41] K. Babel, M. Javaheripi, Y. Ji, M. Kelkar, F. Koushanfar, and A. Juels, “Lanturn: Measuring economic security of smart contracts through adaptive learning,” in Proceedings of the 2023 ACM SIGSAC Conference on Computer and Communications Security, 2023, pp. 1212– 1226.

[42] Z. Li, J. Li, Z. He, X. Luo, T. Wang, X. Ni, W. Yang, X. Chen, and T. Chen, “Demystifying defi mev activities in flashbots bundle,” in Proceedings of the 2023 ACM SIGSAC Conference on Computer and Communications Security, 2023, pp. 165–179.

[43] P. Qian, J. He, L. Lu, S. Wu, Z. Lu, L. Wu, Y. Zhou, and Q. He, “Demystifying random number in ethereum smart contract: Taxonomy, vulnerability identification, and attack detection,” IEEE Transactions on Software Engineering, vol. 49, no. 7, pp. 3793–3810, 2023.

## Appendix A.

## Methodology Supplement

## A.1. Supplementary Prompts in Stage I and II

Figure 8 shows the prompts for information extraction in Stage I. Figure9 and Figure 10 show the general-purpose and attack-specific prompts used in Stage II.

## A.2. Behavior Node Types

TABLE 8 presents the behavior types and propagation rules used for fund-flow reachability analysis in §5.1.

## A.3. Reachability Analysis Algorithm

Algorithm 1 presents the reachability analysis used in fund-flow analysis, as described in §5.4.

## Appendix B.

## Experimental Supplement

## B.1. Dataset Detail

TABLE 9 compares our dataset $\mathcal { D } _ { G T }$ with existing ones, while TABLE 10 presents the detailed type distribution, as described in §7.1.1.

3. Some contracts involve multiple attack types, so the total count may not match the sum of all types. No DoS-related contracts were observed during data collection.

TABLE 8: Behavior types and propagation rules for fund-flow reachability. For each type, we define a source entity set $\mathcal { X } _ { \mathrm { s r c } }$ and a destination entity $x _ { \mathrm { d s t } }$ to identify ingress and egress points. Only fund-relevant behaviors are considered, while neutral behaviors such as returns, logs, or built-in calls are excluded as they do not affect fund-flows.

<table><tr><td>Behavior Type</td><td>NL Description Template</td><td> $\mathcal{X}_{\text{src}}$ </td><td> $\mathbf{x}_{\text{dst}}$ </td></tr><tr><td>assignment</td><td>it updates the state variable x to y</td><td> $\{y\}$  (if not constant)</td><td>x</td></tr><tr><td>external call</td><td>it triggers the external call to contract.function( $x_1, x_2, \ldots, x_n$ )</td><td> $\{x_1, x_2, \ldots, x_n\}$  (if not constant)</td><td>contract.function</td></tr><tr><td>delegate call</td><td>it delegates a call to contract.function( $x_1, x_2, \ldots, x_n$ )</td><td> $\{x_1, x_2, \ldots, x_n\}$  or calldata (if present)</td><td>delegatecall</td></tr><tr><td>contract creation</td><td>it creates a new smart contract with creation code c (and optional salt s), and gets a new address a</td><td> $\{s\}$  (optional, if not constant)</td><td>a</td></tr><tr><td>transfer</td><td>it transfers v wei to a (with gas g)</td><td> $\{v\}$  (if not constant)</td><td>transfer(v)</td></tr><tr><td>return</td><td>it returns  $x_1, x_2, \ldots, x_n$ </td><td> $\emptyset$ </td><td>-</td></tr><tr><td>log emission</td><td>it emits the log event with parameter(s)  $x_1, x_2, \ldots, x_n$ </td><td> $\emptyset$ </td><td>-</td></tr><tr><td>built-in call</td><td>it calls a built-in function f</td><td> $\emptyset$ </td><td>-</td></tr><tr><td>other</td><td>(fallback for unmatched patterns)</td><td> $\emptyset$ </td><td>-</td></tr></table>

Note: In the case of contract creation, the variable s denotes the salt, which is an optional parameter influencing the new contract address. If not present, the source variable set is empty.

![](images/61281a776e37db1355d9585dcda31058127cee01ac30823891000ea34cae6cfa.jpg)  
Figure 8: The prompt for stage I

## B.2. Prompt of Baseline<sub>NL</sub>

Figure 11 shows the prompt template of Baseline<sub>NL</sub> in §7.3, where the NL descriptions are directly fed into the LLM.

<div class="mineru-algorithm" style="white-space: pre-wrap; font-family:monospace;">
Algorithm 1: Reachability Analysis
Input: Variable-level dependency graph
    $G = (V, E)$ with edges labeled by conditions
Output: Fund-Flow paths $\mathcal{P}$ from ingress to egress
$\mathcal{S}_{ingress}, \mathcal{S}_{egress} \leftarrow$ initial_ingress_egress(
initialIngress, initialEgress, $\mathcal{F}$);
$\mathcal{E}_{visited} \leftarrow \{(v, \emptyset) \mid v \in \mathcal{S}_{ingress}\}$; // Visited variables and associated conditions
updated $\leftarrow$ true;
while updated do
    updated $\leftarrow$ false;
    foreach $v \in V$ such that $v \in \text{Dom}(\mathcal{E}_{\text{visited}})$ do
        foreach edge ($v \xrightarrow{\text{conds}} u$) $\in E$ do
            if $u \notin \text{Dom}(\mathcal{E}_{\text{visited}})$ then
                $\mathcal{E}_{\text{visited}} \leftarrow \mathcal{E}_{\text{visited}} \cup \{(u, \text{conds})\}$;
                updated $\leftarrow$ true;
            end
        end
    end
end
$\mathcal{P} \leftarrow$
ReverseDFSPrune($G, \mathcal{E}_{\text{visited}}, \mathcal{S}_{\text{ingress}}, \mathcal{S}_{\text{egress}}$);
return $\mathcal{P}$
</div>

## B.3. Threshold Selection

Figure 12 presents the experimental results of optional threshold selection in §7.5.1, showing the variations of FPR, FNR, and BAC with different threshold values.

## B.4. Disclosed Real-World Attack Cases

TABLE 11 lists adversarial contracts detected by FinDet that have been publicly disclosed.

![](images/34301fccafc9dab095129fa3246539bbdae21d553f70afe0db64c306398b711a.jpg)

![](images/8938363ac16a81b40c81f42a4669cdd62dd5666b131d42f502aea7043d3c3e38.jpg)  
Figure 9: The prompt for general-purpose analysis in Stage II.

TABLE 9: Comparison of dataset size and label availability with existing adversarial contract datasets.

<table><tr><td>Dataset</td><td>Num</td><td>Type Label</td><td>Multi-Type</td></tr><tr><td>BlockWatchdog [3]</td><td>18</td><td>●</td><td>○</td></tr><tr><td>SmartCat [4]</td><td>84</td><td>●</td><td>○</td></tr><tr><td> $\mathcal{D}_{GT}$  (part)</td><td>200</td><td>●</td><td>●</td></tr><tr><td>Lookahead [5]</td><td>375</td><td>○</td><td>●</td></tr><tr><td>Skyeye [6]</td><td>174</td><td>○</td><td>●</td></tr><tr><td> $\mathcal{D}_{GT}$ </td><td>455</td><td> $Ⓔ(200/455)$ </td><td>●</td></tr></table>

## Appendix C. Case Studies

## C.1. Motivation Example: Understanding FinDet

We provide a detailed analysis of FinDet, referring to the motivation example in Figure 4 and the earlier case study in

![](images/752937ae5cfc274a7095355020c952c8d380d489325d15196447c7974cec1d68.jpg)  
Figure 10: The prompt for attack-specific analysis in Stage II.

TABLE 10: Different Adversarial Contract Types in D<sub>GT</sub>.

<table><tr><td>Type ID</td><td>Attack Type</td><td>Num</td><td>Sum</td></tr><tr><td>Type1</td><td>Access Control Vulnerabilities</td><td>13</td><td></td></tr><tr><td>Type2</td><td>Price Oracle Manipulation</td><td>109</td><td></td></tr><tr><td>Type3</td><td>Logic Errors</td><td>29</td><td></td></tr><tr><td>Type4</td><td>Lack of Input Validation</td><td>7</td><td></td></tr><tr><td>Type5</td><td>Reentrancy Attacks</td><td>53</td><td rowspan="2"> $200^3$ </td></tr><tr><td>Type6</td><td>Unchecked External Calls</td><td>2</td></tr><tr><td>Type7</td><td>Flash Loan Attacks</td><td>21</td><td></td></tr><tr><td>Type8</td><td>Integer Overflow and Underflow</td><td>3</td><td></td></tr><tr><td>Type9</td><td>Insecure Randomness</td><td>5</td><td></td></tr><tr><td>Type10</td><td>Denial of Service (DoS) Attacks</td><td>0</td><td></td></tr><tr><td colspan="3">No Type</td><td>255</td></tr><tr><td colspan="3">Total</td><td>455</td></tr></table>

Figure 11: Prompt template for Baseline<sub>NL</sub>.  
§7.6.1. Figure 13 and Figure 14 show the function-centric and contract-centric summaries of this example.

## C.2. Benign Example: Comparison with Baseline<sub>NL</sub>

We conduct a detailed comparison using a benign contract (see Etherscan link) as a representative case supporting Obs1 and Obs3 in §3.3. Baseline tends to focus on lowlevel suspicious patterns while overlooking the high-level semantic intent of contract behavior, often leading to overly conservative judgments and false positives. In this example, it incorrectly labels a benign contract as adversarial because it “contains several unusual patterns and complex logic that are often seen in adversarial contracts”, even though these patterns represent legitimate access-control and compliance mechanisms. This misinterpretation reflects a lack of contextual reasoning; as noted in Obs3, Baseline also defaults to adversarial predictions in ambiguous cases involving nonstandard interactions. In contrast, FinDet performs multiview semantic analysis that integrates diverse behavioral signals. The six prompt scores are summarized in Table X, and after entropy-based fusion, the aggregated scores for adversarial, suspicion, uncertain, and benign are 0.1211, 0.2472, 0.3225, and 0.3092, respectively. The combined adversarial score (0.37) leads to the correct benign classification.

![](images/4e0fb61249785edbb0bf973105312e14b8df504201b87656df6edf8a966c3f2f.jpg)  
Figure 12: Performance metrics across varying adversarial score thresholds.

TABLE 11: Real-world attack cases detected by FinDet and publicly disclosed.

<table><tr><td>Deploy Time</td><td>Report</td><td>Adversarial Contract Address</td></tr><tr><td>2025.04.18</td><td>link</td><td>0x7a4d144307d2dfa2885887368e4cd4678db3c27a</td></tr><tr><td>2025.04.23</td><td>link</td><td>0xdd9a85fd532faadb0c439bbd725e571c4214aedf</td></tr><tr><td>2025.04.26</td><td>link</td><td>0xf6cee497dfe95a04faa26f3138f9244a4d92f942</td></tr><tr><td>2025.04.26</td><td>link</td><td>0x75f2002937507b826b727170728595fd45151d0f</td></tr><tr><td>2025.04.26</td><td>link</td><td>0xcfd3cf61619cbec15e9a8bef0e5cd613a565b6b3</td></tr></table>

![](images/1b968ffd363f410e8d13e89e73bc3647206860c7b270d3e225c7adfda0691fae.jpg)  
Figure 13: Function-centric summary of the contract in Figure 4 and Appendix C.1.

```txt
- contract summary: The contract appears to be designed to perform complex financial operations involving flash loans, token transfers, and balance checks. However, it includes specific restrictions on transaction origin, suggesting it may be intended for use by a single entity, potentially for exploitative purposes.
```  
Figure 14: Contract-centric summary of the contract in Figure 4 and Appendix C.1.

```txt
Path 1:
    caller -[it is required that (0x268d...4080 == sha3(tx.origin)), it is required that the 2nd external call succeeds, ...]-> stor_3.transfer
Path 2:
    DPPFlashLoanCall:param1 -[it is required that (0x268d...4080 == sha3(tx.origin)), ..., it is required that the 1st return data of the 3rd external call]-> stor_3.transfer
Path 3:
    unknownffffcf3a1:param1 -[it is required that (0x268d...4080 == sha3(tx.origin)), ...-> stor_5.flashLoan
```  
Figure 15: Fund-flow analysis of the contract in Figure 4 and Appendix C.1.

Overall, Baseline<sub>NL</sub>’s pattern-driven reasoning causes over-detection, whereas FinDet ’s holistic and confidenceaware fusion yields more accurate and interpretable outcomes, confirming the value of multi-level semantic reasoning for robust adversarial contract detection.

<table><tr><td>Prompt</td><td>G1</td><td>P1</td><td>G2</td><td>P2</td><td>G3</td><td>P3</td><td>G4</td><td>P4</td></tr><tr><td>prompt_g_normal</td><td>B</td><td>60</td><td>A</td><td>25</td><td>C</td><td>10</td><td>D</td><td>5</td></tr><tr><td>prompt_s_normal</td><td>B</td><td>60</td><td>A</td><td>30</td><td>C</td><td>8</td><td>D</td><td>2</td></tr><tr><td>prompt_g_mislead_ad</td><td>A</td><td>60</td><td>B</td><td>30</td><td>C</td><td>8</td><td>D</td><td>2</td></tr><tr><td>prompt_g_mislead_be</td><td>D</td><td>70</td><td>C</td><td>20</td><td>B</td><td>8</td><td>A</td><td>2</td></tr><tr><td>prompt_s_mislead_ad</td><td>A</td><td>80</td><td>B</td><td>15</td><td>C</td><td>5</td><td>D</td><td>0</td></tr><tr><td>prompt_s_mislead_be</td><td>D</td><td>60</td><td>C</td><td>30</td><td>B</td><td>10</td><td>A</td><td>0</td></tr></table>

TABLE 12: Predictions for the contract in Figure 4 and Appendix C.1.

![](images/2bde4a97290150d2202ec3f6fad788419b28a4bcefa5c82ac8831d8e8139e5d2.jpg)  
Figure 16: Baseline detection results for the benign contract in Appendix C.2.

<table><tr><td>Prompt</td><td>G1</td><td>P1</td><td>G2</td><td>P2</td><td>G3</td><td>P3</td><td>G4</td><td>P4</td></tr><tr><td>prompt_g_normal</td><td>D</td><td>50</td><td>C</td><td>30</td><td>B</td><td>15</td><td>A</td><td>5</td></tr><tr><td>prompt_s_normal</td><td>B</td><td>50</td><td>C</td><td>30</td><td>A</td><td>15</td><td>D</td><td>5</td></tr><tr><td>prompt_g_mislead_ad</td><td>C</td><td>40</td><td>D</td><td>30</td><td>B</td><td>20</td><td>A</td><td>10</td></tr><tr><td>prompt_g_mislead_be</td><td>D</td><td>70</td><td>C</td><td>20</td><td>B</td><td>8</td><td>A</td><td>2</td></tr><tr><td>prompt_s_mislead_ad</td><td>A</td><td>70</td><td>B</td><td>20</td><td>C</td><td>8</td><td>D</td><td>2</td></tr><tr><td>prompt_s_mislead_be</td><td>D</td><td>70</td><td>C</td><td>20</td><td>B</td><td>8</td><td>A</td><td>2</td></tr></table>

TABLE 13: Predictions for the contract in Appendix C.2.