## Overview

- 논문 제목: Advancing DeFi Analytics: Efficiency Analysis with Decentralized Exchanges Comparison Service
- 학회/저널: arXiv
- 연도: 2024
- 논문 리뷰일: 2025-12-16

<br>

## Problem Statement

### A. Challenges in Comparing Exchange efficiency in DeFi

`1. Dynamic Liquidity States`

- DeFi의 거래소를 비교하는 데 있어 근본적인 어려움 중 하나는 유동성 상태의 끊임없는 변동성임
- DeFi에서는 한 블록에서 다음 블록으로 넘어갈 때 유동성이 dynamic하게 변할 수 있음
    - 왜냐하면 transaction의 atomic한 상태와 플랫폼의 탈중앙성 때문인데, `누구나 언제든지` 풀에 유동성을 `추가`하거나 `제거`할 수 있기 때문임
- `arbitrage`를 할 수 있는 기회가 증가함에 따라 서로 다른 플랫폼 간의 가격 불일치는 유동성 풀의 빈번한 재조정을 발생시킴
- `yield farming`에서도 유동성 공급자들이 수익을 극대화하기 위해 자산을 다른 프로토콜 간에 이동시키면서 풀 구성의 변화를 야기시킬 수 있음
- `flash loan`은 arbitrage, 담보 교환, 자체 청산을 위해 사용될 수 있는데, 여러 프로토콜에 걸쳐 유동성의 상당하고 즉각적이 변화를 발생시킴. 한 풀에서 유동성을 일시적으로 고갈시키고 다른 풀을 넘치게 만들어 거래소 효율성 비교 작업을 복잡하게 만듦

```
이러한 동적인 유동성 상태는 일관된 비교를 어렵게 함. 불과 몇 초 간격으로 실행된 동일한 거래도 유동성 변화로 다른 환율이 될 수 있기 때문. 유동성이 높은 토큰은 드물지만, 유동성이 낮은 토큰에서 자주 발생함.
```

<br>

`2. Exclusion of Private Market Makers(PMMs)`

- 완전한 PMM 데이터 없이는 유동성의 실제 depth와 사용 가능한 trading opportunities를 평가할 수 없으나, 모든 PMMs를 경로 계산 및 비교에 포함 시킬 수 없음
- PMM의 오프체인 소스로 인해서 온체인 시장 비교에 추가적인 복잡성을 야기함
- 2024년 기준, PMMs가 DEX aggregator 프로토콜 거래량의 1/10 미만을 차지한다고 함
- PMMs의 약 40%는 여전히 시뮬레이션된 거래 경로에 존재함

<br>

`3. Network Congestion and Gas Price Volatility`

- 블록체인 네트워크는 때때로 높은 네트워크 피크가 있어서 가스비가 급등함 - 거래 비용 효율성에 영향을 비침
- 거래가 장기간 보류 상태로 남아 슬리피지가 증가하고 잠재적으로 덜 유리한 거래 결과로 이어질 수 있음

<br>

### B. Intent-related comparision challenges

- `장점`: gasless transactions, 일부 시스템의 MEV 보호 내장 메커니즘
<br>

**[Intent-based solutions]**

|Solution|Description|
|---|---|
|`1inch Fusion`|사용자가 지정한 가격 범위 내에서 최적의 가격을 찾기 위해 `Dutch auction` 방식 사용.온체인 경매를 통해 주문을 가장 효율적으로 실행할 수 있는 `실행자(Resolver)`를 결정. 이 과정은 경쟁을 유도하여 사용자에게 더 유리한 환율을 제공. 가스비가 정산 가격에 포함되어 있어 사용자가 별도의 가스비를 지불할 필요가 없는 구조(`Gasless`)를 제공하며, `1inch Limit Order Protocol`을 기반으로 하여 가스 효율성이 매우 높음. |
|`UniswapX`|사용자가 서명한 주문(Intent)을 `제3자 실행자(Filler)`들이 경쟁하여 채우는 방식. 1inch Fusion과 마찬가지로 실행자가 가스비를 부담하는 구조를 통해 사용자 편의성을 높임. 소액 거래에서는 1inch Fusion과 유사한 성능을 보이지만, 고액 거래(Whale 트랜잭션)나 상위 토큰 쌍 거래에서는 1inch Fusion이 더 효율적인 비율(Rate)을 보이는 경향이 있음.|
|`CoW`|1inch Fusion이나 UniswapX와 달리 상대적으로 `중앙화`된 접근 방식 사용. 주문 매칭과 실행자 선정이 `오프체인 Solver`에 의해 관리됨. `'Coincidence of Wants(욕구의 일치)'`를 찾아 P2P로 거래를 매칭하거나, 여러 주문을 묶어(`Batch`) 처리함으로써 최적의 가격을 찾음. `고정된 수수료` 구조를 가지고 운영됨. |

