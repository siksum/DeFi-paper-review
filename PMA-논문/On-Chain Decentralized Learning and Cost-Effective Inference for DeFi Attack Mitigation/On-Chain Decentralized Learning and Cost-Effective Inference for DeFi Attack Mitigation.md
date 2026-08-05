# On-Chain Decentralized Learning and Cost-Efective Inference for DeFi Attack Mitigation

Abdulrahman Alhaidari #

School of Computing and Information, University of Pittsburgh, Pittsburgh, PA, USA

Balaji Palanisamy #

School of Computing and Information, University of Pittsburgh, Pittsburgh, PA, USA

Prashant Krishnamurthy #

School of Computing and Information, University of Pittsburgh, Pittsburgh, PA, USA

## Abstract

Billions of dollars are lost every year in DeFi platforms by transactions exploiting business logic or accounting vulnerabilities. Existing defenses focus on static code analysis, public mempool screening, attacker contract detection, or trusted of-chain monitors, none of which prevents exploits submitted through private relays or malicious contracts that execute within the same block. We present the first decentralized, fully on-chain learning framework that: (i) performs gas-prohibitive computation on Layer-2 to reduce cost, (ii) propagates verified model updates to Layer-1, and (iii) enables gas-bounded, low-latency inference inside smart contracts. A novel Proof-of-Improvement (PoIm) protocol governs the training process and verifies each decentralized micro update as a self-verifying training transaction. Updates are accepted by PoIm only if they demonstrably improve at least one core metric (e.g., accuracy, F1-score, precision, or recall) on a public benchmark without degrading any of the other core metrics, while adversarial proposals get financially penalized through an adaptable test set for evolving threats. We develop quantization and loop-unrolling techniques that enable inference for logistic regression, SVM, MLPs, CNNs, and gated RNNs (with support for formally verified decision tree inference) within the Ethereum block gas limit, while remaining bit-exact to their of-chain counterparts, formally proven in Z3. We curate 298 unique real-world exploits (2020 - 2025) with 402 exploit transactions across eight EVM chains, collectively responsible for \$3.74 B in losses. We demonstrate that on-chain ML governed by PoIm detects previously unseen attacks with over 97% attack detection accuracy and 82.0% F1. A single inference, such as one made via an external call, typically incurs zero cost. Fully on-chain inference consumes 57,603 gas (≈ \$0.18) for linear models, 143,647 gas (≈ \$0.49) for CNN(F2, K1), and 506,397 gas (≈ \$1.77) for CNN(F8, K4) on L1 (e.g., Ethereum). Our results show that practical and continually evolving DeFi defenses can be embedded directly in protocol logic without trusted guardians, and our solution achieves highly cost-efective protection while filling a critical gap between vulnerability scanners and real-time transaction screening.

2012 ACM Subject Classification Security and privacy → Network security; Security and privacy → Distributed systems security; Security and privacy → Security protocols

Keywords and phrases DeFi attacks, on-chain machine learning, decentralized learning, real-time defense

Funding This material is based upon work supported by the National Science Foundation under Grant #2020071. Any opinions, findings, and conclusions or recommendations expressed in this material are those of the authors and do not necessarily reflect the views of the National Science Foundation.

## 1 Introduction

Vulnerabilities in decentralized finance (DeFi) protocols are triggered through transactions [8]. Attackers usually do not bypass contract safeguards directly [58], instead they craft trans actions that invoke legitimate functions to trigger state changes that the protocol did not intend. Exploits occur when contracts proceed with malicious inputs while assuming invariant state conditions, such as executing a withdrawal without checking balance or allowance or privileged operations without enforcing access control [52]. These attacks can be single transactions or atomic sequences that satisfy syntax checks but produce unauthorized asset control. The execution logic can be formally valid, yet produce outcomes that violate the protocol’s security assumptions [9]. Vulnerabilities come from code bugs and manipulating contracts. Attackers chain operations (for example, in atomic transactions) that appear normal in unexpected ways to generate exploits [12]. However, most previous work focuses on static code-level bugs [54] and overlooks protocol-level flaws [73].

Static analysis detects patterns that violate predefined coding conventions, but fails to capture logic flaws that depend on contract state, cross-function flows, or interactions across multiple protocols [10]. For instance, Cream Finance lost over \$130M through an interaction with another protocol that allowed borrowing without triggering the appropriate collateral checks [49]. Formal verification tools prove that certain invariants hold under all code paths but do not encode financial semantics or simulate attacker incentives. Most verification frameworks cannot model adversarially composed transaction sequences, in which attackers combine individually valid operations to produce exploits. Thus, zero-day vulnerabilities continue to appear in DeFi [73]. Tools for detecting malicious smart contracts assume a time window between contract deployment and first exploitation [56], during which vulnerabilities can be analyzed. In practice, attackers deploy and execute the exploit within a single block or bypass deployment entirely by sending malicious transactions from externally owned accounts (EOA) [43].

Other methods, such as post-attack analysis, provide insight into what has happened but ofer no protection. They begin only after an exploit has occurred and rely on retrospective debugging of protocol states [20]. However, recent attacks in DeFi have compromised over \$79.8 billion in DeFi assets and only \$6.7 billion of them have been recovered [19]. This emphasizes a core challenge in shared threat intelligence, particularly for aftermath attack analysis, where dissemination is usually delayed. This delay increases the likelihood of repeat attacks on existing deployed protocols that may harbor the same unaddressed vulnerabilities. Methods such as front-running protection [72] monitor public mempools (public queue of transactions) but miss transactions submitted through private relays such as Flashbots [26]. While of-chain monitoring systems (e.g., [72, 6, 56]) can detect some attacks, they typically focus on specific classes (e.g., flash loans, reentrancy). Moreover, their efectiveness is diminished by latency, which can lead to costly responses (due to gas fees) and reduced eficacy, particularly against sophisticated attacks [64]. Therefore, there is a need for protocol-integrated security mechanisms that add an evolving layer of protection (i.e., Intrusion Prevention System (IPS)) to smart contracts and react in real time without relying on external entities, such as pre-attack (e.g., auditing) or post-attack (i.e., attack tracing) countermeasures. On the other hand, DeFi protocols currently lack mechanisms to evaluate transaction intent during execution other than hard-coded logic, which makes them vulnerable to attacks that were not anticipated during smart contract design.

Solutions driven by machine learning (ML) show strong capabilities in attack detection [34, 39]. However, deploying ML models directly to Layer-1 blockchains such as Ethereum faces major obstacles in computation and storage. The computational demands of ML algorithms result in prohibitive gas costs. Ethereum’s block gas limits (≈ 30 million units per block [27]) impose constraints on the computational complexity of transactions, making complex ML models impossible to run in a single transaction. A basic model inference might consume 30-50% of an entire block. Storage costs create another barrier. Saving model parameters on-chain is extremely expensive [39]. These combined limitations make direct Layer-1 deployment of ML solutions economically and technically impractical for real-world deployment.

To address the above-mentioned gaps, we propose a decentralized training architecture where all model training and governance occur on Layer-2 (L2), while inference is optimized and happens on Layer-1 (L1) under strict gas constraints (Table 1). In our framework, an ML model is trained to provide a layer of security for smart contracts running on the Ethereum (L1) network. Our framework leverages L2 (e.g., Optimism rollup) for intensive computation and L1 for optimized inference to overcome L1 resource constraints (such as computation limits and gas fees). The rollup provides cost-efective computation while inheriting the base blockchain’s security guarantees [63]. Training is performed on L2, and L1 updates are governed by L2 decentralized nodes, where computation is cheaper, and the learned model is cryptographically verified and propagated to L1 for inference. Inference runs at zero cost through read-only classifiers (for example, pure or view functions) or is fully verified on L1 embedded in on-chain contracts for real-time transaction classification. To support a range of use cases, we propose two tiers: fully zero-cost inference (for users and smart contracts) on L1, and fully on-chain ML on L1. The system acts as a transaction gatekeeper (i.e., firewall) even for high-throughput or low-value use cases. Our approach is model-agnostic and supports various ML algorithms, including linear algorithms (e.g., logistic regression, SVMs) and non-linear models (e.g., neural networks up to 10 layers, including 10-layer convolutional neural networks (CNN)). These models are optimized and serialized into constant-time evaluation logic fully bounded by L1 constraints. We found that even low-overhead mechanisms (linear models) are suficient to detect a wide range of attacks. The models are trained using micro-steps by decentralized peers based on the collective knowledge of the DeFi platforms, and their formally verifiable performance matches traditional on-chain counterparts without approximation, enabling detection of both known and novel transaction behaviors.

As DeFi exploits evolve over time and to maintain continuous learning of new exploits as they appear, we introduce the notion of Proof-of-Improvement (PoIm). PoIm is a decentralized protocol on L2 that governs and verifies micro-step model training and its L1-propagated updates. A deployed model is designed to be shared globally across DeFi protocols, allowing any platform to contribute by training it on one candidate (malicious or normal) transaction at a time. These DeFi platforms collectively shape a model that becomes robust over time and enables unified sharing of attack intelligence. All updates are evaluated against an on-chain (L2) committed, agreed-upon benchmark of past exploit and benign transaction data. Updates are accepted only if they demonstrably improve at least one of the key performance metrics (namely precision, recall, accuracy, or F1-score) without degrading others, as evaluated by PoIm against the on-chain benchmark and it is verifiable by any node. Submitters are rewarded in proportion to the verified performance gain, while failed proposals lose their stake. If a malicious update seemingly improves metrics but enables a detection bypass, peers can vote to roll back the model to the last stable version. PoIm enables the L1 classifier to evolve with newly emerging exploits without relying on centralized oversight.

For our evaluation, since there is no publicly available transaction data for DeFi attacks, we manually collected 298 confirmed exploit transactions from real DeFi attacks across eight major blockchains: Ethereum, Binance Smart Chain (BSC), Polygon, Avalanche, Arbitrum, Fantom, Moonriver, and Base. Our exploit collection generalizes to cover attacks that exploited smart contract vulnerabilities in the past five years (2020 - April 2025), reported in news, social media platforms (e.g. X), blogs, and DeFi attack documentation. We utilize blockchain data from public explorers (for example, Etherscan [23]) and Web3 Remote Procedure Calls (RPCs) (e.g. using libraries/services like Web3.js [15] and Alchemy [4]), along with DeFiHackLabs [61], the DeFi Rekt Database [19], and DeFiLlama [20] as ground truth for guiding historical exploit collection. Each transaction contains the exact exploit transaction data used by the attacker, as observed by the smart contract at execution time, including call parameters, sender addresses (EOAs or smart contracts), and relevant blockchain state at the time of the exploit. In addition, we collected comprehensive metadata for each attack, including protocol names, exploited functions, attack methods, root causes, and financial losses. We use these real-world exploits, which collectively caused over \$3.74 billion in losses. Our evaluation shows that our approach is highly gas-eficient. CNN and RNN models governed by our decentralized PoIm protocol achieve over 97% recall and up to an 82% F1-score on unseen DeFi exploits.

Table 1 Comparison of our work with prior on-chain ML studies. DT = Decentralized Training, MC = Model Consistency (On-/Of-chain), G = Governance, DF = DeFi Focus, IC = Inference Cost, TM = Trust Model, VM = Validation Method.

<table><tr><td>Study</td><td>DT</td><td>MC</td><td>G</td><td>DF</td><td>IC</td><td>TM</td><td>VM</td></tr><tr><td>Our Work</td><td>✓</td><td>✓</td><td>✓</td><td>✓</td><td>Zero / Low</td><td>PoIm</td><td>On-chain + commit-reveal</td></tr><tr><td>ML2SC [44]</td><td>✗</td><td> $Partial^1$ </td><td>✗</td><td>✗</td><td>High</td><td>✗</td><td>✗</td></tr><tr><td>LMST [60]</td><td>✗</td><td> $Partial^2$ </td><td>✗</td><td>✗</td><td>Mod-High</td><td>✗</td><td>✗</td></tr><tr><td>opML [16]</td><td> $✘^3$ </td><td>✓</td><td>✓</td><td>✗</td><td>Low*</td><td>AnyTrust</td><td>Fraud-proof</td></tr></table>

<sup>1</sup> Minor mismatches due to PRBMath; <sup>2</sup> Accuracy drop from fixed-point; <sup>3</sup> Training of-chain, validated via fraud proof; \* Optimistic assumption, only O(1) on-chain arbitration.

Contributions. This paper makes the following contributions:

We design a decentralized learning framework with training and governance on L2 and enable two tiers: zero-cost inference and fee-optimized inference on L1 for real-time classification of DeFi transactions.

We introduce PoIm, a decentralized L2 protocol that governs model training and propagates verified updates to L1 for inference.

We formally verify inference correctness, model update integrity, and L1/L2 consistency under gas and computation constraints.

We evaluated our framework on a curated set of 298 manually collected real-world exploits. It achieves high detection performance: SVM reaches an F1-score of 80%, and CNN(F4, K4) achieves 82% F1, 0.9004 accuracy, and over 97% recall on unseen exploits. L1 inference is eficient, requiring only 57,603 gas for linear models and 143,647 gas for CNN(F2, K1). Zero-cost inference is supported via external EVM nodes.

## 2 Background

EVM blockchains. Ethereum is the largest Ethereum Virtual Machine (EVM) blockchain. It runs with a consensus mechanism as a peer-to-peer network that supports programmable logic or smart contracts executed by the EVM [27]. There are two main types of accounts: Externally controlled accounts (EOAs), operated by cryptographic keys, and smart contract accounts (SCAs), which execute embedded logic upon invocation [57]. EOAs initiate transactions, while SCAs automate protocol behavior or malicious activity [58].

Layer 2 blockchains. Layer 2 (L2) blockchains (e.g., Optimism, Arbitrum, Base) are built on top of Layer 1 (L1) platforms (e.g., Ethereum, Binance Smart Chain (BSC)) to improve scalability and reduce costs [59]. In contrast to L1s that execute and store every transaction directly on-chain [27], L2s batch many transactions and periodically submit compressed proofs to the L1 [67]. This design allows L2s to ofer significantly lower gas fees and faster execution and inherits the security guarantees from L1 [59]. Notably, L2s support the same smart contract logic as L1s but with relaxed resource constraints, which makes them ideal for computationally heavy tasks like model training or repeated inference.

Transactions. Transactions are the main component used to facilitate both asset movement and protocol interactions. A nonce in a transaction acts as a sequence counter to prevent replay attacks, where the attacker could replicate the transaction. The gas fee, influenced by user-specified price and the complexity of transactions, determines inclusion priority, while the gas limit defines the upper computational allowance per transaction [65].

Exploit transactions. Most of the DeFi attacks succeed through a maliciously crafted sequence of instructions (e.g., internal transactions) [21]. These attacks exploit weaknesses in smart contracts rather than modifying contract code at the infrastructure level [76]. Attackers manipulate transaction parameters, call sequences, and attempt to manipulate permission states through public interfaces to gain unauthorized assets [58].

Mempool and private relays. Blockchain transactions are submitted either through the public transaction bufer, known as the mempool, or via private relays [26], where the trans action is sent directly to miners. Transactions submitted via private relays remain hidden from the public, whereas those using the mempool are broadcast and await confirmation [14] Nodes share this queue across the network. The selection of which transactions to include in a block is typically based on miners/validators’ incentives and the gas fees ofered. Transactions with higher fees are generally prioritized [45]. The mempool is publicly visible, enabling a brief window during which real-time monitoring can be used to detect malicious behavior, such as front-running [38]. Since each node maintains a synchronized copy of unconfirmed transactions, this window supports early threat detection [32]. However, detection is not always possible due to the complexity of some attacks, and in many cases, the malicious transaction is submitted directly to miners/validators, bypassing public visibility [48].

DeFi and smart contract vulnerabilities. Decentralized finance (DeFi) is blockchain based financial services operating without centralized intermediaries [13]. Smart contracts, self-executing programs deployed on blockchain platforms such as Ethereum, form the backbone of DeFi platforms [27]. These contracts facilitate autonomous, trust-minimized interactions, enabling services such as decentralized exchanges (DEXs), lending, asset man agement, and stablecoins. The complexity and transparency of smart contracts introduce significant risks. Code-level vulnerabilities, flawed logic, and unintended transaction se quences can lead to severe security breaches and substantial financial loss [35]. Common exploit classes include reentrancy attacks, access control failures, approval abuses, and flash loan-based manipulations [66]. Exploits often occur rapidly and irreversibly, exploiting the immutable nature of blockchain transactions. Although these exploit classes are studied, there is a need for defense solutions that are capable of mitigating malicious transaction behaviors in real time for diverse exploit types [21].

## 2.1 Limitations of Existing Defenses

This section discusses fundamental limitations of existing preventive measures and real-time defense mechanisms. First, of-chain detection systems such as LookAhead [56] rely on a temporal gap between transaction submission and execution. When an attacker deploys a malicious contract and immediately initiates an exploit within a single block, these systems fail to respond on time. Second, private relay services like Flashbots [26] allow attackers to bypass mempool-based detection by submitting transactions directly to miners/validators. In such settings, traditional monitoring tools lose all visibility, allowing stealthy attacks without external traceability.

