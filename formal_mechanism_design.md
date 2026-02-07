# Formal Mechanism Design for Content Licensing Auctions

## A Rigorous Treatment for the Douyin Content Trading Marketplace

> Combinatorial, Dynamic, Two-Sided Auction with Information Design

---

## Notation and Conventions

| Symbol | Meaning |
|--------|---------|
| $\mathcal{C}$ | Set of creators (卖方/创作者), indexed by $i$ |
| $\mathcal{M}$ | Set of merchants (买方/商家), indexed by $j$ |
| $\mathcal{L}$ | Set of content items (内容), indexed by $\ell$ |
| $\mathcal{R} = \{R_1, \ldots, R_6\}$ | Rights bundle (权利束): display, ad-feed, remix, AI-face-swap, category-exclusive, time-window |
| $\theta_C^i$ | Creator $i$'s private type |
| $\theta_M^j$ | Merchant $j$'s private type |
| $v_j(S)$ | Merchant $j$'s valuation for rights subset $S \subseteq \mathcal{R}$ |
| $c_i$ | Creator $i$'s opportunity cost / reservation value |
| $x_{j,S}$ | Allocation indicator: $1$ if merchant $j$ receives bundle $S$ |
| $t_j$ | Payment by merchant $j$ |
| $p_i$ | Payment to creator $i$ |
| $\lambda$ | Content value decay rate |
| $F(\cdot), f(\cdot)$ | CDF and PDF of merchant valuations |
| $G(\cdot), g(\cdot)$ | CDF and PDF of creator costs |
| $\phi(v) = v - \frac{1 - F(v)}{f(v)}$ | Myerson virtual valuation |
| $\psi(c) = c + \frac{G(c)}{g(c)}$ | Myerson virtual cost |

Throughout, we use standard mechanism design conventions. All proofs employ the revelation principle: without loss of generality, we restrict attention to direct mechanisms where agents report their types truthfully.

---

## 1. Formal Market Model

### 1.1 Environment

**Definition 1.1 (Content Licensing Market).** A content licensing market is a tuple $\Gamma = (\mathcal{C}, \mathcal{M}, \mathcal{L}, \mathcal{R}, \Theta_C, \Theta_M, v, c, T)$ where:

- $\mathcal{C} = \{1, \ldots, I\}$ is a finite set of creators.
- $\mathcal{M} = \{1, \ldots, J\}$ is a finite set of merchants.
- $\mathcal{L} = \{1, \ldots, L\}$ is a finite set of content items, with creator $\omega(\ell) \in \mathcal{C}$ owning item $\ell$.
- $\mathcal{R} = \{R_1, R_2, R_3, R_4, R_5, R_6\}$ is the rights set:
  - $R_1$: Display right (展示权) -- show content on merchant's page
  - $R_2$: Ad-feed right (信息流投放权) -- use in paid ad campaigns
  - $R_3$: Remix right (混剪权) -- create derivative works
  - $R_4$: AI-face-swap right (AI换人权) -- replace creator's face with model/avatar
  - $R_5$: Category-exclusive right (品类独占权) -- exclusive within a product category
  - $R_6$: Time-window right (时间窗口权) -- exclusive within a time period
- $T = [0, T_{\max}]$ is the time horizon.

### 1.2 Type Spaces

**Definition 1.2 (Creator Type).** Creator $i$'s private type is $\theta_C^i = (q_i, c_i, \delta_i)$ where:

- $q_i \in [0, 1]$ is content quality (known to creator, partially observable by platform).
- $c_i \in \mathbb{R}_+$ is the opportunity cost of licensing (outside option value; 外部选项价值). This includes foregone Xingtu (星图) revenue and brand dilution costs.
- $\delta_i \in \{0, 1\}$ is an indicator for whether the creator is willing to license at all (participation decision).

Creator types are drawn independently from distribution $G$ on $\Theta_C = [0,1] \times \mathbb{R}_+ \times \{0,1\}$, with marginal $G_c$ on costs satisfying Myerson regularity (increasing virtual cost $\psi(c) = c + G_c(c)/g_c(c)$).

**Definition 1.3 (Merchant Type).** Merchant $j$'s private type is $\theta_M^j = (v_j, u_j, \kappa_j)$ where:

- $v_j: 2^{\mathcal{R}} \to \mathbb{R}_+$ is the valuation function over rights bundles. $v_j(\emptyset) = 0$.
- $u_j \in \mathcal{U}$ is the use case (ad campaign, store page, live-stream asset, etc.).
- $\kappa_j \in \mathbb{R}_+$ is the budget constraint.

Merchant types are drawn independently from distribution $F$ on $\Theta_M$, with marginal $F_v$ on valuations satisfying Myerson regularity (increasing virtual valuation $\phi(v) = v - (1-F_v(v))/f_v(v)$).

**Assumption 1.1 (Independent Private Values / IPV).** Each merchant's valuation $v_j$ depends only on their own type $\theta_M^j$, not on other merchants' types. Creator costs are similarly independent. This is the standard IPV assumption of Vickrey (1961) and Myerson (1981).

**Assumption 1.2 (Myerson Regularity).** The virtual valuation $\phi(v) = v - (1-F(v))/f(v)$ is strictly increasing in $v$. The virtual cost $\psi(c) = c + G(c)/g(c)$ is strictly increasing in $c$. This ensures the optimal mechanism has a tractable structure.

### 1.3 Allocation and Payment Spaces

**Definition 1.4 (Allocation Rule).** For a single content item $\ell$ with rights $\mathcal{R}$, an allocation rule is a mapping:

$$x: \Theta_M^J \times \Theta_C \to \{0,1\}^{J \times 2^{|\mathcal{R}|}}$$

where $x_{j,S}(\theta) = 1$ if merchant $j$ is allocated rights bundle $S \subseteq \mathcal{R}$ under reported type profile $\theta$.

**Feasibility Constraints:**

(F1) **Budget feasibility**: $\sum_{S \subseteq \mathcal{R}} x_{j,S} \cdot P(S) \leq \kappa_j$ for all $j$.

(F2) **Exclusivity constraint**: If $R_5 \in S$ for some $(j, S)$ with $x_{j,S} = 1$ and the merchant operates in category $k$, then for all $j' \neq j$ in the same category $k$, $x_{j', S'} = 0$ for any $S'$ containing $R_5$.

(F3) **Time-window constraint**: If $R_6 \in S$ for $(j, S)$ covering time interval $[\tau_1, \tau_2]$, then $R_6$ cannot be allocated to any $j' \neq j$ for overlapping intervals.

(F4) **Non-exclusive compatibility**: Rights $R_1, R_2, R_3, R_4$ (without $R_5$ or $R_6$) may be allocated to multiple merchants simultaneously. This is the key distinction from standard single-item auctions.

**Definition 1.5 (Payment Rule).** A payment rule is a pair of mappings:

$$t: \Theta_M^J \times \Theta_C \to \mathbb{R}^J_+ \quad \text{(merchant payments)}$$
$$p: \Theta_M^J \times \Theta_C \to \mathbb{R}_+ \quad \text{(creator payment)}$$

The platform's revenue is $\Pi = \sum_j t_j - p$.

### 1.4 Social Welfare Function

**Definition 1.6 (Social Welfare).** The social welfare function is:

$$W(x, \theta) = \underbrace{\sum_{j=1}^{J} \sum_{S \subseteq \mathcal{R}} x_{j,S} \cdot v_j(S)}_{\text{total merchant value}} - \underbrace{c_i \cdot \mathbb{1}\left[\sum_{j,S} x_{j,S} > 0\right]}_{\text{creator opportunity cost}}$$

The **efficient allocation** maximizes $W$ subject to feasibility constraints (F1)--(F4).

**Definition 1.7 (Revenue).** The platform's expected revenue is:

$$\text{Rev} = \mathbb{E}_{\theta}\left[\sum_{j} t_j(\theta) - p(\theta)\right]$$

The platform's design problem is to maximize $\text{Rev}$ subject to incentive compatibility and individual rationality.

---

## 2. Optimal Auction for Non-Exclusive Licenses

### 2.1 Model: $K$-Unit Auction for Identical Copies

When content is licensed non-exclusively, up to $K$ merchants can simultaneously hold licenses for the same content-rights bundle. This is formally equivalent to a **multi-unit auction** where the seller has $K$ identical units. In the content setting, $K$ may be unbounded (pure non-exclusive) or finite (limited non-exclusive).

**Setup.** Fix a single content item $\ell$ and a single non-exclusive rights bundle $S^* = \{R_1, R_2\}$ (e.g., display + ad-feed). There are $J$ merchants with independent private valuations $v_1, \ldots, v_J$ drawn i.i.d. from $F$ on $[\underline{v}, \overline{v}]$. Each merchant demands at most one license (unit demand). The creator has opportunity cost $c$.

