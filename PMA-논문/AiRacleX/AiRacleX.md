# ��������: Automated Detection of Price Oracle Manipulations via LLM-Driven Knowledge Mining and Prompt Generation

BO GAO, YUAN WANG, QINGSONG WEI, YONG LIU, and RICK SIOW MONG GOH, Institute of

High Performance Computing (IHPC), Agency for Science, Technology and Research (A\*STAR), Singapore

DAVID LO, Singapore Management University, Singapore

Decentralized finance (DeFi) applications depend on accurate price oracles to ensure secure transactions, yet these oracles are highly vulnerable to manipulation, enabling attackers to exploit smart contract vulnerabilities for unfair asset valuation and financial gain. Detecting such manipulations traditionally relies on the manual efort of experienced experts, presenting significant challenges.

In this paper, we propose a novel LLM-driven framework that automates the detection of price oracle manipulations by leveraging the complementary strengths of diferent LLM models (LLMs). Our approach begins with domain-specific knowledge extraction, where an LLM model synthesizes precise insights about price oracle vulnerabilities from top-tier academic papers, eliminating the need for profound expertise from developers or auditors. This knowledge forms the foundation for a second LLM model to generate structured, context-aware chain of thought prompts, which guide a third LLM model in accurately identifying manipulation patterns in smart contracts. We validate the framework’s efectiveness through experiments on 60 known vulnerabilities from 46 real-world DeFi attacks or projects spanning 2021 to 2023. The best performing combination of LLMs (Haiku-Haiku-4o-mini) identified by �������� demonstrate a 2.58-times improvement in recall (0.667 vs 0.259) compared to the state-of-the-art tool GPTScan, while maintaining comparable precision. Furthermore, our framework demonstrates the feasibility of replacing commercial models with open-source alternatives, enhancing privacy and security for developers.

CCS Concepts: • Computing methodologies → Natural language processing; • Security and privacy → Software security engineering; • Software and its engineering → Software maintenance tools.

Additional Key Words and Phrases: LLM, Price Oracle Manipulations, Smart Contract Security, Prompt Engineering

## ACM Reference Format:

Bo Gao, Yuan Wang, Qingsong Wei, Yong Liu, Rick Siow Mong Goh, and David Lo. 2025. ��������: Automated Detection of Price Oracle Manipulations via LLM-Driven Knowledge Mining and Prompt Generation. 1, 1 (February 2025), 24 pages. https: //doi.org/10.1145/nnnnnnn.nnnnnnn

## 1 Introduction

Decentralized finance (DeFi) has emerged as a groundbreaking paradigm, revolutionizing the landscape of traditional finance by ofering open, accessible, and permissionless financial services built on the foundation of blockchain technology. At the core of many DeFi applications lie price oracles, providing essential external price data for smart contracts that power a wide array of financial activities, including lending, borrowing, trading, and more. By delivering

Authors’ Contact Information: Bo Gao, gao\_bo@ihpc.a-star.edu.sg; Yuan Wang; Qingsong Wei; Yong Liu; Rick Siow Mong Goh, Institute of High Performance Computing (IHPC), Agency for Science, Technology and Research (A\*STAR), Singapore, Singapore; David Lo, Singapore Management University, Singapore, Singapore.

accurate up-to-date price information, price oracles enable DeFi protocols to operate efectively, execute fair transactions, and maintain ecosystem stability. However, the critical role of price oracles also makes them a prime target fo exploitation, posing a significant risk to the integrity of the entire ecosystem. Manipulating the price data provided by an oracle to mislead smart contracts about the true value of an asset is referred to price oracle manipulation (POM) [37] Through POM, adversaries can exploit misled smart contracts to gain unfair advantages or illicit profits. These attacks can take various forms, such as using flash loans to temporarily distort asset prices, compromise data sources to feed false information to oracles, influence decentralized voting mechanisms to distort price data, or exploit time-weighted average price (TWAP) calculations to create inaccurate price feeds. Recent studies have highlighted the severity of the issue. For example, Zhou et al. [44] reported that on-chain oracle manipulation incidents are the most common protocol layer incident type, acconting for 15% of the total real-world attacks analyzed. Similarly, Zhang et al.[40] find that price oracle manipulation exploits represent 34.3% of the exploits in their real-world dataset, identifying it as the most common exploit among machine-unauditable bugs (MUBs)—a category of vulnerabilities that, as of 2022, remain undetectable by existing automated tools.

Although recent years have seen advancements in automated security tools, the blockchain community still struggles to efectively address POM. Chaliasos et al. [2] highlight that only 25% of real-world attacks are detected by widely used static analysis tools such as ConFuzzius [31], Mythril [4], Oyente [21], Slither [8], and Solhint [27], underscoring the persistent limitations of existing automated approaches. Few works attempting to address POM include DeFiRanger [37], which identifies price oracle manipulation attacks using pattern matching; ProMutator [32], which simulates potential price manipulation attacks to identify weak points in oracle systems; DeFiPoser [43], which uses SMT solvers to detect complex profitable transactions; and OVer [6], which employs symbolic analysis and SMT solvers to ensure the secure operation of DeFi protocols. These approaches often require extensive transaction data, significant computationa resources, or accurate modeling, limiting their performance and adoption. In practice, this kind of task still heavily relies on the manual eforts of experienced experts by analyzing data and patterns. Thus, these approaches are inherently limited by human capacity and expertise, making it dificult to scale and adapt to the rapidly evolving landscape of DeF and smart contract technologies.

With the rapid development of large language models (LLMs), some works have leveraged LLMs for detecting smar contract vulnerabilities. Early exploration by Issac et al. [5] demonstrated the efectiveness of ChatGPT-4 and Claude in conducting smart contract security audits especially identifying logic issues and coding errors, but they generated a significant number of false positives (95% of alarms). Building upon this, Gao et al.[11] further evaluated ChatGPT-4’s performance across six specific categories of bugs, revealing that it can detect an average of 15% of vulnerabilities using bug-type-agnostic prompts. Notably, while it successfully identified 33% of price oracle manipulation bugs, this came with an 87% false positive rate, highlighting the complex trade-of between recall and precision in LLM-based approaches. These initial eforts use LLMs in a straightforward manner and are more evaluation-oriented, lacking dedicated design for POM. Following these works, GPTLens [13] introduced an adversarial framework with LLMs serving as an Auditor and Critic to improve vulnerability detection. By separating generation and discrimination stages, GPTLens reduces false positives while maintaining recall. However, its models struggle to interpret ground truth efectively, resulting in only marginal improvements in a small evaluation of 13 projects. Notably, GPTScan [29] combines ChatGPT’s code analysis capabilities with static analysis to detect logical vulnerabilities in smart contracts While it achieves high precision and recall on detecting vulnerability types, it struggles with complex vulnerabilities like POM, which require long and intricate function calls. GPTScan also depends heavily on ChatGPT’s output format, which is prone to errors even in streamlined JSON formats, complicating integration with static analysis tools. This Manuscript submitted to ACM

��������: Automated Detection of Price Oracle Manipulations via LLM-Driven Knowledge Mining and Prompt Generation

limitation leads to missed vulnerabilities or false alarms when validating strategies by static analysis tools are loosened. Our evaluation confirmed these challenges, showing increased false positive and false negative rates when GPTScan’s results are assessed with standard metrics. Moreover, its fixed warning messages for each vulnerability type limit interpretability, and the static analysis tools it relies on often fail to handle diverse smart contracts. These studies underscore LLMs’ potential in advancing smart contract security but highlight persistent challenges, including high false positive rates, limited coverage of complex bugs, and dificulty delivering actionable insights for real-world applications.

To overcome the limitations of static analysis tools, our work introduces a novel framework leveraging three LLM models, designated as Knowledge Synthesizer, Prompt Generator and Auditor. The process begins with the Knowledge Synthesizer, an LLM dedicated to the extraction and synthesis of domain-specific insights from top-tier academic literature. This initial step is essential, as it filters out noise introduced by lower-quality data sources—such as online forums, blogs, and miscellaneous webpages—commonly found in training datasets. Moreover, it supplies the pipeline with precise, externally validated domain knowledge. Building on the high-fidelity insights provided by the Knowledge Synthesizer, the Prompt Generator plays a critical role in translating these insights into structured and actionable chain of thought (CoT) prompts. This method, demonstrated to significantly enhance LLM performance in various applications [15], ensures that the Auditor is guided with precise and contextually relevant instructions. Equipped with these tailored prompts, the Auditor enhances the detection of POM across diverse projects. To evaluate the efectiveness of our framework, we conducted experiments on a dataset containing 36 bugs from 31 real-world DeFi attacks between 2021 and 2022, as well as 24 bugs from 15 Code4rena projects spanning 2021 to 2023. Our results demonstrate a 2.58-times improvement in recall (0.667 vs 0.259) compared to the state-of-the-art tool GPTScan, while maintaining comparable precision. Additionally, when compared to the zero-shot CoT prompt, our approach achieves a 15% increase in precision (0.313 vs 0.271) with comparable recall. Moreover, our approach streamlines the process for developers, eliminating the need for domain knowledge in smart contracts or crafting problem-specific prompts. To conclude, we make the following contributions:

• A Novel and Transferable Multi-LLM Framework: We propose a multi-LLM framework that synergistically integrates a domain-specific Knowledge Synthesizer, an optimized Prompt Generator, and an automated Auditor to efectively identify POM vulnerabilities in smart contracts. The Knowledge Synthesizer enables seamless extension to other vulnerabilities without the need for predefined rules or code modifications, as seen in tools like GPTScan, by minimizing reliance on expert knowledge. Meanwhile, the Prompt Generator facilitates the automatic creation of structured prompts, eliminating the need for manual intervention and enhancing eficiency.

• Optimized Model Selection and Knowledge Evolution: We evaluate and identify the optimal combination of LLM models for knowledge summarization, prompt generation, and vulnerability detection. Our results highlight the complementary strengths of diferent models, showcasing the efectiveness of manually curated knowledge in improving detection capabilities. Furthermore, we demonstrate how LLM-based knowledge synthesizers can replicate and surpass human-curated performance, paving the way for fully automated systems.

