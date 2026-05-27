# Replicating Huang et al. (2022)

This vignette reproduces the industrial-production (IP) growth column of
Table 4 in Huang, Jiang, Li, Tong, and Zhou (2022), “Scaled PCA: A New
Approach to Dimension Reduction,” *Management Science*, 68(3). The
exercise forecasts one-month-ahead IP growth using factors extracted
from 123 transformed FRED-MD macro variables, and compares two
dimension-reduction routes:

- **PCA**, the standard principal-component factors of the predictor
  matrix, which ignore the forecast target.
- **sPCA**, the *scaled* variant introduced by Huang et al. (2022).
  Before extracting components, each predictor is scaled by the OLS
  slope from regressing the target on that predictor. Predictors with
  stronger predictive content for the target receive larger weight; the
  irrelevant ones get shrunk toward zero. PCA on the scaled predictors
  therefore concentrates forecasting-relevant information into the first
  few factors.

The reported metric is the out-of-sample \\R^2\\, \\R^2\_{OS}\\ (%),
relative to an AR benchmark with SIC-selected lag order.

## Data

The package ships the authors’ original data:

- `huang2022_macro`: a \\720 \times 123\\ matrix of transformed FRED-MD
  predictors (January 1960 to December 2019).
- `huang2022_ip`: a 720-vector of monthly IP growth (log-differences of
  the IP index).

``` r

library(sdim)

data(huang2022_macro)
data(huang2022_ip)

dim(huang2022_macro)
#> [1] 720 123
length(huang2022_ip)
#> [1] 720
```

## Methodology

The forecasting exercise is an expanding-window pseudo out-of-sample
experiment that mirrors the paper:

- **Initial estimation window**: January 1960 to December 1984 (300
  observations).
- **Out-of-sample period**: January 1985 to December 2019 (420
  forecasts).
- **Benchmark**: AR with lag order chosen by SIC (maximum lag 1).
- **Factor models**: ARDL with the AR lags plus one lag of the PCA or
  sPCA factors.

