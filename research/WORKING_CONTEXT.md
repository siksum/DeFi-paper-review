# PMA 연구 작업 맥락

최종 갱신일: 2026-08-12

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
15. 관측된 왜곡의 `조건부 최소 순간자금`을 계산할 때 고정할 공격경로·평가함수·시간 범위·다중자산 가치평가 기준
16. protocol adapter 유무에 따른 semantic coverage·UNKNOWN을 어떤 평가 단위로 분리할지

과거 답변에 위 항목의 후보안이 있어도 자동으로 채택하지 않는다.

DeFiRanger에서 수동 external information 부족과 관련된 semantic FN 295건, 정상 fee·buyback 메커니즘에 의한 26 FP가 확인되었다. 이는 14번과 16번의 직접 검토 근거지만, 한 논문만으로 분야 전체 gap이나 D-008의 해답을 확정하지 않는다.

DeFort는 `역사 가격 이상 + 관련 주소 이익`으로 경보를 만들고 attacker·victim·profiteer·fund flow·associated function까지 출력한다. 따라서 역할·분석 출력이 전혀 없다는 넓은 gap 주장은 약화된다. 다만 역할은 anomaly와 token flow 휴리스틱이며, complex price model과 trace return 누락이 실제 FN으로 이어졌고, full NAV·control clustering·return-value→settlement dependency·counterfactual loss는 검증하지 않았다. 이 근거도 D-008이나 최종 gap을 자동 확정하지 않는다.

DeFiScope는 Transfer Graph로 6개 DeFi operation을 복원하고, verified source의 가격 관련 코드와 token balance/supply delta를 LLM에 제공해 정확한 가격이 아니라 증가·감소 방향을 추론한 뒤 8개 pattern으로 PMA 후보를 탐지한다. D1에서 76/95를 탐지했고, 1,000-transaction semantic benchmark에서 TG recall 0.912를 보고했으며, D2에서는 147/153 precision, D3 96,800건에서는 false alarm 0을 보고했다. 반면 source·compile·ERC-20·single-transaction·수치추론 한계가 실제 FN으로 나타났고, 방법 출력에는 attacker/victim 귀속과 actual/counterfactual P&L·victim loss가 없다. 이는 D-005·007·010의 경계를 강화하지만 기존 ACCEPTED 결정을 변경하지 않는다.

Sereum은 PMA 직접 탐지기가 아니라 재진입 runtime defense이지만, 수정 EVM에서 `SLOAD` 값이 stack·memory를 거쳐 `JUMPI` 조건에 사용되고 같은 contract의 이전 invocation이 동일 storage address에 `SSTORE`하는 경로를 실제 실행으로 연결한다. 이는 호출 존재·시간적 인접성보다 강한 consumption 후보 edge와 event-less observation의 인접 근거다. 반면 packed field·trust·manual mutex 의미를 잃어 오탐이 생기며 가격 왜곡·경제 정산·역할·손익은 다루지 않는다. 따라서 GAP-003·007의 넓은 부재 주장은 약화하지만, Sereum의 PMA 기능 부재 자체는 gap 지지 근거로 사용하지 않는다. 기존 ACCEPTED 결정과 충돌하지 않는다.

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

- `논문읽기_Research_OS_v3/핵심_방법론_논문_정리_템플릿_v3.md`: Track A 논문별 분석 템플릿
- `논문읽기_Research_OS_v3/개념_참고논문_정리_템플릿.md`: Track B 논문별 분석 템플릿
- `논문읽기_Research_OS_v3/연구_OS_상태보드_템플릿_v3.md`: 현재 상태와 열린 문제의 운영 정본
- `논문읽기_Research_OS_v3/리서치갭_통합_매트릭스_템플릿_v3.md`: gap과 cross-paper 비교의 운영 정본
- `논문읽기_Research_OS_v3/인용_아이디어_뱅크_템플릿.md`: 인용·수식·설계 아이디어의 운영 정본
- `research/README.md`: Research OS 정본 경로와 사용 순서
- `research/RESEARCH_GAP_LEDGER.md`: gap 후보와 이를 지원·약화·반박하는 논문 근거 장부
- `research/CITATION_IDEA_BANK.md`: 재사용 가능한 정의·수식·주장·반례의 출처 및 적용 경계 장부
- `research/DECISION_QUEUE.md`: 사용자 결정과 증거 기반 방법론 결정을 분리한 대기열
- `PMA-논문/README.md`: 논문 목록과 연구 진행 단계
- `PMA-논문/related-work-comparison.md`: 검토 완료 논문의 방법·입력·출력·역할·손익·평가·gap을 한데 모은 공통 통합 비교 정본
- `PMA-논문/DeFiRanger/관련 연구 분석.md`: cash-flow tree·semantic lifting·8개 PMA pattern, 평가 오류와 적용 한계의 근거 장부
- `PMA-논문/DeFort/관련 연구 분석.md`: 가격 이상·price read·profit 모델과 attacker/victim/profiteer·자금 흐름 분석의 근거 장부
- `PMA-논문/DeFiScope/관련 연구 분석.md`: Transfer Graph·LLM 가격 방향·8개 pattern과 source/single-tx/역할·손익 한계의 근거 장부
- `PMA-논문/Sereum/관련 연구 분석.md`: dynamic taint·call tree·write lock과 storage-control dependency, semantic-loss 오탐의 인접 근거 장부
- `PMA-논문/CLUE/CLUE.md`: execution property graph
- `PMA-논문/SMARTCAT/SMARTCAT.md`: token-flow 및 control/data-flow 기반 선제 탐지
- `PMA-논문/Identifying_Victims_in_DeFi_Attacks/Identifying_Victims_in_DeFi_Attacks.md`: victim 후보 식별
- `PMA-논문/sok_survey/Oracles in Decentralized Finance/내용 정리.md`: DEX oracle의 조작자원·담보·단순 이익 모델과 실제 사건 이전 한계

## 9. 문헌 검토 운영 방식

2026-08-12에 사용자가 제공한 Research OS v3 문서의 방향을 현재 저장소 규칙에 맞게 통합했다.

- 논문을 Core Method, Concept/Theory, Survey/SoK, Empirical/Incident, Standard/Official Evidence의 역할로 먼저 분류한다.
- 직접 방법론 논문과 개념·수식·실증 참고 논문에 서로 다른 템플릿을 사용한다.
- 논문별 노트에서 끝내지 않고 제공된 상태보드·리서치갭 통합 매트릭스·인용 아이디어 뱅크 파일을 직접 갱신한다.
- 새 논문은 기존 gap을 뒷받침하는 자료뿐 아니라 gap을 약화하거나 닫는 반증으로도 검사한다.
- 자동화는 추출·비교·반증·상태 동기화를 지원하지만 RQ, novelty, contribution과 `ACCEPTED` 결정은 자동 확정하지 않는다.
- 공유 대화의 상태명은 별도로 가져오지 않고 이 저장소의 결정 상태 체계를 유지한다.
- 이 저장소를 현재 연구 자료와 상태의 중심으로 사용한다. citation-network·AI 검색 도구는 발견과 초기 분석 보조이며, Zotero 도입은 현재 필수가 아니다.
