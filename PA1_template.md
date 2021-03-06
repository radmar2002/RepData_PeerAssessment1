
<!--
Load knitr library:
library(knitr)
Obtain the **.md** file:
knit2html("PA1_template.Rmd")
browseURL("PA1_template.html")
-->

## Reproducible Research: Peer Assessment 1

**Marius Florin RADU**
<br>**Cluj-Napoca, Cluj, ROMANIA**
<br>mail: radu_marius_florin@yahoo.com



## Loading and preprocessing the data


Read the data source:

```r
activityDataFrame <- read.table(unz("activity.zip", "activity.csv"), 
                                colClasses = c("numeric", "Date", "numeric"),
                                header=T, quote="\"", sep=",")
head(activityDataFrame, 3)
```

```
##   steps       date interval
## 1    NA 2012-10-01        0
## 2    NA 2012-10-01        5
## 3    NA 2012-10-01       10
```


## What is mean total number of steps taken per day?



```r
# Use data.table to compute mean and median
library(data.table)
```

```
## data.table 1.9.2  For help type: help("data.table")
```

```r
DT <- data.table(activityDataFrame, key = c("interval"))
# Calculate the mean
(mMean <- DT[, sum(steps), by = date][,list(Mean = mean(V1, na.rm = TRUE))])
```

```
##     Mean
## 1: 10766
```

```r
# Calculate the median
(mMedian <- DT[, sum(steps), by = date][,list(Median = median(V1, na.rm = TRUE))])
```

```
##    Median
## 1:  10765
```

The histogram of the total number of steps taken each day

```r
library(ggplot2)
activityCompleteCases <- activityDataFrame[complete.cases(activityDataFrame$steps),] 
DT2 <- data.table(activityCompleteCases, key = c("interval"))
stepsDaily <- as.data.frame(DT2[, list(`Steps per Day`=sum(steps)), by = date])
# Histogram
hist1 <- qplot(`Steps per Day`, data=stepsDaily, geom="histogram") +
        geom_histogram(aes(fill = ..count..)) +
        xlab('Number of Steps per Day') +
        ggtitle('Histogram of the total number of steps taken each day') +
        theme(plot.title = element_text(lineheight=.8, face="bold"))
hist1
```

```
## stat_bin: binwidth defaulted to range/30. Use 'binwidth = x' to adjust this.
## stat_bin: binwidth defaulted to range/30. Use 'binwidth = x' to adjust this.
```

![plot of chunk chart_steps_1](figure/chart_steps_1.png) 

The __mean__ number of steps per day is: __10766__

The __median__ number of steps per day is: __10765__


## What is the average daily activity pattern?



```r
library(ggplot2)
dataFrameforTimeSeries <- DT[, list( `Average # of Steps` = mean(steps, na.rm = TRUE) ), by = interval]
p <- ggplot(dataFrameforTimeSeries , aes(x=interval, y=`Average # of Steps`)) + 
     geom_line(colour = "dark blue", size=0.8) + 
     theme(axis.text.x = element_text(angle = 90, hjust = 1))

p
```

![plot of chunk chart_steps_2](figure/chart_steps_2.png) 


```r
(maxSumSteps<-DT[, list( sumSteps = sum(steps, na.rm = TRUE) ), by = interval][, .SD[which.max(sumSteps)]])
```

```
##    interval sumSteps
## 1:      835    10927
```

```r
theInterval <- maxSumSteps[[1]]
theMaxSum <-maxSumSteps [[2]]
```

The 5-minute interval which contains the maximum number of steps is __835th__ interval, and the corespondent number of steps is __10927__ 

## Imputing missing values


```r
isNA <- dim(activityDataFrame[!complete.cases(activityDataFrame),])[1]
```
The total number of rows with *NAs* is __2304__


**Imputing strategy:** to replace missing values of steps variable in following rows we use **the mean computed for corespondent week** of the record.

```r
head(activityDataFrame, 3)
```

```
##   steps       date interval
## 1    NA 2012-10-01        0
## 2    NA 2012-10-01        5
## 3    NA 2012-10-01       10
```

```r
activityDataFrame02 <- activityDataFrame
activityDataFrame02$Week <- format(activityDataFrame02$date,"%Y-%U")
stepsMeanByWeek <- aggregate(steps ~ Week, activityDataFrame02, FUN = mean, na.rm = TRUE)
for( i in seq( dim(activityDataFrame02)[1] )){
        if( is.na(activityDataFrame02[i , c("steps")]) ){
                activityDataFrame02[i, c("steps")] <- stepsMeanByWeek[which(stepsMeanByWeek$Week == activityDataFrame02[i, c("Week")] ),c("steps")]
        }
}
activityDataFrame02 <- activityDataFrame02[,-c(4)]
```


The result is a new dataset that is equal to the original dataset but with the missing data filled in:

```r
head(activityDataFrame02, 3)
```

```
##   steps       date interval
## 1 36.33 2012-10-01        0
## 2 36.33 2012-10-01        5
## 3 36.33 2012-10-01       10
```

```r
summary(activityDataFrame02)
```

```
##      steps            date               interval   
##  Min.   :  0.0   Min.   :2012-10-01   Min.   :   0  
##  1st Qu.:  0.0   1st Qu.:2012-10-16   1st Qu.: 589  
##  Median :  0.0   Median :2012-10-31   Median :1178  
##  Mean   : 37.0   Mean   :2012-10-31   Mean   :1178  
##  3rd Qu.: 30.3   3rd Qu.:2012-11-15   3rd Qu.:1766  
##  Max.   :806.0   Max.   :2012-11-30   Max.   :2355
```



```r
# Use data.table to compute mean and median
library(data.table)
DTII <- data.table(activityDataFrame02, key = c("interval"))
# Calculate the mean
(mMeanII <- DTII[, sum(steps), by = date][,list(Mean = mean(V1, na.rm = TRUE))])
```

```
##     Mean
## 1: 10643
```

```r
# Calculate the median
(mMedianII <- DTII[, sum(steps), by = date][,list(Median = median(V1, na.rm = TRUE))])
```

```
##    Median
## 1:  10571
```

The histogram of the total number of steps taken each day

```r
library(ggplot2)
activityCompleteCasesII <- activityDataFrame02[complete.cases(activityDataFrame02$steps),] 
DT2II <- data.table(activityCompleteCasesII, key = c("interval"))
stepsDailyII <- as.data.frame(DT2II[, list(`Steps per Day`=sum(steps)), by = date])
# Histogram
hist2 <- qplot(`Steps per Day`, data=stepsDailyII, geom="histogram") +
        geom_histogram(aes(fill = ..count..)) +
        xlab('Number of Steps per Day') +
        ggtitle('Histogram of the total number of steps taken each day after NA replacement') +
        theme(plot.title = element_text(lineheight=.8, face="bold")) + 
        scale_fill_gradient("Count", low = "dark blue", high = "red")
