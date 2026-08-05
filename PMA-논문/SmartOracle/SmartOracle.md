# SmartOracle: Generating Smart Contract Oracle via Fine-Grained Invariant Detection

Jianzhong Su, Jiachi Chen, Zhiyuan Fang, Xingwei Lin, Yutian Tang and Zibin Zheng, Fellow, IEEE,

Abstract—As decentralized applications (DApps) proliferate, the increased complexity and usage of smart contracts have heightened their susceptibility to security incidents and financial losses. Although various vulnerability detection tools have been developed to mitigate these issues, they often suffer poor performance in detecting vulnerabilities, as they either rely on simplistic and general-purpose oracles that may be inadequate for vulnerability detection, or require user-specified oracles, which are labor-intensive to create. In this paper, we introduce SmartOracle, a dynamic invariant detector that automatically generates fine-grained invariants as application-specific oracles for vulnerability detection. From historical transactions, SmartOracle uses pattern-based detection and advanced inference to construct comprehensive properties, and mines multi-layer likely invariants to accommodate the complicated contract functionalities. After that, SmartOracle identifies smart contract vulnerabilities by hunting the violated invariants in new transactions. In the field of invariant detection, SmartOracle detects 50% more ERC20 invariants than existing dynamic invariant detection and achieves 96% precision rate. Furthermore, we build a dataset that contains vulnerable contracts from real-world security incidents. SmartOracle successfully detects 466 abnormal transactions with an acceptable precision rate 96%, involving 31 vulnerable contracts. The experimental results demonstrate its effectiveness in detecting smart contract vulnerabilities, especially those related to complicated contract functionalities.

Index Terms—Smart Contracts, Invariant Analysis, Vulnerability Detection

## 1 INTRODUCTION

Smart contracts are computer programs that run on a blockchain network. They act as the rules for DApps, which manage their funds and status without any central authority [1]. However, smart contracts are susceptible to security incidents due to various types of vulnerabilities, such as Reentrancy [2] and Price Manipulation [3]. According to Slowmist [4], these attacks have resulted in a total loss of approximately 32 billion USD on blockchain platforms, highlighting significant security challenges within the ecosystem.

To detect these vulnerabilities, various tools have been developed by employing a variety of technologies, such as static analysis [5], fuzzing [6], and formal verification [7]. Despite these efforts, the detection of smart contract vulnerabilities, particularly those with complex functionalities, remains ineffective in real-world scenarios [8]. This ineffectiveness arises primarily from two reasons. First, many existing tools [9]–[15] adopt simplistic and general-purpose oracles to cover all patterns of complex vulnerabilities. However, these tools may fail to reveal many vulnerabilities, as analyzing them usually requires application-specific oracles. For example, some tools use “check-effect-interaction” oracle for Reentrancy detection [2]; however, the complicated contract functionalities render such a simple oracle ineffective, resulting in a high rate of false positives [16]. Second, although some tools [17]–[19] allow users to design custom oracles to detect vulnerabilities, these oracles require expert knowledge and significant human effects based on users’ understanding of the contract. Consequently, the current reliance on inadequately tailored oracles limits the effectiveness of existing tools in identifying complex vulnerabilities within smart contracts.

Our solution. In this paper, we aim to propose a method to automatically generate application-specific oracles for detecting complex vulnerabilities within contracts. Program invariants are properties that remain constant at certain execution points and naturally act as rules and oracles for smart contracts. For example, a smart contract is expected to hold the invariant that “the tokens deposited into the contract are equal to the total amount claimed by users” during normal execution. However, this invariant is violated in contracts suffering from a “fake deposit” vulnerability [20], where the amount of tokens deposited is significantly lower than the attacker claims. Given the numerous transactions stored on the blockchain, we perform dynamic invariant detection on transactions to obtain likely invariants [21] of smart contracts and serve them as oracles for detecting vulnerabilities. Furthermore, complicated contract functionalities always require fine-grained invariants to fit. Specifically, some functions implement different functionalities with multiple branches, so we need to capture the corresponding finegrained invariants for each branch. (detailed in Section 3.2).

We introduce SmartOracle, a tool designed to detect complex vulnerabilities in smart contracts via fine-grained dynamic invariant detection. SmartOracle starts the detection by analyzing historical transactions of a target contract and recording their execution traces (e.g., the functions, state variables, and executed branches). Based on the execution traces, SmartOracle delineates invariants of three distinct layers: Contract, Function, and Branch, for accurately fitting contract behavior. At each layer, SmartOracle employs pattern-based property detection and advanced property inference to construct properties. Considering that some historical transactions may be initiated by abnormal behaviors, which could introduce noise for the calculation of invariants. For instance, the contract had been attacked due to the abnormal state maintained by the manager but is still active after recovering to the normal state [22]. Thus, SmartOracle reserves properties that satisfy threshold of historical transactions as likely invariants to enhance robustness, thereby minimizing the impact of anomalies hidden in historical transactions. Based on the mined invariants, SmartOracle performs run-time verification on new transactions and reports their violated cases, indicating abnormal behavior or vulnerabilities triggered in contracts.

To evaluate SmartOracle, we first assess its ability to mine invariants. Utilizing a benchmark of 246 ERC20 contracts [23], SmartOracle identifies 50% more ERC20 invariants than the existing dynamic invariant detector, achieving a precision rate of 96%. Furthermore, we create a dataset of 65 vulnerable contracts derived from real-world security incidents, most of which contain vulnerabilities with complicated functionalities. We employ the detected invariants as oracles to reveal smart contract vulnerabilities and the abnormal transactions they cause. SmartOracle successfully reveals 31 vulnerabilities, involving 466 transactions that violate the mined invariants. In addition, the efficiency of SmartOracle satisfy the requirement of real-time detection on Ethereum. These experimental results demonstrate the effectiveness of SmartOracle in vulnerability detection and run-time verification.

In summary, our main contributions are as follows:

• We propose SmartOracle, a fine-grained dynamic invariant detector for smart contracts. It mines finegrained invariants and serves them as applicationspecific oracles for detecting complex vulnerabilities.

• We evaluate the ability of invariant detection, SmartOracle detects 50% more ERC20 invariants than existing dynamic invariant detector and achieves 96% precision.

• We construct a benchmark containing 65 real-world vulnerable contracts, most of which are complex vulnerabilities. SmartOracle reveals 31 vulnerabilities and 466 related abnormal transactions.

• We publish the code of SmartOracle and related datasets in https://github.com/Demonhero0/ SmartOracle.

## 2 BACKGROUND

## 2.1 Smart Contracts

Smart contracts are Turing-complete programs running on top of the blockchain. They are typically written in a highlevel programming language, such as Solidity [24]. Through compilation, we can obtain their Application Binary Interface (ABI) and bytecode [25]. After deployment, smart contracts run within Ethereum Virtual Machine (EVM) and maintain their account states, which store Ether balances and storage with a data structure serialized as Recursive Length Prefix (RLP) [25]. In particular, storage is a mapping between bytes32 keys to storage slots that record the state variables in bytes32.

![](images/b95c2948e05f2d62493b511f97e9452eef001d01a6af1a5bcced300a4c4b4160.jpg)  
Fig. 1: The Invariants in ERC20 Contract.

## 2.2 Transactions

Transactions are responsible for transferring Ethers or invoking smart contracts. They include information such as sender, receiver, amount of Ethers, and data recording the invoked function and its parameters. As smart contracts can initiate transactions, a transaction can derive multiple transactions arranged in a tree structure. For simplicity, transactions can be divided into two types, i.e., external transactions (initiated by external actors) and internal transactions (initiated by contracts). During transaction execution, smart contracts can emit predefined events to record specific behaviors, which contain address (i.e., the emitter), topics (i.e., the signature and indexed parameters) and data (i.e., non-indexed parameters).

## 2.3 Invariants

For a program, an invariant is a property (i.e., a logical formula) that remains true at certain points during execution. This property is important to the program and is widely used in bug detection, etc [21]. In this paper, we focus on three layers of smart contract invariants, we illustrate them by Fig. 1:

• Contract Invariant: the property holds for any call to the contract. The ERC20 contract uses totalSupply to record the sum of users’ token balance, it satisfies the property Contract Inv#1.

