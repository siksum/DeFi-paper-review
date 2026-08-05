# Cost of Manipulation in AMM-Based Oracles

Sebastian Müller<sup>1,2</sup>, Nordine Moumeni<sup>1</sup>, and Adel Messaoudi<sup>1</sup>

<sup>1</sup> Aix Marseille Univ, CNRS, I2M, Marseille, France <sup>2</sup> IOTA Foundation, Germany

Abstract. We study the robustness of AMM-based on-chain price oracles to strategic manipulation. An attacker trades against constant product automated market makers (CPMMs) to distort an on-chain oracle, arbitrageurs restore cross-pool and cross-venue consistency, and an oracle designer chooses how to aggregate pool quotes.

Taking an eficient-market-hypothesis (EMH) view of the of-chain “true” price, we define the cost of manipulation as the minimal mark-to-market loss that an attacker must incur to move the oracle by a given multiplicative factor. For independent CPMMs, we derive closed-form single-pool manipulation formulas and solve the attacker–designer game for weighted means and weighted medians, showing that liquidity weights maximize the minimum cost of manipulation within these classes for weighted medians (for any distortion level) and, for weighted means, locally as the distortion tends to zero. For larger distortions, weighted means become more fragile: optimal weights can depend on the target distortion and no single choice is uniformly optimal across distortion levels. In a frictionless CPMM model with cross-pool arbitrage, the manipulation cost depends only on the total quote depth and coincides across symmetric aggregators. We extend this framework to multi-asset star architectures, confirming that liquidity weights remain optimal in the same sense. Finally, we bridge theory and practice by incorporating dwell times and rate limits, providing a quantitative yardstick to size oracles against the explicit economic costs of attack.

Keywords: Automated market makers · Price oracles · Decentralized finance · Market manipulation · Liquidity-weighted aggregation · Robust statistics

## 1 Introduction

Automated market makers (AMMs) have become core infrastructure in decentralized finance (DeFi), serving as venues for liquidity provision and as a source of on-chain price information. Lending protocols, derivatives platforms, and stablecoins routinely rely on on-chain oracles to trigger liquidations, margin calls, and redemptions, often for large notional exposures. In current practice, most production oracles still aggregate of-chain data from centralized exchanges, APIs, or market makers, but a smaller and conceptually important class of “pure on-chain” oracles derives prices solely from AMM activity. These designs improve decentralization and verifiability by avoiding external data feeds, yet their security and robustness against manipulation remain only partially understood.

We develop a model of the cost of manipulation for such AMM-based oracles. An attacker strategically trades against constant-product market makers (CP-MMs) in order to push the oracle away from a latent eficient price, an oracle designer chooses how to aggregate pool quotes, and arbitrageurs trade across pools and external venues to restore price consistency. Adopting an eficientmarket-hypothesis (EMH) view of the of-chain price, we measure manipulation cost as the mark-to-market economic loss that an attacker must realize in order to move the oracle by a prescribed multiplicative factor $r \geq 1$ , upward or downward. Our goal is to provide simple, closed-form benchmarks for the robustness of pure on-chain CPMM-based oracles that can serve as a foundation for more elaborate oracle designs.

At a high level, our results show that in CPMMs the cost of manipulating an oracle admits simple expressions that depend only on liquidity depth and aggregation weights, and that within natural mean- and median-type aggregators, liquidity-proportional weights are max–min optimal (exactly for weighted medians and to second order in a small-distortion expansion for weighted means). The rest of the paper makes this statement precise in single-pool, multi-pool, and multiasset settings and quantifies the resulting robustness benchmarks for AMM-based oracles.

Mean vs. median across distortion levels. For small distortions $r = 1 + \varepsilon$ , liquidityweighted means are more expensive to manipulate than liquidity-weighted medians at leading order, reflecting that means “use” all pools while medians require only a majority-weight cover. For larger distortions, however, weighted means can become substantially more fragile because per-pool CPMM manipulation cost grows sublinearly once a pool is pushed beyond the inflection threshold of the single-pool factor. In this regime, designer-optimal mean weights can depend on the target distortion and there is generally no distortion-uniform max–min choice.

Contributions and Organization. Formally, the paper makes five contributions:

– We recall closed-form single-pool formulas for trade size and economic loss as functions of the relative distortion factor r under a CPMM, and extend them to include proportional swap fees via a simple rescaling of inputs (Section 2.1, Appendix ${ \mathrm { B } } ,$ and Section 5).

– We characterize the optimal attacker strategies for multi-pool oracles based on weighted means and weighted medians and solve the corresponding designer’s problem. Liquidity weights are max–min optimal for weighted medians for any distortion level, and are locally max–min optimal for weighted means as $r  1 ;$ we also provide an explicit counterexample showing that weighted means admit no distortion-uniform max–min choice beyond small distortions (Section 3, Section 3.2, Theorems 1 and 2, and Appendices C–D).

– We analyze frictionless cross-pool arbitrage and swap fees, noting that under perfect equalization any symmetric aggregator yields the same oracle price and that manipulation cost depends only on total depth, while fees induce no-arbitrage bands that jointly afect arbitrage eficiency and attack cost (Sections 2.2, 3, and 5).

– We sketch a multi-asset extension for star architectures around a numéraire and show that liquidity weights remain optimal asset-by-asset (Section 4 and Appendix E).

– We discuss practical extensions that embed our static cost benchmarks into dynamic and implementation-aware settings, introducing a dwell-time metric $\mathrm { C o s t } _ { A } ( \boldsymbol { r } , \tau )$ and outlining how platform-specific rate limits and gas costs interact with manipulation incentives in practice (Section 7).

Relation to prior work. Our analysis lies at the intersection of CFMM microstructure, oracle manipulation in $\mathrm { D e F i }$ , and robust aggregation. On the CFMM side, Uniswap $\mathrm { v 2 / v 3 }$ and the CFMM literature $\left[ 2 { , } 3 { , } 6 \right]$ characterize invariant-based pricing, slippage, and arbitrage but do not formulate a general cost of manipulation metric or a max–min designer problem over cross-pool weights. Oracle-security work [26,21,24,8,1,13] studies concrete mechanisms such as TWAPs and documents manipulation events and capital requirements, while treating AMM pricing largely as a black box. Robust-aggregation results [16,5,22] motivate volumeand liquidity-weighted means and medians statistically but abstract away from bonding-curve mechanics and strategic attackers. Our contribution is to connect these strands by deriving closed-form CPMM manipulation costs, solving the associated attacker–designer games for weighted means and medians (in singleasset and star-architecture settings), and showing that liquidity-proportional weights are max–min optimal for weighted medians (for any distortion level) and locally max–min optimal for weighted means near $r = 1$ , while weighted means admit no distortion-uniform max–min weights in general. We refer to Section 6 for a more detailed discussion of related literature.

## 2 Background and Model

## 2.1 Primer on CPMMs and Notation

This subsection collects standard definitions and single-pool formulas for constantproduct market makers and fixes the notation used throughout the paper. In this paper we focus exclusively on Uniswap v2–style constant-product AMMs, which we refer to as constant-product market makers $( C P M M s ) { \mathrm { : } }$ pools whose reserves $( x , y )$ satisfy the invariant $x y = k \ [ 2 , 6 ]$

Constant-product invariant and marginal price. Consider a trading pair $( X , Y )$ with reserves $( x , y )$ in a CPMM pool. The invariant is

$$
k = x y, \quad k > 0,
$$

![](images/53efd0b89ec5c392b43ad0d29a9c677367f34a280ac01071aa29dc5057f9e3a5.jpg)  
Fig. 1. The CPMM bonding curve $x y = k$ and a $Y  X$ trade that increases the marginal price $p = y / x$

so any trade moves $( x , y )$ along the curve $x y = k$ . The marginal price of X in units of Y is the rate at which the reserves trade locally:

marginal price of X in $Y = - { \frac { d y } { d x } } = { \frac { y } { x } } .$

Thus the reserve ratio

$$
p := \frac {y}{x} \quad (Y \text {per} X)
$$

is exactly the marginal price on the CPMM curve. This is why on-chain “spot prices” for CPMMs are typically read as $p = y / x$

Liquidity depth and the role of k. For a fixed external (eficient) price $p ^ { \star }$ , the invariant k encodes the pool’s liquidity depth. At equilibrium $p = p ^ { \star }$ we have

$$
x _ {0} = \sqrt {\frac {k}{p ^ {\star}}}, \qquad y _ {0} = \sqrt {k p ^ {\star}},
$$

so both reserves scale as $\sqrt { k }$ . Moving the on-chain price from $p _ { 0 }$ to a new level $p _ { 1 }$ requires a finite trade that shifts reserves along the same curve. We parametrize distortions multiplicatively: for a factor $r \geq 1$ , an “upward” move targets $p _ { 1 } = r p _ { 0 }$ and a “downward” move targets $p _ { 1 } = p _ { 0 } / r$ . This yields a symmetric notion of manipulation, since the relevant cost factor satisfies $f ( r ) = \sqrt { r } { + } 1 / \sqrt { r } { - } 2 = f ( 1 / r )$ as shown below.

Baseline single-pool formulas (as functions of a multiplicative distortion $r \geq 1 )$ Consider a CPMM initially at price $p _ { 0 }$ with reserves $( x _ { 0 } , y _ { 0 } )$ and invariant $k = x _ { 0 } y _ { 0 }$

– Upward move (factor r). To move the on-chain price to $p _ { 1 } = r p _ { 0 }$ with a $Y  X$ trade, the attacker must supply

$$
y _ {\uparrow} ^ {\mathrm{in}} (r) = y _ {0} \big (\sqrt {r} - 1 \big).\tag{1}
$$

– Downward move (factor $1 / r )$ . To move the price to $p _ { 1 } = p _ { 0 } / r$ with an $X  Y$ trade, the attacker must supply

$$
x _ {\downarrow} ^ {\mathrm{in}} (r) = x _ {0} \big (\sqrt {r} - 1 \big).\tag{2}
$$

– Economic cost (mark-to-market at $p ^ { \star } = p _ { 0 } )$ . When the reference price equals the pre-trade pool price, the economic loss in units of $Y$ is

$$
C _ {Y} ^ {\uparrow} (r) = y _ {0} \bigg (\sqrt {r} + \frac {1}{\sqrt {r}} - 2 \bigg),\tag{3}
$$

$$
C _ {Y} ^ {\downarrow} (r) = y _ {0} \left(\sqrt {r} + \frac {1}{\sqrt {r}} - 2\right).\tag{4}
$$

