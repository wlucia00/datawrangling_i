Datawrangling_i_3
================
Lucia Wang
2023-09-26

``` r
library(tidyverse)
```

    ## ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
    ## ✔ dplyr     1.1.3     ✔ readr     2.1.4
    ## ✔ forcats   1.0.0     ✔ stringr   1.5.0
    ## ✔ ggplot2   3.4.3     ✔ tibble    3.2.1
    ## ✔ lubridate 1.9.2     ✔ tidyr     1.3.0
    ## ✔ purrr     1.0.2     
    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter() masks stats::filter()
    ## ✖ dplyr::lag()    masks stats::lag()
    ## ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors

``` r
options(tibble.print_min=10)
```

## PULSE data

`Pivot_longer()` goes from wide to long

``` r
pulse_df = 
  haven::read_sas("./data_import_examples/public_pulse_data.sas7bdat") |>
  janitor::clean_names() |>
  pivot_longer(
    bdi_score_bl:bdi_score_12m,
    names_to = "visit",
    values_to = "bdi_score",
    names_prefix = "bdi_score_"
  ) |>
  mutate(
    visit = replace(visit, visit == "bl", "00m")
  )

pulse_df
```

    ## # A tibble: 4,348 × 5
    ##       id   age sex   visit bdi_score
    ##    <dbl> <dbl> <chr> <chr>     <dbl>
    ##  1 10003  48.0 male  00m           7
    ##  2 10003  48.0 male  01m           1
    ##  3 10003  48.0 male  06m           2
    ##  4 10003  48.0 male  12m           0
    ##  5 10015  72.5 male  00m           6
    ##  6 10015  72.5 male  01m          NA
    ##  7 10015  72.5 male  06m          NA
    ##  8 10015  72.5 male  12m          NA
    ##  9 10022  58.5 male  00m          14
    ## 10 10022  58.5 male  01m           3
    ## # ℹ 4,338 more rows

first you choose the variables to be pivoted with `:`, then name the new
variable with `names_to`, then what values you want to use with
`values_to`. finally, `names_prefix` gets rid of the prefix.

`replace()` takes in vector, logical statement, and replacement value

### learning assessment

``` r
litters_df =
  read_csv("./data_import_examples/FAS_litters.csv") |>
  janitor::clean_names() |>
  select(litter_number, gd0_weight, gd18_weight) |>
  pivot_longer(
    gd0_weight:gd18_weight, 
    names_to = "gd",
    values_to = "weight"
  ) |>
  mutate(
    gd = case_match(
      gd, 
      "gd0_weight"  ~ 0,
      "gd18_weight" ~ 18
    )
  )
```

    ## Rows: 49 Columns: 8
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (2): Group, Litter Number
    ## dbl (6): GD0 weight, GD18 weight, GD of Birth, Pups born alive, Pups dead @ ...
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
litters_df
```

    ## # A tibble: 98 × 3
    ##    litter_number    gd weight
    ##    <chr>         <dbl>  <dbl>
    ##  1 #85               0   19.7
    ##  2 #85              18   34.7
    ##  3 #1/2/95/2         0   27  
    ##  4 #1/2/95/2        18   42  
    ##  5 #5/5/3/83/3-3     0   26  
    ##  6 #5/5/3/83/3-3    18   41.4
    ##  7 #5/4/2/95/2       0   28.5
    ##  8 #5/4/2/95/2      18   44.1
    ##  9 #4/2/95/3-3       0   NA  
    ## 10 #4/2/95/3-3      18   NA  
    ## # ℹ 88 more rows

`case_match` helps you replace a bunch of stuff using the `~`

## LOTR - bind rows

Import LOTR words data, from 3 tables. `bind_rows()` stacks them
together

``` r
fellowship_df = 
  readxl::read_excel("./data_import_examples/LotR_Words.xlsx", range="B3:D6") |>
  mutate(movie = "fellowship")

twotowers_df = 
  readxl::read_excel("./data_import_examples/LotR_Words.xlsx", range="F3:H6") |>
  mutate(movie = "two towers")

returnoftheking_df = 
  readxl::read_excel("./data_import_examples/LotR_Words.xlsx", range="J3:L6") |>
  mutate(movie = "return of the king")

lotr_df =
  bind_rows(
    fellowship_df, twotowers_df, returnoftheking_df
  ) |>
  pivot_longer(
    Male:Female,
    names_to = "gender",
    values_to = "word"
  ) |>
  relocate(movie)
```
