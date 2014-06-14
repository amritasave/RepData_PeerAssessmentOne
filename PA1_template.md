Reproducible Research Assignment 1
========================================================



```r
install.packages("data.table")
```

```
## Installing package into 'C:/Users/Amit/Documents/R/win-library/3.1'
## (as 'lib' is unspecified)
```

```
## Warning: package 'data.table' is in use and will not be installed
```

```r
library(data.table)

install.packages("ggplot2")
```

```
## Installing package into 'C:/Users/Amit/Documents/R/win-library/3.1'
## (as 'lib' is unspecified)
```

```
## Warning: package 'ggplot2' is in use and will not be installed
```

```r
library(ggplot2)
```
### 1.  Load the data (i.e. read.csv())


```r
activityDF <- read.csv("activity.csv" , sep="," , header =TRUE)
```

### What is mean total number of steps taken per day?
### For this part of the assignment, you can ignore the 
### missing values in the dataset.


```r
newdata <- subset(activityDF, steps != "NA", select=c(steps, date ,interval))
newdataDT <- data.table(newdata)
newdataFinal <- newdataDT[,sum(steps) , by = date]
plotdata <- as.data.frame(newdataFinal)
```

### 1.  Make a histogram of the total number of steps taken each day


```r
hist(plotdata$V1 , main= "Activity Monitoring Data",col= "red",  xlab="Steps")
```

![plot of chunk unnamed-chunk-4](figure/unnamed-chunk-4.png) 


### 2.  Calculate and report the mean and median total number 
### of steps taken per day



```r
mean(plotdata$V1)
```

```
## [1] 10766
```

```r
median(plotdata$V1)
```

```
## [1] 10765
```

### What is the average daily activity pattern?


```r
newdata1 <- subset(activityDF, steps != "NA" , select=c(steps, date ,interval))
DT <- data.table(newdata1)
DTfinal <- DT[, sum(steps) , by = interval ]
```
### 1.  Make a time series plot (i.e. type = "l") of the 
### 5-minute interval (x-axis) and the average number of steps taken,
### averaged across all days (y-axis)


```r
qplot(interval, V1, data = as.data.frame(DTfinal), geom = "line" ,main="Activity" , ylab="Steps")
```

![plot of chunk unnamed-chunk-7](figure/unnamed-chunk-7.png) 


### 2.  Which 5-minute interval, on average across all the days 
### in the dataset, contains the maximum number of steps?



```r
tail(DTfinal[order(DTfinal$V1  ), ], n=1)
```

```
##    interval    V1
## 1:      835 10927
```

### Imputing missing values
### Note that there are a number of days/intervals where there are 
### missing values (coded as NA). The presence of missing days may 
### introduce bias into some calculations or summaries of the data.

### 1.  Calculate and report the total number of missing values in 
### the dataset (i.e. the total number of rows with NAs)



```r
sum(is.na(activityDF)) 
```

```
## [1] 2304
```

```r
activityDF1 <- activityDF
DT3 <- data.table(activityDF1)
```

### 2.  Devise a strategy for filling in all of the missing values 
### in the dataset. The strategy does not need to be sophisticated. 
### For example, you could use the mean/median for that day, 
### or the mean for that 5-minute interval, etc.



```r
DTMean <- DT[, mean(steps) , by = interval ]

MrgDT = merge(DT3 , DTMean , by ="interval" , suffixes =c(".DT3" , ".DTMean"))
na.idx = which(is.na(DT3$steps))
DT3[na.idx , "steps"] = MrgDT[na.idx ,"V1.DTMean"]
```

```
## Warning: NAs introduced by coercion
```
### 3.  Create a new dataset that is equal to the original 
### dataset but with the missing data filled in.



```r
MrgDT$steps[is.na(MrgDT$steps) ] <- MrgDT$V1
```

```
## Warning: number of items to replace is not a multiple of replacement
## length
```

```r
MrgDT$steps <- round(MrgDT$steps ,2)

MrgDTFinal <- MrgDT[,sum(steps) , by = date]

plotdata3 <- as.data.frame(MrgDTFinal)
```


### 4.  Make a histogram of the total number of steps taken 
### each day and Calculate and report the mean and median total 
### number of steps taken per day. Do these values differ from the 
### estimates from the first part of the assignment? What is the 
### impact of imputing missing data on the estimates of the total 
### daily number of steps?


```r
hist(plotdata3$V1 , main= "Activity Monitoring Data",col= "orange",  xlab="Steps")
```

![plot of chunk unnamed-chunk-12](figure/unnamed-chunk-12.png) 



```r
mean(plotdata3$V1)
```

```
## [1] 9371
```

```r
median(plotdata3$V1)
```

```
## [1] 10395
```

### Are there differences in activity patterns between weekdays and weekends?
### For this part the weekdays() function may be of 
### some help here. Use the dataset with the filled-in 
### missing values for this part.



```r
MrgDF <- as.data.frame(MrgDT)

MrgDF[,"date"] <- as.Date(as.character(MrgDF[,"date"]))

MrgDF$Wday <- weekdays(MrgDF[ , "date"] , abbreviate = TRUE)
```

### 1.  Create a new factor variable in the dataset 
### with two levels - "weekday" and "weekend" indicating 
### whether a given date is a weekday or weekend day.



```r
MrgDF$Wday[MrgDF$Wday %in% c("Mon" ,"Tue" ,"Wed" ,"Thu" ,"Fri")] <- "Weekday"

MrgDF$Wday[MrgDF$Wday %in% c("Sat" ,"Sun")] <- "Weekend"

MrgDT1 <- data.table(MrgDF)
```


### 2.  Make a panel plot containing a time series plot 
### (i.e. type = "l") of the 5-minute interval (x-axis) and 
### the average number of steps taken, averaged across all 
### weekday days or weekend days (y-axis). 


```r
setkey(MrgDT1 , interval , Wday)

MrgDT1final <-  MrgDT1[,sum(steps) , by = key(MrgDT1)]
```


```r
qplot(x = interval ,y = round(V1) , data = as.data.frame(MrgDT1final) , group =1)+ facet_grid(Wday ~ . )  + geom_line() +xlab("Interval") +ylab("Steps") + ggtitle("Activity monitoring data")
```

![plot of chunk unnamed-chunk-17](figure/unnamed-chunk-17.png) 