The trade sizes $( 1 ) - ( 2 )$ reflect the $\sqrt { k } .$ -scaling of liquidity depth, and the cost formulas (3)–(4) encode the realized slippage loss relative to the eficient price. Detailed derivations are recalled in Appendix B. These CPMM price-impact and slippage expressions coincide with the formulas used in protocol documentation and CFMM analyses; see, for example, the Uniswap v2 and v3 whitepapers [2,3] and the CFMM framework of Angeris and Chitra [6].

Notation and reference price. Throughout, we use:

$( X , Y )$ : base and quote assets.

$\mathbf { \Phi } - \mathbf { \Phi } ( x , y ) \colon$ current pool reserves; $k = x y$ the CPMM invariant.

$p = y / x \colon$ on-chain marginal price $( Y$ per unit X).

$- \ p ^ { \star } \colon$ external “fair” or eficient price, unafected by on-chain trades (e.g., a robust cross-venue midprice).

When we consider several pools for the same pair, we write $( x _ { i } , y _ { i } ) , k _ { i }$ , and $p _ { i } = y _ { i } / x _ { i }$ for pool i, and collect pool prices into a vector $p = ( p _ { i } ) _ { i }$ . An oracle or price aggregator applies a deterministic functional A (e.g., liquidity-weighted mean, weighted median, trimmed mean) to p to obtain the reported oracle price ${ \hat { p } } = A ( \pmb { p } )$

Agents and manipulation cost. The strategic agents in our model are:

– an oracle designer, who chooses the aggregation rule A and fee/parameter settings;

– an attacker, who submits trades to CPMM pools with the goal of pushing $\hat { p }$ away from $p ^ { \star }$ ;

– arbitrageurs, who trade across pools (and vs. external venues) whenever price discrepancies exceed their frictions.

We measure manipulation cost in units of the quote asset Y as the mark-tomarket loss of the attacker’s trades when valued at $p ^ { \star }$ (slippage loss relative to the eficient price). This quantity underlies the “adversarial cost” or “manipulation cost” studied in the rest of the paper.

## 2.2 Robustness Metric: Cost of Manipulation

We formalize the notion of manipulation cost used throughout the paper. Let $p ^ { \star }$ denote the eficient price of a given asset in units of a reference asset, and let A be an aggregation rule mapping observed pool quotes (and possibly other on-chain data) to an oracle output pˆ. We consider a one-shot setting first and then indicate how to extend to a multi-step dwell requirement.

Definition 1 (Cost of manipulation). Fix an aggregation rule A, an eficient price $p ^ { \star }$ , a distortion factor $r \geq 1$ , and a reference asset $( e . g .$ , the quote asset Y or a numéraire). Let U denote the set of admissible attack strategies (collections of on-chain trades on the relevant pools), and write $C ( u )$ for the mark-to-market economic loss of an attack $u \in \mathcal { U }$ measured in the reference asset at the eficient prices. The cost of manipulation at level r is

$$
\mathrm{Cost} _ {A} (r) := \inf \left\{C (u): u \in \mathcal {U}, \max \left\{\frac {\hat {p} (u)}{p ^ {\star}}, \frac {p ^ {\star}}{\hat {p} (u)} \right\} \geq r \right\},
$$

where ${ \hat { p } } ( u ) : = A$ (quotes after u) is the oracle output after the attack has been executed and before any corrective arbitrage.

In words, Cos ${ \mathrm { ; } } _ { A } ( r )$ is the minimal economic loss an attacker must incur to move the oracle by a factor of at least r up or down relative to $p ^ { \star }$ . In the single-CPMM setting with A equal to the pool price, $\mathrm { C o s t } _ { A } ( r )$ reduces to the one-pool formulas $C _ { Y } ^ { \uparrow , \downarrow } ( r )$ in (3)–(4). In the multi-pool mean and median settings, $\operatorname { C o s t } _ { A } ( r )$ is given by the solutions of the corresponding multi-pool optimization problems. In the rest of the paper we focus on this “one-step” notion under static eficient prices; a dynamic extension with dwell-time constraints is discussed in the outlook section.

## 3 Manipulation of Single-Pair Oracles

Having fixed notation and the robustness metric, we now analyze how much economic loss an attacker must incur to manipulate oracles built from one or several CPMM pools that all trade the same asset pair. We start with two pools, then consider the general case of N independent pools aggregated by means or medians, and finally incorporate cross-pool arbitrage.

We will repeatedly use the single-pool cost factor

$$
f (r) := \sqrt {r} + \frac {1}{\sqrt {r}} - 2,
$$

so that, when $p ^ { \star } = p _ { 0 }$ , moving a CPMM price from $p _ { 0 }$ to $r p _ { 0 }$ (or to $p _ { 0 } / r )$ costs $C _ { Y } ( r ) = y _ { 0 } f ( r )$ in quote units; see (1)–(4).

## 3.1 Two CPMM Pools

As a warm-up and to illustrate the main ideas for the general N-pool setting, we first analyze the case of two independent CPMM pools aggregated by a deterministic oracle A. We index pools by a subscript and indicate time by a superscript in parentheses. Initial (pre-attack) reserves and prices are $( x _ { i } ^ { ( 0 ) } , y _ { i } ^ { ( 0 ) } )$ 1 with invariant $k _ { i } = x _ { i } ^ { ( 0 ) } y _ { i } ^ { ( 0 ) }$ and price $p _ { i } ^ { ( 0 ) } = y _ { i } ^ { ( 0 ) } / x _ { i } ^ { ( 0 ) }$ . An attacker produces post-trade quotes $p _ { i } ^ { ( 1 ) }$ . The oracle aggregates the updated $\{ p _ { i } ^ { ( 1 ) } \}$ with one of the following methods.

![](images/27a194eb1726b10267282513ce9ef82808f8c9cee1a43b8ebef9547258866e9d.jpg)  
Fig. 2. Marginal manipulation cost $f ^ { \prime } ( t ) = ( t - 1 ) / ( 2 t ^ { 3 / 2 } )$ for a single CPMM pool. Marginal cost peaks at $t = 3$ and declines thereafter, which makes concentrated mean attacks attractive once some pools are pushed beyond $3 \times$ their initial price.

Weighted Mean Let weights $w _ { 1 } , w _ { 2 } > 0$ with $w _ { 1 } + w _ { 2 } = 1$ (e.g., by liquidity depth $w _ { i } \ \propto \sqrt { k _ { i } }$ , equivalently $w _ { i } \propto y _ { i } ^ { ( 0 ) }$ when pre-trade prices match). The aggregated post-trade price is

$$
\hat {p} ^ {(1)} = w _ {1} p _ {1} ^ {(1)} + w _ {2} p _ {2} ^ {(1)}.
$$

If the attacker manipulates only pool 1, the aggregator sensitivity is $\partial \hat { p } ^ { ( 1 ) } / \partial p _ { 1 } ^ { ( 1 ) } =$ $w _ { 1 }$ . To move $\hat { p } ^ { ( 1 ) }$ by $\varDelta .$ , one needs $\varDelta p _ { 1 } ^ { ( 1 ) } = \varDelta / w _ { 1 }$ . Plugging $p _ { 1 } ^ { ( 1 ) } \mapsto p _ { 1 } ^ { ( 1 ) } + \varDelta / w _ { 1 }$ into (1) or (2) gives the required trade amounts per direction. We now turn to the question of how an attacker can influence the aggregated price at lowest cost. We start with the optimal attacker strategy for any given weights $w _ { 1 }$ and $w _ { 2 }$

Optimal Manipulation. Assume both pools start at the same fair price $p _ { 0 } = p ^ { \star }$ 2 so $p _ { 1 } ^ { ( 0 ) } = p _ { 2 } ^ { ( 0 ) } = p _ { 0 }$ . Fix a target distortion factor $r \geq 1$ . Write $p _ { i } ^ { ( 1 ) } = s _ { i } ^ { 2 } p _ { 0 }$ with $s _ { i } > 0$ . Achieving $\hat { p } ^ { ( 1 ) } = r p _ { 0 }$ at minimum cost reduces to the program

$$
\min _ {s _ {1}, s _ {2} > 0} y _ {1} ^ {(0)} f (s _ {1} ^ {2}) + y _ {2} ^ {(0)} f (s _ {2} ^ {2}) \quad \text {s.t.} \quad w _ {1} s _ {1} ^ {2} + w _ {2} s _ {2} ^ {2} = r,
$$

where $f ( \cdot )$ is the single-pool factor from (3). A downward distortion $\hat { p } ^ { ( 1 ) } = p _ { 0 } / r$ corresponds to replacing r by $1 / r ;$ since $f ( r ) = f ( 1 / r )$ , the minimal cost is the same in both directions. At an interior optimum, the Lagrange first-order conditions take the form

$$
y _ {i} ^ {(0)} f ^ {\prime} (t _ {i}) = \lambda w _ {i}, \qquad t _ {i} := s _ {i} ^ {2},
$$

for some multiplier $\lambda > 0$ . Thus the marginal cost $f ^ { \prime } ( t _ { i } )$ is proportional to the weight-to-liquidity ratio ${ w _ { i } / y _ { i } ^ { ( 0 ) } }$ . In particular, whenever the relevant multipliers lie in the convex range $t _ { i } \in ( 0 , 3 ]$ (so $f ^ { \prime }$ is increasing, see Figure 2), pools with larger $w _ { i } / y _ { i } ^ { ( 0 ) }$ are pushed to larger $t _ { i }$ , i.e., manipulated more aggressively.

Weighted Median Let $p _ { ( 1 ) } \leq p _ { ( 2 ) }$ be the sorted quotes and corresponding weights $w _ { ( 1 ) } , w _ { ( 2 ) }$ normalized so that $w _ { ( 1 ) } + w _ { ( 2 ) } = 1$ . We use the lower weighted median, defined as the smallest price level whose cumulative weight is at least $1 / 2$ . Thus $\tilde { p } = p _ { ( 1 ) } \mathrm { ~ i f ~ } w _ { ( 1 ) } \geq 1 / 2$ and $\tilde { p } = p _ { ( 2 ) }$ otherwise; under the exact tie $w _ { ( 1 ) } = w _ { ( 2 ) } = 1 / 2$ , this convention returns $\tilde { p } = p _ { ( 1 ) }$

Optimal Manipulation. With two pools, the weighted median is piecewise constant: the aggregator output is the price of whichever pool carries the majority weight. To move $\tilde { p } ,$ the attacker must either (i) manipulate the dominating pool, or (ii) push the manipulated pool past the other so that the majority-weight index flips. In either subcase, the required trade sizes and costs follow from the single-pool formulas with the appropriate target price level.

