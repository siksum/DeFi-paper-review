# 전체 논문 통합 비교 — DeFi Price Manipulation

> 최종 갱신: 2026-08-12
> 이 파일은 **검토를 완료한 논문의 공통 비교 정본**이다. 아직 읽지 않은 논문의 빈 행을 미리 만들지 않는다. 전체 보유 논문 목록과 읽기 상태는 [PMA 논문 목록](./README.md), gap별 세부 판정은 [리서치갭 통합 매트릭스](../논문읽기_Research_OS_v3/리서치갭_통합_매트릭스_템플릿_v3.md), 재사용할 근거는 [인용·아이디어 뱅크](../논문읽기_Research_OS_v3/인용_아이디어_뱅크_템플릿.md)에서 관리한다.

## 1. 현재 포함된 논문

| ID | 논문 | Venue / Year | Track | 현재 연구에서의 역할 | 검토 상태 |
|---|---|---|---|---|---|
| P1 | [Oracles in Decentralized Finance: Attack Costs, Profits and Mitigation Measures](./sok_survey/Oracles%20in%20Decentralized%20Finance/내용%20정리.md) | Entropy 2023 | Primary B / Secondary C | AMM·TWAP oracle의 조작자원, lending 담보, 단순 profit을 구분하는 경제모델 | 원문 확인 완료; 수치 재현 미수행 |
| P2 | [DeFiRanger: Detecting DeFi Price Manipulation Attacks](./DeFiRanger/관련%20연구%20분석.md) | IEEE TDSC 2023 | A | transaction trace를 CFT와 semantic action으로 바꿔 8개 pattern을 탐지하는 직접 baseline | 원문·표·Discussion 확인; 공개 구현 없음 |
| P3 | [DeFort: Automatic Detection and Analysis of Price Manipulation Attacks in DeFi Applications](./DeFort/관련%20연구%20분석.md) | ISSTA 2024 | A | 역사 가격 이상과 관련 주소 profit을 결합하고 역할·fund flow를 분석하는 직접 baseline | 원문·artifact 확인; 구현 비공개 |
| P4 | [DeFiScope: Detecting Various DeFi Price Manipulations with LLM Reasoning](./DeFiScope/관련%20연구%20분석.md) | ASE 2025 | Primary A / Secondary D | Transfer Graph와 LLM 가격 방향을 8개 pattern에 결합하는 직접 baseline | 원문·보충자료·공식 code/dataset 확인; 재실행 미수행 |
| P5 | [Sereum: Protecting Existing Smart Contracts Against Re-Entrancy Attacks](./Sereum/관련%20연구%20분석.md) | NDSS 2019 | Primary A / Secondary D | 동적 taint와 호출 트리로 실제 storage→control 의존성을 검증하는 인접 방법론; PMA 직접 baseline은 아님 | 원문·공식 공격 예제 확인; 전체 구현·재실행 미확인 |

논문 유형이 다르므로 P1의 기능 부재를 직접 detector의 gap 근거로 사용하지 않는다. P2~P4는 직접 PMA 탐지 방법이므로 입력·predicate·출력·평가에서 확인된 부재와 실패 사례를 제한된 gap 근거로 사용할 수 있다. P5는 재진입 방어 논문이므로 PMA 기능 부재는 gap 근거로 세지 않고, 실행 데이터 의존성·관측 가능성·의미 손실에 관한 인접 근거만 사용한다.

## 2. 한눈에 보는 방법과 출력

| Paper | 핵심 질문 | 주요 입력 | 중간 표현 | 실제 판정·계산 | 방법 출력 |
|---|---|---|---|---|---|
| P1 Oracles | 주어진 oracle을 조작하려면 자원·담보가 얼마나 필요한가? | AMM reserve·curve, oracle window, 목표가격, LTV 등 protocol parameter | 목표 TWAP→AMM 조작자원→최소담보·단순 profit | detector가 아니라 사전 경제성 모델 | attack cost/resource, minimum collateral, 단순화된 profit·mitigation |
| P2 DeFiRanger | atomic transaction의 자금 흐름이 알려진 PMA pattern과 일치하는가? | call trace, events, native/ERC-20 transfer, 선택적 protocol external information | CFT→basic action→semantic action | 8개 Type I/II pattern과 cost/revenue 후보 관계 | PMA 경보·pattern; 독립 역할·피해손실 출력 없음 |
| P3 DeFort | 역사 범위를 벗어난 가격과 관련 주소의 양의 이익이 함께 나타나는가? | target metadata, execution trace, events, RPC state, price history, 외부 token price | PCM/HCM/BCM/PDM 기반 behavior state model | price anomaly AND 관련 주소 subset의 환산 `Out-In` profit > 0 | 경보, attacker/victim/profiteer 후보, fund flow, associated function |
| P4 DeFiScope | semantic operation 순서와 pool·token 가격 방향이 알려진 PMA pattern과 일치하는가? | raw transaction, call trace, ERC-20 Transfer, source·ABI, balance/supply delta | time-indexed Transfer Graph→6 operations + LLM price direction | 4 family·8 direction-aware pattern match | PMA 후보·pattern·operation·price direction; 역할·P&L 없음 |
| P5 Sereum | 실행 중인 기존 contract가 inconsistent storage를 이용하는 재진입을 수행하는가? | EVM opcode, stack·memory·storage, 동적 call/return | dynamic call tree + shadow taint + storage write lock | 재진입 subtree의 `SLOAD→JUMPI` 의존 주소를 이전 invocation이 `SSTORE`하면 위반 | 경보, transaction abort·rollback; 역할·경제값·손익 없음 |

