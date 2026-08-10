# PMA 연구 사용자 결정 대기열

최종 갱신일: 2026-08-10

이 문서는 사용자가 선택해야 할 연구 방향과, 문헌·사례 증거를 먼저 조사해야 하는 방법론 문제를 구분한다. 확정 상태는 각 항목과 `research/DECISIONS.md`를 따르며, 추천안만 있고 사용자가 확정하지 않은 항목은 `PROPOSED` 또는 `UNRESOLVED`로 취급한다.

## 1. 결정 책임 구분

### 사용자가 결정할 것

논문의 목적, 범위, 감수할 trade-off처럼 증거만으로 자동 결정되지 않는 연구 방향이다. 에이전트가 선택지와 영향을 설명하고 추천할 수 있지만 최종 선택은 사용자에게 있다.

### 에이전트가 먼저 조사할 것

정의, 그래프 schema, semantic action, 계산식처럼 기술적 타당성이 필요한 항목이다. 사용자가 처음부터 정답을 골라야 하는 문제가 아니다. 에이전트가 문헌 근거, 사례, 반례, 구현 가능성을 비교한 뒤 후보안을 제시해야 한다.

### 공동 확정할 것

에이전트가 증거와 추천안을 제시하고, 사용자가 논문의 목표 및 감당 가능한 범위와 대조하여 승인한다. 승인된 결과만 `DECISIONS.md`에 기록한다.

---

## 2. 확정된 1차 결정

DQ-01부터 DQ-05까지의 연구 방향은 2026-08-10에 확정되었다.

### DQ-01 — 논문의 1차 산출물

- 상태: `ACCEPTED` → `DECISIONS.md` D-001
- 질문: 이 연구가 최우선으로 산출해야 하는 것은 무엇인가?
- 선택지:
  - A. 공격 transaction 분류·탐지
  - B. 공격의 인과 경로 복원과 설명
  - C. 인과 경로를 핵심 표현으로 만들고 탐지·귀속·손익을 그 위에서 파생
- 추천안: **C**
- 추천 이유: 사용자가 제기한 attacker, victim, distortion, consumption, settlement, profit 문제를 한 구조에서 검증할 수 있다.
- 영향: RQ, graph schema, 평가 단위, baseline, 논문 contribution 전체
- 사용자 결정: **C**

### DQ-02 — 분석 시점

- 상태: `ACCEPTED` → `DECISIONS.md` D-002
- 질문: 사후 분석, 실시간 탐지, 실행 전 차단 중 어디까지 주장할 것인가?
- 선택지:
  - A. 사후 forensic analysis
  - B. 실시간 또는 실행 중 탐지
  - C. 실행 전 예방·차단
- 추천안: **A를 먼저 확정**하고, 성능이 허용될 때 B 가능성을 별도 평가
- 추천 이유: 인과 복원과 반사실 계산의 타당성을 먼저 검증할 수 있고, latency 제약이 방법론 정의를 왜곡하는 것을 피할 수 있다.
- 영향: 사용 가능한 trace·state, 알고리즘 비용, 평가 metric
- 사용자 결정: **A**

### DQ-03 — 대상 현상

- 상태: `ACCEPTED` → `DECISIONS.md` D-003
- 질문: 무엇을 연구 대상으로 부를 것인가?
- 선택지:
  - A. 실제 발생해 경제적 결과가 확정된 PMA
  - B. 아직 실행되지 않은 price-manipulation vulnerability
  - C. PMA를 포함한 광범위한 DeFi economic exploit
- 추천안: **A**
- 추천 이유: 실제 실행 증거, 수혜·손실, 정상 반례를 대조할 수 있어 초기 방법론 검증이 가능하다.
- 영향: positive dataset, ground truth, threat model, 탐지 단위
- 사용자 결정: **A**

### DQ-04 — 시간 범위

- 상태: `ACCEPTED` → `DECISIONS.md` D-004
- 질문: 단일 atomic transaction과 multi-transaction/multi-block 공격을 어디까지 포함할 것인가?
- 선택지:
  - A. atomic transaction만 지원하고 범위를 명시
  - B. 처음부터 multi-transaction/multi-block까지 지원
  - C. 공통 schema는 넓게 설계하되 v1 검증은 atomic으로 제한하고 이후 확장
- 추천안: **C**
- 추천 이유: multi-block 문제를 정의에서 지우지 않으면서 초기 구현과 ground truth 구성을 통제할 수 있다.
- 주의: v1 결과로 multi-block 탐지를 지원한다고 주장하면 안 된다.
- 영향: graph의 시간 node·edge, 상태 snapshot, counterfactual window, dataset
- 사용자 결정: **C**

### DQ-05 — 허용할 증거의 경계

- 상태: `ACCEPTED` → `DECISIONS.md` D-005
- 질문: 방법이 사용할 수 있는 정보와 평가에만 사용할 정보를 어떻게 나눌 것인가?
- 선택지:
  - A. 공개 온체인 실행·상태 증거만 방법 입력으로 사용
  - B. verified source, ABI, protocol 문서까지 방법 입력으로 사용
  - C. 사고 보고서와 알려진 attacker/victim label도 입력으로 사용
- 추천안: **A를 필수 기반으로 하고 B는 선택적 enrichment**, C는 ground truth·평가에만 사용
- 추천 이유: 재현 가능성과 일반성을 유지하면서 semantic recovery가 실제로 가능한 범위를 측정할 수 있다.
- 영향: 사양 비의존성 주장, semantic action 복원률, 대상 protocol 범위
- 사용자 결정: **A를 필수 기반으로 하고 B는 선택적 enrichment, C는 평가에만 사용**

---

## 3. 추가 결정 상태

### DQ-06 — 대상 생태계