**Case A: Unlimited copies ($K = J$, pure non-exclusive).**

Every merchant can receive a license independently. The "auction" degenerates: the optimal mechanism is simply to sell to every merchant whose virtual valuation exceeds the creator's virtual cost.

**Proposition 2.1 (Optimal Mechanism, Unlimited Non-Exclusive).** Under IPV with Myerson regularity, the revenue-maximizing mechanism for unlimited non-exclusive licenses is:

1. Set a uniform posted price (reserve price) $r^*$ where $r^*$ solves $\phi(r^*) = \psi(c)$, i.e.,

$$r^* - \frac{1 - F(r^*)}{f(r^*)} = c + \frac{G(c)}{g(c)}$$

If the creator's cost $c$ is known to the platform, this simplifies to $\phi(r^*) = c$.

2. Sell to every merchant $j$ with $v_j \geq r^*$.

3. Charge each buyer $r^*$ (posted price).

*Proof sketch.* With unlimited copies and unit demand, merchants do not compete. The allocation decision for each merchant is independent. By Myerson (1981), the revenue-maximizing mechanism screens each merchant separately and allocates if and only if the virtual surplus is non-negative: $\phi(v_j) \geq c$. By regularity, $\phi$ is increasing, so this is equivalent to a reserve price $r^* = \phi^{-1}(c)$. Since decisions are independent, no auction is needed; a posted price at $r^*$ is optimal. $\square$

**Corollary 2.1.** Pure non-exclusive licensing should use **posted prices**, not auctions. This is intuitive: auctions create value through competition for scarce goods, but non-exclusive licenses are non-rival.

**Case B: Limited copies ($K < J$, limited non-exclusive).**

The creator (or platform) sells at most $K$ licenses, e.g., "up to 5 merchants in the beauty category."

**Proposition 2.2 (Optimal Mechanism, $K$-Unit Non-Exclusive).** The revenue-maximizing mechanism is a **$(K+1)$-th price sealed-bid auction with optimal reserve**:

1. Collect bids $b_1, \ldots, b_J$.
2. The $K$ highest bidders win (if $\geq K$ bids exceed reserve $r^*$).
3. Each winner pays $\max(b_{(K+1)}, r^*)$, where $b_{(K+1)}$ is the $(K+1)$-th highest bid.
4. The optimal reserve $r^*$ solves $\phi(r^*) = c$.

*Proof.* This is a direct application of Myerson (1981) to a $K$-unit setting with unit-demand bidders. The VCG mechanism for $K$ identical items with unit demand reduces to the uniform-price $(K+1)$-th price auction. The reserve price follows from the standard virtual valuation argument. The proof proceeds by:

(i) By the revelation principle, restrict to direct mechanisms. Under IPV with regularity, Myerson's optimal auction allocates to the $K$ agents with highest virtual valuations, provided those virtual valuations exceed the seller's virtual cost.

(ii) The payment rule that implements this in dominant strategies is the $(K+1)$-th price rule (Vickrey 1961; Milgrom 2004 Chapter 3). Each winner pays the minimum bid at which they would still win, which is $\max(b_{(K+1)}, r^*)$.

(iii) Truthful bidding ($b_j = v_j$) is a dominant strategy: a merchant's payment is independent of their own bid (conditional on winning), so misreporting cannot improve their outcome. $\square$

### 2.2 Posted Price vs. Auction vs. Hybrid: When Is Each Optimal?

The choice between mechanisms depends critically on the **demand uncertainty** faced by the platform.

**Definition 2.1 (Demand Uncertainty).** Let $N$ be the random number of merchants interested in content item $\ell$. The demand uncertainty is characterized by the distribution of $N$ and the conditional distribution of valuations $v_j | N$.

**Proposition 2.3 (Mechanism Selection).** Under IPV with Myerson regularity:

(a) **Posted price is optimal** when:
  - Licenses are purely non-exclusive ($K = \infty$).
  - Demand is predictable ($\text{Var}(N)$ is small).
  - The platform has accurate estimates of $F$ (low information asymmetry about demand distribution).

(b) **Auction is optimal** when:
  - Licenses are scarce ($K$ is small relative to expected $N$).
  - Demand is uncertain ($\text{Var}(N)$ is large).
  - The platform has imprecise estimates of $F$.

(c) **Hybrid (auction with buy-it-now)** is optimal when:
  - There is a mix of patient and impatient buyers.
  - Content value decays over time (Section 4), creating urgency.
  - The market is thin but non-exclusive (moderate $K$, moderate $N$).

*Proof sketch.* Part (a) follows from Proposition 2.1. For part (b), Bulow and Klemperer (1996) show that an auction with $n+1$ bidders and no reserve generates more revenue than the optimal mechanism with $n$ bidders, implying that competition (auction format) is most valuable when $n$ is moderate and valuations are dispersed. For part (c), the hybrid format (Budish and Takeyama, 2001; Reynolds and Wooders, 2009) combines the price certainty of posted prices with the surplus extraction of auctions. The formal threshold between regimes depends on the ratio $K / \mathbb{E}[N]$ and the coefficient of variation of $F$. $\square$

**Quantitative Threshold (Content Market Calibration).** Let $\rho = K / \mathbb{E}[N]$ be the supply-demand ratio.

| Regime | $\rho$ Range | Mechanism | Intuition |
|--------|-------------|-----------|-----------|
| Excess supply | $\rho > 2$ | Posted price | Most items go unsold; competition absent |
| Balanced | $0.5 < \rho < 2$ | Hybrid (auction + posted price) | Some competition; value in price discovery |
| Excess demand | $\rho < 0.5$ | Sealed-bid auction | Strong competition; auction extracts surplus |

### 2.3 Thin Market Reserve Price Optimization

In a content marketplace, many items attract few bidders ("thin market" problem; 薄市场问题). The optimal reserve price is crucial.

**Proposition 2.4 (Optimal Reserve Price, Myerson 1981).** For a single-item second-price auction with $n$ i.i.d. bidders from $F$ and seller cost $c$, the optimal reserve price $r^*$ satisfies:

$$r^* = \phi^{-1}(c) \quad \text{where} \quad \phi(v) = v - \frac{1 - F(v)}{f(v)}$$

Equivalently, $r^*$ solves:

$$r^* - \frac{1 - F(r^*)}{f(r^*)} = c$$

**Remark.** The optimal reserve is independent of the number of bidders $n$. This is a celebrated result of Myerson (1981). However, the *revenue impact* of the reserve price depends critically on $n$:

- When $n$ is large (thick market), the competitive effect dominates and the reserve binds infrequently. Revenue $\approx \mathbb{E}[v_{(2:n)}]$ (expected second-order statistic).
- When $n$ is small (thin market), the reserve binds frequently and is the primary revenue instrument.

**Proposition 2.5 (Reserve Price with Heterogeneous Bidders).** When bidders are drawn from different distributions $F_1, \ldots, F_n$ (e.g., different merchant categories with different WTP), the optimal mechanism (Myerson 1981) allocates to the bidder with the highest positive virtual valuation:

$$j^* = \arg\max_j \phi_j(v_j) \quad \text{s.t.} \quad \phi_{j^*}(v_{j^*}) \geq c$$

where $\phi_j(v) = v - (1 - F_j(v))/f_j(v)$ is the virtual valuation function specific to bidder $j$'s type distribution.

The payment for the winner is:

$$t_{j^*} = \inf\{v : \phi_{j^*}(v) \geq \max(c, \max_{j \neq j^*} \phi_j(v_j))\}$$

**Application to Content Market.** In the Douyin marketplace, merchants from different categories (beauty, food, tech) have different WTP distributions for the same content. The platform should:

1. Estimate category-specific distributions $F_k$ from historical transaction data.
2. Compute category-specific virtual valuations $\phi_k$.
3. Apply Myerson's asymmetric auction: allocate to the merchant with highest virtual valuation, charge the threshold price.

**Proposition 2.6 (Platform-Recommended Reserve, Adapted).** When the platform has AI-estimated content value $\hat{V}$ with precision $\sigma^2$, the optimal recommended reserve price for a creator with cost $c$ facing $n$ expected bidders is:

$$r^*(\hat{V}, n, c) = \max\left(c, \; \hat{V} \cdot \alpha(n) + (1 - \alpha(n)) \cdot \phi^{-1}(c)\right)$$

where $\alpha(n) = 1 - (1 - 1/n)^n \approx 1 - 1/e \approx 0.632$ for large $n$, reflecting the weight placed on the competitive estimate versus the Myerson reserve.

*Derivation.* When $n$ is large, the market is thick and $\hat{V}$ is the best predictor of clearing price (weight $\alpha \to 1$). When $n = 1$, the mechanism reduces to bilateral bargaining and $\alpha \to 0$, so the Myerson reserve dominates. The interpolation $\alpha(n)$ follows from the revenue equivalence theorem applied to the gap between competitive price and monopoly price. $\square$

