---
title: "Webscraping FedEx Fuel Surcharges"
subtitle: ""
excerpt: ""
date: 2024-11-07
author: "Steve Ewing"
draft: false
series:
tags: ["FedEx", "rvest", "R"]
categories:
layout: single
---

# Introduction

FedEx sets annual contracts with their customers. In these contracts all the costs are fixed except for the fuel surcharge. The fuel surcharge is a percentage of the total cost of the shipment and is adjusted weekly based on the price of fuel. This surcharge is a way for FedEx to pass on the cost of fuel to the customer.

Since its the only moving part in the contract, it is important to keep track of the fuel surcharge. FedEx publishes the fuel surcharge on their website, but it is not in tidy format. This is a simple webscraping project to get the fuel surcharge from the FedEx website and munged into usable data.

# Load Libraries

``` r
library(tidyverse)
```

```
## ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
## ✔ dplyr     1.1.4     ✔ readr     2.1.5
## ✔ forcats   1.0.0     ✔ stringr   1.5.1
## ✔ ggplot2   3.5.1     ✔ tibble    3.2.1
## ✔ lubridate 1.9.3     ✔ tidyr     1.3.1
## ✔ purrr     1.0.2     
## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
## ✖ dplyr::filter() masks stats::filter()
## ✖ dplyr::lag()    masks stats::lag()
## ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors
```

``` r
library(rvest)
```

```
## 
## Attaching package: 'rvest'
## 
## The following object is masked from 'package:readr':
## 
##     guess_encoding
```

# Scrape Current FedEx Fuel Surcharges

``` r
fedex_html <- read_html('https://www.fedex.com/en-us/shipping/fuel-surcharge.html')

fedex_current_tables <- fedex_html %>%
  html_elements("table") %>%
  html_table()
```

# Munge Data

``` r
fedex_current_cleaned <- fedex_current_tables[[1]][-c(1, 2), ] %>%
  mutate(X1 = str_replace_all(X1, 'Sept', 'Sep')) %>%
  select(-X3, -X8) %>%
  rename(
    date_range = X1,
    fedex_ground_fuel_surcharge_rate = X2,
    fedex_express_fuel_surcharge_rate = X4,
    fedex_freight_fuel_surcharge_rate = X5,
    fedex_export_fuel_surcharge_rate = X6,
    fedex_import_fuel_surcharge_rate = X7
  ) %>%
  mutate(
    start_date = mdy(str_split(date_range, '–', simplify = TRUE)[, 1]),
    end_date = mdy(str_split(date_range, '–', simplify = TRUE)[, 2]),
    fedex_ground_fuel_surcharge_rate = as.numeric(str_remove(fedex_ground_fuel_surcharge_rate, '%')) / 100,
    fedex_express_fuel_surcharge_rate = as.numeric(str_remove(fedex_express_fuel_surcharge_rate, '%')) / 100,
    fedex_import_fuel_surcharge_rate = as.numeric(str_remove(fedex_import_fuel_surcharge_rate, '%')) / 100,
    fedex_export_fuel_surcharge_rate = as.numeric(str_remove(fedex_export_fuel_surcharge_rate, '%')) / 100,
    fedex_freight_fuel_surcharge_per_lbs = as.numeric(str_remove_all(fedex_freight_fuel_surcharge_rate, '\\$| per lb\\.'))
  )
```

# Scraping Historical FedEx Fuel Surcharge Rates


``` r
fedex_history_html <- read_html("https://www.fedex.com/en-us/shipping/historical-fuel-surcharge.html")

fedex_history_tables <- fedex_history_html %>%
  html_elements("table") %>%
  html_table()
```

# Munge Historical Data