## 3. 인과 사슬과 경제 결과 비교

기호:

- `O`: 논문 방법이 명확히 지원
- `△`: 제한적·heuristic·부분 지원
- `X`: 방법이 산출하지 않음
- `—`: 논문 목적상 비대상이며 gap 근거로 세지 않음

| Capability | P1 Oracles | P2 DeFiRanger | P3 DeFort | P4 DeFiScope | P5 Sereum |
|---|:---:|:---:|:---:|:---:|:---:|
| 가격·평가 surface 자동 식별 | △ | △ | △ | △ | — |
| 정상/reference value 구성 | O | X | △ | X | — |
| attacker-induced distortion 확인 | O | △ | △ | △ | — |
| 왜곡값의 실제 소비 확인 | △ | △ | △ | △ | O* |
| 소비→정산·경제 결과 연결 | △ | △ | △ | △ | △* |
| attacker/beneficiary 귀속 | — | X | △ | X | — |
| victim/loss bearer 귀속 | — | X | △ | X | — |
| 실제 transaction P&L | X | X | △ | X | — |
| manipulation-attributable gain | X | X | X | X | — |
| victim counterfactual loss | X | X | X | X | — |
| 정상 유사 행위와의 원리적 구별 | — | △ | △ | △ | △* |
| multi-transaction/multi-block 실행 증거 | △ | X | △ | X | X* |
| event-less state 변화 관측 | — | X | △ | X | O* |

중요한 해석:

- P1의 `O`는 주어진 모델·surface·parameter 안의 이론 계산을 뜻하며, 실제 사건에서 surface를 자동 발견하거나 역할·손익을 복원했다는 의미가 아니다.
- P2의 profitability는 실제 순자산 변화가 아니라 pattern용 cost/revenue 후보 관계다.
- P3는 역할과 환산 profit을 출력하므로 “기존 연구는 역할·이익을 전혀 출력하지 않는다”는 주장을 반박한다. 다만 control clustering, full NAV, causal consumption, 반사실 손익은 없다.
- P4의 price direction은 reference 대비 정확한 왜곡량이 아니며, operation order는 return-value→settlement dataflow가 아니다.
- P5의 `*`는 **재진입 storage-control domain에 한정**된다. `SLOAD→JUMPI→SSTORE` 의존성과 event-less 실행 상태는 직접 관측하지만, 이것을 PMA의 왜곡값 소비·경제 정산·역할·손익 증거로 해석할 수는 없다.

## 4. 관측·사양 의존성과 실패 범위

| Paper | 사람·사양 의존 | 관측되지 않으면 생기는 문제 | 실제 보고된 실패·반례 |
|---|---|---|---|
| P1 Oracles | AMM/oracle/consumer 모델과 parameter가 주어져야 함 | 실제 경로·fee·token mechanics가 다르면 경제성 계산을 직접 이전할 수 없음 | 실제 incident replay와 역할·손실 검증 없음 |
| P2 DeFiRanger | semantic rule, 8 attack patterns, 일부 protocol external information | 미등록 internal ledger·event-less action·pattern 밖 공격 누락 | semantic FN 309 중 295건이 external information 부족과 관련; 정상 fee/buyback mechanics로 26 FP (`E-012/014`) |
| P3 DeFort | target metadata, price template/signature/custom strategy, history DB, 외부 가격 | complex value model이나 trace return이 없으면 PCM·price read 실패 | BeltFinance complex price model과 Discover trace return 누락 FN (`E-019/028`) |
| P4 DeFiScope | price-function keyword, verified source·compile, 6-operation ontology, 8 patterns; Type-II CPMM 가정 | no-source·compile failure·non-ERC20·closed custom model·cross-tx·정밀 수치계산에서 실패 | D1 miss 19건; source missing 3, compile 5, cross-tx 8, non-ERC20 1, 정밀 수치계산 2 (`E-011/012`) |
| P5 Sereum | 수정 EVM, transaction-local 실행, storage word 단위 taint, 기본 `JUMPI` sink | 실행되지 않은 경로·분기 없는 소비를 놓치고 source field·trust 의미가 없어 정상 패턴을 경보할 수 있음 | packed field, delete/zero, constructor callback, tightly-coupled contracts, manual mutex 오탐 (`SER-E10`); 이후 artifact 확장은 논문 평가와 분리 (`SER-E14`) |

