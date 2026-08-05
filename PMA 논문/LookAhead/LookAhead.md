# L<sub>oo</sub>kAh<sub>ea</sub>d<sub>:</sub> P<sub>reven</sub>ti<sub>ng</sub> D<sub>e</sub>Fi At<sub>ac</sub>k<sub>s v</sub>i<sub>a</sub> U<sub>nve</sub>ili<sub>ng</sub> Ad<sub>versar</sub>i<sub>a</sub>l C<sub>on</sub>t<sub>rac</sub>t<sub>s</sub>

<sup>SHOUPENG REN</sup>, Zhejiang University, China

<sup>LIPENG HE</sup>, University of Waterloo, Canada

<sup>TIANYU</sup> <sup>TU</sup>, Zhejiang University, China

<sup>DI</sup> <sup>WU</sup>, Zhejiang University, China

J<sup>IAN</sup> <sup>LIU∗</sup>, Zhejiang University, China

<sup>KUI</sup> <sup>REN</sup>, Zhejiang University, China

<sup>CHUN</sup> <sup>CHEN</sup>, Zhejiang University, China

The exploitation of smart contract vulnerabilities in Decentralized Finance (DeFi) has resulted in financial losses exceeding 3 billion US dollars. Existing defense mechanisms primarily focus on detecting and reacting to adversarial transactions executed by attackers that target victim contracts. However, with the emergence of private transaction pools where transactions are sent directly to miners without first appearing in public mempools, current detection tools face significant challenges in identifying attack activities efectively.

Based on the fact that most attack logic rely on deploying intermediate smart contracts as supporting components to the exploitation of victim contracts, novel detection methods have been proposed that focus on identifying these adversarial contracts instead of adversarial transactions. However, previous state-of-the-art approaches in this direction have failed to produce results satisfactory enough for real-world deployment. In this paper, we propose LookAhead, a new framework for detecting DeFi attacks via unveiling adversarial contracts. LookAhead leverages common attack patterns, code semantics and intrinsic characteristics found in adversarial smart contracts to train Machine Learning (ML)-based classifiers that can efectively distinguish adversarial contracts from benign ones and make timely predictions of diferent types of potential attacks. Experiments on our labeled datasets show that LookAhead achieves an F1-score as high as 0.8966, which represents an improvement of over 44.4% compared to the previous state-of-the-art solution, with a False Positive Rate (FPR) at only 0.16%.

CCS Concepts: • Security and privacy → Intrusion detection systems.

Additional Key Words and Phrases: Blockchain, DeFi, Smart contract, Malware detection

## ACM Reference Format:

Shoupeng Ren, Lipeng He, Tianyu Tu, Di Wu, Jian Liu, Kui Ren, and Chun Chen. 2025. LookAhead: Preventing DeFi Attacks via Unveiling Adversarial Contracts. Proc. ACM Softw. Eng. 2, FSE, Article FSE083 (July 2025), 23 pages. https://doi.org/10.1145/3729353

## 1 I<sub>n</sub>t<sub>ro</sub>d<sub>uc</sub>ti<sub>on</sub>

Decentralized Finance (DeFi) has gained significant traction within the blockchain ecosystem over the recent years, stimulating the rise of a diverse array of DeFi applications such as: decentralized

\*Jian Liu is the corresponding author. Authors’ Contact Information: Shoupeng Ren, Zhejiang University, Hangzhou, China, spren@zju.edu.cn; Lipeng He, University of Waterloo, Waterloo, Canada, lipeng.he@uwaterloo.ca; Tianyu Tu, Zhejiang University, Hangzhou, China, tianyutu@zju.edu.cn; Di Wu, Zhejiang University, Hangzhou, China, wu.di@zju.edu.cn; Jian Liu, Zhejiang University, Hangzhou, China, jian.liu@zju.edu.cn; Kui Ren, Zhejiang University, Hangzhou, China, kuiren@zju.edu.cn; Chun Chen, Zhejiang University, Hangzhou, China, chenc@zju.edu.cn.

exchanges (DEXs) [31, 46] and lending platforms [1, 52]. The total value locked (TVL) in these DeFi protocols has exceeded \$100 billion in mid 2024 [30]. Among the various blockchains, Ethereum [60] and Ethereum Virtual Machine (EVM)-compatible chains [3, 7, 22, 24] have formed a particularly thriving DeFi ecosystem. At the heart of Decentralized Finance are smart contracts, which are self-executing programs deployed on blockchains, enabling trustless and transparent financial interactions without the need for intermediaries. The functionalities of smart contracts are invoked by users through transactions, after which the blockchain consensus algorithms ensure predefined actions and state transitions are execute correctly and autonomously on-chain. However, just like traditional software, smart contracts are fundamentally code-based programs, which are susceptible to vulnerabilities and various other security risks. In fact, DeFi incidents resulting from smart contract attacks have caused over 3 billion US dollars in financial losses [11, 20, 66].

DeFi Attacks. Targeting smart contracts, there are mainly two types of attack patterns commonly used by DeFi attackers: an attacker can either directly invoke specific public functions of the victim contracts in order to exploit a vulnerability [5, 28], or they can deploy an adversarial contract containing the complete attack logic onto the blockchain first, then initiate the attack through the intermediate contract by calling their entry functions afterwards [4, 8, 23, 27].

Detection Based on Adversarial Transactions. Most existing real-time defense mechanisms leverage heuristics [50, 62, 64, 67] and Machine Learning (ML) [38, 56] techniques to detect and respond (e.g. via front-running) to yet-to-be-confirmed attack transactions in public mempools. The main issue with this transaction-based detection approach is that it faces significant challenges in applying to private adversarial transactions. Namely, attackers can utilize private mempool services to send transactions directly to miners, evading visibility from other participants of the blockchain network before it’s confirmed and efectively bypassing attack detection and avoid any preventive actions from taking place. Based on our empirical study of the historical DeFi incidents over the past few years, we found that out of the 142 DeFi attacks on Ethereum collected by us, 80 cases (56.3%) involve transactions using private mempool services. Moreover, the proportion of adversarial transactions sent using private services is observed to exhibit a significant upward trend, rising from 44.4% in the first half of 2022 to a noticeably higher 77.8% in the latter half of 2023.

Detection Based on Adversarial Contracts. The ability for an attacker to execute private adversarial transactions severely undermines the detection capabilities of existing transactionbased solutions, rendering them largely inefective. In contrast, some works have redirected their focus towards developing detection strategies based on identifying adversarial contracts. As an example, Forta [16] applies NLP techniques and uses a simple logistic regression model to analyze contract bytecode for malicious intents, but it fails to achieve satisfactory performance (Table 1). Beyond Forta, the only comparable work is BlockWatchDog [63], which only support detecting reentrancy attacks. We emphasize the critical importance of designing an efective solution for detecting a wide range of DeFi attacks that can remain functional even when private mempool services are employed by the attackers.

Our method. In this paper, we propose the LookAhead system, a new framework for detecting DeFi attacks via unveiling adversarial contracts. It achieves high detection efectiveness through systematic feature design and selection, the construction of a comprehensive contract dataset, and a combination of advanced classifier models. In §3.2, we describe in detail the threat model assumptions and the scope of detection followed by our proposed system.

We make an empirical observation that adversarial contracts used in DeFi attacks often contain similar patterns (> 70% of the adversarial contracts use funds from anonymous sources for deployment; > 98% are closed-sourced, etc.) We also note that depending on the vulnerabilities targeted, attackers write contract code in specific manners. For instance, flashloan attacks rely on complex call chains and the implementation of attack logic within callback functions. To use these behaviours to our advantage for identifying adversarial contracts, we using the stacking technique to design three ML models (transformer, classifier, and a meta classifier) that, when used together, can understand the intrinsic characteristics of malicious contracts based on code semantics and common patterns. To support the models in understanding the behaviour of Solidity code, we develop a specialized tokenization method called Pruned Semantic-Control Flow Tokenization (PSCFT) that is integrated directly into a contract bytecode decompiler.

An important observation made by [66] is that most attacks are not executed atomically within the constructor during adversarial contract deployment, which provides a rescue time frame (after an adversarial contract has been deployed and before the attacker execute the attack logic via the contract) for defenders and victims. We use $t x _ { d e p l o y }$ to denote a contract deployment transaction, and $t x _ { f i r s t }$ to represent the transaction in which the attacker initiates the actual attack via the adversarial contract. In our dataset, over 60% of attacks exhibit a diference between the confirmation time of $t x _ { d e p l o y }$ and $t x _ { f i r s t }$ that satisfies: $t _ { f i r s t } - t _ { d e p l o y } \geq 1 0 0 s$ . By designing an eficient feature extraction pipeline and leveraging modern ML architectures, we can produce predictions for newly created contracts on-chain in a timely manner with amortized $t ^ { p r e d } \ll 1 0 0 s$ . The result of our design is LookAhead, a system that is able to efectively distinguish adversarial contracts from benign ones, and make just-in-time predictions $( t _ { d e p l o y } + t ^ { p r e d } < t _ { f i r s t } )$ of potential attacks.

To ensure the performance and generality of our ML models, we hand-picked then carefully reviewed and identified 375 adversarial contracts used in DeFi incidents between April 2020 and July 2024 from both the Ethereum and Binance Smart Chain (BSC) to form our adversarial contract dataset. And in total 796,437 contracts deployed on Ethereum from June 2022 to June 2024 were collected, with 210, 643 of them being used as benign samples after filtration. We extract a selection of features from the contracts capturing patterns exhibited during both their implementation and their deployment stages. Given that our dataset contains a combination of contracts from both recent and historical attacks, we believe that our feature selection will remain reasonably relevant and efective even in the rapid growing landscape of decentralized finance.

We build our classifiers using multiple supervised ML algorithms and conduct a comprehensive performance evaluation. Experiment results on our labeled datasets show that our method for identifying adversarial contracts performs exceptionally well with an F1-Score produced by the KNN meta classifier reaching as high as 0.8966, and a false positive rate of only 0.16%, which represents an improvement of over 44.4% compared to the previous state-of-the-art solution Forta. Contributions. We summarize our contributions as follows:

• We highlight the limitations in existing transaction and contract-based detection methods and propose a new framework LookAhead that achieves significantly better performance;