Third, signature-based detection methods depend on identifying known function selectors or matching call patterns against static signatures [6]. This approach fails when attackers use proxy contracts, delegate calls, or obfuscated logic flows, where surface-level transaction signatures are intentionally masked. Fourth, current on-chain defenses lack the ability to validate transaction behavior dynamically during execution. Existing systems either rely on static pre-deployment audits or basic access control checks, leaving dynamic runtime behavior, such as unauthorized fund movements or privilege escalations, undetected.

Fifth, most prior defenses prioritize contract vulnerabilities rather than transaction behavior. However, even with this focus, a recent study [10] found that automated tools can detect only 8% of vulnerabilities. Another study found that existing tools can detect only 20% of vulnerabilities [73], which leaves 80% undetected. Consequently, they can be exploited and cause huge financial losses. One reason is that not all bugs are exploitable, and not all correct logic code is vulnerability-free [52]. Thus, the focus on code logic overlooks the fact that many exploits occur through crafted transaction sequences exploiting valid contract interfaces without exploiting traditional code vulnerabilities.

Also, existing solutions typically adopt a centralized approach. Systems that depend on trusted of-chain detectors or external alert mechanisms inherently introduce single points of failure and trust dependencies incompatible with DeFi principles.

Finally, another category of defense focuses on pre-interaction analysis to identify inconsistencies between a project’s documentation and its on-chain bytecode. DeFiAligner [29], for instance, uses symbolic execution and Large Language Models to detect when the implementation and documented logic of a smart contract do not match. This approach is valuable for auditing projects and protecting users from being misled by inaccurate documentation. However, it does not provide real-time defense against malicious transaction behavior when it is executed. It also cannot detect a novel exploit sequence that uses valid functions in an unexpected (malicious) way, which is the gap our work addresses.

## 2.2 Limitations of On-Chain ML

Embedding machine learning models into L1 smart contracts is resource-intensive, and replicating of-chain models directly on-chain can be impossible. L1 blockchains are not designed for heavy computing [3]. However, inference, if designed without inheriting the complexity of of-chain models (e.g., using TensorFlow [2], Keras [40], Scikit-learn [41]) yet performing equivalently, can be feasible if heavy computations are separated from inference.

Previous designs (e.g. [44, 60]) face many practical barriers, including prohibitive gas costs for inference, inconsistent behavior between of-chain and on-chain models due to numerical limitations, and incompatibility with smart contract languages lacking native support for floating-point operations, which causes model output deviation (of-chain vs onchain counterpart). For example, the authors in [60] reported inconsistent accuracy between the PyTorch and on-chain models: 86.00% of-chain versus 81.00% on-chain. The deployment cost also exceeds the current Ethereum block gas limit, with a reported cost of 73,721,648 gas. This is nearly twice Ethereum’s current block gas limit, which is approximately 30 million gas units.

Our work achieves exact consistency (formally and empirically verified) between of-chain and on-chain model outputs. Unlike prior methods, our on-chain design uses an optimized smart contract that replicates the of-chain model’s behavior without approximation to achieve the same outcome. This will allow DeFi attacks mitigation dynamically using practical, low-cost, and fully decentralized on-chain machine learning.

![](images/60866bba777e6fcc496f2f31f7a1101ca2f4d13a385c0a2c1047b5aeed639476.jpg)  
Original tx. feature space $\mathcal { X } \subset \mathbb { R } ^ { 7 }$  
Figure 1 PCA reduces 7D attack transaction features from the original feature space $\mathcal { X } \subset \mathbb { R } ^ { 7 }$ to a 2D latent space $\mathcal { F } \subset \mathbb { R } ^ { 2 }$ for pattern clustering.

## 3 DeFi Attacks and Threat Model

The current state of defenses is not suficient to mitigate the growing threat of unauthorized fund exploits in Decentralized Finance (DeFi). Recent studies [10, 42, 14, 76, 73] show that attack techniques are rapidly advancing but defenses remain inadequate. To address these threats, we need a new defense model based on a real-time, in-protocol (smart contract-level) transaction classification system that must operate independently of external monitors or pre-defined signature lists [76], and it must work even when transactions are hidden from the public mempool. It should also generalize to both known and novel (zero-day) attacks that exploit DeFi contract flaws. However, statistical methods have shown significant promise in detecting not only known attacks but also previously unseen zero-day exploits [34]. This naturally raises several key questions: Can data from past DeFi attacks help us predict and prevent future ones? What are the possible designs for trustless, cost-efective, on-chain inference systems that can evolve with the attack landscape and be governed by decentralized peers? To answer these questions, we investigate whether real historical exploits exhibit detectable patterns that statistical techniques can reliably uncover and learn from.

## 3.1 Empirical Patterns of DeFi Attacks

To explore the feasibility of detecting DeFi attacks using only transaction data observable by smart contracts, we analyzed 402 confirmed DeFi exploit transactions (for 298 attacks). Transaction data includes complete EVM-level execution context such as gas usage, transferred value, calldata, and block metadata. Since previous studies ofer limited analysis of DeFi attacks from a transactional perspective, we investigate whether exploit transactions exhibit measurable similarities that support transaction-level detection.

We selected the numerical features observable during smart contract execution (Table 2) and excluded transaction hashes or addresses. Features were standardized, and dimensionality was reduced (Figure 1) using Principal Component Analysis (PCA). We applied KMeans clustering with k = 3 to detect grouping behavior across attack transactions.

The PCA projection preserved 46.3% of the variance in two dimensions (Figure 2). Out of 402 exploits, 355 (88.3%) formed a distinct cluster. This suggests high behavioral consistency across transactions despite protocol and chain diferences. Clustering quality was supported by a Calinski-Harabasz index of 981.78 and a Silhouette score of 0.773, both indicating well-separated, compact clusters.

![](images/cb478212be3066f61a896a0f4dc8e1b719cc9c056e6d576fb2de947c32f51ff4.jpg)

![](images/eef360ab3c633ee5d288fbdaa9be236692c973de660abace8c68c89c9ad354cf.jpg)  
Figure 2 Left: PCA projection of DeFi attack transactions using two principal components with KMeans clustering (k = 3). Right: The scree plot shows the explained variance ratio per principa component. Even though PC1 and PC2 capture less than 50% of the total variance, the projection shows distinct DeFi attack transaction patterns.

Table 2 Transaction Metadata Visible to Smart Contracts During Execution

<table><tr><td>Attribute</td><td>Description</td></tr><tr><td>msg.sender</td><td>Sender address (initiator of the call)</td></tr><tr><td>msg.value</td><td>ETH amount sent with the call</td></tr><tr><td>msg.data</td><td>Calldata containing function selector and arguments</td></tr><tr><td>tx.origin</td><td>Original externally-owned sender of the transaction</td></tr><tr><td>gas</td><td>Remaining gas (via gasleft())</td></tr><tr><td>gas_price</td><td>Effective gas price paid (via tx.gasprice)</td></tr><tr><td>msg.to</td><td>Receiving contract&#x27;s address (via address(this))</td></tr></table>

These findings indicate that DeFi attacks share consistent runtime characteristics. The ability of PCA + KMeans to cluster these attacks supports the feasibility of transactionlevel classification during execution. We classify the root causes of these attacks into five empirically supported categories, such as access control failures and business logic flaws (see Table C in Appendix).

Exploit Execution Patterns. Despite diferent root causes, several runtime-level patterns are consistent across attacks:

Atomicity: 71.2% of exploits execute within a single transaction with no prior on-chain activity.

Benign interface misuse: 67.5% invoke functions like withdraw(), approve(), or sweep() with malicious arguments.

Cross-chain replication: Identical bytecode deployments (9.7%) are exploited across chains [37] (e.g., Hedgey [1]).

These patterns further support the view that exploitability is determined by transaction behavior rather than only static code.

## 3.2 Challenges in Real-Time Defense

We identified several key challenges that prevent current systems from defending against these attacks in real time:

C1. Private relay invisibility: Transactions sent through Flashbots [26] and similar relays bypass the public mempool, evading pre-inclusion detection.

C2. Multiple attack paths per contract: A single protocol may contain unrelated flaws, making static patching or signature detection inefective.

C3. Benign-looking interfaces: Safe-looking functions are abused with crafted inputs, undermining static signature-based detection.

C4. Cross-chain propagation: An undetected exploit on one chain quickly propagates to other protocols with shared logic.

C5. No rollback: Once executed, DeFi transactions are final. Most protocols do not have pause switches or delayed execution.

## 3.3 Threat Model and Assumptions

Our system defends against two primary threat scenarios. ① An attacker crafts a malicious transaction targeting a vulnerable contract. The transaction is evaluated by our on-chain ML classifier before execution. The attacker may vary calldata, timing, or submission channel (e.g., private relay), but cannot alter the deployed model or governance system. ② A malicious peer attempts to poison the training process by submitting manipulated updates that degrade model performance. The system detects and rejects such updates via on-chain benchmarking. Insider threats, compromised signing keys, and of-chain infrastructure attacks are out of scope for this threat model.

![](images/2fbd99b40fad94057976df2c1d913a947f1c3ed348c7e1a55f03478ddedd5370.jpg)  
Figure 3 Framework overview.

## 4 Decentralized Training, Inference, and Governance Framework

Our objective is to integrate decentralized learning with verifiable and eficient on-chain detection into the execution paths of DeFi contracts to mitigate exploit transactions in real time. We address limitations in existing works (e.g., [44, 72, 5, 56, 60]) from two perspectives: DeFi attack mitigation and on-chain ML design. First, we detect private relay attack transactions and generalize the detection mechanism to unseen attacks in real time. Second, we ensure bitwise consistency between of-chain and on-chain models, and cost-efective decentralized training and inference. We also guarantee bounded resource consumption, such as gas fees for training and inference. Furthermore, we introduce a decentralized mechanism that enables peers (i.e., DeFi platforms) to collectively train ML/DL models (e.g., for attack defense) by proposing training samples. These proposed updates are transparently governed, validated, and challengeable by others. An overview of our end-to-end framework is presented in Figure 3.

Our framework enables decentralized learning by translating traditional of-chain machine learning and deep learning model architectures into gas-eficient, formally verified Solidity contracts for blockchain execution via L2 rollups (e.g., Optimism [50]). It supports a wide range of architectures, from simple to complex, and produces tamper-proof ML/DL contracts with quantized parameters serialized for on-chain weight propagation (Figure 4). This capability not only enhances DeFi threat detection but also provides a foundation for developing adaptive and secure financial systems in other domains. Our approach addresses the limitations of prior work discussed in Section 2.2. Previous eforts (e.g., [44, 60]) are impractical and incur high gas costs when deploying complex models on L1, which can be very complex.

In contrast, our design supports full-scale, real-world models without compromising accuracy due to Solidity’s floating-point limitations, and it enables fully decentralized training and model updates. To achieve this, we introduce Proof-of-Improvement (PoIm), a decentralized governance protocol that tracks each training step and governs model updates. It allows anyone (e.g., any DeFi platform) to incrementally train the model, efectively proposing updates that improve detection metrics. Updates are predictably propagated from L2, where computation and verification are performed, to L1, where inference takes place. Peers can collectively override a propagated model update on L1 in cases of suspected malicious training (poisoning) by another peer. Overall, our decentralized design is motivated by DeFi security requirements, and we show that a decentralized, dynamic, and cost-efective ML/DL-based defense is feasible under current blockchain architectures enabled by L2, capable of mitigating attacks that have caused billions in financial losses.

![](images/0127289c3e01b1f9638355b6f45cf3a4be1c91a3a70fe0e2b1bca15a67bdf320.jpg)  
Figure 4 Serialization example of quantized model parameters.

## 4.1 Decentralized Micro-Step Training and Model Evolution

Our system enables verifiable, decentralized training by decomposing the learning process into micro-step updates. Each micro-step corresponds to a single incremental improvement, such as retraining on one example or applying a localized adjustment, and is proposed directly on-chain. All proposals are immediatel evaluated usin a canonical, public test set stored on-chain.

Only updates that improve at least one core metric (accuracy, precision, recall, or F1- score) without degrading any others are accepted. This logic is enforced deterministically by the PoIm contract, which governs model evolution and ensures that every change is auditable, trustless, and resistant to adversarial manipulation.

Every accepted or rejected update, along with its evaluation results, is permanently logged on-chain. This guarantees transparent, tamper-proof model provenance and allows peers to track, audit, or challenge any step of the training process. In contrast to centralized retraining, this decentralized micro-step protocol allows the model to evolve continuously and securely, driven entirely by peer contributions and verified in real time on-chain.

## 4.2 Inference Architecture

In our architecture, we support two cost-efective tiers for executing ML/DL inference over blockchain networks. Each tier provides a diferent trade-of between execution cost, verifiability, and decentralization.

On-chain Logic, Of-chain Execution (zero-cost). In this tier, the model parameters and execution logic are stored fully on-chain. However, the actual inference computation is performed of-chain by calling view functions, which are executed by any EVM client. This tier incurs zero gas cost for DeFi users or protocols while ensuring that the decision logic is derived directly from verifiable on-chain bytecode and model state. Since every node executes the same bytecode deterministically, outcomes are consistent and tamper-resistant. This design is ideal for platforms seeking lightweight classification with full code transparency and no execution fees.

Fully On-chain Inference (inference verifiable on-chain). Here, the inference is executed entirely on-chain as part of a state-modifying transaction, typically one that interacts with a DeFi protocol. The input is passed to the smart contract, which executes the classifier internally and enforces the classification result. This enables end-to-end verifiability and enforces decisions during transaction execution (e.g., rejecting or allowing access to protocol funds). While this incurs gas costs, our optimized design allows even moderately complex models, such as 10-layer CNNs with quantized integer arithmetic, to execute eficiently within Ethereum’s gas limits. This tier is suited for scenarios like high TVL or security-critical DeFi functions, where classification must be verified on-chain without relying on of-chain entities to interpret results, as the outcome is enforced by on-chain consensus.

## 4.3 Layer-2 to Layer-1 Computation Separation

Model parameters are updated through decentralized training and governed by the Proof of-Improvement (PoIm) protocol on Layer-2 (L2), where gas costs are significantly lower. These L2-validated model parameters must then be securely and accurately propagated to Layer 1 (L1) for use by inference contracts in either Tier 1 or Tier 2. To ensure that L1 inference contract parameters do not get tampered, we employ a commit-verify propagation mechanism.

Commit to Model Hash (L2). Upon acceptance of a training update (yielding new parameters θ<sup>′</sup>) by the PoIm protocol on L2, the contract computes a cryptographic hash modelHash = keccak256(abi.encodePacked(θ<sup>′</sup>)). This hash serves as a tamper-proof commitment to the new model.

Transmit Commitment (L2 → L1). The L2 contract sends this modelHash to L1 via the native L2-to-L1 bridge (e.g., Optimism’s L2ToL1MessagePasser). The hash is recorded by the L1 contract that manages model updates.

![](images/d5966ffec9521f41860850f0aaf2ca87dd0e3e7a14f13c1c93b1692ce4f1219d.jpg)  
Figure 5 PoIm protocol update flow. Each proposed model is evaluated on $\mathcal { D } _ { \mathrm { t e s t } }$ and accepted only if it improves performance.

Transmit Parameters and Verify $( { \bf L 2 }  { \bf L 1 } )$ . In a subsequent transaction, the full model parameters $\theta ^ { \prime }$ are transmitted. The L1 contract recomputes the hash and verifies it against the prior commitment. A mismatch results in the rejection of the update [62].

## 4.4 Formal Bit-Exact Verification

Weights are scaled by $S \in [ 1 0 ^ { 6 } , 1 0 ^ { 1 8 } ]$ then packed into int32[] and up-cast to int128 where safe. A fully-connected layer l executes $z ^ { ( l ) } = \mathrm { i d i v } \big ( \hat { W } ^ { ( l ) } z ^ { ( l - 1 ) } + \hat { b } ^ { ( l ) } , S \big )$ If $\gamma > \Delta$ , sign consistency holds for all validation inputs. For every compiled model, we prove $\forall x \in \mathbb { Z } ^ { d }$ $\mathcal { F } _ { \mathrm { o n } } ( x ) = \mathcal { F } _ { \mathrm { o f f } } ( x )$ under fixed-point scale S. We encode both paths as bit-vector (e.g., 256-bit) formulas and ask Z3 for expr $\mathbf { \varepsilon } _ { - } \circ \mathbf { n } \neq \mathbf { e x p r } _ { - } \circ \mathbf { f } \mathbf { f }$ . All models (linear, CNN, RNN) return unsat, giving a machine-checked guarantee of equality. This design-time formal proof ofers a strong guarantee that the compiled on-chain model faithfully implements the intended of-chain model logic under the specified fixed-point representation for all possible inputs. It ensures the intrinsic correctness of the model’s translation to Solidity. This is distinct from, yet complementary to, the operational consistency checks performed during the lifecycle of the model on-chain. Listing 1 provides an example of a forward pass implemented in Solidity using fixed-point arithmetic.