• Comprehensive Validation and New Discoveries: Through extensive evaluation on diverse real-world datasets, including historical DeFi exploits and Code4rena projects, we validate the efectiveness ofour framework. Notably, our approach successfully identified 20 bugs which can only be detected by �������� but not by SOTA tool GPTScan.

Outline. In the subsequent sections of this paper, we introduce some essential concept in Section 2. Then, we delve deeper into the methodology of our approach, outlining the process of knowledge extraction, prompt generation and Manuscript submitted to ACM automatic audit in Section 3. In Section 4, we present the results of our experiments and discuss the implications of our findings for the broader blockchain and DeFi ecosystems. Section 5 delves into related works and we conclude our paper in Section 6.

## 2 Preliminaries

This section provides some basics about POM, including how price oracle manipulation occurs, the typical POMs and the representative causes of POM vulnerabilities. We assume some familiarity with basic concepts such as blockchain, Ethereum, and smart contracts, and refer readers to [36] for details.

## 2.1 Types of DeFi Applications

DeFi applications aim to provide financial services without traditional intermediaries, leveraging blockchain technology and smart contracts. The main types of DeFi applications include decentralized exchanges (DEXs), stablecoins, lending and borrowing platforms, yield farming and liquidity mining, and decentralized autonomous organizations (DAOs) etc. This work further classifies DeFi applications into two categories based on their role in the ecosystem:

Price Provider Applications. These applications provide price data to other DeFi applications. Examples include Chainlink oracle contracts, decentralized exchanges (DEXs), and other protocols that define their own logic for determining asset prices.

Price Consumer Applications. Applications falling under this category rely on accurate price data for their operational eficacy. Examples include lending and borrowing platforms, which use price data to determine the valuation of collaterals, and DAOs, which use price data to determine voting weights and other governance parameters.

With this classification, we can accurately analyze the root causes of the price oracle vulnerabilities and design eficient prompts to help LLMs detect such kinds of vulnerabilities.

## 2.2 Types of Price Oracles

Price oracles can be generally categorized into three classes: on-chain oracles, of-chain oracles, and hybrid price oracles.

On-Chain Oracles. On-chain oracles, like those used by Uniswap, derive price data directly from on-chain activities such as trading within liquidity pools. These oracles use mechanisms like Constant Product Formula (CPF) ?? to calculate asset prices based on real-time transactions occurring on the blockchain. Despite their susceptibility to manipulation in low-liquidity scenarios, they ofer several advantages:

• Native to Blockchain: Since on-chain oracles operate entirely within the blockchain environment, they provide seamless integration with decentralized applications (DApps) and smart contracts without the need for external dependencies.

• Real-time Pricing: Prices reflect current market conditions as they are derived directly from ongoing transactions on the blockchain.

• Decentralization: Since these oracles are based on decentralized mechanisms (e.g., Uniswap’s liquidity pools), there is no central authority controlling the price feed, reducing single points of failure.

• Full Transparency: Anyone can verify the price data on-chain, ensuring the data’s integrity and preventing manipulation by a central entity.

Manuscript submitted to ACM

��������: Automated Detection of Price Oracle Manipulations via LLM-Driven Knowledge Mining and Prompt Generation

Of-Chain Oracles. Of-chain oracles, exemplified by Chainlink <sup>1</sup>, gather data from external sources and bring it onto the blockchain through a decentralized network of node operators. Their advantages can be summarized as follows:

• Access to Diverse Data: Of-chain oracles can pull price information from a wide range of external sources, including traditional financial markets, making them suitable for use cases requiring data beyond the blockchain ecosystem.

• Robustness to Manipulation: Since of-chain oracles aggregate data from multiple independent nodes or sources, they are generally more resistant to manipulation or data skewing compared to purely on-chain systems.

• Scalability: They are often more scalable than on-chain oracles since they are not dependent on the blockchain’s transaction throughput and can aggregate large volumes of data from diverse sources without congesting the network.

Hybrid Oracles. Hybrid oracles, such as those used by Extra Finace<sup>2</sup> combine features of both on-chain and of-chain oracles to enhance price stability and security. Typically, the prices are derived directly from on-chain data provided by decentralized exchanges (DEXs). However, to mitigate the risk of abnormal price fluctuations, Chainlink price feeds are employed as a safeguard. This hybrid approach ofers several advantages:

• Comprehensive Data Validation: Hybrid oracles cross-reference of-chain and on-chain data to ensure both accuracy and consistency, reducing the risk of manipulation.

• Real-Time Responsiveness: The integration of on-chain mechanisms ensures timely updates to price data, even during volatile market conditions.

• Resilience to Attacks: The use of diverse data sources creates redundancy, making hybrid oracles more robust against single-point failures or targeted attacks.

Despite the popularity of Of-Chain Oracles like Chainlink, On-Chain Oracles remain a viable option for many in the blockchain community. According to an oracle dashboards <sup>3</sup> <sup>4</sup>, as many as 90 projects opted for On-Chain Oracles. This preference is largely due to their inherent consistency with the decentralization philosophy of blockchain systems, as they operate entirely within the blockchain environment, ensuring trustlessness and minimizing reliance on external entities.

## 2.3 Price Oracles Manipulation (POM)

POM can stem from various sources based on above types of oracles. The primary complications arise from on-chain and of-chain oracles.

On-Chain Price Oracle Manipulation. On-chain oracles can be easily manipulated due to their reliance on spot prices from a single source. For instance, an attacker can use a flash loan to temporarily drain liquidity from a pool, causing the price to be artificially inflated or deflated. This manipulation allows the attacker to exploit the manipulated price, leading to significant financial gains, as demonstrated in the PancakeBunny attack<sup>5</sup>. A more detailed example is illustrated in the Appendix C.

Of-Chain Price Oracle Manipulation. Of-chain oracles face diferent challenges. Centralized of-chain oracles depend on a single trusted entity, making them vulnerable to malicious data submission by authorized users for personal gain. Additionally, the compromise of private keys can pose significant risks. Decentralized of-chain oracles mitigate some of these risks by aggregating data from multiple sources, but they are not immune to issues like freeloading or Sybil attacks among data collectors. Further, of-chain infrastructure vulnerabilities—including those in access control, cryptographic implementations, transport, and database security—add layers of complexity in preventing manipulation [7].

While these issues are broad and afect the overall security of price oracles, this paper focuses specifically on vulnerabilities that adversaries can exploit, particularly through specialized inputs to on-chain contracts. This includes manipulations involving on-chain price oracles and the on-chain components of of-chain price oracles, which can lead to significant financial losses or unfair advantages for attackers.

## 2.4 Causes of Price Oracle Manipulation

Price oracle manipulation arises from various factors that exploit weaknesses in both the underlying mechanisms and the broader DeFi ecosystem. Below are some key causes:

Smart Contract Vulnerabilities. Careless bugs or flawed logic while development in the smart contracts governing liquidity pools or price feed mechanisms can lead to incorrect pricing, enabling attackers to manipulate asset values and potentially causing significant financial losses for users and protocols [11].

Flash Loan Attacks. Flash loans allow users to borrow large amounts of capital without collateral, provided the loan is repaid within the same transaction. Attackers exploit this feature by executing large trades to temporarily inflate or deflate the price of assets in on-chain liquidity pools. This manipulated price can then be leveraged in other DeF protocols that depend on the oracle, leading to cascading financial consequences [40].

Front-Running Attacks. Front-running attacks, enabled by the transparency of blockchain transactions, also contribute to price oracle manipulation. Malicious actors monitor pending transactions and strategically place their trades just before large transactions. By doing so, they can profit from the resulting price changes while distorting the price data in liquidity pools [40].

Impermanent Loss Impact. Liquidity providers may sufer from impermanent loss, where the value of their deposited assets changes due to price fluctuations within the pool. Ifa DeFi application relies on the pool’s price without accounting for these fluctuations, it might overestimate or underestimate the true value of assets [17].

Slippage. The diference between the expected and actual executed price of a trade presents another avenue for manipulation. In low-liquidity pools, attackers can exploit slippage by executing large trades that cause significant price deviations. These deviations can propagate through dependent DeFi applications, leading to inaccurate price feeds and destabilizing the broader ecosystem [17].

These factors are inherent features of blockchain and DeFi systems, not deficiencies. While they do not inherently lead to attacks, they can introduce vulnerabilities under certain conditions. The goal is not to eliminate these features but to identify potential weaknesses and mitigate their adverse efects, thereby maximizing their benefits.

Manuscript submitted to ACM

��������: Automated Detection of Price Oracle Manipulations via LLM-Driven Knowledge Mining and Prompt Generation

## 3 Proposed Approach

In this section, we first briefly review the state-of-the-art LLMs and key prompt engineering techniques, establishing the foundations of our work. We then introduce our LLM-driven detection framework, ��������, in detail.

![](images/2f04b2722eba0da7cbce8c72dda5e07e3c8d1ff0969f2b27980ee6e3615bfd6f.jpg)  
Fig. 1. Overview of LLM-driven Automated Detection Framework.

## 3.1 LLMs and Prompt Design Methods

The rapid advancements in large language models (LLMs) have been driven by improvements in machine learning algorithms, computational power, and extensive training datasets. State-of-the-art models like ChatGPT series<sup>6</sup>, Claude series<sup>7</sup>, and open-source models like Llama series<sup>8</sup>, Qwen series<sup>9</sup>have significantly advanced natural language processing (NLP), excelling in tasks such as text generation, summarization, question answering, and program bug detection [18]. While LLMs have demonstrated remarkable performance across various tasks, their ability in reasoning and addressing complex problems remains highly dependent on the quality of the prompts provided [34]. To maximize their potential, innovative prompt engineering techniques have been developed, such as Chain-of-Thought (CoT)[34], Least-to-Most[42], and Complex CoT [9], which guide models to decompose complex tasks into smaller, more structured steps. Remarkably, even simple zero-shot CoT prompts like “let’s think step by step” have demonstrated improvements of up to 60% on specific datasets [15].