## 3.2 Multiple CPMM Pools

We now turn to the general setting of N independent pools. Throughout this subsection, let

$$
y _ {\mathrm{tot}} := \sum_ {i = 1} ^ {N} y _ {i} ^ {(0)}
$$

denote the total quote depth across pools.

Weighted Mean We first define the minimal cost of manipulation $F _ { N } ( \pmb { w } ; r )$ for a specific choice of aggregation weights w and target factor $r \geq 1$ . This is the economic loss an attacker must incur to achieve $\hat { p } ^ { ( 1 ) } = r p _ { 0 }$ when they optimize their trades across pools to minimize cost. With $s _ { i }$ representing the square root of the per-pool price multiplier $( \mathrm { i . e . , } p _ { i } ^ { ( 1 ) } = s _ { i } ^ { 2 } p _ { 0 } )$ ), define

$$
F _ {N} (\boldsymbol {w}; r) := \min _ {\{s _ {i} > 0 \}} \sum_ {i = 1} ^ {N} y _ {i} ^ {(0)} \big (s _ {i} + s _ {i} ^ {- 1} - 2 \big) \quad \text {s.t.} \quad \sum_ {i = 1} ^ {N} w _ {i} s _ {i} ^ {2} = r.
$$

The oracle designer’s problem is to choose w to maximize this adversarial cost. The following theorem gives a small-distortion (second-order) characterization and identifies liquidity weights as locally max–min optimal for weighted-mean oracles near $r = 1$

Theorem 1. Fix $r = 1 + \varepsilon$ with $\varepsilon \to 0 ^ { + }$ . Then, uniformly over weight vectors $\pmb { w } = ( w _ { i } ) _ { i = 1 } ^ { N }$ with $w _ { i } \geq 0$ and $\textstyle \sum _ { i } w _ { i } = 1$

$$
F _ {N} (\pmb {w}; 1 + \varepsilon) = \frac {\varepsilon^ {2}}{4} \frac {1}{\sum_ {i = 1} ^ {N} w _ {i} ^ {2} / y _ {i} ^ {(0)}} + o (\varepsilon^ {2}).
$$

Consequently,

$$
\sup _ {w _ {i} \geq 0, \sum_ {i} w _ {i} = 1} F _ {N} (\boldsymbol {w}; 1 + \varepsilon) = \frac {\varepsilon^ {2}}{4} y _ {\mathrm{tot}} + o (\varepsilon^ {2}),
$$

and the leading-order term is uniquely maximized by the liquidity weights $w _ { i } ^ { \star } =$ $y _ { i } ^ { ( 0 ) } / y _ { \mathrm { t o t } }$

Proof. Deferred to Appendix C.

Convex regime (exact benchmark under a per-pool cap). Fix an upward target $r \in [ 1 , 3 ]$ and consider attacks constrained by $t _ { i } : = p _ { i } ^ { ( 1 ) } / p _ { 0 } \in [ 1 , 3 ]$ . On this range $f$ is convex. Under liquidity weights $w _ { i } ^ { \star } = y _ { i } ^ { ( 0 ) } / y _ { \mathrm { t o t } }$ , Jensen’s inequality implies

$$
\sum_ {i = 1} ^ {N} y _ {i} ^ {(0)} f (t _ {i}) \geq y _ {\mathrm{tot}} f (r),
$$

with equality at $t _ { i } \equiv r .$ . Since $t _ { i } \equiv r$ is feasible for every weight vector, no designer can force manipulation cost above $y _ { \mathrm { t o t } } f ( r )$ , and thus liquidity weights are max–min optimal within this capped convex regime. In the unconstrained model, by contrast, the aggregate condition $r < 3$ does not preclude concentrated attacks with some $t _ { i } > 3$ when some weights are small.

Convexity threshold and concentrated attacks. The per-pool cost factor $f ( t ) =$ $\sqrt { t } + 1 / \sqrt { t } - 2$ has a crucial inflection point at $t = 3 .$ since

$$
f ^ {\prime \prime} (t) = \frac {3 - t}{4 t ^ {5 / 2}},
$$

so $f$ is convex on $( 0 , 3 ]$ and concave on $[ 3 , \infty )$ . Equivalently, the marginal cost $f ^ { \prime } ( t ) = ( t - 1 ) / ( 2 t ^ { 3 / 2 } )$ peaks at $t = 3$ and decreases thereafter; see Fig. 2.

This implies a qualitative shift in attacker incentives: attacks are dispersed when the relevant multipliers remain in $( 0 , 3 ]$ , but can become concentrated once some pool is pushed beyond $t = 3$ . In particular, if $r > 3$ then any feasible $( t _ { i } ) _ { i }$ must satisfy max $t _ { i } \geq r > 3 ,$ , and even if $r < 3$ a concentrated mean attack may still push some pools past $t = 3$ when some weights $w _ { i }$ are small. We leave a full max–min characterization of optimal weighted-mean weights outside the smalldistortion regime to future work; Appendix $\mathrm { C }$ gives a simple counterexample showing that liquidity weights need not be max–min optimal at large distortion levels and discusses alternative weight choices.

Weighted Median Let $N \geq 2$ pools start at a common price $p _ { 0 }$ . Write $y _ { i } ^ { ( 0 ) }$ for the quote reserve of pool i and let $w _ { i } > 0$ be the aggregation weights normalized so that $\textstyle \sum _ { i } w _ { i } = 1$ . Fix a target distortion factor $r \geq 1$ and define the per-pool cost factor

$$
f (r) := \sqrt {r} + \frac {1}{\sqrt {r}} - 2, \qquad C _ {i} (r) = y _ {i} ^ {(0)} f (r).
$$

The weighted median $\widetilde { p }$ at t is the smallest price level such that the cumulative weight at or below that level is at least $1 / 2$ . With all quotes initially $\mathrm { a t } \ p _ { 0 }$ , to enforce an upward distortion $\widetilde { p } \geq r p _ { 0 }$ it is necessary and suficient to move a subset of pools $S \subset \{ 1 , \ldots , N \}$ to $r p _ { 0 }$ so that their cumulative weight covers half the mass:

$$
\sum_ {i \in S} w _ {i} \geq \frac {1}{2} \quad \Longleftrightarrow \quad \widetilde {p} \geq r p _ {0}.\tag{5}
$$

Any pool not in $S$ can remain at $p _ { 0 }$ (moving it to an intermediate price in $( p _ { 0 } , r p _ { 0 } )$ does not change the median). Hence the attacker’s problem reduces to the one-constraint covering program

$$
C _ {\mathrm{med}} (r; \boldsymbol {w}) = \min _ {S \subset \{1, \dots , N \}} \sum_ {i \in S} y _ {i} ^ {(0)} f (r) \quad \text {s.t.} \quad \sum_ {i \in S} w _ {i} \geq \frac {1}{2}.\tag{6}
$$

There is no closed form in general because the feasible set depends on the discrete weight configuration $\{ w _ { i } \}$ . Nevertheless, the structure is simple and yields an explicit strategy:

Optimal strategy (set form). Because $C _ { \mathrm { m e d } } ( r ; w )$ is additive across selected pools, it sufices to choose a subset $S$ with total weight at least $1 / 2$ that minimizes $\textstyle \sum _ { i \in S } y _ { i } ^ { ( 0 ) }$ . This “minimum-cost cover” is easy to compute for the pool counts that arise in practice, and its structure is transparent: optimal attacks prioritize pools with small depth per unit of weight, i.e., small ratios $y _ { i } ^ { ( 0 ) } / w _ { i }$ . A simple greedy candidate is to sort pools by $y _ { i } ^ { ( 0 ) } / w _ { i }$ and add them in this order until the $1 / 2$ threshold is reached; this is exact in the two-pool and equal-weight cases below. In particular, there is never a reason to overshoot $r p _ { 0 }$ on any selected pool.

## Special cases and bounds.

1. Two pools: assume without loss of generality that $w _ { 1 } \geq w _ { 2 }$ . If $\begin{array} { r } { w _ { 1 } > \frac { 1 } { 2 } } \end{array}$ , then the weighted median equals the quote of pool 1, so an optimal attack sets $p _ { 1 } = r p _ { 0 }$ and leaves $p _ { 2 } = p _ { 0 }$ , yielding $C ^ { \mathrm { m e d } } = y _ { 1 } ^ { ( 0 ) } f ( r ) = y _ { \mathrm { m a j o r } } ^ { ( 0 ) } f ( r )$ . In the tie case $\begin{array} { r } { w _ { 1 } = w _ { 2 } = \frac { 1 } { 2 } } \end{array}$ , our lower-median convention returns the smaller quote, so to enforce $\tilde { p } \geq r p _ { 0 }$ one must move both pools to $r p _ { 0 } .$ giving $C ^ { \mathrm { m e d } } = y _ { \mathrm { t o t } } f ( r )$ If instead ties are resolved by interpolation $( \mathrm { e . g . } , \tilde { p } = ( p _ { ( 1 ) } + p _ { ( 2 ) } ) / 2 )$ , then in the tie case it also sufices to leave one pool at $p _ { 0 }$ and move the other to $( 2 r - 1 ) p _ { 0 } ;$ ; choosing the cheaper pool yields cost $y _ { \operatorname* { m i n } } ^ { ( 0 ) } f ( 2 r - 1 )$ with $y _ { \mathrm { m i n } } ^ { ( 0 ) } : = \operatorname* { m i n } ( y _ { 1 } ^ { ( 0 ) } , y _ { 2 } ^ { ( 0 ) } )$ . Hence, under midpoint interpolation, the tie-case cost is min $\{ y _ { \mathrm { t o t } } f ( r ) , y _ { \mathrm { m i n } } ^ { ( 0 ) } f ( 2 r - 1 ) \}$

2. Equal weights $( w _ { i } = 1 / N )$ : let $k = \lceil N / 2 \rceil$ and order depths as $y _ { ( 1 ) } ^ { ( 0 ) } \leq \cdots \leq$ $y _ { ( N ) } ^ { ( 0 ) }$ . An optimal attack moves the k pools with smallest depths to rp<sub>0</sub> (leaving the others at $p _ { 0 } )$ , giving the closed form

$$
C ^ {\mathrm{med}} (r) = f (r) \sum_ {j = 1} ^ {k} y _ {(j)} ^ {(0)}.
$$

3. Liquidity weights $( w _ { i } \propto y _ { i } ^ { ( 0 ) } )$ : covering $1 / 2$ of the total weight requires at least half the total quote depth, so

