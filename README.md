# bike_share_navigate_speedy_success
Case Study: How does bike-share navigate speedy success ?

This case study is part of my Google Data Analytics Professional Specialization.


### Introduction

The director of marketing of a fictional bike-share company based in Chicago believes that the company's future success depends on maximizing the number of annual memberships. Therefore, I want to understand how casual riders and annual members use Cyclistic bikes differently.From these insights, a new marketing strategy will be designed to convert casual riders into annual members.

Cyclistic's finance analysts have concluded that annual members are much more profitable than casual riders. Although the pricing flexibility helps Cyclistic attract more customers, the director of Marketing believes that maximize the number of annual members will be key to future growth.

### Description of data sources used

The data has been made available by a company called Motivate International Inc and has been downloaded from this [link](https://divvy-tripdata.s3.amazonaws.com/index.html). Under this [license](https://www.divvybikes.com/data-license-agreement). 

The datasets are grouped monthly from June 2020 until June 2024 in .csv format.

In our case, I will be using two datasets: 

* Divvy_2019_Q1
* Divvy_2020_Q2

The two datasets are available in this repo.

### Installation of packages

First of all, I am going to install the "tidyverse" package. No I am not going to install "ggplot2" - because it's part of the tidyverse package.

Tidyverse is a collection of packages in R designed specifically for data science tasks. It offers a cohesive framework that streamlines data manipulation, analysis, and visualization.

```
install.packages("tidyverse")

library(tidyverse)

View(q1_2019)

View(q1_2020)
```

### Comparing column names to check if they match perfectly. The order doesn't count.
```
colnames(q1_2019)
colnames(q1_2020)
```

### Renaming columns to make them consistent with q1_2020.

```
  (q1_2019 <- rename(q1_2019
                   ,ride_id = trip_id
                   ,rideable_type = bikeid
                   ,started_at = start_time
                   ,ended_at = end_time
                   ,start_station_name = from_station_name
                   ,start_station_id = from_station_id
                   ,end_station_name = to_station_name
                   ,end_station_id = to_station_id
                   ,member_casual = usertype
                   ))
```

### Inspection of dataframes and look for incongruencies

```
str(q1_2019)
str(q1_2020)
```

### Converting ride_id and rideable_type to character so that thy can stack correctly

```
q1_2019 <- mutate(q1_2019, ride_id = as.character(ride_id)
                  ,rideable_type = as.character(rideable_type))
```

### Stack individual quarter's data frames into one big data frame that will be called all_trips

```
all_trips <- bind_rows(q1_2019, q1_2020)
```

### Removing lat, long, birthyear, and gender fields as this data was dropped beginning n 2020

```
all_trips <- all_trips %>%
  select(-c(start_lat, start_lng, end_lat, end_lng, birthyear, gender, "tripduration"))
```

### Inspecting the new table that has been created

```
colnames(all_trips)
View(all_trips)
nrow(all_trips) #total number of row
dim(all_trips) #dimension of the data frame
head(all_trips) #To see the first 6 rows of data frame
str(all_trips) #To see list of columns and data types (numeric, character, etc)
summary(all_trips) #Statistical summary of data.
```

### Seeing how many observations fall under each usertype

```
table(all_trips$member_casual)
```

### Reassigning to the desired values (Ging with the current 2020 labels)

```
all_trips <- all_trips %>%
  mutate(member_casual = recode(member_casual
                                ,"Subscriber" = "member"
                                ,"Customer" = "casual"))
```

### Checking to make sure the proper number of observations were reassigned

```
table(all_trips$member_casual)
```

### Adding columns taht listthe data, mont, day, and year of each ride.

 It will allow us to aggregate ride data for each month, day, or year before completing these operations we could only aggregate at the ride level

```
all_trips$date <- as.Date(all_trips$started_at) #The default format is yyyy-mm-dd
all_trips$month <- format(as.Date(all_trips$date), "%m")
all_trips$day <- format(as.Date(all_trips$date), "%d")
all_trips$year <- format(as.Date(all_trips$date), "%Y")
all_trips$day_of_week <- format(as.Date(all_trips$date), "%A")
```

### Adding a ride_length calculation to all trips

```
all_trips$ride_length <- difftime(all_trips$ended_at, all_trips$started_at)
```

### Removing bad data
### The data frames includes a few hundred entries when bikes were taken out of docks and checked for quality by Divvy or ride_length was negative

### Let us create a new version of the dataframe (v2) since data is beign removed

```
all_trips_v2 <- all_trips[!(all_trips$start_station_name == "HQ QR" | all_trips$ride_length<0),]
```

### DESCRIPTIVE ANALYSIS

#### On ride_length

```
mean(all_trips_v2$ride_length)
median(all_trips_v2$ride_length)
max(all_trips_v2$ride_length)
min(all_trips_v2$ride_length)
```

### Condesing the four lines above to one line using summary()

```
summary(all_trips_v2$ride_length)
```

### Comparing members and casual users

```
aggregate(all_trips_v2$ride_length ~ all_trips_v2$member_casual, FUN = mean)
aggregate(all_trips_v2$ride_length ~ all_trips_v2$member_casual, FUN = median)
aggregate(all_trips_v2$ride_length ~ all_trips_v2$member_casual, FUN = max)
aggregate(all_trips_v2$ride_length ~ all_trips_v2$member_casual, FUN = min)
```

### Seeing the average ride time by each day for members vs casual users

```
aggregate(all_trips_v2$ride_length ~ all_trips_v2$member_casual + all_trips_v2$day_of_week,
          FUN = mean)
```

### Have notice that the days of the week are out of order and msut be fixed

```
all_trips_v2$day_of_week <- ordered(all_trips_v2$day_of_week, levels=c("Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"))
```

### Now let's run the average ride time by each day for members vs casual users

```
aggregate(all_trips_v2$ride_length ~ all_trips_v2$member_casual + all_trips_v2$day_of_week,
          FUN = mean)
```

### Analyzing ridership data by type and weekday

```
all_trips_v2 %>%
  mutate(weekday = wday(started_at, label = TRUE)) %>% #creates weekday field using wday()
  group_by(member_casual, weekday) %>%
  summarise(number_of_rides = n()
            ,average_duration = mean(ride_length)) %>%
  arrange(member_casual, weekday)
```  

### Let's visualize the number of rides by rider type

```
all_trips_v2 %>%
  mutate(weekday = wday(started_at, label = TRUE)) %>%
  group_by(member_casual, weekday) %>%
  summarise(number_of_rides = n()
            ,average_duration = mean(ride_length)) %>%
  ggplot(aes(x = weekday, y = number_of_rides, fill = member_casual)) +
  geom_col(position = "dodge")
```

![](C:/Users\Davka\Desktop\Divvy_Trips_2019_2020_Q1\first_plot.png)

### Creation of a visualization for average duration

```
all_trips_v2 %>%
  mutate(weekday = wday(started_at, label = TRUE)) %>%
  group_by(member_casual, weekday) %>%
  summarise(number_of_rides = n()
            ,average_duration = mean(ride_length)) %>%
  arrange(member_casual, weekday) %>%
  ggplot(aes(x = weekday, y = average_duration, fill = member_casual)) +
  geom_col(position = "dodge")
  
```

![](C:/Users\Davka\Desktop\Divvy_Trips_2019_2020_Q1\second_plot.png)
