# PMA Research Gap Ledger

이 문서는 research gap을 확정하는 목록이 아니라 후보를 누적하고 반증하는 장부다. 한 논문이 기능을 제공하지 않는다는 사실만으로 분야 전체의 gap으로 확정하지 않는다.

## 판정 원칙

gap 후보는 다음 질문을 모두 통과해야 강한 주제 후보가 된다.

1. 가장 가까운 연구 여러 편에서 반복되는가?
2. 논문의 목적 차이가 아니라 실제 미해결 문제인가?
3. 부재가 false positive, false negative, 잘못된 귀속·계산 또는 적용 불능으로 이어지는가?
4. 원인이 기술적으로 식별되어 있는가?
5. 필요한 증거를 현실적으로 관측할 수 있는가?
6. 기존 기법의 단순 조합만으로 이미 해결되지 않는가?
7. 공격 사례와 hard negative에서 개선을 평가할 수 있는가?
8. 이 gap을 반박할 가장 가까운 연구를 적극적으로 확인했는가?

gap의 연구 상태는 저장소 공통 상태인 `PROPOSED / ACCEPTED / REJECTED / SUPERSEDED / UNRESOLVED`만 사용한다. 논문별 증거 판정은 별도로 `지원 / 부분 지원 / 약화 / 반박 / 무관 / 미확인`을 사용한다.

## 현재 Gap 후보

아래 항목은 문헌 검증 대기 중인 가설이다. 현재 사실이나 novelty 선언이 아니다.

| Gap ID | 후보 명제 | 현재 상태 | 연구 우선순위 | 현재 근거 | 가장 강한 반증 조건 | 다음 검증 |
|---|---|---|---|---|---|---|
| GAP-001 | PMA 분석이 protocol-specific decoder, 함수·주소 목록 또는 공격 패턴에 크게 의존해 미등록 protocol·행위를 놓친다. | `UNRESOLVED` | core | 비교표 초안만 존재 | 사전 수동 지식 없이 여러 protocol에서 동등한 semantic recovery를 보인 연구 | 직접 경쟁 연구의 입력·adapter·누락률 감사 |
| GAP-002 | 가격·평가값이 생성되는 지점과 그 의미를 수동 지정 없이 식별하는 능력이 부족하다. | `UNRESOLVED` | core | 비교표 초안만 존재 | arbitrary value-producing state를 자동 식별하고 평가한 연구 | VeriOracle·DeFiTainter·최신 PMA 연구 감사 |
| GAP-003 | 가격 개입에서 왜곡값의 소비와 정산·경제 결과까지 이어지는 인과 경로를 원시 증거로 연결하지 못한다. | `UNRESOLVED` | core | D-001의 문제 가설 | 해당 경로를 직접 복원하고 node·edge 근거까지 평가한 연구 | DeFiRanger·DeFort·CLUE·SMARTCAT vertical comparison |
| GAP-004 | attacker·beneficiary와 consumer·victim·loss bearer가 외부 label 또는 단순 주소 규칙에 의존해 귀속된다. | `UNRESOLVED` | core | 열린 방법론 문제 | 통제·자금·소비·손실 경로로 역할을 파생하고 정답과 대조한 연구 | victim identification 및 forensic 연구 감사 |
| GAP-005 | 거래 balance delta, 실제 순자산 변화, manipulation-attributable gain과 피해자의 반사실 손실이 혼동된다. | `UNRESOLVED` | core | GE-003은 필요자원·담보·단순 profit을 구분해 넓은 표현을 약화하지만, 이 네 값의 분리·검증은 다루지 않음 | 네 값을 명시적으로 분리하고 재현 가능한 반사실로 검증한 연구 | profit·loss·oracle economics 문헌 감사 |
| GAP-006 | 수익성·가격 충격·행위 패턴만으로 판정해 정상 차익거래, 청산, 대형 swap, JIT liquidity와의 원리적 구별이 약하다. | `UNRESOLVED` | core | 비교표 초안만 존재 | hard negative를 명시하고 공격과 구별하는 필요조건을 평가한 연구 | 직접 탐지 연구의 negative sampling·오류 분석 감사 |
| GAP-007 | 이벤트·로그 중심 관측이 내부 ledger나 event-less 상태 변화를 구조적으로 누락한다. | `UNRESOLVED` | supporting | DeFiRanger 사례 메모 후보 | state diff·storage dependency까지 기본 관측하며 누락률을 평가한 연구 | observation model과 artifact 구현 감사 |
| GAP-008 | atomic transaction을 넘는 PMA에 대한 공통 표현과 검증이 제한된다. | `UNRESOLVED` | deferred | D-004에서 v1 비주장으로 제한; GE-005는 이론적 `m` 모델이 존재함을 보여 넓은 표현을 약화 | multi-transaction/multi-block PMA를 실행 증거로 복원하고 end-to-end 평가한 연구 | v1 이후 확장 조사; 현재 core novelty로 사용하지 않음 |

