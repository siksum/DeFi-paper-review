# 연구 OS 상태보드

> 이 파일은 현재 연구가 어디까지 확정되었는지 한눈에 보기 위한 운영 파일이다.
> 자동화/에이전트가 작업할 때 가장 먼저 읽고, 중요한 결과를 낼 때 마지막에 갱신한다.
>
> 최종 갱신: 2026-08-12. 이 파일은 템플릿 사본이 아니라 현재 프로젝트의 운영 상태보드다.
> 연구 결정 상태는 `PROPOSED / ACCEPTED / REJECTED / SUPERSEDED / UNRESOLVED`만 사용한다.

---

# 1. 현재 연구 상태

```yaml
project:
  name: DeFi PMA causal evidence representation
phase: LITERATURE_REVIEW
version: v0.2

research_problem:
  value: 실제 발생한 DeFi PMA를 온체인 실행·상태 증거에서 개입→왜곡값→소비→정산→수혜·손실로 복원한다.
  status: PROPOSED

core_gap:
  value: 현재 GAP-001~008은 후보이며 직접 경쟁 연구와 hard negative 검증 전이다.
  status: UNRESOLVED

research_question:
  value: 최종 RQ 문구와 contribution 우선순위는 관련 연구 비교 후 결정한다.
  status: UNRESOLVED

key_insight:
  value: 역할·왜곡·손익을 독립 라벨이 아니라 하나의 causal evidence representation에서 파생한다.
  status: PROPOSED

methodology:
  value: Methodology Kernel v0.1과 대표 사건 vertical slice가 다음 설계 산출물 후보이다.
  status: PROPOSED

evaluation:
  value: atomic v1에서 공격·hard negative, 실제 P&L·반사실 계산, UNKNOWN을 분리 평가한다.
  status: UNRESOLVED
```

---

# 2. 현재 ACCEPTED Decisions

> `research/DECISIONS.md`의 요약이다. 새 반증이 있어도 자동 변경하지 않고 버전·영향을 기록한다.

| ID | Decision | Evidence | Accepted at | Version |
|---|---|---|---|---|
| D-001 | causal evidence representation을 1차 산출물로 둠 | 사용자 DQ-01 | 2026-08-10 | v1.0 |
| D-002 | 사후 forensic analysis 우선 | 사용자 DQ-02 | 2026-08-10 | v1.0 |
| D-003 | 실행되어 경제 결과가 발생한 PMA 우선 | 사용자 DQ-03 | 2026-08-10 | v1.0 |
| D-004 | 공통 표현은 확장 가능하게, v1 실증은 atomic으로 제한 | 사용자 DQ-04 | 2026-08-10 | v1.0 |
| D-005 | 공개 온체인 증거 필수, source·ABI 선택, 사고 label은 평가만 | 사용자 DQ-05 | 2026-08-10 | v1.0 |
| D-006 | EVM 및 지정된 다중 chain 후보군 | 사용자 DQ-06 | 2026-08-10 | v1.0 |
| D-007 | 역할·손익을 목표 출력으로 하되 근거 부족 시 UNKNOWN | 사용자 DQ-07 | 2026-08-10 | v1.0 |
| D-010 | FP·FN·UNKNOWN을 분리 보고 | 사용자 DQ-10 | 2026-08-10 | v1.0 |

---

# 3. Human Approval Queue

| ID | 제안 | 왜 중요한 결정인가? | Critic 결과 | 필요한 승인 |
|---|---|---|---|---|
| DQ-08 | protocol-specific decoder를 core와 adapter 중 어디까지 허용할지 | 일반성 주장과 복원률이 달라짐 | 관련 연구·관측 가능성 감사 전 | 문헌·vertical slice 후 사용자 승인 |
| DQ-09 | 표현·탐지·dataset 중 주 contribution | RQ·평가·논문 구조 전체에 영향 | novelty 검증 전 | 비교 연구 후 사용자 승인 |

---

# 4. Open Questions

