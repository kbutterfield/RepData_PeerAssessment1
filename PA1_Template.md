Setting the directory
---------------------

-   Set directory

<!-- -->

     setwd("C:/Users/Kristen/Documents/R/HWRR2")

     getwd()

    ## [1] "C:/Users/Kristen/Documents/R/HWRR2"

Loading the data
----------------

-   Load the data

<!-- -->

    activity=read.csv("activity.csv")

-   Pull up plotting package

<!-- -->

    library(ggplot2)

    ## Warning: package 'ggplot2' was built under R version 3.4.4

    library(dplyr)

    ## Warning: package 'dplyr' was built under R version 3.4.4

    ## 
    ## Attaching package: 'dplyr'

    ## The following objects are masked from 'package:stats':
    ## 
    ##     filter, lag

    ## The following objects are masked from 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union

    library(plyr)

    ## Warning: package 'plyr' was built under R version 3.4.4

    ## -------------------------------------------------------------------------

    ## You have loaded plyr after dplyr - this is likely to cause problems.
    ## If you need functions from both plyr and dplyr, please load plyr first, then dplyr:
    ## library(plyr); library(dplyr)

    ## -------------------------------------------------------------------------

    ## 
    ## Attaching package: 'plyr'

    ## The following objects are masked from 'package:dplyr':
    ## 
    ##     arrange, count, desc, failwith, id, mutate, rename, summarise,
    ##     summarize

-   Create weeekday variable and format date time

<!-- -->

    activity$day <- weekdays(as.Date(activity$date))
    activity$DateTime<- as.POSIXct(activity$date, format="%Y-%m-%d")

Q1: What is mean total number of steps taken per day?
-----------------------------------------------------

-   Calculate the total number of steps taken per day

<!-- -->

    sumsteps <- aggregate(activity$steps ~ activity$date, FUN=sum, )
    colnames(sumsteps)<- c("Date", "Steps")

-   Make a histogram of the total number of steps taken each day

<!-- -->

    hist(sumsteps$Steps, breaks=5, xlab="Steps", main = "Total Steps per Day")

![](PA1_Template_files/figure-markdown_strict/unnamed-chunk-6-1.png)

-   Calculate and report the **mean** and **median** total number of
    steps taken per day

<!-- -->

    as.integer(mean(sumsteps$Steps))

    ## [1] 10766

    as.integer(median(sumsteps$Steps))

    ## [1] 10765

-   The **mean** total number of steps taken per day is 10766
-   The **median** total number of steps taken per day is 10765

Q2 What is the average daily activity pattern?
----------------------------------------------

-   Make a time series plot (i.e. type = "l") of the 5-minute interval
    (x-axis) and the average number of steps taken, averaged across all
    days (y-axis)

-   Creating dataset with missing values removed

<!-- -->

    nomiss <- activity[!is.na(activity$steps),]

-   Create average number of steps per interval

<!-- -->

    intervaldata <- ddply(nomiss, .(interval), summarize, Avg = mean(steps))

-   Create line plot of average number of steps per interval

<!-- -->

    p <- ggplot(intervaldata, aes(x=interval, y=Avg), xlab = "Interval", ylab="Average Number of Steps")
    p + geom_line()+xlab("Interval")+ylab("Average Number of Steps")+ggtitle("Average Number of Steps per Interval")

![](PA1_Template_files/figure-markdown_strict/unnamed-chunk-10-1.png)

\*Which 5-minute interval, on average across all the days in the
dataset, contains the maximum number of steps?

\*Calculate maximum steps by interval

    maximumSteps <- max(intervaldata$Avg)

\*Which interval contains the maximum average number of steps

    intervaldata[intervaldata$Avg==maximumSteps,1]

    ## [1] 835

-   It is interval 835.

Q3 Imputing missing values
--------------------------

-   Calculate and report the total number of missing values in the
    dataset (i.e. the total number of rows with NAs)

<!-- -->

    nrow(activity[is.na(activity$steps),])

    ## [1] 2304

-   Total 2304 rows are missing.

-   Devise a strategy for filling in all of the missing values in the
    dataset. The strategy does not need to be sophisticated. For
    example, you could use the mean/median for that day, or the mean for
    that 5-minute interval, etc.

-   I used mean value for missing values

-   Create the average number of steps per day / interval

<!-- -->

    avgdata <- ddply(nomiss, .(interval, day), summarize, Avg = mean(steps))

-   Create dataset with only NAs

<!-- -->

    missingdata<- activity[is.na(activity$steps),]

-   Merge NA data with average weekday interval

<!-- -->

    mergedata<-merge(missingdata, avgdata, by=c("interval", "day"))

-   Create a new dataset with missing data now filled in.

<!-- -->

    mergedata2<- mergedata[,c(6,4,1,2,5)]
    colnames(mergedata2)<- c("steps", "date", "interval", "day", "DateTime")

\*Merge the missing averages and non-missing data

    FinalData <- rbind(nomiss, mergedata2)

-   Make a histogram of the total number of steps taken each day and
    Calculate and report the mean and median total number of steps taken
    per day.

<!-- -->

    sumsteps2 <- aggregate(FinalData$steps ~ FinalData$date, FUN=sum, )
    colnames(sumsteps2)<- c("Date", "Steps")

\*Mean of Steps with imputed missing data

    as.integer(mean(sumsteps2$Steps))

    ## [1] 10821

\*Median of Steps with imputed missing data

    as.integer(median(sumsteps2$Steps))

    ## [1] 11015

-   The **mean** total number of steps taken per day is 10821

-   The **median** total number of steps taken per day is 11095

\*Creating the histogram of total steps per day - compare dataset with
imputed data to data from first part of assigment to show impact

    hist(sumsteps2$Steps, breaks=5, xlab="Steps", main = "Total Steps per Day with Imputed Data", col="Black")
    hist(sumsteps$Steps, breaks=5, xlab="Steps", main = "Total Steps per Day with Imputed data", col="Grey", add=T)
    legend("topright", c("Imputed Data", "Non-NA Data"), fill=c("black", "grey") )

![](PA1_Template_files/figure-markdown_strict/unnamed-chunk-22-1.png) \*
Do these values differ from the estimates from the first part of the
assignment? What is the impact of imputing missing data on the estimates
of the total daily number of steps?

-   The **mean** and **median** values are both greater after imputting
    data.

Q4 Are there differences in activity patterns between weekdays and weekends?
----------------------------------------------------------------------------

-   Create a new dichotomous variable to identify if the day is a
    weekday or a weekend.

<!-- -->

    FinalData$DayType <- ifelse(FinalData$day %in% c("Saturday", "Sunday"), "Weekend", "Weekday")

-   Make a panel plot containing a time series plot (i.e. type = "l") of
    the 5-minute interval (x-axis) and the average number of steps
    taken, averaged across all weekday days or weekend days (y-axis).
    The plot should look something like the following, which was
    creating using simulated data:

<!-- -->

    library(lattice) 

\*Summarize data by interval and weekend / weekday category

    intervaldata <- ddply(FinalData, .(interval, DayType), summarize, Avg = mean(steps))

\*Plot data in a panel plot

    xyplot(Avg~interval|DayType, data=intervaldata, type="l",  layout = c(1,2),
           main="Average Steps per Interval Based on Type of Day", 
           ylab="Average Number of Steps", xlab="Interval")

![](PA1_Template_files/figure-markdown_strict/unnamed-chunk-26-1.png)
\*The weekday plot shows a higher number of aveage steps than the
weekend plots