hist2
```

```
## stat_bin: binwidth defaulted to range/30. Use 'binwidth = x' to adjust this.
## stat_bin: binwidth defaulted to range/30. Use 'binwidth = x' to adjust this.
```

![plot of chunk chart_comparisons](figure/chart_comparisons.png) 

The __mean__ number of steps per day after **NAs** replacement is: __10643__ and it is not statistically different from the __mean__ before replachement which is __10766__

There is bellow the **t-test** evaluation 

```r
activityDataFrame02$old_steps <- activityDataFrame$steps
(t.test(activityDataFrame02$steps,activityDataFrame02$old_steps))
```

```
## 
## 	Welch Two Sample t-test
## 
## data:  activityDataFrame02$steps and activityDataFrame02$old_steps
## t = -0.3569, df = 31441, p-value = 0.7212
## alternative hypothesis: true difference in means is not equal to 0
## 95 percent confidence interval:
##  -2.782  1.925
## sample estimates:
## mean of x mean of y 
##     36.95     37.38
```

The __median__ number of steps per day after **NAs** replacement is: __10571__ and it is not statistically different from the __median__ before replachement which is __10765__

There is bellow the **Mann-Whitney-Wilcoxon test** evaluation 

```r
(wilcox.test(activityDataFrame02$steps,activityDataFrame02$old_steps))
```

```
## 
## 	Wilcoxon rank sum test with continuity correction
## 
## data:  activityDataFrame02$steps and activityDataFrame02$old_steps
## W = 145161216, p-value < 2.2e-16
## alternative hypothesis: true location shift is not equal to 0
```

The impact of *NAs* replacement with the average on week data is relatively small.

## Are there differences in activity patterns between weekdays and weekends?



```r
# Create a new factor variable in the dataset with two levels: 'weekday' and 'weekend' 
# indicating whether a given date is a weekday or weekend day.
# Decimal Weekday (0=Sunday)
activityDataFrame03 <- activityDataFrame
activityDataFrame03$DayOfWeek <- as.factor(ifelse(format(activityDataFrame$date,"%w") %in% c(0,6), 0, 1))
levels(activityDataFrame03$DayOfWeek) <- c("weekend", "weekday")
table(activityDataFrame03$DayOfWeek)
```

```
## 
## weekend weekday 
##    4608   12960
```

```r
activityCompleteCasesIII <- activityDataFrame03[complete.cases(activityDataFrame03$steps),] 
head(activityCompleteCasesIII, 3)
```

```
##     steps       date interval DayOfWeek
## 289     0 2012-10-02        0   weekday
## 290     0 2012-10-02        5   weekday
## 291     0 2012-10-02       10   weekday
```

```r
# Histogram
tsPlot2 <- ggplot(activityCompleteCasesIII, aes(interval, steps)) +  
        ylab('Number of Steps') +
        ggtitle('# steps taken each interval by day of the week') +
        stat_summary(fun.y = "mean", geom="line", col = "red", size=0.8) + 
        facet_grid(DayOfWeek ~ .) +
        theme(axis.text.x = element_text(angle = 90, hjust = 1))
tsPlot2
```

![plot of chunk plot_by_weekday](figure/plot_by_weekday.png) 

**Yes**, there are different patterns for weekdays vs. weekends number of steps per 5-minute interval. 
