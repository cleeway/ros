Regression and Other Stories: Sample size simulation
================
Andrew Gelman, Jennifer Hill, Aki Vehtari
2021-04-20

-   [16 Design and sample size
    decisions](#16-design-and-sample-size-decisions)
    -   [16.4 Interactions are harder to estimate than main
        effects](#164-interactions-are-harder-to-estimate-than-main-effects)
        -   [Understanding the problem by simulating regressions in
            R](#understanding-the-problem-by-simulating-regressions-in-r)

Tidyverse version by Bill Behrman.

Sample size simulation. See Chapter 16 in Regression and Other Stories.

------------------------------------------------------------------------

``` r
# Packages
library(tidyverse)
library(rstanarm)

# Parameters
  # Seed
SEED <- 660
  # Common code
file_common <- here::here("_common.R")

#===============================================================================

# Run common code
source(file_common)
```

# 16 Design and sample size decisions

## 16.4 Interactions are harder to estimate than main effects

### Understanding the problem by simulating regressions in R

#### Simulated data 1: predictor range: -0.5, 0.5

``` r
set.seed(SEED)

n <- 1000
sigma <- 10

y <- rnorm(n, mean = 0, sd = sigma)
sample_1 <- rep(1:2, n / 2) %>%  sample()
sample_2 <- rep(1:2, n / 2) %>%  sample()

sim <- function(v_1, v_2) {
  tibble(
    y,
    x_1 = c(v_1, v_2)[sample_1],
    x_2 = c(v_1, v_2)[sample_2]
  )
}

data_1 <- sim(-0.5, 0.5)
```

Fit model with one predictor.

``` r
fit_1_1 <- stan_glm(y ~ x_1, data = data_1, refresh = 0, seed = SEED)

fit_1_1
```

    #> stan_glm
    #>  family:       gaussian [identity]
    #>  formula:      y ~ x_1
    #>  observations: 1000
    #>  predictors:   2
    #> ------
    #>             Median MAD_SD
    #> (Intercept) -0.2    0.3  
    #> x_1          0.1    0.6  
    #> 
    #> Auxiliary parameter(s):
    #>       Median MAD_SD
    #> sigma 9.2    0.2   
    #> 
    #> ------
    #> * For help interpreting the printed output see ?print.stanreg
    #> * For info on the priors used see ?prior_summary.stanreg

Fit model with two predictors with an interaction.

``` r
fit_1_2 <- 
  stan_glm(y ~ x_1 + x_2 + x_1:x_2, data = data_1, refresh = 0, seed = SEED)

fit_1_2
```

    #> stan_glm
    #>  family:       gaussian [identity]
    #>  formula:      y ~ x_1 + x_2 + x_1:x_2
    #>  observations: 1000
    #>  predictors:   4
    #> ------
    #>             Median MAD_SD
    #> (Intercept) -0.2    0.3  
    #> x_1          0.2    0.6  
    #> x_2          0.5    0.6  
    #> x_1:x_2      0.9    1.2  
    #> 
    #> Auxiliary parameter(s):
    #>       Median MAD_SD
    #> sigma 9.2    0.2   
    #> 
    #> ------
    #> * For help interpreting the printed output see ?print.stanreg
    #> * For info on the priors used see ?prior_summary.stanreg

Ignore the estimates; they’re pure noise. Just look at the standard
errors. They follow the formulas:

Main effect:

``` r
2 * sigma / sqrt(n)
```

    #> [1] 0.632

Interaction:

``` r
4 * sigma / sqrt(n)
```

    #> [1] 1.26

#### Simulated data 2: predictor range: 0, 1

``` r
data_2 <- sim(0, 1)
```

Fit model with one predictor.

``` r
fit_2_1 <- stan_glm(y ~ x_1, data = data_2, refresh = 0, seed = SEED)

fit_2_1
```

    #> stan_glm
    #>  family:       gaussian [identity]
    #>  formula:      y ~ x_1
    #>  observations: 1000
    #>  predictors:   2
    #> ------
    #>             Median MAD_SD
    #> (Intercept) -0.3    0.4  
    #> x_1          0.1    0.6  
    #> 
    #> Auxiliary parameter(s):
    #>       Median MAD_SD
    #> sigma 9.2    0.2   
    #> 
    #> ------
    #> * For help interpreting the printed output see ?print.stanreg
    #> * For info on the priors used see ?prior_summary.stanreg

Fit model with two predictors with an interaction.

``` r
fit_2_2 <- 
  stan_glm(y ~ x_1 + x_2 + x_1:x_2, data = data_2, refresh = 0, seed = SEED)

fit_2_2
```

    #> stan_glm
    #>  family:       gaussian [identity]
    #>  formula:      y ~ x_1 + x_2 + x_1:x_2
    #>  observations: 1000
    #>  predictors:   4
    #> ------
    #>             Median MAD_SD
    #> (Intercept) -0.3    0.6  
    #> x_1         -0.4    0.9  
    #> x_2          0.0    0.9  
    #> x_1:x_2      1.0    1.2  
    #> 
    #> Auxiliary parameter(s):
    #>       Median MAD_SD
    #> sigma 9.2    0.2   
    #> 
    #> ------
    #> * For help interpreting the printed output see ?print.stanreg
    #> * For info on the priors used see ?prior_summary.stanreg

In this case, the standard error for the interaction is unchanged, and
the standard error for the main effects have increased by a factor of
sqrt(2).

#### Simulated data 3: predictor range: -1, 1

``` r
data_3 <- sim(-1, 1)
```

Fit model with one predictor.

``` r
fit_3_1 <- stan_glm(y ~ x_1, data = data_3, refresh = 0, seed = SEED)

fit_3_1
```

    #> stan_glm
    #>  family:       gaussian [identity]
    #>  formula:      y ~ x_1
    #>  observations: 1000
    #>  predictors:   2
    #> ------
    #>             Median MAD_SD
    #> (Intercept) -0.2    0.3  
    #> x_1          0.1    0.3  
    #> 
    #> Auxiliary parameter(s):
    #>       Median MAD_SD
    #> sigma 9.2    0.2   
    #> 
    #> ------
    #> * For help interpreting the printed output see ?print.stanreg
    #> * For info on the priors used see ?prior_summary.stanreg

Fit model with two predictors with an interaction.

``` r
fit_3_2 <- 
  stan_glm(y ~ x_1 + x_2 + x_1:x_2, data = data_3, refresh = 0, seed = SEED)

fit_3_2
```

    #> stan_glm
    #>  family:       gaussian [identity]
    #>  formula:      y ~ x_1 + x_2 + x_1:x_2
    #>  observations: 1000
    #>  predictors:   4
    #> ------
    #>             Median MAD_SD
    #> (Intercept) -0.2    0.3  
    #> x_1          0.1    0.3  
    #> x_2          0.2    0.3  
    #> x_1:x_2      0.2    0.3  
    #> 
    #> Auxiliary parameter(s):
    #>       Median MAD_SD
    #> sigma 9.2    0.2   
    #> 
    #> ------
    #> * For help interpreting the printed output see ?print.stanreg
    #> * For info on the priors used see ?prior_summary.stanreg

In this case, multiplying the predictors by 2 has the effect of dividing
the standard errors for the main effects by 2 and the standard errors
for the interaction by 4.
