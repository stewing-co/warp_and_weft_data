---
title: "Tidy Tuesday ISO Codes"
subtitle: ""
excerpt: ""
date: 2024-11-11
author: "Steve Ewing"
draft: false
series:
tags: ["maps", "R", "tidy tuesday"]
categories:
layout: single
---

It has been said that "all politics is local." It may feel like that to national candidates and their teams but for the median guy on the street national politics can seem really far away. 

Some places in the world all politics really is local politics. In this weeks Tidy Tuesday I'll look at the countries that don't have ISO_3166_2 records.


``` r
library(tidyverse)
library(ISOcodes)
library(janitor)
library(rnaturalearth)
library(showtext)

font_add_google("Pirata One", "pirata")
showtext_auto()

countries <- ISO_3166_1 |> 
  as_tibble() |> 
  clean_names()

country_subdivisions <- 
  ISO_3166_2 |> 
  as_tibble() |> 
  clean_names() |> 
  mutate(
    alpha_2 = stringr::str_extract(code, "^[^-]+(?=-)")
  )

world_data <- ne_countries(scale = "medium", returnclass = "sf")

no_sub_countries <- world_data |>
  inner_join(countries, by = c("iso_a2_eh" = "alpha_2")) |>
  filter(!is.na(brk_name),
         !iso_a2_eh %in% country_subdivisions$alpha_2)  

ggplot(no_sub_countries) +
  geom_sf(aes(fill = type)) +          
  geom_sf_text(aes(label = brk_name), size = 4, color = "#332f2c", check_overlap = TRUE, fontface = "italic", family = "pirata") + 
  theme(
    plot.background = element_rect(fill = "#add8e6", color = NA),    
    plot.title = element_text(size = 20, face = "bold", family = "pirata", hjust = 0.5, color = "#4b2e2a"),
    plot.margin = margin(20, 20, 20, 20)                             
  ) +
  labs(title = '"Countries" Without Subdivisions', x = NULL, y = NULL)
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-1-1.png" width="672" />

Looks like a bunch of these places aren't even countries per se.


``` r
# Calculate the count of each type
type_counts <- no_sub_countries %>%
  count(type)

# Plot the bar chart
ggplot(type_counts, aes(x = type, y = n, fill = type)) +
  geom_bar(stat = "identity", color = "#4b2e2a", linewidth = 0.3) +    
  theme_minimal() +
  labs(
    title = "Count of Countries by Type",
    x = "Country Type",
    y = "Count"
  )
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-2-1.png" width="672" />

We want only actual countries, what's the list?


``` r
no_sub_2 <- no_sub_countries |>
  filter(type %in% c('Country', 'Sovereign Country'))
 
no_sub_2$name_long 
```

```
## [1] "Jersey"        "Guernsey"      "Isle of Man"   "Aruba"        
## [5] "Curaçao"       "Åland Islands" "Macao"         "Hong Kong"    
## [9] "Sint Maarten"
```

Gah, those aren't independant. They all have soverigns!!


``` r
no_sub_2 |> select(sovereignt, name_long) 
```

```
## Simple feature collection with 9 features and 2 fields
## Geometry type: MULTIPOLYGON
## Dimension:     XY
## Bounding box:  xmin: -70.06611 ymin: 12.04546 xmax: 114.3353 ymax: 60.40581
## Geodetic CRS:  WGS 84
##       sovereignt     name_long                       geometry
## 1 United Kingdom        Jersey MULTIPOLYGON (((-2.018652 4...
## 2 United Kingdom      Guernsey MULTIPOLYGON (((-2.512305 4...
## 3 United Kingdom   Isle of Man MULTIPOLYGON (((-4.412061 5...
## 4    Netherlands         Aruba MULTIPOLYGON (((-69.89912 1...
## 5    Netherlands       Curaçao MULTIPOLYGON (((-68.75107 1...
## 6        Finland Åland Islands MULTIPOLYGON (((19.98955 60...
## 7          China         Macao MULTIPOLYGON (((113.4789 22...
## 8          China     Hong Kong MULTIPOLYGON (((114.0154 22...
## 9    Netherlands  Sint Maarten MULTIPOLYGON (((-63.12305 1...
```

what next...