• Function Invariant: the property holds for any call to the function. The function transfer is responsible for the transfer of the token between users, which would not change the sum of the token balance of users. It satisfies the property Function Inv#2.

• Branch Invariant: the property holds for the execution branch. The function transfer only changes the token balance of the users when executing the branch with condition balances[msg.sender] ≥ amt (line 7). So Branch Inv#3 only holds in the specific branch.

## 3 MOTIVATION

In this section, we use a motivating example to illustrate the challenges of detecting vulnerabilities related to complicated contract functionality. Then, we introduce how to use invariant to address these challenges.

![](images/40fdb43e3678216563f78ce2d618b8ac730964a3fdfd59f98c0826b3de59bc91.jpg)  
Fig. 2: The simplified deposit function.

## 3.1 Challenges

Smart Contract vulnerabilities associated with complex functionalities have led to numerous security incidents and occupy 81% bountied vulnerabilities of Solidity-based smart contracts from September 2021 to May 2023 [26]. These vulnerabilities usually require application-specific oracles for detection [8]. Fig. 2 illustrates a simplified function deposit in thorchain contract <sup>1</sup>. Normally, when a user invokes this function, the contract will transfer user’s tokens to itself by executing transferFrom (line 6 or 10) and emits a deposit certificate, i.e., the event Deposit (line 13). In particular, the function deposit has two conditions (corresponding to two branches):