$$
\frac {1}{2} y _ {\mathrm{tot}} \leq \min _ {S: \sum w _ {i} \geq 1 / 2} \sum_ {i \in S} y _ {i} ^ {(0)} \leq \frac {1}{2} y _ {\mathrm{tot}} + y _ {\mathrm{max}} ^ {(0)},
$$

implying the median manipulation cost satisfies the exact bounds

$$
\frac {1}{2} f (r) y _ {\mathrm{tot}} \leq C _ {\mathrm{med}} (r; \boldsymbol {w} ^ {\star}) \leq f (r) \Big (\frac {1}{2} y _ {\mathrm{tot}} + y _ {\max} ^ {(0)} \Big).
$$

For comparison, for the weighted mean with the same liquidity weights, Theorem 1 yields the small-distortion expansion $F _ { N } ( \pmb { w } ^ { \star } ; 1 + \varepsilon ) = ( \varepsilon ^ { 2 } / 4 ) y _ { \mathrm { t o t } } +$ $o ( \varepsilon ^ { 2 } )$ . Table 1 summarizes the contrast between mean- and median-based aggregation across distortion regimes.

We now show that, within the class of weighted-median oracles, liquidity weighting maximizes the attacker’s minimal cost.

Theorem 2. Let ${ \pmb y } = ( y _ { 1 } ^ { ( 0 ) } , \dots , y _ { N } ^ { ( 0 ) } )$ . Define

$$
\Theta (\boldsymbol {y}) := \min \Bigl \{\sum_ {i \in S} y _ {i} ^ {(0)}: S \subset \{1, \dots , N \}, \sum_ {i \in S} y _ {i} ^ {(0)} \geq \frac {1}{2} y _ {\mathrm{tot}} \Bigr \}.
$$

Then for any $r \geq 1$

$$
\sup _ {\boldsymbol {w} \geq 0, \sum w _ {i} = 1} C _ {\mathrm{med}} (r; \boldsymbol {w}) = f (r) \Theta (\boldsymbol {y}),
$$

attained by the liquidity weights $w _ { i } ^ { \star } = y _ { i } ^ { ( 0 ) } / y _ { \mathrm { t o t } }$ . Moreover, $\begin{array} { r } { \frac 1 2 y _ { \mathrm { t o t } } \ \le \ \theta ( y ) \ \le \ } \end{array}$ $\frac { 1 } { 2 } y _ { \mathrm { t o t } } + y _ { \mathrm { m a x } } ^ { ( 0 ) }$ , and for $N = 2 , \theta ( \pmb { y } ) = \operatorname* { m a x } ( y _ { 1 } ^ { ( 0 ) } , y _ { 2 } ^ { ( 0 ) } )$ .

Proof. Deferred to Appendix D.

## 3.3 Arbitrage Across Pools

We now consider frictionless, immediate arbitrage between CPMM pools that all trade the same pair. Any configuration in which some pools are more distorted than others creates an immediate arbitrage opportunity: an arbitrageur can buy in the cheap pool and sell in the expensive one until prices equalize, earning additional profit at the attacker’s expense and partially undoing the oracle distortion. Hence, in any minimal-cost attack under perfect arbitrage, we may restrict attention to equalized terminal configurations in which all pools end at the same target price $p _ { \mathrm { t a r } } = r p _ { 0 } \ ( \mathrm { o r } \ p _ { \mathrm { t a r } } = p _ { 0 } / r )$

Assume the system starts at a common eficient price $p _ { 0 }$ , so $y _ { i } ^ { ( 0 ) } / x _ { i } ^ { ( 0 ) } = p _ { 0 }$ and $k _ { i } = x _ { i } ^ { ( 0 ) } y _ { i } ^ { ( 0 ) }$ . In the equalized state, each pool is shifted by the same price multiplier, so the total economic loss is the sum of the single-pool costs, yielding

$$
C ^ {\star} (r) = y _ {\mathrm{tot}} f (r).\tag{7}
$$

Thus, under perfect cross-pool arbitrage, the N CPMMs behave like a single efective pool whose quote reserve equals the total depth $y _ { \mathrm { t o t } } { \mathrm { : } }$ the cost (7) is exactly the single-pool expression with y<sub>0</sub> replaced by $y _ { \mathrm { t o t } }$ . Because equalization forces all post-arbitrage quotes to coincide, any symmetric aggregator (weighted mean, median, trimmed mean, etc.) returns $p _ { \mathrm { t a r } }$ , so the cost (7) is independent of the particular symmetric aggregation rule.

Table 1. Summary of manipulation costs and optimal weight designs. Here $t _ { i } : = p _ { i } ^ { ( 1 ) } / p _ { 0 }$ is the per-pool price multiplier, and $\Theta ( \pmb { y } )$ is the minimum depth of a majority-weight subset.

<table><tr><td>Scenario</td><td>Designer Takeaway (Max-Min)</td><td>Minimal Cost</td></tr><tr><td colspan="3">Independent Pools (No Arbitrage)</td></tr><tr><td>Weighted Mean</td><td>Local Optimality ( $r \rightarrow 1$ ): Liquidity weights  $w_{i} \propto y_{i}^{(0)}$  are optimal.Large Distortions: Fragile. Once some  $t_{i} > 3$ , marginal costs decrease and concentrated attacks can dominate; no uniform max-min weights.</td><td>Small distortions: $\propto y_{\text{tot}}$ ;large distortions:sublinear</td></tr><tr><td>Weighted Median</td><td>Global Optimality ( $\forall r \geq 1$ ):Liquidity weights  $w_{i} \propto y_{i}^{(0)}$  are optimal for any target distortion.Robust: requires manipulating a majority-weight subset.</td><td> $f(r) \Theta(\boldsymbol{y})$ </td></tr><tr><td colspan="3">Perfect Cross-Pool Arbitrage</td></tr><tr><td>Any Symmetric Aggregator</td><td>Irrelevance: Arbitrage equalizes terminal prices  $p_{i} \rightarrow p_{\text{tar}}$ . Aggregation rule does not affect cost.</td><td> $f(r) y_{\text{tot}}$ </td></tr></table>

## 4 Multi-Asset Extension

We briefly sketch a multi-asset extension. A particularly clean setting is a star architecture around a numéraire, where each non-numéraire asset is priced by aggregating CPMM pools that trade against the numéraire and manipulation cost is measured in numéraire units. Appendix E states and proves that, in such architectures, liquidity-proportional weights remain optimal asset-by-asset (exactly for weighted medians and locally for weighted means as $r _ { a } \to 1 )$ . Extending this analysis to general CPMM graphs with cross pairs is left for future work.

## 5 Incorporating Swap Fees

Incorporating proportional swap fees into our CPMM cost formulas is straightforward. With an input fee ϕ, only a fraction (1 − ϕ) of the gross input is credited to the pool, so achieving a fixed distortion factor r requires gross trade sizes larger by a factor $1 / ( 1 - \phi )$ than in (1)–(2). In our one-shot formulas, this acts as a simple multiplicative adjustment of the direct manipulation cost. In multi-pool settings, fees also widen classical no-arbitrage bands for cross-pool cycles (see [6]), so small cross-pool discrepancies can persist because arbitrage is unprofitable unless they exceed the fee wedge. This weakens corrective arbitrage and can reduce the gross volume required to sustain a discrepancy, even though each swap pays fees. Finally, implementation costs such as gas fees contribute an additive term per transaction. Since our goal is a clean depth-driven baseline, we focus on the zero-fee case and treat the interaction between fees, arbitrage eficiency, and gas costs as deployment-specific refinements.

## 6 Related Work

CFMM microstructure and AMM oracles. The Uniswap v2 and v3 whitepapers [2,3] and the CFMM framework of Angeris and Chitra [6] describe invariantbased pricing, depth, and arbitrage for production CPMMs. Subsequent work analyzes LP risk, fee design, and predictable losses [9,4,12], multi-token AMMs and closed-form arbitrage in N-asset pools [25], and routing and coupling efects across CFMM pools [7,23], as well as providing broader surveys of DeFi AMMs and CFMM mechanics [11]. These papers treat slippage and arbitrage primarily as descriptive properties or sources of LP risk; to the best of our knowledge, none defines a general, closed-form cost of manipulation metric of the form “minimal loss to move the price by a given factor” nor poses a max–min defender problem over cross-pool weights.

Oracle manipulation, TWAPs, and DeFi security. Empirical and systems work documents oracle deviations and attacks in DeFi [26,21,24], and surveys oracle architectures and TWAP designs [8,27,14]. Uniswap v3 TWAP studies [1,13] compute capital requirements for TWAP manipulation, while large-scale evaluations of Chainlink and cross-chain oracles [20,15] and analysis frameworks such as OVer [18,10] provide risk metrics and stress tests under adversarial inputs. These contributions quantify attack costs for specific oracle mechanisms (primarily arithmetic TWAPs) and propose mitigations such as time windows and circuit breakers, but most treat AMM pricing as a black box or work numerically with particular TWAP implementations; they do not derive closed-form manipulation costs as explicit functions of liquidity and distortion, nor do they analyze a general multi-pool attacker–designer game over aggregation weights.

Robust aggregation, liquidity weights, and positioning. Robust statistics ofers general tools for aggregation under outliers [16,17,19], and recent crypto-specific work [5] derives nonasymptotic error bounds for weighted means and medians applied to exchange price data. Industry indices such as the SIX Crypto Indices [22] implement volume- or liquidity-weighted medians to down-weight small venues. These works justify liquidity and volume weights from a statistical-error perspective under contamination models, but do not model CFMM microstructure or a strategic attacker who must trade against bonding curves, and, to the best of our knowledge, they contain no CPMM-aware max–min optimality result for liquidity weights. Our contribution is deliberately basic: we take the standard CPMM price curve, define a general, closed-form cost of manipulation metric for single and multiple CPMM pools, and then solve the associated attacker–designer problems for weighted means, weighted medians, and a multi-asset star architecture. To our knowledge, this is the first work to show that, in a CPMM microstructure-aware setting, liquidity weights are max–min optimal for weighted medians (for any distortion level) and locally max–min optimal for weighted means near $r = 1$ ， and to show by counterexample that weighted means admit no distortion-uniform max–min weights beyond small distortions.

## 7 Discussion and Future Work

Multi-pool aggregation and on-chain feasibility In the independent-pool regime, our analysis shows that liquidity-weighted medians maximize the minimal cost of manipulation for any distortion level, while liquidity-weighted means are locally optimal near $r = 1$ but can be substantially more fragile under large distortions; under perfect cross-pool arbitrage the efective depth entering the cost formulas is simply the total quote reserve across pools. Robustness is therefore primarily driven by how much CPMM depth backs the oracle and how that depth is distributed across pools, rather than by finer choices among symmetric aggregators.

