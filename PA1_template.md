---
title: "Reproducible Research Assignment 1"
author: "Ong Poh Soon"
date: "6 January 2020"
output:
        html_document:
                keep_md: true
---

This is peer-graded assignment of Reproducible Research course. The assignment requires using Rmarkdown to document a series of tasks about data collected on individual's number of steps taken in 5-minute intervals each day during the months of October and November 2012.  

The data collected was stored in file **"activity.csv"**

There are 3 variables in the data:  
1. steps - number of steps taken in a 5-minute intervals; missing values are coded as **"NA"**.  
2. date - the date on which each measurement was taken.  
3. interval - identifier for the 5-minute interval.  

The data is read and dates are transformed; thereafter, total number of steps by day (StepsByDay) are calculated and dataset "datByDay" is created; *missing values are ignored*.

```r
library(dplyr)
library(lubridate)
dat <- read.csv("activity.csv")
dat$date <- ymd(dat$date)
datByDay <- dat %>% group_by(date) %>% summarise(StepsByDay = sum(steps, na.rm=TRUE))
```

Histogram of total number of steps taken each day (x=date, y=StepsByDay) shows *potential 8 missing days*, include 1 October and 30 November 2012.

```r
library("ggplot2")
g <- ggplot(data=datByDay, aes(x=date, y=StepsByDay), color="blue")
g + geom_bar(stat = "identity", fill="blue") + 
        labs(x="Date", y="Number of Steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-2-1.png)<!-- -->

The mean and median of the total number of steps taken per day are 9,354.23 and 10,395 respectively.

```r
Mean <- mean(datByDay$StepsByDay)
Mean
```

```
## [1] 9354.23
```

```r
Median <- median(datByDay$StepsByDay)
Median
```

```
## [1] 10395
```

Mean number of steps by interval (StepsMean) is calculated to plot a time series. The plot shows a **peak mean at 5-minute interval 835**.

```r
datByInterval <- dat %>% group_by(interval) %>% 
        summarise(StepsMean = mean(steps, na.rm=TRUE))
with(datByInterval, plot(interval, StepsMean, type="l", xlab="Interval", ylab="Mean Number of Steps"))
```

![](PA1_template_files/figure-html/unnamed-chunk-4-1.png)<!-- -->

```r
datByInterval$interval[which.max(datByInterval$StepsMean)]
```

```
## [1] 835
```

The presence of missing days may introduce bias into some calculations or summaries of the data. A sum of missing days confirms there are 8 missing days. This corresponds with the first histogram which shows  *potential 8 missing days*. 

```r
naDay <- dat %>% group_by(date) %>% summarise(total = sum(steps))
sum(is.na(naDay$total))
```

```
## [1] 8
```

Mean value of 5-minute interval is used to impute that interval's missing values **(NA)**. The mean (intMean) is calculated and merged with orginal dataset (dat) to produce a **new dataset (datImpute)**; missing values in the new dataset are imputed. All steps data are stored in variable "imputeSteps"

```r
intGroup <- dat %>% group_by(interval) %>% summarise(intMean = mean(steps, na.rm=T))
datImpute <- merge(dat, intGroup, by="interval")
datImpute$imputeSteps <- ifelse(!is.na(datImpute$steps), datImpute$steps, datImpute$intMean)
```

Histogram of total number of steps taken each day with imputed dataset (x=date, y=imputeSteps) confirms no missing day.

```r
library("ggplot2")
g <- ggplot(data=datImpute, aes(x=date, y=imputeSteps), color="blue")
g + geom_bar(stat = "identity", fill="blue") + 
        labs(x="Date", y="Number of Steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-7-1.png)<!-- -->

Using the new dataset (datImpute), total number of steps on each day is calculated for further generation of mean and median number of steps per day. The mean and median are same at 10,766.19 which is **higher** than non-imputed mean and median of 9,354.23 and 10,395. Imputing missing values **increases** total daily number of steps.

```r
datImputeByDay <- datImpute %>% group_by(date) %>% summarise(totalSteps = sum(imputeSteps))
meanPerDay <- mean(datImputeByDay$totalSteps)
meanPerDay
```

```
## [1] 10766.19
```

```r
medianPerDay <- median(datImputeByDay$totalSteps)
medianPerDay
```

```
## [1] 10766.19
```

To analyse activities between weekends and weekdays, a dataset (datWkDayEnd) is duplicated from imputed dataset (datImpute) to facilitate the workings. The data is categorised into "weekend" and "weekday". Saturday and Sunday are considered "weekend" in this analysis.

```r
datWkDayEnd <- datImpute
datWkDayEnd$date <- as.POSIXct(datImputeByDay$date)
WkEnd <- c("Saturday", "Sunday")
datWkDayEnd$category <- ifelse(weekdays(datWkDayEnd$date) %in% WkEnd, "weekend", "weekday")
```

Average number of steps by inerval and category ("weekend", "weekday") is calculated to plot a time series of 5-minute interval. Except the peak number of steps for "weekday" around interval 835, "weekend" generally having higher number of steps from interval 500 to 2000.

```r
datWkDayEnd_mean <- datWkDayEnd %>% group_by(interval, category) %>% 
        summarise(Mean = mean(imputeSteps))
library(lattice)
xyplot(Mean ~ interval | category, data=datWkDayEnd_mean, type="l", layout=c(1,2), xlab="Interval", ylab="Number of Steps", abline=c(h=75, v=2000, lty=2))
```

![](PA1_template_files/figure-html/unnamed-chunk-10-1.png)<!-- -->



