---
title: "Course_project_01"
author: "as"
date: "2023-10-24"
output: html_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

## First part of the assingment is loading the data

```{r loading}
data <- read.csv("activity.csv")
```

Next, I need to calculate the total number of steps taken each day, ignoring the missing values.
I will do this by grouping the data by column "date".
Finally, a plot shows a total number of steps taken for each day.

```{r grouping data}
library(dplyr)
grouped <- data %>%
  group_by(date) %>%
  summarise(total = sum(steps, na.rm = TRUE) )

grouped$date <- as.Date(grouped$date, "%Y-%m-%d")
days <- seq(1, length(grouped$date))
barplot(grouped$total~days,offset = 0, 
        main = "Total number of steps taken each day",
        xlab = "Days", ylab = "Total number of steps",
        axis.lty = 1)
```

The mean value of the total number of steps taken per day is `r mean(grouped$total)`, while the median is equal to `r median(grouped$total)`.

The second part is looking at the specific time interval during the day. This means that I need to group the data by "interval" column. After grouping the data, I need to calculate the average of steps taken at that interval, for all days in question.

```{r Average daily activity pattern}

interval_grouped <- data %>%
  group_by(interval) %>%
  summarise(average= mean(steps, na.rm = TRUE))
index <- which.max(interval_grouped$average)
plot(interval_grouped, type = "l", 
     xlab = "Time interval", ylab = "Average number of steps taken")
```
The maximum average number of steps is taken at `r interval_grouped[index, 1]`, meaning 8:35.

Following, I have to calculate the total number of the missing data. Total number of the missing data is `r sum(is.na(data$steps))`.
For the replacement data, I will just place the mean value of total number of steps divided by the squared number of intervals.

```{r Replacing missing data}
library(tidyr)
mean_value = mean(grouped$total)/(length(grouped$total)**2)
new_dataset <- data.frame(data)
new_dataset$steps[is.na(new_dataset$steps)] <- mean_value

new_dataset <- new_dataset %>%
  group_by(date) %>%
  summarise(total = sum(steps))

head(new_dataset)
days <- seq(1, length(new_dataset$date))
barplot(new_dataset$total~days,offset = 0, 
        main = "Total number of steps taken each day",
        xlab = "Days", ylab = "Total number of steps",
        axis.lty = 1)
```

The mean value of the total number of steps taken per day is `r mean(new_dataset$total)`, while the median is equal to `r median(new_dataset$total)`. The mean value is slightly higher while the median is the same.

Next, I have to investigate are there any difference in activity patterns between weekdays and weekends. I will be using the new data-set.

```{r Weekdays vs weekends}
library(ggplot2)
new_activity <- data.frame(data)
new_activity$steps[is.na(new_activity$steps)] <- mean_value

new_activity$date <- weekdays(as.Date(new_activity$date, "%Y-%m-%d"), abbreviate = TRUE)

weekdays1 <- c('pon', 'uto', 'sri', 'čet', 'pet')

new_activity$wDay <- factor(new_activity$date %in% weekdays1, 
         levels=c(FALSE, TRUE), labels=c('weekend', 'weekday'))

new_activity <- new_activity %>%
  group_by(wDay, interval) %>%
  summarise(average = mean(steps))

qplot(interval, average, data = new_activity, color = factor(wDay), geom = "line",
      xlab = "Interval", ylab = "Average number of steps taken") + labs(colour = 'Type of day')