Dwell-time robustness and protocol interaction The static cost of manipulation Cost $_ A ( r )$ studied in the main text can be embedded into a dynamic setting via the dwell-time extension $\mathrm { C o s t } _ { A } ( \boldsymbol { r } , \tau )$ (Appendix A). For any given application, one can compute or upper bound the maximal exploitable gain from a mispricing by a factor r maintained for a dwell $\tau ;$ denote this by $B ( r , \tau )$ (e.g., the largest profit from shifting a liquidation threshold or triggering a mispriced payof). The deterrence criterion

$$
\mathrm{Cost} _ {A} (r, \tau) \gg B (r, \tau)
$$

then provides a quantitative notion of “economic safety margin”. Our CPMM cost formulas identify the per-block building blocks entering $\mathrm { C o s t } _ { A } ( \boldsymbol { r } , \tau )$ ; deriving sharp multi-block lower bounds in concrete latency and congestion-control models, and matching them against protocol-specific $B ( r , \tau )$ , is an important avenue for future work.

Aggregation rules under diferent fault models Our aggregation results highlight an important diference between CPMM-based on-chain oracles and the exchangebased setting studied by Allouche et al. [5], where trimmed medians are optimal under contamination. For our economic cost of manipulation metric on on-chain CPMM data, liquidity-weighted means are more expensive to manipulate than liquidity-weighted medians for small distortions in the independent-pool model, reflecting that means “use” all pools while medians only require a majority-weight cover. However, because per-pool CPMM manipulation cost grows sublinearly once a pool is pushed beyond the inflection threshold of the single-pool factor, weighted means can become significantly more fragile under large distortions; in this regime, designer-optimal mean weights can depend on the target distortion and there is no distortion-uniform max–min choice in general. Weighted medians retain a large-distortion guarantee: moving the oracle requires manipulating a subset covering at least half of the aggregation weight, regardless of how large $r$ is. Smart-contract bugs, misconfigured pools, or governance attacks can still create persistently faulty on-chain venues. In such cases, median- or trimmedmean aggregation over pools may be more appropriate to discount structurally broken pools while still using liquidity weights within the remaining set. Liquidityweighted means remain natural when all CPMMs are correct and manipulation occurs only via trading.

Beyond CPMMs In practice, concentrated-liquidity AMMs (CLMMs) such as Uniswap v3 are widely used. A convenient reduced-form view is that they induce a state-dependent efective depth $y _ { \mathrm { e f f } } ( p )$ aggregating all active liquidity at price $p .$ The cost of moving the price from $p _ { 0 }$ to $p _ { 1 } = r p _ { 0 }$ (or $p _ { 1 } = p _ { 0 } / r )$ is then obtained by integrating the CPMM slippage formulas along the path in price with $y _ { 0 }$ replaced locally by $y _ { \mathrm { e f f } } ( p )$ , preserving the convexity and monotonicity properties that underpin our multi-pool optimization at the expense of simple closed forms in $^ { r } \cdot$ Extending our results to CLMMs requires modeling the efective depth profile $y _ { \mathrm { e f f } } ( p ) ;$ we leave a full treatment of CLMM microstructure and tick dynamics to future work.

## 8 Conclusion

We introduced a quantitative notion of cost of manipulation for AMM-based oracles and analyzed how it depends on liquidity depth, aggregation rules, and arbitrage connectivity. For a single CPMM pool, we recalled closed-form formulas for the trade size and economic loss required to move prices by a factor r (upward or downward). In independent multi-pool settings, we solved the attacker–designer game for weighted medians (all distortion levels) and for weighted means locally near $r = 1$ . We also showed by counterexample that weighted means admit no distortion-uniform optimal weights beyond small distortions, while under frictionless cross-pool arbitrage the cost collapses to simple total-depth expressions that are independent of the particular symmetric aggregator. For weighted means, we also highlighted the inflection of the single-pool cost factor at multiplier $t = 3 ,$ which induces a qualitative shift from dispersed to concentrated optimal attacks and motivates either median-type aggregation or distortion-aware weighting that down-weights shallow pools more aggressively in the large-distortion regime. We extended this analysis to a multi-asset star architecture and proved an analogous optimality result for per-asset weights. Finally, the dwell-time and rate-limit extensions illustrate how these static cost benchmarks can be combined with chainand application-specific models to design AMM-based oracles whose manipulation cost dominates the economic gains available from induced mispricings.

## References

1. A. Adams. Uniswap v3 TWAP oracles in proof of stake. SSRN Working Paper 4384409, 2022.

2. H. Adams et al. Uniswap v2 core. Whitepaper, 2020.

3. H. Adams et al. Uniswap v3 core. Whitepaper, 2021.

4. Algebra Finance Research Team. The impact of market conditions and fee algorithms on the design of a competitive AMM. Whitepaper, 2022.

5. M. Allouche, M. Echenim, E. Gobet, and A.-C. Maurice. Statistical error bounds for weighted mean and median, with application to robust aggregation of cryptocurrency data. Preprint, 2024.

6. G. Angeris and T. Chitra. Improved price oracles: Constant function market makers. In Proceedings of the 2nd ACM Conference on Advances in Financial Technologies, pages 80–91. ACM, 2020.

7. G. Angeris, T. Diamandis, M. Resnick, and T. Chitra. Optimal routing for constant function market makers. In Proceedings of the 23rd ACM Conference on Economics and Computation. ACM, 2022.

8. A. T. Aspembitova et al. Oracles in decentralized finance: Attack costs, profits and mitigation. Applied Sciences, 12(24), 2022.

9. P. Bergault et al. Automated market makers: Mean-variance analysis of LPs. arXiv preprint arXiv:2212.00336, 2022.

10. J. Boe et al. Safeguarding DeFi smart contracts against oracle deviations. Proceedings of the ACM on Programming Languages, 2024.

11. Á. Cartea, F. Drissi, and M. Monga. Decentralised finance and automated market making. Journal of Economic Dynamics and Control, 2025.

12. Á. Cartea, F. Drissi, and M. Monga. Decentralised finance and automated market making: Predictable loss and optimal liquidity provision. Journal of Economic Dynamics and Control, 2025. Forthcoming; see also arXiv:2309.08431.

13. Chaos Labs. Block manipulation: Market risk of uniswap v3 TWAP oracles. Research report, 2023.

14. X. Deng, S. M. Beillahi, H. Du, P. Tiwari, and A. Veneris. Analysis of defi oracles. Staf Discussion Paper 2024-10, Bank of Canada, 2024.

15. R. Gansäuer et al. Price oracle accuracy across blockchains. In Proceedings of CAAW 2025, 2025.

16. F. R. Hampel. Robust statistics: A brief introduction and overview. Allgemeines Statistisches Archiv, 85(1):1–18, 2001.

17. P. J. Huber. Robust Statistics. John Wiley & Sons, New York, 1981.

18. Q. Luu et al. OVer: A framework for safeguarding DeFi smart contracts against oracle deviations. In Proceedings of the Web Conference, 2024.

19. R. A. Maronna, R. D. Martin, and V. J. Yohai. Robust Statistics: Theory and Methods. John Wiley & Sons, Chichester, 2006.

20. M. Nadler. Blockchain price oracles: Accuracy and violation recovery. Finance Research Letters, 2025.

21. K. Qin, L. Zhou, B. Livshits, and A. Gervais. Attacking the DeFi ecosystem with flash loans for fun and profit. In Proceedings of the 3rd ACM Conference on Advances in Financial Technologies, 2021.

22. SIX Group. SIX Crypto Indices: Real-Time Median and Real-Time Volume-Weighted Median Methodology, 2020.

23. A. Sterrett and A. Adams. A microstructure analysis of coupling in CFMMs. arXiv preprint arXiv:2510.06095, 2025.

24. W. Yang et al. Flash loan attack is more than just price oracle manipulation. arXiv preprint arXiv:2105.XXX, 2021.

25. X. Yang et al. Closed-form solutions for generic n-token AMM arbitrage. arXiv preprint arXiv:2402.06731, 2024.

26. F. Zhang, E. Cecchetti, K. Croman, A. Juels, and E. Shi. A first look into DeFi oracles. In Proceedings of the IEEE Security and Privacy Workshops, 2020.

27. Y. Zhao, X. Kang, T. Li, C.-K. Chu, and H. Wang. Towards trustworthy defi oracles: Past, present and future. arXiv preprint arXiv:2201.02358, 2022.

## A Sustained Manipulation and Protocol Constraints

The one-step cost of manipulation considered above abstracts away from timing and protocol-level limits. In practice, block structure, rate limits on shared objects, and latency all constrain both attackers and arbitrageurs and motivate a dynamic extension of our metric and of the comparison with application-specific benefit functions $B ( r , \tau )$

Sustained distortions with dwell time. Let time be indexed by blocks $t = 0 , 1 , . . . ,$ and let $A _ { t }$ be the aggregation rule at time t, possibly incorporating time-windowed statistics (e.g., TWAPs). The attacker submits a sequence of trades $u _ { 0 } , \ldots , u _ { T - 1 }$ 2 incurring cumulative loss $C ( u _ { 0 : T - 1 } )$ . For a dwell parameter $\tau _ { \mathrm { { i } } }$ , a natural extension of our robustness metric is the minimal cost needed to keep the oracle distorted by a factor of at least r for τ consecutive blocks:

$$
D _ {t} := \max \Bigl \{\frac {\hat {p} _ {t}}{p _ {t} ^ {\star}}, \frac {p _ {t} ^ {\star}}{\hat {p} _ {t}} \Bigr \}.
$$

$$
\mathrm{Cost} _ {A} (r, \tau ; T) := \inf \left\{C (u _ {0: T - 1}): \exists t _ {0} \text {s.t.} D _ {t} \geq r \forall t \in [ t _ {0}, t _ {0} + \tau - 1 ] \right\},
$$

with $\hat { p } _ { t } ~ = ~ A _ { t }$ (quotes after $u _ { 0 } , \ldots , u _ { t } )$ and $p _ { t } ^ { \star }$ the eficient price process. The asymptotic cost $\mathrm { C o s t } _ { A } ( \boldsymbol { r } , \tau )$ is defined via a liminf as $T \to \infty$ . Our static results identify the per-block building blocks entering $\mathrm { C o s t } _ { A } ( \boldsymbol { r } , \tau )$ ; deriving sharp multiblock lower bounds in specific latency models, and matching them against protocolspecific upper bounds $B ( r , \tau )$ , is a natural direction for further work and for quantitative risk budgeting.