• We design a smart contract bytecode lifting and analysis pipeline with an integrated protocol to extract features and generate PSCFT for Natural Language Processing (NLP) training;

• We construct the first large-scale comprehensive dataset of adversarial (manually-labelled) and benign (methodically selected) smart contracts consisting of a set of useful contract features based on extensive empirical observations with statistical data support;

• We build and evaluate an ML-based system consisting of two open-weights classifiers and one meta classifier model, the results are compared with previous state-of-the-art works to substantiate our system’s efectiveness and practicality.

## 2 B<sub>ac</sub>k<sub>groun</sub>d

## 2.1 Ethereum & Decentralized Finance (DeFi)

Blockchain is a decentralized ledger maintained over a peer-to-peer network via consensus mechanisms. Ethereum [60] is one of the most used blockchain platforms. It was the first to introduce the

Ethereum Virtual Machine (EVM) that support smart contracts, and has inspired the development of a series of EVM-compatible chains. DeFi is a blockchain-based financial ecosystem powered by smart contracts to ofer financial services in a more open and transparent manner.

Accounts. Accounts are entities that can hold the Ether token and initiate transactions. Ethereum has two types of accounts: External Owned Accounts (EOAs) and Contract Accounts (CAs). EOAs are controlled by users holding private keys, while CAs are containers of contract code and storage.

Transactions. Transactions are used to transfer Ether, invoke contracts, or create contracts. After a transaction is sent, it is first placed in the mempool of a blockchain node, waiting to be selected and finalized by block producers.

Smart Contracts. Smart contracts are instantiated objects stored on the blockchain. They are typically written in languages like Solidity [35] and compiled into bytecode that can be executed within the EVM, a stack-based virtual machine supporting Turing-complete instructions.

Tokens. Tokens are cryptocurrencies created using smart contracts. They come in two main types: fungible tokens and non-fungible tokens (NFT). Furthermore, Ether, used as the reward for anyone contributing in the consensus, is referred to as the native token.

Decentralized Exchange (DEX). Unlike centralized exchanges (CEX), DEX doesn’t require users to deposit digital assets into the exchange for trading. Instead, it conducts asset transfers directly on the blockchain via smart contracts, enabling transparency, security, and decentralization.

Flashloan. Flashloan is a unique form of lending that leverages the atomic nature of blockchain transactions to allow users to borrow and repay cryptocurrency within a single transaction. This mechanism lets users temporarily possess large amounts of tokens for a small fee. Despite of its convenience, it has been exploited in numerous DeFi attacks [32, 51, 57].

## 2.2 Maximal Extractable Value (MEV)

Traditionally, miners determine the order of transactions in a block based on gas prices. Since the mempool is publicly accessible, users have the ability to influence the placement of their transactions by adjusting the gas price, allowing for front-running [37, 54]. The profit gained from such manipulation, beyond standard rewards and fees, is known as Maximal Extractable Value [34].

Front-running. Front-running is a fundamental means used to extract MEV, where bots monitor the public mempool for target transactions and raise gas prices to execute their own first. It has also been used in transaction-based intrusion prevention systems [49, 65] to front-run adversarial transactions, thereby preventing DeFi attacks.

Private Mempool Services. In response to MEV, Flashbots’ MEV-auction has been widely adopted [15, 59]. This solution provides private channels for transactions to be submitted directly from the users to block producers without being broadcasted to the network, ensuring that transactions remain private until they are finalized, preventing them from being monitored or intercepted by others. However, these services have increasingly been abused by attackers to conceal adversarial transactions, efectively evading mainstream transaction-based detection methods.

## 3 Overview of LookAhead

## 3<sub>.</sub>1 Motivatin<sub>g</sub> Exam<sub>p</sub>le

On July 21, 2023, DeFi protocol Conic Finance sufered a major exploit [36] causing financial losses exceeding \$3 million. We present an overview of the attack process in Figure 1. Notably, the adversary transaction that triggered the attack was labeled as an MEV Transaction on Etherscan <sup>1</sup>, indicating the use of private mempool services. This allowed the attacker to bypass the public mempool and invalidate any potential front-running defense attempts, highlighting the limitations of transaction-based prevention mechanisms that rely on detecting adversarial transactions.

![](images/8d6aa34d8b0fe922b053eda9c32cc3e94d64adf14bd011af40ba02c5ef4001c8.jpg)  
Fi<sub>g</sub>. 1. Read-onl<sub>y</sub> reentranc<sub>y</sub> atack on Conic Finance<sub>,</sub> usin<sub>g</sub> a <sub>p</sub>rivate adversarial transaction to evade d<sub>e</sub>t<sub>ec</sub>ti<sub>on, cou</sub>ld h<sub>ave</sub> b<sub>een preven</sub>t<sub>e</sub>d d<sub>ur</sub>i<sub>ng</sub> th<sub>e rescue</sub> ti<sub>me</sub> f<sub>rame.</sub>

3.1.1 Rescue Possibility. Despite the attacker using private mempool services to conceal the adversarial transaction, there was still an opportunity to prevent the attack, based on the following facts: 1) The ConicEthPool contract’s address — the vulnerable target — was written in plaintext in the adversarial contract at deployment; 2) There was a 36-second window between the deployment of the adversarial contract and the execution of the adversarial transaction; 3) The ConicEthPool contract had a shutdownPool function, which ofers the ability to perform an emergency pause to block the adversary’s attempt to call the deposit function. Thus, detecting the adversarial contract at the deployment stage, before the confirmation of the adversarial transaction, could have prevented the exploit, regardless of whether the attacker used a private mempool service or not.

3.1.2 Adversarial Paterns. While DeFi attack logic can vary based on the specific vulnerabilities being exploited, adversarial contracts often exhibit consistent behaviours due to the nature of their malicious intent and their profit motives. Our analysis of numerous adversarial contracts reveals a clustered set of patterns including the use of anonymous fund sources, closed-source code, and frequent token-related function calls. Additionally, adversarial contracts often have distinctive interaction logic with victim contracts, which difers significantly from the patterns observed in normal contracts, resulting in easily distinguishable code structures and external call flows. Based on these observations, ML techniques can be employed to efectively identify adversarial contracts (regardless of the specific attack type) by learning and predicting these common patterns.

![](images/1e19b333c305d9b927a613fb688d6d7a5a7af6c354b4f323bee993c2294d3bef.jpg)  
Fi<sub>g</sub>. 2. a) Left: atack directl<sub>y</sub>, b) Ri<sub>g</sub>ht: atack throu<sub>g</sub>h adversarial contracts.

## 3<sub>.</sub>2 Th<sub>rea</sub>t M<sub>o</sub>d<sub>e</sub>l

3.2.1 DeFi Atacks. There are many types of security breaches with complicated root causes, potentially leading to attacks on victims. In various scenarios, an adversary could use distinct techniques for exploitation. In this section, we present these attack methods, define key concepts involved in the attack process, and restrict LookAhead’s detection scope to a certain area of focus. Attack Methods. The attack process of two common attack methods are depicted in Figure 2.

1) Attacking directly via adversarial transactions. In the presence of complex attacking logic involving multiple transactions, the attackers often bear the risk of encountering potential half-wayfailure. A half-way failure arises when only part of an attack was executed successfully and the rest fail to proceed due to various reasons (e.g., transaction conflicts).

2) Attacking through adversarial contracts. This pattern allows the attacker to execute complex attacking logic in a single transaction, hence avoid half-way failures thanks to the atomic nature of a blockchain transaction. Using this method, an attacker first deploys a smart contract on-chain containing the core attack logic, then initiate the adversarial transaction by calling the entry point function of the adversarial contract. Most DeFi attacks follow this pattern, hence we focus on addressing this type of attacks.

Scope of Detection. We specify two attack scenarios that fall outside of the scope this study. 1) Attacks that do not require contract deployments. Those resulting from private key leakage or attackers directly invoking public functions of the victim contracts using EOAs are outside the scope of our detection. 2) A small portion of attacks are completed directly through the constructor function at the time of contract deployment. Since the adversarial contracts used in these attacks already exhibit execution traces of the actual attack at the time of deployment, we consider them as adversarial transactions, which fall outside the scope of our detection.

LookAhead focuses on unveiling attacks that intend to cause financial loss for DeFi protocols by exploiting vulnerabilities in on-chain smart contracts. To identify an attack, our solution detects adversarial contracts that contain the core attack logic. Other contracts, such as those used solely for concealing profits, fake token contracts that facilitate the main attack, and counterattack contracts used by front-running bots will not be included for our analysis.

3.2.2 Adversaries. We consider a resource bounded adversary A that is financially rational, capable ofdeploying an arbitrary amount ofsmart contracts and execute both public and private transactions on EVM-compatible blockchains.

Attacker Assumptions. Based on previous research findings [66] (IEEE S&P’23), our study makes the following assumption about A:

1) A does not initiate attacks by batching $t x _ { d e p l o y }$ and $t x _ { f i r s t }$ together in an atomic transaction during contract deployment, meaning that A splits an attack into two stages following the second pattern described in Figure 2;

![](images/76a1e1094e2bb69ce25a7e66d69090896f5e72b4fe4032db5e8772b79d4248d7.jpg)  
Fi<sub>g.</sub> 3<sub>.</sub> An <sub>o</sub>v<sub>e</sub>rvi<sub>e</sub>w <sub>o</sub>f th<sub>e</sub> LookAhead <sub>sys</sub>t<sub>e</sub>m<sub>.</sub>

In a prior research [48], it was observed that 79.49% of attacks in their dataset exposed the victim contract address before the adversarial transaction. Using the same method, we found 202 attacks (53.9%) with similar exposure behaviour in our dataset of 375 adversarial contracts. Hence, we also consider a weaker adversary A<sup>′</sup> satisfying one more assumption than A:

2) A<sup>′</sup> hard-codes exploitation targets’ addresses in their adversarial contracts whenever possible, and the victim’s address will emerge before the first adversarial transaction $t x _ { f i r s t }$

3.2.3 Victim ofAtacks. We consider those contracts which contain vulnerabilities that have been exploited by attackers and sufered a financial loss the victims of a DeFi attack.