For sPCA, the scaling regression uses the *predictive* alignment
\\y\_{t+1} \sim X\_{i,t}\\ — that is, today’s predictor is regressed on
next month’s target. To dampen extreme slopes the absolute scaling
coefficients are winsorised at the 90th percentile, matching the
authors’ MATLAB code.
[`spca_est()`](https://gabbocg.github.io/sdim/reference/spca_est.md)
supports this directly: when `length(target) < nrow(X)`, the first
`length(target)` rows are used for the scaling regression while *all*
rows of `X` are standardised and used for factor extraction. This is
what makes the predictive alignment possible without losing observations
from the factor panel.

## Out-of-sample loop

The function below runs the recursive forecast. At each step it:

1.  selects an AR lag order via
    [`select_ar_lag_sic()`](https://gabbocg.github.io/sdim/reference/select_ar_lag_sic.md)
    and produces the AR-only forecast;
2.  extracts up to `nfac_max` PCA factors from the standardised
    predictors and up to `nfac_max` sPCA factors with the predictive
    alignment and winsorisation just described;
3.  builds an ARDL forecast for each number of factors using both factor
    sets, with the AR coefficients estimated jointly via
    [`estimate_ardl_multi()`](https://gabbocg.github.io/sdim/reference/estimate_ardl_multi.md).

The loop runs ~420 iterations and takes a few minutes, so it is set to
`eval = FALSE` here:

``` r

run_oos <- function(y, Z, h = 1, p_max = 1, nfac_max = 5) {

  TT <- length(y)
  M  <- (1984 - 1959) * 12
  NN <- TT - M

  FC_AR    <- rep(NA, NN - (h - 1))
  FC_PCA   <- matrix(NA, NN - (h - 1), nfac_max)
  FC_sPCA  <- matrix(NA, NN - (h - 1), nfac_max)
  actual_y <- rep(NA, NN - (h - 1))

  for (n in seq_len(NN - (h - 1))) {

    actual_y[n] <- mean(y[(M + n):(M + n + h - 1)])

    y_n  <- y[1:(M + n - 1)]
    Z_n  <- Z[1:(M + n - 1), ]
    Zs_n <- oos_standardize(Z_n)
    T_n  <- length(y_n)

    y_n_h <- vapply(
      seq_len(T_n - (h - 1)),
      function(t) mean(y_n[t:(t + h - 1)]),
      numeric(1)
    )

    # --- AR benchmark with SIC lag selection ---
    p_ar <- select_ar_lag_sic(y_n, h, p_max)

    if (p_ar > 0L) {

      ar_out   <- estimate_ar_res(y_n, h, p_ar)
      y_n_last <- rev(y_n[(T_n - p_ar + 1):T_n])
      FC_AR[n] <- sum(c(1, y_n_last) * ar_out$a_hat)

    } else {

      FC_AR[n] <- mean(y_n)

    }

    # --- PCA factors ---
    pca_fit <- pca_est(X = Zs_n, nfac = nfac_max)
    z_pc_n  <- predict(pca_fit, Zs_n)

    # --- sPCA factors (predictive alignment + winsorization) ---
    spca_fit <- spca_est(
      target       = y_n_h[2:length(y_n_h)],
      X            = Z_n,
      nfac         = nfac_max,
      winsorize    = TRUE,
      winsor_probs = c(0, 90)
    )

    z_trans_n <- predict(spca_fit, Z_n)

    # --- ARDL forecast for each number of factors ---
    for (cc in seq_len(nfac_max)) {

      for (jj in 1:2) {

        z_f <- if (jj == 1) {

          z_pc_n[, 1:cc, drop = FALSE]

        } else {

          z_trans_n[, 1:cc, drop = FALSE]

        }

        p_ardl <- c(p_ar, 1)

        if (p_ar > 0L) {

          c_hat    <- estimate_ardl_multi(y_n, z_f, h, p_ardl)
          y_n_last <- rev(y_n[(T_n - p_ar + 1):T_n])
          fc       <- sum(c(1, y_n_last, z_f[T_n, ]) * c_hat)

        } else {

          dep   <- y_n_h[2:length(y_n_h)]
          reg   <- cbind(1, z_f[1:(length(y_n_h) - 1 - (h - 1)), 1:cc])
          c_hat <- lm.fit(x = reg, y = dep)$coefficients
          fc    <- sum(c(1, z_f[T_n, 1:cc]) * c_hat)

        }

        if (jj == 1) FC_PCA[n, cc]  <- fc
        if (jj == 2) FC_sPCA[n, cc] <- fc

      }

    }

  }

  # R²_OS for each number of factors
  r2_pca <- r2_spca <- numeric(nfac_max)
  sse_ar <- sum((actual_y - FC_AR)^2)

  for (cc in seq_len(nfac_max)) {

    r2_pca[cc]  <- 100 * (1 - sum((actual_y - FC_PCA[, cc])^2)  / sse_ar)
    r2_spca[cc] <- 100 * (1 - sum((actual_y - FC_sPCA[, cc])^2) / sse_ar)

  }

  data.frame(K = seq_len(nfac_max), PCA = round(r2_pca, 2), sPCA = round(r2_spca, 2))

}

# Run
res <- run_oos(huang2022_ip, huang2022_macro, h = 1, p_max = 1, nfac_max = 5)
print(res)
```

## Results

Running the code above produces:

      K   PCA  sPCA
      1  8.97  9.65
      2  8.06 10.68
      3  8.22 11.09
      4  7.99 11.97
      5  7.88 13.17

With five factors, PCA reaches \\R^2\_{OS}\\ = **7.88%** and sPCA
reaches \\R^2\_{OS}\\ = **13.17%** — both matching the paper to two
decimals. sPCA dominates PCA at every factor count, which is precisely
the result the paper highlights: when many candidate predictors are weak
or irrelevant, weighting each one by its target-predictive slope lets
PCA focus on the components that actually carry forecasting power.

## Key `spca_est()` features used

1.  **Predictive alignment**. Passing a `target` that is one observation
    shorter than `X` (i.e. \\T-1\\ versus \\T\\) makes the scaling
    regression pair \\X\_{i,t}\\ with \\y\_{t+1}\\, while factors are
    still extracted from the full \\T\\-row predictor matrix.

2.  **Winsorisation**. `winsorize = TRUE` with `winsor_probs = c(0, 90)`
    caps the absolute scaling slopes at their 90th percentile,
    reproducing the trimming used in the authors’ MATLAB code.

3.  **[`predict()`](https://rdrr.io/r/stats/predict.html)**. Projects
    the training `X` onto the estimated sPCA loadings using the
    training-window standardisation and scaling, so in-sample and
    out-of-sample factor draws are constructed consistently.

## References

Huang, D., Jiang, F., Li, K., Tong, G., and Zhou, G. (2022). Scaled PCA:
A New Approach to Dimension Reduction. *Management Science*, 68(3),
1678–1695.