Despite these advancements, designing eficient and efective prompts remains a challenge, particularly for complex tasks like POM detection. Building upon these techniques, our work integrates automated prompt design into a systematic, multi-LLM framework. By synthesizing domain-specific knowledge and dynamically generating tailored prompts, we enable LLMs to address the intricate challenge of detecting price oracle manipulation vulnerabilities in a scalable and automated manner. This eliminates reliance on manual intervention, achieving both precision and eficiency.

## 3.2 ��������: LLM-driven Automated Detection Framework

In this section, we introduce the three core components of ��������. As illustrated in Fig. 1, our framework is composed of the Domain Knowledge Synthesizer, Prompt Generator, and Auditor, each playing a crucial role in automating and optimizing the detection process.

3.2.1 Domain Knowledge Synthesizer. This module synthesizes precise insights about price oracle vulnerabilities by leveraging top-tier academic literature, minimizing reliance on developer or auditor expertise through the automated summarization of key definitions and patterns. Furthermore, it ensures the use of high-quality domain-specific information, mitigating the influence of web-scale noisy data used in the pre-training phase of the model.

## Knowledge Synthesizing Example

System: You are a blockchain researcher. You are good at creating precise, comprehensive problem definitions.

User: You are given descriptions of the price oracle manipulation problem from top academic conference papers, delimited by triple backticks “‘. Your task is to analyze these descriptions and provide a concise, unified definition of the price oracle manipulation problem. “‘ + {knowledge} + “‘. Return the definition in the JSON format: {output format}.

Assistant: The price oracle manipulation problem refers to a vulnerability in decentralized finance (DeFi) applications where an attacker can manipulate the price of a cryptocurrency in a decentralized exchange (DEX) that serves as the price oracle for the victim smart contract. The attack consists of two steps: 1. Price Oracle Manipulation (POM): The attacker trades a large volume of the cryptocurrency in the DEX, tilting the balance of the trading pair and causing the price to deviate from the actual market price. 2. Arbitrage: The attacker then exploits the price disparity between the manipulated price and the actual market price to obtain the cryptocurrency at the lower, manipulated price and sell it at the higher, actual market price, profiting from the diference. This vulnerability arises when the victim smart contract relies on the manipulated price provided by the DEX price oracle, rather than using more robust price oracles or mechanisms to mitigate price manipulation.

To achieve this, we selected papers relevant to the topic of price oracle problems from prestigious conferences, such as the IEEE Symposium on Security and Privacy (S&P), International Conference on Software Engineering (ICSE), IEEE European Symposium on Security and Privacy Workshops (EuroS&PW), and ACM SIGSOFT International Symposium on Software Testing and Analysis (ISSTA). From these sources, we identified seven papers [6, 16, 22, 32, 37, 38, 40] tha are particularly relevant to price oracle manipulation. We manually excerpted paragraphs within these papers that describe price oracle problems, including definitions, common attack vectors, techniques, and defense mechanisms. We believe this manual step, while necessary, presents minimal challenges. This is because the published academic papers typically employ clear subtitles to delineate problem descriptions, making the identification and extraction of relevan information straightforward.

Subsequently, we prompt the LLM model to summarize the paragraphs on price oracle manipulation. This synthesized definition serves as the sole knowledge for the prompt generator, ensuring that the generated prompts are grounded in Manuscript submitted to ACM

��������: Automated Detection of Price Oracle Manipulations via LLM-Driven Knowledge Mining and Prompt Generation

accurate and comprehensive domain knowledge. An example of the knowledge synthesis process, including the prompt and response, is provided in Knowledge Synthesizing Example 3.2.1.

This module is pivotal for extending our framework to address other issues. By extracting high-quality, relevant descriptions of specific problems, the framework can be easily adapted to tackle vulnerabilities such as Privilege Escalation and Inconsistent State Updates [40].

## CoT Prompt Generating Example

System: You are an experienced prompt engineer.

User: Your task is to provide a chain-of-thought prompt to an AI language model to accurately detect price oracle manipulation issues which could lead to financial losses or unfair advantages in the given smart contract step by step. Remember to base your generated prompt solely on the price oracle description in triple backticks “‘. Do not make assumptions about external factors or systems not mentioned in the given information. “‘ + {synthesized knowledge} + “‘. Return the prompt in the JSON format: {output format}.

Assistant: description: Detect price oracle manipulation vulnerabilities in a smart contract, steps: step: 1, description: Identify the price oracle used by the smart contract. Determine if the price oracle is a decentralized exchange (DEX) or another external price feed.,

questions:

a. What is the price oracle used by the smart contract?

b. Is the price oracle a decentralized exchange (DEX) or another external price feed?

step: 2, description: Analyze the smart contract’s reliance on ...

3.2.2 Prompt Generator and Auditor. The synthesized domain knowledge is integrated into the Prompt Generator, a component that combines extracted insights with tailored instructions. We employ several techniques to enhance the prompt generator’s ability to produce structured and actionable CoT prompts. As demonstrated in Prompt Generating ExampleA.1, the System and User represent the input provided by ��������, while the Assistant showcases the model’s output. In this example, the sentence highlighted in orange represents a role-based prompt, which assigns a specific relevant role to guide the prompt. This technique has proven efective in various tasks, with Zhang et al. [41] reporting approximately a 20% improvement in accuracy compared to the Simple Prompt approach. Building on this, we introduced the Zero-shot CoT Prompt approach, described in Section 3.1. This method utilizes the “magical” phrase step by step, highlighted in brown, to encourage the model to perform logical reasoning. Another technique employed is the use of positive and negative prompts, inspired by conditional generation models like Stable Difusion [1]. Positive prompts, highlighted in blue, provide explicit guidance, while negative prompts, shown in red, define constraints to avoid irrelevant or misleading outputs. These prompts are important because LLM models are pre-trained on general web-scale data, this pre-trained knowledge may conflict with the actual context of POM and thus interfere the analysi of the underlying task. For instance, such knowledge often leads to extraneous alarms, like presuming that an oracle owner’s potential to modify the oracle inherently makes the “set oracle” function vulnerable. While these findings may hold in broader contexts, they are not classified as POM attacks from a developer’s perspective and therefore increase Manuscript submitted to ACM false alarm (as revealed in the later section). By adhering to the constraints defined by positive and negative prompts, the generated Cot prompt remain highly focused and relevant to the specific vulnerabilities under investigation. Next, we incorporate synthesized domain knowledge, highlighted in brown, from the Knowledge Synthesizer into the prompts, creating what we call Context-based Prompts. With this enriched knowledge, the prompt generator can deliver task-aware and accurate instructions, ensuring that the prompts are highly aligned with the nuances of the vulnerabilities being analyzed. The complete prompt generated by this process is provided in Appendix A.

Lastly, the generated CoT prompt is utilized by the Auditor component, which employs an LLM model to detect POM vulnerabilities in the provided smart contract. The input to the Auditor is shown in Auditor Prompt Example below.

## Auditor Prompt Example

System: You are an experienced expert on auditing price oracle manipulation problems. Your task is to conduct a thorough audit on the provided solidity file to identify all potential price oracle manipulation vulnerabilities.

User: {generated CoT prompt} + {smart contract} + Respond the prompt in the following JSON format: {output format}.

In addition to the techniques outlined above, we emphasize the importance of fostering internal reasoning before generating responses, aligning with OpenAI’s o1 reasoning model.<sup>10</sup> To minimize manual efort across components, we incorporate this concept into the output format. As demonstrated in the Output Format example, the model is instructed to include key fields such as beneficiary, victim, and reason for each finding. These fields encourage the model to engage in critical reasoning, ensuring that relevant actors are accurately identified and justifications are provided for each result. This structured approach not only enhances the quality and interpretability of the output but also mitigates hallucination issues, reduces false alarms, and improves detection accuracy while minimizing noise in the analysis.

## Output Format Example

Analyze the smart contract delimited with “‘. Respond all the vulnerabilities with the following JSON format: {vulnerable: yes, function: functionName, contract: contract name of the vulnerable function, beneficiary: ..., victim: ..., reason:...}

\- vulnerable should be yes if the vulnerability exists, otherwise no.

\- beneficiary should be the role who will gain in the vulnerability.

\- victim should be the role who will sufer a loss or disadvantage in the vulnerability.

\- reason should describe why you think it is vulnerable and how to manipulate the price oracle to exploit this vulnerability.

## 4 Evaluation

In this section, we present the dataset utilized for our evaluation and discuss the findings derived from addressing the following research questions (RQs):

<sup>10</sup>https://platform.openai.com/docs/guides/reasoning

Table 1. Datasets for Evaluation

<table><tr><td>Dataset</td><td>Projects</td><td>Vulnerabilities</td><td>Functions (Avg)</td><td>LoC (Avg)</td></tr><tr><td>DeFiHack</td><td>31</td><td>36</td><td>24</td><td>1,630</td></tr><tr><td>Code4Rena</td><td>14</td><td>24</td><td>210</td><td>11,926</td></tr><tr><td>Total</td><td>45</td><td>60</td><td>null</td><td>null</td></tr></table>

RQ1: How efective are the state-of-the-art (SOTA) tool GPTScan and a zero-shot CoT prompt approach?

RQ2: How efective is �������� when evaluated on diverse real-world projects with human-curated knowledge?

RQ3: How does the Knowledge Synthesizer impact the framework’s performance in comparison to human-curated knowledge?

## 4.1 Dataset

Our dataset is curated to reflect real-world scenarios and challenges, providing a robust benchmark for evaluating the framework. As shown in Table 1, which summarizes the key statistics for the datasets, including the number of projects, the number of vulnerabilities for each dataset, and the average number of functions and lines of code (LoC) for each project, it generally consists of two categories: real-world attacked DeFiHacks projects and Code4Rena audi contest projects. To focus specifically on price oracle manipulation, we extended both datasets, ensuring comprehensive and reliable evaluation. By leveraging projects with documented exploitation reports and rigorous audit reports, we implemented a stringent method to categorize the vulnerabilities, thereby minimizing potential reporting biases and enhancing the empirical reliability of our evaluation framework