Listing 1 Forward pass example for layer l using fixed-point arithmetic.

```c
for (uint i = 0; i < d_1; i++) {
    z[i] = bias_1[i];
    for (uint j = 0; j < d_{1-1}; j++) {
        z[i] += idiv(weights_1[i * d_{1-1} + j] * input[j], SCALE);
    }
}
```

Beyond the formal verification of the model logic itself, our protocol incorporates runtime consistency checks at critical junctures, such as model updates on L2, propagation to L1, and sample inferences, to ensure operational integrity in runtime.

## 4.5 Gas Cost and Runtime Bound Analysis

Opcode Budget. For every multiply-accumulate $\begin{array} { r } { ( \mathrm { M A C } ) \ a \gets a + \frac { w \cdot x } { S } } \end{array}$ inside classi $\mathbf { f } { \mathfrak { y } } .$ the EVM executes (1) $\mathtt { S L O A D } ( w ) = G _ { S } = 1 0 0$ gas (warm), (2) CALLDAT $\mathtt { A L O A D } ( x ) = G _ { C } = 3$ gas, (3) $\mathsf { M U L } = G _ { M } = 5$ gas, (4) D $\mathtt { U } = G _ { D } = 5$ gas, (5) $\mathtt { A D D } = G _ { A } = 3$ gas, (6) loop/stack bookkeeping ≈ $G _ { L } = 8 ~ \mathrm { g a s . ^ { 1 } }$

![](images/5e8d5fec0e78a6ccc8c15413a236cfe066c2b0a5f01f6cab9144394e3b49036c.jpg)

Figure 6 PoIm overview.

Hence, the cost per MAC is $g _ { \mathrm { M A C } } = G _ { S } + G _ { C } + G _ { M } + G _ { D } + G _ { A } + G _ { L } = 1 2 4$ gas. ReLU adds $G _ { R } = 5$ gas per activation, and bias initialization costs $G _ { S } + G _ { A } = 1 0 3 ~ \mathrm { g a s } ,$ Linear Classifiers (LR / SVM). A linear model with d inputs executes one MAC per feature and adds a bias. Hence, $\begin{array} { r } { \Big | G _ { \mathrm { L I N } } ( d ) = 1 2 4 d + 1 0 3 \Big | . } \end{array}$

For logistic regression, we approximate $\sigma ( z )$ by a threshold on the logit, therefore no extra exponentiation is incurred; the cost matches SVM. With d=3 (example feature set) the bound gives $G _ { \mathrm { L I N } } ( 3 ) = 1 2 4 \times 3 + 1 0 3 = 4 7 5$ gas, which is three orders of magnitude below the deep models and negligible at call-sites.

CNN Bound. For an input of length $d ,$ kernel size $K .$ , and $F$ filters, the convolution yields $o = d - K + 1$ positions and evaluates $F o K \mathrm { M A C s } ;$ the fully connected read-out contributes a further F o MACs. Thus, $\boxed { G _ { \mathrm { C N N } } ( d , K , F ) = g _ { \mathrm { M A C } } F o ( K + 1 ) + G _ { R } F o + 1 0 3 F } \boxed { ( 1 ) }$ RNN Bound. Let U be the hidden-state size, T the number of time steps, and $d _ { \mathrm { i n } } = d / T$ the per-step input width. A gated update performs $U ( d _ { \mathrm { i n } } + U )$ MAC operations per time step. The total cost is $\Big | G _ { \mathrm { R N N } } ( d , U , T ) = g _ { \mathrm { M A C } } T U ( d _ { \mathrm { i n } } + U + 1 ) + 1 5 T U \Big |$ (2). We fix the total input dimensionality $d = 3$ for both architectures. Table 3 shows the resulting gas bounds for several example CNN and RNN configurations.

<table><tr><td>Model</td><td>F</td><td>K</td><td>Bound</td></tr><tr><td> $CNN_{2\times 2}$ </td><td>2</td><td>2</td><td>1714</td></tr><tr><td> $CNN_{4\times 2}$ </td><td>4</td><td>2</td><td>3428</td></tr><tr><td> $CNN_{8\times 3}$ </td><td>8</td><td>3</td><td>4832</td></tr></table>

(1)

<table><tr><td>Model</td><td>U</td><td>T</td><td>Bound</td></tr><tr><td> $RNN_{4 \times 2}$ </td><td>4</td><td>2</td><td>7064</td></tr><tr><td> $RNN_{8 \times 4}$ </td><td>8</td><td>4</td><td>40160</td></tr></table>

(2)  
Table 3 Analytic MAC bounds for CNN and RNN models with total input dimension $d = 3 .$

## 4.6 Decentralized Model Update

Only models that show improvement in multiple evaluation metrics are accepted. No external oracle trust is required and all updates are verifiable on-chain within strict gas constraints.

## Table 4 Notations used.

<table><tr><td>Symbol</td><td>Description</td></tr><tr><td> $\theta, \theta'$ </td><td>current and proposed weight vectors</td></tr><tr><td> $b, b'$ </td><td>current and proposed bias scalars</td></tr><tr><td> $S$ </td><td>scaling constant,  $S = 10^{x}$ </td></tr><tr><td> $\mathcal{D}_{\text{test}} = \{(x_{i}, y_{i})\}_{i=1}^{n}$ </td><td>on-chain evaluation dataset</td></tr><tr><td> $x_{i} \in \mathbb{R}^{d}, y_{i} \in \{0, 1\}$ </td><td>feature vector and label</td></tr><tr><td> $f_{\theta}(\cdot)$ </td><td>classifier parameterized by  $(\theta, b)$ </td></tr><tr><td> $s$ </td><td>stake (in wei) deposited by proposer</td></tr><tr><td> $M, M', M_{\text{old}}, \widetilde{M}$ </td><td>metric vectors before/after update</td></tr><tr><td> $\Delta t$ </td><td>challenge window (e.g., 1-day)</td></tr></table>

Proof of Improvement (PoIm). Let $f _ { \theta }$ denote the deployed classifier with an immutable architecture and on-chain weights θ (see Table 4 for notation). The core of our PoIm mechanism relies on $\mathcal { D } _ { \mathrm { t e s t } } = \{ ( x _ { i } , y _ { i } ) \} _ { i = 1 } ^ { n }$ , a canonical evaluation dataset stored directly on-chain. This dataset is intentionally curated to be compact, yet it is representative of critical attack vectors and desired model behaviors rather than being an exhaustive list of all historical transactions. Its manageable size is crucial for enabling eficient on-chain evaluation of proposed model updates within gas limits. Furthermore, to maintain its relevance and resist ossification, $\mathcal { D } _ { \mathrm { t e s t } }$ itself is governed by a decentralized peer-based mechanism (e.g., DAO voting), allowing for agreed-upon additions, modifications, or removals of test samples over time (see Figure 5). Let $\theta ^ { \prime }$ represent a submitted update. We define the classifier $( \mathrm { i . e . } ,$ linear) as $\begin{array} { r l r } { f _ { \theta } ( x ) ~ = ~ \mathrm { s i g n } \Big ( \frac { 1 } { S } \sum _ { i = 1 } ^ { d } \theta _ { i } x _ { i } ~ + ~ b \Big ) } & { { } \in \{ 0 , 1 \} } \end{array}$ . We define the evaluation function as $\operatorname { E v a l } ( f _ { \theta } , \mathcal { D } _ { \mathrm { t e s t } } ) \to ( \operatorname { A c c } , \operatorname { F 1 } , \operatorname { P r e c } , \operatorname { R e c } )$ , where all metrics are computed deterministically using Solidity logic and can be challenged.

New Training Submission. Users can propose a new model update by submitting a new training sample directly to the PoIm contract, which yields a change (if accepted) in new model weights $\theta ^ { \prime }$ and biases $b ^ { \prime } ,$ , and staking a minimum amount of tokens (or, $\mathrm { e . g . }$ , any ERC20 token). Each submission must submit a stake, $\mathrm { e . g . } , s > 0$ . The staked ERC20 serves as collateral for the proposal. If accepted, the contributor receives: $R \ =$ $\begin{array} { r } { s ~ + ~ \sum _ { k \in \{ \mathrm { A c c } , \mathrm { F 1 } , \mathrm { P r e c } , \mathrm { R e c } \} } \alpha _ { k } \left( M _ { k } ^ { \prime } - M _ { k } \right) } \end{array}$ . Each coeficient $\alpha _ { i } \geq 0$ reflects the vault’s value weighting for each metric improvement. For instance, if the vault has accumulated 1 ETH from failed update attempts, the payout R is proportionally distributed based on the magnitude of improvement across the four metrics.

Model Update Acceptance. An update $\theta ^ { \prime }$ is accepted if it improves at least one core metric (accuracy, precision, recall, or F1-score) without degrading any of the other core metrics compared to the current model, based on on-chain evaluation over $\mathcal { D } _ { \mathrm { t e s t } }$ . The contract enforces this by ensuring that any accepted model update demonstrates improvement in at least one core metric without degrading others, thereby preventing overfitting a single target $( \mathrm { e . g . }$ , maximizing recall while degrading precision). This improvement is verified on-chain on $\mathcal { D } _ { \mathrm { t e s t } }$ , which ensures that the proposed model performs better than the current one.

Test Set and Adversarial Update Rollback. We integrate an on-chain DAO mechanism that allows stakers (e.g, DeFi platforms) to collectively manage both model rollbacks and $\mathcal { D } _ { \mathrm { t e s t } }$ test set updates. In our design, participants stake tokens to gain voting rights to propose and vote on critical actions such as adding or modifying test samples and reverting model weights (to previously committed L1 update) if suspicious behavior is observed (e.g., malicious training sample).

In the case of a challenged update (i.e., one that introduces a loophole but still satisfies the acceptance criteria), a revealed update can be rolled back within a fixed period (e.g., <7 days). If the new model weights worsen in any performance metric compared to the previous metrics, the proposer loses their collateral and the proposal is discarded.

## 5 Evaluation

This section details our experimental setup, dataset construction, model configurations, and metrics used to evaluate our proposed framework for on-chain DeFi exploit detection and mitigation across various machine learning architectures. We also present a quantitative comparison against a baseline for on-chain ML.

![](images/1457a76a99df8e8790f21596954a9f5e3b79ca7bbe54702b9e4d3484462789d4.jpg)  
Figure 7 Distributions of normal and attack transactions.

## 5.1 Experimental Setup

All of-chain computations, including model training, parameter generation for Proof-of-Improvement (PoIm) proposals, and performance metric calculations, were developed on a machine equipped with an Intel Core i7 CPU and 16GB of RAM, running Windows 10. Key Python libraries we utilized include scikit-learn [41], pandas [47] (e.g., for data manipulation), NumPy [36] (e.g., for numerical operations), web3.py (e.g., for blockchain interaction), py-solc-x (for Solidity compilation), and the Z3-solver [18] (e.g., for formal verification logic).

For blockchain interactions, local L1 and L2 test environments were used. The L2 environment was configured using a Hardhat Network instance forking a live Optimism L2 rollup state (e.g., Optimism Mainnet), facilitating realistic gas calculations and execution behavior mirroring Optimism’s characteristics. This L2 environment is primarily used for the decentralized training and PoIm mechanism. The L1 environment was simulated using Anvil[28], configured for persistence across experimental runs to reflect a stable mainnet-like chain.

Attack Dataset Construction. A significant challenge in evaluating DeFi exploit detection systems is the general absence of comprehensive, publicly available, raw transaction datasets suitable for behavioral modeling. While some prior work has focused on smart contract code analysis [74, 68, 71], readily usable transactional datasets for exploit detection are scarce. To address this, we undertook a meticulous manual collection and multi-stage verification process to construct a robust dataset for this study, covering attacks from 2020 to 2025. Our initial identification of potential exploits involved a broad survey of diverse sources, including industry security news (e.g., Rekt News [19]), detailed analyses on technical blogs, discussions on social media platforms (e.g., X), and curated public incident databases such as the DeFi Rekt Database [19] and DeFiLlama Exploits Dashboard [20]. Incidents and leads gathered from these channels, often further indexed or summarized by resources like DeFiHackLabs [61], guided our targeted retrieval of the specific transactions that executed each confirmed attack.

These transactions are primarily identified by their hashes on public blockchain explorers (e.g., Etherscan [23], Polygonscan [55]). All transactional records were independently retrieved and subsequently verified through direct blockchain queries via Web3 RPCs [15] to ensure authenticity and completeness of call data, receipts, and traces. The primary features extracted for model input include: gas (gas limit provided by sender), block.timestamp, func\_selector\_encoded (a label-encoded representation of the first four bytes of msg.data) chain ID (label-encoded), msg.sender (label-encoded), tx.origin (label-encoded), and msg.to (label-encoded). Numerical features were standardized, and categorical features were label-encoded. Features such as block.timestamp may overfit if the same timestamp is found in both training and testing. To mitigate this risk, we implemented a strict temporal train-test split. Training is exclusively on historical data, and testing is on future (unseen attacks). Therefore, the model generalizes better based on transaction behaviors rather than memorizing temporal artifacts. Transaction traces and contract interactions were analyzed to produce contextual annotations, including attack\_name, links to exploited contract source files, incident report URLs, dates, identified root causes, and financial loss in USD (normalized to the time of exploit). This process was applied to transactions across multiple EVM-compatible blockchains, including Ethereum, Binance Smart Chain (BSC), Polygon, Avalanche, Arbitrum, Fantom, Moonriver, and Base. Table 5 presents an illustrative subset of these attributes.

Table 5 Example of a subset of fields from our exploit transaction dataset. Full transaction records include additional attributes such as detailed EVM context, blockchain metadata, and semantic annotations.

<table><tr><td>Field</td><td>Example</td><td>Description</td></tr><tr><td>tx_hash</td><td>0x78d7...2df4</td><td>Unique identifier of the exploit transaction</td></tr><tr><td>msg.sender</td><td>0x5aa5...30b7</td><td>Address that initiated the exploit</td></tr><tr><td>msg.value</td><td>0</td><td>ETH directly transferred in the transaction</td></tr><tr><td>gas_used</td><td>298210</td><td>Total gas consumed during execution</td></tr><tr><td>block.timestamp</td><td>1688462834</td><td>Unix timestamp of the exploit</td></tr><tr><td>root_cause</td><td>Unchecked external call</td><td>Vulnerability exploited in the contract</td></tr><tr><td>loss_USD</td><td>$1.94M</td><td>Financial loss via unauthorized token transfers</td></tr><tr><td>chain</td><td>Ethereum</td><td>Blockchain on which the exploit occurred</td></tr></table>

Only a subset of fields is shown here.

We observed that an attack can manifest in a single atomic transaction (approximately 71% of the attacks in our dataset) or across multiple transactions. For multi-transaction attacks, all constituent transactions were grouped and assigned entirely to either the training or testing set to prevent data leakage and maintain the integrity of the attack sequence

The combined dataset of normal and attack transactions was then sorted chronologically by block.timestamp. Table 6 summarizes the distribution of the unique exploits that informed the construction of our dataset.

Table 6 Distribution of unique exploits (2020- 2025) and for DeFi financial losses.

<table><tr><td>Category</td><td>Count</td><td>Loss (USD)</td></tr><tr><td>Total distinct exploits</td><td>298</td><td>-</td></tr><tr><td>Exploits in training set</td><td>202</td><td>$1,877,229,549.86</td></tr><tr><td>Exploits in testing set</td><td>96</td><td>$1,858,400,900.66</td></tr><tr><td>Total exploits loss</td><td>-</td><td>$3,735,630,450.52</td></tr></table>

The resulting transaction data capture fine-grained, executable-level behaviors and intricate attacker-victim dynamics. This detailed resolution enables precise detection and classification of exploits and provides a foundation for exploring potential attack discovery. The final curated dataset of 298 unique attack vectors, corresponding to confirmed financial losses exceeding \$3.74 billion (as per Table 6), combined with temporally aligned normal transactions, ofers a rich and unique foundation for evaluating real-time defense mechanisms and rigorously training on-chain classifiers.

Our dataset construction was guided by the following criteria for a representative sample of DeFi attacks over the past five years (2020 - 2025)<sup>2</sup>:

① Economic impact: Every included incident is a verified on-chain DeFi exploit with financial loss, i.e., $\geq \$ 10,000$ . The cumulative loss from the 298 unique exploits considered totals approximately \$3.74 billion. For dataset construction criteria, consistency with the dataset’s actual total loss is key.

② Root-cause breadth: The 298 attacks span at least 12 distinct attack classes (e.g., business-logic flaws, price manipulation oracle attacks, arbitrary external calls, reentrancy vulnerabilities, access-control lapses), providing comprehensive coverage of common DeFi attack patterns.

③ Exploit focus: Incidents primarily targeting non-fungible tokens (NFTs) were excluded to maintain feature alignment with the fungible-asset liquidity mechanics that are the primary focus of our defense mechanisms.

④ Verifiability: All transactions and afected contracts related to the included exploits are publicly accessible via RPCs on their respective blockchains, enabling deterministic replay, trace analysis, and formal verification of findings.

