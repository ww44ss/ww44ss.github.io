---
layout: post
title: "Machine Learning for High Performance Reverse Geo-Coding"
modified: 
categories: blog
excerpt: "Accelerating Reverse Geo-Coding perfomance ~ 10^4^ with Random Forests"
tags: [R, geo-coding, Machine Learning]
image:
feature:
date: 2016-08-15T08:08:50-04:00
---

### SUMMARY   
Scalable high-performance alternatives to web-lookup and point-in poylgon computation are explored using counties in the state of Oregon as a test case. A Random Forest algorithm improves performance by ~ 10^4^ over web-based API look-ups while meeting >95% accuracy. 

###Problem Statement
Reverse geo-coded GPS coordinates are useful in cases where GPS cordinates from, for instance, a tweet or photograph are to be assigned to a specific political bounday. While reverse-look up of coordinates can be done using various web-services, such as the Google Maps API, it takes on the order of 200msec per (latitude, longitude) pair, severely limiting computing throughput. In addition, only 2500 free queries per day are allowed, limiting the overall amount of data that can be processed.  
The purpose here to improve dramatically computation times while meeting high ( ~ 98% ) accuracy.  


####A Note on Performance Measurements  
Performance results are difficult communicate because they have context only within specific definitions. In this case I standardized data collection as much as possible to ensure some level of comparability. Each measurements was taken in a standardized run framework where the data were divided into training and test data sets and then the modeling and prediction steps were individually timed.   
In some cases these times timing measurements differ slightly from those time observed when computations were done in loops with varying numbers of data points.   
For the API calls I used elapsed-time measurements since these better reflect actual usage.
   
### ANALYSIS  

#### getting started

I use the following packages for this analysis.

{% highlight r %}
library(ggmap)
library(dplyr)
library(randomForest)
library(googleway)
library(choroplethrMaps)
library(nnet)
library(SDMTools)
library(tidyr)
library(stringr)
library(xtable)
    options(xtable.floating = FALSE)
    options(xtable.timestamp = "")
library(gridExtra)
    options(scipen=999)
{% endhighlight %}



#### Google API performance

Data on the reverse geocoding times were collected using the Google Maps API and the `{googleway}` package. Geo-locations are reverse coded using the following function


{% highlight r %}
data <- google_reverse_geocode(location = c(as.numeric(x["lat"]), as.numeric(x["lon"])), key = secret.key)
{% endhighlight %}

where `secret.key` is a private key generated from my Google Maps API account. 

The time for each API call was recorded as the `elapsed time` as measured by the `proc.time()` function, since this reflects the usage model. Throughput is limited by the network speed. 

The data were collected by a separate _R_ program which can be viewed in the associated github repository. The program pusjed several hundred random GPS pairs and stored the time the query took to complete in seconds. The data and time of the experiment were encoded in the file name and that is extracted by the code below. 

{% highlight r %}

directory <- "/Users/Me/Documents/CountyMapperModelTest/"

file.list <- list.files(directory)
data.files <- file.list[grepl(".csv", file.list)]

time.data <- NULL

for (file in data.files) {

    tmp <- read.csv(paste0(directory, file), stringsAsFactors=F) %>% as.data.frame
    tmp$datetime <- rep(str_extract(file, "([0-9]|-|:| )+"), nrow(tmp))

    ## keep only first 374 points from each sample
    if (nrow(tmp) > 374) tmp <- tmp[1:374,]
    ## keep sample only if 374 points
    if (nrow(tmp) == 374) time.data <- rbind(time.data,tmp)
}

## convert to msecs
time.data$time <- 1000* time.data$time

p <- ggplot(time.data, aes(x = time, fill = datetime, color = datetime)) + 
        geom_histogram(stat="bin", breaks=seq(0, 500, 5)) + 
        xlab("time (msec)") + 
        ggtitle("reverse-geo-code User times") + 
        guides(fill=guide_legend(ncol=3))
{% endhighlight %}

<figure>
<a href="/images/blog/blog_rf_rev_geo.png"><img src="/images/blog/blog_rf_rev_geo.png" alt="image"></a>
</figure>

The median measured time per reverse geo-code data point is 176 msec with a 95th quantile of 275 msec, representing a thruput of 0.0057 points/msec.






#### Point in Polygon

Another method is the point in polygon (p-i-p) method. This method is inherently serial (each point must be checked for each boundary)and assumes the GPS boundaries of the counties are known. There are several packages on CRAN which can be used to compute p-i-p. In this case, the I used the `{SDMTools}` version. 

The first step is to define a function to compute PIP for us
{% highlight r %}
## get the boundaries for Oregon counties from ggmap
require(ggmap)
    if (!exists("counties")) counties <- map_data("county")
    if (!exists("state_county")) state_county <- subset(counties, region == 'oregon')

## define function

pip.oregon.county <- function(coords.x){

    ## input: coords.x = data frame of (lat, lon) pairs
    ## returns: data frame with added column pip.county
    ## requires: ggmap and counties data

    coords.x<-as_data_frame(coords.x)
    ## create a placeholder column
    coords.x$pip.county <- rep(0, nrow(coords.x))

    for (county.x in unique(state_county$subregion)) {

        county.x.boundary <- filter(state_county, subregion == county.x)

        A <- pnt.in.poly(coords.x[,1:2], county.x.boundary[,1:2]) %>% as_data_frame

        ## this is a little awkward
        ## possibly not good programming practice to index one df from another
        coords.x[A$pip == 1,3] <- county.x
    }

    ## assign "none" to those not in Oregon
    coords.x[coords.x$pip.county == 0,3] <- "outside"

    return(coords.x)
}
{% endhighlight %}



{% highlight r %}
## compute benchmark
number.to.compute <- 2000

## ensure reproducibility
set.seed(8675309)

northernmost <- max(state_county$lat)
southernmost <- min(state_county$lat)
easternmost <- max(state_county$long)
westernmost <- min(state_county$long)

## generate random points
coords <- data_frame( longitude = runif(number.to.compute, westernmost - 0.1, easternmost + 0.1), latitude = runif(number.to.compute, southernmost - 0.1, northernmost + 0.1) )

GPS_data    
get.county.start.time <- proc.time()

    pip.county.result <- pip.oregon.county(coords.x)

get.county.time <- (proc.time() - get.county.start.time)[1]

{% endhighlight %}

By changing `number.tom.compute` we can get an idea of how the thruput of this method scales. The results of an experiment are shown below. 


<figure>
<a href="/images/blog/blog_rf_rev_geo_pip.png"><img src="/images/blog/blog_rf_rev_geo_pip.png" alt="image"></a>
</figure>

In practice, the P-i-P is much faster than the API call. Based on a stand-alone benchmark run, for 2000 points, the throughput is 2.26 points per msec - an improvement of roughly a factor of 400 over the API calls.

Note, as the number of points gets large, the thruput (points/time) asymptotes to a constant value of roughly 5000 points/msec

#### random forest

{% highlight r %}

{% endhighlight %}




{% highlight r %}

{% endhighlight %}




{% highlight r %}

{% endhighlight %}




{% highlight r %}

{% endhighlight %}




{% highlight r %}

{% endhighlight %}




{% highlight r %}

{% endhighlight %}
   


   
   

  





