# 인용·아이디어 뱅크

> 여러 논문에서 발견한 유용한 정의, 개념, 수식, empirical finding, 설계 아이디어를 논문별 파일 밖에서도 검색할 수 있도록 통합 관리한다.

---

# 1. Concept Bank

| ID | Concept | Source | Locator | 한 줄 의미 | 내 연구 활용 | 상태 |
|---|---|---|---|---|---|---|
| C001 | DEX oracle의 경제적 실행 가능성 | `ORACLES-DEFI-2023` | Sec. 4-4.1, Algorithm 1, pp.6-7 | 목표 가격 왜곡에 필요한 AMM 자원과 lending 담보를 함께 계산한 뒤 현실적 조달 가능성을 별도로 판단한다. | 조건부 최소 순간 자금·실제 동원 자금·실제 비용을 분리하는 평가 구조 | VERIFIED |
| C002 | 공격 필요자본과 공격비용의 구분 | `ORACLES-DEFI-2023` | Algorithm 1; Fig. 3·5 | 공격 실행에 필요한 자원 규모와 실행 후 잃는 경제적 비용은 같은 값이 아니다. | 플래시론 원금을 실제 비용으로 오인하지 않는 출력 설계 | VERIFIED |
| C003 | Consumer parameter가 공격 결과에 미치는 영향 | `ORACLES-DEFI-2023` | Sec. 4.2, Eq. 16-18, Fig. 1 | LTV가 높을수록 더 작은 가격 왜곡으로 담보의 실제 가치 이상을 빌릴 수 있다. | 왜곡률과 함께 LTV·담보계수·차입한도를 보고하는 평가 설계 | VERIFIED |
| C004 | AMM 곡선·초기상태별 조작자원 차이 | `ORACLES-DEFI-2023` | Sec. 4.3-4.4, Fig. 2-5 | 같은 왜곡이라도 AMM 곡선과 공격 전 reserve 상태에 따라 필요한 자원이 달라진다. | Evaluation surface별 자본 민감도 평가 | VERIFIED |

---

# 2. Claim / Citation Bank

| ID | 원 논문의 Claim | Source | Locator | 정확한 Paraphrase | 사용 위치 | Caveat | 상태 |
|---|---|---|---|---|---|---|---|
| CIT001 | Protocol risk parameter, oracle window, manipulation duration, AMM curve와 liquidity로 attack cost와 minimum collateral을 계산하고 경제적 조달 가능성은 별도로 판단한다. | `ORACLES-DEFI-2023` | Sec. 4-4.1, Algorithm 1, pp.6-7 | DEX 가격을 사용하는 프로토콜의 조작 위험은 가격 변동폭뿐 아니라 시장의 유동성과 곡선, 가격을 평균내는 기간, 값을 사용하는 프로토콜의 위험 설정을 함께 고려해야 한다. | Threat Model / Method / Evaluation / Related Work | 실제 transaction detector나 자동 safety threshold가 아니다. | VERIFIED |
| CIT002 | 단순 lending model에서 담보 포기를 상쇄하기 위한 최소 가격 상승률은 `epsilon >= 1/LTV - 1`이다. | `ORACLES-DEFI-2023` | Sec. 4.2, Eq. 16-18, pp.8-9 | 대출 프로토콜이 담보가치 중 더 큰 비율을 빌려줄수록 공격자는 더 작은 가격 왜곡으로도 담보의 실제 가치 이상을 빌릴 수 있다. | Threat Model / Evaluation / Discussion | 거래 손실, fee, gas, borrow cap과 실제 가용 유동성을 제외한 1차 조건이다. | VERIFIED |
| CIT003 | 예제에서 StableSwap은 Constant Product보다 낮은 최소 공격자원을 보였고 초기 reserve imbalance도 조작 난이도에 영향을 줄 수 있다. | `ORACLES-DEFI-2023` | Sec. 4.3-4.4, Fig. 2-5, pp.9-12 | 동일한 가격 왜곡이라도 가격을 만드는 함수와 공격 직전 상태가 다르면 필요한 자본도 달라지므로, 평소 가격 변동성만으로 조작 저항성을 판단할 수 없다. | Related Work / Method / Evaluation / Discussion | LTV 0.4, TWAP 30분, 조작 1분, StableSwap `A=30` 등의 특정 simulation 가정에 한정된다. | VERIFIED |