Model Architectures. Our framework was evaluated with diverse model architectures, in cluding Multi-Layer Perceptrons (MLPs), Convolutional Neural Networks (CNNs), Recurrent Neural Networks (RNNs), and standard classifiers like Logistic Regression (LogReg), Support Vector Machines (SVM), and Decision Trees (DTs). All models utilized the 7 processed input features detailed previously. These features are observable by smart contracts during execution, enabling real-time attack detection. All models were evaluated with fixed-point parameters (scale 10<sup>1</sup> to 10<sup>18</sup>).

Proof-of-Improvement (PoIm) Protocol. Each instance starts from baseline parameters and their performance metrics (Accuracy, F1-score, Precision, Recall) on a fixed test set $\left( \mathcal { D } _ { \mathrm { t e s t } } \right)$ . An update is accepted only if it improves at least one metric without degrading others. Approved parameters are then eligible for propagation to L1 for inference.

Metrics. The performance of our framework was evaluated across three key dimensions: on-chain inference eficiency, attack detection capabilities, and the model update mechanism via Proof-of-Improvement (PoIm). The specific metrics used within each category are detailed in Table 7.

Table 7 Core evaluation metrics for assessing on-chain model performance, detection eficacy, and update mechanisms.

<table><tr><td>Metric</td><td>Description</td></tr><tr><td colspan="2">On-Chain Efficiency</td></tr><tr><td>L1 Deployment Gas</td><td>Total gas to deploy the L1 inference smart contract.</td></tr><tr><td>L1 Bytecode Size</td><td>Size in bytes of the deployed L1 inference contract.</td></tr><tr><td>L1 Set/Update Params Gas</td><td>Gas to set or update model parameters on the L1 contract.</td></tr><tr><td>L1 Inference Gas</td><td>Gas for a single classifyOnChain transaction on L1.</td></tr><tr><td>L1 Inference Throughput</td><td>Number of L1 classifyOnChain transactions processed per second.</td></tr><tr><td>On-chain/Off-chain Consistency</td><td>Proportion of matching predictions between the off-chain (fixed-point) and the on-chain L1 contract for the same inputs and parameters.</td></tr><tr><td colspan="2">Attack Detection Performance</td></tr><tr><td>Accuracy</td><td>Overall fraction of correctly classified (attack or normal) transactions.</td></tr><tr><td>Precision</td><td>Ratio of correctly identified attacks to all transactions flagged as attacks (TP / (TP + FP)).</td></tr><tr><td>Recall (Sensitivity)</td><td>Ratio of correctly identified attacks to all actual attack transactions (TP / (TP + FN)).</td></tr><tr><td>F1-Score</td><td>Harmonic mean of Precision and Recall (2 * (Prec * Rec) / (Prec + Rec)).</td></tr><tr><td>False Positive Rate (FPR)</td><td>Proportion of benign transactions incorrectly classified as attacks (FP / (FP + TN)).</td></tr><tr><td colspan="2">Model Update Mechanism (PoIm)</td></tr><tr><td>L2 PoIm Update Gas</td><td>Total gas for a successful proposeUpdate transaction on the L2 PoIm contract.</td></tr><tr><td>L2-L1 Update Cost</td><td>Gas cost to propagate accepted L2 model parameters to the L1 inference contract.</td></tr></table>

## 5.2 Performance of On-Chain DeFi Exploit Detection

We evaluate the efectiveness of on-chain machine learning classifiers in detecting previously unseen DeFi exploits. Each model was tested on a hold-out test set $\mathcal { D } _ { \mathrm { t e s t } }$ comprising realworld attacks and benign transactions, temporally separated from the training set to simulate generalization to zero-day attacks. Core detection metrics are reported in Table 12, and their corresponding financial impact is detailed in Table 13.

We tested a diverse set of models: Logistic Regression (LogReg), Support Vector Machine (SVM), Decision Tree (DT), Multi-Layer Perceptron (MLP), 15 CNN variants (filters $F \in$ {2, 4, 8, 10, 16}, kernel sizes $K \in \{ 1 , 4 , 5 \} ,$ ), and RNNs with 8 units and with 1 and 7 timesteps (T=1 and T=7). All models utilized a shared 7-feature input vector, preprocessed via standardization and label-encoding as described in Section 5.1.

The fully trained CNN variants demonstrated strong detection capabilities, particularly in terms of recall. Many configurations achieved recall $\geq 0 . 9 6$ , with F1-scores generally ranging from ∼0.78 to 0.82, and precision reaching up to 0.8077 (CNN (F4, K1)). For instance,

CNN(F4, K4) achieved a high accuracy of 0.9004 and an F1-score of 0.8200, preventing an estimated \$1,857.6M in losses. This performance marks a significant improvement over any preliminary simulations where simpler CNN setups might have exhibited degenerate behavior.

RNN models also showed robust and balanced performance. Specifically, RNN(U8, T1) achieved an accuracy of 0.8517 with a high recall of 0.9792, contributing to \$1,858.2M in prevented losses. The RNN(U8, T7) configuration maintained competitive performance, notably achieving higher precision (0.6607) and a lower false positive count (58 FPs) compared to RNN(U8, T1), with a slight trade-of in recall (0.9479).

Among other classifiers, LogReg and SVM performed well, with SVM achieving the highest AUC in this set (0.9739, see Table 12) and LogReg attaining perfect recall (1.00). The Multi-Layer Perceptron (MLP) also demonstrated strong results, with an accuracy of 0.8665, F1-score of 0.7823, and a high recall of 0.9688. The DecisionTree, while achieving perfect recall, did so at the cost of a significantly higher number of false positives (224 FPs), indicating overfitting to the attack class.

## 5.3 Eficiency, Cost, and Consistency

We analyze the on-chain operational costs, resource utilization, and behavioral consistency of the evaluated machine learning models. All quantitative data discussed refer to Table 8, which details L2 and L1 deployment gas, L1 inference gas, contract bytecode sizes, and key Proof-of-Improvement (PoIm) interaction costs where applicable.

Deployment and Inference Gas Costs. The on-chain footprint of simpler models like Logistic Regression (LogReg), Support Vector Machines (SVM), Decision Trees (DT), and a Multi-Layer Perceptron (MLP) showcases varying eficiencies. L2 PoIm contract deployment gas ranged from approximately 0.88M for the DT to 2.14M for the MLP, while their corresponding L1 inference contracts were lighter. Notably, L1 inference gas for LogReg, SVM, and DT was very low (33k–58k gas). The evaluated MLP required approximately 138k gas for L1 inference, still well within practical limits for on-chain execution. Convolutional

![](images/cbd4114792c471acf387dc78621cf9ffe1e6834c9a522f1172591cbf26e04fe6.jpg)  
Figure 8 Bitwise consistency for diferent scaling factors of various models

Neural Network (CNN) variants demonstrated a clear trend where both L2/L1 deployment gas and L1 inference gas scaled with architectural complexity, primarily driven by the number of filters (F) and kernel size (K). For instance, L2 deployment gas ranged from approximately 1.72M for CNN(F2, K1) up to 4.61M for larger configurations like CNN(F16, K1). Similarly, L1 inference gas for these CNNs varied from around 144k (CNN(F2, K1)) to over 969k (CNN(F16, K4)). Bytecode sizes for CNN L1 inference contracts were observed to be consistent for models sharing the same kernel size, as changes in filter count primarily afect the parameter size of the subsequent fully connected layer rather than the convolutional logic structure itself.

For the Recurrent Neural Network (RNN) models evaluated (both with U=8 units), the $\mathrm { T } { = } 1$ configuration (processing all 7 features in a single timestep) incurred higher L2 deployment gas (3.52M) compared to the T=7 configuration (processing 1 feature per 7 timesteps, 2.44M L2 gas). This diference is likely attributable to the larger input-tohidden weight matrix $( W _ { x h } )$ in the RNN(U8, T1) model. Conversely, L1 inference gas was substantially higher for RNN(U8, T7) (1.13M gas) than for RNN(U8, T1) (0.56M gas), reflecting the increased number of recurrent steps executed on-chain for the sequence-based input.

The gas cost for transferring updated model parameters from an L2 PoIm contract to its L1 inference counterpart varied across model types. For SVM and DT, this L2-L1 update was relatively eficient (96k–231k gas). However, the LogReg and the dynamically trained MLP exhibited higher transfer costs (5.2M–5.7M gas, respectively), likely due to the encoding or size of their complete parameter sets being transferred. For the more complex CNN and RNN models, this L2-L1 parameter transfer for their (typically output layer) updates also showed significant gas consumption, ranging from 2.8M to over 22M gas, underscoring the cost implications of updating larger or more intricate models across layers.

Table 8 On-Chain costs, sizes, and PoIm dynamics for LogReg, SVM, DT, MLP, CNN, and RNN models. Ext. Call: inference by an EVM node.

<table><tr><td>Model</td><td>Deploy L2 (Gas)</td><td>Deploy L1 (Gas)</td><td>L1 Inf. Gas (Gas)</td><td>L2 Size (Bytes)</td><td>L1 Size (Bytes)</td><td>L2-L1 (PoIm) (Gas)</td><td>Ext. Call* ($)</td></tr><tr><td>LogReg</td><td>1,185,010</td><td>722,020</td><td>57,603</td><td>4,597</td><td>3,044</td><td>231,168</td><td>0</td></tr><tr><td>SVM</td><td>1,184,998</td><td>722,020</td><td>57,603</td><td>4,597</td><td>3,044</td><td>231,168</td><td>0</td></tr><tr><td>DT</td><td>878,284</td><td>435,490</td><td>33,414</td><td>3,352</td><td>1,707</td><td>96,489</td><td>0</td></tr><tr><td>MLP</td><td>2,137,045</td><td>1,116,415</td><td>138,173</td><td>7,266</td><td>4,866</td><td>5,754,732</td><td>0</td></tr><tr><td>CNN(F2, K1)</td><td>1,721,688</td><td>1,536,046</td><td>143,647</td><td>6,692</td><td>5,834</td><td>2,872,940</td><td>0</td></tr><tr><td>CNN(F4, K1)</td><td>2,134,766</td><td>1,949,124</td><td>244,284</td><td>6,692</td><td>5,834</td><td>4,885,680</td><td>0</td></tr><tr><td>CNN(F8, K1)</td><td>2,961,086</td><td>2,775,539</td><td>445,562</td><td>6,692</td><td>5,834</td><td>8,911,240</td><td>0</td></tr><tr><td>CNN(F10, K1)</td><td>3,374,357</td><td>3,188,810</td><td>546,202</td><td>6,692</td><td>5,834</td><td>10,924,040</td><td>0</td></tr><tr><td>CNN(F16, K1)</td><td>4,614,143</td><td>4,428,596</td><td>848,126</td><td>6,692</td><td>5,834</td><td>16,962,520</td><td>0</td></tr><tr><td>CNN(F2, K4)</td><td>1,721,688</td><td>1,536,046</td><td>158,856</td><td>6,692</td><td>5,834</td><td>3,177,120</td><td>0</td></tr><tr><td>CNN(F4, K4)</td><td>2,134,778</td><td>1,949,136</td><td>274,703</td><td>6,692</td><td>5,834</td><td>5,494,060</td><td>0</td></tr><tr><td>CNN(F8, K4)</td><td>2,961,098</td><td>2,775,551</td><td>506,397</td><td>6,692</td><td>5,834</td><td>10,127,940</td><td>0</td></tr><tr><td>CNN(F10, K4)</td><td>3,374,369</td><td>3,188,822</td><td>622,244</td><td>6,692</td><td>5,834</td><td>12,444,880</td><td>0</td></tr><tr><td>CNN(F16, K4)</td><td>4,614,167</td><td>4,428,620</td><td>969,788</td><td>6,692</td><td>5,834</td><td>19,395,760</td><td>0</td></tr><tr><td>CNN(F2, K5)</td><td>1,721,688</td><td>1,536,046</td><td>150,304</td><td>6,692</td><td>5,834</td><td>3,006,080</td><td>0</td></tr><tr><td>CNN(F4, K5)</td><td>2,134,778</td><td>1,949,136</td><td>257,598</td><td>6,692</td><td>5,834</td><td>5,151,960</td><td>0</td></tr><tr><td>CNN(F8, K5)</td><td>2,961,086</td><td>2,775,539</td><td>472,188</td><td>6,692</td><td>5,834</td><td>9,443,760</td><td>0</td></tr><tr><td>CNN(F10, K5)</td><td>3,374,345</td><td>3,188,798</td><td>579,482</td><td>6,692</td><td>5,834</td><td>11,589,640</td><td>0</td></tr><tr><td>CNN(F16, K5)</td><td>4,614,143</td><td>4,428,596</td><td>901,368</td><td>6,692</td><td>5,834</td><td>18,027,360</td><td>0</td></tr><tr><td>RNN(U8, T1)</td><td>3,520,668</td><td>3,338,663</td><td>561,791</td><td>7,413</td><td>6,571</td><td>11,235,820</td><td>0</td></tr><tr><td>RNN(U8, T7)</td><td>2,437,361</td><td>2,255,356</td><td>1,131,350</td><td>7,413</td><td>6,571</td><td>22,627,000</td><td>0</td></tr></table>

\*Read-only RPC calls consume 0 gas on-chain.

Bitwise Consistency Verification. We evaluate empirically how fixed-point quantization afects on-chain inference consistency under varying scaling factors from $1 0 ^ { 1 }$ to $1 0 ^ { 1 8 }$ , using MLP, logistic regression (LogReg), SVM, CNN, and RNN models (Figure 8). Precision loss diminishes rapidly as the scale increases, and full recovery (zero bitwise loss across all weights) is achieved at or above $1 0 ^ { 1 2 }$ for all models. This confirms that fixed-point quantization, when aligned with suficiently large scaling factors, can reliably preserve inference semantics on-chain across diverse architectures, which was also evaluated in terms of the models

performance.

![](images/0ed954a0816dfc6e8b002e732c20e96c109c3abcba3afdd89065d4e4d1a48af6.jpg)  
Figure 9 PoIm performance under 50% adversarial training samples. Green vertical lines indicate accepted updates. Red lines indicate rejected ones. PoIm stabilizes performance compared to the original training.

PoIm Resilience Under Adversarial Training. We present PoIm resistance against malicious updates via stress testing. We consider bootstrapping the decentralized model with 50 real training samples from both classes, and injecting 50% of the training data size as malicious or fabricated updates. These malicious updates either flip the class label or inject feature-noise. The 50 real samples are randomly selected from the original attack and normal transaction data to quantify how PoIm reacts when training samples degrade one or more metrics, and whether such poisoning afects performance.

PoIm learning is incremental. At each new training step, the update is re-evaluated on the full test set. If accepted by PoIm, the model is updated; otherwise, the current model is retained. We note that the bootstrapping phase influences how PoIm responds to subsequent updates. Overall, PoIm maintains more stable performance than the original (unfiltered) training process across all metrics on the same test data, as shown in Figure 9. Green vertical bands indicate accepted training samples, while reddish bands denote rejected ones. Notably, original linear models (e.g., logistic regression, SVM) sufer severe degradation without PoIm filtering.

Inference Throughput. To evaluate real-time classification capability for the zero-cost eth\_call tier, we measured inference throughput using our 10-layer CNN. Table 9 details these findings, which were obtained by sequentially querying a local Ethereum node. A single transaction is classified in approximately 68ms. While the average per-sample processing time for batches of 5 to 50 transactions stabilizes around 88-91ms, this performance enables a throughput of approximately 11 classifications per second by an of-chain monitoring entity. Such capacity is comparable to Ethereum’s current average TPS (≈ 15), suitable for timely

Table 9 Inference throughput per second for (e.g., 10-layer) CNN for diferent transactions as batches. Ethereum’s current TPS ≈ 15

<table><tr><td>Batch Size</td><td>Total Time (s)</td><td>Avg Time/Sample (s)</td></tr><tr><td>1</td><td>0.0680</td><td>0.0680</td></tr><tr><td>5</td><td>0.3890</td><td>0.0778</td></tr><tr><td>10</td><td>0.9150</td><td>0.0915</td></tr><tr><td>20</td><td>1.7650</td><td>0.0883</td></tr><tr><td>50</td><td>4.5580</td><td>0.0912</td></tr></table>

analysis.

## 5.4 Baseline Comparison

We compare our framework against ML2SC [44], which compiles MLP models into smart contracts for on-chain inference. While training in ML2SC is centralized, on-chain ML studies remain sparse. Our evaluation focuses on gas costs and contract behavior. We re-deployed all ML2SC MLP contracts, but observed that their design targets batch processing over a fixed internal dataset of 50 samples. Specifically, their classify() function returns an aggregate result (e.g., count of correct classifications), whereas our framework supports single-instance inference.

