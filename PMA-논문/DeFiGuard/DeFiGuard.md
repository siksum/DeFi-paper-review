# DeFiGuard: A Price Manipulation Detection Service in DeFi using Graph Neural Networks

Dabao Wang<sup>1</sup>, Bang Wu<sup>1</sup>, Xingliang Yuan<sup>2</sup>, Lei Wu<sup>3</sup>, Yajin Zhou<sup>3</sup>, and Helei Cui<sup>4</sup>

<sup>1</sup>Monash University

<sup>2</sup>The University of Melbourne

<sup>3</sup>Zhejiang University

<sup>4</sup>Northwestern Polytechnical University

Abstract—The prosperity of Decentralized Finance (DeFi) unveils underlying risks, with reported losses surpassing 3.2 billion USD between 2018 and 2022 due to vulnerabilities in Decentralized Applications (DApps). One significant threat is the Price Manipulation Attack (PMA) that alters asset prices during transaction execution. As a result, PMA accounts for over 50 million USD in losses. To address the urgent need for efficient PMA detection, this paper introduces a novel detection service, DeFiGuard, using Graph Neural Networks (GNNs). In this paper, we propose cash flow graphs with four distinct features, which capture the trading behaviors from transactions. Moreover, DeFi-Guard integrates transaction parsing, graph construction, model training, and PMA detection. Evaluations on a dataset of 208 PMA and 2,080 non-PMA transactions show that DeFiGuard with GNN models outperforms the baseline in Accuracy, TPR, FPR, and AUC-ROC. The results of ablation studies suggest suggest that the combination of the four proposed node features enhances DeFiGuard’s efficacy. Moreover, DeFiGuard classifies transactions within 0.892 to 5.317 seconds, which provides sufficient time for the victims (DApps and users) to take action to rescue their vulnerable funds. In conclusion, this research offers a significant step towards safeguarding the DeFi landscape from PMAs using GNNs.

Index Terms—Decentralized Finance, Price Manipulation, GNN.

## I. INTRODUCTION

The surge of Decentralized Finance (DeFi) since 2019 has attracted a significant influx of capital into the blockchain ecosystem. On the one hand, various entities are constructing decentralized applications (DApps) to offer financial services within the DeFi ecosystem. On the other hand, users are increasingly drawn to engage in interacting with these DApps to gain profits. However, the prosperity of DeFi not only introduces the chance of making financial gains but also brings the underlying risks of losing assets [1]–[5]. The research [6] reveals the alarming fact that the code and logic vulnerabilities in DApps resulted in a staggering loss exceeding 3.2 billion USD between April 2018 and April 2022.

The price manipulation attack (PMA), one of the infamous attacks within the DeFi realm, has attracted considerable attention and resulted in financial losses exceeding 200 million USD [7], [8]. Specifically, the PMA refers to the malicious exploitation of vulnerabilities in smart contracts to manipulate asset prices by tampering with the price oracles or altering the trading ratio in exchanges’ liquidity pools.

Initiating a PMA typically necessitates multiple interactions across a range of DApps. Compounding the challenge, astute attackers frequently deploy opaque smart contracts to obscure the execution logic, leading to unexpected state alterations, such as inflating or deflating the asset price.

Existing research [5] extracts semantic knowledge from a collection of DApps to establish two expert-defined patterns, which identify two kinds of PMAs: direct and indirect PMAs. However, this reliance on predefined patterns may impede the comprehensive detection of PMAs. As these patterns are tied to the semantic understanding of a variety of DApps, their adaptability and scalability are limited due to the rapid increase in the number of DApps. Therefore, there is an urgent need to design a highly adaptive methodology capable of detecting an extensive array of PMAs.

Compared to rule-based methodologies, the learning-based approach can offer adaptability in the field of vulnerability detection [9]. In this study, we construct the cash flow graph for the transaction and conduct a graph representation learning using Graph Neural Networks (GNNs) for PMA detection based on two insights. First, after examining various PMAs, we discover that the cash flow graph provides a comprehensive representation of trading information for a transaction. Specifically, the account can be represented as a node and the asset transfer can be represented as an edge in the graph. Second, recent advancements in the field have seen the success of GNNs for handling graph-structured data in the field of cybersecurity [10]–[12]. Given these insights, we believe in the potential of training a GNN model to capture the nuances of the cash flow graphs for PMA detection.

In this study, we introduce DeFiGuard, a detection service for PMA detection using GNNs. To ensure swift updates in response to evolving detection methodologies and GNN algorithms, we structured our service into three components: Transaction Parser, Graph Builder, and Graph Classifier. 1) The Transaction Parser collects the raw transactions from EVM-based blockchains and retrieves transfer-related call traces and event logs. 2) The Graph Builder constructs cash flow graphs with four distinct node features based on the call traces and event logs received from the Transaction Parser. 3) The Graph Classifier builds a GNN model tailored for graph representation learning and employs a pre-trained GNN model for PMA detection. The decomposing nature of DeFiGuard provides a flexible foundation for future improvements, such as updating the Graph Classifier with cutting-edge GNN algorithms.

In our evaluation, we assessed the performance of DeFi-Guard using a dataset of 208 PMA and 2,080 non-PMA transactions. The overall performance reveals that when paired with GNN models, DeFiGuard outperformed the baseline MLP model across metrics such as Accuracy, TPR, FPR, and AUC-ROC value. For instance, when paired with the GraphSage model, DeFiGuard achieves a peak performance of 93.25% Accuracy, 91.67% TPR, 6.67% FPR, and 96.18% AUC-ROC. Moreover, the hyperparameter tuning process shows that training with only 100 PMA and non-PMA transactions across 100 epochs yields strong performance. This further underscores the effectiveness of the cash flow graph in capturing transaction trading behavior. In the ablation studies, we evaluate the impact of the four proposed node features: node type, transfer frequency, transfer diversity, and profit score on classification performance. The results suggest that the classification performance for PMA detection stems from the collective impact of these four node features, with no single feature predominating. Regarding practicality, we evaluated the time consumed by DeFiGuard for PMA detection. The overall for classifying a single transaction ranged from 0.892 to 5.317 seconds, which is faster than the time required to generate a new block on Ethereum. This implies that, with a well-connected node, the potential victims, including DApps and users, can have sufficient time to secure their vulnerable assets after receiving the classification result on provided transactions.

To sum up, this work makes the following contributions:

• This paper introduces a novel detection service, De-FiGuard, designed to detect PMAs using GNNs. De-FiGuard is a comprehensive solution that seamlessly integrates transaction parsing, graph construction, model training, and PMA detection.

• We present a unique approach to encapsulate trading behavior by translating raw transactions into cash flow graphs, enriched with features: node type, transfer frequency, transfer diversity, and profit score. Our findings indicate that the constructed cash flow graph with these four features significantly facilitates PMA detection. Moreover, we will release the graph-based dataset after the publication.

• We evaluated DeFiGuard’s performance in PMA detection using various metrics. Empowered by selected GNN models, DeFiGuard outperforms the baseline MLP model across metrics such as Accuracy, TPR, FPR, and AUC-ROC. An ablation study further confirms the importance of each feature to DeFiGuard’s effectiveness. Notably, DeFiGuard can classify a transaction in 0.892 to 5.317 seconds, faster than Ethereum’s block generation time of approximately 13 seconds.

## II. BACKGROUND

## A. The Primitives of EVM-based Blockchain

EVM-based blockchain. Blockchain technology, first introduced in 2008 by Satoshi Nakamoto [13], revolutionized digital currencies and enabled secure, decentralized financial transactions. The primary purpose of blockchain technology is to create a trustless, transparent, and tamper-proof digital ledger, recording transactions in a peer-to-peer (P2P) network. The participants in the P2P network can validate and store transaction data, thereby eliminating the need for centralized intermediaries such as banks. Transactions in the network are grouped into blocks and cryptographically secured using a consensus algorithm called Proof-of-Work (PoW). As a result, the blockchain provides an immutable record of all transactions since the inception of the network, ensuring data integrity and security. In 2014, Ethereum [14] was introduced to enable the execution of programmable agreements (i.e., smart contracts written in a Turing-completed language), allowing developers to build decentralized applications (DApp) that can automate complex processes. Specifically, the Ethereum blockchain utilizes the Ethereum Virtual Machine (EVM) as a runtime environment for executing smart contracts. Ethereum and Binance Smart Chain <sup>1</sup> are two prominent examples of EVM-based blockchains.

Accounts. In EVM-based blockchain systems, an account refers to a digital entity representing either a user or a smart contract. There are two types of accounts: Externally Owned Accounts (EOAs) and Contract Accounts (CAs). EOAs are controlled by individuals who own the corresponding private key, while CAs are controlled by smart contracts, which are snippets of JavaScript-like code. To create a CA, users must initiate a signed transaction that deploys their smart contract. This process generates a new address on the blockchain, which can handle and manage digital assets, perform complex business logic, and execute other smart contracts.

