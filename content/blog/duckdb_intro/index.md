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
print(mysql_tables)
```

```
##   [1] "a_Comp_Price_Data"                          
##   [2] "a_Comp_Store"                               
##   [3] "a_Comp_URL"                                 
##   [4] "a_Comp_URL_DNC"                             
##   [5] "adcampaigns"                                
##   [6] "address_book"                               
##   [7] "address_format"                             
##   [8] "adgroups"                                   
##   [9] "admin"                                      
##  [10] "admin_activity"                             
##  [11] "admin_customer_searches"                    
##  [12] "admin_files"                                
##  [13] "admin_groups"                               
##  [14] "admin_messages"                             
##  [15] "admin_password_resets"                      
##  [16] "admin_quotes"                               
##  [17] "admin_sessions"                             
##  [18] "admin_settings"                             
##  [19] "admin_tasks"                                
##  [20] "admin_widgets"                              
##  [21] "affiliate_sales"                            
##  [22] "affiliates"                                 
##  [23] "api_auth"                                   
##  [24] "article_collection_categories"              
##  [25] "article_product_groups"                     
##  [26] "article_reviews"                            
##  [27] "article_reviews_description"                
##  [28] "articles"                                   
##  [29] "articles_collections"                       
##  [30] "articles_description"                       
##  [31] "articles_guides"                            
##  [32] "articles_related"                           
##  [33] "articles_templates"                         
##  [34] "articles_to_topics"                         
##  [35] "articles_types"                             
##  [36] "articles_updates"                           
##  [37] "articles_vote"                              
##  [38] "articles_xml_schema"                        
##  [39] "articles_xsell"                             
##  [40] "attribute_options_groups"                   
##  [41] "audit_log"                                  
##  [42] "auth_codes"                                 
##  [43] "authnet_payment_profiles"                   
##  [44] "authors"                                    
##  [45] "authors_info"                               
##  [46] "autoship_orders"                            
##  [47] "autoshippable_product_replacements"         
##  [48] "autoships"                                  
##  [49] "batch_jobs"                                 
##  [50] "best_sellers"                               
##  [51] "block_locations"                            
##  [52] "block_schedules"                            
##  [53] "block_template_zones"                       
##  [54] "block_templates"                            
##  [55] "block_zones"                                
##  [56] "blocked_ip"                                 
##  [57] "blocks"                                     
##  [58] "bought_together"                            
##  [59] "box_input_box_subscription"                 
##  [60] "box_input_option_box_program"               
##  [61] "box_input_options"                          
##  [62] "box_inputs"                                 
##  [63] "box_program_addons"                         
##  [64] "box_program_step_products"                  
##  [65] "box_program_step_windows"                   
##  [66] "box_program_steps"                          
##  [67] "box_programs"                               
##  [68] "box_subscription_order_products"            
##  [69] "box_subscription_orders"                    
##  [70] "box_subscriptions"                          
##  [71] "brands"                                     
##  [72] "brands_recommended_products"                
##  [73] "bulk_coupon_codes"                          
##  [74] "bulk_pick"                                  
##  [75] "cache"                                      
##  [76] "cache_reports"                              
##  [77] "calendar"                                   
##  [78] "calendar_dates"                             
##  [79] "call_center_agent_states"                   
##  [80] "call_center_agent_status_histories"         
##  [81] "call_center_contacts"                       
##  [82] "call_center_dispositions"                   
##  [83] "call_center_performance"                    
##  [84] "call_center_skills_summary"                 
##  [85] "call_notes"                                 
##  [86] "call_notes_reasons"                         
##  [87] "cancellation_reasons"                       
##  [88] "cancellations"                              
##  [89] "cashregister"                               
##  [90] "categories"                                 
##  [91] "categories_description"                     
##  [92] "chemnames"                                  
##  [93] "collection_templates"                       
##  [94] "company_hours"                              
##  [95] "configuration"                              
##  [96] "configuration_group"                        
##  [97] "contacts_list"                              
##  [98] "conveyor_scan_log"                          
##  [99] "cookies_disabled"                           
## [100] "counter"                                    
## [101] "counter_history"                            
## [102] "countries"                                  
## [103] "coupon_categories"                          
## [104] "coupon_email_track"                         
## [105] "coupon_gv_customer"                         
## [106] "coupon_gv_queue"                            
## [107] "coupon_redeem_track"                        
## [108] "coupons"                                    
## [109] "coupons_description"                        
## [110] "credit_memo_items"                          
## [111] "credit_memos"                               
## [112] "currencies"                                 
## [113] "customer_autoship_replacements"             
## [114] "customer_box_program_schedules"             
## [115] "customer_box_programs"                      
## [116] "customer_fertilizer_lists"                  
## [117] "customer_list_items"                        
## [118] "customer_lists"                             
## [119] "customer_partner_opt_ins"                   
## [120] "customer_profile_sf_answers"                
## [121] "customer_profile_sf_solutions"              
## [122] "customers"                                  
## [123] "customers_basket"                           
## [124] "customers_basket_attributes"                
## [125] "customers_business_info"                    
## [126] "customers_checkout"                         
## [127] "customers_credit_accounts"                  
## [128] "customers_documents"                        
## [129] "customers_documents_types"                  
## [130] "customers_groups"                           
## [131] "customers_history"                          
## [132] "customers_history_types"                    
## [133] "customers_info"                             
## [134] "customers_pco_licenses"                     
## [135] "customers_price_quotes"                     
## [136] "customers_price_tiers"                      
## [137] "customers_sms_blacklist"                    
## [138] "customers_sms_subscriptions"                
## [139] "customers_subscriptions"                    
## [140] "customers_to_customers_price_tiers"         
## [141] "deliveries"                                 
## [142] "domo_product_groups"                        
## [143] "domo_product_groups_old"                    
## [144] "dropship_suppliers"                         
## [145] "dropshipper_info"                           
## [146] "dropshipper_notification_settings"          
## [147] "dropshipper_notifications"                  
## [148] "dropshipper_products"                       
## [149] "email_banners"                              
## [150] "email_hashes"                               
## [151] "epa_expirations"                            
## [152] "exit_ads"                                   
## [153] "exit_ads_to_assets"                         
## [154] "external_api_users"                         
## [155] "external_orders"                            
## [156] "failed_jobs"                                
## [157] "fedex_tnt"                                  
## [158] "fees"                                       
## [159] "fertilizer_npk_ratios"                      
## [160] "fertilizers"                                
## [161] "field_types"                                
## [162] "formulation"                                
## [163] "found_us"                                   
## [164] "fraud_scores"                               
## [165] "geo_zipcodes"                               
## [166] "geo_zones"                                  
## [167] "hazmat_info"                                
## [168] "hazmat_package_info"                        
## [169] "history_types"                              
## [170] "holiday_message"                            
## [171] "import_cubiscan_temp"                       
## [172] "incentive_programs_actions"                 
## [173] "iot_products"                               
## [174] "job_batches"                                
## [175] "journal"                                    
## [176] "label_info"                                 
## [177] "languages"                                  
## [178] "manufacturers"                              
## [179] "manufacturers_info"                         
## [180] "manufacturers_recommended_products"         
## [181] "marketplace_orders_products"                
## [182] "marketplace_pending_cancels"                
## [183] "marketplace_pending_orders"                 
## [184] "marketplace_products"                       
## [185] "meta_tags"                                  
## [186] "migrations"                                 
## [187] "newsletters"                                
## [188] "notifications"                              
## [189] "optimize_check"                             
## [190] "order_cancellation_reasons"                 
## [191] "order_cancellations"                        
## [192] "orders"                                     
## [193] "orders_address_failures"                    
## [194] "orders_data_tracking"                       
## [195] "orders_exception"                           
## [196] "orders_fees"                                
## [197] "orders_hold_reasons"                        
## [198] "orders_holds"                               
## [199] "orders_products"                            
## [200] "orders_products_attributes"                 
## [201] "orders_products_download"                   
## [202] "orders_products_fees"                       
## [203] "orders_products_kit_parts"                  
## [204] "orders_products_pick"                       
## [205] "orders_products_removed"                    
## [206] "orders_status"                              
## [207] "orders_status_history"                      
## [208] "orders_to_orders_exceptions"                
## [209] "orders_total"                               
## [210] "orders_total_original"                      
## [211] "organization_type"                          
## [212] "package_weight_variances"                   
## [213] "pages"                                      
## [214] "payments_charges"                           
## [215] "payments_reconciliation_log"                
## [216] "payments_refunds"                           
## [217] "payments_refunds_products"                  
## [218] "payments_types"                             
## [219] "payments_void"                              
## [220] "pending_autoship_order_products"            
## [221] "personal_access_tokens"                     
## [222] "pest_groups"                                
## [223] "pests"                                      
## [224] "pests_to_pest_groups"                       
## [225] "pests_to_products"                          
## [226] "pests_to_products_use_sites"                
## [227] "pick_stats"                                 
## [228] "pick_stats_3_week"                          
## [229] "pick_stats_3_week_last"                     
## [230] "pick_stats_last"                            
## [231] "pick_stats_weekly"                          
## [232] "press_releases"                             
## [233] "previews"                                   
## [234] "price_matches"                              
## [235] "print_log"                                  
## [236] "print_type"                                 
## [237] "product_document_types"                     
## [238] "product_documents"                          
## [239] "product_features"                           
## [240] "product_highlights"                         
## [241] "product_loss_additional_losses"             
## [242] "product_loss_events"                        
## [243] "product_loss_items"                         
## [244] "product_loss_reasons"                       
## [245] "product_losses"                             
## [246] "product_message_macros"                     
## [247] "product_update_batches"                     
## [248] "product_update_data"                        
## [249] "product_update_notes"                       
## [250] "product_updates"                            
## [251] "products"                                   
## [252] "products_actives"                           
## [253] "products_addons"                            
## [254] "products_alternate_descriptions"            
## [255] "products_attributes"                        
## [256] "products_attributes_download"               
## [257] "products_attributes_vendors"                
## [258] "products_barcodes"                          
## [259] "products_barcodes_to_options"               
## [260] "products_boxes"                             
## [261] "products_bundles"                           
## [262] "products_comments"                          
## [263] "products_description"                       
## [264] "products_dropshipper_feed"                  
## [265] "products_editing"                           
## [266] "products_export"                            
## [267] "products_extra_fields"                      
## [268] "products_extra_fields_options"              
## [269] "products_groups"                            
## [270] "products_history"                           
## [271] "products_inventory"                         
## [272] "products_label_data"                        
## [273] "products_media"                             
## [274] "products_media_types"                       
## [275] "products_notifications"                     
## [276] "products_notifications_history"             
## [277] "products_options"                           
## [278] "products_options_values"                    
## [279] "products_options_values_to_products_options"
## [280] "products_parts"                             
## [281] "products_predictions"                       
## [282] "products_pricing_history"                   
## [283] "products_promotions"                        
## [284] "products_properties"                        
## [285] "products_properties_values"                 
## [286] "products_questions"                         
## [287] "products_questions_vote"                    
## [288] "products_selected_options"                  
## [289] "products_stock_status_emails"               
## [290] "products_to_accessories"                    
## [291] "products_to_article_product_groups"         
## [292] "products_to_assets"                         
## [293] "products_to_categories"                     
## [294] "products_to_pests"                          
## [295] "products_to_product_features"               
## [296] "products_to_products_actives"               
## [297] "products_to_products_extra_fields"          
## [298] "products_to_products_groups"                
## [299] "products_to_products_properties_values"     
## [300] "products_to_products_uom"                   
## [301] "products_to_projects"                       
## [302] "products_to_sites"                          
## [303] "products_to_use_locations"                  
## [304] "products_to_vendors"                        
## [305] "products_to_zones"                          
## [306] "products_uom"                               
## [307] "products_use_sites"                         
## [308] "products_websites"                          
## [309] "products_weight_log"                        
## [310] "products_xsell"                             
## [311] "products_xsell2"                            
## [312] "program_region_exceptions"                  
## [313] "projects"                                   
## [314] "quickbooks_config"                          
## [315] "quickbooks_log"                             
## [316] "quickbooks_queue"                           
## [317] "quickbooks_recur"                           
## [318] "quickbooks_ticket"                          
## [319] "quickbooks_user"                            
## [320] "rebate_partners"                            
## [321] "redemption_codes"                           
## [322] "redemption_tracking"                        
## [323] "referrals_emails_sent"                      
## [324] "referrals_generation_history"               
## [325] "referrals_usage_history"                    
## [326] "regions"                                    
## [327] "report_request"                             
## [328] "reviews"                                    
## [329] "reviews_description"                        
## [330] "reviews_vote"                               
## [331] "rewards_points_history"                     
## [332] "rules"                                      
## [333] "save_for_later_basket"                      
## [334] "sb_accessories"                             
## [335] "sb_box_input_box_subscription"              
## [336] "sb_box_input_options"                       
## [337] "sb_box_inputs"                              
## [338] "sb_box_subscriptions"                       
## [339] "sb_components"                              
## [340] "sb_components_sb_versions"                  
## [341] "sb_customer_box_program_program_groups"     
## [342] "sb_discount_products"                       
## [343] "sb_pricing_options"                         
## [344] "sb_product_group_products"                  
## [345] "sb_product_groups"                          
## [346] "sb_program_groups"                          
## [347] "sb_programs"                                
## [348] "sb_programs_components"                     
## [349] "sb_region_exceptions"                       
## [350] "sb_schedules"                               
## [351] "sb_shipping_frequencies"                    
## [352] "sb_signup_pages"                            
## [353] "sb_signup_trackbar_waypoints"               
## [354] "sb_version_activations"                     
## [355] "sb_versions"                                
## [356] "sessions"                                   
## [357] "sf_answers"                                 
## [358] "sf_questions"                               
## [359] "sf_rules"                                   
## [360] "sf_wizards"                                 
## [361] "shipment_assignments"                       
## [362] "shipment_package_products"                  
## [363] "shipment_package_queue"                     
## [364] "shipment_packages"                          
## [365] "shipments"                                  
## [366] "shipments_packages_history"                 
## [367] "shipments_status"                           
## [368] "shipments_status_history"                   
## [369] "shipping_audits"                            
## [370] "shipping_differences"                       
## [371] "shipping_events"                            
## [372] "shipping_rate_records"                      
## [373] "shopping_engines"                           
## [374] "shopping_feeds_categories"                  
## [375] "shopping_feeds_extra_fields"                
## [376] "shopping_feeds_products_blocked"            
## [377] "short_urls"                                 
## [378] "site_groups"                                
## [379] "site_options"                               
## [380] "sites"                                      
## [381] "sites_to_site_groups"                       
## [382] "sli_product_data"                           
## [383] "sli_product_types"                          
## [384] "snippets"                                   
## [385] "specials"                                   
## [386] "step_products_email_content"                
## [387] "stock_change_reasons"                       
## [388] "store_credit_history"                       
## [389] "subscription_coupon_subscriptions"          
## [390] "subscription_coupons"                       
## [391] "subscription_credit_history"                
## [392] "subscription_discount_products"             
## [393] "subscription_history"                       
## [394] "subscription_order_addons"                  
## [395] "subscription_orders"                        
## [396] "suggestions"                                
## [397] "suppliers_import"                           
## [398] "suppliers_import_settings"                  
## [399] "suppliers_import_temp"                      
## [400] "tags"                                       
## [401] "tags_to_assets"                             
## [402] "talkdesk_actions"                           
## [403] "talkdesk_actions_data"                      
## [404] "talkdesk_inbound_call_starts"               
## [405] "tax_capture_log"                            
## [406] "tax_class"                                  
## [407] "tax_rates"                                  
## [408] "temp_admin_recounts"                        
## [409] "temp_serialmagic"                           
## [410] "testimonials"                               
## [411] "tmp_tax_rates"                              
## [412] "topics"                                     
## [413] "topics_description"                         
## [414] "tpb_accounts"                               
## [415] "tracking"                                   
## [416] "trusted_stores_feed"                        
## [417] "ups_billing"                                
## [418] "ups_billing_uploads"                        
## [419] "ups_import"                                 
## [420] "use_locations"                              
## [421] "usps_mess"                                  
## [422] "usps_regional_rates"                        
## [423] "usps_tnt"                                   
## [424] "usps_tnt_temp"                              
## [425] "usps_zones"                                 
## [426] "usps_zones_exceptions"                      
## [427] "vendor_price_temp"                          
## [428] "vendors"                                    
## [429] "vendors_actions"                            
## [430] "vendors_address_book"                       
## [431] "vendors_admin"                              
## [432] "vendors_customers"                          
## [433] "vendors_invoice_status"                     
## [434] "vendors_invoices"                           
## [435] "vendors_orders"                             
## [436] "vendors_orders_history"                     
## [437] "vendors_orders_products"                    
## [438] "vendors_products"                           
## [439] "vendors_products_history"                   
## [440] "vendors_roles"                              
## [441] "vendors_to_vendors_actions"                 
## [442] "video_collections"                          
## [443] "videos"                                     
## [444] "videos_categories"                          
## [445] "videos_details"                             
## [446] "videos_products"                            
## [447] "videos_to_collections"                      
## [448] "videos_to_entities"                         
## [449] "visitor_logs"                               
## [450] "votes"                                      
## [451] "warehouse_actions"                          
## [452] "warehouse_events"                           
## [453] "warehouse_personnel"                        
## [454] "warehouse_personnel_clockin_history"        
## [455] "warehouse_personnel_types"                  
## [456] "warehouse_zones"                            
## [457] "warehouses"                                 
## [458] "webhook_calls"                              
## [459] "website_email_types"                        
## [460] "website_emails"                             
## [461] "websites"                                   
## [462] "whos_online"                                
## [463] "wm_achievements"                            
## [464] "wm_boxes"                                   
## [465] "wm_carriers"                                
## [466] "wm_cart_locations"                          
## [467] "wm_carts"                                   
## [468] "wm_cycle_count_history"                     
## [469] "wm_cycle_count_settings"                    
## [470] "wm_cycle_count_warehouse_settings"          
## [471] "wm_damages"                                 
## [472] "wm_inventory_transactions"                  
## [473] "wm_invoice_items"                           
## [474] "wm_invoices"                                
## [475] "wm_labor_tracking"                          
## [476] "wm_labor_tracking_zones"                    
## [477] "wm_lot_types"                               
## [478] "wm_message_store"                           
## [479] "wm_pack_stations"                           
## [480] "wm_packer_stats"                            
## [481] "wm_payment_terms"                           
## [482] "wm_personnel_achievements"                  
## [483] "wm_pick_waves"                              
## [484] "wm_picker_stats"                            
## [485] "wm_po_comments"                             
## [486] "wm_po_history"                              
## [487] "wm_po_item_check_history"                   
## [488] "wm_po_item_stocking_assignments"            
## [489] "wm_po_items"                                
## [490] "wm_po_items_tracking"                       
## [491] "wm_po_sent_via"                             
## [492] "wm_po_status"                               
## [493] "wm_printers"                                
## [494] "wm_product_promos"                          
## [495] "wm_product_promos_payments"                 
## [496] "wm_product_promos_products"                 
## [497] "wm_product_promos_tiers"                    
## [498] "wm_promos_to_warehouses"                    
## [499] "wm_purchase_orders"                         
## [500] "wm_restock_log"                             
## [501] "wm_rma_conditions"                          
## [502] "wm_rma_history"                             
## [503] "wm_rma_products"                            
## [504] "wm_rma_products_status"                     
## [505] "wm_rma_reasons"                             
## [506] "wm_rma_resolutions"                         
## [507] "wm_rma_status"                              
## [508] "wm_rma_tracking"                            
## [509] "wm_rmas"                                    
## [510] "wm_shipment_buckets"                        
## [511] "wm_shipments_rate_shop"                     
## [512] "wm_stocking_assignments"                    
## [513] "wm_stocking_issues"                         
## [514] "wm_supplier_po_settings"                    
## [515] "wm_supplier_split_po_warehouses"            
## [516] "wm_suppliers"                               
## [517] "wm_suppliers_contacts"                      
## [518] "wm_suppliers_to_products"                   
## [519] "wm_task_time_history"                       
## [520] "wm_task_times"                              
## [521] "wm_tasks"                                   
## [522] "wm_totes"                                   
## [523] "wm_zone_assignments"                        
## [524] "wm_zone_bays"                               
## [525] "wm_zones"                                   
## [526] "zips_to_fips"                               
## [527] "zones"                                      
## [528] "zones_pest_regions"                         
## [529] "zones_to_geo_zones"
```

If you want to load only specific tables, you can define them:


``` r
# Specify the tables you want to load
tables_to_load <- c("orders", "products", "orders_products", "orders_products_pick", "orders_total", "shipments", "ups_import", "products_to_products_groups", "products_groups", "fees", "payments_charges", "payments_refunds", "payments_types", "wm_po_items", "wm_purchase_orders", "wm_boxes", "autoship_orders", "box_subscription_orders", "products_bundles", "products_inventory", "wm_inventory_transactions", "wm_suppliers", "wm_invoices", "wm_po_history", "shipments_status", "orders_status", "manufacturers", "customers", "customers_credit_accounts", "address_book", "wm_product_promos", "wm_product_promos_products", "wm_product_promos_tiers", "customers_groups", "suppliers_import", "wm_suppliers_to_products", "products_to_products_extra_fields", "products_extra_fields")
```

### Transfer Tables from MySQL to DuckDB

We'll iterate over each table, read it from MySQL, and write it into DuckDB.


``` r
# Use the full list or your specified list
tables <- tables_to_load  # or use 'mysql_tables' for all tables

