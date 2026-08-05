# Related Work 비교 정리 (PMA)

> README 전체 75편 템플릿. Venue는 비움(필요 시 나중에). 채우면서 `?` → 값 / ✔.

## 범례

| 기호 | 의미 |
|:---:|:---|
| ✔ / ~ / ? | 확인 완료 / 추정 / 미확인 |
| ● / ◐ / ○ / — | 지원 / 부분 / 미지원 / 해당없음 |
| **TBD** | 우리 쪽 미확정 |

---

## 0. 축 설계 근거 (왜 이 7개인가)

PMA 탐지는 아래 4단계 질문으로 분해된다. 축은 이 질문에 1:1로 대응해야 한다.

```
① 무엇이 "가격"인가        → 오라클/가격 지점 식별        → G2
② 그 값이 조작되었는가      → 조작 유형 커버리지 + 시간 범위 → G3, G6
③ 조작이 피해로 이어졌는가  → 인과 연결 · 손실 귀속        → G4
④ 정상 행위와 어떻게 구분   → 차익거래·청산 판별           → G5
   (전 단계 공통) 얼마나 손으로 짜맞췄나 / 무엇을 관측하나  → G1, G7
```

> **서술 열로 강등한 것 (●/○ 아님):** 시점(When)·대응(Resp)·메커니즘(Mech).
> 배치 모델이지 보장이 아니고, `DET-SC`(배포 전)와 `DET-TX`(사후)를 같은 ●/○ 축에 놓으면 표가 왜곡된다.
>
> **버린 것:** "설명가능성" — ML/LLM 계열 몇 편 빼면 전부 ●이라 변별력이 없고, 형식화 없이는 guarantee로 인정받지 못한다. 필요하면 G4(손실 귀속 근거 제시)에 흡수시킬 것.

---

## 1. 우리 포지셔닝

### 1-1. 한 줄
> **TBD** — `<입력>`으로 `<보장>`하에 `<범위의 PMA>`를 `<시점>`에 탐지/차단.

### 1-2. G1~G7 열 정의

| ID | 축 | 묻는 질문 | 값 기준 (●) | 기존 한계 예시 |
|:---:|:---|:---|:---|:---|
| **G1** | **사양 비의존성** | 사람이 미리 짜맞춘 것이 얼마나 필요한가 — 프로토콜 어댑터/decoder, 함수 시그니처·주소 WL, 공격 패턴 목록, 라벨 데이터 | 넷 다 불필요 | DeFiRanger: 프로토콜별 decoder → 미등록 프로토콜에서 FN 295/309 (95%) |
| **G2** | **가격·오라클 지점 식별** ★ | "무엇이 가격으로 쓰이는 값인가"를 어떻게 아는가 | 수동 지정 없이 자동 식별 | 대부분 오라클 함수/주소를 사전 지정. 미지정 sink는 원천 미탐 |
| **G3** | **조작 유형 일반성** | 어떤 조작 벡터까지 하나의 모델로 커버하는가 — spot/reserve, TWAP, LP share·totalSupply, 외부 feed, donation·rebase | 5종 이상 | 유형별 패턴 하드코딩 → 새 유형마다 패턴 추가 |
| **G4** | **인과 연결 · 손실 귀속** ★ | 조작된 값이 소비된 지점(sink)까지 추적하고, 피해자·손실액을 산출하는가 | sink 추적 + 피해자·손실액 | "공격 tx다"에서 끝남. 대응·검증 불가 |
| **G5** | **정상 행위 판별** ★ | 차익거래·청산·JIT LP·대형 스왑과 구분하는 **원리**가 있는가 (수익성만으로 판정하지 않는가) | 원리적 구분 기준 보유 | hoard-and-dump + 수익성은 정상 차익거래와 동형 → FP의 근원 |
| **G6** | **시간 범위** | 단일 atomic tx를 넘어서는가 | multi-tx / multi-block | atomic tx 가정 → TWAP 조작·지연 공격 원천 미탐 |
| **G7** | **관측 완전성** | 이벤트 없이 내부 ledger만 바꾸는 상태 변화를 포착하는가 | 상태 diff까지 관측 | 이벤트/로그만 보면 구조적 누락 |