## 논문별 Gap Evidence

한 셀마다 논문의 Evidence ID 또는 정확한 locator를 연결한다.

| Entry ID | Gap ID | Paper ID | 논문별 판정 | 실제 근거 | 논문 목적상 비대상인가 | 실패 결과·영향 | 검토일 |
|---|---|---|---|---|---|---|---|
| GE-001 | GAP-002 | `ORACLES-DEFI-2023` | 무관 | E-002, E-007: AMM price function과 lending consumption 구조를 입력 전제로 둠 | 예 | 자동 surface 식별을 목적으로 하지 않으므로, 기능 부재를 GAP-002의 지지 근거로 사용하지 않음 | 2026-08-12 |
| GE-002 | GAP-003 | `ORACLES-DEFI-2023` | 무관 | E-006-E-008, Sec. 4: intervention, TWAP distortion, lending valuation과 profit을 제한된 이론 사슬로 연결 | 예 | 실제 call·state·data dependency 복원이 목적이 아니므로 GAP-003을 지지하지 않음; 다만 causal schema의 이론 참고자료로 사용 | 2026-08-12 |
| GE-003 | GAP-005 | `ORACLES-DEFI-2023` | 약화 | E-007, E-008, E-012: attack cost, collateral과 simplified profit을 구분함 | 아니오 | 모든 경제량이 혼동된다는 넓은 주장은 성립하지 않음. 다만 balance delta, 실제 순자산, manipulation-attributable gain, victim counterfactual loss의 분리·검증은 남음 | 2026-08-12 |
| GE-004 | GAP-006 | `ORACLES-DEFI-2023` | 무관 | E-011: profitability와 economic feasibility를 모델링하지만 detector나 hard-negative 평가는 없음 | 예 | 수익성만으로 공격을 판정하지 않으며 정상 arbitrage·liquidation과 구별하는 근거로 사용할 수 없음 | 2026-08-12 |
| GE-005 | GAP-008 | `ORACLES-DEFI-2023` | 약화 | E-006, Sec. 4.1: 조작 block 수 `m`, infrequent trading, multi-block MEV 가능성을 모델·논의함 | 아니오 | multi-block이 전혀 모델링되지 않았다는 넓은 주장은 기각해야 함. 실행 증거 기반 복원과 end-to-end 평가는 다루지 않아 좁은 gap은 남음 | 2026-08-12 |

## Gap별 종합 감사

gap 후보를 `ACCEPTED`하거나 논문의 novelty로 사용하기 전에 작성한다.

### GAP-XXX — 제목

- 현재 상태:
- 가장 가까운 연구:
- 반복해서 관측된 한계:
- 예외 또는 해결한 연구:
- 부재가 만드는 실제 실패:
- root cause:
- 우리 해결 가설:
- 필요한 온체인 증거와 관측 가능성:
- 기존 기법의 단순 결합으로 해결 가능한가:
- 공격 vertical slice 결과:
- 대응 hard negative 결과:
- 구현 가능성:
- 평가 설계와 성공 조건:
- 가장 강한 반론:
- 판정: 유지 / 좁힘 / 약화 / 기각 / 미결
- RQ·방법론·데이터·평가에 미치는 영향:
- 사용자 승인:

## Gap 변경 이력

| 날짜 | Gap ID | 이전 상태·문장 | 새 증거·반례 | 판정 | 새 상태·문장 | 영향 |
|---|---|---|---|---|---|---|
| 2026-08-12 | GAP-005 | 경제량이 넓게 혼동된다는 후보 해석 | GE-003: 필요자원·담보·단순 profit을 분리한 모델 존재 | 넓은 표현 약화; 네 경제량의 분리·반사실 검증으로 한정 | `UNRESOLVED`; 현재 표의 좁은 문장 유지 | 이 논문 자체를 gap 지지 인용으로 사용하지 않고, 남은 네 값의 명시적 검증 연구를 탐색 |
| 2026-08-12 | GAP-008 | multi-block 문제를 넓게 미지원으로 해석할 위험 | GE-005: 조작 block 수 `m`과 multi-block MEV를 반영한 이론 모델 존재 | 넓은 표현 약화; 실행 증거 기반 공통 표현·end-to-end 평가로 한정 | `UNRESOLVED`; 현재 표의 좁은 문장 유지 | atomic 이후 조사에서 이론 모델과 실제 복원·검증을 구분 |