Deployment and setup costs difer significantly between our approach and the baseline. Our models embed parameters directly via the constructor and become operational with 1.07M–2.17M gas. In contrast, ML2SC baselines incur higher cumulative setup costs (\~12.9M– 14.0M gas) due to separate contract deployment (1.96M–2.36M gas), parameter setting (0.30M–1.04M gas), and dataset population (\~10.63M gas for 50 samples), as detailed in Table 10. For inference, our framework supports eficient single-input predictions (81k–261k gas) with measurable throughput (\~7.6–8.0 calls/sec), whereas baseline contracts consume \~2.0M–2.4M gas per call, reflecting batch evaluation of 50 internal samples. Their view/pure classify() functions are not suited for single-instance transactional inference. Finally, our contracts are smaller in size (\~4,224 bytes) compared to baseline contracts (\~9,425–11,151 bytes). The results demonstrate the eficiency and suitability of our framework for singleinstance, real-time ML inference on-chain, especially in terms of gas cost and inference flexibility.

Table 10 Our Framework vs. Ml2SC as a baselines [44].

<table><tr><td>Metric</td><td>Our Framework</td><td>Baseline Method (ML2SC [44])</td></tr><tr><td>Deployment Gas (Contract + Params)</td><td>1.07M - 2.17M gas</td><td>1.96M - 2.36M gas (contract code only)</td></tr><tr><td>Separate Parameter Setting Gas</td><td>0 gas</td><td>0.30M - 1.04M gas</td></tr><tr><td>Data Setting Gas</td><td>N/A</td><td>~10.63M gas (for 50 internal samples)</td></tr><tr><td>Total Setup Gas</td><td>1.07M - 2.17M gas</td><td>~12.9M - ~14.0M gas</td></tr><tr><td>Bytecode Size (Bytes)</td><td>~4,224 bytes</td><td>~9,425 - ~11,151 bytes</td></tr><tr><td>Inference Gas (direct classification on chain L1)</td><td>81k - 261k gas (per single call)</td><td>~2.0M - ~2.4M gas (for 50 internal samples)</td></tr><tr><td>Output Type</td><td>Prediction (0 or 1)</td><td>Batch Result (e.g., count like 0 or 14)</td></tr></table>

Baseline classify() output (e,g., 14 (for MLP 1 laver and 1 neuron), 0 for others) indicates the number of correct predictions from its internal batch

## 6 Discussion

Transaction Class Imbalance Mitigation. In our decentralized training, the model is updated incrementally by peers through submitted training samples. If normal transactions exceed attack samples by a large margin threshold (e.g., 5x), PoIm blocks further normal samples. Only attack samples are accepted until the balance is restored. This prevents skewed updates, which would decrease attack transaction detection accuracy.

Trust and Inference Verification. Our system is designed under the assumption that decentralized participants (i.e., DeFi platforms) perform training on an L2 network. However, we do not assume these participants are honest. To defend against adversarial training data injection, we incorporate per-sample verification and adversarial robustness checks at each training step. Specifically, after each submitted training sample, the PoIm evaluates the model against a fixed, immutable (agreed upon by peers) test set using four metrics: accuracy, precision, recall, and F1 score. The update is accepted only if it maintains or improves at least one of these metrics without degrading any others. Otherwise, the training step is discarded, and the data point is excluded from the model. This method is efective in both cases: when malicious training samples are used or when honest user data fails to improve the model’s performance. It ensures that even if malicious actors attempt to inject poisoned or manipulative data, their contributions cannot degrade the classifier’s performance. Furthermore, all model training steps are conducted on L2 via a verifiable smart contract, and every accepted update is auditable through its associated state transition and event log. This guarantees transparency and accountability for each model state change. Once a model has reached a finalized state on L2 (e.g., by consensus or performance threshold), its quantized weights are serialized and committed via a cryptographic hash. This hash is propagated to L1 along with the raw weights. The L1 inference contract is a static model (non-trainable) that accepts the weights only if the hash matches, ensuring the integrity of the L2-originated model and protecting the L1 blockchain from tampering or weight substitution attacks. Thus, our architecture enforces correctness and robustness both during training (via metric-based rejection of adversarial updates) and during propagation (via hash-based integrity verification), with no reliance on external oracles, centralized validators.

Inference Tiers. Our two-tier inference architecture imposes a trade-of for protocol designers. The zero-cost tier uses eth\_call execution. It can be used for protocols that do not need on-chain inference verifiability, yet get the on-chain verified data, such as wallet interfaces, warning users of potentially malicious transactions without incurring gas fees. For fully verified on-chain defense, protocols integrate the fully on-chain tier, which acts as a gatekeeper (e.g., IPS) by embedding the classification logic interface directly within a state-modifying transaction.

Future Work. The PoIm protocol only accepts updates that yield metric improvements. This approach might lead to convergence to a local optimum, where no single micro-update can further improve the model, even if a better global solution exists. Future work could explore other acceptance criteria, such as stochastic policies like simulated annealing [33], which would permit occasional slightly degrading steps to encourage broader exploration of the model space. Also, PoIm, similar to other stake-based governance systems, could be susceptible to centralization if voting rights, represented by tokens (e.g., linear voting), are openly tradable. A malicious actor could accumulate enough stake to influence the update of the test set or roll back stable updates. Mitigating this, for example, through quadratic voting or identity-based mechanisms such as know-your-customer (KYC) [24, 25], is a potential avenue for future work. Our framework assumes a decentralized and engaged set of participants (e.g., DeFi protocols themselves) whose long-term incentive is to ensure model integrity to protect individual DeFi protocols’ funds. Finally, future work may study sophisticated adversarial strategies. This includes addressing game-theoretic risks such as front-running and detecting if latent backdoors in model updates exist.

## 7 Related Work

On-chain AI research seeks transparent, tamper-proof inference but faces the EVM’s fixedpoint arithmetic and gas limits [31, 46, 60, 44, 27]. Translators from ML to smart contract code such as ML2SC compile MLPs from PyTorch to Solidity, proving feasibility for small models yet incurring high gas per call on complex networks [60, 44].

For DeFi exploit detection, LookAhead [56], STING [72], and FlashGuard [5] inspect mempools or historical transactions to flag and mitigate attacks. Of-chain placement of these systems introduces latency [72], misses private-relay transactions, and introduces centralized control. Of-chain ML with on-chain verification is another direction of research. zkML [11], for example, attaches zk-SNARK [53] proofs to each inference, preserving privacy but multiplying compute and memory requirements by orders of magnitude [51, 69, 30].

Our cryptographic verification is distinct from zkML systems. ZKML frameworks use zero-knowledge proofs (e.g., zk-SNARKs) to verify that a specific computation, such as an ML inference, was executed honestly with a private model [11]. This provides computational integrity but has substantial overhead. For instance, ZKML needs powerful hardware (up to 1TB of RAM for a distilled GPT-2 model) and can have proving times of nearly an hour [11]. In contrast, our framework employs a much simpler and more gas-eficient commit-verify scheme. We use a cryptographic hash (keccak256) to ensure the data integrity of the model parameters as they are propagated from L2 to L1. This process guarantees that the model used for inference on L1 is bit-for-bit identical to the one approved by the PoIm governance protocol on L2, rather than proving the correctness of each inference itself. Our approach prioritizes provenance and data integrity over computational privacy since the DeFi attacks are public. This makes our approach practical for low-cost, real-time use and suitable for mitigating DeFi attacks.

opML [16] (fraud-proof) treats results as valid unless a verifier proves otherwise, reducing prover cost at the price of economic guarantees. However, it does not provide cryptographic security [51, 17]. Agatha applies similar fraud proofs to DNNs on Ethereum [75].

Proposals for decentralized model marketplaces, federated learning with ZK privacy, and DAO-based model governance [7] either ofload heavy compute or evaluate on limited node sets [16]. In contrast, our system shows end-to-end, fully audited inference while remaining within L2 and L1 mainnet gas limits. Our L1 contracts provide a variety of ML and neural network-based models, bit-exact to the of-chain specification [70, 18] and run eficiently, for example, it incurs only ≈ 57k gas for simple models. More complex non-linear models remain cost-efective, such as MLPs at ≈ 138k. L2 PoIm governs weight updates via on-chain benchmark tests instead of high-cost zk- or fraud-proof mechanisms. The design keeps the training process and inference decentralized with computation separation (L2 for computation and governance, and L1 purely for inference). Since it is built on smart contracts, it inherently detects transactions coming from any source, such as private relays that seek attack evasion, while allowing continuous community-driven improvement.

## 8 Conclusion

We presented a fully decentralized and verifiable, on-chain ML/DL framework for real-time DeFi exploit detection. Our approach enables classification of transactions at execution time using a deterministic, gas-free inference mechanism embedded in smart contracts. We proposed Proof-of-Improvement (PoIm), a decentralized, stake-based model update protocol that accepts only provably superior updates. The system guarantees inference consistency, bounded gas usage, and resistance to adversarial submissions. Empirical evaluation on 298 real-world DeFi exploits indicates high detection performance and practical feasibility. This work establishes a new model for integrating ML-driven defenses into DeFi protocols with minimal latency, overhead, and maximal decentralization.

## References

1 Hedgey finance. https://hedgey.finance/, 2024. Accessed: 2024-05-08.

2 Martín Abadi. Tensorflow: learning functions at scale. In Proceedings of the 21st ACM SIGPLAN international conference on functional programming, pages 1–1, 2016.

Sa’ed Abed, Reem Jafal, Bassam J Mohd, and Mohammad Al-Shayeji. An analysis and evaluation of lightweight hash functions for blockchain-based iot devices. Cluster computing, 24:3065–3084, 2021.

4 Alchemy. Web3 Development Platform. https://www.alchemy.com, 2024.

5 Abdulrahman Alhaidari, Balaji Palanisamy, and Prashant Krishnamurthy. Poster: Flashguard: Real-time disruption of non-price flash loan attacks in defi. In Proceedings of the 2024 ACM SIGSAC Conference on Computer and Communications Security (CCS ’24), page 3, Salt Lake City, UT, USA, October 14–18 2024. ACM. doi:10.1145/3658644.3691385.

6 Abdulrahman Alhaidari, Balaji Palanisamy, and Prashant Krishnamurthy. Protecting defi platforms against non-price flash loan attacks. In Proceedings of the Fifteenth ACM Conference on Data and Application Security and Privacy (CODASPY ’25), pages 1–12, Pittsburgh, PA, USA, 2025. ACM. doi:10.1145/3714393.3726503.

7 Dana Alsagheer, Lei Xu, and Weidong Shi. Decentralized machine learning governance: Overview, opportunities, and challenges. IEEE Access, 11:96718–96732, 2023.

8 Raphael Auer, Bernhard Haslhofer, Stefan Kitzler, Pietro Saggese, and Friedhelm Victor. The technology of decentralized finance (defi). Digital Finance, 6(1):55–95, 2024.

9 Nic Carter and Linda Jeng. Defi protocol risks: The paradox of defi. Regtech, suptech and beyond: innovation and technology in financial services” riskbooks–forthcoming Q, 3, 2021.

10 Stefanos Chaliasos, Marcos Antonios Charalambous, Liyi Zhou, Rafaila Galanopoulou, Arthur Gervais, Dimitris Mitropoulos, and Benjamin Livshits. Smart contract and defi security tools: Do they meet the needs of practitioners? In Proceedings of the 46th IEEE/ACM International Conference on Software Engineering, pages 1–13, 2024.

11 Bing-Jyue Chen, Suppakit Waiwitlikhit, Ion Stoica, and Daniel Kang. Zkml: An optimizing system for ml inference in zero-knowledge proofs. In Proceedings of the Nineteenth European Conference on Computer Systems, pages 560–574, 2024.

12 Huashan Chen, Marcus Pendleton, Laurent Njilla, and Shouhuai Xu. A survey on ethereum systems security: Vulnerabilities, attacks, and defenses. ACM Computing Surveys (CSUR), 53(3):1–43, 2020.

13 Yan Chen and Cristiano Bellavitis. Blockchain disruption and decentralized finance: The rise of decentralized business models. Journal of Business Venturing Insights, 13:e00151, 2020.

14 Arka Rai Choudhuri, Sanjam Garg, Julien Piet, and Guru-Vamsi Policharla. Mempool privacy via batched threshold encryption: Attacks and defenses. Cryptology ePrint Archive, 2024.

15 Web3.js Contributors. Web3.js, 2024. URL: https://github.com/web3/web3.js.

16 KD Conway, Cathie So, Xiaohang Yu, and Kartin Wong. opml: Optimistic machine learning on blockchain. arXiv preprint arXiv:2401.17555, 2024.

17 Sourav Das, Vinay Joseph Ribeiro, and Abhijeet Anand. Yoda: Enabling computationally intensive contracts on blockchains with byzantine and selfish nodes. arXiv preprint arXiv:1811.03265, 2018.

18 Leonardo De Moura and Nikolaj Bjørner. Z3: An eficient smt solver. In International conference on Tools and Algorithms for the Construction and Analysis of Systems, pages 337–340. Springer, 2008.

19 De.Fi. Rekt db, 2024. Accessed: 2024-06-07. URL: https://de.fi/rekt-database.

20 DeFiLlama. DeFi Dashboard. https://defillama.com, 2024. Accessed: 2024-05-14.

21 Kaustubh Dwivedi, Ankit Agrawal, Ashutosh Bhatia, and Kamlesh Tiwari. A novel classification of attacks on blockchain layers: Vulnerabilities, attacks, mitigations, and research directions. arXiv preprint arXiv:2404.18090, 2024.

22 Ethereum Foundation. Ethereum improvement proposals. https://eips.ethereum.org/, 2025.

23 Etherscan. Ethereum blockchain explorer. http://etherscan.io/, 2024.

24 Andres Fabrega, Amy Zhao, Jay Yu, James Austgen, Sarah Allen, Kushal Babel, Mahimna Kelkar, and Ari Juels. Voting-Bloc Entropy: A New Metric for DAO Decentralization. In 2025 USENIX Security Symposium (USENIX Security 25), 2025.

25 Rainer Feichtinger, Robin Fritsch, Lioba Heimbach, Yann Vonlanthen, and Roger Wattenhofer. Sok: Attacks on daos. In 6th Conference on Advances in Financial Technologies (AFT 2024), pages 28–1. Schloss Dagstuhl–Leibniz-Zentrum für Informatik, 2024.

26 Flashbots. Mitigating MEV in Blockchain. https://www.flashbots.net, 2024. Accessed: 2024-07-23.

27 Ethereum Foundation. Ethereum, 2024. URL: https://ethereum.org/.

28 Foundry. Blazing fast, portable and modular toolkit for Ethereum application development written in Rust. https://getfoundry.sh, 2024.

29 Rundong Gan, Liyi Zhou, Le Wang, Kaihua Qin, and Xiaodong Lin. Defialigner: Leveraging symbolic analysis and large language models for inconsistency detection in decentralized finance. In 6th Conference on Advances in Financial Technologies (AFT 2024), pages 7–1. Schloss Dagstuhl–Leibniz-Zentrum für Informatik, 2024.

30 Bianca-Mihaela Ganescu and Jonathan Passerat-Palmbach. Trust the process: Zeroknowledge machine learning to enhance trust in generative ai interactions. arXiv preprint arXiv:2402.06414, 2024.

31 Caleb Geren, Amanda Board, Gaby G Dagher, Tim Andersen, and Jun Zhuang. Blockchain for large language model security and safety: A holistic survey. ACM SIGKDD Explorations Newsletter, 26(2):1–20, 2025.

32 Arthur Gervais, Ghassan O Karame, Karl Wüst, Vasileios Glykantzis, Hubert Ritzdorf, and Srdjan Capkun. On the security and performance of proof of work blockchains. In Proceedings of the 2016 ACM SIGSAC conference on computer and communications security, pages 3–16, 2016.

33 Thomas Guilmeau, Emilie Chouzenoux, and Víctor Elvira. Simulated annealing: A review and a new scheme. In 2021 IEEE statistical signal processing workshop (SSP), pages 101–105. IEEE, 2021.

34 Yang Guo. A review of machine learning-based zero-day attack detection: Challenges and future directions. Computer communications, 198:175–185, 2023.

35 Wejdene Haouari, Abdelhakim Senhaji Hafid, and Marios Fokaefs. Vulnerabilities of smart contracts and mitigation schemes: A comprehensive survey. arXiv preprint arXiv:2403.19805, 2024.

36 Charles R Harris, K Jarrod Millman, Stéfan J Van Der Walt, Ralf Gommers, Pauli Virtanen, David Cournapeau, Eric Wieser, Julian Taylor, Sebastian Berg, Nathaniel J Smith, et al. Array programming with numpy. Nature, 585(7825):357–362, 2020.

37 Ningyu He, Lei Wu, Haoyu Wang, Yao Guo, and Xuxian Jiang. Characterizing code clones in the ethereum smart contract ecosystem. In International Conference on Financial Cryptography and Data Security, pages 654–675. Springer, 2020.

38 Lioba Heimbach and Roger Wattenhofer. Eliminating sandwich attacks with the help of game theory. In Proceedings of the 2022 ACM on Asia Conference on Computer and Communications Security, pages 153–167, 2022.

39 Safak Kayikci and Taghi M Khoshgoftaar. Blockchain meets machine learning: a survey. Journal of Big Data, 11(1):9, 2024.

40 Nikhil Ketkar. Introduction to keras. In Deep learning with python: a hands-on introduction, pages 97–111. Springer, 2017.

