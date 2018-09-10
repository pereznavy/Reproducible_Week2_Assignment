---
title: "Reproducible Research: Week 2 Assignment"
author: "PerezNavy"
date: "8/9/2018"
output: html_document
---

## Reproducible Research: Peer assessment 1

### INTRODUCTION
This document presents the results regarding the Peer asingment fo Reproductible research week 2.

This assignment makes use of data from a personal activity monitoring device. This device collects data at 5 minute intervals through out the day. The data consists of two months of data from an anonymous individual collected during the months of October and November, 2012 and include the number of steps taken in 5 minute intervals each day.

##### This document presents the answers for the following questions regarding the Activity Monitoring Data

1.- What is mean total number of steps taken per day?

2.- What is the average daily activity pattern?

3.-Are there differences in activity patterns between weekdays and weekends?

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

Firstly, get the packages you need

```{R}
library(dplyr)
```

```{R}
library(ggplot2)
```

Load the data into your environment using read.csv()

```{R }
activity <- read.csv("~/Documents/activity.csv")
```

Look at your data

```{R }
str(activity)
```


```{R }
summary(activity)
```

```{R }
head(activity)
```

Remove the missing values

```{R }
act.complete <- na.omit(activity)
```

Try dplyr library to sort by day and summarize the steps

```{R }
act.day <- group_by(act.complete, date)
act.day <- summarize(act.day, steps=sum(steps))
```

Look at the data per day

```{R }
summary(act.day)
```

##### Plot the histogram of the total steps per day

```{R include=FALSE}
qplot(steps, data=act.day)
```

### Calculate the mean and median

```{R }
mean(act.day$steps)
```

```{R }
median(act.day$steps)
```

### What is the average daily activity pattern?

Store a data frame in which steps are aggregated into averages within each 5 minute interval:

```{R }
act.int <- group_by(act.complete, interval)
act.int <- summarize(act.int, steps=mean(steps))
```

Plot the average daily steps

```{R include=FALSE}
ggplot(act.int, aes(interval, steps)) + geom_line()
```

### The 5-minute interval that, on average, contains the maximum number of steps

```{R }
act.int[act.int$steps==max(act.int$steps),]
```

### Check the recuency of the missing values

```{R }
nrow(activity)-nrow(act.complete)
```

Fill in all of the missing values in the dataset. The strategy does not need to be sophisticated.
replace missing values with the mean number of steps for each interval across all of the days. The act.int data frame contains these means.

```{R }
names(act.int)[2] <- "mean.steps"
act.impute <- merge(activity, act.int)
```

```{R }
act.impute$steps[is.na(act.impute$steps)] <- act.impute$mean.steps[is.na(act.impute$steps)]
```

Plot the histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment?

```{R }
act.day.imp <- group_by(act.impute, date)
act.day.imp <- summarize(act.day.imp, steps=sum(steps))
```

```{R include=FALSE}
qplot(steps, data=act.day.imp)
```

### Mean and Median

```{R }
mean(act.day.imp$steps)
```

```{R }
median(act.day.imp$steps)
```

### Are there differences in activity patterns between weekdays and weekends?

Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.

```{R }
act.impute$dayofweek <- weekdays(as.Date(act.impute$date))
act.impute$weekend <-as.factor(act.impute$dayofweek=="Saturday"|act.impute$dayofweek=="Sunday")
levels(act.impute$weekend) <- c("Weekday", "Weekend")
```

Make a panel plot containing a time series plot (i.e. type = “l”) of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis).
Create separate data frames for weekends and weekdays:

```{R }
act.weekday <- act.impute[act.impute$weekend=="Weekday",]
act.weekend <- act.impute[act.impute$weekend=="Weekend",]
```

Then for each one, I find the mean number of steps across days for each 5 minute interval:

```{R }
act.int.weekday <- group_by(act.weekday, interval)
act.int.weekday <- summarize(act.int.weekday, steps=mean(steps))
act.int.weekday$weekend <- "Weekday"
act.int.weekend <- group_by(act.weekend, interval)
act.int.weekend <- summarize(act.int.weekend, steps=mean(steps))
act.int.weekend$weekend <- "Weekend"
```

### Panel plot comparing the average number of steps taken per 5-minute interval across weekdays and weekends


```{R }
act.int <- rbind(act.int.weekday, act.int.weekend)
act.int$weekend <- as.factor(act.int$weekend)
```

```{R include=FALSE}
ggplot(act.int, aes(interval, steps)) + geom_line() + facet_grid(weekend ~ .)
```
