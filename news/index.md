# Changelog

## sdim (development version)

- Reworded the package title to comply with CRAN policy (no longer
  starts with “An R Package for…”).

## sdim 0.1.0

CRAN release: 2026-07-11

- Initial CRAN release.
- Five factor extraction methods for asset pricing and macroeconomic
  forecasting:
  - Principal Component Analysis
    ([`pca_est()`](https://gabbocg.github.io/sdim/reference/pca_est.md))
  - Partial Least Squares
    ([`pls_est()`](https://gabbocg.github.io/sdim/reference/pls_est.md))
  - Scaled PCA
    ([`spca_est()`](https://gabbocg.github.io/sdim/reference/spca_est.md)),
    Huang, Jiang, Li, Tong, and Zhou (2022) <doi:10.1287/mnsc.2021.4020>
  - Reduced-Rank Approach
    ([`rra_est()`](https://gabbocg.github.io/sdim/reference/rra_est.md)),
    He, Huang, Li, and Zhou (2023) <doi:10.1287/mnsc.2022.4563>
  - Instrumented PCA
    ([`ipca_est()`](https://gabbocg.github.io/sdim/reference/ipca_est.md)),
    Kelly, Pruitt, and Su (2019) <doi:10.1016/j.jfineco.2019.05.001>
- Unified `sdim_fit()` wrapper with
  [`print()`](https://rdrr.io/r/base/print.html),
  [`summary()`](https://rdrr.io/r/base/summary.html),
  [`plot()`](https://rdrr.io/r/graphics/plot.default.html), and
  [`predict()`](https://rdrr.io/r/stats/predict.html) methods.
- Factor evaluation utility
  ([`eval_factors()`](https://gabbocg.github.io/sdim/reference/eval_factors.md))
  reporting R-squared, trace R-squared, and canonical correlations
  against benchmark factors.
- Out-of-sample forecasting helpers:
  [`oos_standardize()`](https://gabbocg.github.io/sdim/reference/oos_standardize.md),
  [`select_ar_lag_sic()`](https://gabbocg.github.io/sdim/reference/select_ar_lag_sic.md),
  [`estimate_ar_res()`](https://gabbocg.github.io/sdim/reference/estimate_ar_res.md),
  [`estimate_ardl_multi()`](https://gabbocg.github.io/sdim/reference/estimate_ardl_multi.md).
- Bundled datasets replicating results from the source papers:
  `grunfeld`, `he2023_factors`, `he2023_ff17vw`, `he2023_ff30vw`,
  `he2023_ff48vw`, `he2023_ff48ew`, `he2023_ff5`, `he2023_dacheng202`,
  `huang2022_ip`, `huang2022_macro`.
- Four vignettes: package overview, Huang et al. (2022) Table 4
  replication, He et al. (2023) Table 3 replication, and IPCA on the
  Grunfeld panel.
