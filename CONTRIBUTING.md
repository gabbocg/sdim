# Contributing to sdim

Thanks for your interest in improving **sdim**.

## Reporting bugs and asking questions

Open an issue at <https://github.com/gabbocg/sdim/issues>. For bugs,
please include a minimal reproducible example (ideally with
[reprex](https://reprex.tidyverse.org)), the output of
[`sessionInfo()`](https://rdrr.io/r/utils/sessionInfo.html), and the
version of sdim you are using.

## Suggesting features

Open an issue describing the method or feature, with a reference to the
paper it comes from if relevant. Please discuss larger changes in an
issue before opening a pull request.

## Pull requests

1.  Fork the repository and create a branch from `main`.
2.  Make your change. Follow the existing code style, and document new
    or changed functions with roxygen2 (`devtools::document()`).
3.  Add or update tests in `tests/testthat/`.
4.  Run `devtools::check()` and make sure it passes with no errors or
    warnings.
5.  Add a bullet to `NEWS.md` describing the change.
6.  Open the pull request against `main`.

## Code of conduct

Please be respectful and constructive in all interactions.