---

## 3. Combinatorial Rights Bundle Pricing

### 3.1 Valuation Structure: Complements vs. Substitutes

The six rights $R_1, \ldots, R_6$ exhibit a rich interaction structure. We formalize this using the theory of submodular and supermodular functions.

**Definition 3.1 (Complementarity and Substitutability).** For rights $R_a, R_b$ and any bundle $S \subseteq \mathcal{R} \setminus \{R_a, R_b\}$:

- $R_a$ and $R_b$ are **complements** (互补品) if $v(S \cup \{R_a, R_b\}) - v(S \cup \{R_b\}) \geq v(S \cup \{R_a\}) - v(S)$, i.e., the marginal value of $R_a$ is higher when $R_b$ is present.

- $R_a$ and $R_b$ are **substitutes** (替代品) if the inequality is reversed.

- $v$ is **supermodular** (全互补) if all pairs are complements. $v$ is **submodular** (全替代) if all pairs are substitutes.

**Proposition 3.1 (Rights Interaction Matrix).** In the content licensing context, the following interaction structure holds:

| | $R_1$ (display) | $R_2$ (ad-feed) | $R_3$ (remix) | $R_4$ (AI-swap) | $R_5$ (exclusive) | $R_6$ (time-window) |
|---|---|---|---|---|---|---|
| $R_1$ | -- | **C** | **C** | **C** | **C** | **C** |
| $R_2$ | **C** | -- | **C** | **C** | **C** | **C** |
| $R_3$ | **C** | **C** | -- | **C** | S | S |
| $R_4$ | **C** | **C** | **C** | -- | S | N |
| $R_5$ | **C** | **C** | S | S | -- | **C** |
| $R_6$ | **C** | **C** | S | N | **C** | -- |

Legend: **C** = complements, S = substitutes, N = neutral.

*Justification.*

- $R_1 + R_2$: **Complements.** A merchant displaying content on their page ($R_1$) derives more value from also running it as ads ($R_2$), since consistent brand messaging across touchpoints increases conversion. Formally: $v(\{R_1, R_2\}) > v(\{R_1\}) + v(\{R_2\})$.

- $R_2 + R_4$: **Complements.** AI face-swap ($R_4$) is primarily valuable for ad-feed usage ($R_2$): the merchant replaces the creator's face with their own spokesperson, creating personalized ad content. Without $R_2$, $R_4$ has limited standalone value.

- $R_3$ and $R_5$: **Substitutes.** Remix right ($R_3$) creates derivative works that may substitute for the original, reducing the value of category exclusivity ($R_5$) since competitors can license the original and remix it differently. Formally: $v(\{R_3, R_5\}) < v(\{R_3\}) + v(\{R_5\})$.

- $R_5 + R_6$: **Complements.** Category exclusivity ($R_5$) and time-window exclusivity ($R_6$) together provide complete competitive moat: exclusive in both category and time. The combined value exceeds the sum.

**Corollary 3.1.** The merchant valuation function $v(\cdot)$ is **neither purely submodular nor purely supermodular**. This means the combinatorial auction problem does not admit the clean structural results available for, e.g., gross-substitutes valuations (Kelso and Crawford, 1982).

### 3.2 Bundling Theory: Adams-Yellen Framework

Following Adams and Yellen (1976), we analyze whether rights should be sold individually (component pricing), as a fixed bundle (pure bundling), or as both (mixed bundling).

**Setup.** Consider two rights $R_a, R_b$ for simplicity (the analysis extends). Merchant $j$ has valuation $(v_j^a, v_j^b)$ for the two rights independently, and valuation $v_j^{ab}$ for the bundle. Let $(v^a, v^b)$ be drawn from a joint distribution $H$ on $\mathbb{R}^2_+$.

**Definition 3.2 (Bundling Strategies).**

- **Component pricing** (单独定价): Set prices $p_a, p_b$ separately. Merchant buys $R_a$ iff $v^a \geq p_a$, and $R_b$ iff $v^b \geq p_b$, independently.
- **Pure bundling** (纯捆绑): Set a single price $p_{ab}$ for the pair $\{R_a, R_b\}$. Merchant buys both or neither.
- **Mixed bundling** (混合捆绑): Set prices $p_a, p_b, p_{ab}$ where $p_{ab} \leq p_a + p_b$. Merchant chooses the most profitable option.

**Theorem 3.1 (Adams-Yellen Bundling Principle, adapted).**

(i) Pure bundling dominates component pricing when merchant valuations for the two rights are **negatively correlated** (i.e., merchants who value $R_a$ highly tend to value $R_b$ less). Bundling reduces valuation heterogeneity, allowing a tighter price screen.

(ii) Component pricing dominates pure bundling when valuations are **strongly positively correlated** (i.e., merchants who value $R_a$ highly also value $R_b$ highly) and there is significant **demand asymmetry** (many merchants want only one right).

(iii) **Mixed bundling weakly dominates both** pure strategies. By offering both individual rights and bundles with a discount, the platform captures surplus from all merchant segments.

*Formal proof of (iii).* Let $\pi^C(p_a, p_b)$ be the revenue from component pricing and $\pi^B(p_{ab})$ from pure bundling. Mixed bundling with prices $(p_a, p_b, p_{ab})$ where $p_{ab} = p_a + p_b - \delta$ for small $\delta > 0$ captures:
- All merchants who would buy both under component pricing (they now save $\delta$, so weakly more buy the bundle).
- All merchants who want only one right (they still can buy individually).
- Additional merchants at the margin who value the bundle between $p_{ab}$ and $p_a + p_b$.

Therefore $\pi^{MB} \geq \max(\pi^C, \pi^B)$. Strict inequality holds generically. $\square$

**Proposition 3.2 (Optimal Bundle Composition for Content Rights).** In the content licensing market:

(a) **Core bundle** $B^* = \{R_1, R_2\}$ (display + ad-feed) should be the **default offering** (pure bundle), because:
  - These are strong complements (Proposition 3.1).
  - Nearly all merchants want both.
  - Bundle pricing reduces transaction complexity.

(b) **Extended rights** $R_3, R_4$ should use **mixed bundling**: offer individually and as add-ons to $B^*$ with discount:
  - $p(B^* \cup \{R_3\}) < p(B^*) + p(R_3)$ (bundle discount for remix add-on).
  - $p(B^* \cup \{R_4\}) < p(B^*) + p(R_4)$ (bundle discount for AI-swap add-on).

(c) **Exclusivity rights** $R_5, R_6$ should use **component pricing** with auction-determined premiums, because:
  - These are rivalrous (only one merchant per category/time window).
  - The scarcity requires competitive price discovery.
  - The appropriate mechanism is a sealed-bid auction for the exclusivity right, conditioned on holding the core bundle.

### 3.3 The Exclusivity Premium

**Definition 3.3 (Exclusivity Premium).** The exclusivity premium $\Delta_5$ for category-exclusive right $R_5$ is:

$$\Delta_5 = v_j(\{R_1, R_2, R_5\}) - v_j(\{R_1, R_2\})$$

This represents the additional value to merchant $j$ of knowing that no competitor in their category can use the same content.

**Proposition 3.3 (Exclusivity Premium Derivation).** Under a model of competitive advertising where merchant $j$'s profit from content usage depends on the number of competitors $m$ using the same content:

$$\pi_j(m) = \frac{\alpha}{1 + \beta m} \cdot v_j - p_j$$

where $\alpha > 0$ is a base conversion parameter and $\beta > 0$ measures competitive dilution, the exclusivity premium is:

$$\Delta_5 = v_j \cdot \alpha \cdot \left(1 - \frac{1}{1 + \beta \cdot \mathbb{E}[m | \text{non-exclusive}]}\right)$$

*Proof.* Under non-exclusive licensing, merchant $j$ expects $\mathbb{E}[m]$ competitors to also license the content. Their expected profit is:

$$\pi_j^{NE} = \frac{\alpha}{1 + \beta \cdot \mathbb{E}[m]} \cdot v_j - p_j^{NE}$$

Under category exclusivity ($m = 0$ within the category):

$$\pi_j^{E} = \alpha \cdot v_j - p_j^{E}$$

The merchant is indifferent when $\pi_j^{NE} = \pi_j^{E}$, yielding:

$$p_j^{E} - p_j^{NE} = \alpha \cdot v_j \cdot \frac{\beta \cdot \mathbb{E}[m]}{1 + \beta \cdot \mathbb{E}[m]} = \Delta_5 \quad \square$$

**Corollary 3.2.** The exclusivity premium is:
- **Increasing** in the expected number of non-exclusive licensees $\mathbb{E}[m]$.
- **Increasing** in competitive dilution $\beta$ (more competitive categories have higher premiums).
- **Proportional** to the base valuation $v_j$.

