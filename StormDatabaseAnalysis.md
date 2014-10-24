NOAA_Storm_Database_Analysis
============================

## Synopsis
This report analyzes data in the U.S. National Oceanic and Atmospheric Association's storm database for the purpose of answering a number of questions concerning public health and economic consequences of weather activity.

The data to be analyzed has been collected into a comma-seperated-value file which is compressed using the bzip2 algorithm. The report author followed the assignment instructions by downloading the compressed file from https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2FStormData.csv.bz2. 


There are also two reference documents that the author has reviewed:

* [Storm Data Documentation](https://d396qusza40orc.cloudfront.net/repdata%2Fpeer2_doc%2Fpd01016005curr.pdf)

* [National Climatic Data Center Storm Events FAQ](https://d396qusza40orc.cloudfront.net/repdata%2Fpeer2_doc%2FNCDC%20Storm%20Events-FAQ%20Page.pdf)


## Data Processing
This section covers the processing of the data.

#### Steps Taken
1. Download the compressed data file.
2. Decompress the file, feeding the output to the read.csv function to read the data file into a data.frame.
3. Sum the fatalities by the event type across all regions and store in a new
data frame named harm.

4. Sort harm by fatalities in descending order, bringing the most harmful event types to the top.

5. Display the top six most harmful storm event types.

#### Data Processing Code

```r
library(downloader)
download(url="https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2FStormData.csv.bz2", destfile="StormData.csv.bz2")

data1 <- read.csv(bzfile("StormData.csv.bz2"))

harm <- aggregate(data1$FATALITIES, by=list(data1$EVTYPE), FUN=sum)
colnames(harm) <- c("EventType", "Fatalities")
harm <- harm[with(harm, order(-Fatalities)),]
head(harm)
```

```
##          EventType Fatalities
## 834        TORNADO       5633
## 130 EXCESSIVE HEAT       1903
## 153    FLASH FLOOD        978
## 275           HEAT        937
## 464      LIGHTNING        816
## 856      TSTM WIND        504
```
