# PMA 관련 연구 분석 템플릿

이 템플릿의 목적은 논문을 요약하는 것이 아니라 다음 세 가지를 검증하는 것이다.

1. 우리가 제기한 문제와 gap이 실제 문헌에서 확인되는가?
2. 각 논문은 `개입 → 왜곡값 → 소비 → 정산 → 수혜·손실` 중 무엇을 직접 복원하고 무엇을 가정하는가?
3. 우리가 생각하는 방법론을 실제 온체인 증거로 구현하고 평가할 수 있는가?

논문마다 이 파일을 복사해 `research/related-work/<paper-id>.md` 형태로 작성한다.

---

## 작성 규칙

### 근거 상태

모든 핵심 답변 앞에 아래 상태를 붙인다.

- `[V] VERIFIED`: 논문 본문·부록·공식 artifact에서 직접 확인함
- `[I] INFERRED`: 명시적 설명이 없어 정황으로 추론함
- `[NR] NOT REPORTED`: 논문이 보고하지 않음
- `[U] UNRESOLVED`: 확인하지 못했거나 해석이 충돌함
- `[NA] NOT APPLICABLE`: 해당 논문의 목적상 적용되지 않음

`[V]`에는 반드시 section, page, figure, table, algorithm, 공식 코드 위치 중 하나를 적는다. 초록이나 다른 논문의 related work 문장만으로 `[V]`를 부여하지 않는다.

### 분리해서 기록할 것

- 저자가 명시적으로 주장한 내용과 우리의 해석
- 방법의 입력과 평가에만 사용한 정답 라벨
- 방법이 직접 복원한 사실과 외부에서 주어진 사실
- 실제 관측값과 반사실·reference 추정값
- 논문이 인정한 한계와 우리가 발견한 한계
- 기능이 없다는 것과, 논문의 목적상 필요하지 않다는 것

---

# 1. 서지정보와 검토 상태

| 항목 | 내용 |
|---|---|
| Paper ID | |
| 제목 | |
| 저자 | |
| 연도 | |
| Venue / 상태 | 출판 / preprint / thesis 등 |
| 버전 날짜 | |
| DOI / 공식 URL | |
| 로컬 원문 | |
| 공식 artifact | 코드·데이터·모델 링크 |
| 검토일 / 검토자 | |
| 검토 상태 | 1차 읽기 / 근거 확인 / artifact 확인 / 완료 |

## 관련성 판정

- 분류: `핵심 PMA / 인접 탐지 / 의미 복원 / 귀속·손익 / 오라클·경제 모델 / 배경 / 제외`
- 우리 RQ와 연결되는 이유:
- 직접 비교 대상인가, 구성요소 참고 대상인가:
- 제외한다면 이유:

> `관련성이 낮다`와 `방법이 나쁘다`를 혼동하지 않는다.

---

# 2. 논문이 푸는 문제

## 저자의 문제 정의

- `[상태]` 저자가 명시한 문제:
- 근거 위치:
- 대상 현상:
- 분석 단위: contract / call / transaction / bundle / block / multi-block / 기타
- 실행 시점: 배포 전 / 실행 전 / 실행 중 / 사후
- 출력: 취약점 / 경보 / 공격 라벨 / 인과 설명 / 공격자 / 피해자 / P&L / exploit / 방어

## 저자의 핵심 주장

| 주장 | 저자 주장 또는 우리의 해석 | 근거 위치 | 검증 방식 | 현재 판단 |
|---|---|---|---|---|
| | 저자 주장 / 해석 | | | `[V/I/U]` |

## 우리의 문제와의 관계

- 직접 해결하는 부분:
- 전제로 주어지는 부분:
- 다루지 않는 부분:
- 이 논문을 근거로 말할 수 없는 것:

---

# 3. Scope와 Threat Model