Transaction Call Trace. The transaction call trace in an EVM-based blockchain provides a detailed account of the execution flow of a transaction. Essentially, it is a step-by-step outline of how a transaction is processed, illustrating every action taken from the transaction’s initiation to its conclusion. The call trace captures the intricate interactions within and between smart contracts. Within a call trace, vital information is presented, including the caller account (initiator of a call), the callee account, the invocation call data, and the associated value (any native asset sent in the call).

Transaction Event Log. Transaction event logs in EVMbased blockchain systems are immutable records produced during smart contract execution. They offer transparency by logging pre-defined events in smart contracts. Analyzing these logs provides insights into contract interactions and outcomes. In this paper, we focus on the Transfer event in ERC20 standard token contracts, which log token transfers between accounts. For example, the parameters of the event Transfer(address from, address to, uint256 value) typically include the sender’s address (from), the recipient’s address (to), and the number of tokens being transferred (value). Moreover, the event emitter for this Transfer event is usually the ERC20 token contract address. The Transfer event offers a transparent record of all ERC20 token movements, ensuring traceability within the ERC20 ecosystem.

## B. Graph Neural Networks

Graph neural networks (GNNs) are a type of powerful machine learning method designed to work with graph-structured data. They have made significant advances across various applications, such as smart contract vulnerabilities detection [15] [9] [16] and other security analysis [17] [18]. Unlike other deep neural networks operating on Euclidean data (e.g., analyzing images via convolution neural networks), GNNs can gather graph structure information by iteratively aggregating each node’s information among their connections in a graph, which makes them an effective and useful tool for graph data exploration. Specifically, they employ message-passing algorithms to propagate information along the edges of a graph and update the node features. The updated node features can then be used for various downstream graph analysis tasks, such as node classification, link prediction, and graph classification. A typical GNN model will iteratively update each node’s representation (also known as node embedding) as:

$$
\begin{array}{l} \mathbf {m} _ {v} ^ {(t)} = \sum_ {u \in \mathcal {N} (v)} M _ {t} (\mathbf {h} _ {u} ^ {(t - 1)}, \mathbf {h} _ {v} ^ {(t - 1)}, \mathbf {e} _ {u v}), \\ \mathbf {h} _ {v} ^ {(t)} = U _ {t} (\mathbf {h} _ {v} ^ {(t - 1)}, \mathbf {m} _ {v} ^ {(t)}), \end{array}\tag{1}
$$

where $\mathbf { h } _ { v } ^ { ( t ) }$ indicates the embedding of node v at layer $t \in$ $\{ 1 , \ldots , T \}$ , and $\mathcal { N } ( v )$ denotes the set of neighbours of v in graph G. $M _ { t } ( \cdot , \cdot , \cdot )$ and $U _ { t } ( \cdot , \cdot )$ are the message function and the embedding updating function at layer $t ,$ respectively. Once the embeddings for each node in a graph are produced, the representation of the entire graph can be generated by combining them $( \mathrm { e . g . }$ , calculating the average value), which can then be used for downstream tasks (e.g., predicting the label of a graph).

## III. PROBLEM DEFINITION

## A. Problem

Given the proven efficacy of GNNs in graph learning tasks, we propose to depict transactions as graph data for convenient analysis and formulate cash flow graphs, denoted as G. Specifically, we use edges E to represent directed asset transfers, and nodes V to symbolize accounts sending or receiving assets. Therefore, the PMA detection task, which determines whether a transaction is executing a PMA for illicit financial gain, can be conceptualized as a task of GNNbased graph representation learning followed by classification. Formally, given a cash flow graph G, the objective is to build a graph classification model ${ \mathcal F } ,$ which outputs a prediction $\mathcal { P } .$ The prediction determines if transactions are PMAs $( \mathcal { P } = 1 )$ or non-PMAs $( \mathcal { P } = 0 )$ , as illustrated in Equation 2.

$$
\mathcal {P} \to \mathcal {F} (\mathcal {G}), w h e r e \mathcal {P} = 0 o r 1\tag{2}
$$

## B. Requirements

To accomplish the aforementioned task within the context of the DeFi domain, we enumerate several fundamental requirements.

Effectiveness. The proposed methodology should reliably detect an extensive spectrum of PMAs, with a strong performance on key metrics like Accuracy, True Positive Rate, False Positive Rate, and AUC-ROC.

Generability. The proposed methodology must learn and understand diverse behaviors with DApps, capture attacks from a limited dataset, and identify unknown attacks using various GNN algorithms.

Automation. System automation is crucial, spanning from transaction parsing to classification, ensuring seamless PMA detection in real time.

Adaptability. The proposed methodology should prioritize adaptability and quick updates to stay current with emerging detection techniques, including new transaction analysis methods and advanced GNN algorithms.

## IV. DESIGN

In this section, we first identify two challenges in our task. Then, we elaborate on the design rationale and the high-level architecture of our PMA detection framework DeFiGuard.

## A. Challenges

After formulating our problem as a task of learning graph representation and classification in a cash flow graph, we identified the following two main challenges.

Detecting an extensive spectrum of PMAs. Analyzing a diverse array of PMAs poses considerable difficulty when employing pattern-based detection techniques due to the evolving nature of the DeFi ecosystem. Prior work, e.g., DeFiRanger, proposes two patterns focusing on detecting direct and indirect PM $\mathbf { A } \mathbf { s } ,$ it strongly relies on the semantic knowledge extracted from DApps, which requires a significant effort to update their patterns to detect new PMAs. To effectively overcome the evolving nature of such attacks, it becomes imperative for the pattern design to exhibit robust adaptability and versatility. Consequently, the endeavor of crafting and revising these patterns demands substantial effort and resources.

Building an efficient and adaptive framework for PMA detection. Detecting PMAs in the EVM-based blockchains presents a challenge, particularly with regard to the temporal implications. Notably, the Ethereum network exhibits a mining duration of approximately 12 to 15 seconds for each new block. Consequently, if the task of detection and alerting fails to occur (i.e., prior to the subsequent block), the attack’s impact may intensify and disseminate more extensively.

## B. Cash Flow Graph

As aforementioned, it is challenging to identify a broader spectrum of PMAs with fixed patterns due to the evolving nature of PMAs in the DeFi ecosystem. In this paper, we introduce the concept of the cash flow graph, which encapsulates comprehensive information regarding asset transfers, making it a robust representation of trading behaviors. Specifically, the cash flow graph inherently embodies an abundance of information (such as transfer frequency, transfer diversity, and profit score) that can unveil malicious trading activities. Moreover, leveraging the cash flow graph with GNN techniques allows for a dynamic approach rather than adhering to static rule patterns. When training on the cash flow graph, the GNN model can discern and learn the intricacies of various PMAs. This facilitates the detection of a broader spectrum of PMAs, making it a more adaptive and robust solution.

![](images/817d222dbba06b77fef28068ea9655f3161fb0b85e92c04f1e2e727472d9f466.jpg)  
Fig. 1: The high-level architecture of DeFiGuard.

To facilitate a deeper understanding of the proposed cash flow graph, we elucidate a list of pertinent terminologies:

Node V: The node, denoted by V, represents a set of accounts sending or receiving assets. In this study, we classify nodes based on their account type and transparency.

Edges E: The edge in the graph represents the asset transfer in terms of the native asset and the ERC20 standard asset. Specifically, $\mathcal { E } \subseteq ( \mathcal { V } \mathrm { { X } } \mathcal { V } )$ is a set of directed edges.

Graph Metadata $\kappa \colon$ The graph metadata, denoted by $\kappa ,$ comprises $\kappa ^ { V }$ and ${ \boldsymbol { \kappa } } ^ { E }$ . It is extracted during the construction of the cash flow graph. Specifically, $\bar { \kappa } ^ { V }$ represents the metadata of nodes, which includes the account address, while ${ \boldsymbol { \kappa } } ^ { E }$ signifies the metadata of edges, encompassing both asset address and asset amount.

Node Feature X: Node feature, denoted by X, represents a set of node features extracted using the feature function $f _ { e x t r a c t }$ . The function $f _ { e x t r a c t }$ is designed to extract and normalize the feature set based on the nodes V, edges E, and graph metadata K of a provided transaction. Specifically, the function $f _ { e x t r a c t } ( \nu , \mathcal { E } , \mathcal { K } )$ accepts V, E and K as inputs and returns the feature set X.

Cash Flow Graph G: The cash flow graph, denoted by $\mathcal { G } =$ (V, E, X ), represents the asset transfer information within a

transaction.

## C. The Design of DeFiGuard

In this paper, we introduce DeFiGuard, an automatic detection service designed to address the two aforementioned challenges and perform PMA detection. To offer a highlevel overview of DeFiGuard, we discuss the design purpose behind each component. As depicted in Figure 1, DeFiGuard comprises three main components: Transaction Parser, Graph Builder, and Graph Classifier.

