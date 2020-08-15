---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---



## Introduction

It is now possible to collect a large amount of data about personal
movement using activity monitoring devices such as a
[Fitbit](http://www.fitbit.com), [Nike
Fuelband](http://www.nike.com/us/en_us/c/nikeplus-fuelband), or
[Jawbone Up](https://jawbone.com/up). These type of devices are part of
the "quantified self" movement -- a group of enthusiasts who take
measurements about themselves regularly to improve their health, to
find patterns in their behavior, or because they are tech geeks. But
these data remain under-utilized both because the raw data are hard to
obtain and there is a lack of statistical methods and software for
processing and interpreting the data.

This assignment makes use of data from a personal activity monitoring
device. This device collects data at 5 minute intervals through out the
day. The data consists of two months of data from an anonymous
individual collected during the months of October and November, 2012
and include the number of steps taken in 5 minute intervals each day.

## Data

The data for this assignment can be downloaded from the course web
site:

* Dataset: [Activity monitoring data](https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip) [52K]

The variables included in this dataset are:

* **steps**: Number of steps taking in a 5-minute interval (missing
    values are coded as `NA`)

* **date**: The date on which the measurement was taken in YYYY-MM-DD
    format

* **interval**: Identifier for the 5-minute interval in which
    measurement was taken




The dataset is stored in a comma-separated-value (CSV) file and there
are a total of 17,568 observations in this
dataset.


## Assignment

This assignment will be described in multiple parts. You will need to
write a report that answers the questions detailed below. Ultimately,
you will need to complete the entire assignment in a **single R
markdown** document that can be processed by **knitr** and be
transformed into an HTML file.

Throughout your report make sure you always include the code that you
used to generate the output you present. When writing code chunks in
the R markdown document, always use `echo = TRUE` so that someone else
will be able to read the code. **This assignment will be evaluated via
peer assessment so it is essential that your peer evaluators be able
to review the code for your analysis**.

For the plotting aspects of this assignment, feel free to use any
plotting system in R (i.e., base, lattice, ggplot2)

Fork/clone the [GitHub repository created for this
assignment](http://github.com/rdpeng/RepData_PeerAssessment1). You
will submit this assignment by pushing your completed files into your
forked repository on GitHub. The assignment submission will consist of
the URL to your GitHub repository and the SHA-1 commit ID for your
repository state.

NOTE: The GitHub repository also contains the dataset for the
assignment so you do not have to download the data separately.



### Loading and preprocessing the data

Show any code that is needed to

1. Load the data (i.e. `read.csv()`)

```r
library(dplyr)
filename <- "activity.zip"
# Checking if archieve already exists.
if (!file.exists(filename)){
    fileURL <- "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip"
    download.file(fileURL, filename, method="curl")
}

# Checking if folder exists
if (!file.exists("activity.csv")) { 
    unzip(filename) 
}

activity<-read.table("activity.csv",sep = ",",header = TRUE)
```


2. Process/transform the data (if necessary) into a format suitable for your analysis

```r
#library(dplyr)
activity$date<-as.Date(activity$date,"%Y-%m-%d")
```


### What is mean total number of steps taken per day?

For this part of the assignment, you can ignore the missing values in
the dataset.

1. Make a histogram of the total number of steps taken each day

```r
#Groupping and calculating 
Total_activity<-activity%>%na.omit%>%group_by(date)%>%summarise(Total=sum(steps))
hist(Total_activity$Total,main = "Total Number of steps per day",xlab = "Steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-3-1.png)<!-- -->




2. Calculate and report the **mean** and **median** total number of steps taken per day

```r
c(Mean_Steps = mean(Total_activity$Total, na.rm = TRUE), Median_Steps = median(Total_activity$Total, na.rm = TRUE))
```

```
##   Mean_Steps Median_Steps 
##     10766.19     10765.00
```



### What is the average daily activity pattern?

1. Make a time series plot (i.e. `type = "l"`) of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

```r
library(ggplot2)
Mean_steps<-activity%>%na.omit%>%group_by(interval)%>%summarise(Mean=mean(steps))
ggplot(Mean_steps, aes(x = interval , y = Mean)) + 
    geom_line(color="black", size=1) + 
    labs(title = "Average number of steps over all days", x = "5 min interval", y = "Avg. Steps per day")
```

![](PA1_template_files/figure-html/unnamed-chunk-6-1.png)<!-- -->



2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```r
Mean_steps%>%filter(Mean==max(Mean))%>%select(interval)
```

```
## # A tibble: 1 x 1
##   interval
##      <int>
## 1      835
```



### Imputing missing values

Note that there are a number of days/intervals where there are missing
values (coded as `NA`). The presence of missing days may introduce
bias into some calculations or summaries of the data.

1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with `NA`s)


```r
sum(is.na(activity$steps))
```

```
## [1] 2304
```

2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.


```r
Na_value<-activity[is.na(activity$steps),]
step_mean<-activity%>%na.omit%>%group_by(interval)%>%summarise(steps=mean(steps))
Na_value$steps<-factor(Na_value$interval,levels = step_mean$interval,labels = step_mean$steps)
```


3. Create a new dataset that is equal to the original dataset but with the missing data filled in.


```r
activity$steps[is.na(activity$steps)]<-as.numeric(levels(Na_value$steps))[Na_value$steps]
head(activity,10)
```

```
##        steps       date interval
## 1  1.7169811 2012-10-01        0
## 2  0.3396226 2012-10-01        5
## 3  0.1320755 2012-10-01       10
## 4  0.1509434 2012-10-01       15
## 5  0.0754717 2012-10-01       20
## 6  2.0943396 2012-10-01       25
## 7  0.5283019 2012-10-01       30
## 8  0.8679245 2012-10-01       35
## 9  0.0000000 2012-10-01       40
## 10 1.4716981 2012-10-01       45
```


4. Make a histogram of the total number of steps taken each day and Calculate and report the **mean** and **median** total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?


```r
Total_activity<-activity%>%na.omit%>%group_by(date)%>%summarise(Total=sum(steps))
hist(Total_activity$Total,main = "Total Number of steps perday",xlab = "Steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-12-1.png)<!-- -->





```r
c(Mean_Steps = mean(Total_activity$Total, na.rm = TRUE), Median_Steps = median(Total_activity$Total, na.rm = TRUE))
```

```
##   Mean_Steps Median_Steps 
##     10766.19     10766.19
```

### Are there differences in activity patterns between weekdays and weekends?

For this part the `weekdays()` function may be of some help here. Use
the dataset with the filled-in missing values for this part.

1. Create a new factor variable in the dataset with two levels -- "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.


```r
activity$date<-weekdays(activity$date)
activity$date<-ifelse(activity$date=="sábado"|activity$date=="domingo","Weekend","Weekday")
head(activity,10)
```

```
##        steps    date interval
## 1  1.7169811 Weekday        0
## 2  0.3396226 Weekday        5
## 3  0.1320755 Weekday       10
## 4  0.1509434 Weekday       15
## 5  0.0754717 Weekday       20
## 6  2.0943396 Weekday       25
## 7  0.5283019 Weekday       30
## 8  0.8679245 Weekday       35
## 9  0.0000000 Weekday       40
## 10 1.4716981 Weekday       45
```


1. Make a panel plot containing a time series plot (i.e. `type = "l"`) of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). The plot should look something like the following, which was created using **simulated data**:


```r
Mean_step<-activity%>%group_by(interval,date)%>%summarise(Mean=mean(steps))%>%arrange(date)
colnames(Mean_step)[2]<-"Type_of_day"
g<-ggplot(Mean_step,aes(x=interval,y=Mean,fill=Type_of_day,color=Type_of_day))
g+geom_line()+facet_grid(.~Type_of_day)+
    labs(x = "Interval", y = "Number of Steps") +ggtitle ("Avg. Daily Steps by Weektype")
```

![](PA1_template_files/figure-html/unnamed-chunk-16-1.png)<!-- -->