Rate limits and shared-object congestion (Sui example). On high-throughput platforms with parallel execution such as $\mathrm { S u i . }$ , contention on shared objects (e.g., a CPMM pool) becomes a bottleneck, so protocols rate-control how many transactions per checkpoint may touch the same shared object. This interacts with the dwell-time notion of $\mathrm { C o s t } _ { A } ( \boldsymbol { r } , \tau )$ by limiting how quickly arbitrageurs can respond to distortions and how many manipulative trades an attacker can sustain per block. A detailed analysis of such rate limits—combining our static cost formulas with models of block-level access constraints, spam behavior, and explicit $B ( r , \tau )$ for concrete lending and derivatives protocols—is left for future work and is particularly relevant for Sui-style shared-object architectures.

## B Derivations for Single-Pool Formulas

We briefly derive the baseline single-pool manipulation formulas used throughout the paper.

## B.1 Trade needed to hit a relative price target

Consider a CPMM with reserves $( x _ { 0 } , y _ { 0 } )$ , invariant $k = x _ { 0 } y _ { 0 }$ and initial price $p _ { 0 } = y _ { 0 } / x _ { 0 }$ . Fix a target factor $r \geq 1$ . For an upward move we target $p _ { 1 } = r p _ { 0 }$ 2 and for a downward move we target $p _ { 1 } = p _ { 0 } / r$ . Post-trade reserves $( x _ { 1 } , y _ { 1 } )$ must satisfy

$$
p _ {1} = \frac {y _ {1}}{x _ {1}}, \qquad x _ {1} y _ {1} = k.
$$

Solving these two equations yields

$$
x _ {1} = \sqrt {\frac {k}{p _ {1}}}, \qquad y _ {1} = \sqrt {k p _ {1}}.
$$

Using $y _ { 0 } = \sqrt { k p _ { 0 } }$ and $x _ { 0 } = \sqrt { k / p _ { 0 } }$ , write $s = { \sqrt { r } }$ . Then:

$$
(p _ {1} = r p _ {0}) \colon y _ {1} = y _ {0} s, \text {so} y _ {\uparrow} ^ {\mathrm{in}} (r) = y _ {1} - y _ {0} = y _ {0} (s - 1)
$$

– Downward move $( p _ { 1 } = p _ { 0 } / r ) \colon x _ { 1 } = x _ { 0 } s , \mathrm { s o } x _ { \downarrow } ^ { \mathrm { i n } } ( r ) = x _ { 1 } - x _ { 0 } = x _ { 0 } ( s - 1 )$ , which is (2).

These expressions make explicit that trade size scales like $\sqrt { k }$ and depends on the target only through r.

## B.2 Economic cost at $p ^ { \star } = p _ { 0 }$

We mark attacker losses to market at a reference price $p ^ { \star }$ representing the external eficient value. For the baseline case $p ^ { \star } = p _ { 0 }$ :

– Upward move. A $Y  X$ trade of size $y _ { \uparrow } ^ { \mathrm { i n } }$ sends reserves from $\left( x _ { 0 } , y _ { 0 } \right) \mathrm { t o } \left( x _ { 1 } , y _ { 1 } \right)$ with $x _ { 1 } y _ { 1 } = k$ . The attacker receives

$$
x ^ {\mathrm{out}} = x _ {0} - x _ {1} = x _ {0} - \frac {k}{y _ {0} + y _ {\uparrow} ^ {\mathrm{in}}}.
$$

Valued at $p ^ { \star } = p _ { 0 }$ , the loss in $Y$ units is

$$
C _ {Y} ^ {\uparrow} = y _ {\uparrow} ^ {\mathrm{in}} - p _ {0} x ^ {\mathrm{out}}.
$$

Substituting $y _ { \uparrow } ^ { \mathrm { i n } } = y _ { 0 } ( s - 1 ) , x ^ { \mathrm { o u t } } = x _ { 0 } ( 1 - 1 / s ) \ \mathrm { a n d } \ p _ { 0 } x _ { 0 } = y _ { 0 }$ yields

$$
C _ {Y} ^ {\uparrow} (r) = y _ {0} \big (s + \frac {1}{s} - 2 \big),
$$

which is (3).

– Downward move. An $X  Y$ trade of size $x _ { \downarrow } ^ { \mathrm { i n } }$ yields

$$
y ^ {\mathrm{out}} = y _ {0} - y _ {1} = y _ {0} - \frac {k}{x _ {0} + x _ {\downarrow} ^ {\mathrm{in}}}.
$$

The loss in Y units is

$$
C _ {Y} ^ {\downarrow} = p _ {0} x _ {\downarrow} ^ {\mathrm{in}} - y ^ {\mathrm{out}}.
$$

With $x _ { \downarrow } ^ { \mathrm { { i n } } } = x _ { 0 } ( s - 1 ) , y ^ { \mathrm { { o u t } } } = y _ { 0 } ( 1 - 1 / s )$ and $p _ { 0 } x _ { 0 } = y _ { 0 }$ , we obtain

$$
C _ {Y} ^ {\downarrow} (r) = y _ {0} \big (s + \frac {1}{s} - 2 \big),
$$

which is (4).

In both directions, $C _ { Y }$ coincides with the standard notion of slippage loss (difference between what the attacker pays and the fair value of what they receive) used in CFMM analyses such as [6].

## C Weighted-Mean Multi-Pool Optimization

In this appendix we provide a detailed proof of Theorem 1. We work with price multipliers $t _ { i } : = s _ { i } ^ { 2 }$ (so that $t _ { i } = p _ { i } ^ { ( 1 ) } / p _ { 0 } )$ and write the attacker’s problem as

$$
F _ {N} (\boldsymbol {w}; r) = \min \Big \{\sum_ {i = 1} ^ {N} y _ {i} ^ {(0)} f (t _ {i}): t _ {i} > 0, \sum_ {i = 1} ^ {N} w _ {i} t _ {i} = r \Big \}, \qquad f (t) = \sqrt {t} + \frac {1}{\sqrt {t}} - 2.
$$

Why we focus on small distortions. The key technical feature is that the per-pool factor $f$ is not globally convex: a direct calculation gives

$$
f ^ {\prime \prime} (t) = \frac {3 - t}{4 t ^ {5 / 2}},
$$

so f is convex on $( 0 , 3 ]$ and concave on $[ 3 , \infty )$ . Equivalently, the marginal cost $f ^ { \prime } ( \dot { t } ) = ( t - 1 ) / ( 2 \dot { t } ^ { 3 / 2 } )$ increases on (0, 3] and decreases on $\lbrack 3 , \infty )$ ; see $\mathrm { F i g . ~ 2 }$ When some pools are pushed beyond the inflection point $t = 3 .$ , the attacker can benefit from concentrating distortion on a small-weight pool while leaving most pools close to $t = 1$ ; this may occur even for moderate aggregate targets r if some w<sub>i</sub> are small. For a concrete illustration, take $N = 2$ pools with quote depths $( y _ { 1 } ^ { ( 0 ) } , y _ { 2 } ^ { ( 0 ) } ) = ( 1 , M )$ and liquidity weights $\begin{array} { r } { w ^ { \star } = ( \frac { 1 } { M + 1 } , \frac { M } { M + 1 } ) } \end{array}$ . Fix any target $r > 1$ and set $t _ { 2 } = 1$ and $t _ { 1 } = 1 + ( r - 1 ) ( M + 1 )$ ; then

$$
w _ {1} ^ {\star} t _ {1} + w _ {2} ^ {\star} t _ {2} = \frac {1}{M + 1} \big (1 + (r - 1) (M + 1) \big) + \frac {M}{M + 1} \cdot 1 = r,
$$

so the attack is feasible at cost $F _ { 2 } ( w ^ { \star } ; r ) \leq f ( 1 + ( r - 1 ) ( M + 1 ) )$ , while the pooled benchmark costs $( M + 1 ) f ( r )$ . Since $f ( t ) \leq { \sqrt { t } }$ for all $t \geq 1$ and $f ( t ) \sim \sqrt { t }$ as $t \to \infty$ , the ratio between these two costs is $O ( M ^ { - 1 / 2 } )$ as $M  \infty .$ . Thus the pooled-liquidity benchmark $y _ { \mathrm { t o t } } f ( r )$ cannot hold uniformly in r away from $r = 1$ . Moreover, the same example shows that liquidity weights need not be max–min optimal for weighted means at large distortion levels: if the designer instead ignores the shallow pool and sets weights $\widetilde { w } = ( 0 , 1 )$ , then the oracle depends only on pool 2 and any successful attack must set $t _ { 2 } = r ,$ , incurring cost $F _ { 2 } ( \widetilde { w } ; r ) = M f ( r )$ . For fixed $r > 1$ (in particular for $r > 3 )$ , this scales like $\Theta ( M )$ , while the feasible concentrated attack above achieves cost at most $f ( 1 + ( r - 1 ) ( M + 1 ) ) = O ( { \sqrt { M } } )$ ) under liquidity weights. Hence, for M suficiently large, we yields strictly larger minimal manipulation cost than $w ^ { \star }$ . Accordingly, Theorem 1 is stated as a small-distortion result.

A concentration bound and a large-distortion design heuristic. For any weights w and any target $r \geq 1$ , a feasible attack is to leave all pools at $t _ { j } = 1$ except one pool $i ,$ and set $t _ { i } = 1 + ( r - 1 ) / w _ { i } ;$ this yields

$$
F _ {N} (\boldsymbol {w}; r) \leq \min _ {i} y _ {i} ^ {(0)} f \Big (1 + \frac {r - 1}{w _ {i}} \Big).
$$

Since $f ( t ) \sim \sqrt { t }$ as $t \to \infty$ , this upper bound behaves like $\sqrt { r - 1 }$ min<sub>i</sub> $y _ { i } ^ { ( 0 ) } / \sqrt { w _ { i } }$ for large distortions, suggesting quadratic liquidity weights $w _ { i } \propto ( y _ { i } ^ { ( 0 ) } ) ^ { 2 }$ as a conservative way to equalize the cost of such single-pool attacks.

## Second-order expansion and attacker optimum

Fix weights $w _ { i } \geq 0$ with $\textstyle \sum _ { i } w _ { i } = 1$ and set $r = 1 + \varepsilon$ with $\varepsilon  0 ^ { + }$ . Write $t _ { i } = 1 + \delta _ { i }$ with $\delta _ { i } > - 1$ , so the constraint becomes

$$
\sum_ {i = 1} ^ {N} w _ {i} \delta_ {i} = \varepsilon .
$$

