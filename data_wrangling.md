Data import and Wrangling
================

``` r
library (tidyverse)
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
library(dplyr)
```

# Step 1: Data import.

1)  In this step I download and renamed the
    Mental_Health_Care_in_the_Last_4_Weeks.csv to mental_health.csv to
    my computer, then I imported it to R and name the dataset as
    rawdata.
2)  during this step, I removing Unnecessary Columns including : phase,
    quartile range and suppression flag. Then, I also renamed some
    columns: Improving column names for clarity or consistency.

``` r
rawdata = read_csv("data/mental_health.csv") |> 
  janitor::clean_names() |>
  mutate(indicator = str_replace(indicator, ", Last 4 Weeks", "")) |> 
  select(-phase,-quartile_range,-suppression_flag) |> 
  mutate(group = str_replace(group, "By", ""))|> 
  select(start_date=time_period_start_date,end_date=time_period_end_date,everything()) 
```

    ## Rows: 10404 Columns: 15
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (10): Indicator, Group, State, Subgroup, Phase, Time Period Label, Time ...
    ## dbl  (5): Time Period, Value, LowCI, HighCI, Suppression Flag
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
