### Reference

The use of the Schmitt trigger as a state-dependent trading decision mechanism is based on Graham L. Giller’s discussion of hysteresis in barrier trading algorithms.

Giller describes how a stochastic alpha fluctuating near a decision boundary can produce *“jitter at the threshold”* and repeated position reversals, causing a whipsaw effect and losses through transaction costs. He compares the solution to a Schmitt trigger: an electronic circuit whose hysteretic response prevents noisy signals from repeatedly switching the system state.

In Giller’s framework, transaction costs split a single decision threshold into separate entry and reversal barriers. The strategy retains its previous position while the alpha remains between those barriers and changes state only after the opposite barrier is crossed.

> **Source:** Graham L. Giller, *Essays on Trading Strategy*, World Scientific, 2024, Essay 5: “Barrier Trading Algorithms,” especially §5.4.9, “The Optimal Net Profit Maximizing Algorithm,” pp. 117–118; §5.4.10, “The Effect of Hysteresis in Signal Processing,” pp. 118–119; and §5.5, “Conclusions,” p. 127.

> **Implementation note:** The use of static, adaptive, symmetric, or asymmetric thresholds—and their estimation from ATR, realized volatility, rolling standard deviation, or other volatility features—is an extension of Giller’s core hysteresis concept. These specific threshold-construction methods are not prescribed directly in the cited sections.