- 질문: Ethereum mainnet, EVM-compatible chains, non-EVM까지 어디를 대상으로 할 것인가?
- 추천안: Ethereum/EVM으로 방법론과 평가를 먼저 닫고 외적 타당성은 별도로 기술
- 상태: `ACCEPTED` → `DECISIONS.md` D-006
- 사용자 결정: EVM 대상. Ethereum, BSC, Optimism, Avalanche의 EVM 범위, Arbitrum, Base를 후보군에 포함

### DQ-07 — 최종 출력의 필수성과 UNKNOWN 허용

- 질문: 모든 탐지 결과에 attacker, victim, gain, loss를 강제로 출력할 것인가?
- 추천안: 각 항목을 파생하되 근거가 부족하면 `UNKNOWN`과 신뢰 근거를 출력. 강제 귀속하지 않음
- 상태: `ACCEPTED` → `DECISIONS.md` D-007
- 사용자 결정: 각 항목의 출력을 목표로 하되 근거 부족 시 `UNKNOWN`과 신뢰 근거를 출력

### DQ-08 — protocol-specific 지식의 허용 수준

- 질문: protocol별 decoder·함수 목록·주소 whitelist를 어느 수준까지 허용할 것인가?
- 추천안: core 판정에는 수동 protocol rule을 요구하지 않고, 선택적 adapter의 추가 효과를 분리 평가
- 상태: `UNRESOLVED` → `DECISIONS.md` D-008
- 사용자 결정: 관련 연구와 복원 가능성 분석 후 다시 결정

### DQ-09 — 핵심 contribution의 우선순위

- 질문: 새 표현·방법론, 새 탐지 알고리즘, 새 dataset 중 무엇을 논문의 주 contribution으로 둘 것인가?
- 추천안: 관련 연구 분석 전에는 확정하지 않음. 현재 후보는 causal evidence representation과 그 파생 방법론
- 상태: `UNRESOLVED` → `DECISIONS.md` D-009
- 사용자 결정: 관련 연구 분석 전에는 확정하지 않음. 현재 후보는 causal evidence representation과 그 파생 방법론

### DQ-10 — 평가에서 중시할 오류

- 질문: false positive와 false negative 중 어느 위험을 더 크게 볼 것인가?
- 추천안: 사용 시점과 대응 방식이 정해진 후 결정. 사후 연구 분석에서는 두 오류와 `UNKNOWN`을 분리 보고
- 상태: 현재 연구 단계의 보고 정책 `ACCEPTED` → `DECISIONS.md` D-010; 운영 우선순위는 `UNRESOLVED`
- 사용자 결정: 사후 분석에서는 false positive, false negative, `UNKNOWN`을 분리 보고하고 우선순위는 사용 시점·대응 방식이 정해진 뒤 결정

---

## 4. 사용자가 지금 혼자 정하면 안 되는 사항

아래는 문헌과 실제 사례를 분석한 뒤 에이전트가 후보안을 제시해야 한다.

| ID | 기술적 결정 | 먼저 필요한 증거 |
|---|---|---|
| MQ-01 | PMA의 필요조건·충분조건 | 기존 정의, 포함·제외 사례, 정상 반례 |
| MQ-02 | 최소 causal witness | 실제 공격 vertical slice와 관측 가능한 trace·state |
| MQ-03 | graph node·edge schema | 각 node·edge의 원시 생성 근거와 누락 분석 |
| MQ-04 | semantic action 최소 ontology | 여러 protocol에서 반복되는 경제 효과와 복원 실패율 |
| MQ-05 | distortion과 reference value | AMM·oracle·share price별 경제 모델과 반사실 가능성 |
| MQ-06 | consumption·settlement 판정 | data/state dependency와 실제 실행 사례 |
| MQ-07 | attacker·beneficiary 귀속 | control 관계, 자금 조달·회수 경로, hard cases |
| MQ-08 | consumer·victim·loss bearer 귀속 | 직접 소비와 최종 손실 부담의 분리 사례 |
| MQ-09 | P&L·조작 귀속 이익·피해 손실 계산 | 수동 계산 정답, 수수료·부채·비유동성 자산 처리 |
| MQ-10 | 최종 detection predicate | positive와 hard negative에 대한 discrimination test |
| MQ-11 | threshold | 개발 세트와 독립 평가 세트, 민감도 분석 |
| MQ-12 | baseline·metric·dataset | RQ와 위협 모델, ground-truth 품질 감사 |

이 항목에 대해 에이전트는 다음 형식으로 제안해야 한다.

1. 후보안
2. 문헌 근거
3. 실제 사례 적용 결과
4. 반례·실패 조건
5. 관측 가능성
6. 다른 결정에 미치는 영향
7. 추천안과 남는 불확실성

---

## 5. 결정 순서

1. DQ-01~DQ-07, DQ-10의 현재 정책 확정 — **완료**
2. 임시 Problem Statement와 RQ 작성 — **다음 단계**
3. 관련 연구 evidence matrix 작성
4. 대표 공격·hard negative vertical slice
5. MQ-01~MQ-10 후보안 검증
6. DQ-08 protocol-specific 지식 허용 수준 재결정
7. DQ-09 핵심 contribution 확정
8. 최종 Problem Statement, RQ, Scope, Threat Model 확정

결정이 바뀌면 조용히 덮어쓰지 않고 `research/DECISIONS.md`에 변경 이유와 영향을 기록한다.

---

## 6. 기록된 1차 답변

아래 선택은 `DECISIONS.md` D-001~D-005에 기록되어 있다.

```text
DQ-01 1차 산출물: C
DQ-02 분석 시점: A
DQ-03 대상 현상: A
DQ-04 시간 범위: C
DQ-05 증거 경계: A+B 선택적, C는 평가만
```
