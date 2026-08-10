# PMA 연구 결정 로그

이 파일에는 사용자가 명시적으로 확정한 연구 결정과 그 변경 이력만 기록한다.

## 상태 정의

- `PROPOSED`: 검토 전
- `ACCEPTED`: 현재 채택
- `REJECTED`: 기각
- `SUPERSEDED`: 후속 결정으로 대체
- `UNRESOLVED`: 미결

## 현재 결정

### D-001 — 논문의 1차 산출물

- 상태: `ACCEPTED`
- 버전: `v1.0`
- 날짜: `2026-08-10`
- 질문: 연구의 최우선 산출물을 무엇으로 둘 것인가?
- 채택 내용: causal evidence representation을 핵심 표현으로 구성하고, PMA 탐지·역할 귀속·손익 계산을 그 표현에서 파생하는 통합 방법론을 목표로 한다.
- 근거: 개입, 왜곡값, 소비, 정산, 수혜·손실을 하나의 검증 가능한 경로로 연결해야 attacker·victim·profit 정의가 서로 모순되지 않는다.
- 반례 및 한계: causal evidence representation의 구체 schema와 이것이 실제 novelty인지 여부는 아직 확정하지 않았다.
- 영향 받는 항목: RQ, observation model, graph schema, 출력 형식, evaluation
- 사용자 확인: `DQ-01: C`

### D-002 — 분석 시점

- 상태: `ACCEPTED`
- 버전: `v1.0`
- 날짜: `2026-08-10`
- 질문: 사후 분석, 실시간 탐지, 실행 전 차단 중 무엇을 우선할 것인가?
- 채택 내용: 우선 사후 forensic analysis를 연구 범위로 한다.
- 근거: 인과 복원과 손익 계산의 타당성을 latency 제약과 분리해 먼저 검증한다.
- 반례 및 한계: 현재 결정은 실시간 가능성을 주장하거나 부정하지 않는다.
- 영향 받는 항목: 사용 가능한 증거, 성능 요구, 평가 metric, 논문 주장 범위
- 사용자 확인: `DQ-02: A`

### D-003 — 대상 현상

- 상태: `ACCEPTED`
- 버전: `v1.0`
- 날짜: `2026-08-10`
- 질문: 실행된 공격, 잠재 취약성, 광범위한 economic exploit 중 무엇을 대상으로 할 것인가?
- 채택 내용: 실제 실행되어 경제적 결과가 발생한 DeFi price manipulation attack을 우선 대상으로 한다.
- 근거: 실행 trace·state와 실제 수혜·손실을 ground truth 및 사례 분석과 대조할 수 있다.
- 반례 및 한계: 실행되지 않은 price-manipulation vulnerability 탐지와 PMA가 아닌 광범위한 economic exploit은 현재 핵심 범위가 아니다.
- 영향 받는 항목: problem statement, threat model, positive dataset, output semantics
- 사용자 확인: `DQ-03: A`

### D-004 — 시간 범위와 단계적 검증

- 상태: `ACCEPTED`
- 버전: `v1.0`
- 날짜: `2026-08-10`
- 질문: atomic과 multi-transaction/multi-block PMA를 어디까지 지원할 것인가?
- 채택 내용: 공통 표현은 multi-transaction/multi-block 확장을 막지 않도록 설계하되, v1의 구현·실증 검증은 atomic transaction으로 제한한다.
- 근거: 장기 범위를 정의에서 제거하지 않으면서 초기 관측·ground truth·반사실 계산을 통제한다.
- 반례 및 한계: v1 결과로 multi-transaction/multi-block 탐지를 지원한다고 주장하지 않는다. 확장 가능성은 별도로 검증해야 한다.
- 영향 받는 항목: graph 시간 모델, snapshot 범위, dataset, evaluation claim
- 사용자 확인: `DQ-04: C`

### D-005 — 방법 입력과 평가 증거의 경계

- 상태: `ACCEPTED`
- 버전: `v1.0`
- 날짜: `2026-08-10`
- 질문: 방법 입력과 평가에 어떤 증거를 허용할 것인가?
- 채택 내용:
  - 공개 온체인 실행·상태 증거를 필수 기반으로 사용한다.
  - verified source와 ABI는 선택적 enrichment로 허용하며, 없을 때의 성능과 실패를 분리해 보고한다.
  - 사고 보고서, 알려진 attacker·victim·attack-type label은 방법 입력이 아니라 ground truth 구축과 평가에만 사용한다.
- 근거: 재현성과 일반성을 유지하면서 semantic recovery의 실제 한계를 측정한다.
- 반례 및 한계: 온체인 증거만으로 복원할 수 없는 항목은 `UNKNOWN`이 될 수 있다.
- 영향 받는 항목: observation model, semantic lifting, 사양 의존성 주장, dataset leakage 통제
- 사용자 확인: `DQ-05: A+B 선택적, 사고 label은 평가만`

