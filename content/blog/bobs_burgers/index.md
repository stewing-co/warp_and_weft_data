---
title: "Tidy Tuesday: 2024-11-19"
subtitle: "Bob's Burgers"
excerpt: ""
date: 2024-11-18
author: "Steve Ewing"
draft: false
series:
tags: ["R"]
categories: ["Tidy Tuesday"]
layout: single
---

First copy to cool graphic that Steven Ponce made.


``` r
## 1. LOAD PACKAGES & SETUP ----
if (!require("pacman")) install.packages("pacman")
pacman::p_load(
  tidyverse,         # Easily Install and Load the 'Tidyverse'
  ggtext,            # Improved Text Rendering Support for 'ggplot2'
  showtext,          # Using Fonts More Easily in R Graphs
  janitor,           # Simple Tools for Examining and Cleaning Dirty Data
  skimr,             # Compact and Flexible Summaries of Data
  scales,            # Scale Functions for Visualization
  glue,              # Interpreted String Literals
  bobsburgersR,      # Bob's Burgers Datasets for Data Visualization
  tidytext,          # Text Mining using 'dplyr', 'ggplot2', and Other Tidy Tools 
  textdata,          # Download and Load Various Text Datasets 
  patchwork          # The Composer of Plots
)    

### |- figure size ----
camcorder::gg_record(
  dir    = here::here("temp_plots"),
  device = "png",
  width  =  9,
  height =  10,
  units  = "in",
  dpi    = 320
)
```

```
## Warning in camcorder::gg_record(dir = here::here("temp_plots"), device = "png",
## : Writing to a folder that already exists. gg_playback may use more files than
## intended!
```

``` r
### |- resolution ----
showtext_opts(dpi = 320, regular.wt = 300, bold.wt = 800)
```


``` r
bobsburgersR::transcript_data
```

```
## # A tibble: 181,031 × 6
##    season episode title        line raw_text                            dialogue
##     <dbl>   <dbl> <chr>       <dbl> <chr>                               <chr>   
##  1      1       1 Human Flesh     1 <NA>                                <NA>    
##  2      1       1 Human Flesh     2 <NA>                                <NA>    
##  3      1       1 Human Flesh     3 <NA>                                <NA>    
##  4      1       1 Human Flesh     4 Listen, pep talk.                   Listen,…
##  5      1       1 Human Flesh     5 Big day today.                      Big day…
##  6      1       1 Human Flesh     6 It's our grand re-re-re-opening.    It's ou…
##  7      1       1 Human Flesh     7 It's labor day weekend, And it loo… It's la…
##  8      1       1 Human Flesh     8 So we have to-- Big day for anothe… So we h…
##  9      1       1 Human Flesh     9 Go ahead, sorry.                    Go ahea…
## 10      1       1 Human Flesh    10 Go ahead, do your pep.              Go ahea…
## # ℹ 181,021 more rows
```


``` r
glimpse(transcript_data)
```

```
## Rows: 181,031
## Columns: 6
## $ season   <dbl> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1…
## $ episode  <dbl> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1…
## $ title    <chr> "Human Flesh", "Human Flesh", "Human Flesh", "Human Flesh", "…
## $ line     <dbl> 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18…
## $ raw_text <chr> NA, NA, NA, "Listen, pep talk.", "Big day today.", "It's our …
## $ dialogue <chr> NA, NA, NA, "Listen, pep talk.", "Big day today.", "It's our …
```

``` r
skim(transcript_data)
```


Table: <span id="tab:unnamed-chunk-3"></span>Table 1: Data summary

|                         |                |
|:------------------------|:---------------|
|Name                     |transcript_data |
|Number of rows           |181031          |
|Number of columns        |6               |
|_______________________  |                |
|Column type frequency:   |                |
|character                |3               |
|numeric                  |3               |
|________________________ |                |
|Group variables          |None            |


**Variable type: character**

|skim_variable | n_missing| complete_rate| min|  max| empty| n_unique| whitespace|
|:-------------|---------:|-------------:|---:|----:|-----:|--------:|----------:|
|title         |         0|          1.00|   5|   78|     0|      272|          0|
|raw_text      |     33159|          0.82|   1| 1233|     0|   127978|          0|
|dialogue      |     33331|          0.82|   1| 1157|     0|   127532|          0|


**Variable type: numeric**

