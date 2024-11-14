---
title: "Bluesky Data in R"
subtitle: ""
excerpt: ""
date: 2024-11-13
author: "Steve Ewing"
draft: false
series:
tags: ["R", "bluesky"]
categories:
layout: single
---

Just needed to add BLUESKY_APP_USER and BLUESKY_APP_PASS to .Renviron. If you are using a custom handle the user name is that. You get the password from the advanced area in settings in the bluesky app.


``` r
#install.packages('bskyr')
library(bskyr)
```

```
## Warning: package 'bskyr' was built under R version 4.4.2
```

``` r
library(tidyverse)

following <- bs_get_follows('warpandweftdata.com', limit = 3000) |>
  mutate(created_at = as.Date(created_at))
```

``` r
# Group by Year-Month and count the number of accounts
monthly_counts <- following %>%
  mutate(year_month = floor_date(created_at, "month")) %>%
  group_by(year_month) %>%
  summarise(accounts_created = n())

# Plotting with ggplot2
ggplot(monthly_counts, aes(x = year_month, y = accounts_created)) +
  geom_line(linewidth = 1) +  # Line plot for continuity
  geom_point(size = 2) + # Points for emphasis on monthly counts
  labs(
    title = "When Did the People I Follow Get Here?",
    x = "Month",
    y = "Number of Accounts Created"
  ) +
  theme_minimal() +
  theme(
    plot.title = element_text(hjust = 0.5, face = "bold"),
    axis.text.x = element_text(angle = 45, hjust = 1)
  ) +
  scale_x_date(date_labels = "%b %Y", date_breaks = "1 month")
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-2-1.png" width="672" />



``` r
bs_post(
  text = "Since I'm posting this from R-Studio I guess bsky already has drafts and scheduling. #Rstats @bskyr.bsky.social"
)
```

```
## # A tibble: 2 × 4
##   uri                                             cid   commit validation_status
##   <chr>                                           <chr> <name> <chr>            
## 1 at://did:plc:2hrlgtim52k5abklvizj7qbn/app.bsky… bafy… <chr>  valid            
## 2 at://did:plc:2hrlgtim52k5abklvizj7qbn/app.bsky… bafy… <chr>  valid
```
