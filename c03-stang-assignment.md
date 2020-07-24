Aluminum Data
================
(Your name here)
2020-

  - [Grading Rubric](#grading-rubric)
      - [Individual](#individual)
      - [Team](#team)
      - [Due Date](#due-date)
  - [Loading and Wrangle](#loading-and-wrangle)
  - [EDA](#eda)
      - [Initial checks](#initial-checks)
      - [Visualize](#visualize)
  - [References](#references)

*Purpose*: When designing structures such as bridges, boats, and planes,
the design team needs data about *material properties*. Often when we
engineers first learn about material properties through coursework, we
talk about abstract ideas and look up values in tables without ever
looking at the data that gave rise to published properties. In this
challenge you’ll study an aluminum alloy dataset: Studying these data
will give you a better sense of the challenges underlying published
material values.

In this challenge, you will load a real dataset, wrangle it into tidy
form, and perform EDA to learn more about the data.

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
library(tidyverse)
```

    ## -- Attaching packages ---------------------------------------------------------------------------------------------------- tidyverse 1.3.0 --

    ## v ggplot2 3.3.2     v purrr   0.3.4
    ## v tibble  3.0.1     v dplyr   1.0.0
    ## v tidyr   1.1.0     v stringr 1.4.0
    ## v readr   1.3.1     v forcats 0.5.0

    ## -- Conflicts ------------------------------------------------------------------------------------------------------- tidyverse_conflicts() --
    ## x dplyr::filter() masks stats::filter()
    ## x dplyr::lag()    masks stats::lag()

``` r
library(viridis)
```

    ## Loading required package: viridisLite

*Background*: In 1946, scientists at the Bureau of Standards tested a
number of Aluminum plates to determine their
[elasticity](https://en.wikipedia.org/wiki/Elastic_modulus) and
[Poisson’s ratio](https://en.wikipedia.org/wiki/Poisson%27s_ratio).
These are key quantities used in the design of structural members, such
as aircraft skin under [buckling
loads](https://en.wikipedia.org/wiki/Buckling). These scientists tested
plats of various thicknesses, and at different angles with respect to
the [rolling](https://en.wikipedia.org/wiki/Rolling_\(metalworking\))
direction.

# Loading and Wrangle

<!-- -------------------------------------------------- -->

The `readr` package in the Tidyverse contains functions to load data
form many sources. The `read_csv()` function will help us load the data
for this challenge.

``` r
## NOTE: If you extracted all challenges to the same location,
## you shouldn't have to change this filename
filename <- "./data/stang.csv"

## Load the data
df_stang <- read_csv(filename)
```

    ## Parsed with column specification:
    ## cols(
    ##   thick = col_double(),
    ##   E_00 = col_double(),
    ##   mu_00 = col_double(),
    ##   E_45 = col_double(),
    ##   mu_45 = col_double(),
    ##   E_90 = col_double(),
    ##   mu_90 = col_double(),
    ##   alloy = col_character()
    ## )

``` r
df_stang
```

    ## # A tibble: 9 x 8
    ##   thick  E_00 mu_00  E_45  mu_45  E_90 mu_90 alloy  
    ##   <dbl> <dbl> <dbl> <dbl>  <dbl> <dbl> <dbl> <chr>  
    ## 1 0.022 10600 0.321 10700  0.329 10500 0.31  al_24st
    ## 2 0.022 10600 0.323 10500  0.331 10700 0.323 al_24st
    ## 3 0.032 10400 0.329 10400  0.318 10300 0.322 al_24st
    ## 4 0.032 10300 0.319 10500  0.326 10400 0.33  al_24st
    ## 5 0.064 10500 0.323 10400  0.331 10400 0.327 al_24st
    ## 6 0.064 10700 0.328 10500  0.328 10500 0.32  al_24st
    ## 7 0.081 10000 0.315 10000  0.32   9900 0.314 al_24st
    ## 8 0.081 10100 0.312  9900  0.312 10000 0.316 al_24st
    ## 9 0.081 10000 0.311    -1 -1      9900 0.314 al_24st

Note that these data are not tidy\! The data in this form are convenient
for reporting in a table, but are not ideal for analysis.

**q1** Tidy `df_stang` to produce `df_stang_long`. You should have
column names `thick, alloy, angle, E, mu`. Make sure the `angle`
variable is of correct type. Filter out any invalid values.

*Hint*: You can reshape in one `pivot` using the `".value"` special
value for `names_to`.

``` r
## TASK: Tidy `df_stang`
df_stang_long <-
  df_stang %>%
  pivot_longer(
    names_sep = "_",
    names_to = c(".value", "angle"),
    names_transform = list(angle = as.integer),
    cols = c(2:7)
  ) %>%
  filter(E > 0)
  

df_stang_long
```

    ## # A tibble: 26 x 5
    ##    thick alloy   angle     E    mu
    ##    <dbl> <chr>   <int> <dbl> <dbl>
    ##  1 0.022 al_24st     0 10600 0.321
    ##  2 0.022 al_24st    45 10700 0.329
    ##  3 0.022 al_24st    90 10500 0.31 
    ##  4 0.022 al_24st     0 10600 0.323
    ##  5 0.022 al_24st    45 10500 0.331
    ##  6 0.022 al_24st    90 10700 0.323
    ##  7 0.032 al_24st     0 10400 0.329
    ##  8 0.032 al_24st    45 10400 0.318
    ##  9 0.032 al_24st    90 10300 0.322
    ## 10 0.032 al_24st     0 10300 0.319
    ## # ... with 16 more rows

Use the following tests to check your work.

``` r
## NOTE: No need to change this
## Names
assertthat::assert_that(
              setequal(
                df_stang_long %>% names,
                c("thick", "alloy", "angle", "E", "mu")
              )
            )
```

    ## [1] TRUE

``` r
## Dimensions
assertthat::assert_that(all(dim(df_stang_long) == c(26, 5)))
```

    ## [1] TRUE

``` r
## Type
assertthat::assert_that(
              (df_stang_long %>% pull(angle) %>% typeof()) == "integer"
            )
```

    ## [1] TRUE

``` r
print("Very good!")
```

    ## [1] "Very good!"

# EDA

<!-- -------------------------------------------------- -->

## Initial checks

<!-- ------------------------- -->

**q2** Perform a basic EDA on the aluminum data *without visualization*.
Use your analysis to answer the questions under *observations* below. In
addition, add your own question that you’d like to answer about the
data.

``` r
##
df_stang_long %>% select(-alloy) %>% arrange(thick, angle)
```

    ## # A tibble: 26 x 4
    ##    thick angle     E    mu
    ##    <dbl> <int> <dbl> <dbl>
    ##  1 0.022     0 10600 0.321
    ##  2 0.022     0 10600 0.323
    ##  3 0.022    45 10700 0.329
    ##  4 0.022    45 10500 0.331
    ##  5 0.022    90 10500 0.31 
    ##  6 0.022    90 10700 0.323
    ##  7 0.032     0 10400 0.329
    ##  8 0.032     0 10300 0.319
    ##  9 0.032    45 10400 0.318
    ## 10 0.032    45 10500 0.326
    ## # ... with 16 more rows

``` r
# number of entries for each alloy -- shows there is only one alloy
df_stang_long %>% group_by(alloy) %>% count()
```

    ## # A tibble: 1 x 2
    ## # Groups:   alloy [1]
    ##   alloy       n
    ##   <chr>   <int>
    ## 1 al_24st    26

``` r
# number of angles tested
df_stang_long %>% group_by(angle) %>% summarize()
```

    ## `summarise()` ungrouping output (override with `.groups` argument)

    ## # A tibble: 3 x 1
    ##   angle
    ##   <int>
    ## 1     0
    ## 2    45
    ## 3    90

``` r
# thickness values tested
df_stang_long %>% group_by(thick) %>% summarize()
```

    ## `summarise()` ungrouping output (override with `.groups` argument)

    ## # A tibble: 4 x 1
    ##   thick
    ##   <dbl>
    ## 1 0.022
    ## 2 0.032
    ## 3 0.064
    ## 4 0.081

**Observations**:

  - Is there “one true value” for the material properties of Aluminum?
    No. For the same thickness and angle, there are often multiple
    measured values for mu and E.

  - How many aluminum alloys were tested? How do you know? One alloy.
    All the rows have alloy == al\_24st.

  - What angles were tested? 0, 45, and 90 degrees.

  - What thicknesses were tested? .022, .032, .064, and .081 inches.

  - What is the relationship between thickness and E?

## Visualize

<!-- ------------------------- -->

**q3** Create a visualization to investigate your question from q1
above. Can you find an answer to your question using the dataset? Would
you need additional information to answer your question?

``` r
## TASK: Investigate your question from q1 here

df_stang_long %>%
  ggplot() +
  ggtitle("E vs thickness for different angles") +
  geom_point(mapping = aes(x = factor(thick), y = E, group = angle), size = 2, alpha = .3) +
  facet_wrap( ~ angle)
```

![](c03-stang-assignment_files/figure-gfm/q3-task-1.png)<!-- -->

``` r
df_stang_long %>%
  ggplot() +
  ggtitle("E vs thickness") +
  geom_boxplot(mapping = aes(x = factor(thick), y = E))
```

![](c03-stang-assignment_files/figure-gfm/q3-task-2.png)<!-- --> - Three
of the tested thicknesses have almost the same measured elasticity, but
the thickest sample shows a much lower elasticity. - For thicknesses
0.022, 0.032, and 0.064, the measured values for elasticity just barely
intersect. Only the thickest material shows a definite decrease in
elasticity. - However, the average elasticity was actually higher at
0.064 inches than at 0.032 inches. If there is a relationship between
thickness and elasticity, either it’s not monotonic or this is an effect
of random error that might disappear with more data samples.

``` r
df_stang_long %>%
  ggplot() +
  ggtitle("E vs angle") +
  #scale_fill_viridis() +
  geom_boxplot(mapping = aes(x = factor(angle), y = E))
```

![](c03-stang-assignment_files/figure-gfm/unnamed-chunk-1-1.png)<!-- -->

  - The angle of force doesn’t affect elasticity much.

<!-- end list -->

``` r
df_stang_long %>%
  ggplot() +
  ggtitle("mu vs thickness") +
  geom_boxplot(mapping = aes(x = factor(thick), y = mu))
```

![](c03-stang-assignment_files/figure-gfm/unnamed-chunk-2-1.png)<!-- -->
- Poisson’s ratio shows the same pattern that elasticity showed with
regard to thickness. Three of the material samples tested have
approximately the same value for mu, but the thickest sample has a much
lower average for mu.

``` r
df_stang_long %>%
  ggplot() +
  ggtitle("mu vs angle") +
  geom_boxplot(mapping = aes(x = factor(angle), y = mu))
```

![](c03-stang-assignment_files/figure-gfm/unnamed-chunk-3-1.png)<!-- -->
- Over different angles, the measured ranges for mu overlap almost
completely. Poisson’s ratio is probably not affected by angle of force.

**Observations**:

  - (Address your question from q1 here)

**q4** Consider the following statement:

“A material’s property (or material property) is an intensive property
of some material, i.e. a physical property that does not depend on the
amount of the material.”\[2\]

Note that the “amount of material” would vary with the thickness of a
tested plate. Does the following graph support or contradict the claim
that “elasticity `E` is an intensive material property.” Why or why not?
Is this evidence *conclusive* one way or another? Why or why not?

``` r
## NOTE: No need to change; run this chunk
df_stang_long %>%

  ggplot(aes(mu, E, color = as_factor(thick))) +
  geom_point(size = 3) +
  theme_minimal()
```

![](c03-stang-assignment_files/figure-gfm/q4-vis-1.png)<!-- -->

**Observations**:

  - This graph contradicts the claim above. The samples with thickness
    0.081 inches have consistently lower elasticity than the other three
    material thicknesses. It appears that the amount of material does
    affect elasticity.

# References

<!-- -------------------------------------------------- -->

\[1\] Stang, Greenspan, and Newman, “Poisson’s ratio of some structural
alloys for large strains” (1946) Journal of Research of the National
Bureau of Standards, (pdf
link)\[<https://nvlpubs.nist.gov/nistpubs/jres/37/jresv37n4p211_A1b.pdf>\]

\[2\] Wikipedia, *List of material properties*, accessed 2020-06-26,
(link)\[<https://en.wikipedia.org/wiki/List_of_materials_properties>\]
