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


``` r
library(tidyverse)
library(ISOcodes)
library(janitor)
library(rnaturalearth)
library(showtext)

font_add_google("Pirata One", "pirata")
showtext_auto()

countries <- ISOcodes::ISO_3166_1 |> 
  tibble::as_tibble() |> 
  janitor::clean_names()

country_subdivisions <- 
  ISOcodes::ISO_3166_2 |> 
  tibble::as_tibble() |> 
  janitor::clean_names() |> 
  dplyr::mutate(
    alpha_2 = stringr::str_extract(code, "^[^-]+(?=-)")
  )

# Load world map data
world_data <- ne_countries(scale = "medium", returnclass = "sf")

# Join world_data with countries to get names for labeling
no_sub_countries_map <- world_data |>
  left_join(countries, by = c("iso_a2" = "alpha_2")) |>
  filter(!is.na(brk_name),
         !iso_a2 %in% country_subdivisions$alpha_2)  

# Plot the map
ggplot(no_sub_countries_map) +
  geom_sf(fill = "#8b4513", color = "#4b2e2a", size = 0.4) +          
  geom_sf_text(aes(label = brk_name), size = 3, color = "#332f2c", check_overlap = TRUE, fontface = "italic", family = "pirata") + 
  theme(
    plot.background = element_rect(fill = "#add8e6", color = NA),    
    plot.title = element_text(size = 20, face = "bold", family = "pirata", hjust = 0.5, color = "#4b2e2a"),
    plot.margin = margin(20, 20, 20, 20)                             
  ) +
  labs(title = "Countries Without Subdivisions", x = NULL, y = NULL)
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-1-1.png" width="672" />
