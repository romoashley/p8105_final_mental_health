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

# Step 2: create a tidy date.

1)  Create a new column named year and convert it into numeric data.
2)  Convert dates to date format and change separate the
    time_period_label into two columns: month_period_label and years.
    Then delete years (because this years variable is unable convert
    into numeric without causing missing value problem, so I have to
    delete it)
3)  make a comment.

``` r
tidydata=rawdata |> 
  mutate(start_dates=mdy(pull(rawdata, start_date)),
         end_dates=mdy(pull(rawdata, end_date)))|> 
        mutate(tpl=time_period_label) |>  
        mutate(year = as.numeric(str_extract(tpl, "\\d{4}")))|> 
         separate(col=time_period_label, into=c("month_period_label","years"), sep = ", ", remove = FALSE,extra="merge") |> 
  select(-time_period_label,-start_date,-end_date) |>
  mutate(week_number=time_period-12) |> select(indicator,year,start_dates,end_dates,week_number,month_period_label,group,state,subgroup,value,low_ci,high_ci,confidence_interval)
```

### Comment:

There are 13 columns in the rawdata:

- Indicator (chr): The type of mental health care.
- year(num): year for the time period.
- Start Date (Date): Start date of the time period.
- End Date(Date): End date of the time period.
- Week_number(num):Numeric code for the time period minus 12.
- Month Period Label(chr): Label for the month time period.
- Group(chr): The demographic group.
- State(chr): The state or national level data.
- Subgroup(chr): The specific subgroup within the group.
- Value(num): The value (percentage or number) representing the
  indicator.
- LowCI(num): Lower bound of the confidence interval.
- HighCI(num): Upper bound of the confidence interval.
- Confidence Interval(chr): Confidence interval as a range.