4.1.1 Real-World Atacked Projects. The dataset includes 31 projects that have experienced real-world attacks, sourced from two reliable datasets

• 11 Projects from GPTScan’s DeFiHack Dataset [29]: This dataset, designed by GPTScan, originally contained 13 projects. We excluded two projects that were unrelated to oracle manipulation to ensure relevance.

• 20 Projects from SOK [44]: These projects, categorized in the original SOK paper as on-chain oracle manipulation, liquidity borrowing and depositing issues, and slippage exploitation observed between 2021 and 2022, are included to increase the dataset size.

4.1.2 Code4Rena Projects. Code4rena [3] is a leading audit contest platform for pre-deployment projects. The platform engages project developers to commit bounties up to \$1M as incentives to draw participants from all over the world. Community experts selected and developers collaboratively review the submitted bug reports and reward the participants based on the severity and frequency of a particular bug submission. This incentive-driven process guarantees the integrity and credibility of the bugs reported, forming the ground truth for our study.

The dataset incorporates 14 projects from the Code4Rena platform:

• 6 Projects from Zhang et al. [40]: These are derived from Zhang et al.’s original set of 11 price oracle manipulation projects on Code4Rena. To ensure fair comparison with GPTScan, we excluded two misclassified projects and three incomplete ones.

• 8 Projects Newly Curated: To enhance the robustness of the evaluation, we extended the dataset by including additional projects from the Code4Rena platform<sup>11</sup> directly. We filtered projects from 2022 and 2023 using keywords such as frontrunning, slippage, flash loan, sandwich, price oracle, and manipulation in the audit reports. Each selected bug was manually checked to ensure it was a POM bug, and all projects relevant were confirmed to compile successfully. As a result, 8 projects were added to the dataset.

By focusing solely on price oracle manipulation, we enhanced both real-world attacked and Code4Rena datasets to ensure reliable evaluation.

## 4.2 Baseline

Since very few works have addressed the problem of price oracle manipulation, most existing analyses are postmortem and rely on transaction data, which difers from our approach. Our goal is to prevent attacks before they occur. To evaluate our framework, we select one state-of-the-art (SOTA) LLM-based tool, GPTScan, as our primary baseline. GPTScan utilizes an LLM (initially ChatGPT-3.5) to analyze pre-tagged functions potentially susceptible to specific vulnerability types through predefined scenarios and rules. The tool then applies static analysis to validate the LLM-generated findings and filter out false positives, thereby enhancing detection precision. Given the deprecation of ChatGPT-3.5, we substituted it with ChatGPT-4o-mini, which OpenAI recommends as a more cost-efective and improved alternative.<sup>12</sup> Additionally, the original GPTScan implementation sufered from unstructured LLM output, which hindered systematic analysis by the static analysis module. To address this limitation, we modified the tool’s code to enforce structured JSON output supported by ChatGPT-4o-mini. These modifications ensure more consisten and reliable vulnerability assessments, improving the overall performance of the analysis pipeline.

To complement GPTScan, we design a zero-shot CoT prompt based on the assumption that a common LLM user, equipped with basic knowledge of prompt engineering and price oracle issues, could generate. This baseline zero-shot CoT prompt generally follows the zero-shot prompting guideline outlined in the Prompt Engineering Guide. <sup>13</sup> We adopt the role where the model acts as an experienced expert on auditing price oracle manipulation problems, with the task of identifying all potential vulnerabilities in a provided Solidity file. The final prompt, structured for clarity and consistency, is shown below:

## Zero-shot CoT Prompt

System: You are an experienced expert on auditing price oracle manipulation problems. Your task is to conduct a thorough audit on the provided solidity file step by step to identify all potential price oracle manipulation vulnerabilities.

User: Analyze the smart contract delimited with “‘. Respond with all the vulnerabilities with the following JSON format: {output format}.

Table 2. Model Descriptions

<table><tr><td>Models</td><td>Versions</td><td>Knowledge Cutoff</td><td>Context Window</td><td>Max Tokens</td></tr><tr><td>ChatGPT-4o</td><td>2024-05-13</td><td>Oct 2023</td><td>128k</td><td>4,096</td></tr><tr><td>Claude3.5-Sonnet</td><td>2024-06-20</td><td>July 2024</td><td>200k</td><td>8,192</td></tr><tr><td>ChatGPT-4o-mini</td><td>2024-07-18</td><td>Oct 2023</td><td>128k</td><td>16,384</td></tr><tr><td>Claude3-haiku</td><td>2024-07-18</td><td>Aug 2023</td><td>200k</td><td>4,096</td></tr></table>

Table 3. Model Parameters

<table><tr><td>Parameters</td><td>Knowledge Synthesizer</td><td>Generator</td><td>Auditor</td></tr><tr><td>temperature</td><td>0</td><td>0</td><td>1.0</td></tr><tr><td>top_p</td><td>1.0</td><td>1.0</td><td>1.0</td></tr></table>

## 4.3 Model Choices and Hyperparameters

We utilized four language models, as described in Table 2. These models were selected to represent the latest advancements in industry that are both accessible and afordable for everyday users. The Knowledge Cutof column is the cutof date of the training data, indicating the recency of the knowledge embedded in the model. The Context Window is the maximum combined length of input tokens and output tokens that the model can process in a single query. The max\_tokens is the upper limit of tokens that the model can generate as output. Although these models allow for higher token generation, the maximum output token limit was set to 1, 024 in our experiments to prioritize longer input.

In this study, we evaluated the performance of the models by adjusting two key parameters: temperature and top\_p. The temperature parameter controls the randomness of the output, with higher values promoting greater variability and lower values yielding more deterministic results. In contrast, top\_p applies nucleus sampling, where only tokens contributing to the top p probability mass are considered. For instance, a top\_p value of 0.1 restricts the model to tokens comprising the top 10% of cumulative probability. OpenAI generally advises modifying either top\_p or temperature, but not both simultaneously.

In this evaluation, we set the top\_p parameter to its default value of 1.0, ensuring that the full probability mass was considered. The temperature parameter was configured as follows: a value of 0 was applied for the Knowledge Synthesizer and Prompt Generator to prioritize accuracy and consistency, while the Auditor was assigned a value of 1.0 to encourage diverse and comprehensive vulnerability identification, thereby fully leveraging the model’s capabilities. All other parameter settings adhered to their default values as outlined in the OpenAI API documentation [26].

To mitigate the impact of randomness inherent in the models, each prompt was executed on the dataset three times. The models were instructed to identify vulnerable functions and provide detailed explanations of the vulnerabilities. The outputs were formatted in JSON to facilitate eficient post-processing.

## 4.4 Identifying the Best Configuration for ��������

This section is to establish the most efective combination of components for the POM detection task. We start by evaluating the performance of the Auditor component in isolation. Subsequently, we enhance the setup by incorporating the Prompt Generator with human-curated knowledge to improve performance. Finally, we automate the knowledge extraction process by integrating the Knowledge Synthesizer for better usability.

Table 4. Average Performance of Baselines on DeFiHacks

<table><tr><td rowspan="2">Models</td><td rowspan="2">GPTScan4o-mini</td><td colspan="4">Zero-shot CoT</td></tr><tr><td>4o</td><td>Sonnet</td><td>4o-mini</td><td>Haiku</td></tr><tr><td>FN</td><td>26.67</td><td>25.67</td><td>29.67</td><td>14.33</td><td>26.33</td></tr><tr><td>TP</td><td>9.33</td><td>10.33</td><td>6.33</td><td>21.67</td><td>9.67</td></tr><tr><td>FP</td><td>20.67</td><td>19.67</td><td>23.00</td><td>58.33</td><td>19.33</td></tr><tr><td>Recall</td><td>0.259</td><td>0.287</td><td>0.176</td><td>0.602</td><td>0.269</td></tr><tr><td>Precision</td><td>0.311</td><td>0.344</td><td>0.216</td><td>0.271</td><td>0.333</td></tr><tr><td>F1</td><td>0.283</td><td>0.313</td><td>0.194</td><td>0.374</td><td>0.297</td></tr></table>

4.4.1 Identifying the Best Auditor Model. We assessed the zero-shot CoT prompt (described in Section 4.2) on the DeFiHacks dataset to identify the most suitable model as the auditor for detecting POM issues. For comparison, we also evaluated GPTScan on the same dataset. The DeFiHacks dataset was chosen because it allows for more eficient manual verification of the outputs, facilitating the identification of optimal parameter combinations.

The results are summarized in Table 4. The metrics used to assess performance include False Negatives (FN), which stands for the number of vulnerabilities incorrectly classified as safe; True Positives (TP), the number of the vulnerabilities correctly identified and False Positives (FP), the number of instances incorrectly flagged as vulnerable but are actually safe. We evaluated performance using precision, recall, and F1 score as key indicators. Precision measures the accuracy of positive predictions, while Recall assesses the model’s ability to identify all relevant instances. The F1 score, as the harmonic mean of precision and recall, ofers a balanced metric, particularly valuable for imbalanced class distributions. For detailed formulas, refer to Appendix D.

As shown in Table 4, almost all the models achieved higher F1 Score performance compared to GPTScan. Among the evaluated models, 4o-mini from the zero-shot CoT approach demonstrated superior performance, achieving the highest F1 Score (0.374) on the DeFiHacks dataset, outperforming GPTScan (0.283) and other models, including 4o version. Similarly, Claude’s Haiku outperformed its more advanced counterpart, Sonnet, in terms of F1 Score (0.297 vs. 0.194). A detailed review of the results shows that more complex and advanced models tend to generate more conservative output, while this leads to lower false positive, it produces fewer overall predictions, which is more likely to miss true vulnerabilities.

Notably, while GPTScan significantly reduced false positives, it also increased false negatives. This trade-of resulted in a marginally higher precision (0.311 vs. 0.271) but at the expense of a markedly lower recall (0.259 vs. 0.602).