1) asset == RUNE: the asset tokens are transferred to vault (line 6, Branch#1).

2) asset != RUNE: the asset tokens are transferred to this (line 10, Branch#2).

However, since the contract lacks verification of whether tokens are actually deposited, the attacker could perform a “fake deposit” attack. Specifically, the attacker first deploys a fake token contract as fake token, which does not transfer any tokens in function transferFrom. The attacker then calls the function deposit with the parameter asset = fake token. As a result, the contract would normally execute Branch #2 (line 8-10) and emit an event Deposit (line 13), and the attacker would obtain the deposit certificate without paying any tokens.

To detect the above vulnerability, it is essential to understand the deposit logic and design specific oracles, that is, verifying “the amount of tokens deposited is equal to the amount claimed by the user”. Unfortunately, these application-specific oracles always require a great deal of expert knowledge and a significant human effect in the design. Consequently, both the variety of vulnerabilities and the limitations of existing oracles pose significant challenges in protecting smart contracts from the harm of vulnerabilities.

## 3.2 Our Insights

Invariant and Vulnerability Detection. We notice that smart contracts should strictly follow some invariants, which are regular rules for their functionalities [23], [26]. As shown in Fig. 2, when users calls the function deposit, they should pay safeAmt amount of ast tokens, the function satisfies function invariant Inv#3 (token[ast][msg.sender] means the ast token balance of msg.sender). Compared to function invariant, branch invariant is more granular and matches the functionality of the contract more accurately. For example, in Fig. 2, Branch#1 and Branch#2 execute different functionalities (i.e., sending tokens to different addresses), so they satisfy different branch invariants, Inv#1 and Inv#2, respectively. However, since the attacker does not pay any tokens to the contract when invoking the function deposit, the branch invariant Inv#2 is violated.

![](images/d4c60a94109b8ed1e00d965983d5b8c8b00b5bf3ce74d6df271539fa3217d7de.jpg)  
Fig. 3: Overview of SmartOracle. SmartOracle mines invariants from historical txs and detects the violations of invariants in new txs.

In fact, some existing work has used invariants as oracles, which verifies the satisfaction / violation of the invariant for vulnerability detection [17], [27]. However, their requirement of user-specific invariant limits their availability.

Invariant from Transactions. Previous works [23], [28] prove that historical transactions contain abundant information about the functionalities of smart contracts. To obtain smart contract invariants, we have gained insight by observing the transactions in the case mentioned above. In benign transactions calling the function deposit, the parameter amt and the token balance always satisfy some properties (e.g., Inv#3), we could summarize these hiding properties as likely invariants [21]. In particular, since some functionalities are distinguished by different branches, the corresponding invariant is only satisfied in a certain branch (e.g., Inv#1,#2). In these cases, the techniques that focus only on function or contract invariants (e.g., InvCon [23]) may fail to detect effective invariants.

To this end, our goal is to use dynamic invariant detection to extract fine-grained likely invariants to approximate real invariants, which serve as application-specific oracles for vulnerability detection.

## 4 SMARTORACLE

We propose SmartOracle, it automatically mines finegrained likely invariants for detecting smart contract vulnerabilities. As shown in Fig. 3, SmartOracle consists of three components, i.e., Transaction Parser, Invariant Miner and Invariant Checker. Given a target contract and its historical transactions (historical txs), Transaction Parser extracts the ABI and storage layout of the target contract, based on which it parses historical txs and extracts their execution traces (traces). To better fit the complicated contract functionalities, Invariant Miner mine invariants of three layers, Contract, Function, and Branch. For each layer, it employs pattern-based property detection and advanced property inference to construct properties, based on which it reserves the properties that satisfy threshold of transactions as likely invariants efficiently. Finally, given new transactions (new txs), Invariant Checker detects the transactions that violate the mined likely invariants to detect vulnerabilities in the target contract.

## 4.1 Transaction Parser

In this stage, we aim to extract the execution traces of transactions. Since both historcial txs and new txs need to be parsed into execution traces for further analysis, we use transaction uniformly to describe their extraction procedure for simplicity. As presented in Fig. 4, transaction parser extracts the ABI and the storage layout of the contract, based on which it parses the transactions and extracts their execution traces (trace).

## 4.1.1 Parsing Target Contract

Given a smart contract, we utilize a static analysis framework called Slither [14] to obtain its ABI and storage layout. The ABI records the name and type of the parameters in functions (events), which allows us to recover the value of parameters of functions (events) from their data in bytes. The storage layout records the name, type, location, and offset of storage slots for the state variable. It allows us to find state variables in storage slots and recover their value from bytes.

## 4.1.2 Parsing Transactions

Given a transaction of the target contract, we utilize our instrumented EVM to execute the raw transaction and record it as a call flow tree. Note that a call flow tree may contain multiple (internal) transactions derived from the external transaction. Therefore, we reserve the relevant transactions that call the target contracts and extract their execution traces as preparation for invariant detection.

Instrumented EVM. In this part, instead of using the RPC debug traceTransaction [29], we instrument EVM to extract execution traces for higher efficiency [30]. Specifically, we insert our code into the function ApplyMessage() [31], which is responsible for executing transactions and computing new states in EVM. In this way, a call stack is maintained to construct the tree structure of transactions and identify the transaction being executed. Furthermore, we insert recording codes into the handler of CALL, the way to initiate new transactions, to record the execution information of each transaction, such as emitted events and related account states.

Call Flow Tree (CFT). With the instrumented EVM, we execute the transaction to construct a tree structure that includes it and its derived internal transactions, as the CFT shown in Fig. 4. Specifically, there are two types of nodes in CFT, transaction nodes (e.g., 1 in CFT) and event nodes (e.g., 3 in CFT). The transaction node records the information of the call, including sender, receiver, the data recording the invoked function and its parameters, the amount of transferred ether, and related account states (i.e., the Ether balance and the storage slots being touched in the transaction). Furthermore, the transaction node also records the dynamic control flow of the call. The directed edge between the transaction nodes indicates that the parent transaction derives a new transaction. The event node includes the emitted address and parameters. The directed edge between the transaction node and the event node means that the transaction triggers the event. To filter out transactions relevant to the target contract, we traverse the CFT and reserve the related subtrees in which the root transaction node calls the target contract.

![](images/e6673ee350bd45594515efe57e566e1ffecdcae9a38964b65cd0eaebfa1544b1.jpg)  
Fig. 4: Workflow of Transaction Parser.

## 4.1.3 Extracting Execution Trace

In this part, we illustrate the extraction of execution traces based on the information gathered from contracts and transactions. For each subtree that calls the target contract, we first recover its execution branch from the dynamic control flow. Note that JUMPI is the only opcode that handles and changes the execution branches, so we record the destination of each JUMPI as branch. Then we extract its trace and divide it into three categories, Entry, Log and State, which are shown in Table 1.

(1) Entry. Entry means the input of the contract call, which can be handled by the caller and represents the intention of the sender. It includes the information of the transaction and the invoked function.

(i) transaction records the transaction information, such as the sender, receiver, block, and timestamp of the transaction.

(ii) function records the function and its parameters. Based on the ABI of the target contract, we parse the bytecode of the transaction that records the signature and parameters of the invoked function to recover the value and types of parameters.

(2) Log. Log records the logs (event) defined by developers during contract execution. As a result, events include execution information (e.g., execution failed or succeeded) and indicate the behaviors of the contract. Similarly to function, we use ABI to parse the bytecode of the event and recover the value and types of parameters. In particular, there may be multiple events within one transaction; we include all of them in the execution trace.

TABLE 1: Variables of execution trace.

<table><tr><td>Category</td><td>Variable</td><td>Description</td></tr><tr><td>Entry</td><td>transaction function</td><td>sender, receiver, block, timestamp the parameters of function</td></tr><tr><td>Log</td><td>event</td><td>the information of emitted events</td></tr><tr><td>State</td><td>state-variable token</td><td>the value of state variables the value of token balances</td></tr></table>

(3) State. State includes dynamic variables throughout the execution of the contract, representing the dynamic state of the contract. Since dynamic variables may change during the execution of the transaction, we record them in some essential program points [21]. On the one hand, the point that before and after the transaction execution is widely used in dynamic invariant analysis [23]. On the other hand, it is of great significance to record the dynamic variables at the points that the contract initiates internal transactions calling external contracts. This is because external contracts may alter the dynamic state and further exploit vulnerabilities in the target contract. For example, the attacker could exploit a Reentrancy vulnerability during the contract calling external contracts [16]. Consequently, we record the dynamic variables of the contract as part of the execution trace at these record points: (i) before and after the transaction execution, as the Pre-Call and Post-Call in the (2) of Fig. 4; and (ii) before and after the contract initiating internal transaction calling external contracts, as the Pre-SubCall and Post-SubCall in the (2) of Fig. 4. At each record point, we record these dynamic variables: state-variable and token.

(i) state-variable records the state variables of the contract. In particular, we only record the state variables that are read or written during the execution of the transaction to improve the efficiency of the analysis.

Note that the smart contract stores its state variables in storage (a mapping from bytes32 keys to storage slots that record bytes32 values), which does not indicate their corresponding state variables. In other words, given the storage of the contract, we cannot directly recover their state variables. To this end, according to the storage layout of the contract, we first build the relationships between state variables and storage slots. We then find the corresponding storage slots of state variables and recover their names, types, and values. Specifically, state variables with Value Types [24] (e.g., address, uint256) are arranged in the storage slots in order. Therefore, we can directly read the values of the state variables in their storage slots. For state variables with Reference Types [24] (e.g., dynamic array), we can calculate the hash value of the slot to obtain the start slot of the array and recover the value of the state variable. For state variables with Mapping Types [24], they are stored in special storage slots (i.e., the hash value of the mapping key and the slot [25]). In these cases, we first collect interesting values (e.g., the sender, the parameters of invoked function) as mapping keys. Then, we use them to calculate the hash values by keccak256 to locate the corresponding storage slots of state variables, and recover their value and types.

(ii) token records the balances of Ether and ERC20 tokens, which represent the financial properties of the contract. For Ether, we directly read the balances of related addresses from their account states. For ERC20 tokens, we obtain the token balances of the related addresses by executing the balanceOf function in ERC20 token contracts. For convenience, we record token variables as Mapping Types, that is, token[tokenAddress][user], where tokenAddress means the address of the token contract and user means the address of the user. Similarly to state variable, we reserve only token balances that are read or written during transaction execution to improve the efficiency of the analysis.

## 4.2 Invariant Miner

Based on the execution traces of transactions, we utilize an effective and efficient invariant detection to mine finegrained smart contract invariants.

Multi-Layer Invariant. As discussed in Section 3.2, a more fine-grained layer (i.e., branch layer) can accommodate more invariants of contract functionalities. However, due to the limited number of transactions for some branches (or functions), there may not be enough transactions to build confident invariants. In these cases, we use coarser invariants to fill the gap and ensure a comprehensive fit with contract functionalities. As in the example in Fig. 2, if the transaction only invokes the deposit function to execute Branch#1, we cannot extract any invariants about Branch#2 (e.g., Inv#2). Nonetheless, it is still possible to mine function invariants (e.g., Inv#3) as oracles.

Therefore, we divide execution traces into three layer groups: (1) Branch-layer group contains the traces of the same branch in a function; (2) Function-layer group contains the traces of the same function; (3) Contract-layer group contains all traces of the contract.

Consequently, we perform an invariant mining on each group to obtain its corresponding invariants. Specifically, we perform pattern-based detection and advanced inference to construct properties, and reserve the properties that satisfy across transactions as likely invariants. Furthermore, we propose a heuristic threshold-based mining strategy to increase the usefulness of invariant mining.

## 4.2.1 Property Construction

For each execution trace, we first use a pattern-based property detection to obtain basic properties. Then, we perform advanced property inference to derive more advanced properties. Both serve as candidate invariants for the final likely invariants.

Pattern-based Detection. We design property patterns of variables based on the specific characteristics of smart contracts, then adapt them to the trace variables and obtain the candidate basic invariants. Next, we illustrate the main patterns as follows (x, y and z denote variables; a and b refer to calculated constants).

(i) Comparison pattern refers to common properties among variables, such as $x = y , x = - y ,$ and $x \ = = \ a .$ These patterns are prevalent in smart contracts.

(ii) Membership pattern means that a variable is a member of another variable. Considering y is an Array variable that can include elements in Value Types $( \mathrm { e . g . }$ , uint256) or Struct, we denote $x \in \ y$ if one of the following conditions is satisfied: (i) if the member of y is Value Types, x is equal to a member of y; (ii) if the member of y is Struct, x is equal to one of the components of a member of y (e.g., y[0].address = x). This pattern also occurs frequently in the smart contracts with access control checks. For example, some functions implement access control through a whitelist, so they hold the property like msg.sender ∈ whitelist.

(iii) Arithmetic pattern describes the arithmetic relations between variables, including linear relations $\begin{array} { r l } { ( \mathbf { e . g . } , \ z } & { { } = } \end{array}$ ax + by + c) and quadratic relations (e.g., xy = a). In particular, DeFi apps always use these arithmetic relations to carry out their financial models. For example, Uniswap V2 [32] performs its swap business based on the relation xy = a. Thus, using these arithmetic patterns can effectively model the relation of variables in financial models.

Advanced Inference. Based on the basic properties, we further derive them to obtain advanced properties, which are more abstract and fit more complicated functionalities. Precisely, we infer new properties based on properties containing Mapping Types variables, which are essential for implementing contract functionalities. We use the example in Fig. 2 to explain the inference procedure. Assume that Comparison pattern constructs a property within one execution trace:

$$
\begin{array}{c} s a f t A m t = P o s t (t o k e n [ a d d r \_ A ] [ a d d r \_ B ]) \\ - P r e (t o k e n [ a d d r \_ A ] [ a d d r \_ B ]) \end{array}\tag{1}
$$

where addr A and addr B are the values of address type. And, the variables in the execution trace satisfy:

$$
a s t = a d d r \_ A; \quad m s g. s e n d e r = a d d r \_ B\tag{2}
$$

In this case, the Comparsion property (1) only represents the relation between safeAmt and a normal token balance. Given the variables in (2), we could infer that the addr A and addr B in property (1) are decided by the parameter ast and the sender of transaction msg.sender, and derive a new property as follows:

$$
\begin{array}{c} s a f t A m t = P o s t (t o k e n [ a s t ] [ m s g. s e n d e r ]) \\ - P r e (t o k e n [ a s t ] [ m s g. s e n d e r ]) \end{array}\tag{3}
$$

In this way, we replace a concrete value with a symbolic value to construct a new abstract property that may be suitable for more complicated contract functionalities. Furthermore, considering that some properties can be inferred many times, we iteratively perform this advanced inference on properties until no new properties are constructed.

## 4.2.2 Mining Strategy

Based on property construction, we heuristically propose a threshold mechanism to improve the robustness of invariant mining.

Threshold. Normally, we should only reserve properties that satisfy all historical transactions as likely invariants [23]. However, we cannot ensure that all historical transactions are normal; that is, the transaction is benign and executed normally without any exception. In contrast, abnormal cases hiding in historical transactions may interfere with mining invariants. For example, some contracts had been attacked due to the abnormal state maintained by the manager but are still active after being recovered, or only emit event Failure but do not revert when an exception occurs. As a result, we may fail to recognize invariants to distinguish normal transactions from abnormal transactions (e.g., attacks) that trigger vulnerabilities.

```javascript
Algorithm 1: Workflow of Invariant Mining.
input : traces, threshold
output: invariants
/* Property Searching */
candiInvs := mapping();
for tr in traces do
    basicProperties := patternDetection(tr);
    for basicProperty in basicProperties do
        candiInvs[basicProperty] += 1;
        advancedProperties := [];
        oldPs := [basicProperty];
        for len(oldPs) > 0 do
            newPs := inference(oldPs, tr);
            oldPs = newPs;
            advancedProperties.extend(newPs);
        for property in advancedProperties do
            candiInvs[property] += 1;

/* Invariant Filtering */
for inv in candiInvs do
    if candiInvs[inv] < threshold * len(traces)
        then
            candiInvs.pop(inv);
invariants := remoevRedundant(candiInvs);
```

To this end, we heuristically use a hyperparameter threshold ∈ (0, 1] to increase the robustness of the mining strategy. Specifically, we decrease threshold and consider properties that satisfy threshold percentage of historical transactions as invariants, which can reduce the interference of hidden abnormal cases in historical transactions.

However, since decreasing threshold means loss of confidence, SmartOracle may mine biased or false invariants and report false positives in detection results. Therefore, we adjust threshold to suit different practical requirements. For example, we increase threshold when we need more precise invariants as oracles.

Algorithm. We illustrate the invariant mining workflow as described in Algorithm 1. The key procedures are as follows:

1) Property Searching. For each trace in traces, we use pattern-based property detection to construct basicProperties. For each property in basicProperties, we perform an iterative advanced inference to obtain advancedProperties. Both serve as candidate invariants (candiInvs).

2) Property Filtering. For each candidate invariant, we count the number of traces that satisfy the invariant and discard the candidate invariants that cannot be satisfied by threshold of traces. Finally, we remove redundant candidate invariants and set the remaining cases as likely invariants.