| 질문 | 답 | 근거 | 상태 |
|---|---|---|---|
| 어떤 공격 또는 이상을 대상으로 하는가? | | | |
| PMA를 어떻게 정의하는가? | | | |
| 성공한 공격만 보는가, 취약성도 보는가? | | | |
| 명시적으로 제외하는 현상은 무엇인가? | | | |
| 공격자가 할 수 있는 일은 무엇인가? | | | |
| 공격자가 알아야 하는 정보는 무엇인가? | | | |
| flash loan 또는 atomicity를 가정하는가? | | | |
| 여러 주소·컨트랙트의 공모를 허용하는가? | | | |
| 공격 성공 또는 피해의 기준은 무엇인가? | | | |
| 대상 chain·VM·protocol 범위는 무엇인가? | | | |

### 범위 반례

- 이 정의가 공격으로 포함하지만 정상일 수 있는 사례:
- 실제 PMA이지만 이 정의에서 제외될 수 있는 사례:
- 시간 범위 때문에 놓치는 사례:

---

# 4. Observation Model: 무엇을 실제로 보는가

| 증거 | 입력 여부 | 용도 | 필수/선택 | 획득 방법 | 누락 시 영향 | 근거 |
|---|:---:|---|:---:|---|---|---|
| Transaction / block metadata | | | | | | |
| Internal call trace | | | | | | |
| Opcode trace / stack / memory | | | | | | |
| Event / log | | | | | | |
| Storage read·write / state diff | | | | | | |
| Native asset transfer | | | | | | |
| ERC-20/721/1155 transfer | | | | | | |
| Contract bytecode | | | | | | |
| Verified source / ABI | | | | | | |
| Protocol-specific decoder | | | | | | |
| 함수명·주소·토큰 whitelist | | | | | | |
| Oracle / market price | | | | | | |
| Historical state | | | | | | |
| Mempool / ordering 정보 | | | | | | |
| 공격자·피해자·공격 유형 label | | | | | | |
| Off-chain report / 수동 annotation | | | | | | |
| 기타 | | | | | | |

## 입력과 정답의 분리

- 탐지·분석 입력으로 들어가는 사전 지식:
- 평가에만 사용하는 라벨:
- paper artifact에 포함되어 있지만 실사용 시 얻기 어려운 정보:
- preferred source → fallback → 둘 다 없을 때 결과:

---

# 5. 최소 인과 사슬 커버리지

아래 표는 논문이 각 요소를 언급하는지가 아니라 **원시 증거에서 직접 복원하거나 검증하는지**를 기록한다.

| 인과 요소 | 논문의 표현 | 생성·추출 근거 | 직접 복원 / 수동 입력 / 가정 / 미지원 | 실패·모호성 | 근거 위치 |
|---|---|---|---|---|---|
| 행위자 후보 | | | | | |
| 통제 관계·주소 군집 | | | | | |
| 가격 또는 상태에 대한 개입 | | | | | |
| 개입으로 인한 상태 변화 | | | | | |
| 파생된 경제값·평가값 | | | | | |
| 정상값/reference/counterfactual | | | | | |
| 왜곡 판정 | | | | | |
| 왜곡값의 read·consumption | | | | | |
| 의사결정·정산 | | | | | |
| 자산·부채·청구권 변화 | | | | | |
| 최종 수혜자 | | | | | |
| 공격자 귀속 | | | | | |
| 직접 consumer | | | | | |
| 피해자·loss bearer | | | | | |
| 실제 P&L | | | | | |
| manipulation-attributable gain | | | | | |
| 피해자 반사실 손실 | | | | | |

## 인과 연결의 강도

- 논문이 실제로 연결하는 가장 긴 경로:
- 중간에 외부 label이나 수동 지식으로 건너뛰는 지점:
- 이벤트의 시간적 인접성을 데이터 의존성으로 간주하는가:
- 인과관계가 아니라 상관·패턴인 부분:
- 완전한 causal witness를 만들 수 없는 이유:

---

# 6. Semantic Model과 그래프

## 계층

| 계층 | 표현 단위 | 생성 규칙 | UNKNOWN 보존 여부 | 근거 |
|---|---|---|---|---|
| 원시 실행 증거 | | | | |
| Semantic action | | | | |
| 경제·인과 역할 | | | | |

