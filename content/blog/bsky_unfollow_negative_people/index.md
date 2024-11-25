---
title: "Bluesky Unfollowing Negative Follows"
subtitle: "If you don't follow me and are negative you can take a hike"
excerpt: ""
date: 2024-11-24
author: "Steve Ewing"
draft: false
series:
tags: ["R", "DuckDB", "Bluesky"]
categories: ["Bluesky"]
layout: single
---

I followed a ton of people using the new starter packs. Now I'm going to keep all my mutuals and unfollow the people I'm following who aren't following me back and who have a net negative sentiment score on their posts.


``` r
# Load necessary libraries
library(DBI)
library(duckdb)
library(tidyverse)
library(httr2)
library(tidytext)
library(parsedate)  

get_follow_relationships <- function(db_path) {
  library(tidyverse)
  library(httr2)
  library(DBI)
  library(duckdb)
  
  # Create a logged-in API session object
  session <- request("https://bsky.social/xrpc/com.atproto.server.createSession") |>
    req_method("POST") |>
    req_body_json(list(
      identifier = Sys.getenv("BLUESKY_APP_USER"),
      password = Sys.getenv("BLUESKY_APP_PASS")
    )) |>
    req_perform() |>
    resp_body_json()
  
  # Get your own DID (Decentralized Identifier)
  self_handle <- session$handle
  self_did <- session$did
  
  # Function to get all follows or followers
  get_all_follows <- function(endpoint_url, user_did) {
    all_users <- list()
    cursor <- NULL
    
    repeat {
      req <- request(endpoint_url) |>
        req_method("GET") |>
        req_headers(Authorization = paste0("Bearer ", session$accessJwt)) |>
        req_url_query(actor = user_did)
      
      if (!is.null(cursor)) {
        req <- req |> req_url_query(cursor = cursor)
      }
      
      resp <- req |>
        req_error(
          is_error = function(resp)
            FALSE
        ) |>
        req_perform()
      
      if (resp_status(resp) >= 400) {
        message("An error occurred fetching data: ",
                resp_status_desc(resp))
        print(resp_body_string(resp))
        break
      }
      
      data <- resp_body_json(resp)
      
      # Determine if fetching followers or following
      if ("followers" %in% names(data)) {
        users <- data$followers
      } else if ("follows" %in% names(data)) {
        users <- data$follows
      } else {
        message("Unexpected data format.")
        break
      }
      
      # Check if users list is empty
      if (length(users) == 0) {
        break
      }
      
      all_users <- c(all_users, users)
      
      if (!is.null(data$cursor)) {
        cursor <- data$cursor
      } else {
        break
      }
      
      Sys.sleep(0.1) # Pause to respect rate limits
    }
    
    # Process the users into a tibble
    users_df <- map_dfr(all_users, function(user) {
      tibble(
        did = user$did,
        handle = user$handle,
        displayName = user$displayName %||% NA_character_,
        description = user$description %||% NA_character_,
        createdAt = user$createdAt %||% NA_character_
      )
    })
    
    return(users_df)
  }
  
  # Get followers
  message("Fetching your followers...")
  followers_df <- get_all_follows("https://bsky.social/xrpc/app.bsky.graph.getFollowers",
                                  self_did)
  
  # Get following
  message("Fetching users you are following...")
  following_df <- get_all_follows("https://bsky.social/xrpc/app.bsky.graph.getFollows",
                                  self_did)
  
  # Identify relationships
  followers_handles <- followers_df$handle
  following_handles <- following_df$handle
  
  # Users you are following who aren't following you back
  wwd_following <- following_df %>%
    filter(!handle %in% followers_handles) %>%
    mutate(tag = "wwd-following")
  
  # Users who are following you but you aren't following back
  wwd_follower <- followers_df %>%
    filter(!handle %in% following_handles) %>%
    mutate(tag = "wwd-follower")
  
  # Mutual follows
  wwd_mutuals <- following_df %>%
    filter(handle %in% followers_handles) %>%
    mutate(tag = "wwd-mutuals")
  
  # Combine all
  relationships_df <- bind_rows(wwd_following, wwd_follower, wwd_mutuals)
  
  # Connect to the database
  con <- dbConnect(duckdb::duckdb(), db_path)
  
  # Save relationships to the database
  dbWriteTable(con, "follow_relationships", relationships_df, overwrite = TRUE)
  
  # Disconnect from the database
  dbDisconnect(con, shutdown = TRUE)
  
  message("Follow relationships have been stored in the database.")
  
  # Return the data frame
  return(relationships_df)
}

# Define the path to your database
db_path <- "C:/Users/steph/OneDrive/warp_and_weft_data/content/blog/bsky_sentiment_index/bluesky_data.duckdb"

# Call the function with the database path
relationships <- get_follow_relationships(db_path)

# View the results
print(relationships)
```


