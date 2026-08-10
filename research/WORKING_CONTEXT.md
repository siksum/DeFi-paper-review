# PMA 연구 작업 맥락

최종 갱신일: 2026-08-10

## 1. 이 문서의 지위

이 문서는 현재 작업공간의 맥락과 열린 문제를 보존한다. 연구 정의의 확정본은 아니다.

- 명시적으로 확정된 결정은 `DECISIONS.md`를 따른다.
- 첨부된 과거 GPT 답변은 문제 진단과 아이디어의 출처이지만 현재 방법론의 권위 있는 정의가 아니다.
- 다른 PMA/AVE 작업공간의 Codex 메모리는 이 저장소의 현재 상태와 혼합하지 않는다.

## 2. 사용자의 목표

Codex와 함께 DeFi price manipulation attack 연구 방법론을 논리적이고 재현 가능한 형태로 설계한다.

단순한 공격 탐지 pipeline이 아니라 다음 요소가 하나의 일관된 정보 그래프에서 연결되어야 한다.

- 행위자 및 통제 관계
- 공격자와 수혜자 귀속
- 소비자와 피해자 귀속
- semantic action의 범위와 복원 근거
- 개입 지점과 왜곡된 경제값
- 왜곡값의 소비 지점
- 정산 또는 의사결정
- 자산·부채 변화
- 실제 P&L, 반사실 추가이익, 피해자 손실
- 정상 차익거래·청산 등과의 구별

## 3. 반복되었던 실패

- 개념을 하나씩 독립적으로 정하면서 뒤 단계의 반례가 앞 단계의 정의를 계속 흔들었다.
- `tx.from = attacker`, `손실 주소 = victim` 같은 느슨한 후보 규칙이 정의처럼 취급되었다.
- semantic action 목록을 목적과 포함 기준보다 먼저 나열했다.
- 왜곡을 만든 상태, 파생된 왜곡값, 그 값을 소비한 지점을 섞었다.
- transaction balance delta와 manipulation-attributable gain을 섞었다.
- 사용자의 지적을 검증하기보다 동의하고 새 설명으로 갈아탔다.
- collect/lift/graph/detect 같은 구현 pipeline이 proof obligation보다 먼저 나왔다.

## 4. 현재 합의된 작업 원칙

아래는 방법론의 구체적 정의가 아니라 협업 과정에 관한 원칙이다.

- 기존 주장을 조용히 바꾸지 않는다.
- 변경에는 반례, 증거, 전후 차이, 하위 영향이 따라야 한다.
- 공격자와 피해자는 초기 라벨이 아니라 인과 증거로부터 파생되는 역할이어야 한다.
- 정보 그래프는 적어도 `개입 → 상태 변화 → 왜곡값 → 소비 → 정산 → 가치 결과`를 설명해야 한다.
- 서로 연결된 설계를 전체 구조 안에서 검토하되, 사용자에게는 작은 결정 단위로 설명한다.
- 실제 사건의 end-to-end vertical slice와 정상 반례로 설계를 깨뜨린다.
- 구현 전에 필요한 사실과 각 사실의 증거 출처·fallback을 정한다.
- 불명확한 항목은 확정하지 않고 `UNRESOLVED`로 남긴다.

## 5. 아직 열린 연구 결정

아래 항목은 현재 작업공간에서 아직 확정되지 않았다.

1. 확정된 연구 방향을 반영한 최종 Research Question과 세부 주장 범위
2. PMA와 상위 개념 AVE를 사용할지 및 그 관계
3. atomic v1 이후 multi-transaction/multi-block 확장의 관측·구현·평가 조건
4. 개입 지점, distortion surface, 평가값의 형식 정의
5. 왜곡값 reference 또는 counterfactual 구성 방식
6. 소비 지점과 settlement의 증거 기준
7. 주소·컨트랙트 control clustering과 attacker attribution
8. consumer, direct victim, loss bearer의 구분
9. semantic action 최소 ontology와 확장 규칙
10. 실제 순자산 변화와 manipulation-attributable gain 계산
11. victim loss, bad debt, 2차 손실의 구분
12. 정보 그래프의 최소 node·edge schema
13. 최종 detection predicate
14. 공격 사례와 hard-negative 평가 세트

과거 답변에 위 항목의 후보안이 있어도 자동으로 채택하지 않는다.

## 6. 확정된 연구 방향 요약

세부 내용과 변경 이력은 `DECISIONS.md`를 따른다.

- causal evidence representation을 핵심으로 하고 탐지·귀속·손익 계산을 그 위에서 파생한다.
- 우선 사후 forensic analysis와 실제 발생해 경제 결과가 있는 PMA를 대상으로 한다.
- 공통 표현은 multi-transaction/multi-block 확장을 막지 않되 v1 실증은 atomic transaction으로 제한한다.
- 공개 온체인 실행·상태 증거를 필수 기반으로 하며 source·ABI는 선택적으로 사용한다.
- 사고 보고서와 알려진 역할·공격 label은 ground truth와 평가에만 사용한다.
- EVM을 대상으로 Ethereum, BSC, Optimism, Avalanche의 EVM 범위, Arbitrum, Base를 대상 후보군에 포함한다.
- 역할·손익 항목은 출력을 목표로 하지만 근거가 부족하면 `UNKNOWN`과 신뢰 근거를 출력한다.
- protocol-specific 지식 허용 수준과 최종 contribution 우선순위는 아직 미결이다.
- 현재 사후 분석 단계에서는 false positive, false negative, `UNKNOWN`을 분리 보고한다.

## 7. 제안된 다음 산출물

상태: `PROPOSED`

`Methodology Kernel v0.1`을 만든다. 최소 포함 내용은 다음과 같다.

1. 대상 공격의 필요조건과 비대상
2. 최소 causal witness
3. graph node·edge 후보와 각 생성 증거
4. attacker·victim attribution 후보 규칙
5. semantic action 승격 기준
6. distortion 및 consumption 판정 후보 규칙
7. P&L·counterfactual gain·loss 계산 후보
8. 최종 판정식 후보
9. 서로 다른 공격 family와 정상 반례에 대한 수동 적용
10. 실패하거나 관측할 수 없는 부분

이 산출물의 정확한 범위와 사례 목록은 아직 `ACCEPTED`가 아니다.

## 8. 현재 저장소의 관련 자료

- `research/templates/RELATED_WORK_ANALYSIS_TEMPLATE.md`: 논문별 근거 중심 분석 템플릿
- `research/DECISION_QUEUE.md`: 사용자 결정과 증거 기반 방법론 결정을 분리한 대기열
- `PMA-논문/README.md`: 논문 목록과 연구 진행 단계
- `PMA-논문/related-work-comparison.md`: G1~G7 비교축과 현재 gap 초안
- `PMA-논문/DeFiRanger/내용 정리.md`: cash-flow tree와 semantic lifting 정리
- `PMA-논문/DeFort/DeFort.md`: 가격 이상·price read·profit 모델
- `PMA-논문/CLUE/CLUE.md`: execution property graph
- `PMA-논문/SMARTCAT/SMARTCAT.md`: token-flow 및 control/data-flow 기반 선제 탐지
- `PMA-논문/Identifying_Victims_in_DeFi_Attacks/Identifying_Victims_in_DeFi_Attacks.md`: victim 후보 식별