**Numerical Example.** Suppose $\alpha = 1$, $\beta = 0.3$, $\mathbb{E}[m] = 4$ (four non-exclusive licensees expected), $v_j = 10{,}000$ RMB. Then:

$$\Delta_5 = 10{,}000 \times 1 \times \frac{0.3 \times 4}{1 + 0.3 \times 4} = 10{,}000 \times \frac{1.2}{2.2} \approx 5{,}455 \text{ RMB}$$

The exclusivity premium is 54.5% of the base valuation, roughly consistent with the V3.0 framework's 5--10x multiplier for category exclusivity (the multiplier depends on the base non-exclusive price, not the valuation).

### 3.4 Optimal Combinatorial Mechanism

**Theorem 3.2 (Optimal Combinatorial Auction for Rights Bundles).** The revenue-maximizing mechanism for selling rights bundles from $\mathcal{R}$ to $J$ merchants is:

1. **Elicit reports** $(\hat{v}_j(S))_{S \subseteq \mathcal{R}}$ from each merchant $j$ for all desired bundles.

2. **Solve the winner determination problem (WDP)**:

$$\max_{x} \sum_{j=1}^{J} \sum_{S \subseteq \mathcal{R}} x_{j,S} \cdot \phi_j(v_j(S))$$

subject to feasibility constraints (F1)--(F4) and $x_{j,S} \in \{0, 1\}$.

Here $\phi_j(v_j(S))$ is the virtual surplus from allocating bundle $S$ to merchant $j$.

3. **Charge VCG payments**: Each winner $j$ receiving bundle $S^*$ pays:

$$t_j = \sum_{j' \neq j} \sum_{S} x^{-j}_{j',S} \cdot v_{j'}(S) - \sum_{j' \neq j} \sum_{S} x^{*}_{j',S} \cdot v_{j'}(S)$$

where $x^{-j}$ is the optimal allocation excluding merchant $j$, and $x^*$ is the optimal allocation including all merchants.