3.2.4 Rescue Time Frame. Our threat model leaves a short rescue time frame for detection and prevention to take place by the defenders and the targets, and enables the possibility of on-time reaction by victims of ongoing attacks. The rescue window is defined by $t ^ { w i n d o w } = t _ { f i r s t } - t _ { d e p l o y } -$ �<sup>����</sup> , it has a typical value observed in our dataset of $t ^ { w i n d o w } \leq 1 0 0$ . Within the rescue time frame several counterattack actions could be taken, we describe those in more details in §3.3.

## 3.3 S<sub>y</sub>stem Model

In order for the LookAhead system to provide just-in-time identification of adversarial contracts as they are deployed onto the blockchain, a streamlined process consisting of a series of components is used. We show an overview of our system in Figure 3.

Chain Monitoring. Through a blockchain network node or third-party node monitoring services, we monitor on-chain data in real-time, and acquire the deployed contracts and their basic information such as deployer, input data, gas used, bytecode and so on whenever needed. Via chain explorers such as Etherscan, we are able to retrieve the verification status of contracts as well. These data will be combined and fed into a classifier as features for ML inference at a later stage.

Binary Lifting. We do not use any source code information from the contracts, but only the bytecode deployed on the blockchain to ensure our model is capable of analyzing any smart contract on EVM blockchains. To do that, LookAhead uses the binary lifting and analysis library Gigahorse [39] to decompile the contract bytecode and generate intermeidate representation (IR).

PSCFT and Feature Extraction. With the raw IR, we use a specialized feature extraction program to pull out features including but not limited to: token-related call information, flashloan usage information, etc. Finally, we convert the IR into a condensed text form (PSCFT) and populate it with semantic information such as function names and external call target address labels.

Classifier Inference. Input features are properly tokenized and formatted before they run through our trained classifiers. These ML models are made up of a transformer, an optimal candidate classifier, and a meta classifier trained using supervised machine learning methods for binary classification. They are able to output either the classification result $l a b e l _ { f r e d } \in \{ 0 , 1 \}$ indicating whether the contract is adversarial (1) or not (0), or they can produce a confidence score $0 \% \leq$ $p _ { { p r e d } } \leq 1 0 0 \%$ representing the possibility of the input contract being adversarial.

Defense Mechanisms. After detecting an ongoing attack, LookAhead enables several potential defense mechanisms for defenders and victim contract owners: 1) To defend against ${ \mathcal { A } } ^ { \prime } { } _ { \mathrm { : } }$ , notifications are sent to targets, allowing them to take defensive actions, such as executing an emergency pause. If contract owners grant admin permissions to LookAhead, the system can autonomously trigger shutdown methods on behalf of them. 2) To mitigate against A, we propose a blacklist service for DeFi protocols, enabling them to block transactions from flagged adversarial contracts.

## 4 D<sub>a</sub>t<sub>ase</sub>t B<sub>u</sub>ildi<sub>ng</sub>

## 4<sub>.</sub>1 C<sub>on</sub>t<sub>rac</sub>t C<sub>o</sub>ll<sub>ec</sub>ti<sub>on</sub>

To perform supervised training of our classifier models, we collect a large set of smart contracts from the Ethereum and BSC. For the positive (adversarial) dataset, we curated a collection of adversarial contracts used in real-world DeFi incidents. For the negative (benign) dataset, we follow a methodical process to collect contract samples that are very unlikely to be related to any attacks.

4.1.1 Adversarial Contracts. To collect adversarial contracts, we gather information on DeFi incidents from various publicly available sources, such as GitHub repositories documenting DeFi incidents [11, 20], and alerts from security companies [6, 9, 26]. We conduct a meticulous review of these incidents to identify the contracts used by attackers. Between April 2020 and July 2024, we collected 506 DeFi attacks. In these, 29 attacks (5.7%) involved direct attacks from EOAs without using contracts, and 37 attacks (7.3%) were executed directly within the contract constructor. Both cases fall outside the scope of our detection. While attacks spanned multiple blockchains, the majority were on Ethereum and BSC. To simplify the study, we exclude 65 incidents happened on other chains. This left us with 375 adversarial contracts: 142 from Ethereum and 233 from BSC. Following prior research [66], we classify the attacks into three categories: 1) Untrusted or unsafe calls (13.3%), 2) Access control mistakes (14.4%), and 3) Coding mistake (72.3%). The first category includes issues like reentrancy, the second covers access control flaws, and the third involves coding mistakes such as arithmetic errors and logic absences.

4.1.2 Benign Contracts. As contract-based DeFi attack detection approach is still a largely underexplored research direction, to date, there is no publicly available dataset that provides a comprehensive and representative set of ground truth benign contracts.

Consequently, we need to build our benign dataset from scratch. Given the vast number and variety of smart contracts deployed on the blockchain, constructing a large-scale dataset with suficient sample diversity and label accuracy requires extensive manual review eforts and is prohibitively time-consuming. In response to this, previous ML-based work Forta [16] follow a heuristic approach for collecting benign samples.

Previous Approach. Forta assumes that all contracts verified on the chain explorer are benign. Despite being an intuitive design choice, analysis of our attack samples reveals that a small proportion of adversarial contracts are nonetheless verified. This may be because verification is an inexpensive task (often automated by development toolkits) and it has negligible impact on attack efectiveness, as existing detection methods do not focus on identifying adversarial contracts. Consequently, a more robust collection method is needed.

Key Observations. A smart contract can be deployed either 1) through a normal transaction, where its to\_address field is NULL with bytecode included in the input\_data field, or 2) via an internal transaction (also known as a message call) initiated by an existing contract during its execution. Between June 2022 and June 2024, the number of contracts created via internal transactions is significantly higher than those created via normal transactions (approximately 20 times more). A prior research [41] that analyzed 1.16 million contracts created via internal transactions found that these contracts were generated by only 9, 228 creator contracts, indicating that a small number of contracts were responsible for deploying a vast number of contract instances. Most of these instances originate from factory contracts, e.g., Uniswap’s factory contract <sup>2</sup>, which are designed to repeatedly create similar contracts. Additionally, we observe that all adversarial samples in our dataset are deployed through normal transactions. These contracts are typically invoked by a very limited number of unique addresses. Specifically, 355 of them (94.7%) interacting with only a single address, namely the attacker’s account, and a maximum of 6 unique users. This low interaction count is consistent with the behavior of adversarial contracts, which typically feature access control and are designed for single-use only to attack specific victims.

Our Method. Based on the observations, we construct our benign dataset following two key principles to ensure evaluation fairness, as well as dataset diversity and accuracy.

• To ensure fair evaluation, we focus solely on contracts created via normal transactions, consistent with the deployment patterns of all adversarial samples, while avoiding internal transactions that could drastically inflate the dataset with numerous near-identical instances.

• We select contracts with over a defined number of unique interacting addresses as an indicator of their benign nature, aiming to ensure reliability while maintaining sample diversity.

We first query Google’s BigQuery service to collect contracts deployed via normal transactions between June 2022 and June 2024, identifying 796, 437 contracts. Then we analyzed their transaction history, focusing on the number of unique interacting addresses from June 2022 to August 2024, and retained contracts with at least 10 unique interacting addresses, resulting in a final benign dataset of 210, 643 contracts. While no heuristic method can fully eliminate potential bias, our approach ofers a pragmatic trade-of between dataset scale, diversity, and correctness.

## 4<sub>.</sub>2 F<sub>ea</sub>t<sub>ure</sub> S<sub>e</sub>l<sub>ec</sub>ti<sub>on</sub>

Based on empirical observations, we construct a selection of features for use in LookAhead’s ML model training and inference. The set consists of features spanning both the implementation and the deployment stages of a DeFi attack, aiming to capture any potential unexpected patterns of behaviour exhibited by attackers. In this section, we explain our rationale behind the choice of features and provide supporting statistical data whenever applicable.

4.2.1 Deployment Features. A typical workflow for deploying a contract to a blockchain consists of 1) Sourcing the fund to cover deployment costs, 2) Sending a transaction containing the contract bytecode and other related data on-chain, 3) Verifying the source code ofthe contract to demonstrate its security and trustworthiness to the wider community. During the process, a malicious individual often behave very diferently from a normal one, we use this to our advantage by selecting several measurable properties as supporting factors to help us make such a distinction.

Creator Attributes: {�����, ����\_������}. ����� indicates number of transactions sent from the contract creator before the deployment of the contract. A higher ����� suggest a more active EOA address. We are also interested in the ������ from which the contract creator initially obtained their funds. To conceal their identities, attackers typically fund their attacks from a mixer (e.g,

Tornado Cash [58]) or an instant exchange [10, 25] without sign-up or KYC (Know Your Customer) requirements, then transfer the funds to a new account for subsequent transactions. In contrast, regular users are more likely to source their funds from KYC-required centralized exchanges.

We categorize fund sources into four types based on address information:

1) Safe: Fund sources with relatively secure origins, such as KYC-required centralized exchanges, which are less likely to be used by attackers.

2) Anonymous: Fund sources with anonymous origins, including mixers and some KYC-free exchanges, which are more susceptible to be used for malicious purposes.

3) Bridge: Funds originating from cross-chain transfers. The source could be anonymous, hence they are considered potentially usable for attacks.

4) Unknown: Fallback label for those that fail to fall into the above categories.

Statistical Data: In benign samples, we observe that more than 75% of them have Safe sources, and approximately 10% of them are funded through Anonymous sources. In contrast, more than 70% of the adversarial contracts have Anonymous sources, with only about 10% having Safe labels.

Transaction Data: {�����, �����\_����\_�����ℎ, ���\_����}. Depending on the contract’s design, the contract creator can transfer native tokens to the contract upon deployment. Adversaries typically do not perform such a transfer. To avoid the potential discrepancy across Ethereum and BSC, we binarize the value attribute to indicate whether such a transfer occurred (true or false), rather than using the exact amount. The input data field of a deployment transaction holds the contract’s bytecode, we measure its length to account for the size of the contract. Additionally, we use ���\_���� to measure the computational resources consumed during deployment, which depends on the bytecode complexity and EVM execution, rather than gas fees, which relies not only on ���\_���� but also on the dynamically fluctuating gas price influenced by network congestion.