In particular, since we solely search for properties for each transaction and reserve those that satisfy threshold percentage of all historical transactions, this algorithm is deterministic without randomness.

## 4.3 Invariant Checker

Based on the mined likely invariants (invariants), SmartOracle detects smart contract vulnerabilities by hunting the violations of invariants within transactions. Given each transaction in new txs, SmartOracle first parses the transactions to obtain its execution trace. Then SmartOracle finds the corresponding most fine-grained set of invariants (checked invs), the detailed procedure is as follows: (1) if there is the corresponding branch-layer invariant set, using it as checked invs; otherwise (2) if there is the corresponding function-layer invsariant set, using it as checked invs; otherwise (3) using the contract-layer invariant set as checked invs. Finally, SmartOracle checks the satisfaction of checked invs for the trace and reports the violated invariants of the transaction.

## 5 EXPERIMENTS

In this section, we conduct experiments to evaluate the effectiveness of SmartOracle and aim to answer the following research questions:

• RQ1: How effective is SmartOracle in mining smart contract invariants?

• RQ2: How effective is SmartOracle in detecting smart contract vulnerabilities?

• RQ3: What types of invariants does SmartOracle extract from smart contracts?

• RQ4: How efficient is SmartOracle?

## 5.1 RQ1: Effectiveness of Invariant Detection

To evaluate the effectiveness of invariant detection, we utilize a benchmark from InvCon [23], which contains 246 ERC20 contracts and their invariants. Note that ERC20 contracts satisfy the ERC20 standard [33] and largely follow the same set of common invariants. Table 2 demonstrates the common ERC20 invariants [23], representing the basic functionalities of the contract and its three functions (i.e., transfer, approve, and transferFrom). For example, inv#2 describes the relation between the function parameter amt and the state variable balances during token transfer. To this end, we apply SmartOracle to this benchmark and use these seven common invariants to evaluate the effectiveness of invariant detection. Furthermore, we run SmartOracle with the same input as InvCon to ensure a fair comparison. Table 3 shows the statistical results of the experiments, #N means the number of detected likely invariants, and #TP means the number of invariants detected correctly. To ensure the correctness of the results, we perform manual verification of the detected invariants based on the labels in the benchmark.

## 5.1.1 Ability of Invariant Detection

Since InvCon only extracts invariants that are satisfied in all transactions, we also run SmartOracle with the corresponding hyperparameter threshold = 1. Overall, SmartOracle detects at least one ERC20 invariant in 167 unique contracts, and a total of 518 likeyly invariants (50% $ { \bigl ( } { \frac { 5 1 8 - 2 5 6 } { 5 1 8 } }  { \bigr ) }$ ) more than those detected by InvCon). For function transfer, the likely invariants detected by SmartOracle are much more than those detected by InvCon. This is mainly because InvCon only focuses on function invariants, while SmartOracle can handle more fine-grained invariants $( \mathrm { i . e . , }$ branch invariants). As shown in Fig. 1, the ERC20 Inv#3 is only satisfied for the branch with a specific condition (line 7) but not for the whole function so that this invariant can be detected by SmartOracle but not by InvCon. In addition, SmartOracle only detects a few Inv#3,#4,#5 invariants due to the lack of transactions calling function transferFrom. However, since many ERC20 contracts involve only very few historical transactions, SmartOracle cannot detect their ERC20 invariants.

TABLE 2: Common ERC20 invariants.

<table><tr><td></td><td>ID</td><td>Invariant</td></tr><tr><td>Contract</td><td>Inv#1</td><td>SUM(balances) = totalSupply</td></tr><tr><td rowspan="2">transfer</td><td>Inv#2</td><td>amt = Pre(balances[msg.sender]) -Post(balances[msg.sender])</td></tr><tr><td>Inv#3</td><td>amt = Post(balances[to]) - Pre(balances[to])</td></tr><tr><td>approve</td><td>Inv#4</td><td>amt = Post(allowance[msg.sender][spender])</td></tr><tr><td rowspan="3">transferFrom</td><td>Inv#5</td><td>amt = Post(allowance[from][msg.sender]) -Pre(allowance[from][msg.sender])</td></tr><tr><td>Inv#6</td><td>amt = Pre(balances[from]) - Post(balances[from])</td></tr><tr><td>Inv#7</td><td>amt = Post(balances[to]) - Pre(balances[to])</td></tr></table>

TABLE 3: Statistics about the detected ERC20 invariants.

<table><tr><td rowspan="3">Inv (num.)</td><td rowspan="2" colspan="2">InvCon</td><td colspan="6">SmartOracle</td></tr><tr><td colspan="2">threshold=1.0</td><td colspan="2">threshold=0.9</td><td colspan="2">threshold=0.8</td></tr><tr><td>#TP</td><td>#N</td><td>#TP</td><td>#N</td><td>#TP</td><td>#N</td><td>#TP</td><td>#N</td></tr><tr><td>Inv#1 (233)</td><td>121</td><td>123</td><td>126</td><td>126</td><td>126</td><td>126</td><td>126</td><td>126</td></tr><tr><td>Inv#2 (225)</td><td>39</td><td>45</td><td>160</td><td>169</td><td>169</td><td>181</td><td>170</td><td>183</td></tr><tr><td>Inv#3 (225)</td><td>20</td><td>22</td><td>125</td><td>134</td><td>132</td><td>143</td><td>133</td><td>145</td></tr><tr><td>Inv#4 (244)</td><td>49</td><td>50</td><td>58</td><td>58</td><td>62</td><td>62</td><td>63</td><td>63</td></tr><tr><td>Inv#5 (228)</td><td>2</td><td>2</td><td>9</td><td>9</td><td>9</td><td>9</td><td>9</td><td>9</td></tr><tr><td>Inv#6 (219)</td><td>8</td><td>9</td><td>12</td><td>13</td><td>12</td><td>13</td><td>12</td><td>13</td></tr><tr><td>Inv#7 (219)</td><td>4</td><td>5</td><td>9</td><td>9</td><td>9</td><td>9</td><td>9</td><td>9</td></tr><tr><td>All</td><td>243</td><td>256</td><td>499</td><td>518</td><td>519</td><td>543</td><td>522</td><td>548</td></tr></table>

