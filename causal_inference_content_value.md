# Causal Inference Methodology for Content Value Measurement on Douyin

## A Technical Framework for the Content Value Quantification and Trading Marketplace

---

## Table of Contents

1. [Foundational Causal Framework](#1-foundational-causal-framework)
2. [Douyin's Existing Causal Inference Infrastructure](#2-douyins-existing-causal-inference-infrastructure)
3. [State-of-the-Art Causal Inference Methods](#3-state-of-the-art-causal-inference-methods)
4. [The Fundamental Attribution Problem](#4-the-fundamental-attribution-problem)
5. [Modeling Approaches for Each Value Layer](#5-modeling-approaches-for-each-value-layer)
6. [Real Implementation Challenges](#6-real-implementation-challenges)
7. [Integrated System Architecture](#7-integrated-system-architecture)
8. [References](#8-references)

---

## 1. Foundational Causal Framework

### 1.1 The Potential Outcomes Framework (Rubin Causal Model)

The entire content value measurement system rests on the **Neyman-Rubin Potential Outcomes Framework** (Rubin, 1974; Holland, 1986). For each user-content interaction, we define:

**Notation:**
- `i`: user index
- `j`: content piece index
- `W_i in {0, 1}`: treatment indicator (exposed to content j = 1, not exposed = 0)
- `Y_i(1)`: potential outcome if user i IS exposed to content j
- `Y_i(0)`: potential outcome if user i is NOT exposed to content j
- `X_i`: vector of pre-treatment covariates (user features, behavioral history)

**Individual Treatment Effect (ITE):**

```
tau_i = Y_i(1) - Y_i(0)
```

**The Fundamental Problem of Causal Inference** (Holland, 1986): We can never observe both `Y_i(1)` and `Y_i(0)` for the same unit. This is the core challenge: for any user who watched a piece of content, we cannot directly observe what they would have done without it.

**Conditional Average Treatment Effect (CATE):**

```
tau(x) = E[Y(1) - Y(0) | X = x]
```

**Average Treatment Effect (ATE):**

```
ATE = E[Y(1) - Y(0)] = E[tau(X)]
```

**Average Treatment Effect on the Treated (ATT):**

```
ATT = E[Y(1) - Y(0) | W = 1]
```

In content valuation, ATT is often more relevant than ATE because we want to know: "For users who actually consumed this content, what was the incremental value?"

### 1.2 Key Identification Assumptions

**SUTVA (Stable Unit Treatment Value Assumption):**

```
Y_i = W_i * Y_i(1) + (1 - W_i) * Y_i(0)
```

This requires: (a) no interference between units -- one user's content exposure does not affect another user's outcome; (b) no hidden variations of treatment -- watching the same content produces the same version of treatment. As we discuss in Section 6, SUTVA is systematically violated on social platforms.

**Unconfoundedness (Ignorability):**

```
{Y(1), Y(0)} _||_ W | X
```

Conditional on observed covariates X, treatment assignment is as good as random. This is the core assumption enabling observational causal inference.

**Overlap (Positivity):**

```
0 < P(W = 1 | X = x) < 1  for all x in support(X)
```

Every user must have a non-zero probability of both seeing and not seeing the content. This is naturally satisfied on Douyin because the recommendation algorithm does not deterministically assign content.

### 1.3 Structural Causal Model (Pearl's Framework)

Pearl's SCM framework (Pearl, 2009) provides complementary tools through **Directed Acyclic Graphs (DAGs)** and the **do-calculus**.

**DAG for Content-Commerce on Douyin:**

```
                    U_algo
                      |
                      v
User_Features(X) --> Algo_Score --> Content_Exposure(W) --> User_Engagement --> Commerce_Outcome(Y)
       |                                    ^                      |                    ^
       |                                    |                      |                    |
       +------------------------------------+                      +--------------------+
       |                                                                                |
       +--- Product_Interest(Z) -------------------------------------------------------+
       |                                                                                |
       +--- IP_Affinity(A) ------------------------------------------------------------+
       |                                                                                |
       +--- Temporal_Context(T) -------------------------------------------------------+
```

The **do-operator** allows us to express interventional queries:

```
P(Y | do(W = 1)) != P(Y | W = 1)
```

The left side asks "what would happen if we FORCE exposure to content" (causal), while the right side asks "what is the outcome among those who HAPPENED to see the content" (observational, confounded). The entire methodology below is designed to bridge this gap.

**Backdoor Criterion:** A set of variables Z satisfies the backdoor criterion relative to (W, Y) if: (a) no node in Z is a descendant of W, and (b) Z blocks every path between W and Y that contains an arrow into W.

**Adjustment Formula (Pearl):**

```
P(Y | do(W = w)) = SUM_z P(Y | W = w, Z = z) * P(Z = z)
```

---

## 2. Douyin's Existing Causal Inference Infrastructure

### 2.1 The O-5A People Asset Model

Douyin's O-5A framework (developed through Juliangliangtu / Ocean Engine) maps the consumer journey across six stages:

```
O (Opportunity) --> A1 (Aware) --> A2 (Appeal) --> A3 (Ask/Seed) --> A4 (Act/Purchase) --> A5 (Advocate/Repurchase)
```

**Stage Definitions with Behavioral Indicators:**

| Stage | Definition | Behavioral Signals | Measurement |
|-------|-----------|-------------------|-------------|
| O | Potential audience pool | Matches category interest tags | DMP tag overlap |
| A1 | Aware | 1-3 brand exposures, passive impression | Impression logs |
| A2 | Appeal | Active engagement: like, comment, short watch | Engagement events |
| A3 | Ask/Seed (zhongcao) | Deep engagement: search, add-to-cart, long watch, share, full video completion | Conversion funnel events |
| A4 | Act | First purchase | Transaction records |
| A5 | Advocate | Repeat purchase, UGC creation, referral | Repeat transaction + content creation logs |

**Transition Measurement:**

The transition probability matrix is:

```
P(A_k+1 | A_k, Content_j, X_i)
```

For content value measurement, the key question is the **incremental transition probability**:

```
Delta_P(A_k -> A_k+1 | Content_j) = P(A_k+1 | A_k, exposed_to_j) - P(A_k+1 | A_k, NOT_exposed_to_j)
```

This is precisely a CATE estimation problem, where the treatment is exposure to content_j and the outcome is transition to the next 5A stage.

**SPU-Level 5A (Product-Level):**

Juliangliangtu has extended the model to SPU (Single Product Unit) level, tracking individual product seeding journeys. This creates a three-dimensional measurement space: `(User_i, Content_j, Product_k)` with outcomes measured at each 5A transition.

### 2.2 Volcengine DataTester (A/B Testing Infrastructure)

ByteDance's internal A/B testing platform, externalized as Volcengine DataTester, is one of the world's largest experimentation platforms. Key architectural components:

**Traffic Splitting Architecture:**

```
Total Traffic
    |
    v
[Hash Function: user_id -> bucket_id]
    |
    v
[Layer System: orthogonal experiment layers]
    |
    +--> Layer 1 (UI experiments): Buckets 0-99
    +--> Layer 2 (Algo experiments): Buckets 0-99  (independent hash)
    +--> Layer 3 (Content experiments): Buckets 0-99 (independent hash)
    |
    v
[Per-Layer Allocation]
    Experiment A: Buckets 0-9 (10% traffic)
    Experiment B: Buckets 10-19 (10% traffic)
    Control: Buckets 20-99 (80% traffic)
```

The layer system enables orthogonal experiments -- experiments in different layers are statistically independent, allowing simultaneous testing of content treatments, algorithm changes, and UI modifications.

**Statistical Engine:**

- **Primary method**: Two-sample t-test / z-test for metric differences
- **Variance reduction**: CUPED (Controlled-experiment Using Pre-Existing Data) (Deng et al., 2013)
- **Sequential testing**: Supports always-valid p-values for continuous monitoring
- **Multiple testing correction**: Benjamini-Hochberg FDR control

**CUPED Variance Reduction:**

The CUPED adjustment creates a variance-reduced metric:

```
Y_i^cv = Y_i - theta * (X_i^pre - E[X_i^pre])
```

where:

```
theta* = Cov(Y_i, X_i^pre) / Var(X_i^pre)
```

The variance reduction factor is:

```
Var(Y^cv) / Var(Y) = 1 - rho^2(Y, X^pre)
```

where `rho` is the correlation between the post-treatment outcome and the pre-treatment covariate. For metrics with strong temporal autocorrelation (e.g., daily active usage, purchase frequency), CUPED can reduce variance by 30-50%, effectively doubling the speed of experiments.

**Application to Content Experiments:**

For content value measurement, DataTester enables:
1. **Holdout experiments**: Randomly withhold content from a control group
2. **Ghost ads methodology**: For ad-content, identify users who *would have* been shown the content, randomly serve a placeholder instead, measure the counterfactual
3. **Geo-randomized experiments**: For large-scale content campaigns, randomize at the city level
4. **Time-based experiments**: Content exposure on/off periods for time-series analysis

### 2.3 Incremental Measurement (Organic vs. Incremental)

The core attribution question: For a user who saw content and then purchased, would they have purchased anyway?

**Douyin's Attribution Model:**

```
Total_Conversions = Organic_Conversions + Incremental_Conversions

Incrementality_Rate = Incremental_Conversions / Total_Conversions
```

**The Seeding (Zhongcao) Attribution Chain:**

Douyin's content seeding model treats A3 population as the core measurable seeding audience:

```
Content Exposure --> A3 Accumulation --> Search Behavior --> Livestream Entry --> Purchase
                                              |
                                              v
                                    [Brand Search via "Blue Words"]
                                              |
                                              v
                                    [Brand Zone / Product Page]
                                              |
                                              v
                                    [Livestream / Direct Purchase]
```

Research from Douyin's marketing science team shows that brand search zone (pinzhuan) conversion GMV is approximately 28% higher than organic search GMV, attributable to the pre-seeding of A3 audiences.

**Measurement Windows:**

- **Click-through attribution**: 1-day, 7-day, 15-day, 30-day windows
- **View-through attribution**: 1-day, 7-day windows
- **Seeding attribution (A3)**: 7-day, 15-day, 30-day windows for measuring downstream conversion of seeded audiences
- **Long-tail seeding value**: SPU-level tracking of seeded users over 30-90 day periods

### 2.4 Juliangxingtu (Star Map) Content Value Measurement

Juliangxingtu, Douyin's KOL/creator marketing platform, provides two key measurement tools:

**Xing Li Fang (Star Cube):** Measures creator content effectiveness across:
- Propagation value (reach, impressions, engagement rate)
- Seeding value (A3 accumulation, search lift)
- Conversion value (direct attribution GMV, assisted conversions)

**Full-Domain Measurement (Quanyu Duliang):** Measures creator content impact across both in-platform and out-of-platform channels, tracking cross-domain attribution of content seeding effects.

---

## 3. State-of-the-Art Causal Inference Methods

### 3.1 Meta-Learners for Uplift Modeling

Meta-learners estimate CATE by combining off-the-shelf ML models with causal reasoning. These are the workhorses for heterogeneous content effect estimation.

#### 3.1.1 S-Learner (Single Model)

**Approach:** Train a single model including the treatment indicator as a feature.

```
mu(x, w) = E[Y | X = x, W = w]

tau_S(x) = mu(x, 1) - mu(x, 0)
```

**Implementation:**

```python
# S-Learner
model = GradientBoostingRegressor()
X_augmented = np.column_stack([X, W])  # Append treatment indicator
model.fit(X_augmented, Y)

# Estimate CATE
tau_hat = model.predict(np.column_stack([X, np.ones(n)])) - \
          model.predict(np.column_stack([X, np.zeros(n)]))
```

**Strengths:** Simple; uses all data for training.
**Weaknesses:** Regularization can shrink the treatment effect toward zero if the base learner penalizes the treatment indicator; model may "ignore" the treatment variable if other features are more predictive.

**When to use for content:** Best when treatment effects are expected to be small relative to outcome variance (e.g., measuring subtle V2 commerce stimulation effects of non-commercial content).

#### 3.1.2 T-Learner (Two-Model)

**Approach:** Train separate models for treated and control groups.

```
mu_1(x) = E[Y | X = x, W = 1]    (model on treated group)
mu_0(x) = E[Y | X = x, W = 0]    (model on control group)

tau_T(x) = mu_1(x) - mu_0(x)
```

**Implementation:**

```python
# T-Learner
model_1 = GradientBoostingRegressor()
model_0 = GradientBoostingRegressor()
model_1.fit(X[W==1], Y[W==1])
model_0.fit(X[W==0], Y[W==0])

tau_hat = model_1.predict(X) - model_0.predict(X)
```

**Strengths:** Allows complete flexibility in modeling treated vs. control outcomes.
**Weaknesses:** Does not share information between treatment and control groups; can perform poorly when groups are unbalanced (common when content exposure is rare or very common).

**When to use for content:** Best for V4 brand conversion analysis where treatment (ad exposure) and control (holdout) groups are experimentally balanced.

#### 3.1.3 X-Learner (Kunzel et al., 2019)

**Approach:** Three-stage procedure that transfers information between treatment and control groups.

**Stage 1:** Fit outcome models separately.

```
mu_1(x) = E[Y | X = x, W = 1]
mu_0(x) = E[Y | X = x, W = 0]
```

**Stage 2:** Compute imputed treatment effects.

```
For treated units:    D_i^1 = Y_i^1 - mu_0(X_i^1)    (observed - imputed counterfactual)
For control units:    D_i^0 = mu_1(X_i^0) - Y_i^0    (imputed counterfactual - observed)
```

Then fit two new models:

```
tau_1(x) = E[D^1 | X = x]    (CATE estimated from treated group)
tau_0(x) = E[D^0 | X = x]    (CATE estimated from control group)
```

**Stage 3:** Combine using propensity score weighting.

```
tau_X(x) = g(x) * tau_0(x) + (1 - g(x)) * tau_1(x)
```

where `g(x) = P(W = 1 | X = x)` is the propensity score. This ensures we weight toward the estimate with more data: if most users see the content (high propensity), we weight more toward `tau_1` which was trained on the larger treated group.

**Key advantage for content:** On Douyin, viral content creates extreme imbalance (90%+ exposure rates in target demographics). The X-learner exploits this imbalance by using the large exposed group to improve estimates for the small unexposed group.

#### 3.1.4 R-Learner (Nie & Wager, 2021)

**Approach:** Based on Robinson's (1988) decomposition, directly targets the treatment effect by removing nuisance components.

**Robinson Decomposition:**

Starting from the partially linear model:

```
Y_i = tau(X_i) * W_i + f(X_i) + epsilon_i
W_i = e(X_i) + eta_i
```

Subtract conditional expectations:

```
Y_i - m(X_i) = tau(X_i) * (W_i - e(X_i)) + epsilon_i
```

where `m(x) = E[Y|X=x]` and `e(x) = E[W|X=x] = P(W=1|X=x)`.

**R-Loss function:**

```
L_R(tau) = (1/n) * SUM_i [ (Y_i - m_hat(X_i) - tau(X_i) * (W_i - e_hat(X_i)))^2 ]
```

**Estimation procedure:**

1. Estimate nuisance functions: `m_hat(x)` (outcome model) and `e_hat(x)` (propensity model) using cross-fitting
2. Minimize R-loss over function class for `tau`

```python
# R-Learner implementation sketch
from sklearn.model_selection import KFold

# Step 1: Cross-fitted nuisance estimates
kf = KFold(n_splits=5)
m_hat = np.zeros(n)
e_hat = np.zeros(n)
for train_idx, test_idx in kf.split(X):
    outcome_model.fit(X[train_idx], Y[train_idx])
    propensity_model.fit(X[train_idx], W[train_idx])
    m_hat[test_idx] = outcome_model.predict(X[test_idx])
    e_hat[test_idx] = propensity_model.predict_proba(X[test_idx])[:, 1]

# Step 2: Compute residuals
Y_residual = Y - m_hat
W_residual = W - e_hat

# Step 3: Weighted regression of Y_residual on W_residual
# with weights W_residual^2
tau_model.fit(X, Y_residual / W_residual, sample_weight=W_residual**2)
```

**Key property:** Rate double robustness -- the estimator achieves root-n consistency if EITHER the outcome model or the propensity model converges at a rate faster than n^(-1/4), not requiring both to be correct.

**When to use for content:** The R-learner is ideal for V2 and V3 estimation where confounding is complex and we need robustness to model misspecification. The propensity score (recommendation probability) is directly observable from Douyin's algorithm logs.

### 3.2 Causal Forests (Athey, Tibshirani & Wager, 2019)

**Generalized Random Forests (GRF)** extend Breiman's random forests to solve local moment equations, with causal forests as the primary application for CATE estimation.

**Core Idea:** Each tree partitions the covariate space to maximize heterogeneity in treatment effects, not prediction accuracy.

**Splitting Criterion:**

At each node, GRF solves a local estimating equation. For causal forests, the splitting criterion maximizes:

```
Delta(C_1, C_2) = ( SUM_{i in C_1} alpha_i * (Y_i - Y_bar_C1) * (W_i - W_bar_C1) /
                     SUM_{i in C_1} alpha_i * (W_i - W_bar_C1)^2
                   -
                    SUM_{i in C_2} alpha_i * (Y_i - Y_bar_C2) * (W_i - W_bar_C2) /
                     SUM_{i in C_2} alpha_i * (W_i - W_bar_C2)^2 )^2
```

This is the squared difference in local CATE estimates between the two child nodes.

**Honesty:**

Causal forests use an "honest" approach:
1. Split the training data into a splitting subsample and an estimation subsample
2. Use the splitting subsample to determine tree structure
3. Use the estimation subsample to populate leaf estimates

This separation ensures valid confidence intervals.

**CATE Estimation:**

For a new point x:

```
tau_hat(x) = SUM_i alpha_i(x) * Y_i
```

where the forest weights `alpha_i(x)` are determined by co-occurrence in leaf nodes:

```
alpha_i(x) = (1/B) * SUM_{b=1}^{B} [ 1(X_i in L_b(x)) / |{j : X_j in L_b(x)}| ]
```

where `L_b(x)` is the leaf containing x in tree b.

**Asymptotic Properties (Athey et al., 2019):**

Under regularity conditions:

```
(tau_hat(x) - tau(x)) / sigma_hat(x)  -->  N(0, 1)
```

This enables valid pointwise confidence intervals for heterogeneous treatment effects.

**Implementation for content value:**

```r
library(grf)

# Fit causal forest
cf <- causal_forest(
  X = user_features,           # User covariates
  Y = purchase_outcome,        # Outcome variable
  W = content_exposure,        # Treatment (content exposure)
  num.trees = 2000,
  honesty = TRUE,
  tune.parameters = "all"
)

# Estimate CATE for each user
tau_hat <- predict(cf, estimate.variance = TRUE)
cate <- tau_hat$predictions
ci_lower <- cate - 1.96 * sqrt(tau_hat$variance.estimates)
ci_upper <- cate + 1.96 * sqrt(tau_hat$variance.estimates)

# Variable importance for treatment heterogeneity
importance <- variable_importance(cf)

# Best linear projection: which features drive heterogeneity?
blp <- best_linear_projection(cf, user_features[, top_features])
```

### 3.3 Double/Debiased Machine Learning (DML)

**Reference:** Chernozhukov, Chetverikov, Demirer, Duflo, Hansen, Newey, & Robins (2018)

DML provides a rigorous framework for using ML methods to estimate low-dimensional causal parameters (like ATE) while controlling for high-dimensional confounders.

**The Problem:** Naive plug-in ML estimators suffer from regularization bias. If we simply run a LASSO or neural net to predict Y from (W, X), the coefficient on W is biased because regularization shrinks it toward zero.

**DML Solution: Two Key Ingredients**

**(1) Neyman-Orthogonal Score:**

The partially linear model:

```
Y = theta_0 * W + g_0(X) + U,    E[U | X, W] = 0
W = m_0(X) + V,                    E[V | X] = 0
```

The orthogonal score function:

```
psi(Y, W, X; theta, eta) = (Y - theta*W - g(X)) * (W - m(X))
```

This score has the property that its derivative with respect to the nuisance parameters (g, m) is zero at the true values -- hence "orthogonal."

**(2) Cross-Fitting:**

Split data into K folds. For each fold k:
- Estimate nuisance parameters (g_hat, m_hat) on data EXCLUDING fold k
- Compute residuals on fold k using the estimated nuisance functions

```
theta_hat_DML = [ SUM_k SUM_{i in I_k} (W_i - m_hat_{-k}(X_i)) * (Y_i - g_hat_{-k}(X_i)) ] /
                [ SUM_k SUM_{i in I_k} (W_i - m_hat_{-k}(X_i))^2 ]
```

**Asymptotic Properties:**

```
sqrt(N) * (theta_hat_DML - theta_0) --> N(0, sigma^2)
```

where:

```
sigma^2 = E[(Y - theta_0*W - g_0(X))^2 * (W - m_0(X))^2] / (E[(W - m_0(X))^2])^2
```

Crucially, this holds even when the nuisance parameter estimators converge at rates slower than root-n, as long as the product of their rates exceeds n^(-1/2).

**Interactive Regression Model (IRM) for Heterogeneous Effects:**

For CATE estimation:

```
Y = g_0(W, X) + U
W = m_0(X) + V
```

The Augmented Inverse Propensity Weighted (AIPW) score:

```
psi_AIPW(theta) = g_hat(1,X) - g_hat(0,X) + W*(Y - g_hat(1,X))/e_hat(X) - (1-W)*(Y - g_hat(0,X))/(1-e_hat(X)) - theta
```

This is **doubly robust**: consistent if either `g` or `e` is correctly specified.

**Application to content value:**

```python
from doubleml import DoubleMLPLR, DoubleMLIRM, DoubleMLData
from sklearn.ensemble import GradientBoostingRegressor, GradientBoostingClassifier

# Prepare data
dml_data = DoubleMLData.from_arrays(
    x=user_features,
    y=commerce_outcome,
    d=content_exposure
)

# Partially Linear Regression for ATE
dml_plr = DoubleMLPLR(
    dml_data,
    ml_l=GradientBoostingRegressor(),  # outcome nuisance
    ml_m=GradientBoostingClassifier(), # treatment nuisance
    n_folds=5,
    score='partialling out'
)
dml_plr.fit()
print(f"ATE: {dml_plr.coef[0]:.4f}, SE: {dml_plr.se[0]:.4f}")

# Interactive Regression Model for ATE with doubly robust score
dml_irm = DoubleMLIRM(
    dml_data,
    ml_g=GradientBoostingRegressor(),
    ml_m=GradientBoostingClassifier(),
    n_folds=5,
    score='ATE'
)
dml_irm.fit()
```

### 3.4 Instrumental Variable Approaches

When unconfoundedness fails (there are unobserved confounders), IV methods can still identify causal effects.

**The Endogeneity Problem in Content:**

```
DAG with unobserved confounder:

Content_Exposure(W) --> Commerce_Outcome(Y)
       ^                        ^
       |                        |
       +--- U (unobserved: e.g., latent purchase intent) ---+
```

Users who choose to watch product-related content already have higher purchase intent -- this creates endogeneity that X alone cannot resolve.

**Instrument Construction for Douyin:**

Potential instruments (Z must affect Y ONLY through W):

1. **Algorithm exploration shocks**: Random perturbations in the recommendation algorithm's exploration phase serve as exogenous variation in content exposure. When Douyin's algorithm randomly "explores" by showing content from outside the user's usual preferences, this creates quasi-random variation.

```
Z_1 = Indicator(user received exploration-mode recommendation for content_j)
```

2. **Creator posting time**: The exact time a creator publishes content is exogenous to user purchase intent. Users active at that time have higher exposure probability.

```
Z_2 = f(Content_j_publish_time, User_i_active_time_overlap)
```

3. **Device/network speed**: Temporary network congestion affects video loading speed and thus completion rates, but should not directly affect purchase intent.

```
Z_3 = Video_load_latency_for_user_i_at_time_t
```

**Two-Stage Least Squares (2SLS):**

**Stage 1 (First Stage):**
```
W_i = pi * Z_i + gamma * X_i + v_i
```

**Stage 2 (Second Stage):**
```
Y_i = tau * W_hat_i + beta * X_i + epsilon_i
```

where `W_hat_i` is the predicted value from Stage 1.

**Local Average Treatment Effect (LATE):**

With binary instrument and binary treatment:

```
LATE = E[Y | Z=1] - E[Y | Z=0]
       --------------------------
       E[W | Z=1] - E[W | Z=0]
```

This estimates the treatment effect for **compliers** -- users whose content exposure is actually affected by the instrument.

**Instrument Validity Tests:**

- **Relevance (F-test):** First-stage F-statistic > 10 (Staiger & Stock, 1997)
- **Exclusion restriction:** Cannot be directly tested; requires domain expertise
- **Monotonicity:** The instrument must shift exposure in the same direction for all users

### 3.5 Difference-in-Differences (DiD)

DiD exploits temporal variation in content exposure to identify causal effects with panel data.

**Basic DiD Setup:**

For user i at time t, with content exposure starting at time T*:

```
Y_it = alpha + beta * Treated_i + gamma * Post_t + tau * (Treated_i * Post_t) + epsilon_it
```

where `tau` is the DiD estimator of the treatment effect.

**Parallel Trends Assumption:**

```
E[Y_it(0) | Treated_i = 1] - E[Y_i,t-1(0) | Treated_i = 1] =
E[Y_it(0) | Treated_i = 0] - E[Y_i,t-1(0) | Treated_i = 0]
```

In absence of treatment, treated and control groups would have followed parallel outcome trajectories.

**Two-Way Fixed Effects (TWFE):**

```
Y_it = alpha_i + lambda_t + tau * W_it + X_it * beta + epsilon_it
```

where `alpha_i` = user fixed effects, `lambda_t` = time fixed effects.

**Staggered DiD (Callaway & Sant'Anna, 2021):**

For content that reaches different users at different times (staggered adoption):

```
ATT(g,t) = E[Y_t - Y_g-1 | G = g] - E[Y_t - Y_g-1 | C = 1]
```

where g is the cohort (time of first content exposure) and t is the calendar time.

**Application to content campaigns:** When a brand launches a content campaign on Douyin, DiD can compare users who were exposed (treatment) to similar users who were not (control), before and after the campaign, controlling for pre-existing trends.

**Continuous Treatment DiD:**

For varying levels of content exposure (dose-response):

```
Y_it = alpha_i + lambda_t + tau(D_it) + epsilon_it
```

where `D_it` is the continuous treatment intensity (e.g., number of content pieces viewed).

### 3.6 Regression Discontinuity Design (RDD)

RDD exploits threshold-based assignment rules in Douyin's recommendation algorithm.

**The Threshold Opportunity:**

Douyin's recommendation system operates through a multi-stage funnel:

```
Content Pool
    |
    v
[Recall Layer: thousands of candidates]
    |
    v
[Pre-ranking: hundreds of candidates, coarse scoring]
    |
    v
[Ranking: final scoring, Score_j for each content piece]
    |
    v
[Threshold: Score_j > c_t => shown to user]
    |
    v
User Feed
```

Content pieces with scores just above the threshold are quasi-randomly different from those just below.

**Sharp RDD:**

```
W_i = 1(Score_i >= c)

tau_RDD = lim_{s->c+} E[Y | Score = s] - lim_{s->c-} E[Y | Score = s]
```

**Estimation via Local Linear Regression:**

```
For observations near the threshold (|Score_i - c| < h):

Y_i = alpha + tau * 1(Score_i >= c) + beta_1 * (Score_i - c) + beta_2 * (Score_i - c) * 1(Score_i >= c) + epsilon_i
```

**Bandwidth Selection:** Use methods from Imbens & Kalyanaraman (2012) or Calonico, Cattaneo & Titiunik (2014) for optimal bandwidth selection.

**Fuzzy RDD:**

In practice, the threshold is not perfectly sharp (content with similar scores may be swapped due to diversity requirements). Fuzzy RDD uses an IV approach:

```
tau_FRDD = [lim_{s->c+} E[Y|S=s] - lim_{s->c-} E[Y|S=s]] / [lim_{s->c+} E[W|S=s] - lim_{s->c-} E[W|S=s]]
```

**Application to content value:** The RDD approach allows estimation of the local content value at the margin -- the commercial value of content that is "borderline" recommended. This is especially useful for calibrating the recommendation algorithm's content-commerce trade-offs.

---

## 4. The Fundamental Attribution Problem

### 4.1 Decomposing Observed Outcomes

The total observed commerce outcome for a user can be decomposed:

```
Y_observed = f(Content_Effect, IP_Effect, Product_Effect, Timing_Effect, Algorithm_Effect, Baseline_Intent)
```

**DAG for the Full Attribution Problem:**

```
                  Creator_IP(I)
                      |
                      v
User_Baseline(B) --> Content_Exposure(W) --> Engagement(E) --> Commerce(Y)
       |                 ^        |                                ^
       |                 |        |                                |
       |          Algorithm(A)    +-- Product_Features(P) --------+
       |                 ^                                         |
       |                 |                                         |
       +-----------------+--- Temporal_Context(T) ----------------+
       |
       +--- Latent_Purchase_Intent(U) ----> Commerce(Y)  [UNOBSERVED CONFOUNDER]
```

**The Decomposition Challenge:**

We need to separate:

```
tau_content(x) = tau_total(x) - tau_IP(x) - tau_product(x) - tau_timing(x) - tau_algorithm(x)
```

But these effects are not additive in general -- there are interaction effects:

```
tau_total(x) = tau_content + tau_IP + tau_product + tau_timing + tau_algo
             + tau_content*IP + tau_content*product + tau_content*timing + ...
             + higher-order interactions
```

### 4.2 Multi-Touch Attribution Models

#### Shapley Value Attribution

The Shapley value from cooperative game theory provides a principled way to allocate credit among touchpoints.

**Setup:** Let N = {1, 2, ..., n} be the set of touchpoints (content pieces) in a user's journey. For any coalition S subset of N, let v(S) = conversion probability given touchpoints in S.

**Shapley Value for touchpoint i:**

```
phi_i = SUM_{S subset N\{i}} [ |S|! * (|N|-|S|-1)! / |N|! ] * [v(S union {i}) - v(S)]
```

**Properties:**
- **Efficiency:** `SUM_i phi_i = v(N) - v(empty)` (total value is fully allocated)
- **Symmetry:** If two touchpoints contribute identically to all coalitions, they receive equal credit
- **Null player:** If a touchpoint adds no value to any coalition, it receives zero credit
- **Linearity:** Shapley values are additive across games

**Computational Challenge:**

Computing exact Shapley values requires evaluating `2^n` coalitions. For a user journey with 20 content touchpoints, this means ~1 million evaluations. Approximation methods:

1. **Sampling-based:** Monte Carlo sampling of permutations (Castro et al., 2009)
2. **Kernel SHAP:** Weighted linear regression approximation (Lundberg & Lee, 2017)
3. **Structured approximations:** Exploit temporal ordering of touchpoints

**Implementation for content paths:**

```python
import itertools
import numpy as np

def shapley_attribution(touchpoints, conversion_model):
    """
    touchpoints: list of content_ids in user journey
    conversion_model: f(subset) -> P(conversion)
    """
    n = len(touchpoints)
    shapley_values = np.zeros(n)

    for i in range(n):
        others = [j for j in range(n) if j != i]
        marginal_contributions = []

        for size in range(len(others) + 1):
            for S in itertools.combinations(others, size):
                S_set = set(S)
                S_with_i = S_set | {i}

                # Marginal contribution of i to coalition S
                mc = conversion_model(S_with_i) - conversion_model(S_set)

                # Weight: |S|! * (n-|S|-1)! / n!
                weight = (np.math.factorial(len(S_set)) *
                         np.math.factorial(n - len(S_set) - 1) /
                         np.math.factorial(n))
                marginal_contributions.append(weight * mc)

        shapley_values[i] = sum(marginal_contributions)

    return shapley_values
```

#### Markov Chain Attribution

Models the user journey as a Markov chain where states represent touchpoints.

**Transition Matrix:**

```
P = [p_ij]  where p_ij = P(next state = j | current state = i)
```

States include: Start, Content_1, Content_2, ..., Content_n, Conversion, Non-Conversion.

**Removal Effect:**

The attribution for channel i is based on its "removal effect":

```
RE_i = 1 - P(Conversion | channel_i_removed) / P(Conversion | all_channels)
```

**Normalized attribution:**

```
Attribution_i = RE_i / SUM_j RE_j
```

**Advantage over Shapley:** Computationally feasible for large numbers of touchpoints because the Markov property means we only need the transition matrix, not all 2^n coalitions.

### 4.3 Self-Selection Bias

The most pernicious problem in content value measurement: users who watch product-related content are already more likely to purchase.

**Formal Statement:**

```
E[Y | W=1] - E[Y | W=0] = E[Y(1) - Y(0) | W=1]  +  {E[Y(0)|W=1] - E[Y(0)|W=0]}
                            ^                           ^
                            ATT (causal effect)         Selection Bias
```

The selection bias term `E[Y(0)|W=1] - E[Y(0)|W=0]` represents the baseline difference in purchase probability between users who would watch the content and those who would not, EVEN WITHOUT the content.

**Mitigation Strategies:**

1. **Propensity Score Methods:**

```
e(x) = P(W=1 | X=x)
```

- **Matching:** Match each treated user with a control user having similar propensity score
- **IPW:** Inverse Propensity Weighting

```
ATE_IPW = (1/n) * SUM_i [ W_i*Y_i/e(X_i) - (1-W_i)*Y_i/(1-e(X_i)) ]
```

- **AIPW (Doubly Robust):**

```
ATE_AIPW = (1/n) * SUM_i [ mu_1(X_i) - mu_0(X_i)
           + W_i*(Y_i - mu_1(X_i))/e(X_i)
           - (1-W_i)*(Y_i - mu_0(X_i))/(1-e(X_i)) ]
```

2. **On Douyin specifically:** The propensity score `e(x)` is directly available from the recommendation algorithm's scoring function -- this is a massive advantage over other platforms.

3. **Sensitivity Analysis (Rosenbaum, 2002):** After propensity adjustment, conduct sensitivity analysis to assess how large an unobserved confounder would need to be to invalidate the causal conclusion.

### 4.4 View-Through Attribution Biases

**The "In-Market" Bias:**

Users targeted by the algorithm may already be "in-market" for a product. The algorithm's targeting is itself confounded:

```
Algorithm_Targeting <-- User_Purchase_Intent --> Purchase_Outcome
         |                                            ^
         v                                            |
    Content_Exposure ---------------------------------+
```

The algorithm targets users likely to convert, creating a spurious association between content exposure and conversion.

**Ghost Ads Solution (Johnson, Lewis & Nubbemeyer, 2017):**

For each ad/content impression opportunity:
1. Identify the user who would receive the content
2. With probability p, show the content (treatment)
3. With probability 1-p, show a neutral placeholder (control)

This creates true randomization conditional on algorithm selection, solving both the self-selection and in-market bias problems.

**Predicted Ghost Ads (for non-experimental settings):**

When randomization is not feasible:
1. Train a model to predict which content would have been shown to each user
2. Compare outcomes of users who saw the predicted content vs. those who did not
3. This approximates the ghost ad methodology observationally

---

## 5. Modeling Approaches for Each Value Layer

### 5.1 V1: Attention Value (Counterfactual Attention Estimation)

**Definition:** V1 measures the value of user attention captured by content -- the opportunity cost of the user's time that could have been spent elsewhere on the platform.

**Counterfactual Question:** "What would this user have done with the time they spent watching this content?"

**Model Architecture:**

```
            User_i arrives at feed position t
                        |
                        v
             [Recommendation Algorithm]
                   /          \
                  v            v
          Shows Content_j    Shows Counterfactual Content_j'
              (factual)       (counterfactual -- from candidate pool)
                  |                |
                  v                v
           Watch_Time = t_j   Watch_Time = t_j'
           Engagement = e_j   Engagement = e_j'
           Next_Action = a_j  Next_Action = a_j'
```

**Counterfactual Estimation via IPS:**

Using logged recommendation data:

```
V1(content_j) = E_users[ Y(content_j) - Y(counterfactual) | shown content_j ]
```

With Inverse Propensity Scoring:

```
V1_IPS = (1/n) * SUM_i [ (Y_i * 1(A_i = j)) / pi(j | X_i, context_i) ]
```

where `pi(j | X_i, context_i)` is the probability that the algorithm would recommend content j to user i in context.

**Doubly Robust Estimator for Attention Value:**

```
V1_DR = (1/n) * SUM_i [ mu_hat(X_i, j) + (1(A_i=j) / pi(j|X_i)) * (Y_i - mu_hat(X_i, j)) ]
```

**Practical Metrics for V1:**

```
V1_attention = SUM_users [ (watch_time_j - E[watch_time_counterfactual | X_i]) * attention_unit_price
               + (engagement_j - E[engagement_counterfactual | X_i]) * engagement_unit_price ]
```

**Attention Unit Price Calibration:**

The attention unit price is derived from Douyin's ad auction:

```
attention_unit_price = E[eCPM | user_segment, time_slot, content_category]
```

This represents the revenue Douyin foregoes by showing organic content instead of an ad.

### 5.2 V2: Commerce Stimulation Value (Purchase Probability Uplift)

**Definition:** V2 measures the incremental effect of non-commercial content on purchase probability -- content that is not explicitly selling but that increases purchase intent.

**This is the hardest value layer to measure** because the effect is indirect and confounded.

**Formal Model:**

```
V2(content_j, user_i) = P(Purchase_within_window | exposed_to_j, X_i) - P(Purchase_within_window | NOT_exposed_to_j, X_i)
```

This is exactly the CATE: `tau_j(X_i)`.

**Recommended Estimation Strategy: Causal Forest + DML Hybrid**

**Step 1: DML for Average Effect**

```python
# Estimate average V2 for content_j
dml_data = DoubleMLData.from_arrays(
    x=user_features,
    y=purchase_within_30d,
    d=exposed_to_content_j
)

dml_irm = DoubleMLIRM(dml_data, ml_g=XGBRegressor(), ml_m=XGBClassifier())
dml_irm.fit()
average_V2 = dml_irm.coef[0]
```

**Step 2: Causal Forest for Heterogeneous Effects**

```r
cf <- causal_forest(
  X = user_features,
  Y = purchase_within_30d,
  W = exposed_to_content_j,
  W.hat = recommendation_propensity  # Use algorithm's own propensity
)

# Heterogeneous V2 by user segment
user_V2 <- predict(cf)$predictions
```

**Step 3: Aggregate to Content-Level V2**

```
V2(content_j) = SUM_i [ tau_hat_j(X_i) * monetization_value_i ] / n_exposed
```

**DAG for V2 Identification:**

```
User_Interest(X) --> Algo_Exposure(W) --> V2_Effect --> Purchase(Y)
       |                  ^                               ^
       |                  |                               |
       +---> Latent_Intent(U) ---------------------------+

Instrument: Z = Algorithm_Exploration_Shock
```

When unconfoundedness is doubtful, use IV with algorithm exploration shocks as instruments.

### 5.3 V3: Category Influence Value (IP x Category Interaction)

**Definition:** V3 measures the effect of a creator/IP's content on category-level purchasing behavior -- how an IP shifts spending in a product category.

**Panel Data Model:**

```
Y_ict = alpha_ic + lambda_ct + gamma_it + tau * Exposure_ict + beta * X_ict + epsilon_ict
```

where:
- `i` = user, `c` = category, `t` = time period
- `alpha_ic` = user-category fixed effect (controls for baseline category preference)
- `lambda_ct` = category-time fixed effect (controls for category-level trends)
- `gamma_it` = user-time fixed effect (controls for individual spending shocks)
- `Exposure_ict` = user i's exposure to IP content in category c at time t

**IP x Category Interaction Model:**

```
Y_ict = alpha_ic + lambda_ct + SUM_k [ tau_k * IP_k_Exposure_ict ] + SUM_k SUM_c [ delta_kc * IP_k * Category_c * Exposure_ict ] + epsilon_ict
```

The interaction terms `delta_kc` capture the IP-specific category influence.

**Estimation with DML for Panel Data:**

```python
# For each (IP_k, Category_c) pair:
dml_data = DoubleMLData.from_arrays(
    x=np.column_stack([user_category_features, time_features]),
    y=category_spend_ict,
    d=ip_k_exposure_in_category_c
)

dml_plr = DoubleMLPLR(dml_data, ml_l=LGBMRegressor(), ml_m=LGBMClassifier())
dml_plr.fit()
V3_kc = dml_plr.coef[0]  # IP k's causal effect on category c spending
```

**Cross-Category Spillover Detection:**

```
V3_spillover(IP_k, Category_c, Category_c') =
    Effect of IP_k's content in Category_c on spending in Category_c'
```

This can be estimated via the same DML framework but with `Y = spending_in_category_c'` and `D = exposure_to_IP_k_content_in_category_c`.

### 5.4 V4: Brand Conversion Value (Direct Attribution)

**Definition:** V4 measures the direct causal effect of branded content (ads, sponsored content) on brand-specific conversions.

**This is the most measurable layer** because randomized experiments are feasible.

**Gold Standard: Holdout Experiment**

```
Population eligible for brand content
            |
            v
    [Random Assignment]
         /        \
        v          v
   Treatment      Control (Holdout)
   (see brand     (do NOT see brand
    content)       content)
        |              |
        v              v
   Measure Y_T     Measure Y_C
        |              |
        v              v
   V4 = Y_T - Y_C
```

**With Covariates (ANCOVA):**

```
V4_ANCOVA = (Y_T_bar - Y_C_bar) - beta * (X_T_bar - X_C_bar)
```

The covariate adjustment removes residual imbalance and reduces variance.

**Incremental ROAS (iROAS):**

```
iROAS = (Revenue_treatment - Revenue_control) / Cost_of_content
```

This is the gold standard for V4 measurement.

**Ghost Ads for Scalable V4 Measurement:**

For brands running always-on content campaigns, ghost ads enable continuous measurement:

1. For each ad impression opportunity, with probability p_ghost, divert to control
2. The control user sees what they would have seen without the brand content
3. Compare conversion rates between exposed and ghost-control groups

```
V4_ghost = E[Y | treated, eligible] - E[Y | ghost_control, eligible]
```

**Multi-Period Brand Lift:**

For measuring V4 over time (brand building effects):

```
V4(t) = SUM_{s=0}^{t} [ delta(s) * lambda^(t-s) ]
```

where `delta(s)` is the instantaneous brand conversion effect at time s and `lambda` is the decay rate.

---

## 6. Real Implementation Challenges

### 6.1 Sample Size Requirements

**Fundamental Constraint:** Uplift effects are typically small (1-5% relative lift), requiring large samples.

**Power Analysis for Uplift:**

For a two-sample comparison with binary outcome:

```
n = (z_{alpha/2} + z_beta)^2 * [p_T*(1-p_T) + p_C*(1-p_C)] / (p_T - p_C)^2
```

where:
- `z_{alpha/2}` = 1.96 for 95% significance
- `z_beta` = 0.84 for 80% power
- `p_T` = treatment conversion rate
- `p_C` = control conversion rate

**Example Calculation for Douyin:**

| Scenario | Baseline Rate (p_C) | Expected Uplift | Absolute Lift | Required n per arm |
|----------|-------------------|-----------------|---------------|-------------------|
| V4: Ad conversion | 2.0% | 10% relative | 0.2pp | 381,000 |
| V2: Purchase uplift | 0.5% | 5% relative | 0.025pp | 24,000,000 |
| V1: Engagement uplift | 15.0% | 3% relative | 0.45pp | 156,000 |
| V3: Category spend | 1.0% | 8% relative | 0.08pp | 3,000,000 |

**Key insight:** V2 (commerce stimulation from non-commercial content) requires the most data because the baseline conversion rate is low and the expected effect is small. This is why individual content-level V2 estimation requires pooling across user segments or using Bayesian shrinkage.

**CUPED Reduction:**

With CUPED using pre-exposure purchase history (typical correlation rho = 0.4):

```
n_CUPED = n_standard * (1 - rho^2) = n_standard * 0.84
```

This provides a ~16% reduction in required sample size.

**For CATE estimation (heterogeneous effects):**

Causal forest estimates require substantially more data than ATE estimates. Rule of thumb from Athey & Wager (2019):

```
n_CATE >= 10 * n_ATE * p_features
```

where `p_features` is the number of covariates used for heterogeneity detection.

### 6.2 SUTVA Violations (Interference Effects)

SUTVA is systematically violated on Douyin through multiple channels:

**Type 1: Social Spillover**

When user A watches and shares content, their friends (user B, C) are more likely to see it through social recommendations.

```
W_A = 1 --> Share --> Social_Feed_B --> W_B's exposure probability increases
```

User A's treatment status directly affects user B's potential outcomes.

**Type 2: Market-Level Effects**

If a content piece drives massive demand for a product, it may cause stockouts or price changes that affect all users.

**Type 3: Algorithm Feedback Loops**

Content that receives high engagement gets boosted in the algorithm, creating network-level treatment effects:

```
W_A = 1, High_Engagement_A --> Algo_Boost --> More users exposed --> W_B, W_C, ...
```

**Mitigation Strategies:**

**(a) Cluster Randomization:**

Instead of individual-level randomization, randomize at the cluster level (e.g., geographic regions, social network communities):

```
Treatment_effect = E[Y_bar_treated_clusters] - E[Y_bar_control_clusters]
SE = sqrt(Var(Y_bar_clusters) * (1/n_T + 1/n_C))
```

Design effect for cluster randomization:

```
DEFF = 1 + (m-1) * ICC
```

where m = cluster size and ICC = intra-cluster correlation coefficient.

**(b) Exposure Mapping (Aronow & Samii, 2017):**

Define each user's "effective treatment" as a function of their own treatment AND their neighbors' treatments:

```
D_i = f(W_i, W_N(i))
```

where N(i) is user i's social network neighborhood.

**(c) Two-Stage Randomized Design:**

Stage 1: Randomly assign clusters to saturation levels (0%, 50%, 100% content exposure)
Stage 2: Within each cluster, randomly assign individuals to treatment/control

This allows estimation of both direct effects and spillover effects:

```
Y_i = alpha + tau_direct * W_i + tau_spillover * W_bar_N(i) + epsilon_i
```

### 6.3 Time-Varying Treatment Effects (Content Value Decay)

Content value is not static -- it decays over time. The decay curve is crucial for content pricing.

**Adstock Model (Broadbent, 1979):**

```
Adstock_t = Exposure_t + lambda * Adstock_{t-1}
```

where `lambda in [0,1]` is the decay rate. The half-life is:

```
Half_Life = -log(2) / log(lambda)
```

**Content-Specific Decay Estimates:**

| Content Type | Typical Half-Life | Lambda |
|-------------|-------------------|--------|
| Trending/viral short video | 2-3 days | 0.70-0.80 |
| Product review content | 1-2 weeks | 0.90-0.95 |
| Brand story content | 3-4 weeks | 0.95-0.97 |
| Educational/tutorial | 4-8 weeks | 0.97-0.99 |
| Evergreen product demo | 8-12 weeks | 0.98-0.99 |

**Time-Varying CATE Estimation:**

Model the treatment effect as a function of time since exposure:

```
tau(x, t) = tau_0(x) * h(t)
```

where h(t) is the decay function. Common specifications:

**Exponential decay:**
```
h(t) = exp(-delta * t)
```

**Weibull decay (allows non-monotonic):**
```
h(t) = (delta * k) * (delta * t)^(k-1) * exp(-(delta*t)^k)
```

**Piecewise estimation:**

```
tau(x, t) = tau_0(x) * SUM_s [ beta_s * 1(t in [s, s+1)) ]
```

This non-parametrically estimates the decay curve at weekly intervals.

**Implementation:**

```python
# Estimate time-varying treatment effects using DML
for lag in range(0, 90, 7):  # Weekly windows up to 90 days
    outcome_window = purchase_within_window(lag, lag+7)
    dml = DoubleMLIRM(data_with_outcome=outcome_window, ...)
    dml.fit()
    tau_at_lag[lag] = dml.coef[0]

# Fit exponential decay to the estimated ATEs
from scipy.optimize import curve_fit
def exp_decay(t, tau_0, delta):
    return tau_0 * np.exp(-delta * t)

params, _ = curve_fit(exp_decay, lags, tau_at_lag)
half_life = np.log(2) / params[1]
```

### 6.4 Heterogeneous Treatment Effects Across User Segments

Not all users respond equally to content. Understanding this heterogeneity is essential for accurate content pricing.

**Key Heterogeneity Dimensions:**

1. **5A Stage:** A user at A1 (barely aware) responds differently to content than a user at A3 (actively considering)
2. **Purchase recency:** Recent buyers may be saturated; lapsed buyers may be re-activatable
3. **Content genre affinity:** Users with high affinity for the creator's style show larger effects
4. **Price sensitivity:** Content may shift purchase timing for price-sensitive users but not for affluent users
5. **Platform tenure:** New users are more susceptible to content influence

**Sorted Group Average Treatment Effects (GATES):**

```
1. Estimate tau_hat(X_i) using causal forest
2. Sort users by predicted CATE
3. Divide into quintiles Q1 (lowest) to Q5 (highest)
4. Estimate group-level ATE for each quintile using DML

GATE_q = E[Y(1) - Y(0) | tau_hat(X_i) in Quintile_q]
```

**Calibration Test (Chernozhukov et al., 2018):**

```
Y_i = alpha + beta_1 * (tau_hat(X_i) - tau_bar) + beta_2 * tau_hat(X_i) * (W_i - e_hat(X_i)) + epsilon_i
```

- `beta_2 > 0` indicates the CATE estimates have genuine predictive power for treatment heterogeneity
- `beta_1 = 1` indicates the heterogeneity is well-calibrated

**Pricing Implications:**

Content value varies by user segment. The content marketplace should price content based on the expected CATE distribution:

```
Content_Price_j = SUM_segments [ P(segment_s) * tau_j(segment_s) * monetization_value_s * expected_reach_s ]
```

---

## 7. Integrated System Architecture

### 7.1 The Content Value Estimation Pipeline

```
[Data Layer]
    |
    +-- User behavior logs (impressions, clicks, watch time, engagement)
    +-- Commerce logs (add-to-cart, purchase, GMV, return)
    +-- Algorithm logs (recommendation scores, propensity scores)
    +-- Creator metadata (IP features, content history)
    +-- Product catalog (category, price, attributes)
    |
    v
[Feature Engineering Layer]
    |
    +-- User features: 5A stage, purchase history, content preferences, demographics
    +-- Content features: creator ID, category, format, text/audio/video embeddings
    +-- Context features: time of day, day of week, seasonal events
    +-- Propensity features: algorithm recommendation probability (direct from algo logs)
    |
    v
[Causal Estimation Layer]
    |
    +-- V1 Engine: IPS/DR estimator for counterfactual attention
    +-- V2 Engine: Causal Forest + DML for commerce stimulation
    +-- V3 Engine: Panel DML for IP x Category effects
    +-- V4 Engine: Holdout experiments + Ghost Ads for brand conversion
    |
    v
[Aggregation Layer]
    |
    +-- Content-level value: V_total(j) = w1*V1(j) + w2*V2(j) + w3*V3(j) + w4*V4(j)
    +-- Decay adjustment: V_total(j, t) = V_total(j) * h(t - t_publish)
    +-- Confidence intervals: 95% CI for each value estimate
    +-- Heterogeneity report: CATE distribution across user segments
    |
    v
[Marketplace Layer]
    |
    +-- Real-time content pricing based on estimated V_total
    +-- Creator compensation based on incremental value generated
    +-- Brand billing based on causal iROAS
    +-- Content recommendation optimization incorporating value estimates
```

### 7.2 Validation Framework

**A/B Test Calibration:**

Regularly run holdout experiments to validate observational estimates:

```
Calibration_Score = Corr(tau_hat_observational, tau_hat_experimental)
Bias = E[tau_hat_observational - tau_hat_experimental]
```

**Placebo Tests:**

1. **Pre-treatment placebo:** Estimate "treatment effect" using pre-treatment outcomes. Should be zero.
2. **Random content placebo:** Randomly assign content labels and estimate effects. Should be zero.
3. **Known-effect validation:** For V4 (ad-driven), compare observational estimates to experimental truth.

**Sensitivity Analysis:**

Rosenbaum bounds to assess robustness to unmeasured confounding:

```
For Gamma = 1 (no hidden bias) to Gamma = 3:
    Compute p-value bounds for the treatment effect
    Report the Gamma at which significance is lost
```

If the treatment effect remains significant up to Gamma = 2, it means an unobserved confounder would need to double the odds of treatment to invalidate the finding.

---

## 8. References

### Foundational Causal Inference

- **Holland, P.W.** (1986). "Statistics and Causal Inference." *Journal of the American Statistical Association*, 81(396), 945-960.
- **Rubin, D.B.** (1974). "Estimating Causal Effects of Treatments in Randomized and Nonrandomized Studies." *Journal of Educational Psychology*, 66(5), 688-701.
- **Pearl, J.** (2009). *Causality: Models, Reasoning, and Inference*. 2nd Edition. Cambridge University Press.
- **Imbens, G.W. & Rubin, D.B.** (2015). *Causal Inference for Statistics, Social, and Biomedical Sciences*. Cambridge University Press.
- **Hernan, M.A. & Robins, J.M.** (2020). *Causal Inference: What If*. Chapman & Hall/CRC.

### Meta-Learners and Uplift Modeling

- **Kunzel, S.R., Sekhon, J.S., Bickel, P.J., & Yu, B.** (2019). "Metalearners for Estimating Heterogeneous Treatment Effects using Machine Learning." *Proceedings of the National Academy of Sciences*, 116(10), 4156-4165.
- **Nie, X. & Wager, S.** (2021). "Quasi-Oracle Estimation of Heterogeneous Treatment Effects." *Biometrika*, 108(2), 299-319.
- **Gutierrez, P. & Gerardy, J.Y.** (2017). "Causal Inference and Uplift Modelling: A Review of the Literature." *Proceedings of Machine Learning Research*, 67, 1-13.

### Causal Forests and Generalized Random Forests

- **Athey, S. & Imbens, G.W.** (2016). "Recursive Partitioning for Heterogeneous Causal Effects." *Proceedings of the National Academy of Sciences*, 113(27), 7353-7360.
- **Wager, S. & Athey, S.** (2018). "Estimation and Inference of Heterogeneous Treatment Effects using Random Forests." *Journal of the American Statistical Association*, 113(523), 1228-1242.
- **Athey, S., Tibshirani, J., & Wager, S.** (2019). "Generalized Random Forests." *Annals of Statistics*, 47(2), 1148-1178.

### Double/Debiased Machine Learning

- **Chernozhukov, V., Chetverikov, D., Demirer, M., Duflo, E., Hansen, C., Newey, W., & Robins, J.** (2018). "Double/Debiased Machine Learning for Treatment and Structural Parameters." *The Econometrics Journal*, 21(1), C1-C68.
- **Chernozhukov, V., Demirer, M., Duflo, E., & Fernandez-Val, I.** (2018). "Generic Machine Learning Inference on Heterogeneous Treatment Effects in Randomized Experiments." *NBER Working Paper No. 24678*.

### Attribution and Marketing Measurement

- **Shapley, L.S.** (1953). "A Value for n-Person Games." In *Contributions to the Theory of Games*, Vol. 2, Princeton University Press.
- **Johnson, G.A., Lewis, R.A., & Nubbemeyer, E.I.** (2017). "Ghost Ads: Improving the Economics of Measuring Online Ad Effectiveness." *Journal of Marketing Research*, 54(6), 867-884.
- **Dalessandro, B., Perlich, C., Stitelman, O., & Provost, F.** (2012). "Causally Motivated Attribution for Online Advertising." *Proceedings of the 6th International Workshop on Data Mining for Online Advertising*.

### Difference-in-Differences

- **Callaway, B. & Sant'Anna, P.H.C.** (2021). "Difference-in-Differences with Multiple Time Periods." *Journal of Econometrics*, 225(2), 200-230.
- **de Chaisemartin, C. & D'Haultfoeuille, X.** (2020). "Two-Way Fixed Effects Estimators with Heterogeneous Treatment Effects." *American Economic Review*, 110(9), 2964-2996.

### Regression Discontinuity

- **Imbens, G.W. & Kalyanaraman, K.** (2012). "Optimal Bandwidth Choice for the Regression Discontinuity Estimator." *Review of Economic Studies*, 79(3), 933-959.
- **Calonico, S., Cattaneo, M.D., & Titiunik, R.** (2014). "Robust Nonparametric Confidence Intervals for Regression-Discontinuity Designs." *Econometrica*, 82(6), 2295-2326.

### Instrumental Variables

- **Angrist, J.D. & Pischke, J.S.** (2009). *Mostly Harmless Econometrics*. Princeton University Press.
- **Staiger, D. & Stock, J.H.** (1997). "Instrumental Variables Regression with Weak Instruments." *Econometrica*, 65(3), 557-586.

### Interference and SUTVA

- **Aronow, P.M. & Samii, C.** (2017). "Estimating Average Causal Effects under General Interference." *Annals of Applied Statistics*, 11(4), 1912-1947.
- **Hudgens, M.G. & Halloran, M.E.** (2008). "Toward Causal Inference with Interference." *Journal of the American Statistical Association*, 103(482), 832-842.

### Advertising Decay and Time-Varying Effects

- **Broadbent, S.** (1979). "One-Way TV Advertisements Work." *Journal of the Market Research Society*, 21(3), 139-166.
- **Naik, P.A., Mantrala, M.K., & Sawyer, A.G.** (2005). "Planning Media Schedules in the Presence of Dynamic Advertising Quality." *Marketing Science*, 24(1), 35-49.

### Sensitivity Analysis

- **Rosenbaum, P.R.** (2002). *Observational Studies*. 2nd Edition. Springer.

### Experimentation Infrastructure

- **Deng, A., Xu, Y., Kohavi, R., & Walker, T.** (2013). "Improving the Sensitivity of Online Controlled Experiments by Utilizing Pre-Experiment Data." *Proceedings of the 6th ACM Conference on Web Search and Data Mining*, 123-132.
- **Kohavi, R., Tang, D., & Xu, Y.** (2020). *Trustworthy Online Controlled Experiments: A Practical Guide to A/B Testing*. Cambridge University Press.

### Douyin/ByteDance Platform

- **Volcengine DataTester Documentation**: https://www.volcengine.com/docs/56651/783739
- **Ocean Engine O-5A Methodology**: Published by Roland Berger in collaboration with Juliangliangyin, 2022.
- **Juliangxingtu Content Value Measurement Guide**: https://www.xingtu.cn/help-center/demander/140449

---

*Document prepared for the Content Value Quantification and Trading Marketplace technical architecture. All formulas and model specifications are designed for implementation with full Douyin data access.*
