
# performance <img src='man/figures/logo.png' align="right" height="139" />

[![publication](https://img.shields.io/badge/Cite-Unpublished-yellow)](https://github.com/easystats/performance/blob/master/inst/CITATION)
[![downloads](http://cranlogs.r-pkg.org/badges/performance)](https://cran.r-project.org/package=performance)
[![total](https://cranlogs.r-pkg.org/badges/grand-total/performance)](https://cranlogs.r-pkg.org/)

***Test if your model is a good model\!***

A crucial aspect when building regression models is to evaluate the
quality of modelfit. It is important to investigate how well models fit
to the data and which fit indices to report. Functions to create
diagnostic plots or to compute fit measures do exist, however, mostly
spread over different packages. There is no unique and consistent
approach to assess the model quality for different kind of models.

The primary goal of the **performance** package is to fill this gap and
to provide utilities for computing **indices of model quality** and
**goodness of fit**. These include measures like r-squared (R2), root
mean squared error (RMSE) or intraclass correlation coefficient (ICC) ,
but also functions to check (mixed) models for overdispersion,
zero-inflation, convergence or singularity.

## Installation

[![CRAN](http://www.r-pkg.org/badges/version/performance)](https://cran.r-project.org/package=performance)
[![R
check](https://github.com/easystats/performance/workflows/R-check/badge.svg?branch=master)](https://github.com/easystats/performance/actions)

Run the following to install the stable release of **performance** from
CRAN:

``` r
install.packages("performance")
```

Or this one to install the latest development version:

``` r
install.packages("remotes")
remotes::install_github("easystats/performance")
```

## Citation

<!-- TO DO: once accepted, replace psyarxiv doi with joss doi -->

To cite performance in publications use:

``` r
citation("performance")
#> 
#>   Lüdecke et al., (2021). Assessment, Testing and
#>   Comparison of Statistical Models using R. Journal
#>   of Open Source Software, 6(59), 3112.
#>   https://doi.org/10.31234/osf.io/vtq8f
#> 
#> A BibTeX entry for LaTeX users is
#> 
#>   @Article{,
#>     title = {Assessment, Testing and Comparison of Statistical Models using R},
#>     author = {Daniel Lüdecke and Mattan S. Ben-Shachar and Indrajeet Patil and Philip Waggoner and Dominique Makowski},
#>     year = {2021},
#>     journal = {Journal of Open Source Software},
#>     volume = {6},
#>     number = {59},
#>     pages = {3112},
#>     doi = {10.31234/osf.io/vtq8f},
#>   }
```

## Documentation

[![Documentation](https://img.shields.io/badge/documentation-performance-orange.svg?colorB=E91E63)](https://easystats.github.io/performance/)
[![Blog](https://img.shields.io/badge/blog-easystats-orange.svg?colorB=FF9800)](https://easystats.github.io/blog/posts/)
[![Features](https://img.shields.io/badge/features-performance-orange.svg?colorB=2196F3)](https://easystats.github.io/performance/reference/index.html)

There is a nice introduction into the package on
[youtube](https://www.youtube.com/watch?v=EPIxQ5i5oxs).

## The *performance* workflow

<img src="man/figures/figure_workflow.png" width="75%" />

### Assessing model quality

#### R-squared

**performance** has a generic `r2()` function, which computes the
r-squared for many different models, including mixed effects and
Bayesian regression models.

`r2()` returns a list containing values related to the “most
appropriate” r-squared for the given model.

``` r
model <- lm(mpg ~ wt + cyl, data = mtcars)
r2(model)
#> # R2 for Linear Regression
#>        R2: 0.830
#>   adj. R2: 0.819

model <- glm(am ~ wt + cyl, data = mtcars, family = binomial)
r2(model)
#> # R2 for Logistic Regression
#>   Tjur's R2: 0.705

library(MASS)
data(housing)
model <- polr(Sat ~ Infl + Type + Cont, weights = Freq, data = housing)
r2(model)
#>   Nagelkerke's R2: 0.108
```

The different R-squared measures can also be accessed directly via
functions like `r2_bayes()`, `r2_coxsnell()` or `r2_nagelkerke()` (see a
full list of functions
[here](https://easystats.github.io/performance/reference/index.html#section-r-functions)).

For mixed models, the *conditional* and *marginal* R-squared are
returned. The *marginal R-squared* considers only the variance of the
fixed effects and indicates how much of the model’s variance is
explained by the fixed effects part only. The *conditional R-squared*
takes both the fixed and random effects into account and indicates how
much of the model’s variance is explained by the “complete” model.

For frequentist mixed models, `r2()` (resp. `r2_nakagawa()`) computes
the *mean* random effect variances, thus `r2()` is also appropriate for
mixed models with more complex random effects structures, like random
slopes or nested random effects (Johnson 2014; Nakagawa, Johnson, and
Schielzeth 2017).

``` r
set.seed(123)
library(rstanarm)

model <- stan_glmer(Petal.Length ~ Petal.Width + (1 | Species), data = iris, cores = 4)

r2(model)
#> # Bayesian R2 with Standard Error
#> 
#>   Conditional R2: 0.953 (89% CI [0.944, 0.962])
#>      Marginal R2: 0.825 (89% CI [0.749, 0.889])

library(lme4)
model <- lmer(Reaction ~ Days + (1 + Days | Subject), data = sleepstudy)
r2(model)
#> # R2 for Mixed Models
#> 
#>   Conditional R2: 0.799
#>      Marginal R2: 0.279
```

#### Intraclass Correlation Coefficient (ICC)

Similar to R-squared, the ICC provides information on the explained
variance and can be interpreted as “the proportion of the variance
explained by the grouping structure in the population” (Hox 2010).

`icc()` calculates the ICC for various mixed model objects, including
`stanreg` models.

``` r
library(lme4)
model <- lmer(Reaction ~ Days + (1 + Days | Subject), data = sleepstudy)
icc(model)
#> # Intraclass Correlation Coefficient
#> 
#>      Adjusted ICC: 0.722
#>   Conditional ICC: 0.521
```

…and models of class `brmsfit`.

``` r
library(brms)
set.seed(123)
model <- brm(mpg ~ wt + (1 | cyl) + (1 + wt | gear), data = mtcars)
```

``` r
icc(model)
#> # Intraclass Correlation Coefficient
#> 
#>      Adjusted ICC: 0.930
#>   Conditional ICC: 0.771
```

### Model diagnostics

#### Check for overdispersion

Overdispersion occurs when the observed variance in the data is higher
than the expected variance from the model assumption (for Poisson,
variance roughly equals the mean of an outcome).
`check_overdispersion()` checks if a count model (including mixed
models) is overdispersed or not.

``` r
library(glmmTMB)
data(Salamanders)
model <- glm(count ~ spp + mined, family = poisson, data = Salamanders)
check_overdispersion(model)
#> # Overdispersion test
#> 
#>        dispersion ratio =    2.946
#>   Pearson's Chi-Squared = 1873.710
#>                 p-value =  < 0.001
```

Overdispersion can be fixed by either modelling the dispersion parameter
(not possible with all packages), or by choosing a different
distributional family (like Quasi-Poisson, or negative binomial, see
(Gelman and Hill 2007)).

#### Check for zero-inflation

Zero-inflation (in (Quasi-)Poisson models) is indicated when the amount
of observed zeros is larger than the amount of predicted zeros, so the
model is *underfitting* zeros. In such cases, it is recommended to use
negative binomial or zero-inflated models.

Use `check_zeroinflation()` to check if zero-inflation is present in the
fitted model.

``` r
model <- glm(count ~ spp + mined, family = poisson, data = Salamanders)
check_zeroinflation(model)
#> # Check for zero-inflation
#> 
#>    Observed zeros: 387
#>   Predicted zeros: 298
#>             Ratio: 0.77
```

#### Check for singular model fits

A “singular” model fit means that some dimensions of the
variance-covariance matrix have been estimated as exactly zero. This
often occurs for mixed models with overly complex random effects
structures.

`check_singularity()` checks mixed models (of class `lme`, `merMod`,
`glmmTMB` or `MixMod`) for singularity, and returns `TRUE` if the model
fit is singular.

``` r
library(lme4)
data(sleepstudy)

# prepare data
set.seed(123)
sleepstudy$mygrp <- sample(1:5, size = 180, replace = TRUE)
sleepstudy$mysubgrp <- NA
for (i in 1:5) {
    filter_group <- sleepstudy$mygrp == i
    sleepstudy$mysubgrp[filter_group] <- sample(1:30, size = sum(filter_group), replace = TRUE)
}

# fit strange model
model <- lmer(Reaction ~ Days + (1 | mygrp/mysubgrp) + (1 | Subject), data = sleepstudy)

check_singularity(model)
#> [1] TRUE
```

Remedies to cure issues with singular fits can be found
[here](https://easystats.github.io/performance/reference/check_singularity.html).

#### Check for heteroskedasticity

Linear models assume constant error variance (homoskedasticity).

The `check_heteroscedasticity()` functions assess if this assumption has
been violated:

``` r
data(cars)
model <- lm(dist ~ speed, data = cars)

check_heteroscedasticity(model)
#> Warning: Heteroscedasticity (non-constant error variance) detected (p = 0.031).
```

#### Comprehensive visualization of model checks

**performance** provides many functions to check model assumptions, like
`check_collinearity()`, `check_normality()` or
`check_heteroscedasticity()`. To get a comprehensive check, use
`check_model()`.

``` r
# defining a model
model <- lm(mpg ~ wt + am + gear + vs * cyl, data = mtcars)

# checking model assumptions
check_model(model)
```

<img src="man/figures/unnamed-chunk-15-1.png" width="100%" />

### Model performance summaries

`model_performance()` computes indices of model performance for
regression models. Depending on the model object, typical indices might
be r-squared, AIC, BIC, RMSE, ICC or LOOIC.

#### Linear model

``` r
m1 <- lm(mpg ~ wt + cyl, data = mtcars)
model_performance(m1)
#> # Indices of model performance
#> 
#> AIC     |     BIC |    R2 | R2 (adj.) |  RMSE | Sigma
#> -----------------------------------------------------
#> 156.010 | 161.873 | 0.830 |     0.819 | 2.444 | 2.568
```

#### Logistic regression

``` r
m2 <- glm(vs ~ wt + mpg, data = mtcars, family = "binomial")
model_performance(m2)
#> # Indices of model performance
#> 
#> AIC    |    BIC | Tjur's R2 |  RMSE | Sigma | Log_loss | Score_log | Score_spherical |   PCP
#> --------------------------------------------------------------------------------------------
#> 31.298 | 35.695 |     0.478 | 0.359 | 0.934 |    0.395 |   -14.903 |           0.095 | 0.743
```

#### Linear mixed model

``` r
library(lme4)
m3 <- lmer(Reaction ~ Days + (1 + Days | Subject), data = sleepstudy)
model_performance(m3)
#> # Indices of model performance
#> 
#> AIC      |      BIC | R2 (cond.) | R2 (marg.) |   ICC |   RMSE |  Sigma
#> -----------------------------------------------------------------------
#> 1755.628 | 1774.786 |      0.799 |      0.279 | 0.722 | 23.438 | 25.592
```

### Models comparison

The `compare_performance()` function can be used to compare the
performance and quality of several models (including models of different
types).

``` r
counts <- c(18, 17, 15, 20, 10, 20, 25, 13, 12)
outcome <- gl(3, 1, 9)
treatment <- gl(3, 3)
m4 <- glm(counts ~ outcome + treatment, family = poisson())

compare_performance(m1, m2, m3, m4)
#> # Comparison of Model Performance Indices
#> 
#> Name |   Model |      AIC |      BIC |   RMSE |  Sigma | Score_log | Score_spherical |    R2 | R2 (adj.) | Tjur's R2 | Log_loss |   PCP | R2 (cond.) | R2 (marg.) |   ICC | Nagelkerke's R2
#> -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
#> m1   |      lm |  156.010 |  161.873 |  2.444 |  2.568 |           |                 | 0.830 |     0.819 |           |          |       |            |            |       |                
#> m2   |     glm |   31.298 |   35.695 |  0.359 |  0.934 |   -14.903 |           0.095 |       |           |     0.478 |    0.395 | 0.743 |            |            |       |                
#> m3   | lmerMod | 1755.628 | 1774.786 | 23.438 | 25.592 |           |                 |       |           |           |          |       |      0.799 |      0.279 | 0.722 |                
#> m4   |     glm |   56.761 |   57.747 |  3.043 |  1.132 |    -2.598 |           0.324 |       |           |           |          |       |            |            |       |           0.657
```

#### General index of model performance

One can also easily compute and a [**composite
index**](https://easystats.github.io/performance/reference/compare_performance.html#details)
of model performance and sort the models from the best one to the worse.

``` r
compare_performance(m1, m2, m3, m4, rank = TRUE)
#> # Comparison of Model Performance Indices
#> 
#> Name |   Model |      AIC |      BIC |   RMSE |  Sigma | Performance-Score
#> --------------------------------------------------------------------------
#> m2   |     glm |   31.298 |   35.695 |  0.359 |  0.934 |           100.00%
#> m4   |     glm |   56.761 |   57.747 |  3.043 |  1.132 |            96.21%
#> m1   |      lm |  156.010 |  161.873 |  2.444 |  2.568 |            92.46%
#> m3   | lmerMod | 1755.628 | 1774.786 | 23.438 | 25.592 |             0.00%
```

#### Visualisation of indices of models’ performance

Finally, we provide convenient visualisation (the `see` package must be
installed).

``` r
plot(compare_performance(m1, m2, m4, rank = TRUE))
```

<img src="man/figures/unnamed-chunk-21-1.png" width="100%" />

### Testing models

`test_performance()` (and `test_bf`, its Bayesian sister) carries out
the most relevant and appropriate tests based on the input (for
instance, whether the models are nested or not).

``` r
set.seed(123)
data(iris)

lm1 <- lm(Sepal.Length ~ Species, data = iris)
lm2 <- lm(Sepal.Length ~ Species + Petal.Length, data = iris)
lm3 <- lm(Sepal.Length ~ Species * Sepal.Width, data = iris)
lm4 <- lm(Sepal.Length ~ Species * Sepal.Width + Petal.Length + Petal.Width, data = iris)

compare_performance(lm1, lm2, lm3, lm4)
#> # Comparison of Model Performance Indices
#> 
#> Name | Model |     AIC |     BIC |    R2 | R2 (adj.) |  RMSE | Sigma
#> --------------------------------------------------------------------
#> lm1  |    lm | 231.452 | 243.494 | 0.619 |     0.614 | 0.510 | 0.515
#> lm2  |    lm | 106.233 | 121.286 | 0.837 |     0.833 | 0.333 | 0.338
#> lm3  |    lm | 187.092 | 208.167 | 0.727 |     0.718 | 0.431 | 0.440
#> lm4  |    lm |  78.797 | 105.892 | 0.871 |     0.865 | 0.296 | 0.305

test_bf(lm1, lm2, lm3, lm4)
#> Bayes Factors for Model Comparison
#> 
#>     Model                                                  BF
#> [2] Species + Petal.Length                             > 1000
#> [3] Species * Sepal.Width                              > 1000
#> [4] Species * Sepal.Width + Petal.Length + Petal.Width > 1000
#> 
#> * Against Denominator: [1] Species
#> *   Bayes Factor Type: BIC approximation
```

# Code of Conduct

Please note that the performance project is released with a [Contributor
Code of
Conduct](https://contributor-covenant.org/version/2/0/CODE_OF_CONDUCT.html).
By contributing to this project, you agree to abide by its terms.

# Contributing

We are happy to receive bug reports, suggestions, questions, and (most
of all) contributions to fix problems and add features.

Please follow contributing guidelines mentioned here:

<https://easystats.github.io/performance/CONTRIBUTING.html>

## References

<div id="refs" class="references hanging-indent">

<div id="ref-gelman_data_2007">

Gelman, Andrew, and Jennifer Hill. 2007. *Data Analysis Using Regression
and Multilevel/Hierarchical Models*. Analytical Methods for Social
Research. Cambridge ; New York: Cambridge University Press.

</div>

<div id="ref-hox_multilevel_2010">

Hox, J. J. 2010. *Multilevel Analysis: Techniques and Applications*. 2nd
ed. Quantitative Methodology Series. New York: Routledge.

</div>

<div id="ref-johnson_extension_2014">

Johnson, Paul C. D. 2014. “Extension of Nakagawa & Schielzeth’s R2 GLMM
to Random Slopes Models.” Edited by Robert B. O’Hara. *Methods in
Ecology and Evolution* 5 (9): 944–46.
<https://doi.org/10.1111/2041-210X.12225>.

</div>

<div id="ref-nakagawa_coefficient_2017">

Nakagawa, Shinichi, Paul C. D. Johnson, and Holger Schielzeth. 2017.
“The Coefficient of Determination R2 and Intra-Class Correlation
Coefficient from Generalized Linear Mixed-Effects Models Revisited and
Expanded.” *Journal of the Royal Society Interface* 14 (134): 20170213.
<https://doi.org/10.1098/rsif.2017.0213>.

</div>

</div>