## 5.1.2 Impact of Threshold

To evaluate the impact of threshold in mining likely invariants, we run SmartOracle with hyperparameter $( t h r e s h o l d = 1 . 0 , 0 . 9 , 0 . 8 )$ and show the results in Table 3. As the threshold decreases from 1.0 to 0.8, the confidence of likely invariants decreases, and the precision rate of detected likely invariants slightly decreases from 96% to 95%. However, the total number of true detected invariants increases by approximately $5 \% ( \frac { 5 2 2 - 4 9 9 } { 4 9 9 } )$ ). Therefore, slightly decreasing the threshold can effectively detect more invariants with an acceptable loss of precision. These experimental results demonstrate the usefulness of our design threshold mechanism in practice applications.

## 5.1.3 Impact of Transactions

As SmartOracle mines likely invariants from transactions, the number of transactions may affect its effectiveness. To investigate the impact, we select 17 ERC20 contracts that involve more than 2,000 transactions, ensuring sufficient data for further evaluation. With threshold=1, we use SmartOracle to mine invariants from the earliest $( N = 2 0 0 , . . . , 2 0 0 0 )$ transactions of contracts, the experimental results are shown in Fig. 5.

The total number of invariants and the number of invariants in each layer show the same growth trend. Initially, these few transactions (e.g., 200) cover only a few functionalities. As the amount of transactions increases, the mined number of mined invariants increases rapidly and eventually levels off. After reaching 1400, the number of invariants decreases slightly as more transactions enable SmartOracle to remove biased or false invariants. In particular, the number of branch invariants is 7-13% more than that of function invariants, indicating that branch invariants can accommodate more fine-grained contract properties.

![](images/cb494c67829234a087215a19b0d0603018e5e564a151cca5dffa394e85c3863d.jpg)  
Fig. 5: Average number of detected invariants along transactions.

## 5.2 RQ2: Effectiveness of Vulnerability Detection

To detect smart contract vulnerabilities with invariants, we need to mine the essential invariants. These invariants should be satisfied with respect to benign behavior, but their violation implies malicious behavior (e.g., triggering vulnerabilities). First, we build a benchmark that contains several real-world vulnerabilities. Then, we apply SmartOracle to this benchmark and evaluate its effectiveness in mining essential invariants for vulnerability detection.

## 5.2.1 Dataset

We reference and expand on an existing dataset [34] by investigating real-world security incidents on Ethereum. Specifically, we collect the security incidents between January 2020 and June 2022 in two well-known online libraries, namely Slowmist [4] and Rekt [35]. From these sources, we exclude incidents related to social engineering (e.g., rug pull and private-key leak) and focus solely on smart contracts that were attacked due to vulnerabilities.

Our dataset contains 65 vulnerable smart contracts from real-world DApps, with an average of 1,973 lines of code (LOC) and associated with over 1.6 million related transactions. According to previous work [26], we manually classify these vulnerabilities, including 7 Reentrancy (RE), 11 Privilege Escalation (PE), 17 Price Manipulation (PM) and 30 Business Logic Flaw (BLF).

## 5.2.2 Experimental Setup

We divide the related transactions of the contract into two parts: (i) transactions that occurred before the attack (i.e., historical txs) and (ii) transactions of attacks and the subsequent 1,000 transactions (i.e., new txs). Given historical txs, we use SmartOracle to mine likely invariants with hyperparameter $( t h r e s h o l d \ = \ 0 . 9 0 , 0 . 9 2 , . . . , 1 . 0 0 )$ . Then, SmartOracle utilizes mined invariants to reveal the transactions violating invariants in new txs (a total of 14,973 transactions).

TABLE 4: The results of SmartOracle with different threshold.

<table><tr><td rowspan="2">threshold</td><td colspan="3">Contract</td><td colspan="2">Transaction</td></tr><tr><td>#TPs</td><td>#TP&amp;FPs</td><td>#FPs</td><td>#TPs</td><td>#FPs</td></tr><tr><td>0.90</td><td>36</td><td>10</td><td>13</td><td>575</td><td>326</td></tr><tr><td>0.92</td><td>36</td><td>8</td><td>11</td><td>575</td><td>272</td></tr><tr><td>0.94</td><td>34</td><td>6</td><td>10</td><td>555</td><td>62</td></tr><tr><td>0.96</td><td>32</td><td>4</td><td>7</td><td>515</td><td>24</td></tr><tr><td>0.98</td><td>31</td><td>3</td><td>5</td><td>466</td><td>17</td></tr><tr><td>1.00</td><td>29</td><td>3</td><td>5</td><td>312</td><td>15</td></tr></table>

## 5.2.3 Evaluation of SmartOracle

The experimental results are shown in Table 4. For transactions that violate mined invariants (namely violated transactions), we mark those indeed caused by vulnerabilities as True Positive (TP); otherwise, mark them as False Positive (FP). In terms of smart contracts, if a contract contains TP violated transactions, we label the contract as TP, and vice versa. Particularly, if SmartOracle detects TP and FP violated transactions in the contract at the same time, we mark it as TP&FP.

Impact of Threshold. When threshold = 0.90, SmartOracle reports many FP violated transactions since SmartOracle mines biased invariants that lead to false positives. As the hyperparameter threshold increases from 0.90 to 0.98, the confidence of invariants increases, and the number of false positives decreases. In particular, increasing threshold also results in a decrease in true positives, since abnormal cases in historical txs interfere with the mining invariants of normal transactions. Fortunately, compared to false positives $( 6 2 \% = \frac { 1 3 - 5 } { 1 3 } )$ , the reduction of true positives $( 1 4 \% = \frac { 3 6 - 3 1 } { 3 6 } )$ is worth and acceptable.

True Positive. In the context of threshold = 0.98, SmartOracle totally detects 483 violated transactions, 96% ( <sup>466</sup><sub>483</sub> ) 483 of which are true positives (i.e., caused by smart contract vulnerabilities). Regarding contracts, SmartOracle successfully reveals true vulnerabilities in 31 vulnerable contracts, 28 of which contain only TP violated transactions. Moreover, as shown in Table 5, most detected vulnerabilities are associated with contract functionalities, such as 6 PM, 6 PE, and 14 BLF vulnerabilities. These results prove the feasibility of mining likely invariants from historical transactions as application-specific oracles for vulnerability detection. SmartOracle can effectively detect smart contract vulnerabilities with an acceptable precision rate.

False Positive. SmartOracle also reports 5 FP vulnerable contracts. However, these vulnerable contracts only involve 17 FP violated transactions. After manual analysis, we find that these violated transactions are mainly caused by atypical scenarios within the smart contracts. These scenarios are benign but have not occurred in historical txs, which makes SmartOracle mine biased invariants and report false positives. Fortunately, these atypical cases are easily distinguished from TP violated transactions with very little manual effect.

False Negative. However, there are still some vulnerable contracts that SmartOracle cannot detect. We summarize the main reasons as follows: (1) before being attacked, the vulnerable function had been invoked with too few transactions (even no transaction) that cover limited functionalities, so SmartOracle could not mine effective invariants for detection; (2) the smart contract vulnerabilities violate the invariants that involve the variables across multiple contracts. For example, exploiting some Price Manipulation requires manipulating the state variables in other contracts (e.g., the variables in price oracle contract [36]). In these cases, SmartOracle cannot handle the variables in other contracts and mine the corresponding invariant for vulnerability detection. Since handling multiple contracts would involve significant computational overhead on SmartOracle, we do not deal with this kind of case in this work.

TABLE 5: The experimental results of evaluating SmartOracle and other tools on our benchmark.

