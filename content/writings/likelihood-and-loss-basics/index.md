---
title: "From Likelihood to Loss"
date: 2026-10-12
description: "Why Common Loss Functions Take Their Form"
---


**Preliminaries and Notation**

\(P\): true distribution. \(p_\theta\): model probability mass/density.
Observations are i.i.d. from \(P\); all logs are natural.

**Maximum likelihood estimation (MLE)**

$$
J(\theta)=\mathbb E_{X\sim P}[\log p_\theta(X)]
\approx \widehat J(\theta)=\frac1n\sum_i\log p_\theta(x_i).
$$

$$
\hat\theta
=\arg\max_\theta\widehat J(\theta)
=\arg\max_\theta\prod_i p_\theta(x_i).
$$

Population objective: minimize \(D_{\mathrm{KL}}(P\|p_\theta)\).
The \(1/n\) is an empirical averaging weight, not a true input probability.

**Conditional MLE**

$$
\mathbb E_{(X,Y)\sim P}[\log p_\theta(Y\mid X)]
\approx \frac1n\sum_i\log p_\theta(y_i\mid x_i).
$$

Maximizing the sample average is equivalent to maximizing
\(\prod_i p_\theta(y_i\mid x_i)\).

For \(q_\theta(x,y)=P_X(x)p_\theta(y\mid x)\), drop
\(\log P_X(x)\) because it is independent of \(\theta\).
A learned \(p_\theta(x)\) generally cannot be dropped from joint likelihood.

**Gaussian regression: negative log-likelihood (NLL)**

\(Y_i\mid X_i=x_i\sim\mathcal N(\mu_i,v_i)\), where
\(\mu_i=\mu_\theta(x_i)\), variance \(v_i>0\), residual \(r_i=y_i-\mu_i\).

$$
\mathcal L
=-\frac1n\sum_i\log p_\theta(y_i\mid x_i)
=\frac1{2n}\sum_i
\left[\log(2\pi)+\log v_i+\frac{r_i^2}{v_i}\right].
$$

Define \(\mathrm{MSE}=\frac1n\sum_i r_i^2\).
Equivalent objectives omit parameter-independent additive constants
and positive multiplicative constants.

| Variance assumption | Minimize |
|---|---|
| Fixed, shared \(v\) (homoscedastic) | \(\mathrm{MSE}\) |
| Fixed, varying \(v_i\) (heteroscedastic) | \(\frac1n\sum_i r_i^2/v_i\) (weighted MSE) |
| Learned, shared \(v\) | \(\log v+\mathrm{MSE}/v\) |
| Learned, varying \(v_i=v_\theta(x_i)\) | \(\frac1n\sum_i[\log v_i+r_i^2/v_i]\) |