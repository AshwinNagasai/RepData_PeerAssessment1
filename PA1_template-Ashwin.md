---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document: 
    keep_md: true
    fig_caption: true
---
## Loading and preprocessing the data

``` r
unzip("activity.zip")
data <- read.csv("activity.csv")
head (data)
```

```
##   steps       date interval
## 1    NA 2012-10-01        0
## 2    NA 2012-10-01        5
## 3    NA 2012-10-01       10
## 4    NA 2012-10-01       15
## 5    NA 2012-10-01       20
## 6    NA 2012-10-01       25
```

``` r
data <- transform (data, date = as.Date(date))
```

## What is mean total number of steps taken per day?


``` r
# Load the ggplot2 package
library (ggplot2)

# create a variable for total steps per day and name the date column

total_steps <- aggregate(data$steps, by = list(Date = data$date), FUN = "sum")

# calculate mean and median of total steps per day

mean_total <- mean(total_steps[,2], na.rm = TRUE)
median_total <- median(total_steps[,2], na.rm = TRUE)
```

The mean total number of steps taken per day is 1.0766189\times 10^{4}  
The median total number of steps taken per day is 10765


``` r
histogram <- hist(total_steps$x, col = "blue", breaks = 20,
     main = "Total number of steps taken each day",
     xlab = "Number of steps per day")
```

![](PA1_template-Ashwin_files/figure-html/histogram-1.png)<!-- -->

## What is the average daily activity pattern?


``` r
# Create a variable avg_daily
avg_daily <- aggregate(x=list(Steps = data$steps), 
                       by=list(Interval=data$interval), 
                       FUN="mean", na.rm=TRUE)
```


``` r
# Create a time series plot using ggplot function  

time_plot <- ggplot(data=avg_daily, aes(x= Interval, y= Steps)) +    
            geom_line() + xlab("5-minute interval") +
            ylab("average number of steps taken") 
time_plot
```

![](PA1_template-Ashwin_files/figure-html/timeseries plot-1.png)<!-- -->

# Find the interval with the maximum number of steps


``` r
# Find the maximum number of steps within the available intervals
max_steps <- which.max(avg_daily$Steps)
# Find the interval where the maximum steps occurs
max_interval <- avg_daily[max_steps,1]
```

The interval with the maximum number of steps is 835\
The maximum number of steps within the above interval is 104

## Inputing missing values


``` r
# Find the total number of missing values
total_NA <- sum(is.na(data$steps))
```

The number of missing values is 2304


``` r
# Create a new dataset
complete_data <- data
# Create a new variable which has location of all missing values
na_location <- which(is.na(complete_data$steps))
# Load the hmisc package
library(Hmisc)
```

```
## 
## Attaching package: 'Hmisc'
```

```
## The following objects are masked from 'package:base':
## 
##     format.pval, units
```

``` r
# Fill in the missing values with the mean value
complete_data$steps <- impute(complete_data$steps, fun = mean)
# Calculate the total steps, mean and median for the new dataset
Step_total <- aggregate(complete_data$steps, 
                           by = list(Date = complete_data$date), 
                           FUN = "sum")
Step_mean <- mean(Step_total[,2], na.rm = TRUE)
Step_median <- median(Step_total[,2], na.rm = TRUE)
mean_difference <- Step_mean - mean_total
median_difference <- Step_median - median_total
```

Create a histogram with the filled in data


``` r
new_histogram <- hist(Step_total$x, col = "green", breaks = 20,
     main = "Total number of steps taken each day (filled in data)",
     xlab = "Number of steps per day")
```

![](PA1_template-Ashwin_files/figure-html/new histogram-1.png)<!-- -->

``` r
new_histogram
```

```
## $breaks
##  [1]     0  1000  2000  3000  4000  5000  6000  7000  8000  9000 10000 11000
## [13] 12000 13000 14000 15000 16000 17000 18000 19000 20000 21000 22000
## 
## $counts
##  [1]  2  0  1  1  1  2  1  2  5  2 18  6  6  4  2  5  0  1  0  0  1  1
## 
## $density
##  [1] 3.278689e-05 0.000000e+00 1.639344e-05 1.639344e-05 1.639344e-05
##  [6] 3.278689e-05 1.639344e-05 3.278689e-05 8.196721e-05 3.278689e-05
## [11] 2.950820e-04 9.836066e-05 9.836066e-05 6.557377e-05 3.278689e-05
## [16] 8.196721e-05 0.000000e+00 1.639344e-05 0.000000e+00 0.000000e+00
## [21] 1.639344e-05 1.639344e-05
## 
## $mids
##  [1]   500  1500  2500  3500  4500  5500  6500  7500  8500  9500 10500 11500
## [13] 12500 13500 14500 15500 16500 17500 18500 19500 20500 21500
## 
## $xname
## [1] "Step_total$x"
## 
## $equidist
## [1] TRUE
## 
## attr(,"class")
## [1] "histogram"
```

The new mean total number of steps taken per day is 1.0766189\times 10^{4}  
The new total number of steps taken per day is 1.0766189\times 10^{4}     
There was no difference in mean, but a 1.1886792 change in median after filling the missing values.  

It can be concluded that there was no significant difference seen in the mean and median after filling in the data  

## Are there differences in activity patterns between weekdays and weekends?


``` r
# Define the weekdays in a new variable
weekdays <- c("Monday", "Tuesday", "Wednesday", "Thursday", "Friday")

# use the new variable to factor between weekday and weekend
complete_data$Day <- weekdays(complete_data$date)
complete_data$Type_of_Day <- factor(complete_data$Day %in% weekdays,
                                    levels = c(FALSE, TRUE),
                                    labels = c("weekend", "weekday"))
# Separate weekday and weekend activity data in 2 new variables 
weekday_activity <- complete_data[complete_data$Type_of_Day == "weekday",]
weekend_activity <- complete_data[complete_data$Type_of_Day == "weekend",]
# Calculate the total number of steps for both weekdays and weekends 
weekday_steps <- aggregate(x=list(Steps = weekday_activity$steps),
                           by = list(Type_of_Day = weekday_activity$Type_of_Day, 
                                     Interval = weekday_activity$interval),
                           FUN = "mean")
weekend_steps <- aggregate(x=list(Steps = weekend_activity$steps), 
                           by = list(Type_of_Day = weekend_activity$Type_of_Day,
                                     Interval = weekend_activity$interval), 
                           FUN = "mean")
# Make a new variable containing step data for both weekdays and weekends with intervals 
day_steps <- rbind (weekday_steps, weekend_steps)
```
Using a panel plot to compare the activity patterns between weekdays and weekends 


``` r
#Create a 2 panel plot comparing the activity patterns for weekdays and weekends
panel_plot <- ggplot(day_steps, aes(x = Interval, y = Steps)) + 
geom_line() + 
facet_grid(Type_of_Day ~ .) + 
xlab("5-minute intervals") + 
ylab("Average number of steps taken") + 
ggtitle("Weekdays and weekends activity patterns") 
panel_plot
```

![](PA1_template-Ashwin_files/figure-html/panel plot-1.png)<!-- -->

From the above plot, it can be observed that weekends have overall higher average activity, but there is an significant peak in activity during the mornings in weekdays
