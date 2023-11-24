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

## Step 1: Data import

First, I download the .csv to my computer and renamed the file from
Mental_Health_Care_in_the_Last_4_Weeks.csv to mental_health.csv. Then, I
imported it to R with `read_csv` and saved the data set as `rawdata`.

``` r
rawdata =
  read_csv("data/mental_health.csv")
```

    ## Rows: 10404 Columns: 15
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (10): Indicator, Group, State, Subgroup, Phase, Time Period Label, Time ...
    ## dbl  (5): Time Period, Value, LowCI, HighCI, Suppression Flag
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

## Step 2: Rename and clean columns

I removed unnecessary columns, including phase, quartile range, and
suppression flag, and cleaned and renamed some columns for clarity or
consistency. Additionally, I dropped missing values in the `value`
column.

``` r
tidydata = 
  rawdata |> 
  janitor::clean_names() |>
  mutate(indicator = str_replace(indicator, ", Last 4 Weeks", "")) |> 
  select(-phase, -quartile_range, -suppression_flag) |> 
  mutate(group = str_replace(group, "By ", "")) |>
  drop_na(value) |>
  select(start_date = time_period_start_date, end_date = time_period_end_date, everything())
```

## Step 3: Create a tidy date

I converted dates to the date format using `mdy` and created a new
variable named `year` and convert it to a numeric variable. I separated
the original `time_period_label` variable into two variables:
`week_label` and `years` and deleted `years` (because this variable is
unable convert into numeric without causing missing value problem and we
already had a `year` variable).

``` r
tidydata =
  tidydata |> 
  mutate(
    start_dates = mdy(pull(tidydata, start_date)),
    end_dates = mdy(pull(tidydata, end_date))) |> 
  mutate(tpl = time_period_label) |>  
  mutate(year = as.numeric(str_extract(tpl, "\\d{4}")))|> 
  separate(col = time_period_label, into = c("week_label", "years"), sep = ", ", remove = FALSE, extra = "merge") |> 
  select(-time_period_label, -start_date, -end_date) |>
  mutate(week_number = time_period - 12) |>
  select(indicator, year, start_dates, end_dates, week_number, week_label, state, group, subgroup, value, low_ci, high_ci, confidence_interval)
```

## Comment on variable names

There are 13 variables in the `tidydata` data set:

- `indicator` (chr): The type of mental health care.
- `year` (num): The year of the time period for data collection.
- `start_dates` (date): The start date of the time period for data
  collection.
- `end_dates` (date): The end date of the time period for data
  collection.
- `week_number` (num): The week number of data collection.
  - It should be noted that the data set refers to “weeks” as the
    two-week data collection period, so each week number indicates a
    two-week period.
- `week_label` (chr): The dates for the corresponding week.
- `state` (chr): The state or national level data.
- `group` (chr): The demographic group.
- `subgroup` (chr): The specific subgroup within the group.
- `value` (num): The value (percentage or number) representing the
  indicator.
- `low_ci` (num): Lower bound of the confidence interval of the value.
- `high_ci` (num): Upper bound of the confidence interval of the value.
- `confidence_interval` (chr): Confidence interval as a range.
