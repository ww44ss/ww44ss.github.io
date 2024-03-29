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



{% highlight r %}
knitr::opts_chunk$set(echo = FALSE, fig.width=7, fig.height=4.5, messages=FALSE, warning = FALSE, fig.align = 'center')
options(scipen=999)
{% endhighlight %}

{% endhighlight %}

library(dplyr)
library(ggplot2)
library(RColorBrewer)
library(gridExtra)

library(okcupiddata)
library(randomForest)
library(gbm)

```

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

```{r, message=FALSE, warning=FALSE}
colnames(profiles)

cleaned <- profiles %>% select(-essay0, -last_online, -sign)
cleaned <- cleaned %>% filter(!is.na(income))
cleaned <- cleaned %>% filter(!is.na(age))


cleaned <- cleaned %>% as_data_frame

## straighten into factors
cleaned$body_type <- cleaned$body_type  %>% as.factor
cleaned$diet <- cleaned$diet %>% as.factor
cleaned$drinks <- cleaned$drinks %>% as.factor


cleaned <- cleaned %>% mutate(education_simple = gsub(' [A-z /\\-\\.]*', '', education)) 
cleaned$education_simple <- cleaned$education_simple %>% as.character %>% as.factor
cleaned$education <- cleaned$education %>% as.factor


cleaned$ethnicity <- gsub(',( [a-z_]*)', '', cleaned$ethnicity) %>% as.factor()


cleaned$location <- cleaned$location %>% as.factor
cleaned$height <- cleaned$height %>% as.factor
cleaned$income <- cleaned$income %>% as.character %>% as.numeric
cleaned <- cleaned %>% mutate(log_income = log10(income))
cleaned$job <- cleaned$job %>% as.factor
cleaned$offspring <- cleaned$offspring %>% as.factor
cleaned$orientation <- cleaned$orientation %>% as.factor
cleaned$pets <- cleaned$pets %>% as.factor
cleaned$religion <- cleaned$religion %>% as.factor
cleaned <- cleaned %>% mutate(religious_affil = gsub(' [A-z ]*', '', religion))
cleaned$religious_affil <- cleaned$religious_affil %>% as.factor

cleaned$sex <- cleaned$sex %>% as.factor
cleaned$smokes <- cleaned$smokes %>% as.factor
cleaned$status <- cleaned$status %>% as.factor
```


# Inferring Income

Since the point here is to understand what variables have greatest influence and not necessarily build a highly accurate model, I did not spend a lot of effort in feature engineering and instead modeled with just the base parameters listed below. 



```{r, echo=FALSE, warning = FALSE, fig.align='center', tidy = TRUE, comment=""}
## select variables
cleaned_sel <- cleaned %>% select(log_income, sex, drinks, religious_affil, education_simple, age, job, height) #drop ethnicity and offspring

#print(c("cleaned before complete cases", nrow(cleaned_sel)))
cleaned_sel <- cleaned_sel[complete.cases(cleaned_sel),]
#print(c("cleaned after complete cases", nrow(cleaned_sel)))

## reduce span of incomes
cleaned_sel <- cleaned_sel %>% filter(log_income < log10(200010))
cleaned_sel <- cleaned_sel %>% filter(log_income > log10(20010))


cleaned_sel <- cleaned_sel %>% as_data_frame

cleaned_sel %>% colnames
```

The data are split into __training__ and __test__ data sets using a well known methods. Here is a snapshot of the training data. 

```{r, echo=1:7, warning = FALSE, fig.align='center', tidy = TRUE, comment=""}
# split data into training and test data sets
set.seed(8675309)
data_cut <- sample(1: nrow(cleaned_sel), 0.6*nrow(cleaned_sel))

## split data
train_data <- cleaned_sel[data_cut,] 
test_data <- cleaned_sel[-data_cut,]

train_data
```

## Random-Forest Model

```{r, echo=1:7, warning = FALSE, fig.align='center', tidy = TRUE}

model.start <- Sys.time()
income_model <- randomForest(log_income ~., train_data, importance = TRUE, ntree = 300)
model.time <- Sys.time()-model.start

grounded.truth <- test_data$log_income

model.output <- predict(income_model, test_data %>% select(-log_income))

```

Overall it took `r model.time %>% round(3)` seconds to model `r nrow(train_data)` points. The variable importance is shown below as a % increase in mean square error, with larger values being more important.   

```{r, echo=FALSE, warning = FALSE, fig.align='center', tidy = TRUE}

# sort model importance
importance(income_model, type=1)

plot_df <- cbind(grounded.truth, model.output) %>% as_data_frame
plot_df$sample <- 1:nrow(plot_df)

log_breaks = c(20000, 30000, 40000, 50000, 70000, 80000, 100000, 150000, 200000, 500000, 1000000)

random_forest_plot <- ggplot(plot_df, aes(y = 10^(model.output), x = 10^(grounded.truth), group=grounded.truth)) + 
geom_boxplot(fill = "#CCCCCC")+
#geom_boxplot() + 
geom_jitter(pch=21, fill = "#77EE11", color = "#7788EE", width = .06, size = 0.8) +
scale_y_log10(breaks = log_breaks, labels=log_breaks) + 
scale_x_log10(breaks = log_breaks, labels=log_breaks) + 
ggtitle("randomForest income model") +
xlab("actual income") + 
ylab("inferred income") +
geom_smooth()