41 Oliver Kramer and Oliver Kramer. Scikit-learn. Machine learning for evolution strategies, pages 45–53, 2016.

42 Wenkai Li, Xiaoqi Li, Yuqing Zhang, and Zongwei Li. Defitail: Defi protocol inspection through cross-contract execution analysis. In Companion Proceedings of the ACM on Web Conference 2024, pages 786–789, 2024.

43 Xiaofan Li, Jin Yang, Jiaqi Chen, Yuzhe Tang, and Xing Gao. Characterizing ethereum upgradable smart contracts and their security implications. In Proceedings of the ACM Web Conference 2024, pages 1847–1858, 2024.

44 Zhikai Li, Steve Vott, and Bhaskar Krishnamachari. Ml2sc: Deploying machine learning models as smart contracts on the blockchain. In 2024 IEEE International Conference on Blockchain and Cryptocurrency (ICBC), pages 645–649. IEEE, 2024.

45 Yulin Liu, Yuxuan Lu, Kartik Nayak, Fan Zhang, Luyao Zhang, and Yinhong Zhao. Empirical analysis of eip-1559: Transaction fees, waiting times, and consensus security. In Proceedings of the 2022 ACM SIGSAC Conference on Computer and Communications Security, pages 2099–2113, 2022.

46 Rischan Mafrur. Ai-based crypto tokens: The illusion of decentralized ai? IET Blockchain, 5(1):e70015, 2025.

47 Wes McKinney et al. pandas: a foundational python library for data analysis and statistics. Python for high performance and scientific computing, 14(9):1–9, 2011.

48 Johnnatan Messias, Vabuk Pahari, Balakrishnan Chandrasekaran, Krishna P Gummadi, and Patrick Loiseau. Dissecting bitcoin and ethereum transactions: On the lack of transaction contention and prioritization transparency in blockchains. In International Conference on Financial Cryptography and Data Security, pages 221–240. Springer, 2023.

49 Nexus Mutual. CREAM Finance Hack. https://nexusmutual.io/claims-stories/ cream-finance-hack, 2025. Accessed: 2025-02-05.

50 Optimism Foundation. Optimism. https://www.optimism.io/, 2025. Accessed: 2025-02-21.

51 Zhizhi Peng, Taotao Wang, Chonghe Zhao, Guofu Liao, Zibin Lin, Yifeng Liu, Bin Cao, Long Shi, Qing Yang, and Shengli Zhang. A survey of zero-knowledge proof based verifiable machine learning. arXiv preprint arXiv:2502.18535, 2025.

52 Daniel Perez and Benjamin Livshits. Smart contract vulnerabilities: Vulnerable does not imply exploited. In 30th USENIX Security Symposium (USENIX Security 21), pages 1325–1341, 2021.

53 Maksym Petkus. Why and how zk-snark works. arXiv preprint arXiv:1906.07221, 2019.

54 Valentina Piantadosi, Giovanni Rosa, Davide Placella, Simone Scalabrino, and Rocco Oliveto. Detecting functional and security-related issues in smart contracts: A systematic literature review. Software: Practice and experience, 53(2):465–495, 2023.

55 PolygonScan. Polygon Gas Tracker. https://polygonscan.com/gastracker, 2024. Accessed: 2024-08-30.

56 Shoupeng Ren, Lipeng He, Tianyu Tu, Di Wu, Jian Liu, Kui Ren, and Chun Chen. Lookahead: Preventing defi attacks via unveiling adversarial contracts. arXiv preprint arXiv:2401.07261, 2024.

57 Chinmay Saraf and Siddharth Sabadra. Blockchain platforms: A compendium. In 2018 IEEE International Conference on Innovative Research and Development (ICIRD), pages 1–6. IEEE, 2018.

58 Sarwar Sayeed, Hector Marco-Gisbert, and Tom Caira. Smart contract: Attacks and protections. Ieee Access, 8:24416–24427, 2020.

59 Cosimo Sguanci, Roberto Spatafora, and Andrea Mario Vergani. Layer 2 blockchain scaling: A survey. arXiv preprint arXiv:2107.10881, 2021.

60 Nikumbh Sarthak Sham, Sandip Chakraborty, and Shamik Sural. Generation of optimized solidity code for machine learning models using llms. arXiv preprint arXiv:2503.06203, 2025.

61 SunWeb3Sec. DeFiHackLabs. https://github.com/SunWeb3Sec/DeFiHackLabs, 2024. Accessed: 2024-05-08.

62 Jason Teutsch and Christian Reitwießner. A scalable verification solution for blockchains. In Aspects of Computation and Automata Theory with Applications, pages 377–424. World Scientific, 2024.

63 Louis Tremblay Thibault, Tom Sarry, and Abdelhakim Senhaji Hafid. Blockchain scaling using rollups: A comprehensive survey. IEEE Access, 10:93039–93054, 2022.

64 Xiaojie Wang, Hanxue Li, Ling Yi, Zhaolong Ning, Song Guo, and Yan Zhang. A survey on of-chain networks: Frameworks, technologies, solutions and challenges. arXiv preprint arXiv:2311.10298, 2023.

65 Gavin Wood et al. Ethereum: A secure decentralised generalised transaction ledger. Ethereum project yellow paper, 151(2014):1–32, 2014.

66 Jiahua Xu and Yebo Feng. Reap the harvest on blockchain: A survey of yield farming protocols. IEEE Transactions on Network and Service Management, 20(1):858–869, 2022.

67 Zihuan Xu and Lei Chen. L2chain: Towards high-performance, confidential and secure layer-2 blockchain solution for decentralized applications. Proceedings of the VLDB Endowment, 16(4):986–999, 2022.

68 Chavhan Sujeet Yashavant, Saurabh Kumar, and Amey Karkare. Scrawld: A dataset of real world ethereum smart contracts labelled with vulnerabilities. arXiv preprint arXiv:2202.11409, 2022.

69 Zheming Ye, Xiaodong Qi, Zhao Zhang, and Cheqing Jin. Yoimiya: A scalable framework for optimal resource utilization in zk-snark systems. arXiv preprint arXiv:2502.18288, 2025.

70 Z3Prover. Z3 theorem prover. https://github.com/Z3Prover/z3, 2025. Accessed: 2025-02-18.

71 Pengcheng Zhang, Feng Xiao, and Xiapu Luo. A framework and dataset for bugs in ethereum smart contracts. In 2020 IEEE international conference on software maintenance and evolution (ICSME), pages 139–150. IEEE, 2020.

72 Zhuo Zhang, Zhiqiang Lin, Marcelo Morales, Xiangyu Zhang, and Kaiyuan Zhang. Your exploit is mine: Instantly synthesizing counterattack smart contract. In 32nd USENIX Security Symposium (USENIX Security 23), pages 1757–1774, 2023.

73 Zhuo Zhang, Brian Zhang, Wen Xu, and Zhiqiang Lin. Demystifying exploitable bugs in smart contracts. In 2023 IEEE/ACM 45th International Conference on Software Engineering (ICSE), pages 615–627. IEEE, 2023.

74 Zibin Zheng, Jianzhong Su, Jiachi Chen, David Lo, Zhijie Zhong, and Mingxi Ye. Dappscan: building large-scale datasets for smart contract weaknesses in dapp projects. IEEE Transactions on Software Engineering, 2024.

75 Zihan Zheng, Peichen Xie, Xian Zhang, Shuo Chen, Yang Chen, Xiaobing Guo, Guangzhong Sun, Guangyu Sun, and Lidong Zhou. Agatha: Smart contract for dnn computation. arXiv preprint arXiv:2105.04919, 2021.

76 Liyi Zhou, Xihan Xiong, Jens Ernstberger, Stefanos Chaliasos, Zhipeng Wang, Ye Wang, Kaihua Qin, Roger Wattenhofer, Dawn Song, and Arthur Gervais. Sok: Decentralized finance (defi) attacks. In 2023 IEEE Symposium on Security and Privacy (SP), pages 2444–2461. IEEE, 2023.

## A L1 Inference Cost on Diferent Blockchains.

To understand the practical financial implications of our on-chain inference, we estimated the USD cost of a single L1 inference transaction for a diverse set of models we evaluated across eight blockchains: Ethereum, Polygon PoS, BNB Smart Chain (BSC), Avalanche C-Chain, Arbitrum One, Optimism, Fantom Opera, Moonriver, and Base. Table 11 presents these detailed estimated costs. These calculations utilize the L1 inference gas units from Table 8 and specific gas prices and native token USD values in the table’s caption.

Table 11 L1 Inference costs (USD) recomputed using the provided gas prices and token USD values.

<table><tr><td>Model</td><td>Chain</td><td>L1 Inf. Gas</td><td>Inf. Cost (USD)</td></tr><tr><td>LogReg</td><td>Ethereum</td><td>57,603</td><td>$0.173981</td></tr><tr><td>LogReg</td><td>Polygon PoS</td><td>57,603</td><td>$0.000406</td></tr><tr><td>LogReg</td><td>BSC</td><td>57,603</td><td>$0.114054</td></tr><tr><td>LogReg</td><td>Avalanche</td><td>57,603</td><td>$0.001309</td></tr><tr><td>LogReg</td><td>Arbitrum One</td><td>57,603</td><td>$0.002030</td></tr><tr><td>LogReg</td><td>Optimism</td><td>57,603</td><td>$0.000145</td></tr><tr><td>LogReg</td><td>Moonriver</td><td>57,603</td><td>$0.001312</td></tr><tr><td>LogReg</td><td>Fantom Opera</td><td>57,603</td><td>$0.000041</td></tr><tr><td>LogReg</td><td>Base</td><td>57,603</td><td>$0.000696</td></tr><tr><td>SVM</td><td>Ethereum</td><td>57,603</td><td>$0.173981</td></tr><tr><td>SVM</td><td>Polygon PoS</td><td>57,603</td><td>$0.000406</td></tr><tr><td>SVM</td><td>BSC</td><td>57,603</td><td>$0.114054</td></tr><tr><td>SVM</td><td>Avalanche</td><td>57,603</td><td>$0.001309</td></tr><tr><td>SVM</td><td>Arbitrum One</td><td>57,603</td><td>$0.002030</td></tr><tr><td>SVM</td><td>Optimism</td><td>57,603</td><td>$0.000145</td></tr><tr><td>SVM</td><td>Moonriver</td><td>57,603</td><td>$0.001312</td></tr><tr><td>SVM</td><td>Fantom Opera</td><td>57,603</td><td>$0.000041</td></tr><tr><td>SVM</td><td>Base</td><td>57,603</td><td>$0.000696</td></tr><tr><td>DT</td><td>Ethereum</td><td>33,414</td><td>$0.100922</td></tr><tr><td>DT</td><td>Polygon PoS</td><td>33,414</td><td>$0.000235</td></tr><tr><td>DT</td><td>BSC</td><td>33,414</td><td>$0.066160</td></tr><tr><td>DT</td><td>Avalanche</td><td>33,414</td><td>$0.000759</td></tr><tr><td>DT</td><td>Arbitrum One</td><td>33,414</td><td>$0.001177</td></tr><tr><td>DT</td><td>Optimism</td><td>33,414</td><td>$0.000084</td></tr><tr><td>DT</td><td>Moonriver</td><td>33,414</td><td>$0.000761</td></tr><tr><td>DT</td><td>Fantom Opera</td><td>33,414</td><td>$0.000024</td></tr><tr><td>DT</td><td>Base</td><td>33,414</td><td>$0.000404</td></tr><tr><td>MLP</td><td>Ethereum</td><td>138,173</td><td>$0.417329</td></tr><tr><td>MLP</td><td>Polygon PoS</td><td>138,173</td><td>$0.000973</td></tr><tr><td>MLP</td><td>BSC</td><td>138,173</td><td>$0.273583</td></tr><tr><td>MLP</td><td>Avalanche</td><td>138,173</td><td>$0.003140</td></tr><tr><td>MLP</td><td>Arbitrum One</td><td>138,173</td><td>$0.004869</td></tr><tr><td>MLP</td><td>Optimism</td><td>138,173</td><td>$0.000348</td></tr><tr><td>MLP</td><td>Moonriver</td><td>138,173</td><td>$0.003146</td></tr><tr><td>MLP</td><td>Fantom Opera</td><td>138,173</td><td>$0.000099</td></tr><tr><td>MLP</td><td>Base</td><td>138,173</td><td>$0.001669</td></tr><tr><td>CNN(F2, K1)</td><td>Ethereum</td><td>143,647</td><td>$0.433863</td></tr><tr><td>CNN(F2, K1)</td><td>Polygon PoS</td><td>143,647</td><td>$0.001011</td></tr><tr><td>CNN(F2, K1)</td><td>BSC</td><td>143,647</td><td>$0.284421</td></tr><tr><td>CNN(F2, K1)</td><td>Avalanche</td><td>143,647</td><td>$0.003264</td></tr><tr><td>CNN(F2, K1)</td><td>Arbitrum One</td><td>143,647</td><td>$0.005062</td></tr><tr><td>CNN(F2, K1)</td><td>Optimism</td><td>143,647</td><td>$0.000362</td></tr><tr><td>CNN(F2, K1)</td><td>Moonriver</td><td>143,647</td><td>$0.003271</td></tr><tr><td>CNN(F2, K1)</td><td>Fantom Opera</td><td>143,647</td><td>$0.000102</td></tr><tr><td>CNN(F2, K1)</td><td>Base</td><td>143,647</td><td>$0.001735</td></tr><tr><td>CNN(F4, K1)</td><td>Ethereum</td><td>244,284</td><td>$0.737821</td></tr><tr><td>CNN(F4, K1)</td><td>Polygon PoS</td><td>244,284</td><td>$0.001720</td></tr><tr><td>CNN(F4, K1)</td><td>BSC</td><td>244,284</td><td>$0.483682</td></tr><tr><td>CNN(F4, K1)</td><td>Avalanche</td><td>244,284</td><td>$0.005551</td></tr><tr><td>CNN(F4, K1)</td><td>Arbitrum One</td><td>244,284</td><td>$0.008608</td></tr><tr><td>CNN(F4, K1)</td><td>Optimism</td><td>244,284</td><td>$0.000615</td></tr><tr><td>CNN(F4, K1)</td><td>Moonriver</td><td>244,284</td><td>$0.005562</td></tr><tr><td>CNN(F4, K1)</td><td>Fantom Opera</td><td>244,284</td><td>$0.000174</td></tr><tr><td>CNN(F4, K1)</td><td>Base</td><td>244,284</td><td>$0.002951</td></tr><tr><td>CNN(F8, K1)</td><td>Ethereum</td><td>445,562</td><td>$1.345749</td></tr><tr><td>CNN(F8, K1)</td><td>Polygon PoS</td><td>445,562</td><td>$0.003137</td></tr><tr><td>CNN(F8, K1)</td><td>BSC</td><td>445,562</td><td>$0.882213</td></tr><tr><td>CNN(F8, K1)</td><td>Avalanche</td><td>445,562</td><td>$0.010125</td></tr><tr><td>CNN(F8, K1)</td><td>Arbitrum One</td><td>445,562</td><td>$0.015700</td></tr><tr><td>CNN(F8, K1)</td><td>Optimism</td><td>445,562</td><td>$0.001121</td></tr><tr><td>CNN(F8, K1)</td><td>Moonriver</td><td>445,562</td><td>$0.010145</td></tr></table>

Continued on next page

Table 11 – continued from previous page