<table><tr><td rowspan="2">Tool</td><td colspan="2">PE (11)</td><td colspan="2">PM (17)</td><td colspan="2">RE (7)</td><td>BLF (30)</td></tr><tr><td>#TPs</td><td>#FPs</td><td>#TPs</td><td>#FPs</td><td>#TPs</td><td>#FPs</td><td>#TPs</td></tr><tr><td>SmartOracle</td><td>6</td><td>/</td><td>6</td><td>/</td><td>5</td><td>/</td><td>14</td></tr><tr><td>AChecker</td><td>0</td><td>0</td><td>/</td><td>/</td><td>/</td><td>/</td><td>/</td></tr><tr><td>DeFiTainter</td><td>/</td><td>/</td><td>6</td><td>0</td><td>/</td><td>/</td><td>/</td></tr><tr><td>Slither</td><td>/</td><td>/</td><td>/</td><td>/</td><td>4</td><td>39</td><td>/</td></tr><tr><td>Sailfish</td><td>/</td><td>/</td><td>/</td><td>/</td><td>1</td><td>8</td><td>/</td></tr></table>

## 5.2.4 Comparing SmartOracle with existing tools

We also adapt existing tools to our collected dataset for comparison. Based on the vulnerability types in the vulnerable contracts, we select Slither [14], Sailfish [11] for Reentrancy, AChecker [37] for Privilege Escalation, and DeFiTainter [36] for Price Manipulation. For 65 vulnerable contracts, we reference previous work [34] and set the analysis timeout of tools to 30 minutes for each smart contract. SmartOracle performs an analysis with threshold = 0.98. Table 5 shows the detection results; True positive (TP) means that the tool detects the true vulnerability, while False positive (FP) means that the tool reports the vulnerability that the contract does not contain.

For Reentrancy, Slither reveals 4 true positives but reports 39 false positives, while Sailfish detects 1 true positive alongside 8 false positives. The high false-positive rates question their practicality in real-world applications. For Privilege Escalation, AChecker fails to reveal any vulnerabilities. It suffers from out-of-memory errors in many contracts, indicating that AChecker is impractical in realworld complex DApp contracts (e.g., with thousands of lines of code). For Price Manipulation, SmartOracle is close to DeFiTainter in effectiveness. In addition to the above three vulnerabilities, SmartOracle detects other 14 Business Logic Flaw vulnerabilities, covering a broader scope of vulnerabilities in real-world security incidents. Overall, existing tools have limitations in detecting smart contract vulnerabilities in real-world DApps, while SmartOracle can provide more effective protection by run-time verification.

## 5.3 RQ3: Invariants of Smart Contracts

In this experiment, we study the distributions of the smart contract invariants extracted by SmartOracle. We count the proportion of variables (construction ways) in invariants mined from historical txs (namely mined invariants) and invariants violated in new txs (namely violated invariants), respectively.

![](images/a36e249df28cdf491ed095bbdb3f6fcf50c9b6ef7fac99aae0f549b5e45593c0.jpg)  
Fig. 6: Proportions of variables in the invariants mined from historical transactions (mined invariants) and the invariants violated in new transactions (violated invariants).

![](images/717617c7b4b2dc58fcc8bdab202f807fde8b6ebb92fbc1194e2e09a59c33bbc4.jpg)  
Fig. 7: Proportions of construction in the invariants mined from historical transactions (mined invariants) and the invariants violated in new transactions (violated invariants).

Variables in invariants. Fig. 6 shows the proportion of each variable (listed in Table 1) in invariants extracted by SmartOracle. Notably, invariants involving token variables constitute the highest proportion in both mined invariants (59%) and violated invariants (43%). This is because token variable is the essential element of many business operations on Ethereum, especially of common financial servers. About 19% of the mined and violated invariants involve state-variable, which are the basis of smart contracts to record states and participate in most functionalities of smart contracts. In addition, function and event variables occupy similar proportions in the violated invariants.

Construction of invariants. Fig. 7 shows the proportion of invariant construction extracted by SmartOracle. We find that each construction has a similar proportion between the mined invariants and the violated invariants. For the basic invariants constructed by pattern-based detection, the invariants involving Comparison occupy the most significant proportion since Comparison is the most common property in smart contracts. However, since there are very few Array variables in smart contracts, Membership only constructs very few invariants. In addition, Arithmetic pattern contains the minimal part of the invariants, but the invariants still successfully reveal some smart contract vulnerabilities. Inference means the invariants constructed by advanced inference; it occupies approximately 30% of all invariants, indicating the great effectiveness of our designed advanced inference in constructing invariants.

## 5.4 RQ4: Efficiency of SmartOracle

In this RQ, we evaluate the efficiency of SmartOracle in real-time detection. Specifically, we perform SmartOracle on related transactions of smart contracts, involving 31,184 transactions for mining and 14,973 transactions for checking. We record the average execution time per transaction of each procedure, including Transaction Parser, Invariant Miner, and Invariant Checker.

TABLE 6: The average time consumption of analyzing per transaction. Total Checking Time is the sum of time consumption of Transaction Parser and Invariant Checker.

<table><tr><td>Action</td><td>Average Time ( $\times 10^{-3}$  second)</td></tr><tr><td>Transaction Parser</td><td>44.66</td></tr><tr><td>Invariant Miner</td><td>37.96</td></tr><tr><td>Invariant Checker</td><td>7.02</td></tr><tr><td>Total Checking Time</td><td>51.68 (44.66 + 7.02)</td></tr></table>

The experimental results are presented in Table 6. We only carry out Invariant Miner once for each smart contract, then use the mined invariants for bug detection. Therefore, in the checking scenario, the Total Checking Time is the sum of the time consumption of Transaction Parser and Invariant Checker. SmartOracle checks each transaction with $5 1 . 6 8 \times 1 0 ^ { - 3 }$ seconds in average. In particular, SmartOracle’s TPS $\begin{array} { r } { ( 1 9 . 3 = \frac { 1 0 0 0 } { 5 1 . 6 8 } ) } \end{array}$ is higher than that of Ethereum (approximately 12.0) [38], which means that SmartOracle has the ability to perform real-time audits and promptly reveal violated transactions as they occur. Furthermore, for bugs that require multiple transactions to be exploited, SmartOracle can immediately hunt the violated transactions before the whole exploit is finished. For example, the entire exploit of bZx DApp [39] involves multiple transactions, which span nearly 8 minutes from the first violated transaction detected by SmartOracle to the final profitable transaction. Therefore, if we could reveal the first violated transaction and implement emergency protective measures, we could prevent the attack and rescue a substantial amount of funds.

## 6 DISCUSSION

## 6.1 Vulnerability and Invariant

This section discusses the scope of smart contract vulnerabilities detected by SmartOracle and their violated invariants.

Reentrancy [11] is a common smart contract vulnerability that allows attackers to re-enter the contract for stealing funds during external calls. However, re-entering the contract may violate the invariants between Pre-SubCall and Post-SubCall of the external call.

Pribilege Escalation [37] means the contract lacks a complete access control check. Since many contracts use the whitelist to record privileged users, triggering this bug may violate the invariant msg.sender ∈ whiteList.

Price Manipulation [36] is always caused by the poor implementation of the price oracle, which allows the attacker to manipulate the token price to make a profit. Exploiting this bug would abnormally change the token balances and violate their related invariants.

Business Logic Flaw strongly relies on the specific contract functionalities. SmartOracle can mine the essential invariants that fit the complicated contract functionalities, which serve as application-specific oracles for vulnerability detection. As in the example shown in Section 3.1, SmartOracle mines the essential invariant between the function parameter and the change of token balance from benign transactions, which are violated by the attack transactions triggering the vulnerability.

## 6.2 Threats to Validity

External Validity. The invariant mining procedure relies on the historical transactions of smart contracts, which prevents its application to the contract without any transactions. However, in our dataset, we find that about 80% $\textstyle { \big ( } { \frac { 5 2 } { 6 5 } } { \big ) }$ contracts had over 100 transactions before being attacked, and $4 9 \% ( \frac { 3 2 } { 6 5 } )$ contracts had more than 1000 transactions, indicating that there are adequate historical transactions for invariant mining. Furthermore, the unit tests designed by developers can also provide transactions for invariant mining. Both indicate that our approach is available in most scenarios.

Internal Validity. Manually collecting real-world incidents, locating their vulnerable contracts, and related attack transactions require significant labor, which may involve some mistakes. To ensure accuracy, all manual processes were performed by at least two experienced researchers. Additionally, although the dataset only contains 65 vulnerable contracts, they are representative and diverse from realworld DApps, so this dataset can evaluate the practical effectiveness of tools.

## 7 RELATED WORK

## 7.1 Vulnerability Detection of Smart Contracts

