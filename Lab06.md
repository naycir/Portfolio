Lab 06 Notebook
================
Nigarhan Gurpinar
(March 12, 2018)

Introduction
------------

This is my 6th Lab assignment's R Notebook for the course SOC 5650: Introduction to GISc!

Project Set Up
--------------

``` r
knitr::opts_knit$set(root.dir = here::here())
```

Load Dependencies
-----------------

``` r
library(here) # working directory
```

    ## here() starts at F:/GitHub/Labs/Lab-06

``` r
library(sf)#reading shapfiles
```

    ## Linking to GEOS 3.6.1, GDAL 2.2.0, proj.4 4.9.3

``` r
library(dplyr)#data wrangling
```

    ## 
    ## Attaching package: 'dplyr'

    ## The following objects are masked from 'package:stats':
    ## 
    ##     filter, lag

    ## The following objects are masked from 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union

``` r
library(lubridate) #dates
```

    ## 
    ## Attaching package: 'lubridate'

    ## The following object is masked from 'package:here':
    ## 
    ##     here

    ## The following object is masked from 'package:base':
    ## 
    ##     date

``` r
library(stringr) #strings
```

Load Data
---------

The following code loads the data package and assigns our data to a data frame in our global environment:

``` r
data <- st_read("data/rawData/METRO_WX_Hail.shp", stringsAsFactors = FALSE)
```

    ## Reading layer `METRO_WX_Hail' from data source `F:\GitHub\Labs\Lab-06\data\rawData\METRO_WX_Hail.shp' using driver `ESRI Shapefile'
    ## Simple feature collection with 2155 features and 22 fields
    ## geometry type:  LINESTRING
    ## dimension:      XY
    ## bbox:           xmin: 643088 ymin: 4214150 xmax: 835220.7 ymax: 4345017
    ## epsg (SRID):    26915
    ## proj4string:    +proj=utm +zone=15 +datum=NAD83 +units=m +no_defs

Part 3
------

### Question 10

Removing yr, mo and dy variables from the shapefile using the select function

``` r
newData <- select(data, -yr, -mo, -dy)
```

### Question 11

Formatting dates in a pipeline using mutate function.

``` r
newData %>%
  mutate(date = ymd(date))%>%
  mutate(year = year(date))%>%
  mutate(month = month(date))%>%
  mutate(day = date(date)) -> newData
```

### Question 12

The following code edits st's observations MO and IL.

``` r
newData %>%
  mutate(st = case_when (st == "MO" ~ "State of Missouri",
                        st == "IL" ~ "State of Illinois"))->newData
```

### Question 13

The following code in a pipeline

``` r
newData %>%
  mutate(st = str_replace(st, "[;]", "")) %>%
  mutate(st = str_replace(st, "Missouri", "MO")) %>% 
  mutate(st = str_replace(st, "Illinois", "IL")) %>%
  mutate(st = str_to_upper(st)) %>%
  mutate(stAbbrev = ifelse(word(st, 3) == "MO", "MO", "IL")) %>%
  mutate(illinois = ifelse(str_detect(stAbbrev, "IL"), TRUE, FALSE)) %>%
  mutate(missouri = ifelse(str_detect(stAbbrev, "MO"), TRUE, FALSE)) -> newData1
```

### Question 14

``` r
pre2000Hail <- filter(newData1, year < 2000)
post2000Hail <- filter(newData1, year >= 2000)
```

### Question 15

Saving these data subsets into shapefiles. First we need to convert logical data as characters.

``` r
pre2000Hail %>%
  mutate(illinois = as.character(illinois)) %>%
  mutate(missouri = as.character(missouri)) -> pre2000Hail
post2000Hail %>%
  mutate(illinois = as.character(illinois)) %>%
  mutate(missouri = as.character(missouri)) -> post2000Hail
```

Now saving the data as shapefiles.(\*\*st\_write did not work for me so i had to find another function on the internet)

``` r
write_sf(pre2000Hail,"data/cleanedData/pre2000Hail.shp")
write_sf(post2000Hail,"data/cleanedData/post200Hail.shp")
```