1) The Transaction Parser component is designed to parse transactions collected from EVM-based blockchains to retrieve their transfer-related call traces and event logs. As illustrated in Equation 3, we define the function $f _ { p a r s e }$ that accepts a transaction, tx, as its input and returns the call traces and event logs as its output.

$$
f _ {p a r s e} (t x) = T r a c e s _ {t x}, E v e n t s _ {t x}\tag{3}
$$

2) The Graph Builder is designed to construct a cash flow graph G using the filtered call traces and event logs. Within this component, we also extract and normalize four distinct classes of node features to facilitate the learning process. Specifically, the function $f _ { c o n s t r u c t }$ , as outlined in Equation 4, takes the filtered call traces and event logs as inputs and yields nodes V, edges E, and graph metadata K as outputs. Additionally, the function $f _ { e x t r a c t }$ in Equation 5 extracts the node features X and generates a cash flow graph G using the outputs from f<sub>construct</sub>.

$$
f _ {c o n s t r u c t} (T r a c e s _ {t x}, E v e n t s _ {t x}) = (\mathcal {V}, \mathcal {E}, \mathcal {K})\tag{4}
$$

$$
f _ {e x t r a c t} (\mathcal {V}, \mathcal {E}, \mathcal {K}) = \mathcal {G}\tag{5}
$$

3) The Graph Classifier is designed to predict whether the cash flow graph $\mathcal { G }$ corresponds to a PMA or non-PMA. During the training phase, given a collection of cash flow graphs $\mathcal { G } s$ associated labels $L a b e l s .$ , and a selected GNN algorithm $\mathcal { M } ,$ the function $f _ { t r a i n }$ (refer to Equation 6) outputs a trained GNN model ${ \mathcal F } .$ . For the inference phase, provided with a collection of cash flow graphs $\mathcal { G } s$ and a trained GNN model ${ \mathcal { F } } _ { : }$ the function $f _ { i n f e r e n c e }$ (refer to Equation 7) generates a prediction result P. This result indicates if the graph is a PMA $( \mathcal { P } = 1 )$ or non-PMA $( \mathcal { P } = 0 )$ .

$$
f _ {t r a i n} (\mathcal {G} s, L a b e l s, \mathcal {M}) = \mathcal {F}\tag{6}
$$

$$
f _ {i n f e r e n c e} (\mathcal {G} s, \mathcal {F}) = \mathcal {P}, w h e r e \mathcal {P} = 0 o r 1\tag{7}
$$

Further implementation details of each component in $D e F i .$ Guard can be found in Section V.

## V. THE DETAILS OF DeFiGuard

In this section, we delve deeper into the implementation details of DeFiGuard.

## A. Transaction Parser

The Transaction Parser aims to retrieve all transfer-related call traces and event logs from a transaction collected from the EVM-based blockchains. In an EVM-based blockchain, a function call trace refers to a detailed record of the execution step taken during the invocation of a smart contract function. As for the transaction event log, it is an immutable record pre-defined by the smart contract and produced during the transaction execution.

As shown in Figure 1, three steps are followed to generate the transfer-related call traces and event logs for the subsequent component. First, the Transaction Parser retrieves the raw transaction from EVM-based blockchains using a well-connected archive node. Second, the Transaction Parser replays the raw transaction to capture all associated call traces and event logs. Lastly, the Transaction Parser refines collected call traces and event logs by identifying the native and ERC20 standard asset transfers. Specifically, the native token transfer can be identified from the call traces by examining their value sector. Meanwhile, the ERC20 standard asset transfers are discerned based on the signature of the Transfer events. It is worth noting that transactions without asset transfers are not forwarded to the next component.

After selecting all transfer-related call traces and event logs for a transaction, the Transaction Parser sends them to the subsequent component for graph construction. Additionally, during the training phase, the transaction label is also provided to aid the training process.

## B. Graph Builder

Upon receiving the filtered call traces and event logs from the Transaction Parser, the Graph Builder undertakes the tasks of graph construction and feature extraction, subsequently generating the cash flow graph.

1) Graph Construction: As demonstrated in Figure 2, the Graph Builder iterates over all filtered call traces and event logs to translate them into a cash flow graph with the graph metadata. This translation is achieved by pinpointing senders, receivers, assets, and the respective asset amounts from both call traces and event logs.

The asset transfer regarding the native asset can be extracted from the call traces by analyzing three factors: caller, callee, and the sent native asset amount. In a filtered call trace, the function caller acts as the asset sender (caller acts as a $v _ { s e n d e r } )$ , while the function callee stands as the asset receiver (callee acts as a $v _ { r e c e i v e r } )$ . Moreover, the caller and callee address will be recorded in $\kappa ^ { V }$ . Subsequent to node identification, the edge $e ~ : ~ v _ { s e n d e r } \xrightarrow [ ] { K ^ { e } } ~ v _ { r e c e i v e r }$ is formulated, where $K ^ { e }$ encompasses both the asset type and its amount. Specifically, the asset type refers to the native token (e.g., ether on the Ethereum blockchain), and the asset amount is derived from the value sector within the call trace.

The asset transfer regarding the ERC20 token can be extracted from the Transfer event. As prescribed by the ERC20 token standard, the parameters of the Transfer <sup>2</sup> event delineates the sender, receiver, and asset amount. The event emitter signifies the asset transferred. After identifying the necessary information, the Graph Builder can produce the corresponding edge e and metadata K<sup>v</sup>, K<sup>e</sup>.

Once all nodes V, edges E, and their pertinent metadata K have been structured, the Graph Builder advances to extract features $\mathcal { X }$ to output the completed cash flow graph G.

2) Feature Extraction: Given the nodes V, edges E, and corresponding graph metadata K, the Graph Builder extracts four distinct classes (in Table I) of node features for each node. Node Type. In the EVM-based blockchains, there are two types of addresses: EOAs and CAs. Unlike EOAs, CAs can be triggered to execute complex logic based on their code. Moreover, the majority of DApps disclose and authenticate their source code during deployment, thereby allowing users to scrutinize the intricacies of their business logic. Conversely, smart contracts deployed by attackers typically manifest as unverified and lack transparency. Therefore, the Graph Builder captures the feature node type by encapsulating both the account type and the transparency for each node.

To verify the transparency of nodes, we constructed a keyvalue database, denoted as $D B _ { a c c o u n t }$ , through verifying precollected CA on etherscan.io and bscscan.io. In the database, the key corresponds to the account address, while the value, a boolean value, indicates whether the collected $\mathrm { C A s }$ are verified with their source code. Leveraging the $D B _ { a c c o u n t }$ database, the Graph Builder employs the account address recorded in $\kappa ^ { V }$ to retrieve both account type and associated transparency.

As illustrated in Feature Extraction 1, the Graph Builder iterates V and verifies the corresponding account address (retrieved from $\kappa ^ { V } )$ in $D B _ { a c c o u n t } .$ As a result, the node type feature, denoted by $\mathcal { X } _ { t y p e } ,$ , is extracted for all nodes V.

Transfer Frequency. An unusual number of asset transfers can act as a flag for potential price manipulation. Repeatedly trading a pair of assets in a transaction is not prevalent and is mostly considered malicious behavior such as washing trading, price manipulation, or reentrancy attack. Therefore, we capture the feature transfer frequency to represent a node’s asset sending and receiving frequency.

![](images/de51c59bf2cc2d0fc3d785c53ef934f5881ce7dd089558f28103b2964420a32d.jpg)  
The ‘value\*’ represents the amount of native assets, which can be extracted from the value sector of the function call trace. Fig. 2: The graph construction process.

TABLE I: The node features.

<table><tr><td>Feature</td><td>Notation</td><td>Description</td><td>Value Range</td></tr><tr><td>Node Type</td><td> $X_{type}$ </td><td>The type and transparency of an account</td><td>[0, 0, 1]or[0, 1, 0]or[0, 0, 1]</td></tr><tr><td>Transfer Frequency</td><td> $X_{frequency}$ </td><td>The number of times an account sends/receives assets.</td><td>∈ (0, 1]</td></tr><tr><td>Transfer Diversity</td><td> $X_{diversity}$ </td><td>The number of asset types sent/received by an account</td><td>∈ (0, 1]</td></tr><tr><td>Profit Score</td><td> $X_{profit}$ </td><td>The normalized value of the node’s profit</td><td>∈ [−1, 1]</td></tr></table>

<div class="mineru-algorithm" style="white-space: pre-wrap; font-family:monospace;">
Feature Extraction 1 Node Type
Input: $\mathcal{V}, \mathcal{K}^V, DB_{account}$
Output: $\mathcal{X}_{type}$ $\mathcal{X}_{type} \leftarrow \{\}$
    for $(v, \mathcal{K}^v) \in (\mathcal{V}, \mathcal{K}^V)$ do
        $account \leftarrow \mathcal{K}^v$
        if $DB_{account}[account]$ exists then
            if $DB_{account}[account] == False$ then
                $\mathcal{X}_{type}[v] \leftarrow [1, 0, 0]$ //Opaque CA
                $\mathcal{X}_{type}[v] \leftarrow [0, 1, 0]$ //Transparent CA
            $\mathcal{X}_{type}[v] \leftarrow [0, 0, 1]$ //EOA