## 5. 평가 결과 비교

| Paper | Positive / detection 평가 | Semantic·analysis 평가 | Benign·운영 평가 | 해석 한계 |
|---|---|---|---|---|
| P1 Oracles | 실제 incident detection 없음 | simulation으로 AMM·LTV별 조작자원 비교 | 없음 | 이론 경제모델이며 empirical detector benchmark가 아님 |
| P2 DeFiRanger | 26 incident day의 92,325,423 tx에서 155 alert, 129 TP·26 FP | 15,272 tx·8,117 actions에서 precision 0.996, recall 0.962 | 14 zero-day와 15 historical incident를 수동 조사 후 보고 | incident-day 모집단은 전체 PMA recall을 주지 않음; 공개 artifact 없음 |
| P3 DeFort | D1 54건 중 52건 탐지 | 52건 중 50건의 associated function·behavior description을 수동 확인 | assumed-normal 435 apps·428,523 tx에서 alert 0; 5-chain live deployment 5건 보고 | 역할·금액·causal edge별 metric과 complete negative GT 없음 |
| P4 DeFiScope | D1 95건 중 76건 탐지 | 1,000 tx의 204 operations에서 TG precision 0.984, recall 0.912 | D2 suspicious 968건: 147/153 TP; D3 random benign 96,800건: FP 0 | D2는 profit-biased suspicious set, D3는 hard-negative family별 GT가 아님; end-to-end 재실행 전 |
| P5 Sereum | cross-function·delegated·create-based crafted attack 각 1건 탐지 | 77,987,922 tx replay에서 49,080 경보; DAO 2,294건·DSEthToken 43건 확인 | 52 관련 contract를 16 code group으로 묶어 6개 source group 분석; 9.6% runtime overhead | 0.063%는 전체 tx 대비 경보율이지 alert precision이 아님; 전체 attack 모집단 recall·현대 chain 일반화 없음 |

## 6. 논문이 서로 바꾼 현재 판단

### P1 Oracles가 추가한 것

- 필요 유동성·조달 규모, 회수 불가능한 공격비용, lending 담보, 단순 profit을 같은 값으로 취급하면 안 된다.
- oracle window·AMM curve·초기 reserve·consumer LTV가 경제적 실행 가능성을 바꾼다.
- 실제 P&L과 manipulation-attributable gain으로 직접 이전하려면 incident replay와 counterfactual이 추가로 필요하다.

### P2 DeFiRanger가 추가한 것

- raw trace를 semantic action으로 lifting한 뒤 pattern을 판정하는 구조는 유용한 baseline이다.
- protocol external information 부족이 실제 semantic coverage 하락으로 이어졌다.
- pattern과 profitability 후보만으로는 정상 fee·buyback mechanics를 배제하지 못했다.

### P3 DeFort가 수정한 것

- 기존 연구에도 attacker/victim/profiteer·fund flow·profit 후보를 출력하는 시스템이 있으므로 “역할·손익 출력이 전혀 없다”는 넓은 gap은 유지할 수 없다.
- 남는 문제는 역할 label의 존재가 아니라 controller·beneficiary·final loss bearer의 근거와 role별 정확도다.
- `Out-In` 환산 profit은 actual NAV나 manipulation-attributable gain과 구분해야 한다.

### P4 DeFiScope가 수정한 것

- 모든 custom price model에 사람이 직접 식을 작성해야 한다는 넓은 주장은 약화된다. open-source code와 LLM으로 가격 방향을 추론하는 방법이 있다.
- 그러나 source·compile·keyword·closed-model 의존은 남고 실제 FN으로 나타났다.
- direction+semantic pattern은 탐지 후보를 만들지만 exact distortion·value consumption·역할·손익을 계산하지 않는다.

### P5 Sereum이 수정한 것

- 실제 실행의 stack·memory·storage를 추적하면 호출 존재나 시간적 인접성보다 강한 `storage read → control decision → later write` 데이터 의존성을 만들 수 있다.
- 따라서 “event가 없으면 모든 state/data 의존성을 관측할 수 없다”는 넓은 주장은 약화된다. 다만 수정 EVM이 필요하고, 논문은 PMA 경제값·정산을 추적하지 않는다.
- semantics 없이도 program dependency는 판정할 수 있지만 packed field·trust·manual lock 의미를 잃어 오탐이 생겼다. program dependence와 economic meaning을 별도 계층으로 유지해야 한다.

