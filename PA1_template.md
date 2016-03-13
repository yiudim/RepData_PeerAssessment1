---
title: "Reproducible Data Programming Assignment 1"
author: "yiudim"
output: "html_document"
---

## load necessary libraries

```r
require(dplyr)
require(ggplot2)
require(Hmisc)
```
	
## read the data file

```r
fileurl <- "./activity.csv"
pa1raw <- read.csv(fileurl)
```

## raw data pre-process -> make "date" into Date format

```r
pa1raw <- pa1raw %>% 
mutate(fdate = as.Date(date, format("%Y-%m-%d"))) %>% 
mutate(intervalt = as.POSIXct(formatC(interval, width=4, flag="0"),format="%H%M")) %>%
select(-date)
```
		

# What is the mean total of steps taken per day?
## calcuate total steps per day

```r
pa1 <- pa1raw %>% 
group_by(fdate) %>%
summarise(total_steps = sum(steps))
```

## show total steps per day in a histogram

```r
png(filename="./instructions_fig/plot1.png", width=480, height=480)
hist(pa1$total_steps, xlab="Total Steps per Day", col="green", breaks=30)
dev.off()
```

```
## quartz_off_screen 
##                 2
```

## calculate and report mean and median of the total number of steps taken per day

```r
meanvalue <- as.integer(mean(pa1$total_steps,na.rm=TRUE))
medianvalue <- median(pa1$total_steps,na.rm=TRUE)
```

Mean is 10766

Median is 10765

## What is the daily average activity pattern?
## calcuate average steps per 5-minute interval

```r
pa1 <- pa1raw %>% 
group_by(intervalt) %>%
summarise(average_steps = mean(steps, na.rm=TRUE))
```

## plot average steps per 5-min interval in time series plot

```r
png(filename="./instructions_fig/plot2.png", width=480, height=480)
with(pa1, plot(intervalt, average_steps, type="l", xlab="5-minute Intervals", 
main="Average Steps Taken per 5-min Intervals"))
abline(v=pa1$intervalt[which(pa1$average==max(pa1$average_steps))], col="blue")
```

	pa1$interval[which(pa1$average==max(pa1$average_steps))]
	## 835


	## --------  IMPUTING MISSING VALUES -----------
	## Calculate and report the total number of missing values in the dataset
	sum(is.na(pa1raw))
	## There is a total of 2304 NAs in the given dataset (activities.csv)

	## Assign average value of interval if step is NA
	pa1c <- pa1raw
	for(i in 1:nrow(pa1c)) {
		if(is.na(pa1c$steps[i])) {
			pa1c$steps[i] <- as.integer(pa1$average_steps[which(pa1$intervalt == pa1c$intervalt[i])])
		}
	}

	## Re-look at total, mean, and median values
        pa2 <- pa1c %>% 
		group_by(fdate) %>%
		summarise(total_steps = sum(steps))

	## show total steps per day in a histogram
	png(filename="./instructions_fig/plot3.png", width=480, height=480)
	hist(pa2$total_steps, xlab="Total Steps per Day", col="green", breaks=30)
	dev.off()

	## calculate and report mean and median of the total number of steps taken per day
	mean(pa2$total_steps,na.rm=TRUE)
	## mean is 10749.77
	median(pa2$total_steps,na.rm=TRUE)
	## median is 10641
	## values are different from when we calcuated the same stats excluding the NAs in the dataset


	## --------  ACTIVITY PATTERNS WEEKDAY vs WEEKEND -----------
	for(i in 1:nrow(pa1c)) {
		dofw <- weekdays(pa1c$fdate[i]) 
		if(dofw %in% c("Saturday","Sunday")){
			pa1c$dayofweeks[i] <- "weekend"
		} else {
			pa1c$dayofweeks[i] <- "weekday"
		}
	}

	# setup graphing panel
        png(filename="./instructions_fig/plot4.png", width=480, height=480)
	par(mfrow=c(2,1))
	par(cex = 0.6)
	par(mar = c(0, 0, 0, 0), oma = c(4, 4, 0.5, 0.5))


	pa3 <- pa1c %>% 
		filter(dayofweeks == "weekday") %>%
		group_by(intervalt) %>%
		summarise(average_steps = mean(steps, na.rm=TRUE))

	## plot average steps per 5-min interval in time series plot
	with(pa3, plot(intervalt, average_steps, type="l", xlab="5-minute Intervals")) 
	title(main="Average Steps Taken per 5-min Intervals - Weekdays", line = -2)
	abline(v=pa3$intervalt[which(pa3$average==max(pa3$average_steps))], col="blue")

	pa3$interval[which(pa3$average==max(pa3$average_steps))]
	## 835

	pa4 <- pa1c %>% 
		filter(dayofweeks == "weekend") %>%
		group_by(intervalt) %>%
		summarise(average_steps = mean(steps, na.rm=TRUE))

	## plot average steps per 5-min interval in time series plot
	with(pa4, plot(intervalt, average_steps, type="l", xlab="5-minute Intervals")) 
	title(main="Average Steps Taken per 5-min Intervals - Weekends", line=-2)
	abline(v=pa4$intervalt[which(pa4$average==max(pa4$average_steps))], col="blue")

	pa4$interval[which(pa4$average==max(pa4$average_steps))]
	## 915

	dev.off()
	