return $\mathcal{X}_{type}$
</div>

As illustrated in Feature Extraction 2, given the nodes V and edges E, the Graph Builder iterates edges E and accumulates the number of incoming and outgoing edges for nodes V. Then, the Graph Builder further normalizes each node’s incoming and outgoing edge count by dividing the largest incoming and outgoing edge count. As a result, the transfer frequency feature, denoted by $\mathcal { X } _ { f r e q }$ is extracted for nodes V.

Transfer Diversity. Sophisticated PMAs often aim to create artificial arbitrage opportunities such as the infamous bZx attack [19]. An unusual number of assets being involved in a transaction can be indicative of such an attack. Therefore, we capture the feature transfer diversity to describe the number of different assets sent and received by the node.

<div class="mineru-algorithm" style="white-space: pre-wrap; font-family:monospace;">
Feature Extraction 2 Transfer Frequency
Input: $\mathcal{V}, \mathcal{E}$
Output: $\mathcal{X}_{frequency}$ $inEdge \leftarrow \{\}, outEdge \leftarrow \{\}, X_{freq} \leftarrow \{\}$
    for $e \in \mathcal{E}$ do
        $inEdge[e.receiver]++$ $outEdge[e.sender]++$ $maxIn \leftarrow max(inEdge)$ $maxOut \leftarrow max(outEdge)$
    for $v \in \mathcal{V}$ do
        $X_{frequency}[v] \leftarrow [\frac{inEdge[v]}{maxIn}, \frac{outEdge[v]}{maxOut}]$
    return $\mathcal{X}_{frequency}$
</div>

As illustrated in Feature Extraction 2, given the nodesV, edges E and metadata ${ \boldsymbol { \kappa } } ^ { E }$ , the Graph Builder iterates edges E and accumulates the number of incoming and outgoing assets for nodes V. Then, Graph Builder further normalizes each node’s incoming and outgoing asset count into the range of (0, 1] by dividing the largest incoming and outgoing asset count in the graph. As a result, the transfer diversity feature, denoted by $\mathcal { X } _ { d i v e r s i t y }$ is extracted for nodes V.

Profit Score. In a PMA, one or a few addresses often stand to gain disproportionately compared to others. Given the fact that the attacker’s goal is to gain illicit profits, anomalously high profits for a particular node amidst a complex transaction can be a red flag pointing toward manipulation. However, in a transaction, accurately calculating each node’s actual profit requires obtaining the involved assets’ decimals and the price data. Acquiring such information is expensive and might cause time delays for the graph construction task due to third-party resource dependency. Therefore, in this work, we propose a normalized value profit score to describe each node’s profit regardless of the asset decimals and price data.

<div class="mineru-algorithm" style="white-space: pre-wrap; font-family:monospace;">
Feature Extraction 3 Transfer Diversity
Input: $\mathcal{V}, \mathcal{E}, \mathcal{K}^E$
Output: $\mathcal{X}_{diversy}$ $inAsset \leftarrow \{\}$, $outAsset \leftarrow \{\}$, $X_{diversy} \leftarrow \{\}$
    for $(e, \mathcal{K}^e) \in (\mathcal{E}, \mathcal{K}^E)$ do
        $inAsset[e.receiver].append(\mathcal{K}^e.asset)$ $outAsset[e.sender].append(\mathcal{K}^e.asset)$ $maxIn \leftarrow MaxCount(inAsset)$ $maxOut \leftarrow MaxCount(outAsset)$
    for $v \in \mathcal{V}$ do
        $inCount \leftarrow set(inAsset[v]).length$ $outCount \leftarrow set(outAsset[v]).length$ $X_{diversity}[v] \leftarrow [\frac{inCount}{maxIn}, \frac{outCount}{maxOut}]$
    return $\mathcal{X}_{diversity}$
</div>

As demonstrated in Feature Extraction 4, given the edges $\mathcal { E }$ and metadata ${ \boldsymbol { \kappa } } ^ { E }$ , the Graph Builder iterates metadata ${ \boldsymbol { \kappa } } ^ { E }$ and finds the largest transfer amount for each asset. Then, the Graph Builder iterates edges E and accumulates the normalized amount for nodes based on the largest amount. As a result, the normalized profit, denoted by $\chi _ { p r o f i t }$ , is calculated for all nodes V.

<div class="mineru-algorithm" style="white-space: pre-wrap; font-family:monospace;">
Feature Extraction 4 Profit Score
Input: $\mathcal{E}, \mathcal{K}^E$
Output: $\mathcal{X}_{profit}$
    MaxAmount ← {}, $\mathcal{X}_{profit} \leftarrow {}$
    for $\mathcal{K}^e \in \mathcal{K}^E$ do
        asset ← $\mathcal{K}^e$.asset, amount ← $\mathcal{K}^e$.amount
        if amount &gt; MaxAmount(asset] then
            MaxAmount(asset] ← amount
    for $(e, \mathcal{K}^e) \in (\mathcal{E}, \mathcal{K}^E)$ do
        asset ← $\mathcal{K}^e$.asset, amount ← $\mathcal{K}^e$.amount
        $\mathcal{X}_{profit}[e.sender] = amount / MaxAmount(asset]$ $\mathcal{X}_{profit}[e.receiver] += amount / MaxAmount(asset]$
    return $\mathcal{X}_{profit}$
</div>

In conclusion, after the feature extraction process, a set of node features, denoted by $\chi \ = \ X _ { t y p e } + X _ { f r e q u e n c y } +$ $X _ { d i v e r s i t y } + X _ { p r o f i t }$ , is generated. Subsequent to it, the Graph Builder feeds the completed cash flow graph $\mathcal { G } = ( \nu , \mathcal { E } , \mathcal { X } )$ to the Graph Classifier for the training and inference process.

3) An Illustrative Example: To help understand the features, we elaborate on an illustrative example in Figure 3. Taking the node v1 as an example, since the account address of node v1 (i.e., the value of $\mathcal { K } ^ { v 1 } )$ does not exist in the $D B _ { a c c o u n t } .$ , so the x1 of v1 is [0, 0, 1]. The node v1 has 1 incoming and 1 outgoing edges. Based on analyzing the cash flow graph, we discover that the largest incoming and outgoing edge counts are both 2. Therefore, we can further calculate the value of $x 2 = \textstyle { \frac { 1 } { 2 } } = 0 . 5$ and $x 3 = \textstyle { \frac { 1 } { 2 } } = 0 . 5$ representing node v1’s transfer frequency. Similary, we can also calculate the value of $x 3 = \textstyle { \frac { 1 } { 2 } } = 0 . 5$ and $x 4 = \textstyle { \frac { 1 } { 2 } } = 0 . 5$ representing node v1’s transfer diversity. As for the profit score of the node v1, there are only two assets circulated in this example and the node v1 sent 1e17ether and received 1.1e17ether. Thus, we can calculate v1’s profit score $\begin{array} { r } { x 6 = \frac { 1 . 1 e 1 7 - 1 e 1 7 } { 1 . 1 e 1 7 } + \frac { 0 } { 1 . 2 e 8 } } \end{array}$ ≈ 0.091. As a result, given all values x1, x2, x3, x4, x5, and x6, we formulate the feature vector for the node v1, as denoted $\mathcal { X } [ v 1 ] = x 1 + [ x 2 , x 3 , x 4 , x 5 , x 6 ] =$ $\left[ 0 , 0 , 1 , 0 . 5 , 0 . 5 , 0 . 5 , 0 . 5 , 0 . 5 , 0 . 0 \dot { 9 } 1 \right]$

## C. Graph Classifier

As the last component of DeFiGuard, the Graph Classifier will be developed for predicting whether the transaction is benign or malicious based on the outputs from the Graph Builder. This component consists of two phases, the training phase which learns a GNN model, and the inference phase which predicts the labels based on the well-trained GNN.

Training Phase. During the training phase, the Graph Classifier component trains the GNN model with cash flow graphs $\mathcal { G }$ and their corresponding labels fed by the Graph Builder component. Then, it feeds the pre-trained GNN models to the next component for prediction. To fully utilize the fact that a raw transaction can be constructed as a cash flow graph, we intuitively choose to build a GNN model that directly learns graph knowledge and predicts graphs’ labels. In this work, we set out to explore multiple edge-cutting GNN algorithms that are able to conduct graph representation learning by incorporating neighbor node features and graph structural features for the graph classification task. Therefore, we can directly apply underlying GNN algorithms and output the trained model. Specifically, we set up to have two layers (i.e., L=2) using the GNN algorithms in DGL [20].

Inference Phase. During the inference phase, the Graph Classifier loads the pre-trained GNN model from the GNN learner component and predicts the labels of new cash flow graphs G fed by the Graph Builder. Particularly, the Graph Classifier classifies G based on its returned binary numbers ‘0’ and $\cdot _ { 1 } \cdot$ indicating benign and malicious transaction respectively.

