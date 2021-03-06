# Reproducible Research: Peer Assessment 1


# 1. Loading and Preprocessing the Data

First I download the zip file containing the activity data and extract the data file from the zip archive.


```r
download.file("https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip", "activity.zip", method = "curl")
unzip("activity.zip")
```

This creates the file *activity.csv* in our local directory. I use **read.csv()** to read-in the file to a dataframe **activity**.


```r
activity <- read.csv("activity.csv")
```

I remove the files *activity.csv* and *activity.zip* since I no longer need them.


```r
file.remove(c("activity.csv", "activity.zip"))
```

## 2. Mean & Median Total Number of Steps Taken per Day

### 2.1. Total Number of Steps Taken per Day

To find the mean total number of steps per day, I first remove all rows with no data for **steps**.


```r
completedata <- activity[complete.cases(activity),]
```

Now I use **dplyr** to find the total number of steps taken per day.


```r
library(dplyr)
totalnum <- completedata   %>%
            group_by(date) %>%
            summarise(total = sum(steps))
```

I make a histogram of the total number of steps taken per day.


```r
hist(totalnum$total, xlab = "total # of steps per day", main = "Histogram of Total Steps per Day")
```

![](PA1_template_files/figure-html/unnamed-chunk-6-1.png) 

### 2.2. Mean Number of Steps Taken per Day


```r
mean1 <- round(mean(totalnum$total))
mean1
```

```
## [1] 10766
```

### 2.3. Median Number of Steps Taken per Day


```r
median1 <- median(totalnum$total)
median1
```

```
## [1] 10765
```

## 3. Average Daily Activity Patterns

### 3.1. Activity Pattern Time Series

To find the average activity patterns, I group the data by **interval** and plot a time-series.


```r
mean_interval <- completedata       %>%
                 group_by(interval) %>%
                 summarise(mean = mean(steps))
plot(mean_interval, type = "l", xlab = "interval", ylab = "mean # steps", 
     main = "Mean Number of Steps per Interval")
```

![](PA1_template_files/figure-html/unnamed-chunk-9-1.png) 

### 3.2. Maximum 5-Minute Interval


```r
by_mean <- arrange(mean_interval, -mean)[1, "interval"]
by_mean[1,1]
```

```
## Source: local data frame [1 x 1]
## 
##   interval
## 1      835
```

## 4. Imputing Missing Values

### 4.1 Number of Missing Values

I find the number of rows with missing data by taking the difference in the number of rows between the complete and original data.


```r
nrow(activity) - nrow(completedata)
```

```
## [1] 2304
```

### 4.2 Filling In Missing Values

I fill in any missing step data with the mean value for that particular interval.


```r
activity_complete <- activity
for (i in 1:nrow(activity)) {
  if (is.na(activity_complete[i,"steps"])) {
    activity_complete[i, "steps"] <- filter(mean_interval, 
                                            interval == activity_complete[i, "interval"])[1,"mean"]
  }
}
```

I then find the total number of steps taken per day with the new data and plot a histogram.


```r
totalnum2 <- activity_complete %>%
             group_by(date)    %>%
             summarise(total = sum(steps))
hist(totalnum2$total, xlab = "total # of steps per day", 
     main = "Histogram of Total Steps per Day (with missing values)")
```

![](PA1_template_files/figure-html/unnamed-chunk-13-1.png) 

### 4.3 Mean Number of Steps Taken per Day


```r
mean2 <- round(mean(totalnum2$total))
```

### 4.4 Median Number of Steps Taken per Day


```r
median2 <- median(totalnum2$total)
```

### 4.5 Analysis

Filling in the missing values has no impact on the mean. However, the two median values differ by 1.1886792.

## 5. Weekday/Weekend Differences

### 5.1. Day Type Factor

To find any differences between weekdays and weekends, I first add a new factor stating whether the given date is a weekday or a weekend.


```r
activity_complete$day <- factor(x = weekdays.Date(as.Date(activity_complete$date)) %in% c("Saturday", "Sunday"), 
                                labels = c("weekday", "weekend"))
```

### 5.2. Time Series Plot

I then use **ggplot2** plot the average number of steps taken per interval for both weekdays and weekends.


```r
library(ggplot2)
mean_interval_weekend <- activity_complete %>%
                         group_by(interval, day) %>%
                         summarise(mean = mean(steps))
ggplot(mean_interval_weekend, aes(interval, mean)) + geom_line() + facet_grid(day ~ .) + 
  xlab("Interval") + ylab("Number of steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-17-1.png) 
