---
title: Bias-Variance Tradeoff
date: 2024-08-03 16:00:00 +0800
author: ycy
categories: [Data Science]
math: true
---

> Parameter tuning is a dark art in machine learning,
> the optimal parameters of a model can depend on many scenarios.
{: .prompt-tip }

Recently I've been exploring resources about [XGBoost][xgboost] and seeking good approaches to parameter tuning.
While going through the official docs of XGBoost library,
I found a [section][notes] about parameter tuning,
starting with *understanding bias-variance tradeoff*,
which inspired me to revisit some foundational concepts in ML.

The **Bias-Variance Tradeoff** is one of the basic but important concepts in ML or statistics.
It reveals that we are always facing the dilemma over
simultaneously minimizing the bias and variance.

## Bias-Variance Decomposition of MSE

Given a dataset $ \mathcal{D} = \\{(x_i, y_i) \mid y_i = f(x_i) + \varepsilon_i \\} $,
where $ \varepsilon $ is the noise with mean zero and variance $ \sigma_\varepsilon^2 $,
we do supervised learning and find an $ \hat f $.
The bias and the variance is then

$$
\DeclareMathOperator{\expt}{\mathbb{E}}
\begin{align*}
    \operatorname{Bias}\left[\hat f(x; \mathcal{D})\right]
    &= \expt\left[\hat f(x; \mathcal{D}) - f(x)\right]
    = \expt\left[\hat f(x; \mathcal{D})\right] - f(x) \\
    \operatorname{Var}\left[\hat f(x; \mathcal{D})\right]
    &= \expt\left[\left(\hat f(x; \mathcal{D}) 
    - \expt\left[\hat f(x; \mathcal{D})\right]\right)^2\right]
\end{align*}
$$

Let's abbreviate $ f = f(x) $ and $ \hat f = \hat f(x; \mathcal{D}) $.
The MSE of the model $ \hat f $ is

$$
\begin{align*}
    \text{MSE} = \expt\left[\left(y - \hat f \right)^2\right]
    = \expt\left[y^2\right] - 2 \expt\left[y \hat f\right] + \expt\left[\hat f^2\right]
\end{align*}
$$

Note that 

$$
\begin{align*}
    \expt\left[y^2\right] 
    &= \expt\left[f^2 + 2 \varepsilon f + \varepsilon^2\right]
    = f^2 + 0 + \operatorname{Var}[\varepsilon]
    = f^2 + \sigma_\varepsilon^2 \\
    \expt\left[y \hat f\right] 
    &= \expt\left[f\hat f + \varepsilon \hat f\right]
    = f \expt\left[\hat f\right] + 0
    = f \expt\left[\hat f\right]
    \\
    \expt\left[\hat f^2\right] 
    &= \operatorname{Var}\left[\hat f\right] + \expt\left[\hat f\right]^2
\end{align*}
$$

Plug in and we get

$$
\begin{align*}
    \text{MSE} 
    &= f^2 + \sigma_\varepsilon^2 - 2f \expt\left[\hat f\right] 
    + \operatorname{Var}\left[\hat f\right] + \expt\left[\hat f\right]^2 \\
    &= \underbrace{\left(f - \expt\left[\hat f\right]\right)^2}_{\text{Bias}^2} 
    + \underbrace{\operatorname{Var}\left[\hat f\right]}_{\text{Variance}} + \sigma_\varepsilon^2
\end{align*}
$$

This says, the expected generalized error of a model
(learned from a certain algorithm) is the sum of
squared bias, variance, and the noises in the dataset itself, also called irreducible error.

## Reduce Variance with Bagging?

The idea of Bagging (Bootstrap Aggregating) is:

1. Create subsets from the original dataset *with replacement* (bootstrap sampling)
2. Train models on each subset
3. Aggregate the predictions of all models to get final prediction

Suppose we draw subsets $ \mathcal{D}\_1, \mathcal{D}\_2, \cdots, \mathcal{D}\_n $ from $ \mathcal{D} $,
the final prediction is $ \hat f_B = \frac1n \sum_{i=1}^n \hat f(x; \mathcal{D}_i) $

$$
\operatorname{Var}\left[\hat f_B\right]
= \frac{1}{n^2} \operatorname{Var}\left[\sum_{i=1}^n\hat f(x; \mathcal{D}_i)\right]
$$

Since all subsets are identically drawn from $ \mathcal{D} $,
$ \hat f(x; \mathcal{D}_i) $ can be viewed as identically distributed, with variance $ \sigma^2 $.
We further assume they have pairwise correlation $ \rho > 0 $.

$$
\begin{align*}
    \operatorname{Var}\left[\hat f_B\right]
    &= \frac{1}{n^2} \sum_{i=1}^n \sum_{j=1}^n \operatorname{Cov} \left[\hat f(x; \mathcal{D}_i), \hat f(x; \mathcal{D}_j)\right] \\
    &= \frac{1}{n^2} \left[n \sigma^2 + n(n-1) \rho \sigma^2\right] \\
    &= \frac{\sigma^2}{n} \left[1 + (n-1) \rho \right] \\
    &= \rho \sigma^2 + \frac{1-\rho}{n} \sigma^2
    = \frac{(n-1)\rho+1}{n} \sigma^2
\end{align*}
$$

It is easy to see that $ \rho \sigma^2 \leqslant \operatorname{Var}\left[\hat f_B\right] \leqslant \sigma^2 $
and $ \operatorname{Var}\left[\hat f_B\right] \to \rho \sigma^2 $ when $ n \to \infty $.

However, this **does not guarantee the decrease of variance**, compared with training on the whole dataset.
Note that the variance of the model trained on $ \mathcal{D} $ may not equal to $ \sigma^2 $,
and $ \sigma^2 $ depends on how we draw subsets.
Therefore, maybe it's better to think bagging as a technique to produce more robust models. 

## References
+ [Wiki page](https://en.wikipedia.org/wiki/Bias%E2%80%93variance_tradeoff) of bias–variance tradeoff
+ Scott Fortmann-Roe, [Understanding the Bias-Variance Tradeoff](https://scott.fortmann-roe.com/docs/BiasVariance.html)
+ [What are the theoretical guarantees of bagging](https://stats.stackexchange.com/q/141186)

[xgboost]: https://github.com/dmlc/xgboost
[notes]: https://xgboost.readthedocs.io/en/stable/tutorials/param_tuning.html