Finding 1: 4o-mini demonstrates potential as a better auditor. “Mini” versions like 4o-mini and Claude’s Haiku outperformed their flagship counterparts, with 4o-mini achieving the highest F1 Score (0.374) among all models.

In response to RQ1: Despite advancements in model performance, both GPTScan and the zero-shot CoT approaches show limited efectiveness in reliably detecting price oracle problems, emphasizing the need for further refinement.

��������: Automated Detection of Price Oracle Manipulations via LLM-Driven Knowledge Mining and Prompt Generation

To clarify the discrepancy between our evaluation and the report from GPTScan’s paper, it is important to note that the metrics used in both evaluations difer. In GPTScan’s report, they emphasize the identification of vulnerability types across projects. For instance, in the project Hack-20210603-PancakeHunny, 7 vulnerabilities of Flash Loan Price (FLP) were found in their report, but only 1 true positive (TP) was counted in their precision and recall calculations. This diference in how vulnerabilities are calculated and counted contributes to the significant variation in the results presented here.

4.4.2 Identifying the Best Prompt Generator with Manually Curated Knowledge. Building on the results of the previous section, where “4o-mini” was identified as the best auditor model using the zero-shot CoT prompt, we now explore varying models for the prompt generator to determine the optimal combination for detecting POM vulnerabilities. This section also tries to validate the performance of the best auditor model identified again.

To balance computational eficiency with reliability, we adopted a strategic optimization approach. Specifically, we stabilized the domain knowledge component by integrating human-curated expertise into the prompt generator. This refinement allowed us to constrain the experimental space and focus on identifying optimal combinations of models for vulnerability detection. With the optimal model combinations identified, we can systematically reduce manual intervention with the knowledge synthesizer and progressively automate our vulnerability detection framework, ultimately advancing towards a more autonomous and robust oracle manipulation detection system. The curated domain knowledge that underpins this optimization is presented in Appendix B.1.

![](images/2beedad2fb8f71d61a5f59a6d495bd5771d93167fb43c1ee3c6da5db478e2db3.jpg)  
Fig. 2. F1 Score of Varying Prompt Generator and Auditor with Human-curated Knowledge.

The results are presented in Figure 2, which illustrates a comparative analysis of diferent model configurations applied to the DeFiHacks dataset. The raw data for these results is provided in Table 5. In this figure, the y-axis represents performance, measured by the F1-score. Each bar cluster in the x-axis denote the choice of auditor model (ranges in “4o”, “Sonnet”, “4o-mini” and “Haiku”). Within each cluster, diferent bars represent diferent choices of the paired generator models (ranges in “4o-generator”, “Sonnet-generator”, “4o-mini-generator” and “Haiku-generator”) as well as the two Manuscript submitted to ACM baselines (GPTScan and zero-shot CoT baseline). Note that, we stick to use 4o-mini for GPTScan in all the comparisons, and the results reported in Section 4.4 is reused for the zero-shot CoT.

Table 5. Average Performance of Varying Prompt Generator and Auditor with Human-curated Knowledge

<table><tr><td>Prompt Gen Auditor</td><td>4o 4o</td><td>Sonnet 4o</td><td>4o-mini 4o</td><td>Haiku 4o</td><td>4o Sonnet</td><td>Sonnet Sonnet</td><td>4o-mini Sonnet</td><td>Haiku Sonnet</td><td>4o 4o-mini</td><td>Sonnet 4o-mini</td><td>4o-mini 4o-mini</td><td>Haiku 4o-mini</td><td>4o Haiku</td><td>Sonnet Haiku</td><td>4o-mini Haiku</td><td>Haiku Haiku</td></tr><tr><td>FN</td><td>23.33</td><td>23.00</td><td>25.33</td><td>24.67</td><td>30.33</td><td>28.67</td><td>30.67</td><td>27.67</td><td>18.00</td><td>12.33</td><td>12.67</td><td>13.20</td><td>22.33</td><td>26.67</td><td>24.00</td><td>25.67</td></tr><tr><td>TP</td><td>12.67</td><td>13.00</td><td>10.67</td><td>11.33</td><td>5.67</td><td>7.33</td><td>5.33</td><td>8.33</td><td>18.00</td><td>23.67</td><td>23.33</td><td>22.80</td><td>13.67</td><td>9.33</td><td>12.00</td><td>10.33</td></tr><tr><td>FP</td><td>16.67</td><td>15.67</td><td>15.33</td><td>11.67</td><td>23.33</td><td>22.00</td><td>23.33</td><td>21.67</td><td>51.00</td><td>67.00</td><td>60.00</td><td>48.80</td><td>15.33</td><td>20.00</td><td>16.67</td><td>18.67</td></tr><tr><td>Recall</td><td>0.352</td><td>0.361</td><td>0.296</td><td>0.315</td><td>0.157</td><td>0.204</td><td>0.148</td><td>0.231</td><td>0.500</td><td>0.657</td><td>0.648</td><td>0.633</td><td>0.380</td><td>0.259</td><td>0.333</td><td>0.287</td></tr><tr><td>Precision</td><td>0.432</td><td>0.453</td><td>0.410</td><td>0.493</td><td>0.195</td><td>0.250</td><td>0.186</td><td>0.278</td><td>0.261</td><td>0.261</td><td>0.280</td><td>0.318</td><td>0.471</td><td>0.318</td><td>0.419</td><td>0.356</td></tr><tr><td>F1</td><td>0.388</td><td>0.402</td><td>0.344</td><td>0.384</td><td>0.174</td><td>0.224</td><td>0.165</td><td>0.253</td><td>0.343</td><td>0.374</td><td>0.391</td><td>0.424</td><td>0.421</td><td>0.286</td><td>0.371</td><td>0.318</td></tr></table>

As demonstrated in the Figure, the performance varies significantly across diferent combinations of prompt generators and auditor models. Nevertheless, our framework consistently outperforms the zero-shot CoT across 12 out of 16 model configurations, underscoring the eficacy of leveraging human-curated knowledge and optimized, context-aware prompts. Among the results obtained and showed in Table 5 and Figure 2, two combinations stand out: 1). “Haiku” as the prompt generator paired with “4o-mini” as the auditor: This combination achieved the highest F1 score of 0.424 representing a 13.4% improvement over the zero-shot CoT of 4o-mini. It also attained the highest recall (0.633), making it ideal for identifying a broader range of vulnerabilities. 2). “4o” as the prompt generator paired with “Haiku” as the auditor: This combination achieved an F1 score of 0.421, a 41.8% improvement over the zero-shot CoT of Haiku. It exhibited higher precision, making it preferable for scenarios where minimizing false positives is critical. These results highlight the synergistic benefits of pairing complementary model architectures, enabling tailored optimization for varying detection priorities.

Finding 2: Our framework, enhanced with human-curated knowledge, improves upon the baseline prompt in most cases (12 out of 16). Notably, two combinations—‘Haiku-generator’ with the ‘4o-mini’ auditor and ‘4o-generator’ with the ‘Haiku’ auditor—achieved the highest F1-scores of 0.424 and 0.421, compared to the baseline’s best of 0.374.

In response to RQ2: �������� identifies two combinations that outperform the best baseline performance with human-curated knowledge, demonstrating the eficacy of ��������.

4.4.3 Identifying the Best Knowledge Synthesizer for Automation. With the best auditor and prompt generator combinations identified, the next step is to optimize the knowledge synthesizer for summarizing domain-specific information, ultimately aiming for a fully automated vulnerability detection framework. The knowledge synthesizer plays a critica role in utilizing domain knowledge from external domain-specific sources, and integrating them into the detection pipeline. To systematically evaluate the knowledge synthesizer’s efectiveness, we fixed the two top-performing combinations of Prompt Generator and Auditor from Finding 2 (Haiku-generator with 4o-mini auditor, and 4o-generator with Haiku auditor) while varying the knowledge synthesizer models. Performance metrics included consistency, accuracy, and the overall F1-score.

As summarized in Table 6, the combination of Haiku as the Knowledge Synthesizer, Prompt Generator, and Auditor (Haiku-Haiku-4o-mini) achieved the best performance, with an F1-score of 0.426. Remarkably, this result slightly Manuscript submitted to ACM

Table 6. Average Performance of Varying Knowledge Synthesizer

<table><tr><td>Knowledge Synthesizer</td><td>4o</td><td>Sonnet</td><td>4o-mini</td><td>Haiku</td><td>4o</td><td>Sonnet</td><td>4o-mini</td><td>Haiku</td></tr><tr><td>Prompt Generator Auditor</td><td>Haiku 4o-mini</td><td>Haiku 4o-mini</td><td>Haiku 4o-mini</td><td>Haiku 4o-mini</td><td>4o Haiku</td><td>4o Haiku</td><td>4o Haiku</td><td>4o Haiku</td></tr><tr><td>FN</td><td>11.00</td><td>10.00</td><td>14.00</td><td>12.00</td><td>26.00</td><td>23.00</td><td>26.00</td><td>25.00</td></tr><tr><td>TP</td><td>25.00</td><td>26.00</td><td>22.00</td><td>24.00</td><td>10.00</td><td>13.00</td><td>10.00</td><td>11.00</td></tr><tr><td>FP</td><td>86.00</td><td>74.67</td><td>56.00</td><td>52.67</td><td>18.00</td><td>17.00</td><td>20.00</td><td>19.00</td></tr><tr><td>Recall</td><td>0.694</td><td>0.722</td><td>0.611</td><td>0.667</td><td>0.278</td><td>0.361</td><td>0.278</td><td>0.306</td></tr><tr><td>Precision</td><td>0.225</td><td>0.258</td><td>0.282</td><td>0.313</td><td>0.357</td><td>0.433</td><td>0.333</td><td>0.367</td></tr><tr><td>F1</td><td>0.340</td><td>0.380</td><td>0.386</td><td>0.426</td><td>0.313</td><td>0.394</td><td>0.303</td><td>0.333</td></tr></table>