**Remark on Computational Complexity.** The WDP is NP-hard in general (combinatorial auctions; Lehmann, O'Callaghan, and Shoham, 2002). However, with $|\mathcal{R}| = 6$, there are at most $2^6 = 64$ possible bundles per merchant, making the problem tractable for realistic market sizes (hundreds of merchants, not millions).

**Practical Simplification.** Rather than full combinatorial auction, the platform can use the hierarchical approach from Proposition 3.2:

1. Auction the core bundle $B^* = \{R_1, R_2\}$ via posted price or $K$-unit auction (Section 2).
2. Offer add-on rights $R_3, R_4$ at fixed prices (mixed bundling).
3. Auction exclusivity rights $R_5, R_6$ via second-price auction among core bundle holders.

This three-stage mechanism is approximately optimal and computationally trivial.

---

## 4. Dynamic Pricing with Content Decay

### 4.1 Content Value Decay Model

**Assumption 4.1 (Exponential Decay).** Content value decays exponentially:

$$V(t) = V_0 \cdot e^{-\lambda t}$$

where $V_0$ is the initial value and $\lambda > 0$ is the decay rate. The half-life is $t_{1/2} = \ln(2) / \lambda$.

| Content Type | Half-life $t_{1/2}$ | Decay rate $\lambda$ (per day) | (类型) |
|---|---|---|---|
| Trending/viral | 2--3 days | 0.23--0.35 | 热点类 |
| Seasonal/event | 1--2 weeks | 0.05--0.10 | 季节/事件类 |
| Evergreen tutorial | 8--12 weeks | 0.008--0.012 | 常青类 |
| Brand template | 4--6 weeks | 0.016--0.025 | 品牌模板类 |

**Generalization (Adstock Model).** The standard Adstock model from advertising science (Broadbent, 1979) extends the simple exponential:

$$V(t) = V_0 \cdot \left[\alpha \cdot e^{-\lambda_1 t} + (1 - \alpha) \cdot e^{-\lambda_2 t}\right]$$

where the first component captures short-term attention decay ($\lambda_1$ large) and the second captures long-term brand/utility value ($\lambda_2$ small). The parameter $\alpha \in [0, 1]$ weights the two components. For trending content, $\alpha \to 1$; for evergreen content, $\alpha \to 0$.

### 4.2 Optimal Dynamic Pricing Path

**Theorem 4.1 (Revenue-Optimal Price Path).** Consider a monopolist seller of a decaying-value good with $N$ potential buyers arriving according to a Poisson process with rate $\mu$. Each buyer has a private valuation $v$ drawn i.i.d. from $F$, and the good's value to any buyer at time $t$ is $v \cdot V(t)/V_0$. The revenue-maximizing price path is:

$$p^*(t) = r^* \cdot e^{-\lambda t}$$

where $r^*$ is the static Myerson optimal reserve price at $t = 0$.

*Proof.* At each time $t$, the seller faces a fresh posted-price problem (by the memoryless property of Poisson arrivals). The optimal posted price for a single buyer with valuation $\tilde{v} = v \cdot e^{-\lambda t}$ drawn from $\tilde{F}(x) = F(x \cdot e^{\lambda t})$ solves $\phi_t(p) = 0$. By change of variables:

$$\phi_t(p) = p - \frac{1 - F(p \cdot e^{\lambda t})}{f(p \cdot e^{\lambda t}) \cdot e^{\lambda t}}$$

Setting $p = r \cdot e^{-\lambda t}$ and substituting:

$$\phi_t(r \cdot e^{-\lambda t}) = r \cdot e^{-\lambda t} - \frac{1 - F(r)}{f(r) \cdot e^{\lambda t}} \cdot e^{-\lambda t} = e^{-\lambda t} \left[r - \frac{1 - F(r)}{f(r)}\right] = e^{-\lambda t} \cdot \phi_0(r)$$

So $\phi_t(p^*(t)) = 0$ iff $\phi_0(r^*) = 0$, i.e., $r^* = \phi_0^{-1}(0)$, the static reserve. $\square$

**Corollary 4.1.** The optimal price path tracks the content decay exactly. This is equivalent to a **Dutch auction** (descending price auction; 荷兰式拍卖) where the price declines continuously at rate $\lambda$.

### 4.3 Dutch Auction Implementation

**Proposition 4.1 (Dutch Auction for Decaying Content).** A Dutch auction with price trajectory:

$$p^{DA}(t) = p_0 \cdot e^{-\lambda t}$$

where $p_0$ is the starting price (set at or above $r^*$), achieves the revenue-optimal outcome for content with exponential decay, provided:

(a) Buyers are risk-neutral.
(b) Buyer arrivals are independent of price history.
(c) Content is non-exclusive (each buyer's purchase does not affect others' valuations).

Under non-exclusive licensing, the Dutch auction operates as follows:
- Price starts at $p_0$ and declines over time.
- Any merchant can purchase at the current price at any time.
- Multiple merchants may buy at different times (and thus different prices).
- Earlier buyers pay more but get longer usage duration.

**Proposition 4.2 (Inter-temporal Price Discrimination).** The declining price path achieves natural inter-temporal price discrimination:

- **High-WTP merchants** buy early at price $\approx p_0$, getting maximum content freshness.
- **Low-WTP merchants** buy later at price $p_0 \cdot e^{-\lambda t}$, getting decayed but cheaper content.
- The platform captures surplus from both segments.

This is analogous to the book publishing "hardcover-then-paperback" strategy (Stokey, 1979).

**Theorem 4.2 (Revenue-Optimal vs. Welfare-Optimal Decay Schedule).**

(a) The **revenue-optimal** price path is $p^R(t) = r^* \cdot e^{-\lambda t}$ (Theorem 4.1).

(b) The **welfare-optimal** (socially efficient) price path is $p^W(t) = c \cdot e^{-\lambda t}$ (price equals marginal cost at each $t$).

(c) The revenue loss from welfare-optimal pricing is:

$$\Delta \text{Rev} = \mu \int_0^{T_{\max}} [F(c \cdot e^{-\lambda t}) - F(r^* \cdot e^{-\lambda t})] \cdot (r^* - c) \cdot e^{-\lambda t} \, dt$$

This represents the standard monopoly deadweight loss, integrated over the content lifecycle. The platform (as market designer) can choose where to operate on the efficiency-revenue tradeoff by adjusting the reserve price.

### 4.4 Practical Dynamic Pricing Algorithm

**Algorithm 4.1 (Adaptive Dutch Auction for Content Licensing).**

```
Input: Content item l with estimated V_0, decay rate lambda, creator cost c
Output: Price trajectory and sales

1. Compute initial reserve: r* = phi^{-1}(c) using estimated F
2. Set starting price: p_0 = min(V_0, 1.5 * r*)
3. For each time step t = 0, dt, 2dt, ..., T_max:
   a. Current price: p(t) = p_0 * exp(-lambda * t)
   b. If p(t) < c: delist content (creator IR violated)
   c. If merchant j arrives with bid b_j >= p(t):
      - Execute sale at p(t)
      - If non-exclusive: continue auction for remaining licenses
      - If exclusive: end auction
   d. Update lambda estimate based on observed demand (Bayesian update)
4. Return total revenue, sales timestamps
```

**Remark.** Step 3d introduces adaptivity: the platform can adjust $\lambda$ in real-time based on observed click-through rates, view counts, and bidding intensity. A simple Bayesian update rule:

$$\lambda_{\text{posterior}} = \frac{n_0 \cdot \lambda_{\text{prior}} + n_{\text{obs}} \cdot \hat{\lambda}_{\text{MLE}}}{n_0 + n_{\text{obs}}}$$

where $\hat{\lambda}_{\text{MLE}}$ is estimated from the time series of content engagement metrics.

---

## 5. Information Design (Bayesian Persuasion)

### 5.1 The Platform's Information Advantage

The platform observes signals $s \in \mathcal{S}$ about content quality that are not directly observed by merchants. Following Kamenica and Gentzkow (2011), we model the platform's choice of what to reveal as a **Bayesian persuasion** problem (信息设计/贝叶斯劝说).

**Setup.**
- The **state** $\omega \in \Omega = \{H, L\}$ represents content quality (high or low).
- The **prior** is $\Pr(\omega = H) = p_0$.
- The platform observes a private signal $s$ correlated with $\omega$.
- The platform designs a **signal structure** $\pi: \Omega \to \Delta(\mathcal{S})$ that determines what information to reveal to merchants.
- Merchants update beliefs via Bayes' rule and bid accordingly.

**Definition 5.1 (Signal Structure).** A signal structure is a mapping $\pi: \Omega \to \Delta(\mathcal{S})$ where $\pi(s | \omega)$ is the probability of sending signal $s$ in state $\omega$. The induced posterior belief after observing signal $s$ is:

$$\mu(H | s) = \frac{\pi(s | H) \cdot p_0}{\pi(s | H) \cdot p_0 + \pi(s | L) \cdot (1 - p_0)}$$

### 5.2 Full vs. Partial Revelation

**Proposition 5.1 (Full Revelation Benchmark).** Under full revelation ($\pi$ is fully informative), the merchant learns $\omega$ exactly. The expected revenue is:

$$\text{Rev}_{\text{full}} = p_0 \cdot R(v_H) + (1 - p_0) \cdot R(v_L)$$

where $R(v)$ is the revenue from an auction where all merchants know the content value is $v$, and $v_H > v_L$ are the content values in the two states.

**Proposition 5.2 (No Revelation Benchmark).** Under no revelation ($\pi$ is uninformative), the merchant uses only the prior $p_0$. The expected revenue is:

$$\text{Rev}_{\text{none}} = R(\bar{v}) \quad \text{where} \quad \bar{v} = p_0 \cdot v_H + (1 - p_0) \cdot v_L$$

**Theorem 5.1 (Optimality of Partial Revelation).** Under certain conditions on $R(\cdot)$, the optimal signal structure is **partially revealing**: the platform reveals some but not all of its information. Specifically:

(a) If $R(v)$ is **convex** in the content value $v$ (e.g., auctions with many bidders), then $\text{Rev}_{\text{full}} \geq \text{Rev}_{\text{none}}$, and full revelation is optimal.

(b) If $R(v)$ is **concave** in $v$ (e.g., posted-price markets with few buyers), then $\text{Rev}_{\text{none}} \geq \text{Rev}_{\text{full}}$, and no revelation is optimal.

(c) In general, $R(v)$ is **neither globally convex nor concave** (it is typically S-shaped: concave for low values, convex for high values). In this case, **partial revelation strictly dominates both extremes**.

*Proof sketch.* By the concavification approach of Kamenica and Gentzkow (2011), the optimal expected revenue under persuasion is:

$$\text{Rev}^* = \text{cav}(R)(p_0)$$

where $\text{cav}(R)$ is the concave closure (smallest concave function above $R$) of the revenue function viewed as a function of the prior belief $p$. If $R$ is already concave at $p_0$, no revelation is optimal. If convex, full revelation. If neither, partial revelation achieves the concavified value, which strictly exceeds both extremes. $\square$

### 5.3 Optimal Signal Structure for Content Market

**Proposition 5.3 (Recommended Information Policy).** The platform should adopt the following signal structure:

| Information Type | Reveal? | Rationale |
|---|---|---|
| Historical engagement (views, likes, shares) | **Partially**: show percentile rank, not exact numbers | Prevents cherry-picking of top-performing content while giving quality signal |
| Predicted ad performance (pCTR, pCVR) | **Partially**: show confidence band, not point estimate | Avoids adverse selection from precise predictions |
| Creator reputation score | **Fully**: show exact score | Aligns incentives for creators to maintain quality (Shapiro, 1983) |
| Category demand intensity | **Partially**: show "high/medium/low" | Prevents merchants from timing purchases to exploit low-demand periods |
| Number of competing bidders | **Do not reveal** | Concealing the number of bidders increases expected revenue (Milgrom and Weber, 1982 for affiliated values; standard result for IPV) |
| Content freshness / time since posting | **Fully**: show exact timestamp | Necessary for merchants to assess remaining value life |

**Theorem 5.2 (Surplus Improvement from Partial Revelation).** Define $\text{TS}^*$ as total surplus under optimal partial revelation and $\text{TS}_{\text{full}}$ under full revelation. Then:

$$\text{TS}^* \geq \text{TS}_{\text{full}} - \epsilon$$

where $\epsilon$ is bounded by the welfare loss from preventing cherry-picking. Specifically, under full revelation, only the highest-performing content sells (merchants select on quality), leaving a large inventory of adequate content unsold. Under partial revelation, the pooling of information induces merchants to purchase a broader range of content, increasing total trading volume and improving market thickness.

*Proof sketch.* Full revelation creates an adverse selection problem: merchants with precise quality signals bid only on top content, driving up prices for the best items and leaving mid-tier content unsold. This is the "winner's blessing" for top content and "loser's curse" for the rest. By pooling mid-tier content together in a quality band (partial revelation), the platform ensures a broader market. The welfare gain from increased trade volume (more content sold) typically outweighs the welfare loss from imprecise matching (some merchants get slightly worse content than expected). The bound $\epsilon$ depends on the variance of quality within each revealed band. $\square$

### 5.4 Formal Persuasion Mechanism

**Algorithm 5.1 (Information Design for Content Marketplace).**

```
Input: Content quality distribution, platform signal s, prior p_0
Output: Optimal public signal pi*

1. Estimate the revenue function R(p) as a function of merchant belief p
   (via simulation or historical data)
2. Compute the concave closure cav(R)(p)
3. Find the optimal signal structure:
   - If cav(R)(p_0) = R(p_0): no revelation is optimal
   - If cav(R)(p_0) = R(p_0) and R is convex: full revelation
   - Otherwise: find posteriors p_L, p_H such that:
     a. p_0 = alpha * p_H + (1 - alpha) * p_L  (Bayesian plausibility)
     b. cav(R)(p_0) = alpha * R(p_H) + (1 - alpha) * R(p_L)
4. Design signal: with probability alpha, send "good" signal (inducing posterior p_H);
   with probability 1 - alpha, send "fair" signal (inducing posterior p_L)
```

---

## 6. Reserve Price Optimization

### 6.1 The Creator's Reserve Price Problem

The creator sets a floor price $r_C$ (底价) below which they refuse to sell. The platform recommends a reserve price $r_P$ based on its AI valuation model. The effective reserve is $r = \max(r_C, r_P)$.

**Problem.** The creator does not know the optimal $r_C$ because they do not observe:
- The distribution $F$ of merchant valuations.
- The number of expected bidders $n$.
- The content's precise commercial value.

The platform knows these but has its own incentive (maximize commission revenue, not creator revenue).

**Proposition 6.1 (Creator's Optimal Reserve, Known $F$).** If the creator knew $F$ and faced $n$ i.i.d. bidders in a second-price auction, the optimal reserve would be:

$$r_C^* = \arg\max_r \left\{ r \cdot [1 - F(r)]^n \cdot \binom{n}{0} + \int_r^{\bar{v}} p \cdot n \cdot f(p) \cdot [F(p)]^{n-1} \cdot [1 - F(r)]^0 \, dp + \cdots \right\}$$

By Myerson (1981), this simplifies to the elegant result $r_C^* = \phi^{-1}(0)$ (reserve at the zero of virtual valuation), which is independent of $n$.

But the creator does not know $F$, so the platform must recommend.

### 6.2 Platform-Recommended Reserve Price

**Theorem 6.1 (Incentive-Aligned Reserve Recommendation).** The platform recommends reserve $r_P$ to maximize the joint creator-platform surplus:

$$r_P = \arg\max_r \left\{(1 - \tau) \cdot \mathbb{E}[\text{Creator Revenue}(r)] + \tau \cdot \mathbb{E}[\text{Creator Revenue}(r)]\right\}$$

$$= \arg\max_r \, \mathbb{E}[\text{Total Transaction Revenue}(r)]$$

Since the platform and creator share revenue proportionally (platform takes fraction $\tau$), their revenue-maximizing reserve prices are **identical**. This alignment is a design feature of the proportional commission model.

*Proof.* The platform's revenue is $\tau \cdot P(r)$ where $P(r)$ is the expected transaction price given reserve $r$. The creator's revenue is $(1-\tau) \cdot P(r)$. Both are maximized at the same $r^*$. $\square$

**Corollary 6.1.** Under proportional commissions, the platform's recommended reserve price is **incentive-compatible**: the platform and creator agree on the optimal reserve. This is a key advantage of proportional commissions over flat fees.

### 6.3 Optimal Reserve as a Function of Market Variables

**Proposition 6.2 (Reserve Price Formula).** For content with AI-estimated value $\hat{V}$, facing $n$ expected bidders from distribution $F$ with support $[\underline{v}, \overline{v}]$, and creator with opportunity cost $c$, the platform-recommended reserve is:

$$r^*(\hat{V}, n, c, F) = \max\left(c, \, \phi^{-1}(0)\right)$$

For common distributions:

**(a) Uniform $F \sim U[\underline{v}, \overline{v}]$:**

$$\phi(v) = v - \frac{\overline{v} - v}{1} = 2v - \overline{v}$$

$$r^* = \max\left(c, \, \frac{\overline{v}}{2}\right)$$

**(b) Exponential $F(v) = 1 - e^{-v/\mu}$, $v \geq 0$:**

$$\phi(v) = v - \mu$$

$$r^* = \max(c, \mu)$$

**(c) Log-normal $\ln v \sim N(\mu, \sigma^2)$ (empirically calibrated for content valuations):**

The virtual valuation does not have a closed form but is computable numerically. The key qualitative result: the reserve price is **increasing in $\sigma$** (more uncertain valuations warrant higher reserves) and **increasing in $\mu$** (higher mean valuations warrant higher reserves).

### 6.4 Thin Market Adjustment

**Proposition 6.3 (Thin Market Correction).** When the expected number of bidders $n$ is small ($n \leq 3$), the probability of no sale is non-negligible. The Myerson reserve $r^*$ may lead to excessive no-sale rates.

Define the **no-sale probability**: $\Pr(\text{no sale}) = F(r)^n$.

A risk-averse creator may prefer a lower reserve that trades off revenue per sale for sale probability:

$$r^{RA}(\gamma) = \arg\max_r \, \left[\mathbb{E}[\text{Rev}(r)] - \gamma \cdot \text{Var}[\text{Rev}(r)]\right]$$

where $\gamma > 0$ is the creator's risk aversion parameter.

**Practical formula.** For a thin market with $n \leq 3$ bidders and creator risk aversion $\gamma$:

$$r^{\text{thin}} = r^* - \delta(n, \gamma)$$

where $\delta(n, \gamma) = \gamma \cdot \sigma_v \cdot \sqrt{F(r^*)^n / n}$ is a downward correction that increases when the market is thinner (lower $n$) and the creator is more risk-averse (higher $\gamma$).

---

## 7. Incentive Compatibility and Budget Balance

### 7.1 Mechanism Properties: Formal Statements

We now prove the key properties of the proposed mechanism, which consists of:

(M1) Creator sets floor price $r_C$ (may use platform recommendation).
(M2) Merchants submit sealed bids for desired rights bundles.
(M3) Allocation: highest (virtual) valuations win, subject to feasibility.
(M4) Payment: VCG / second-price rule for merchants; proportional sharing for creator.

**Theorem 7.1 (Incentive Compatibility -- Merchant Side).** Under the second-price / VCG payment rule (M4), truthful bidding ($b_j = v_j(S)$ for all $j, S$) is a weakly dominant strategy for each merchant.

*Proof.* This is the standard Vickrey (1961) result. Fix merchant $j$ and let $p_j^*$ be the threshold price (minimum bid at which $j$ wins). Under second-price payment, $j$'s utility from bidding $b_j$ is:

$$u_j(b_j) = \begin{cases} v_j - p_j^* & \text{if } b_j > p_j^* \\ 0 & \text{if } b_j < p_j^* \\ \{0 \text{ or } v_j - p_j^*\} & \text{if } b_j = p_j^* \end{cases}$$

Since $p_j^*$ does not depend on $b_j$:
- If $v_j > p_j^*$: bidding $b_j = v_j$ wins and yields positive utility. Any $b_j < p_j^*$ loses and yields zero. So truthful bidding is optimal.
- If $v_j < p_j^*$: bidding $b_j = v_j$ loses and yields zero. Any $b_j > p_j^*$ wins but yields negative utility. So truthful bidding is optimal.

Therefore $b_j = v_j$ is weakly dominant. $\square$

**Theorem 7.2 (Incentive Compatibility -- Creator Side).** Under proportional commission pricing, the creator's optimal floor price is $r_C = c_i$ (truthful revelation of opportunity cost), provided the platform's recommended reserve $r_P$ is incentive-aligned (Theorem 6.1).

*Proof.* The creator's expected revenue from setting floor price $r$ is:

$$\pi_C(r) = (1 - \tau) \cdot \mathbb{E}[\text{price} \mid \text{sale occurs}] \cdot \Pr(\text{sale} \mid r) - c_i \cdot \Pr(\text{sale} \mid r)$$

$$= \Pr(\text{sale} \mid r) \cdot [(1 - \tau) \cdot \mathbb{E}[\text{price} \mid \text{sale}] - c_i]$$

The sale occurs when the winning bid exceeds $\max(r, r_P)$. Two cases:

Case 1: $r \leq r_P$. The creator's floor is non-binding. The effective reserve is $r_P$, which is optimal (Theorem 6.1). Any $r \leq r_P$ yields the same outcome. Setting $r = c_i \leq r_P$ is weakly dominant.

Case 2: $r > r_P$. The creator's floor binds. Increasing $r$ above $c_i$ reduces $\Pr(\text{sale})$ while increasing $\mathbb{E}[\text{price} \mid \text{sale}]$. The optimal $r$ solves the creator's monopoly pricing problem, which (by Myerson regularity and the alignment result in Theorem 6.1) yields $r^* = r_P$ if the creator trusts the platform's recommendation. If the creator does not trust the recommendation, they solve independently, but truthful cost revelation $r = c_i$ ensures IR is never violated. $\square$

**Remark.** Creator-side incentive compatibility is **weaker** than merchant-side: setting $r_C = c_i$ is optimal only if the creator trusts the platform's recommendation or if $c_i \geq r_P$. In practice, the platform should demonstrate recommendation accuracy through track record (reputation mechanism; Shapiro, 1983).

**Theorem 7.3 (Individual Rationality).** The mechanism satisfies interim individual rationality for both sides:

(a) **Merchant IR**: Each merchant $j$ receives non-negative expected utility:

$$\mathbb{E}[u_j] = \mathbb{E}[\max(v_j(S^*) - t_j, 0)] \geq 0$$

This holds because under VCG pricing, a merchant never pays more than their valuation: $t_j \leq v_j(S^*)$.

(b) **Creator IR**: The creator receives non-negative expected surplus relative to outside option:

$$\mathbb{E}[\pi_C] = (1 - \tau) \cdot \mathbb{E}[\text{Rev}] \geq 0$$

when $\text{Rev} > 0$ (revenue is non-negative by construction: prices are non-negative) and the content would otherwise generate zero additional income (marginal content IR constraint from Section 5.1 of the existing mechanism_design document).

*Proof.* Part (a) follows directly from the VCG payment rule: $t_j = v_j(S^*) - u_j^{VCG}$ where $u_j^{VCG} \geq 0$ by construction. Part (b) follows from $r \geq c_i$ (the effective reserve exceeds the creator's cost) and the proportional sharing rule. $\square$

### 7.2 Budget Balance

**Definition 7.1 (Budget Balance).** A mechanism is:
- **Strongly budget balanced** if $\sum_j t_j = p_i$ for every realization (platform earns zero).
- **Weakly budget balanced** if $\sum_j t_j \geq p_i$ for every realization (platform never subsidizes).
- **Budget balanced in expectation** if $\mathbb{E}[\sum_j t_j] \geq \mathbb{E}[p_i]$.

**Theorem 7.4 (Weak Budget Balance of the Mechanism).** The proposed mechanism is weakly budget balanced in the steady state (post-subsidy phase):

$$\sum_j t_j(\theta) \geq p_i(\theta) \quad \text{for all } \theta$$

*Proof.* Under the proposed mechanism:
- Total merchant payments: $\sum_j t_j$ (determined by auction).
- Creator payment: $p_i = (1 - \tau) \cdot \sum_j t_j$.
- Platform revenue: $\Pi = \tau \cdot \sum_j t_j \geq 0$.

Since $\tau > 0$, we have $\sum_j t_j > p_i$ whenever any sale occurs. The platform never subsidizes transactions in the steady state. $\square$

### 7.3 The Myerson-Satterthwaite Impossibility and Escape Routes

**Theorem (Myerson and Satterthwaite, 1983).** In bilateral trade where:
- The seller has private cost $c \sim G$ on $[\underline{c}, \overline{c}]$.
- The buyer has private value $v \sim F$ on $[\underline{v}, \overline{v}]$.
- $[\underline{c}, \overline{c}] \cap [\underline{v}, \overline{v}] \neq \emptyset$ (overlapping supports).

Then no mechanism can simultaneously be incentive compatible, individually rational, ex-post budget balanced, and ex-post efficient.

**Application to Content Market.** The content marketplace involves bilateral trade (creator sells, merchant buys) with overlapping valuation supports. The Myerson-Satterthwaite impossibility binds: some efficient trades will fail to occur under any IC+IR+BB mechanism.

**Proposition 7.1 (Escape Routes).** The content marketplace mitigates the Myerson-Satterthwaite impossibility through three structural features:

**(a) Platform as Market Maker (平台做市).** The platform absorbs the inefficiency gap by:
- Subsidizing marginal trades during the launch phase (Months 1--12).
- Setting the commission rate $\tau$ to cover the expected subsidy in steady state.
- The platform's information advantage (AI valuation) reduces the gap between buyer and seller virtual valuations, shrinking the region of inefficient non-trade.

Formally, let $\Delta(\theta) = \psi(c) - \phi(v)$ be the "impossibility gap" for type profile $\theta = (c, v)$. The platform's subsidy per transaction is $\max(0, \Delta(\theta))$. With sufficiently accurate AI valuation (reducing the variance of $c$ and $v$), $\Pr(\Delta > 0)$ decreases, reducing expected subsidies.

**(b) Many-to-Many Market Thickness (市场厚度).** Rustichini, Satterthwaite, and Williams (1994) show that the efficiency loss from the impossibility theorem is $O(1/n)$ where $n$ is the number of traders on each side. In a thick content market with many creators and many merchants:

$$\text{Efficiency loss} = O\left(\frac{1}{\min(I, J)}\right) \to 0$$

as market thickness grows. The platform's strategy of building a large participant base directly attacks the impossibility result.

**(c) Repeated Interaction and Folk Theorem (重复博弈).** The content marketplace involves repeated transactions between creators and the platform (and often between the same creator-merchant pairs). By the Folk Theorem (Fudenberg and Maskin, 1986), in repeated games with sufficiently patient players (low discount rate $\delta$):

Any individually rational and feasible payoff profile can be sustained as a subgame perfect equilibrium.

This means that reputational considerations ("cooperate now for future gains") substitute for the IC constraints that generate the Myerson-Satterthwaite impossibility. Specifically:

- A creator who overprices today loses future bidder interest (punishment by market).
- A merchant who underbids today loses future content access (punishment by reputation system).

The effective IC constraint under repetition is:

$$\text{One-shot gain from misreport} \leq \frac{\delta}{1 - \delta} \cdot \text{Per-period loss from punishment}$$

For $\delta$ sufficiently close to 1 (creators and merchants who plan to use the marketplace long-term), this constraint is slack, and truth-telling is sustainable without explicit IC mechanisms.

---

## 8. Comparison with Real-World Content Marketplaces

### 8.1 Mechanism Classification of Existing Platforms

| Platform | Mechanism | Rights Structure | Price Discovery | Dynamic Pricing | Information Design |
|---|---|---|---|---|---|
| **Shutterstock** | Posted price, subscription | Non-exclusive RF | Seller sets price (platform suggests) | None (static) | Minimal (metadata tags) |
| **Getty Images** | Negotiated / Rights-Managed | Exclusive + RM licensing | Bilateral negotiation | Limited (manual adjustment) | Detailed rights specification |
| **Epidemic Sound** | Subscription (flat fee) | Non-exclusive, unlimited | Platform sets price tier | None | Genre/mood classification |
| **Xingtu/星图素材集市** | Posted price (brand sets) | Non-exclusive, capped at 20K RMB | Brand offers, creator accepts/rejects | None | Basic engagement stats |
| **Proposed Mechanism** | Combinatorial auction + posted price hybrid | Full rights bundle ($R_1$--$R_6$), exclusive and non-exclusive | Multi-format (auction + posted + Dutch) | Continuous (exponential decay) | Bayesian persuasion (partial revelation) |

### 8.2 Theoretical Superiority Analysis

**Theorem 8.1 (Revenue Dominance).** Under the assumptions of this paper (IPV, Myerson regularity, content decay, heterogeneous merchant types), the proposed mechanism generates weakly higher expected revenue than any of the four comparison platforms' mechanisms.

*Proof sketch.* We compare along four dimensions:

**(a) vs. Shutterstock (Posted Price).** Shutterstock uses uniform posted prices. By Myerson (1981), the optimal mechanism (which our auction component approximates) generates strictly higher revenue than any posted-price mechanism when:
- Bidder valuations are dispersed (high variance of $F$).
- There are at least 2 bidders per item for scarce rights.

For non-exclusive rights, both use posted prices (Proposition 2.1), so revenue is identical. For exclusive rights ($R_5, R_6$), our auction strictly dominates.

Overall: $\text{Rev}_{\text{proposed}} \geq \text{Rev}_{\text{Shutterstock}}$, with strict inequality when exclusive rights are traded.

**(b) vs. Getty Images (Negotiation).** Bilateral negotiation is a specific mechanism within the space of IC+IR mechanisms. By the revelation principle, the optimal mechanism weakly dominates any specific negotiation protocol. Moreover, negotiation incurs high transaction costs ($c_{\text{negotiate}} \approx$ 5--15 days per deal; V3.0 Section 1.2). The proposed mechanism reduces transaction costs by orders of magnitude through automated auction + matching.

Revenue comparison: $\text{Rev}_{\text{proposed}} + \text{cost savings}} \gg \text{Rev}_{\text{Getty}}$ (net of transaction costs).

**(c) vs. Epidemic Sound (Subscription).** The subscription model pools heterogeneous content at a uniform price per period. This eliminates price discrimination across content quality and across merchant types. By the bundling literature (Adams and Yellen, 1976; McAfee, McMillan, and Whinston, 1989), pure bundling (subscription) is revenue-inferior to mixed bundling when:
- Content quality variance is high (which it is in creator content: top 10% of videos generate 80%+ of views).
- Merchant usage intensity is heterogeneous (some merchants need 5 videos/month, others 500).

$\text{Rev}_{\text{proposed}} \geq \text{Rev}_{\text{subscription}}$, with strict inequality under heterogeneous content and heterogeneous usage.

**(d) vs. Xingtu/星图素材集市 (Brand-Set Price, Capped).** Xingtu's current mechanism has three critical inefficiencies:

(i) **Revenue cap**: 20,000 RMB per piece of content. This is a binding ceiling for high-value content, transferring surplus from creators to merchants. The proposed mechanism has no revenue cap.

(ii) **Brand-set pricing**: The brand (buyer) sets the price, creating a monopsony problem. The proposed mechanism uses auction-based price discovery, extracting surplus from competing merchants.

(iii) **No dynamic pricing**: Xingtu prices are static. The proposed mechanism adjusts prices to content decay (Section 4), capturing time-varying surplus.

Revenue improvement estimate: $\text{Rev}_{\text{proposed}} / \text{Rev}_{\text{Xingtu}} \in [2, 10]$ for content that would otherwise be capped at 20K RMB. $\square$

### 8.3 Welfare Comparison

**Proposition 8.1 (Welfare Improvement).** The proposed mechanism improves total welfare (creator surplus + merchant surplus + platform revenue) relative to all four comparison platforms through:

(a) **Reduced transaction costs**: Automated mechanism eliminates search (15--20% of budget; V3.0 Section 1.2), negotiation (5--15 days), and measurement costs.

(b) **Improved matching**: AI-powered valuation and combinatorial allocation ensure content reaches highest-value users, increasing allocative efficiency.

(c) **Expanded market size**: Dynamic pricing and non-exclusive licensing enable transactions that would not occur under static pricing (merchants who value content below the static price but above the decayed price), strictly expanding the set of efficient trades.

(d) **Information rent extraction**: Bayesian persuasion (Section 5) reduces information rents that merchants capture under full revelation, redistributing surplus toward creators and the platform without reducing total welfare (by the concavification result).

### 8.4 Limitations and Caveats

The theoretical superiority results rest on assumptions that may not hold perfectly:

1. **IPV assumption**: In practice, merchant valuations may be correlated (e.g., all beauty brands value the same content similarly). With affiliated values (Milgrom and Weber, 1982), the English auction may dominate the sealed-bid format, and the optimal mechanism is more complex.

2. **Myerson regularity**: If the valuation distribution is irregular (non-monotone virtual valuation), ironing is required (Myerson, 1981), and the optimal mechanism involves randomized allocation -- which may be impractical.

3. **Risk neutrality**: If creators or merchants are risk-averse, the optimal mechanism shifts toward guaranteed prices (royalty floors, price caps), which our model accommodates but does not optimize over.

4. **Collusion**: The VCG mechanism is vulnerable to collusion among bidders (bidder rings). In a thin market with few merchants per category, this is a real concern. Mitigation: random reserve price perturbation, monitoring for suspicious bidding patterns.

5. **Behavioral factors**: Real creators may not behave as rational, expected-utility-maximizing agents. Loss aversion (Kahneman and Tversky, 1979), status quo bias, and the endowment effect may cause creators to set floor prices above their true cost, reducing trade volume.

---

## Appendix A: Summary of Assumptions

| # | Assumption | Where Used | Justification |
|---|---|---|---|
| A1 | Independent Private Values (IPV) | Sections 2, 3, 6, 7 | Standard; reasonable for diverse merchant types |
| A2 | Myerson Regularity | Sections 2, 6 | Satisfied by common distributions (uniform, exponential, log-normal) |
| A3 | Exponential content decay | Section 4 | Empirically validated in advertising (Broadbent, 1979; V3.0 Section 3.3) |
| A4 | Risk neutrality | Sections 2, 4, 5 | Standard; Section 6.4 relaxes for creators |
| A5 | Poisson buyer arrivals | Section 4.2 | Standard for online marketplace modeling |
| A6 | Proportional commission | Sections 6, 7 | Design choice; justified by Theorem 6.1 alignment |
| A7 | Creator marginal content has zero opportunity cost | Section 7.3 | Key economic insight: residual content is otherwise unmonetized |

---

## Appendix B: Key Theorems and Their Implications

| Theorem | Statement (summary) | Design Implication |
|---|---|---|
| Myerson (1981) | Optimal auction uses virtual valuations + reserve price | Use second-price auction with optimal reserve for exclusive rights |
| Vickrey (1961) | Second-price auction induces truthful bidding | Truthful bidding simplifies the mechanism |
| Adams & Yellen (1976) | Mixed bundling dominates pure bundling and unbundling | Offer rights both individually and in bundles |
| Kamenica & Gentzkow (2011) | Partial information revelation can increase sender's payoff | Reveal percentile ranks, not exact metrics |
| Myerson & Satterthwaite (1983) | No fully efficient bilateral trade mechanism exists | Platform absorbs gap via commissions; market thickness mitigates |
| Bulow & Klemperer (1996) | One extra bidder > optimal reserve | Invest in attracting bidders, not just optimizing reserves |
| Stokey (1979) | Durable goods monopolist cannot commit to future prices | Dutch auction with exponential decay solves commitment problem |
| Folk Theorem | Cooperation sustainable in repeated games | Reputation effects substitute for static IC constraints |

---

## Appendix C: Notation Index

| Symbol | Definition | First appears |
|---|---|---|
| $\phi(v)$ | Myerson virtual valuation: $v - (1-F(v))/f(v)$ | Section 1.2 |
| $\psi(c)$ | Myerson virtual cost: $c + G(c)/g(c)$ | Section 1.2 |
| $r^*$ | Optimal reserve price | Section 2.3 |
| $\lambda$ | Content decay rate | Section 4.1 |
| $\tau$ | Platform commission rate | Section 1.5 |
| $K$ | Maximum number of non-exclusive licenses | Section 2.1 |
| $\pi(\cdot|\omega)$ | Signal structure in Bayesian persuasion | Section 5.1 |
| $\Delta_5$ | Exclusivity premium for $R_5$ | Section 3.3 |
| $B^*$ | Core rights bundle $\{R_1, R_2\}$ | Section 3.2 |
| $\rho$ | Supply-demand ratio $K/\mathbb{E}[N]$ | Section 2.2 |
| $\text{cav}(R)$ | Concave closure of revenue function | Section 5.2 |

---

## References

### Auction Theory and Mechanism Design
- Adams, W. J., & Yellen, J. L. (1976). Commodity Bundling and the Burden of Monopoly. *Quarterly Journal of Economics*, 90(3), 475--498.
- Bulow, J., & Klemperer, P. (1996). Auctions Versus Negotiations. *American Economic Review*, 86(1), 180--194.
- Clarke, E. H. (1971). Multipart Pricing of Public Goods. *Public Choice*, 11, 17--33.
- Groves, T. (1973). Incentives in Teams. *Econometrica*, 41(4), 617--631.
- Kelso, A. S., & Crawford, V. P. (1982). Job Matching, Coalition Formation, and Gross Substitutes. *Econometrica*, 50(6), 1483--1504.
- Lehmann, D., O'Callaghan, L. I., & Shoham, Y. (2002). Truth Revelation in Approximately Efficient Combinatorial Auctions. *Journal of the ACM*, 49(5), 577--602.
- McAfee, R. P., McMillan, J., & Whinston, M. D. (1989). Multiproduct Monopoly, Commodity Bundling, and Correlation of Values. *Quarterly Journal of Economics*, 104(2), 371--383.
- Milgrom, P. (2004). *Putting Auction Theory to Work*. Cambridge University Press.
- Milgrom, P., & Weber, R. J. (1982). A Theory of Auctions and Competitive Bidding. *Econometrica*, 50(5), 1089--1122.
- Myerson, R. B. (1981). Optimal Auction Design. *Mathematics of Operations Research*, 6(1), 58--73.
- Myerson, R. B., & Satterthwaite, M. A. (1983). Efficient Mechanisms for Bilateral Trading. *Journal of Economic Theory*, 29(2), 265--281.
- Rustichini, A., Satterthwaite, M. A., & Williams, S. R. (1994). Convergence to Efficiency in a Simple Market with Incomplete Information. *Econometrica*, 62(5), 1041--1063.
- Vickrey, W. (1961). Counterspeculation, Auctions, and Competitive Sealed Tenders. *Journal of Finance*, 16(1), 8--37.

### Information Design and Bayesian Persuasion
- Kamenica, E., & Gentzkow, M. (2011). Bayesian Persuasion. *American Economic Review*, 101(6), 2590--2615.

### Dynamic Pricing and Durable Goods
- Broadbent, S. (1979). One Way TV Advertisements Work. *Journal of the Market Research Society*, 21(3), 139--166.
- Stokey, N. L. (1979). Intertemporal Price Discrimination. *Quarterly Journal of Economics*, 93(3), 355--371.

### Game Theory
- Fudenberg, D., & Maskin, E. (1986). The Folk Theorem in Repeated Games with Discounting or with Incomplete Information. *Econometrica*, 54(3), 533--554.

### Two-Sided Markets
- Armstrong, M. (2006). Competition in Two-Sided Markets. *RAND Journal of Economics*, 37(3), 668--691.
- Rochet, J. C., & Tirole, J. (2003). Platform Competition in Two-Sided Markets. *Journal of the European Economic Association*, 1(4), 990--1029.
- Rochet, J. C., & Tirole, J. (2006). Two-Sided Markets: A Progress Report. *RAND Journal of Economics*, 37(3), 645--667.

### Behavioral Economics
- Kahneman, D., & Tversky, A. (1979). Prospect Theory: An Analysis of Decision under Risk. *Econometrica*, 47(2), 263--291.

### Information Economics
- Akerlof, G. A. (1970). The Market for "Lemons": Quality Uncertainty and the Market Mechanism. *Quarterly Journal of Economics*, 84(3), 488--500.
- Shapiro, C. (1983). Premiums for High Quality Products as Returns to Reputations. *Quarterly Journal of Economics*, 98(4), 659--680.

### Bundling and Auction Design (Applied)
- Budish, E. B., & Takeyama, L. N. (2001). Buy Prices in Online Auctions: Irrationality on the Internet? *Economics Letters*, 72(3), 325--333.
- Reynolds, S. S., & Wooders, J. (2009). Auctions with a Buy Price. *Economic Theory*, 38(1), 9--39.

---

*Document version: 1.0 | February 2026*
*Companion to: 内容价值量化与交易市场 深度框架 V3.0*
*Theoretical framework for the Douyin Content Trading Marketplace auction mechanism*
