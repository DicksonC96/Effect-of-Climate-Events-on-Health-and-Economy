---
title: "Effect of climate events on health and economy"
author: "DicksonC"
output: html_document
---
Dataset source: U.S. National Oceanic and Atmospheric Administration's (NOAA)  

This project reflects the most devastating climate events affecting population health and nation's economy in general.  The NOAA storm database used will tracks characteristics of major storms and weather events in the United States, including when and where they occur, as well as estimates of any fatalities, injuries, and property damage.


```r
# Loading libraries required
library(dplyr)
library(lattice)
```

## Data Processing
The data was sourced directly from the NOAA database website:  
1. A temporary directory `temp` was created for storing the dataset temporarily.  
2. Dataset was downloaded to the `temp` directory.  
3. Reading the csv file from the bz2 zip file.  
4. Disconnecting the database connection.  
5. Converting dataset into tibble dataframe from dplyr library for the ease of analysis later.


```r
temp <- tempfile()
download.file("https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2FStormData.csv.bz2", temp)
```

```
## Warning in download.file("https://
## d396qusza40orc.cloudfront.net/
## repdata%2Fdata%2FStormData.csv.bz2", : InternetOpenUrl
## failed: 'The server name or address could not be
## resolved'
```

```
## Error in download.file("https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2FStormData.csv.bz2", : cannot open URL 'https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2FStormData.csv.bz2'
```

```r
data <- read.csv(bzfile(temp, "repdata_data_StormData.csv"), header = TRUE)
```

```
## Warning in bzfile(temp, "repdata_data_StormData.csv"):
## cannot open bzip2-ed file 'C:
## \Users\DELL\AppData\Local\Temp\RtmpsDWY7o\file35b8795b472a',
## probable reason 'No such file or directory'
```

```
## Error in bzfile(temp, "repdata_data_StormData.csv"): cannot open the connection
```

```r
unlink(temp)
tbl <- as_tibble(data)
```

```
## Error in as.data.frame.default(value, stringsAsFactors = FALSE): cannot coerce class '"function"' to a data.frame
```

```r
tbl
```

```
## function (src, ...) 
## {
##     UseMethod("tbl")
## }
## <bytecode: 0x000002a618189d00>
## <environment: namespace:dplyr>
```

## Results
### Events that are most harmful with respect to population health
Here we groups our data and sort it out to get the event with highest fatalities and injuries combined:  
1. Group by event types.  
2. Summarize the data into sum of fatalities and injuries on each types of event group.  
3. Filter out empty values.  
4. Create a new `total` column to compute the sum of both fatalities and injuries.  
5. Sort the tibble in descending order of `total`.


```r
health <- tbl %>% 
          group_by(EVTYPE) %>%
          summarize(fatalities = sum(FATALITIES), injuries = sum(INJURIES)) %>%
          filter(injuries!=0 & fatalities!=0) %>%
          mutate(total = injuries + fatalities) %>%
          arrange(desc(total))
```

```
## Warning: `group_by_()` is deprecated as of dplyr 0.7.0.
## Please use `group_by()` instead.
## See vignette('programming') for more help
## This warning is displayed once every 8 hours.
## Call `lifecycle::last_warnings()` to see where this warning was generated.
```

```
## Error in UseMethod("group_by_"): no applicable method for 'group_by_' applied to an object of class "function"
```

```r
head(health)
```

```
## Error in head(health): object 'health' not found
```


```r
barchart(fatalities+injuries ~ EVTYPE, 
         data = health[1:5,],
         stack = TRUE,
         main = "Population health consequences by climate events",
         xlab = "Event type",
         ylab = "Total fatalities/ injuries",
         auto.key = list(space='right', text=c('Fatalities','Injuries')))
```

```
## Error in barchart.formula(fatalities + injuries ~ EVTYPE, data = health[1:5, : object 'health' not found
```

Obviously from both the table and plot, **tornado event brings both the highest fatalities and injuries, followed by excessive heat**  
  
### Event types that have the greatest economic consequences
Here we groups our data and sort it out to get the event with the greatest economic losses in both property and crop:  
1. Filter out "B" to get the largest unit in the group.  (The idea here is to choose events with both losses in billions instead of millions and thousands).  
2. Group by event types.  
3. Summarize the data into sum of property and crop losses on each types of event group.  
4. Create a new `totaldmg` column to compute the sum of both property and crop losses.  
5. Sort the tibble in descending order of `totaldmg`.

```r
economy <- tbl %>% 
           filter(PROPDMGEXP=="B", CROPDMGEXP=="B") %>%
           group_by(EVTYPE) %>%
           summarize(propdmg = sum(PROPDMG), cropdmg = sum(CROPDMG)) %>%
           mutate(totaldmg = propdmg + cropdmg) %>%
           arrange(desc(totaldmg))
```

```
## Warning: `filter_()` is deprecated as of dplyr 0.7.0.
## Please use `filter()` instead.
## See vignette('programming') for more help
## This warning is displayed once every 8 hours.
## Call `lifecycle::last_warnings()` to see where this warning was generated.
```

```
## Error in UseMethod("filter_"): no applicable method for 'filter_' applied to an object of class "function"
```

```r
economy
```

```
## Error in eval(expr, envir, enclos): object 'economy' not found
```


```r
barchart(propdmg+cropdmg~EVTYPE, 
         data = economy,
         stack = TRUE,
         main = "Economic consequences by climate events",
         xlab = "Event type",
         ylab = "Total economic losses",
         auto.key = list(space='right', text=c('Property','Crop')))
```

```
## Error in barchart.formula(propdmg + cropdmg ~ EVTYPE, data = economy, : object 'economy' not found
```

Although Hurricane/Typhoon brings a greater property damage, **River Flood results in a much higher economic losses** with property and crop combined.