## 7. 현재 Gap 통합 판정

모든 항목은 아직 `UNRESOLVED`다. 아래 표는 확정된 novelty 선언이 아니라 다음 문헌과 vertical slice로 반증할 후보를 요약한다.

| Gap | 현재까지의 지원 근거 | 넓은 주장을 약화하는 근거 | 다음 검증 |
|---|---|---|---|
| GAP-001 protocol-specific 지식 의존 | P2 external information FN, P3 price adapter FN, P4 source·compile FN | P3 공통 behavior core, P4 source+LLM custom-model 해석 | DeFiTainter no-adapter·unknown-protocol 비교 |
| GAP-002 surface/reference 자동 식별 | P2 call 후보, P3 template/signature, P4 keyword candidate는 exact surface·reference가 아님 | P3 PCM/history, P4 code reasoning이 부분 자동화 | VeriOracle·value-flow 연구 감사 |
| GAP-003 개입→소비→정산 연결 | P2~P4 모두 semantic/temporal association은 있으나 return-value dataflow 없음 | P3 fund flow, P4 operation+direction, P5의 instruction-level storage→control 의존성이 부분 해결 방향을 제공 | 실제 사건 execution vertical slice에서 external return→settlement까지 확장 가능성 검증 |
| GAP-004 역할·loss bearer 귀속 | P2·P4 독립 schema 없음; P3도 controller·final bearer·role metric 없음 | P3가 역할 후보를 실제 출력 | victim/beneficiary attribution 연구 |
| GAP-005 actual·counterfactual 손익 분리 | 네 논문 모두 full NAV·조작 귀속 gain·victim counterfactual loss 미제공 | P1이 자원·비용·profit 개념을 구분하고 P3가 부분 profit 계산 | canonical ledger·counterfactual replay |
| GAP-006 정상 행위 discrimination | P2 26 FP, P4 6 FP; family별 hard-negative 부재. P5는 인접 domain에서 field·trust 의미 손실이 오탐으로 이어짐을 보임 | P3 anomaly+profit, P4 direction+pattern과 대규모 benign 평가; P5는 실제 dependency로 단순 순서 규칙을 강화 | arbitrage·liquidation·JIT·대형 swap 대조 |
| GAP-007 관측 완전성 | P2 event/internal-ledger, P3 trace return, P4 Transfer/source 실패가 FN으로 연결 | P3 RPC state·history, P4 source, P5의 modified-EVM opcode·stack·memory·storage 관측이 전면 부재 주장을 약화 | 공개 replay에서 state diff·opcode dataflow의 비용·배치·누락 비교 |
| GAP-008 multi-block 실행 증거 | P2·P4 single tx; P4 cross-tx 8건 miss | P1의 `m` 이론모델, P3 history·related-flow가 전면 부재 주장 약화 | v1 이후 multi-block causal window 검증 |

## 8. 현재 연구 포지션

현재 방어 가능한 통합 주장은 다음과 같다.

> `PROPOSED`: 기존 연구는 이론적 공격 경제성, transaction semantic pattern, 역사 가격 이상+profit, LLM 가격 방향을 제공하고, 인접 연구는 instruction-level data dependency도 구현한다. 그러나 공개 실행·상태 증거에서 **개입→정확한 왜곡값→실제 소비→정산→역할·actual/counterfactual gain·loss**를 하나의 검증 가능한 표현으로 연결하는지는 아직 확인되지 않았다.

이 문장은 최종 novelty나 contribution이 아니다. 직접 경쟁 연구와 대표 사건·정상 반례를 더 검토한 뒤에만 좁히거나 기각한다. 현재 `ACCEPTED` 범위와 출력 결정은 [연구 결정 로그](../research/DECISIONS.md)를 따른다.

## 9. 업데이트 규칙

새 논문을 읽은 뒤 이 파일에서는 다음만 갱신한다.

1. 검토 완료 논문 목록과 track
2. 방법·입력·중간 표현·판정·출력 비교
3. 역할·손익·시간·관측 범위
4. 평가 denominator와 failure 사례
5. 기존 gap을 지원·약화·반박하는 변화
6. 현재 통합 주장에 미치는 영향

세부 Evidence ID, 반증 조건, 가설과 상태 변경 이력은 이 파일에 중복해서 늘리지 않고 활성 [리서치갭 통합 매트릭스](../논문읽기_Research_OS_v3/리서치갭_통합_매트릭스_템플릿_v3.md)에 기록한다.