## Semantic action

- semantic action의 목적:
- 포함 기준:
- action 목록을 만드는 방법: 수동 ontology / ABI / event / trace semantics / 학습 / 기타
- protocol-specific adapter 필요 여부:
- 하나의 low-level 동작이 여러 의미를 가질 때 처리:
- 복원 실패 시 처리:
- 새 protocol·action에 대한 확장 규칙:

## Graph 또는 중간 표현

| 항목 | 내용 | 근거 |
|---|---|---|
| Node 종류 | | |
| Edge 종류 | | |
| 각 node·edge의 원시 근거 | | |
| control flow | | |
| data flow | | |
| token/value flow | | |
| state dependency | | |
| 시간 관계 | | |
| 주소·주체 관계 | | |
| 그래프가 보존하지 못하는 정보 | | |

---

# 7. Distortion, Profit, Loss 계산

| 계산 대상 | 정의·수식 | 기준 시점·창 | 가격 단위 | 수수료·가스·부채 처리 | reference/counterfactual | 근거 |
|---|---|---|---|---|---|---|
| 가격·평가값 왜곡 | | | | | | |
| 단순 balance delta | | | | | | |
| Transaction P&L | | | | | | |
| 공격자 실제 순자산 변화 | | | | | | |
| 조작 귀속 추가이익 | | | | | | |
| 피해자 손실 | | | | | | |
| Bad debt / 사회화된 손실 | | | | | | |

### 계산 감사

- profit이 공격 판정 조건인가, 결과 보고값인가:
- 공격자 이익과 피해자 손실이 같다고 가정하는가:
- 비유동성·LP token·debt token·NFT를 어떻게 평가하는가:
- 중간 자금 조달과 최종 회수를 어떻게 구분하는가:
- 계산을 재현할 수 있는가:
- 수동 정답 또는 반사실 실행과 대조했는가:

---

# 8. 최종 판정과 정상 반례

## 판정 규칙

- 탐지·분류 predicate 또는 모델 출력:
- 필수조건:
- threshold와 선정 근거:
- 학습·규칙 생성에 필요한 label:
- 공격 유형을 미리 알아야 하는가:

## Hard negative

| 정상 또는 경계 사례 | 데이터에 포함 | 구별 근거 | 별도 평가 결과 | 남는 혼동 |
|---|:---:|---|---|---|
| 정상 차익거래 | | | | |
| 정상 청산 | | | | |
| 대형 swap | | | | |
| JIT liquidity | | | | |
| MEV backrun / sandwich | | | | |
| Donation / rebase 등 정상 상태 변화 | | | | |
| 가격 충격은 있으나 외부 소비가 없는 거래 | | | | |
| 이익은 있으나 manipulation이 없는 거래 | | | | |

- 수익성 이외의 구별 원리:
- 가장 가능성 높은 false positive:
- 가장 가능성 높은 false negative:

---

# 9. Evaluation과 Ground Truth

| 항목 | 내용 | 근거 |
|---|---|---|
| Dataset 기간·chain | | |
| Positive 수와 family 분포 | | |
| Negative 수와 선택 방식 | | |
| Hard negative 구성 | | |
| Ground truth 출처 | | |
| Annotation 절차·검토자 일치도 | | |
| Train/validation/test 분리 | | |
| 시간·protocol leakage 방지 | | |
| Baseline | | |
| Metric | | |
| Ablation | | |
| 오류 분석 | | |
| 재현 가능성 | | |

## 평가가 실제로 보장하는 것

- 결과가 뒷받침하는 주장:
- 결과만으로 뒷받침되지 않는 주장:
- 데이터 구성 때문에 과대평가될 수 있는 부분:
- 새로운 protocol·attack family에 대한 일반화 근거:

---

# 10. 가정·한계·재현성 감사