## VI. DATASET

In this section, we first present the basic statistics of the collected dataset. Then, we conduct a distribution analysis in terms of transaction complexity.

## A. Dataset Collection

We collected 208 PMA and 2,080 non-PMA transactions in our transaction collection, sampled from the time range between Jan 2021 and Jan 2022. The PMA transactions were painstakingly curated by analyzing over 100 incidents reported in a public source <sup>3</sup>. Meanwhile, for the non-PMA transactions, we adopted a random sampling approach, targeting transactions on both Ethereum and BSC blockchains.

![](images/d3aa15b11cc7f59f853b26c0045f4ff5ca66f8a91869b1adc45a2b6520994824.jpg)  
Fig. 3: An example of a cash flow graph with four distinct features.

To maintain a high-quality dataset, we excluded transactions devoid of any asset transfers. Furthermore, to better mirror real-world complexities, we categorized our non-PMA transactions based on the number of asset transfers they encompassed. In particular, our complex non-PMA transaction subset only features transactions with more than 100 asset transfers. Table II reveals that the volume of non-PMA transactions we gathered is tenfold the PMA transactions. Specifically, the collection of non-PMA transactions consists of an even split: 1,040 simple and 1,040 complex transactions.

## B. Distribution Analysis

We present the basic statistics of collected transactions in terms of graph-based and transaction-based metrics in Table II. Moreover, we plot probability distribution figures for each metric to show the difference between collected PMA and non-PMA transactions in Figure 4 and 5.

TABLE II: The statistics of collected transactions in terms of graph- and transaction-based metrics.

<table><tr><td rowspan="2">Aspects</td><td rowspan="2">Metrics</td><td rowspan="2">PMA</td><td colspan="3">non-PMA</td></tr><tr><td>overall</td><td>simple</td><td>complex</td></tr><tr><td rowspan="4">Graph-based</td><td>Node Count</td><td>12.4</td><td>38.8</td><td>6.9</td><td>70.7</td></tr><tr><td>Edge Count</td><td>114.3</td><td>73.7</td><td>11.1</td><td>136.4</td></tr><tr><td>Asset Count</td><td>6.0</td><td>3.5</td><td>3.5</td><td>3.5</td></tr><tr><td>Node Degree*</td><td>21.5</td><td>18.9</td><td>3.1</td><td>34.7</td></tr><tr><td rowspan="2">Tx-based</td><td>Gas Cost (Ether)</td><td>1.4429</td><td>0.0715</td><td>0.0367</td><td>0.1063</td></tr><tr><td>Trace Count</td><td>858.2</td><td>192.7</td><td>73.5</td><td>311.9</td></tr><tr><td colspan="2">Transaction Count</td><td>208</td><td>2,080</td><td>1,040</td><td>1,040</td></tr></table>

Graph-based Analysis. Upon analyzing the dataset using graph-based metrics such as node count, edge count, asset count, and node degree, we observed that PMA transactions have higher average values for edge count, asset count, and node degree compared to non-PMA transactions. Probability distribution analysis further revealed distinct patterns: PMA transactions show peak occurrences in the ranges of (6,9] for node count, (40,60] for edge count, (2, 4] for asset count, and (10,15] for node degree. Conversely, non-PMA transactions fall within the initial range for these metrics, highlighting a marked differentiation between the two transaction types.

Transaction-based Analysis. Upon scrutinizing transaction complexities using the transaction-based metrics (i.e., gas cost and trace count), distinct differences emerge between PMA and non-PMA transactions. The statistical result (in Table II) indicates that PMA transactions incur an average gas cost of 1.4429 ether, in contrast to a significantly lower 0.0715 ether for non-PMA transactions. Additionally, the trace count for PMA transactions stands at a substantial 858.2 on average, surpassing the non-PMA average of 192.7. The probability distribution of these metrics further clarifies this distinction. About 50% of non-PMA transactions have a gas cost confined to the range [0, 2e+16], whereas a mere 25% of PMA transactions adhere to this interval. Instead, the latter’s gas cost displays a conspicuous spike within the range (2.6e17, 2.8e17]. In terms of trace count, while over half of the non-PMA transactions fall within the bounds of (0, 100], a mere 20% of PMA transactions do so, with notable peaks observed in the intervals (1100,1200] and (1300,1400]. Notably, the probability distribution for non-PMA transactions reveals an exponential decline as both gas cost and trace count increase.

In conclusion, the collected PMA transactions are more complex than the sampled non-PMA transactions in terms of graph-based and transaction-based metrics.

## VII. EVALUATION

In this section, we conduct to evaluate DeFiGuard by answering the following research questions:

• RQ1: How does the proposed graph-based approach perform in detecting PMAs across various graph neural network models?

• RQ2: How significantly do extracted features enhance the detection of PMAs?

![](images/cb11be1587856d815b279994445ce9da2c0f853af8b5a10033d30ba04ee06a13.jpg)

![](images/7777d577f97275d2088c0e4eac6c652709a35ef5337888bb9b33d9aa7ab409e2.jpg)

![](images/cd903ee75cbdb59ffdc2e925566de11b01908bdd7c13cd439d3770551f913da7.jpg)

![](images/78f4eedfb9da2107848d2ef65d036c63feaebdc0569bd512b345d53a5c695751.jpg)

![](images/dc2b180da52ab0af21b53f98f16a072b614ebbb119378bf99fc42db3f4428954.jpg)

![](images/71cc19ff45a2e8a49eef377764b8648ebd4a4931acffea015ae247840c0152d1.jpg)

![](images/59a531bfdd2bf7121c6fe9be444a3a86cfc64e721a45f7a02006c087f5655fb5.jpg)

![](images/350964cb2b4d8fc55d4818a956b9abcd81c43e90dfaa3cb2c625854a5cfc9072.jpg)  
Fig. 4: Probability distribution analysis for graph-based metrics.

• RQ3: How practical is our service in identifying PMA transactions within real-world blockchains?

To answer RQ1, we assess the performance of the PMA detection task using several leading GNN models, including GCN, GAT, GraphSage, and GIN, in comparison with a baseline model, MLP. For a concise evaluation of the models effectiveness, we consider a range of metrics, including Accuracy, TPR, FPR, and AUC-ROC, for our trained classifiers.

To answer RQ2, we undertake an ablation study, highlighting the impact of our proposed features on the PMA detection task. We evaluate several variants, namely: without node type, without transfer frequency, without transfer diversity, and without profit score.

To answer RQ3, we measure the time cost of each component in DeFiGuard in the real-world scenario. Specifically, we start the evaluation from the phase of collecting newly processed transactions on the chain to the phase of predicting whether the transaction is launching the PMA.

![](images/2c4ea262e13ea578427895edea6577acfc871dadeda2fa0e1417fc036920cd2a.jpg)

![](images/52bc1b452155b4d98d5b78084f41e1a9a186559ee4256aade41947f6385d7a8d.jpg)

![](images/4fc013aa991ebdf3b62155fc5a2102684e28e13368318354cb1e04333a71c801.jpg)

![](images/c03dadb4078a030d30b456123b64add28833589fef4972dbef05e322e2c05a51.jpg)  
Fig. 5: Probability distribution analysis for transactionbased metrics.

## A. Experimental Setup

Selected Models. For the assessment of GNNs in detecting PMAs, we have chosen four cutting-edge GNN models (i.e., GCN [21], GAT [22], GraphSAGE [23], GIN [24]) and a baseline model (i.e., MLP [25]).

Hyperparameters. For all models presented in Section VII-B and Section VII-C, we used ReLU as the non-linear activation function and Adam optimizer as the optimization algorithm. All GNN and the baseline models we devised consist of two layers, including a hidden layer with a dimensionality of 16 and an output layer with a dimension of 2 to conduct a binary classification task. Based on the experiments on evaluating the hyperparameters in Section VII-B, we trained all models for 100 epochs with a balanced train size of 100.

Hardware and Software. We conduct our evaluations on a Mac Mini machine, equipped with an Apple M1 chip (8-core CPU) and 16GB of RAM. Besides, DeFiGuard is implemented with Python 3.8.9, Pytorch 1.3.1 <sup>4</sup>, and DGL 0.9.1 <sup>5</sup>.

Evaluation Metrics. In evaluating the performance of binary classification, we employ Accuracy, TPR, FPR, and AUC-ROC metrics that are recognized and utilized in assessing binary classification outcomes: 1) Accuracy is a metric used to measure the percentage of correctly predicted outcomes; 2) True positive rate $\begin{array} { r } { ( T \bar { P } R = \frac { T P } { T P + F N } ) } \end{array}$ is a metric used to evaluate the ability to correctly identify positive cases in the data; 3) False positive rate $\begin{array} { r } { ( \dot { F } P R = \frac { F P } { F P + T N } ) } \end{array}$ is a metric used to evaluate the ability to falsely identify negative cases as positive; 4) Area under the curve (AUC-ROC, as shown in Equation 8) is a common metric used to evaluate the performance of binary classification models.

