---
title: "Using DuckDB in R: A Walkthrough Starting with MySQL"
subtitle: ""
excerpt: ""
date: 2024-11-01
author: "Steve Ewing"
draft: false
images:
series:
tags:
categories:
layout: single
---

# Introduction

In this walkthrough, we will demonstrate how to use DuckDB in R, starting by connecting to a MySQL server. By integrating MySQL and DuckDB, you can leverage the strengths of both databases within your data science workflows. This guide will help you retrieve data from MySQL, import it into DuckDB, and perform efficient analytical queries.

## Prerequisites

R and RStudio: Ensure you have R and RStudio installed.
MySQL Server: Access to a MySQL server with the necessary permissions.
Required R Packages: ```RMySQL```, ```DBI```, ```duckdb```, ```dplyr```, ```dbplyr```, ```ggplot2```

## Loading Data into DuckDB From MySQL

First, we need to connect to a MySQL database to retrieve data.

### Installation


``` r
# install.packages("RMySQL")
# install.packages("DBI")  # Database interface
```

Load the libraries:


``` r
library(RMySQL)
```

```
## Warning: package 'RMySQL' was built under R version 4.4.1
```

```
## Loading required package: DBI
```

```
## Warning: package 'DBI' was built under R version 4.4.1
```

``` r
library(DBI)
```

### Establishing a Connection

Set up the connection to your MySQL server:


``` r
# Replace with your MySQL server details
con_mysql <- dbConnect(RMySQL::MySQL(),
                       dbname = Sys.getenv("DO_DB_NAME"),
                       host = Sys.getenv("DO_DB_HOST"), 
                       port = as.integer(Sys.getenv("DO_DB_PORT")),
                       user = Sys.getenv("DO_DB_USER"),
                       password = Sys.getenv("DO_DB_PASSWORD"))
```

### Connect to DuckDB


``` r
# DuckDB connection
con_duckdb <- dbConnect(duckdb::duckdb(), dbdir = "dmo_data.duckdb")
```

### Retrieve the List of Tables from MySQL

Let's retrieve data from a MySQL table:


``` r
# List available tables
mysql_tables <- dbListTables(con_mysql)

# View the list of tables
head(mysql_tables)
```

```
## [1] "a_Comp_Price_Data" "a_Comp_Store"      "a_Comp_URL"       
## [4] "a_Comp_URL_DNC"    "adcampaigns"       "address_book"
```

If you want to load only specific tables, you can define them:


``` r
# Specify the tables you want to load
tables_to_load <- c("orders", "products", "orders_products")
```

### Transfer Tables from MySQL to DuckDB

We'll iterate over each table, read it from MySQL, and write it into DuckDB.


```
## Loading table: orders
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 1 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 42 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 44 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 45 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 46
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 47 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 48 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 49 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 50 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 53 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 56 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 89 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 115
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 118
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 119 imported
## as numeric
```

```
## Loading table: products
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 29 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 30 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 31 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 32 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 36 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 37 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 46
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 61 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 62 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 63 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 64 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 65 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 66 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 67 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 68 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 87 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 88 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 91 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 92 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 93 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 97 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 98 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 100 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 105 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 136
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 139
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 150 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 152 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 153 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 155 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 157 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 158 imported
## as numeric
```

```
## Loading table: orders_products
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 7 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 8 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 9 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 10 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 13 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 15
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 18 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 19 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 23
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 24
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 25
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 34 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 35 imported
## as numeric
```

### Disconnect From the Server


``` r
# Disconnect from MySQL
dbDisconnect(con_mysql)
```

```
## [1] TRUE
```

### Verify the Tables in DuckDB

List the tables in DuckDB to confirm they have been loaded:


``` r
duckdb_tables <- dbListTables(con_duckdb)
print(duckdb_tables)
```

```
##  [1] "address_book"                      "autoship_orders"                  
##  [3] "box_subscription_orders"           "customers"                        
##  [5] "customers_credit_accounts"         "customers_groups"                 
##  [7] "fees"                              "manufacturers"                    
##  [9] "orders"                            "orders_products"                  
## [11] "orders_products_pick"              "orders_status"                    
## [13] "orders_total"                      "payments_charges"                 
## [15] "payments_refunds"                  "payments_types"                   
## [17] "products"                          "products_bundles"                 
## [19] "products_extra_fields"             "products_groups"                  
## [21] "products_inventory"                "products_to_products_extra_fields"
## [23] "products_to_products_groups"       "shipments"                        
## [25] "shipments_status"                  "suppliers_import"                 
## [27] "ups_import"                        "wm_boxes"                         
## [29] "wm_inventory_transactions"         "wm_invoices"                      
## [31] "wm_po_history"                     "wm_po_items"                      
## [33] "wm_product_promos"                 "wm_product_promos_products"       
## [35] "wm_product_promos_tiers"           "wm_purchase_orders"               
## [37] "wm_suppliers"                      "wm_suppliers_to_products"
```

## Performing Queries in DuckDB

With the tables loaded into DuckDB, you can perform efficient queries using SQL or dplyr.

### Using SQL Queries


``` r
result <- dbGetQuery(con_duckdb, "
  SELECT op.products_id, 
         COUNT(op.qty_returned) AS returns
  FROM orders_products op
  LEFT JOIN products p USING(products_id)
  LEFT JOIN orders o USING(orders_id)
  WHERE op.qty_returned > 0
    AND op.qty_returned IS NOT NULL
  GROUP BY op.products_id, p.name
  ORDER BY returns DESC
  LIMIT 10;
")

print(result)
```

```
##    products_id returns
## 1          304    1469
## 2          184    1348
## 3         1813     728
## 4         1877     519
## 5          100     502
## 6         1816     489
## 7         1817     481
## 8         3546     427
## 9           40     398
## 10          65     351
```