<table><tr><td>Model</td><td>Chain</td><td>L1 Inf. Gas</td><td>Inf. Cost (USD)</td></tr><tr><td>CNN(F8, K1)</td><td>Fantom Opera</td><td>445,562</td><td>$0.000318</td></tr><tr><td>CNN(F8, K1)</td><td>Base</td><td>445,562</td><td>$0.005383</td></tr><tr><td>CNN(F10, K1)</td><td>Ethereum</td><td>546,202</td><td>$1.649716</td></tr><tr><td>CNN(F10, K1)</td><td>Polygon PoS</td><td>546,202</td><td>$0.003845</td></tr><tr><td>CNN(F10, K1)</td><td>BSC</td><td>546,202</td><td>$1.081480</td></tr><tr><td>CNN(F10, K1)</td><td>Avalanche</td><td>546,202</td><td>$0.012412</td></tr><tr><td>CNN(F10, K1)</td><td>Arbitrum One</td><td>546,202</td><td>$0.019247</td></tr><tr><td>CNN(F10, K1)</td><td>Optimism</td><td>546,202</td><td>$0.001375</td></tr><tr><td>CNN(F10, K1)</td><td>Moonriver</td><td>546,202</td><td>$0.012437</td></tr><tr><td>CNN(F10, K1)</td><td>Fantom Opera</td><td>546,202</td><td>$0.000390</td></tr><tr><td>CNN(F10, K1)</td><td>Base</td><td>546,202</td><td>$0.006599</td></tr><tr><td>CNN(F16, K1)</td><td>Ethereum</td><td>848,126</td><td>$2.561629</td></tr><tr><td>CNN(F16, K1)</td><td>Polygon PoS</td><td>848,126</td><td>$0.005971</td></tr><tr><td>CNN(F16, K1)</td><td>BSC</td><td>848,126</td><td>$1.679289</td></tr><tr><td>CNN(F16, K1)</td><td>Avalanche</td><td>848,126</td><td>$0.019274</td></tr><tr><td>CNN(F16, K1)</td><td>Arbitrum One</td><td>848,126</td><td>$0.029886</td></tr><tr><td>CNN(F16, K1)</td><td>Optimism</td><td>848,126</td><td>$0.002135</td></tr><tr><td>CNN(F16, K1)</td><td>Moonriver</td><td>848,126</td><td>$0.019312</td></tr><tr><td>CNN(F16, K1)</td><td>Fantom Opera</td><td>848,126</td><td>$0.000605</td></tr><tr><td>CNN(F16, K1)</td><td>Base</td><td>848,126</td><td>$0.010247</td></tr><tr><td>CNN(F2, K4)</td><td>Ethereum</td><td>158,856</td><td>$0.479799</td></tr><tr><td>CNN(F2, K4)</td><td>Polygon PoS</td><td>158,856</td><td>$0.001118</td></tr><tr><td>CNN(F2, K4)</td><td>BSC</td><td>158,856</td><td>$0.314535</td></tr><tr><td>CNN(F2, K4)</td><td>Avalanche</td><td>158,856</td><td>$0.003610</td></tr><tr><td>CNN(F2, K4)</td><td>Arbitrum One</td><td>158,856</td><td>$0.005598</td></tr><tr><td>CNN(F2, K4)</td><td>Optimism</td><td>158,856</td><td>$0.000400</td></tr><tr><td>CNN(F2, K4)</td><td>Moonriver</td><td>158,856</td><td>$0.003617</td></tr><tr><td>CNN(F2, K4)</td><td>Fantom Opera</td><td>158,856</td><td>$0.000113</td></tr><tr><td>CNN(F2, K4)</td><td>Base</td><td>158,856</td><td>$0.001919</td></tr><tr><td>CNN(F4, K4)</td><td>Ethereum</td><td>274,703</td><td>$0.829696</td></tr><tr><td>CNN(F4, K4)</td><td>Polygon PoS</td><td>274,703</td><td>$0.001934</td></tr><tr><td>CNN(F4, K4)</td><td>BSC</td><td>274,703</td><td>$0.543912</td></tr><tr><td>CNN(F4, K4)</td><td>Avalanche</td><td>274,703</td><td>$0.006243</td></tr><tr><td>CNN(F4, K4)</td><td>Arbitrum One</td><td>274,703</td><td>$0.009680</td></tr><tr><td>CNN(F4, K4)</td><td>Optimism</td><td>274,703</td><td>$0.000691</td></tr><tr><td>CNN(F4, K4)</td><td>Moonriver</td><td>274,703</td><td>$0.006255</td></tr><tr><td>CNN(F4, K4)</td><td>Fantom Opera</td><td>274,703</td><td>$0.000196</td></tr><tr><td>CNN(F4, K4)</td><td>Base</td><td>274,703</td><td>$0.003319</td></tr><tr><td>CNN(F8, K4)</td><td>Ethereum</td><td>506,397</td><td>$1.529491</td></tr><tr><td>CNN(F8, K4)</td><td>Polygon PoS</td><td>506,397</td><td>$0.003565</td></tr><tr><td>CNN(F8, K4)</td><td>BSC</td><td>506,397</td><td>$1.002666</td></tr><tr><td>CNN(F8, K4)</td><td>Avalanche</td><td>506,397</td><td>$0.011508</td></tr><tr><td>CNN(F8, K4)</td><td>Arbitrum One</td><td>506,397</td><td>$0.017844</td></tr><tr><td>CNN(F8, K4)</td><td>Optimism</td><td>506,397</td><td>$0.001275</td></tr><tr><td>CNN(F8, K4)</td><td>Moonriver</td><td>506,397</td><td>$0.011531</td></tr><tr><td>CNN(F8, K4)</td><td>Fantom Opera</td><td>506,397</td><td>$0.000361</td></tr><tr><td>CNN(F8, K4)</td><td>Base</td><td>506,397</td><td>$0.006118</td></tr><tr><td>CNN(F10, K4)</td><td>Ethereum</td><td>622,244</td><td>$1.879388</td></tr><tr><td>CNN(F10, K4)</td><td>Polygon PoS</td><td>622,244</td><td>$0.004381</td></tr><tr><td>CNN(F10, K4)</td><td>BSC</td><td>622,244</td><td>$1.232043</td></tr><tr><td>CNN(F10, K4)</td><td>Avalanche</td><td>622,244</td><td>$0.014140</td></tr><tr><td>CNN(F10, K4)</td><td>Arbitrum One</td><td>622,244</td><td>$0.021926</td></tr><tr><td>CNN(F10, K4)</td><td>Optimism</td><td>622,244</td><td>$0.001566</td></tr><tr><td>CNN(F10, K4)</td><td>Moonriver</td><td>622,244</td><td>$0.014168</td></tr><tr><td>CNN(F10, K4)</td><td>Fantom Opera</td><td>622,244</td><td>$0.000444</td></tr><tr><td>CNN(F10, K4)</td><td>Base</td><td>622,244</td><td>$0.007518</td></tr><tr><td>CNN(F16, K4)</td><td>Ethereum</td><td>969,788</td><td>$2.929089</td></tr><tr><td>CNN(F16, K4)</td><td>Polygon PoS</td><td>969,788</td><td>$0.006827</td></tr><tr><td>CNN(F16, K4)</td><td>BSC</td><td>969,788</td><td>$1.920180</td></tr><tr><td>CNN(F16, K4)</td><td>Avalanche</td><td>969,788</td><td>$0.022038</td></tr><tr><td>CNN(F16, K4)</td><td>Arbitrum One</td><td>969,788</td><td>$0.034173</td></tr></table>

Table 11 – continued from previous page

<table><tr><td>Model</td><td>Chain</td><td>L1 Inf. Gas</td><td>Inf. Cost (USD)</td></tr><tr><td>CNN(F16, K4)</td><td>Optimism</td><td>969,788</td><td>$0.002441</td></tr><tr><td>CNN(F16, K4)</td><td>Moonriver</td><td>969,788</td><td>$0.022082</td></tr><tr><td>CNN(F16, K4)</td><td>Fantom Opera</td><td>969,788</td><td>$0.000692</td></tr><tr><td>CNN(F16, K4)</td><td>Base</td><td>969,788</td><td>$0.011716</td></tr><tr><td>CNN(F2, K5)</td><td>Ethereum</td><td>150,304</td><td>$0.453969</td></tr><tr><td>CNN(F2, K5)</td><td>Polygon PoS</td><td>150,304</td><td>$0.001058</td></tr><tr><td>CNN(F2, K5)</td><td>BSC</td><td>150,304</td><td>$0.297602</td></tr><tr><td>CNN(F2, K5)</td><td>Avalanche</td><td>150,304</td><td>$0.003416</td></tr><tr><td>CNN(F2, K5)</td><td>Arbitrum One</td><td>150,304</td><td>$0.005296</td></tr><tr><td>CNN(F2, K5)</td><td>Optimism</td><td>150,304</td><td>$0.000378</td></tr><tr><td>CNN(F2, K5)</td><td>Moonriver</td><td>150,304</td><td>$0.003422</td></tr><tr><td>CNN(F2, K5)</td><td>Fantom Opera</td><td>150,304</td><td>$0.000107</td></tr><tr><td>CNN(F2, K5)</td><td>Base</td><td>150,304</td><td>$0.001816</td></tr><tr><td>CNN(F4, K5)</td><td>Ethereum</td><td>257,598</td><td>$0.778034</td></tr><tr><td>CNN(F4, K5)</td><td>Polygon PoS</td><td>257,598</td><td>$0.001813</td></tr><tr><td>CNN(F4, K5)</td><td>BSC</td><td>257,598</td><td>$0.510044</td></tr><tr><td>CNN(F4, K5)</td><td>Avalanche</td><td>257,598</td><td>$0.005854</td></tr><tr><td>CNN(F4, K5)</td><td>Arbitrum One</td><td>257,598</td><td>$0.009077</td></tr><tr><td>CNN(F4, K5)</td><td>Optimism</td><td>257,598</td><td>$0.000648</td></tr><tr><td>CNN(F4, K5)</td><td>Moonriver</td><td>257,598</td><td>$0.005866</td></tr><tr><td>CNN(F4, K5)</td><td>Fantom Opera</td><td>257,598</td><td>$0.000184</td></tr><tr><td>CNN(F4, K5)</td><td>Base</td><td>257,598</td><td>$0.003112</td></tr><tr><td>CNN(F8, K5)</td><td>Ethereum</td><td>472,188</td><td>$1.426168</td></tr><tr><td>CNN(F8, K5)</td><td>Polygon PoS</td><td>472,188</td><td>$0.003324</td></tr><tr><td>CNN(F8, K5)</td><td>BSC</td><td>472,188</td><td>$0.934932</td></tr><tr><td>CNN(F8, K5)</td><td>Avalanche</td><td>472,188</td><td>$0.010730</td></tr><tr><td>CNN(F8, K5)</td><td>Arbitrum One</td><td>472,188</td><td>$0.016639</td></tr><tr><td>CNN(F8, K5)</td><td>Optimism</td><td>472,188</td><td>$0.001188</td></tr><tr><td>CNN(F8, K5)</td><td>Moonriver</td><td>472,188</td><td>$0.010752</td></tr><tr><td>CNN(F8, K5)</td><td>Fantom Opera</td><td>472,188</td><td>$0.000337</td></tr><tr><td>CNN(F8, K5)</td><td>Base</td><td>472,188</td><td>$0.005705</td></tr><tr><td>CNN(F10, K5)</td><td>Ethereum</td><td>579,482</td><td>$1.750233</td></tr><tr><td>CNN(F10, K5)</td><td>Polygon PoS</td><td>579,482</td><td>$0.004080</td></tr><tr><td>CNN(F10, K5)</td><td>BSC</td><td>579,482</td><td>$1.147374</td></tr><tr><td>CNN(F10, K5)</td><td>Avalanche</td><td>579,482</td><td>$0.013169</td></tr><tr><td>CNN(F10, K5)</td><td>Arbitrum One</td><td>579,482</td><td>$0.020419</td></tr><tr><td>CNN(F10, K5)</td><td>Optimism</td><td>579,482</td><td>$0.001459</td></tr><tr><td>CNN(F10, K5)</td><td>Moonriver</td><td>579,482</td><td>$0.013195</td></tr><tr><td>CNN(F10, K5)</td><td>Fantom Opera</td><td>579,482</td><td>$0.000413</td></tr><tr><td>CNN(F10, K5)</td><td>Base</td><td>579,482</td><td>$0.007001</td></tr><tr><td>CNN(F16, K5)</td><td>Ethereum</td><td>901,368</td><td>$2.722438</td></tr><tr><td>CNN(F16, K5)</td><td>Polygon PoS</td><td>901,368</td><td>$0.006346</td></tr><tr><td>CNN(F16, K5)</td><td>BSC</td><td>901,368</td><td>$1.784709</td></tr><tr><td>CNN(F16, K5)</td><td>Avalanche</td><td>901,368</td><td>$0.020484</td></tr><tr><td>CNN(F16, K5)</td><td>Arbitrum One</td><td>901,368</td><td>$0.031762</td></tr><tr><td>CNN(F16, K5)</td><td>Optimism</td><td>901,368</td><td>$0.002269</td></tr><tr><td>CNN(F16, K5)</td><td>Moonriver</td><td>901,368</td><td>$0.020524</td></tr><tr><td>CNN(F16, K5)</td><td>Fantom Opera</td><td>901,368</td><td>$0.000643</td></tr><tr><td>CNN(F16, K5)</td><td>Base</td><td>901,368</td><td>$0.010890</td></tr><tr><td>RNN(U8, T1)</td><td>Ethereum</td><td>561,791</td><td>$1.696800</td></tr><tr><td>RNN(U8, T1)</td><td>Polygon PoS</td><td>561,791</td><td>$0.003955</td></tr><tr><td>RNN(U8, T1)</td><td>BSC</td><td>561,791</td><td>$1.112346</td></tr><tr><td>RNN(U8, T1)</td><td>Avalanche</td><td>561,791</td><td>$0.012767</td></tr><tr><td>RNN(U8, T1)</td><td>Arbitrum One</td><td>561,791</td><td>$0.019796</td></tr><tr><td>RNN(U8, T1)</td><td>Optimism</td><td>561,791</td><td>$0.001414</td></tr><tr><td>RNN(U8, T1)</td><td>Moonriver</td><td>561,791</td><td>$0.012792</td></tr><tr><td>RNN(U8, T1)</td><td>Fantom Opera</td><td>561,791</td><td>$0.000401</td></tr><tr><td>RNN(U8, T1)</td><td>Base</td><td>561,791</td><td>$0.006787</td></tr><tr><td>RNN(U8, T7)</td><td>Ethereum</td><td>1,131,350</td><td>$3.417062</td></tr><tr><td>RNN(U8, T7)</td><td>Polygon PoS</td><td>1,131,350</td><td>$0.007965</td></tr><tr><td>RNN(U8, T7)</td><td>BSC</td><td>1,131,350</td><td>$2.240073</td></tr></table>

Table 11 – continued from previous page

<table><tr><td>Model</td><td>Chain</td><td>L1 Inf. Gas</td><td>Inf. Cost (USD)</td></tr><tr><td>RNN(U8, T7)</td><td>Avalanche</td><td>1,131,350</td><td>$0.025710</td></tr><tr><td>RNN(U8, T7)</td><td>Arbitrum One</td><td>1,131,350</td><td>$0.039866</td></tr><tr><td>RNN(U8, T7)</td><td>Optimism</td><td>1,131,350</td><td>$0.002848</td></tr><tr><td>RNN(U8, T7)</td><td>Moonriver</td><td>1,131,350</td><td>$0.025761</td></tr><tr><td>RNN(U8, T7)</td><td>Fantom Opera</td><td>1,131,350</td><td>$0.000807</td></tr><tr><td>RNN(U8, T7)</td><td>Base</td><td>1,131,350</td><td>$0.013668</td></tr></table>

As of May 2025. ETH L1: 1.20 Gwei, \$2516 95/ETH Polugon PoS: 32 0 Gwei, \$0 22/MATIC. BSC: 3.0 Gwei, \$660/BNB. Avalanche: 1.01 Gwei \$22.50/AVAX. Arbitrum One: 0.0140 Gwei (ETH for gas). Optimism: 0.0010 Gwei (ETH for gas). Moonriver: 3.3 Gwei, \$6.90/MOVR. Fantom Opera: 1.23 Gwei, \$0.58/FTM. Base: 0.0048 Gwei (ETH for gas). ETH price of \$2516.95 used for Arbitrum, Optimism, and Base gas cost calculations.

## B Detection and Financial Impact

First, Table 12 presents the core detection metrics, including accuracy, F1-score, precision, recall, AUC, and False Positive Rate (FPR), for each model when evaluated against previously unseen DeFi exploits from our curated dataset. These metrics quantify the efectiveness of the classifiers in distinguishing between malicious and benign transactions. The recall is computed based on the 96 unique attacks (120 attack transactions) in the test set, while the FPR is computed over the 376 normal transactions in the same set.

Table 12 Model performance metrics for detection of unseen attacks. Recall is computed as TP% of 96 unique attacks (120 attack transactions). FP% is computed over the 376 normal transactions. The test set has 472 transactions in total.

