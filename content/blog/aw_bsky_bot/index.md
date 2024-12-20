---
title: "Andrew Wheiss's Bluesky Bot"
subtitle: ""
excerpt: ""
date: 2024-11-21
author: "Andrew Wheiss"
draft: false
series:
tags: ["R", "Bluesky"]
categories: ["Bluesky"]
layout: single
---

This is very cool and works with only httr2 and ggplot2 for images.

https://github.com/andrewheiss/bsky-bot-r


``` r
library(httr2)
library(ggplot2)

# Create a logged-in API session object
session <- request("https://bsky.social/xrpc/com.atproto.server.createSession") |>
  req_method("POST") |>
  req_body_json(list(
    identifier = Sys.getenv("BLUESKY_APP_USER"),
    password = Sys.getenv("BLUESKY_APP_PASS")
  )) |>
  req_perform() |>
  resp_body_json()

# Send a plot to Bluesky!
random_beta_plot <- function() {
  # Egypt color palette from MetBrewer
  clrs <- c("#dd5129", "#0f7ba2", "#43b284", "#fab255")

  shape1 <- round(runif(1, min = 1, max = 10), 1)
  shape2 <- 1 + rpois(1, lambda = 2)

  dist_details <- glue::glue("Beta({shape1}, {shape2})")

  dist_plot <- ggplot() +
    stat_function(
      geom = "area",
      fun = \(x) dbeta(x, shape1 = shape1, shape2 = shape2),
      fill = sample(clrs, 1)
    ) +
    labs(title = dist_details) +
    theme_void() +
    theme(
      plot.title = element_text(face = "bold", hjust = 0.5)
    )

  # There's probably a better way to convert this object to a binary version of
  # a png, but this works

  # Create a temporary file
  plot_file <- tempfile()

  # Save the plot there
  ggsave(plot_file, dist_plot, width = 4, height = 2.5, device = "png")
  
  # Also save it for the blog cover
  ggsave("featured.jpg", dist_plot, width = 4, height = 2.5, device = "jpg")

  # Read that temporary file as a binary object
  plot_bytes <- readBin(plot_file, "raw", file.info(plot_file)$size)

  # Get rid of the temporary file
  unlink(plot_file)

  # Return a bunch of stuff
  return(list(dist_plot = dist_plot, dist_details = dist_details, raw = plot_bytes))
}

# Create a plot
thing <- random_beta_plot()
thing$dist_plot
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-1-1.png" width="672" />

``` r
thing$dist_details
```

```
## Beta(2.1, 4)
```

``` r
thing$raw
```

```
##     [1] 89 50 4e 47 0d 0a 1a 0a 00 00 00 0d 49 48 44 52 00 00 04 b0 00 00 02 ee
##    [25] 08 06 00 00 00 be 75 ef 72 00 00 00 06 62 4b 47 44 00 00 00 00 00 00 f9
##    [49] 43 bb 7f 00 00 00 09 70 48 59 73 00 00 2e 23 00 00 2e 23 01 78 a5 3f 76
##    [73] 00 00 20 00 49 44 41 54 78 9c ec dd 79 98 a4 57 41 2e f0 f7 54 cf 4c 66
##    [97] 26 09 49 40 c2 16 24 99 99 24 dd d3 31 04 c2 12 11 25 28 5b c0 64 a6 27
##   [121] 34 8a 08 b2 19 15 44 05 f5 ba 5c 94 28 a8 a0 a8 57 44 b9 82 20 a0 22 24
##   [145] 64 01 44 dc 41 bd 2a 48 14 88 4c 26 61 a6 7b 22 41 64 89 46 b8 64 9b e9
##   [169] ae ef fe 51 c9 25 24 dd d5 cb 54 f7 a9 e5 f7 7b 9e 7a 1e ed af be 53 ef
##   [193] d7 5d 43 aa df 3e e7 7c 09 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
##   [217] 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
##   [241] 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
##   [265] 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
##   [289] 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 30 e2 4a ed 00
##   [313] 00 d0 c7 6e 4a 72 f4 11 9c df 24 39 9c e4 cb 49 3e 9f e4 0b 49 3e 97 e4
##   [337] 5f 93 7c 24 c9 3f 25 b9 fd 08 33 d2 5b 63 49 ae 4a 32 79 b7 af 9f 95 e4
##   [361] 9a 15 8e f5 80 24 4f 4a 72 4e 92 47 26 39 35 c9 a6 24 5f 49 e7 3d 70 55
##   [385] 3a ef 83 bf 4c f2 a5 d5 47 5e 17 af 4e f2 b2 05 be fe 73 49 7e 61 9d b3
##   [409] 0c 8a 56 92 83 e9 bc 0f ee d4 a4 f3 3e f8 74 95 44 00 00 00 0c a5 af a4
##   [433] f3 0b e7 5a 3d e6 92 bc 36 c9 b1 eb 75 41 eb e0 eb 93 bc aa 76 88 23 f0
##   [457] 63 b9 e7 cf e9 aa 15 8e f1 d8 24 9f 5c 60 9c 6e 8f 3f 4b b2 ed c8 e3 af
##   [481] 89 e3 93 cc 67 e1 dc 3f 5b 31 d7 7a f9 fa 74 8a e6 43 0b 3c be 6d 89 73
##   [505] bf 27 f7 fc 9e 5d 9d 4e b9 05 00 00 00 3d b1 d6 05 d6 9d 8f 9b d3 99 a5
##   [529] 33 c8 ee 9b e4 cd e9 5c cf c7 2a 67 59 ad 07 66 e1 9f cf 93 96 79 fe f1
##   [553] 49 3e b8 c8 18 cb 7d bc 26 9d 59 60 fd e4 0d 59 3c ef b0 17 58 1b 92 ec
##   [577] cb e2 d7 ff 84 25 ce df 92 4e 51 7d f7 f3 5e b0 46 79 01 00 00 18 41 eb
##   [601] 55 60 dd f9 f8 a6 f5 b9 ac 9e ba 57 92 5f ce d7 5e c7 a0 16 58 7f 9d 7b
##   [625] fe 4c be 94 e5 15 4a 27 a5 b3 e4 b4 17 ef 83 3f 4d a7 38 e9 07 df 96 ee
##   [649] 59 87 bd c0 7a 4d ba 5f ff 52 05 56 92 fc da 02 e7 1d 4a a7 f0 04 00 00
##   [673] 80 23 b6 de 05 d6 ed f9 da fd 72 fa d9 96 24 3f 95 85 67 97 0c 62 81 75
##   [697] 6e 16 fe 99 fc d8 32 ce 3d 26 c9 17 17 39 7f b5 8f df ec c9 55 1d 99 ed
##   [721] 49 6e cd e8 16 58 8f ce d2 3f a7 e5 14 58 27 2f 72 ee 3b 7a 9e 18 00 00
##   [745] 80 91 b4 de 05 56 93 e4 6f d7 e5 ca 8e dc eb b2 f8 35 0c 5a 81 d5 4a 32
##   [769] 93 85 af e5 3e cb 38 ff ed 8b 9c 7b a4 8f 6f e9 c5 c5 ad d2 69 e9 2c 6d
##   [793] 5d 2a e3 b0 16 58 c7 24 f9 af f4 a6 c0 4a 92 7f 58 e4 fc ed 3d 4d 0d 00
##   [817] 43 cc 06 92 00 d0 5f be 39 c9 99 b5 43 8c 98 0b b2 f0 06 ea 7f 96 e4 3f
##   [841] 97 38 f7 21 49 9e dd f3 44 1d af 5f a3 71 97 f2 d4 74 ee b8 b8 b5 d2 eb
##   [865] f7 83 3f 48 72 42 0f c7 7b f5 22 5f 7f 5d 0f 5f 03 00 00 80 11 d5 6d 06
##   [889] d6 5c 97 f3 ca 1d 8f 56 3a fb 27 1d 95 ce 7e 37 df 9c e4 6f ba 8c 79 e7
##   [913] e3 d7 7b 7f 29 3d 37 2c 33 b0 4a 92 7f cb c2 d7 f1 c4 65 9c 7f f7 fd bf
##   [937] 16 7a fc 74 92 07 a5 f3 5e 28 e9 ec 1b f6 ed 49 3e bb 8c 73 c7 8f fc 12
##   [961] 97 6d 73 92 37 2d 23 d3 b0 cf c0 7a 66 96 7f fd cb 9d 81 b5 39 8b df c9
##   [985] 71 3d 7f c6 00 00 00 0c a1 6e 05 d6 a1 55 8e 59 92 fc 46 97 71 9b 74 0a
##  [1009] 95 7e 37 2c 05 d6 b9 59 fc e7 bb 71 89 73 4b 96 5e 66 da 6d 36 dd 96 24
##  [1033] 7b 97 38 ff 79 2b bf a4 15 2b 49 9e 92 e4 f3 4b 64 19 85 02 eb 41 59 bc
##  [1057] 68 3a 92 02 2b 49 de b2 c8 18 bf df a3 ec 00 00 00 8c a8 b5 28 b0 92 64
##  [1081] 53 96 de 1c bb df 97 f9 0f 4b 81 f5 91 2c 7c 0d bf b3 8c 73 1f b4 c8 b9
##  [1105] 77 3e 5e b1 8c 31 ce 58 62 8c 37 2f ff 52 56 6c 43 3a b3 cc 3e b1 44 86
##  [1129] 51 29 b0 c6 b2 f2 ef c5 4a 0a ac 6f ec 32 ce bd 7b 72 05 00 30 c4 fa fd
##  [1153] c3 31 00 0c a3 43 49 2e 5f e2 39 47 f2 df e8 87 24 79 51 92 cb d2 d9 9c
##  [1177] fc d6 74 ca b8 4f a6 33 db e3 f9 49 1e 7c 04 e3 af b5 fb a4 b3 c4 ee d7
##  [1201] 92 fc 69 3a 33 d2 6e 4e e7 2e 8d ff 95 ce 75 bc 3f c9 cf 27 79 5a 3a cb
##  [1225] 33 57 63 5b 92 47 2d 72 6c 39 b3 62 1e b9 c4 f1 37 2d 63 8c 4f a6 fb 3e
##  [1249] 5b 0f 5a c6 18 ab f5 8a 24 7f 1e 7b ae dd e9 15 59 db ef c5 55 49 da 8b
##  [1273] 1c 7b fe 1a be 2e 00 00 00 43 6e ad 66 60 25 c9 2f 75 19 fb 2b ab 1c f3
##  [1297] 8c 2c 7e b7 b3 85 1e 1f 49 72 f6 32 c7 3e 2e 9d 6b be f3 b1 d4 d8 87 16
##  [1321] 78 94 25 5e e3 b4 24 ef 59 41 fe bb 3e de 93 95 97 3d af 5a 64 ac f9 74
##  [1345] 66 c9 2d e5 e2 2e 79 6e 5e 41 8e f7 76 19 e7 bd 2b 18 67 a5 5e d9 e5 75
##  [1369] ef 7c fc 6d ba 2f 2d 1c 96 19 58 0f cf ea de 77 2b 99 81 95 24 ef 5a 64
##  [1393] 9c 1b e3 0f cb 00 00 00 ac d2 5a 16 58 97 77 19 fb 8f 57 38 56 2b 9d bb
##  [1417] 9c ad e6 17 f0 26 c9 1b b3 f4 7e 4f c7 1d c1 f8 77 3e ba 15 58 2f ed c1
##  [1441] f8 4d 92 ef 5c e2 3a ee d4 4a f2 e5 45 c6 b8 72 99 63 3c 20 c9 63 93 4c
##  [1465] df 91 ff d7 d3 29 d2 f6 a6 73 07 c3 e5 5a ac d4 68 d2 99 85 b6 56 96 2a
##  [1489] b0 fe 3a 9d 1b 10 ec ef f2 9c 61 28 b0 b6 26 f9 62 16 be be bd e9 cc 62
##  [1513] ec 55 81 75 7e 97 b1 cc 84 03 00 00 60 55 d6 aa c0 3a 36 dd 37 8a 7e c6
##  [1537] 0a c6 da 90 4e e1 75 a4 c5 cf 3f a4 73 a7 b4 c5 ac 65 81 f5 fd 3d 18 fb
##  [1561] ae 8f 5d dd be 61 77 d8 d9 e5 fc 67 2d e3 fc 5e fa 54 97 2c d3 6b f8 ba
##  [1585] dd 0a ac b7 a4 b3 27 54 32 fc 05 d6 62 05 e2 7c 3a 4b 6d 17 db 27 6d 35
##  [1609] 05 d6 7d bb 8c f5 4b 47 78 1d 00 00 00 8c a8 b5 28 b0 36 a6 b3 7f d3 62
##  [1633] e3 de 9a ce ac 97 e5 28 49 2e ed 32 d6 4a 1f 7f 93 c5 97 31 ad 55 81 75
##  [1657] 66 0f f3 df f5 67 73 cc 12 df bb 97 75 39 7f fb 12 e7 f6 d2 52 df d7 53
##  [1681] d7 f0 b5 17 2b b0 5e 9c af fd 59 0d 73 81 75 61 16 bf b6 3b ef 00 d9 cb
##  [1705] 02 2b 49 be b4 c8 58 ff b1 da 8b 00 00 00 60 b4 f5 a2 c0 6a a5 53 5a dd
##  [1729] 37 c9 d3 d3 d9 90 bc 5b 61 f1 dd 2b c8 f7 82 25 c6 5a cd e3 c7 16 79 ad
##  [1753] b5 28 b0 5a e9 2c d1 ea f5 35 34 49 76 2f f1 bd bb ba cb b9 4b 2d a7 ec
##  [1777] a5 97 74 c9 f1 f9 2c bd 6f d8 91 b8 7b 81 b5 37 9d 1b 00 dc dd b0 16 58
##  [1801] f7 cb e2 fb b9 7d 20 5f fd de f7 ba c0 fa a3 2e e3 dd 77 75 97 02 00 00
##  [1825] c0 28 eb 56 60 dd 59 62 dd fd 31 b7 c4 39 dd 1e bf 9f e5 17 16 c7 2f e3
##  [1849] b5 de 93 64 32 9d 0d c9 cb 1d e7 3c 3b 9d 0d c6 bb 9d 77 e2 02 af b7 16
##  [1873] 05 d6 37 2d f1 fc 0f a6 b3 b9 f6 e6 3b ce 6d 25 b9 57 92 6f 49 f2 d1 25
##  [1897] ce ed 76 07 c0 a3 bb 9c f7 a9 2e e7 f5 da bd d2 99 71 b7 58 96 b5 be 33
##  [1921] dd 9d 05 d6 8d e9 2c 55 5c 6c f6 dd 30 16 58 ad 74 ee 0a b8 d0 35 7d 39
##  [1945] 9d 9f cd 9d 7a 5d 60 fd 50 97 f1 ce 5f c5 78 00 00 00 8c b8 a5 0a ac 5e
##  [1969] 3e de 96 af ee 39 b4 1c 4b 6d da fe c3 5d ce bd 4f ba cf 04 fb 8d 65 bc
##  [1993] fe 6f 75 39 ff 13 cb bc 86 6e 77 1c fc 8b 74 bf 2b db c6 24 b3 5d ce bf
##  [2017] ba cb b9 dd f6 bf fa 9d 65 66 3f 52 ad 74 0a ba c5 72 7c 26 6b 3f 13 ec
##  [2041] c5 e9 cc f8 5b ea 75 86 b1 c0 fa a9 2c 7e 4d 8f bb db 73 7b 5d 60 7d 4b
##  [2065] 97 f1 5e bb 8a f1 00 00 00 18 71 eb 51 60 dd 98 e4 69 2b cc 75 54 ba cf
##  [2089] be fa a3 65 8c 71 5a 97 f3 e7 b2 f4 3e 5c af eb 72 fe c7 96 f1 fa 5b ba
##  [2113] 9c df 24 79 e0 32 c6 78 7e 97 f3 6f ef 72 de 33 ba 9c f7 e2 65 bc ee 91
##  [2137] 6a 25 79 47 97 0c 4d 3a b3 d3 fa c5 b0 15 58 df 90 c5 af e7 75 0b 3c bf
##  [2161] d7 05 d6 43 ba 8c f7 91 55 8c 07 00 23 61 43 ed 00 00 30 e2 36 27 f9 e6
##  [2185] 24 ff 92 e5 6f e2 fc 8d e9 3e 5b eb c7 97 31 c6 a7 92 5c 9e 64 cf 02 c7
##  [2209] c6 ee 78 8d 0f 2d 33 cf 6a dc 9a 64 6b 3a fb 10 9d 9c 4e a1 f6 d0 24 67
##  [2233] 27 39 98 e4 b3 cb 18 a3 db 4c af 3b 97 4d 36 0b 1c 7b 74 97 f3 0e 2e e3
##  [2257] 75 8f 44 49 f2 e6 24 cf ec f2 9c 77 26 f9 fb 35 ce 31 aa 36 27 f9 b3 45
##  [2281] 8e cd 26 f9 d1 75 c8 f0 9f 5d 8e 3d 22 9d 82 b3 bd 0e 39 00 00 00 18 12
##  [2305] eb b9 84 f0 ce e5 43 9b 96 91 eb b5 5d c6 98 59 c1 f5 3d 79 89 2c dd 1c
##  [2329] e9 0c ac 5e 98 e8 92 a1 c9 e2 4b e3 fe b4 cb 39 0f 5d c3 bc 25 c9 1b 97
##  [2353] c8 fc 6f e9 94 2c fd 64 98 66 60 bd 3d 8b 5f cb b6 45 ce e9 f5 0c ac 3b
##  [2377] 8b d5 c5 1e c7 af 62 4c 00 18 7a dd f6 96 00 00 d6 d7 8f a6 33 13 eb d8
##  [2401] 25 9e f7 94 2e c7 fe 6a 05 af b7 af cb b1 95 2e 6b 5c 2f ad 74 8a 86 ef
##  [2425] 4d 67 0f ad d5 38 b9 cb b1 9b 56 39 e6 52 5a e9 94 57 df db e5 39 5f 4e
##  [2449] 72 4e 92 db d6 28 c3 a8 7b 5a 3a 37 31 58 c8 8b d3 99 81 b5 1e 9a 74 9f
##  [2473] 85 75 af 2e c7 00 60 64 59 42 08 00 fd 65 32 c9 87 93 9c 95 e4 f0 02 c7
##  [2497] cb 1d cf 59 cc 4a 96 c0 75 fb 25 7a 3c 9d cf 09 73 2b 18 6f 2d 6c 4a a7
##  [2521] 70 7a 78 3a 77 68 db 93 23 9f a1 b4 d8 4c 9b 24 b9 e5 08 c7 5e c8 58 3a
##  [2545] fb 92 4d 77 79 ce 97 93 9c 91 e5 2f 23 65 65 ee 93 ce 92 d9 85 7c 28 c9
##  [2569] 1b d6 2f 4a 92 e4 f3 e9 64 5a c8 bd 93 7c 7a 1d b3 00 c0 40 30 03 0b 00
##  [2593] 56 67 3e 9d 32 69 b1 47 2b 9d e2 62 53 3a 33 2a ee 9f ce fe 4e 3f 92 ee
##  [2617] c5 51 d2 b9 4b de ff 5c e4 d8 d6 25 ce fd c5 24 87 96 f9 f8 d2 12 63 d5
##  [2641] 98 09 32 96 4e 79 f7 8a 74 96 22 de 9e e4 ba 74 0a a0 ef ca ca ca ab 85
##  [2665] f6 bf da 90 ee 77 dd eb f5 ec a7 8d 49 de 9f e5 95 57 37 f4 f8 b5 e9 68
##  [2689] 25 79 6f 16 5e 9e 7b 4b 3a a5 e8 42 ef 95 b5 74 63 97 63 f7 5e b7 14 00
##  [2713] 00 00 0c 85 6e 7b 60 1d 3a 82 71 b7 a4 53 c8 2c b5 27 d6 71 0b 9c 7b df
##  [2737] 65 9c d7 ab 47 b7 99 4a bd de 03 eb b4 74 96 d8 1d ea 61 fe 85 36 ba 3f
##  [2761] 6a 89 73 7a b9 ff d4 e6 24 ff b0 c4 eb 7d 3e c9 83 7a f8 9a 6b 61 d0 f7
##  [2785] c0 7a 69 16 cf ff c4 65 9c df eb 3d b0 92 e4 8f bb 8c 79 de 2a c7 04 80
##  [2809] a1 66 06 16 00 ac bf 5b 93 3c 2b c9 bf 2e f1 bc f3 17 f8 da 72 36 79 ef
##  [2833] 95 63 d6 e1 35 36 26 f9 9d 74 66 59 7d 6f ba cf 8e 5a a9 85 66 d5 2c f5
##  [2857] d9 a7 57 77 7f db 92 e4 ff a4 73 37 c7 c5 1c 48 67 23 fa 7f ef d1 6b b2
##  [2881] b0 d7 74 39 f6 81 2c 3d 53 f1 51 5d ce ff b3 05 9e bf 73 19 99 ba 2d cd
##  [2905] 2d cb 38 1f 00 46 8e 02 0b 00 ea 68 27 79 d5 12 cf d9 b3 c8 79 eb 65 ad
##  [2929] ef 86 b7 21 9d fd 87 2e 5a e3 d7 b9 ab a5 be 7f bd f8 6c b4 25 c9 df a7
##  [2953] b3 64 74 31 ff 9c ce 52 c9 ff ea c1 eb b1 7a 63 e9 94 a6 dd 1e dd b4 16
##  [2977] 78 fe 72 0a a8 6e ef b3 db 97 71 3e 00 8c 1c 9b b8 03 40 3d 1f 5e e2 f8
##  [3001] 39 0b 7c ed d6 b5 08 b2 88 85 96 e0 f5 d2 af 25 79 cc 0a 9e 7f 53 3a 33
##  [3025] 66 fe 3a 9d 82 e8 b8 74 ff 1e 2e 34 03 6b a1 8d f1 ef ea 48 af f9 a8 74
##  [3049] b2 3d ac cb 73 fe 32 9d 3b e2 1d c9 32 54 06 5b b7 d9 8d ff bd 6e 29 00
##  [3073] 60 80 28 b0 00 a0 9e a5 0a 8c fb 2d f0 b5 2f 2f 71 ce 77 27 f9 c3 d5 c5
##  [3097] 59 57 0f 4a f2 92 25 9e f3 85 24 bf 9b ce 7e 41 d7 e4 9e 9b ce 4f ac e2
##  [3121] 75 db 49 6e 4e 72 f4 22 c7 b7 dc 71 7c 35 c6 d2 d9 b0 bd 5b 79 f5 9e 24
##  [3145] 17 a6 73 13 00 46 d7 89 5d 8e dd b4 6e 29 00 60 80 28 b0 00 a0 9e 93 97
##  [3169] 38 be d0 f2 b2 76 92 cf 24 39 69 91 73 76 1c 49 a0 1e 59 ce 32 bc e7 2e
##  [3193] 71 fc 87 93 bc 3e dd 97 fc ad 76 3f b0 d9 24 df b0 c8 b1 13 d2 fd 0e 71
##  [3217] 8b 29 49 de 9e e4 db ba 3c e7 8a 74 ee 46 a8 bc e2 fe 5d 8e 2d 75 77 50
##  [3241] 00 18 49 f6 c0 02 80 7a 7e 74 89 e3 1f 5f e4 eb 7f d1 e5 9c a7 ae 32 cb
##  [3265] 4a 2d b4 3c ef 4e cb f9 03 d9 33 ba 1c 7b 77 3a 77 39 5c 6a bf aa fb 2c
##  [3289] 71 7c b1 8c d7 1e c1 98 8b 79 51 92 ef ea 72 fc 9f d3 b9 66 e5 15 25 dd
##  [3313] df 67 66 60 01 c0 02 14 58 00 50 c7 45 49 9e be c4 73 fe 64 91 af bf bb
##  [3337] cb 39 8f 4a f7 d9 1d 77 f5 cb e9 ec d7 f4 db 49 7e 30 c9 93 d3 59 96 77
##  [3361] 42 96 fe 8c d0 ad 88 59 ce 9d 04 cf e8 72 ec ca 65 9c 9f 24 0f 5d e2 f8
##  [3385] 62 9b 69 ff 53 97 73 ba 2d ed 5a cc 59 e9 cc 16 5b cc e1 24 e7 a5 fb 9d
##  [3409] e7 18 1d 5b bb 1c bb 2e eb 7b a3 06 00 00 00 86 c0 57 d2 99 c5 b3 d0 63
##  [3433] 25 1b 70 b7 d2 59 ee 76 42 92 c7 a6 b3 11 f9 62 e3 de f5 b1 6d 91 f1 8e
##  [3457] 5f e2 bc cb 97 91 e9 b8 74 0a 95 c5 c6 f8 c0 12 e7 bf aa cb b9 9f 5d e2
##  [3481] dc b2 44 fe d7 2c 23 ff d6 74 96 58 76 1b 67 b1 0d d9 9f d8 e5 9c a5 f6
##  [3505] e5 ba bb 4d e9 5c ef 52 3f cb 43 ab 7c fc d9 22 af db 5a e2 bc 0b 57 78
##  [3529] 1d dd ec ef 72 5d 3f bb c2 b1 1e 90 ee b9 4f ef 4d e4 9e fa 68 16 bf fe
##  [3553] 27 af 62 bc 87 74 19 ef 2d 3d c8 0b 00 43 c9 0c 2c 00 58 9d 8d e9 fe 8b
##  [3577] f8 5d cb a1 f9 24 b7 a7 53 b8 fc 5d 92 a7 2c 63 fc 8f a5 b3 57 d3 42 fe
##  [3601] 3b 9d fd 96 16 33 95 e4 e2 2c 3e 03 a9 24 79 53 ba df 71 ef 95 4b e4 fb
##  [3625] 4a 97 63 0f e8 f2 da 49 e7 7b f2 9f 5d 8e bf 34 9d 4d de 17 73 af 74 ee
##  [3649] e4 77 42 97 e7 24 8b ef 91 d5 6d 09 61 b7 0d d8 17 f2 03 e9 5c ef 52 36
##  [3673] 1e c1 63 35 63 f6 f3 67 bc 6e b9 bb bd 6f 6a e9 36 23 6a 35 4b 42 17 2b
##  [3697] a6 93 ce 1d 36 01 80 05 f4 f3 87 1b 00 e8 77 dd 7e 11 ef 56 0e 2d c7 0b
##  [3721] 97 38 fe 73 4b 1c 7f 45 92 ff 93 e4 9c 24 9b ef f8 da 58 3a 4b 04 3f 90
##  [3745] ce 66 e2 8b d9 9b e4 1f 97 18 ff bf 97 38 fe 23 77 bc 5e 2b c9 d7 27 79
##  [3769] 41 be b6 9c f8 ab 2e e7 6e 4c e7 ae 83 bb f3 d5 e5 56 ad 74 36 ae ff b1
##  [3793] 24 9f 4f f2 8d 4b bc 7e f2 d5 eb be bb cf 24 b9 6d 91 63 8f 5d c6 b8 77
##  [3817] 3a 2a c9 2f ae e0 f9 90 24 67 77 39 76 d5 ba a5 00 00 00 60 68 74 5b 42
##  [3841] b8 96 8f d7 2d 33 df ef ac d1 eb 2f 76 87 be bb 7a f2 2a c6 3d fa 2e e7
##  [3865] 9f b7 46 d9 ef fa d8 de 25 ff bb ba 9c b7 dc bb 1b 7e e7 3a 5c c3 62 33
##  [3889] 72 5a 4b 9c d7 ad a0 5c a9 5e 2f 21 ec 96 7b bc 37 91 7b ea 23 59 3c ef
##  [3913] 13 56 31 de 7b ba 8c b7 da 3b 6b 02 c0 d0 33 03 0b 00 fa cb 5f a6 b3 84
##  [3937] 6e 39 7e 28 9d 72 a1 97 5e 91 e4 5f 97 f1 bc 4f ac 62 ec bb 2e f9 fb cb
##  [3961] 24 5f 58 c5 18 77 f7 b9 2e c7 26 bb 1c eb b6 51 fc c9 cb 7c ed e5 fe 9c
##  [3985] e0 4e 25 c9 e3 17 39 f6 a1 ac 6c 6f 3d 00 18 29 0a 2c 00 e8 1f 6f 4d 67
##  [4009] 66 d2 72 f7 d5 b9 3d c9 a3 93 7c ba 47 af ff c6 2c bd f7 d5 9d 3e 97 e4
##  [4033] ea 15 8e 7f bf bb fc df 87 93 7c fb 0a cf bf bb 0b d2 b9 7b e2 62 be b3
##  [4057] cb b1 6e 7b 0d 9d b3 8c d7 3e 36 9d 3b 3e c2 4a 7c 5d 3a ef 9d 85 74 db
##  [4081] d7 0e 00 46 9e 02 0b 00 ea fb a7 74 8a a8 e7 a5 b3 f9 fb 4a dc 94 ce be
##  [4105] 56 ef 3f c2 0c 2f 4f f2 fd e9 2c 63 5a ae e7 ae f0 35 ee be 31 fb 47 93
##  [4129] 7c f7 0a c7 48 92 9b d3 d9 6c fd 7d 49 fe b6 cb f3 9e 99 c5 f7 c1 fa 7c
##  [4153] 16 9f bd f6 f4 65 64 98 58 c6 73 58 b9 95 be ff 07 cd c3 bb 1c 5b ec 8e
##  [4177] 93 00 40 14 58 00 b0 16 e6 d3 59 0a 74 5b 3a 9b 9d ff 7b 3a 65 c9 d5 e9
##  [4201] 6c 8e 7e 45 92 5f 4d e7 6e 81 0f 48 a7 bc fa a7 23 78 bd 5b d2 99 cd f4
##  [4225] f8 ac 7c 56 d4 7b 92 ec 48 f2 0b 59 59 79 95 74 ee 94 78 76 3a 25 da 72
##  [4249] 3c 64 81 af fd 61 3a 4b fd ae 59 e6 18 6f 4d f2 c0 24 1f bf e3 ff ff 62
##  [4273] ba df 55 70 57 97 63 af 5d e4 eb 4f cb d2 9b f0 3f 74 89 e3 ac ce 62 9b
##  [4297] eb 0f 8b a7 2d f2 f5 8f 25 f9 ec 7a 06 01 00 00 80 da 1e 9c ce 6c ae df
##  [4321] 4b e7 17 e3 2f a5 53 aa 7d 25 9d 0d a9 df 94 ce 26 df f7 ed d1 eb 6d 4c
##  [4345] a7 40 fb 9d 24 b3 e9 cc a2 b9 35 9d 62 e9 d2 74 f6 ea da 99 ee 7f 38 2b
##  [4369] e9 6c 1e ff ca 74 f6 c7 ba e9 8e cc ff 91 e4 bd e9 dc 95 b1 57 79 ef 74
##  [4393] 42 16 df 4c db 0c ab f5 f1 53 f9 da ef fb 31 75 e3 ac a9 92 ce bf c5 b5
##  [4417] de 74 1f 00 00 00 18 32 97 64 e1 42 e1 65 35 43 8d 90 b7 e5 6b bf ef c3
##  [4441] bc 3a 60 5b 16 2f 4c b7 56 cc 05 00 00 00 f4 b9 47 64 e1 42 61 a6 66 a8
##  [4465] 11 51 d2 99 69 77 e7 f7 fc a3 75 e3 ac b9 1f cc c2 ef b5 37 d4 0c 05 00
##  [4489] 00 00 f4 bf 92 ce 52 c7 85 8a 85 85 f6 ec a2 77 9e 90 af fd 7e bf a8 6e
##  [4513] 9c 35 b7 3f 0b bf cf ee 7e 73 03 00 00 00 80 7b b8 7b 91 72 e7 e3 67 6b
##  [4537] 86 1a 72 db 93 7c 39 5f fd 5e df 9a e1 de ff 6a 7b 16 7e 8f 5d 52 33 14
##  [4561] 00 00 00 30 38 4a 16 9e 1d f3 95 74 36 a8 a7 77 8e 49 f2 e3 b9 e7 f7 ba
##  [4585] db dd 22 87 c1 af 65 e1 02 eb d4 9a a1 00 00 00 80 c1 f2 ad 59 b8 60 b8
##  [4609] a0 66 a8 21 b4 50 79 f5 93 55 13 ad bd ad e9 dc 51 f3 ee d7 fd fb 35 43
##  [4633] 01 00 00 00 83 e9 6f 73 cf 92 e1 93 55 13 0d 9f a3 93 1c ca 57 67 b8 3d
##  [4657] b5 6e 9c 75 f1 c2 dc f3 7d 35 9f e4 3e 35 43 01 00 00 00 83 e9 94 2c 3c
##  [4681] 0b eb 8c 9a a1 86 d0 0f 27 f9 e9 74 66 26 0d bb 56 92 cf e7 9e ef a9 1f
##  [4705] af 19 0a 00 00 00 18 6c af cd 3d cb 86 cb aa 26 62 90 3d 29 f7 7c 3f 7d
##  [4729] 36 c9 a6 9a a1 00 00 00 80 c1 76 54 16 9e 31 f3 c0 9a a1 18 48 25 c9 de
##  [4753] dc f3 bd f4 e8 9a a1 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
##  [4777] 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
##  [4801] 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
##  [4825] 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
##  [4849] 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
##  [4873] 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
##  [4897] 00 00 00 00 46 57 a9 1d 00 60 d0 5d 75 d1 d9 1b 4f b8 e9 a6 ad 63 f3 47
##  [4921] 6d 3d dc 1e db da 6a e6 b6 b6 d2 da 3a 3f 36 bf b5 a4 b5 b5 34 65 6b bb
##  [4945] 69 1f 5d 5a 65 6b 93 32 df 4a f9 cf f9 f6 fc 8d a5 94 1b c7 36 b4 6e fc
##  [4969] cf 13 36 ff e7 23 de f8 cf 87 6b 5f 07 00 00 40 bf 52 60 01 2c 43 93 94
##  [4993] 83 17 9c 71 e2 7c 6b 6e a2 d5 2a 3b 9b 94 89 d2 34 13 49 76 26 79 40 0f
##  [5017] 5e e2 cb 25 e5 c6 26 cd 8d 69 72 63 29 b9 b1 69 9a d9 76 69 7d 7c ac 34
##  [5041] 1f 3f e5 f2 7d 9f 2e 49 d3 83 d7 01 00 00 18 38 0a 2c 80 bb d9 3b 3d 79
##  [5065] cc d6 f6 fc 37 b5 db 99 6c 52 26 4a ca 44 d2 ec 4c 72 42 c5 58 37 25 e5
##  [5089] e3 49 f3 f1 26 f9 78 69 37 1f bb e9 7e 47 5f 6b e6 16 00 00 30 0a 14 58
##  [5113] c0 c8 eb cc ae 3a fd b4 66 ac f5 d4 34 79 6a 4a 1e 97 64 63 ed 5c cb 70
##  [5137] 28 c9 27 53 f2 b1 d2 2e 1f 3a 3c 3f f7 a7 a7 bf ef 53 37 d6 0e 05 00 00
##  [5161] d0 6b 0a 2c 60 24 dd 30 7d d2 96 db 0f df eb dc b4 f2 d4 56 93 a7 36 69
##  [5185] b6 d5 ce d4 03 ed 94 7c b8 a4 bc 3f 4d 79 ff 29 57 ec bd da b2 43 00 00
##  [5209] 60 18 28 b0 80 91 71 70 f7 f8 c9 4d 29 4f 6b 3a b3 ac be 35 c9 e6 da 99
##  [5233] d6 56 f3 99 24 ef 2f ed bc 7f 73 fb e8 bf 7a e0 fb fe f9 96 da 89 00 00
##  [5257] 00 56 43 81 05 0c b5 66 3a 63 07 0f 8d 3f b5 69 95 97 24 79 62 ed 3c 15
##  [5281] dd 96 e4 83 29 f9 e3 b9 c3 f3 97 58 6a 08 00 00 0c 12 05 16 30 94 6e 98
##  [5305] 9e bc f7 ed 73 ed 17 94 e4 45 49 4e ae 9d a7 cf 1c 6e 92 2b 4a 2b 6f da
##  [5329] f6 0d fb fe ba 5c 9c 76 ed 40 00 00 00 dd 28 b0 80 a1 72 70 f7 e4 59 ed
##  [5353] 56 fb 07 d3 e4 59 19 fa 25 82 47 ae a4 cc a6 e4 cd ad 34 bf 77 f2 e5 fb
##  [5377] fe a3 76 1e 00 00 80 85 28 b0 80 81 77 d5 45 67 6f 3c fe c6 5b f6 94 a6
##  [5401] bc 24 69 be a9 76 9e 01 35 5f 92 3f 6e 4a de b4 6d 6c df 9f 96 4b 33 5f
##  [5425] 3b 10 00 00 c0 9d 14 58 c0 c0 da 3b 3d 79 cc 96 b9 f6 0f 37 c9 8b 93 3c
##  [5449] a0 76 9e e1 d1 7c 26 69 bd 25 1b ca 9b b7 5f ba f7 d3 b5 d3 00 00 00 28
##  [5473] b0 80 81 d3 4c 67 6c 76 6e e7 73 92 e6 17 a2 b8 5a 4b ed a4 f9 a3 b1 6c
##  [5497] f8 85 93 af f8 e4 be da 61 00 00 80 d1 a5 c0 02 06 ca 81 a9 c9 6f 2d 69
##  [5521] ff 6a 92 b3 6a 67 19 21 4d 92 77 97 56 fb 55 db 2e bb ee ea da 61 00 00
##  [5545] 80 d1 a3 c0 02 06 c2 ec 05 a7 9f de 1e 2b bf 5c 52 2e a8 9d 65 a4 95 5c
##  [5569] d9 4e 5e 79 ea e5 fb fe a5 76 14 00 00 60 74 28 b0 80 be b6 6f 6a fc 3e
##  [5593] 47 25 3f db a4 bc 28 c9 86 da 79 f8 ff de df 6a 97 57 9e f2 9e 6b 3e 52
##  [5617] 3b 08 00 00 30 fc 14 58 40 5f da 7f de 8e a3 5a 9b 37 bc 38 29 3f 93 e4
##  [5641] f8 da 79 58 d4 5f 94 76 79 e5 b6 f7 5c f3 77 b5 83 00 00 00 c3 4b 81 05
##  [5665] f4 9d d9 5d e3 e7 37 ad f2 eb 49 b6 d7 ce c2 72 95 0f b6 5b f9 d1 53 2f
##  [5689] bb e6 63 b5 93 00 00 00 c3 47 81 05 f4 8d bd d3 93 c7 6c 9e 6f ff af 34
##  [5713] 79 41 ed 2c ac 4a d3 24 bf 9b 6c 7c f9 8e 2b ae fe 42 ed 30 00 00 c0 f0
##  [5737] 50 60 01 7d e1 e0 ae 9d 8f 6e 8f 35 7f 90 26 3b 6a 67 e1 88 7d 39 69 7e
##  [5761] ee b6 0d 63 af 9f bc 74 ef a1 da 61 00 00 80 c1 a7 c0 02 aa fa e0 b9 e7
##  [5785] 6e 78 f0 bd 3f ff 53 a5 c9 2b 92 8c d5 ce 43 4f 5d d7 34 e5 a5 3b ae bc
##  [5809] e6 03 b5 83 00 00 00 83 4d 81 05 54 f3 a9 0b 4f db 36 d6 8c fd 7e 9a 3c
##  [5833] a6 76 16 d6 50 93 3f 29 ed f6 cb b6 bd f7 ba eb 6a 47 01 00 00 06 93 02
##  [5857] 0b 58 77 4d 52 66 a6 c6 9f 53 52 7e 33 c9 b1 b5 f3 b0 2e e6 92 f2 ba 6c
##  [5881] b8 ed e7 b7 5f 3a fb a5 da 61 00 00 80 c1 a2 c0 02 d6 d5 0d d3 93 f7 3e
##  [5905] 34 d7 fe df 49 a6 6b 67 a1 8a 2f 26 e5 27 b7 5d 71 cd ef 95 a4 a9 1d 06
##  [5929] 00 00 18 0c 0a 2c 60 dd 1c 98 9a fc d6 92 f6 db 93 3c a8 76 16 2a 6b f2
##  [5953] 97 a5 95 17 6e bb 7c df bf d5 8e 02 00 00 f4 3f 05 16 b0 e6 9a 8b d3 3a
##  [5977] 78 f5 c4 cf 35 4d 5e 5e 3b 0b 7d e5 2b 49 f9 f1 6d 0f bd e6 8d e5 e2 b4
##  [6001] 6b 87 01 00 00 fa 97 02 0b 58 53 37 4c 9f b4 e5 d0 dc b1 6f 8b 25 83 2c
##  [6025] aa f9 eb b1 b9 e6 85 27 bf ef ba 83 b5 93 00 00 00 fd 49 81 05 ac 99 03
##  [6049] 53 67 9e 98 1c 7e 4f 49 ce a9 9d 85 3e 57 72 73 69 f2 13 a7 3c 74 df 1b
##  [6073] cc c6 02 00 00 ee 4e 81 05 ac 89 fd bb 26 77 b6 5a ed f7 27 39 b9 76 16
##  [6097] 06 47 93 fc 4d d3 8c bd e0 d4 2b 3f 39 53 3b 0b 00 00 d0 3f 14 58 40 cf
##  [6121] cd 5c 38 f1 84 b4 f3 ee 24 c7 d5 ce c2 40 ba a5 49 f9 a9 ed 0f bd e6 f5
##  [6145] 66 63 01 00 00 89 02 0b e8 b1 d9 3d 13 2f 6c 9a bc 21 c9 86 da 59 18 78
##  [6169] 7f d7 cc b7 9e bf e3 bd 7b 0f d4 0e 02 00 00 d4 a5 c0 02 7a a2 b9 38 ad
##  [6193] d9 4f 4c fc 62 92 9f a8 9d 85 21 52 72 73 69 97 ef df 76 e5 35 7f 50 3b
##  [6217] 0a 00 00 50 8f 02 0b 38 62 9d 3b 0d 1e f3 f6 a4 3c bd 76 16 86 55 f3 7b
##  [6241] 47 df bc e9 25 f7 ff f3 ab 6f ae 9d 04 00 00 58 7f 0a 2c e0 88 cc 5e 70
##  [6265] c6 fd 9a b1 f9 f7 26 79 54 ed 2c 0c b7 26 d9 97 e4 19 3b ae d8 f7 c9 da
##  [6289] 59 00 00 80 f5 d5 aa 1d 00 18 5c d7 4f 9d 31 d1 8c cd 7f 24 ca 2b d6 41
##  [6313] 49 26 4a f2 d1 d9 3d 13 2f 6c fc 01 06 00 00 46 8a 5f 00 80 55 b9 7e ea
##  [6337] 8c 89 f9 cc 7d 28 29 27 d6 ce c2 48 7a 67 fb b6 c3 df 77 ea 07 0e 7c b9
##  [6361] 76 10 00 00 60 ed 29 b0 80 15 9b bd e0 f4 d3 9b b1 d6 87 92 dc bf 76 16
##  [6385] 46 da 4c bb e4 19 a7 5e be ef 5f 6a 07 01 00 00 d6 96 25 84 c0 8a cc 4c
##  [6409] 8d 9f d6 8c b5 3e 18 e5 15 f5 6d 6f 35 f9 c7 03 7b 76 be c4 92 42 00 00
##  [6433] 18 6e 3e f0 03 cb 76 e0 82 c9 1d 19 6b ff 4d 49 1e 58 3b 0b 7c 8d 92 2b
##  [6457] 37 dc be e1 f9 0f 79 ff bf de 54 3b 0a 00 00 d0 7b 0a 2c 60 59 f6 ef 3e
##  [6481] 63 7b ab cc 7d 28 29 27 d5 ce 02 0b 29 29 b3 f3 ad 5c 70 ea 65 d7 ec ad
##  [6505] 9d 05 00 00 e8 2d 4b 08 81 25 7d ea c2 d3 b6 b5 ca fc 07 95 57 f4 b3 26
##  [6529] cd b6 56 bb f9 f0 fe a9 9d bb 6a 67 01 00 00 7a 4b 81 05 74 75 70 f7 f8
##  [6553] c9 63 f3 63 1f 4c f2 e0 da 59 60 19 8e 69 a5 b9 f2 c0 9e 89 9f b1 2f 16
##  [6577] 00 00 0c 0f 1f ee 81 45 cd ee 99 78 48 d3 e4 43 49 4e ae 1c 05 56 a1 79
##  [6601] f7 6d 1b c6 9e 37 79 e9 de af d4 4e 02 00 00 1c 19 05 16 b0 a0 03 bb ce
##  [6625] 78 70 19 9b ff 9b 34 39 a5 76 16 38 02 57 b7 9a 66 d7 29 57 5e 7b 7d ed
##  [6649] 20 00 00 c0 ea 59 42 08 dc c3 fe 3d 67 9e 54 5a f3 1f 54 5e 31 04 ce 6c
##  [6673] 4a b9 6a ff ee f1 73 6b 07 01 00 00 56 4f 81 05 7c 8d eb f7 4c 3c a0 95
##  [6697] c3 1f 4c b2 bd 76 16 e8 85 26 b9 4f ab 94 bf 98 d9 33 f1 22 fb 62 01 00
##  [6721] c0 60 f2 41 1e f8 ff 3e f7 a4 33 8f be f9 e8 c3 7f 9b e4 e1 b5 b3 c0 5a
##  [6745] 28 a5 bc f1 d6 b1 f2 92 c9 4b f7 1e aa 9d 05 00 00 58 3e 33 b0 80 24 49
##  [6769] 33 9d b1 5b 8e 3e fc 87 51 5e 31 c4 9a a6 b9 68 f3 5c fb af 0e 4c 9d 79
##  [6793] 62 ed 2c 00 00 c0 f2 29 b0 80 24 c9 cc e1 9d af 69 92 5d b5 73 c0 3a 78
##  [6817] 6c 29 87 3f 3c 33 35 7e 5a ed 20 00 00 c0 f2 58 42 08 e4 c0 9e 9d df 57
##  [6841] 9a e6 7f d7 ce 01 eb a9 24 ff 39 9f e6 fc 53 af b8 f6 1f 6b 67 01 00 00
##  [6865] ba 53 60 c1 88 9b 9d 9a 78 62 93 7c 20 c9 58 ed 2c 50 c1 6d 4d 93 67 ee
##  [6889] b8 72 df 95 b5 83 00 00 00 8b b3 84 10 46 d8 fe 0b 77 4e 36 c9 bb a3 bc
##  [6913] 62 74 6d 2e 25 97 cf 4e 4d bc b8 76 10 00 00 60 71 66 60 c1 88 9a bd e0
##  [6937] 8c fb 35 63 f3 1f 49 f2 90 da 59 a0 4f bc 66 db 43 f7 fd 74 b9 38 ed da
##  [6961] 41 00 00 80 af a5 c0 82 11 74 c3 f4 49 5b 0e cd 1f fb c1 34 79 74 ed 2c
##  [6985] d0 57 9a f2 8e f6 ed 87 9e 7f ea 07 0e dc 5e 3b 0a 00 00 f0 55 96 10 c2
##  [7009] 88 69 2e 4e eb d0 dc b1 6f 53 5e c1 02 4a f3 5d ad cd 9b 3e 70 70 f7 59
##  [7033] c7 d7 8e 02 00 00 7c 95 02 0b 46 cc cc 27 26 7e 3e c9 74 ed 1c d0 bf 9a
##  [7057] c7 b7 cb ed 7f 77 60 d7 19 0f ae 9d 04 00 00 e8 b0 84 10 46 c8 81 dd e3
##  [7081] cf 2d a5 fc 5e ed 1c 30 20 fe 3d ed e6 bc ed ef b9 f6 5f 6b 07 01 00 80
##  [7105] 51 a7 c0 82 11 b1 7f f7 f8 b9 ad 52 fe 3c c9 c6 da 59 60 80 7c b9 dd 34
##  [7129] bb 4e bd f2 da 0f d5 0e 02 00 00 a3 4c 81 05 23 60 ff ee 33 b6 b7 ca fc
##  [7153] 47 93 9c 50 3b 0b 0c a0 db d2 ca 85 db 2f db f7 27 b5 83 00 00 c0 a8 b2
##  [7177] 07 16 0c b9 fd e7 ed 38 aa 55 e6 2f 89 f2 0a 56 6b 73 da b9 72 76 6a a7
##  [7201] bd e3 00 00 a0 12 05 16 0c b9 b2 65 d3 6b 93 3c bc 76 0e 18 70 1b 9b 34
##  [7225] ef 3c b0 7b fc b9 b5 83 00 00 c0 28 b2 84 10 86 d8 cc ee f1 0b 53 ca bb
##  [7249] 6b e7 80 61 52 9a e6 25 db ae bc f6 f5 b5 73 00 00 c0 28 51 60 c1 90 fa
##  [7273] d4 85 a7 6d 1b 6b 8f 7d 2c c9 bd 6a 67 81 a1 d3 34 3f bd fd ca 6b 7f a9
##  [7297] 76 0c 00 00 18 15 96 10 c2 10 da 7f de 8e a3 c6 e6 c7 de 15 e5 15 ac 8d
##  [7321] 52 7e 71 66 6a e7 2f 35 fe 10 04 00 00 eb 42 81 05 43 a8 b5 79 e3 6b 52
##  [7345] f2 88 da 39 60 b8 35 3f 39 3b 35 f1 9b cd c5 fe 5b 0a 00 00 6b cd 5f 8e
##  [7369] 61 c8 1c d8 33 31 55 9a 5c 5e 3b 07 8c 8a 92 bc ed df 6e ba df 0b 1f ff
##  [7393] a1 0f cd d5 ce 02 00 00 c3 4a 81 05 43 e4 fa f3 4f 3f 65 7e 43 eb 5f 92
##  [7417] 1c 5f 3b 0b 8c 94 52 2e bb 6d ac 7c d7 e4 a5 7b 0f d5 8e 02 00 00 c3 c8
##  [7441] b2 07 18 12 7b a7 27 37 cd 6f 28 ef 8a f2 0a d6 5f d3 5c b8 65 6e fe 3d
##  [7465] 37 4c 9f b4 a5 76 14 00 00 18 46 0a 2c 18 12 9b e7 da af 4e ca 23 6b e7
##  [7489] 80 51 d5 a4 3c e5 d0 e1 63 df ab c4 02 00 80 de b3 84 10 86 c0 fe a9 9d
##  [7513] bb 5a 69 ae ac 9d 03 48 92 fc c5 a6 0d ff 77 d7 83 2f fd cc ad b5 83 00
##  [7537] 00 c0 b0 50 60 c1 80 9b dd 33 f1 90 a6 c9 c7 63 e9 20 f4 93 3f 6f 1d 77
##  [7561] eb ae 53 de 7a fd 6d b5 83 00 00 c0 30 b0 84 10 06 d8 55 17 9d bd b1 dd
##  [7585] e4 9d 51 5e 41 bf 79 d2 fc 7f 6f bd f2 e0 73 4f de 5c 3b 08 00 00 0c 03
##  [7609] 05 16 0c b0 e3 bf 78 f3 2f 96 e4 9c da 39 80 7b 2a a5 79 72 f3 a5 cd 57
##  [7633] 28 b1 00 00 e0 c8 59 42 08 03 6a 66 f7 ce 6f 4b 69 fe b2 76 0e 60 09 4d
##  [7657] fe a4 7d fb e1 3d a7 7e e0 c0 ed b5 a3 00 00 c0 a0 52 60 c1 00 ba f6 82
##  [7681] d3 8f dd 38 d6 fa d7 24 0f a9 9d 05 58 06 25 16 00 00 1c 11 4b 08 61 00
##  [7705] 6d dc d0 7a 4d 94 57 30 38 4a 9e da da bc f1 b2 fd e7 ed 38 aa 76 14 00
##  [7729] 00 18 44 66 60 c1 80 39 30 35 f9 ad 25 ed bf aa 9d 03 58 85 52 de d7 be
##  [7753] f5 d0 b4 99 58 00 00 b0 32 0a 2c 18 20 7b a7 27 8f d9 3c df be 3a 4d 4e
##  [7777] a9 9d 05 58 a5 52 de 77 db 58 79 fa e4 a5 7b 0f d5 8e 02 00 00 83 c2 12
##  [7801] 42 18 20 47 cd 37 bf a4 bc 82 01 d7 34 e7 6f 99 6b 5f b2 77 7a 72 53 ed
##  [7825] 28 00 00 30 28 14 58 30 20 0e 4e 8d 3f ae 34 cd 0f d6 ce 01 1c b9 26 d9
##  [7849] 75 d4 5c fb f7 9b e9 8c d5 ce 02 00 00 83 40 81 05 03 e0 73 4f 3a f3 e8
##  [7873] 26 ad b7 d4 ce 01 f4 4e 49 9e 31 3b 3f f1 3b 8d e5 fc 00 00 b0 24 05 16
##  [7897] 0c 80 9b 8f 3e fc 0b 4d 9a 6d b5 73 00 3d d6 e4 05 b3 53 13 bf aa c4 02
##  [7921] 00 80 ee 2c 5d 80 3e 37 bb 6b e7 37 a7 e4 7f c7 2f b8 30 ac be f1 a6 f1
##  [7945] af 6b 5e 77 ed 8d 7f 53 3b 08 00 00 f4 2b bf 10 43 1f fb ec f9 67 6f bd
##  [7969] 75 e3 2d 9f 48 93 1d b5 b3 00 6b ad f9 d1 ed 57 5c fb 6b b5 53 00 00 40
##  [7993] 3f b2 84 10 fa d8 ad 1b 6e 79 95 f2 0a 46 45 f9 d5 d9 3d 13 2f ac 9d 02
##  [8017] 00 00 fa 91 19 58 d0 a7 0e ec 1a 7f 4c 69 95 ff 13 ff 4e 61 94 34 4d c9
##  [8041] 33 77 5c be ef 5d b5 83 00 00 40 3f f1 8b 31 f4 a1 1b a6 4f da 72 68 ee
##  [8065] d8 8f 27 39 ad 76 16 60 dd cd 25 cd d4 f6 2b ae fd e3 da 41 00 00 a0 5f
##  [8089] 58 42 08 7d e8 f6 b9 63 7e 3e ca 2b 18 55 1b 92 f2 ee 03 7b 4e 7f 7c ed
##  [8113] 20 00 00 d0 2f cc c0 82 3e 73 70 cf c4 39 ed 26 7f 1f 05 33 8c ba af b4
##  [8137] da e5 09 a7 bc e7 9a 8f d4 0e 02 00 00 b5 29 b0 a0 8f ec 3f 6f c7 51 ad
##  [8161] cd 9b 3e 9e 34 e3 b5 b3 00 7d e1 a6 d2 6a 9f bb ed b2 eb ae ae 1d 04 00
##  [8185] 00 6a 32 c3 03 fa 48 6b f3 c6 1f 51 5e 01 77 71 42 d3 6e fd f9 fe dd 67
##  [8209] 6c af 1d 04 00 00 6a 32 03 0b fa c4 fe 3d 67 9e d4 ca e1 6b d3 e4 e8 da
##  [8233] 59 80 be 33 53 e6 c7 be 69 db 7b 3f f9 f9 da 41 00 00 a0 06 33 b0 a0 4f
##  [8257] 8c b5 0f bf 56 79 05 2c 62 7b 33 36 ff 27 d7 5e a6 c3 75 7e 00 00 20 00
##  [8281] 49 44 41 54 70 fa b1 b5 83 00 00 40 0d 66 60 41 1f 38 b0 e7 f4 c7 97 a6
##  [8305] f5 d7 b5 73 00 7d ae c9 5f de b6 b1 f5 b4 c9 4b f7 1e aa 1d 05 00 00 d6
##  [8329] 93 19 58 50 d9 55 17 9d bd b1 34 ad df ac 9d 03 18 00 25 4f d8 3c d7 7e
##  [8353] 6b 73 b1 ff 7e 03 00 30 5a 7c 00 86 ca 4e b8 f1 96 17 27 99 ac 9d 03 18
##  [8377] 18 cf 9c fd c4 c4 6b 1b b3 a8 01 00 18 21 3e fc 42 45 07 a7 27 ef df 9e
##  [8401] 6b 5f 97 e4 5e b5 b3 00 83 a5 a4 fc f8 b6 2b ae 79 6d ed 1c 00 00 b0 1e
##  [8425] cc c0 82 8a 9a b9 f6 ab a3 bc 02 56 a1 49 f3 2b b3 7b 26 9e 5d 3b 07 00
##  [8449] 00 ac 07 33 b0 a0 92 03 bb c6 1f 53 5a e5 ef 6b e7 00 06 da 5c 93 d6 f9
##  [8473] 3b ae d8 fb a7 b5 83 00 00 c0 5a 32 03 0b 2a 68 a6 33 d6 2a e5 f5 b5 73
##  [8497] 00 03 6f 43 29 ed 77 cf ec 99 7c 64 ed 20 00 00 b0 96 14 58 50 c1 ec dc
##  [8521] ce ef 6d 4a 1e 56 3b 07 30 04 9a 1c 9d a6 fd 27 33 53 e3 a7 d5 8e 02 00
##  [8545] 00 6b c5 12 42 58 67 fb a6 c6 ef b3 29 e5 53 49 ee 5d 3b 0b 30 54 ae 1f
##  [8569] 2b 79 cc c9 97 ef fb 8f da 41 00 00 a0 d7 cc c0 82 75 b6 29 f9 85 28 af
##  [8593] 80 de 3b 79 be 69 3e b0 ff bc 1d 6e 0c 01 00 c0 d0 51 60 c1 3a 3a b0 7b
##  [8617] fc ec a4 5c 54 3b 07 30 ac ca 43 cb 51 9b 2e b9 ea a2 b3 37 d6 4e 02 00
##  [8641] 00 bd a4 c0 82 75 d2 5c 9c 56 3a 1b b7 5b ba 0b ac 99 52 9a 27 1f ff c5
##  [8665] 5b 7e ab f1 bf 35 00 00 0c 11 05 16 ac 93 99 8f 8f 3f a7 24 e7 d4 ce 01
##  [8689] 0c bf 92 7c ef ec d4 c4 ff a8 9d 03 00 00 7a c5 5f 67 61 1d 1c dc 7d d6
##  [8713] f1 ed 72 db 75 49 39 b1 76 16 60 74 34 25 df b9 e3 f2 7d ef aa 9d 03 00
##  [8737] 00 8e 94 19 58 b0 0e da e5 f6 8b 95 57 c0 7a 2b 4d de 36 33 35 f9 d8 da
##  [8761] 39 00 00 e0 48 99 81 05 6b 6c 66 6a fc b4 a4 5c 93 64 ac 76 16 60 24 fd
##  [8785] 57 5a 39 67 fb 65 fb f6 d7 0e 02 00 00 ab 65 06 16 ac b9 f2 aa 28 af 80
##  [8809] 7a ee 9d 76 3e b0 7f 7a c7 7d 6b 07 01 00 80 d5 32 03 0b d6 d0 cc 85 a7
##  [8833] 3f 22 ed d6 47 6b e7 00 28 c9 3f 6e dc f0 7f bf ed c1 97 7e e6 d6 da 59
##  [8857] 00 00 60 a5 cc c0 82 35 d2 24 25 ed f2 9a da 39 00 92 a4 49 be f1 d0 dc
##  [8881] 31 6f 6f 2e f6 df 7e 00 00 06 8f 0f b1 b0 46 0e ee 1e 7f 62 52 be b5 76
##  [8905] 0e 80 af 2a 4f 3f f8 89 89 57 d7 4e 01 00 00 2b a5 c0 82 35 d0 99 e1 50
##  [8929] fc 92 08 f4 9d 26 f9 f1 99 dd 13 3f 50 3b 07 00 00 ac 84 02 0b d6 c0 ec
##  [8953] c7 c7 bf a3 29 79 58 ed 1c 00 0b 2a 79 fd cc 85 13 4f ad 1d 03 00 00 96
##  [8977] cb 26 ee d0 63 7b a7 27 37 6d 99 6b f6 35 69 b6 d5 ce 02 b0 a8 92 9b 4b
##  [9001] 69 3f 66 db 65 d7 5d 5d 3b 0a 00 00 2c c5 0c 2c e8 b1 2d 87 e7 2f 52 5e
##  [9025] 01 7d af c9 d1 cd 7c eb 7d b3 17 9c 71 bf da 51 00 00 60 29 66 60 41 0f
##  [9049] ed 9d 9e 3c 66 f3 dc fc 4c 52 4e ac 9d 05 60 39 9a e4 c3 63 c7 dd fa f8
##  [9073] 53 de 7a fd 6d b5 b3 00 00 c0 62 cc c0 82 1e da 3c 37 ff 32 e5 15 30 48
##  [9097] 4a 72 4e fb bf b7 be b9 f1 47 2d 00 00 fa d8 58 ed 00 30 2c 0e 4c 9d 79
##  [9121] 62 49 fb 5d 49 36 d5 ce 02 b0 22 25 df f0 5f e3 f7 9d 7b dd b5 37 fe 6d
##  [9145] ed 28 00 00 b0 10 33 b0 a0 47 5a 39 f4 3f 93 1c 53 3b 07 c0 6a 94 92 57
##  [9169] ce 4e ed 9c ae 9d 03 00 00 16 62 b9 00 f4 c0 a7 2e 3c 6d db 58 7b ec da
##  [9193] 24 1b 6b 67 01 38 02 b7 a6 d5 fe 96 ed 97 5d 77 55 ed 20 00 00 70 57 66
##  [9217] 60 41 0f 8c b5 5b 3f 1f e5 15 30 f8 b6 a4 dd 7a ef fe 3d 67 9e 54 3b 08
##  [9241] 00 00 dc 95 19 58 70 84 0e ee 9e 3c ab 5d da ff 12 ff 9e 80 e1 f1 2f 47
##  [9265] df bc f1 5b ee ff e7 57 df 5c 3b 08 00 00 24 66 60 c1 11 6b ca fc 2f 45
##  [9289] 79 05 0c 97 87 df 7c f4 e1 b7 37 17 fb 9c 00 00 40 7f f0 c1 14 8e c0 81
##  [9313] 3d a7 3f be 49 79 4a ed 1c 00 6b 60 cf cc 27 c6 5f 59 3b 04 00 00 24 0a
##  [9337] 2c 58 b5 26 29 a5 69 bd ba 76 0e 80 b5 52 52 7e 7a 76 cf c4 b3 6b e7 00
##  [9361] 00 00 05 16 ac d2 ec ee f1 3d 49 1e 55 3b 07 c0 5a 6a 9a fc ee 81 5d e3
##  [9385] 8f a9 9d 03 00 80 d1 a6 c0 82 55 68 a6 33 96 d2 7a 55 ed 1c 00 eb 60 53
##  [9409] 69 95 2b 67 a6 27 bf be 76 10 00 00 46 97 02 0b 56 61 66 7e e2 e9 49 33
##  [9433] 5e 3b 07 c0 3a b9 6f 39 dc be f2 b3 e7 9f bd b5 76 10 00 00 46 93 02 0b
##  [9457] 56 a8 b9 38 ad d2 e4 e5 b5 73 00 ac a7 a6 e4 61 b7 6e b8 e5 cd 8d bb ae
##  [9481] 02 00 50 81 02 0b 56 68 e6 ea 89 5d 49 ce a8 9d 03 a0 82 ef 9c 9d 1a ff
##  [9505] 89 da 21 00 00 18 3d 0a 2c 58 81 26 29 ad 76 7e a6 76 0e 80 7a ca 2f ce
##  [9529] 4c 8d 7f 7b ed 14 00 00 8c 16 05 16 ac c0 ec 9e 89 a7 36 25 0f ab 9d 03
##  [9553] a0 a2 92 94 77 5c 3f 75 c6 44 ed 20 00 00 8c 0e 05 16 2c d3 1d fb be 98
##  [9577] 7d 05 90 1c 3b 9f f9 f7 fc db d3 be e1 84 da 41 00 00 18 0d 0a 2c 58 a6
##  [9601] 83 53 13 4f 48 93 47 d7 ce 01 d0 27 4e 3d bc 71 fe 8f 9a e9 8c d5 0e 02
##  [9625] 00 c0 f0 53 60 c1 32 34 49 69 92 9f ad 9d 03 a0 9f 94 d2 3c 79 66 6e fc
##  [9649] d5 b5 73 00 00 30 fc dc 0a 1b 96 61 ff ee f1 73 5b a5 7c b0 76 0e 80 7e
##  [9673] 54 9a f2 ec 6d 57 5e f3 07 b5 73 00 00 30 bc cc c0 82 65 68 15 7b 5f 01
##  [9697] 2c a6 29 cd ef ce ec 99 7c 64 ed 1c 00 00 0c 2f 33 b0 60 09 07 76 8d 3f
##  [9721] a6 b4 ca df d7 ce 01 d0 cf 9a e4 b3 1b 4a 1e 71 f2 e5 fb fe a3 76 16 00
##  [9745] 00 86 8f 19 58 b0 84 56 cb ec 2b 80 a5 94 e4 81 73 4d 2e df 7f de 8e a3
##  [9769] 6a 67 01 00 60 f8 28 b0 a0 8b 99 3d 93 8f 6c 52 9e 52 3b 07 c0 20 28 c9
##  [9793] 39 ad cd 1b 7f bb 31 c3 1b 00 80 1e 53 60 41 17 4d 33 ff f2 da 19 00 06
##  [9817] cc f3 67 77 4f 7c 7f ed 10 00 00 0c 17 7f 21 85 45 cc 4e 4d 3e b4 49 fb
##  [9841] e3 b5 73 00 0c a0 c3 4d bb 39 77 c7 7b ae fd 87 da 41 00 00 18 0e 66 60
##  [9865] c1 22 9a 98 7d 05 b0 4a 1b 4b ab 5c f6 e9 0b 4e 7f 60 ed 20 00 00 0c 07
##  [9889] 33 b0 60 01 fb 77 4d ee 6c b5 da 9f 8c 7f 23 00 ab 57 f2 0f b7 8d b5 1e
##  [9913] 3f 79 e9 de 43 b5 a3 00 00 30 d8 cc c0 82 05 b4 4a f3 3f a3 bc 02 38 32
##  [9937] 4d 1e b3 79 be fd bf 6a c7 00 00 60 f0 f9 05 1d ee 66 66 6a fc b4 a4 ec
##  [9961] 8b 82 17 a0 47 ca 0b b6 5f 71 cd 5b 6a a7 00 00 60 70 f9 05 1d ee a6 a4
##  [9985] fc 74 fc db 00 e8 a1 e6 0d 33 7b 26 1f 59 3b 05 00 00 83 cb 0c 2c b8 8b
## [10009] 4f 5d 78 da b6 b1 f6 d8 a7 92 8c d5 ce 02 30 5c 9a cf 34 d9 74 f6 8e 2b
## [10033] ae fe 42 ed 24 00 00 0c 1e b3 4c e0 2e 36 34 1b 7e 22 ca 2b 80 35 50 4e
## [10057] 4a 0e 5f 72 d5 45 67 6f ac 9d 04 00 80 c1 a3 c0 82 3b 1c 98 3a f3 c4 a6
## [10081] 69 be a7 76 0e 80 61 55 92 c7 9d f0 85 5b 7e b9 76 0e 00 00 06 8f 02 0b
## [10105] ee 50 ca dc 8b 92 1c 55 3b 07 c0 50 2b f9 91 99 3d e3 df 55 3b 06 00 00
## [10129] 83 c5 1e 58 90 e4 86 e9 93 b6 1c 9a 3b f6 df 92 dc b7 76 16 80 11 70 6b
## [10153] ab 69 3d e6 94 2b f7 7e bc 76 10 00 00 06 83 19 58 90 e4 f0 dc 31 df 1d
## [10177] e5 15 c0 7a d9 d2 2e ed 2b f6 4d 8d df a7 76 10 00 00 06 83 02 8b 91 d7
## [10201] 5c 9c 56 3b e5 a5 b5 73 00 8c 98 93 37 a5 bc a3 99 76 e3 0c 00 00 96 a6
## [10225] c0 62 e4 cd fe eb c4 53 4a 32 51 3b 07 c0 08 7a d2 c1 c3 13 af a8 1d 02
## [10249] 00 80 fe a7 c0 82 76 5e 56 3b 02 c0 a8 6a 4a 7e 66 66 6a fc db 6b e7 00
## [10273] 00 a0 bf d9 c4 9d 91 76 70 f7 e4 59 ed d2 fe 58 ed 1c 00 23 ee bf e7 5b
## [10297] f3 67 9f 76 d9 a7 66 6b 07 01 00 a0 3f 99 81 c5 48 9b 4f db ec 2b 80 fa
## [10321] 8e 1f 6b 8f 5d 76 c3 f4 49 5b 6a 07 01 00 a0 3f 29 b0 18 59 d7 4d 9f f6
## [10345] a0 52 f2 cc da 39 00 48 92 9c 75 68 fe 5e bf dd 98 1d 0e 00 c0 02 14 58
## [10369] 8c ac 0d 73 1b 7e 30 c9 86 da 39 00 b8 43 d3 3c 77 66 f7 f8 f7 d6 8e 01
## [10393] 00 40 ff f1 57 4e 46 d2 de e9 c9 63 36 cf b5 6f 48 72 7c ed 2c 00 7c 8d
## [10417] 43 29 ad c7 6e bf 7c ef 47 6b 07 01 00 a0 7f 98 81 c5 48 da 72 78 fe b9
## [10441] 51 5e 01 f4 a3 4d 69 da 97 5d 77 fe 69 5f 57 3b 08 00 00 fd 43 81 c5 c8
## [10465] 69 a6 33 96 d2 7a 69 ed 1c 00 2c ea c1 1b 36 8c bd a3 99 ce 58 ed 20 00
## [10489] 00 f4 07 05 16 23 67 66 7e e2 82 26 cd b6 da 39 00 e8 ea 89 07 e7 27 2e
## [10513] ae 1d 02 00 80 fe a0 c0 62 e4 94 a6 fc 68 ed 0c 00 2c ad 69 f2 f2 d9 5d
## [10537] e3 e7 d7 ce 01 00 40 7d 36 71 67 a4 1c dc b5 f3 d1 ed 56 f3 e1 da 39 00
## [10561] 58 b6 2f b5 9b b1 b3 4f bd f2 93 33 b5 83 00 00 50 8f 19 58 8c 94 f9 56
## [10585] f3 b2 da 19 00 58 91 e3 5a 65 ee b2 cf 9e 7f f6 d6 da 41 00 00 a8 47 81
## [10609] c5 c8 38 b8 7b fc e4 92 3c bd 76 0e 00 56 aa 3c f4 b6 0d b7 fc 76 63 e6
## [10633] 38 00 c0 c8 52 60 31 32 da a5 fc 50 bc e7 01 06 52 93 7c cf c1 dd 13 cf
## [10657] af 9d 03 00 80 3a fc 25 93 91 30 33 bd ed b8 cc 1d 75 43 92 63 6b 67 01
## [10681] 60 d5 6e 6f 35 ad 73 4e b9 72 ef c7 6b 07 01 00 60 7d 99 8d c2 48 28 f3
## [10705] 9b 5e 18 e5 15 c0 a0 3b aa 5d da ef 9e 99 de 76 5c ed 20 00 00 ac 2f 05
## [10729] 16 43 ef aa 8b ce de d8 34 e5 87 6b e7 00 a0 27 b6 67 ee a8 df b3 1f 16
## [10753] 00 c0 68 51 60 31 f4 8e ff e2 2d 17 26 79 70 ed 1c 00 f4 cc d4 ec d4 f8
## [10777] 4b 6b 87 00 00 60 fd 28 b0 18 7a ad e4 87 6a 67 00 a0 d7 ca 2f cf 4c 4d
## [10801] 3e b6 76 0a 00 00 d6 87 e9 f7 0c b5 83 bb 27 cf 6a 97 f6 c7 6a e7 00 a0
## [10825] f7 9a e4 b3 c9 c6 87 ed b8 e2 ea 2f d4 ce 02 00 c0 da 32 03 8b a1 d6 b4
## [10849] 9a 1f a8 9d 01 80 b5 51 92 07 96 1c 7e 47 33 9d b1 da 59 00 00 58 5b 0a
## [10873] 2c 86 d6 cc f4 b6 e3 9a a6 f9 ee da 39 00 58 53 df 76 f0 f0 c4 2b 6a 87
## [10897] 00 00 60 6d 29 b0 18 5a e5 f0 a6 67 27 d9 5a 3b 07 00 6b ab 29 f9 99 03
## [10921] 53 93 4f a9 9d 03 00 80 b5 a3 c0 62 28 35 49 69 97 f2 a2 da 39 00 58 1f
## [10945] ad b4 ff 60 66 7a f2 eb 6b e7 00 00 60 6d 28 b0 18 4a d7 4f 8d 7f 4b 49
## [10969] 26 6a e7 00 60 7d 34 c9 7d 32 df be 64 ef f4 e4 a6 da 59 00 00 e8 3d 05
## [10993] 16 43 69 3e 66 5f 01 8c 9c 26 8f de 32 37 ff 2b b5 63 00 00 d0 7b a5 76
## [11017] 00 e8 b5 83 d3 93 f7 6f cf b5 6f 48 b2 a1 76 16 00 d6 5f 69 ca 77 6c bb
## [11041] f2 9a 4b 6a e7 00 00 a0 77 cc c0 62 e8 cc 1f 6e bf 30 ca 2b 80 91 d5 94
## [11065] e6 77 67 2e 9c 38 b5 76 0e 00 00 7a 47 81 c5 50 f9 e0 b9 e7 6e 28 25 17
## [11089] d5 ce 01 40 55 c7 a6 9d 4b 0e 3e f7 e4 cd b5 83 00 00 d0 1b 0a 2c 86 ca
## [11113] 49 27 7c e1 69 49 1e 5c 3b 07 00 d5 9d d5 fe d2 e6 ff 55 3b 04 00 00 bd
## [11137] a1 c0 62 a8 b4 d2 d8 bc 1d 80 3b 94 ef 9b d9 3d fe cc da 29 00 00 38 72
## [11161] 36 71 67 68 1c b8 60 72 47 19 6b ef af 9d 03 80 3e 52 72 73 ab cc 3f e2
## [11185] 94 cb 3e 75 6d ed 28 00 00 ac 9e 19 58 0c 8f 56 f3 fd b5 23 00 d0 67 9a
## [11209] 1c dd 6e 8f 5d fa d9 f3 cf de 5a 3b 0a 00 00 ab a7 c0 62 28 dc 30 7d d2
## [11233] 96 52 9a e7 d5 ce 01 40 5f 3a e3 d6 0d b7 fc 66 ed 10 00 00 ac 9e 02 8b
## [11257] a1 70 fb dc 31 cf 48 72 ef da 39 00 e8 5b cf 3f 30 35 f1 9c da 21 00 00
## [11281] 58 1d 05 16 43 a1 94 f2 03 b5 33 00 d0 df 4a f2 86 fd bb 26 77 d6 ce 01
## [11305] 00 c0 ca 29 b0 18 78 fb f7 4c 3c 3c 4d 1e 5d 3b 07 00 7d 6f 6b ab d5 be
## [11329] f4 73 4f 3a f3 e8 da 41 00 00 58 19 05 16 03 af 34 31 fb 0a 80 e5 da 79
## [11353] cb d1 87 7f bb 71 27 66 00 80 81 e2 c3 1b 03 ed e0 ee b3 8e 6f 97 db 3f
## [11377] 9b 64 4b ed 2c 00 0c 92 f2 82 ed 57 5c f3 96 da 29 00 00 58 1e 33 b0 18
## [11401] 68 f3 e5 d0 73 a2 bc 02 60 c5 9a df 9a d9 35 fe 0d b5 53 00 00 b0 3c 0a
## [11425] 2c 06 56 93 94 92 e6 45 b5 73 00 30 90 36 a7 55 2e bd f6 82 d3 8f ad 1d
## [11449] 04 00 80 a5 29 b0 18 58 33 53 93 8f 4f 72 7a ed 1c 00 0c ac d3 37 8e 95
## [11473] 37 d8 0f 0b 00 a0 ff 29 b0 18 58 25 f3 36 6f 07 e0 08 95 67 1d dc 3d f1
## [11497] fc da 29 00 00 e8 ce 5f 1c 19 48 9f be e0 f4 07 1e 1e 6b 7d 3a c9 58 ed
## [11521] 2c 00 0c bc 5b 9b e4 51 3b ae d8 f7 c9 da 41 00 00 58 98 19 58 0c a4 43
## [11545] 1b 5a 2f 88 f2 0a 80 de d8 92 e4 92 cf 3d e9 cc a3 6b 07 01 00 60 61 0a
## [11569] 2c 06 4e 73 71 5a 25 79 5e ed 1c 00 0c 8f 92 4c dc 72 f4 e1 df aa 9d 03
## [11593] 00 80 85 29 b0 18 38 33 57 9f fe b8 34 39 a5 76 0e 00 86 4b 93 7c cf 81
## [11617] a9 f1 ef a9 9d 03 00 80 7b 52 60 31 70 4a d3 b2 d9 2e 00 6b a2 a4 fc f6
## [11641] fe 5d 93 3b 6b e7 00 00 e0 6b d9 c4 9d 81 32 33 bd ed b8 cc 1d f5 b9 24
## [11665] 9b 6b 67 01 60 68 ed dd 32 b7 f5 51 0f 7c df 3f df 52 3b 08 00 00 1d 66
## [11689] 60 31 50 9a c3 9b be 23 ca 2b 00 d6 d6 e4 ad 1b 6f 79 5d ed 10 00 00 7c
## [11713] 95 02 8b 81 52 5a c5 f2 41 00 d6 5e 93 17 cc 4c ed 7c 56 ed 18 00 00 74
## [11737] 58 42 c8 c0 d8 7f e1 ce c9 56 bb f9 64 ed 1c 00 8c 88 92 9b cb 5c fb ec
## [11761] 6d ef bd ee ba da 51 00 00 46 9d 19 58 0c 8c 32 9f e7 d5 ce 00 c0 08 69
## [11785] 72 74 33 d6 ba e4 86 e9 93 b6 d4 8e 02 00 30 ea 14 58 0c 84 ab 2e 3a 7b
## [11809] 63 29 ed 67 d7 ce 01 c0 c8 39 f3 d0 dc 31 bf 5e 3b 04 00 c0 a8 53 60 31
## [11833] 10 8e fb e2 ad 4f 4d ca 89 b5 73 00 30 8a ca f7 1d d8 33 f1 1d b5 53 00
## [11857] 00 8c 32 05 16 03 a1 a4 6d f3 76 00 aa 29 4d de 74 e0 82 c9 1d b5 73 00
## [11881] 00 8c 2a 9b b8 d3 f7 0e 4e 4f de bf 3d d7 fe 4c 92 b1 da 59 00 18 69 ff
## [11905] d2 be ed f0 63 4e fd c0 81 db 6b 07 01 00 18 35 66 60 d1 f7 e6 e7 da cf
## [11929] 8e f2 0a 80 fa 1e de da bc f1 35 b5 43 00 00 8c 22 05 16 7d ad 49 4a 49
## [11953] b1 7c 10 80 7e f1 c3 fb a7 76 ee aa 1d 02 00 60 d4 58 42 48 5f 3b b8 67
## [11977] e2 9c 76 93 7f ac 9d 03 00 ee e2 a6 6c 68 9d b5 fd d2 bd 9f ae 1d 04 00
## [12001] 60 54 98 81 45 5f 9b 6f 62 f6 15 00 fd e6 84 cc b7 ff e8 aa 8b ce de 58
## [12025] 3b 08 00 c0 a8 50 60 d1 b7 3e f7 a4 33 8f 2e c9 77 d6 ce 01 00 f7 d0 e4
## [12049] 31 27 dc 78 eb cf d5 8e 01 00 30 2a 14 58 f4 ad af 1c 7d f8 c2 24 c7 d6
## [12073] ce 01 00 0b 6a 9a 9f 9a dd 3d fe a4 da 31 00 00 46 81 02 8b 7e 66 f9 20
## [12097] 00 7d ad 29 f9 fd eb f7 4c 3c a0 76 0e 00 80 61 a7 c0 a2 2f ed df 7d c6
## [12121] f6 92 3c ae 76 0e 00 e8 ae 9c 38 df 34 7f d0 4c 67 ac 76 12 00 80 61 a6
## [12145] c0 a2 2f 8d b5 e6 9f 5b 3b 03 00 2c 4f f9 d6 99 c3 13 3f 55 3b 05 00 c0
## [12169] 30 2b b5 03 c0 dd 35 d3 19 9b 9d 1b bf 3e 29 27 d5 ce 02 00 cb d4 2e 25
## [12193] 8f df 76 f9 be bf ad 1d 04 00 60 18 99 81 45 df b9 be 3d fe 04 e5 15 00
## [12217] 03 a6 d5 34 79 c7 75 e7 9f f6 75 b5 83 00 00 0c 23 05 16 7d 67 be 5d 6c
## [12241] de 0e c0 20 7a d0 86 0d 63 6f 6d cc 70 07 00 e8 39 05 16 7d e5 86 e9 c9
## [12265] 7b 97 64 77 ed 1c 00 b0 4a 4f 9b 9d 1a 7f 69 ed 10 00 00 c3 46 81 45 5f
## [12289] 39 7c 78 fe bb 92 6c aa 9d 03 00 56 af bc 66 76 f7 e4 a3 6a a7 00 00 18
## [12313] 26 0a 2c fa 8c e5 83 00 0c bc 0d 4d ab fd ce 99 e9 6d c7 d5 0e 02 00 30
## [12337] 2c 14 58 f4 8d 83 bb 27 cf 6a 4a 1e 56 3b 07 00 1c b1 26 a7 34 73 47 bd
## [12361] d1 7e 58 00 00 bd a1 c0 a2 6f b4 4b fb 39 b5 33 00 40 af 94 e4 19 07 a7
## [12385] c6 5f 58 3b 07 00 c0 30 f0 57 41 fa 42 33 9d b1 d9 b9 89 1b 92 3c a0 76
## [12409] 16 00 e8 a1 db 9a e4 91 3b ae d8 f7 c9 da 41 00 00 06 99 19 58 f4 85 d9
## [12433] f6 c4 e3 a3 bc 02 60 f8 6c 2e c9 bb 3e 7b fe d9 5b 6b 07 01 00 18 64 0a
## [12457] 2c fa 43 53 9e 55 3b 02 00 ac 91 9d b7 6c b8 e5 37 6a 87 00 00 18 64 96
## [12481] 10 52 dd 0d d3 27 6d 39 34 77 ec e7 93 1c 5b 3b 0b 00 ac 95 26 79 e6 8e
## [12505] 2b f6 bd b3 76 0e 00 80 41 64 06 16 d5 1d 3e 7c af f3 a3 bc 02 60 c8 95
## [12529] e4 8d fb 77 9f b1 bd 76 0e 00 80 41 a4 c0 a2 ba 76 69 5b 3e 08 c0 28 38
## [12553] b6 95 f9 77 ee 9d 9e dc 54 3b 08 00 c0 a0 51 60 51 d5 0d d3 93 f7 2e 29
## [12577] e7 d5 ce 01 00 eb a2 e4 11 9b e7 9a 5f aa 1d 03 00 60 d0 28 b0 a8 ea f6
## [12601] f9 66 3a c9 c6 da 39 00 60 fd 34 2f 9b d9 33 f1 b4 da 29 00 00 06 89 02
## [12625] 8b aa 4a d3 58 3e 08 c0 c8 29 4d de 76 dd f4 69 0f aa 9d 03 00 60 50 28
## [12649] b0 a8 66 76 cf c4 43 92 7c 73 ed 1c 00 b0 de 9a e4 3e 63 73 63 7f d8 4c
## [12673] 67 ac 76 16 00 80 41 a0 c0 a2 9a a6 69 9e 59 3b 03 00 d4 52 92 c7 cd 1e
## [12697] 1e 7f 79 ed 1c 00 00 83 40 81 45 45 c5 f2 41 00 46 5b 29 3f 7b 70 6a fc
## [12721] 71 b5 63 00 00 f4 bb 52 3b 00 a3 69 f6 c2 d3 cf 6c da ad 4f d4 ce 01 00
## [12745] 7d e0 df e7 e6 e6 cf 3a fd 7d 9f ba b1 76 10 00 80 7e 65 06 16 55 34 4d
## [12769] cb ec 2b 00 e8 78 d0 86 0d 63 bf d7 f8 c3 22 00 c0 a2 14 58 ac bb e6 e2
## [12793] b4 d2 c4 fe 57 00 f0 55 df 3e bb 7b e2 87 6a 87 00 00 e8 57 0a 2c d6 dd
## [12817] f5 9f 18 ff e6 24 0f ae 9d 03 00 fa 4a c9 af 1c d8 3d 7e 76 ed 18 00 00
## [12841] fd 48 81 c5 ba 9b b7 79 3b 00 2c 64 63 69 95 77 5e 7b c1 e9 c7 d6 0e 02
## [12865] 00 d0 6f 14 58 ac ab fd e7 ed 38 aa 24 d3 b5 73 00 40 5f 6a b2 63 e3 58
## [12889] 79 83 fd b0 00 00 be 96 02 8b 75 55 8e da 78 5e 92 e3 6b e7 00 80 fe 55
## [12913] 9e 35 33 35 fe 9c da 29 00 00 fa 89 02 8b 75 55 4a 63 f9 20 00 2c a1 a4
## [12937] fc f6 ec 05 a7 9f 5e 3b 07 00 40 bf 30 3d 9d 75 33 33 bd ed b8 cc 1d f5
## [12961] f9 24 47 d5 ce 02 00 fd af f9 44 eb b8 db ce 39 e5 ad d7 df 56 3b 09 00
## [12985] 40 6d 66 60 b1 7e e6 36 ef 89 f2 0a 00 96 a9 3c 74 fe cb 5b 7f a5 76 0a
## [13009] 00 80 7e a0 c0 62 1d 59 3e 08 00 2b 51 9a e6 07 0f ec 9e d8 5d 3b 07 00
## [13033] 40 6d 96 10 b2 2e ae 9b 3e ed 41 1b e6 c6 6e 88 f7 1c 00 ac d4 4d d9 d0
## [13057] 3a 6b fb a5 7b 3f 5d 3b 08 00 40 2d 66 60 b1 2e 36 ce b7 be 33 ca 2b 00
## [13081] 58 8d 13 32 d7 bc e3 83 e7 9e bb a1 76 10 00 80 5a 14 58 ac 8f 76 b1 7c
## [13105] 10 00 56 ad f9 a6 07 9f f0 f9 8b 6b a7 00 00 a8 c5 8c 18 d6 dc f5 53 67
## [13129] 4c cc 67 fe 9a da 39 00 60 c0 35 4d 5a 4f d8 71 c5 de bf ae 1d 04 00 60
## [13153] bd 99 81 c5 9a 9b cb bc d9 57 00 70 e4 4a 49 fb 0f 0f 4c 9d 79 62 ed 20
## [13177] 00 00 eb 4d 81 c5 9a 6a 92 52 4a be ab 76 0e 00 18 12 f7 2f 39 fc d6 e6
## [13201] 62 9f e1 00 80 d1 e2 c3 0f 6b ea c0 d4 f8 39 69 72 4a ed 1c 00 30 44 ce
## [13225] 3b 78 f5 f8 4b 6b 87 00 00 58 4f 0a 2c d6 54 2b e5 3b 6a 67 00 80 61 d3
## [13249] 34 e5 d5 33 7b 26 1f 59 3b 07 00 c0 7a 51 60 b1 66 ee 58 de f0 f4 da 39
## [13273] 00 60 08 6d 28 4d f3 ce 99 e9 6d c7 d5 0e 02 00 b0 1e 14 58 ac 99 99 8f
## [13297] 8d 9f 93 e4 41 b5 73 00 c0 30 6a d2 6c 2b 87 8f fa 9d c6 5d a5 01 80 11
## [13321] a0 c0 62 cd 94 52 a6 6b 67 00 80 61 d6 94 7c c7 c1 dd 13 cf af 9d 03 00
## [13345] 60 ad 29 b0 58 13 cd c5 69 a5 34 96 0f 02 c0 1a 6b 4a 7e 73 ff ae c9 9d
## [13369] b5 73 00 00 ac 25 05 16 6b e2 c0 27 c6 1f 9d 94 93 6a e7 00 80 11 b0 a5
## [13393] d5 6a bf eb 86 e9 93 b6 d4 0e 02 00 b0 56 14 58 ac 89 56 53 9e 51 3b 03
## [13417] 00 8c 90 33 0e cd 1d f3 eb b5 43 00 00 ac 15 05 16 3d 67 f9 20 00 d4 50
## [13441] be 6f 76 6a a7 fd 27 01 80 a1 a4 c0 a2 e7 2c 1f 04 80 3a 9a 34 6f ba fe
## [13465] fc d3 4f a9 9d 03 00 a0 d7 14 58 f4 5c 2b ee 3e 08 00 95 1c 37 bf b1 f5
## [13489] 47 57 5d 74 f6 c6 da 41 00 00 7a 49 81 45 4f 35 17 a7 95 c4 f2 41 00 a8
## [13513] a5 c9 a3 4f b8 f1 96 57 d5 8e 01 00 d0 4b 0a 2c 7a ea fa ab 27 1e 95 e4
## [13537] c1 b5 73 00 c0 48 6b f2 3f 66 77 8f 3f a9 76 0c 00 80 5e 51 60 d1 53 ed
## [13561] c6 f2 41 00 e8 07 4d c9 ef 1f 9c 9e bc 7f ed 1c 00 00 bd a0 c0 a2 67 3a
## [13585] cb 07 1b 05 16 00 f4 85 72 62 7b ae fd 07 77 2c ef 07 00 18 68 3e d0 d0
## [13609] 33 96 0f 02 40 df f9 b6 83 57 ef fc c9 da 21 00 00 8e 94 02 8b 9e b1 7c
## [13633] 10 00 fa 4f d3 34 3f 7f 60 cf e9 df 54 3b 07 00 c0 91 50 60 d1 13 4d 52
## [13657] d2 34 ee 3e 08 00 fd 67 ac 34 ad 3f ba 61 7a f2 de b5 83 00 00 ac 96 02
## [13681] 8b 9e b8 7e d7 ce 47 a5 e4 eb 6b e7 00 00 16 f4 e0 43 f3 ed 37 37 49 a9
## [13705] 1d 04 00 60 35 14 58 f4 c4 7c 89 e5 83 00 d0 cf 9a ec 3e b8 7b fc c5 b5
## [13729] 63 00 00 ac 86 02 8b 23 d6 24 a5 b8 fb 20 00 f4 bd a6 94 5f dd 7f e1 ce
## [13753] 87 d5 ce 01 00 b0 52 0a 2c 8e 98 e5 83 00 30 30 36 b5 da cd bb ae bd e0
## [13777] f4 63 6b 07 01 00 58 09 05 16 47 cc f2 41 00 18 28 a7 6e 68 b5 7e ab 76
## [13801] 08 00 80 95 50 60 71 44 9a a4 94 e2 ee 83 00 30 48 4a c9 b3 0f 4c 4d 3c
## [13825] a7 76 0e 00 80 e5 72 27 1a 8e c8 ec ee c9 47 35 a5 fd 91 da 39 00 80 15
## [13849] 2a b9 b9 cc b5 cf de f6 de eb ae ab 1d 05 00 60 29 66 60 71 44 da 65 de
## [13873] f2 41 00 18 44 4d 8e 6e c6 ca bb 0e 3e f7 e4 cd b5 a3 00 00 2c 45 81 c5
## [13897] aa 75 ee 3e 58 14 58 00 30 b0 ca 43 db 5f da f2 da da 29 00 00 96 a2 c0
## [13921] 62 d5 66 f7 4c 3e 22 c9 43 6a e7 00 00 8e c8 8b 67 a6 c6 f7 d4 0e 01 00
## [13945] d0 8d 02 8b 55 6b 9a f9 67 d4 ce 00 00 f4 42 79 cb c1 dd e3 27 d7 4e 01
## [13969] 00 b0 18 05 16 ab 62 f9 20 00 0c 95 e3 da ad f2 ce ab 2e 3a 7b 63 ed 20
## [13993] 00 00 0b 51 60 b1 2a 96 0f 02 c0 90 69 f2 e8 7b 7f f1 96 5f a8 1d 03 00
## [14017] 60 21 0a 2c 56 a5 34 6d b3 af 00 60 c8 34 c9 8f 1f d8 bd f3 bc da 39 00
## [14041] 00 ee 4e 81 c5 8a 35 49 69 12 05 16 00 0c a1 52 9a b7 7f fa 82 d3 1f 58
## [14065] 3b 07 00 c0 5d 29 b0 58 b1 99 dd e3 0f 4f 72 72 ed 1c 00 c0 9a f8 ba b9
## [14089] b1 d6 1f 36 d3 19 ab 1d 04 00 e0 4e 0a 2c 56 ae 94 a9 da 11 00 80 b5 d3
## [14113] 24 e7 ce 1e 1e 7f 79 ed 1c 00 00 77 52 60 b1 62 25 d9 5d 3b 03 00 b0 c6
## [14137] 4a f9 d9 fd bb c7 cf ad 1d 03 00 20 49 4a ed 00 0c 96 03 17 4c ee 28 63
## [14161] ed fd b5 73 00 00 eb e2 3f da 1b 0e 3f f4 d4 4b 0f 7c b1 76 10 00 60 b4
## [14185] 99 81 c5 8a b4 36 cc ef aa 9d 01 00 58 37 0f 68 cd 6d 7c 5b 73 b1 cf 8c
## [14209] 00 40 5d 3e 8c b0 22 4d 53 2c 1f 04 80 d1 72 de cc 27 26 7e b4 76 08 00
## [14233] 60 b4 59 42 c8 b2 1d 98 3a f3 c4 92 c3 9f 8b f7 0d 00 8c 9a b9 56 c9 37
## [14257] 9f 72 f9 be 0f d7 0e 02 00 8c 26 33 b0 58 b6 56 73 f8 fc 28 af 00 60 14
## [14281] 6d 68 37 79 e7 bf 3d ed 1b 4e a8 1d 04 00 18 4d 0a 2c 96 ad 29 ee 3e 08
## [14305] 00 23 ec 21 73 9b e6 7e b7 f1 c7 2c 00 a0 02 05 16 cb b2 77 7a f2 98 24
## [14329] 4f ac 9d 03 00 a8 6a cf c1 dd e3 2f ae 1d 02 00 18 3d 0a 2c 96 65 f3 e1
## [14353] f9 27 27 39 aa 76 0e 00 a0 ae a6 94 5f dd bf 67 e2 e1 b5 73 00 00 a3 45
## [14377] 81 c5 f2 14 77 1f 04 00 92 24 9b 5a 4d 2e d9 7f de 8e 7b d5 0e 02 00 8c
## [14401] 0e 05 16 4b ba ea a2 b3 37 26 f9 f6 da 39 00 80 be b1 bd 6c de f8 26 fb
## [14425] 61 01 00 eb 45 81 c5 92 4e f8 c2 ad df 92 e4 f8 da 39 00 80 fe 51 92 67
## [14449] cc ec d9 79 51 ed 1c 00 c0 68 50 60 b1 a4 a6 e5 ee 83 00 c0 3d 95 a6 f9
## [14473] 8d d9 a9 c9 87 d6 ce 01 00 0c 3f 05 16 5d 35 49 29 4d 5b 81 05 00 2c e4
## [14497] a8 26 ed 4b ae bd e0 f4 63 6b 07 01 00 86 9b 02 8b ae 66 76 8f 3f 3c 29
## [14521] 27 d5 ce 01 00 f4 ad d3 36 8e 95 37 d8 0f 0b 00 58 4b 0a 2c ba 6a b5 dc
## [14545] 7d 10 00 58 4a 79 d6 ec d4 ce e7 d5 4e 01 00 0c 2f 05 16 5d 35 8d fd af
## [14569] 00 80 e5 68 5e bf ff c2 9d 93 b5 53 00 00 c3 49 81 c5 a2 0e 5c 30 b9 23
## [14593] c9 19 b5 73 00 00 03 61 4b 69 37 97 7e ee 49 67 1e 5d 3b 08 00 30 7c 14
## [14617] 58 2c aa b5 61 7e 57 ed 0c 00 c0 e0 28 c9 c4 cd 47 1f fa cd da 39 00 80
## [14641] e1 a3 c0 62 51 4d 63 ff 2b 00 60 a5 ca f3 66 f7 4c 3c bb 76 0a 00 60 b8
## [14665] b8 5b 0c 0b 3a 30 75 e6 89 25 87 3f 17 ef 11 00 60 e5 6e 19 cb d8 23 4e
## [14689] be e2 93 fb 6a 07 01 00 86 83 19 58 2c a8 d5 1c 3e 3f ca 2b 00 60 75 b6
## [14713] ce 67 fe 92 cf 9e 7f f6 d6 da 41 00 80 e1 a0 c0 62 41 4d 71 f7 41 00 e0
## [14737] 88 9c 71 eb c6 5b 5e 57 3b 04 f0 ff da bb fb 37 cb eb bb be e3 ef cf f7
## [14761] cc ec ce 72 13 d8 90 04 42 80 c0 b2 b0 73 66 16 82 ae 91 44 2b 0c 09 21
## [14785] 25 96 dd 39 b3 8e bd 8c 36 12 ad 68 b4 b1 b6 f6 c6 5e f6 d2 bd 8c 57 7b
## [14809] 5d d6 aa 6d 6e 6c a3 09 49 6c 9a c2 72 27 98 50 73 53 48 8c d6 1b a8 21
## [14833] 30 dc ed 9c 19 04 cd 05 81 6c 6e 88 bb b0 33 e7 fb e9 0f 26 91 24 b0 ec
## [14857] cd cc 7c ce 39 df c7 e3 2f 78 5e fb c3 99 d9 d7 7c bf ef 03 30 1c 3c 61
## [14881] c3 b7 99 9b 9d 3c 6e 6c b9 7e 22 22 d6 97 6e 01 00 06 5b 4a f1 a6 4d 37
## [14905] dc f7 7b a5 3b 00 80 c1 e6 09 2c be cd d8 52 ef f5 61 bc 02 00 56 40 ce
## [14929] f1 df 1e ea 6c 6d 97 ee 00 00 06 9b 01 8b 6f 97 7c fb 20 00 b0 62 8e e9
## [14953] 45 6f f7 a3 97 9d 7f 6c e9 10 00 60 70 19 b0 f8 26 77 5c b5 6d 34 22 fe
## [14977] 51 e9 0e 00 60 a8 4c fe ed b1 07 de 5e 3a 02 00 18 5c 06 2c be c9 c6 cf
## [15001] ef bf 28 22 4e 2c dd 01 00 0c 9b f4 e6 f9 ce f8 8f 96 ae 00 00 06 93 01
## [15025] 8b 6f 92 2b df 3e 08 00 ac 8e 14 e9 b7 f7 ec 9c 98 2c dd 01 00 0c 1e 03
## [15049] 16 df 90 23 52 ca b5 01 0b 00 58 2d 1b 52 9d dd c3 02 00 0e 9b 01 8b 6f
## [15073] e8 4e 8f 7f 67 44 3a ad 74 07 00 30 bc 52 44 7b df b1 4b ef ca 11 a9 74
## [15097] 0b 00 30 38 0c 58 7c 43 55 f9 f6 41 00 60 f5 e5 88 37 2d 74 26 ae 2c dd
## [15121] 01 00 0c 0e 03 16 df 90 b3 fb 57 00 c0 5a c9 ef 9c ef b4 b7 96 ae 00 00
## [15145] 06 83 01 8b 88 88 78 70 e7 b9 9b 22 c2 2f 91 00 c0 5a d9 90 22 ed 9e 9b
## [15169] 9d 3c ae 74 08 00 d0 ff 0c 58 44 44 44 55 8f fc a3 d2 0d 00 40 d3 e4 f1
## [15193] b1 e5 fa b7 dd c3 02 00 9e 8f 01 8b 88 88 48 91 bf bf 74 03 00 d0 48 3f
## [15217] b2 38 d3 fe f1 d2 11 00 40 7f f3 d7 2e 62 6e 76 f2 b8 b1 e5 fa 0b 11 b1
## [15241] ae 74 0b 00 d0 48 4f 57 b9 7a d5 59 37 cd 7d a6 74 08 00 d0 9f 3c 81 45
## [15265] ac 5f aa 2f 0d e3 15 00 50 ce fa ba aa 77 77 67 37 9d 50 3a 04 00 e8 4f
## [15289] 06 2c 22 55 e1 fe 15 00 50 56 8e cd d1 1b 7b 8f 7b 58 00 c0 b3 31 60 35
## [15313] 5c de 15 55 e4 70 ff 0a 00 28 2f e7 9d 0b d3 ed 9f 2d 9d 01 00 f4 1f 03
## [15337] 56 c3 75 3f 33 fe 1d 11 71 4a e9 0e 00 80 88 88 48 f1 eb 8b 33 ed 57 95
## [15361] ce 00 00 fa 8b 01 ab e1 52 78 7d 10 00 e8 2b 23 75 8e 6b ef eb 8c 9f 54
## [15385] 3a 04 00 e8 1f 06 ac a6 4b 5e 1f 04 00 fa ce e9 eb 22 fd 5e de e5 77 55
## [15409] 00 e0 ef f8 a5 a0 c1 16 67 27 4f 89 48 af 2c dd 01 00 f0 2c 2e 5f fc ec
## [15433] c4 2f 94 8e 00 00 fa 83 01 ab c1 ea e5 7c 79 e9 06 00 80 e7 92 73 7e db
## [15457] fc cc 96 4b 4a 77 00 00 e5 19 b0 1a 2d bb 7f 05 00 f4 b3 2a e5 ea 43 0f
## [15481] cd b4 5f 5a 3a 04 00 28 cb 80 d5 50 73 b3 93 eb 22 e2 b2 d2 1d 00 00 cf
## [15505] e3 e4 e5 1c 1f ba 6d 6a 6a a4 74 08 00 50 8e 01 ab a1 c6 ea fa a2 88 38
## [15529] ae 74 07 00 c0 f3 49 11 17 9f f1 c2 cf ff 4a e9 0e 00 a0 1c 03 56 53 f5
## [15553] 7c fb 20 00 30 40 72 fe 77 dd 9d ed 37 94 ce 00 00 ca 30 60 35 55 15 ee
## [15577] 5f 01 00 83 a5 8e df 5b 98 69 bf bc 74 06 00 b0 f6 0c 58 0d d4 ed 8c 9f
## [15601] 1b 39 36 97 ee 00 00 38 4c 2f cc 39 ef de 73 f9 e6 f5 a5 43 00 80 b5 65
## [15625] c0 6a 26 4f 5f 01 00 03 2a bd b2 5a 3f fa 1b a5 2b 00 80 b5 65 c0 6a 26
## [15649] f7 af 00 80 c1 95 e2 a7 bb 9d 89 1f 2e 9d 01 00 ac 9d 54 3a 80 b5 d5 9d
## [15673] dd 74 42 2c af 7f 22 22 7c 15 35 00 30 c8 f6 e5 88 0b 37 df 78 df 3d a5
## [15697] 43 00 80 d5 e7 09 ac 86 49 bd 0d af 0b e3 15 00 30 f8 8e 49 11 d7 ef b9
## [15721] 7c f3 0b 4a 87 00 00 ab cf 80 d5 34 b9 76 ff 0a 00 18 16 e7 56 63 23 ef
## [15745] c9 de 2a 00 80 a1 67 c0 6a 90 bc 2b aa 1c f1 86 d2 1d 00 00 2b 27 fd c0
## [15769] 42 a7 fd 73 a5 2b 00 80 d5 65 c0 6a 90 87 fe 72 e2 95 11 f1 e2 d2 1d 00
## [15793] 00 2b ec d7 ba 9d c9 7f 50 3a 02 00 58 3d 06 ac 06 c9 ad ec f5 41 00 60
## [15817] 18 8d 44 d4 d7 2e 6c df 7a 72 e9 10 00 60 75 18 b0 9a a4 8e ef 2f 9d 00
## [15841] 00 b0 4a 5e 9a 5b f5 87 6e 9b 9a f2 65 35 00 30 84 0c 58 0d f1 c0 ec b9
## [15865] 2f cb 29 be a3 74 07 00 c0 ea c9 97 9c b1 f1 f3 6f 2b 5d 01 00 ac 3c 03
## [15889] 56 43 8c 2e 57 8e b7 03 00 0d 90 7f 61 61 66 72 7b e9 0a 00 60 65 19 b0
## [15913] 1a a2 8e 70 ff 0a 00 68 84 9c eb 0f ec 99 de 7a 76 e9 0e 00 60 e5 18 b0
## [15937] 1a 60 f1 ca 33 c7 52 a4 4b 4b 77 00 00 ac 91 13 aa b4 7c fd 23 b3 a7 6d
## [15961] 28 1d 02 00 ac 0c 03 56 03 f4 be 7c ec 54 44 1c 53 ba 03 00 60 ed a4 57
## [15985] 1c e8 bd e0 5d 39 22 95 2e 01 00 8e 9e 01 ab 09 52 f6 ed 83 00 40 f3 e4
## [16009] 7c e5 42 67 e2 27 4b 67 00 00 47 cf 80 35 e4 72 44 4a 39 bb 7f 05 00 34
## [16033] 54 fe af 8b 3b 26 2e 2c 5d 01 00 1c 1d 03 d6 90 9b df 31 d9 8e 88 33 4b
## [16057] 77 00 00 14 32 5a 57 f9 fa f9 ce f9 2f 29 1d 02 00 1c 39 03 d6 90 4b a9
## [16081] f6 f4 15 00 d0 74 2f ab 62 e9 9a db a6 a6 46 4a 87 00 00 47 c6 80 35 e4
## [16105] 52 0a f7 af 00 80 c6 cb 11 53 2f df f8 d8 7f 28 dd 01 00 1c 19 df ca 32
## [16129] c4 16 a7 2f 38 b1 4e 4f 3f 11 11 ad d2 2d 00 00 fd 20 a5 6a 76 d3 0d 73
## [16153] d7 95 ee 00 00 0e 8f 27 b0 86 58 ae 96 2e 0d e3 15 00 c0 37 e4 5c 5f fd
## [16177] 50 67 6b bb 74 07 00 70 78 0c 58 43 ac ce f5 eb 4b 37 00 00 f4 99 e3 7a
## [16201] d1 bb 71 cf e5 9b 5f 50 3a 04 00 38 74 06 ac 21 95 23 52 8a fc 0f 4b 77
## [16225] 00 00 f4 a1 2d d5 86 75 ef cd ce 69 00 c0 c0 30 60 0d a9 f9 9d 13 13 11
## [16249] e9 b4 d2 1d 00 00 7d 29 e7 9d dd e9 f6 bf 2e 9d 01 00 1c 1a 03 d6 90 aa
## [16273] 6a af 0f 02 00 1c 4c 4a f1 1f e7 3b 93 af 29 dd 01 00 3c 3f 03 d6 d0 4a
## [16297] 5e 1f 04 00 38 b8 2a 45 7d cd fc 8e ad a7 97 0e 01 00 0e ce 80 35 84 3e
## [16321] 77 c5 b6 63 22 e2 a2 d2 1d 00 00 03 e0 45 a9 ea 5d b7 e7 f2 cd eb 4b 87
## [16345] 00 00 cf cd 80 35 84 f6 b5 f6 5f 1c 11 7e 09 03 00 38 34 df 9d c6 46 df
## [16369] 5e 3a 02 00 78 6e 06 ac 21 54 a5 da eb 83 00 00 87 21 45 fc c4 fc cc c4
## [16393] 4f 96 ee 00 00 9e 9d 01 6b 08 65 f7 af 00 00 0e 5b ca f9 ed f3 3b c6 bf
## [16417] a7 74 07 00 f0 ed 52 e9 00 56 d6 43 57 6c 39 ab 37 52 2d 94 ee 00 00 18
## [16441] 50 8f 8e f6 ea 6d 67 dc fc c0 e7 4a 87 00 00 7f cf 13 58 43 a6 37 d2 7a
## [16465] 7d e9 06 00 80 01 76 ca 72 ab 72 d4 55 ee d1 34 00 00 1d 19 49 44 41 54
## [16489] 1d 00 fa 8c 01 6b c8 a4 c8 5e 1f 04 00 38 0a 39 e2 d5 d5 d8 c8 7f 29 dd
## [16513] 01 00 fc 3d af 10 0e 91 b9 d9 c9 75 63 cb f5 13 11 71 7c e9 16 00 80 41
## [16537] 97 22 5f b5 e9 c6 fb 7f a7 74 07 00 e0 09 ac a1 72 cc 72 ef d5 61 bc 02
## [16561] 00 58 11 39 d2 3b f7 74 c6 5f 5d ba 03 00 30 60 0d 95 3a 2a af 0f 02 00
## [16585] ac 9c d1 2a d2 f5 0f cd b4 5f 5a 3a 04 00 9a ce 80 35 44 52 ce 0e b8 03
## [16609] 00 ac ac 97 f6 22 ae 9b 9b 9d 5c 57 3a 04 00 9a cc 80 35 24 16 67 27 4f
## [16633] c9 29 be a3 74 07 00 c0 d0 c9 f1 3d 63 bd fa b7 4a 67 00 40 93 19 b0 86
## [16657] 44 6f b9 be ac 74 03 00 c0 d0 ca f1 96 85 e9 f6 8f 97 ce 00 80 a6 32 60
## [16681] 0d 89 94 93 d7 07 01 00 56 51 4e f1 ae c5 1d 13 17 96 ee 00 80 26 32 60
## [16705] 0d 81 3c 1b ad 94 dc bf 02 00 58 65 eb 7a 55 be c1 51 77 00 58 7b 06 ac
## [16729] 21 b0 d0 9b fc ce 1c 71 52 e9 0e 00 80 61 97 22 4e ad 73 5c bf e7 f2 cd
## [16753] eb 4b b7 00 40 93 18 b0 86 40 ae 6b 4f 5f 01 00 ac 91 1c f1 ea 6a 6c f4
## [16777] 5d 39 22 95 6e 01 80 a6 30 60 0d 81 94 d2 3f 2c dd 00 00 d0 30 3f d6 ed
## [16801] 4c bc b5 74 04 00 34 85 bf 1a 0d b8 c5 e9 0b 4e ac d3 d3 4f 44 44 ab 74
## [16825] 0b 00 40 c3 f4 22 a7 d7 9f 7d d3 bd 9f 28 1d 02 00 c3 ce 13 58 03 ae 8e
## [16849] a7 5e 1b c6 2b 00 80 12 5a 91 f2 b5 0f ee 3c 77 53 e9 10 00 18 76 06 ac
## [16873] 01 97 bd 3e 08 00 50 d2 0b 5b 75 eb e6 fb b7 6f 39 be 74 08 00 0c 33 03
## [16897] d6 00 cb 11 29 45 36 60 01 00 94 35 39 3a 52 7d 20 ef f2 bb 35 00 ac 16
## [16921] 3f 64 07 d8 fc 8e c9 76 44 3a ad 74 07 00 40 e3 e5 98 5e b8 6b e2 97 4b
## [16945] 67 00 c0 b0 32 60 0d b0 aa ea 79 fa 0a 00 a0 6f e4 5f ea 4e 8f ef 2c 5d
## [16969] 01 00 c3 c8 80 35 d0 dc bf 02 00 e8 2b 29 7d 60 61 e7 96 f3 4b 67 00 c0
## [16993] b0 49 a5 03 38 32 9f bb 62 db 31 fb 47 f6 ed 8d 88 f5 a5 5b 00 00 f8 26
## [17017] 0f 2d 2f f7 5e b9 e5 96 07 9f 28 1d 02 00 c3 c2 13 58 03 6a 5f 6b ff c5
## [17041] 61 bc 02 00 e8 47 67 8e 8e b4 76 df 71 d5 b6 d1 d2 21 00 30 2c 0c 58 03
## [17065] aa 4a b5 d7 07 01 00 fa 54 8e 98 da f8 f8 be df 2c dd 01 00 c3 c2 80 35
## [17089] a0 72 a4 d7 97 6e 00 00 e0 a0 7e a6 db 99 f8 a9 d2 11 00 30 0c dc c0 1a
## [17113] 40 0f 5d b1 e5 ac de 48 b5 50 ba 03 00 80 e7 d5 cb 51 5d b6 f9 c6 b9 ff
## [17137] 53 3a 04 00 06 99 27 b0 06 50 6f a4 e5 e9 2b 00 80 c1 d0 4a 51 5f d7 dd
## [17161] d9 3e a7 74 08 00 0c 32 03 d6 20 4a d9 80 05 00 30 38 36 46 1d b7 2c 4e
## [17185] 5f 70 62 e9 10 00 18 54 06 ac 01 73 c7 55 db 46 23 c7 6b 4b 77 00 00 70
## [17209] 58 b6 d4 e9 e9 6b 6e 9b 9a 1a 29 1d 02 00 83 c8 80 35 60 4e 7c e2 ab df
## [17233] 1d 11 c7 97 ee 00 00 e0 b0 5d f6 f2 13 1f fd 8d d2 11 00 30 88 0c 58 03
## [17257] 26 d5 e9 d2 d2 0d 00 00 1c 99 9c d2 5b 7d 33 21 00 1c 3e 03 d6 a0 49 06
## [17281] 2c 00 80 c1 96 df 31 df 99 7c 4d e9 0a 00 18 24 a9 74 00 87 ee fe ed 5b
## [17305] 8e 1f 6d 55 7b 23 c2 ed 04 00 80 c1 f6 c5 a8 e2 c2 b3 af bf 6f 4f e9 10
## [17329] 00 18 04 9e c0 1a 20 a3 ad 74 71 18 af 00 00 86 81 6f 26 04 80 c3 60 c0
## [17353] 1a 28 5e 1f 04 00 18 22 be 99 10 00 0e 91 01 6b b0 bc ae 74 00 00 00 2b
## [17377] ca 37 13 02 c0 21 70 03 6b 40 3c bc 7d cb a9 4b ad ea 6f 4a 77 00 00 b0
## [17401] 1a d2 5b ce be f1 de ff 56 ba 02 00 fa 95 27 b0 06 c4 81 56 e5 f5 41 00
## [17425] 80 a1 95 df d1 dd d9 f6 fb 1e 00 3c 07 03 d6 a0 c8 e1 17 1a 00 80 e1 d5
## [17449] 8a 3a ae db b3 63 72 a2 74 08 00 f4 23 03 d6 00 c8 11 29 25 03 16 00 c0
## [17473] 90 3b a1 aa ea 0f 2f 6c df 7a 72 e9 10 00 e8 37 06 ac 01 30 bf 73 62 22
## [17497] 22 5e 5a ba 03 00 80 55 77 66 1e e9 fd fe 23 b3 a7 6d 28 1d 02 00 fd c4
## [17521] 80 35 00 aa 5e f6 f4 15 00 40 53 e4 b8 f0 c0 f2 f1 ef cf bb fc ae 0e 00
## [17545] 5f e7 87 e2 20 f0 fa 20 00 40 d3 cc 76 ef 1a 7f 5b e9 08 00 e8 17 a9 74
## [17569] 00 07 77 c7 55 db 46 37 3e be 6f 6f 44 1c 57 ba 05 00 80 b5 96 7e ec ec
## [17593] 1b ef bd ba 74 05 00 94 e6 09 ac 3e b7 f1 f1 a7 2e 0c e3 15 00 40 43 e5
## [17617] 77 cf cf 6c b9 a4 74 05 00 94 66 c0 ea 77 c9 fd 2b 00 80 06 1b 49 b9 ba
## [17641] 61 61 fb 96 2d a5 43 00 a0 24 03 56 df cb af 2b 5d 00 00 40 51 27 e6 56
## [17665] f5 e1 07 ae 38 f7 45 a5 43 00 a0 14 03 56 1f db 73 f9 e6 17 44 8e 0b 4b
## [17689] 77 00 00 50 dc d9 23 23 23 37 2d 5e 79 e6 58 e9 10 00 28 c1 80 d5 c7 5a
## [17713] 1b d6 4f 45 44 ab 74 07 00 00 fd 20 7f 6f fd a5 63 de 93 7d 11 13 00 0d
## [17737] 64 c0 ea 67 75 cf fd 2b 00 00 fe 5e ca 6f 5c e8 4c ec 2a 9d 01 00 6b cd
## [17761] 80 d5 c7 ea 94 0c 58 00 00 7c 8b fc 4b dd ce c4 9b 4b 57 00 c0 5a f2 f8
## [17785] 71 9f da 33 73 fe 69 55 5e 7a a4 74 07 00 00 7d 69 39 e5 fc fd 9b 6e ba
## [17809] ff a3 a5 43 00 60 2d 78 02 ab 4f a5 7c e0 b5 a5 1b 00 00 e8 5b 23 39 a5
## [17833] eb 16 3a 93 af 28 1d 02 00 6b c1 80 d5 a7 52 78 7d 10 00 80 83 3a be 8e
## [17857] fa 23 f3 3b b6 9e 5e 3a 04 00 56 9b 01 ab 0f 7d ed 9b 65 0c 58 00 00 1c
## [17881] 54 8a 38 35 55 bd 5b 17 a7 2f 38 b1 74 0b 00 ac 26 03 56 1f ea 76 da 93
## [17905] 11 71 4a e9 0e 00 00 06 c2 64 9d 0e dc 30 37 3b b9 ae 74 08 00 ac 16 03
## [17929] 56 1f 4a c9 d3 57 00 00 1c 8e 7c c9 d8 72 fd 9e ec 4b 9a 00 18 52 06 ac
## [17953] 7e 54 c7 eb 4a 27 00 00 30 70 7e a4 db 19 ff d5 d2 11 00 b0 1a fc 85 a6
## [17977] cf cc cd 4e ae 1b eb d5 7b 23 c7 b1 a5 5b 00 00 18 3c 39 e7 9f dc 7c d3
## [18001] fd ef 2e dd 01 00 2b c9 13 58 7d 66 43 af 7e 95 f1 0a 00 80 23 95 52 7a
## [18025] 57 77 67 fb 0d a5 3b 00 60 25 19 b0 fa 4d ed fe 15 00 00 47 a5 15 39 ae
## [18049] 9d 9f 1e df 56 3a 04 00 56 8a 01 ab df 38 e0 0e 00 c0 d1 ca 71 6c 4a e9
## [18073] c3 8b d3 e3 67 96 4e 01 80 95 e0 06 56 1f e9 ce 6e 3a 21 96 d7 7f 21 22
## [18097] 5a a5 5b 00 00 18 0a 0f 2c 2f f7 fe c1 96 5b 1e 7c a2 74 08 00 1c 0d 4f
## [18121] 60 f5 91 7a 79 6c 2a 8c 57 00 00 ac 9c 2d 23 a3 ad 3f 78 f4 b2 f3 dd 58
## [18145] 05 60 a0 19 b0 fa 48 f2 fa 20 00 00 2b 2d c7 85 7f 7b cc d2 b5 77 5c b5
## [18169] 6d b4 74 0a 00 1c 29 03 56 1f 49 39 bf ae 74 03 00 00 43 28 c5 1b 5e f8
## [18193] f8 be df c9 4e 88 00 30 a0 0c 58 7d 62 7e c7 d6 d3 23 62 4b e9 0e 00 00
## [18217] 86 53 8e f8 d1 85 4e fb 3f 96 ee 00 80 23 61 c0 ea 17 69 f9 b5 a5 13 00
## [18241] 00 18 7a ff b6 3b d3 fe b9 d2 11 00 70 b8 0c 58 7d 22 45 e5 f5 41 00 00
## [18265] 56 5f 8e df ec 4e 8f ff 50 e9 0c 00 38 1c 06 ac 3e 90 23 52 a4 ec 09 2c
## [18289] 00 00 d6 46 4a ef 5f e8 b4 fd 01 15 80 81 61 c0 ea 03 f3 3b 26 db 11 71
## [18313] 72 e9 0e 00 00 1a 63 34 47 dc 30 3f 3d be ad 74 08 00 1c 0a 03 56 1f 68
## [18337] a5 de 6b 4a 37 00 00 d0 38 c7 a5 94 6e ed ee 6c 9f 53 3a 04 00 9e 8f 01
## [18361] ab 0f e4 94 2e 29 dd 00 00 40 23 bd 38 72 fc e1 e2 ec e4 29 a5 43 00 e0
## [18385] 60 0c 58 85 e5 5d 51 45 84 01 0b 00 80 32 72 9c 55 2f d7 b7 ee b9 7c f3
## [18409] 0b 4a a7 00 c0 73 31 60 15 36 7f f7 c4 2b 22 62 63 e9 0e 00 00 1a ed 82
## [18433] d6 d8 e8 ef 2f 5e 79 e6 58 e9 10 00 78 36 06 ac c2 5a b9 76 ff 0a 00 80
## [18457] e2 72 c4 54 ef cb 63 d7 dc 71 d5 b6 d1 d2 2d 00 f0 ad 0c 58 85 e5 ec fe
## [18481] 15 00 00 fd 21 45 da be f1 f1 7d ef fd da 99 0b 00 e8 1b 7e 30 15 f4 b5
## [18505] bf 6e 5d 5c ba 03 00 00 9e e1 47 16 3f 33 fe 5b 39 22 95 0e 01 80 af 33
## [18529] 60 15 74 d2 13 fb b6 45 c4 71 a5 3b 00 00 e0 99 72 4a 6f 5d e8 4c ec 2a
## [18553] dd 01 00 5f 67 c0 2a 28 47 72 ff 0a 00 80 3e 95 7f a9 3b d3 fe b9 d2 15
## [18577] 00 10 61 c0 2a 2a d7 d9 fd 2b 00 00 fa 57 8e df ec 76 26 de 5c 3a 03 00
## [18601] bc d7 5e c8 9e cb 37 af af c6 46 bf 14 11 be aa 18 00 80 7e 56 47 e4 d9
## [18625] b3 6f bc ff 86 d2 21 00 34 97 27 b0 0a 19 19 1b 79 55 18 af 00 00 e8 7f
## [18649] 55 44 fa 50 77 67 fb d2 d2 21 00 34 97 01 ab 90 3a 2a f7 af 00 00 18 14
## [18673] eb 22 c7 4d 8b 33 ed 57 95 0e 01 a0 99 0c 58 c5 b8 7f 05 00 c0 00 c9 71
## [18697] 6c 9d e3 d6 ee 8e f1 f3 4a a7 00 d0 3c 6e 60 15 f0 e8 65 e7 1f fb b7 c7
## [18721] 2e 7d 31 22 46 4b b7 00 00 c0 61 7a 34 f7 aa ef db 7c f3 dc 7c e9 10 00
## [18745] 9a c3 13 58 05 ec 3b e6 c0 f7 86 f1 0a 00 80 c1 74 4a 6a d5 1f ef ce 4e
## [18769] 9e 51 3a 04 80 e6 30 60 15 90 53 72 ff 0a 00 80 41 f6 f2 e8 d5 9f 78 78
## [18793] fb 96 53 4b 87 00 d0 0c 06 ac 32 dc bf 02 00 60 b0 e5 d8 7c a0 55 7d 7c
## [18817] be 73 fe 4b 4a a7 00 30 fc dc c0 5a 63 dd d9 4d 27 c4 f2 fa bd 61 3c 04
## [18841] 00 60 38 7c 76 dd 48 75 c9 e9 bb e7 f6 96 0e 01 60 78 19 51 d6 58 3a b0
## [18865] ee a2 f0 ef 0e 00 c0 f0 38 ff c0 72 fd d1 ee ec a6 13 4a 87 00 30 bc 0c
## [18889] 29 6b cc fd 2b 00 00 86 d0 b6 e8 ad ff c8 dc ec e4 71 a5 43 00 18 4e 06
## [18913] ac b5 96 b2 fb 57 00 00 0c 9f 1c df b3 61 b9 be e5 73 57 6c 3b a6 74 0a
## [18937] 00 c3 c7 0d ac 35 f4 c0 15 e7 be 68 64 a4 f5 78 e9 0e 00 00 58 45 1f ad
## [18961] 4e d8 bf e3 ac f7 3d f4 54 e9 10 00 86 87 27 b0 d6 d0 e8 e8 e8 54 e9 06
## [18985] 00 00 58 65 97 d5 5f 39 e6 da b9 d9 c9 75 a5 43 00 18 1e 06 ac 35 94 a3
## [19009] 76 ff 0a 00 80 e1 97 f3 15 63 cb bd 0f de 36 35 35 52 3a 05 80 e1 60 c0
## [19033] 5a 4b 39 b9 7f 05 00 40 43 a4 1f 38 63 e3 a3 ef cb b3 d1 2a 5d 02 c0 e0
## [19057] 33 60 ad 91 87 b7 6f 39 35 22 8f 97 ee 00 00 80 b5 93 7e 78 61 79 fc 77
## [19081] f2 2e ff ef 00 e0 e8 f8 41 b2 46 96 5a 2d 4f 5f 01 00 d0 40 e9 cd 0b 77
## [19105] 8d ff ae 11 0b 80 a3 e1 87 c8 9a c9 ee 5f 01 00 d0 50 46 2c 00 8e 8e 1f
## [19129] 20 6b 25 85 27 b0 00 00 68 30 23 16 00 47 2e 95 0e 68 82 87 ae d8 72 56
## [19153] 6f a4 5a 28 dd 01 00 00 c5 a5 f4 be 4d ad 7b ff 69 da 1d bd d2 29 00 0c
## [19177] 0e 7f fd 58 03 bd 11 f7 af 00 00 20 22 22 72 be 72 a1 37 f1 bb be 9d 10
## [19201] 80 c3 61 c0 5a 13 b5 fb 57 00 00 f0 75 39 5f b9 b8 dc 7e 8f 11 0b 80 43
## [19225] 65 c0 5a 65 39 22 e5 48 9e c0 02 00 80 67 c8 11 3f 6a c4 02 e0 50 19 b0
## [19249] 56 d9 e2 f6 2d e7 a6 88 53 4b 77 00 00 40 bf f9 da 88 f5 5e 23 16 00 cf
## [19273] c7 80 b5 ca 72 55 79 7d 10 00 00 9e 43 8e 78 93 11 0b 80 e7 63 c0 5a 6d
## [19297] 29 1b b0 00 00 e0 20 8c 58 00 3c 1f 03 d6 2a ca bb a2 8a 48 53 a5 3b 00
## [19321] 00 a0 df e5 88 37 2d 2c 4d 7c e0 b6 a9 a9 91 d2 2d 00 f4 1f 03 d6 2a 5a
## [19345] bc 7b cb d6 88 78 51 e9 0e 00 00 18 08 29 bf f1 8c 8d 8f 5d 33 37 3b b9
## [19369] ae 74 0a 00 fd c5 80 b5 8a 72 76 ff 0a 00 00 0e d3 cc d8 52 7d e3 23 b3
## [19393] a7 6d 28 1d 02 40 ff 30 60 ad aa 64 c0 02 00 80 c3 95 e2 0d 07 96 8f fb
## [19417] 83 b9 d9 c9 e3 4a a7 00 d0 1f 0c 58 ab 24 cf 46 2b 72 be a8 74 07 00 00
## [19441] 0c a6 f4 9a b1 5e fd 87 dd d9 4d 27 94 2e 01 a0 3c 03 d6 2a 99 ef b5 5f
## [19465] 11 11 7e d8 02 00 c0 91 ca f1 3d b1 bc fe 13 f7 75 c6 4f 2a 9d 02 40 59
## [19489] 06 ac 55 52 e5 3c 55 ba 01 00 00 86 c0 b6 75 91 6e 5f d8 be f5 e4 d2 21
## [19513] 00 94 63 c0 5a 25 39 e2 e2 d2 0d 00 00 30 24 b6 e6 56 ef 53 7b 66 ce 3f
## [19537] ad 74 08 00 65 18 b0 56 41 9e 8d 56 8a e4 fe 15 00 00 ac 9c 73 ab 58 fa
## [19561] d4 43 57 6c 39 ab 74 08 00 6b cf 80 b5 0a e6 eb 89 f3 23 e2 c4 d2 1d 00
## [19585] 00 30 54 72 9c d5 1b 49 9f ea 76 c6 cf 2d 9d 02 c0 da 32 60 ad 82 aa 76
## [19609] ff 0a 00 00 56 47 3a 2d 22 7d aa bb 63 fc bc d2 25 00 ac 1d 03 d6 2a c8
## [19633] 61 c0 02 00 80 55 74 72 54 e9 53 f3 33 5b be b7 74 08 00 6b c3 80 b5 c2
## [19657] dc bf 02 00 80 35 71 62 ca d5 c7 ba 3b db 6f 28 1d 02 c0 ea 33 60 ad 30
## [19681] f7 af 00 00 60 cd 6c 88 3a 7e bf db 99 f8 e1 d2 21 00 ac 2e 03 d6 0a 73
## [19705] ff 0a 00 00 d6 d4 48 44 fe 1f f3 9d 89 9f 2d 1d 02 c0 ea 31 60 ad 30 f7
## [19729] af 00 00 60 ed a5 c8 ff 65 61 ba fd 2b 39 22 95 6e 01 60 e5 f9 70 5f 41
## [19753] 79 57 54 0b 77 b5 9f 88 88 8d a5 5b 00 00 a0 91 52 fc f6 a6 d6 7d 6f 4d
## [19777] bb a3 57 3a 05 80 95 63 c0 5a 41 8b d3 93 17 d4 a9 fe cb d2 1d 00 00 d0
## [19801] 64 39 e2 da a7 47 aa 7f 32 b9 7b ee 40 e9 16 00 56 86 57 08 57 50 5d d5
## [19825] 53 a5 1b 00 00 a0 e9 52 c4 0f 8e 2d d7 b7 cc cd 4e 1e 57 ba 05 80 95 61
## [19849] c0 5a 41 29 c7 54 e9 06 00 00 20 22 22 2e 1b eb d5 1f bf af 33 7e 52 e9
## [19873] 10 00 8e 9e 01 6b 85 e4 5d 51 e5 88 8b 4a 77 00 00 00 5f 93 e3 c2 d1 48
## [19897] 7f 34 bf 63 eb e9 a5 53 00 38 3a 06 ac 15 f2 d0 67 26 cf 0f c7 db 01 00
## [19921] a0 af a4 88 76 aa 7a ff 77 61 e7 96 f3 4b b7 00 70 e4 0c 58 2b c4 fd 2b
## [19945] 00 00 e8 5b 2f cb 75 f5 e9 ee ce f6 a5 a5 43 00 38 32 06 ac 15 e2 fe 15
## [19969] 00 00 f4 b5 e3 a3 8e 5b e7 3b ed 37 95 0e 01 e0 f0 19 b0 56 80 fb 57 00
## [19993] 00 30 10 46 52 c4 fb e7 a7 db ff 3e 47 a4 d2 31 00 1c 3a 1f da 2b 60 71
## [20017] 7a f2 82 3a d5 7f 59 ba 03 00 00 38 34 39 e2 77 1f f9 e2 c9 6f b9 e4 f6
## [20041] db 97 4b b7 00 f0 fc 3c 81 b5 02 ea a8 2f 2e dd 00 00 00 1c ba 14 f1 4f
## [20065] cf d8 f8 d8 cd 73 b3 93 c7 95 6e 01 e0 f9 19 b0 56 42 e5 fe 15 00 00 0c
## [20089] a0 cb c7 96 eb 4f 2e ce 4e 9e 52 3a 04 80 83 33 60 1d a5 bc 2b aa c8 e1
## [20113] 09 2c 00 00 18 4c df 59 2f d7 ff 77 71 e7 b9 e3 a5 43 00 78 6e 06 ac a3
## [20137] b4 78 d7 e4 79 11 b1 b1 74 07 00 00 70 c4 ce ac eb d6 9f 2c ec 98 f8 be
## [20161] d2 21 00 3c 3b 03 d6 51 ca b9 9e 2a dd 00 00 00 1c b5 8d b9 ca 1f eb ce
## [20185] 8c bf b1 74 08 00 df ce 80 75 b4 dc bf 02 00 80 61 b1 3e 72 fa e0 c2 74
## [20209] fb 57 f2 2e ff 57 02 e8 27 a9 74 c0 20 cb bb a2 5a b8 ab fd 44 78 85 10
## [20233] 00 00 86 4c be 6e c3 f2 b1 3f 7a ea 2d 77 ee 2b 5d 02 80 27 b0 8e 8a fb
## [20257] 57 00 00 30 ac d2 0f ec 6f ed fb e4 c3 db b7 9c 5a ba 04 00 03 d6 51 71
## [20281] ff 0a 00 00 86 58 8a ef 3a d0 aa fe 62 cf 4c fb 3b 4b a7 00 34 9d 01 eb
## [20305] 68 b8 7f 05 00 00 43 2d 45 9c 5a e5 f8 74 77 7a 7c 67 e9 16 80 26 33 60
## [20329] 1d a1 bc 2b aa c8 71 51 e9 0e 00 00 60 d5 6d 88 94 ae 5b e8 b4 7f 31 bb
## [20353] 23 0c 50 84 0f df 23 b4 b0 73 cb f9 b9 ae ee 2a dd 01 00 00 ac a9 ff 51
## [20377] 9d b0 ff 27 ce 7a df 43 4f 95 0e 01 68 12 4f 60 1d a1 ba 6e 4d 95 6e 00
## [20401] 00 00 d6 dc 8f e4 2f 6f f8 3f 0b db b7 9e 5c 3a 04 a0 49 0c 58 47 28 45
## [20425] 9e 2a dd 00 00 00 ac bd 1c f1 ea dc ea fd d9 e2 f4 e4 05 a5 5b 00 9a c2
## [20449] 80 75 04 f2 ae a8 22 e2 e2 d2 1d 00 00 40 31 2f af 53 fd 27 dd 99 f1 37
## [20473] 96 0e 01 68 02 03 d6 11 58 bc 7b cb d6 88 78 61 e9 0e 00 00 a0 a8 0d 91
## [20497] d3 07 bb 9d f6 6f dc 36 35 35 52 3a 06 60 98 19 b0 8e 80 fb 57 00 00 c0
## [20521] 33 fc 8b 33 36 7e fe a3 7b 66 37 bf b8 74 08 c0 b0 32 60 1d 01 f7 af 00
## [20545] 00 80 6f 96 2f a9 96 46 ef 98 9f 1e df 56 ba 04 60 18 19 b0 0e 93 fb 57
## [20569] 00 00 c0 b3 4a 71 46 4a e9 8f e7 3b ed 37 95 4e 01 18 36 a9 74 c0 a0 e9
## [20593] ee 18 3f 2f aa f4 d9 d2 1d 00 00 40 ff 4a 39 bf 7d ef 4b 8e fd f9 ef 7a
## [20617] f7 9d 4b a5 5b 00 86 81 27 b0 0e 53 6e 55 53 a5 1b 00 00 80 fe 96 53 7a
## [20641] eb c6 c7 f7 7d 7c 61 fb d6 93 4b b7 00 0c 03 03 d6 61 4a d9 fd 2b 00 00
## [20665] e0 90 5c 94 5b bd 3b 17 a6 27 bf bb 74 08 c0 a0 33 60 1d 86 bc 2b aa e4
## [20689] fe 15 00 00 70 e8 5e 96 53 fd 47 dd 99 f6 4f 67 27 5c 00 8e 98 0f d0 c3
## [20713] 30 df 69 6f 4d 11 77 97 ee 00 00 00 06 4f ca 71 4d ef e9 a5 ab ce b9 75
## [20737] fe 2b a5 5b 00 06 8d 27 b0 0e 43 e5 f5 41 00 00 e0 08 e5 14 ff b8 1a 1b
## [20761] bd 63 a1 33 f9 8a d2 2d 00 83 c6 80 75 18 72 f2 fa 20 00 00 70 54 ce c9
## [20785] 51 ff e9 42 67 fc 27 bc 52 08 70 e8 7c 60 1e a2 1c 91 16 3a ed c7 22 e2
## [20809] c5 a5 5b 00 00 80 61 90 3f f8 d4 48 eb a7 26 77 cf 7d b5 74 09 40 bf 33
## [20833] 60 1d a2 3d 3b 26 27 aa aa 9e 2b dd 01 00 00 0c 93 74 7f 8e 3c bb f9 c6
## [20857] fb ee 29 5d 02 d0 cf bc 42 78 88 aa 54 7b 7d 10 00 00 58 61 79 3c 45 fc
## [20881] f9 fc f4 f8 95 a5 4b 00 fa 99 01 eb 10 a5 70 ff 0a 00 00 58 15 1b 52 4a
## [20905] 57 77 67 26 ae fe dc 15 db 8e 29 1d 03 d0 8f 0c 58 87 20 47 a4 9c 62 aa
## [20929] 74 07 00 00 30 c4 72 be 72 ff c8 be 3f ef ee 18 3f af 74 0a 40 bf 31 60
## [20953] 1d 82 c5 ed 5b ce 8d 88 93 4b 77 00 00 00 43 6f 32 aa f4 17 dd e9 f6 3f
## [20977] cf bb fc 7f 0d e0 eb 7c 20 1e 82 7a a4 35 55 ba 01 00 00 68 8c f5 91 e2
## [21001] b7 ba 9f 99 f8 c8 e2 ec e4 29 a5 63 00 fa 81 01 eb 10 a4 9c dd bf 02 00
## [21025] 00 d6 54 4a f9 f5 f5 72 7d f7 c2 8e f1 2b 4a b7 00 94 66 c0 7a 1e 39 22
## [21049] 65 07 dc 01 00 80 32 5e 94 ab 74 73 77 a6 fd 2e 07 de 81 26 4b a5 03 fa
## [21073] 5d 77 67 fb 9c a8 e3 c1 d2 1d 00 00 40 b3 e5 88 fb 5a b9 7a e3 59 37 cd
## [21097] 7d a6 74 0b c0 5a f3 04 d6 f3 48 d9 d3 57 00 00 40 79 29 a2 5d a7 fa cf
## [21121] 17 66 c6 7f de 81 77 a0 69 7c e8 3d 8f 6c c0 02 00 00 fa c7 68 ce e9 d7
## [21145] 17 3e d3 fe c3 87 b7 6f 39 b5 74 0c c0 5a 31 60 1d 44 8e 48 11 79 aa 74
## [21169] 07 00 00 c0 37 49 71 e9 52 ab ba bb 3b 3d fe 43 d9 69 18 a0 01 7c d0 1d
## [21193] c4 83 3b cf dd d4 aa 5b dd d2 1d 00 00 00 07 71 63 ea b5 de b2 e9 e6 7b
## [21217] 1e 2b 1d 02 b0 5a 3c 81 75 10 ad 7a 64 aa 74 03 00 00 c0 f3 e8 44 ab 37
## [21241] 37 3f d3 fe c7 9e c6 02 86 95 01 eb 20 52 64 f7 af 00 00 80 be 97 23 4e
## [21265] 4a 39 fe d7 c2 cc c4 ee f9 ce f9 2f 29 dd 03 b0 d2 0c 58 07 91 c3 01 77
## [21289] 00 00 60 80 e4 bc 33 c5 d2 dc 42 67 62 b6 74 0a c0 4a f2 78 e9 73 58 9c
## [21313] 1e 3f b3 4e 69 b1 74 07 00 00 c0 11 da 5d 8f 2c fd cc 39 bb e7 1f 2f 1d
## [21337] 02 70 b4 3c 81 f5 1c 7a c9 d3 57 00 00 c0 40 9b ad 96 47 e7 ba d3 e3 3b
## [21361] 4b 87 00 1c 2d 03 d6 73 48 5e 1f 04 00 00 06 df 8b 23 a5 eb e6 3b ed 6b
## [21385] 16 67 27 4f 29 1d 03 70 a4 0c 58 cf 25 a5 a9 d2 09 00 00 00 2b 21 45 fc
## [21409] 60 bd 5c df 37 3f 3d 7e 55 de e5 ff 81 c0 e0 71 03 eb 59 cc ef d8 7a 7a
## [21433] aa 7a 0f 97 ee 00 00 00 58 79 e9 8f eb 2a 7e f2 9c eb ef 9d 2b 5d 02 70
## [21457] a8 2c ef cf a2 4a b5 d7 07 01 00 80 21 95 bf b7 aa f3 5f ce 77 da bf fa
## [21481] c8 ec 69 1b 4a d7 00 1c 0a 03 d6 b3 c8 55 9e 2a dd 00 00 00 b0 8a 46 53
## [21505] c4 2f 1e e8 1d ff d9 ee ce f6 a5 a5 63 00 9e 8f 01 eb d9 79 02 0b 00 00
## [21529] 18 7e 39 36 47 1d 1f eb 76 da bf b7 67 76 f3 8b 4b e7 00 3c 17 37 b0 be
## [21553] c5 03 b3 e7 be 6c 64 b9 f5 d7 a5 3b 00 00 00 d6 d8 de 88 f4 af 36 dd 78
## [21577] ef fb 52 44 2e 1d 03 f0 4c 9e c0 fa 16 23 bd ca d3 57 00 00 40 13 bd 30
## [21601] 22 bf 77 a1 d3 fe e4 42 67 f2 15 a5 63 00 9e c9 80 f5 2d 52 18 b0 00 00
## [21625] 80 46 fb be 1c f5 ff eb ce b4 df 75 5f 67 fc a4 d2 31 00 11 06 ac 6f 93
## [21649] b3 03 ee 00 00 40 e3 55 91 e3 2d eb 22 ed 59 e8 b4 7f e6 b6 a9 a9 91 d2
## [21673] 41 40 b3 b9 81 f5 0c 0f cd b4 5f da cb f1 b9 d2 1d 00 00 00 7d e6 ee 3a
## [21697] e7 9f 3d e7 a6 fb 6f 2f 1d 02 34 93 27 b0 9e 61 39 e2 a2 d2 0d 00 00 00
## [21721] 7d e8 bc 2a a5 db ba 9d f6 b5 0b 33 ed 97 97 8e 01 9a c7 80 f5 0c 29 c7
## [21745] 54 e9 06 00 00 80 3e 36 9b 73 dc df 9d 1e ff e5 47 66 4f db 50 3a 06 68
## [21769] 0e 03 d6 33 e4 08 07 dc 01 00 00 0e 6e 2c 52 da 75 60 e9 f8 fb 17 a6 27
## [21793] 7e 30 3b 4d 03 ac 01 1f 34 5f b3 b0 7d eb c9 b9 d5 7b b4 74 07 00 00 c0
## [21817] 60 c9 7f 51 e7 f8 37 ee 63 01 ab c9 13 58 5f d7 aa dd bf 02 00 00 38 6c
## [21841] e9 95 55 4a b7 75 a7 db 1f ee ee 18 3f af 74 0d 30 9c 0c 58 5f 93 23 7b
## [21865] 7d 10 00 00 e0 48 a5 78 43 54 e9 ae ee cc c4 d5 f3 3b b6 9e 5e 3a 07 18
## [21889] 2e 06 ac bf 37 55 3a 00 00 00 60 c0 a5 c8 f9 ca 54 f5 f6 2c 74 da bf f6
## [21913] 57 df 7f de c6 d2 41 c0 70 70 03 2b 22 1e b8 e2 dc 17 8d 8c b4 1e 2f dd
## [21937] 01 00 00 30 64 be 98 22 fd 87 74 c2 be 77 9c f5 be 87 9e 2a 1d 03 0c 2e
## [21961] 4f 60 45 c4 c8 48 e5 fe 15 00 00 c0 ca db 98 23 ff a7 fa 4b 1b 1e e8 76
## [21985] 26 de 7c db d4 d4 48 e9 20 60 30 19 b0 22 22 a7 ea 92 d2 0d 00 00 00 43
## [22009] 2b c5 19 11 f9 bd 67 6c 7c ec fe f9 e9 f1 2b 0d 59 c0 e1 f2 0a 61 44 cc
## [22033] 77 da f7 a6 88 76 e9 0e 00 00 80 86 e8 e6 c8 6f 7b e4 8b a7 7c f0 92 db
## [22057] 6f 5f 2e 1d 03 f4 bf c6 0f 58 7b 66 ce 3f ad ca 4b 8f 94 ee 00 00 00 68
## [22081] 20 43 16 70 48 1a ff 0a 61 ca 07 5e 5b ba 01 00 00 a0 a1 ce 4e 91 de 77
## [22105] c6 0b 1f bb 6f be d3 7e 93 57 0b 81 e7 62 c0 8a 74 69 e9 06 00 00 80 46
## [22129] cb b1 39 45 bc df 90 05 3c 97 46 bf 42 98 23 d2 42 a7 fd b9 88 38 a5 74
## [22153] 0b 00 00 00 5f 93 62 31 d5 f9 37 8e d9 b7 ee ea 53 3e fa d9 bf 2d 9d 03
## [22177] 94 d7 e8 01 6b be d3 de 9a 22 ee 2e dd 01 00 00 c0 b7 4b 11 5f c8 29 bd
## [22201] a3 6e 1d 78 e7 39 bb e7 1f 2f dd 03 94 d3 e8 57 08 53 c4 eb 4a 37 00 00
## [22225] 00 f0 ec 72 c4 49 91 f3 2f 57 cb a3 0f 77 a7 db ef dc 33 bd f5 ec d2 4d
## [22249] 40 19 8d 1e b0 22 87 fb 57 00 00 00 fd 6f 2c 52 fc 74 95 7a 0f 76 3b ed
## [22273] 6b bb 33 93 af 2c 1d 04 ac ad c6 be 42 38 37 3b b9 6e ac 57 ef 8d 1c c7
## [22297] 96 6e 01 00 00 e0 f0 a4 88 db eb 9c 7e ed ec 9b ee fd df 29 22 97 ee 01
## [22321] 56 57 63 07 ac 85 99 f6 45 39 c7 27 4b 77 00 00 00 70 34 d2 fd 29 d7 ef
## [22345] ec 3d bd fc 81 73 6e 9d ff 4a e9 1a 60 75 34 f7 15 c2 da eb 83 00 00 00
## [22369] 83 2f 8f e7 94 de 5e 8d 8d fe 4d 77 a6 fd ae 3d 3b 27 26 4b 17 01 2b af
## [22393] b9 4f 60 75 da 7f 92 23 5e 5d ba 03 00 00 80 95 95 22 6e cf 39 bf e3 e1
## [22417] 2f 9d f2 fb 97 dc 7e fb 72 e9 1e e0 e8 35 72 c0 ea ce 6e 3a 21 96 d7 7f
## [22441] 21 22 5a a5 5b 00 00 00 58 35 7f 93 53 fc f7 56 ab fa 9d b3 76 cf 3d 5a
## [22465] 3a 06 38 72 8d 1c b0 f6 74 26 76 54 91 6f 2a dd 01 00 00 c0 9a 58 8a 88
## [22489] eb ea 9c df bd f9 82 fb 3f 95 76 45 5d 3a 08 38 3c 8d 1c b0 e6 67 26 de
## [22513] 9e 72 fe 67 a5 3b 00 00 00 58 63 29 16 73 c4 d5 a9 55 bd ff ec dd 73 0f
## [22537] 97 ce 01 0e 4d 23 07 ac 6e a7 7d 7f 44 6c 29 dd 01 00 00 40 31 39 72 7c
## [22561] 22 22 bf 77 dd e8 57 6f 3a 7d f7 5f ef 2f 1d 04 3c b7 c6 0d 58 f3 3b b6
## [22585] 9e 9e aa 9e 95 1d 00 00 80 af fb 72 44 7c 28 52 f5 de 4d 37 cc dd 91 22
## [22609] 72 e9 20 e0 9b 35 6f c0 9a 1e bf 32 a5 74 75 e9 0e 00 00 00 fa d2 5c 4a
## [22633] f9 ea 91 e5 fc a1 33 6e 7e e0 73 a5 63 80 bf d3 b8 01 ab 3b 3d f1 c1 48
## [22657] f9 8d a5 3b 00 00 00 e8 6b 39 45 7c 32 47 ba a6 1e 39 70 fd 39 bb e7 1f
## [22681] 2f 1d 04 4d d6 a8 01 2b 47 a4 85 ce f8 a3 11 e9 25 a5 5b 00 00 00 18 18
## [22705] bd 88 f8 44 44 ba 66 e4 40 eb c6 97 7f f8 ee 2f 96 0e 82 a6 69 d4 80 b5
## [22729] b0 73 cb f9 b9 ae ee 2a dd 01 00 00 c0 c0 5a 8a 88 3f 8c 48 ff 6b a9 d7
## [22753] bb 79 fc e6 07 9e 2c 1d 04 4d d0 a8 01 ab db 19 ff 97 11 e9 3f 97 ee 00
## [22777] 00 00 60 28 3c 15 11 1f 89 48 37 ac 1b 49 b7 9e be 7b 6e 6f e9 20 18 56
## [22801] 0d 1b b0 da 1f 89 88 cb 4b 77 00 00 00 30 74 7a 29 e2 8f 72 c4 cd b9 57
## [22825] dd b2 f9 e6 b9 f9 d2 41 30 4c 1a 33 60 ed b9 7c f3 fa 6a 6c 74 6f 44 1c
## [22849] 53 ba 05 00 00 80 e1 96 23 ee 4b 11 37 e7 3a df 7c f6 ba fb ff 2c ed 8e
## [22873] 5e e9 26 18 64 8d 19 b0 16 3b e3 17 d7 91 6e 2f dd 01 00 00 40 e3 3c 1e
## [22897] 91 ff 20 a7 74 4b de bf f4 89 73 6e 9d ff 4a e9 20 18 34 8d 19 b0 16 66
## [22921] da 6f cb 39 fe 7d e9 0e 00 00 00 1a ad 17 29 fe 2c 22 7d 2c 47 ef 63 5f
## [22945] 7a d1 71 7f fe 5d ef be 73 a9 74 14 f4 bb c6 0c 58 dd 99 f6 9f 46 8e 0b
## [22969] 4b 77 00 00 00 c0 33 3c 99 23 df 56 e5 f8 58 6a d5 1f 3f f3 fa 07 1f 48
## [22993] 11 b9 74 14 f4 9b 46 0c 58 8b d3 17 9c 58 a7 a7 bf 10 11 55 e9 16 00 00
## [23017] 00 38 88 47 22 f2 c7 23 c5 c7 eb 58 f7 a9 73 6e f8 ec 5f 97 0e 82 7e d0
## [23041] 88 01 6b 7e ba 3d 9d 52 dc 58 ba 03 00 00 00 0e d3 43 11 f1 e9 9c d2 a7
## [23065] 73 8a 4f 6f 3e ef de fb d2 ae a8 4b 47 c1 5a 6b c4 80 d5 9d 6e bf 33 52
## [23089] fc 74 e9 0e 00 00 00 38 4a 5f 8c 88 3f 8e 88 4f e7 54 7f 3a ef ef dd 71
## [23113] ce ad f3 4f 97 8e 82 d5 d6 8c 01 ab d3 7e 20 22 ce 2d dd 01 00 00 00 2b
## [23137] ec e9 88 74 47 e4 fc 17 29 d2 9d a9 b5 7c c7 99 e7 3d f8 a0 a7 b4 18 36
## [23161] 43 3f 60 75 67 27 cf 88 e5 fa af 4a 77 00 00 00 c0 1a 79 32 47 fc bf 14
## [23185] e9 ce c8 f5 1d d1 4a 77 6c 3a ef be ae 51 8b 41 36 fc 03 56 67 e2 c7 22
## [23209] f2 7b 4a 77 00 00 00 40 41 5f 89 48 77 e6 a8 ef 8c 88 7b 22 c7 3d c7 f4
## [23233] 8e bd ef d4 5b ee dc 57 3a 0c 0e 45 03 06 ac f6 ff 8c 88 1f 2a dd 01 00
## [23257] 00 00 7d 26 47 8a 6e 44 dc 93 22 ee a9 23 ee c9 29 dd f3 e5 93 36 3c f8
## [23281] 5d ef be 73 a9 74 1c 3c d3 50 0f 58 79 57 54 0b 77 b5 1f 8d 88 17 97 6e
## [23305] 01 00 00 80 01 b1 14 11 0f e4 88 7b 23 62 4f e4 3c 1f 55 de 13 79 fd 9e
## [23329] b3 6f fc ec e3 29 22 97 0e a4 79 86 7a c0 5a e8 4c be 22 47 fd 99 d2 1d
## [23353] 00 00 00 30 24 9e 8c 88 3d 39 62 3e 9e 31 6e b5 5a 23 dd 33 27 e7 3e ef
## [23377] ce 16 ab 65 b8 07 ac 99 f1 9f cf 39 fd 7a e9 0e 00 00 00 68 80 03 11 f1
## [23401] 48 8a 78 24 22 1e 8e 14 0f d7 75 7e 24 52 eb e1 5c c7 c3 bd bc fc c8 f8
## [23425] cd 0f 3c 59 3a 92 c1 34 d4 03 d6 fc f4 c4 ff 4e 29 bf be 74 07 00 00 00
## [23449] 10 11 11 5f 8a 88 bf 8e 88 c7 22 f2 a3 11 d5 63 39 f2 a3 55 8a 47 a3 ce
## [23473] 8f 45 2b 3f 1a 4b a3 8f 9d b5 fe 9e 27 d2 ee e8 95 8e a5 7f 0c ed 80 b5
## [23497] e7 f2 cd eb ab b1 d1 2f 46 c4 86 d2 2d 00 00 00 c0 61 a9 23 e2 f1 88 f8
## [23521] 7c 8e d8 9b 22 f6 46 8a bd 29 c7 de 1c 79 6f ce b1 b7 aa 5a 7b eb 1c 7b
## [23545] 5b 39 f6 d6 a3 b1 37 62 ff 97 37 c5 c2 57 0d 5f c3 69 78 07 ac e9 f1 a9
## [23569] 2a a5 db 4a 77 00 00 00 00 6b 6a 5f 44 3c 19 29 9e 4c 75 3c 99 53 3c 19
## [23593] 7f 77 bb eb c9 94 d2 57 ea 3a 9e ac 22 ef cb 91 f7 47 95 f6 e7 48 fb 23
## [23617] e7 fd 55 a4 fd 39 ea fd 91 ab fd b9 ea 3d 95 a3 da 3f 9a 5b fb 0f 54 07
## [23641] 9e 4e 79 ec 40 6b b9 5e 6a f5 d2 81 af 1e 53 2f 45 c4 81 89 dd 73 4b 0e
## [23665] da af 9d a1 1d b0 e6 77 6c 3d 3d 8d f4 b6 94 ee 00 00 00 00 86 53 55 e7
## [23689] a5 5e 1d 4b 69 24 1f 48 f5 c8 52 1d 75 6f 24 5a bd 5e 2c f7 ea 3c d2 1b
## [23713] 89 e5 5e 3d da aa 97 63 a9 b7 2e 46 7b cb f1 74 6f 2c d6 f7 0e c4 fe 5e
## [23737] fd d5 2a f7 46 5b b9 b7 7e 24 9f f4 e5 56 ae d7 8f e6 de d8 63 f9 a9 af
## [23761] 8e d5 07 8e 5b 9f f7 6f 1c cb 4f 3e 78 7c 9e 7a f1 ed 39 26 9f 65 28 db
## [23785] f5 ec e3 99 51 0d 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
## [23809] 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
## [23833] 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
## [23857] 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
## [23881] 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
## [23905] 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
## [23929] 00 9e cb ff 07 dc 5a e2 36 f8 b1 1a b9 00 00 00 00 49 45 4e 44 ae 42 60
## [23953] 82
```

``` r
# Upload image to Bluesky
img_resp <- request("https://bsky.social/xrpc/com.atproto.repo.uploadBlob") |>
  req_method("POST") |>
  req_headers(
    `Content-Type` = "img/png",
    Authorization = paste0("Bearer ", session$accessJwt)
  ) |>
  req_body_raw(thing$raw) |>
  req_perform()

uploaded_images <- resp_body_json(img_resp)

full_post <- list(
  "$type" = "app.bsky.feed.post",
  text = glue::glue("A randomly generated {x} distribution. Thanks for the code @andrew.heiss.phd", x = thing$dist_details),
  createdAt = format(as.POSIXct(Sys.time(), tz = "UTC"), "%Y-%m-%dT%H:%M:%OS6Z"),
  langs = list("en-US"),
  embed = list(
    "$type" = "app.bsky.embed.images",
    images = list(list(
      alt = thing$dist_details,
      image = uploaded_images$blob
    ))
  )
)

# Post the post
resp <- request("https://bsky.social/xrpc/com.atproto.repo.createRecord") |>
  req_method("POST") |>
  req_headers(Authorization = paste0("Bearer ", session$accessJwt)) |>
  req_body_json(list(
    repo = session$did,
    collection = "app.bsky.feed.post",
    record = full_post
  ))

# Actually make the request and post the thing
# resp |> req_dry_run()
resp |> req_perform()
```

```
## <httr2_response>
```

```
## POST https://bsky.social/xrpc/com.atproto.repo.createRecord
```

```
## Status: 200 OK
```

```
## Content-Type: application/json
```

```
## Body: In memory (276 bytes)
```