``` r
fedex_history_cleaned <- fedex_history_tables[[1]][-c(1, 2), ] %>%
  mutate(X1 = str_replace_all(X1, 'Sept', 'Sep')) %>%
  rename(
    date_range = X1,
    fedex_ground_fuel_surcharge_rate = X2,
    fedex_express_fuel_surcharge_rate = X3,
    fedex_freight_fuel_surcharge_rate = X4,
    fedex_export_fuel_surcharge_rate = X5,
    fedex_import_fuel_surcharge_rate = X6
  ) %>%
  mutate(
    start_date = mdy(str_split(date_range, '–', simplify = TRUE)[, 1]),
    end_date = mdy(str_split(date_range, '–', simplify = TRUE)[, 2]),
    fedex_ground_fuel_surcharge_rate = as.numeric(str_remove(fedex_ground_fuel_surcharge_rate, '%')) / 100,
    fedex_express_fuel_surcharge_rate = as.numeric(str_remove(fedex_express_fuel_surcharge_rate, '%')) / 100,
    fedex_import_fuel_surcharge_rate = as.numeric(str_remove(fedex_import_fuel_surcharge_rate, '%')) / 100,
    fedex_export_fuel_surcharge_rate = as.numeric(str_remove(fedex_export_fuel_surcharge_rate, '%')) / 100,
    fedex_freight_fuel_surcharge_per_lbs = as.numeric(str_remove_all(fedex_freight_fuel_surcharge_rate, '\\$| per lb\\.'))
  )
```

# Combine Current and Historical Data


``` r
fedex_rates <- bind_rows(fedex_current_cleaned, fedex_history_cleaned) %>%
  filter(!is.na(fedex_ground_fuel_surcharge_rate))
```

# Expanding Date Ranges to Daily Rates


``` r
fedex_full_table <-
    tibble(
        date = date(),
        fedex_ground_fuel_surcharge_rate = numeric(),
        fedex_express_fuel_surcharge_rate = numeric(),
        fedex_import_fuel_surcharge_rate = numeric(),
        fedex_export_fuel_surcharge_rate = numeric(),
        fedex_freight_fuel_surcharge_rate = numeric()
        ,
    )

for (i in 1:dim(fedex_rates)[1]) {
    fedex2 <-
        tibble(
            date = seq(fedex_rates$start_date[i], fedex_rates$end_date[i], by = 'days'),
            fedex_ground_fuel_surcharge_rate = fedex_rates$fedex_ground_fuel_surcharge_rate[i],
            fedex_express_fuel_surcharge_rate = fedex_rates$fedex_express_fuel_surcharge_rate[i],
            fedex_import_fuel_surcharge_rate = fedex_rates$fedex_import_fuel_surcharge_rate[i],
            fedex_export_fuel_surcharge_rate = fedex_rates$fedex_export_fuel_surcharge_rate[i],
            fedex_freight_fuel_surcharge_rate = fedex_rates$fedex_freight_fuel_surcharge_rate[i]
        )
    fedex_full_table <- rbind(fedex_full_table, fedex2)
}
```

# Visualizing the Fuel Surcharge Rates


``` r
# Reshape the data for plotting
fedex_plot_data <- fedex_full_table %>%
  select(date, fedex_ground_fuel_surcharge_rate, fedex_express_fuel_surcharge_rate, fedex_import_fuel_surcharge_rate, fedex_export_fuel_surcharge_rate) %>%
  pivot_longer(cols = -date, names_to = "surcharge_type", values_to = "rate") %>%
  mutate(surcharge_type = recode(surcharge_type,
    fedex_ground_fuel_surcharge_rate = "Ground",
    fedex_express_fuel_surcharge_rate = "Express",
    fedex_import_fuel_surcharge_rate = "Import",
    fedex_export_fuel_surcharge_rate = "Export"
  )) |>
  filter(surcharge_type %in% c("Ground", "Express"))

# Create the stepped line graph
ggplot(fedex_plot_data, aes(x = date, y = rate, color = surcharge_type)) +
  geom_step(linewidth = 1.5) +
  labs(
    title = "FedEx Fuel Surcharge Rates Over Time",
    x = "Date",
    y = "Fuel Surcharge Rate",
    color = "Surcharge Type"
  ) +
  scale_y_continuous(labels = scales::percent) 
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-8-1.png" width="672" />