Statistical Data: In our dataset, about 80% of benign contracts have input data exceeding 15,000 bytes, while over 50% of adversarial contracts fall below this threshold. To eliminate potential EVM-induced diferences in ���\_���� between Ethereum and BSC, we simulate the deployment of all BSC adversarial samples on Ethereum using identical bytecode to obtain a consistent gas\_used value. However, 24% of the contracts failed due to interactions with contracts absent on Ethereum. Among the successfully samples, 94% showed less than a 1% deviation compared to original value. Given this negligible diference, for the failed samples, we directly use their original gas\_used value.

Verification Status: {��������}. The �������� status can be granted to contracts if their source code and compiler settings are submitted to platforms like Etherscan. Verified contracts are opensource, anyone can view their code. Many contract development frameworks, such as Foundry [17], ofer options to verify contracts upon deployment. It has become increasingly common for developers to verify their contracts to demonstrate security and trustworthiness. Attackers, on the other hand, typically do not have the incentives to do so since it would expose their malicious intentions.

Statistical Data: We observe in our dataset that only 6 adversarial contracts (1.6%) are verified , while more than 85% of benign contracts are verified.

4.2.2 Implementation Features. Based on empirical observations, some of the characteristics that can be used to determine whether a contract is adversarial or not include: implemented functions, internal and external function calls. To capture other hidden or intrinsic properties of a contract, we also design a special technique called Pruned Semantic-Control Flow Tokenization (PSCFT) that provides a condensed representation of the behaviour of smart contracts.

Implemented Functions: {� ���\_�����, � ���ℎ����\_��������\_����� }. Since adversarial contracts typically do not involve complex business logic, they often implement fewer functions. We also calculate the number of flashloan callback functions and the proportion they take up over all public functions. This choice is motivated by the fact that many attack scenarios require attackers to have significant token holdings, which most adversaries do not naturally possess. Consequently, they resort to utilizing flashloan to temporarily borrow a substantial amount of tokens and execute attack logic within corresponding callback functions. Since the size of lending pools are limited, attackers may borrow tokens from multiple sources, resulting in multiple flashloan callbacks.

![](images/2d72e953124aef82966dfebd8a5c29bdd2518c052c96ecc4e16be3175ac02c0e.jpg)  
Fi<sub>g</sub>. 4. Side-b<sub>y</sub>-side com<sub>p</sub>arison of raw IR and PSCFT.

Statistical Data: We can observe from the dataset that over 75% of benign contracts incorporate at least 20 public functions, whereas more than 75% of adversarial contracts implement 10 or fewer. Additionally, over 60% of adversarial contracts include flashloan callbacks, while more than 99% benign contracts involve no such callbacks.

Function Calls: {�����\_����\_�����,���\_�����\_����\_�����, ���\_�����\_����\_�����, ��������\_� ���\_�����, ������������\_�����}. We take into consideration external calls related to tokens, includ ing their count and proportion. Generally, the objective of DeFi attacks is to gain financial benefits, i.e., acquire valuable tokens. Therefore, external calls within adversarial contracts often involve token-related functions. The token-related functions we selected include, but are not limited to, common token standard [12, 13] functions and popular DEX [31] functions.

Furthermore, we introduce two additional features to assess the behaviour of public functions in a smart contract. ���\_�����\_����\_����� is designed to capture the maximum number of tokenrelated interactions within the execution flow of any public function of a contract, which helps us identify contracts with exceptionally high token interactions. Moreover, ���\_�����\_����\_����� is also measured to demonstrate the average number of token-related calls per public function.

We also include the number of delegatecall instructions, as these are often implemented in proxy contracts. Additionally, some attackers have a tendency to self-destruct their contracts after completing the attack, so we also consider the presence of the selfdestruct instruction.

Statistical Data: It can be observed from our dataset that over 90% of benign contracts do not have token-related calls. In contrast, almost all adversarial contracts involve token-related calls, with over 70% of them having 10 or more.

Code Semantics: {�����}. To understand the behaviours exhibited by DeFi attack adversaries and capture features that are potentially hidden from our empirical observations, we design a new representation for smart contracts called Pruned Semantic-Control Flow Tokenization (����� ).

In smart contracts, functionalities (including potential attack logic) are implemented through functions, and interactions with external entities, such as invoking the functions implemented in other deployed contracts, are performed via message calls. For adversarial contracts, such interactions (e.g., with victim contracts) are indispensable. Therefore, each function, along with its message calls and control flows, is capable of capturing the intrinsic behavioral characteristics of a contract, providing key information for determining whether the contract is adversarial or benign.

![](images/7e5ed4e8a898d0617c8588a871014abbb01c174a6278d3e4bca4b74cc4b8a540.jpg)  
Fi<sub>g</sub>. 5. LookAhead’s s<sub>y</sub>stem buildin<sub>g</sub> workflow

To extract these critical features, we use Gigahorse [39] to lift the contract bytecode into an intermediate representation (IR), followed by a pruning process that remove extraneous elements while reconstructing control flows. To provide richer contextual information, the refined IR is fed into an augmentation pipeline that retrieves and replaces function signatures and external addresses with appropriate semantic labels. The resulting output is the fully PSCFT, which efectively represents a textual summary of function canonical names, control-flow graphs and external call chains. Figure 4 presents an excerpt comparing the raw IR of a contract with the corresponding PSCFT. We provide the detailed construction process of PSCFT in §5.2.

## 5 M<sub>e</sub>th<sub>o</sub>d<sub>o</sub>l<sub>ogy</sub>

In this section, we present our methodology for building the LookAhead system that performs adversarial contract classification. An overview of the complete framework is depicted in Figure 5.

## 5<sub>.</sub>1 E<sub>x</sub>t<sub>erna</sub>l D<sub>a</sub>t<sub>a</sub> R<sub>e</sub>t<sub>r</sub>i<sub>eva</sub>l

Some of the features used in our dataset are obtained from external sources, we identify related third-party services in this section. Based the data, we use a algorithm to determine fund source for the contracts, enabling subsequent training and inference of our machine learning models.

Transaction Data. In this study, we focus on investigating and demonstrating the viability and efectiveness of the LookAhead system. Therefore, for the sake of simplicity, instead of running our own blockchain nodes, we obtain transaction raw data from an RPC (Remote Procedure Call) provider such as Alchemy [2] that relays our requests directly to other nodes in the blockchain network. Prior to the practical deployment of LookAhead, we recommend setting up dedicated nodes for minimized data processing delays. We store and format contract deployment transactions and their associated attributes such as nonce, value, and gas used, along with the sender and input data properties for use in subsequent phases of our feature extraction pipeline.

Chain Explorer Data. We leverage the APIs provided by chain explorers, Etherscan and BscScan to retrieve information about the verification status of contracts. However, we cannot obtain the exact time for contract verification. The efectiveness of this feature is based on the assumption that developers verify their contracts upon deployment (with minimal to no delay). We describe the potential impact that this assumption could bring to the performance of LookAhead in $\ S 6 . 2 .$

Furthermore, we develop a fund source labeling algorithm to determine a contract’s fund source. We first construct an address label dataset using the address label cloud [18, 19] and the public name tags provided by chain explorers, following the definitions in §4.2.1. As an example, Tornado.Cash [29] is given the label of Anonymous. The algorithm traces fund flows from the contract deployer, recursively identifying the earliest funding source. If the source address has a known label, it is assigned; otherwise, tracing continues up to a predefined threshold depth. If no label is identified within this limit, the fund source of the contract is marked as Unknown.

## 5.2 Pruned Semantic-Control Flow Tokenization (PSCFT)

Based on EVM bytecode binary lifter framework Gigahorse [39], we design a pipeline for 1) Constructing intra-contract Control Flow Graphs (CFG) and external call chains based on Intermediate Representation (IR), 2) Aggregating and extracting dataset features from EVM bytecode, and 3) Generating textual summaries for the smart contracts in our dataset.

5.2.1 Implementation Feature Extraction. To enable the extraction of implementation features (see §4.2.2) simultaneously with the generation of the IR, we implemented a custom-designed Soufle Datalog program based on existing rules from Gigahorse. 1) First, by utilizing the Statement\_Opcode rule, which maps each statement to its corresponding opcode, we can directly extract the number of occurrences of critical opcodes, such as DELEGATECALL and SELFDESTRUCT, which corresponds to features ��������\_����\_����� and selfdestruct\_count. 2) Next, we leverage the FunctionEntry rule, which contains all function entry points (identifiers), with the help of PublicFunctionSelector, which maps public functions to their selectors, to determine the total number of public and private functions. This combination also helps identify flashloan callback functions. 3) Finally, to analyze the external function calls, we design the ExternalCallResolved rule, providing details about all resolved external calls made within the contract, including their selectors and signatures. With the help of ExternalCallResolved, we can keep track of token-related calls and their related information including their total count, maximum occurrences, and average usage.

5.2.2 CFG Construction and Pruning. Based on the IR of a contract, we construct a function-byfunction visualization $\mathcal { F } = \{ \mathcal { F } ^ { p r i v a t e } \cup \mathcal { F } ^ { p u b l i c } \}$ of the internal structure of contract methods, each consisted ofcontrol flow blocks and programming statements. Functions have 4-byte selector derived from the Keccak hash (SHA3) of their canonical names and can be either private or public. Each function contains multiple basic blocks (BB) chained together $( \{ F _ { i } = ( B B _ { 0 } ^ { i } , B B _ { 1 } ^ { i } , \cdot \cdot \cdot , B B _ { | F _ { i } | } ^ { i } ) \} )$ , and each block contains multiple statements. Blocks and statements are identifiable via unique numerical IDs, for example, 0�11�9. The relationship between two blocks can be discovered by following the predecessor and successor IDs associated with each block, each block can have multiple predecessors and successors, hence we can create a directed graph $G _ { i } = \left( B _ { i } , A _ { i } \right)$ such that the set of nodes $B _ { i } = \left\{ B B _ { j } ^ { i } : B B _ { j } ^ { i } \in F _ { i } \right\}$ where $F _ { i } \in { \mathcal { F } }$ , and $( B B _ { x } ^ { i } , B B _ { y } ^ { i } )$ is an arc in $A _ { i }$ if $B B _ { x } ^ { i }$ is a predecessor of $B B _ { y } ^ { i }$ and $B B _ { y } ^ { i }$ is a successor of $B B _ { x } ^ { i }$ . Statements could be constant/variable declarations, internal/external function calls, function terminations, memory copies, storage initialization, etc., we focus on constructing a CFG based on function call statements.