---

# 3. Formula / Model Bank

| ID | Model | Source | Inputs | Assumptions | 내 연구 적용 | 판정 |
|---|---|---|---|---|---|---|
| M001 | Constant Product 목표가격의 필요 투입량 | `ORACLES-DEFI-2023`, App. A, Eq. A1-A5 | 공격 전 reserve `x,y`, 목표가격 `p_j` | Constant Product, fee·router·token tax·concentrated liquidity 생략 | Atomic Constant Product 사건의 조건부 최소 input을 실제 투입량과 비교 | ADAPTABLE |
| M002 | Arithmetic·geometric TWAP target price | `ORACLES-DEFI-2023`, Sec. 3, Eq. 9-15 | 정상가격, window, time weight, 조작 구간 `m`, 목표 TWAP | 관측구간과 정상가격을 알고 실제 arbitrage 반응을 단순화 | 향후 multi-block 분석 후보; atomic v1 핵심식으로는 사용하지 않음 | INSPIRATION_ONLY |
| M003 | Lending price break-even `epsilon >= 1/LTV - 1` | `ORACLES-DEFI-2023`, Sec. 4.2, Eq. 16-18 | 담보 `C`, LTV, 가격 상승률 | 담보 포기, 최대 차입, fee·cap·가용 유동성 생략 | Consumer parameter가 필요한 왜곡을 바꾼다는 parameter 분석 | ADAPTABLE |
| M004 | DEX oracle safety Algorithm 1 | `ORACLES-DEFI-2023`, Sec. 4.1, p.7 | Protocol risk parameter, window, `m`, AMM liquidity·curve | 공격경로·target과 모델을 알고 있음 | 공격경로 복원 후 경제적 실행 가능성을 정량화하는 후처리 구조 | ADAPTABLE |

판정:  
`DIRECTLY_USABLE / ADAPTABLE / INSPIRATION_ONLY / NOT_APPLICABLE`

---

# 4. Parameter Evidence Bank

| ID | Parameter | Value/Range | Source | Setting | 내 연구 적용 여부 |
|---|---|---|---|---|---|
| P001 | LTV | 예제 `0.4` | `ORACLES-DEFI-2023`, Sec. 4.3-4.4 | Constant Product·StableSwap simulation | 보편값이 아니라 consumer parameter 민감도 예시로만 사용 |
| P002 | TWAP window | 예제 `30분` | `ORACLES-DEFI-2023`, Sec. 4.3 | Simulation | v1 핵심 범위 아님; 향후 multi-block 평가 후보 |
| P003 | Arbitrage 없는 조작시간 | 예제 `1분` | `ORACLES-DEFI-2023`, Sec. 4.3 | Simulation | 실제 arbitrage latency의 실증 근거로 사용하지 않음 |
| P004 | StableSwap amplification `A` | 예제 `30` | `ORACLES-DEFI-2023`, Sec. 4.4 | StableSwap simulation | 실제 pool 구현과 상태를 확인한 뒤에만 적용 |
| P005 | Pool liquidity와 reserve imbalance | pool별 상이 | `ORACLES-DEFI-2023`, Alg. 1; Sec. 4.4 | AMM 상태 | 공격 직전 state에서 직접 관측할 계산 입력 후보 |

---

# 5. Empirical Evidence Bank

| ID | Finding | Source | Dataset/Setting | 뒷받침할 내 Claim | 일반화 한계 |
|---|---|---|---|---|---|
| E001 | 해당 없음 | `ORACLES-DEFI-2023` | 실제 incident dataset·transaction replay 없음 | 이 논문을 empirical evidence로 사용하지 않음 | Figure 1-5는 empirical finding이 아니라 simulation finding임 |

---

# 6. Design Inspiration Bank

