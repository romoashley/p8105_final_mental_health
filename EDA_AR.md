EDA_AR
================
ASHLEY ROMO
2023-11-26

Loading key packages

``` r
library(tidyverse)
```

Importing clean and tidy dataset

``` r
data = 
  read.csv("data/tidydata.csv") |> 
  mutate(
    start_dates = as.Date(start_dates)
  )
```

## EDA for National Average

``` r
avg_year =
  data |> 
  filter(state == "United States") |> 
  select(indicator, year, week_number, value, start_dates) |> 
  group_by(year, start_dates, indicator) |> 
  summarize(
    mean = mean(value),
    indicator_total = n())
```

    ## `summarise()` has grouped output by 'year', 'start_dates'. You
    ## can override using the `.groups` argument.

``` r
# plot average for each start date for each indicator
avg_year |> 
  ggplot(aes(x = start_dates, y = mean, color = indicator)) +
  geom_line() +
  labs(
    x = "Start Date",
    y = "Average Value",
    title = "National Average (2020-2022)") +
  theme(
    legend.position = "bottom",
    axis.text.x = element_text(angle=90, hjust=1),
    strip.text = element_text(size = 4)) +
  guides(color = guide_legend(nrow = 4))
```

![](EDA_AR_files/figure-gfm/unnamed-chunk-3-1.png)<!-- -->

On average, those who took prescription medication for mental health
and/or received counseling or therapy had a highest values while those
who received counseling or therapy had the lowest value.

## Confidence Intervals

I relabeled the observations for the variable indicator to “received” if
they did obtain the mental health services they needed and “not
received” if they did not obtain the mental health services they needed.

``` r
received_service = 
  data |> 
  mutate(
    resolution = case_when(
      indicator == "Took Prescription Medication for Mental Health" ~ "received",
      indicator == "Received Counseling or Therapy" ~ "received",
      indicator == "Took Prescription Medication for Mental Health And/Or Received Counseling or Therapy" ~ "received",
      indicator == "Needed Counseling or Therapy But Did Not Get It" ~ "not received"))
```

I created a data frame where I grouped by states to determine the number
of total services, which includes the services received as well as the
number of services not received.

``` r
states_df =
  received_service |> 
  select(state, indicator, resolution) |> 
  filter(state != "United States") |> 
  group_by(state) |> 
  summarize(
    services_total = n(),
    services_not_received = sum(resolution == "not received")) |> 
  mutate(
    services_total = as.numeric(services_total),
    services_not_received = as.numeric(services_not_received)
  )
```

Now, I focus on the state of New York. Using the `prop.test` and
`broom::tidy` functions, I obtain an estimate and CI of the proportion
of mental health services not received in New York (shown in the table
below).

``` r
ny_test = 
  prop.test(
    x = filter(states_df, state == "New York") %>% pull(services_not_received),
    n = filter(states_df, state == "New York") %>% pull(services_total)) 

broom::tidy(ny_test) %>% 
  knitr::kable(digits = 3)
```

| estimate | statistic | p.value | parameter | conf.low | conf.high | method                                               | alternative |
|---------:|----------:|--------:|----------:|---------:|----------:|:-----------------------------------------------------|:------------|
|     0.25 |    32.008 |       0 |         1 |    0.181 |     0.334 | 1-sample proportions test with continuity correction | two.sided   |

I apply prop.test and broom:tidy to obtain estimates and confidence
intervals for the proportion of mental health services not received for
each state.

``` r
services_states =
  states_df |> 
  mutate(
    prop_tests = map2(services_not_received, services_total, \(x, y) prop.test(x = x, n = y)),
    tidy_tests = map(prop_tests, broom::tidy)) |> 
  select(-prop_tests) |> 
  unnest(tidy_tests) |> 
  select(state, estimate, conf.low, conf.high) |> 
  mutate(state = fct_reorder(state, estimate))
```

Lastly, below is a plot showing the estimate (and CI) of the proportion
of mental health services not received in each state.

``` r
services_states %>% 
  mutate(state = fct_reorder(state, estimate)) %>% 
  ggplot(aes(x = state, y = estimate)) + 
  geom_point() + 
  geom_errorbar(aes(ymin = conf.low, ymax = conf.high)) + 
  theme(axis.text.x = element_text(angle = 90, hjust = 1)) +
  labs(
    x = "State",
    y = "Estimate",
    title = "Proportion of Mental Health Services Not Received by States"
  )
```

![](EDA_AR_files/figure-gfm/unnamed-chunk-8-1.png)<!-- -->

The majority of states have a similar estimate of proportion of mental
health services not received in each states. Vermont, Wyoming, Hawaii,
and North Dakota had distinct estimates, with a lower estimate compared
to the remaining states. Vermont had the lowest estimate.
