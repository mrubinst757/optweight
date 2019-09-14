
<!-- README.md is generated from README.Rmd. Please edit that file -->

# optweight

[![CRAN\_Status\_Badge](http://r-pkg.org/badges/version-last-release/optweight?color=0047ab)](https://cran.r-project.org/package=optweight)
[![CRAN\_Downloads\_Badge](http://cranlogs.r-pkg.org/badges/optweight?color=0047ab)](https://cran.r-project.org/package=optweight)

`optweight` contains functions to estimate weights that balance
treatments to given balance thresholds. It solves a quadratic
programming problem to minimize an objective function of the weights
using `solve_osqp()` in the `osqp` package. This is the method described
in Zubizarreta (2015). `optweight` extends the method to multinomial,
continuous, and longitudinal treatments and provides a simple user
interface and compatibility with the `cobalt` package.

Below is an example of estimating weights with `optweight` and assessing
balance on the covariates with `cobalt`.

``` r
devtools::install_github("ngreifer/optweight") #development version
library("optweight")
library("cobalt")
```

``` r
data("lalonde")

#Estimate weights
ow <- optweight(treat ~ age + educ + race + nodegree + married +
                  re74 + re75 + I(re74 == 0) + I(re75 == 0),
                data = lalonde, estimand = "ATT", tols = .01)
ow
```

    An optweight object
     - number of obs.: 614
     - sampling weights: none
     - treatment: 2-category
     - estimand: ATT (focal: 1)
     - covariates: age, educ, race, nodegree, married, re74, re75, I(re74 == 0), I(re75 == 0)

``` r
summary(ow)
```

    Summary of weights:
    
    - Weight ranges:
            Min                                  Max
    treated   1     ||                        1.0000
    control   0 |---------------------------| 7.4319
    
    - Units with 5 greatest weights by group:
                                               
                  2      3      4      5      6
     treated      1      1      1      1      1
                608    574    559    573    303
     control 7.2344 7.3161 7.4058 7.4058 7.4319
    
            Coef of Var Mean Abs Dev
    treated      0.0000       0.0000
    control      1.9019       1.3719
    overall      1.5897       0.9585
    
    - Effective Sample Sizes:
               Control Treated
    Unweighted 429.000     185
    Weighted    92.917     185

``` r
bal.tab(ow)
```

    Call
     optweight(formula = treat ~ age + educ + race + nodegree + married + 
        re74 + re75 + I(re74 == 0) + I(re75 == 0), data = lalonde, 
        tols = 0.01, estimand = "ATT")
    
    Balance Measures
                    Type Diff.Adj
    age          Contin.     0.01
    educ         Contin.     0.01
    race_black    Binary     0.01
    race_hispan   Binary     0.00
    race_white    Binary    -0.01
    nodegree      Binary     0.01
    married       Binary    -0.01
    re74         Contin.     0.01
    re75         Contin.     0.01
    I(re74 == 0)  Binary     0.01
    I(re75 == 0)  Binary     0.01
    
    Effective sample sizes
               Control Treated
    Unadjusted 429.000     185
    Adjusted    92.917     185

``` r
#Estimate a treatment effect
library("jtools")
summ(lm(re78 ~ treat, data = lalonde, weights = ow$weights),
     confint = TRUE, robust = TRUE, model.fit = FALSE, 
     model.info = FALSE)
```

<table class="table table-striped table-hover table-condensed table-responsive" style="width: auto !important; margin-left: auto; margin-right: auto;">

<thead>

<tr>

<th style="text-align:left;">

</th>

<th style="text-align:right;">

Est.

</th>

<th style="text-align:right;">

2.5%

</th>

<th style="text-align:right;">

97.5%

</th>

<th style="text-align:right;">

t val.

</th>

<th style="text-align:right;">

p

</th>

</tr>

</thead>

<tbody>

<tr>

<td style="text-align:left;font-weight: bold;">

(Intercept)

</td>

<td style="text-align:right;">

5342.94

</td>

<td style="text-align:right;">

4058.75

</td>

<td style="text-align:right;">

6627.13

</td>

<td style="text-align:right;">

8.17

</td>

<td style="text-align:right;">

0.00

</td>

</tr>

<tr>

<td style="text-align:left;font-weight: bold;">

treat

</td>

<td style="text-align:right;">

1006.20

</td>

<td style="text-align:right;">

\-710.34

</td>

<td style="text-align:right;">

2722.74

</td>

<td style="text-align:right;">

1.15

</td>

<td style="text-align:right;">

0.25

</td>

</tr>

</tbody>

<tfoot>

<tr>

<td style="padding: 0; border: 0;" colspan="100%">

<sup></sup> Standard errors: Robust, type = HC3

</td>

</tr>

</tfoot>

</table>

The lower-level function `optweight.fit` operates on the covariates and
treatment variables directly.

In addition to estimating balancing weights for estimating treatment
effects, `optweight` can estimate sampling weights for generalizing an
estimate to a new target population defined by covariate moments using
the function `optweight.svy`.