``` r
# Function to get posts from a vector of handles, fetching only new posts and writing to db as it fetches
get_posts_from_handles <- function(handles, session, db_path) {
  # Load parsedate library
  library(parsedate)
  
  # Connect to the database
  con <- dbConnect(duckdb::duckdb(), db_path)
  
  # Loop over each handle
  for (handle in handles) {
    message("Fetching posts for handle: ", handle)
    
    # Get the latest record_createdAt for this handle from the posts table
    latest_time_query <- sprintf(
      "SELECT MAX(record_createdAt) as latest_time FROM posts WHERE author_handle = '%s'",
      handle
    )
    latest_time_result <- dbGetQuery(con, latest_time_query)
    latest_time <- latest_time_result$latest_time[1]
    
    # If there is no latest_time (i.e., no posts for this handle), set to NULL
    if (is.na(latest_time) || is.null(latest_time)) {
      latest_time <- NULL
      message("No existing posts found for handle: ", handle)
    } else {
      message("Latest post time for ", handle, ": ", latest_time)
    }
    
    # Initialize variables for fetching posts
    # We will process and write posts as we fetch them
    cursor <- NULL
    keep_fetching <- TRUE
    
    repeat {
      # Construct the GET request
      req <- request("https://bsky.social/xrpc/app.bsky.feed.getAuthorFeed") |>
        req_method("GET") |>
        req_headers(Authorization = paste0("Bearer ", session$accessJwt)) |>
        req_url_query(actor = handle)
      
      # If cursor is not NULL, add it to the query
      if (!is.null(cursor)) {
        req <- req |> req_url_query(cursor = cursor)
      }
      
      # Perform the request
      resp <- req |>
        req_error(
          is_error = function(resp)
            FALSE
        ) |>
        req_perform()
      
      # Check if the response is an error
      if (resp_status(resp) >= 400) {
        message("An error occurred fetching posts for ",
                handle,
                ": ",
                resp_status_desc(resp))
        # Optionally, print the response body for more details
        print(resp_body_string(resp))
        break
      }
      
      # Parse the response
      feed_data <- resp_body_json(resp)
      
      # Check if feed is empty
      if (length(feed_data$feed) == 0) {
        message("No posts found for handle: ", handle)
        break
      }
      
      # Process posts and check timestamps
      posts_to_add <- list()
      for (post in feed_data$feed) {
        post_time <- post$post$record$createdAt %||% ""
        if (post_time == "")
          next  # Skip if no timestamp
        
        # Parse the timestamps using parsedate::parse_iso_8601
        post_time_parsed <- parse_iso_8601(post_time)
        latest_time_parsed <- if (!is.null(latest_time))
          parse_iso_8601(latest_time)
        else
          NULL
        
        # Compare post_time with latest_time
        if (!is.null(latest_time_parsed) &&
            !is.na(post_time_parsed) &&
            post_time_parsed <= latest_time_parsed) {
          # Reached posts we've already fetched
          keep_fetching <- FALSE
          break
        } else {
          # New post, add to the list
          posts_to_add <- c(posts_to_add, list(post))
        }
      }
      
      # If there are new posts to add, process and write them to the database
      if (length(posts_to_add) > 0) {
        posts_df <- map_dfr(posts_to_add, function(post) {
          tibble(
            uri = post$post$uri,
            cid = post$post$cid,
            author_did = post$post$author$did %||% NA_character_,
            author_handle = post$post$author$handle %||% NA_character_,
            author_displayName = post$post$author$displayName %||% NA_character_,
            record_type = post$post$record$`$type` %||% NA_character_,
            record_text = post$post$record$text %||% NA_character_,
            record_createdAt = post$post$record$createdAt %||% NA_character_,
            reply_root = post$post$record$reply$root$uri %||% NA_character_,
            reply_parent = post$post$record$reply$parent$uri %||% NA_character_,
            repost_count = post$post$repostCount %||% NA_integer_,
            reply_count = post$post$replyCount %||% NA_integer_,
            like_count = post$post$likeCount %||% NA_integer_,
            indexed_at = post$post$indexedAt %||% NA_character_
          )
        })
        
        # Write the new posts to the 'posts' table, appending
        dbWriteTable(con, "posts", posts_df, append = TRUE)
        message("Wrote ", nrow(posts_df), " new posts for handle: ", handle)
      } else {
        message("No new posts to add for handle: ", handle)
      }
      
      # Break the loop if we've reached existing posts
      if (!keep_fetching) {
        break
      }
      
      # Update cursor
      if (!is.null(feed_data$cursor)) {
        cursor <- feed_data$cursor
      } else {
        # No more pages to fetch
        break
      }
      
      # Optional: Pause to respect rate limits
      Sys.sleep(0.1)
    }
  }
  
  # Disconnect from the database
  dbDisconnect(con, shutdown = FALSE)  # Keep the database running for other operations
}

# Function to compute sentiment scores for new posts
compute_sentiment_scores_for_new_posts <- function(db_path) {
  library(DBI)
  library(duckdb)
  library(tidyverse)
  library(tidytext)
  
  # Connect to the database
  con <- dbConnect(duckdb::duckdb(), db_path)
  
  # Get the list of uris of posts that already have sentiment scores
  existing_scores <- dbGetQuery(con, "SELECT uri FROM post_sentiment_scores")
  
  # Get the list of uris of all posts
  all_posts <- dbGetQuery(con, "SELECT uri, record_text FROM posts")
  
  # Filter posts that don't have sentiment scores yet
  posts_to_score <- all_posts %>%
    filter(!uri %in% existing_scores$uri) %>%
    filter(!is.na(record_text))
  
  # If there are no new posts to score, exit the function
  if (nrow(posts_to_score) == 0) {
    message("No new posts to compute sentiment scores for.")
    dbDisconnect(con, shutdown = FALSE)
    return(NULL)
  }
  
  # Tokenize the text
  posts_tokens <- posts_to_score %>%
    unnest_tokens(word, record_text)
  
  # Get the AFINN sentiment lexicon
  afinn <- get_sentiments("afinn")
  
  # Join tokens with sentiment lexicon
  posts_sentiment <- posts_tokens %>%
    inner_join(afinn, by = "word")
  
  # Compute sentiment score per post
  post_sentiment_scores <- posts_sentiment %>%
    group_by(uri) %>%
    summarise(sentiment_score = sum(value), .groups = "drop")
  
  # Handle posts without sentiment words
  all_uris <- posts_to_score$uri
  post_sentiment_scores <- tibble(uri = all_uris) %>%
    left_join(post_sentiment_scores, by = "uri") %>%
    mutate(sentiment_score = replace_na(sentiment_score, 0))
  
  # Write the new sentiment scores to the database, appending
  dbWriteTable(con,
               "post_sentiment_scores",
               post_sentiment_scores,
               append = TRUE)
  
  # Disconnect from the database
  dbDisconnect(con, shutdown = FALSE)
  
  message("Sentiment scores computed for new posts.")
}

# Main script to get posts and compute average sentiment scores
main <- function() {
  # Create a logged-in API session object once
  session <- request("https://bsky.social/xrpc/com.atproto.server.createSession") |>
    req_method("POST") |>
    req_body_json(list(
      identifier = Sys.getenv("BLUESKY_APP_USER"),
      password = Sys.getenv("BLUESKY_APP_PASS")
    )) |>
    req_perform() |>
    resp_body_json()
  
  # Connect to the database
  con <- dbConnect(duckdb::duckdb(), db_path)
  
  # Get the list of 'wwd-following' handles
  wwd_following_df <- dbGetQuery(con,
                                 "SELECT * FROM follow_relationships WHERE tag = 'wwd-following'")
  wwd_following_handles <- unique(wwd_following_df$handle)
  
  # Disconnect from the database
  dbDisconnect(con, shutdown = FALSE)
  
  # Fetch posts for the 'wwd-following' handles, only new posts, writing to db as we fetch
  get_posts_from_handles(wwd_following_handles, session, db_path)
  
  # Compute sentiment scores for new posts
  compute_sentiment_scores_for_new_posts(db_path)
  
  # Connect to the database
  con <- dbConnect(duckdb::duckdb(), db_path)
  
  # Build the list of handles for the SQL query
  handle_list <- paste(sprintf("'%s'", wwd_following_handles), collapse = ",")
  
  # Build the SQL query using sprintf
  sql_query <- sprintf(
    "
    SELECT p.*, s.sentiment_score
    FROM posts p
    LEFT JOIN post_sentiment_scores s ON p.uri = s.uri
    WHERE p.author_handle IN (%s)
",
handle_list
  )
  
  # Run the query
  posts_with_sentiment <- dbGetQuery(con, sql_query)
  
  # Disconnect from the database
  dbDisconnect(con, shutdown = FALSE)
  
  # Compute average sentiment score per person
  average_sentiment_per_person <- posts_with_sentiment %>%
    group_by(author_handle) %>%
    summarise(average_sentiment = mean(sentiment_score, na.rm = TRUE),
              .groups = "drop")
  
  # View the results
  print(average_sentiment_per_person)
  
  # Optionally, save the results to the database
  con <- dbConnect(duckdb::duckdb(), db_path)
  dbWriteTable(
    con,
    "wwd_following_average_sentiment",
    average_sentiment_per_person,
    overwrite = TRUE
  )
  dbDisconnect(con, shutdown = TRUE)
  
  message("Average sentiment scores computed and stored.")
}

# Run the main function
main()
```