for (table_name in tables) {
  # Read data from MySQL
  message("Loading table: ", table_name)
  data <- dbReadTable(con_mysql, table_name)
  
  # Write data to DuckDB
  dbWriteTable(con_duckdb, table_name, data, overwrite = TRUE)
}
```

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

```
## Loading table: orders_products_pick
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 0
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 1
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 2
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 5
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 6 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 9
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 14 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 15 imported
## as numeric
```

```
## Loading table: orders_total
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 0
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 4 imported
## as numeric
```

```
## Loading table: shipments
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 0
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 1
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 2
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 3
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 4
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 17 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 19
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 20
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 26
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 27
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 28
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 29
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 30
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 31
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 32
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 33
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 34
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 35
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 36
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 37
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 38 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 39 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 40
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 47 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 52
## imported as numeric
```

```
## Loading table: ups_import
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 2 imported
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
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 11 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 15 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 16 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 17 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 18 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): unrecognized MySQL field type 7
## in column 20 imported as character
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 28 imported
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
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 33 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 34 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 35 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 36 imported
## as numeric
```

```
## Loading table: products_to_products_groups
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 0
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 1
## imported as numeric
```

```
## Loading table: products_groups
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 0
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 2
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 3
## imported as numeric
```

```
## Loading table: fees
```

```
## Loading table: payments_charges
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 0
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 1
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 3
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 12
## imported as numeric
```

```
## Loading table: payments_refunds
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 0
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 1
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 4
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 8
## imported as numeric
```

```
## Loading table: payments_types
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 0
## imported as numeric
```

```
## Loading table: wm_po_items
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 0
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 1
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 2
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 3
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 8 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 12
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 15
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 17 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 18 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 20
## imported as numeric
```

```
## Loading table: wm_purchase_orders
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 0
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 1
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 2
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 6
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 10
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 11
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 14
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 22 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 23 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 24 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 25 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 37
## imported as numeric
```

```
## Loading table: wm_boxes
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 4 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 6 imported
## as numeric
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
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 11 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 12 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 18 imported
## as numeric
```

```
## Loading table: autoship_orders
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 0
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): unrecognized MySQL field type 7
## in column 4 imported as character
```

```
## Warning in dbSendQuery(conn, statement, ...): unrecognized MySQL field type 7
## in column 5 imported as character
```

```
## Loading table: box_subscription_orders
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 0
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): unrecognized MySQL field type 7
## in column 4 imported as character
```

```
## Warning in dbSendQuery(conn, statement, ...): unrecognized MySQL field type 7
## in column 5 imported as character
```

```
## Loading table: products_bundles
```

```
## Loading table: products_inventory
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 0
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 1
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 17
## imported as numeric
```

```
## Loading table: wm_inventory_transactions
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 0
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 1
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 2
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 3
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 4 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 5 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 7 imported
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
## Loading table: wm_suppliers
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 0
## imported as numeric
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
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 26
## imported as numeric
```

```
## Loading table: wm_invoices
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 0
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 3
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 4 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 5
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 7
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 10
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 11 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 12 imported
## as numeric
```

```
## Loading table: wm_po_history
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 0
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 1
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 2
## imported as numeric
```

```
## Loading table: shipments_status
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 0
## imported as numeric
```

```
## Loading table: orders_status
```

```
## Loading table: manufacturers
```

```
## Loading table: customers
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 18
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 32 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 34 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 43
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 44
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 45
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 46
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 47
## imported as numeric
```

```
## Loading table: customers_credit_accounts
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 0
## imported as numeric
```

```
## Loading table: address_book
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 21 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 22 imported
## as numeric
```

```
## Loading table: wm_product_promos
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 0
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 1
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 2
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 5 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 7
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 10
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 11
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 12
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 13
## imported as numeric
```

```
## Loading table: wm_product_promos_products
```

```
## Loading table: wm_product_promos_tiers
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 3 imported
## as numeric
```

```
## Loading table: customers_groups
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 0
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 4
## imported as numeric
```

```
## Loading table: suppliers_import
```

```
## Warning in dbSendQuery(conn, statement, ...): unrecognized MySQL field type 7
## in column 5 imported as character
```

```
## Loading table: wm_suppliers_to_products
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 0
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 1
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 2
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 6 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 11
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Decimal MySQL column 13 imported
## as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 14
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 15
## imported as numeric
```

```
## Warning in dbSendQuery(conn, statement, ...): Unsigned INTEGER in col 18
## imported as numeric
```

```
## Loading table: products_to_products_extra_fields
```

```
## Loading table: products_extra_fields
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
