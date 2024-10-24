---
title: "Ship Till X"
subtitle: "Shipping Orders Based on Cutoff Times and Shipping Days"
excerpt: "Analyzing the distribution of weekly orders based on cutoff times and shipping days."
date: 2024-10-24
author: "Steve Ewing"
draft: false
tags:
  - hugo-site
categories:
  - Business
# layout options: single or single-sidebar
layout: single
---

## Business Question

Take all the orders for the last fiscal year. For weeks 1-18 there is
5 day a week shipping and for weeks 19-52 there is 6 day a week shipping. Assuming
we ship all orders that arrive before a given cutoff that day and all orders that arrive
after the cutoff the next business day what percent of the weekly orders fall on each
day of the week?

``` r
library(tidyverse)
library(arrow)
library(hms)
library(tinytable)

orders <- arrow::read_parquet("./data/orders_recent.parquet") |>
  mutate(
    date_of_date_purchased = as.Date(date_purchased),
    time_of_date_purchased = as_hms(format(date_purchased, "%H:%M:%S"))
  ) |>
  left_join(fiscal_calendar, by = c("date_of_date_purchased" = "date")) |>
  filter(fiscal_year == 2024) |>
  select(
    orders_id,
    date_of_date_purchased,
    time_of_date_purchased,
    fiscal_week = fiscl_week,
    day_of_week
  )

head(orders)
```

    ## # A tibble: 6 × 5
    ##   orders_id date_of_date_purchased time_of_date_purchased fiscal_week
    ##       <int> <date>                 <time>                       <dbl>
    ## 1  18519115 2023-10-01             20:11:14                         1
    ## 2  18519117 2023-10-01             20:13:16                         1
    ## 3  18519119 2023-10-01             20:13:35                         1
    ## 4  18519121 2023-10-01             20:21:15                         1
    ## 5  18519123 2023-10-01             20:21:47                         1
    ## 6  18519125 2023-10-01             20:22:33                         1
    ## # ℹ 1 more variable: day_of_week <ord>

``` r
# Define the cutoff times for Weeks 1-18 (Monday to Friday)
cutoff_times_1_18 <- c(
  "08:00:00",  # Monday (1)
  "10:00:00",  # Tuesday (2)
  "12:00:00",  # Wednesday (3)
  "14:00:00",  # Thursday (4)
  "14:00:00"   # Friday (5)
)

# Define the cutoff times for Weeks 19-52 (Monday to Saturday)
cutoff_times_19_52 <- c(
  "08:00:00",  # Monday (1)
  "10:00:00",  # Tuesday (2)
  "12:00:00",  # Wednesday (3)
  "12:00:00",  # Thursday (4)
  "14:00:00",  # Friday (5)
  "14:00:00"   # Saturday (6)
)
```

## Munging

First write and apply the function to adjust the shipping day

``` r
# Adjust shipping day function with variable cutoff times for each period
adjust_shipping_day <- function(day_of_week, time_of_day, fiscal_week) {
  
  # Determine the shipping days and cutoff times based on fiscal week
  if (fiscal_week <= 18) {
    shipping_days <- 1:5  # Monday to Friday
    cutoff_times <- cutoff_times_1_18
  } else {
    shipping_days <- 1:6  # Monday to Saturday
    cutoff_times <- cutoff_times_19_52
  }
  
  # Check if the current day is a shipping day
  if (day_of_week %in% shipping_days) {
    # Get the cutoff time for the current day
    cutoff_time <- as_hms(cutoff_times[which(shipping_days == day_of_week)])
    
    # If placed after the cutoff time, shift to the next day
    if (time_of_day > cutoff_time) {
      next_day <- day_of_week + 1
    } else {
      next_day <- day_of_week
    }
  } else {
    # If current day is not a shipping day, start from the next day
    next_day <- day_of_week + 1
  }
  
  # Wrap around if next_day exceeds 7 (Sunday)
  if (next_day > 7) {
    next_day <- next_day - 7
  }
  
  # Find the next available shipping day
  next_shipping_day <- shipping_days[shipping_days >= next_day][1]
  
  # If no shipping day is found, wrap around to the first shipping day
  if (is.na(next_shipping_day)) {
    next_shipping_day <- shipping_days[1]
  }
  
  return(next_shipping_day)
}

# Create a vector of day labels starting from Monday (1) to Sunday (7)
day_labels <- c("Monday",
                "Tuesday",
                "Wednesday",
                "Thursday",
                "Friday",
                "Saturday",
                "Sunday")

# Apply the adjusted shipping day function to orders
orders <- orders |>
  mutate(
    day_of_week = wday(date_of_date_purchased, label = FALSE, week_start = 1),
    time_of_date_purchased = as_hms(time_of_date_purchased),
    adjusted_day = mapply(
      adjust_shipping_day,
      day_of_week,
      time_of_date_purchased,
      fiscal_week
    ),
    adjusted_day_label = day_labels[adjusted_day]
  )
```

Now, we group the orders by the adjusted shipping day and calculate the percentage distribution.

