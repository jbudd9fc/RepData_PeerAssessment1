---
output:
  html_document: default
  keep_md: TRUE
  word_document: default
  pdf_document: default
---
# Reproducible Research Course Project 1
## By John S. Budd

### Loading and preprocessing the data 
The first component of this analysis focuses on unzipping the 'activity.zip' 
file and reading the dataset into R, creating a dataframe called 'active_set'
to support answering the questions that follow.


```r
## Gets the file from the url and unzips it to the working directory
url <- "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip"
temp <- tempfile()
download.file(url, temp)
Unzipdir <- unzip(temp)
## Creates a dataframe using the read.csv of the activity file
active_set <- read.csv("activity.csv")
## Converts the dates into a date format
active_set$date <- as.Date(active_set$date)
```

### What is the mean total number of steps taken per day?
During this next code chunk, the code will do the following:  

1. Calculate the total number of steps taken per day
The code chunk below loads the dplyr package and calculates thessteps taken per
day


```r
## loads the dplyr package
library(dplyr)
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
## finds the sum of each day
sum_set <- active_set %>%
  group_by(date) %>%
  summarise(steps = sum(steps))
```

```
## `summarise()` ungrouping output (override with `.groups` argument)
```
2. Make a histogram of the total number of steps taken each day

```r
hist(sum_set$steps, xlab = "Number of Steps", ylab = "Number of days")
```

![plot of chunk make_hist](figure/make_hist-1.png)
3. Calculate and report the mean and median of the total number of steps taken 
per day

```r
## Finds the mean, ignoring NAs
mean(sum_set$steps, na.rm = TRUE)
```

```
## [1] 10766.19
```

```r
## Finds the median, ignoring NAs
median(sum_set$steps, na.rm = TRUE)
```

```
## [1] 10765
```

### What is the average daily activity pattern? 
During this next code chunk, the code will do the following

1. Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis)
and the average number of steps taken averaged across all days (y-axis)

```r
## creates a tibble with the average of each five minute interval ignoring NAs
avg_set <- active_set %>%
  group_by(interval) %>%
  summarise(steps = mean(steps, na.rm = TRUE))
```

```
## `summarise()` ungrouping output (override with `.groups` argument)
```

```r
## plots the avg_set 
plot(avg_set$interval,avg_set$steps, type = "l", xlab = "5-minute interval", 
     ylab = "# of Steps")
```

![plot of chunk time_series_plot](figure/time_series_plot-1.png)

2. Which 5-minute interval, on average across all days in the dataset, contains
the maximum number of steps?

```r
avg_set[which.max(avg_set$steps),]
```

```
## Warning: `...` is not empty.
## 
## We detected these problematic arguments:
## * `needs_dots`
## 
## These dots only exist to allow future extensions and should be empty.
## Did you misspecify an argument?
```

```
## # A tibble: 1 x 2
##   interval steps
##      <int> <dbl>
## 1      835  206.
```

### Imputing missing values
During this next code chunk, the code will do the following:

1. Calculate and report the total number of missing values in the dataset 
(i.e. the total number of rows with NAs)

```r
## provides an overall summary of the dataset including the number of NAs
summary(active_set)
```

```
##      steps             date               interval     
##  Min.   :  0.00   Min.   :2012-10-01   Min.   :   0.0  
##  1st Qu.:  0.00   1st Qu.:2012-10-16   1st Qu.: 588.8  
##  Median :  0.00   Median :2012-10-31   Median :1177.5  
##  Mean   : 37.38   Mean   :2012-10-31   Mean   :1177.5  
##  3rd Qu.: 12.00   3rd Qu.:2012-11-15   3rd Qu.:1766.2  
##  Max.   :806.00   Max.   :2012-11-30   Max.   :2355.0  
##  NA's   :2304
```

2. Devise a strategy for filling in all of the missing values in the dataset. 
The strategy does not need to be sophisticated. For example, you could use the
mean/median for that day, or the mean for that 5-minute interval, etc.
3. Create a new dataset that is equal to the original dataset but with the missing data filled in

The code chunk below addresses both questions above:

```r
## finds the mean for each interval
avg_set <- active_set %>%
  group_by(interval) %>%
  summarise(steps = mean(steps, na.rm = TRUE))
```

