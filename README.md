Homework 3
================
Christine Hou
2022-10-13

``` r
library(tidyverse)
```

    ## ── Attaching packages ─────────────────────────────────────── tidyverse 1.3.2 ──
    ## ✔ ggplot2 3.3.6      ✔ purrr   0.3.4 
    ## ✔ tibble  3.1.8      ✔ dplyr   1.0.10
    ## ✔ tidyr   1.2.1      ✔ stringr 1.4.1 
    ## ✔ readr   2.1.2      ✔ forcats 0.5.2 
    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter() masks stats::filter()
    ## ✖ dplyr::lag()    masks stats::lag()

``` r
library(ggplot2)
library(dbplyr)
```

    ## 
    ## Attaching package: 'dbplyr'
    ## 
    ## The following objects are masked from 'package:dplyr':
    ## 
    ##     ident, sql

``` r
library(nycflights13)
```

#### 1. Compute the average delay by destination, then join on the airports data frame so you can show the spatial distribution of delays. Here’s an easy way to draw a map of the United States:

``` r
airports %>%
  semi_join(flights, c("faa" = "dest")) %>%
  ggplot(aes(lon, lat)) +
    borders("state") +
    geom_point() +
    coord_quickmap()
```

![](README_files/figure-gfm/unnamed-chunk-2-1.png)<!-- -->

#### (Don’t worry if you don’t understand what semi_join() does — you’ll learn about it next.)

#### You might want to use the size or colour of the points to display the average delay for each airport.

``` r
flights %>%
  group_by(dest) %>%
  summarize(avg_delay = mean(arr_delay, na.rm = TRUE)) %>%
  left_join(airports, by = c("dest" = "faa")) %>%
  ggplot(aes(lon, lat, size = avg_delay, color = avg_delay)) +
  borders("state") +
  geom_point() +
  coord_quickmap()
```

    Warning: Removed 5 rows containing missing values (geom_point).

![](README_files/figure-gfm/unnamed-chunk-3-1.png)<!-- -->

#### 2. Add the location of the origin and destination (i.e. the lat and lon) to flights.

``` r
flights %>%
  left_join(airports, by = c("dest" = "faa")) %>%
  left_join(airports, by = c("origin" = "faa"),suffix = c('.dest', '.origin')) %>%
  select(dest, origin, contains("lat"), contains("lon"))
```

    # A tibble: 336,776 × 6
       dest  origin lat.dest lat.origin lon.dest lon.origin
       <chr> <chr>     <dbl>      <dbl>    <dbl>      <dbl>
     1 IAH   EWR        30.0       40.7    -95.3      -74.2
     2 IAH   LGA        30.0       40.8    -95.3      -73.9
     3 MIA   JFK        25.8       40.6    -80.3      -73.8
     4 BQN   JFK        NA         40.6     NA        -73.8
     5 ATL   LGA        33.6       40.8    -84.4      -73.9
     6 ORD   EWR        42.0       40.7    -87.9      -74.2
     7 FLL   EWR        26.1       40.7    -80.2      -74.2
     8 IAD   LGA        38.9       40.8    -77.5      -73.9
     9 MCO   JFK        28.4       40.6    -81.3      -73.8
    10 ORD   LGA        42.0       40.8    -87.9      -73.9
    # … with 336,766 more rows

#### 3. Is there a relationship between the age of a plane and its delays?

``` r
flights %>%
  group_by(tailnum) %>%
  summarize(avg_arr_delay = mean(arr_delay, na.rm = TRUE),
            avg_dep_delay = mean(dep_delay, na.rm = TRUE)) %>%
  pivot_longer(c(avg_arr_delay,avg_dep_delay), names_to = "mode", values_to = "delay") %>%
  left_join(planes, by = "tailnum") %>%
  ggplot(mapping = aes(x = year, y = delay)) +
  geom_point() +
  geom_smooth(se = FALSE) +
  facet_wrap(vars(mode))
```

    `geom_smooth()` using method = 'gam' and formula 'y ~ s(x, bs = "cs")'

    Warning: Removed 1596 rows containing non-finite values (stat_smooth).

    Warning: Removed 1596 rows containing missing values (geom_point).

![](README_files/figure-gfm/unnamed-chunk-5-1.png)<!-- -->