surpasses that of the human-curated knowledge framework (0.426 vs. 0.424), demonstrating the potential of fully automated domain knowledge synthesis in enhancing vulnerability detection capabilities. This improvement is particularly noteworthy as it highlights the ability of automated frameworks to match—and even exceed—manual approaches. Additionally, the Haiku-Haiku-4o-mini configuration achieved this result without requiring extensive manual intervention, marking a significant step toward a robust and autonomous system.

Finding 3: The combination of Haiku-Haiku-4o-mini achieved the highest F1 score (0.426), slightly outperforming the human-curated knowledge framework (0.424). This result underscores the potential of fully automated domain knowledge synthesis for advancing vulnerability detection.

Finding 4: The best performance is achieved using a combination of less complex models (Haiku-Haiku-4o-mini), demonstrating that larger models do not necessarily lead to better results. This finding highlights the potential for accessibility and suggests the feasibility of using alternative models similar in size and complexity.

In response to RQ3: the Knowledge Synthesizer automates knowledge extraction and slightly surpasses humancurated knowledge in both recall and precision, enhancing the framework’s performance, while reducing manual efort.

## 4.5 Comparison of �������� and GPTScan on Code4Rena

To evaluate the efectiveness of �������� on fully developed projects with all supporting files, we applied the bestperforming configuration identified in previous sections—Haiku as the Knowledge Synthesizer and Prompt Generator, paired with 4o-mini as the Auditor—on the Code4Rena dataset. This dataset presents additional challenges due to its complexity and diverse set of components, providing a rigorous test of the system’s capabilities. For comparison, we also evaluated the state-of-the-art tool GPTScan on the same dataset.

The performance comparison between �������� and GPTScan are summarized in Table 7. �������� demonstrated a significant improvement in recall (0.54) compared to GPTScan (0.13), indicating a superior ability to detect vulnerabilities in the dataset. This higher recall highlights the efectiveness of the LLM generated knowledge synthesis and optimized Manuscript submitted to ACM prompt generation. However, ��������’s precision (0.092) remains comparable to GPTScan (0.094), reflecting the challenges in reducing false positives in complex datasets. Despite the trade-ofs, �������� achieved a higher overal F1-score of 0.157 compared to GPTScan’s 0.107, underscoring its improvements on complex dataset.

Table 7. Performance of �������� and GPTScan on Code4rena Datase

<table><tr><td>Metrics</td><td>FN</td><td>TP</td><td>FP</td><td>Recall</td><td>Precision</td><td>F1</td></tr><tr><td>AiRacleX</td><td>11</td><td>13</td><td>128.3</td><td>0.54</td><td>0.092</td><td>0.157</td></tr><tr><td>GPTScan</td><td>21</td><td>3</td><td>29</td><td>0.13</td><td>0.094</td><td>0.107</td></tr></table>

These findings highlight that a simple and general framework �������� can outperform state-of-the-art tools like GPTScan, even in complex and challenging datasets. This positions �������� as a promising solution for advancing vulnerability detection frameworks

Finding 5: �������� outperforms GPTScan on the challenging Code4Rena dataset, achieving higher recall (0.54 vs. 0.13) and F1-score (0.157 vs. 0.107). However, its precision remains on par with GPTScan, reflecting the need for further refinements to reduce false positives.

## 5 Related Work

Smart contract vulnerability detection has advanced significantly, with numerous tools and techniques proposed to address security issues. Despite these eforts, detecting and mitigating manipulative behaviors in price oracles remains a persistent challenge. Existing approaches to this problem can be broadly classified into static analysis, dynamic analysis, machine learning-based methods, and emerging techniques leveraging large language models (LLMs).

## 5.1 Static Analysis

Static analysis techniques examine the source code or bytecode of smart contracts without execution, employing methods such as symbolic execution, formal verification, and pattern matching to identify vulnerabilities. Several tools have been developed for various types of vulnerabilities. For instance, Oyente[21] uses symbolic execution to detect issues such as reentrancy, transaction order dependency, suicidal contracts, and integer overflows. SmartCheck[30] applies rule-based techniques to identify vulnerabilities and bad practices in Solidity contracts, while Slither [8] combines dataflow analysis, taint analysis, and pattern matching to detect a wide range of vulnerabilities eficiently. Formal verification methods further enhance static analysis by modeling smart contract behavior using formal languages and verifying properties with SMT solvers or theorem provers. Examples include VeriSmart[28] and sVerify[10], which are tailored for smart contract verification against predefined specifications

However, few works focus explicitly on price oracle manipulation vulnerabilities. Recent research has started addressing this gap: Foray [35] is an attack synthesis framework for DeFi protocols that uses a domain-specific language to convert smart contracts into token flow graphs. While it identifies strategic paths and synthesizes attacks via symbolic compilation, its focus is limited to four specific types of logical flaws, which partially overlap but do not fully align with our target vulnerabilities. OVer [6] employs symbolic analysis to model DeFi protocol behavior under skewed oracle inputs, identifying secure parameters and generating guard statements to mitigate manipulation attacks. While efective, its focus on optimizing parameters for specific protocols limits its generalizability to broader applications. Manuscript submitted to ACM

��������: Automated Detection of Price Oracle Manipulations via LLM-Driven Knowledge Mining and Prompt Generation

VeriOracle [22] introduces a formal verification framework that deploys a semantic model on the blockchain to monitor smart contract states and detect problematic price feed transactions in real time. DeFiTainter [16] leverages innovative mechanisms to construct call graphs and semantically track inter-contract taint data for detecting price manipulation vulnerabilities. However, they both require extensive on-chain transaction data, demanding significant resources and difering from our approach.

## 5.2 Dynamic Analysis

Dynamic analysis techniques execute smart contract code and monitor its runtime behavior to identify vulnerabilities. These methods often employ fuzzing, symbolic execution, and runtime monitoring to detect issues such as assertion failures, overflows, and frozen ether. Tools like Mythril[24], Manticore[23], sFuzz[25], and ContractFuzzer[14] have been widely used for identifying common vulnerabilities.

Despite their success, traditional dynamic analysis tools have rarely addressed price oracle manipulation vulnerabili ties. Only a few works have specifically targeted this challenge. DeFiRanger [37] recovers high-level DeFi semantics from raw Ethereum transactions and identifies price oracle manipulation attacks through pattern matching. However, its approach is post-mortem, as it can only detect observed attack transactions, limiting its usefulness for proactive vulnerability detection. ProMutator [32] models typical DeFi usage patterns by analyzing existing transactions and simulates potential price manipulation attacks through mutated transactions. This approach efectively identifies weak points in oracle systems before exploitation. However, accurately modeling DeFi transaction patterns is challenging, especially for novel attack vectors, and its simulation-based method requires significant computational resources, impacting scalability and real-time applicability. DeFiPoser [43] employs a dual approach: DEFIPOSER-ARB for identifying arbitrage opportunities and SMT solvers to create logical models for detecting complex profitable transactions. While it can uncover new vulnerabilities in real time, the system relies on manual and costly modeling of DeFi protocols, making it resource-intensive. Furthermore, its efectiveness may be limited by the rapid evolution of DeFi protocols requiring frequent updates to maintain accuracy and relevance.

These limitations highlight the advantages of our AI-driven framework, which eliminates reliance on expert knowledge and enhances scalability, efectively overcoming the ineficiencies and adaptability challenges in existing methods.

## 5.3 Machine Learning-based Methods

Machine learning-based methods have gained traction in recent years for smart contract vulnerability detection. These approaches typically involve extracting features from the contract’s source code or bytecode and training classifiers or deep learning models to predict the presence of vulnerabilities. Some notable works in this domain include ContractWard [33], which trains a classifier based on features extracted from the contract’s bytecode, and the hybrid approach proposed by Liu et al. [19], which combines pure neural networks with interpretable graph features and expert patterns. Graph neural networks have also been explored for smart contract vulnerability detection. These approaches represent the contract’s control flow graph or data dependency graph as a graph-structured data and apply graph neural networks to learn vulnerability patterns. EtherGIS [39] is an example of a vulnerability detection framework that utilizes graph learning features to detect vulnerabilities in Ethereum smart contracts.

While these methods achieve high accuracy for various vulnerabilities, they rely heavily on large labeled datasets and often struggle with novel or unseen patterns. Moreover, limited work has specifically addressed price oracle manipulation vulnerabilities, leaving a gap that requires innovative solutions.

Manuscript submitted to ACM

## 5.4 Large Language Model-based Methods

Recent advancements in large language models (LLMs) have opened up new possibilities for smart contract vulnerability detection. LLMs, such as GPT, have demonstrated remarkable capabilities in understanding and generating human-like text, and researchers have begun exploring their application in smart contract analysis. Gao et al.[11] explored LLMs for detecting complex bugs, including price oracle manipulation, using diverse prompts. However, this early work demonstrated limited performance, making it unsuitable for practical use. Similarly, Issac et al.[5] evaluated ChatGPT-4 and Claude for smart contract audits, identifying logic flaws and coding errors but reporting an unacceptably high false positive rate (95%), which hinders real-world adoption. GPTLens [13] proposed an adversarial framework leveraging LLMs in dual roles to enhance detection accuracy, but its efectiveness on price oracle manipulation vulnerabilities remains limited. GPTScan [29] combines GPT with program analysis techniques to identify logic vulnerabilities in smart contracts. By leveraging GPT’s code understanding and static confirmation, GPTScan reduces false positives and achieves high precision and recall in terms of vulnerability type detection across diverse datasets.

In contrast, our work develops a fully LLM-driven approach focused on prompt engineering for detecting POM vulnerabilities. By utilizing domain-specific knowledge extraction and context-aware prompt generation, we enable LLMs to automatically identify manipulation patterns. Our method is user-friendly, generalizable, and provides actionable feedback by leveraging LLMs’ capacity to understand the contextual nuances of price oracle manipulations.

## 6 Conclusion and Future Work

POM attacks represent a pressing challenge in the DeFi space. �������� provides an innovative, LLM-driven solution that automates the detection of these vulnerabilities by leveraging domain-specific knowledge and context-aware prompt generation. By streamlining the analysis process and ensuring actionable outputs, �������� ofers a scalable and efective approach to safeguarding DeFi ecosystems.

