# Cyclistic-Bike-share
---
title: "Cyclistic bike share analysis"
author: "Shobowale Ayomide"
date: "2022-10-18"
output: html_document
---

# Processing

The processing stage involves cleaning the data after confirming that it is from a good source and it satisfies the necessary conditions.It also involves confirming that the data is complete, correct and relevant. I chose R studio as the tool to use for this phase of the analysis process. I started by installing and then loading the relevant packages.
```{r}
install.packages("tidyverse")
```
```{r}
install.packages("lubridate")
```
```{r}
install.packages("ggplot2")
```

```{r}
library(tidyverse)
library(lubridate)
library(ggplot2)
```



```{r}
install.packages("fs")
```



### Loading datasets into the environment with the correct naming conventions.


```{r}
October_2021 <- read_csv("202110-divvy-tripdata.csv")
November_2021 <- read_csv("202111-divvy-tripdata.csv")
December_2021 <- read_csv("202112-divvy-tripdata.csv")
January_2022 <- read_csv("202201-divvy-tripdata.csv")
February_2022 <- read_csv("202202-divvy-tripdata.csv")
March_2022 <- read_csv("202203-divvy-tripdata.csv")
April_2022 <- read_csv("202204-divvy-tripdata.csv")
May_2022 <- read_csv("202205-divvy-tripdata.csv")
June_2022 <- read_csv("202206-divvy-tripdata.csv")
July_2022 <- read_csv("202207-divvy-tripdata.csv")
August_2022 <- read_csv("202208-divvy-tripdata.csv")
September_2022 <- read_csv("202209-divvy-publictripdata.csv")
```


### Compare column names each of the files
I now have to compare the column names before im able to join the datasets.


```{r}
#comparing columns
colnames(October_2021)
colnames(November_2021)
colnames(December_2021)
colnames(January_2022)
colnames(February_2022)
colnames(March_2022)
colnames(April_2022)
colnames(May_2022)
colnames(June_2022)
colnames(July_2022)
colnames(August_2022)
colnames(September_2022)
```

The column names are consistent, so i can now inspect the data frame

```{r}
#Inspecting dataframes

str(October_2021)
str(November_2021)
str(December_2021)
str(January_2022)
str(February_2022)
str(March_2022)
str(April_2022)
str(May_2022)
str(June_2022)
str(July_2022)
str(August_2022)
str(September_2022)

```
They are all consistent so i can now move forward and join the data frames into one 

```{r}
#Binding dataframes

all_trips <- bind_rows(October_2021,November_2021,December_2021,January_2022,February_2022,March_2022,April_2022,May_2022,June_2022,July_2022,August_2022,September_2022)

```


## Cleaning up the data to prepare for Analysis
Running some functions to get a better idea of the data frame.
```{r}
# Inspect the new table that has been created

colnames(all_trips)  #List of column names

```
```{r}
nrow(all_trips)  #How many rows are in data frame?
```
```{r}
dim(all_trips)  #Dimensions of the data frame?
```
```{r}
head(all_trips)  #See the first 6 rows of data frame.
```
```{r}

str(all_trips)  #See list of columns and data types 
```
```{r}
summary(all_trips)  #Statistical summary of data
```
```{r}
all_trips <-  all_trips %>% 
  mutate(member_casual = recode(member_casual
                           ,"Subscriber" = "member"
                           ,"Customer" = "casual"))

```

```{r}
table(all_trips$member_casual)

```

 We will now add columns that list the date, month, day, and year of each ride


```{r}
# This will allow us to aggregate ride data for each month, day, or year

all_trips$date <- as.Date(all_trips$started_at) 
all_trips$month <- format(as.Date(all_trips$date), "%m")
all_trips$day <- format(as.Date(all_trips$date), "%d")
all_trips$year <- format(as.Date(all_trips$date), "%Y")
all_trips$day_of_week <- format(as.Date(all_trips$date), "%A")

```

```{r}
head(all_trips)
```

Adding the ride length. This can be gotten from the difference between the “started_at” and “ended_at” time of each trip.

```{r}
#Adding a ride_length calculation

all_trips$ride_length <- difftime(all_trips$ended_at,all_trips$started_at)


```

```{r}
str(all_trips)
```

 Convert "ride_length" from Factor to numeric so we can run calculations on the data

```{r}
#Converting ride_length to numeric
is.factor(all_trips$ride_length)
all_trips$ride_length <- as.numeric(as.character(all_trips$ride_length))
is.numeric(all_trips$ride_length)


```

Checking the structure of the dataset again to confirm that each variable is in the right form for analysis.
```{r}
str(all_trips)
```
### Iwould now get rid of null values

```{r}
#removing null values
all_trips_v2 <- drop_na(all_trips)
```


```{r}
str(all_trips_v2)
```
### Deleting the entries with negative ride_length since the end time should ideally not be earlier than the start time.
```{r}
# Removing negative values in ride_length
all_trips_v2 <- filter (all_trips_v2, ride_length > 0)

```

```{r}
#Extracting the longitude and latitude data from the data set as it will not be needed in this phase of the analysis.
lat_lng1 <- select(all_trips_v2,start_station_name,end_station_name,start_lat,start_lng,end_lat,end_lng,member_casual)
```
```{r}
all_trips_v2 <- all_trips_v2 %>%
select (-c(start_lat, start_lng, end_lat, end_lng))
```

