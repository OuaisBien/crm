#  Merton’s Model in Credit Risk Modelling with Importance Sampling
## Overview

This project Implements a Level 2 Importance Sampling simulation to explicitly model discrete, catastrophic default scenarios. Standard regulatory models (Basel ASRF) assume portfolios are infinitely granular. This underestimates tail risk for "lumpy" portfolios where idiosyncratic defaults of large borrowers do not diversify away.

## Analysis of the Result
The simulation reveals a significant **Granularity Add-on of $11.6 million**, representing
an 11% increase over the baseline capital requirement.
This gap exists because the ”Smooth” model essentially draws a line through the
average of the loss distribution. In a concentrated portfolio, the actual loss distribution
is multimodal (jagged), containing discrete scenarios where specific large obligors default.
Level 2 IS successfully samples these discrete ”jumps,” proving that the assumption of
infinite granularity is unsafe for this asset class.

<img src = "https://github.com/OuaisBien/crm/blob/main/comparison.png" width = 700>