<table><tr><td>Model</td><td>Acc.</td><td>F1</td><td>Prec.</td><td>Recall (TP%)</td><td>AUC</td><td>FP%</td></tr><tr><td>LogReg</td><td>0.8538</td><td>0.7738</td><td>0.6310</td><td>1.0000</td><td>0.9516</td><td>0.1862</td></tr><tr><td>DecisionTree</td><td>0.5381</td><td>0.5169</td><td>0.3485</td><td>1.0000</td><td>0.6836</td><td>0.5957</td></tr><tr><td>CNN(F10, K4)</td><td>0.8750</td><td>0.7950</td><td>0.6705</td><td>0.9792</td><td>0.9552</td><td>0.1543</td></tr><tr><td>CNN(F10, K5)</td><td>0.8729</td><td>0.7907</td><td>0.6686</td><td>0.9792</td><td>0.9611</td><td>0.1569</td></tr><tr><td>CNN(F16, K1)</td><td>0.8665</td><td>0.7840</td><td>0.6554</td><td>0.9792</td><td>0.9665</td><td>0.1649</td></tr><tr><td>CNN(F8, K4)</td><td>0.8665</td><td>0.7840</td><td>0.6554</td><td>0.9792</td><td>0.9505</td><td>0.1649</td></tr><tr><td>CNN(F16, K5)</td><td>0.8644</td><td>0.7811</td><td>0.6517</td><td>0.9792</td><td>0.9576</td><td>0.1676</td></tr><tr><td>RNN(U8, T1)</td><td>0.8517</td><td>0.7656</td><td>0.6304</td><td>0.9792</td><td>0.9391</td><td>0.1835</td></tr><tr><td>CNN(F8, K5)</td><td>0.8729</td><td>0.7893</td><td>0.6706</td><td>0.9688</td><td>0.9495</td><td>0.1516</td></tr><tr><td>CNN(F16, K4)</td><td>0.8686</td><td>0.7836</td><td>0.6628</td><td>0.9688</td><td>0.9679</td><td>0.1569</td></tr><tr><td>MLP</td><td>0.8665</td><td>0.7823</td><td>0.6571</td><td>0.9688</td><td>0.9666</td><td>0.1622</td></tr><tr><td>CNN(F10, K1)</td><td>0.8665</td><td>0.7823</td><td>0.6571</td><td>0.9688</td><td>0.9658</td><td>0.1622</td></tr><tr><td>SVM</td><td>0.8814</td><td>0.8000</td><td>0.6867</td><td>0.9583</td><td>0.9739</td><td>0.1383</td></tr><tr><td>CNN(F8, K1)</td><td>0.8792</td><td>0.7960</td><td>0.6848</td><td>0.9583</td><td>0.9554</td><td>0.1410</td></tr><tr><td>RNN(U8, T7)</td><td>0.8644</td><td>0.7736</td><td>0.6607</td><td>0.9479</td><td>0.9728</td><td>0.1543</td></tr><tr><td>CNN(F4, K4)</td><td>0.9004</td><td>0.8200</td><td>0.7415</td><td>0.9271</td><td>0.9577</td><td>0.1037</td></tr><tr><td>CNN(F4, K1)</td><td>0.8877</td><td>0.7962</td><td>0.7338</td><td>0.8750</td><td>0.9450</td><td>0.1011</td></tr><tr><td>CNN(F2, K1)</td><td>0.8072</td><td>0.6789</td><td>0.5780</td><td>0.8125</td><td>0.8431</td><td>0.1915</td></tr><tr><td>CNN(F2, K4)</td><td>0.7627</td><td>0.3055</td><td>0.5750</td><td>0.2083</td><td>0.8097</td><td>0.0479</td></tr><tr><td>CNN(F4, K5)</td><td>0.7775</td><td>0.2205</td><td>0.9333</td><td>0.1250</td><td>0.8626</td><td>0.0027</td></tr></table>

We also present the financial savings resulting from implementing each on-chain model (see Table 13).

## C Root Causes of Attacks

We classify the root causes of these attacks into five empirically supported categories [19, 61, 20]:

Access control failures: Contracts lack proper access modifiers (e.g., onlyOwner) or expose privileged functions to unauthorized users (e.g., Dexible, GFOX)

Table 13 Financial Impact of Detection (Ordered by Loss Prevented). F: #filters and K: #kernels

<table><tr><td>Model</td><td>Loss Prevented ($)</td><td>Loss Missed ($)</td></tr><tr><td>DecisionTree</td><td>1,858.4M</td><td>0.0M</td></tr><tr><td>LogReg</td><td>1,858.4M</td><td>0.0M</td></tr><tr><td>CNN(F10, K4)</td><td>1,858.2M</td><td>0.2M</td></tr><tr><td>CNN(F10, K5)</td><td>1,858.2M</td><td>0.2M</td></tr><tr><td>CNN(F16, K1)</td><td>1,858.2M</td><td>0.2M</td></tr><tr><td>CNN(F16, K5)</td><td>1,858.2M</td><td>0.2M</td></tr><tr><td>CNN(F8, K4)</td><td>1,858.2M</td><td>0.2M</td></tr><tr><td>CNN(F10, K1)</td><td>1,858.2M</td><td>0.2M</td></tr><tr><td>RNN(U8, T1)</td><td>1,858.2M</td><td>0.2M</td></tr><tr><td>CNN(F16, K4)</td><td>1,858.2M</td><td>0.2M</td></tr><tr><td>MLP</td><td>1,858.2M</td><td>0.2M</td></tr><tr><td>CNN(F8, K1)</td><td>1,858.1M</td><td>0.3M</td></tr><tr><td>SVM</td><td>1,858.1M</td><td>0.3M</td></tr><tr><td>CNN(F8, K5)</td><td>1,857.8M</td><td>0.6M</td></tr><tr><td>RNN(U8, T7)</td><td>1,857.7M</td><td>0.7M</td></tr><tr><td>CNN(F4, K4)</td><td>1,857.6M</td><td>0.8M</td></tr><tr><td>CNN(F2, K1)</td><td>1,805.0M</td><td>53.4M</td></tr><tr><td>CNN(F4, K1)</td><td>1,774.5M</td><td>83.9M</td></tr><tr><td>CNN(F2, K4)</td><td>145.3M</td><td>1,713.1M</td></tr><tr><td>CNN(F4, K5)</td><td>109.5M</td><td>1,748.9M</td></tr></table>

Business logic flaws: Valid transactions result in unintended outcomes, such as undercollateralized loans or unchecked withdrawals. (e.g., Euler, Platypus, PineProtocol).

Slippage and oracle manipulation: Exploits that manipulate oracles, liquidity curves, or swap sequences to skew pricing (e.g., KyberSwap)

Unchecked external calls and delegatecall misuse: Insecure call forwarding or forged approvals lead to attacker-controlled execution. (e.g., Seneca, RabbyRouter, RevertFinance).

Storage layout collisions: Overlapping storage slots in upgradeable contracts results in overwrite of admin roles or token balances (e.g., Telcoin, EFVault).

Transaction labeling is derived directly from exploit disclosures and does not rely on heuristic classification.

## D Consistency Verification of On-Chain Models

This appendix details the pseudocode for an algorithm designed to ensure the consistency of model parameters and inference behavior when deployed and managed on a blockchain. It outlines procedures for initializing model registries on Layer 2 (L2), updating models via a Proof-of-Improvement (PoIm) mechanism where the L2 contract evaluates proposals on-chain, transferring model parameters to Layer 1 (L1) for inference, and verifying the consistency of on-chain predictions. The algorithms (1-4) reflect the logic for maintaining data integrity and computational accuracy in a decentralized ML, consistent with the mechanisms described in the main paper, and is applicable to various parametric model architectures.

Table 14 Summary of notations

<table><tr><td>Symbol</td><td>Description</td></tr><tr><td> $M_{\text{off-chain}}$ </td><td>Off-chain Machine Learning model.</td></tr><tr><td> $\Theta = \{W, b, L\}$ </td><td>Model parameters, where  $W$  are weights,  $b$  are biases, and  $L$  are layer sizes (or other structural parameters).</td></tr><tr><td> $P = \{Acc, F1, Prec, Rec\}$ </td><td>Model performance metrics (Accuracy, F1-score, Precision, Recall).</td></tr><tr><td> $S_w$ </td><td>Scaling factor for converting  $\Theta$  components (weights, biases) to fixed-point integers. Also referred to as  $S$  in the main paper.</td></tr><tr><td> $S_m$ </td><td>Scaling factor for converting  $P$  components to fixed-point integers.</td></tr><tr><td> $\mathcal{D}_{test}$ </td><td>Canonical on-chain evaluation dataset used by  $C_{PoIm}$ </td></tr><tr><td> $C_{PoIm}$ </td><td>Deployed L2 smart contract for  $PoIm$ . Stores  $\Theta_{best\_fixed}$  and  $P_{best\_fixed}$ .</td></tr><tr><td> $C_{Infer}$ </td><td>Deployed L1 Smart Contract for Inference. Stores  $\Theta_{infer\_fixed}$ .</td></tr><tr><td> $\Theta_{fixed}$ </td><td>Generic fixed-point representation of model parameters.</td></tr><tr><td> $P_{fixed}$ </td><td>Generic fixed-point representation of performance metrics (often used for off-chain calculated metrics).</td></tr><tr><td> $P_{fixed}^{(new, on-chain)}$ </td><td>Fixed-point performance metrics of a new model, as calculated on-chain by  $C_{PoIm}$  using  $\mathcal{D}_{test}$ .</td></tr><tr><td> $\Theta_{best\_fixed}$ </td><td>Current best fixed-point parameters stored in  $C_{PoIm}$ .</td></tr><tr><td> $P_{best\_fixed}$ </td><td>Current best fixed-point metrics stored in  $C_{PoIm}$  (derived from on-chain).</td></tr><tr><td> $\Theta_{infer\_fixed}$ </td><td>Fixed-point parameters stored in  $C_{Infer}$  for inference.</td></tr><tr><td> $X_{sample}$ </td><td>A sample input feature vector.</td></tr><tr><td> $X_{sample\_fixed}$ </td><td>Fixed-point representation of  $X_{sample}$ .</td></tr><tr><td> $Y_{off-chain}$ </td><td>Prediction output from the off-chain model.</td></tr><tr><td> $Y_{on-chain}$ </td><td>Prediction output from the on-chain model.</td></tr><tr><td>to_fixed(val, scale)</td><td>Conceptual function to convert val to fixed-point using scale.</td></tr><tr><td>from_fixed(val, scale)</td><td>Conceptual function to convert fixed-point val back using scale.</td></tr><tr><td>integer_cast(val)</td><td>Conceptual function to cast val to an integer type.</td></tr></table>

<div class="mineru-algorithm" style="white-space: pre-wrap; font-family:monospace;">
Algorithm 1 Part 1: InitializeL2PoImContract($M_{\text{off-chain}}^{(0)}$, $\mathcal{D}_{\text{test\_off-chain}}$, $S_w$, $S_m$, $C_{\text{PoIm\_address}}$)
Extract initial parameters $\Theta_{\text{off-chain}}^{(0)}$ from $M_{\text{off-chain}}^{(0)}$.
Convert parameters to fixed-point: $\Theta_{\text{fixed}}^{(0)} \leftarrow \{\text{to\_fixed}(W^{(0)}, S_w)$, to\_fixed($b^{(0)}, S_w$), $L^{(0)}\}$ $P^{(0)}\_expected\_off-chain \leftarrow M_{\text{off-chain}}^{(0)}.$ evaluate($\mathcal{D}_{\text{test\_off-chain}}$).
Deploy $C_{\text{PoIm}}$ to L2. The constructor takes ($\Theta_{\text{fixed}}^{(0)}$, $S_w$, $S_m$) and internally evaluates $\Theta_{\text{fixed}}^{(0)}$ on the on-chain $\mathcal{D}_{\text{test}}$ to establish the initial $P_{\text{best\_fixed}}$.
Let $P_{\text{on-chain\_eval}}^{(0)} \leftarrow$ metrics resulting from $C_{\text{PoIm}}$'s internal initial evaluation.
Verification 1 (Initial State Consistency on L2):
Retrieve current best parameters from $C_{\text{PoIm}}$: $\Theta_{\text{best\_fixed}} \leftarrow C_{\text{PoIm}}.$ getCurrentModelParameters().
Retrieve current best metrics from $C_{\text{PoIm}}$: $P_{\text{best\_fixed}} \leftarrow C_{\text{PoIm}}.$.getCurrentModelMetrics().
Assert that $\Theta_{\text{best\_fixed}} = \Theta_{\text{fixed}}^{(0)}$.
Assert that $P_{\text{best\_fixed}} = P_{\text{on-chain\_eval}}^{(0)}$.
</div>

<div class="mineru-algorithm" style="white-space: pre-wrap; font-family:monospace;">
Algorithm 2 Part 2: ProposeAndUpdateL2PoIm($M_{\text{off-chain}}^{(\text{new})}, C_{\text{PoIm}}, S_w$)
Extract new parameters $\Theta_{\text{off-chain}}^{(\text{new})}$ from $M_{\text{off-chain}}^{(\text{new})}$.
Convert new parameters to fixed-point: $\Theta_{\text{fixed}}^{(\text{new})} \leftarrow \{\text{to\_fixed}(W^{(\text{new})}, S_w), \text{to\_fixed}(b^{(\text{new})}, S_w), L^{(\text{new})}\}$.
Store the current on-chain best parameters and metrics (before transaction):
$\Theta_{\text{best\_fixed\_old}} \leftarrow C_{\text{PoIm.ilineCurrentModelParameters}}(\cdot)$.
$P_{\text{best\_fixed\_old}} \leftarrow C_{\text{PoIm.ilineCurrentModelMetrics}}(\cdot)$.
tx_receipt $\leftarrow C_{\text{PoIm.}proposeModelUpdate}(\Theta_{\text{fixed}}^{(\text{new})}, \text{stake } s)$.
accepted $\leftarrow$ determine_from_event_or_return(tx_receipt).
Let $P_{\text{fixed}}^{(\text{new, on-chain})}$ from_event_if_accepted be the new metrics if update was accepted.
if accepted then
Verification 2.1 (Accepted Update - State Change):
Retrieve current parameters from $C_{\text{PoIm}}$: $\Theta_{\text{best\_fixed\_current}}$.
Retrieve current metrics from $C_{\text{PoIm}}$: $P_{\text{best\_fixed\_current}}$.
Assert that $\Theta_{\text{best\_fixed\_current}} = \Theta_{\text{fixed}}^{(\text{new})}$.
Assert that $P_{\text{best\_fixed\_current}} = P_{\text{fixed}}^{(\text{new, on-chain})}$ from_event_if_accepted.
else
Verification 2.2 (Rejected Update - State Unchanged):
Retrieve current parameters from $C_{\text{PoIm}}$: $\Theta_{\text{best\_fixed\_current}}$.
Retrieve current metrics from $C_{\text{PoIm}}$: $P_{\text{best\_fixed\_current}}$.
Assert that $\Theta_{\text{best\_fixed\_current}} = \Theta_{\text{best\_fixed\_old}}$.
Assert that $P_{\text{best\_fixed\_current}} = P_{\text{best\_fixed\_old}}$.
return accepted.
</div>

<div class="mineru-algorithm" style="white-space: pre-wrap; font-family:monospace;">
Algorithm 3 Part 3: TransferAndVerifyL1InferenceModel($C_{\text{PoIm}}$, $C_{\text{Infer}}$)
Retrieve best parameters from $C_{\text{PoIm}}$:
    $L_{\text{L2}} \leftarrow C_{\text{PoIm}}.\text{getCurrentModelParameters()}.layerSizes.$ $W_{\text{L2}} \leftarrow C_{\text{PoIm}}.\text{getCurrentModelParameters()}.weights.$ $b_{\text{L2}} \leftarrow C_{\text{PoIm}}.\text{getCurrentModelParameters()}.biases.$ $S_{w,\text{L2}} \leftarrow C_{\text{PoIm}}.\text{getScalingFactorWeights()}.$
Let $\Theta_{\text{from\_L2}} \leftarrow \{W_{\text{L2}}, b_{\text{L2}}, L_{\text{L2}}\}.$
Call $C_{\text{Infer}}.\text{setModelParameters}(\Theta_{\text{from\_L2}}, S_{w,\text{L2}})$ on the L1 network to update $C_{\text{Infer}}$.
Verification 3 (L1 Parameter Consistency after Transfer):
    Retrieve parameters from $C_{\text{Infer}}$:
        $L_{\text{L1}} \leftarrow C_{\text{Infer}}.\text{getLayerSizes()}, W_{\text{L1}} \leftarrow C_{\text{Infer}}.\text{getWeights()}, b_{\text{L1}} \leftarrow C_{\text{Infer}}.\text{getBiases()}.$ $S_{w,\text{L1}} \leftarrow C_{\text{Infer}}.\text{getScalingFactorWeights()}.$
    Assert that $\{W_{\text{L1}}, b_{\text{L1}}, L_{\text{L1}}\}$ equals $\Theta_{\text{from\_L2}}$.
    Assert that $S_{w,\text{L1}}$ equals $S_{w,\text{L2}}$.
</div>

# On-Chain Decentralized Learning and Cost-Efective Inference for DeFi Attack Mitigation

<div class="mineru-algorithm" style="white-space: pre-wrap; font-family:monospace;">
Algorithm 4 Part 4: VerifyL1OnChainInference($X_{\text{sample}}$, $M_{\text{off-chain}}^{\text{L1}}$, $C_{\text{Infer}}$)
Retrieve $W_{\text{L1}}$, $b_{\text{L1}}$, $L_{\text{L1}}$, $S_{w,\text{L1}}$ from $C_{\text{Infer}}$ (as per getters in Part 3).
Configure $M_{\text{off-chain}}^{\text{L1}}$ with parameters from_fixed($W_{\text{L1}}$, $S_{w,\text{L1}}$), from_fixed($b_{\text{L1}}$, $S_{w,\text{L1}}$), and $L_{\text{L1}}$.
$Y_{\text{off-chain}} \leftarrow M_{\text{off-chain}}^{\text{L1}}.$ predict($X_{\text{sample}}$).
$X_{\text{sample\_fixed}} \leftarrow \text{to\_fixed}(X_{\text{sample}}, S_{w,\text{L1}})$.
$Y_{\text{on-chain}} \leftarrow C_{\text{Infer}}.$ classify($X_{\text{sample\_fixed}}$).
Verification 4 (Inference Output Consistency):
Assert that $Y_{\text{on-chain}}$ equals integer_cast($Y_{\text{off-chain}}$).
</div>