Since the constant choice $\delta _ { i } \equiv \varepsilon$ is feasible, we have ${ \cal F } _ { N } ( { \pmb w } ; 1 + \varepsilon ) \le y _ { \mathrm { t o t } } f ( 1 + \varepsilon ) =$ $O ( \varepsilon ^ { 2 } )$ uniformly in w. Because $f ( 1 + \delta )  \infty$ as $\delta \downarrow - 1$ or $\delta  \infty ,$ , any minimizer must satisfy ma $\bar { \mathbf { \rho } } _ { \bar { \mathbf { \rho } } _ { i } } \left| \delta _ { i } \right| \to 0$ as $\varepsilon \to 0$ (otherwise the objective would be bounded below by a positive constant). Hence we may use the Taylor expansion

$$
f (1 + \delta) = \frac {\delta^ {2}}{4} + O (\delta^ {3}) \qquad (\delta \to 0),
$$

to obtain

$$
F _ {N} (\boldsymbol {w}; 1 + \varepsilon) = \frac {1}{4} \min \left\{\sum_ {i = 1} ^ {N} y _ {i} ^ {(0)} \delta_ {i} ^ {2}: \sum_ {i = 1} ^ {N} w _ {i} \delta_ {i} = \varepsilon \right\} + o (\varepsilon^ {2}).
$$

The quadratic program is solved by Cauchy–Schwarz:

$$
\varepsilon^ {2} = \Big (\sum_ {i = 1} ^ {N} w _ {i} \delta_ {i} \Big) ^ {2} \leq \Big (\sum_ {i = 1} ^ {N} \frac {w _ {i} ^ {2}}{y _ {i} ^ {(0)}} \Big) \Big (\sum_ {i = 1} ^ {N} y _ {i} ^ {(0)} \delta_ {i} ^ {2} \Big),
$$

with equality if $\delta _ { i } \propto w _ { i } / y _ { i } ^ { ( 0 ) }$ . Therefore,

$$
\min \left\{\sum_ {i = 1} ^ {N} y _ {i} ^ {(0)} \delta_ {i} ^ {2}: \sum_ {i = 1} ^ {N} w _ {i} \delta_ {i} = \varepsilon \right\} = \frac {\varepsilon^ {2}}{\sum_ {i = 1} ^ {N} w _ {i} ^ {2} / y _ {i} ^ {(0)}},
$$

and thus

$$
F _ {N} (\boldsymbol {w}; 1 + \varepsilon) = \frac {\varepsilon^ {2}}{4} \frac {1}{\sum_ {i = 1} ^ {N} w _ {i} ^ {2} / y _ {i} ^ {(0)}} + o (\varepsilon^ {2}).
$$

## Designer optimum

To maximize the leading-order term, the oracle designer minimizes $\textstyle \sum _ { i } w _ { i } ^ { 2 } / y _ { i } ^ { ( 0 ) }$ over all weights $w _ { i } \geq 0$ with $\textstyle \sum _ { i } w _ { i } = 1$ . By Cauchy–Schwarz,

$$
1 = \Big (\sum_ {i = 1} ^ {N} w _ {i} \Big) ^ {2} \leq \Big (\sum_ {i = 1} ^ {N} \frac {w _ {i} ^ {2}}{y _ {i} ^ {(0)}} \Big) \Big (\sum_ {i = 1} ^ {N} y _ {i} ^ {(0)} \Big) = y _ {\mathrm{tot}} \sum_ {i = 1} ^ {N} \frac {w _ {i} ^ {2}}{y _ {i} ^ {(0)}},
$$

with equality if $w _ { i } \propto y _ { i } ^ { ( 0 ) }$ . Hence $\textstyle \sum _ { i } w _ { i } ^ { 2 } / y _ { i } ^ { ( 0 ) } \geq 1 / y _ { \mathrm { t o t } }$ , and the leading-order cost is uniquely maximized by the liquidity weights $w _ { i } ^ { \star } = y _ { i } ^ { ( 0 ) } / y _ { \mathrm { t o t } }$ , giving

$$
\sup _ {w _ {i} \geq 0, \sum_ {i} w _ {i} = 1} F _ {N} (\boldsymbol {w}; 1 + \varepsilon) = \frac {\varepsilon^ {2}}{4} y _ {\mathrm{tot}} + o (\varepsilon^ {2}).
$$

## D Weighted-Median Multi-Pool Optimization

We now prove the weighted-median weight-design result stated in Theorem 2. Recall the setup of the N-pool median: for quote reserves $y _ { i } ^ { ( 0 ) } > 0$ and weights $w _ { i } \geq 0$ with $\textstyle \sum _ { i } w _ { i } = 1$ , the attacker’s minimal cost to enforce a distortion factor $r \geq 1$ is

$$
\begin{array}{c} C ^ {\mathrm{med}} (r; w) = f (r)   m (w), \\ m (w) := \min \Big \{\sum_ {i \in S} y _ {i} ^ {(0)}: S \subset \{1, \dots , N \}, \sum_ {i \in S} w _ {i} \geq \frac {1}{2} \Big \}. \end{array}
$$

Thus the oracle designer solves

$$
\sup _ {w \in \varDelta_ {N}} C ^ {\mathrm{med}} (r; w) = f (r) \sup _ {w \in \varDelta_ {N}} m (w), \qquad \varDelta_ {N} = \{w _ {i} \geq 0, \sum_ {i} w _ {i} = 1 \}.
$$

We therefore focus on the purely combinatorial quantity $m ( w )$

Let $\pmb { y } = ( y _ { 1 } ^ { ( 0 ) } , \dots , y _ { N } ^ { ( 0 ) } )$ and $\begin{array} { r } { y _ { \mathrm { t o t } } = \sum _ { i } y _ { i } ^ { ( 0 ) } } \end{array}$ . Define

$$
\Theta (\boldsymbol {y}) := \min \Big \{\sum_ {i \in S} y _ {i} ^ {(0)}: S \subset \{1, \dots , N \}, \sum_ {i \in S} y _ {i} ^ {(0)} \geq \frac {1}{2} y _ {\mathrm{tot}} \Big \},
$$

i.e., the smallest total depth carried by any subset whose total depth is at least half of $y _ { \mathrm { t o t } }$

Lemma 1. For any y as above,

$$
\frac {1}{2} y _ {\mathrm{tot}} \leq \Theta (\boldsymbol {y}) \leq \frac {1}{2} y _ {\mathrm{tot}} + y _ {\max} ^ {(0)},
$$

where $y _ { \mathrm { m a x } } ^ { ( 0 ) } : = \operatorname* { m a x } _ { i } y _ { i } ^ { ( 0 ) }$ . Moreover, when $N = 2$ we have $\theta ( \pmb { y } ) = \operatorname* { m a x } ( y _ { 1 } ^ { ( 0 ) } , y _ { 2 } ^ { ( 0 ) } )$ Proof. By definition, every admissible subset S satisfies $\begin{array} { r } { \sum _ { i \in S } y _ { i } ^ { ( 0 ) } \ge \frac { 1 } { 2 } y _ { \mathrm { t o t } } } \end{array}$ , so the minimum is at least $\frac { 1 } { 2 } y _ { \mathrm { t o t } }$ . For the upper bound, sort indices so that $y _ { ( 1 ) } ^ { ( 0 ) } \leq$ $\cdots \le y _ { ( N ) } ^ { ( 0 ) }$ and let k be the smallest index with $\textstyle \sum _ { i = 1 } ^ { k } y _ { ( i ) } ^ { ( 0 ) } \geq { \frac { 1 } { 2 } } y _ { \mathrm { t o t } }$ . Then $\Theta ( \pmb { y } ) =$ $\textstyle \sum _ { i = 1 } ^ { k } y _ { ( i ) } ^ { ( 0 ) }$ and

$$
\Theta (\boldsymbol {y}) \leq \frac {1}{2} y _ {\mathrm{tot}} + y _ {(k)} ^ {(0)} \leq \frac {1}{2} y _ {\mathrm{tot}} + y _ {\max} ^ {(0)}.
$$

When $N = 2$ , either $y _ { 1 } ^ { ( 0 ) } \ge \frac { 1 } { 2 } y _ { \mathrm { t o t } }$ or $y _ { 2 } ^ { ( 0 ) } \ge \frac { 1 } { 2 } y _ { \mathrm { t o t } }$ (or both), and the minimal subset achieving the half-depth threshold is the index with larger depth, so $\Theta ( \pmb { y } ) = \operatorname* { m a x } ( y _ { 1 } ^ { ( 0 ) } , y _ { 2 } ^ { ( 0 ) } )$ .

Lemma 2. For any $w \in \varDelta _ { N }$ and $r > 0$

$$
C _ {\mathrm{med}} (r; w) = f (r)   m (w),
$$

with $m ( w )$ as defined above.

Proof. By definition of the weighted median, moving the median from $p _ { 0 } ~ \mathrm { t o } ~ r p _ { 0 }$ requires the attacker to select a subset S of pools whose cumulative weight is at least $1 / 2$ and move exactly those pools to $r p _ { 0 } ;$ moving any pool to a price in $( p _ { 0 } , r p _ { 0 } )$ does not change the median level. Since all pools start at $p _ { 0 }$ and each moved pool must be at $r p _ { 0 }$ , the attack cost is the sum of per-pool manipulation costs $C _ { i } ( r ) = y _ { i } ^ { ( 0 ) } f ( r )$ over $i \in S$ . Minimizing over all subsets with $\textstyle \sum _ { i \in S } w _ { i } \geq 1 / 2$ yields precisely the expression for $m ( w )$

Proposition 1. For any y as above and any $r \geq 1$ 2

$$
\sup _ {w \in \varDelta_ {N}} C _ {\mathrm{med}} (r; w) = f (r)   \Theta (\boldsymbol {y}),
$$

attained by the liquidity weights $v _ { i } ^ { \star } = y _ { i } ^ { ( 0 ) } / y _ { \mathrm { t o t } }$ . In particular,

$$
\sup _ {w \in \varDelta_ {N}} m (w) = \Theta (\boldsymbol {y}).
$$

Proof. By Lemma 2 it sufices to study $\mathrm { s u p } _ { w \in \varDelta _ { N } } m ( w )$

Lower bound and attainment. Take liquidity weights $w _ { i } ^ { \star } = y _ { i } ^ { ( 0 ) } / y _ { \mathrm { t o t } }$ . For any subset $S _ { ; }$ ,

$$
\sum_ {i \in S} w _ {i} ^ {\star} \geq \frac {1}{2} \quad \Longleftrightarrow \quad \sum_ {i \in S} y _ {i} ^ {(0)} \geq \frac {1}{2} y _ {\mathrm{tot}}.
$$

