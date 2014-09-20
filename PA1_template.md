# Coursera-PeerAssignment01
Dasapta Erwin Irawan  
15/09/2014  

This is my peer assignment 1 for Coursera: Reproducible Research. This document is my own work, based on several examples of workflow that I found in the Rpubs and StackOverflow site. 

# 1. Loading libraries and data

Here I am using `dplyr` and `ggplot2` from CRAN.


```r
require(dplyr)
```

```
## Loading required package: dplyr
## 
## Attaching package: 'dplyr'
## 
## The following objects are masked from 'package:stats':
## 
##     filter, lag
## 
## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, setequal, union
```

```r
require(ggplot2)
```

```
## Loading required package: ggplot2
```

```r
data <- read.csv("activity.csv", 
                   header = TRUE, 
                   colClasses = c("numeric", "Date", "numeric"))
str(data)
```

```
## 'data.frame':	17568 obs. of  3 variables:
##  $ steps   : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ date    : Date, format: "2012-10-01" "2012-10-01" ...
##  $ interval: num  0 5 10 15 20 25 30 35 40 45 ...
```

# 2. What is mean total number of steps taken per day?

The steps are:

+ Creating a new data frame
+ Making a histogram
+ Calculating mean and median


```r
# New data frame
actData <- data %.% 
              group_by(date) %.% 
              summarise(Steps = sum(steps))

# Histogram: total steps/day
ggplot(data = actData, aes(x = Steps)) + 
        geom_histogram(fill = "green", colour = "black") + 
        scale_x_continuous("Total steps/day") + 
        scale_y_continuous("Freq") + 
        ggtitle("Total steps/day")
```

```
## stat_bin: binwidth defaulted to range/30. Use 'binwidth = x' to adjust this.
```

![plot of chunk unnamed-chunk-2](./PA1_template_files/figure-html/unnamed-chunk-2.png) 

```r
# Mean and median: total steps/day
mean(actData$Steps, na.rm = TRUE)      
```

```
## [1] 10766
```

```r
median(actData$Steps, na.rm = TRUE)
```

```
## [1] 10765
```

# 3. What is the average daily activity pattern?

The steps are:

+ Setting the interval (day)
+ Making time series plot 


```r
## Set interval (day)
actInt <- data %.% 
            group_by(interval) %.% 
            summarise(meanSteps = mean(steps, 
            na.rm = TRUE))

## Plotting time series: steps/interval
ggplot(data = actInt, aes(x = interval, y = meanSteps)) + 
        geom_line(colour = "blue") + 
        scale_x_continuous("Interval", 
        breaks = seq(min(actInt$interval), 
        max(actInt$interval), 100)) + 
        scale_y_continuous("Average Steps") + 
        ggtitle("Average Steps/Interval")
```

![plot of chunk unnamed-chunk-3](./PA1_template_files/figure-html/unnamed-chunk-3.png) 

# 4. Imputing missing values 

The steps are:

+ Calculating missing values (NAs)
+ Imputing using mean in new column
+ Making a new data frame with no NAs
+ Making histogram of new data frame 
+ Calculating new mean and median


```r
# Calculate sum of NAs 
sum(is.na(data$steps))    
```

```
## [1] 2304
```

```r
# Fill using mean
## Merging original activity data (data) with the average/interval data (actInt) to data2
data2 <- data %.% left_join(actInt, by = "interval")

## Imputing missing data with mean value in new column
data2$fillSteps <- ifelse(is.na(data2$steps), data2$meanSteps, data2$steps)

## Rearranging the columns
data2$steps <- NULL
data2$meanSteps <- NULL
colnames(data2) <- c("date", "interval", "steps")
data2 <- data2[, c(3, 1, 2)]
head(data2)
```

```
##     steps       date interval
## 1 1.71698 2012-10-01        0
## 2 0.33962 2012-10-01        5
## 3 0.13208 2012-10-01       10
## 4 0.15094 2012-10-01       15
## 5 0.07547 2012-10-01       20
## 6 2.09434 2012-10-01       25
```

```r
## Create a new data frame: steps/day 
actData2 <- data2 %.% 
              group_by(date) %.% 
              summarise(Steps = sum(steps))

ggplot(data = actData2, aes(x = Steps)) + 
  geom_histogram(fill = "green", colour = "black") + 
  scale_x_continuous("Steps/Day") + 
  scale_y_continuous("Freq") + 
  ggtitle("Total steps/day: After imputations")
```

```
## stat_bin: binwidth defaulted to range/30. Use 'binwidth = x' to adjust this.
```

![plot of chunk unnamed-chunk-4](./PA1_template_files/figure-html/unnamed-chunk-4.png) 

```r
## Calculating the new mean and median
mean(actData2$Steps)
```

```
## [1] 10766
```

```r
median(actData2$Steps)
```

```
## [1] 10766
```

# 3. Are there differences in activity patterns between weekdays and weekends?

The steps are:

+ Making weekday and weekend classification
+ Creating new data frame with classification
+ Making panel plot
+ Evaluating the pattern

Result:
+ subject made more steps in weekend than in weekday.
+ subject made more steps in early hours of a day than in late hours.


```r
## Subsetting weekday and weekend
data2$weekdayType <- ifelse(weekdays(actData$date) %in% 
                                c("Saturday", "Sunday"), 
                                "weekend", "weekday")
head(data2)
```

```
##     steps       date interval weekdayType
## 1 1.71698 2012-10-01        0     weekday
## 2 0.33962 2012-10-01        5     weekday
## 3 0.13208 2012-10-01       10     weekday
## 4 0.15094 2012-10-01       15     weekday
## 5 0.07547 2012-10-01       20     weekday
## 6 2.09434 2012-10-01       25     weekend
```

```r
## Creating a new data frame: average steps/interval (actInt2)
actInt2 <- data2 %.% 
            group_by(interval, weekdayType) %.% 
            summarise(meanSteps = mean(steps, na.rm = TRUE))

## Creating panel plot
ggplot(data = actInt2, aes(x = interval, y = meanSteps)) + 
              geom_line() + 
              facet_grid(weekdayType ~ .) + 
              scale_x_continuous("Interval (day)", 
              breaks = seq(min(actInt2$interval), 
              max(actInt2$interval), 100)) + 
              scale_y_continuous("Average Steps") + 
              ggtitle("Average Steps/Interval")
```

![plot of chunk unnamed-chunk-5](./PA1_template_files/figure-html/unnamed-chunk-5.png) 