The pruning process starts with the sorting and reordering of functions based on function types (public first, then private) and their canonical names. Gigahorse can only recover the names for some of the public functions and external call methods, hence if the derivation of canonical names from function selectors failed, we will leave them as-is and recover their names in the semantic recovery stage. For any private function $F _ { j } ^ { p r i v a t e } \in \mathcal { F } ^ { p r i v a t e }$ , we give it a canonical name of ���������������� . Next, we remove all statements not related to function calls from every block $B B _ { j } ^ { i } \in B _ { i }$ . Then, we perform a depth-first search (DFS) over � to remove any block $B B _ { k } ^ { i }$ that contains no statements (i.e. $| B B _ { k } ^ { i } | = 0 )$ from $B _ { i } ,$ along with its incoming and outgoing arcs from $A _ { i }$ , and update the the predecessor and successor IDs for its neighbours. Finally, we assign canonical names ${ B B } _ { j } ^ { i } , j \in \left[ 0 , | { B } | - 1 \right]$ to basic blocks that are still left in $B _ { i }$

5.2.3 Semantic Recovery. For an external call statement like the one shown in §PSCFT and Feature Construction in Figure 5, we aim to recover: 1) An address label for 0x5777d92f208679db4b9778590 fa3cab3ac9e2168, 2) A human-readable name for the target method 0x490e6cbc.

Similar to our fund source labelling algorithm, we consider to match the target address with an address associated with a known label in chain explorers’ address label cloud and public name tags. For contracts that cannot be matched with an existing label, we check if they are open source, when they are, we use their contract name as the label. Contract developers might choose to store the target address in the EVM storage or as a variable/constant, in which case we use the same technique as [43] to run a Datalog program that finds the slot location storing the address value and retrieves the value from the target storage. If this process did not result in a matching label, we assign the address a default label UnknownTarget.

Function selectors are generated by hashing the function signatures with a secure hashing algorithm, and the hashing process is a one-way trap door. However, thanks to the deterministic nature of hash functions, third-party service such 4byte [14] are able to construct and maintain a large database of publicly known hash-to-name mappings. Therefore, in hopes to include as much function names and as little unknown functions in the PSCFT as possible, we attempt to extend Gigahorse’s name recovery ability with 4byte’s service by sending a request to their API upon failed recovery by Gigahorse. If the recovery still fails, we fallback to the default label UnknownFunc.

If the aforementioned process was completed successfully, the external call statement will be converted to UniswapV3.flash(...args). And we can produce a textual summary for smart contracts containing control-flow and semantic call information simply by concatenating $F _ { i } = ( B B _ { 0 } ^ { i } , \cdot \cdot \cdot , B B _ { | F _ { i } - 1 | } ^ { i } )$ $\in { \mathcal { F } }$ together in the form shown in Figure 4.

## 5<sub>.</sub>3 Cl<sub>ass</sub>ifi<sub>er</sub> D<sub>es</sub>i<sub>gn</sub>

5.3.1 Transformer. We train a transformer-based Natural Language Processing (NLP) model M<sup>�����</sup> <sup>�</sup> <sup>�����</sup> solely on the PSCFT constructed from the contracts in our dataset to learn the semantics and behaviours of the implementation of both benign and adversarial smart contracts. Following the standard architecture of transformers [55], we design:

1) A word2vec [47] embedding layer for positional encoding of PSCFT syntax and semantics,

2) Multiple encoder layers chained together, each made up of a multi-head attention layer followed by a feed-forward layer,

3) Pooling and dense layers for outputting classification results.

To ensure the model’s generalization ability and improve training eficiency, we implement regularization techniques such as early-stopping and dropouts to prevent model over-fitting. Overall, this transformer model provides a semantic view of smart contracts.

5.3.2 Ensemble Classifier. Aiming to embed the features selected based on our empirical observations into the semantic view of the transformer model, we design an ensemble classifier that merges the classifier models trained on empirical features with the transformer model trained on PSCFT together to produce a meta classifier model with better predictive performance.

Candidate Classifiers. Once all features have been extracted, we can proceed to train a supervised learning classifier using a variety of ML algorithms, outputting multiple candidate models for further evaluation. These models $\boldsymbol { \mathcal { M } } = \{ M _ { L R } ^ { c l a s s i f i e r } , M _ { D T } ^ { c l a s s i f i e r } , M _ { R F } ^ { c l a s s i f i e r } , M _ { X G B M } ^ { c l a s s i f i e r } \}$ include classical methods such as Logistic Regression and Decision Trees, ensemble learning methods such as bagging-based Random Forest and XGBoost which is based on boosting

Due to the limited quantity of adversarial samples, to mitigate the bias introduced by class imbalance, we employ the ADASYN oversampling method [42] that synthesizes additional samples to bring the number of adversarial samples closer to that of the benign ones in the training dataset.

For the fund source feature, which consists of categorical data (see §4.2.1) with low cardinality, we apply one-hot encoding. To facilitate better comparability among input values in numerical terms of varying dimensions, we standardize the dataset using Z-score normalization. This process brings each feature value around to the centre of 0 with a standard deviation of 1.

After training has completed for each of the candidate classifiers, we evaluate their performance using a function $f : { \mathcal { M } } \to \mathbb { R }$ that incorporates metrics such as Recall and F1-Score to determine an optimal model $M ^ { c l a s s i f i e r }$ such that $M ^ { c l a s s i f i e r } = \mathrm { a r g m a x } _ { M \in \mathcal { M } } f ( M )$ for the meta classifier training and inference in the next stage.

Meta Classifier. Using the stacked generalization (stacking) machine learning algorithm, we design a blender that stacks �<sup>�����</sup> <sup>�</sup> <sup>�����</sup> and �<sup>������</sup> <sup>�</sup> <sup>���</sup> together to form a higher level model �<sup>����</sup> known as a meta classifier. By feeding the predictions produced by the two base models into the meta model, we are able to 1) Combine the strengths of each model and automatically learn the optimal way to construct the inference result based on model significance, 2) Optimize the final classification boundaries to achieve more accurate detection of adversarial contracts.

Given that the meta classifier operates on low-dimensional inputs (only the two probability values between 0-1 outputted from the $M ^ { c l a s s i f i e r }$ and �<sup>�����</sup> <sup>�</sup> <sup>�����</sup>, indicating the likelihood of a contract being adversarial) and without incorporating any additional information about the contracts, employing a simple model is both suficient and efective. This not only ensures eficiency but also mitigates the risk of overfitting by limiting model complexity. Accordingly, we only consider simple models, including K-Nearest Neighbors (KNN), Logistic Regression (LR), Support Vector Machines (SVM), and Decision Trees (DT), and evaluate their performance to select the most suitable meta classifier in experiments.

## 6 E<sub>va</sub>l<sub>ua</sub>ti<sub>on</sub>

We implement LookAhead in Python using open source machine learning frameworks PyTorch and Tensorflow to achieve both CPU and GPU-accelerated training and inference of our classifier models. All experiments are conducted on a server running a 3.70GHz Intel(R) Xeon(R) processor with 32 threads and a NVIDIA A100 GPU with 40GB of VRAM.

## 6.1 Ex<sub>p</sub>eriment Setu<sub>p</sub>

6.1.1 Datasets. Token and proxy contracts are two of the primary types of contracts used on blockchains, while they are commonly used by developers, most adversaries do not rely on those types of contracts for their core attack logic. To identify contract types, we perform a simple check based on PSCFT to determine whether a contract follows known standards such as ERC20 and ERC1967. By analyzing whether the contracts match common standards, we found that in the dataset we built, 184, 386 benign samples were token contracts, 10, 037 were proxy contracts, and none of the adversary contracts belonged to these two categories. Therefore, we remove them from both our training and testing dataset the same way as it was in previous work Forta [16].

A distinguishing aspect of our research when compared to traditional supervised machine learning problems is the explicit chronological order of contract deployment. Contracts deployed over a diferent periods of time may exhibit varying characteristics and vulnerabilities, meaning that adversarial contracts may evolve and change over time as well. One of LookAhead’s objectives is to provide a ML classifier model that is pre-trained on past DeFi incidents but capable of detecting potential future attacks. Consequently, we sort the contract samples chronologically and use the earliest 80% of the contracts for training and reserve the latest 20% for testing. Out of the 80% of contracts used as training data, we take the last 25% of most recent samples as the training set for the meta classifier, and the rest for the transformer and candidate models.

T<sub>a</sub>bl<sub>e</sub> 1<sub>.</sub> P<sub>er</sub>f<sub>ormance</sub> b<sub>rea</sub>kd<sub>own o</sub>f dif<sub>eren</sub>t <sub>mo</sub>d<sub>e</sub>l<sub>s.</sub>

<table><tr><td></td><td>Model</td><td>Accuracy</td><td>Precision</td><td>Recall</td><td>F1-Score</td><td>FPR</td></tr><tr><td rowspan="4">Candidate Classifiers</td><td>LR</td><td>0.9925</td><td>0.8788</td><td>0.7838</td><td>0.8286</td><td>0.0025</td></tr><tr><td>DT</td><td>0.9838</td><td>0.6222</td><td>0.7568</td><td>0.6829</td><td>0.0108</td></tr><tr><td>RF</td><td>0.9913</td><td>0.8594</td><td>0.7432</td><td>0.7971</td><td>0.0029</td></tr><tr><td>XGBoost</td><td>0.9932</td><td>0.8333</td><td>0.8784</td><td>0.8553</td><td>0.0041</td></tr><tr><td rowspan="2">NLP Classifier</td><td>Forta</td><td>-</td><td>0.8778</td><td>0.5520</td><td>0.6208</td><td>-</td></tr><tr><td>Transformer</td><td>0.9876</td><td>0.7429</td><td>0.7027</td><td>0.7222</td><td>0.0057</td></tr><tr><td rowspan="4">Meta Classifier</td><td>LR</td><td>0.9950</td><td>0.9683</td><td>0.8133</td><td>0.8841</td><td>0.0006</td></tr><tr><td>DT</td><td>0.9913</td><td>0.7901</td><td>0.8533</td><td>0.8205</td><td>0.0054</td></tr><tr><td>SVM</td><td>0.9947</td><td>0.9143</td><td>0.8533</td><td>0.8828</td><td>0.0019</td></tr><tr><td>KNN</td><td>0.9953</td><td>0.9286</td><td>0.8667</td><td>0.8966</td><td>0.0016</td></tr></table>

