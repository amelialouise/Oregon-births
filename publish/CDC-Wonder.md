Extracting CDC Data
================

This notebook documents natality data downloaded from
<a href="https://wonder.cdc.gov/natality.html" class="uri">CDC
Wonder</a> and [data.cdc.gov](https://data.cdc.gov/).

All raw, downloaded CDC data are stored in this project’s **/data/**
folder.

``` r
library(here)
library(dplyr)
library(vroom)
```

# Expanded Natality data (2016 - 2019)

Here are the latest [technical
notes](https://www.cdc.gov/nchs/nvss/vsrr/natality-technical-notes.htm)
on CDC’s Natality data and the [CDC data
dictionary](https://wonder.cdc.gov/wonder/help/Natality-expanded.html#)

## Low-risk deliveries

This dataset is inspired by the Maternal, Infant, and Child Health
objective identified by [Healthy People
2030](https://health.gov/healthypeople) to *reduce cesarean births among
low-risk women with no prior births* to a target of 23.6% nationwide. A
low-risk birth is
[defined](https://health.gov/healthypeople/objectives-and-data/browse-objectives/pregnancy-and-childbirth/reduce-cesarean-births-among-low-risk-women-no-prior-births-mich-06/data-methodology)
as

- **nulliparous**: first birth,

- **singleton**: a single fetus (not multiple),

- **term**: at least 37 weeks of gestation based on obstetric estimate
  of gestation at delivery, and

- **vertex**: not breech / head is facing in a downward position for
  delivery.

We’ll first get national totals for low-risk cesarean births by
selecting the following criteria in CDC Wonder:

In section 10. “Select delivery characteristics”

- **Fetal Presentation**: Cephalic

- **Final Route and Delivery Method**: Cesarean

- **Delivery Method Expanded**: Primary C-section; Repeat C-section;
  C-section (unknown if previous c-section)

In section 5. “Select pregnancy history and prenatal care
characteristics”

- **Live Birth Order**: 1

In section 12. “Select infant characteristics”

- **OE Gestational Age Recode 11**: 37-38 weeks; 39 weeks; 40 weeks; 41
  weeks; 42 weeks or more

- **Plurality**: Single

This yields the **US Natality, 2016-2021 expanded_low-risk
cesarean.txt** data file stored in **/data/**.

    ##   Year Cesarean_births
    ## 1 2021         316,349
    ## 2 2020         310,303
    ## 3 2019         314,016
    ## 4 2018         319,022
    ## 5 2017         325,086
    ## 6 2016         329,614

These match the low-risk cesarean totals reported in the [Jan. 2023
National Vital Statistics
Report](https://www.cdc.gov/nchs/data/nvsr/nvsr72/nvsr72-01.pdf). Next,
we’ll get total low-risk births so that we can calculate the rate of
low-risk cesareans among these. This data is obtained by removing the
filters for delivery method:

- **Fetal Presentation**: Cephalic

- **Live Birth Order**: 1

- **OE Gestational Age Recode 11**: 37-38 weeks; 39 weeks; 40 weeks; 41
  weeks; 42 weeks or more

- **Plurality**: Single

``` r
low_risk_all_deliveries_totals <- 
  vroom(
    here("data", "US Natality, 2016-2021 expanded_low-risk births.txt"),
    n_max = 6,
    col_types = cols("c", "i", "i", "i"),
    delim = "\t"
  ) %>% 
  select(Year, Births) %>%  
  as.data.frame()

low_risk_all_deliveries_totals %>%
  mutate(Births = prettyNum(Births, big.mark = ",")) %>% 
  arrange(desc(Year))
```

    ##   Year    Births
    ## 1 2021 1,204,358
    ## 2 2020 1,198,613
    ## 3 2019 1,226,476
    ## 4 2018 1,231,332
    ## 5 2017 1,250,875
    ## 6 2016 1,280,607

We can then join these into a single dataset with totals and calculate
low-risk rates.

``` r
df_low_risk_births <- 
  left_join(
    low_risk_all_deliveries_totals, 
    low_risk_cesarean_totals, 
    by = "Year"
  )

df_low_risk_births %>% 
  mutate(
    low_risk_cesarean_rate = scales::percent(Cesarean_births/Births, accuracy = .1),
    Births = prettyNum(Births, big.mark = ","),
    Cesarean_births = prettyNum(Cesarean_births, big.mark = ",")
  ) %>% 
  arrange(desc(Year))
```

    ##   Year    Births Cesarean_births low_risk_cesarean_rate
    ## 1 2021 1,204,358         316,349                  26.3%
    ## 2 2020 1,198,613         310,303                  25.9%
    ## 3 2019 1,226,476         314,016                  25.6%
    ## 4 2018 1,231,332         319,022                  25.9%
    ## 5 2017 1,250,875         325,086                  26.0%
    ## 6 2016 1,280,607         329,614                  25.7%

For the latest rates, visit the [NCHS - VSRR Quarterly provisional
estimates for selected birth
indicators](https://data.cdc.gov/d/76vv-a7x8) dataset on socrata. This
does not include totals but has cesarean and low-risk cesarean birth
rates at the national level by race/ethnicity.