|skim_variable | n_missing| complete_rate|   mean|     sd| p0| p25| p50| p75| p100|hist  |
|:-------------|---------:|-------------:|------:|------:|--:|---:|---:|---:|----:|:-----|
|season        |         0|             1|   9.31|   4.00|  1|   6|  10|  13|   14|▂▃▂▃▇ |
|episode       |         0|             1|  10.54|   6.14|  1|   5|  10|  16|   23|▇▆▇▅▅ |
|line          |         0|             1| 483.43| 459.72|  1| 167| 333| 557| 2004|▇▃▁▁▁ |


``` r
# Calculate metrics 
episode_metrics <- transcript_data |>
  filter(!is.na(dialogue)) |>
  group_by(season, episode) |>
  summarise(
    # Basic dialogue metrics
    dialogue_density = n() / max(line),
    avg_length       = mean(str_length(dialogue)),
    
    # Sentiment analysis - AFINN Sentiment Lexicon
    sentiment_variance = dialogue |>
      tibble(text = _) |>
      unnest_tokens(word, text) |>
      inner_join(get_sentiments("afinn"), by = "word") |>
      pull(value) |>
      var(na.rm = TRUE),
    
    # Word and punctuation metrics  
    unique_words      = dialogue |>
      str_split("\\s+") |>
      unlist() |>
      n_distinct(),
    
    question_ratio    = mean(str_detect(dialogue, "\\?")),
    exclamation_ratio = mean(str_detect(dialogue, "!")),
    
    .groups = "drop"
  ) |>
  # Scale all metrics
  mutate(across(dialogue_density:exclamation_ratio, scale))


# Prepare data for visualization 
episode_metrics_long <- episode_metrics |>
  pivot_longer(
    cols = c(dialogue_density:exclamation_ratio),
    names_to = "metric",
    values_to = "value"
  ) |>
  mutate(
    angle = (as.numeric(factor(metric)) - 1) * 2 * pi / 6,
    hjust = ifelse(angle < pi, 1, 0),
    metric = case_when(
      metric == "dialogue_density" ~ "Dialogue\nDensity",
      metric == "avg_length" ~ "Average\nLength",
      metric == "sentiment_variance" ~ "Sentiment\nVariance",
      metric == "unique_words" ~ "Unique\nWords",
      metric == "question_ratio" ~ "Question\nRatio",
      metric == "exclamation_ratio" ~ "Exclamation\nRatio"
    )
  )
```


``` r
### |- plot aesthetics ----
bkg_col      <- "#f5f5f2"  
title_col    <- "gray20"           
subtitle_col <- "gray20"     
caption_col  <- "gray30"   
text_col     <- "gray20"   

### |-  titles and caption ----
# icons
tt <- str_glue("Source: {{bobsburgersR}}")
li <- str_glue("<span style='font-family:sans'>&#xf08c;</span>")
gh <- str_glue("<span style='font-family:sans'>&#xf09b;</span>")
bs <- str_glue("<span style='font-family:sans'>&#xe671; </span>")

# text
light_purple <- str_glue("<span style='color:#A374C2'>**Light Purple**</span>")
dark_purple  <- str_glue("<span style='color:#8856a7'>**Dark Purple**</span>")

title_text    <- str_glue("Bob's Burgers Episode Fingerprints by Season")
subtitle_text <- str_glue("Analyzing dialogue patterns across seasons<br><br>
                          **Note:** Metrics are standardized (**z-scores**). { light_purple } polygons represent individual episodes.<br>
                          { dark_purple } line shows season average.")
caption_text  <- str_glue("{li} stevenponce &bull; {gh} poncest &bull; #rstats #ggplot2 &bull; {tt}")

### |-  fonts ----
# font_add("fa6-brands", here::here("fonts/6.4.2/Font Awesome 6 Brands-Regular-400.otf"))
font_add_google("Oswald", regular.wt = 400, family = "title")
font_add_google("Source Sans Pro", family = "subtitle")
font_add_google("Source Sans Pro", family = "text")  
font_add_google("Roboto Mono", family = "numbers")  
font_add_google("Noto Sans", regular.wt = 400, family = "caption")
showtext_auto(enable = TRUE)


### |-  plot theme ----
theme_set(theme_minimal(base_size = 14, base_family = "text"))                

theme_update(
  plot.title.position   = "plot",
  plot.caption.position = "plot",
  legend.position       = "plot",
  plot.background       = element_rect(fill = bkg_col, color = bkg_col),
  panel.background      = element_rect(fill = bkg_col, color = bkg_col),
  plot.margin           = margin(t = 10, r = 10, b = 10, l = 10),
  axis.title.x          = element_text(margin = margin(10, 0, 0, 0), size = rel(1.1), 
                                       color = text_col, family = "text", face = "bold", hjust = 0.5),
  axis.title.y          = element_text(margin = margin(0, 10, 0, 0), size = rel(1.1), 
                                       color = text_col, family = "text", face = "bold", hjust = 0.5),
  axis.text             = element_text(size = rel(0.5), color = text_col, family = "text"),
  strip.text            = element_text(size = rel(1), face = "bold", margin = margin(b = 10), family = "text"),
  panel.grid.major      = element_line(color = "gray90", linewidth = 0.2),
  panel.spacing.x       = unit(3, "lines"),  
  panel.spacing.y       = unit(1, "lines"),  
  aspect.ratio          = 1  
)
```


