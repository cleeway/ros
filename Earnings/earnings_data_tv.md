Regression and Other Stories: Earnings data
================
Andrew Gelman, Jennifer Hill, Aki Vehtari
2021-04-20

-   [A Computing in R](#a-computing-in-r)
    -   [A.6 Working with messy data](#a6-working-with-messy-data)
        -   [Reading in survey data, one question at a
            time](#reading-in-survey-data-one-question-at-a-time)
        -   [Cleaning data within R](#cleaning-data-within-r)
        -   [Looking at the data](#looking-at-the-data)

Tidyverse version by Bill Behrman.

Read in and prepare earnings data. See Appendix A in Regression and
Other Stories.

Source: Ross, Catherine E. Work, Family, and Well-Being in the United
States, 1990. 1996-06-10. <https://doi.org/10.3886/ICPSR06666.v1>

------------------------------------------------------------------------

``` r
# Packages
library(tidyverse)

# Parameters
  # Earnings data
file_earnings <- here::here("Earnings/data/wfw90.dat")
  # Common code
file_common <- here::here("_common.R")

#===============================================================================

# Run common code
source(file_common)
```

# A Computing in R

## A.6 Working with messy data

### Reading in survey data, one question at a time

Read in survey data.

``` r
earnings <- 
  file_earnings %>% 
  read_fwf(
    col_positions = 
      fwf_cols(
        height_feet = 144,
        height_inches = c(145, 146),
        weight = c(147, 149),
        earn_exact = c(203, 208),
        earn2 = c(209, 210),
        sex = 219
      ),
    col_types = cols(.default = col_double())
  )
```

### Cleaning data within R

1.  Look at the data.
2.  Identify errors or missing data.

``` r
earnings
```

    #> # A tibble: 2,031 x 6
    #>    height_feet height_inches weight earn_exact earn2   sex
    #>          <dbl>         <dbl>  <dbl>      <dbl> <dbl> <dbl>
    #>  1           5             6    140         NA    90     2
    #>  2           5             4    150         NA    90     1
    #>  3           6             2    210      50000    NA     1
    #>  4           5             6    125      60000    NA     2
    #>  5           5             4    126      30000    NA     2
    #>  6           5             5    200         NA    25     2
    #>  7           5             3    110      50000    NA     2
    #>  8           5             8    165         NA    62     2
    #>  9           5             3    190      51000    NA     2
    #> 10           5             4    125       9000    NA     2
    #> # … with 2,021 more rows

From the first rows, it appears as though for each observation only one
of `earn_exact` or `earn2` is non-`NA`. Let’s see if this holds for all
observations.

``` r
earnings %>% 
  count(!is.na(earn_exact), !is.na(earn2))
```

    #> # A tibble: 2 x 3
    #>   `!is.na(earn_exact)` `!is.na(earn2)`     n
    #>   <lgl>                <lgl>           <int>
    #> 1 FALSE                TRUE              651
    #> 2 TRUE                 FALSE            1380

It does.

``` r
summary(earnings)
```

    #>   height_feet   height_inches      weight      earn_exact         earn2     
    #>  Min.   :4.00   Min.   : 0.0   Min.   : 80   Min.   :     0   Min.   : 1    
    #>  1st Qu.:5.00   1st Qu.: 3.0   1st Qu.:130   1st Qu.:  6000   1st Qu.:15    
    #>  Median :5.00   Median : 5.0   Median :150   Median : 16450   Median :25    
    #>  Mean   :5.14   Mean   : 5.5   Mean   :174   Mean   : 20290   Mean   :46    
    #>  3rd Qu.:5.00   3rd Qu.: 8.0   3rd Qu.:180   3rd Qu.: 28000   3rd Qu.:90    
    #>  Max.   :9.00   Max.   :99.0   Max.   :999   Max.   :400000   Max.   :96    
    #>                                              NA's   :651      NA's   :1380  
    #>       sex      
    #>  Min.   :1.00  
    #>  1st Qu.:1.00  
    #>  Median :2.00  
    #>  Mean   :1.63  
    #>  3rd Qu.:2.00  
    #>  Max.   :2.00  
    #> 

`height_feet` has values of 9 feet, `height_inches` has values of 99
inches, and `weight` has values of 999 pounds. From the data codebook,
we see that the missing data codes are:

-   Don’t know: 8, 98, 998
-   No response: 9, 99, 999

We’ll recode these values as `NA`s.

``` r
earnings <- 
  earnings %>% 
  mutate(
    height_feet = height_feet %>% na_if(8) %>% na_if(9),
    height_inches = height_inches %>% na_if(98) %>% na_if(99),
    weight = weight %>% na_if(998) %>% na_if(999)
  )

earnings %>% 
  select(height_feet, height_inches, weight) %>% 
  summary()
```

    #>   height_feet   height_inches       weight   
    #>  Min.   :4.00   Min.   : 0.00   Min.   : 80  
    #>  1st Qu.:5.00   1st Qu.: 3.00   1st Qu.:130  
    #>  Median :5.00   Median : 5.00   Median :150  
    #>  Mean   :5.12   Mean   : 5.09   Mean   :156  
    #>  3rd Qu.:5.00   3rd Qu.: 8.00   3rd Qu.:180  
    #>  Max.   :7.00   Max.   :11.00   Max.   :342  
    #>  NA's   :8      NA's   :8       NA's   :42

Let’s look at the observations with `height_feet` of 7 feet.

``` r
earnings %>% 
  filter(height_feet == 7)
```

    #> # A tibble: 1 x 6
    #>   height_feet height_inches weight earn_exact earn2   sex
    #>         <dbl>         <dbl>  <dbl>      <dbl> <dbl> <dbl>
    #> 1           7             7    110         NA    25     2

The observation of someone 7 feet 7 inches tall who weighs 110 pounds is
probably a data error, so we will recode the `height_feet` value of 7 as
missing.

``` r
earnings <- 
  earnings %>% 
  mutate(height_feet = height_feet %>% na_if(7))
```

3. Transform or combine raw data into summaries of interest.

Using the codebook, we’ll first recode `sex`.

``` r
earnings <- 
  earnings %>% 
   mutate(
    sex = 
      case_when(
        sex == 1 ~ "Male",
        sex == 2 ~ "Female",
        TRUE ~ NA_character_
      )
  )
```

We’ll next create a combined variable `height`.

``` r
earnings <- 
  earnings %>% 
  mutate(height = 12 * height_feet + height_inches)
```

Using the codebook, we’ll create a new variable `earnings_approx` from
`earn2`.

``` r
earnings <- 
  earnings %>% 
  mutate(
    earn_approx =
      case_when(
        earn2 >= 90 ~ NA_real_,
        earn2 == 1 ~ 
          median(earn_exact[earn_exact > 100000], na.rm = TRUE) / 1000,
        TRUE ~ earn2
      )
  )
```

We’ll finally create a combined earnings variable `earn`.

``` r
earnings <- 
  earnings %>% 
  mutate(earn = if_else(!is.na(earn_exact), earn_exact, 1000 * earn_approx))
```

The new `earn` variable still has 237 missing values (out of 2031
respondents in total) and is imperfect in various ways, but we have to
make some choices when working with real data.

### Looking at the data

Sex distribution.

``` r
earnings %>% 
  ggplot(aes(sex)) +
  geom_bar() +
  labs(title = "Sex distribution")
```

<img src="earnings_data_tv_files/figure-gfm/unnamed-chunk-13-1.png" width="100%" />

We have a greater number of women than men in the data, and a greater
proportion than in the general adult population.

Height distributions.

``` r
earnings %>% 
  drop_na(height) %>% 
  ggplot(aes(height)) +
  geom_bar() +
  facet_grid(rows = vars(sex)) +
  scale_x_continuous(breaks = scales::breaks_width(2)) +
  labs(title = "Height distributions")
```

<img src="earnings_data_tv_files/figure-gfm/unnamed-chunk-14-1.png" width="100%" />

There appears to be an excess of women who at 5 feet (60 inches) and 5
feet 6 inches (66 inches), and an excess of men who are 6 feet (72
inches), probably due to respondents rounding.

Weight distributions.

``` r
earnings %>% 
  drop_na(weight) %>% 
  ggplot(aes(weight)) +
  geom_histogram(binwidth = 10, boundary = 0) +
  facet_grid(rows = vars(sex)) +
  scale_x_continuous(breaks = scales::breaks_width(50)) +
  labs(title = "Weight distributions")
```

<img src="earnings_data_tv_files/figure-gfm/unnamed-chunk-15-1.png" width="100%" />

Nothing remarkable.

Earnings distributions.

``` r
earnings %>% 
  drop_na(earn) %>% 
  ggplot(aes(earn)) +
  geom_histogram(binwidth = 5e3, boundary = 0) +
  coord_cartesian(xlim = c(NA, 70e3)) +
  facet_grid(rows = vars(sex)) +
  scale_x_continuous(breaks = scales::breaks_width(10e3)) +
  labs(title = "Earnings distributions")
```

<img src="earnings_data_tv_files/figure-gfm/unnamed-chunk-16-1.png" width="100%" />

There are spikes in the lowest bin. Let’s check to see if these are due
to respondents with no earnings.

``` r
v <- 
  earnings %>% 
  filter(earn == 0) %>% 
  count(sex)

v
```

    #> # A tibble: 2 x 2
    #>   sex        n
    #> * <chr>  <int>
    #> 1 Female   172
    #> 2 Male      15

187 respondents had no earnings, with 172 of these women.

Earnings distributions.

``` r
earnings %>% 
  drop_na(earn) %>% 
  ggplot(aes(sex, earn)) +
  geom_boxplot() +
  coord_cartesian(ylim = c(NA, 70e3)) +
  labs(title = "Earnings distributions")
```

<img src="earnings_data_tv_files/figure-gfm/unnamed-chunk-18-1.png" width="100%" />

Men have a higher median earnings.

``` r
v <- 
  earnings %>% 
  group_by(sex) %>% 
  summarize(earn_median = median(earn, na.rm = TRUE))

v
```

    #> # A tibble: 2 x 2
    #>   sex    earn_median
    #> * <chr>        <dbl>
    #> 1 Female       15000
    #> 2 Male         25000

The median earnings of men is 10000 higher.
