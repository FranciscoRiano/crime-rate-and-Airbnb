---
Title: "Data Preparation NYC"
Author: "Team 99"
Date: "3/26/2022"
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```
```{r, include=FALSE}
# Loading packages
library(plyr)
library(dplyr)
library(tidyverse)
library(ggplot2)
library(readr)
library(data.table)
```

```{r}
# Import raw data
load("../../gen/data-preparation/temp/calendar.RData")
load("../../gen/data-preparation/temp/listings.RData")
load("../../gen/data-preparation/temp/listings_2.RData")
load("../../gen/data-preparation/temp/neighbourhoods.RData")
load("../../gen/data-preparation/temp/NYPD-complaint.RData")
```


## Data exploration
#### Airbnb data
The Airbnb dataset of New York city consists 38277 rows with data from different listings.
Below an example of the data and the variables is shown:

```{r}
head(listings)
```
A lot of these variables are not needed and that is why we will use the listings_2 dataset that is provided.
This looks as follows:
```{r}
head(listings_2)
```

The most listings in NYC are in the neighbourhood Manhattan.
```{r listings, echo=FALSE}
unique(listings$neighbourhood_group) 
neighbourhood_count <- listings %>% group_by(neighbourhood_group) %>% summarize(count=n())
percentages <- round(neighbourhood_count$count/sum(neighbourhood_count$count)*100, 2)
percentages <- paste0(percentages, "%")
neighbourhood_count$neighbourhood_group <- paste0(neighbourhood_count$neighbourhood_group, percentages )
pie(neighbourhood_count$count,neighbourhood_count$neighbourhood_group, main= "listings per neighbourhood")
```

####Crime data 
The crime data consists of 7375993 rows of crimes comitted in New York City.
```{r}
head(crime_data)
```
There are 35 different variables. The ones that are most relevant are selected.

```{r}
crime_data<- crime_data[,c('CMPLNT_NUM', 'ADDR_PCT_CD', 'BORO_NM','CMPLNT_FR_DT','CMPLNT_TO_DT','CRM_ATPT_CPTD_CD','KY_CD','LAW_CAT_CD','OFNS_DESC','RPT_DT','Latitude','Longitude')]
head(crime_data)
```
####Dictionary of the variables:
Column name         | Column description
-------------       | -------------
CMPLNT_NUM          | Randomly generated persistent ID for each complaint 
ADDR_PCT_CD         | The precinct in which the incident occurred
BORO                | The name of the borough in which the incident occurred
CMPLNT_FR_DT        | Exact date of occurrence for the reported event (or starting date of occurrence, if CMPLNT_TO_DT exists)
CMPLNT_TO_DT        | Ending date of occurrence for the reported event, if exact time of occurrence is unknown
CRM_ATPT_CPTD_CD    | Indicator of whether crime was successfully completed or attempted, but failed or was interrupted prematurely
KY_CD               | Three digit offense classification code
LAW_CAT_CD          | Level of offense: felony, misdemeanor, violation 
OFNS_DESC           | Description of offense corresponding with key code
RPT_DT              | Date event was reported to police 
Latitude            | Midblock Latitude coordinate for Global Coordinate System, WGS 1984, decimal degrees (EPSG 4326) 
Longitude           | Midblock Longitude coordinate for Global Coordinate System, WGS 1984, decimal degrees (EPSG 4326)


Outliers in the dates should be removed from the dataset, because this information is not reliable. 
```{r}
crime_data$CMPLNT_FR_DT = as.Date(crime_data$CMPLNT_FR_DT, format = "%m/%d/%Y")
crime_data$CMPLNT_TO_DT = as.Date(crime_data$CMPLNT_TO_DT, format = "%m/%d/%Y")
crime_data$RPT_DT = as.Date(crime_data$RPT_DT, format = "%m/%d/%Y")
```

From the below boxplots it can be concluded that there are some outliers in the date of occurence and the ending date of occurence, but in the boxplot where the event was reported to the police all dates are in a realistic timeframe. This means that all the data is reliable. Moreover we are only interested in the crimes that occured in the past five years.
```{r}
boxplot(crime_data$CMPLNT_FR_DT, main = "Date of occurence for the reported event",
        xlab = "years",
        ylab = "date",
        horizontal = TRUE,
        notch = TRUE
)
boxplot(crime_data$CMPLNT_TO_DT, main = "Ending Date of occurence for the reported event",
        xlab = "years",
        ylab = "date",
        horizontal = TRUE,
        notch = TRUE
)
boxplot(crime_data$RPT_DT, main = "Date event was reported to police",
        xlab = "years",
        ylab = "date",
        horizontal = TRUE,
        notch = TRUE
)
```



Below a piechart of the crimes per neighbourhood is shown. The most crimes are commited in Brooklyn, followed by Manhattan and Bronx. These are also the biggest neighbourhoods in NYC.
```{r crimes, echo=FALSE}
crimes_per_neighbourhood <- crime_data %>% group_by(BORO_NM) %>% summarize(count=n())
percentages <- round(crimes_per_neighbourhood$count/sum(crimes_per_neighbourhood$count)*100, 2)
percentages <- paste0(percentages, "%")
crimes_per_neighbourhood$BORO_NM <- paste0(crimes_per_neighbourhood$BORO_NM, percentages )
pie(crimes_per_neighbourhood$count,crimes_per_neighbourhood$BORO_NM, main= "crimes per neighbourhood")
```

Misdemeanor is the most commited crime in all neighbourhoods in NYC. 

Types Of Criminal Charges In New York State:

A **Violation** is an offense other than a traffic infraction for which a sentence to a term of imprisonment of up to 15 days may be imposed (New York State Penal Law, Article 10). It is the least serious type of proscribed activity and encompasses such offenses as harassment, trespass, and disorderly conduct. A person arrested for committing a violation may be taken into custody but will usually be issued an appearance ticket indicating the time and place that he must appear in court. A violation is not a crime.

A **Misdemeanor** is an offense other than traffic infraction of which a sentence in excess of 15 days but not greater than one year may be imposed (New York State Penal Law, Article 10). A misdemeanor is a crime. Petit larceny, criminal mischief in the fourth degree and assault in the third degree all fall into this category. Misdemeanors are grouped into one of three classes: Class A, Class B, or Unclassified. Upon conviction of a Class “A” misdemeanor, a court may sentence an individual to a maximum of one year in jail or three years probation. In addition, a fine of up to $1,000 or twice the amount of the individual’s gain from the crime may be imposed. Offenders found guilty of Class “B” misdemeanors face maximum penalties of up to three months imprisonment or one year probation. In addition, a fine of up to five hundred dollars or double the amount of the defendant’s gain from the commission of the crime may be imposed. An unclassified misdemeanor is any offense not defined in the Penal law (other than a traffic violation) for which a sentence of imprisonment of greater than 15 days but not in excess of one year may be imposed.

A **Felony** is an offense for which a sentence to a term of imprisonment in excess of one year may be imposed (New York State Penal Law, Article 10). A felony is a crime. There are five categories and two subcategories of felonies (A-I, A-II, B, C, D, and E) ranging from the most to least serious in terms of severity of offense and the degree of potential punishment incurred. The penalty can vary from a term of probation to life imprisonment. In addition, the Penal Law authorizes the imposition of a fine not exceeding the higher of $5,000 or double the amount of the defendant’s gain from commission of the crime.

In the Penal Law’s description of each crime, the “degrees” of an offense determine the seriousness of the offense. For example, burglary in the third degree is a Class D felony and burglary in the second degree, the more serious offense, is a Class C felony.
(source:https://omh.ny.gov/omhweb/forensic/manual/html/chapter1.htm)
```{r}
crime_cat <- crime_data %>% 
  group_by(BORO_NM, LAW_CAT_CD) %>% 
  summarize(count = n())
ggplot(crime_cat,                                     
       aes(x = BORO_NM,
           y = count,
           fill = LAW_CAT_CD)) +
  geom_bar(stat = "identity",
           position = "dodge")+ ggtitle("Crime categories per neighbourhood") +
  xlab("neighbourhood") + ylab("Count") +labs(fill="Category")
```

##Data Cleaning
As seen from above barchart and piechart there are some crimes where no borough/neigbourhood is mentioned. This are 11329 crime and will be deleted from the dataset.
```{r}
#number of missing neighbourhoods
crime_cat
sum(filter(crime_cat, BORO_NM=="")[3])

crime_data_cleaned <-crime_data[!(crime_data$BORO_NM ==''),]

```


## Store the cleaned data
```{r}
dir.create('../../gen/data-preparation/output/')
save(crime_data_cleaned,file="../../gen/data-preparation/output/data_cleaned.RData")
```


Note that the `echo = FALSE` parameter was added to the code chunk to prevent printing of the R code that generated the plot.

```