``` r
### |- main plot ---- 
main_plot <- episode_metrics_long |>   
  ggplot(aes(x = metric, y = value)) +

  # Geoms
  # Add grid lines
  geom_hline(yintercept = seq(-2, 7, by = 1), color = "gray90", linewidth = 0.2) +
  
  # Add episode polygons
  geom_polygon(aes(group = interaction(season, episode)),
               fill = "#8856a7",
               alpha = 0.2) +
  
  # Add season average line
  stat_summary(aes(group = season),
               fun = 'mean', 
               geom = "path",
               color = "#8856a7",
               size = 0.8,
               alpha = 0.9,
               na.rm = TRUE) +
  
  # Scales
  scale_x_discrete(expand = expansion(add = 1.2)) +  
  scale_y_continuous(
    expand = expansion(add = c(0.5, 1)),
    limits = c(-2, 7)                 
  ) +
  coord_polar(clip = 'off') +

  # Labs
  labs(
    x = NULL,
    y = NULL,
  ) +
  
  # Facet 
  facet_wrap(~season, nrow = 4, 
             labeller = labeller(season = function(x) paste("Season", x))
  ) +
  
  # Theme
  theme(
    axis.text.y = element_blank(),
    axis.title = element_blank(),
    plot.margin = margin(0, 0, 0, 0)
  )
```

```
## Warning: Using `size` aesthetic for lines was deprecated in ggplot2 3.4.0.
## ℹ Please use `linewidth` instead.
## This warning is displayed once every 8 hours.
## Call `lifecycle::last_lifecycle_warnings()` to see where this warning was
## generated.
```

``` r
### |- key pattern plot ----
key_patterns_plot <- ggplot() +
  annotate(
    "richtext",
    x = 0,
    y = 0,
    label = glue::glue(
      "<span style='font-family:title;font-size:12pt;color:{title_col}'>**Key Patterns:**</span><br>
      <span style='font-family:subtitle;font-size:9pt;color:{text_col}'>
      • Early seasons (1-3): more experimental patterns<br>
      • Middle seasons (4-8): consistent style<br>
      • Later seasons: refined structure<br>
      • Higher variance: character episodes<br>
      • Higher question ratios: mystery plots
      </span>"
    ),
    fill = NA,
    label.color = NA,
    hjust = 0,
    vjust = 1.2,
  ) +
  theme_void() +
  theme(
    plot.margin = margin(5, 10, 5, 10)      
  )

### |- title & subtitle plot ----
title_plot <- ggplot() +
  labs(
    title = title_text,
    subtitle = subtitle_text
  ) +
  theme_void() +
  theme(
    plot.title      = element_text(
      size          = rel(1.8),
      family        = "title",
      face          = "bold",
      color         = title_col,
      lineheight    = 1.1,
      margin        = margin(t = 5, b = 5)
    ),   
    plot.subtitle   = element_markdown(
      size          = rel(1.1),
      family        = "subtitle",
      color         = subtitle_col,
      lineheight    = 1.1,
      margin        = margin(t = 5, b = 15)
    ),
    plot.background = element_rect(fill = bkg_col, color = bkg_col),
    plot.margin     = margin(10, 10, 0, 10)
  )

### |- combined plot ----

# Define layout design with adjusted areas

# area(t, l, b, r)

# where:
# t = top row position
# l = left column position
# b = bottom row position
# r = right column position

design <- c(
    area(1, 1, 1, 6),      # title area
    area(2, 1, 5, 6),      # main plot area
    area(4, 2, 5, 6)       # key patterns area 
)

combined_plot <- title_plot +  main_plot + key_patterns_plot +
  plot_layout(
    design = design,
    heights = c(0.8, 4, 4, 4, 0.2), 
    widths = c(1, 1, 1, 1, 1, 0.2)
  ) +
  plot_annotation(
    caption = caption_text,
    theme = theme(
      plot.background = element_rect(fill = bkg_col, color = bkg_col),
      plot.margin     = margin(10, 10, 10, 10),
      plot.caption    = element_markdown(
        size          = rel(0.65),
        family        = "caption",
        color         = caption_col,
        lineheight    = 1.1,
        hjust         = 0.5,
        margin        = margin(t = 10, b = 5)
      )
    )
  )

combined_plot
```

