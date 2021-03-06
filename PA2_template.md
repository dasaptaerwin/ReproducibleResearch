# Weather data exploration: Coursera-PeerAssignment02
Dasapta Erwin Irawan  
15/09/2014  

#Questions:

Which types of events (as indicated in the EVTYPE variable) across the United States: 

1. are most harmful with respect to population health, across the United States?
2. have the greatest economic consequences?

#Requirements:

1. Publish to RPubs. 
2. Compile to HTML.
3. Doc layout:

+ Language: English.
+ Title: summarizes your data analysis
+ Synopsis section: describes and summarizes the analysis in at most 10 complete sentences.
+ Data Processing section: code and narration 
+ Result section
+ Figures: max of three figures. 

#1. Synopsis

This is my peer assignment 2 for Coursera: Reproducible Research. This document is my own work, based on several examples of workflow that I found in the Rpubs and StackOverflow site. One of the main reference is [Ryan Cheung Git](https://github.com/ryancheunggit). 

The following are the results:

+ Injuries: 
    + Tornado is the cause for most injuries (66%). 
    + tornado, wind and heat are the top three weather events for over 80% injuries.

+ Fatalities: 
    + Tornado is the cause for most fatalities (38.4%). 
    + Tornado, heat, and flood are the top three weather events for about 70% fatalities.
    
+ Porperty Damage: Flood is the cause for most porperty damages(48%) among all weather events. Flood and tornado together are responsible for about 65% of all porperty damages.

+ Crop Damage: Drought is the cause for most crop damages(31%) among all weather events. Drought, flood and ice storm together are responsible for over 69% of all crop damages.


#2. Data processing

##2.1 Preparation (set wd, load libraries, read data)


```r
library(ggplot2)
library(scales)

data <- read.csv("repdata-data-StormData.csv", header = T)
names(data) <- tolower(names(data))
names(data) <- gsub("_","",names(data))
```

##2.2 Clean the values in `EVTYPE` column

According to the dataset documentation the event name should be one of those fifty listed. Here I use `tolower()` function.


```r
data$evtype <- tolower(as.character(data$evtype))
data$evtype <- gsub("^(([^:]+)://)?([^:/]+)(:([0-9]+))?(/.*)","",data$evtype)
```

##2.3 Removing unwanted values

+ Too small values


```r
data <- subset(data,nchar(data$evtype)>=2)
```

+ Values starts with "summary"


```r
data$evtype[grep("summary", data$evtype)] <- "tbm"
data <- subset(data,data$evtype != "tbm")
```

##2.4 Consolidating values

+ Using `grep()` function to the dataset


```r
data$evtype[grep("hail", data$evtype)] <- "Hail"
data$evtype[grep("wind", data$evtype)] <- "Wind"
data$evtype[grep("tornado", data$evtype)] <- "Tornado"
data$evtype[grep("flood", data$evtype)] <- "Flood"
data$evtype[grep("lightning", data$evtype)] <- "Lightning"
data$evtype[grep("snow", data$evtype)] <- "Snow"
data$evtype[grep("rain", data$evtype)] <- "Rain"
data$evtype[grep("winter", data$evtype)] <- "Winter"
data$evtype[grep("heat", data$evtype)] <- "Heat"
data$evtype[grep("fog", data$evtype)] <- "Fog"
data$evtype[grep("surf", data$evtype)] <- "Surf"
data$evtype[grep("ice storm", data$evtype)] <- "Ice storm"
data$evtype[grep("fire", data$evtype)] <- "Wild fire"
data$evtype[grep("storm surge", data$evtype)] <- "Strom surge"
data$evtype[grep("hurricane", data$evtype)] <- "Hurricane"
data$evtype[grep("drought", data$evtype)] <- "Drought"
data$evtype[grep("thunderstorm", data$evtype)] <- "Thunderstorm"
```

+ The following major weather event types takes up 97% of all observations.
+ The rest 3% weather events are changed into new category named "other".


```r
sum(data$evtype %in% c("Flood","Wind","Snow",
                       "Tornado","Hail","Rain",
                       "Lightning","Winter","Fog",
                       "Heat","Surf","Ice storm",
                       "Wild fire","Storm surge",
                       "Hurricane","Drought",
                       "Thunderstorm"))/nrow(data)
```

```
## [1] 0.9778
```

```r
tbc <- data$evtype %in% c("Flood","Wind","Snow",
                          "Tornado","Hail","Rain",
                          "Lightning","Winter","Heat",
                          "Surf","Fog","Ice storm",
                          "Wild fire","Storm surge",
                          "Hurricane","Drought",
                          "Thunderstorm") == F
                          data$evtype[tbc == T] <- "other"
```

+ Summary of counts of cleaned event type values.


```r
sort(table(data$evtype))
```

```
## 
## Thunderstorm    Hurricane         Surf          Fog    Ice storm 
##           92          199          833         1883         2006 
##      Drought         Heat    Wild fire         Rain    Lightning 
##         2488         2630         2781        12136        15762 
##         Snow       Winter        other      Tornado        Flood 
##        17569        18492        19813        60688        81967 
##         Hail         Wind 
##       289338       362164
```

##2.5 Calculating Damages

###A. Preparation


```r
data$propdmgexp <- as.character(data$propdmgexp)
data$propdmgexp[grep("K", data$propdmgexp)] <- "1000"
data$propdmgexp[grep("M", data$propdmgexp)] <- "1000000"
data$propdmgexp[grep("m", data$propdmgexp)] <- "1000000"
data$propdmgexp[grep("B", data$propdmgexp)] <- "1000000000"
tbc <- data$propdmgexp %in% c("1000","1000000","1000000000") == F
data$propdmgexp[tbc == T] <- "1"
data$propdmgexp <- as.numeric(data$propdmgexp)

data$cropdmgexp <- as.character(data$cropdmgexp)
data$cropdmgexp[grep("K", data$cropdmgexp)] <- "1000"
data$cropdmgexp[grep("M", data$cropdmgexp)] <- "1000000"
data$cropdmgexp[grep("m", data$cropdmgexp)] <- "1000000"
data$cropdmgexp[grep("B", data$cropdmgexp)] <- "1000000000"
tbc <- data$cropdmgexp %in% c("1000","1000000","1000000000") == F
data$cropdmgexp[tbc == T] <- "1"
data$cropdmgexp <- as.numeric(data$cropdmgexp)
```

###B. Calculating and stored damages as new variable "propDamage" and "cropDamage"


```r
data$propdamage <- data$propdmg * data$propdmgexp
data$cropdamage <- data$cropdmg * data$cropdmgexp
```


#3. Results

##3.1 Damages to Population Health

+ Injuries: 
    + Tornado is the cause for most injuries (66%). 
    + tornado, wind and heat are the top three weather events for over 80% injuries.

+ Fatalities: 
    + Tornado is the cause for most fatalities (38%). 
    + Tornado, heat, and flood are the top three weather events for about 70% fatalities.

###A. The event that caused most injuries.


```r
totalInjuries <- tapply(data$injuries, data$evtype, sum)
sort(totalInjuries, decreasing = T)[1]
```

```
## Tornado 
##   91365
```

###B. The percentage of injuries caused by the tornado.


```r
sum(sort(totalInjuries, decreasing = T)[1])/sum(totalInjuries)
```

```
## [1] 0.661
```

###C. The top 3 events responsible for most injuries


```r
sort(totalInjuries, decreasing = T)[1:3]
```

```
## Tornado    Wind    Heat 
##   91365   11319    9224
```

###D. The percentage of injuries caused by the top 3 weather events


```r
sum(sort(totalInjuries, decreasing = T)[1:3])/sum(totalInjuries)
```

```
## [1] 0.8096
```

###E. The event that caused most fatalities


```r
totalFatal <- tapply(data$fatalities, data$evtype, sum)
sort(totalFatal, decreasing = T)[1]
```

```
## Tornado 
##    5633
```

###F. The percentage of fatalities caused by the tornado


```r
sum(sort(totalFatal, decreasing = T)[1])/sum(totalFatal)
```

```
## [1] 0.3846
```

###G. The top 3 events responsible for most fatalities


```r
sort(totalFatal, decreasing = T)[1:3]
```

```
## Tornado    Heat   Flood 
##    5633    3119    1488
```

###H. The percentage of death caused by the top 3 weather events


```r
sum(sort(totalFatal, decreasing = T)[1:3])/sum(totalFatal)
```

```
## [1] 0.6992
```

##3.2 Economic Consequences

+ Porperty Damage: Flood is the cause for most porperty damages(48%) among all weather events. Flood and tornado together are responsible for about 65% of all porperty damages.

+ Crop Damage: Drought is the cause for most crop damages(31%) among all weather events. Drought, flood and ice storm together are responsible for over 69% of all crop damages.

###A. The event that caused most prop damages


```r
totalPropDamage <- tapply(data$propdamage, data$evtype, sum)
sort(totalPropDamage, decreasing = T)[1]
```

```
##    Flood 
## 1.67e+11
```

###B. The percentage of prop damages caused by flood


```r
sort(totalPropDamage, decreasing = T)[1]/sum(totalPropDamage)
```

```
##  Flood 
## 0.4814
```

###C.Top 2 weather events causing the most prop damages


```r
sort(totalPropDamage, decreasing = T)[1:2]
```

```
##     Flood   Tornado 
## 1.670e+11 5.694e+10
```

###D. Percentage of total prop damages caused by the top 2 weather events


```r
sum(sort(totalPropDamage, decreasing = T)[1:2])/sum(totalPropDamage)
```

```
## [1] 0.6456
```

###E. The event that caused most crop damages


```r
totalCropDamage <- tapply(data$cropdamage, data$evtype, sum)
sort(totalCropDamage, decreasing = T)[1]
```

```
##   Drought 
## 1.397e+10
```

###F. The percentage of crop damages caused by drought


```r
sort(totalCropDamage, decreasing = T)[1]/sum(totalCropDamage)
```

```
## Drought 
##  0.3106
```

###G. Top 3 weather events causing the most crop damages


```r
sort(totalCropDamage, decreasing = T)[1:3]
```

```
##   Drought     Flood Ice storm 
## 1.397e+10 1.217e+10 5.022e+09
```

###H. Percentage of total porperty damages caused by the top 3 weather events


```r
sum(sort(totalCropDamage, decreasing = T)[1:3])/sum(totalCropDamage)
```

```
## [1] 0.6927
```

*Figure 1 Barplot of damages to population health*


```r
plot.new()
par(mfrow=c(1,2))
barplot(sort(totalInjuries, 
             decreasing = T), 
             main = "Number of injuries")

barplot(sort(totalFatal, 
             decreasing = T), 
             main = "Number of fatalities")
```

![plot of chunk unnamed-chunk-26](./PA2_template02_files/figure-html/unnamed-chunk-26.png) 

*Figure 2 Barplot of weather events to economic consequences*


```r
par(mfrow=c(1,2))
barplot(sort(totalPropDamage, 
             decreasing = T), 
             main = "Number of property damage")

barplot(sort(totalCropDamage, 
             decreasing = T), main = "Number of crop damages")
```

![plot of chunk unnamed-chunk-27](./PA2_template02_files/figure-html/unnamed-chunk-27.png) 

*References*

1. [Ryan Cheung Git](https://github.com/ryancheunggit)
2. [Dplyr on Stackoverflow](http://stackoverflow.com/questions/tagged/dplyr)