## 6.1.2 Research Questions. We attempt to address the following research questions:

1) RQ1. How efective is LookAhead in detecting the adversarial contracts used in DeFi attacks?

2) RQ2. How do diferent empirical features contribute to model decision-making and how does PSCFT improve performance compared to using raw IR?

3) RQ3. How eficient can LookAhead be in identifying adversarial contracts?

4) RQ4. How practical is LookAhead in real-world scenarios?

## 6.2 RQ1: Efectiveness

6.2.1 Evaluation Metrics. In addition to commonly used binary classification performance metrics such as Accuracy, Precision, Recall and F1-Score, we also calculate False Positive Rates (FPR), which holds particular significance in our scenario.

6.2.2 Evaluation Results. Table 1 presents our experimental results evaluating the performance of various ML models (§5.3) for the LookAhead system, including four candidate classifiers, a transformer-based classifier, and four meta classifiers. Among the candidate classifiers, XGBoost achieves the highest F1-score (0.8553), while Random Forest has the lowest recall (0.7432). Overall the XGBoost-based classifier outperforms other candidate classifiers, demonstrating the best perfor mance across most metrics compared to others. It achieves a false positive rate as low as 0.0041 while maintaining a high efective detection rate for adversarial samples. Other models also perform well. Except the one based on DT, the F1-scores of all other classifiers exceed 0.79, proving the validity of our method. Building upon the results of the candidate models, diferent meta classifiers were constructed by combining the optimal candidate model XGBoost with the transformer. The results show that three of the four meta classifiers achieved higher F1-scores compared to the standalone XGBoost classifier. Among them, the KNN-based classifier demonstrated the best performance, achieving the highest F1-score (0.8966) while maintaining a low FPR (0.0016).

6.2.3 Comparison with Previous Works. We compare our approach with two state-of-the-art baselines: Forta [16], a NLP-based method for detecting all adversarial contracts, and BlockWatch-Dog [63], a specialized work focused exclusively on detecting reentrancy attack contracts.

Universality. To evaluate the universality of the LookAhead system, we compare its performance against BlockWatchDog, which specifically targets reentrancy attack contracts. We test our meta classifier on all the reentrancy related adversarial contracts in our test dataset, we observe that LookAhead successfully identified 83.33% of them, which is on par with the results presented by [63]. This observation along with the experiment results presented in §6.2.2 show that LookAhead not only generalizes well across diverse types of DeFi attacks but also maintains competitive performance when applied to specific attack categories.

Performance. The only similar work that uses machine learning for adversarial contract detection publicly available so far is the Forta Network [16]. We use Forta as the baseline for comparing the performance of our ML models.

Forta is an NLP-based method that relies solely on EVM bytecode, while our method is based on a combination of multi-dimensional features and semantic tokenization (PSCFT) of smart contracts. We perform a direct comparison between their model and our transformer model first, then we measure how much benefits can our meta classifier approach provide over Forta’s implementation. As the results in Table 1 indicate, our transformer alone can achieve much higher F1-Score than Forta, demonstrating the advantage of our PSCFT design. LookAhead’s meta classifier pushes the boundary even further by combining the capabilities of transformer model with the capabilities of other candidate classifiers to achieve an F1-Score of0.8966. As described in §5.1, contract verification status feature introduces some bias into our dataset, to provide a fair comparison, we also evaluate our models with �������� feature removed. The results show that even without this feature, our classifiers still significantly outperform the most recent model published by Forta (released on February 6, 2023), achieving an F1-Socre of 0.8696, highlighting the efectiveness of our solution.

6.2.4 Time Series Cross-Validation. As outlined in 6.1.1, our evaluation respects the chronological order of contract deployment, accounting for both the evolving nature of adversarial behaviours and LookAhead’s design to detect future attacks based on historical data. Table 1 shows the model performance results obtained by using the entire dataset with a 4:1 chronological split. We identify XGBoost as the optimal candidate classifier and KNN as the best-performing meta model. To validate the most appropriate candidate and meta models for our system and ensure its robustness under varying data conditions, we conduct evaluations using expanding window cross-validation. During this process, the dataset is chronologically divided into five equal splits. The training window starts with the first split and progressively expands by including additional splits, while the subsequent split serves as the test set in each iteration.

After cross-validation, we conclude that XGBoost achieves the highest average F1-score (0.8173), demonstrating stable performance across varying data splits. While it did not always outperform others in every split, its overall consistency reafirms its suitability as the optimal candidate classifier for our system. For the meta classifiers, which combine the optimal candidate model XGBoost with the transformer, the KNN-based model achieves the highest average F1-score (0.8355). However, its advantage over other models were marginal, suggesting that the choice of meta classifier models has limited impact on overall performance.

## 6.3 RQ2: Inter<sub>p</sub>retabilit<sub>y</sub>

6.3.1 Empirical Features. Understanding the weight distribution of features is useful for gaining insights into the underlying decision-making process of the models. To this end, we apply the interpretability method SHAP (Shapley Additive exPlanations) [44] to analyze the importance of empirical features for each candidate model.

The results reveal that both deployment and implementation features play significant roles in guiding the models to a decision. In the best-performing XGBoost model, the top three most important features are avg\_token\_call\_count, verified and fund\_from\_Anonymous, all of which also ranked within the top 10 across other candidate models. Additionally, balanceOf\_call\_count, fund\_from\_Safe and func\_count consistently appear within the top 15 in at least three candidate models, indicating their stable contribution to the model’s decision-making process. The notable importance of these features may be attributed to their strong correlation with typical attack patterns, allowing the model to efectively capture behavioural characteristics of adversarial contracts.

6.3.2 PSCFT. The design of PSCFT abstracts the IR of a contract into a more concise and semantically meaningful format. To concretely demonstrate how this design enhances model performance, we compare the transformer classifier trained on PSCFT with one trained on the raw IR generated by Gigahorse. The results show a significant performance gap, where the model using raw IR achieves an F1-score of only 0.4068, substantially lower than the 0.7222 achieved by the PSCFT-based model.

This performance discrepancy highlights the limitations of directly using raw IR, which contains numerous low-level instructions (e.g., arithmetic operations, memory reads/writes) and excessive variables that obscure critical call patterns and their control flows. Such extraneous details increase input complexity, making it harder for the model to identify meaningful behavioural patterns relevant to adversarial contract. Our pruning process can efectively remove redundant instructions and enable semantic recovery, allowing PSCFT to focus on critical external calls and control flows, leading to clearer and more reliable model decisions and improved prediction accuracy.

## 6.4 RQ3: Eficienc<sub>y</sub>

LookAhead aims to detect adversarial contracts and enable rapid responses within the rescue time frame (see §3.2.4). To this end, we evaluate the eficiency using all adversarial samples.

We employed a linear detection pipeline, which processes each sample through the following steps: 1) retrieve deployment features via external APIs, 2) execute our custom-designed Soufle Datalog and PSCFT generation scripts to extract implementation features, and 3) preprocess the data and feed it into pre-trained XGBoost and transformer classifiers. Their outputs are then combined by a pre-trained meta-classifier for the final prediction. We measure the time taken at each step.

The results showed that, on average, external API requests took 3.78 seconds, while data preprocessing and model inference required only 0.01 seconds. The most time-consuming step is binary lifting and analysis. Except for one large contract (taking 458 seconds), most PSCFTs were generated under 20 seconds, averaging 7.52 seconds. Furthermore, by considering the rescue time frames of the corresponding DeFi attacks, we found that in 92.8% of the cases, our system successfully detected the adversarial contract before the adversarial transaction occurred. Notably, eficiency can be further improved by employing multi-threading and local blockchain nodes, which could substantially reduce the time required to obtain deployment features.

## 6.5 RQ4: Practicalit<sub>y</sub>

Setup We conducted live experiments targeting the Ethereum mainnet across two time periods, 14–18 August 2024 and 23 January–9 February 2025, by deploying the LookAhead system to the server. The system continuously monitored the transactions in each new block via RPC providers [2] in near real-time. Upon detecting a contract creation (through a normal transaction), scripts were executed to extract features, which were then preprocessed and fed into the pre-trained classifiers (XGBoost and KNN, the best-performing setup as identified in RQ1). If classified as adversarial, the contract address was recorded for further verification.

Table 2. Re<sub>p</sub>resentative true <sub>p</sub>ositive adversarial contracts detected durin<sub>g</sub> the live ex<sub>p</sub>eriment<sub>,</sub> involved in atacks triggered by diferent root causes. Window indicates the time interval between contract deployment and the adversarial transaction. Runtime denotes the detection time, and Victim specifies whether the victim <sub>a</sub>dd<sub>ress was expose</sub>d d<sub>ur</sub>i<sub>ng con</sub>t<sub>rac</sub>t <sub>crea</sub>ti<sub>on.</sub>

<table><tr><td>Address</td><td>Time</td><td>Window</td><td>Runtime</td><td>Victim</td><td>Root Cause</td><td>Loss ($)</td></tr><tr><td>0x9f2790</td><td>2024/8/16</td><td>25344s</td><td>11.53s</td><td>No</td><td>Price Manipulation</td><td>15.5k</td></tr><tr><td>0x90744c</td><td>2024/8/16</td><td>636s</td><td>4.84s</td><td>Yes</td><td>Stale Oracle</td><td>21.5k</td></tr><tr><td>0x152619</td><td>2024/8/18</td><td>336s</td><td>5.67s</td><td>Yes</td><td>Access Control Flaw</td><td>1.2k</td></tr><tr><td>0x172133</td><td>2025/1/23</td><td>3504s</td><td>6.12s</td><td>No</td><td>Absence of Input Validation</td><td>2.5k</td></tr><tr><td>0x203f20</td><td>2025/2/6</td><td>120s</td><td>14.35s</td><td>Yes</td><td>Arbitrary Transfer</td><td>37k</td></tr><tr><td>0xbfe102</td><td>2025/2/6</td><td>96s</td><td>6.82s</td><td>Yes</td><td>Reentrancy</td><td>8.7k</td></tr></table>

