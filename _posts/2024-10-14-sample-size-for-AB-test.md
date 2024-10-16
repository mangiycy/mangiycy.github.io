---
title: Minimum sample size for A/B Test
date: 2024-10-14 10:00:00 -0400
author: ycy
categories: [Data Science]
tags: [找工]
math: true
---

<!-- When it comes interview questions for a DS position, A/B test would be a common topic. -->

## Denotation

+ $ N $, the minimum sample size
+ $ \alpha $, the confidence level (probability of Type I error) 
+ $ \beta $, the max probability of Type II error allowed[^power]
+ $ X_c, X_t $, the random variables of the metrics of control group and treatment group
+ $ x_c, x_t $, the observation of control group and treatment group metrics
+ $ X_d = X_t - X_c $, $ x_d = x_t - x_c $

## The Logic

In a normal hypothesis test, what we do is to calculate a statistic and either compare it with the critical value or check its corresponding p-value.

## Formula

Suppose we want to conduct an A/B test to determine if a specific metric (measured at the individual user level) improves with a certain treatment. After running the experiment, we collect data for both the control group $ (x_c) $​ and the treatment group $ (x_t) $​, representing the metric values for each group respectively. To evaluate the effect of the treatment, we will compare the **means** of the two groups. 

Let $ \mu_c $ and $ \mu_t $ denote the expectation of the metrics for control and treatment respectively. We want to test:

$$
\begin{gather*}
    H_0: \mu_t - \mu_c \iff \mu_t - \mu_c = 0 \\
    H_A: \mu_t > \mu_c \iff \mu_t - \mu_c = \delta > 0
\end{gather*}
$$

According to the [Central Limited Theorem][CTL], the sample mean would converge to a normal distribution. Assume the two groups are independent,

$$
\begin{gather*}
    \bar X_c \sim \mathcal{N}\left(\mu_c, \frac{\sigma^2}{N}\right),
    \bar X_t \sim \mathcal{N}\left(\mu_t, \frac{\sigma^2}{N}\right) \\
    \bar X_d = \bar X_c - \bar X_t \sim \mathcal{N}\left(\mu_c - \mu_t, \frac{2 \sigma^2}{N}\right)
\end{gather*}
$$

We constrain Type II error probability to be less than $ \beta $, 

$$
\begin{align*}
    \text{Type II Error} &= P(\text{Accept } H_0 \mid H_A \text{ is True}) \\
    &= P\left(\frac{\bar x_d - 0}{\sqrt{2 \sigma^2 / N}} < z_\alpha \,\Big|\, \mu_t - \mu_c = \delta > 0\right) \\
    &= P\left(\frac{\bar x_d - \delta}{\sqrt{2 \sigma^2 / N}} < z_\alpha - \frac{\delta}{\sqrt{2 \sigma^2 / N}}\right) \\
    &= \Phi \left(z_\alpha - \frac{\delta}{\sqrt{2 \sigma^2 / N}}\right)
    \leqslant \beta
\end{align*}
$$

With the properties of standard normal distribution, we have

$$
\begin{equation*}
    z_\alpha - \frac{\delta}{\sqrt{2 \sigma^2 / N}} \leqslant z_{1-\beta}
    \,\Longrightarrow\,
    N \geqslant \frac{2 \sigma^2 (z_\alpha + z_\beta)^2}{\delta^2}
\end{equation*}
$$

In real world, $ \delta $ is usually replaced by a pre-set number called **minimum detectable effect**.


[^power]: Note that $ 1 - \beta $ is the power of test. Some would say to set a test power, which is the same.

[CTL]: https://en.wikipedia.org/wiki/Central_limit_theorem