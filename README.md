# Case Study: Bellabeat Fitness Data Analysis
**Author: Clara Bach**


**Date: August 09, 2023**

[Tableau Dashboard](https://public.tableau.com/app/profile/clara.bach/viz/BellabeatCaseStudy_16964524098930/Dashboard1)

## Introduction

**Bellabeat** is a high-tech company that manufactures health-focused smart products for women, including Bellabeat application, Leaf wellness tracker, Time wellness watch, Spring water bottle and Bellabeat memberships. By colleting data on avtivity, sleep, stress and reproductive health, Bellabeat aim to empower women with knowledge about their own health and habits.
 
With the expectation to become a larger player in the global smart device market, Urška Sršen, Cofounder and Sando Mur, Chief Creative Officer of Bellabeat, believes that analyzing smart device fitness data could help them to unlock new growth opportunities for the company. The insights from the finding will then hep guide marketing strategy for the company. 

This project will follow the Data Analysis Processes: **Ask, Prepare,Process, Analyze, Share and Act.**

## STEP 1: ASK

#### Business task:

Analyzing smart device usage to gain insight into how consumers usenon-Bellabeat smart devices and suggesting suitable data-driven marketing strategy for Bellabeat by answering these question:

  1.  Identify trends in smart device usage.
  2.  Apply insights to Bellabeat customers.
  3.  Describe how these trends identified help influence Bellabeat marketing strategy

#### Stakeholders:

-   Company's owners
-   Marketing analytics department
-   Head of the department

## STEP 2: PREPARE
#### Information on Data Source:

 - Data is publicly available on [**Kaggle:Fitbit Fitness Traker Data**](https://www.kaggle.com/datasets/arashnic/fitbit) and stored in 18 csv files.
 - Collected from an Amazon Mechanical Turk survey conducted between March 12, 2016, and May 12, 2016, by respondents.
 - 30 FitBit users agreed to share their personal tracker data.
 - Data collected includes physical activity recorded in minutes, heart rate, sleep monitoring, daily activity and steps.
 
#### The Credibility & Integrity of the data: The data follow the ***ROCCC Analysis***: 

  - Reliability: ***LOW*** --- There were only 30 individuals involved in this survey whose gender is unknown.
  - Originality: ***LOW*** --- Data was collected via third-party survey by Amazon Mechanical Turk.
  - Comprehensive: ***MEDIUM*** --- Dataset contains multiple fields ondaily activity intensity, daily steps taken, calories used, daily sleep time, and weight record for business task requirements.
  - Current: ***LOW*** --- Data is 8 years old and coverd a short period of April - May 2016. The period is so shortand might be not represented for the whole year activities.
  - Cited: ***HIGH*** --- This dataset is under CCO: public domain, made available by Mobius stored in Kaggle.
  
#### Limitation of Data Set: 

  - Only 30 user data is available. This sample size is not representative of the entire fitness population.
  - Since the data is gathered through a survey, we cannot guarantee its integrity or accuracy.
  - Data collected in 2016 may not accurately reflect users' current daily activity, fitness, sleep, diet, and food habits, making it potentially outdated and less relevant.

#### Data selection:
The following file is selected and copied for analysis.

  - dailyActivity_merged.csv
  - sleepDay_merged.csv
  - hourlySteps_merged.csv

## STEP 3: PROCESS

#### Install and load the packages: 
```{r load, echo=TRUE, message=FALSE, warning=TRUE}
#install.packages("tidyverse")
#install.packages("ggplot2")
...
library(tidyverse)
library(ggplot2)
....
```
#### Prepare the data and combine them in one data framme if needed
**Set the working directory**
**Import data**
```{r import, warning=FALSE}
activity <- read.csv ("dailyActivity_merged.csv")
sleep_day <- read_csv("sleepDay_merged.csv")
hourly_step <- read.csv("hourlySteps_merged.csv")
```

**Inspect data to see if there are any error**
```{r inspect}
str(activity)
str(sleep_day)
str(hourly_step)
```

**Check for NA and duplicates**
```{r check}
sum(is.na(activity))
sum(is.na(sleep_day))

sum(duplicated(activity))
sum(duplicated(sleep_day))
```

**Remove duplicates**
```{r remove}
sleep_day <- sleep_day[!duplicated(sleep_day), ]
sum(duplicated(sleep_day))
```


**Verify number of user**
```{r verify}
n_distinct(activity$Id)
n_distinct(sleep_day$Id)
n_distinct(hourly_step$Id)
```
With the expected 30 user participating in the survey, but have 3 extra from `activity`and 6 less from the `sleep_day` table. 

**Clean and rename columns**
We should clean and format the column names using lowercase to ensure they use the same syntax format.
```{r columnnames}
activity <- clean_names(activity)
sleep_day <- clean_names(sleep_day)
hourly_step <-clean_names(hourly_step)
```
**Transform data type**

  - After checking the date, we figure out the date/time columns are formatted as CHR not as a date format. Therefore, we will convert them as_datetime format and separate into date and time columns.
  - Then add a column for the **weekday** into `activity` data frame and order from Monday to Sunday for further analysis and plot

```{r datetime}
activity <- activity %>%
  rename(date = activity_date) %>%
  mutate(date = as_date(date, format = "%m/%d/%Y")) %>%
  mutate(weekday = weekdays(date)) %>%
  mutate(weekday = ordered(weekday, levels=c("Monday", "Tuesday", "Wednesday", "Thursday","Friday", "Saturday", "Sunday")))

sleep_day <- sleep_day %>%
  rename(date = sleep_day) %>%
  mutate(date = as_date(date, format = "%m/%d/%Y %I:%M:%S %p"))

hourly_step <- hourly_step %>%
  mutate(activity_hour = as.POSIXct(activity_hour, format = "%m/%d/%Y %I:%M:%S %p")) %>%
  mutate(hour = format(activity_hour,format="%H"))
``` 

**Combine data from `activity`, `sleep_day` and `hourly_step` based on id and date to have a data frame for further analysis**
```{r merge}
merge_asw <- merge(activity, sleep_day, by=c("id","date"), all=TRUE)
merge_aswh <-merge(merge_asw, hourly_step, by =c("id"), all=TRUE)
```

**Check for NA and duplicates in merged data**
```{r checkmerge}
sum(is.na(merge_asw))
sum(duplicated(merge_asw))
n_distinct(merge_asw)
```

## Step 4 & 5: ANALYZE & SHARE 

#### The Statistical summary of datasets
```{r summary}
#Steps and Calories
merge_asw %>%
  select(total_steps, total_distance,calories) %>%
  summary()

# Activity level
merge_asw %>%
  select(very_active_minutes, lightly_active_minutes, fairly_active_minutes, sedentary_minutes) %>%
  summary()

# Sleep
merge_asw %>%
  select(total_minutes_asleep, total_time_in_bed, total_sleep_records) %>%
  summary()

```
<img width="639" alt="Screenshot 2023-10-04 at 6 01 00 PM" src="https://github.com/bachbaongan/Google_Data_Analytics_Bellabeat_Casestudy/assets/144385168/26d60178-43a3-415c-a65b-96d4c1f57c01">

**Insights from the summary:**

  - The Average ***steps*** are ***7638*** per day which is a bit lesser than what is considered a healthy count of steps that is 8000 per for more benefit. Also taking 12,000 steps per day was associated with a 65% lower risk to disease mortality compared with taking 4,000 steps according to CDC.
  - The average participant burn per day is ***2304 calories*** - 97 calories/hour
  - Average sedentary time is ***991 minutes*** or 16 hours, which we should take an action to reduce
  - Participants' average sleep pattern is ***419 minutes*** or 7 hours once a day.
  

#### Analyze Activity level tracked by time
The activity level is recorded in minutes for each activity segment, namely `sedentary_minutes`, `lightly_active_minutes`, `fairly_active_minutes`, `very_active_minutes`. To analyze activity level, I create a data frame which transfer each activity segment time summarized by Id. 
```{r activitytime}
merge_asw1<- merge_aswh %>% 
  pivot_longer(
    cols = ends_with("minutes"),
    names_to = "Activity_level",
    values_to = "minutes") %>%
  mutate(Activity_level= str_replace(Activity_level, "very_active_minutes","Very Active")) %>%
  mutate(Activity_level= str_replace(Activity_level, "fairly_active_minutes","Fairly Active")) %>%
  mutate(Activity_level= str_replace(Activity_level, "lightly_active_minutes","Lightly Active")) %>%
  mutate(Activity_level= str_replace(Activity_level, "sedentary_minutes","Sedentary")) %>%
  mutate(Activity_level= ordered(Activity_level, levels=c("Very Active","Lightly Active","Fairly Active","Sedentary")))
```

![Activity level distribution classified by minutes](https://github.com/bachbaongan/Google_Data_Analytics_Bellabeat_Casestudy/assets/144385168/e6029fdd-b2d1-4e04-809b-cf8487edb404)




The data reveals that sedentary time occupies over 80% of the recorded total activity time, indicating that individuals spend the majority of their waking hours in a seated or lying position. Conversely, active time accounts for less than 20% of the total awake time.


![Sedentary minutes per weekday](https://github.com/bachbaongan/Google_Data_Analytics_Bellabeat_Casestudy/assets/144385168/08614dd7-82ac-4e67-a84c-77e6521e62a1)


The bar graph shows that users spent LESS time in sedentary minutes on Saturday, compared to other days.


![Activity time   Distance](https://github.com/bachbaongan/Google_Data_Analytics_Bellabeat_Casestudy/assets/144385168/8095d9ad-3a93-46ae-b6a2-11da3caa7b1c)


The chart depicts a positive correlation between active minutes and total distance, which diminishes as very active minutes surpass 150. Additionally, an initial positive correlation exists between active minutes and distance, which transforms into a negative correlation with increased sedentary minutes. To summarize, individuals with **more active minutes tend to cover greater daily distances**, whereas increased inactivity leads to shorter daily distances.


![Activity level   Calories burnt](https://github.com/bachbaongan/Google_Data_Analytics_Bellabeat_Casestudy/assets/144385168/8742a276-022b-43b5-9914-dec767d1ab94)


The visualization illustrates a strong connection between the `Very Active` level and the number of calories burned. Furthermore, the chart
showcases a pattern characterized by an initial positive correlation, succeeded by a negative correlation as sedentary minutes increase. This means that as sedentary minutes rise, the calorie burn decreases after an initial increase. Therefore, ***individuals who engage in very active activities tend to burn more calories per day, and as their inactivity time increases, their daily calorie expenditure decreases***.


![Activity time   Sleep quality](https://github.com/bachbaongan/Google_Data_Analytics_Bellabeat_Casestudy/assets/144385168/94d52e59-19a0-4591-bf7c-2ecda42d1405)

The chart represent an inverse correlation between inactivity, specifically the "Sedentary" level and the duration of time spent asleep, **as the number of minutes spent being inactive increases, the amount of time spent asleep in bed tends to decrease**. After 50 minutes of physical activity, we observe the following trends in relation to sleep quality and other active level:

  - A negative association between fairly active periods and sleepquality.
  - A positive correlation between very active periods and sleepquality.
  - For individuals who engage in light activity and those whoaccumulate less than 50 minutes of moderate or high activity, their total sleep duration typically falls within the range of 250 to 550 minutes.


#### Analyze Steps

**Hourly activities**

![Hourly Steps](https://github.com/bachbaongan/Google_Data_Analytics_Bellabeat_Casestudy/assets/144385168/ae4a68d9-37d9-4d6c-9c83-432a3ff399a9)

We can figure out from the chart that the period from 5PM to 7PM, the users take the most steps. 

**Activity level by steps**

The CDC recommend that most adults aim for 10,000 steps per day, with fewer than 5,000 steps being sign of a sedentary lifestyle. One of the most popular activity tracker is pedometer - which help to count the steps by detecting the motion of the hips. Most pedometers provide guidelines as per 10,000-step protocol. To analyze the steps, I create a data frame based on the pedometer classify to segment each activity level as follows:

  * Sedentary: less than 5,000 steps per day
  * Fairly active: 5,001 to 7,499 steps per day
  * Lightly active: 7,500 to 9,999 steps per day
  * Very active: more than 10,000 steps per day

```{r user}
merge_asw2 <- merge_asw1 %>%
  mutate(Level_active = case_when (total_steps <= 5000 ~ "Sedentary",
                                total_steps > 5000 & total_steps <= 7499 ~ "Fairly Active",
                                total_steps > 7499 & total_steps <= 9999 ~ "Lightly Active",
                                total_steps > 9999 ~ "Very Active"))%>%
  mutate(Level_active= ordered(Level_active, levels=c("Very Active","Lightly Active","Fairly Active","Sedentary")))
```
![Activity level distribution classified by steps](https://github.com/bachbaongan/Google_Data_Analytics_Bellabeat_Casestudy/assets/144385168/be7c9794-dacb-4e12-8852-c64ec4b0f849)


![activity level distribution classified by steps](https://github.com/bachbaongan/Google_Data_Analytics_Bellabeat_Casestudy/assets/144385168/cddea0c3-19d4-4310-a3f0-d6d01d09f28a)

With classification based on total daily steps, our user type are focused mostly on "Very Active" group and "Sedentary" group. This insight would give the marketing team a hint for their target audience in marketing strategy. 


```{r userday, echo=FALSE, message=FALSE, warning=FALSE}
ggplot(data=activity, aes(x=weekday, y=total_steps, fill=weekday)) +
  geom_bar(stat="identity") +
  theme(legend.position="none")+
  labs(title="Total steps per Weekday", x="", y="Total Steps")
```


**Steps and Distance**

```{r stepdis22, echo=FALSE}
ggplot(data=merge_asw2,aes(x=total_steps, y=total_distance)) +
  geom_point(color="#B2182B", size=0.2, na.rm=TRUE) +
  geom_smooth( color="yellow", linewidth=0.6, method = 'loess' ,formula = 'y ~ x', na.rm=TRUE) +
  labs(title="Steps & Distance", y="Total distance", x="Total steps",)
```


Now we can observe the highly relationship between the Distance and Steps, **meaning the more steps user can made the more distance is covered**.

**Steps and Sleep quality per Weekday**

```{r stepsleep, echo=FALSE, warning=FALSE}
merge_asw2 %>% group_by(weekday) %>%
  summarise(across(c(total_steps, total_minutes_asleep), mean, na.rm=TRUE)) %>%
  ggplot() +
  geom_bar(aes(x=weekday,y=total_steps, fill=total_steps>8000), stat='identity',labels=NULL, width = 0.8) +
  geom_text(aes(label=round(total_steps,digits=0), x=weekday,y=total_steps), position = position_stack(1.035), colour="#4D4D4D", size=4, fontface = "bold")+
  geom_line(aes(x=weekday,y=10*total_minutes_asleep), stat='identity', group=1, na.rm=TRUE, color="#D6604D", linewidth=1.15) +
  geom_text(aes(label=round(total_minutes_asleep,digits=0), x=weekday,y=10*total_minutes_asleep), position = position_stack(vjust = 1.075), colour='#B2182B', size=4, fontface="bold")+
  scale_y_continuous(sec.axis=sec_axis(~.*0.1,name = "Average Minutes Sleep"))+
  scale_fill_manual(aes(x=weekday,y=total_steps), values = c("#BABABA","#F4A582"))+
  theme(legend.position="none")+
  labs(title="Average Steps & Minutes Sleep per Weekday", y="Average Steps", x=NULL)
```

According to the chart, we can figure out that:

-   The user's daily average steps count is higher than the `7500` steps goal, so we can categorize as Somewhat active level.
-   The user's daily average sleep duration, which falls short of the recommended 8 hours (equivalent to 480 minutes) of daily sleep. 
Therefore, we should issue an alert to the user when their step count and sleep duration recorded by their tracker lower than these threhold.

#### Analazy Calories

```{r calnweekday, echo=FALSE}
merge_asw2 %>% group_by(weekday) %>%
  summarise(avg_cal=mean(calories, na.rm=TRUE)) %>%
  ggplot(aes(x=reorder(weekday, -avg_cal),y=avg_cal, fill=reorder(weekday, -avg_cal))) +
  geom_bar(stat='identity', width = 0.8) + theme(legend.position="none") +
  geom_text(aes(label=round(avg_cal,digits=0)), position = position_stack(0.95), colour="#FDDBC7", size=4, fontface = "bold")+
  scale_fill_manual(values = rev(RColorBrewer::brewer.pal(9,"Oranges")))+
  labs(title="Average Calories Burnt per Weekday", y="Calories", x=NULL)
```


```{r stepdis1, echo=FALSE}
ggplot(data=merge_asw2,aes(x=total_steps, y=calories)) +
  geom_point(color="#B2182B", size=0.2, na.rm=TRUE) +
  geom_smooth( color="yellow", linewidth=0.6, method = 'loess' ,formula = 'y ~ x', na.rm=TRUE) +
  labs(title="Steps & Calories", y="Calories", x="Total steps",)
```


```{r stepcal, echo=FALSE}
ggplot(merge_asw2, aes(x=Level_active, y=calories, fill=Level_active),fig(30,4)) +
    geom_boxplot()+
    scale_fill_brewer(palette ="Reds")+
    labs(title= " Calories burnt by User Type", x ="Activity Level", y= "Calories")
```

These charts provide a clear illustration that both the Very Active and Lightly Active segments lead to the highest calorie burn. This highlights the positive correlation between the number of daily steps and calorie expenditure, indicating that increasing daily step count can lead to higher calorie burn.
Besides, we can also figure out the user tends to exhibit higher levels of activity on **Tuesday and Saturday**, as evidenced by their increased step count and calorie expenditure on these particular days.

## STEP 6: ACT

  1. **Inactive Minutes Alert**: Bellabeat can consider adding a feature to their application that notifies users who tend to accumulate higher levels of non-active minutes. This can serve as a gentle reminder to stay active throughout the day. Additionally, they can implement a function to alert users if they haven't worn their tracker devices for an extended period.
  2. **Sedentary Behavior Notifications**: Bellabeat can incorporate timely notifications in their Leaf/Time devices to motivate users to reduce sedentary minutes by moving regularly. These reminders can prompt users to take short breaks and engage in physical activity.
  3. **Daily Step Goal Promotion**: Encouraging users to aim for the recommended 7,500 steps per day, as per CDC guidelines, can be beneficial. Bellabeat could introduce a feature that promotes the advantages of achieving this daily step goal and tracks users' progress toward it.
  4. **Enhanced Sleep Tracking**: Bellabeat can improve their sleep tracking function by emphasizing the link between sleep and inactivity. They can notify users if they consistently sleep less than 8 hours a day, allowing users to set their desired sleep duration. Moreover, offering articles or podcasts related to sleep techniques and relaxing music can assist users in achieving better sleep quality.

By implementing these data-driven recommendations, Bellabeat can enhance its market strategy and provide users with valuable features and insights for a healthier and more active lifestyle.