Review Process We use a meticulous review process as follows to verify the outputs:

1) Wait for 2 weeks after each experimental period to ensure attacks have been confirmed;

2) Manually inspect follow-up transactions involving the contracts to validate adversarial nature;

3) Analyze and record key data, e.g. the financial damage caused by confirmed attacks, the time window between contract deployment and corresponding adversarial transaction.

Results During the experiment period, LookAhead analyzed a total of 23, 463 contracts and flagged 129 as adversarial. Among these, 62 involved follow-up transactions, in which we finally found 26 true positives that were used to perform attacks, resulting in over \$150,000 USD in financial losses. These attacks were driven by various underlying issues, with Arbitrary Transfer [45] being the most common, involving 6 contracts. Table 2 presents representative adversarial contracts involved in attacks stemming from diferent root causes, highlighting the universality of our approach which efectively identifies various types of adversarial behaviours.

In terms of detection eficiency, except only two cases where the deployment and attack transaction were confirmed within the same block, LookAhead produced a response for all the attack events before the attacker actually initiated the adversarial transaction. Additionally, we observed most of the false positives are related to arbitrage, showing that arbitrage behaviour resembles some patterns in adversarial activities, such as reliance on flashloans and frequent token-related function calls. To evaluate potential false negatives, we reviewed the alerts posted by security company (e.g. [6, 9, 26]) during the experiment period, as manually checking for false negatives among a large number of contracts is challenging. This process identified four reported attacks that fell within our detection scope (§3.2.1), all of which had already been correctly flagged.

The fact that, during our live experiment, LookAhead was able to respond in a timely manner before the majority of real-world attacks occurred, and successfully detected all externally reported attacks, further demonstrates its practicality in real-world scenarios.

## 7 Di<sub>scuss</sub>i<sub>on</sub>

## 7<sub>.</sub>1 Th<sub>rea</sub>t<sub>s</sub> t<sub>o</sub> V<sub>a</sub>lidit<sub>y</sub>

1) Limited by the APIs given by the blockchain explorers, we cannot determine whether contract verification occurred upon deployment; 2) Our dataset sufers from a class imbalance of approximately 1 : 40 due to the scarcity of adversarial contracts, which may afect model performance even after oversampling; 3) Benign contracts were selected based on heuristic approach. There is still a possibility that we have mislabelled some adversarial contracts as benign ones; 4) As discussed in §4.1.2, our study does not consider contracts deployed via internal transactions. The inherent dif ferences between the two deployment methods may afect certain deployment features, introducing potential bias to our classifier when detecting contracts deployed via internal transactions.

## 7<sub>.</sub>2 Li<sub>m</sub>it<sub>a</sub>ti<sub>ons</sub>

In addition to the inherent limitations of our threat model definition (§3.2), there are mainly two aspects of LookAhead’s design that could potentially hinder its performance.

Detection Evasion Since our classifier relies on extracted features and historical attack patterns, it remains susceptible to evasion techniques. In the cat-and-mouse game of DeFi attack and defense, attackers may adapt their strategies to bypass our feature-based and model-driven approach. For instance, they could adopt code obfuscation to alter the contract’s CFG or utilize KYC-required exchanges as funding sources, making it harder for our system to recognize adversarial intentions. NLP-based Approach LookAhead’s NLP-based models may not make full use of the control flow graph structure present in smart contract code. More advanced ML architectures such as Graph Neural Network (GNN) could be used in future works in combination with a more intricate representation design that capture both intra-function CFGs and inter-function call relationships.

## 8 R<sub>e</sub>l<sub>a</sub>t<sub>e</sub>d W<sub>or</sub>k

Adversarial Transaction Detection. Prior research primarily focus on detecting adversarial transactions used in DeFi attacks. Some works [33, 40, 53, 61] modify EVM to obtain more detailed transaction execution information for detecting adversarial transactions. BlockGPT [38] applies large language model to detect abnormal transactions. Qin et al. [50] introduce the Execution Prop erty Graph to enable timely detection. However, these methods become inefective in identifying attacks when attackers leverage private mempool services to initiate adversarial transactions.

Adversarial Contract Detection. The study of adversarial contract detection in DeFi attacks remains relatively underexplored. Forta [16] adopts an NLP-based approach for adversarial contract detection; however, its reliance on simple bytecode-level analysis results in limited efectiveness. Beyond Forta, the only comparable work is BlockWatchDog [63]. However, their approach is restricted to reentrancy attacks and relies on prior knowledge of victim addresses, thereby limiting its practical utility in real-world scenarios. Our method integrates multi-faceted features with semantic tokenization to allow more efective detection across various DeFi attack types.

## 9 C<sub>onc</sub>l<sub>us</sub>i<sub>on</sub>

In this paper, we propose a new framework for efectively detecting DeFi attacks. We build the first large-scale comprehensive dataset for training classifiers for the binary classification of adversarial contracts. Through the analysis of DeFi attack patterns, we extract a multi-dimensional feature set along with a special tokenization technique that is able to capture the intrinsic behaviours hidden in adversarial contracts. Utilizing these features, we carefully design multiple classifiers and evaluate them comprehensively. Experiments show that our method performs exceptionally well in detecting adversarial contracts compared to the previous state-of-the-arts with our models reaching F1-Scores as high as 0.8966 and a false positive rate as low as 0.16% on our labeled datasets.

## 10 D<sub>a</sub>t<sub>a</sub> A<sub>va</sub>il<sub>a</sub>bilit<sub>y</sub>

To promote open science, we have made our dataset and source code publicly available [21].

## A<sub>c</sub>k<sub>now</sub>l<sub>e</sub>d<sub>gmen</sub>t<sub>s</sub>

This work is sponsored in part by National Key Research and Development Program of China (2023YFB2704000).

## R<sub>e</sub>f<sub>erences</sub>

[1] 2020. AAVE Protocol Whitepaper. https://github.com/aave/aave-protocol/blob/master/docs/Aave\_Protocol\_ Whitepaper\_v1\_0.pdf Accessed 2024.

[2] 2024. Alchemy. https://www.alchemy.com/ Accessed 2024.

[3] 2024. Arbitrum Documentation. https://docs.arbitrum.io/ Accessed 2024.

[4] 2024. Attacker Drains \$182M From Beanstalk Stablecoin Protocol. https://www.coindesk.com/tech/2022/04/17/attackerdrains-182m-from-beanstalk-stablecoin-protocol/ Accessed 2024.

[5] 2024. Attacker Hacks Arbitrum’s Treasure DAO for Over 100 NFTs by Leveraging Marketplace Exploit. https://news. bitcoin.com/attacker-hacks-arbitrums-treasure-dao-for-over-100-nfts-by-leveraging-marketplace-exploit/ Accessed 2024.

[6] 2024. Blocksec. https://blocksec.com/ Accessed 2024.

[7] 2024. BNB Chain Documentation. https://docs.bnbchain.org/docs/overview Accessed 2024.

[8] 2024. BurgerSwap Hit by Flash Loan Attack Netting Over \$7M. https://www.coindesk.com/markets/2021/05/28 burgerswap-hit-by-flash-loan-attack-netting-over-7m/ Accessed 2024.

[9] 2024. Certik. https://www.certik.com/ Accessed 2024.

[10] 2024. ChangeNOW: Instant Cryptocurrency Exchange. https://changenow.io/ Accessed 2024.

[11] 2024. DeFi Hacks Reproduce. https://github.com/SunWeb3Sec/DeFiHackLabs Accessed 2024.

[12] 2024. ERC20 Token Standard. https://eips.ethereum.org/EIPS/eip-20 Accessed 2024.

[13] 2024. ERC721 Token Standard. https://eips.ethereum.org/EIPS/eip-721 Accessed 2024.

[14] 2024. Ethereum Signature Dataset. https://www.4byte.directory/ Accessed 2024.

[15] 2024. Flashbots Documentation. https://docs.flashbots.net/ Accessed 2024.

[16] 2024. Forta Network. https://forta.org/blog/how-fortas-predictive-ml-models-detect-attacks-before-exploitation Accessed 2024.

[17] 2024. Foundry. https://github.com/foundry-rs/foundry Accessed 2024.

[18] 2024. Label Word Cloud on BscScan. https://bscscan.com/labelcloud Accessed 2024.

[19] 2024. Label Word Cloud on Etherscan. https://etherscan.io/labelcloud Accessed 2024.

[20] 2024. Learn EVM Attacks. https://github.com/coinspect/learn-evm-attacks Accessed 2024.

[21] 2024. LookAhead dataset and source code. https://github.com/zju-abclab/LookAhead Accessed 2024.

[22] 2024. Optimism Documentation. https://community.optimism.io/ Accessed 2024.

[23] 2024. PancakeBunny tanks 96% following \$200M flash loan exploit. https://cointelegraph.com/news/pancakebunnytanks-96-following-200m-flash-loan-exploit Accessed 2024.

[24] 2024. Polygon Documentation. https://docs.polygon.technology/ Accessed 2024.

[25] 2024. SideShift: No Sign-Up Crypto Exchange. https://sideshift.ai/ Accessed 2024.

[26] 2024. SlowMist Hacked Events. https://hacked.slowmist.io/ Accessed 2024.

[27] 2024. SushiSwap Exchange Sufers Major \$3.3 Million Smart Contract Hack. https://cryptonews.com/news/sushiswapexchange-sufers-major-33-million-smart-contract-hack-heres-what-happened.htm Accessed 2024.

[28] 2024. SushiSwap’s Token Launchpad Hackedfor Over \$3M in Ethereum. https://decrypt.co/81120/sushiswaps-tokenlaunchpad-hacked-over-3m-ethereum Accessed 2024.

[29] 2024. Tornado Cash Label on Etherscan. https://etherscan.io/accounts/label/tornado-cash Accessed 2024

