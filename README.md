# Adaptive Hysteresis for Financial Signals Handbook


Reference & inspiration from personal research : 


## Schmitt-Trigger Trading Logic

Graham L. Giller uses the **Schmitt trigger** as an engineering analogy for a transaction-cost-aware trading strategy.

A conventional threshold strategy changes position whenever the alpha crosses a single decision boundary. When the alpha is noisy and remains close to that boundary, the strategy can repeatedly reverse its position, generating whipsaw and unnecessary transaction costs.

Giller describes this behavior as:

> “jitter at the threshold”

In electronics, the equivalent problem is addressed using:

> “the Schmitt trigger, a circuit element with hysteresis in its response function.”

The trading equivalent is a **state-dependent, two-barrier holding rule**: the position changes only after the alpha crosses a sufficiently strong opposing threshold.

### Maximum-net-profit rule

Let:

* (\alpha_t) be the current expected return, or alpha;
* (h_t) be the current position;
* (L) be the position limit;
* (\kappa) be transaction cost expressed in alpha or return units.

For the risk-limited maximum-net-profit problem, Giller obtains:

[
\hat b_1=+\kappa,
\qquad
\hat b_2=-\kappa.
]

The barriers are therefore separated by (2\kappa).

The resulting holding function is:

[
h_t=
\begin{cases}
+L, & \alpha_t\geq+\kappa,[3pt]
-L, & \alpha_t\leq-\kappa,[3pt]
h_{t-1}, & -\kappa<\alpha_t<+\kappa.
\end{cases}
]

In practical terms:

* Go long only when the alpha reaches the upper barrier.
* Go short only when the alpha reaches the lower barrier.
* Inside the barrier interval, retain the previous position.
* Do not reverse merely because the alpha changes sign.

```python
def schmitt_position(
    alpha: float,
    previous_position: int,
    upper_barrier: float,
    lower_barrier: float,
) -> int:
    """Return -1, 0, or +1 using a hysteretic trading rule."""

    if lower_barrier >= upper_barrier:
        raise ValueError("lower_barrier must be below upper_barrier")

    if alpha >= upper_barrier:
        return 1

    if alpha <= lower_barrier:
        return -1

    return previous_position
```

### Why hysteresis matters

Consider a long position entered when the alpha is slightly positive. A single-threshold strategy may immediately close or reverse that position when the next noisy alpha estimate becomes slightly negative.

The Schmitt-trigger rule does not react to that small change. The long position is retained until the alpha crosses the lower barrier and becomes sufficiently negative to justify paying the reversal cost.

The strategy therefore has memory:

[
h_t=h(\alpha_t,h_{t-1}),
]

rather than being determined only by the current alpha:

[
h_t=h(\alpha_t).
]

This hysteresis band acts as a turnover filter. It ignores small changes in the signal that are unlikely to justify their execution costs.

### Risk aversion versus transaction costs

Giller separates the effects of risk aversion and transaction costs:

* **Risk aversion** moves the trade-entry threshold away from zero, requiring a stronger signal-to-noise ratio before taking risk.
* **Transaction costs** split the threshold into two state-dependent barriers, creating hysteresis and reducing unnecessary churn.

His conclusion is that transaction costs split the barriers, introducing hysteresis into the response function.

### Sharpe-maximizing heuristic

For the more general Sharpe-maximizing problem, Giller proposes the approximate standardized barriers:

[
\hat B_1(K)=\zeta(K)+K,
\qquad
\hat B_2(K)=\zeta(K)-K,
]

where:

[
K=\frac{\kappa}{\omega_t},
]

(\omega_t) is the scale of the alpha distribution, and (\zeta(K)) is a slowly varying function that can sometimes be approximated by:

[
\zeta(K)\approx0.6.
]

In return units:

[
\hat b_1(\kappa)=\omega_t\zeta(0)+\kappa,
\qquad
\hat b_2(\kappa)=\omega_t\zeta(0)-\kappa.
]

The midpoint represents the underlying risk or selectivity threshold:

[
\frac{\hat b_1+\hat b_2}{2}
===========================

\omega_t\zeta(0),
]

while the separation represents the transaction-cost hysteresis:

[
\hat b_1-\hat b_2=2\kappa.
]

These equations are presented as heuristic approximations rather than universal constants; the optimal barriers depend on the alpha distribution and the trader’s effective costs.

### Implementation principle

The Schmitt-trigger logic should be applied to an estimated **alpha**, not mechanically to raw market prices:

```text
Market data
    ↓
Causal alpha estimate
    ↓
State-dependent two-barrier rule
    ↓
Target position
    ↓
Execution and risk controls
```

The central design principle is:

> Change position only after a decisive threshold crossing; otherwise preserve the current state inside a transaction-cost-dependent hysteresis band.

### Reference

Giller, Graham L. *Essays on Trading Strategy*. World Scientific, 2024. Essay 5, “Barrier Trading Algorithms,” particularly §§5.4.9–5.4.10, pp. 117–119, and §5.5, p. 127.