``` r
# Define the database path
db_path <- "C:/Users/steph/OneDrive/warp_and_weft_data/content/blog/bsky_sentiment_index/bluesky_data.duckdb"

# Function to unfollow users based on criteria, excluding those who are following you
unfollow_users_based_on_criteria <- function(db_path) {
  # Create a logged-in API session object
  session <- request("https://bsky.social/xrpc/com.atproto.server.createSession") %>%
    req_method("POST") %>%
    req_body_json(list(
      identifier = Sys.getenv("BLUESKY_APP_USER"),
      password = Sys.getenv("BLUESKY_APP_PASS")
    )) %>%
    req_perform() %>%
    resp_body_json()
  
  # Get your own DID (Decentralized Identifier)
  self_did <- session$did
  
  # Connect to the database
  con <- dbConnect(duckdb::duckdb(), db_path)
  
  # Retrieve users with 0 posts
  users_with_zero_posts <- dbGetQuery(
    con,
    "
    SELECT handle
    FROM follow_relationships
    WHERE tag = 'wwd-following'
    AND handle NOT IN (SELECT DISTINCT author_handle FROM posts)
  "
  )
  
  # Retrieve users with average sentiment score below -5
  users_with_low_sentiment <- dbGetQuery(
    con,
    "
    SELECT author_handle AS handle
    FROM wwd_following_average_sentiment
    WHERE average_sentiment < 5
  "
  )
  
  # Combine the users to unfollow
  users_to_unfollow <- unique(c(
    users_with_zero_posts$handle,
    users_with_low_sentiment$handle
  ))
  
  # Retrieve the list of users who are following you
  message("Fetching your followers...")
  # Function to get all followers
  get_all_followers <- function(session, user_did) {
    all_followers <- list()
    cursor <- NULL
    
    repeat {
      req <- request("https://bsky.social/xrpc/app.bsky.graph.getFollowers") %>%
        req_method("GET") %>%
        req_headers(Authorization = paste0("Bearer ", session$accessJwt)) %>%
        req_url_query(actor = user_did)
      
      if (!is.null(cursor)) {
        req <- req %>% req_url_query(cursor = cursor)
      }
      
      resp <- req %>%
        req_error(
          is_error = function(resp)
            FALSE
        ) %>%
        req_perform()
      
      if (resp_status(resp) >= 400) {
        message("An error occurred fetching followers: ",
                resp_status_desc(resp))
        print(resp_body_string(resp))
        break
      }
      
      data <- resp_body_json(resp)
      
      followers <- data$followers
      
      if (length(followers) == 0) {
        break
      }
      
      all_followers <- c(all_followers, followers)
      
      if (!is.null(data$cursor)) {
        cursor <- data$cursor
      } else {
        break
      }
      
      Sys.sleep(0.1) # Pause to respect rate limits
    }
    
    # Process the followers into a tibble
    followers_df <- map_dfr(all_followers, function(user) {
      tibble(
        did = user$did,
        handle = user$handle,
        displayName = user$displayName %||% NA_character_,
        description = user$description %||% NA_character_,
        createdAt = user$createdAt %||% NA_character_
      )
    })
    
    return(followers_df)
  }
  
  # Get your followers
  followers_df <- get_all_followers(session, self_did)
  
  # Exclude users who are following you from users_to_unfollow
  followers_handles <- followers_df$handle
  users_to_unfollow <- setdiff(users_to_unfollow, followers_handles)
  
  # If there are no users to unfollow after exclusion, exit the function
  if (length(users_to_unfollow) == 0) {
    message("No users meet the criteria for unfollowing after excluding followers.")
    dbDisconnect(con, shutdown = TRUE)
    return(NULL)
  }
  
  # Fetch your follow records to get the rkey (record key)
  message("Fetching your follow records...")
  
  get_follow_records <- function(session, repo_did) {
    all_records <- list()
    cursor <- NULL
    
    repeat {
      req <- request("https://bsky.social/xrpc/com.atproto.repo.listRecords") %>%
        req_method("GET") %>%
        req_headers(Authorization = paste0("Bearer ", session$accessJwt)) %>%
        req_url_query(repo = repo_did,
                      collection = "app.bsky.graph.follow",
                      limit = 100)
      
      if (!is.null(cursor)) {
        req <- req %>% req_url_query(cursor = cursor)
      }
      
      resp <- req %>%
        req_error(
          is_error = function(resp)
            FALSE
        ) %>%
        req_perform()
      
      if (resp_status(resp) >= 400) {
        message("An error occurred fetching follow records: ",
                resp_status_desc(resp))
        print(resp_body_string(resp))
        break
      }
      
      data <- resp_body_json(resp)
      
      records <- data$records
      
      if (length(records) == 0) {
        break
      }
      
      all_records <- c(all_records, records)
      
      if (!is.null(data$cursor)) {
        cursor <- data$cursor
      } else {
        break
      }
      
      Sys.sleep(0.1) # Pause to respect rate limits
    }
    
    # Process the records into a tibble
    records_df <- map_dfr(all_records, function(record) {
      tibble(
        uri = record$uri,
        rkey = record$uri %>% basename(),
        cid = record$cid,
        createdAt = record$value$createdAt %||% NA_character_,
        subject_did = record$value$subject %||% NA_character_
      )
    })
    
    return(records_df)
  }
  
  # Get your follow records
  follow_records_df <- get_follow_records(session, self_did)
  
  # Get the mapping of DIDs to handles from the database
  did_handle_map <- dbGetQuery(
    con,
    "
    SELECT did, handle
    FROM follow_relationships
    WHERE did IS NOT NULL AND handle IS NOT NULL
  "
  )
  
  # Merge handles into follow_records_df
  follow_records_df <- follow_records_df %>%
    left_join(did_handle_map, by = c("subject_did" = "did"))
  
  # For any DIDs not found in the database, optionally resolve them via API (if necessary)
  missing_handles <- follow_records_df %>%
    filter(is.na(handle)) %>%
    distinct(subject_did)
  
  if (nrow(missing_handles) > 0) {
    message("Resolving missing DIDs to handles via API...")
    resolve_dids_to_handles <- function(dids) {
      handles <- c()
      for (did in dids) {
        req <- request("https://bsky.social/xrpc/com.atproto.identity.resolveHandle") %>%
          req_method("GET") %>%
          req_url_query(did = did)
        
        resp <- req %>%
          req_error(
            is_error = function(resp)
              FALSE
          ) %>%
          req_perform()
        
        if (resp_status(resp) == 200) {
          data <- resp_body_json(resp)
          handles <- c(handles, data$handle)
        } else {
          handles <- c(handles, NA_character_)
        }
        Sys.sleep(0.1)
      }
      return(handles)
    }
    
    # Resolve missing DIDs
    resolved_handles <- resolve_dids_to_handles(missing_handles$subject_did)
    resolved_map <- tibble(subject_did = missing_handles$subject_did, handle = resolved_handles)
    
    # Update follow_records_df with resolved handles
    follow_records_df <- follow_records_df %>%
      left_join(resolved_map,
                by = "subject_did",
                suffix = c("", ".resolved")) %>%
      mutate(handle = coalesce(handle, handle.resolved)) %>%
      select(-handle.resolved)
  }
  
  # Find the records corresponding to users_to_unfollow
  records_to_delete <- follow_records_df %>%
    filter(handle %in% users_to_unfollow)
  
  # If there are no matching records, exit the function
  if (nrow(records_to_delete) == 0) {
    message("No matching follow records found to unfollow.")
    dbDisconnect(con, shutdown = TRUE)
    return(NULL)
  }
  
  # Function to unfollow a user by deleting the follow record
  unfollow_user <- function(session, repo_did, rkey) {
    req <- request("https://bsky.social/xrpc/com.atproto.repo.deleteRecord") %>%
      req_method("POST") %>%
      req_headers(Authorization = paste0("Bearer ", session$accessJwt)) %>%
      req_body_json(list(
        collection = "app.bsky.graph.follow",
        repo = repo_did,
        rkey = rkey
      ))
    
    resp <- req %>%
      req_error(
        is_error = function(resp)
          FALSE
      ) %>%
      req_perform()
    
    if (resp_status(resp) >= 400) {
      message("Failed to unfollow rkey ",
              rkey,
              ": ",
              resp_status_desc(resp))
      print(resp_body_string(resp))
    } else {
      message("Successfully unfollowed rkey ", rkey)
    }
  }
  
  # Unfollow each user
  message("Unfollowing users who meet the criteria...")
  
  for (i in seq_len(nrow(records_to_delete))) {
    rkey <- records_to_delete$rkey[i]
    handle <- records_to_delete$handle[i]
    message("Unfollowing user: ", handle)
    unfollow_user(session, self_did, rkey)
    Sys.sleep(0.1) # Pause to respect rate limits
  }
  
  # Disconnect from the database
  dbDisconnect(con, shutdown = TRUE)
  
  message("Unfollowing process completed.")
}

# Run the function
unfollow_users_based_on_criteria(db_path)
```


