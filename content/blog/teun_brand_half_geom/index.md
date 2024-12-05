---
title: "Teun van den Brand's Plots"
subtitle: "A collection of plots from Teun van den Brand's github"
excerpt: ""
date: 2024-12-05
author: "Teun van den Brand"
draft: false
series:
tags: ["R", "ggplot2"]
categories: 
layout: single
---

[Here's the source](https://github.com/teunbrand/ggplot_tricks)


``` r
library(ggplot2)
library(scales)

theme_set(
  # Pick a starting theme
  theme_gray() +
  # Add your favourite elements
  theme(
    axis.line        = element_line(),
    panel.background = element_rect(fill = "white"),
    panel.grid.major = element_line("grey95", linewidth = 0.25),
    legend.key       = element_rect(fill = NA) 
  )
)

my_mapping <- aes(x = foo, y = bar)

aes(colour = qux, !!!my_mapping)
```

```
## Aesthetic mapping: 
## * `x`      -> `foo`
## * `y`      -> `bar`
## * `colour` -> `qux`
```

``` r
#> Aesthetic mapping: 
#> * `x`      -> `foo`
#> * `y`      -> `bar`
#> * `colour` -> `qux`

my_fill <- aes(fill = after_scale(alpha(colour, 0.3)))

ggplot(mpg, aes(displ, hwy)) +
  geom_point(aes(colour = factor(cyl), !!!my_fill), shape = 21)
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-1-1.png" width="672" />


``` r
contrast <- function(colour) {
  out   <- rep("black", length(colour))
  light <- farver::get_channel(colour, "l", space = "hcl")
  out[light < 50] <- "white"
  out
}

autocontrast <- aes(colour = after_scale(contrast(fill)))

cors <- cor(mtcars)

# Melt matrix
df <- data.frame(
  col = colnames(cors)[as.vector(col(cors))],
  row = rownames(cors)[as.vector(row(cors))],
  value = as.vector(cors)
)

# Basic plot
p <- ggplot(df, aes(row, col, fill = value)) +
  geom_raster() +
  geom_text(aes(label = round(value, 2), !!!autocontrast)) +
  coord_equal()

p + scale_fill_viridis_c(direction =  1)
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-2-1.png" width="672" />

``` r
p + scale_fill_viridis_c(direction = -1)
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-2-2.png" width="672" />


``` r
# A basic plot to reuse for examples
p <- ggplot(mpg, aes(class, displ, colour = class, !!!my_fill)) +
  guides(colour = "none", fill = "none") +
  labs(y = "Engine Displacement [L]", x = "Type of car")

p + geom_boxplot(aes(xmin = after_scale(x)))
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-3-1.png" width="672" />


``` r
p + geom_errorbar(
  stat = "summary",
  fun.data = mean_se,
  aes(xmin = after_scale(x))
)
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-4-1.png" width="672" />


``` r
update_geom_defaults("violin", list(xmin = NULL))

p + geom_violin(aes(xmin = after_scale(x)))
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-5-1.png" width="672" />


``` r
# A small nudge offset
offset <- 0.025

# We can pre-specify the mappings if we plan on recycling some
right_nudge <- aes(
  xmin = after_scale(x), 
  x = stage(class, after_stat = x + offset)
)
left_nudge  <- aes(
  xmax = after_scale(x),
  x = stage(class, after_stat = x - offset)
)

# Combining
p +
  geom_violin(right_nudge) +
  geom_boxplot(left_nudge) +
  geom_errorbar(left_nudge, stat = "boxplot", width = 0.3)
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-6-1.png" width="672" />


``` r
set.seed(0)
df <- USArrests[sample(nrow(USArrests), 5), ]
df$state <- rownames(df)

q <- ggplot(df, aes(Murder, Rape, label = state)) +
  geom_point()
q + geom_text()
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-7-1.png" width="672" />


``` r
q + geom_label(
  aes(
    label = gsub(" ", "\n", state),
    hjust = Murder > mean(Murder),
    vjust = Rape > mean(Rape)
  ),
  label.padding = unit(5, "pt"),
  label.size = NA, fill = NA
)
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-8-1.png" width="672" />