<br>

**[기존 스왑 프토토콜(Imperative Approach) vs Intent-based system(Declarative Approach)]**

|Approach|Description|
|---|---|
|`Imperative(명령형 접근)`|사용자가 "지금 당장, A토큰을 B토큰으로 이 경로를 통해 바꿔라"라고 구체적인 명령을 내리는 것과 같음. 사용자가 직접 트랜잭션을 제출하는 순간 즉시 실행됨. 그러나 찰나의 순간 더 좋은 가격 생겨도 유연하게 대처하기 어려움.|
|`Declarative(선언형 접근)`|사용자가 "나는 A토큰을 B토큰으로 바꾸고 싶은데, 최소한 이만큼은 받아야 해(의도, Intent)"라고 목표만 선언함. 구체적인 경로나 방법은 전문가(Resolver/Solver)에게 맡김. 목표만 정해두었기 때문에, 가장 좋은 가격을 찾기 위해 주문 체결을 약간 지연(Delay)시킬 수 있음. 이 짧은 지연 시간 동안 전문 중개자들이 경쟁하거나(경매 방식 등), 더 좋은 유동성을 찾아내어 결과적으로 사용자가 받는 금액(effective amount)을 최대화할 수 있음. |


<br>

---

## Decentralized Exchanges Comparison Service Overview




---
## 몰랐던 개념 정리

``` text
"However, despite their innova-tive nature, these early AMMs faced significant challenges,including liquidity fragmentation and capital inefficiency,stemming from the uniform distribution of liquidity acrossall price ranges, irrespective of market demand."
```

초기 AMM의 `x*y=k` 공식으로 인해서 0~무한대의 범위에 걸친 범위를 갖게 됨. 그래서 이후에 특정 가격 범위를 구간화하는 CPMM이 나온 것!

<br>

## 몰랐던 용어 정의

| Term | Description |
|---|---|
|`price impact`|사용자가 거래를 실행할 때 그 거래의 규모로 인해서 시장 가격이 얼마나 변동하는지를 나타내는 말|
|`limit order protocol`|현재 시장 가격이 아닌 자신이 원하는 지정가에 토큰을 거래할 수 있도록 지원하는 시스템(오더북). 1inch가 처음에는 단순히 최적의 경로를 찾아주는 aggregation router에 집중했지만, 이후 사용자가 선호하는 거래 가격을 지정할 수 있도록 limit order protocol 도입함|
|`Dutch auction mechanism`|네덜란드식 경매 메커니즘. 1inch Fusion과 같은 defi 프로토콜에서 최적의 교환 비율을 찾기 위해 사용하는 가격 결정 방식. 일반적인 경매와는 반대로 높은 가격에서 시작해서 점차 가격이 내려가는 방식|
|`Hot liquidity`|언제든지 거래에 사용될 수 있는 "항상 대기 중인" 유동성. 주로 AMM 내에 존재하며 사용자가 swap을 요청하면 즉시 접근하여 거래를 체결할 수 있음|
|`Cold liquidity`|평소에는 직접적으로 보이지 않지만, 특정 조건(주로 가격 변동이나 대규모 주문)이 발생할 때 활성화 되는 "잠재적인" 유동성. 개인 지갑, PMM, Arbitrageurs, CEX 등에 분산되어 있음. 특히 대규모 주문으로 슬리피지가 발생할 때 시장에 참여함. 슬리피지를 줄이는 데 기여함.|
|`PMM(Private Market Makers)`|일반적인 DEX의 유동성 풀(AMM)과는 달리, 전문적인 기관이나 대규모 자금을 가진 주체들이 독자적인 가격 결정 알고리즘과 자금(유동성)을 활용해 거래를 성사시키는 참여자. PMM은 오프체인(블록체인 밖)이나 비공개 방식으로 주문을 처리한 뒤 결과만 블록체인에 기록하는 경우가 많음. 거래가 확정되기 전까지는 이들이 어떤 가격을 제시할지 블록체인 상에서 미리 파악하기 어려움.|