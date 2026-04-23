# Purpose

This tutorial generates two figures related to government debt and its
relation to real GDP growth. The figures are created for BRICS (Brazil,
Russia, India, China and South Africa). All data is sourced from the
FRED API. All the code to recreate the results is displayed in this
README, but reproduction will require setting your own API key.

# Set-up

Data is obtained from FRED as follows:

``` r
library(fredr)
fredr_set_key("f287f35d692bc08c79b8d88c6ee3ba2d")

start_date <- as.Date("2000-01-01")

# First, Gross Government Debt as a % of GDP
debt_bra <- fredr(series_id = "GGGDTABRA188N", observation_start = start_date)
debt_rus <- fredr(series_id = "GGGDTARUA188N", observation_start = start_date)
debt_ind <- fredr(series_id = "GGGDTAINA188N", observation_start = start_date)
debt_chn <- fredr(series_id = "GGGDTACNA188N", observation_start = start_date)
debt_zaf <- fredr(series_id = "GGGDTAZAA188N", observation_start = start_date)

# Then, Real GDP 
gdp_bra <- fredr(series_id = "NGDPRXDCBRA", observation_start = start_date)
gdp_rus <- fredr(series_id = "NGDPRXDCRUA", observation_start = start_date)
gdp_ind <- fredr(series_id = "NGDPRXDCINA", observation_start = start_date)
gdp_chn <- fredr(series_id = "NGDPRXDCCNA", observation_start = start_date)
gdp_zaf <- fredr(series_id = "NGDPRXDCZAA", observation_start = start_date)
```

# Formatting

In order to ready the data for merging and variable creation, I first
combined the debt and gdp data.

``` r
# Combining the debt data for each country
debt_panel <- bind_rows(
  debt_bra |> mutate(country = "Brazil"),
  debt_rus |> mutate(country = "Russia"),
  debt_ind |> mutate(country = "India"),
  debt_chn |> mutate(country = "China"),
  debt_zaf |> mutate(country = "South Africa")
) |>
  select(country, date, value) |>
  rename(debt_gdp = value)

# Combining the gdp data for each country
gdp_panel <- bind_rows(
  gdp_bra |> mutate(country = "Brazil"),
  gdp_rus |> mutate(country = "Russia"),
  gdp_ind |> mutate(country = "India"),
  gdp_chn |> mutate(country = "China"),
  gdp_zaf |> mutate(country = "South Africa")
) |>
  select(country, date, value) |>
  rename(gdp = value)
```

Them, I calculated GDP growth as follows:

``` r
gdp_growth_panel <- gdp_panel |> 
  group_by(country) |> 
  mutate(
    gdp_growth = (gdp/lag(gdp) - 1) * 100 # gdp growth is calculated as a percentage
  )|> 
  ungroup()
```

Then, I merged all the data into a combined panel.

``` r
# data is merged with a left_join
brics <- debt_panel |>
  left_join(
    gdp_growth_panel, 
    by = c("country", "date")
  )
```

# Generating graphs

## General Government Gross Debt (% of GDP) for BRICS Countries

This figure illustrates that most of the BRICS countries have
experienced a rise in government debt as a percentage of GDP since 2000.
Russia appears to be the only country bucking this trend, although it
too has seen a growth in debt since 2010. All countries experienced a
upward spike in debt during the COVID-19 pandemic.

``` r
brics |> 
  ggplot(
    aes(x = date, y = debt_gdp, colour = country)
  ) +
  geom_line() +
  labs(
    title = "Gross Debt (% of GDP) for BRICS Countries from 2000-2024",
    x = NULL,
    y = "debt as a % of GDP"
  ) +
  theme_minimal()
```

![](README_files/figure-markdown_github/unnamed-chunk-6-1.png)

## The Relationship Between Debt & GDP Growth for BRICS Countries

This figure illustrates the relationship between government debt as a
percentage of GDP and real GDP growth. Nearly all countries exhibit a
negative relationship between government debt and GDP growth, except for
Russia.

``` r
brics |> 
  ggplot(
    aes(x = debt_gdp, y = gdp_growth, colour = country)
  ) +
  geom_point() +
  geom_smooth(method = "lm", se = FALSE) +
  labs(
    title = "Gross Debt (% of GDP) vs Real GDP Growth",
    x = "debt as a % of GDP",
    y = "Real GDP Growth"
  ) +
  theme_minimal()
```

![](README_files/figure-markdown_github/unnamed-chunk-7-1.png)