For future work, several promising directions can be explored to enhance and expand the capabilities of ��������:

• Extension to Additional Problems: ��������’s framework can be expanded to address other DeFi issues, such as the accounting problem, privilege escalation, and inconsistent state updates.

• Automating Knowledge Synthesis: To reduce manual efort, future versions of�������� will enhance automation in knowledge synthesis, potentially using Retrieval-augmented Generation (RAG) [12] for eficient data extraction and summarization.

• Instruction-tuned LLM: This efort may include the construction of suitable instruction-following dataset and further supervised fine-tuning of the prompt generator LLM and/or the auditor LLM [20] to better follow the instructions and prompts given and align the model’s behave towards desired output.

• Reducing False Alarms: Enhancing usability by exploring advanced and fine-grained prompt techniques, such as adaptive prompts and context-aware filtering, to minimize false alarms while maintaining high detection accuracy.

By pursuing these directions, �������� can continue to evolve, ofering comprehensive protection against a growing array of vulnerabilities in the DeFi ecosystem, while enhancing its eficiency and user-friendliness.

## References

[1] Yuanhao Ban, Ruochen Wang, Tianyi Zhou, Minhao Cheng, Boqing Gong, and Cho-Jui Hsieh. 2025. Understanding the Impact of Negative Prompts: When and How Do They Take Efect?. In European Conference on Computer Vision. Springer, 190–206.

Manuscript submitted to ACM

[2] Stefanos Chaliasos, Marcos Antonios Charalambous, Liyi Zhou, Rafaila Galanopoulou, Arthur Gervais, Dimitris Mitropoulos, and Benjamin Livshits. 2024. Smart Contract and DeFi Security Tools: Do They Meet the Needs of Practitioners?. In Proceedings ofthe 46th IEEE/ACM International Conference on Software Engineering. 1–13.

[3] Code4rena. 2023. Code4rena Contest Platform. https://code4rena.com/ Accessed: 2023-04-16.

[4] Consensys. 2024. Mythril: Security analysis tool for EVM bytecode. https://github.com/Consensys/mythril Accessed: 2024-06-06.

[5] Isaac David, Liyi Zhou, Kaihua Qin, Dawn Song, Lorenzo Cavallaro, and Arthur Gervais. 2023. Do you still need a manual smart contract audit? arXiv preprint arXiv:2306.12338 (2023).

[6] Xun Deng, Sidi Mohamed Beillahi, Cyrus Minwalla, Han Du, Andreas Veneris, and Fan Long. 2024. Safeguarding DeFi Smart Contracts against Oracle Deviations. In Proceedings of the IEEE/ACM 46th International Conference on Software Engineering. 1–12.

[7] Dominik. 2025. Smart Contract Security Field Guide. https://scsfg.io/hackers/oracle-manipulation/#of-chain-infrastructure Accessed: 2025-01-06.

[8] Josselin Feist, Gustavo Grieco, and Alex Groce. 2019. Slither: a static analysis framework for smart contracts. In 2019 IEEE/ACM 2nd International Workshop on Emerging Trends in Software Engineering for Blockchain (WETSEB). IEEE, 8–15.

[9] Yao Fu, Hao Peng, Ashish Sabharwal, Peter Clark, and Tushar Khot. 2022. Complexity-based prompting for multi-step reasoning. arXiv preprint arXiv:2210.00720 (2022)

[10] Bo Gao, Ling Shi, Jiaying Li, Jialiang Chang, Jun Sun, and Zijiang Yang. 2021. sVerify: Verifying Smart Contracts Through Lazy Annotation and Learning. In Leveraging Applications ofFormal Methods, Verification and Validation: 10th International Symposium on Leveraging Applications of Formal Methods, ISoLA 2021, Rhodes, Greece, October 17–29, 2021, Proceedings 10. Springer, 453–469.

[11] Bo Gao, Qingsong Wei, Yong Liu, and Rick Siow Mong Goh. 2024. Unveiling the Potential of ChatGPT in Detecting Machine Unauditable Bugs in Smart Contracts: A Preliminary Evaluation and Categorization. In 2024 IEEE Conference on Artificial Intelligence (CAI). IEEE, 1481–1486.

[12] Yunfan Gao, Yun Xiong, Xinyu Gao, Kangxiang Jia, Jinliu Pan, Yuxi Bi, Yi Dai, Jiawei Sun, and Haofen Wang. 2023. Retrieval-augmented generation for large language models: A survey. arXiv preprint arXiv:2312.10997 (2023).

[13] Sihao Hu, Tiansheng Huang, Fatih İlhan, Selim Furkan Tekin, and Ling Liu. 2023. Large language model-powered smart contract vulnerability detection: New perspectives. arXiv preprint arXiv:2310.01152 (2023).

[14] Bo Jiang, Ye Liu, and Wing Kwong Chan. 2018. Contractfuzzer: Fuzzing smart contracts for vulnerability detection. In Proceedings of the 33rd ACM/IEEE international conference on automated software engineering. 259–269.

[15] Takeshi Kojima, Shixiang Shane Gu, Machel Reid, Yutaka Matsuo, and Yusuke Iwasawa. 2022. Large language models are zero-shot reasoners Advances in neural information processing systems 35 (2022), 22199–22213.

[16] Queping Kong, Jiachi Chen, Yanlin Wang, Zigui Jiang, and Zibin Zheng. 2023. Defitainter: Detecting price manipulation vulnerabilities in defi protocols. In Proceedings ofthe 32nd ACM SIGSOFT International Symposium on Software Testing and Analysis. 1144–1156.

[17] Mauricio Labadie. 2022. Impermanent loss and slippage in Automated Market Makers (AMMs) with constant-product formula. Available at SSRN 4053924 (2022).

[18] Qingyao Li, Lingyue Fu, Weiming Zhang, Xianyu Chen, Jingwei Yu, Wei Xia, Weinan Zhang, Ruiming Tang, and Yong Yu. 2023. Adapting large language models for education: Foundational capabilities, potentials, and challenges. arXiv preprint arXiv:2401.08664 (2023).

[19] Zhenguang Liu, Peng Qian, Xiaoyang Wang, Yuan Zhuang, Lin Qiu, and Xun Wang. 2021. Combining graph neural networks with expert knowledge for smart contract vulnerability detection. IEEE Transactions on Knowledge and Data Engineering 35, 2 (2021), 1296–1310.

[20] Shayne Longpre, Le Hou, Tu Vu, Albert Webson, Hyung Won Chung, Yi Tay, Denny Zhou, Quoc V Le, Barret Zoph, Jason Wei, et al. 2023. The flan collection: Designing data and methods for efective instruction tuning. In International Conference on Machine Learning. PMLR, 22631–22648

[21] Loi Luu, Duc-Hiep Chu, Hrishi Olickel, Prateek Saxena, and Aquinas Hobor. 2016. Making smart contracts smarter. In Proceedings of the 2016 ACM SIGSAC conference on computer and communications security. 254–269.

[22] Yifan Mo, Jiachi Chen, Yanlin Wang, and Zibin Zheng. 2023. Toward automated detecting unanticipated price feed in smart contract. In Proceedings ofthe 32nd ACM SIGSOFT International Symposium on Software Testing and Analysis. 1257–1268.

[23] Mark Mossberg, Felipe Manzano, Eric Hennenfent, Alex Groce, Gustavo Grieco, Josselin Feist, Trent Brunson, and Artem Dinaburg. 2019. Manticore A user-friendly symbolic execution framework for binaries and smart contracts. In 2019 34th IEEE/ACM International Conference on Automated Software Engineering (ASE). IEEE, 1186–1189.

[24] Bernhard Mueller, Nikhil Parasaram, Joran Honig, and Dominik Muhs. 2024. Mythril. https://github.com/Consensys/mythril. Accessed: 2024-05-23

[25] Tai D Nguyen, Long H Pham, Jun Sun, Yun Lin, and Quang Tran Minh. 2020. sfuzz: An eficient adaptive fuzzer for solidity smart contracts. In Proceedings ofthe ACM/IEEE 42nd International Conference on Software Engineering. 778–788

[26] Openai. 2024. Openai API References. https://platform.openai.com/docs/api-reference/chat/create Accessed: 2024-05-26.

[27] Protofire. 2024. Solhint: provide a linting utility for Solidity code. https://github.com/protofire/solhint Accessed: 2024-06-06.

[28] Sunbeom So, Myungho Lee, Jisu Park, Heejo Lee, and Hakjoo Oh. 2020. VeriSmart: A highly precise safety verifier for Ethereum smart contracts. In 2020 IEEE Symposium on Security and Privacy (SP). IEEE, 1678–1694.

[29] Yuqiang Sun, Daoyuan Wu, Yue Xue, Han Liu, Haijun Wang, Zhengzi Xu, Xiaofei Xie, and Yang Liu. 2023. When gpt meets program analysis: Towards intelligent detection of smart contract logic vulnerabilities in gptscan. arXiv preprint arXiv:2308.03314 (2023)

[30] Sergei Tikhomirov, Ekaterina Voskresenskaya, Ivan Ivanitskiy, Ramil Takhaviev, Evgeny Marchenko, and Yaroslav Alexandrov. 2018. Smartcheck: Static analysis of ethereum smart contracts. In Proceedings ofthe 1st international workshop on emerging trends in software engineering for blockchain. 9–16.

[31] Christof Ferreira Torres, Antonio Ken Iannillo, Arthur Gervais, and Radu State. 2021. Confuzzius: A data dependency-aware hybrid fuzzer for smar contracts. In 2021 IEEE European Symposium on Security and Privacy (EuroS&P). IEEE, 103–119.

[32] Shih-Hung Wang, Chia-Chien Wu, Yu-Chuan Liang, Li-Hsun Hsieh, and Hsu-Chun Hsiao. 2021. ProMutator: Detecting vulnerable price oracles in DeFi by mutated transactions. In 2021 IEEE European Symposium on Security and Privacy Workshops (EuroS&PW). IEEE, 380–385.

