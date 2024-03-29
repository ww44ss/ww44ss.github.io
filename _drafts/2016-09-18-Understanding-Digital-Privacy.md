---
layout: post
title: "Understanding Digital Privacy with OkCUpid"
modified: 
categories: blog
excerpt: "Using the OkCupid dataset to understand privacy in the digital era"
tags: [R, OkCupid, Privacy, Machine Learning, Random Forest, GBM]
image:
feature:
date: 2016-09-18T08:08:50-04:00
---

This computation uses these packages:


{% highlight r %}
library(dplyr)
library(ggplot2)
library(RColorBrewer)
library(gridExtra)

library(okcupiddata)
library(randomForest)
library(gbm)
{% endhighlight %}

# Introduction

Privacy is a concern to everyone. But, in an era of machine-learning and advanced data analytics, how does one go about protecting it?   

Intuitively we're familiar with physical privacy. When taking a shower we might step back from an open door or close and lock the door for more robust privacy. Through simple acts we control our physical privacy by limiting what others may see or learn about us. It is our control over information that gives us "privacy."

With the now widespread use of predictive analytics, our personal information faces ever greater threats. Many cases have been popularized, such as the case of [Target knowing a girl was pregnant before her father did](http://www.forbes.com/sites/kashmirhill/2012/02/16/how-target-figured-out-a-teen-girl-was-pregnant-before-her-father-did/#791d11a134c6).

In the case of physical privacy, for example the shower, we know how to protect information. But protecting our digital privacy is another matter. Our intuition fails us when trying to decide what data in one area may compromise more sentivie information in another area; control is not as simple as just closing a door. 

The purpose here is to help build that understanding. I use demographic data from the recently published [OkCupid package on CRAN](https://cran.r-project.org/web/packages/okcupiddata/index.html) to model specific "private" information about users and may contribute to what constitutes an attack on user privacy. While this is case is purely hypothetical, lessons learned in this exercise will help build intuition about what constitutes "closing the door" when it comes to protecting our digital-privacy.

# Problem Statement

Personal income and age are among our most private information. But how private is it, really? Can a "trained" machine, not actually knowing my age, for example, but knowing other things about me, approximatey infer my age (and therefore pose a threat to my privacy). If so, what specific data plays a role in the accuracy of this inference? How might I control it? 

# Data and Data-Treatment

The data from OkCupid are downloaded from the published package on [CRAN](https://cran.r-project.org/web/packages/okcupiddata/index.html). For simplicity, the columns `essay0`, (zodiac) `sign`, and `last_online` are dropped.  
Data are cleaned and simplified for modeling in the following way:   
* rows with NA in age or income are dropped.   
* variables are convereted to factors.   
* education is simplified to include only the first word of the description (this could be improved with some more work) `education_simplified`.   
* ethnicity is simplified to the first descriptor only.    
* religion is simplified to the first descriptor only `religious_affil`.  
* incomes <= 20000 and > 150000 were excluded. Including this data resulted in poor performing models.  
* a `log_income` is computed.  

Here are the variable names of the raw data

{% highlight r %}
colnames(profiles)
{% endhighlight %}

{% highlight r %}
##  [1] "age"         "body_type"   "diet"        "drinks"      "drugs"  
##  [6] "education"   "ethnicity"   "height"      "income"      "job"           
## [11] "last_online" "location"    "offspring"   "orientation" "pets"         
## [16] "religion"    "sex"         "sign"        "smokes"      "speaks"       
## [21] "status"      "essay0"   
{% endhighlight %}

This cleaning method is a bit pedantic, but in practice it was quicker to list everything out and then do modifications as necessary to make the models work.

{% highlight r %}
cleaned <- profiles %>% select(-essay0, -last_online, -sign)
cleaned <- cleaned %>% filter(!is.na(income))
cleaned <- cleaned %>% filter(!is.na(age))


cleaned <- cleaned %>% as_data_frame


cleaned$body_type <- cleaned$body_type  %>% as.factor
cleaned$diet 	  <- cleaned$diet %>% as.factor
cleaned$drinks    <- cleaned$drinks %>% as.factor


cleaned <- cleaned %>% 
	mutate(education_simple = gsub(' [A-z /\\-\\.]*', '', education))
cleaned$education_simple <- cleaned$education_simple %>% as.factor

cleaned$education <- cleaned$education %>% as.factor


cleaned$ethnicity <- gsub(',( [a-z_]*)', '', cleaned$ethnicity) %>% as.factor()


cleaned$location  <- cleaned$location %>% as.factor
cleaned$height    <- cleaned$height %>% as.factor

cleaned$income    <- cleaned$income %>% as.character %>% as.numeric
cleaned           <- cleaned %>% mutate(log_income = log10(income))

cleaned$job       <- cleaned$job %>% as.factor
cleaned$offspring <- cleaned$offspring %>% as.factor
cleaned$orientation <- cleaned$orientation %>% as.factor
cleaned$pets      <- cleaned$pets %>% as.factor
cleaned$religion  <- cleaned$religion %>% as.factor

cleaned           <- cleaned %>% mutate(religious_affil = gsub(' [A-z ]*', '', religion))
cleaned$religious_affil <- cleaned$religious_affil %>% as.factor

cleaned$sex       <- cleaned$sex %>% as.factor
cleaned$smokes    <- cleaned$smokes %>% as.factor
cleaned$status    <- cleaned$status %>% as.factor
{% endhighlight %}


# Inferring Income

Since the point here is to understand what variables have greatest influence and not necessarily build a highly accurate model, I did not spend a lot of effort in feature engineering and instead modeled with just the base parameters listed below. 

{% highlight r %}
## select variables
cleaned_sel <- cleaned %>% select(log_income, sex, drinks, religious_affil, education_simple, age, job, height) #drop ethnicity and offspring

cleaned_sel <- cleaned_sel[complete.cases(cleaned_sel),]

## reduce span of incomes
cleaned_sel <- cleaned_sel %>% filter(log_income < log10(200010))
cleaned_sel <- cleaned_sel %>% filter(log_income > log10(20010))


cleaned_sel <- cleaned_sel %>% as_data_frame
{% highlight r %}

Here are the columns of the cleaned and selected data

{% endhighlight %}
cleaned_sel %>% colnames

[1] "log_income"       "sex"              "drinks"          
[4] "religious_affil"  "education_simple" "age"             
[7] "job"              "height"           
{% endhighlight %}

The span of incomes was reduced because including zero income and very high income degraded the model performance substantially. 

The data are split into __training__ and __test__ data sets using a well known methods. Here is a snapshot of the training data. 

{% highlight r %}
# split data into training and test data sets
set.seed(8675309)
data_cut <- sample(1: nrow(cleaned_sel), 0.6*nrow(cleaned_sel))

## split data
train_data <- cleaned_sel[data_cut,] 
test_data <- cleaned_sel[-data_cut,]
{% endhighlight %}

Here is what the training data looks like

{% highlight r %}
train_data


# A tibble: 3,349 × 8  
   log_income    sex     drinks religious_affil education_simple   age  
        <dbl> <fctr>     <fctr>          <fctr>           <fctr> <int>  
1    4.903090      m     rarely     agnosticism        graduated    59  
2    4.903090      m   socially           other        graduated    38  
3    5.000000      m   socially         atheism        graduated    43  
4    4.778151      m      often         atheism        graduated    28  
5    4.477121      m     rarely         atheism   graduated-year    50  
6    4.845098      m   socially         atheism        graduated    26   
7    4.903090      m   socially     catholicism        graduated    27  
8    4.602060      m   socially     agnosticism   graduated-year    26  
9    4.477121      m not at all    christianity             high    43  
10   5.176091      m     rarely           other        graduated    58   
# ... with 3,339 more rows, and 2 more variables: job <fctr>,   
#   height <fctr>   
{% endhighlight %}

## Random-Forest Model

{% highlight r %}
model.start <- Sys.time()
income_model <- randomForest(log_income ~., train_data, importance = TRUE, ntree = 300)
model.time <- Sys.time()-model.start

grounded.truth <- test_data$log_income

model.output <- predict(income_model, test_data %>% select(-log_income))
{% endhighlight %}

Overall it took 15.69 seconds to model 3349 points points. The variable importance is shown below as a % increase in mean square error, with larger values being more important.   

Here is the importanve of the income model.

{% highlight r %}
# sort model importance
importance(income_model, type=1)

##                    %IncMSE
## sex              18.416627
## drinks            2.629561
## religious_affil   8.079117
## education_simple 39.414630
## age              55.716573
## job              74.630527
## height            5.692647
{% endhighlight %}

Your job, age and education level have the greatest influence on income.  
Another way to understand privacy implications is to plot predicted income (from the model) against actual income (known from test data) as below.

{% highlight r %}
plot_df <- cbind(grounded.truth, model.output) %>% as_data_frame
plot_df$sample <- 1:nrow(plot_df)

log_breaks = c(20000, 30000, 40000, 50000, 70000, 80000, 100000, 150000, 200000, 500000, 1000000)

random_forest_plot <- ggplot(plot_df, aes(y = 10^(model.output), x = 10^(grounded.truth), group=grounded.truth)) + 
    geom_boxplot(fill = "#CCCCCC")+
    geom_jitter(pch=21, fill = "#77EE11", color = "#7788EE", width = .06, size = 0.8) +
    scale_y_log10(breaks = log_breaks, labels=log_breaks) + 
    scale_x_log10(breaks = log_breaks, labels=log_breaks) + 
    ggtitle("randomForest income model") +
    xlab("actual income") + 
    ylab("inferred income") 


random_forest_plot %>% print

{% endhighlight %}

![center](/figures/2016-09-18-Understanding-Digital-Privacy/unnamed-chunk-8-1.png) 

The graph shows both individual data points as well as a box-plot, showing the mean and standard deviation of the predictions. Clearly, the model does not predict income with high accuracy (the model is very basic and no doubt could be improved with added work). But, it does a reasonable job finding the trend. Without giving the model any information about income, it can, based on other information about you, at least reasonably guess your relative income. If you viewed your income as private, your privacy would be under attack.

Overall:   
* Based on the given data, the `randomForest` model reproduces an income trend, though not with high accuracy. It would be sufficient to tell if you had high or low income, but not necessarily what level.   
* Variables having the most influence are (in order): job, age, education, sex, and religious affiliation.    
* It is likely that with more _feature engineering_ the accuracy of the model could be improved.   


## Gradient Boosting Machine (gbm)

The `gbm` model is another very commonly used regression model and provides good insight into the influence of variables.  

{% highlight r %}
model.start <- Sys.time()
income_model <- gbm(log_income ~ sex + drinks + religious_affil + education_simple + age + job + height, 
    data = train_data, 
    n.trees = 200, 
    shrinkage = 0.001,
    interaction.depth=5,
    n.minobsinnode = 10,
    bag.fraction = 0.1,
    distribution = "gaussian")

model.time <- Sys.time()-model.start

grounded.truth <- test_data$log_income

model.output <- predict(income_model, test_data %>% select(-log_income), n.trees = 100)

{% endhighlight %}

Overall it took 0.21 seconds to model 3349 points. This is a factor of roughly 60 times faster than the random forest. The relative influence of variables is shown below. 

{% highlight r %}
relative.influence(income_model,n.trees = 100)
{% endhighlight %}

`
##              sex           drinks  religious_affil education_simple 
##        0.4575169        0.0000000        6.6358175       55.3464937 
##              age              job           height 
##      104.8505992      235.1825972       53.0836639
`

The most important factors are your job, age, education, though interestingly in this case it finds height is also a predictor. 

A plot of the predicted incomes against actual incomes show that in general the `gbm` does not do as well as the `randomForest` regression model. 

{% highlight r %}
plot_df <- cbind(grounded.truth, model.output) %>% as_data_frame
plot_df$sample <- 1:nrow(plot_df)

log_breaks = c(20000, 30000, 40000, 50000, 60000, 70000, 80000, 100000, 150000, 200000, 500000, 1000000)

ggplot(plot_df, aes(y = 10^model.output, x = 10^grounded.truth, group=grounded.truth)) + 
    geom_boxplot(fill = "#CCCCCC")+
    geom_jitter(pch=21, fill = "#77EE11", color = "#7788EE", width = .06, size = 0.8) +
    scale_y_log10(breaks = log_breaks, labels=log_breaks) + 
    scale_x_log10(breaks = log_breaks, labels=log_breaks) + 
    ggtitle("gbm income model") +
    xlab("actual income") + 
    ylab("inferred income") +
    geom_smooth()
{% endhighlight %}


![center](/figures/2016-09-18-Understanding-Digital-Privacy/unnamed-chunk-11-1.png)

Again:  
* Job, education, and age have predictive value in a gbm scenario.  This agrees with the finding of the `randomForest` and thus shows a kind of "robustness" of the power of certain variables in predicting others.   
* Sex, drinking, religious_affiliation, and height have no influence.   
* The gbm does a poorer job of predicting income than `randomForest` but nevertheless has some predictive value.   

# Inferring Age

We construct an age model in the same way as above, though only for the `randomForest` case.

{% highlight r %}
model.start <- Sys.time()
age_model <- randomForest(age ~., train_data, importance = TRUE, ntree = 300)
model.time <- Sys.time()-model.start

grounded.truth <- test_data$age

model.output <- predict(age_model, test_data %>% select(-age))
{% endhighlight %}

Overall it took 12.60 seconds to model 3349 points. The model importance is shown below.  

{% highlight r %}
importance(age_model, type=1)
{% endhighlight %}

`
##                    %IncMSE
## log_income       38.920995
## sex               4.837325
## drinks           19.695133
## religious_affil  11.657625
## education_simple 22.064568
## job              36.017347
## height            1.774081
`

We can see that while the numerical accuracy of the age predictions is not great, it does accurately predict an age trend and thus is reasonably capable of reliably distinguishing something like an age-group. 

![center](/figures/2016-09-18-Understanding-Digital-Privacy/unnamed-chunk-14-1.png)

This means:  
* Your income, job, education, drinking habits, and religious affiliation, when taken together, provide information about your age.  

# Summary and Conclusions  
In the age of heavy analytics and shared data, seemingly ancillary data, when combined with other data, can reveal information about us we had assumed was private.  
We can see it is not just a single piece of information providing the lynch-pin in protecting our privacy, but rather an ensemble. This increases the difficulty of protecting your data. 

If we assume our income, for example, is private, but reveal our job, education, and age (as I have done in consumer surveys), it may be as though we have lost our privacy all the same (whether or not the model is actually built). 

What protections do we have? While worthy of another blog, I'll just list some brainstorm ideas here in case that takes a while. Note these are still to be evaluated.  

Possible steps to control digital-privacy:  
* Use dummy data. I noticed that the predictive models were thrown off when I included both the lower and highest income levels. It is imaginable that this data was less than reliable. Dummy data skews models.   
* Understand what data are being collected and what can be inferred from it. That is easier said than done, but perhaps a kind of "dashboard" could be created or provided as a service. (buying diapers and baby stroller ->> baby, for example?). This could be promising since we have some evidence of "robustness" of variables independent of the model. 
* Enable consumers to protect what can be modeled about them thru extensions of privacy laws. This is obviously problematic from an enforcement perspective. 
* others???    

