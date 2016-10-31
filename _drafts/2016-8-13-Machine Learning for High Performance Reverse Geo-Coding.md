---
layout: post
title: "Machine Learning for High Performance Reverse Geo-Coding"
modified: 
categories: blog
excerpt: "Machine Learning and Accelerating Reverse Geo-Coding Throughput by ~ 10^4"
tags: [R, geo-coding, Machine Learning, oregon]
image:
feature:
date: 2016-08-15T08:08:50-04:00
---

### SUMMARY   
Scalable high-performance alternatives to web-lookup and point-in poylgon computation are explored using counties in the state of Oregon as a test case. A Random Forest algorithm improves performance by ~ 10^4^ over web-based API look-ups while meeting >95% accuracy. 

This blog is a higher level summary of a more complete ["computed" blog on RPubs](http://rpubs.com/ww44ss/ML_ReverseGeo)

###Problem Statement
Reverse geo-coded GPS coordinates are useful in cases where GPS cordinates from, for instance, a tweet or photograph are to be assigned to a specific political bounday. While reverse-look up of coordinates can be done using various web-services, such as the Google Maps API, it takes on the order of 200msec per (latitude, longitude) pair, severely limiting computing throughput. In addition, only 2500 free queries per day are allowed, limiting the overall amount of data that can be processed.  
The purpose here to improve dramatically computation times while meeting high ( ~ 98% ) accuracy.  


#### A Note on Performance Measurements  
Performance results can be difficult communicate because they have context only within specific definitions. In this case I standardized data collection as much as possible to ensure a level of comparability. Each measurements was taken in a standardized run framework where the data were divided into training and test data sets and then the modeling and prediction steps were individually timed.   
In some cases these times timing measurements differ slightly from those time observed when computations were done in loops with varying numbers of data points.   
For the API calls I used elapsed-time measurements since these better reflect actual usage.
   
### ANALYSIS  

This analysis has three parts, 

#### getting started

I use the following packages for this analysis.

{% highlight r %}
library(ggmap)
library(dplyr)
library(randomForest)
library(googleway)
library(choroplethrMaps)
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

With the function defined we can start production

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
coords <- data_frame(longitude = runif(number.to.compute, westernmost - 0.1, easternmost + 0.1), 
            latitude = runif(number.to.compute, southernmost - 0.1, northernmost + 0.1) 
            )

## compute counties
get.county.start.time <- proc.time()

    pip.county.result <- pip.oregon.county(coords.x)

get.county.time <- (proc.time() - get.county.start.time)[1]

{% endhighlight %}

By changing `number.to.compute` we can get an idea of how the thruput of this method scales. The results of an experiment are shown below. 


<figure>
<a href="/images/blog/blog_rf_rev_geo_pip.png"><img src="/images/blog/blog_rf_rev_geo_pip.png" alt="image"></a>
</figure>

In practice, the P-i-P is much faster than the API call. Based on a stand-alone benchmark run, for 2000 points, the throughput is 2.26 points per msec - an improvement of roughly a factor of 400 over the API calls.

Note, as the number of points gets large, the thruput (points/time) asymptotes to a constant value of roughly 5.0 points/msec.

#### accuracy of the google API

As a side experiment, I checked the consistency of the results from a Google API call versus the P-i-P method (which is, in principle, exact provided the political boundaries are accurate). Without going into detail here, the result was that taccuracy of the API, as graded against the P-i-P method, is 97.8%.

<figure>
<a href="/images/blog/blog_rf_rev_geo_pip_vs_api.png"><img src="/images/blog/blog_rf_rev_geo_pip_vs_api.png" alt="image"></a>
</figure>

You can see from the above map, most, but not all, of the disagreement points happend on or near boundaries. We'll see this pattern below as well, and it is to be expected.
Deeper investigation is saved as a later activity. 

#### Machine-Learning / random forest

The problem of reverse geocoding, when you think of it, is really almost ideally solved as a multiple-classification tree problem. A series of questions resolving inequalities for latitude and longitide will quickly zero in on the right political boundary.

Since high accuracy is a goal and the accuracy of a random forest improves with the number of training points,  the first step was to find out how many training cases are needed to meet a target of 98% accuracy.

<figure>
<a href="/images/blog/blog_rf_rev_geo_rf_acc_v_ntrain.png"><img src="/images/blog/blog_rf_rev_geo_rf_acc_v_ntrain.png" alt="image"></a>
</figure>
' 

The above figure is produced by this code:


{% highlight r %}
## set number of points to graph
n.accuracy.points <- 21

## prebuild results data_frame
rf.model.char.df <- data_frame("n" = 1:n.accuracy.points, 
"n_train"= 1:n.accuracy.points, 
"model_time" = 1.*1:n.accuracy.points, 
"predict_time" = 1.*1:n.accuracy.points, 
"accuracy" = 1.*1:n.accuracy.points, 
"thruput" = 1.*1:n.accuracy.points, 
"n_test" = 1.*1:n.accuracy.points)

set.seed(8675309)

#Step thru the accuracy points
for (jj in 1:n.accuracy.points){

    rf.model.char.df[jj,1] <- jj

    coords.y <- coords

    n_model <- 510+jj*731

    rf.model.char.df[jj,2] <- n_model

    ## split data sets
    data.select <- sample(1:nrow(coords.y), n_model)
    train.set <- coords.y[data.select,]


    test.set <- coords.y[-data.select,]

    rf.model.char.df[jj, 7] <- nrow(test.set)


    #set.seed(8675309)

    model.start.time <- proc.time()

    rf.model <- randomForest(county ~., train.set, ntree=29)

    rf.model.char.df[jj,3] <- 1000*(proc.time() - model.start.time)[1]

    predict.start.time <- proc.time()
    predicted.test <- predict(rf.model, test.set)

    rf.model.char.df[jj,4] <- 1000*(proc.time() - predict.start.time)[1]

    county.compare <- cbind("actual" = as.character(test.set$county), "model" = as.character(predicted.test)) %>% as.data.frame()
    county.compare$accurate <- "yes"

    ## get rid of factors again
    county.compare$actual <- as.character(county.compare$actual)
    county.compare$model <- as.character(county.compare$model)

    county.compare$accurate[(county.compare[,1] != county.compare[,2])] <- "no"

    plot.data <- cbind("lat" = test.set$lat, "lon" = test.set$lon, "accurate" = county.compare$accurate) %>% as.data.frame

    plot.data$lat <- plot.data$lat %>% as.character %>% as.numeric
    plot.data$lon <- plot.data$lon %>% as.character %>% as.numeric

    plot.data$accurate <- as.factor(plot.data$accurate)


    predict.sum<-table(plot.data$accurate)

    ## calculated accuracy in %
    model.accuracy <- (100*predict.sum[2]/(predict.sum[1]+predict.sum[2])) %>% round(3)

    rf.model.char.df[jj,5] <- model.accuracy

    rf.model.char.df[jj,6] <- n_model/rf.model.char.df[jj,4]

}

{% endhighlight %}

Clearly n_train has a large influence on the accuracy of the model. Since accuracy near 98% is desired will choose about 15000 data points.

Computation times (both total and normalize to the number of points) are shown below. Of interest, the modeling time per point for the randomForest is relatively constant.


<figure>
<a href="/images/blog/blog_rf_rev_geo_ntrain_tpt.png"><img src="/images/blog/blog_rf_rev_geo_ntrain_tpt.png" alt="image"></a>
</figure>

Indeed it is from this data that we can see the Random Forest has ~98% accuracy requirement and also delivers a substantial peformance improvement. 

#### RandomForest performance benchmark

To get formal results we can run this specific benchmark, again relying on `proc.time()` for our measurement.

{% highlight r %}
set.seed(8675309)
## set number of trees
ntree.x = 29

## start model
start.time <- proc.time()

rf.model <- randomForest(county ~., train.set, ntree = ntree.x)

rf.model.time <- (1000.*(proc.time() - start.time)[1]) %>% round(1) %>% as.numeric
rf.model.thruput <- (nrow(train.set) / rf.model.time) %>% round(1)

## start prediction
start.time <- proc.time()

predicted.test <- predict(rf.model, test.set)

rf.predict.time <- (1000.*((proc.time() - start.time)[1])) %>% as.numeric %>% round(2)
rf.predict.thruput <- (nrow(test.set) / rf.predict.time) %>% round(1)
{% endhighlight %}

In a standardized run it took 1110 msec to model the 14700 data points. Predictions made on the remaining 5000 data points took 40 msec, for a throughput of 125 data points per msec.

Model accuracy is 97.6%.

The results of applying a test data set to the model results are mapped below. For the randomForest errors appear on the boundaries of counties. No points lie within the interiors, thus being qualitatively different than errors of the API. Note that errors occur along both straight and complex boundaries, though error rates appear to increase with boundary complexity, as we might expect.


{% highlight r %}

{% endhighlight %}


<figure>
<a href="/images/blog/blog_rf_rev_geo_rf_acc.png"><img src="/images/blog/blog_rf_rev_geo_rf_acc.png" alt="image"></a>
</figure>

#### A note on other classification methods

I tried using the `nnet` model available as a package on [CRAN](https://cran.r-project.org/). Generally the model was inferior. Performance was measured in the same way as for the randomForest. Model times were substantially longer than the randomForest. It took 152.08 seconds to model 10001 data points, while prediction of 5000 points took 30 msec. 

The model was called with the statement below.

{% highlight r %}
decay.x <- 2e-4

nn.model <- nnet(county.class ~., train.set[,-3], size = 12, rang = 0.02, decay = decay.x, maxit = 6000, softmax = TRUE, trace=FALSE) 
{% endhighlight %}






{% highlight r %}

{% endhighlight %}

### Conclusions



method	accuracy (%)	predict.thruput	model.thruput
nnet	94.60	166.67	0.07
randomForest	97.60	125.00	13.20
P-i-P	100.00	2.26	
API	97.80	0.01	

{% highlight r %}

{% endhighlight %}
   


   
   

  




