Access to Mental Health Care in the United States from 2020-2022
================
Sarah Younes (smy2122), Caleigh Dwyer (crd2162), Ashley Romo (ar4459),
Hsin Yi (Cindy) Tseng (ht2607), Wen Dai (wz2629)

``` r
library(tidyverse)
data = read_csv("data/mental_health.csv")
```

*All five team members wrote this proposal collaboratively in our team
Google Docs before converting it to .Rmd. All of our contributions were
equal, as we each proposed one data source and ultimately voted on this
data source, we all were present in the writing of this proposal, and we
all shared ideas for analysis/visualization and suggested coding
challenges.*

### Motivation:

COVID-19 has had serious behavioral and mental health implications. We
are interested in understanding trends and differences in indicators of
mental health care access across the United States from August 2020-May
2022 and disaggregating data by state, age group, sex, race/ethnicity,
and education level, as well as sexual orientation and disability status
in data collected in mid-2021.

### Intended final products:

We will create a website to display our work. The website will include
an embedded screencast as well as a dashboard.

### Anticipated data sources:

We are using the data set from in the .csv file from this data source:
<https://catalog.data.gov/dataset/mental-health-care-in-the-last-4-weeks>.
Data was collected as part of the Pulse Household Survey across the
United States with the U.S. Census Bureau and multiple federal agencies.
The mental health portion of the survey asks respondents to look back in
the last 4 weeks and answer whether they: took prescription medication
for mental health, or needed counseling or therapy but did not get it.

### Planned analyses/visualizations/coding challenges:

- Changes in indicator variables over time (2020-2022): line graphs
- Differences in indicator variables by state: US map visualization
- Differences in indicator variables by identity (age group, sex,
  race/ethnicity, education level, sexual orientation, disability
  status): bar graphs

### Coding challenges:

- Interpreting/merging of confidence intervals
- Tidying columns and rows (e.g., to disaggregate by education level and
  other variables)
- Figuring out how create a US state map visualization

### Planned timeline:

- Week 1: import & data wrangling
- Week 2: exploratory data analysis
- Week 3: graphs & dashboard
- Week 4: filming the screencast & publishing the website
