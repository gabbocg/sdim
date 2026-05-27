# Get started with sdim

## Overview

**sdim** implements five dimension-reduction methods used in asset
pricing and macroeconomic forecasting. They all turn a large set of
candidate predictors or factor proxies into a small number of factors,
but they differ in *what* they ask the factors to do.

| Function | Method | What it optimises | Reference |
|----|----|----|----|
| [`pca_est()`](https://gabbocg.github.io/sdim/reference/pca_est.md) | Principal Component Analysis | Maximises own-variance of the predictor matrix (target ignored). | He et al. (2023) |
| [`pls_est()`](https://gabbocg.github.io/sdim/reference/pls_est.md) | Partial Least Squares | Maximises predictive covariance with the target. | He et al. (2023) |
| [`rra_est()`](https://gabbocg.github.io/sdim/reference/rra_est.md) | Reduced-Rank Approach | Finds the rank-\\K\\ subspace of proxies that prices the target. | He et al. (2023) |
| [`spca_est()`](https://gabbocg.github.io/sdim/reference/spca_est.md) | Scaled PCA | PCA after scaling each predictor by its OLS slope on the target. | Huang et al. (2022) |
| [`ipca_est()`](https://gabbocg.github.io/sdim/reference/ipca_est.md) | Instrumented PCA | Latent factors with loadings linear in observed characteristics. | Kelly, Pruitt & Su (2019) |

All five estimators return S3 objects with
[`print()`](https://rdrr.io/r/base/print.html),
[`summary()`](https://rdrr.io/r/base/summary.html), and
[`predict()`](https://rdrr.io/r/stats/predict.html) methods, so the same
workflow applies regardless of which method you choose.

## Quick start

We start with a synthetic panel: a \\T \times L\\ matrix of factor
proxies `X` and a \\T \times N\\ matrix of returns `ret`.

``` r

library(sdim)

set.seed(42)
X   <- matrix(rnorm(200 * 20), 200, 20)
ret <- matrix(rnorm(200 * 30) / 100, 200, 30)
```

### PCA, PLS, and RRA

These three methods share the same interface: a multivariate target
(here, returns) and a matrix of factor proxies `X`. They differ only in
the objective used to pick the \\K\\ linear combinations of `X`:

``` r

fit_pca <- pca_est(target = ret, X = X, nfac = 3)
fit_pls <- pls_est(target = ret, X = X, nfac = 3)
fit_rra <- rra_est(target = ret, X = X, nfac = 3)

print(fit_rra)
#> <sdim_fit [rra]>
#>  Observations : 200 
#>  Predictors   : 20 
#>  Factors      : 3
```

### Scaled PCA

sPCA takes a *univariate* target. It runs `y` on each column of `X`
separately, multiplies each column by its OLS slope, and then takes the
principal components of the rescaled matrix. The rescaling assigns more
weight to columns that move with the target and damps the rest.

When `length(target) < nrow(X)`, the first `length(target)` rows are
used for the scaling regression while *all* rows are used for factor
extraction. That asymmetric setup is what supports the
predictive-alignment trick (\\y\_{t+1} \sim X\_{i,t}\\) commonly used in
out-of-sample forecasting.

``` r

y <- rnorm(200)

fit_spca <- spca_est(target = y, X = X, nfac = 3)
print(fit_spca)
#> <sdim_spca>
#>  Observations : 200 
#>  Predictors   : 20 
#>  Factors      : 3
```

### IPCA

IPCA expects panel data with observable, time-varying characteristics
per asset. Latent factors are extracted under the restriction that
loadings are linear in those characteristics, so the characteristics
play the role of *instruments* for otherwise-unobservable conditional
betas.

The input shapes are a \\T \times N\\ return matrix and a \\T \times N
\times L\\ characteristics array:

``` r

TT      <- 120
K       <- 50
n_chars <- 6

ret_panel <- matrix(rnorm(TT * K) / 100, TT, K)
Z         <- array(rnorm(TT * K * n_chars), dim = c(TT, K, n_chars))

fit_ipca <- ipca_est(ret_panel, Z, nfac = 3)
#> Warning in ipca_als_cpp(ret_list, z_list, K = nfac, max_iter = max_iter, :
#> ipca_est: ALS did not converge in 100 iterations
print(fit_ipca)
#> <sdim_fit [ipca]>
#>  Observations    : 120 
#>  Characteristics : 6 
#>  Factors         : 3 
#>  Factor mean     : zero
```

## Prediction

[`predict()`](https://rdrr.io/r/stats/predict.html) projects new
predictors onto the loadings that were estimated during fitting. For
sPCA it also reapplies the training-window standardisation and scaling,
so out-of-sample factors are constructed on the same footing as the
in-sample ones:

``` r

X_new <- matrix(rnorm(5 * 20), 5, 20)

# PCA projection
F_new <- predict(fit_pca, X_new)
dim(F_new)
#> [1] 5 3

# sPCA projection (standardises newdata using training parameters)
F_spca_new <- predict(fit_spca, X_new)
dim(F_spca_new)
#> [1] 5 3
```

## Factor evaluation

[`eval_factors()`](https://gabbocg.github.io/sdim/reference/eval_factors.md)
reports the diagnostics defined in He et al. (2023, §2.4) —
root-mean-square pricing error, total adjusted \\R^2\\, the
maximum-Sharpe ratio attainable from the factors, and the average
absolute correlation between factor mimicking portfolios:

``` r

eval_factors(ret = ret, factors = fit_rra$factors)
#> Factor Evaluation
#> ---------------------------------------- 
#>  Portfolios       30
#>  Factors          3
#> 
#> Performance (He et al., 2023, §2.4)
#> ---------------------------------------- 
#>  RMSPE              0.9875  (%)
#>  Total adj-R²       2.9593  (%)
#>  SR                 0.0522
#>  A2R                0.9443
```

## Bundled datasets

The package ships several datasets used in the replication vignettes:

- **`grunfeld`**: Grunfeld (1958) investment panel (11 firms, 20 years).
  Used as the IPCA validation example.
- **`he2023_*`**: Seven datasets from He et al. (2023) — factor proxies
  and portfolio returns for replicating the paper’s pricing exercise.
- **`huang2022_macro`**: \\720 \times 123\\ matrix of transformed
  FRED-MD predictors used in Huang et al. (2022).
- **`huang2022_ip`**: 720-vector of monthly IP growth (the forecast
  target of the Huang et al. (2022) out-of-sample exercise).

See
[`vignette("ipca-grunfeld")`](https://gabbocg.github.io/sdim/articles/ipca-grunfeld.md),
[`vignette("he2023-table3")`](https://gabbocg.github.io/sdim/articles/he2023-table3.md),
and
[`vignette("huang2022-table4")`](https://gabbocg.github.io/sdim/articles/huang2022-table4.md)
for fully worked replications.