### D-006 — 대상 EVM 생태계

- 상태: `ACCEPTED`
- 버전: `v1.0`
- 날짜: `2026-08-10`
- 질문: 어떤 chain·VM을 연구 대상으로 할 것인가?
- 채택 내용: EVM을 대상으로 하며 Ethereum, BSC, Optimism, Avalanche의 EVM 범위, Arbitrum, Base를 대상 chain 후보군으로 포함한다.
- 근거: 사용자가 요구한 다중 EVM 생태계 범위다.
- 반례 및 한계: 모든 chain에서 동일한 데이터 가용성·trace semantics·archive 접근성을 보장하지 않는다. 실제 evaluation에 포함할 chain 수와 사건 수는 데이터 감사를 거쳐 확정한다.
- 영향 받는 항목: 데이터 수집, chain normalization, 외적 타당성, evaluation strata
- 사용자 확인: `DQ-06`에서 대상 chain을 명시함

### D-007 — 역할·손익 출력과 UNKNOWN

- 상태: `ACCEPTED`
- 버전: `v1.0`
- 날짜: `2026-08-10`
- 질문: 모든 사건에 역할과 손익을 강제로 귀속할 것인가?
- 채택 내용: attacker, beneficiary, consumer, victim/loss bearer, gain/loss 등 정의된 각 항목을 출력하는 것을 목표로 한다. 근거가 부족하면 값을 강제하지 않고 `UNKNOWN`과 함께 사용한 증거 및 신뢰 근거를 출력한다.
- 근거: 거짓 귀속을 완전한 출력으로 오인하지 않으면서 관측 한계를 명시할 수 있다.
- 반례 및 한계: confidence의 표현과 calibration 방법은 아직 미정이다.
- 영향 받는 항목: output schema, evaluation metric, annotation policy, 사용자 해석
- 사용자 확인: `DQ-07`에서 명시함

### D-008 — Protocol-specific 지식 허용 수준

- 상태: `UNRESOLVED`
- 버전: `v0.1`
- 날짜: `2026-08-10`
- 질문: protocol별 decoder, 함수 목록, 주소 whitelist를 core method에 어느 수준까지 허용할 것인가?
- 후보안: core 판정에는 수동 protocol rule을 요구하지 않고 선택적 adapter의 추가 효과를 분리 평가한다.
- 근거: 관련 연구의 의존성과 실제 semantic recovery 가능성을 먼저 비교해야 한다.
- 영향 받는 항목: 일반성 주장, 복원률, 구현 비용, baseline 비교
- 사용자 확인: `DQ-08: 잘 모르겠어`; 증거 조사 후 다시 결정

### D-009 — 핵심 Contribution 우선순위

- 상태: `UNRESOLVED`
- 버전: `v0.1`
- 날짜: `2026-08-10`
- 질문: 표현·방법론, 탐지 알고리즘, dataset 중 무엇을 주 contribution으로 둘 것인가?
- 후보안: causal evidence representation과 그 파생 방법론
- 근거: 관련 연구 분석 전에는 novelty와 차별성을 확정할 수 없다.
- 영향 받는 항목: RQ, paper positioning, artifact, evaluation
- 사용자 확인: 관련 연구 분석 후 확정하기로 함

### D-010 — 사후 분석 단계의 오류 보고 정책

- 상태: `ACCEPTED`
- 버전: `v1.0`
- 날짜: `2026-08-10`
- 질문: false positive와 false negative 중 무엇을 우선할 것인가?
- 채택 내용: 현재 사후 분석 연구 단계에서는 어느 한 오류를 우선한다고 확정하지 않고 false positive, false negative, `UNKNOWN`을 분리 보고한다.
- 근거: 실제 사용 시점과 대응 방식이 정해져야 오류 비용을 비교할 수 있다.
- 반례 및 한계: 운영 목적이 정해진 뒤 오류 비용과 threshold 정책을 다시 결정해야 한다.
- 영향 받는 항목: evaluation metric, threshold, 결과 표, 운영 주장
- 사용자 확인: `DQ-10`에서 명시함

## 기록 형식

### D-000 — 결정 제목

- 상태: `PROPOSED`
- 버전: `v0.1`
- 날짜: `YYYY-MM-DD`
- 질문:
- 후보안:
- 채택 내용:
- 근거:
- 반례 및 한계:
- 관측 가능성:
- 영향 받는 항목:
- 대체한 결정:
- 사용자 확인:

## 변경 규칙

기존 `ACCEPTED` 결정을 바꾸는 경우 원래 항목을 지우지 않는다. 기존 항목은 `SUPERSEDED`로 남기고 새 결정 ID에서 변경 이유와 영향을 연결한다.