$$
\mathrm{AUC} = \int_ {- \infty} ^ {\infty} T P R \left(F P R ^ {- 1} (x)\right), d F (x)\tag{8}
$$

## B. Performance Analysis (RQ1)

Overall Performance. We systematically assessed the performance of four GNN models against a baseline model using our collected dataset. A concise summary of their performance, given identical training and testing configurations, is delineated in Table III. Evidently, DeFiGuard when integrated with GNN models consistently outperformed the baseline across all evaluated metrics. For instance, the combination of DeFiGuard with the GraphSage model emerges as particularly distinguished, achieving 93.25% in Accuracy, 91.67% in TPR, 6.67% in FPR, and 96.18% in AUC. From the analysis, several key insights can be gleaned. Primarily, DeFiGuard, when parameterized with a train size of 100, delivers respectful performance spanning Accuracy, TPR, FPR, and AUC-ROC. This validates the effectiveness of our cash flow graph design and four introduced features. On the other hand, when compared to the baseline MLP model, all GNN models exhibited enhanced performance. This indicates DeFiGuard, when paired with GNN models, effectively utilizes the proposed features to identify structural variances, ensuring accurate classification of PMAs.

TABLE III: Summary of the models’ performance for PMA detection on our collected dataset.

<table><tr><td>Model</td><td>Accuracy</td><td>TPR</td><td>FPR</td><td>AUC</td></tr><tr><td>MLP (baseline)</td><td>0.8223</td><td>0.8333</td><td>0.1783</td><td>0.8911</td></tr><tr><td>GCN</td><td>0.8530</td><td>0.8796</td><td>0.1485</td><td>0.9341</td></tr><tr><td>GAT</td><td>0.9009</td><td>0.8704</td><td>0.0975</td><td>0.9518</td></tr><tr><td>GIN</td><td>0.8812</td><td>0.9167</td><td>0.1207</td><td>0.9544</td></tr><tr><td>GraphSAGE</td><td>0.9325</td><td>0.9167</td><td>0.0667</td><td>0.9618</td></tr></table>

Tuning Epoch&Train Size. In the optimization process of our model, it is imperative to determine the most suitable hyperparameters to ensure both robustness and efficiency. In the tuning process, two critical parameters Epoch and Train Size are under consideration. The epoch determines how many times the learning algorithm will work through the entire training dataset, while the training size indicates the number of PMA and non-PMA transactions used for each learning iteration. To pinpoint the optimal pairing of these hyperparameters, we systematically evaluated various combinations and monitored their respective performances.

Using the GraphSage algorithm as a representative example, we visualized the results through a series of heatmaps (in Figure 6), each emblematic of a specific performance metric: Accuracy, TPR, FPR, and AUC-ROC. Each heatmap delineates train size on the x-axis and epoch on the y-axis. The epoch varies from 20 to 100 with intervals of 10, and analogously, the train size ranges similarly. Our observations from this evaluation are multifaceted. Firstly, the classifier’s performance in terms of Accuracy, FPR, and AUC-ROC escalates with increasing values of both the epoch and train size. Secondly, it’s discernible that the classifier can yield respectable accuracy even with relatively smaller train sizes and epochs, specifically greater than 60 in both dimensions. Most notably, the classifier manifests its peak performance in Accuracy, FPR, and AUC-ROC metrics at the right top of the heatmaps (epoch = 100 and train size = 100), while its TPR is the second best. Given this superior performance profile, we have elected to adopt the hyperparameters of epoch = 100 and train size = 100 for our subsequent evaluations.

Answer to RQ 1: Upon integration with GNN models, DeFiGuard markedly outperforms the baseline MLP model. The distinct advantage ofGNN models lies in their ability to recognize structural differences, an aspect that the MLP model appears to be less efficient regarding PMA detection. Moreover, the tuning results suggest that training with only 100 PMA and non-PMA transactions across 100 epochs yields strong performance, highlighting the cash flow graph’s effectiveness in capturing the trading behavior of the transaction.

## C. Ablation Study (RQ2)

To investigate how node features embedded in the cash flow graph impact the performance of DeFiGuard, we conducted ablation studies on four variants: 1) variant without node type, 2) variant without transfer frequency, 3) variant without transfer diversity, and 4) variant without the profit score.

Using the GraphSage model as an illustrative example, Fig ure 7 depicts the performance metrics for the four variants in relation to the original, which integrates all extracted features. Collectively, the performance of these variants of DeFiGuard is inferior to the original. When the node type is excluded (variant 1), there is a decrease in DeFiGuard’s Accuracy by 12.10%, TPR by 2.96%, and AUC-ROC by 5.54%. Conversely, its FPR escalates by 12.59%. In the absence of the transfer frequency feature (variant 2), the Accuracy of DeFiGuard diminishes by 6.22%, TPR by 1.57%, and AUC-ROC by 2.39%. Meanwhile, its FPR rises by 6.47%. Omitting the transfer diversity feature (variant 3) results in a decline in DeFiGuard’s Accuracy by 5.43%, TPR by 0.83%, and AUC-ROC by 1.32%. Additionally, there’s an increment in FPR by 5.67%. Without the profit score (variant 4), DeFiGuard witnesses a reduction in Accuracy and AUC-ROC by 4.80% and 0.28% respectively. On the other hand, both TPR and FPR exhibit increases of 0.55% and 5.08% respectively. The result implies that DeFiGuard is contributed by all features rather than certain dominating factors.

Answer to RQ 2: The ablation results indicate that the efficacy of DeFiGuard is contributed by the combination of all extracted features. This collaborative feature interplay ensures that DeFiGuard’s performance is not dominated by any single factor.

![](images/25dedc4ef2bef90d9baa66fc17d3e3c15a59262eb2349ef451e508b86ec0a40d.jpg)

![](images/07fef2651925a7c47c23e2a1e09b96b1d7833f0b4bae91be2a0f8e6b1122e62e.jpg)

<table><tr><td></td><td colspan="3">Accuracy</td><td>0.8</td><td>0.86</td><td>0.85</td><td>0.88</td><td>0.88</td><td>0.91</td></tr><tr><td rowspan="9">Epoch</td><td>100</td><td>0.85</td><td>0.82</td><td>0.84</td><td>0.8</td><td>0.86</td><td>0.85</td><td>0.88</td><td>0.88</td></tr><tr><td>90</td><td>0.83</td><td>0.83</td><td>0.82</td><td>0.86</td><td>0.82</td><td>0.82</td><td>0.88</td><td>0.9</td></tr><tr><td>80</td><td>0.79</td><td>0.8</td><td>0.81</td><td>0.87</td><td>0.9</td><td>0.89</td><td>0.89</td><td>0.88</td></tr><tr><td>70</td><td>0.76</td><td>0.86</td><td>0.83</td><td>0.86</td><td>0.87</td><td>0.86</td><td>0.89</td><td>0.88</td></tr><tr><td>60</td><td>0.79</td><td>0.79</td><td>0.86</td><td>0.79</td><td>0.85</td><td>0.86</td><td>0.86</td><td>0.86</td></tr><tr><td>50</td><td>0.82</td><td>0.78</td><td>0.84</td><td>0.87</td><td>0.8</td><td>0.85</td><td>0.87</td><td>0.88</td></tr><tr><td>40</td><td>0.7</td><td>0.77</td><td>0.8</td><td>0.76</td><td>0.83</td><td>0.83</td><td>0.85</td><td>0.86</td></tr><tr><td>30</td><td>0.71</td><td>0.76</td><td>0.74</td><td>0.78</td><td>0.86</td><td>0.83</td><td>0.85</td><td>0.83</td></tr><tr><td>20</td><td>0.68</td><td>0.73</td><td>0.78</td><td>0.82</td><td>0.81</td><td>0.84</td><td>0.82</td><td>0.81</td></tr></table>

Fig. 6: The summary of GraphSage model’s performance in the hyperparameter tuning process.

![](images/38b20da606d9e87d1f5f4dd720912dd9d92a04b57b8581d4afc43aa6a62b8fe5.jpg)  
Fig. 7: The ablation study on four variants.

## D. Time Cost (RQ3)

Detecting and responding to attacking transactions in a timely manner is crucial to minimize the potential damage caused by malicious activities in the DeFi ecosystem. A service with low time cost can quickly identify suspicious transactions, trigger alerts, and enable swift mitigation measures to prevent further harm to the DeFi ecosystem. To assess the practicality of DeFiGuard, we conduct a comprehensive evaluation by quantifying the time cost associated with detecting PMA transactions. This evaluation encompasses the entire workflow, encompassing the transactions parsing, graph construction, and graph classification. By measuring the time required for each stage, we can effectively evaluate the efficiency and feasibility of DeFiGuard in detecting PMAs (in Table IV).

TABLE IV: Time cost.