```{r}
head(all_trips_v2)
```
After I concluded that my data was in the correct format and has been properly cleaned, I moved on to the next phase of the analysis process..

# Analysis
Analysis is the process of making sense of the data collected (and cleaned). The goal is to identify trends and relationships within the data that will help in solving the business task. The main purpose of the analysis was to deduce all the ways that cyclist members and cyclist casuals used the bicycles differently, in line with the business task.
```{r}
# Descriptive analysis on ride_length (all figures in seconds)
mean(all_trips_v2$ride_length) 
median(all_trips_v2$ride_length) 
max(all_trips_v2$ride_length) 
min(all_trips_v2$ride_length)

```
```{r}
summary(all_trips_v2$ride_length)

```

```{r}
# Compare members and casual users
aggregate(all_trips_v2$ride_length ~ all_trips_v2$member_casual, FUN = mean)
aggregate(all_trips_v2$ride_length ~ all_trips_v2$member_casual, FUN = median)
aggregate(all_trips_v2$ride_length ~ all_trips_v2$member_casual, FUN = max)
aggregate(all_trips_v2$ride_length ~ all_trips_v2$member_casual, FUN = min)


```

```{r}
# See the average ride time by each day for members vs casual users
aggregate(all_trips_v2$ride_length ~ all_trips_v2$member_casual + all_trips_v2$day_of_week, FUN = mean)

```
```{r}
# Notice that the days of the week are out of order. Let's fix that.

all_trips_v2$day_of_week <- ordered(all_trips_v2$day_of_week, levels=c("Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"))


```


```{r}
# Now, let's run the average ride time by each day for members vs casual users
aggregate(all_trips_v2$ride_length ~ all_trips_v2$member_casual + all_trips_v2$day_of_week, FUN = mean)

```
Analyze ridership data by type and weekday
`
```{r}
# analyze ridership data by type and weekday
all_trips_v2 %>% 
  mutate(weekday = wday(started_at, label = TRUE)) %>%  #creates weekday field using wday()
  group_by(member_casual, weekday) %>%  #groups by usertype and weekday
  summarise(number_of_rides = n()							#calculates the number of rides and average duration 
  ,average_duration = mean(ride_length)) %>% 		# calculates the average duration
  arrange(member_casual, weekday)		
```
Most popular start stations for casual riders
```{r}

all_trips_v2 %>%
  group_by(start_station_name, member_casual) %>%
  summarise(number_of_trips=n()) %>%
  arrange(desc (number_of_trips)) %>%
  filter(member_casual== "casual") %>%
  select(start_station_name, number_of_trips)
```
### Number of rides and Average ride length segmented by rider type
```{r}
all_trips_v2 %>%
  group_by(member_casual) %>%
  summarise(Average_ride_length=mean(ride_length)) %>%
  ggplot(aes(x= member_casual, y=Average_ride_length, fill=member_casual)) + geom_col() + labs(title = "Average ride length by rider type", x="Rider type", y="Average ride length")

```


### Let's visualize the number of rides by rider type
```{r}
all_trips_v2 %>% 
  mutate(weekday = wday(started_at, label = TRUE)) %>% 
  group_by(member_casual, weekday) %>% 
  summarise(number_of_rides = n()
            ,average_duration = mean(ride_length)) %>% 
  arrange(member_casual, weekday)  %>% 
  ggplot(aes(x = weekday, y = number_of_rides, fill = member_casual)) +
  geom_col(position = "dodge")

```

### Let's create a visualization for average duration
```{r}
all_trips_v2 %>% 
  mutate(weekday = wday(started_at, label = TRUE)) %>% 
  group_by(member_casual, weekday) %>% 
  summarise(number_of_rides = n()
            ,average_duration = mean(ride_length)) %>% 
  arrange(member_casual, weekday)  %>% 
  ggplot(aes(x = weekday, y = average_duration, fill = member_casual)) +
  geom_col(position = "dodge")

```


### Number of rides and Average ride length segmented by rideable type
```{r}
all_trips_v2 %>%
  group_by(rideable_type, member_casual) %>%
  summarise(number_of_rides=n()) %>%
  ggplot(aes(x=rideable_type, y=number_of_rides, fill=member_casual)) + geom_col(position = "dodge") + labs(title="Number of rides per rideable type" , x="Rideable Type", y="Number of rides")

```

```{r}
all_trips_v2 %>%
  group_by(rideable_type, member_casual) %>%
  summarise(Average_ride_length = mean(ride_length)) %>%
  ggplot(aes(x=rideable_type, y=Average_ride_length, fill=member_casual)) + geom_col(position = "dodge") + labs(title="Average ride length per rideable type" , x="Rideable Type", y="Average ride length")
```
The document, plots and tables are then exported for further analysis. Further visualisations also performed on Power BI. Saving the dataframes as CSV files on my local desktop.

```{r}
counts <- aggregate(all_trips_v2$ride_length ~ all_trips_v2$member_casual + all_trips_v2$day_of_week, FUN = mean)
write.csv(counts, file ="C:\\Users\\This PC\\Documents\\Capstone_Project") 

```
```{r}
write.csv(all_trips_v2,"C:\\Users\\This PC\\Documents\\Capstone_Project, row.names=FALSE)
```

