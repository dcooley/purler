
<!-- README.md is generated from README.Rmd. Please edit that file -->

# purler <img src="man/figures/logo.png" align="right" height=230/>

<!-- badges: start -->

![](https://img.shields.io/badge/cool-useless-green.svg)
![](https://img.shields.io/badge/dependencies-zero-blue.svg)
[![Lifecycle:
experimental](https://img.shields.io/badge/lifecycle-experimental-orange.svg)](https://www.tidyverse.org/lifecycle/#experimental)
[![R build
status](https://github.com/coolbutuseless/purler/workflows/R-CMD-check/badge.svg)](https://github.com/coolbutuseless/purler/actions)
<!-- badges: end -->

`purler` contains tools for run-length encoding vector data.

<img src="man/figures/example-annotated.png" style="border: 1px solid #000000;" />

#### Key features:

  - `NA` values are considered identical (unlike `base::rle()`)
  - Results returned as a `data.frame` (rather than a list), but still
    compatible with `base::inverse.rle()`
  - Faster\! Includes a C implementation for regular atomic vectors, and
    an R version compatible with every input `base::rle()` accepts.

## What’s in the box

  - `rlenc()` is C code for run-length encoding of raw, logical,
    integer, numeric and character vectors.
      - Groups `NA` values into a run (unlike `base::rle()`)
      - Returns a **data.frame** rather than a list
      - Returned object is compatible with `base::inverse.rle()`
      - Can be **10x faster** than `base::rle()`
  - `rlenc_compat()`
      - A pure R version of `rlenc()` which is compatible with all
        inputs that `base::rle()` accepts
  - `rleid()` returns an integer vector numbering the runs of identical
    values within a vector of numeric or character data. This is very
    similar to `data.table::rleid()`, execpt the `data.table()` version
    is much more configurable and flexible. This version is probably
    only useful if you wanted to avoid pulling in `data.table` as a
    dependency.

## Installation

You can install from [GitHub](https://github.com/coolbutuseless/purler)
with:

``` r
# install.package('remotes')
remotes::install_github('coolbutuseless/purler')
```

## ToDo

  - Long vector support in `rlenc()`

## `rlenc()` - run-length encoding output as a data.frame

``` r
input <- c(1, 1, 1, 2, 2, 8, 8, 8, 8, 8, NA, NA, NA, NA)

(result <- purler::rlenc(input))
```

    #>   lengths values start
    #> 1       3      1     1
    #> 2       2      2     4
    #> 3       5      8     6
    #> 4       4     NA    11

``` r
inverse.rle(result)
```

    #>  [1]  1  1  1  2  2  8  8  8  8  8 NA NA NA NA

## `rlenc()` benchmark

``` r
library(tidyr)
library(bench)
library(dplyr)
library(ggplot2)

N <- 1000
M <- 10

zz <- sample(seq_len(M), N, replace = TRUE)

res <- bench::mark(
  rle(zz),
  rlenc(zz),
  rlenc_compat(zz),
  check = FALSE
)

plot(res) + theme_bw()
```

<img src="man/figures/README-unnamed-chunk-5-1.png" width="100%" />

## Run-length encoding with NAs

In `base::rle()`, runs of NA values are *not* treated as a group.

All functions in `purler` do treat NAs as identical for the purpose of
creating groups

``` r
input <- c(1, 1, 2, NA, NA, NA, NA, 4, 4, 4)

base::rle(input)
```

    #> Run Length Encoding
    #>   lengths: int [1:7] 2 1 1 1 1 1 3
    #>   values : num [1:7] 1 2 NA NA NA NA 4

``` r
purler::rlenc_compat(input)
```

    #>   lengths values start
    #> 1       2      1     1
    #> 2       1      2     3
    #> 3       4     NA     4
    #> 4       3      4     8

``` r
purler::rlenc(input)
```

    #>   lengths values start
    #> 1       2      1     1
    #> 2       1      2     3
    #> 3       4     NA     4
    #> 4       3      4     8

``` r
purler::rlenc_id(input)
```

    #>  [1] 1 1 2 3 3 3 3 4 4 4

## Run-length encoded group ids

`rlenc_id()` numbers the runs of identical values in a numeric or
character vector.

For a more complete approach to this problem, see `data.table::rleid()`

``` r
input <- c(11, 11, 12, 12, 12, NA, NA, NA, NA)

rlenc_id(input)
```

    #> [1] 1 1 2 2 2 3 3 3 3

## Related Software

  - `base::rle()`
  - `data.table::rleid()`

## Acknowledgements

  - R Core for developing and maintaining the language.
  - CRAN maintainers, for patiently shepherding packages onto CRAN and
    maintaining the repository