| Q-ID | Question | 관련 논문/Gap | 필요한 Evidence | 우선순위 |
|---|---|---|---|---|
| Q-01 | 조건부 최소 순간자금을 계산할 때 어떤 경로와 reference를 고정할 것인가? | ORACLES-DEFI-2023; GAP-005 | 실제 사건 balance timeline과 경로별 재실행 | 높음 |
| Q-02 | 실제 P&L·조작 귀속 추가이익·victim loss를 어떻게 분리할 것인가? | GAP-005 | 공통 가치기준과 counterfactual replay | 높음 |
| Q-03 | 왜곡값의 생성·소비 surface를 수동 label 없이 찾을 수 있는가? | GAP-002·003 | trace·storage dependency vertical slice | 높음 |
| Q-04 | 정상 차익거래·청산·대형 swap과 PMA를 어떤 필요조건으로 구분할 것인가? | GAP-006 | 대응 hard-negative 사례 | 높음 |
| Q-05 | multi-block 이론 모델과 실제 실행 증거 복원 사이의 차이를 어떻게 평가할 것인가? | GAP-008 | multi-block incident와 ground truth | 낮음(v1 이후) |
| Q-06 | protocol adapter가 없을 때의 coverage 하락을 어떻게 측정하고 UNKNOWN으로 보존할 것인가? | `DeFiRanger-2023` E-005·012·016; GAP-001·007 | adapter/no-adapter ablation과 미등록 protocol 평가 | 높음 |
| Q-07 | 자기 pool의 역사 가격 범위를 reference로 쓸 때 정상 시장 이동·저유동성·점진적 오염을 어떻게 구분할 것인가? | `DeFort-2024` E-004·011·021; GAP-002·006 | 독립 reference와 정상 대형 swap·arbitrage 대조 | 높음 |

---

# 5. Current Gap Candidates

| Gap | 상태 | Supporting Papers | Opposing/Solving Papers | 다음 검증 |
|---|---|---|---|---|
| GAP-001 | UNRESOLVED | DeFiRanger E-005·012와 DeFort E-008~010·019·028에서 수동 protocol/value 지식 의존과 관련 FN이 반복 | DeFort의 behavior core와 일부 mainstream template는 범용화를 부분 제공 | DeFiScope·DeFiTainter의 no-adapter coverage 비교 |
| GAP-002 | UNRESOLVED | DeFiRanger E-008은 call 존재, DeFort E-009·010·019는 template/signature/custom strategy로 surface를 찾음 | DeFort는 PCM과 역사 bound를 제공해 단순 부재 주장을 약화 | VeriOracle·value-flow 연구와 자동 surface 비교 |
| GAP-003 | UNRESOLVED | DeFiRanger E-006~010과 DeFort E-005·013~015·031은 action/price-read/fund-flow를 연결하지만 return value→정산 dataflow는 없음 | DeFort는 anomaly→profit→행동분석의 더 긴 경로를 제공 | BGLD/LeetSwap vertical slice로 edge별 근거 대조 |
| GAP-004 | UNRESOLVED | DeFort E-013·022는 attacker/victim/profiteer를 출력해 '귀속 없음'을 약화하지만 control cluster·component metric·final loss bearer는 없음 | DeFiRanger는 독립 attribution schema가 없음 | victim/beneficiary attribution 연구 감사 |
| GAP-005 | UNRESOLVED | DeFort E-006·012·032는 `Out-In` 환산 profit을 계산하나 full NAV·반사실 gain/loss는 없음 | Oracles는 자원·담보·단순 profit을 구분해 넓은 주장을 약화 | canonical ledger와 counterfactual 연구 감사 |
| GAP-006 | UNRESOLVED | DeFiRanger E-014는 정상 mechanics FP를 보고; DeFort E-002·021은 anomaly+profit과 assumed-normal D2 0 alert를 제공 | DeFort가 수익성 단독 판정보다 강해 넓은 주장을 약화하지만 hard-negative family별 검증 없음 | 정상 대형 swap·arbitrage·liquidation 대조 |
| GAP-007 | UNRESOLVED | DeFiRanger E-005·016과 DeFort E-019·028에서 event/trace·return-value 관측 누락이 실제 FN으로 연결 | DeFort는 RPC state·prior DB로 일부 보완 | state diff·opcode dataflow 연구 감사 |
| GAP-008 | UNRESOLVED | DeFiRanger는 single tx; DeFort E-005·014는 history와 관련 multi-transaction flow를 추적 | Oracles의 `m` 모델과 DeFort가 '완전 미지원' 주장을 약화하지만 실행 증거 창·cross-tx 인과는 미보고 | v1 이후 multi-block 연구 감사 |