### 1-3. 주장하지 않을 것
- **TBD**

---

## 2. 비교 표 (README 전체)

> `Cat`: `DET-TX` · `DET-SC` · `DEF` · `INV` · `SYN` · `ORA` · `INF` · `SOK`
> `When`: 배포 전 / 실행 전(mempool) / 실행 중 / 사후 · `Resp`: 탐지 / 경보 / 차단 / 구제 / 합성

| # | Work | Year | Cat | When | Resp | Mech | G1 | G2 | G3 | G4 | G5 | G6 | G7 | Gap |
|:--:|:---|:---:|:---:|:---|:---|:---|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---|
| 0 | **Ours** | **TBD** | **TBD** | **TBD** | **TBD** | **TBD** | **TBD** | **TBD** | **TBD** | **TBD** | **TBD** | **TBD** | **TBD** | — |
| 1 | [Sereum](./Sereum/Sereum.md) | 2019 | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? |
| 2 | [Improved Price Oracles](./Improved%20Price%20Oralces/Improved%20Price%20Oralces.md) | 2020 | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? |
| 3 | [Securing Smart Contract with Runtime Valid…](./SolyThesis/SolyThesis.md) | 2020 | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? |
| 4 | [SODA](./SODA/SODA.md) | 2020 | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? |
| 5 | [TxSpector](./TxSpector/TxSpector.md) | 2020 | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? |
| 6 | [ÆGIS](./AEGIS/AEGIS.md) | 2020 | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? |
| 7 | [An Empirical Study of DeFi Liquidations](./sok_survey/An%20Empirical%20Study%20of%20DeFi%20Liquidations/An%20Empirical%20Study%20of%20DeFi%20Liquidations.md) | 2021 | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? |
| 8 | [Attacking the DeFi Ecosystem with Flash Lo…](./sok_survey/Attacking%20the%20DeFi%20Ecosystem%20with%20Flash%20Loans%20for%20Fun%20and%20Profit/Attacking%20the%20DeFi%20Ecosystem%20with%20Flash%20Loans%20for%20Fun%20and%20Profit.md) | 2021 | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? |
| 9 | [BLOCKEYE](./BlockEye/BlockEye.md) | 2021 | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? |
| 10 | [On the Just-In-Time Discovery of Profit-Ge…](./DeFiPoser/DeFiPoser.md) | 2021 | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? |
| 11 | [Smart Contract Vulnerabilities](./sok_survey/Smart%20Contract%20Vulnerabilities/Smart%20Contract%20Vulnerabilities.md) | 2021 | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? |
| 12 | [SoK](./sok_survey/SoK_Oracles%20from%20the%20Ground%20Truth%20to%20Market%20Manipulation/SoK_Oracles%20from%20the%20Ground%20Truth%20to%20Market%20Manipulation.md) | 2021 | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? |
| 13 | [The Eye of Horus](./Horus/Horus.md) | 2021 | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? |
| 14 | [TWAP Oracle Attacks](./TWAP%20Oracle%20Attacks/TWAP%20Oracle%20Attacks.md) | 2021 | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? |
| 15 | [Clockwork Finance](./Clockwork%20Finance/Clockwork%20Finance.md) | 2022 | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? |
| 16 | [Demystifying Exploitable Bugs in Smart Con…](./sok_survey/Demystifying%20Exploitable%20Bugs%20in%20Smart%20Contracts/Demystifying%20Exploitable%20Bugs%20in%20Smart%20Contracts.md) | 2022 | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? |
| 17 | [Detecting Flash Loan Based Attacks in Ethe…](./Detecting_Flash_Loan_Based_Attacks_in_Ethereum/Detecting_Flash_Loan_Based_Attacks_in_Ethereum.md) | 2022 | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? |
| 18 | [InvCon](./InvCon/InvCon.md) | 2022 | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? |
| 19 | [Oracles in Decentralized Finance](./sok_survey/Oracles%20in%20Decentralized%20Finance/Oracles%20in%20Decentralized%20Finance.md) | 2022 | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? |
| 20 | [Quantifying Blockchain Extractable Value](./sok_survey/Quantifying%20Blockchain%20Extractable%20Value/Quantifying%20Blockchain%20Extractable%20Value.md) | 2022 | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? |
| 21 | [Security Analysis of DeFi](./sok_survey/Security%20Analysis%20of%20DeFi/Security%20Analysis%20of%20DeFi.md) | 2022 | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? |
| 22 | [SoK](./sok_survey/SoK_Decentralized%20Finance/SoK_Decentralized%20Finance.md) | 2022 | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? |
| 23 | [Time-travel Investigation](./EthScope/EthScope.md) | 2022 | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? |
| 24 | [Uniswap v3 TWAP Oracles in Proof of Stake](./UniswapV3%20TWAP%20Oracles%20in%20Proof%20of%20Stake/UniswapV3%20TWAP%20Oracles%20in%20Proof%20of%20Stake.md) | 2022 | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? |
| 25 | [A Robust Front-Running Methodology for Mal…](./FrontDef/FrontDef.md) | 2023 | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? |
| 26 | [BACKRUNNER](./BackRunner/BackRunner.md) | 2023 | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? |
| 27 | [Beyond the Public Mempool](./Beyond_the_Public_Mempool_final/Beyond_the_Public_Mempool_final.md) | 2023 | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? |
| 28 | [Blockchain Large Language Models](./BlockGPT/BlockGPT.md) | 2023 | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? |
| 29 | [DeFiRanger](./DeFiRanger/DeFiRanger.md) | 2023 | `DET-TX` | 사후/실시간 | 경보 | CFT+lifting+패턴8 | ○ | ○ | ◐ | ◐ | ○ | ○ | ○ | decoder FN295; evt-less 누락; 패턴 밖 미탐; 수익성 기준 |
| 30 | [DeFiTainter](./DeFiTainter/DeFiTainter.md) | 2023 | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? |
| 31 | [DeFiWarder](./DeFiWarder/DeFiWarder.md) | 2023 | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? |
| 32 | [Flash Loan Attack Is More Than Just Price …](./Flash%20Loan%20Attack%20Is%20More%20Than%20Just%20Price%20Oracle%20Manipulation/Flash%20Loan%20Attack%20Is%20More%20Than%20Just%20Price%20Oracle%20Manipulation.md) | 2023 | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? |
| 33 | [ItyFuzz](./ItyFuzz/ItyFuzz.md) | 2023 | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? |
| 34 | [POMABuster](./POMABuster/POMABuster.md) | 2023 | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? |
| 35 | [SoK](./sok_survey/SoK_Decentralized%20Finance%20Attacks/SoK_Decentralized%20Finance%20Attacks.md) | 2023 | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? |
| 36 | [The Blockchain Imitation Game](./Blockchain%20Imitation%20Game/Blockchain%20Imitation%20Game.md) | 2023 | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? |
| 37 | [Timely Identification of Victim Addresses …](./Identifying_Victims_in_DeFi_Attacks/Identifying_Victims_in_DeFi_Attacks.md) | 2023 | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? |
| 38 | [Toward Automated Detecting Unanticipated P…](./VeriOracle/VeriOracle.md) | 2023 | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? |
| 39 | [Your Exploit is Mine](./STING/STING.md) | 2023 | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? |
| 40 | [BlockScan](./BlockScan/BlockScan.md) | 2024 | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? |
| 41 | [DeFiGuard](./DeFiGuard/DeFiGuard.md) | 2024 | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? |
| 42 | [DeFort](./DeFort/DeFort.md) | 2024 | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? |
| 43 | [Demystifying Invariant Effectiveness for S…](./Trace2Inv/Trace2Inv.md) | 2024 | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? |
| 44 | [FlashSyn](./FlashSyn/FlashSyn.md) | 2024 | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? |
| 45 | [FORAY](./FORAY/FORAY.md) | 2024 | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? |
| 46 | [Instrumenting Transaction Trace Properties…](./Instrumenting%20Transaction%20Trace%20Properties%20in%20Smart%20Contracts/Instrumenting%20Transaction%20Trace%20Properties%20in%20Smart%20Contracts.md) | 2024 | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? |
| 47 | [Midas](./Midas/Midas.md) | 2024 | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? |
| 48 | [Revealing Adversarial Smart Contracts thro…](./FinDet/FinDet.md) | 2024 | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? |
| 49 | [Safeguarding DeFi Smart Contracts against …](./OVer/OVer.md) | 2024 | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? |
| 50 | [SecPLF](./SecPLF/SecPLF.md) | 2024 | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? |
| 51 | [SMARTINV](./SmartInv/SmartInv.md) | 2024 | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? |
| 52 | [SmartOracle](./SmartOracle/SmartOracle.md) | 2024 | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? |
| 53 | [AI Agent Smart Contract Exploit Generation](./AI%20Agent%20Smart%20Contract%20Exploit%20Generation/AI%20Agent%20Smart%20Contract%20Exploit%20Generation.md) | 2025 | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? |
| 54 | [AiRacleX](./AiRacleX/AiRacleX.md) | 2025 | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? |
| 55 | [Automated Attack Synthesis for Constant Pr…](./CPMMX/CPMMX.md) | 2025 | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? |
| 56 | [Automated Invariant Generation for Solidit…](./InvCon+/InvCon+.md) | 2025 | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? |
| 57 | [Detecting Various DeFi Price Manipulations…](./DeFiScope/DeFiScope.md) | 2025 | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? |
| 58 | [Enhancing Smart Contract Security Analysis…](./CLUE/CLUE.md) | 2025 | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? |
| 59 | [EvoPoC](./EvoPoC/EvoPoC.md) | 2025 | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? |
| 60 | [FLAMES](./Flames/Flames.md) | 2025 | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? |
| 61 | [Following Devils' Footprint](./SMARTCAT/SMARTCAT.md) | 2025 | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? |
| 62 | [LookAhead](./LookAhead/LookAhead.md) | 2025 | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? |
| 63 | [On-Chain Decentralized Learning and Cost-E…](./On-Chain%20Decentralized%20Learning%20and%20Cost-Effective%20Inference%20for%20DeFi%20Attack%20Mitigation/On-Chain%20Decentralized%20Learning%20and%20Cost-Effective%20Inference%20for%20DeFi%20Attack%20Mitigation.md) | 2025 | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? |
| 64 | [Ormer](./Ormer/Ormer.md) | 2025 | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? |
| 65 | [Penetrating the Hostile](./DeFiTail/DeFiTail.md) | 2025 | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? |
| 66 | [PropertyGPT](./PropertyGPT/PropertyGPT.md) | 2025 | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? |
| 67 | [Smart Contract Fuzzing Towards Profitable …](./VERITE/VERITE.md) | 2025 | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? |
| 68 | [Surviving in Dark Forest](./EVScope/EVScope.md) | 2025 | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? |
| 69 | [TraceLLM](./TraceLLM/TraceLLM.md) | 2025 | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? |
| 70 | [Cost of Manipulation in AMM-Based Oracles](./Cost%20of%20Manipulation%20in%20AMM-Based%20Oracles/Cost%20of%20Manipulation%20in%20AMM-Based%20Oracles.md) | 2026 | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? |
| 71 | [Decentralized finance security](./sok_survey/Decentralized%20finance%20security/Decentralized%20finance%20security.md) | 2026 | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? |
| 72 | [DeFiTrace](./DeFiTrace/DeFiTrace.md) | 2026 | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? |
| 73 | [Enforcing Control Flow Integrity on DeFi S…](./CrossGuard/CrossGuard.md) | 2026 | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? |
| 74 | [HOUSTON](./HOUSTON/HOUSTON.md) | 2026 | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? |
| 75 | [LLM-Powered Detection of Price Manipulatio…](./PMDetector/PMDetector.md) | 2026 | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? | ? |