Numerous approaches have been proposed to detect smart contract vulnerabilities. Some approaches use general oracles to detect vulnerabilities [2], [6], [14], [36], [40]. Luu et al. [2] originally proposed a symbolic executor for smart contracts, namely Oyente, that detects vulnerabilities with low-level and general oracles. Zhou et al. [40] manually extract patterns from known adversarial transactions, based on which they reveal the adversarial transactions by matching the logs with the patterns. They demonstrate great performance in revealing implementation vulnerability (e.g., integer overflow).

Meanwhile, some approaches use user-specific oracles to verify smart contracts [17], [27]. For example, Duan et al. [17] proposed VETSC that utilizes model checking to vet smart contracts based on user-provided specifications. They can effectively detect vulnerabilities related to contract functionalities but rely on manual effort. In addition, Liu et al. [28] proposed SPCon, which mines the roles of addresses from transactions and detects permission bugs in smart contracts. Similarly, SmartOracle also extracts rules from transactions, but for more kinds of vulnerabilities other than permission bugs.

Difference. Most existing tools use general or user-specified oracles to deal with vulnerabilities, which face ineffective detection or require too much manual effort. On the contrary, SmartOracle automatically extracts likely invariants as application-specific oracles, which are suitable for detecting various kinds of vulnerability with complicated functionalities.

## 7.2 Invariant Detection

Due to its efficiency and scalability, invariant detection has been widely used in software testing and software verification [41]–[43]. Among them, Daikon [21] is a commonly used basic approach that dynamically extracts likely invariants from running programs with a pattern-based inference engine. Based on Daikon, various approaches related to invariant detection are proposed [44]–[47]. For example, Perkins et al. [47] proposed an incremental algorithm to improve the efficiency and scalability of dynamic invariant detection. Liu et al. [23] proposed a dynamic invariant detector for Ethereum smart contracts based on Daikon, namely InvCon. However, InvCon is only suitable to extract the likely invariants within ERC20 token contracts, which limits its usage. Wang et al. proposed SmartInv [26], which uses a Large Language Model (LLM) to construct contract invariants based on source code and code comments. SmartInv requires labeled contracts to fine-tune the model. However, code comments may not be reliable or inconsistent with the code, which would interfere with the analysis.

Difference. Compared to existing approaches, SmartOracle is a more suitable and practical invariant detector for smart contracts. SmartOracle extracts more fine-grained invariants, uses a threshold mechanism to improve usefulness, and provides automated run-time verification. Thus, SmartOracle has more ability to handle various smart contract vulnerabilities.

## 8 CONCLUSION

In this paper, we propose an automatic invariant detector, namely SmartOracle, that generates fine-grained invariants as application-specific oracles for detecting smart contract vulnerabilities. Specifically, SmartOracle first parses the smart contract and its transactions to extract execution traces. Then, SmartOracle utilizes a threshold-based algorithm to mine likely invariants, based on which it detects smart contract bugs by checking whether the transactions violate the invariants. In experiments, SmartOracle outperforms existing dynamic invariant detection, detects 50% more ERC20 invariants and achieves 96% precision. Furthermore, SmartOracle shows its effectiveness in generating application-specific oracles for detecting real-world vulnerabilities. It detects 31 vulnerable contracts and their related 466 transactions that violated mined invariants.

## REFERENCES

[1] Z. Zheng, S. Xie, H.-N. Dai, W. Chen, X. Chen, J. Weng, and M. Imran, “An overview on smart contracts: Challenges, advances and platforms,” Future Generation Computer Systems, vol. 105, pp. 475–491, 2020.

[2] L. Luu, D.-H. Chu, H. Olickel, P. Saxena, and A. Hobor, “Making smart contracts smarter,” in Proceedings of the 2016 ACM SIGSAC Conference on Computer and Communications Security, ser. CCS ’16. New York, NY, USA: Association for Computing Machinery, 2016, p. 254–269. [Online]. Available: https://doi.org/10.1145/2976749.2978309

[3] S. Wu, D. Wang, J. He, Y. Zhou, L. Wu, X. Yuan, Q. He, and K. Ren, “Defiranger: Detecting price manipulation attacks on defi applications,” arXiv preprint arXiv:2104.15068, 2021.

[4] “Slowmist hacked events,” 2022. [Online]. Available: https: //hacked.slowmist.io/

[5] P. Tsankov, A. Dan, D. Drachsler-Cohen, A. Gervais, F. Bunzli,¨ and M. Vechev, “Securify: Practical security analysis of smart contracts,” in Proceedings of the 2018 ACM SIGSAC Conference on Computer and Communications Security, ser. CCS ’18. New York, NY, USA: Association for Computing Machinery, 2018, p. 67–82. [Online]. Available: https://doi.org/10.1145/3243734.3243780

[6] J. Su, H.-N. Dai, L. Zhao, Z. Zheng, and X. Luo, “Effectively generating vulnerable transaction sequences in smart contracts with reinforcement learning-guided fuzzing,” in 37th IEEE/ACM International Conference on Automated Software Engineering, ser. ASE22. New York, NY, USA: Association for Computing Machinery, 2023. [Online]. Available: https://doi.org/10.1145/ 3551349.3560429

[7] W. Wang, W. Huang, Z. Meng, Y. Xiong, F. Miao, X. Fang, C. Tu, and R. Ji, “Automated inference on financial security of ethereum smart contracts,” in 32nd USENIX Security Symposium (USENIX Security 23). Anaheim, CA: USENIX Association, Aug. 2023, pp. 3367–3383. [Online]. Available: https://www.usenix. org/conference/usenixsecurity23/presentation/wang-wansen

[8] Z. Zhang, B. Zhang, W. Xu, and Z. Lin, “Demystifying exploitable bugs in smart contracts.” ICSE, 2023.

[9] L. Su, X. Shen, X. Du, X. Liao, X. Wang, L. Xing, and B. Liu, “Evil under the sun: Understanding and discovering attacks on ethereum decentralized applications.” in USENIX Security Symposium, 2021, pp. 1307–1324.

[10] J. Choi, D. Kim, S. Kim, G. Grieco, A. Groce, and S. K. Cha, “Smartian: Enhancing smart contract fuzzing with static and dynamic data-flow analyses,” in 2021 36th IEEE/ACM International Conference on Automated Software Engineering (ASE), 2021, pp. 227– 239.

[11] P. Bose, D. Das, Y. Chen, Y. Feng, C. Kruegel, and G. Vigna, “Sailfish: Vetting smart contract state-inconsistency bugs in seconds,” in 2022 IEEE Symposium on Security and Privacy (SP), 2022, pp. 161– 178.

[12] T. D. Nguyen, L. H. Pham, J. Sun, Y. Lin, and Q. T. Minh, “Sfuzz: An efficient adaptive fuzzer for solidity smart contracts,” in Proceedings of the ACM/IEEE 42nd International Conference on Software Engineering, ser. ICSE ’20. New York, NY, USA: Association for Computing Machinery, 2020, p. 778–788. [Online]. Available: https://doi.org/10.1145/3377811.3380334

[13] M. Zhang, X. Zhang, Y. Zhang, and Z. Lin, “Txspector: Uncovering attacks in ethereum from transactions,” in Proceedings of the 29th USENIX Conference on Security Symposium, ser. SEC’20. USA: USENIX Association, 2020.

[14] J. Feist, G. Grieco, and A. Groce, “Slither: a static analysis framework for smart contracts,” in 2019 IEEE/ACM 2nd International Workshop on Emerging Trends in Software Engineering for Blockchain (WETSEB). IEEE, 2019, pp. 8–15.

[15] B. Jiang, Y. Liu, and W. K. Chan, “Contractfuzzer: Fuzzing smart contracts for vulnerability detection,” in Proceedings of the 33rd ACM/IEEE International Conference on Automated Software Engineering, ser. ASE ’18. New York, NY, USA: Association for Computing Machinery, 2018, p. 259–269. [Online]. Available: https://doi.org/10.1145/3238147.3238177

[16] Z. Zheng, N. Zhang, J. Su, Z. Zhong, M. Ye, and J. Chen, “Turn the rudder: A beacon of reentrancy detection for smart contracts on ethereum,” in Proceedings of the ACM/IEEE 45th International Conference on Software Engineering. ICSE, 2023.