So that's it! We replicated the original plot. Now lets add on the wikipedia data by episode and see if any of the metrics influence the ratings. It isnt rendering in the document when it gets published so here the png.

![finished product](./content/blog/bobs_burgers/plot.jpg)



``` r
imdb_wikipedia_data <- bobsburgersR::imdb_wikipedia_data

imdb_wikipedia_data 
```

```
## # A tibble: 275 × 11
##    episode_overall imdb_aired_date  year season episode imdb_title        rating
##              <dbl> <date>          <dbl>  <dbl>   <dbl> <chr>              <dbl>
##  1               1 2011-01-08       2011      1       1 Human Flesh          7.7
##  2               2 2011-01-15       2011      1       2 Crawl Space          8.1
##  3               3 2011-01-22       2011      1       3 Sacred Cow           7.5
##  4               4 2011-02-12       2011      1       4 Sexy Dance Fight…    7.4
##  5               5 2011-02-19       2011      1       5 Hamburger Dinner…    7.5
##  6               6 2011-03-05       2011      1       6 Sheesh! Cab, Bob?    8.3
##  7               7 2011-03-12       2011      1       7 Bed & Breakfast      7.5
##  8               8 2011-03-19       2011      1       8 Art Crawl            7.9
##  9               9 2011-03-26       2011      1       9 Spaghetti Wester…    7.6
## 10              10 2011-04-09       2011      1      10 Burger War           7.7
## # ℹ 265 more rows
## # ℹ 4 more variables: synopsis <chr>, wikipedia_directed_by <chr>,
## #   wikipedia_written_by <chr>, wikipedia_viewers <dbl>
```


``` r
glimpse(imdb_wikipedia_data)
```

```
## Rows: 275
## Columns: 11
## $ episode_overall       <dbl> 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 1…
## $ imdb_aired_date       <date> 2011-01-08, 2011-01-15, 2011-01-22, 2011-02-12,…
## $ year                  <dbl> 2011, 2011, 2011, 2011, 2011, 2011, 2011, 2011, …
## $ season                <dbl> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 2, 2, 2, …
## $ episode               <dbl> 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 1, 2,…
## $ imdb_title            <chr> "Human Flesh", "Crawl Space", "Sacred Cow", "Sex…
## $ rating                <dbl> 7.7, 8.1, 7.5, 7.4, 7.5, 8.3, 7.5, 7.9, 7.6, 7.7…
## $ synopsis              <chr> "On top of watching his dysfunctional family cru…
## $ wikipedia_directed_by <chr> "Anthony Chun", "Kyounghee Lim", "Jennifer Coyle…
## $ wikipedia_written_by  <chr> "Loren Bouchard & Jim Dauterive", "Loren Bouchar…
## $ wikipedia_viewers     <dbl> 9.38, 5.07, 4.81, 4.19, 4.87, 4.91, 4.10, 4.43, …
```

``` r
skim(imdb_wikipedia_data)
```


Table: <span id="tab:unnamed-chunk-8"></span>Table 2: Data summary

|                         |                    |
|:------------------------|:-------------------|
|Name                     |imdb_wikipedia_data |
|Number of rows           |275                 |
|Number of columns        |11                  |
|_______________________  |                    |
|Column type frequency:   |                    |
|character                |4                   |
|Date                     |1                   |
|numeric                  |6                   |
|________________________ |                    |
|Group variables          |None                |


**Variable type: character**

|skim_variable         | n_missing| complete_rate| min| max| empty| n_unique| whitespace|
|:---------------------|---------:|-------------:|---:|---:|-----:|--------:|----------:|
|imdb_title            |         0|          1.00|   5|  77|     0|      275|          0|
|synopsis              |         0|          1.00|  18| 872|     0|      275|          0|
|wikipedia_directed_by |         6|          0.98|   3|  33|     0|       33|          0|
|wikipedia_written_by  |         6|          0.98|   3|  77|     0|       31|          0|