``` r
# Function to create a histogram of average sentiment scores of remaining follows
create_histogram_of_avg_sentiment <- function(db_path) {
  # Connect to the database
  con <- dbConnect(duckdb::duckdb(), db_path)
  
  # Retrieve the list of users you are currently following
  follows_df <- dbGetQuery(con,
                           "SELECT * FROM follow_relationships WHERE tag = 'wwd-following'")
  
  # Retrieve the average sentiment scores
  avg_sentiment_df <- dbGetQuery(
    con,
    "SELECT author_handle AS handle, average_sentiment FROM wwd_following_average_sentiment"
  )
  
  # Merge the data to get the average sentiment scores of your current follows
  merged_df <- follows_df %>%
    inner_join(avg_sentiment_df, by = "handle")
  
  # Disconnect from the database
  dbDisconnect(con, shutdown = TRUE)
  
  # Check if there are any data to plot
  if (nrow(merged_df) == 0) {
    message("No data available to plot.")
    return(NULL)
  }
  
  # Plot the histogram
  ggplot(merged_df, aes(x = average_sentiment)) +
    geom_histogram(
      binwidth = 1,
      fill = "steelblue",
      color = "black"
    ) +
    labs(title = "Histogram of Average Sentiment Scores of Remaining Follows", x = "Average Sentiment Score", y = "Number of Users") +
    theme_minimal()
}

# Run the function to create the histogram
create_histogram_of_avg_sentiment(db_path)
```