[33] Wei Wang, Jingjing Song, Guangquan Xu, Yidong Li, Hao Wang, and Chunhua Su. 2020. Contractward: Automated vulnerability detection models for ethereum smart contracts. IEEE Transactions on Network Science and Engineering 8, 2 (2020), 1133–1144.

[34] Jason Wei, Xuezhi Wang, Dale Schuurmans, Maarten Bosma, Fei Xia, Ed Chi, Quoc V Le, Denny Zhou, et al. 2022. Chain-of-thought prompting elicits reasoning in large language models. Advances in Neural Information Processing Systems 35 (2022), 24824–24837.

[35] Hongbo Wen, Hanzhi Liu, Jiaxin Song, Yanju Chen, Wenbo Guo, and Yu Feng. 2024. FORAY: Towards Efective Attack Synthesis against Deep Logical Vulnerabilities in DeFi Protocols. arXiv preprint arXiv:2407.06348 (2024)

[36] Gavin Wood et al. 2014. Ethereum: A secure decentralised generalised transaction ledger. Ethereum project yellow paper 151, 2014 (2014), 1–32

[37] Siwei Wu, Dabao Wang, Jianting He, Yajin Zhou, Lei Wu, Xingliang Yuan, Qinming He, and Kui Ren. 2021. Defiranger: Detecting price manipulation attacks on defi applications. arXiv preprint arXiv:2104.15068 (2021)

[38] Rui Xi, Zehua Wang, and Karthik Pattabiraman. 2024. POMABuster: Detecting Price Oracle Manipulation Attacks in Decentralized Finance. In 2024 IEEE Symposium on Security and Privacy (SP). IEEE Computer Society, 240–240.

[39] Qingren Zeng, Jiahao He, Gansen Zhao, Shuangyin Li, Jingji Yang, Hua Tang, and Haoyu Luo. 2022. EtherGIS: A Vulnerability Detection Framework for Ethereum Smart Contracts Based on Graph Learning Features. In 2022 IEEE 46th Annual Computers, Software, and Applications Conference (COMPSAC) JEEE, 1742-1749

[40] Zhuo Zhang, Brian Zhang, Wen Xu, and Zhiqiang Lin. 2023. Demystifying exploitable bugs in smart contracts. In 2023 IEEE/ACM 45th International Conference on Software Engineering (ICSE). IEEE, 615–627.

[41] Mingqian Zheng, Jiaxin Pei, and David Jurgens. 2023. Is“ a helpful assistant” the best role for large language models? a systematic evaluation of social roles in system prompts. arXiv preprint arXiv:2311.10054 8 (2023).

[42] Denny Zhou, Nathanael Schärli, Le Hou, Jason Wei, Nathan Scales, Xuezhi Wang, Dale Schuurmans, Claire Cui, Olivier Bousquet, Quoc Le, et al. 2022. Least-to-most prompting enables complex reasoning in large language models. arXiv preprint arXiv:2205.10625 (2022).

[43] Liyi Zhou, Kaihua Qin, Antoine Cully, Benjamin Livshits, and Arthur Gervais. 2021. On the just-in-time discovery of profit-generating transaction in defi protocols. In 2021 IEEE Symposium on Security and Privacy (SP). IEEE, 919–936

[44] Liyi Zhou, Xihan Xiong, Jens Ernstberger, Stefanos Chaliasos, Zhipeng Wang, Ye Wang, Kaihua Qin, Roger Wattenhofer, Dawn Song, and Arthu Gervais. 2023. Sok: Decentralized finance (defi) attacks. In 2023 IEEE Symposium on Security and Privacy (SP). IEEE, 2444–2461.

��������: Automated Detection of Price Oracle Manipulations via LLM-Driven Knowledge Mining and Prompt Generation

## A Appendix: Prompts of ��������

## A.1 Prompt Generated by Prompt Generator

## Prompt Generated by Prompt Generator

description: Detect price oracle manipulation vulnerabilities in a smart contract, steps: step: 1, description: Identify the price oracle used by the smart contract. Determine if the price oracle is a decentralized exchange (DEX) or another external price feed., questions: What is the price oracle used by the smart contract? Is the price oracle a decentralized exchange (DEX) or another external price feed? step: 2, description: Analyze the smart contract’s reliance on the price oracle. Determine if the contract uses the price provided by the oracle directly, without any additional validation or cross-checking., questions: Does the smart contract rely solely on the price provided by the price oracle, without any additional validation? Are there any mechanisms in place to detect and mitigate price manipulation in the price oracle? step: 3, description: Evaluate the potential impact of a price oracle manipulation attack on the smart contract. Determine if the attack could lead to financial losses or unfair advantages for the attacker., questions: What are the potential financial consequences if the price oracle is manipulated? Could the price oracle manipulation lead to unfair advantages for the attacker? step: 4, description: Provide recommendations to mitigate the price oracle manipulation vulnerability, such as using multiple price oracles, implementing price validation mechanisms, or using more robust price feeds., questions: What are the recommended mitigation strategies to address the price oracle manipulation vulnerability? How can the smart contract be modified to reduce the risk of price oracle manipulation attacks?

## B Appendix: Knowledge

## B.1 Human-curated Domain Knowledge

## Human-curated Domain Knowledge

Liquidity pools provide the underlying liquidity for decentralized exchanges (DEXs) like Uniswap by holding token reserves. These pools can also act as on-chain price oracles, ofering real-time price data to other applications. When discussing smart contracts vulnerable to price oracle manipulation, we focus on two main categories of vulnerabilities:

1. Vulnerabilities in DEXes and Liquidity Pools/reserves: a) Lack of Slippage Tolerance: Users may sufer unexpected losses due to significant price fluctuations during swaps or trades if the smart contracts do not have adequate slippage tolerance settings. b) Susceptibility to Front-Running or Sandwich Attacks: Smart contracts that do not mitigate front-running or sandwich attacks can expose users to losses when malicious actors manipulate transaction order and timing.

2. Vulnerabilities in Price-Dependent Applications: a) Price Oracle Manipulation: Smart contracts that rely on external price oracles can be manipulated through tampering with the price feed, leading to incorrect asset valuations and financial loss. b) Unfair asset valuation: Poorly designed smart contracts may allow malicious users to manipulate asset valuations, even if the oracle price is accurate. Inadequate safeguards can enable unfair trading practices, disadvantaging other users.

## C Appendix: Illustration of POM

As discussed in Section 1, price oracles are integral to the functionality of DeFi applications but are also vulnerable to manipulation. One example of POM is depicted in Figure 3, which is excerpted from project Behodler<sup>14</sup>.

```javascript
function burnAsset(address asset, uint256 amount) public isLive incrementFate {
    require(assetApproved(asset], "LimboDAO: illegal asset");
    address sender = _msgSender();
    require(ERC677(asset).transferFrom(sender, address(this), amount), "LimboDAO: transferFailed");
    uint256 fateCreated = fateState[_msgSender()].fateBalance;
    uint256 actualEyeBalance = IERC20(domainConfig.eye).balanceOf(asset);
    require(actualEyeBalance > 0, "LimboDAO: No EYE");
    uint256 totalSupply = IERC20(asset).totalSupply();
    uint256 eyePerUnit = (actualEyeBalance * ONE) / totalSupply;
    uint256 impliedEye = (eyePerUnit * amount) / ONE;
    fateCreated = impliedEye * 20;
    fateState[_msgSender()].fateBalance += fateCreated;
    emit assetBurnt(_msgSender(), asset, fateCreated);
}
```  
Fig. 3. Example function burnAsset with potential flash loan atack vulnerability.

The burnAsset function is designed to remove tokens from circulation by burning assets and crediting Fate tokens to users. Fate tokens serve as a governance currency within the ecosystem, granting holders voting power. This function interacts with EYE-based asset tokens, but the asset pricing formula is vulnerable to flash loan manipulation.

Consider a scenario where there are 1000 EYE and 1000 LINK tokens in a UniswapV2 LINK-EYE pool. The pool’s total supply is 1000, and the attacker holds 100 LP tokens. If the attacker calls the burnAsset function to burn their 100 LP tokens, with the formula in line 9-11, he can earn $1 0 0 0 \times 1 0 0 / 1 0 0 0 \times 2 0 = 2 0 0 0$ amount of Fate. Here, 1000 is the actualEyeBalance and 1000 is the pool’s total LP supply. Thus, the attacker rightfully receives 2000 Fate tokens.

However, the attacker can exploit the system by swapping in 1000 EYE and receiving 500 LINK from the poo (according $\tan x \times y = k ,$ ignoring fees for simplicity). The pool then contains 2000 EYE and 500 LINK tokens. The actualEyeBalance becomes 2000, while the pool’s total LP supply and the attacker’s LP tokens remain at 1000 and 100, respectively. After this manipulation, the attacker can call the burnAsset function to burn their LP tokens and receive $2 0 0 0 \times 1 0 0 / 1 0 0 0 \times 2 0 = 4 0 0 0$ amount of Fate tokens.Subsequently, the attacker can swap 500 LINK back into the pool to retrieve their 1000 EYE. Ultimately, the attacker incurs only the transaction fee, yet they gain double the Fate tokens (4000) compared to the legitimate amount (2000). With this increased Fate, the attacker gains more voting power to influence the system’s decisions or can convert Fate to other tokens for direct profit.

This example illustrates how the ratio of pool tokens can be manipulated through flash loans to exploit price oracles, leading to significant imbalances and vulnerabilities in DeFi applications.

## D Appendix: Formulas

$$
\text {Precision} = \frac {\text {True Positives (TP)}}{\text {True Positives (TP)} + \text {False Positives (FP)}}
$$

$$
\text {Recall} = \frac {\text {True Positives (TP)}}{\text {True Positives (TP)} + \text {False Negatives (FN)}}
$$

$$
F _ {1} \text {-score} = 2 \cdot \frac {\text {Precision} \cdot \text {Recall}}{\text {Precision} + \text {Recall}}
$$