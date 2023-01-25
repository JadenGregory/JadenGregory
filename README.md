## Case Study: BellaBeat Fitness

### About Company 
Bellabeat is a high tech company that manufactures health-focused smart products which inspireand inform women around world. Since it was founded in 2013, Bellabeat has grown dramatically as a tech-driven wellness company for women. By 2016, Bellabeat had opened office around the world and launched multiple products such as Bellabeat app, leaf, time and spring. Bellabeat products are available through online retailers in addition to their own e-commerce website.


### Google Data Analytics Capstone: Case Study 2

There are 6 phases or steps to this process, Ask, Prepare, Process, Analyze, Share, Act.

## 1. Objective (Ask)

### Business Task
Gain insight on trends from non-BellaBeat products and provide recommendations for ways 
Bellabeat marketing strategy can make improvements based on trends in smart device usage.

#### Key Stakeholders
* Urška Sršen: Bellabeat's cofounder and Chief Creative Officer
* Sando Mur: Mathematician and Bellabeat’s co-founder

#### Questions
* What are some trends in smart device usage?
* How could these trends apply to BellaBeat customers?
* How could these trends help influence BellaBeat marketing strategies?

## 2. Prepare 

The data in this case study was downloaded from [FitBit Fitness Tracker Data](https://www.kaggle.com/datasets/arashnic/fitbit). 

The data was uploaded, stored, and checked using RStudio. The data set contains thirty eligible Fitbit users that consented to the submission of personal tracker data, including minute-level output for physical activity, heart rate, and sleep monitoring. It includes information about daily activity, steps, and heart rate that can be used to explore users’ habits.

There are 18 csv files contained in the data set, organized in long format.

#### Loading Packages
````
library(tidyverse)
library(lubridate) 
library(dplyr)
library(ggplot2)
library(tidyr)
library(janitor)
````

## 3. Process

### Importing the datasets
````
activity <- read_csv("../input/fitbit/Fitabase Data 4.12.16-5.12.16/dailyActivity_merged.csv")
calories <- read_csv("../input/fitbit/Fitabase Data 4.12.16-5.12.16/dailyCalories_merged.csv")
intensities <- read_csv("../input/fitbit/Fitabase Data 4.12.16-5.12.16/hourlyIntensities_merged.csv")
sleep <- read_csv("../input/fitbit/Fitabase Data 4.12.16-5.12.16/sleepDay_merged.csv")
weight <- read_csv("../input/fitbit/Fitabase Data 4.12.16-5.12.16/weightLogInfo_merged.csv")
````

Once all the dataframes were imported. I will have a look at each data frame and familiarize myself with the data and check for any errors.
````
head(activity)
````
![Capture](https://user-images.githubusercontent.com/123005774/214688246-aa1fcbb0-3af3-47fe-a367-074a2c3998e7.PNG)

````
colnames(activity)
````
![Capture](https://user-images.githubusercontent.com/123005774/214688534-5727411c-2eb2-4849-8b15-4e27f8bd23c4.PNG)

While checking the data I noticed some problems with the timestamp data. To correct this I converted it to date time format.
````
# intensities
intensities$ActivityHour=as.POSIXct(intensities$ActivityHour, format="%m/%d/%Y %I:%M:%S %p", tz=Sys.timezone())
intensities$time <- format(intensities$ActivityHour, format = "%H:%M:%S")
intensities$date <- format(intensities$ActivityHour, format = "%m/%d/%y")
# activity
activity$ActivityDate=as.POSIXct(activity$ActivityDate, format="%m/%d/%Y", tz=Sys.timezone())
activity$date <- format(activity$ActivityDate, format = "%m/%d/%y")
# sleep
sleep$SleepDay=as.POSIXct(sleep$SleepDay, format="%m/%d/%Y %I:%M:%S %p", tz=Sys.timezone())
sleep$date <- format(sleep$SleepDay, format = "%m/%d/%y")
````
The dates are properly formatted and all the dataframes were checked for any errors.

## 4. Analyze

For the analyze phase, I checked the total number of participants for each category.
````# Finding number of participants in each category
n_distinct(activity$Id)  
n_distinct(calories$Id)   
n_distinct(intensities$Id)
n_distinct(sleep$Id)
n_distinct(weight$Id)
````
Activity [33]
Calories [33]
Intensities [33]
Sleep [24]
Weight [8]

After checking all the data I noticed the fact that there are only 8 participants in the weight dataset means that more data would be needed to make a strong recommendation or conclusion. So I decided to check for any significant changes.

````
weight%>%
group_by(Id)%>%
summarise(min(WeightKg),max(WeightKg))
````
![image](https://user-images.githubusercontent.com/123005774/214694795-c0ceeac1-56b6-43e6-ae00-5daaea2e242a.png)

There is no significant changes in weight, this with the combination of only 8 participants I decided not to use that dataset.

Lets see the summaries for the rest of the datasets:

<img width="502" alt="RStudio" src="https://user-images.githubusercontent.com/123005774/214695967-d3350cac-31b9-46bb-8b70-f7277c6c39e9.png">

### Observations
* Sedetary minutes is 16.5 hours on average.
* The average number of steps per day is 7638.
* Most of the participants are lightly active.
* The average participant burns 97 calories per hour
* On average, participants sleep for 7 hours.

### Merging Data

The two datasets that will be merged using inner join are the activity and sleep dataset on columns id and date.
````
merged_data <- merge(sleep, activity, by = c('Id', 'date'))
head(merged_data) 
````
![Capture](https://user-images.githubusercontent.com/123005774/214700133-7e182388-2757-4618-849e-606efa46af1d.PNG)

Now that the two datasets are merged its time to show the results through visualization.

## 5. Share

I would like to check if there is a correlation between number of steps taken and calories burned.
````
ggplot(data = activity, aes(x = TotalSteps, y = Calories)) + geom_point() + geom_smooth() + labs(title = "Total Steps vs. Calories")
````
![Rplot01](https://user-images.githubusercontent.com/123005774/214701232-324ed833-a799-4b93-ab9f-57741c8c5a63.png)

looking at the scatter chart above, it seems there is a correlation between total number number of steps and calories burned.
````
ggplot(data = sleep, aes(x = TotalMinutesAsleep, y = TotalTimeInBed)) + geom_point() + labs(title = "Total time asleep vs Total time in bed")
````
![Rplot](https://user-images.githubusercontent.com/123005774/214702096-2b2e1bf9-8628-4922-8e02-045b0ff351c8.png)

There is positive correlation between total time asleep vs total time in bed. 
````
ggplot(data = merged_data, mapping = aes(x = SedentaryMinutes, y = TotalMinutesAsleep)) + 
  geom_point() + labs(title= "Sleep Duration and Sedentary Time")
````
![Rplot02](https://user-images.githubusercontent.com/123005774/214703038-c60ab457-f5a0-47c5-8ee6-8e1e416d7779.png)

Judging from the graph above we can see a negative correlation between sedentary minutes and total minutes asleep.
This shows that less active the participant the less sleep they seem to get.

Lets see if the day of the weeks affects the activity levels and sleep.
````
merged_data <- mutate(merged_data, 
                                        day = wday(SleepDay, label = TRUE))
summarized_activity_sleep <- merged_data %>% 
  group_by(day) %>% 
  summarise(AvgDailySteps = mean(TotalSteps),
            AvgAsleepMinutes = mean(TotalMinutesAsleep),
            AvgAwakeTimeInBed = mean(TotalTimeInBed), 
            AvgSedentaryMinutes = mean(SedentaryMinutes),
            AvgLightlyActiveMinutes = mean(LightlyActiveMinutes),
            AvgFairlyActiveMinutes = mean(FairlyActiveMinutes),
            AvgVeryActiveMinutes = mean(VeryActiveMinutes), 
            AvgCalories = mean(Calories))
head(summarized_activity_sleep)
````
![Capture](https://user-images.githubusercontent.com/123005774/214705317-3cf70b08-27a0-4ecb-bb90-f8c89dfb571c.PNG)
````
ggplot(data = summarized_activity_sleep, mapping = aes(x = day, y = AvgDailySteps)) +
geom_col(fill = "blue") + labs(title = "Daily Step Count")
````
![Rplot03](https://user-images.githubusercontent.com/123005774/214705524-c5bd2742-025b-4bd2-9c92-553fee16c2fa.png)

This bar graph shows that participants are most active on saturdays and least active on sundays.

## 6.Act

![image](https://user-images.githubusercontent.com/123005774/214709453-d3beb7f8-15f9-41e8-9fc0-643a1fa180e3.png)


### Summary
Bellabeat is a high tech company that manufactures health-focused smart products which inspireand inform women around world. Since it was founded in 2013, Bellabeat has grown dramatically as a tech-driven wellness company for women. By 2016, Bellabeat had opened office around the world and launched multiple products such as Bellabeat app, leaf, time and spring. Bellabeat products are available through online retailers in addition to their own e-commerce website.

After Analyzing the FitBit Fitness Tracker Data, here are the recommendations for **Bellabeat marketing strategy based on trends in smart device usage.**.

1. The majority of the participants are lightly active. BellaBeat should offer some sort of reward system to encourage participants to something that might make them become fairly active.

2. The average number of steps per day is 7,638, which is lower then the CDC recommendations. The CDC's official website states: 8,000 steps per day was associated with a 51% lower risk for all-cause morality (or death from all causes). BellaBeat can recommend users to at least take 8,000 steps per day and give an explanation and a link to the CDC's website.

3. Saturdays are the most active for participants. BellaBeat can use this insight to remind users to go out on saturdays and motivate users to exercise on sundays.