---

# 6. Hypotheses

| H-ID | Hypothesis | 상태 | 반증 조건 | 다음 실험/검증 |
|---|---|---|---|---|
| H-01 | 동일 왜곡이라도 evaluation surface와 초기상태에 따라 필요한 자본이 달라진다. | PROPOSED | 여러 surface에서 공통식이 실제 실행비용을 정확히 설명 | Constant Product·StableSwap 사건 재현 |
| H-02 | 수익성만으로 PMA와 정상 경제행위를 구별할 수 없다. | PROPOSED | profitability가 hard negative 없이도 충분조건임을 보인 연구 | arbitrage·liquidation 대조 |
| H-03 | protocol-specific enrichment를 선택 adapter로 분리하고 no-adapter 성능을 함께 보고해야 일반성과 복원률을 정직하게 평가할 수 있다. | PROPOSED | adapter 없이도 미등록 protocol에서 동등한 semantic coverage를 보이는 방법 | DeFiRanger 재평가 또는 후속 연구의 ablation 감사 |
| H-04 | 자기 pool의 역사 가격 이상과 시간적으로 관련된 양의 이익만으로는 manipulation-attributable causality를 보장할 수 없다. | PROPOSED | 정상 시장 충격·arbitrage에서도 해당 조합이 발생하지 않거나 인과적으로 모두 배제됨 | DeFort predicate를 정상 대형 swap·backrun에 적용 |

---

# 7. Methodology Decisions

| D-ID | Decision | 상태 | 버전 | 다음 Gate |
|---|---|---|---|---|
| D-001~007, D-010 | 범위·증거·출력 방향 | ACCEPTED | v1.0 | Definition/Observation gate |
| D-008 | protocol-specific 지식 허용 수준 | UNRESOLVED | v0.1 | 문헌·vertical slice |
| D-009 | 핵심 contribution | UNRESOLVED | v0.1 | Gap validation |

---

# 8. Contradictions

| ID | Topic | Evidence A | Evidence B | 상태 |
|---|---|---|---|---|
| CON-01 | attack cost의 의미 | ORACLES-DEFI-2023은 이론적 AC·담보·profit을 계산 | 실제 사건 P&L에는 fee·debt·unwind·valuation 필요 | UNRESOLVED; 직접 이전 금지 |
| CON-02 | platform-independent의 의미 | `DeFiRanger-2023`은 Ethereum·BSC에 적용 가능한 일반 방법을 주장 | E-005·012·016은 internal ledger용 수동 정보와 ERC-20 event 의존을 보고 | UNRESOLVED; core 이식성과 adapter-free coverage를 분리해야 함 |
| CON-03 | 자동화된 공격 분석의 의미 | `DeFort-2024`는 attacker·victim·profiteer·fund flow·associated function을 자동 출력하고 50/52 분석 성공을 보고 | E-013·022·031·032는 역할·금액·causal edge별 metric, control clustering, 반사실 손익이 없음을 보임 | UNRESOLVED; 사건 요약 정확도와 세부 귀속 정확도를 분리해야 함 |

---

# 9. Counterexample Queue