| ID | Idea | Source | 원래 용도 | 내가 바꾸어 쓸 가능성 | 필요한 검증 |
|---|---|---|---|---|---|
| D001 | 목표 왜곡 → 필요자본 → economic feasibility | `ORACLES-DEFI-2023`, Algorithm 1 | DEX-based TWAP oracle의 사전 위험평가 | 복원된 실제 PMA의 조건부 최소 순간 자금과 실제 동원자금 비교 | 자산별 시간순 balance, 실제 fee·execution price, 경로 고정 범위 |
| D002 | 실제 순이익과 조작 귀속 추가이익 분리 | `ORACLES-DEFI-2023`에서 영감을 얻은 확장 | 원 논문은 simplified profit만 계산 | 실제 P&L과 no-manipulation counterfactual 차이를 별도 출력 | Reference value, counterfactual replay, address control |
| D003 | Consumer risk parameter를 포함한 사건 결과표 | `ORACLES-DEFI-2023`, Eq. 16-18, Fig. 1 | LTV에 따른 공격 가능 영역 분석 | LTV·담보계수·차입한도·가용 유동성을 왜곡률과 함께 보고 | Protocol별 decision logic과 parameter source |
| D004 | Evaluation surface별 자본 민감도 분석 | `ORACLES-DEFI-2023`, Fig. 2-5 | Constant Product·StableSwap 비교 | AMM·vault·lending별 초기상태와 평가함수에 따른 최소자본 변화 평가 | Family별 모델 검증과 실제 incident·hard negative 대조 |

---

# 7. Counterargument Bank

| ID | 반론/반례 | Source | 공격하는 내 Claim | 영향 | 상태 |
|---|---|---|---|---|---|
| X001 | 수익이 양수인 정상 차익거래·청산도 존재하므로 profitability만으로 PMA를 판정할 수 없다. | `ORACLES-DEFI-2023`의 적용 경계 + 현재 방법론 원칙 | 수익성은 PMA의 충분조건이다. | MAJOR | VERIFIED |
| X002 | Constant Product 예제의 총 필요자원 약 9.3L은 특정 parameter 조합의 결과다. | `ORACLES-DEFI-2023`, Sec. 4.3 | 모든 PMA에는 pool liquidity 대비 공통 최소자본 비율이 존재한다. | MAJOR | VERIFIED |
| X003 | StableSwap 결과는 동일 왜곡률이라도 평가함수와 초기상태에 따라 조작자원이 달라짐을 보여준다. | `ORACLES-DEFI-2023`, Sec. 4.4 | 최소자본을 모든 PMA에 하나의 식으로 계산할 수 있다. | MAJOR | VERIFIED |
| X004 | 이론적 attack cost는 실제 공격자의 최소 보유자본, flash-loan principal 또는 realized loss와 동일하지 않다. | `ORACLES-DEFI-2023`, Algorithm 1·App. A-B의 이전 검토 | 논문의 attack cost를 실제 사건의 비용으로 직접 사용할 수 있다. | MAJOR | VERIFIED |

---

# 8. 실제 논문 초안 사용 추적

| Citation ID | 내 논문 Section | 사용 목적 | 실제 사용 여부 | 원문 검증 |
|---|---|---|---:|---:|
| CIT001 | 미정 | DEX oracle 경제성 평가의 선행연구 | 아니오 | 완료 |
| CIT002 | 미정 | LTV와 공격 목표의 관계 | 아니오 | 완료 |
| CIT003 | 미정 | AMM 구조·초기상태별 조작저항성 | 아니오 | 완료 |

---

# 9. 상태 정의

- `CANDIDATE`: 유용해 보이지만 원문 의미/위치 최종 확인 전
- `VERIFIED`: 원문 위치, 의미, scope 확인 완료
- `ADOPTED`: 방법론/설계에 실제 반영
- `USED`: 논문 초안에 실제 인용
- `REJECTED`: 부적합 또는 과장 위험

> 이 상태는 출처·차용 상태다. 연구 결정은 `research/DECISIONS.md`의 `PROPOSED / ACCEPTED / REJECTED / SUPERSEDED / UNRESOLVED`를 따른다.

---

# 10. 사용 규칙

1. 중요한 아이디어는 반드시 원문 locator를 기록한다.
2. `PAPER` 내용과 내 해석을 분리한다.
3. 내 방법론에 사용할 때 원 논문의 가정이 유지되는지 확인한다.
4. 특정 모델의 결과를 더 넓은 protocol family에 자동 일반화하지 않는다.
5. survey에서 찾은 핵심 claim은 가능하면 primary source까지 추적한다.
6. 실제 초안에 사용할 때 `VERIFIED` 상태인지 확인한다.
