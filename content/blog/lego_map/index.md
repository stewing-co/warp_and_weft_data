---
title: "Benjamin Nowak's Lego Maps"
subtitle: ""
excerpt: ""
date: 2024-11-26
author: "Benjamin Nowak"
draft: false
series:
tags: ["R", "bluesky", "maps"]
categories:
layout: single
---


``` r
library(tidyverse) 
```

```
## ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
## ✔ dplyr     1.1.4     ✔ readr     2.1.5
## ✔ forcats   1.0.0     ✔ stringr   1.5.1
## ✔ ggplot2   3.5.1     ✔ tibble    3.2.1
## ✔ lubridate 1.9.3     ✔ tidyr     1.3.1
## ✔ purrr     1.0.2     
## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
## ✖ dplyr::filter() masks stats::filter()
## ✖ dplyr::lag()    masks stats::lag()
## ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors
```

``` r
library(sf)
```

```
## Linking to GEOS 3.12.2, GDAL 3.9.3, PROJ 9.4.1; sf_use_s2() is TRUE
```

``` r
map <- sf::read_sf('https://github.com/BjnNowak/lego_map/raw/main/data/france_sport.gpkg')

ggplot(map, aes(fill=value))+
  geom_sf()
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-1-1.png" width="672" />


``` r
# Create classes
clean<-map%>%
  mutate(clss=case_when(
    value<18~"1",
    value<20~"2",
    value<22~"3",
    value<24~"4",
    value<26~"5",
    TRUE~"6"
  ))

# Set color palette
pal <- c("#bb3e03","#ee9b00","#e9d8a6","#94d2bd","#0a9396","#005f73")
# Set color background
bck <- "#001219"

# Set theme 
theme_custom <- theme_void()+
  theme(
    plot.margin = margin(1,1,10,1,"pt"),
    plot.background = element_rect(fill=bck,color=NA),
    legend.position = "bottom",
    legend.title = element_text(hjust=0.5,color="white",face="bold"),
    legend.text = element_text(color="white")
  )

# Make choropleth
ggplot(clean, aes(fill=clss))+
  geom_sf()+
  labs(fill="Member of a sport association")+
  guides(
    fill=guide_legend(
      nrow=1,
      title.position="top",
      label.position="bottom"
    )
  )+
  scale_fill_manual(
    values=pal,
    label=c("< 18 %","< 20 %","< 22 %","< 24 %","< 26 %", "≥ 26 %")
  )+
  theme_custom
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-2-1.png" width="672" />


``` r
# Make grid
grd<-st_make_grid(
    clean, # map name 
    n = c(60,60) # number of cells per longitude/latitude
  )%>%
  # convert back to sf object
  st_sf()%>%
  # add a unique id to each cell 
  # (will be useful later to get back centroids data)
  mutate(id=row_number())
  
# Extract centroids
cent<-grd%>%
  st_centroid()
```

```
## Warning: st_centroid assumes attributes are constant over geometries
```

``` r
# Take a look at the results
ggplot()+
  geom_sf(grd,mapping=aes(geometry=geometry))+
  geom_sf(cent,mapping=aes(geometry=geometry),pch=21,size=0.5)+
  theme_void()
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-3-1.png" width="672" />


``` r
# Intersect centroids with basemap
cent_clean<-cent%>%
  st_intersection(clean)
```

```
## Warning: attribute variables are assumed to be spatially constant throughout
## all geometries
```

``` r
# Make a centroid without geom
# (convert from sf object to tibble)
cent_no_geom <- cent_clean%>%
  st_drop_geometry()

# Join with grid thanks to id column
grd_clean<-grd%>%
  #filter(id%in%sel)%>%
  left_join(cent_no_geom)
```

```
## Joining with `by = join_by(id)`
```


``` r
ggplot()+
  geom_sf(
    # drop_na() is one way to suppress the cells outside the country
    grd_clean%>%drop_na(), 
    mapping=aes(geometry=geometry,fill=clss)
  )+
  geom_sf(cent_clean,mapping=aes(geometry=geometry),fill=NA,pch=21,size=0.5)+
  labs(fill="Member of a sport association")+
  guides(
    fill=guide_legend(
      nrow=1,
      title.position="top",
      label.position="bottom"
    )
  )+
  scale_fill_manual(
    values=pal,
    label=c("< 18 %","< 20 %","< 22 %","< 24 %","< 26 %", "≥ 26 %")
  )+
  theme_custom
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-5-1.png" width="672" />


``` r
# Set offset
# (value here is not fixed, it depends on your map, system of coordinates, etc...)
off <- 2000

# Create second centroid
cent_off <- cent_clean%>%
  # Extract longitude and latitude and add offset to longitude
  mutate(
    lon=st_coordinates(.)[,1]+off,
    lat=st_coordinates(.)[,2],
  )%>%
  # Drop old geometry
  st_drop_geometry()%>%
  # Make new geometry based on new {lon;lat}
  st_as_sf(coords=c('lon','lat'))%>%
  # Specify Coordinates Reference System
  st_set_crs(st_crs(cent_clean))

# Make map !
ggplot()+
  geom_sf(
    grd_clean%>%drop_na(), 
    mapping=aes(geometry=geometry,fill=clss)
  )+
  # Centroid for shaded effect
  geom_sf(cent_off,mapping=aes(geometry=geometry),color=alpha("black",0.5),size=0.5)+
  # 'Real' centroid
  geom_sf(cent_clean,mapping=aes(geometry=geometry,color=clss),size=0.5)+
  geom_sf(cent_clean,mapping=aes(geometry=geometry),fill=NA,pch=21,size=0.5)+
  labs(fill="Member of a sport association")+
  guides(
    color='none',
    fill=guide_legend(
      nrow=1,
      title.position="top",
      label.position="bottom"
    )
  )+
  scale_fill_manual(
    values=pal,
    label=c("< 18 %","< 20 %","< 22 %","< 24 %","< 26 %", "≥ 26 %")
  )+
  scale_color_manual(values=pal)+
  theme_custom
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-6-1.png" width="672" />