**Variable type: Date**

|skim_variable   | n_missing| complete_rate|min        |max        |median     | n_unique|
|:---------------|---------:|-------------:|:----------|:----------|:----------|--------:|
|imdb_aired_date |         0|             1|2011-01-08 |2024-09-21 |2018-03-17 |      266|


**Variable type: numeric**

|skim_variable     | n_missing| complete_rate|    mean|    sd|      p0|     p25|    p50|    p75|    p100|hist  |
|:-----------------|---------:|-------------:|-------:|-----:|-------:|-------:|------:|------:|-------:|:-----|
|episode_overall   |         0|          1.00|  138.00| 79.53|    1.00|   69.50|  138.0|  206.5|  275.00|▇▇▇▇▇ |
|year              |         0|          1.00| 2017.47|  3.72| 2011.00| 2014.00| 2018.0| 2021.0| 2024.00|▆▇▆▇▆ |
|season            |         0|          1.00|    7.84|  3.80|    1.00|    5.00|    8.0|   11.0|   14.00|▆▇▅▇▇ |
|episode           |         0|          1.00|   10.73|  6.19|    1.00|    5.00|   10.0|   16.0|   23.00|▇▆▇▅▅ |
|rating            |         2|          0.99|    7.70|  0.44|    6.70|    7.40|    7.6|    8.0|    9.60|▂▇▃▁▁ |
|wikipedia_viewers |         8|          0.97|    2.35|  1.29|    0.59|    1.47|    2.0|    3.1|    9.38|▇▃▁▁▁ |

``` r
combinded_data <- episode_metrics |> 
  left_join(imdb_wikipedia_data, by = c("season", "episode")) |> 
  mutate(
    rating = as.numeric(rating),
    wikipedia_viewers = as.numeric(wikipedia_viewers)
  )

predictors <- combinded_data %>%
  select(dialogue_density, avg_length, sentiment_variance, 
         unique_words, question_ratio, exclamation_ratio, rating)

glm_model <- glm(rating ~ ., data = predictors, family = gaussian())

summary(glm_model)
```

```
## 
## Call:
## glm(formula = rating ~ ., family = gaussian(), data = predictors)
## 
## Coefficients:
##                     Estimate Std. Error t value Pr(>|t|)    
## (Intercept)         7.694419   0.025434 302.531  < 2e-16 ***
## dialogue_density    0.153914   0.048316   3.186 0.001618 ** 
## avg_length         -0.197340   0.056095  -3.518 0.000512 ***
## sentiment_variance  0.025194   0.026505   0.951 0.342711    
## unique_words        0.002158   0.029711   0.073 0.942156    
## question_ratio      0.043562   0.032210   1.352 0.177386    
## exclamation_ratio   0.138396   0.032051   4.318 2.23e-05 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## (Dispersion parameter for gaussian family taken to be 0.1752897)
## 
##     Null deviance: 53.244  on 270  degrees of freedom
## Residual deviance: 46.276  on 264  degrees of freedom
##   (1 observation deleted due to missingness)
## AIC: 306.08
## 
## Number of Fisher Scoring iterations: 2
```

Looks like people like them short dense and punchy.


``` r
predictors2 <- combinded_data %>%
  select(dialogue_density, avg_length, exclamation_ratio, rating)

glm_model2 <- glm(rating ~ ., data = predictors2, family = gaussian())

summary(glm_model2)
```

```
## 
## Call:
## glm(formula = rating ~ ., family = gaussian(), data = predictors2)
## 
## Coefficients:
##                   Estimate Std. Error t value Pr(>|t|)    
## (Intercept)        7.69464    0.02541 302.837  < 2e-16 ***
## dialogue_density   0.15753    0.04572   3.446 0.000661 ***
## avg_length        -0.16881    0.04760  -3.546 0.000461 ***
## exclamation_ratio  0.14059    0.02891   4.864 1.97e-06 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## (Dispersion parameter for gaussian family taken to be 0.1749506)
## 
##     Null deviance: 53.244  on 270  degrees of freedom
## Residual deviance: 46.712  on 267  degrees of freedom
##   (1 observation deleted due to missingness)
## AIC: 302.61
## 
## Number of Fisher Scoring iterations: 2
```

