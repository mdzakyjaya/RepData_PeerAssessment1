---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
editor_options: 
  chunk_output_type: console
---



## Loading and preprocessing the data

unzip and load data


```r
unzip('activity.zip')
data <- read.csv('activity.csv')
```

input data as data table


```r
data <- data.table::as.data.table(data)
```

## What is mean total number of steps taken per day?

1.  Total number of steps taken per day


```r
TotalEachDay <- data %>% group_by(date) %>% summarise(sum = sum(steps, na.rm = T))
```

```
## `summarise()` ungrouping output (override with `.groups` argument)
```

```r
TotalEachDay
```

```
## # A tibble: 61 x 2
##    date         sum
##    <chr>      <int>
##  1 2012-10-01     0
##  2 2012-10-02   126
##  3 2012-10-03 11352
##  4 2012-10-04 12116
##  5 2012-10-05 13294
##  6 2012-10-06 15420
##  7 2012-10-07 11015
##  8 2012-10-08     0
##  9 2012-10-09 12811
## 10 2012-10-10  9900
## # ... with 51 more rows
```

2.  histogram of the total number of steps taken each day


```r
ggplot(TotalEachDay) + geom_histogram(aes(sum),
                                      bins = 8, 
                                      fill = 'blue',
                                      col = 'black',
                                      ) +
    xlab('Steps Each Daya') + ylab('freq')
```

![](PA1_template_files/figure-html/unnamed-chunk-4-1.png)<!-- -->

3.  the mean and meadian of the total number of steps taken per day


```r
mean <- mean(TotalEachDay$sum)
median <- median(TotalEachDay$sum)
paste('the mean is',mean, 'and the median is', median)
```

```
## [1] "the mean is 9354.22950819672 and the median is 10395"
```

## What is the average daily activity pattern?

1.  time series plot of the 5-minute interval and average steps taken


```r
AvgPerInt <- data %>% group_by(interval) %>% 
                summarise(avg = mean(steps, na.rm = T))
```

```
## `summarise()` ungrouping output (override with `.groups` argument)
```

```r
ggplot(AvgPerInt) + geom_line(aes(x = interval, y = avg)) +
    ylab('average accross all days')
```

![](PA1_template_files/figure-html/unnamed-chunk-6-1.png)<!-- -->

2.  interval with the maximum number of steps on average across all days


```r
AvgPerInt[AvgPerInt$avg == max(AvgPerInt$avg),]
```

```
## # A tibble: 1 x 2
##   interval   avg
##      <int> <dbl>
## 1      835  206.
```

## Imputing missing values

1.  total number of missing values


```r
sum(is.na(data$steps))
```

```
## [1] 2304
```

```r
# alternative code
nrow(data[is.na(steps),])
```

```
## [1] 2304
```

2.  dealing with missing values


```r
# fill missing values in steps with mean
avgsteps <- mean(data$steps, na.rm = T)
data$steps <- replace(data$steps, is.na(data$steps), avgsteps)
```

3.  create new data set with missing values filled in


```r
data.table::fwrite(x = data, file = 'tidydata.csv', quote = F)
```

4.\
histogram for total of steps taken each day using tidied data


```r
TotalTidied <- data %>% group_by(date) %>% summarise(total = sum(steps))
```

```
## `summarise()` ungrouping output (override with `.groups` argument)
```

```r
ggplot(TotalTidied) + geom_histogram(aes(total),
                                     col = 'black',
                                     fill = 'blue',
                                     bins = 8) +
    xlab('total steps each day') + ylab('freq')
```

![](PA1_template_files/figure-html/unnamed-chunk-11-1.png)<!-- -->

the mean and median in tidied data


```r
avgtidied <- mean(TotalTidied$total)
medtidied <- median(TotalTidied$total)

paste('the mean is', avgtidied, 'and the median is', medtidied)
```

```
## [1] "the mean is 10766.1886792453 and the median is 10766.1886792453"
```

| dealing with null values | mean  | median |
|--------------------------|-------|--------|
| before                   | 9354  | 10395  |
| after                    | 10766 | 10766  |

## Are there differences in activity patterns between weekdays and weekends?
1. categorized weekend and weekdays

```r
# formating the date column as date type
data[, date := as.POSIXct(date, format = '%Y-%m-%d')]

# create new column with day name
data[, `day of week` := weekdays(date)]

# idetify weekday and weekend
data[grepl(pattern = 'Senin|Selasa|Rabu|Kamis|Jumat', x = `day of week`),
     'DayType'] <- 'weekday'
data[grepl(pattern = 'Sabtu|Minggu', x = `day of week`),
     'DayType' ] <- 'weekend'

data$DayType <- as.factor(data$DayType)
```
2. time series plot

```r
AvgIntTidied <- data %>% group_by(interval, DayType) %>% summarise(avg = mean(steps))
```

```
## `summarise()` regrouping output by 'interval' (override with `.groups` argument)
```

```r
ggplot(AvgIntTidied) + geom_line(aes(x = interval, y = avg)) +
    facet_wrap(~DayType) + ylab('avergae across all days')
```

![](PA1_template_files/figure-html/unnamed-chunk-14-1.png)<!-- -->