Thus

$$
m (w ^ {\star}) = \min \Big \{\sum_ {i \in S} y _ {i} ^ {(0)}: \sum_ {i \in S} y _ {i} ^ {(0)} \geq \frac {1}{2} y _ {\mathrm{tot}} \Big \} = \Theta (\boldsymbol {y}),
$$

so $\operatorname* { s u p } _ { w } m ( w ) \geq m ( w ^ { \star } ) = \theta ( y )$ and therefore

$$
\sup _ {w} C _ {\mathrm{med}} (r; w) \geq f (r) \Theta (\boldsymbol {y}).
$$

Upper bound. Fix an arbitrary w $\in \varDelta _ { N }$ . By definition of $\Theta ( \pmb { y } )$ there are only finitely many candidate subsets $S \subset \{ 1 , \ldots , N \}$ , so the minimum in its definition is attained. Let $S ^ { \theta }$ be any subset achieving this minimum, i.e.,

$$
\sum_ {i \in S ^ {\theta}} y _ {i} ^ {(0)} = \Theta (\boldsymbol {y}), \quad \sum_ {i \in S ^ {\theta}} y _ {i} ^ {(0)} \geq \frac {1}{2} y _ {\mathrm{tot}}.
$$

There are two cases.

$\begin{array} { r } { - \mathrm { ~ I f ~ } \sum _ { i \in S ^ { \theta } } w _ { i } \geq \frac { 1 } { 2 } } \end{array}$ , then $S ^ { \theta }$ is feasible for $m ( w )$ , so

$$
m (w) \leq \sum_ {i \in S ^ {\theta}} y _ {i} ^ {(0)} = \Theta (\boldsymbol {y}).
$$

If $\textstyle \sum _ { i \in S ^ { \theta } } w _ { i } < { \frac { 1 } { 2 } }$ , let $S ^ { c }$ be its complement. Then

$$
\sum_ {i \in S ^ {c}} w _ {i} = 1 - \sum_ {i \in S ^ {\theta}} w _ {i} > \frac {1}{2},
$$

so $S ^ { c }$ is feasible for $m ( w )$ . Its cost is

$$
\sum_ {i \in S ^ {c}} y _ {i} ^ {(0)} = y _ {\mathrm{tot}} - \Theta (\boldsymbol {y}) \leq y _ {\mathrm{tot}} - \frac {1}{2} y _ {\mathrm{tot}} = \frac {1}{2} y _ {\mathrm{tot}} \leq \Theta (\boldsymbol {y}),
$$

where the first inequality uses the lower bound $\begin{array} { r } { \Theta ( \pmb { y } ) \ge \frac { 1 } { 2 } y _ { \mathrm { t o t } } } \end{array}$ from Lemma 1 (which implies $y _ { \mathrm { t o t } } - \Theta ( \pmb { y } ) \leq y _ { \mathrm { t o t } } - \frac { 1 } { 2 } y _ { \mathrm { t o t } } )$ , and the last inequality uses the same bound to conclude $\begin{array} { r } { \frac 1 2 y _ { \mathrm { t o t } } \le \Theta ( y ) } \end{array}$ . Hence

$$
m (w) \leq \sum_ {i \in S ^ {c}} y _ {i} ^ {(0)} \leq \Theta (\boldsymbol {y}).
$$

In all cases we have $m ( w ) \leq \Theta ( { \pmb y } )$ , so $\operatorname* { s u p } _ { w } m ( w ) \leq \theta ( { \pmb y } )$ . Combined with the lower bound and attainment at $w ^ { \star }$ , this proves the claim.

## E Star-Architecture Multi-Asset Extension

Theorem 3 (Star-architecture optimal weights). Let the asset set be $A =$ $\{ 0 , 1 , \ldots , M \}$ , with asset 0 a numéraire $( e . g .$ , a stablecoin). For each $a \ne 0$ suppose there are CPMM pools indexed by $i \in \mathcal { T } _ { a , 0 }$ quoting a against the numéraire with quote reserves $y _ { a , i } ^ { ( 0 ) }$ , and let the oracle report

$$
\hat {p} _ {a} = A _ {a} \big (\{p _ {a, i} ^ {(1)} \} _ {i \in \mathcal {I} _ {a, 0}} \big), \qquad a = 1, \dots , M,
$$

where each $A _ { a }$ is either a weighted mean or a weighted median with weights $w _ { a , i } > 0 , \sum _ { i } w _ { a , i } = 1$

Fix target distortion factors ${ r _ { a } } = 1 + { \varepsilon } _ { a }$ with $\varepsilon _ { a } \to 0 ^ { + }$ for $a = 1 , \dots , M$ , and write Cost ${ _ { x } } ( r _ { a } ; w _ { a , \cdot } )$ for the minimal economic loss required to distort $\hat { p } _ { a }$ by a factor of at least $r _ { a }$ (upward or downward). Then,

1. for each asset $^ { a , }$ liquidity weights $w _ { a , i } ^ { \star } \propto y _ { a , i } ^ { ( 0 ) }$ maximize the leading-order (second-order in $ { \varepsilon } _ { a } )$ term of $\mathrm { C o s t } _ { a } ( 1 + \varepsilon _ { a } ; w _ { a , \cdot } )$ when $A _ { a }$ is a weighted mean, and

2. maximize $\mathrm { C o s t } _ { a } ( r _ { a } ; w _ { a , \cdot } )$ for any $r _ { a } \geq 1$ when $A _ { a }$ is a weighted median

Moreover, the star architecture decouples assets, so the minimal total cost to distort the vector $( \hat { p } _ { a } ) _ { a \ne 0 }$ by factors $r _ { a }$ equals $\begin{array} { r } { \sum _ { a = 1 } ^ { M } \mathrm { C o s t } _ { a } ( r _ { a } ; w _ { a , \cdot } ^ { \star } ) } \end{array}$

Proof. Throughout, assets are indexed by $a \ \in \ \{ 1 , \ldots , M \}$ , with asset 0 a numéraire. For each a we write $\mathcal { T } _ { a , 0 }$ for the set of CPMM pools quoting a against the numéraire, and we write $p _ { a , i } ^ { ( t ) }$ for the on-chain marginal price of one unit of a in numéraire units in pool $i \in \mathcal { T } _ { a , 0 }$ at time $t \in \{ 0 , 1 \}$ (pre-attack $t = 0$ post-attack $t = 1 )$

## Reduction to Single-Asset Problems

Fix an asset $a \neq 0$ and a target distortion factor ${ r _ { a } } = 1 + { \varepsilon } _ { a }$ with $\varepsilon _ { a } \to 0 ^ { + }$ . In the star architecture, the oracle price for a is

$$
\hat {p} _ {a} = A _ {a} \big (\{p _ {a, i} ^ {(1)} \} _ {i \in \mathcal {I} _ {a, 0}} \big),
$$

where $A _ { a }$ is either a weighted mean or a weighted median with weights $w _ { a , i } > 0$ $\textstyle \sum _ { i } w _ { a , i } = 1$ . By construction, $\hat { p } _ { a }$ depends only on the pools in $\mathcal { T } _ { a , 0 }$ , and trades on pools for other assets do not enter $A _ { a }$

Let $\mathrm { C o s t } _ { a } ( r _ { a } ; w _ { a , \cdot } )$ denote the minimal economic loss (in numéraire units) that an attacker must incur to enforce a distortion of at least $r _ { a }$ on $\hat { p } _ { a }$ (upward or downward) using trades on the CPMMs in $\mathcal { T } _ { a , 0 } .$ , holding all other assets fixed. This is exactly the single-asset multi-pool manipulation problem analyzed in the main text, with $( X , Y )$ replaced by $( a , 0 )$ , quote reserves $\{ y _ { a , i } ^ { ( 0 ) } \}$ , and aggregation weights $\{ w _ { a , i } \}$

Consequently:

– If $A _ { a }$ is a weighted mean, Theorem 1 applies $( \mathrm { a s } \ \varepsilon _ { a } \to 0 )$ and implies that the maximizing weights $w _ { a , i } ^ { \star }$ are the liquidity weights

$$
w _ {a, i} ^ {\star} = \frac {y _ {a , i} ^ {(0)}}{\sum_ {j \in \mathcal {I} _ {a , 0}} y _ {a , j} ^ {(0)}}.
$$

– If $A _ { a }$ is a weighted median, Theorem 2 yields the same conclusion: liquidity weights $w _ { a , i } ^ { \star } \propto y _ { a , i } ^ { ( 0 ) }$ maximize the minimal manipulation cost for $\hat { p } _ { a }$

Thus, for each asset separately, liquidity weights are max–min optimal within weighted medians and asymptotically max–min optimal within weighted means $( \mathrm { a s } \ \varepsilon _ { a } \to 0 )$

## Separability Across Assets

In the star architecture, the CPMM pools split into M disjoint groups $\{ { \mathcal { T } } _ { a , 0 } \} _ { a = 1 } ^ { M } ,$ one per asset–numéraire pair. The attacker’s total cost to distort a vector of prices $( \hat { p } _ { a } ) _ { a \ne 0 }$ by factors $r _ { a }$ is the sum of the per-asset costs:

$$
\sum_ {a = 1} ^ {M} \mathrm{Cost} _ {a} (r _ {a}; w _ {a, \cdot}),
$$

because trades on $\mathcal { T } _ { a , 0 }$ afect only asset a and costs are measured in the common numéraire. There are no cross-terms in the objective, and no constraints couple trades across diferent $\mathcal { T } _ { a , 0 }$

The oracle designer’s max–min problem is therefore

$$
\sup _ {\{w _ {a, \cdot} \}} \inf _ {\text {attacks}} \sum_ {a = 1} ^ {M} \mathrm{Cost} _ {a} (r _ {a}; w _ {a, \cdot}) = \sum_ {a = 1} ^ {M} \sup _ {w _ {a, \cdot}} \inf _ {\text {attacks on} \mathcal {I} _ {a, 0}} \mathrm{Cost} _ {a} (r _ {a}; w _ {a, \cdot}),
$$

where the equality follows from separability of both the cost and the admissible attack sets across assets. Maximizing each term on the right-hand side independently and using the single-asset results above shows that the joint optimizer is given by choosing, for every $^ { a , }$ the liquidity weights $w _ { a , i } ^ { \star } \propto y _ { a , i } ^ { ( 0 ) }$

With this choice, the minimal total cost decomposes as the sum of the optimal per-asset values $\mathrm { C o s t } _ { a } ( r _ { a } ; w _ { a , \cdot } ^ { \star } )$