```

Your job, age and education level have the greatest influence on income.  
Another way to understand privacy implications is to plot predicted income (from the model) against actual income (known from test data) as below.

```{r}
random_forest_plot %>% print
```

The graph shows both individual data points as well as a box-plot, showing the mean and standard deviation of the predictions. Clearly, the model does not predict income with high accuracy (the model is very basic and no doubt could be improved with added work). But, it does a reasonable job finding the trend. Without giving the model any information about income, it can, based on other information about you understand your relative income. If information about your income was someting you wanted to protect, your privacy would be under attack.

Overall:   
* Based on the given data, the `randomForest` model reproduces an income trend, though not with high accuracy. It would be sufficient to tell if you had high or low income, but not necessarily what level.   
* Variables having the most influence are (in order): job, age, education, sex, and religious affiliation.    
* It is likely that with more _feature engineering_ the accuracy of the model could be improved.   


## Gradient Boosting Machine (gbm)

The `gbm` model is another very commonly used regression model and provides good insight into the influence of variables.  

```{r, echo=3:8, warning = FALSE, fig.align='center', tidy = TRUE}

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

```

Overall it took `r model.time` seconds to model `r nrow(train_data)` points. The relative influence of variables is shown below. 

```{r}
relative.influence(income_model,n.trees = 100)
```


A plot of the predicted incomes against actual incomes show that in general the `gbm` does not do as well as the `randomForest` regression model. 

```{r, echo=FALSE, warning = FALSE, fig.align='center', tidy = TRUE}


plot_df <- cbind(grounded.truth, model.output) %>% as_data_frame
plot_df$sample <- 1:nrow(plot_df)

log_breaks = c(20000, 30000, 40000, 50000, 60000, 70000, 80000, 100000, 150000, 200000, 500000, 1000000)

ggplot(plot_df, aes(y = 10^model.output, x = 10^grounded.truth, group=grounded.truth)) + 
geom_boxplot(fill = "#CCCCCC")+
#geom_boxplot() + 
geom_jitter(pch=21, fill = "#77EE11", color = "#7788EE", width = .06, size = 0.8) +
scale_y_log10(breaks = log_breaks, labels=log_breaks) + 
scale_x_log10(breaks = log_breaks, labels=log_breaks) + 
ggtitle("gbm income model") +
xlab("actual income") + 
ylab("inferred income") +
geom_smooth()



```

Again:  
* Job, education, and age have predictive value in a gradient-boost model scenario.  This agrees with the finding of the `randomForest` and thus shows a kind of "robustness" of the power of certain variables in predicting others.   
* Sex, drinking, religious_affiliation, and height have no influence.   
* The `gbm` does a poorer job of predicting income than `randomForest` but nevertheless has some predictive value.   

# Inferring Age

We construct an age model in the same way as above, though only for the `randomForest` case.

```{r, echo=3, warning = FALSE, fig.align='center', tidy = TRUE}

model.start <- Sys.time()
age_model <- randomForest(age ~., train_data, importance = TRUE, ntree = 300)
model.time <- Sys.time()-model.start

grounded.truth <- test_data$age

model.output <- predict(age_model, test_data %>% select(-age))

```
Overll it took `r model.time` seconds to model `r nrow(train_data)` points. The model importance is shown below.   

```{r, echo=FALSE, warning = FALSE, fig.align='center', tidy = TRUE}

# sort model importance
importance(age_model, type=1)

plot_df <- cbind(grounded.truth, model.output) %>% as_data_frame
plot_df$sample <- 1:nrow(plot_df)

log_breaks = c(20000, 30000, 40000, 50000, 70000, 80000, 100000, 150000, 200000, 500000, 1000000)

random_forest_plot <- ggplot(plot_df, aes(y = model.output, x = grounded.truth, group=grounded.truth)) + 
geom_boxplot(fill = "#CCCCCC")+
#geom_boxplot() + 
geom_jitter(pch=21, fill = "#77EE11", color = "#7788EE", width = .3, size = 0.6) +
#scale_y_log10(breaks = log_breaks, labels=log_breaks) + 
#scale_x_log10(breaks = log_breaks, labels=log_breaks) + 
ggtitle("OkCupid age model") +
xlab("actual age") + 
ylab("inferred age") +
geom_smooth()



```

We can see that while the numerical accuracy of the age predictions is not high, it does accurately predict an age trend, capable of distinguishing something like an age-group with reasonable accuracy. 

```{r}
random_forest_plot %>% print
```

This means:  
* Your income, job, education, drinking habits, and religious affiliation, when taken together, provide information about your age.  

# Summary and Conclusions  
In the age of heavy analytics and shared data, seemingly ancillary data, when combined with other data, can reveal information about us we had assumed was private.  
We can see from the above it not just a single piece of information provides a lynch-pin in protecting our privacy, but rather an ensemble.  

If we assume our income, for example, is private, but reveal our job, education, and age (as I have done in consumer surveys), perhaps it is as though we have lost our privacy all the same (whether or not the model is actually built). 

What protections do we have? While worthy of another blog, I'll just list some brainstorm ideas here in case that takes a while. Note these are still to be evaluated.  

Possible steps to control digital-privacy:  
* Use dummy data. I noticed that the predictive models were thrown off when I included both the lower and highest income levels. It is imaginable that this data was less than reliable. Dummy data skews models.   
* Understand what data are being collected and what can be inferred from it. That is easier said than done, but perhaps we could create a kind of "dashboard" for folks. (buying diapers and baby stroller ->> baby, for example?) since above we have some evidence of this "robustness" independent of the model. 
* Enable consumers to protect what can be modeled about them thru extensions of privacy laws. This is obviously problematic from an enforcement perspective. 
* others???    


   
   

  





