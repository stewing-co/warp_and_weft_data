---
title: "Tidy Tuesday Customs and Border Protection"
subtitle: ""
excerpt: ""
date: 2024-11-26
author: "Steve Ewing"
draft: false
series:
tags: ["R", "tidy tuesday"]
categories:
layout: single
---
<script src="{{< blogdown/postref >}}index_files/kePrint/kePrint.js"></script>
<link href="{{< blogdown/postref >}}index_files/lightable/lightable.css" rel="stylesheet" />
<script src="{{< blogdown/postref >}}index_files/kePrint/kePrint.js"></script>
<link href="{{< blogdown/postref >}}index_files/lightable/lightable.css" rel="stylesheet" />
<script src="{{< blogdown/postref >}}index_files/kePrint/kePrint.js"></script>
<link href="{{< blogdown/postref >}}index_files/lightable/lightable.css" rel="stylesheet" />

This weeks Tidy Tuesday data is from the Customs and Border Protection. The data is available from 2020 to present. The data is available from the [CBP website](https://www.cbp.gov/document/stats/nationwide-encounters). It was written up first for the [Tidy Tuesday blog](https://www.tidytuesday.com/) by
Tony Galvan [here](https://gdatascience.github.io/us_border_patrol_encounters/us_border_patrol_encounters.html).

"Encounter data includes U.S. Border Patrol Title 8 apprehensions, Office of Field Operations Title 8 inadmissibles, and all Title 42 expulsions for fiscal years 2020 to date. Data is available for the Northern Land Border, Southwest Land Border, and Nationwide (i.e., air, land, and sea modes of transportation) encounters.

Data is extracted from live CBP systems and data sources. Statistical information is subject to change due to corrections, systems changes, change in data definition, additional information, or encounters pending final review. Final statistics are available at the conclusion of each fiscal year." - [CBP](https://www.cbp.gov/document/stats/nationwide-encounters)


``` r
library(tidyverse)
library(janitor)

cbp_resp <- bind_rows(
  read_csv("https://www.cbp.gov/sites/default/files/assets/documents/2023-Nov/nationwide-encounters-fy20-fy23-aor.csv", show_col_types = FALSE),
  read_csv("https://www.cbp.gov/sites/default/files/2024-10/nationwide-encounters-fy21-fy24-aor.csv", show_col_types = FALSE)
) |>
  janitor::clean_names() |>
  unique()

cbp_state <- bind_rows(
  read_csv("https://www.cbp.gov/sites/default/files/assets/documents/2023-Nov/nationwide-encounters-fy20-fy23-state.csv", show_col_types = FALSE),
  read_csv("https://www.cbp.gov/sites/default/files/2024-10/nationwide-encounters-fy21-fy24-state.csv", show_col_types = FALSE)
) |>
  janitor::clean_names() |>
  unique()
```

## Data exploration


``` r
cbp_resp |>
  glimpse()
```

```
## Rows: 68,815
## Columns: 12
## $ fiscal_year            <dbl> 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020,…
## $ month_grouping         <chr> "FYTD", "FYTD", "FYTD", "FYTD", "FYTD", "FYTD",…
## $ month_abbv             <chr> "APR", "APR", "APR", "APR", "APR", "APR", "APR"…
## $ component              <chr> "Office of Field Operations", "Office of Field …
## $ land_border_region     <chr> "Northern Land Border", "Northern Land Border",…
## $ area_of_responsibility <chr> "Boston Field Office", "Boston Field Office", "…
## $ aor_abbv               <chr> "Boston", "Boston", "Boston", "Boston", "Boston…
## $ demographic            <chr> "FMUA", "FMUA", "Single Adults", "Single Adults…
## $ citizenship            <chr> "BRAZIL", "CANADA", "CANADA", "CANADA", "CHINA,…
## $ title_of_authority     <chr> "Title 8", "Title 8", "Title 42", "Title 8", "T…
## $ encounter_type         <chr> "Inadmissibles", "Inadmissibles", "Expulsions",…
## $ encounter_count        <dbl> 3, 1, 2, 239, 1, 0, 1, 6, 1, 1, 1, 18, 52, 1, 2…
```

encounter_type (character) - The category of encounter based on Title of Authority and component (Title 8 for USBP = Apprehensions; Title 8 for OFO = Inadmissibles; Title 42 = Expulsions)


``` r
table(cbp_resp$encounter_type)
```

```
## 
## Apprehensions    Expulsions Inadmissibles 
##         22137          9638         37040
```
component	(character) -	Which part of CBP was involved in the encounter ("Office of Field Operations" or "U.S. Border Patrol")


``` r
table(cbp_resp$component)
```

```
## 
## Office of Field Operations         U.S. Border Patrol 
##                      40483                      28332
```

title_of_authority (character) - The authority under which the noncitizen was processed (Title 8: The standard U.S. immigration law governing the processing of migrants, including deportations, asylum procedures, and penalties for unauthorized border crossings. Title 42: A public health order used during the COVID-19 pandemic to rapidly expel migrants at the border without standard immigration processing, citing health concerns.)


``` r
table(cbp_resp$title_of_authority)
```

```
## 
## Title 42  Title 8 
##     9638    59177
```

What is this month grouping?


``` r
table(cbp_resp$month_grouping)
```

```
## 
##  FYTD 
## 68815
```

Who did they use the title 42 expulsion power on?


``` r
library(kableExtra)
```

```
## 
## Attaching package: 'kableExtra'
```

```
## The following object is masked from 'package:dplyr':
## 
##     group_rows
```

``` r
expulsions <- cbp_resp %>%
  filter(title_of_authority == "Title 42") %>%
  group_by(citizenship, fiscal_year) %>%
  summarise(total = sum(encounter_count), .groups = "drop") %>%
  pivot_wider(names_from = fiscal_year, values_from = total, names_prefix = "FY ") %>%
  arrange(desc(`FY 2023`)) %>%
  mutate(across(starts_with("FY"), ~ format(., big.mark = ",", scientific = FALSE)))

expulsions %>%
  kbl(
    caption = "Title 42 Expulsions by Citizenship and Fiscal Year",
    format = "html",
    digits = 0
  ) %>%
  kable_styling(
    bootstrap_options = c("striped", "hover", "condensed", "responsive"),
    full_width = TRUE,
    font_size = 12
  ) %>%
  add_header_above(c(" " = 1, "Fiscal Years" = ncol(expulsions) - 1)) %>%
  column_spec(1, width = "10em", bold = TRUE) %>% 
  column_spec(2:ncol(expulsions), width = "8em") 
```

<table class="table table-striped table-hover table-condensed table-responsive" style="font-size: 12px; margin-left: auto; margin-right: auto;">
<caption style="font-size: initial !important;">(\#tab:expulsions)Title 42 Expulsions by Citizenship and Fiscal Year</caption>
 <thead>
<tr>
<th style="empty-cells: hide;border-bottom:hidden;" colspan="1"></th>
<th style="border-bottom:hidden;padding-bottom:0; padding-left:3px;padding-right:3px;text-align: center; " colspan="4"><div style="border-bottom: 1px solid #ddd; padding-bottom: 5px; ">Fiscal Years</div></th>
</tr>
  <tr>
   <th style="text-align:left;"> citizenship </th>
   <th style="text-align:left;"> FY 2020 </th>
   <th style="text-align:left;"> FY 2021 </th>
   <th style="text-align:left;"> FY 2022 </th>
   <th style="text-align:left;"> FY 2023 </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;width: 10em; font-weight: bold;"> MEXICO </td>
   <td style="text-align:left;width: 8em; "> 157,952 </td>
   <td style="text-align:left;width: 8em; "> 582,813 </td>
   <td style="text-align:left;width: 8em; "> 693,044 </td>
   <td style="text-align:left;width: 8em; "> 349,685 </td>
  </tr>
  <tr>
   <td style="text-align:left;width: 10em; font-weight: bold;"> GUATEMALA </td>
   <td style="text-align:left;width: 8em; "> 15,161 </td>
   <td style="text-align:left;width: 8em; "> 173,679 </td>
   <td style="text-align:left;width: 8em; "> 154,302 </td>
   <td style="text-align:left;width: 8em; "> 63,098 </td>
  </tr>
  <tr>
   <td style="text-align:left;width: 10em; font-weight: bold;"> HONDURAS </td>
   <td style="text-align:left;width: 8em; "> 17,041 </td>
   <td style="text-align:left;width: 8em; "> 167,388 </td>
   <td style="text-align:left;width: 8em; "> 134,095 </td>
   <td style="text-align:left;width: 8em; "> 54,452 </td>
  </tr>
  <tr>
   <td style="text-align:left;width: 10em; font-weight: bold;"> VENEZUELA </td>
   <td style="text-align:left;width: 8em; "> 49 </td>
   <td style="text-align:left;width: 8em; "> 1,283 </td>
   <td style="text-align:left;width: 8em; "> 707 </td>
   <td style="text-align:left;width: 8em; "> 38,853 </td>
  </tr>
  <tr>
   <td style="text-align:left;width: 10em; font-weight: bold;"> EL SALVADOR </td>
   <td style="text-align:left;width: 8em; "> 5,946 </td>
   <td style="text-align:left;width: 8em; "> 56,769 </td>
   <td style="text-align:left;width: 8em; "> 56,322 </td>
   <td style="text-align:left;width: 8em; "> 21,132 </td>
  </tr>
  <tr>
   <td style="text-align:left;width: 10em; font-weight: bold;"> ECUADOR </td>
   <td style="text-align:left;width: 8em; "> 2,245 </td>
   <td style="text-align:left;width: 8em; "> 54,683 </td>
   <td style="text-align:left;width: 8em; "> 1,155 </td>
   <td style="text-align:left;width: 8em; "> 14,429 </td>
  </tr>
  <tr>
   <td style="text-align:left;width: 10em; font-weight: bold;"> COLOMBIA </td>
   <td style="text-align:left;width: 8em; "> 56 </td>
   <td style="text-align:left;width: 8em; "> 1,069 </td>
   <td style="text-align:left;width: 8em; "> 12,327 </td>
   <td style="text-align:left;width: 8em; "> 13,108 </td>
  </tr>
  <tr>
   <td style="text-align:left;width: 10em; font-weight: bold;"> OTHER </td>
   <td style="text-align:left;width: 8em; "> 1,095 </td>
   <td style="text-align:left;width: 8em; "> 5,532 </td>
   <td style="text-align:left;width: 8em; "> 13,567 </td>
   <td style="text-align:left;width: 8em; "> 7,208 </td>
  </tr>
  <tr>
   <td style="text-align:left;width: 10em; font-weight: bold;"> CUBA </td>
   <td style="text-align:left;width: 8em; "> 4,751 </td>
   <td style="text-align:left;width: 8em; "> 7,229 </td>
   <td style="text-align:left;width: 8em; "> 4,905 </td>
   <td style="text-align:left;width: 8em; "> 4,085 </td>
  </tr>
  <tr>
   <td style="text-align:left;width: 10em; font-weight: bold;"> INDIA </td>
   <td style="text-align:left;width: 8em; "> 310 </td>
   <td style="text-align:left;width: 8em; "> 1,012 </td>
   <td style="text-align:left;width: 8em; "> 5,133 </td>
   <td style="text-align:left;width: 8em; "> 4,072 </td>
  </tr>
  <tr>
   <td style="text-align:left;width: 10em; font-weight: bold;"> NICARAGUA </td>
   <td style="text-align:left;width: 8em; "> 370 </td>
   <td style="text-align:left;width: 8em; "> 3,293 </td>
   <td style="text-align:left;width: 8em; "> 4,158 </td>
   <td style="text-align:left;width: 8em; "> 3,064 </td>
  </tr>
  <tr>
   <td style="text-align:left;width: 10em; font-weight: bold;"> PERU </td>
   <td style="text-align:left;width: 8em; "> 50 </td>
   <td style="text-align:left;width: 8em; "> 1,063 </td>
   <td style="text-align:left;width: 8em; "> 922 </td>
   <td style="text-align:left;width: 8em; "> 2,789 </td>
  </tr>
  <tr>
   <td style="text-align:left;width: 10em; font-weight: bold;"> CANADA </td>
   <td style="text-align:left;width: 8em; "> 538 </td>
   <td style="text-align:left;width: 8em; "> 1,822 </td>
   <td style="text-align:left;width: 8em; "> 3,339 </td>
   <td style="text-align:left;width: 8em; "> 760 </td>
  </tr>
  <tr>
   <td style="text-align:left;width: 10em; font-weight: bold;"> CHINA, PEOPLES REPUBLIC OF </td>
   <td style="text-align:left;width: 8em; "> 95 </td>
   <td style="text-align:left;width: 8em; "> 318 </td>
   <td style="text-align:left;width: 8em; "> 1,401 </td>
   <td style="text-align:left;width: 8em; "> 671 </td>
  </tr>
  <tr>
   <td style="text-align:left;width: 10em; font-weight: bold;"> BRAZIL </td>
   <td style="text-align:left;width: 8em; "> 268 </td>
   <td style="text-align:left;width: 8em; "> 2,587 </td>
   <td style="text-align:left;width: 8em; "> 5,017 </td>
   <td style="text-align:left;width: 8em; "> 484 </td>
  </tr>
  <tr>
   <td style="text-align:left;width: 10em; font-weight: bold;"> HAITI </td>
   <td style="text-align:left;width: 8em; "> 751 </td>
   <td style="text-align:left;width: 8em; "> 10,211 </td>
   <td style="text-align:left;width: 8em; "> 12,358 </td>
   <td style="text-align:left;width: 8em; "> 349 </td>
  </tr>
  <tr>
   <td style="text-align:left;width: 10em; font-weight: bold;"> PHILIPPINES </td>
   <td style="text-align:left;width: 8em; "> 33 </td>
   <td style="text-align:left;width: 8em; "> 107 </td>
   <td style="text-align:left;width: 8em; "> 336 </td>
   <td style="text-align:left;width: 8em; "> 334 </td>
  </tr>
  <tr>
   <td style="text-align:left;width: 10em; font-weight: bold;"> UKRAINE </td>
   <td style="text-align:left;width: 8em; "> 7 </td>
   <td style="text-align:left;width: 8em; "> 21 </td>
   <td style="text-align:left;width: 8em; "> 390 </td>
   <td style="text-align:left;width: 8em; "> 190 </td>
  </tr>
  <tr>
   <td style="text-align:left;width: 10em; font-weight: bold;"> RUSSIA </td>
   <td style="text-align:left;width: 8em; "> 22 </td>
   <td style="text-align:left;width: 8em; "> 69 </td>
   <td style="text-align:left;width: 8em; "> 163 </td>
   <td style="text-align:left;width: 8em; "> 129 </td>
  </tr>
  <tr>
   <td style="text-align:left;width: 10em; font-weight: bold;"> ROMANIA </td>
   <td style="text-align:left;width: 8em; "> 40 </td>
   <td style="text-align:left;width: 8em; "> 98 </td>
   <td style="text-align:left;width: 8em; "> 188 </td>
   <td style="text-align:left;width: 8em; "> 99 </td>
  </tr>
  <tr>
   <td style="text-align:left;width: 10em; font-weight: bold;"> TURKEY </td>
   <td style="text-align:left;width: 8em; "> 3 </td>
   <td style="text-align:left;width: 8em; "> 25 </td>
   <td style="text-align:left;width: 8em; "> 131 </td>
   <td style="text-align:left;width: 8em; "> 93 </td>
  </tr>
  <tr>
   <td style="text-align:left;width: 10em; font-weight: bold;"> MYANMAR (BURMA) </td>
   <td style="text-align:left;width: 8em; "> NA </td>
   <td style="text-align:left;width: 8em; "> 4 </td>
   <td style="text-align:left;width: 8em; "> 6 </td>
   <td style="text-align:left;width: 8em; "> NA </td>
  </tr>
</tbody>
</table>

What did the inadmissibles look like?


``` r
inadmissibles <- cbp_resp |>
  filter(title_of_authority == "Title 8", encounter_type == "Inadmissibles") |>
  group_by(citizenship, fiscal_year) |>
  summarise(total = sum(encounter_count), .groups = "drop") |>
  pivot_wider(names_from = fiscal_year, values_from = total, names_prefix = "FY ") %>%
  arrange(desc(`FY 2023`)) %>%
  mutate(across(starts_with("FY"), ~ format(., big.mark = ",", scientific = FALSE)))

inadmissibles %>%
  kbl(
    caption = "Title 8 Inadmissibles by Citizenship and Fiscal Year",
    format = "html",
    digits = 0
  ) %>%
  kable_styling(
    bootstrap_options = c("striped", "hover", "condensed", "responsive"),
    full_width = TRUE,
    font_size = 12
  ) %>%
  add_header_above(c(" " = 1, "Fiscal Years" = ncol(inadmissibles) - 1))
```

<table class="table table-striped table-hover table-condensed table-responsive" style="font-size: 12px; margin-left: auto; margin-right: auto;">
<caption style="font-size: initial !important;">(\#tab:inadmissibles)Title 8 Inadmissibles by Citizenship and Fiscal Year</caption>
 <thead>
<tr>
<th style="empty-cells: hide;border-bottom:hidden;" colspan="1"></th>
<th style="border-bottom:hidden;padding-bottom:0; padding-left:3px;padding-right:3px;text-align: center; " colspan="5"><div style="border-bottom: 1px solid #ddd; padding-bottom: 5px; ">Fiscal Years</div></th>
</tr>
  <tr>
   <th style="text-align:left;"> citizenship </th>
   <th style="text-align:left;"> FY 2020 </th>
   <th style="text-align:left;"> FY 2021 </th>
   <th style="text-align:left;"> FY 2022 </th>
   <th style="text-align:left;"> FY 2023 </th>
   <th style="text-align:left;"> FY 2024 </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> HAITI </td>
   <td style="text-align:left;"> 739 </td>
   <td style="text-align:left;"> 2,508 </td>
   <td style="text-align:left;"> 25,566 </td>
   <td style="text-align:left;"> 161,155 </td>
   <td style="text-align:left;"> 218,007 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> OTHER </td>
   <td style="text-align:left;"> 42,112 </td>
   <td style="text-align:left;"> 37,392 </td>
   <td style="text-align:left;"> 75,094 </td>
   <td style="text-align:left;"> 142,570 </td>
   <td style="text-align:left;"> 134,973 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> MEXICO </td>
   <td style="text-align:left;"> 47,452 </td>
   <td style="text-align:left;"> 44,790 </td>
   <td style="text-align:left;"> 59,053 </td>
   <td style="text-align:left;"> 138,236 </td>
   <td style="text-align:left;"> 161,877 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> VENEZUELA </td>
   <td style="text-align:left;"> 3,246 </td>
   <td style="text-align:left;"> 2,652 </td>
   <td style="text-align:left;"> 1,917 </td>
   <td style="text-align:left;"> 132,880 </td>
   <td style="text-align:left;"> 178,850 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> UKRAINE </td>
   <td style="text-align:left;"> 7,672 </td>
   <td style="text-align:left;"> 9,315 </td>
   <td style="text-align:left;"> 96,396 </td>
   <td style="text-align:left;"> 101,543 </td>
   <td style="text-align:left;"> 78,557 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> CUBA </td>
   <td style="text-align:left;"> 4,038 </td>
   <td style="text-align:left;"> 805 </td>
   <td style="text-align:left;"> 1,307 </td>
   <td style="text-align:left;"> 78,833 </td>
   <td style="text-align:left;"> 203,929 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> PHILIPPINES </td>
   <td style="text-align:left;"> 45,815 </td>
   <td style="text-align:left;"> 46,235 </td>
   <td style="text-align:left;"> 54,769 </td>
   <td style="text-align:left;"> 51,043 </td>
   <td style="text-align:left;"> 48,381 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> RUSSIA </td>
   <td style="text-align:left;"> 5,895 </td>
   <td style="text-align:left;"> 12,663 </td>
   <td style="text-align:left;"> 30,899 </td>
   <td style="text-align:left;"> 49,641 </td>
   <td style="text-align:left;"> 20,526 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> INDIA </td>
   <td style="text-align:left;"> 18,356 </td>
   <td style="text-align:left;"> 27,088 </td>
   <td style="text-align:left;"> 40,336 </td>
   <td style="text-align:left;"> 49,497 </td>
   <td style="text-align:left;"> 50,681 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> CANADA </td>
   <td style="text-align:left;"> 20,224 </td>
   <td style="text-align:left;"> 20,504 </td>
   <td style="text-align:left;"> 43,727 </td>
   <td style="text-align:left;"> 43,784 </td>
   <td style="text-align:left;"> 44,783 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> NICARAGUA </td>
   <td style="text-align:left;"> 999 </td>
   <td style="text-align:left;"> 837 </td>
   <td style="text-align:left;"> 953 </td>
   <td style="text-align:left;"> 40,846 </td>
   <td style="text-align:left;"> 57,815 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> HONDURAS </td>
   <td style="text-align:left;"> 2,737 </td>
   <td style="text-align:left;"> 11,532 </td>
   <td style="text-align:left;"> 15,231 </td>
   <td style="text-align:left;"> 34,758 </td>
   <td style="text-align:left;"> 32,621 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> CHINA, PEOPLES REPUBLIC OF </td>
   <td style="text-align:left;"> 17,019 </td>
   <td style="text-align:left;"> 22,834 </td>
   <td style="text-align:left;"> 24,376 </td>
   <td style="text-align:left;"> 27,920 </td>
   <td style="text-align:left;"> 40,725 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> COLOMBIA </td>
   <td style="text-align:left;"> 2,366 </td>
   <td style="text-align:left;"> 4,452 </td>
   <td style="text-align:left;"> 5,536 </td>
   <td style="text-align:left;"> 12,668 </td>
   <td style="text-align:left;"> 22,508 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> BRAZIL </td>
   <td style="text-align:left;"> 2,084 </td>
   <td style="text-align:left;"> 1,166 </td>
   <td style="text-align:left;"> 5,618 </td>
   <td style="text-align:left;"> 10,542 </td>
   <td style="text-align:left;"> 5,798 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> EL SALVADOR </td>
   <td style="text-align:left;"> 1,018 </td>
   <td style="text-align:left;"> 3,170 </td>
   <td style="text-align:left;"> 4,352 </td>
   <td style="text-align:left;"> 9,305 </td>
   <td style="text-align:left;"> 11,409 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> GUATEMALA </td>
   <td style="text-align:left;"> 1,408 </td>
   <td style="text-align:left;"> 4,541 </td>
   <td style="text-align:left;"> 4,174 </td>
   <td style="text-align:left;"> 7,747 </td>
   <td style="text-align:left;"> 10,914 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> MYANMAR (BURMA) </td>
   <td style="text-align:left;"> 3,061 </td>
   <td style="text-align:left;"> 3,836 </td>
   <td style="text-align:left;"> 4,462 </td>
   <td style="text-align:left;"> 4,253 </td>
   <td style="text-align:left;"> 4,993 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> TURKEY </td>
   <td style="text-align:left;"> 2,499 </td>
   <td style="text-align:left;"> 3,597 </td>
   <td style="text-align:left;"> 3,980 </td>
   <td style="text-align:left;"> 3,425 </td>
   <td style="text-align:left;"> 2,678 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> PERU </td>
   <td style="text-align:left;"> 1,643 </td>
   <td style="text-align:left;"> 2,012 </td>
   <td style="text-align:left;"> 2,650 </td>
   <td style="text-align:left;"> 3,422 </td>
   <td style="text-align:left;"> 5,195 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ECUADOR </td>
   <td style="text-align:left;"> 871 </td>
   <td style="text-align:left;"> 1,028 </td>
   <td style="text-align:left;"> 832 </td>
   <td style="text-align:left;"> 3,391 </td>
   <td style="text-align:left;"> 7,584 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ROMANIA </td>
   <td style="text-align:left;"> 1,120 </td>
   <td style="text-align:left;"> 1,015 </td>
   <td style="text-align:left;"> 1,245 </td>
   <td style="text-align:left;"> 1,187 </td>
   <td style="text-align:left;"> 1,019 </td>
  </tr>
</tbody>
</table>



What did the apprehensions look like?


``` r
apprehensions <- cbp_resp |>
  filter(title_of_authority == "Title 8", encounter_type == "Apprehensions") |>
  group_by(citizenship, fiscal_year) |>
  summarise(total = sum(encounter_count), .groups = "drop") |>
  pivot_wider(names_from = fiscal_year, values_from = total, names_prefix = "FY ") %>%
  arrange(desc(`FY 2023`)) %>%
  mutate(across(starts_with("FY"), ~ format(., big.mark = ",", scientific = FALSE)))

apprehensions %>%
  kbl(
    caption = "Title 8 Apprehensions by Citizenship and Fiscal Year",
    format = "html",
    digits = 0
  ) %>%
  kable_styling(
    bootstrap_options = c("striped", "hover", "condensed", "responsive"),
    full_width = TRUE,
    font_size = 12
  ) %>%
  add_header_above(c(" " = 1, "Fiscal Years" = ncol(apprehensions) - 1))
```

<table class="table table-striped table-hover table-condensed table-responsive" style="font-size: 12px; margin-left: auto; margin-right: auto;">
<caption style="font-size: initial !important;">(\#tab:apprehensions)Title 8 Apprehensions by Citizenship and Fiscal Year</caption>
 <thead>
<tr>
<th style="empty-cells: hide;border-bottom:hidden;" colspan="1"></th>
<th style="border-bottom:hidden;padding-bottom:0; padding-left:3px;padding-right:3px;text-align: center; " colspan="5"><div style="border-bottom: 1px solid #ddd; padding-bottom: 5px; ">Fiscal Years</div></th>
</tr>
  <tr>
   <th style="text-align:left;"> citizenship </th>
   <th style="text-align:left;"> FY 2020 </th>
   <th style="text-align:left;"> FY 2021 </th>
   <th style="text-align:left;"> FY 2022 </th>
   <th style="text-align:left;"> FY 2023 </th>
   <th style="text-align:left;"> FY 2024 </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> MEXICO </td>
   <td style="text-align:left;"> 103,826 </td>
   <td style="text-align:left;"> 47,136 </td>
   <td style="text-align:left;"> 70,960 </td>
   <td style="text-align:left;"> 248,016 </td>
   <td style="text-align:left;"> 506,211 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> VENEZUELA </td>
   <td style="text-align:left;"> 1,225 </td>
   <td style="text-align:left;"> 46,564 </td>
   <td style="text-align:left;"> 186,896 </td>
   <td style="text-align:left;"> 163,181 </td>
   <td style="text-align:left;"> 134,646 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> GUATEMALA </td>
   <td style="text-align:left;"> 32,867 </td>
   <td style="text-align:left;"> 106,071 </td>
   <td style="text-align:left;"> 74,585 </td>
   <td style="text-align:left;"> 151,004 </td>
   <td style="text-align:left;"> 196,057 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> OTHER </td>
   <td style="text-align:left;"> 4,266 </td>
   <td style="text-align:left;"> 14,433 </td>
   <td style="text-align:left;"> 48,667 </td>
   <td style="text-align:left;"> 149,797 </td>
   <td style="text-align:left;"> 132,602 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> COLOMBIA </td>
   <td style="text-align:left;"> 346 </td>
   <td style="text-align:left;"> 4,974 </td>
   <td style="text-align:left;"> 113,108 </td>
   <td style="text-align:left;"> 141,612 </td>
   <td style="text-align:left;"> 112,168 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> HONDURAS </td>
   <td style="text-align:left;"> 23,579 </td>
   <td style="text-align:left;"> 142,229 </td>
   <td style="text-align:left;"> 65,649 </td>
   <td style="text-align:left;"> 126,818 </td>
   <td style="text-align:left;"> 111,752 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> CUBA </td>
   <td style="text-align:left;"> 5,226 </td>
   <td style="text-align:left;"> 31,269 </td>
   <td style="text-align:left;"> 218,395 </td>
   <td style="text-align:left;"> 117,369 </td>
   <td style="text-align:left;"> 13,686 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ECUADOR </td>
   <td style="text-align:left;"> 9,776 </td>
   <td style="text-align:left;"> 41,363 </td>
   <td style="text-align:left;"> 22,949 </td>
   <td style="text-align:left;"> 99,667 </td>
   <td style="text-align:left;"> 116,439 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> NICARAGUA </td>
   <td style="text-align:left;"> 1,795 </td>
   <td style="text-align:left;"> 46,592 </td>
   <td style="text-align:left;"> 159,489 </td>
   <td style="text-align:left;"> 94,819 </td>
   <td style="text-align:left;"> 33,234 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> PERU </td>
   <td style="text-align:left;"> 391 </td>
   <td style="text-align:left;"> 2,102 </td>
   <td style="text-align:left;"> 49,616 </td>
   <td style="text-align:left;"> 71,991 </td>
   <td style="text-align:left;"> 34,005 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> INDIA </td>
   <td style="text-align:left;"> 1,217 </td>
   <td style="text-align:left;"> 2,562 </td>
   <td style="text-align:left;"> 18,458 </td>
   <td style="text-align:left;"> 43,348 </td>
   <td style="text-align:left;"> 39,734 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> EL SALVADOR </td>
   <td style="text-align:left;"> 10,749 </td>
   <td style="text-align:left;"> 39,524 </td>
   <td style="text-align:left;"> 37,123 </td>
   <td style="text-align:left;"> 32,409 </td>
   <td style="text-align:left;"> 45,798 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> CHINA, PEOPLES REPUBLIC OF </td>
   <td style="text-align:left;"> 1,281 </td>
   <td style="text-align:left;"> 319 </td>
   <td style="text-align:left;"> 1,979 </td>
   <td style="text-align:left;"> 24,109 </td>
   <td style="text-align:left;"> 37,976 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> BRAZIL </td>
   <td style="text-align:left;"> 6,795 </td>
   <td style="text-align:left;"> 54,306 </td>
   <td style="text-align:left;"> 46,386 </td>
   <td style="text-align:left;"> 21,466 </td>
   <td style="text-align:left;"> 26,332 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> TURKEY </td>
   <td style="text-align:left;"> 78 </td>
   <td style="text-align:left;"> 1,367 </td>
   <td style="text-align:left;"> 15,359 </td>
   <td style="text-align:left;"> 15,468 </td>
   <td style="text-align:left;"> 10,633 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> RUSSIA </td>
   <td style="text-align:left;"> 29 </td>
   <td style="text-align:left;"> 508 </td>
   <td style="text-align:left;"> 5,209 </td>
   <td style="text-align:left;"> 7,393 </td>
   <td style="text-align:left;"> 1,365 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ROMANIA </td>
   <td style="text-align:left;"> 326 </td>
   <td style="text-align:left;"> 4,046 </td>
   <td style="text-align:left;"> 5,978 </td>
   <td style="text-align:left;"> 2,406 </td>
   <td style="text-align:left;"> 1,529 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> HAITI </td>
   <td style="text-align:left;"> 3,801 </td>
   <td style="text-align:left;"> 36,008 </td>
   <td style="text-align:left;"> 18,672 </td>
   <td style="text-align:left;"> 2,277 </td>
   <td style="text-align:left;"> 2,791 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> CANADA </td>
   <td style="text-align:left;"> 75 </td>
   <td style="text-align:left;"> 45 </td>
   <td style="text-align:left;"> 60 </td>
   <td style="text-align:left;"> 156 </td>
   <td style="text-align:left;"> 262 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> UKRAINE </td>
   <td style="text-align:left;"> 9 </td>
   <td style="text-align:left;"> 42 </td>
   <td style="text-align:left;"> 591 </td>
   <td style="text-align:left;"> 82 </td>
   <td style="text-align:left;"> 48 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> PHILIPPINES </td>
   <td style="text-align:left;"> 7 </td>
   <td style="text-align:left;"> 11 </td>
   <td style="text-align:left;"> 12 </td>
   <td style="text-align:left;"> 22 </td>
   <td style="text-align:left;"> 40 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> MYANMAR (BURMA) </td>
   <td style="text-align:left;"> 1 </td>
   <td style="text-align:left;"> 1 </td>
   <td style="text-align:left;"> 2 </td>
   <td style="text-align:left;"> 4 </td>
   <td style="text-align:left;"> 11 </td>
  </tr>
</tbody>
</table>

Make a facet plot of the data.


``` r
plot_data <- cbp_resp %>%
  group_by(citizenship, fiscal_year, encounter_type) %>%
  summarise(total = sum(encounter_count, na.rm = TRUE), .groups = 'drop') %>%
  mutate(fiscal_year = as.factor(fiscal_year))

ggplot(plot_data, aes(x = fiscal_year, y = total, fill = encounter_type)) +
  geom_bar(stat = "identity", position = "stack") +  
  facet_wrap(~ citizenship, scales = "free_y") +     
  labs(
    title = "Encounters by Citizenship and Year",
    x = "Fiscal Year",
    y = "Total Encounters",
    fill = "Encounter Type"
  ) +
  theme_minimal() +
  theme(
    strip.text = element_text(size = 8, face = "bold"),  
    axis.text.x = element_text(angle = 45, hjust = 1)    
  )
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-7-1.png" width="672" />