| ID | Claim/Method | Counterexample | 심각도 | 상태 |
|---|---|---|---|---|
| CE-01 | 가격 안정성이 조작 저항성을 뜻함 | StableSwap 예제는 안정적 가격에도 낮은 조작자원을 보임 | 높음 | 확인; 일반화는 금지 |
| CE-02 | 플래시론 원금이 공격비용임 | 원금은 상환 가능한 조달 규모이고 실제 비용과 다름 | 높음 | 방법론 반영 후보 |
| CE-03 | 수익이 양수면 공격임 | 정상 arbitrage·liquidation도 양의 수익 가능 | 높음 | hard-negative 검증 대기 |
| CE-04 | DeFi action pattern과 수익성이면 PMA임 | transfer-fee token·DEX fee·buyback이 같은 형태를 만들어 DeFiRanger에서 26 FP 발생 | 높음 | `DeFiRanger-2023` E-014 확인; discrimination 조건 미확정 |
| CE-05 | state/balance/totalSupply 호출 존재가 실제 가격 의존을 증명함 | 호출값이 소비되지 않거나 다른 분기로 버려질 수 있음 | 높음 | value-level data dependency 검증 대기 |
| CE-06 | 역사 가격 이상+관련 주소 이익이면 PMA임 | 정상 시장 급변·저유동성 대형 swap 뒤 arbitrageur도 같은 조건을 만족할 수 있음 | 높음 | DeFort D2는 family별 반례를 명시하지 않아 검증 대기 |
| CE-07 | 가격 이상 이후 손실 flow가 있는 pool이 최종 victim임 | 직접 pool 손실이 LP·lender·user·bad debt로 전가되거나 반대로 회복될 수 있음 | 높음 | loss bearer vertical slice 대기 |

---

# 10. Literature Queue

| Priority | Paper | 읽는 이유 | 검증할 Gap/Hypothesis |
|---|---|---|---|
| 1 | DeFiScope | 확장 benchmark에서 DeFort의 price-model coverage와 일반성 재검증 | GAP-001~003·007 |
| 2 | DeFiTainter | value-flow가 DeFort의 adapter·return-value 한계를 해결하는지 확인 | GAP-001~003 |
| 3 | 실제 incident replay 기반 P&L 연구 | DeFort `Out-In`과 realized P&L·victim loss 비교 | GAP-004·005 |
| 4 | TWAP Oracle Attacks: Easier Done than Said? | multi-block·MEV 가정과 비용 정의 확인 | GAP-008, Q-05 |

---

# 11. 자동 업데이트 체크리스트

새 논문 완료 시:

- [x] Paper Evidence DB: Oracles E-001~012, DeFiRanger E-001~018, DeFort E-001~035
- [x] Literature Map: 세 논문의 related research seed와 DeFort 후속 DeFiScope 후보 기록
- [x] Comparison Matrix: P1(Track B/C), P2·P3(Track A)의 기능·predicate 비교
- [x] Coverage Matrix: 이론 모델, atomic detector, history+related-flow detector 범위 기록
- [x] Limitation Matrix: adapter/event/rule, return-value, 역할·P&L heuristic, 공개 code 부재 기록
- [x] Contradiction Matrix: cost 의미, platform independence, 자동 분석 출력 강도 기록
- [x] Gap Ledger: DeFort가 GAP-004·006·008의 넓은 표현을 약화하면서 좁은 causal gap을 지원함을 기록
- [x] Hypothesis Ledger: H-01~H-04 후보 등록
- [x] Methodology Decision 영향: DeFort가 DQ-08의 core/adapter 분리 근거를 강화했으나 ACCEPTED 변경 없음
- [x] Counterexample Queue: CE-01~07 등록
- [x] Human Approval Queue: 새 승인 요청 없음

---

# 12. 다음 작업

```text
NEXT_ACTION: DeFiScope 또는 DeFiTainter로 DeFort의 adapter·value model·return-value dataflow 한계를 반증하거나 반복 확인
WHY: DeFort가 역할·자금 흐름까지 출력해 넓은 gap 주장을 약화했지만 component-level 정확도와 causal linkage는 아직 미검증
EXPECTED_OUTPUT: P4 근거 장부와 GAP-001~007 판정의 유지·좁힘·약화·반박
STOP_CONDITION: 다음 논문 지정 또는 기존 문헌의 직접 비교 완료
```

---

# 운영 원칙

> 반복 작업은 자동화하되 중요한 연구 판단은 명시적으로 승인한다.
> UNKNOWN은 제거 대상이 아니라 관리 대상이다.
> 확정된 결정은 새 반증 없이 변경하지 않는다.
> 논문 한 편이 끝날 때마다 이 파일, 리서치갭 통합 매트릭스, 인용·아이디어 뱅크를 함께 갱신한다.