[30] 2024. Total Value Locked - DefiLlama. https://defillama.com/ Accessed 2024.

[31] Hayden Adams, Noah Zinsmeister, Moody Salem, River Keefer, and Dan Robinson. 2021. Uniswap v3 Core. https: //uniswap.org/whitepaper-v3.pdf Accessed 2024.

[32] Yixin Cao, Chuanwei Zou, and Xianfeng Cheng. 2021. Flashot: a snapshot of flash loan attack on DeFi ecosystem. arXiv preprint arXiv:2102.00626 (2021).

[33] Ting Chen, Rong Cao, Ting Li, Xiapu Luo, Guofei Gu, Yufei Zhang, Zhou Liao, Hang Zhu, Gang Chen, Zheyuan He, et al. 2020. SODA: A Generic Online Detection Framework for Smart Contracts.. In NDSS.

[34] Philip Daian, Steven Goldfeder, Tyler Kell, Yunqi Li, Xueyuan Zhao, Iddo Bentov, Lorenz Breidenbach, and Ari Juels. 2020. Flash boys 2.0: Frontrunning in decentralized exchanges, miner extractable value, and consensus instability. In 2020 IEEE Symposium on Security and Privacy (SP). IEEE, 910–927.

[35] Chris Dannen. 2017. Introducing Ethereum and solidity. Vol. 1. Springer.

[36] DLNews. 2024. Conic Finance sufers \$3m exploit in twist to ‘typical re-entrancy attack’. https://www.dlnews.com/ articles/defi/conic-finance-sufers-exploit-similar-to-the-dao-hack/ Accessed 2024.

[37] Shayan Eskandari, Seyedehmahsa Moosavi, and Jeremy Clark. 2020. Sok: Transparent dishonesty: front-running attacks on blockchain. In Financial Cryptography and Data Security: FC 2019 International Workshops, VOTING and WTSC, St. Kitts, St. Kitts and Nevis, February 18–22, 2019, Revised Selected Papers 23. Springer, 170–189.

[38] Yu Gai, Liyi Zhou, Kaihua Qin, Dawn Song, and Arthur Gervais. 2023. Blockchain large language models. arXiv preprint arXiv:2304.12749 (2023).

[39] Neville Grech, Lexi Brent, Bernhard Scholz, and Yannis Smaragdakis. 2019. Gigahorse: thorough, declarative de compilation of smart contracts. In 2019 IEEE/ACM 41st International Conference on Software Engineering (ICSE). IEEE, 1176–1186.

[40] Shelly Grossman, Ittai Abraham, Guy Golan-Gueta, Yan Michalevsky, Noam Rinetzky, Mooly Sagiv, and Yoni Zohar. 2017. Online detection of efectively callback free objects with applications to smart contracts. Proceedings of the ACM on Programming Languages 2, POPL (2017), 1–28.

[41] Bishwas C Gupta and Sandeep K Shukla. 2019. A study of inequality in the ethereum smart contract ecosystem. In 2019 Sixth International Conference on Internet of Things: Systems, Management and Security (IOTSMS). IEEE, 441–449.

[42] Haibo He, Yang Bai, Edwardo A Garcia, and Shutao Li. 2008. ADASYN: Adaptive synthetic sampling approach for imbalanced learning. In 2008 IEEE international joint conference on neural networks (IEEE world congress on computational intelligence). Ieee, 1322–1328.

[43] Queping Kong, Jiachi Chen, Yanlin Wang, Zigui Jiang, and Zibin Zheng. 2023. Defitainter: Detecting price manipulation vulnerabilities in defi protocols. In Proceedings of the 32nd ACM SIGSOFT International Symposium on Software Testing and Analysis. 1144–1156.

[44] Scott M Lundberg and Su-In Lee. 2017. A unified approach to interpreting model predictions. Advances in neural information processing systems 30 (2017).

[45] Fuchen Ma, Meng Ren, Lerong Ouyang, Yuanliang Chen, Juan Zhu, Ting Chen, Yingli Zheng, Xiao Dai, Yu Jiang, and Jiaguang Sun. 2023. Pied-piper: Revealing the backdoor threats in ethereum erc token contracts. ACM Transactions on Software Engineering and Methodology 32, 3 (2023), 1–24.

[46] Fernando Martinelli and Nikolai Mushegian. 2019. A non-custodial portfolio manager, liquidity provider, and price sensor. https://balancer.fi/whitepaper.pdf Accessed 2024.

[47] Tomas Mikolov. 2013. Eficient estimation of word representations in vector space. arXiv preprint arXiv:1301.3781 (2013).

[48] Bahareh Parhizkari, Antonio Ken Iannillo, Christof Ferreira Torres, Sebastian Banescu, Joseph Xu, and Radu State. 2023. Timely Identification of Victim Addresses in DeFi Attacks. In European Symposium on Research in Computer Security. Springer, 394–410.

[49] Kaihua Qin, Stefanos Chaliasos, Liyi Zhou, Benjamin Livshits, Dawn Song, and Arthur Gervais. 2023. The blockchain imitation game. arXiv preprint arXiv:2303.17877 (2023).

[50] Kaihua Qin, Zhe Ye, Zhun Wang, Weilin Li, Liyi Zhou, Chao Zhang, Dawn Song, and Arthur Gervais. 2023. Towards Automated Security Analysis of Smart Contracts based on Execution Property Graph. arXiv preprint arXiv:2305.14046 (2023).

[51] Kaihua Qin, Liyi Zhou, Benjamin Livshits, and Arthur Gervais. 2021. Attacking the defi ecosystem with flash loans for fun and profit. In International conference on financial cryptography and data security. Springer, 3–32.

[52] Leshner Robert and Hayes Geofrey. 2019. Compound: The Money Market Protocol. https://compound.finance documents/Compound.Whitepaper.pdf Accessed 2024.

[53] Michael Rodler, Wenting Li, Ghassan O Karame, and Lucas Davi. 2018. Sereum: Protecting existing smart contracts against re-entrancy attacks. arXiv preprint arXiv:1812.05934 (2018).

[54] Christof Ferreira Torres, Ramiro Camino, et al. 2021. Frontrunner jones and the raiders of the dark forest: An empirical study of frontrunning on the ethereum blockchain. In 30th USENIX Security Symposium (USENIX Security 21). 1343–1359.

[55] A Vaswani. 2017. Attention is all you need. Advances in Neural Information Processing Systems (2017).

[56] Bin Wang, Xiaohan Yuan, Li Duan, Hongliang Ma, Chunhua Su, and Wei Wang. 2022. DeFiScanner: Spotting DeF Attacks Exploiting Logic Vulnerabilities on Blockchain. IEEE Transactions on Computational Social Systems (2022).

[57] Dabao Wang, Siwei Wu, Ziling Lin, Lei Wu, Xingliang Yuan, Yajin Zhou, Haoyu Wang, and Kui Ren. 2020. Towards understanding flash loan and its applications in defi ecosystem. arXiv preprint arXiv:2010.12252 (2020).

[58] Zhipeng Wang, Stefanos Chaliasos, Kaihua Qin, Liyi Zhou, Lifeng Gao, Pascal Berrang, Benjamin Livshits, and Arthur Gervais. 2023. On how zero-knowledge proof blockchain mixers improve, and worsen user privacy. In Proceedings of the ACM Web Conference 2023. 2022–2032.

[59] Ben Weintraub, Christof Ferreira Torres, Cristina Nita-Rotaru, and Radu State. 2022. A flash (bot) in the pan: measuring maximal extractable value in private pools. In Proceedings ofthe 22nd ACM Internet Measurement Conference. 458–471.

[60] Gavin Wood et al. 2014. Ethereum: A secure decentralised generalised transaction ledger. Ethereum project yellow paper 151, 2014 (2014), 1–32.

[61] Siwei Wu, Lei Wu, Yajin Zhou, Runhuai Li, Zhi Wang, Xiapu Luo, Cong Wang, and Kui Ren. 2022. Time-travel investigation: toward building a scalable attack detection framework on ethereum. ACM Transactions on Software Engineering and Methodology (TOSEM) 31, 3 (2022), 1–33.

[62] Siwei Wu, Zhou Yu, Dabao Wang, Yajin Zhou, Lei Wu, Haoyu Wang, and Xingliang Yuan. 2023. DeFiRanger: Detecting DeFi Price Manipulation Attacks. IEEE Transactions on Dependable and Secure Computing (2023).

[63] Shuo Yang, Jiachi Chen, Mingyuan Huang, Zibin Zheng, and Yuan Huang. 2024. Uncover the premeditated attacks: detecting exploitable reentrancy vulnerabilities by identifying attacker contracts. In Proceedings of the IEEE/ACM 46th International Conference on Software Engineering. 1–12.

[64] Mengya Zhang, Xiaokuan Zhang, Yinqian Zhang, and Zhiqiang Lin. 2020. {TXSPECTOR}: Uncovering attacks in ethereum from transactions. In 29th USENIX Security Symposium (USENIX Security 20). 2775–2792.

[65] Zhuo Zhang, Zhiqiang Lin, Marcelo Morales, Xiangyu Zhang, and Kaiyuan Zhang. 2023. Your exploit is mine: Instantly synthesizing counterattack smart contract. In 32nd USENIX Security Symposium (USENIX Security 23). 1757–1774.

[66] Liyi Zhou, Xihan Xiong, Jens Ernstberger, Stefanos Chaliasos, Zhipeng Wang, Ye Wang, Kaihua Qin, Roger Wattenhofer, Dawn Song, and Arthur Gervais. 2023. Sok: Decentralized finance (defi) attacks. In 2023 IEEE Symposium on Security and Privacy (SP). IEEE, 2444–2461.

[67] Shunfan Zhou, Malte Möser, Zhemin Yang, Ben Adida, Thorsten Holz, Jie Xiang, Steven Goldfeder, Yinzhi Cao, Martin Plattner, Xiaojun Qin, et al. 2020. An ever-evolving game: Evaluation of real-world attacks and defenses in ethereum ecosystem. In 29th USENIX Security Symposium (USENIX Security 20). 2793–2810.

Received 2024-09-13; accepted 2025-04-01