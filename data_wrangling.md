Data import and Wrangling
================

``` r
library (tidyverse)
```

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

## Step 3: Tidy the dates

First, I created an indicator for `week_number` using `match` to assign
a number to the position of the week number. Then, I converted dates to
the date format using `mdy`. Next, I created a new variable named `year`
and converted it to a numeric variable. Finally, I separated the
original `time_period_label` variable into two variables: `week_label`
and `years` and selected the variables I need (excluding `years` because
this variable is unable convert into numeric without causing missing
value problem and we already had a `year` variable).

``` r
tidydata =
  tidydata |>
  mutate(week_number = match(time_period_label, unique(time_period_label))) |>
  mutate(
    start_dates = mdy(pull(tidydata, start_date)),
    end_dates = mdy(pull(tidydata, end_date))) |> 
  mutate(tpl = time_period_label) |>  
  mutate(year = as.numeric(str_extract(tpl, "\\d{4}")))|> 
  separate(col = time_period_label, into = c("week_label", "years"), sep = ", ", remove = FALSE, extra = "merge") |>
  select(indicator, year, start_dates, end_dates, week_number, week_label = time_period_label, state, group, subgroup, value, low_ci, high_ci, confidence_interval)
```

# Step 4: exports the tidy data to a csv file named tidydata.

``` r
 write.csv(tidydata, "data/tidydata.csv", row.names=FALSE)
```

## Comment on variable names

There are 13 variables in the `tidydata` data set:

- `indicator` (chr): The type of mental health care.
- `year` (num): The year of the time period for data collection.
  - If the time period for data collection continued onto the following
    year, the year that data collection began was used for this
    variable.
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

# Step 5: Extra Data wrangling:

1)  Separate the dataset into four groups based on the indicator’s
    categories.

``` r
Indicator1=tidydata |> filter(indicator=="Took Prescription Medication for Mental Health")
Indicator2=tidydata |> filter(indicator=="  
Received Counseling or Therapy")
Indicator3=tidydata |> filter(indicator=="Took Prescription Medication for Mental Health And/Or Received Counseling or Therapy")
Indicator2=tidydata |> filter(indicator=="  
Needed Counseling or Therapy But Did Not Get It")
```

2.  Did some data cleanning for Indicator1.Break the dataset Indicator1
    into different smaller dataset based on the different categories
    from the subgroup. Indicator1 is the people who Took Prescription
    Medication for Mental Health.

``` r
age_data=Indicator1 |> filter(group=="Age")
race_data=Indicator1 |> filter(group=="Race/Hispanic ethnicity")
sex_data=Indicator1 |> filter(group=="Sex")
education_data=Indicator1 |> filter(group=="Education")
state_data=Indicator1 |> filter(group=="State")
Symptoms_data=Indicator1 |> filter(group=="Presence of Symptoms of Anxiety/Depression")
```