``` r
# Calculate distributions for Weeks 1-18
distribution_1_18 <- orders |>
  filter(fiscal_week <= 18) |>
  mutate(week_set = "Weeks 1-18") |>
  group_by(adjusted_day, adjusted_day_label, week_set) |>
  summarize(order_count = n(), .groups = "drop") |>
  ungroup() |>  # Remove grouping to calculate total orders across all days
  mutate(
    total_orders = sum(order_count),
    percentage = round((order_count / total_orders) * 100, 2)
  )

# Calculate distributions for Weeks 19-52
distribution_19_52 <- orders |>
  filter(fiscal_week >= 19) |>
  mutate(week_set = "Weeks 19-52") |>
  group_by(adjusted_day, adjusted_day_label, week_set) |>
  summarize(order_count = n(), .groups = "drop") |>
  ungroup() |>  # Remove grouping
  mutate(
    total_orders = sum(order_count),
    percentage = round((order_count / total_orders) * 100, 2)
  )

# Combine the distributions
combined_distribution <- bind_rows(distribution_1_18, distribution_19_52) |>
  select(!total_orders) |>  # Remove the total orders column
  select(!order_count) # Remove the order count column)

# Display the final distribution
tt(combined_distribution,
   theme = "striped")
```

<!-- preamble start -->
&#10;    <script>
      function styleCell_z5yshz3k0d56yn6wsrf7(i, j, css_id) {
        var table = document.getElementById("tinytable_z5yshz3k0d56yn6wsrf7");
        table.rows[i].cells[j].classList.add(css_id);
      }
      function insertSpanRow(i, colspan, content) {
        var table = document.getElementById('tinytable_z5yshz3k0d56yn6wsrf7');
        var newRow = table.insertRow(i);
        var newCell = newRow.insertCell(0);
        newCell.setAttribute("colspan", colspan);
        // newCell.innerText = content;
        // this may be unsafe, but innerText does not interpret <br>
        newCell.innerHTML = content;
      }
      function spanCell_z5yshz3k0d56yn6wsrf7(i, j, rowspan, colspan) {
        var table = document.getElementById("tinytable_z5yshz3k0d56yn6wsrf7");
        const targetRow = table.rows[i];
        const targetCell = targetRow.cells[j];
        for (let r = 0; r < rowspan; r++) {
          // Only start deleting cells to the right for the first row (r == 0)
          if (r === 0) {
            // Delete cells to the right of the target cell in the first row
            for (let c = colspan - 1; c > 0; c--) {
              if (table.rows[i + r].cells[j + c]) {
                table.rows[i + r].deleteCell(j + c);
              }
            }
          }
          // For rows below the first, delete starting from the target column
          if (r > 0) {
            for (let c = colspan - 1; c >= 0; c--) {
              if (table.rows[i + r] && table.rows[i + r].cells[j]) {
                table.rows[i + r].deleteCell(j);
              }
            }
          }
        }
        // Set rowspan and colspan of the target cell
        targetCell.rowSpan = rowspan;
        targetCell.colSpan = colspan;
      }
    </script>
&#10;    <style>
    </style>
    <div class="container">
      <table class="table table-striped" id="tinytable_z5yshz3k0d56yn6wsrf7" style="width: auto; margin-left: auto; margin-right: auto;" data-quarto-disable-processing='true'>
        <thead>
        &#10;              <tr>
                <th scope="col">adjusted_day</th>
                <th scope="col">adjusted_day_label</th>
                <th scope="col">week_set</th>
                <th scope="col">percentage</th>
              </tr>
        </thead>
        &#10;        <tbody>
                <tr>
                  <td>1</td>
                  <td>Monday   </td>
                  <td>Weeks 1-18 </td>
                  <td>29.72</td>
                </tr>
                <tr>
                  <td>2</td>
                  <td>Tuesday  </td>
                  <td>Weeks 1-18 </td>
                  <td>18.91</td>
                </tr>
                <tr>
                  <td>3</td>
                  <td>Wednesday</td>
                  <td>Weeks 1-18 </td>
                  <td>18.45</td>
                </tr>
                <tr>
                  <td>4</td>
                  <td>Thursday </td>
                  <td>Weeks 1-18 </td>
                  <td>17.12</td>
                </tr>
                <tr>
                  <td>5</td>
                  <td>Friday   </td>
                  <td>Weeks 1-18 </td>
                  <td>15.80</td>
                </tr>
                <tr>
                  <td>1</td>
                  <td>Monday   </td>
                  <td>Weeks 19-52</td>
                  <td>19.91</td>
                </tr>
                <tr>
                  <td>2</td>
                  <td>Tuesday  </td>
                  <td>Weeks 19-52</td>
                  <td>18.37</td>
                </tr>
                <tr>
                  <td>3</td>
                  <td>Wednesday</td>
                  <td>Weeks 19-52</td>
                  <td>18.22</td>
                </tr>
                <tr>
                  <td>4</td>
                  <td>Thursday </td>
                  <td>Weeks 19-52</td>
                  <td>15.06</td>
                </tr>
                <tr>
                  <td>5</td>
                  <td>Friday   </td>
                  <td>Weeks 19-52</td>
                  <td>16.85</td>
                </tr>
                <tr>
                  <td>6</td>
                  <td>Saturday </td>
                  <td>Weeks 19-52</td>
                  <td>11.59</td>
                </tr>
        </tbody>
      </table>
    </div>
<!-- hack to avoid NA insertion in last line -->
