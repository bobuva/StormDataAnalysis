Analysis of NOAA Storm Database
===============================

## Synopsis
This report analyzes data in the U.S. National Oceanic and Atmospheric Association's storm database for the purpose of answering a number of questions concerning public health and economic consequences of weather activity.

The data to be analyzed has been collected into a comma-seperated-value file which is compressed using the bzip2 algorithm. The report author followed the assignment instructions by downloading the compressed file from https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2FStormData.csv.bz2. 


There are also two reference documents that the author has reviewed:

* [Storm Data Documentation](https://d396qusza40orc.cloudfront.net/repdata%2Fpeer2_doc%2Fpd01016005curr.pdf)

* [National Climatic Data Center Storm Events FAQ](https://d396qusza40orc.cloudfront.net/repdata%2Fpeer2_doc%2FNCDC%20Storm%20Events-FAQ%20Page.pdf)


## Data Processing
This section covers the processing of the data.

#### Steps Taken
1. Set the knitr cache option to TRUE for the code block in which the data is downloaded, decompressed and read into a data.frame. These are the most time-consuming operations and will only be re-executed if there is no cached data or if there was a change in the data.
2. Download the compressed data file.
3. Decompress the file, feeding the output to the read.csv function to read the data file into a data.frame.

 TODO: Subset the data by removing all entries where fatalities is 0 or injuries is 0. Then create groups of the EVTYPE factor like this:
 
 levels(df$type) <- list(
    animal = c("cow", "pig"),
    bird = c("eagle", "pigeon")
    
4. Process the data for population health effects.

    Step 1: Sum the fatalities by the event type across all regions and store in a new data frame named 'fatals'.
    Step 2: Sort 'fatals' by fatalities in descending order, bringing the most fatal event types to the top.
    Step 3: Create a new data.frame named 'mostfatal' containing the top ten rows of the 'fatals' data.frame.
    Step 4: Perform steps 1 through 3 again but using the INJURIES column of the original data in place of FATALITIES.
5. Display side-by-side histograms showing the most fatal and the most injurious event types.
6. Process the data for economic consequences.


#### Data Processing Code

```r
library(downloader)
download(url="https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2FStormData.csv.bz2", destfile="StormData.csv.bz2")

data1 <- read.csv(bzfile("StormData.csv.bz2"))
```


```r
fatals <- aggregate(data1$FATALITIES, by=list(data1$EVTYPE), FUN=sum)
colnames(fatals) <- c("EventType", "Fatalities")
fatals <- fatals[with(fatals, order(-Fatalities)),]
mostfatal <- fatals[1:10,]

injuries <- aggregate(data1$INJURIES, by=list(data1$EVTYPE), FUN=sum)
colnames(injuries) <- c("EventType", "Injuries")
injuries <- injuries[with(injuries, order(-Injuries)),]
mostinjuries <- injuries[1:10,]
```

```r
damage <- data1[, c("EVTYPE", "PROPDMG", "PROPDMGEXP", "CROPDMG", "CROPDMGEXP")]

damage$PropDamage <- damage$PROPDMG
hasB_prop <- which(damage$PROPDMG > 0 & damage$PROPDMGEXP == "B")
for (i in hasB_prop) damage[i, "PropDamage"] <- damage[i, "PROPDMG"] * 1000000

hasM_prop <- which(damage$PROPDMG > 0 & damage$PROPDMGEXP == "M")
for (i in hasM_prop) damage[i, "PropDamage"] <- damage[i, "PROPDMG"] * 1000

damage$CropDamage <- damage$CROPDMG
hasB_crop <- which(damage$CROPDMG > 0 & damage$CROPDMGEXP == "B")
for (i in hasB_crop) damage[i, "CropDamage"] <- damage[i, "CROPDMG"] * 1000000

hasM_crop <- which(damage$CROPDMG > 0 & damage$CROPDMGEXP == "M")
for (i in hasM_crop) damage[i, "CropDamage"] <- damage[i, "CROPDMG"] * 1000

propdamage <- aggregate(damage$PropDamage, by=list(damage$EVTYPE), FUN=sum)
cropdamage <- aggregate(damage$CropDamage, by=list(damage$EVTYPE), FUN=sum)
colnames(propdamage) <- c("EventType", "Damage")
colnames(cropdamage) <- c("EventType", "Damage")
propdamage <- propdamage[with(propdamage, order(-Damage)),]
cropdamage <- cropdamage[with(cropdamage, order(-Damage)),]

combined <- merge(propdamage, cropdamage, by="EventType", all.x=TRUE, all.y=TRUE)
combined <- transform(combined, TotalDamage = Damage.x + Damage.y)
colnames(combined) <- c("EventType", "Property", "Crop", "Total")
combined <- combined[with(combined, order(-Total, -Property, -Crop)),]
mostdamage <- combined[1:10,]

mostpropdamage <- propdamage[1:10,]
mostcropdamage <- cropdamage[1:10,]
```

#### Results
These plots show just the top ten most fatal and the top ten most injurious event types. As the histogram plots show, by a large margin, tornadoes cause the most fatalities and the most injuries of the storm event types in the study. 

```r
library(ggplot2)
library(grid)

# function for displaying any plot in the grid
vplayout <- function(x, y) viewport(layout.pos.row = x, layout.pos.col = y)

plot1 <- ggplot(mostfatal, aes( x = EventType, y = Fatalities)) + geom_histogram(stat= "identity") + theme(axis.text.x = element_text(size=8, angle=45, hjust=1))
plot2 <- ggplot(mostinjuries, aes( x = EventType, y = Injuries)) + geom_histogram(stat= "identity") + theme(axis.text.x = element_text(size=8, angle=45, hjust=1))

grid.newpage()
pushViewport(viewport(layout = grid.layout(1, 2)))
print(plot1, vp = vplayout(1, 1))
print(plot2, vp = vplayout(1, 2))
```

![plot of chunk unnamed-chunk-2](figure/unnamed-chunk-2.png) 

The next plots show the top ten most damaging events to property on the left and to crops on the right.


```r
plot3 <- ggplot(mostdamage, aes(x=EventType, y=Total)) + geom_histogram(stat= "identity") +  geom_bar(width = 0.8, position = position_dodge(width = 0.9)) + theme(axis.text.x = element_text(size=8, angle=45, hjust=1))
plot4 <- ggplot(mostpropdamage, aes(x=EventType, y=Damage)) + geom_histogram(stat= "identity") + theme(axis.text.x = element_text(size=8, angle=45, hjust=1))
plot5 <- ggplot(mostcropdamage, aes(x=EventType, y=Damage)) + geom_histogram(stat= "identity") + theme(axis.text.x = element_text(size=8, angle=45, hjust=1))

grid.newpage()
pushViewport(viewport(layout = grid.layout(3, 1)))
print(plot3, vp = vplayout(1, 1))
print(plot4, vp = vplayout(2, 1))
print(plot5, vp = vplayout(3, 1))
```

![plot of chunk unnamed-chunk-3](figure/unnamed-chunk-3.png) 