<table><tr><td rowspan="2">Phase</td><td rowspan="2">Malicious</td><td colspan="2">Benign</td></tr><tr><td>complex</td><td>simple</td></tr><tr><td>Transaction Parsing</td><td>4,383ms</td><td>1,198ms</td><td>394ms</td></tr><tr><td>Graph Construction</td><td>934ms</td><td>894ms</td><td>435ms</td></tr><tr><td>Graph Classification</td><td>0.093ms</td><td>0.098ms</td><td>0.090ms</td></tr><tr><td>Total Time Cost</td><td>5,317ms</td><td>2,047ms</td><td>829ms</td></tr></table>

Transaction Parsing. In the transaction parsing phase, we measure the time cost spent by the transaction parser in extracting call traces and event logs from the raw transaction. The time consumption in this phase depends on the amount of original call traces and event logs retrieved after the transaction replaying process. As a result, embedded with a node well-connected to the blockchain network, transaction parser spends 4,383ms, 1,198ms, and 394ms to extract call traces and event logs from the PMA, complex non-PMA, and simple non-PMA transactions on average. The reason why parsing the PMA transaction has the highest time cost is that the PMA transaction normally consists of more invocation to execute the complex attacking logic.

Graph Construction. In the graph construction phase, we measure the time cost spent by the graph builder in constructing the cash flow graph and extracting the corresponding features. The time consumption in this phase depends on the amount of call traces and event logs fed by the transaction parser. As a result, graph builder spends 934ms, 894ms, and 435ms on average to extract call traces and event logs from the PMA, complex non-PMA, and simple non-PMA transactions.

Graph Classification. In the graph classification phase, we measure the time cost spent by Graph Classifier in predicting the received cash flow graphs. As a result, the graph classifier consumes less than 0.1ms (0.094ms for a single transaction on average) to predict all types of transactions and provides an average throughput of 10,605 transactions per second.

In conclusion, DeFiGuard spends 5,317ms, 2,047ms, and 829ms to complete the classification task from the parsing to predicting. It is worth noting that the time cost of predicting all types of transactions is less than the block mining time (12-14 seconds) on Ethereum. With this rapid classification, the project can deploy DeFiGuard to evaluate the transaction interacting with their smart contracts. The rapid detection enables the project to activate the pausing mechanism to prevent potential loss.

Answer to RQ 3: Overall, the time cost of completing the classification for a single transaction ranges from 0.892 to 5.317 seconds, which is less than the time of creating a new block on Ethereum. The result implies that DeFiGuard is feasible and practical in terms of time cost so that the victims (including DApps and users) can have sufficient time to rescue their vulnerable assets.

## VIII. RELATED WORK

DeFi Security. Smart contract vulnerability detection is crucial for ensuring the security and reliability of the DeFi ecosystem. Given the irreversible and transparent nature of blockchain transactions, smart contract vulnerabilities can result in significant financial losses. Numerous academic endeavors have delved into this domain, leveraging a spectrum of methods spanning static analysis, dynamic analysis, and learning-driven methods. Static analysis identifies vulnerabilities through systematic code path probing (e.g., symbolic execution [26]– [34]) and pattern recognition (i.e., formal verification [35], [36]). Dynamic analysis (e.g., fuzzing techniques [37]–[46]) examines the hidden vulnerabilities in in-execution analysis. Meanwhile, learning-based methods have also shown great promise in detecting vulnerabilities (e.g., utilizing GNNs for scrutinizing control- and data-flow graphs extracted from smart contracts [9], [15], [16]). Beyond the realm of smart contract vulnerability detection, a substantial body of research is probing security concerns associated with token systems, DApps, and other related facets. Qin et al. [1], [2] investigated extractable values latent in the blockchain network, subsequently introducing an attack strategy that emulates profitable transactions sourced from the P2P network. The scholarly discourse has extensively addressed security predicaments, including but not limited to, attack detection [47] [5], frontrunning [3], [4], [48], governance issues [49], flash loan attack [50]. Complementing these, Sam et al. [51] executed a comprehensive assessment of security challenges, both theorized and those manifesting in real-world scenarios.

GNNs for Cybersecurity. GNNs have emerged as a powerful tool in the field of cybersecurity due to their ability to model complex relationships between entities and events. Particularly,

GNNs can effectively capture the structural dependencies and interactions to detect and mitigate cyber threats in the field of code vulnerability detection [52]–[56], network intrusion [17], and spam detection in social networks [10]–[12]. For instance, Mirsky et al. [52]. introduced VulChecker, a tool employing a new program representation, slicing strategy, and messagepassing graph neural network that precisely pinpoints and classifies vulnerabilities in source code. King et al. [18] proposed a framework combining GNNs and recurrent neural networks and achieving state-of-the-art performance in anomalous lateral movement detection. Yang et al. [17] designed a web tracking and advertising detection framework based on GNNs to analyze HTTP network traffic. Bian et al. [12] introduced the Bi-Directional Graph Convolutional Networks (Bi-GCN), a novel bi-directional graph model that captures both propagation and dispersion characteristics of rumors on social media.

## IX. CONCLUSION

This paper introduces the novel detection service, DeFi-Guard, which utilizes Graph Neural Networks (GNNs) for PMA detection. By transforming raw transactions into cash flow graphs enriched with four distinct node features and capitalizing on the advantages of GNN models, DeFiGuard consistently surpasses traditional models across various metrics. Furthermore, time cost evaluations validate DeFiGuard’s efficiency, ensuring potential victims have sufficient time to secure their assets upon PMA detection. This work serves as a pivotal advancement in safeguarding the DeFi landscape from PMAs.

## REFERENCES

[1] K. Qin, L. Zhou, and A. Gervais, “Quantifying blockchain extractable value: How dark is the forest?” in SP. IEEE, 2022, pp. 198–214.

[2] K. Qin, S. Chaliasos, L. Zhou, B. Livshits, D. Song, and A. Gervais, “The blockchain imitation game,” arXiv preprint arXiv:2303.17877, 2023.

[3] P. Daian, S. Goldfeder, T. Kell, Y. Li, X. Zhao, I. Bentov, L. Breidenbach, and A. Juels, “Flash boys 2.0: Frontrunning in decentralized exchanges, miner extractable value, and consensus instability,” in SP. IEEE, 2020, pp. 910–927.

[4] L. Zhou, K. Qin, C. F. Torres, D. V. Le, and A. Gervais, “High-frequency trading on decentralized on-chain exchanges,” in SP. IEEE, 2021, pp. 428–445.

[5] S. Wu, D. Wang, J. He, Y. Zhou, L. Wu, X. Yuan, Q. He, and K. Ren, “Defiranger: Detecting price manipulation attacks on defi applications,” arXiv preprint arXiv:2104.15068, 2021.

[6] L. Zhou, X. Xiong, J. Ernstberger, S. Chaliasos, Z. Wang, Y. Wang, K. Qin, R. Wattenhofer, D. Song, and A. Gervais, “Sok: Decentralized finance (defi) attacks,” in SP. IEEE, 2023, pp. 2444–2461.

[7] SlowMist, “Slowmist hacked,” https://hacked.slowmist.io/, 2022.

[8] De.Fi, “De.fi rekt database,” https://de.fi/rekt-database, 2022.

[9] Y. Zhuang, Z. Liu, P. Qian, Q. Liu, X. Wang, and Q. He, “Smart contract vulnerability detection using graph neural network,” in IJCAI. ijcai.org, 2020, pp. 3283–3290.

[10] H. Wang, T. Xu, Q. Liu, D. Lian, E. Chen, D. Du, H. Wu, and W. Su, “MCNE: an end-to-end framework for learning multiple conditional network representations of social network,” in KDD. ACM, 2019, pp. 1064–1072.

[11] W. Fan, Y. Ma, Q. Li, Y. He, Y. E. Zhao, J. Tang, and D. Yin, “Graph neural networks for social recommendation,” in WWW. ACM, 2019, pp. 417–426.

[12] T. Bian, X. Xiao, T. Xu, P. Zhao, W. Huang, Y. Rong, and J. Huang, “Rumor detection on social media with bi-directional graph convolutional networks,” in AAAI. AAAI Press, 2020, pp. 549–556.

[13] S. Nakamoto, “Bitcoin: A peer-to-peer electronic cash system,” Decentralized business review, p. 21260, 2008.

[14] G. Wood et al., “Ethereum: A secure decentralised generalised transaction ledger,” Ethereum project yellow paper, vol. 151, no. 2014, pp. 1–32, 2014.

[15] Z. Liu, P. Qian, X. Wang, L. Zhu, Q. He, and S. Ji, “Smart contract vulnerability detection: from pure neural network to interpretable graph feature and expert pattern fusion,” arXiv preprint arXiv:2106.09282, 2021.

[16] Z. Liu, P. Qian, X. Wang, Y. Zhuang, L. Qiu, and X. Wang, “Combining graph neural networks with expert knowledge for smart contract vulnerability detection,” IEEE Trans. Knowl. Data Eng., vol. 35, no. 2, pp. 1296–1310, 2023.

