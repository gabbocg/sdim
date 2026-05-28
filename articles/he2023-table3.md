# Replicating He et al. (2023)

This vignette reproduces Table 3 of He, Huang, Li, and Zhou (2023). The
table compares four ways of summarising a large set of candidate factor
proxies down to a few risk factors that price the 48 Fama-French
value-weighted industry portfolios. Performance is measured by the total
adjusted \\R^2\\ (%) of the pricing regressions.

The four methods are:

- **FF**: use the Fama-French five factors plus momentum directly,
  taking the first \\K\\ of them.
- **PCA**: extract the \\K\\ principal components that maximise the
  variance of the factor-proxy matrix \\G\\.
- **PLS**: extract the \\K\\ components of \\G\\ that are most
  predictive of the test-asset returns.
- **RRA**: the reduced-rank approach of He et al. (2023). It searches
  for \\K\\ linear combinations of the proxies in \\G\\ whose loadings
  best explain the returns of the basis assets. Formally, the loading
  matrix in the regression of returns on proxies is restricted to rank
  \\K\\, and that restriction is what shrinks \\L \approx 70\\ proxies
  down to a handful of usable factors.

The motivation is empirical: many candidate factors have been proposed
in the literature, but most carry little incremental pricing information
once one accounts for the others. RRA is designed to find the small
linear subspace that retains the pricing-relevant content.

## Setup

The bundled `he2023_*` datasets come from the authors’ replication
package. The factor proxies in `he2023_factors` end twelve months
earlier than the portfolio panels, so we slice the rows to align them
and convert percentages to decimals. Returns are taken in excess of the
one-month Treasury bill rate `RF`:

``` r

library(sdim)

he2023_ff48 <- he2023_ff48vw[1:516, -1] / 100 - he2023_ff5$RF[127:642] / 100
G <- he2023_factors[1:516, -1] / 100

# First 6 columns of G are Fama-French 5 + momentum
f5 <- G[, 1:6]
```

## Replication

We loop over the same factor counts as the paper. For each \\K\\:

- the **FF** row uses the first \\K\\ of the six Fama-French/momentum
  factors directly (only defined for \\K \le 6\\);
- the **PCA**, **PLS**, and **RRA** rows fit the corresponding `*_est()`
  function on the full proxy set \\G\\, then pass the extracted factors
  to
  [`eval_factors()`](https://gabbocg.github.io/sdim/reference/eval_factors.md)
  to get the total adjusted \\R^2\\ on the test assets:

``` r

nfact   <- c(1, 3, 5, 6, 10)
methods <- c("FF", "PCA", "PLS", "RRA")

total_r2 <- matrix(NA, nrow = length(methods), ncol = length(nfact))
rownames(total_r2) <- methods
colnames(total_r2) <- paste(nfact, "factors")

for (j in seq_along(nfact)) {

  k <- nfact[j]

  if (k <= 6) {

    total_r2["FF", j] <- eval_factors(he2023_ff48, f5[, 1:k])["TotalR2"]

  }

  fit_pca <- pca_est(target = he2023_ff48, X = G, nfac = k)
  total_r2["PCA", j] <- eval_factors(he2023_ff48, fit_pca$factors)["TotalR2"]

  fit_pls <- pls_est(target = he2023_ff48, X = G, nfac = k)
  total_r2["PLS", j] <- eval_factors(he2023_ff48, fit_pls$factors)["TotalR2"]

  fit_rra <- rra_est(target = he2023_ff48, X = G, nfac = k)
  total_r2["RRA", j] <- eval_factors(he2023_ff48, fit_rra$factors)["TotalR2"]

}
```

## Results

``` r

round(total_r2, 2)
#>     1 factors 3 factors 5 factors 6 factors 10 factors
#> FF      51.39     55.57     57.77     58.34         NA
#> PCA     16.74     20.49     29.91     33.13      40.78
#> PLS     23.42     47.19     58.97     61.10      64.28
#> RRA     54.60     61.11     64.75     65.38      67.40
```

RRA delivers the highest total adjusted \\R^2\\ at every factor count.
This is the headline finding of He et al. (2023): once we look for
factors that are constructed *to price* the basis assets — rather than
factors that maximise own-variance (PCA) or predictive covariance with
returns one column at a time (PLS) — a small number of linear
combinations of the 70 proxies recovers nearly all the pricing
information.

## References

He, A., Huang, D., Li, J., and Zhou, G. (2023). Shrinking Factor
Dimension: A Reduced-Rank Approach. *Management Science*, 69(9),
5501–5522.
