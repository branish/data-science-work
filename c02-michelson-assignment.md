Michelson Speed-of-light Measurements
================
(Your name here)
2020-

  - [Grading Rubric](#grading-rubric)
      - [Individual](#individual)
      - [Team](#team)
      - [Due Date](#due-date)
      - [Bibliography](#bibliography)

*Purpose*: When studying physical problems, there is an important
distinction between *error* and *uncertainty*. The primary purpose of
this challenge is to dip our toes into these factors by analyzing a real
dataset.

*Reading*: [Experimental Determination of the Velocity of
Light](https://play.google.com/books/reader?id=343nAAAAMAAJ&hl=en&pg=GBS.PA115)
(Optional)

<!-- include-rubric -->

# Grading Rubric

<!-- -------------------------------------------------- -->

Unlike exercises, **challenges will be graded**. The following rubrics
define how you will be graded, both on an individual and team basis.

## Individual

<!-- ------------------------- -->

| Category    | Unsatisfactory                                                                   | Satisfactory                                                               |
| ----------- | -------------------------------------------------------------------------------- | -------------------------------------------------------------------------- |
| Effort      | Some task **q**’s left unattempted                                               | All task **q**’s attempted                                                 |
| Observed    | Did not document observations                                                    | Documented observations based on analysis                                  |
| Supported   | Some observations not supported by analysis                                      | All observations supported by analysis (table, graph, etc.)                |
| Code Styled | Violations of the [style guide](https://style.tidyverse.org/) hinder readability | Code sufficiently close to the [style guide](https://style.tidyverse.org/) |

## Team

<!-- ------------------------- -->

| Category   | Unsatisfactory                                                                                   | Satisfactory                                       |
| ---------- | ------------------------------------------------------------------------------------------------ | -------------------------------------------------- |
| Documented | No team contributions to Wiki                                                                    | Team contributed to Wiki                           |
| Referenced | No team references in Wiki                                                                       | At least one reference in Wiki to member report(s) |
| Relevant   | References unrelated to assertion, or difficult to find related analysis based on reference text | Reference text clearly points to relevant analysis |

## Due Date

<!-- ------------------------- -->

All the deliverables stated in the rubrics above are due on the day of
the class discussion of that exercise. See the
[Syllabus](https://docs.google.com/document/d/1jJTh2DH8nVJd2eyMMoyNGroReo0BKcJrz1eONi3rPSc/edit?usp=sharing)
for more information.

``` r
# Libraries
library(tidyverse)
library(googlesheets4)

url <- "https://docs.google.com/spreadsheets/d/1av_SXn4j0-4Rk0mQFik3LLr-uf0YdA06i3ugE6n-Zdo/edit?usp=sharing"

# Parameters
LIGHTSPEED_VACUUM    <- 299792.458 # Exact speed of light in a vacuum (km / s)
LIGHTSPEED_MICHELSON <- 299944.00  # Michelson's speed estimate (km / s)
LIGHTSPEED_PM        <- 51         # Michelson error estimate (km / s)
```

*Background*: In 1879 Albert Michelson led an experimental campaign to
measure the speed of light. His approach was a development upon the
method of Foucault, and resulted in a new estimate of
\(v_0 = 299944 \pm 51\) kilometers per second (in a vacuum). This is
very close to the modern *exact* value of `r LIGHTSPEED_VACUUM`. In this
challenge, you will analyze Michelson’s original data, and explore some
of the factors associated with his experiment.

I’ve already copied Michelson’s data from his 1880 publication; the code
chunk below will load these data from a public googlesheet.

*Aside*: The speed of light is *exact* (there is **zero error** in the
value `LIGHTSPEED_VACUUM`) because the meter is actually
[*defined*](https://en.wikipedia.org/wiki/Metre#Speed_of_light_definition)
in terms of the speed of light\!

``` r
## Note: No need to edit this chunk!
gs4_deauth()
ss <- gs4_get(url)
df_michelson <-
  read_sheet(ss) %>%
  select(Date, Distinctness, Temp, Velocity) %>%
  mutate(Distinctness = as_factor(Distinctness))
```

    ## Reading from "michelson1879"

    ## Range "Sheet1"

``` r
df_michelson %>% glimpse
```

    ## Rows: 100
    ## Columns: 4
    ## $ Date         <dttm> 1879-06-05, 1879-06-07, 1879-06-07, 1879-06-07, 1879-...
    ## $ Distinctness <fct> 3, 2, 2, 2, 2, 2, 3, 3, 3, 3, 2, 2, 2, 2, 2, 1, 3, 3, ...
    ## $ Temp         <dbl> 76, 72, 72, 72, 72, 72, 83, 83, 83, 83, 83, 90, 90, 71...
    ## $ Velocity     <dbl> 299850, 299740, 299900, 300070, 299930, 299850, 299950...

*Data dictionary*:

  - `Date`: Date of measurement
  - `Distinctness`: Distinctness of measured images: 3 = good, 2 = fair,
    1 = poor
  - `Temp`: Ambient temperature (Fahrenheit)
  - `Velocity`: Measured speed of light (km / s)

**q1** Re-create the following table (from Michelson (1880), pg. 139)
using `df_michelson` and `dplyr`. Note that your values *will not* match
those of Michelson *exactly*; why might this be?

| Distinctness | n  | MeanVelocity |
| ------------ | -- | ------------ |
| 3            | 46 | 299860       |
| 2            | 39 | 299860       |
| 1            | 15 | 299810       |

``` r
## TODO: Compute summaries
df_q1 <-
  df_michelson %>%
  group_by(Distinctness) %>%
  summarize(n = n(), MeanVelocity = mean(Velocity))
```

    ## `summarise()` ungrouping output (override with `.groups` argument)

``` r
df_q1 %>%
  arrange(desc(Distinctness)) %>%
  knitr::kable()
```

| Distinctness |  n | MeanVelocity |
| :----------- | -: | -----------: |
| 3            | 46 |     299861.7 |
| 2            | 39 |     299858.5 |
| 1            | 15 |     299808.0 |

``` r
df_michelson %>%
  ggplot() +
  labs(title = "Velocity vs Distinctness", caption = "black line: LIGHTSPEED_MICHELSON; blue line: Michelson's adjusted lightspeed; green line: actual light speed in vacuum") +
  ylab("Velocity (km/s)") +
  geom_boxplot(mapping = aes(x = Distinctness, y = Velocity)) +
  geom_hline(yintercept = LIGHTSPEED_MICHELSON) +
  geom_hline(yintercept = 299852.4, color = "blue") +
  geom_hline(yintercept = LIGHTSPEED_VACUUM, color="green")
```

![](c02-michelson-assignment_files/figure-gfm/q1-task-1.png)<!-- -->

``` r
df_michelson %>%
  ggplot() +
  ggtitle("All velocity measurements") +
  labs(caption = "green line: actual light speed in vacuum") +
  xlab("Velocity (km/s)") +
  geom_boxplot(mapping = aes(x = Velocity, y = "all samples")) +
  geom_vline(xintercept = LIGHTSPEED_VACUUM, color="green")
```

![](c02-michelson-assignment_files/figure-gfm/q1-task-2.png)<!-- -->

**Observations**: Michelson’s observations in that table are rounded to
the nearest 10. Their average cannot be more precise than the original
measurements. The averages that I calculated ought to be rounded to the
correct number of significant digits, too.

The `Velocity` values in the dataset are the speed of light *in air*;
Michelson introduced a couple of adjustments to estimate the speed of
light in a vacuum. In total, he added \(+92\) km/s to his mean estimate
for `VelocityVacuum` (from Michelson (1880), pg. 141). While this isn’t
fully rigorous (\(+92\) km/s is based on the mean temperature), we’ll
simply apply this correction to all the observations in the dataset.

**q2** Create a new variable `VelocityVacuum` with the \(+92\) km/s
adjustment to `Velocity`. Assign this new dataframe to `df_q2`.

``` r
## TODO: Adjust the data, assign to df_q2
df_q2 <-
  df_michelson %>%
  mutate(VelocityVacuum = Velocity + 92)
```

As part of his study, Michelson assessed the various potential sources
of error, and provided his best-guess for the error in his
speed-of-light estimate. These values are provided in
`LIGHTSPEED_MICHELSON`—his nominal estimate—and
`LIGHTSPEED_PM`—plus/minus bounds on his estimate. Put differently,
Michelson believed the true value of the speed-of-light probably lay
between `LIGHTSPEED_MICHELSON - LIGHTSPEED_PM` and `LIGHTSPEED_MICHELSON
+ LIGHTSPEED_PM`.

Let’s introduce some terminology:\[2\]

  - **Error** is the difference between a true value and an estimate of
    that value; for instance `LIGHTSPEED_VACUUM - LIGHTSPEED_MICHELSON`.
  - **Uncertainty** is an analyst’s *assessment* of the error.

Since a “true” value is often not known in practice, one generally does
not know the error. The best they can do is quantify their degree of
uncertainty. We will learn some means of quantifying uncertainty in this
class, but for many real problems uncertainty includes some amount of
human judgment.\[2\]

**q3** Compare Michelson’s speed of light estimate against the modern
speed of light value. Is Michelson’s estimate of the error (his
uncertainty) greater or less than the true error?

``` r
## TODO: Compare Michelson's estimate and error against the true value
## Your code here!
errs <- c(abs(LIGHTSPEED_VACUUM - LIGHTSPEED_MICHELSON), LIGHTSPEED_PM)
names <- c("True error", "Michelson's estimated error")
ggplot() +
  ylab("Error (km/s)") +
  xlab("") +
  geom_col(mapping = aes(x = names, y = errs))
```

![](c02-michelson-assignment_files/figure-gfm/q3-task-1.png)<!-- -->

**Observations**: - Michelson’s uncertainty value was a third of the
actual error.

**q4** You have access to a few other variables. Construct a few
visualizations of `VelocityVacuum` against these other factors. Are
there other patterns in the data that might help explain the difference
between Michelson’s estimate and `LIGHTSPEED_VACUUM`?

``` r
# VelocityVacuum distribution
df_q2 %>%
  ggplot() +
  ggtitle("VelocityVacuum density") +
  geom_density(mapping = aes(x = VelocityVacuum, color = Distinctness)) +
  geom_vline(aes(xintercept = LIGHTSPEED_VACUUM))
```

![](c02-michelson-assignment_files/figure-gfm/unnamed-chunk-2-1.png)<!-- -->

``` r
df_q2 %>%
  ggplot() +
  ggtitle("VelocityVacuum histogram") +
  geom_histogram(aes(x = VelocityVacuum)) +
  geom_vline(aes(xintercept = LIGHTSPEED_VACUUM))
```

    ## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.

![](c02-michelson-assignment_files/figure-gfm/unnamed-chunk-2-2.png)<!-- -->

``` r
# Distinctness vs Temp
df_q2 %>%
  ggplot() +
  ggtitle("Distinctness vs Temp") +
  geom_boxplot(aes(x = Temp, y = Distinctness))
```

![](c02-michelson-assignment_files/figure-gfm/unnamed-chunk-2-3.png)<!-- -->

``` r
# VelocityVacuum vs Distinctness
df_q2 %>%
  ggplot() +
  ggtitle("VelocityVacuum vs Distinctness") +
  geom_boxplot(aes(x = VelocityVacuum, y = Distinctness)) +
  geom_vline(aes(xintercept = LIGHTSPEED_VACUUM))
```

![](c02-michelson-assignment_files/figure-gfm/unnamed-chunk-2-4.png)<!-- -->

``` r
#VelocityVacuum vs Temp
df_q2 %>%
  ggplot() +
  ggtitle("VelocityVacuum vs Temp") +
  geom_point(aes(x = Temp, y = VelocityVacuum, color = Date)) +
  geom_smooth(aes(x = Temp, y = VelocityVacuum)) +
  geom_hline(aes(yintercept = LIGHTSPEED_VACUUM))
```

    ## `geom_smooth()` using method = 'loess' and formula 'y ~ x'

![](c02-michelson-assignment_files/figure-gfm/unnamed-chunk-2-5.png)<!-- -->

``` r
df_q2 %>%
  ggplot() +
  ggtitle("VelocityVacuum vs Temp") +
  geom_point(aes(x = Temp, y = VelocityVacuum, color = Date)) +
  geom_hline(aes(yintercept = LIGHTSPEED_VACUUM))
```

![](c02-michelson-assignment_files/figure-gfm/unnamed-chunk-2-6.png)<!-- -->

``` r
# VelocityVacuum vs Date
df_q2 %>%
  ggplot() +
  ggtitle("VelocityVacuum vs Date") +
  geom_point(aes(x = Date, y = VelocityVacuum, color = Temp)) +
  geom_smooth(aes(x = Date, y = VelocityVacuum)) +
  geom_hline(aes(yintercept = LIGHTSPEED_VACUUM))
```

    ## `geom_smooth()` using method = 'loess' and formula 'y ~ x'

![](c02-michelson-assignment_files/figure-gfm/unnamed-chunk-2-7.png)<!-- -->

``` r
#Temp vs Date
df_q2 %>%
  ggplot() +
  ggtitle("Temp vs Date") +
  geom_point(mapping = aes(x = Date, y = Temp, color = VelocityVacuum))
```

![](c02-michelson-assignment_files/figure-gfm/unnamed-chunk-2-8.png)<!-- -->

``` r
df_q2 %>%
  mutate(Error = VelocityVacuum - LIGHTSPEED_VACUUM) %>%
  ggplot() +
  ggtitle("Error vs Date") +
  geom_point(mapping = aes(x = Date, y = Error, color = Distinctness))
```

![](c02-michelson-assignment_files/figure-gfm/unnamed-chunk-2-9.png)<!-- -->

## Bibliography

  - \[1\] Michelson, [Experimental Determination of the Velocity of
    Light](https://play.google.com/books/reader?id=343nAAAAMAAJ&hl=en&pg=GBS.PA115)
    (1880)
  - \[2\] Henrion and Fischhoff, [Assessing Uncertainty in Physical
    Constants](https://www.cmu.edu/epp/people/faculty/research/Fischoff-Henrion-Assessing%20uncertainty%20in%20physical%20constants.pdf)
    (1986)