[17] Z. Yang, W. Pei, M. Chen, and C. Yue, “WTAGRAPH: web tracking and advertising detection using graph neural networks,” in SP. IEEE, 2022, pp. 1540–1557.

[18] I. J. King and H. H. Huang, “Euler: Detecting network lateral movement via scalable temporal graph link prediction,” in NDSS. The Internet Society, 2022.

[19] “bzx hack full disclosure (with detailed profit analysis),” https://de.fi/ rekt-database/bzx, 2020.

[20] “Deep graph library: Towards efficient and scalable deep learning on graphs,” in ICLR. OpenReview.net, 2019.

[21] T. N. Kipf and M. Welling, “Semi-supervised classification with graph convolutional networks,” arXiv preprint arXiv:1609.02907, 2016.

[22] P. Velickovic, G. Cucurull, A. Casanova, A. Romero, P. Liò, and Y. Bengio, “Graph attention networks,” in ICLR (Poster). OpenReview.net, 2018.

[23] W. L. Hamilton, Z. Ying, and J. Leskovec, “Inductive representation learning on large graphs,” in NIPS, 2017, pp. 1024–1034.

[24] K. Xu, W. Hu, J. Leskovec, and S. Jegelka, “How powerful are graph neural networks?” arXiv preprint arXiv:1810.00826, 2018.

[25] M. Defferrard, X. Bresson, and P. Vandergheynst, “Convolutional neural networks on graphs with fast localized spectral filtering,” in NIPS, 2016, pp. 3837–3845.

[26] L. Luu, D. Chu, H. Olickel, P. Saxena, and A. Hobor, “Making smart contracts smarter,” in CCS. ACM, 2016, pp. 254–269.

[27] E. Albert, P. Gordillo, B. Livshits, A. Rubio, and I. Sergey, “Ethir: A framework for high-level analysis of ethereum bytecode,” in ATVA, ser. Lecture Notes in Computer Science, vol. 11138. Springer, 2018, pp. 513–520.

[28] I. Nikolic, A. Kolluri, I. Sergey, P. Saxena, and A. Hobor, “Finding the greedy, prodigal, and suicidal contracts at scale,” in ACSAC. ACM, 2018, pp. 653–663.

[29] C. F. Torres, M. Steichen, and R. State, “The art of the scam: Demystifying honeypots in ethereum smart contracts,” arXiv preprint arXiv:1902.06976, 2019.

[30] M. Mossberg, F. Manzano, E. Hennenfent, A. Groce, G. Grieco, J. Feist, T. Brunson, and A. Dinaburg, “Manticore: A user-friendly symbolic execution framework for binaries and smart contracts,” in ASE. IEEE, 2019, pp. 1186–1189.

[31] S. Wang, C. Zhang, and Z. Su, “Detecting nondeterministic payment bugs in ethereum smart contracts,” Proc. ACM Program. Lang., vol. 3, no. OOPSLA, pp. 189:1–189:29, 2019.

[32] C. F. Torres, J. Schütte, and R. State, “Osiris: Hunting for integer bugs in ethereum smart contracts,” in ACSAC. ACM, 2018, pp. 664–676.

[33] P. Bose, D. Das, Y. Chen, Y. Feng, C. Kruegel, and G. Vigna, “SAIL-FISH: vetting smart contract state-inconsistency bugs in seconds,” in SP. IEEE, 2022, pp. 161–178.

[34] S. So, S. Hong, and H. Oh, “Smartest: Effectively hunting vulnerable transaction sequences in smart contracts through language model-guided symbolic execution,” in USENIX Security Symposium. USENIX Association, 2021, pp. 1361–1378.

[35] S. Azzopardi, J. Ellul, and G. J. Pace, “Monitoring smart contracts: Contractlarva and open challenges beyond,” in RV, ser. Lecture Notes in Computer Science, vol. 11237. Springer, 2018, pp. 113–137.

[36] J. Frank, C. Aschermann, and T. Holz, “ETHBMC: A bounded model checker for smart contracts,” in USENIX Security Symposium. USENIX Association, 2020, pp. 2757–2774.

[37] C. Schneidewind, I. Grishchenko, M. Scherer, and M. Maffei, “ethor: Practical and provably sound static analysis of ethereum smart contracts,” in CCS. ACM, 2020, pp. 621–640.

[38] L. Brent, N. Grech, S. Lagouvardos, B. Scholz, and Y. Smaragdakis, “Ethainter: a smart contract security analyzer for composite vulnerabilities,” in PLDI. ACM, 2020, pp. 454–469.

[39] A. Ghaleb, J. Rubin, and K. Pattabiraman, “etainter: detecting gasrelated vulnerabilities in smart contracts,” in ISSTA. ACM, 2022, pp. 728–739.

[40] S. Kalra, S. Goel, M. Dhawan, and S. Sharma, “ZEUS: analyzing safety of smart contracts,” in NDSS. The Internet Society, 2018.

[41] P. Tsankov, A. M. Dan, D. Drachsler-Cohen, A. Gervais, F. Bünzli, and M. T. Vechev, “Securify: Practical security analysis of smart contracts,” in CCS. ACM, 2018, pp. 67–82.

[42] F. Contro, M. Crosara, M. Ceccato, and M. D. Preda, “Ethersolve: Computing an accurate control-flow graph from ethereum bytecode,” in ICPC. IEEE, 2021, pp. 127–137.

[43] N. Grech, M. Kong, A. Jurisevic, L. Brent, B. Scholz, and Y. Smaragdakis, “Madmax: surviving out-of-gas conditions in ethereum smart contracts,” Proc. ACM Program. Lang., vol. 2, no. OOPSLA, pp. 116:1– 116:27, 2018.

[44] M. Rodler, W. Li, G. O. Karame, and L. Davi, “Sereum: Protecting existing smart contracts against re-entrancy attacks,” arXiv preprint arXiv:1812.05934, 2018.

[45] J. Feist, G. Grieco, and A. Groce, “Slither: a static analysis framework for smart contracts,” in WETSEB@ICSE. IEEE / ACM, 2019, pp. 8–15.

[46] S. Tikhomirov, E. Voskresenskaya, I. Ivanitskiy, R. Takhaviev, E. Marchenko, and Y. Alexandrov, “Smartcheck: Static analysis of ethereum smart contracts,” in WETSEB@ICSE. ACM, 2018, pp. 9– 16.

[47] Y. Gai, L. Zhou, K. Qin, D. Song, and A. Gervais, “Blockchain large language models,” arXiv preprint arXiv:2304.12749, 2023.

[48] S. Eskandari, S. Moosavi, and J. Clark, “Sok: Transparent dishonesty: Front-running attacks on blockchain,” in Financial Cryptography Workshops, ser. Lecture Notes in Computer Science, vol. 11599. Springer, 2019, pp. 170–189.

[49] L. Gudgeon, D. Perez, D. Harz, A. Gervais, and B. Livshits, “The decentralized financial crisis: Attacking defi,” arXiv preprint arXiv:2002.08099, 2020.

[50] K. Qin, L. Zhou, B. Livshits, and A. Gervais, “Attacking the defi ecosystem with flash loans for fun and profit,” arXiv preprint arXiv:2003.03810, 2020.

[51] S. M. Werner, D. Perez, L. Gudgeon, A. Klages-Mundt, D. Harz, and W. J. Knottenbelt, “Sok: Decentralized finance (defi),” arXiv preprint arXiv:2101.08778, 2021.

[52] Y. Mirsky, G. Macon, M. D. Brown, C. Yagemann, M. Pruett, E. Downing, S. Mertoguno, and W. Lee, “Vulchecker: Graph-based vulnerability localization in source code,” in USENIX Security Symposium. USENIX Association, 2023.

[53] Y. Zhou, S. Liu, J. K. Siow, X. Du, and Y. Liu, “Devign: Effective vulnerability identification by learning comprehensive program semantics via graph neural networks,” in NeurIPS, 2019, pp. 10 197–10 207.

[54] X. Cheng, H. Wang, J. Hua, G. Xu, and Y. Sui, “Deepwukong: Statically detecting software vulnerabilities using deep graph neural network,” ACM Trans. Softw. Eng. Methodol., vol. 30, no. 3, pp. 38:1–38:33, 2021.

[55] H. Wang, G. Ye, Z. Tang, S. H. Tan, S. Huang, D. Fang, Y. Feng, L. Bian, and Z. Wang, “Combining graph-based learning with automated data collection for code vulnerability detection,” IEEE Trans. Inf. Forensics Secur., vol. 16, pp. 1943–1958, 2021.

[56] V. Nguyen, D. Q. Nguyen, V. Nguyen, T. Le, Q. H. Tran, and D. Phung, “Regvd: Revisiting graph neural networks for vulnerability detection,” in ICSE-Companion. ACM/IEEE, 2022, pp. 178–182.