```
## `summarise()` ungrouping output (override with `.groups` argument)
```

```r
## uses the transform function to identify areas where there is NA in the 
## active_set and replaces it with the average for the five-minute interval
## and creates a new_imputeset

new_imputeset <- transform(active_set,
                              steps = ifelse(is.na(active_set$steps),
                                             avg_set$steps[match(active_set$interval, x =  avg_set$interval)],
                                             active_set$steps))
```

4. Make a histogram of the total number of steps taken each day:

```r
## finds the sum of each day
impsum_set <- new_imputeset %>%
  group_by(date) %>%
  summarise(steps = sum(steps))
```

```
## `summarise()` ungrouping output (override with `.groups` argument)
```
2. Make a histogram of the total number of steps taken each day

```r
hist(impsum_set$steps, xlab = "Number of Steps", ylab = "Number of days")
```

![plot of chunk make_hist2](figure/make_hist2-1.png)

Calculate and report the mean and median total number of steps taken per day:

```r
## provides an overall summary of the dataset including the number of NAs
summary(impsum_set)
```

```
##       date                steps      
##  Min.   :2012-10-01   Min.   :   41  
##  1st Qu.:2012-10-16   1st Qu.: 9819  
##  Median :2012-10-31   Median :10766  
##  Mean   :2012-10-31   Mean   :10766  
##  3rd Qu.:2012-11-15   3rd Qu.:12811  
##  Max.   :2012-11-30   Max.   :21194
```

Do these values differ from the estimates from the first part of the assignment? 

```r
## runs the summary of the non-imputed set for comparison
summary(sum_set)
```

```
##       date                steps      
##  Min.   :2012-10-01   Min.   :   41  
##  1st Qu.:2012-10-16   1st Qu.: 8841  
##  Median :2012-10-31   Median :10765  
##  Mean   :2012-10-31   Mean   :10766  
##  3rd Qu.:2012-11-15   3rd Qu.:13294  
##  Max.   :2012-11-30   Max.   :21194  
##                       NA's   :8
```
What is the impact of imputing missing data on the estimates of the total daily 
number of steps:

The overall impact appears to be minimal with the mean and median staying close
and the third quartile moving slightly higher. 

Are there differences in activity patterns between weekdays and weekends?
For this part the weekdays()|weekdays() function may be of some help here.
Use the dataset with the filled-in missing values for this part.


```r
### adds a column with the day of week to the imputed set and a column that 
### identifies whether or not it is a weekend
new_imputeset$weekday <- weekdays(new_imputeset$date)
new_imputeset <- cbind(new_imputeset, 
                      daytype=ifelse(new_imputeset$weekday == "Saturday" | 
                                     new_imputeset$weekday == "Sunday", "weekend"
                                     ,"weekday"))
```

Make a panel plot containing a time series plot of the 5-minute interval (x-axis) 
and the average number of steps taken, averaged across all weekday days or 
weekend days (y-axis). See the README file in the GitHub repository to see an 
example of what this plot should look like using simulated data.


```r
## creates two subsets, one for weekends and one for weekdays
weekend_set <- subset(new_imputeset, new_imputeset$daytype == "weekend")
weekday_set <- subset(new_imputeset, new_imputeset$daytype == "weekday")

## Summarizes each set into the average by interval
avg_weset <- weekend_set %>%
  group_by(interval) %>%
  summarise(steps = mean(steps, na.rm = TRUE))
```

```
## `summarise()` ungrouping output (override with `.groups` argument)
```

```r
avg_wdset <- weekday_set %>%
  group_by(interval) %>%
  summarise(steps = mean(steps, na.rm = TRUE))
```

```
## `summarise()` ungrouping output (override with `.groups` argument)
```

```r
## creates a two panel plot
par(mfrow=c(2,1))
plot(avg_weset$interval,avg_weset$steps, type = "l", ylab="avg weekend steps",
     xlab = "interval")
plot(avg_wdset$interval,avg_weset$steps, type = "l", ylab="avg weekday steps",
     xlab = "interval")
```

![plot of chunk unnamed-chunk-4](figure/unnamed-chunk-4-1.png)




