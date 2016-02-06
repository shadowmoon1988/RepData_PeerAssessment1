Introduction
============

With the advancement of technology and the growth of the **big data** movement, it is now possible to collect a large amount of data about personal movement using activity monitoring devices. Such examples are: [Fitbit](http://www.fitbit.com/), [Nike Fuelband](http://www.nike.com/us/en_us/c/nikeplus-fuelband), or [Jawbone Up](https://jawbone.com/up). These kinds of devices are part of the "quantified self" movement: those who measurements about themselves on a regular basis in order to improve their health, find patterns in their behaviour, or because they are simply technology geeks. However, these data remain severely underused due to the fact that the raw data are hard to obtain and there is a lack of statistical methods and software for processing and interpreting the data.

This assignment makes use of data from a personal activity monitoring device. This device collects data at 5 minute intervals throughout the day. The data consists of two months of data from an anonymous individual collected during the months of October and November 2012 and include the number of steps taken in 5 minute intervals each day.

The overall goal of this assignment is to make some basic exploratory data analysis to assess some activity patterns with regards to the anonymous individual's walking patterns. For each day, there are readings taken at particular 5-minute intervals. These readings correspond to the number of **steps** taken by the anonymous individual between the previous 5-minute interval to the current 5-minute interval.

Data Layout
===========

These intervals are assigned a unique ID which corresponds to the interval taken at that day. As an example, the ID of `15` means that at the third observation (i.e. at the 15 minute mark), that is when the reading was taken for this day. There are 61 days between October and November, and as such there are `61` observations recorded at the 15 minute mark. Furthermore, there are 61 observations taken at the same time interval as they were taken each day within these two months. As such, there are 61 observations for the 20 minute mark, the 30 minute mark, the 45 minute mark and so on. In addition, for each day, there are `288` intervals / observations were recorded at per day.

As such, the data provided to us is a 3-column data frame:

1.  Column 1 - *steps*: Indicates the number of steps taken during a neighbouring 5-minute interval (between 5 minutes and 10 minutes, 10 minutes and 15 minutes, etc.). There were some instances where there were no readings taken, most likely due to the fact that the subject was sleeping or the device was shut off. These are coded in as `NA` values.
2.  Column 2 - *date*: Indicates the date at which the measurement was taken. This is in the `YYYY-MM-DD` format.
3.  Column 3 - *interval*: The aforementioned ID that determines at which 5-minute interval the reading was taken at (5, 10, 15, 20, etc.)

Procedure
=========

Preamble
--------

There are five steps overall in our analysis:

1.  Loading in and preprocessing the data
2.  Plotting a histogram of the total number of steps taken each day and calculating the mean and median of each day.
3.  Determining the average daily activity pattern: Plotting a time-series plot for each 5-minute interval (the x-axis) with the average number of steps averaged across all days (the y-axis). This means that for each 5-minute interval, calculate the average of the 61 observations taken at each interval. We also calculate the median of the 61 observations taken at each interval as well.
4.  Imputing missing values: There were some instances where the amount of steps read for an interval were missing. These were coded in as `NA` values. The presence of missing days may introduce bias into some calculations or summaries of the data. In this step, a simple strategy was performed to replace the missing values in the dataset with a filler value. This value was chosen to be the **mean within the 5-minute interval the value was missing for**. For example, suppose on October 5, 2012, at the 10-minute interval there is missing data. This data is filled in by the mean of whatever data was available overall (the 61 observations excluding the `NA` values) for this particular interval. The analysis of Step \#2 is repeated with similarities and differences being reported.
5.  The last part of the analysis is to split up the data into weekdays and weekends and observe if there is any difference in activity patterns between these two classes. The same kind of plot is repeated like in Step \#3, but now there are two separate plots to reflect the activity on the weekdays and weekends.

Loading in and preprocessing data
---------------------------------

Before we begin, it is imperative that the reader set the working directory to be where this R Markdown file (and subsequently all of the data for the analysis) is located. This can be done with the `setwd()` command.

The first thing we obviously need to do is read in the data and clean it up so that it is presentable for analysis. Dr. Roger Peng, being the most generous person on the planet, has already cleaned up the data for us. However, the only thing that will be done is to convert the dates into a `POSIXlt` class for easier processing. The data have been provided in a `.zip` archive file. What we will do is unarchive the `.zip` file and read the data in using `read.csv()`. After, the dates will be transformed into the `POSIXlt` class.

``` r
dat <- read.csv("activity.csv") # Read in data file

# Turn the date data into a valid date class
# Allows for easier processing
# Dates are in YYYY-MM-DD format
dates <- strptime(dat$date, "%Y-%m-%d")
dat$date <- dates

# Keep a list of all possible days
uniqueDates <- unique(dates)
# Keep a list of all possible intervals
uniqueIntervals <- unique(dat$interval)
```

As for `uniqueDates` and `uniqueIntervals`, these are variables that store a list of all possible dates and intervals. These will primarily be used to help plot our necessary data and for data processing.

What is the mean total number of steps taken per day?
-----------------------------------------------------

First, let's plot a histogram of the total number of steps taken for each day. Before we do this, let's split up the data into individual data frames where each data frame represents the data for a particular day. After this, we will create a vector that accumulates all of the steps taken for each day and let it get stored into a vector. Each element in this vector represents the total number of steps taken for a particular day (61 in total). It should be noted that `NA` values will be ignored for the time being. After, we will plot a histogram where the x-axis represents the particular day in question, while the y-axis denotes how many steps were taken in total for each day. We will also calculate what the mean and median number of steps per day were as well.

``` r
# Part 2 - Create a histogram of the total number of steps taken
# each day
# First split up the data frame for steps by day
stepsSplit <- split(dat$steps, dates$yday)

# Next find the total number of steps over each day
totalStepsPerDay <- sapply(stepsSplit, sum, na.rm=TRUE)

# Plot a (pseudo) histogram where the x-axis denotes the day
# and the y-axis denotes the total number of steps taken 
# for each day
plot(uniqueDates, totalStepsPerDay, main="Histogram of steps taken each day", 
     xlab="Date (October to November 2012)", ylab="Frequency", type="h", lwd=4, col="blue")
```

![](PA1_template_files/figure-markdown_github/unnamed-chunk-2-1.png)
 The mean steps per day are:

``` r
meanStepsPerDay <- sapply(stepsSplit, mean, na.rm=TRUE)
meanDataFrame <- data.frame(date=uniqueDates, meanStepsPerDay=meanStepsPerDay, row.names=NULL)
meanDataFrame
```

    ##          date meanStepsPerDay
    ## 1  2012-10-01             NaN
    ## 2  2012-10-02       0.4375000
    ## 3  2012-10-03      39.4166667
    ## 4  2012-10-04      42.0694444
    ## 5  2012-10-05      46.1597222
    ## 6  2012-10-06      53.5416667
    ## 7  2012-10-07      38.2465278
    ## 8  2012-10-08             NaN
    ## 9  2012-10-09      44.4826389
    ## 10 2012-10-10      34.3750000
    ## 11 2012-10-11      35.7777778
    ## 12 2012-10-12      60.3541667
    ## 13 2012-10-13      43.1458333
    ## 14 2012-10-14      52.4236111
    ## 15 2012-10-15      35.2048611
    ## 16 2012-10-16      52.3750000
    ## 17 2012-10-17      46.7083333
    ## 18 2012-10-18      34.9166667
    ## 19 2012-10-19      41.0729167
    ## 20 2012-10-20      36.0937500
    ## 21 2012-10-21      30.6284722
    ## 22 2012-10-22      46.7361111
    ## 23 2012-10-23      30.9652778
    ## 24 2012-10-24      29.0104167
    ## 25 2012-10-25       8.6527778
    ## 26 2012-10-26      23.5347222
    ## 27 2012-10-27      35.1354167
    ## 28 2012-10-28      39.7847222
    ## 29 2012-10-29      17.4236111
    ## 30 2012-10-30      34.0937500
    ## 31 2012-10-31      53.5208333
    ## 32 2012-11-01             NaN
    ## 33 2012-11-02      36.8055556
    ## 34 2012-11-03      36.7048611
    ## 35 2012-11-04             NaN
    ## 36 2012-11-05      36.2465278
    ## 37 2012-11-06      28.9375000
    ## 38 2012-11-07      44.7326389
    ## 39 2012-11-08      11.1770833
    ## 40 2012-11-09             NaN
    ## 41 2012-11-10             NaN
    ## 42 2012-11-11      43.7777778
    ## 43 2012-11-12      37.3784722
    ## 44 2012-11-13      25.4722222
    ## 45 2012-11-14             NaN
    ## 46 2012-11-15       0.1423611
    ## 47 2012-11-16      18.8923611
    ## 48 2012-11-17      49.7881944
    ## 49 2012-11-18      52.4652778
    ## 50 2012-11-19      30.6979167
    ## 51 2012-11-20      15.5277778
    ## 52 2012-11-21      44.3993056
    ## 53 2012-11-22      70.9270833
    ## 54 2012-11-23      73.5902778
    ## 55 2012-11-24      50.2708333
    ## 56 2012-11-25      41.0902778
    ## 57 2012-11-26      38.7569444
    ## 58 2012-11-27      47.3819444
    ## 59 2012-11-28      35.3576389
    ## 60 2012-11-29      24.4687500
    ## 61 2012-11-30             NaN

The median steps per day are:

``` r
medianStepsPerDay <- sapply(stepsSplit, median, na.rm=TRUE)
medianDataFrame <- data.frame(date=uniqueDates, medianStepsPerDay=medianStepsPerDay, row.names=NULL)
medianDataFrame
```

    ##          date medianStepsPerDay
    ## 1  2012-10-01                NA
    ## 2  2012-10-02                 0
    ## 3  2012-10-03                 0
    ## 4  2012-10-04                 0
    ## 5  2012-10-05                 0
    ## 6  2012-10-06                 0
    ## 7  2012-10-07                 0
    ## 8  2012-10-08                NA
    ## 9  2012-10-09                 0
    ## 10 2012-10-10                 0
    ## 11 2012-10-11                 0
    ## 12 2012-10-12                 0
    ## 13 2012-10-13                 0
    ## 14 2012-10-14                 0
    ## 15 2012-10-15                 0
    ## 16 2012-10-16                 0
    ## 17 2012-10-17                 0
    ## 18 2012-10-18                 0
    ## 19 2012-10-19                 0
    ## 20 2012-10-20                 0
    ## 21 2012-10-21                 0
    ## 22 2012-10-22                 0
    ## 23 2012-10-23                 0
    ## 24 2012-10-24                 0
    ## 25 2012-10-25                 0
    ## 26 2012-10-26                 0
    ## 27 2012-10-27                 0
    ## 28 2012-10-28                 0
    ## 29 2012-10-29                 0
    ## 30 2012-10-30                 0
    ## 31 2012-10-31                 0
    ## 32 2012-11-01                NA
    ## 33 2012-11-02                 0
    ## 34 2012-11-03                 0
    ## 35 2012-11-04                NA
    ## 36 2012-11-05                 0
    ## 37 2012-11-06                 0
    ## 38 2012-11-07                 0
    ## 39 2012-11-08                 0
    ## 40 2012-11-09                NA
    ## 41 2012-11-10                NA
    ## 42 2012-11-11                 0
    ## 43 2012-11-12                 0
    ## 44 2012-11-13                 0
    ## 45 2012-11-14                NA
    ## 46 2012-11-15                 0
    ## 47 2012-11-16                 0
    ## 48 2012-11-17                 0
    ## 49 2012-11-18                 0
    ## 50 2012-11-19                 0
    ## 51 2012-11-20                 0
    ## 52 2012-11-21                 0
    ## 53 2012-11-22                 0
    ## 54 2012-11-23                 0
    ## 55 2012-11-24                 0
    ## 56 2012-11-25                 0
    ## 57 2012-11-26                 0
    ## 58 2012-11-27                 0
    ## 59 2012-11-28                 0
    ## 60 2012-11-29                 0
    ## 61 2012-11-30                NA

The median measure may look a bit puzzling, but if you observe the data for the amount of steps over some of the days, you will see that a majority of the steps taken are zero and so the mean would logically represent the 50th percentile. Here is a sample between October 10th to October 12th of 2012:

``` r
stepsSplit[10:12]
```

    ## $`283`
    ##   [1]  34  18   7   0   0   0   0   0   0   0   0   0   0   0   0   0   0
    ##  [18]   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0
    ##  [35]   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0
    ##  [52]   0   0   0   0   0   0   0   0   0   0  34   0   0   0   0   0   0
    ##  [69]   0   0   0   0   0   7   9  36   0  47  67   0  49  23  15  29  42
    ##  [86]  49  92  28  33  63  97  90 101  55  75  40  47  22  61   0   0   0
    ## [103]   0   0   0  60  54  16 135  61  69  32   0   0  17   0   0  69   0
    ## [120]  20 400 105 292 291  30   0   0  40  38   0   0   0   0   0   0  72
    ## [137]  37   0   0  25  17   0   0  88   7 413 326  93 334 317   0   0   0
    ## [154]   0  68 129   0   0   0   0   0   0   0   0   0   0   0   0   0   0
    ## [171] 103 119   0   0   0  70 125   0   0   0   0   0   0   0   0   0 176
    ## [188]  71  43 340   7  13  15   0   0   0   0   0   0   0   0   0   0  15
    ## [205]  50 271 106 272 308   0   0 111 281  11 139  36   0   0   0   0   0
    ## [222]   0  58  63 260  82 310   0   0   0   8  12 364 219   0   0   0 174
    ## [239] 205  12   0   0  11  17   0   0  37   0   0 105  34   0 152   0   0
    ## [256]   0   0   0   0   0   0   0   0   0   0   0   0 112  23  12   8   0
    ## [273]   0   0   0   0   7   0   0   0   0   0   0   0   0   0   8   0
    ## 
    ## $`284`
    ##   [1]   0   0   0   8   0   0   0   0   0   0   0   0   0   0   8   0   0
    ##  [18]   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0
    ##  [35]   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0
    ##  [52]   0   0   0   0   0 139  15   0   0   0   0   0   0   0   0   0   0
    ##  [69]   0   0   0  11   0  10  40   0   0  32  34 105  33   8  16  18   0
    ##  [86]   9   0   0  27  22   0  50   0   0   0  23  43  70 619 743 446 748
    ## [103] 424 747 739 741 726 166 548 343  13  26  64   0   0   0   0   0   0
    ## [120]   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0
    ## [137]   0   7  46   0   0   0   0   0   0   0   0  31  45   0   0   0   0
    ## [154]   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0
    ## [171]   0   0   0   0  22  27   0   0   0   0   0   0   0   0   0   0  75
    ## [188] 119 395  78 292 416  35   0   0  27  32   0   0   0   0   0  49  57
    ## [205]  34   0   0   0   0   0  39  30   9  41   7   0   0  40  22  31  19
    ## [222]   0   8  22  62  60   0   0   0   0   0   0   0   0   0   0   0   0
    ## [239]   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0  95
    ## [256]   0  91  50  31   0   0   0  20  11   0   0   0   0   0   0   0   0
    ## [273]   0   0   0   0  11   0   0   0   0   4   0   0   0   0   0   0
    ## 
    ## $`285`
    ##   [1]   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0
    ##  [18]   0  38   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0
    ##  [35]   7   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0
    ##  [52]   0   0   0  48   0   0   0   0   0   0   0   0   0   0   0   0   0
    ##  [69]  30  92   0  11   0  10  19 111  38  16  29   9  45  35  53  43   8
    ##  [86]  40   0  32  57  35 117 117  25  95  29 141  51 123 440 687 614 474
    ## [103] 750 742 770 735 746 748 802 280  31   0   0   0   0   0   0   7  92
    ## [120]   0   0   0   0   0  46   7   0 328 156   0   0   0 129 339 150   0
    ## [137]   0   0  70   0   9   0   0   0  70   0   0   0   0   0   0   0  18
    ## [154]  91   0   0   0  75   0   0   0   0  99   0   0   0   0  96  16  20
    ## [171] 144 321 267   0   0   0   0   0   9   0   0  24  78   0  26  35   0
    ## [188]   0   0 365  90 432 275  34   0  92  15   0   0   0   0  20  10   9
    ## [205]   0   0  32  24   0   0  38  40  19  71   2  21   0 433 463 511 298
    ## [222] 500 473 506  24  35  41  46   0   0   0  16  23   0   0   0  18  54
    ## [239]  36   0   0   0   0   0   0   0   0   0   0  18  30  23  70 113   0
    ## [256]   0   0   0   0   0   0   0   0   0   0   0   0   9   0   0   8   0
    ## [273]   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0

What is the average daily activity pattern?
-------------------------------------------

What we now need to do is split up this data again so that individual data frames represent the steps taken **over each time interval**. As such, there will be a data frame for interval 5, another data frame for interval 10 and so on. Once we extract out these individual data frames, we thus compute the mean for each time interval. It is imperative to note that we again will ignore `NA` values. We will thus plot the data as a time-series plot (of `type="l"`). Once we have done this, we will locate where in the time-series plot the maximum is located and will draw a red vertical line to denote this location:

``` r
# Part 3 - Time-series plot (type="l")
# x-axis - Time interval (5, 10, 15, ...)
# y-axis - Average number of steps taken across all days for this time interval

# Split up the data according to the interval
intervalSplit <- split(dat$steps, dat$interval)

# Find the average amount of steps per time interval - ignore NA values
averageStepsPerInterval <- sapply(intervalSplit, mean, na.rm=TRUE)

# Plot the time-series graph
plot(uniqueIntervals, averageStepsPerInterval, type="l",
     main="Average number of steps per interval across all days", 
     xlab="Interval", ylab="Average # of steps across all days", 
     lwd=2, col="blue")

# Find the location of where the maximum is
maxIntervalDays <- max(averageStepsPerInterval, na.rm=TRUE)
maxIndex <- as.numeric(which(averageStepsPerInterval == maxIntervalDays))

# Plot a vertical line where the max is
maxInterval <- uniqueIntervals[maxIndex]
abline(v=maxInterval, col="red", lwd=3)
```

![](PA1_template_files/figure-markdown_github/unnamed-chunk-6-1.png)
 With reference to the above plot, the interval that records the maximum number of steps averaged across all days is:

``` r
maxInterval
```

    ## [1] 835

Imputing missing values
-----------------------

First, let's calculate the total number of missing values there are. This denotes the total number of observations that did not have any steps recorded (i.e. those rows which are `NA`)

``` r
# Part 4 - Calculate total amount of missing values in the data set
# Use complete.cases to find a logical vector that returns TRUE
# if it is a complete row (a.k.a. no NA values) and FALSE otherwise
completeRowsBool <- complete.cases(dat$steps)
numNA <- sum(as.numeric(!completeRowsBool))
numNA
```

    ## [1] 2304

The strategy that we will use to fill in the missing values in the data set is to replace all `NA` values with the mean of that particular 5-minute interval the observation falls on. Now that we have devised this strategy, let's replace all of the `NA` values with the aforementioned strategy.

``` r
# Modify the meanStepsPerDay vector that contains the mean steps taken
# for this 5 minute interval
# Each day consists of 288 intervals and there are 61 days in total
# First remove NaN values and replace with 0.  
# NaN values are produced when the entire day was filled with NA values
# Essentially the mean and median would be zero anyway!
meanStepsPerDay[is.nan(meanStepsPerDay)] <- 0

# Now create a replicated vector 288 times
# The reason why we're doing this is because the slots
# in the vector naturally line up with the interval for
# a particular day.  Now, all we have to do is find where
# in the data set there are missing steps, and simply do
# a copy from one vector to the other
meanColumn <- rep(meanStepsPerDay, 288)

# The steps before replacement
rawSteps <- dat$steps

# Find any values that are NA in the raw steps data
stepsNA <- is.na(rawSteps)

# Now replace these values with their corresponding mean
rawSteps[stepsNA] <- meanColumn[stepsNA]

# Throw these back into a new data frame
datNew <- dat
datNew$steps <- rawSteps
```

Now that this is finished, let's plot a histogram of the new data:

``` r
# Repeat Part 2 now
# First split up the data frame for steps by day
stepsSplitNew <- split(datNew$steps, dates$yday)

# Next find the total number of steps over each day
# There should not be an NA values and so we don't need
# to set the flag
totalStepsPerDayNew <- sapply(stepsSplitNew, sum)

# Plot a (pseudo) histogram where the x-axis denotes the day
# and the y-axis denotes the total number of steps taken 
# for each day
par(mfcol=c(2,1))
# Plot the original histogram first
plot(uniqueDates, totalStepsPerDay, main="Histogram of steps taken each day before imputing", 
     xlab="Date (October to November 2012)", ylab="Frequency", type="h", lwd=4, col="blue")
# Plot the modified histogram after
plot(uniqueDates, totalStepsPerDayNew, main="Histogram of steps taken each day after imputing", 
     xlab="Date (October to November 2012)", ylab="Frequency", type="h", lwd=4, col="blue")
```

![](PA1_template_files/figure-markdown_github/unnamed-chunk-10-1.png)
 With this new data, let's calculate the mean over all days (like in Part 2). As a side-by-side comparison, we will place the data before imputing, as well as the new one in the same data frame. Bear in mind that we have replaced all of the `NaN` values to `0`. As such, the mean steps per day of the new data are:

``` r
meanStepsPerDayNew <- sapply(stepsSplitNew, mean)
meanDataFrameNew <- data.frame(date=uniqueDates, meanStepsPerDay=meanStepsPerDay, 
                               meanStepsPerDayNew=meanStepsPerDayNew, row.names=NULL)
meanDataFrameNew
```

    ##          date meanStepsPerDay meanStepsPerDayNew
    ## 1  2012-10-01       0.0000000         32.3355276
    ## 2  2012-10-02       0.4375000          0.4375000
    ## 3  2012-10-03      39.4166667         39.4166667
    ## 4  2012-10-04      42.0694444         42.0694444
    ## 5  2012-10-05      46.1597222         46.1597222
    ## 6  2012-10-06      53.5416667         53.5416667
    ## 7  2012-10-07      38.2465278         38.2465278
    ## 8  2012-10-08       0.0000000         32.2632378
    ## 9  2012-10-09      44.4826389         44.4826389
    ## 10 2012-10-10      34.3750000         34.3750000
    ## 11 2012-10-11      35.7777778         35.7777778
    ## 12 2012-10-12      60.3541667         60.3541667
    ## 13 2012-10-13      43.1458333         43.1458333
    ## 14 2012-10-14      52.4236111         52.4236111
    ## 15 2012-10-15      35.2048611         35.2048611
    ## 16 2012-10-16      52.3750000         52.3750000
    ## 17 2012-10-17      46.7083333         46.7083333
    ## 18 2012-10-18      34.9166667         34.9166667
    ## 19 2012-10-19      41.0729167         41.0729167
    ## 20 2012-10-20      36.0937500         36.0937500
    ## 21 2012-10-21      30.6284722         30.6284722
    ## 22 2012-10-22      46.7361111         46.7361111
    ## 23 2012-10-23      30.9652778         30.9652778
    ## 24 2012-10-24      29.0104167         29.0104167
    ## 25 2012-10-25       8.6527778          8.6527778
    ## 26 2012-10-26      23.5347222         23.5347222
    ## 27 2012-10-27      35.1354167         35.1354167
    ## 28 2012-10-28      39.7847222         39.7847222
    ## 29 2012-10-29      17.4236111         17.4236111
    ## 30 2012-10-30      34.0937500         34.0937500
    ## 31 2012-10-31      53.5208333         53.5208333
    ## 32 2012-11-01       0.0000000         32.0149498
    ## 33 2012-11-02      36.8055556         36.8055556
    ## 34 2012-11-03      36.7048611         36.7048611
    ## 35 2012-11-04       0.0000000         32.4504726
    ## 36 2012-11-05      36.2465278         36.2465278
    ## 37 2012-11-06      28.9375000         28.9375000
    ## 38 2012-11-07      44.7326389         44.7326389
    ## 39 2012-11-08      11.1770833         11.1770833
    ## 40 2012-11-09       0.0000000         32.3078945
    ## 41 2012-11-10       0.0000000         32.8706718
    ## 42 2012-11-11      43.7777778         43.7777778
    ## 43 2012-11-12      37.3784722         37.3784722
    ## 44 2012-11-13      25.4722222         25.4722222
    ## 45 2012-11-14       0.0000000         32.9865210
    ## 46 2012-11-15       0.1423611          0.1423611
    ## 47 2012-11-16      18.8923611         18.8923611
    ## 48 2012-11-17      49.7881944         49.7881944
    ## 49 2012-11-18      52.4652778         52.4652778
    ## 50 2012-11-19      30.6979167         30.6979167
    ## 51 2012-11-20      15.5277778         15.5277778
    ## 52 2012-11-21      44.3993056         44.3993056
    ## 53 2012-11-22      70.9270833         70.9270833
    ## 54 2012-11-23      73.5902778         73.5902778
    ## 55 2012-11-24      50.2708333         50.2708333
    ## 56 2012-11-25      41.0902778         41.0902778
    ## 57 2012-11-26      38.7569444         38.7569444
    ## 58 2012-11-27      47.3819444         47.3819444
    ## 59 2012-11-28      35.3576389         35.3576389
    ## 60 2012-11-29      24.4687500         24.4687500
    ## 61 2012-11-30       0.0000000         32.2280213

Like the above, the median steps per day are:

``` r
medianStepsPerDayNew <- sapply(stepsSplitNew, median)
medianDataFrameNew <- data.frame(date=uniqueDates, medianStepsPerDay=medianStepsPerDay, 
                                 medianStepsPerDayNew=medianStepsPerDayNew, row.names=NULL)
medianDataFrameNew
```

    ##          date medianStepsPerDay medianStepsPerDayNew
    ## 1  2012-10-01                NA             36.09375
    ## 2  2012-10-02                 0              0.00000
    ## 3  2012-10-03                 0              0.00000
    ## 4  2012-10-04                 0              0.00000
    ## 5  2012-10-05                 0              0.00000
    ## 6  2012-10-06                 0              0.00000
    ## 7  2012-10-07                 0              0.00000
    ## 8  2012-10-08                NA             36.09375
    ## 9  2012-10-09                 0              0.00000
    ## 10 2012-10-10                 0              0.00000
    ## 11 2012-10-11                 0              0.00000
    ## 12 2012-10-12                 0              0.00000
    ## 13 2012-10-13                 0              0.00000
    ## 14 2012-10-14                 0              0.00000
    ## 15 2012-10-15                 0              0.00000
    ## 16 2012-10-16                 0              0.00000
    ## 17 2012-10-17                 0              0.00000
    ## 18 2012-10-18                 0              0.00000
    ## 19 2012-10-19                 0              0.00000
    ## 20 2012-10-20                 0              0.00000
    ## 21 2012-10-21                 0              0.00000
    ## 22 2012-10-22                 0              0.00000
    ## 23 2012-10-23                 0              0.00000
    ## 24 2012-10-24                 0              0.00000
    ## 25 2012-10-25                 0              0.00000
    ## 26 2012-10-26                 0              0.00000
    ## 27 2012-10-27                 0              0.00000
    ## 28 2012-10-28                 0              0.00000
    ## 29 2012-10-29                 0              0.00000
    ## 30 2012-10-30                 0              0.00000
    ## 31 2012-10-31                 0              0.00000
    ## 32 2012-11-01                NA             35.93576
    ## 33 2012-11-02                 0              0.00000
    ## 34 2012-11-03                 0              0.00000
    ## 35 2012-11-04                NA             36.17014
    ## 36 2012-11-05                 0              0.00000
    ## 37 2012-11-06                 0              0.00000
    ## 38 2012-11-07                 0              0.00000
    ## 39 2012-11-08                 0              0.00000
    ## 40 2012-11-09                NA             35.93576
    ## 41 2012-11-10                NA             36.09375
    ## 42 2012-11-11                 0              0.00000
    ## 43 2012-11-12                 0              0.00000
    ## 44 2012-11-13                 0              0.00000
    ## 45 2012-11-14                NA             36.09375
    ## 46 2012-11-15                 0              0.00000
    ## 47 2012-11-16                 0              0.00000
    ## 48 2012-11-17                 0              0.00000
    ## 49 2012-11-18                 0              0.00000
    ## 50 2012-11-19                 0              0.00000
    ## 51 2012-11-20                 0              0.00000
    ## 52 2012-11-21                 0              0.00000
    ## 53 2012-11-22                 0              0.00000
    ## 54 2012-11-23                 0              0.00000
    ## 55 2012-11-24                 0              0.00000
    ## 56 2012-11-25                 0              0.00000
    ## 57 2012-11-26                 0              0.00000
    ## 58 2012-11-27                 0              0.00000
    ## 59 2012-11-28                 0              0.00000
    ## 60 2012-11-29                 0              0.00000
    ## 61 2012-11-30                NA             35.93576

By looking at the above data frames, the only values that have changed are those days where all of the observations were missing (i.e. those days having all zeroes / `NA`). The rest of the observations have stayed the same, which actually make sense. The reason why this is is because if you insert more data values into a vector that are the mean of that particular vector, taking the mean will still give you the same answer. With regards to the median vector, those rows that were filled with `NA` had all of their observations on that day filled with the mean value and so the median would obviously be the mean as well.

Are there differences in activity patterns between weekdays and weekends?
-------------------------------------------------------------------------

With the new data set we have just created, we are going to split up the data into two data frames - one data frame consists of all steps taken on a weekday, while the other data frame consists of all steps taken on a weekend. The following `R` code illustrates this for us:

``` r
# Part 5 - Now split up the data so that it's sorted by weekday or weekend
# We have casted the dates to a POSIXlt class so wday is part of this class
# wday is an integer ranging from 0 to 6 that represents the day of the week
# 0 is for Sunday, 1 is for Monday, going up to 6 for Saturday
# Store this into wdays
wdays <- dates$wday

# Create a new factor variable that classifies the day as either a weekday or weekend
# First, create a numeric vector with 2 levels - 1 is for a weekday, 2 for a weekend
classifywday <- rep(0, 17568) # 17568 observations overall

# Any days that are from Monday to Friday, set the numeric vector in these positions
# as 1
classifywday[wdays >= 1 & wdays <= 5] <- 1

# Any days that are on Saturday or Sunday, set the numeric vector in these positions
# as 2
classifywday[wdays == 6 | wdays == 0] <- 2

# Create a new factor variable that has labels Weekdays and Weekends
daysFactor <- factor(classifywday, levels=c(1,2), labels=c("Weekdays", "Weekends"))

# Create a new column that contains this factor for each day
datNew$typeOfDay <- daysFactor

# Now split up into two data frames
datWeekdays <- datNew[datNew$typeOfDay == "Weekdays", ]
datWeekends <- datNew[datNew$typeOfDay == "Weekends", ]
```

Now that we have accomplished this, let's split up the data for each data frame so that we will have two sets of individual data frames. One set is for weekdays and within this data frame are individual data frames. Each data frame contains the steps for each interval recorded on a weekday. The other set is for weekends, and within this data frame are individual data frames. Like previously, each data frame here contains the steps for each interval recorded on a weekday. Once we have these two sets of data frames, we will now calculate the mean amount of steps for each interval for the weekdays data frame and weekends data frame. This will result in two vectors - one for the weekdays and the other for weekends. The following `R` code does this for us:

``` r
# Further split up the Weekdays and Weekends into their own intervals
datSplitWeekdays <- split(datWeekdays$steps, datWeekdays$interval)
datSplitWeekends <- split(datWeekends$steps, datWeekends$interval)

# Find the average for each interval
meanStepsPerWeekdayInterval <- sapply(datSplitWeekdays, mean)
meanStepsPerWeekendInterval <- sapply(datSplitWeekends, mean)

par(mfcol=c(2,1))
plot(uniqueIntervals, meanStepsPerWeekdayInterval, type="l",
     main="Average number of steps per interval across all weekdays", 
     xlab="Interval", ylab="Average # of steps across all weekdays", 
     lwd=2, col="blue")
plot(uniqueIntervals, meanStepsPerWeekendInterval, type="l",
     main="Average number of steps per interval across all weekends", 
     xlab="Interval", ylab="Average # of steps across all weekends", 
     lwd=2, col="blue")
```

![](PA1_template_files/figure-markdown_github/unnamed-chunk-14-1.png)
 What is interesting about this plot is that it very much sums up the activity that any normal person would undergo depending on whether it is a weekday or weekend. For both days, the intervals between `0` and about `525` are uniform. This most likely represents when the subject was sleeping. The differences start at between `525` and roughly `800`. On the weekdays, movement is most likely attributed to the subject getting ready to go to work or starting their day. On the weekends, movement is less frequent. This could be attributed to the subject sleeping in and perhaps making breakfast to start their day.

There is a huge jump at roughly `830` for the weekdays. This could be attributed to the subject walking or making their way to work. On the weekends this behaviour is mimicked as well, perhaps due to making their way to begin engaging in some extracurricular activities that the subject is engaged in. Now where it really gets different is at roughly the `1000` mark. On the weekdays, this could be reflected with the subject being at work and there is not much movement. A good indication of this behaviour could be attributed to the fact that the subject has a low stress job that requires minimal movement. On the weekends, the behaviour varies wildly. This could be attributed to the extra amount of movement required to engage in such extracurricular activities.

Where it starts to become similar again is at roughly the `1700` mark. This could mean that during the weekdays, the subject is getting ready to head home for the day which is why there is an increased amount of moment in comparison to the interval between `1000` to `1700`. At the same time with the weekends, there is a relative amount of increased moment after the `1700` mark but it could reflect that at this time of day, the subject may be socializing.

All in all, this seems to reflect the behaviour of an individual going through a normal work day during the weekdays, and having a relaxing time on the weekends.
