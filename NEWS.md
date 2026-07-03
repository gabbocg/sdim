# sdim 0.1.0

* Initial CRAN release.
* Five factor extraction methods for asset pricing and macroeconomic
  forecasting:
    - Principal Component Analysis (`pca_est()`)
    - Partial Least Squares (`pls_est()`)
    - Scaled PCA (`spca_est()`), Huang, Jiang, Li, Tong, and Zhou (2022)
      <doi:10.1287/mnsc.2021.4020>
    - Reduced-Rank Approach (`rra_est()`), He, Huang, Li, and Zhou (2023)
      <doi:10.1287/mnsc.2022.4563>
    - Instrumented PCA (`ipca_est()`), Kelly, Pruitt, and Su (2019)
      <doi:10.1016/j.jfineco.2019.05.001>
* Unified `sdim_fit()` wrapper with `print()`, `summary()`, `plot()`, and
  `predict()` methods.
* Factor evaluation utility (`eval_factors()`) reporting R-squared, trace
  R-squared, and canonical correlations against benchmark factors.
* Out-of-sample forecasting helpers: `oos_standardize()`,
  `select_ar_lag_sic()`, `estimate_ar_res()`, `estimate_ardl_multi()`.
* Bundled datasets replicating results from the source papers:
  `grunfeld`, `he2023_factors`, `he2023_ff17vw`, `he2023_ff30vw`,
  `he2023_ff48vw`, `he2023_ff48ew`, `he2023_ff5`, `he2023_dacheng202`,
  `huang2022_ip`, `huang2022_macro`.
* Four vignettes: package overview, Huang et al. (2022) Table 4
  replication, He et al. (2023) Table 3 replication, and IPCA on the
  Grunfeld panel.