[17] Y. Duan, X. Zhao, Y. Pan, S. Li, M. Li, F. Xu, and M. Zhang, “Towards automated safety vetting of smart contracts in decentralized applications,” in Proceedings of the 2022 ACM SIGSAC Conference on Computer and Communications Security, ser. CCS ’22. New York, NY, USA: Association for Computing Machinery, 2022, p. 921–935. [Online]. Available: https://doi.org/10.1145/3548606.3559384

[18] G. Grieco, W. Song, A. Cygan, J. Feist, and A. Groce, “Echidna: Effective, usable, and fast fuzzing for smart contracts,” in Proceedings of the 29th ACM SIGSOFT International Symposium on Software Testing and Analysis, ser. ISSTA 2020. New York, NY, USA: Association for Computing Machinery, 2020, p. 557–560. [Online]. Available: https://doi.org/10.1145/3395363.3404366

[19] Y. Wang, S. K. Lahiri, S. Chen, R. Pan, I. Dillig, C. Born, I. Naseer, and K. Ferles, “Formal verification of workflow policies for smart contracts in azure blockchain,” in Verified Software. Theories, Tools, and Experiments: 11th International Conference, VSTTE 2019, New York City, NY, USA, July 13–14, 2019, Revised Selected Papers 11. Springer, 2020, pp. 87–106.

[20] “Thorchain hacked,” 2021. [Online]. Available: https://medium. com/thorchain/eth-parsing-error-and-exploit-3b343aa6466f

[21] M. D. Ernst, J. H. Perkins, P. J. Guo, S. McCamant, C. Pacheco, M. S. Tschantz, and C. Xiao, “The daikon system for dynamic detection of likely invariants,” Science of computer programming, vol. 69, no. 1-3, pp. 35–45, 2007.

[22] “Qubit bridge hacked event,” 2022. [Online]. Available: https://certik.medium.com/ qubit-bridge-collapse-exploited-to-the-tune-of-80-million-a7ab90

[23] Y. Liu and Y. Li, “Invcon: A dynamic invariant detector for ethereum smart contracts,” in Proceedings of the 37th IEEE/ACM International Conference on Automated Software Engineering, ser. ASE ’22. New York, NY, USA: Association for Computing Machinery, 2023. [Online]. Available: https://doi.org/10.1145/ 3551349.3559539

[24] “Solidity documentation,” 2022. [Online]. Available: https: //docs.soliditylang.org/en/v0.8.16/

[25] “Ethereum yellow paper: a formal specification of ethereum, a programmable blockchain,” 2022. [Online]. Available: https: //ethereum.github.io/yellowpaper/paper.pdf

[26] S. Wang, K. Pei, and J. Yang, “Smartinv: Multimodal learning for smart contract invariant inference,” in 2024 IEEE Symposium on Security and Privacy (SP). Los Alamitos, CA, USA: IEEE Computer Society, may 2024, pp. 125–125. [Online]. Available: https://doi.ieeecomputersociety.org/10.1109/SP54263.2024.00126

[27] A. Permenev, D. Dimitrov, P. Tsankov, D. Drachsler-Cohen, and M. Vechev, “Verx: Safety verification of smart contracts,” in 2020 IEEE Symposium on Security and Privacy (SP), 2020, pp. 1661–1677.

[28] Y. Liu, Y. Li, S.-W. Lin, and C. Artho, “Finding permission bugs in smart contracts with role mining,” in Proceedings of the 31st ACM SIGSOFT International Symposium on Software Testing and Analysis, ser. ISSTA 2022. New York, NY, USA: Association for Computing Machinery, 2022, p. 716–727. [Online]. Available: https://doi.org/10.1145/3533767.3534372

[29] “Go-ethereum documentation,” 2023. [Online]. Available: https: //geth.ethereum.org/docs/interacting-with-geth/rpc/ns-debug

[30] T. Chen, Y. Zhang, Z. Li, X. Luo, T. Wang, R. Cao, X. Xiao, and X. Zhang, “Tokenscope: Automatically detecting inconsistent behaviors of cryptocurrency tokens in ethereum,” in Proceedings of the 2019 ACM SIGSAC Conference on Computer and Communications Security, ser. CCS ’19. New York, NY, USA: Association for Computing Machinery, 2019, p. 1503–1520. [Online]. Available: https://doi.org/10.1145/3319535.3345664

[31] “Go-ethereum,” 2023. [Online]. Available: https: //github.com/ethereum/go-ethereum/blob/master/core/ state transition.go#L181

[32] “Uniswap v2,” 2022. [Online]. Available: https://uniswap.org/

[33] “Erc-20 token standard,” 2022. [Online]. Available: https:// ethereum.org/en/developers/docs/standards/tokens/erc-20/

[34] J. Su, X. Lin, Z. Fang, Z. Zhu, J. Chen, Z. Zheng, W. Lv, and J. Wang, “Defiwarder: Protecting defi apps from token leaking vulnerabilities,” in 2023 38th IEEE/ACM International Conference on Automated Software Engineering (ASE), 2023, pp. 1664–1675.

[35] “Rekt,” 2022. [Online]. Available: https://rekt.news/

[36] Q. Kong, J. Chen, Y. Wang, Z. Jiang, and Z. Zheng, “Defitainter: Detecting price manipulation vulnerabilities in defi protocols,” in Proceedings of the 32nd ACM SIGSOFT International Symposium on Software Testing and Analysis, ser. ISSTA 2023. New York, NY, USA: Association for Computing Machinery, 2023, p. 1144–1156. e1a0 [Online]. Available: https://doi.org/10.1145/3597926.3598124

[37] A. Ghaleb, J. Rubin, and K. Pattabiraman, “Achecker: Statically detecting smart contract access control vulnerabilities,” in Proceedings of the 45th IEEE/ACM International Conference on Software Engineering, 2023.

[38] “Etherscan,” 2023. [Online]. Available: https://etherscan.io/

[39] “bzx network hacked,” 2020. [Online]. Available: https://www.coindesk.com/markets/2020/09/14/ defi-lender-bzx-loses-8m-in-third-attack-this-year/

[40] S. Zhou, Z. Yang, J. Xiang, Y. Cao, M. Yang, and Y. Zhang, “An ever-evolving game: Evaluation of real-world attacks and defenses in ethereum ecosystem,” in Proceedings of the 29th USENIX Conference on Security Symposium, ser. SEC’20. USA: USENIX Association, 2020.

[41] S. K. Sahoo, J. Criswell, C. Geigle, and V. Adve, “Using likely invariants for automated software fault localization,” in Proceedings of the eighteenth international conference on Architectural support for programming languages and operating systems, 2013, pp. 139–152.

[42] P. He, C. Meister, and Z. Su, “Structure-invariant testing for machine translation,” ser. ICSE ’20. New York, NY, USA: Association for Computing Machinery, 2020, p. 961–973. [Online]. Available: https://doi.org/10.1145/3377811.3380339

[43] A. Fioraldi, D. C. D’Elia, and D. Balzarotti, “The use of likely invariants as feedback for fuzzers,” in 30th USENIX Security Symposium (USENIX Security 21), 2021, pp. 2829–2846.

[44] C. Lemieux, D. Park, and I. Beschastnikh, “General ltl specification mining (t),” in 2015 30th IEEE/ACM International Conference on Automated Software Engineering (ASE). IEEE, 2015, pp. 81–92.

[45] D. Schuler, V. Dallmeier, and A. Zeller, “Efficient mutation testing by checking invariant violations,” in Proceedings of the eighteenth international symposium on Software testing and analysis, 2009, pp. 69–80.

[46] I. Beschastnikh, Y. Brun, S. Schneider, M. Sloan, and M. D. Ernst, “Leveraging existing instrumentation to automatically infer invariant-constrained models,” in Proceedings of the 19th ACM SIGSOFT Symposium and the 13th European Conference on Foundations of Software Engineering, ser. ESEC/FSE ’11. New York, NY, USA: Association for Computing Machinery, 2011, p. 267–277. [Online]. Available: https://doi.org/10.1145/2025113.2025151

[47] J. H. Perkins and M. D. Ernst, “Efficient incremental algorithms for dynamic detection of likely invariants,” vol. 29, no. 6, p. 23–32, oct 2004. [Online]. Available: https://doi.org/10.1145/1041685. 1029901