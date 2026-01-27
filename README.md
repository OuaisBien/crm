#  Merton’s Model in Credit Risk Modelling with Importance Sampling
## Overview

Standard regulatory capital models (Basel II/III ASRF) assume portfolios are perfectly granular, diversifying away idiosyncratic risk. For portfolios with significant name concentration (e.g., interbank or large corporate exposures), this assumption fails, leading to an underestimation of tail risk. 
This project Implements a Level 2 Importance Sampling simulation to explicitly model discrete, catastrophic default scenarios. Standard regulatory models (Basel ASRF) assume portfolios are infinitely granular. This underestimates tail risk for *lumpy* portfolios where idiosyncratic defaults of large borrowers do not diversify away.
We define the **Granularity Adjustment (GA)** as the difference between the true VaR (accounting for lumpiness) and the smooth VaR, i.e.

$$\text{GA} = \text{VaR}_{\text{Lumpy}} - \text{VaR}_{\text{Smooth}}.$$

## Context: The Limits of the Asymptotic Model

The foundation of modern credit risk regulation is the Asymptotic Single Risk Factor (ASRF) model, derived from the Vasicek-Merton framework. The asset value of firm $i$, $A_i$, is driven by a common systemic factor $Z$ and an idiosyncratic factor $\epsilon_i$:

$$A_i = \sqrt{\rho} Z + \sqrt{1-\rho} \epsilon_i$$

Under the assumption of an infinitely granular portfolio ($N \to \infty$), the idiosyncratic noise $\epsilon_i$ diversifies away completely. The portfolio loss becomes deterministic conditional on $Z$. The regulatory capital is simply the Expected Loss conditional on a conservative realisation of the economy (e.g., $Z = -3.09$ for 99.9\% confidence).

### The Granularity Problem
For real-world portfolios containing large exposures, e.g., a *lumpy* portfolio of 50 large banks, the law of large numbers does not fully apply. Idiosyncratic events ($\epsilon_i$) do not cancel out; a single default can cause a discrete jump in losses. 


## Methodology: Two-Level Importance Sampling
Consider a synthetic *lumpy* portfolio with $N=50$ obligors, exposures following a Pareto distribution (Top 3 names = 12.9\% of total EAD) and $\rho = 0.2$ correlation. The default probability of companies is assumed to be low. Here, it is assumed to be i.i.d and uniform in $[0.001, 0.01]$.
### Level 1: Macro Shift of the Systemic Driver
The dominant driver of portfolio default is the economy $Z$. We shift the mean of $Z$ to sample directly from *crisis* scenarios.
<ul>
  <li><b>Original</b>: $Z \sim N(0, 1)$</li>
  <li><b>Shifted</b>: $Z^* \sim N(\mu, 1)$, where $\mu \approx -2.4$ (Optimised for target VaR).</li>
</ul>
This captures only the *expected* surge in defaults.

## Level 2: Exponential Twisting of the Idiosyncratic Noise
To capture the *lumpiness*, we must simulate discrete default events $1_i \in \{0, 1\}$. Given a macro state $z$, the conditional default probability is $p_i(z)$. We *exponentially twist* this to a higher probability $q_i(z)$ to force rare idiosyncratic events:

$$q_{i}(z, \theta) = \frac{p_{i}(z)e^{\theta}}{1 + p_{i}(z)(e^{\theta}-1)}$$

where $\theta > 1$ is the twist factor, and then apply the correction

$$W_{\epsilon} = \prod_{i=1}^{N} \left(\frac{p_i}{q_i}\right)^{1_i} \left(\frac{1-p_i}{1-q_i}\right)^{1-1_i}.$$

## Analysis of the Result
The simulation reveals a significant **Granularity Add-on of $11.4 million**, representing an 11% increase over the baseline capital requirement.
This gap exists because the *Smooth* model essentially draws a line through the average of the loss distribution. In a concentrated portfolio, the actual loss distribution is multimodal (jagged), containing discrete scenarios where specific large obligors default. Level 2 IS successfully samples these discrete *jumps*, proving that the assumption of infinite granularity is unsafe for this asset class.

### Numerical Comparison
   
| Metric	| Level 1 (Smooth) | Level 2 (Lumpy) | Difference |
|--------|--------|--------|--------|
| 99.9% VaR | $32,431,428 | $43,827,802 | +$11,396,374 |
| Interpretation | Regulatory Formula | Economic Capital| GA |
<img src = "https://github.com/OuaisBien/crm/blob/main/comparison.png" width = 700>


Ignoring the discrete nature of idiosyncratic risk (Level 2) leads to a material underestimation of tail risk. For internal capital adequacy (ICAAP), reliance on the smooth ASRF model would leave the institution undercapitalised against name concentration risk.

