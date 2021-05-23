---
title: "basicstatistics_scatterplot"
author: "yi-hui-wang"
date: "May 22, 2021"
output: 
  html_document: 
    keep_md: yes
---


## Step 1: Exploring the data


```r
library(dplyr)
```

```
## Warning: package 'dplyr' was built under R version 4.0.5
```

```
## 
## Attaching package: 'dplyr'
```

```
## The following objects are masked from 'package:stats':
## 
##     filter, lag
```

```
## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, setequal, union
```

```r
library(ggplot2)
```

```
## Warning: package 'ggplot2' was built under R version 4.0.5
```

```r
### load and process actual fantasy points
# created by webscrapping_weeklyactualpts.py
obspt <- read.csv('D:/fantasyfootball/data2020/2020_weekly_actualpt.csv')

str(obspt)
```

```
## 'data.frame':	724 obs. of  21 variables:
##  $ X       : int  0 1 2 3 4 5 6 7 8 9 ...
##  $ Player  : chr  "Davante Adams" "Josh Jacobs" "Calvin Ridley" "Russell Wilson" ...
##  $ Team    : chr  "GB" "LV" "ATL" "SEA" ...
##  $ Position: chr  "WR" "RB" "WR" "QB" ...
##  $ Week1   : num  41.6 35.9 33.9 31.8 31 30.8 29.1 28.4 28.2 28.2 ...
##  $ Week2   : num  6.6 13.5 29.9 34.4 6.1 19.2 20.8 24.8 34.5 6.3 ...
##  $ Week3   : num  NA 9.3 16.7 36.8 12.3 24.5 23.7 NA 32.2 6.3 ...
##  $ Week4   : num  NA 10.3 0 21.9 26.2 29.6 11.1 NA 25.4 7.8 ...
##  $ Week5   : num  NA 22.5 21.6 25.5 29.3 NA 25.1 NA 18.3 20.9 ...
##  $ Week6   : num  12.1 NA 18.9 NA 14.1 5.8 9.3 NA 16.1 11.8 ...
##  $ Week7   : num  44.6 6.1 19.9 32.9 NA 27.3 24.3 NA 16.4 4.3 ...
##  $ Week8   : num  30.3 12.9 7.2 28.7 5.7 22.5 NA NA 13.5 10.6 ...
##  $ Week9   : num  33.3 13.8 NA 24.1 5.8 28.9 6 37.1 36 1.8 ...
##  $ Week10  : num  18.6 29.6 NA 11.9 20.3 26.4 25.7 NA 29.4 14.3 ...
##  $ Week11  : num  23.6 13.4 14 20.1 32.3 22.7 10.1 NA NA NA ...
##  $ Week12  : num  18.1 5.4 17 14.4 NA 25.6 10.5 NA 16.5 0 ...
##  $ Week13  : num  34.1 NA 15.8 16 21.5 23.5 19.2 NA 30.1 2.4 ...
##  $ Week14  : num  24.5 10.4 26.4 23.1 6.9 30.9 22.6 NA 19.3 6.1 ...
##  $ Week15  : num  11.2 20.4 32.3 13 9.1 18.3 30 NA 37.7 11.4 ...
##  $ Week16  : num  43.2 6.9 17.3 19.9 23.7 26.1 12.8 NA 32.3 4.6 ...
##  $ Week17  : num  16.6 20.9 10.6 18.1 9.7 26 7.5 NA 20.3 4.2 ...
```

```r
# remove X column
obspt <- obspt %>% dplyr::select(-X)
```

## Step 2: Calculating statistics

```r
obspt.stats <- obspt %>%
  
  # compute a row-at-a-time
  rowwise() %>%
  
  # add a mean column
  mutate(mean = mean(c_across(Week1:Week17), na.rm = TRUE)) %>%
  
  # add a sd column
  mutate(sd = sd(c_across(Week1:Week17), na.rm = TRUE)) %>%
  
  # add a CV colume
  mutate(cv = sd/mean)

head(obspt.stats[,c("Player","mean","sd","cv")])
```

```
## # A tibble: 6 x 4
## # Rowwise: 
##   Player          mean    sd    cv
##   <chr>          <dbl> <dbl> <dbl>
## 1 Davante Adams   25.6 12.5  0.487
## 2 Josh Jacobs     15.4  8.83 0.572
## 3 Calvin Ridley   18.8  9.25 0.493
## 4 Russell Wilson  23.3  7.85 0.337
## 5 Adam Thielen    16.9  9.86 0.583
## 6 Aaron Rodgers   24.3  6.15 0.253
```

## Plot statistics 
We can subset wide receivers and plot their mean scores on x-axis and standard deviation on y-axis.


```r
obspt.stats.wr <- obspt.stats %>% filter(Position == 'WR')

ggplot(obspt.stats.wr, aes(x = mean, y = sd)) +
  geom_point(color = "red") +
  
  # add straight lines with known slope
  geom_abline(slope = 0.5) +
  geom_abline(slope = 1) +
  geom_abline(slope = 2)
```

```
## Warning: Removed 19 rows containing missing values (geom_point).
```

![](MeanSDCV_scatterplot_files/figure-html/unnamed-chunk-3-1.png)<!-- -->