| 항목 | 저자 명시 | 우리 분석 | 심각도 | 근거 |
|---|---|---|:---:|---|
| 관측 누락 | | | | |
| protocol-specific knowledge | | | | |
| 사전 공격 label·pattern | | | | |
| 시간 범위 | | | | |
| 주소 통제·귀속 | | | | |
| reference/counterfactual | | | | |
| 이익·손실 계산 | | | | |
| hard negative | | | | |
| dataset bias / leakage | | | | |
| artifact·재현성 | | | | |

---

# 11. 우리 연구에 대한 판정

## 후보 gap별 판정

| Gap 후보 | `지원 / 부분 지원 / 반박 / 무관 / 미확인` | 논문의 실제 근거 | 예외·한정어 |
|---|---|---|---|
| protocol-specific 사전 지식 의존 | | | |
| 가격·평가 지점 자동 식별 부족 | | | |
| 개입과 소비·정산의 인과 연결 부족 | | | |
| 공격자·피해자 귀속 부족 | | | |
| 실제 P&L과 조작 귀속 이익 혼동 | | | |
| 정상 유사 행위와의 구별 부족 | | | |
| atomic transaction 범위 제한 | | | |
| 이벤트·로그 중심 관측 누락 | | | |

> 위 gap은 검증 후보이며, 현재 저장소의 G1~G7을 자동으로 사실로 인정하지 않는다.

## 이 논문에서 가져올 수 있는 것

- 재사용 가능한 정의:
- 재사용 가능한 관측·분석 기법:
- 재사용 가능한 데이터·평가 방식:
- 그대로 가져오면 안 되는 가정:
- 우리 설계를 바꿀 수 있는 반례:

## 한 문장 결론

> 이 논문은 ______를 ______ 근거로 제공하지만, ______를 ______로 가정하거나 다루지 않으므로, 우리의 ______ gap을 `지원/부분 지원/반박`한다.

---

# 12. Evidence Ledger

핵심 비교표의 각 셀을 이 표의 근거 ID와 연결한다.

| Evidence ID | 확인하려는 명제 | 논문 위치 | paraphrase | 증거 종류 | 상태·신뢰도 | 충돌·주의사항 |
|---|---|---|---|---|---|---|
| E-001 | | Sec./p./Fig./Table/Alg. | | 본문 / 부록 / 코드 / 데이터 | `[V/I/U]` | |

---

# 13. Master Matrix용 한 줄 요약

아래 값은 Evidence Ledger가 채워진 뒤에만 작성한다.

| 필드 | 값 |
|---|---|
| Paper ID | |
| Category | |
| Scope / time horizon | |
| Required prior knowledge | |
| Primary evidence | |
| Intervention | `D/M/P/0/?/—` |
| Distorted value | `D/M/P/0/?/—` |
| Consumption | `D/M/P/0/?/—` |
| Settlement | `D/M/P/0/?/—` |
| Attacker / beneficiary | `D/M/P/0/?/—` |
| Victim / loss bearer | `D/M/P/0/?/—` |
| Actual P&L | `D/M/P/0/?/—` |
| Counterfactual gain/loss | `D/M/P/0/?/—` |
| Hard-negative principle | `D/M/P/0/?/—` |
| Evaluation strength | |
| Main limitation | |
| Gap verdict | |
| Evidence IDs | |

코드 의미:

- `D`: 방법이 관측 증거로부터 직접 derive함
- `M`: 수동 지정·외부 label·protocol knowledge로 주어짐
- `P`: 일부만 복원하거나 heuristic으로 추정함
- `0`: 산출하지 않음
- `?`: 아직 확인하지 못함
- `—`: 논문 목적상 해당 없음

---

# 14. 남은 검증 작업

- [ ] 본문 근거 위치 재확인
- [ ] 부록 확인
- [ ] 공식 코드에서 실제 구현 확인
- [ ] Dataset·label 생성 절차 확인
- [ ] 저자 주장과 실험 보장의 차이 확인
- [ ] 우리 gap을 반박하는 요소 확인
- [ ] 대표 공격 사례에 수동 적용
- [ ] hard negative에 수동 적용

