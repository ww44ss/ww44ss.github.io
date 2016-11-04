---
layout: post
title: "Understanding Digital Privacy with OkCUpid"
modified: 
categories: blog
excerpt: "Using the OkCupid dataset to understand privacy in the digital era"
tags: [R, OkCupid, Privacy, Machine Learning]
image:
feature:
date: 2016-09-18T08:08:50-04:00
---

## Introduction

Privacy is a concern to everyone. But, in an era of machine-learning and advanced data analytics, how does one go about protecting it?   

Intuitively we're familiar with physical privacy. When getting dressed we might step back from an open door or even close and lock the door for more robust privacy. Through simple acts we control our physical privacy by limiting what others may see or learn about us. Privacy is our ability to control what others know, directly or by inference, about us.

We know how to protect our physical privacy. Protecting our digital privacy is another matter. Our intuition fails us when trying to decide what data in one area may compromise more sentivie information in another area; control is not as simple as just closing a door. 

With the now widespread use of predictive analytics, our personal information faces ever greater threats, both direct and indirect. Many cases have been popularized, such as the case of [Target knowing a girl was pregnant before her father did](http://www.forbes.com/sites/kashmirhill/2012/02/16/how-target-figured-out-a-teen-girl-was-pregnant-before-her-father-did/#791d11a134c6).

The purpose here is to help build that understanding. I use demographic data from the recently published [OkCupid package on CRAN](https://cran.r-project.org/web/packages/okcupiddata/index.html) to model specific "private" information about users. These models may contribute to what constitutes an "attack" on user privacy.  

It should be noted that for an "attack" to succeed it need not expose perfect information. The ability to infer even a reasonable guess might in many cases may constitute an unacceptable loss of privacy for someone. 

## Problem Statement

Personal income and age are among our most private information. But how private is it, really? Can a "trained" machine, not actually knowing my age, for example, but knowing other things about me, approximatey infer my age (and therefore pose a threat to my privacy). If so, what specific data plays a role in the accuracy of this inference? How might I control it? 

## Analysis

We start with these R packages.

{% highlight r %}
library(dplyr)
library(ggplot2)
library(RColorBrewer)
library(gridExtra)

library(okcupiddata)
library(randomForest)
library(gbm)
{% endhighlight %}

The OkCupid data are loaded with the package 'okcupid' and the data are contrained in a dataframe 'profiles'. To start the analysis we create a cleaned data set and convert it to a tibble.

{% highlight r %}
cleaned <- profiles %>% select(-essay0, -last_online, -sign)
cleaned <- cleaned %>% filter(!is.na(income))
cleaned <- cleaned %>% filter(!is.na(age))

cleaned <- cleaned %>% as_data_frame
{% endhighlight %}

Some deeper cleaning is needed to make the data intelligible. This is a bit brute force-ish but, since some handcrafting of the data was necessary I found it easier to call out lines separately in order to modify them at will. 

{% highlight r %}

cleaned$body_type <- cleaned$body_type  %>% as.factor

cleaned$diet <- cleaned$diet %>% as.factor

cleaned$drinks <- cleaned$drinks %>% as.factor

## simplify education level to single descriptor
cleaned <- cleaned %>% mutate(education_simple = gsub(' [A-z /\\-\\.]*', '', education)) 
cleaned$education_simple <- cleaned$education_simple %>% as.character %>% as.factor
cleaned$education <- cleaned$education %>% as.factor


cleaned$ethnicity <- gsub(',( [a-z_]*)', '', cleaned$ethnicity) %>% as.factor()

## compute a log income
cleaned$income <- cleaned$income %>% as.character %>% as.numeric
cleaned <- cleaned %>% mutate(log_income = log10(income))

cleaned$location <- cleaned$location %>% as.factor
cleaned$height <- cleaned$height %>% as.factor
cleaned$job <- cleaned$job %>% as.factor
cleaned$offspring <- cleaned$offspring %>% as.factor
cleaned$orientation <- cleaned$orientation %>% as.factor
cleaned$pets <- cleaned$pets %>% as.factor

## simplify religious affiliation
cleaned <- cleaned %>% mutate(religious_affil = gsub(' [A-z ]*', '', religion))
cleaned$religious_affil <- cleaned$religious_affil %>% as.factor
cleaned$religion <- cleaned$religion %>% as.factor

cleaned$sex <- cleaned$sex %>% as.factor
cleaned$smokes <- cleaned$smokes %>% as.factor
cleaned$status <- cleaned$status %>% as.factor
{% endhighlight %}


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

Here is an example of the cleaned and tidy data

| X   | name       | text    |
|:---:|-----------:|:--------|
| 1   | RADDATZ    | ladies and gentlemen the republican nominee... |
| 2   | COOPER     | thank you very much for being here we re going to begin with a question... |
| 3   | AUDIENCE   | thank you and good evening the last debate could have been rated as mature... |
| 4   | CLINTON    | well thank you are you a teacher yes i think that that s a very good question... |
| ========  |   |  |
|(...)  |   | |
| 447   | RADDATZ  | please tune in on october th for the final presidential debate   |
{: .table}

          

The methods used here closely follow those hightlighted in recent posts by [David Robinson](http://varianceexplained.org/r/trump-tweets/) and [Julia Silge](http://juliasilge.com/blog/Life-Changing-Magic/) who looked at text sentiment analysis in both literary and political contexts as well as those used in a [previous post on the VP debates](http://rpubs.com/ww44ss/vp_debate).


### ANALYSIS  

#### Getting Started

I use the following packages for this analysis.

{% highlight r %}
library(dplyr)  
library(animation)  
library(ggplot2)  
library(tidytext)
{% endhighlight %}


Before getting started I also want to define the function 'yo'

{% highlight r %}
yo <- function(x){x}
{% endhighlight %}

`yo` is a trivially idempotent function used to punctuate a string of piped commands. It greatly simplifies debugging long chains of pipes (since individual lines can simply be commented-out rather than modified). When speaking, `yo` is enunciated as Aaron Paul's character in _Breaking Bad_ would have done at the end of a sentence, yo.   


#### computing dictionaries and sentiment

To read files I just search the directory for appropriate files and then filter the list.

{% highlight r %}
directory <- "/Users/Me/Documents/oct_2016_pres_debate/"
list_of_files <- list.files(directory)

## filter for Second debate and the _tidy version of the text
data_file <- list_of_files[grepl("_tidy", list_of_files) & 
                            grepl("Second", list_of_files)]

debate_text <- data_file %>% paste0(directory,.) %>% 
    read.csv(stringsAsFactors = FALSE) %>% 
    as_data_frame %>%
    yo
{% endhighlight %}

We now begin processing by taking the text, unnesting the sentences, and removing stop words using the snowball lexicon.   

{% highlight r %}
## create list of stop words
list_of_stop_words <- stop_words %>%
    filter(lexicon == "snowball") %>% 
    select(word) %>% 
    yo
## unnest the text words and remove stop words
words_from_the_debate <- debate_text %>%
    unnest_tokens(word, text) %>%
    filter(!word %in% list_of_stop_words) %>% 
    yo
{% endhighlight %}


We create a "sentiment dictionary" from the information stored in the  package and use a  to associate words with the sentiment values.  

{% highlight r %}

word_sentiment_dict <- sentiments %>%
    filter(lexicon == "AFINN") %>%
    select(word, sentiment = score) %>%
    yo

## assign sentiment with left_join 
debate_words_sentiments <-words_from_the_debate %>%
    left_join(word_sentiment_dict, by = "word") %>%
    mutate(sentiment = ifelse(is.na(sentiment), 0, sentiment)) %>%
    mutate(sentiment = as.numeric(sentiment)) %>%
    yo

## create list of non-zero sentiment words
nonzero_sentiment_debate_words <- debate_words_sentiments %>% filter(sentiment != 0)
{% endhighlight %}


#### computing sentiment trends

To look at the trend of the sentiment, we can create an exponentially damped cummulative sum function to apply to the data. The idea is that words have immediate punch when spoken, but their impact on overall sentiment wanes as time and words pass. The choice of damping factor, which is essentially arbitrary from an analytical standpoint, is made to capture "longer term trends" in the speech patterns. For most computations a factor around 0.02 worked well. 


{% highlight r %}
decay_sum <- function(x, decay_rate = 0.1421041) {
    ## EXPONENTIALLY DAMPED CUMMULATIVE SUM
    ## input:   x (a vector of length >1)
    ##          decay_rate (exponential damping factor)
    ## output:  decay_sum (a vector with the cummulatve sum)

    ## create output vector
    if (length(x) > 1) decay_sum <- 1.*(1:length(x))

    ## initialize
    decay_sum[1] <- x[1]*1.

    ## compute the sum using a dreaded loop.
    if (length(x) > 1) {
        for (i in 2:length(x)){
            decay_sum[i] <- x[i] + exp(-1.*decay_rate) * decay_sum[i-1]
            }
        }

    return(decay_sum)
}
{% endhighlight %}

With this awkward function defined, we can now proceed to use the machinery of R for a simple calculation

{% highlight r %}
## compute sentiment of debate responses by regrouping and compute means and cumsums
debate_sentiment <- debate_words_sentiments %>%
    group_by(X, name) %>%
    summarize(sentiment = sum(sentiment)) %>%
    group_by(name) %>%
    mutate(cumm_sent = decay_sum(sentiment, decay_rate = 0.02)) %>%
    yo
{% endhighlight %}

The final step is to pull it all together into a plotable data frame

{% highlight r %}
## create data_frame for plotting. Since some X have no entry, need to fix those
plot_df <- debate_sentiment %>% left_join(debate_text, by = c("X", "name")) %>%
    select("X" = X, "name" = name, sentiment, cumm_sent, text) %>%
    group_by(name) %>%
    yo

## suppress mediator and audience questioner text
plot_df <- plot_df %>% filter(name == "TRUMP" | name == "CLINTON")
{% endhighlight %}

#### creating debate text annotation 


The first part of annotating is selecting the specific text to use. For the fist choice, I simply selected for high sentiment lines.

{% highlight r %}
## go back and add words for strong sentiment
gif_steps <- c(plot_df$X[plot_df$sentiment < -8 | plot_df$sentiment > 8 ], plot_df$X[nrow(plot_df)])
## get rid of first line since it is usually parasitic
gif_steps <- gif_steps[-1]

{% endhighlight %}

To make the animation effectively communicate the flow of the debate, I had to manually add annotated text. I couldn't find an easy way to automate this. Using just "high senitment" words, while interesting, didn't tell a narrative, and using the full text was too much to read in a .GIF format. So in this case I have paraphrased the text. 

This solution, while adequate here, has the problem that it doesn't scale (If I were to annotate each line it could, but that would take more time than I have). Here are the annotations. 


{% highlight r %}
annote = c("i have a very positive and optimistic view - the slogan of my campaign is stronger together",
"i want to do things - making our inner cities better for the african american citizens that are so great", 
"yes i m very embarrassed by it i hate it but it's locker room talk", "so this is who donald trump is - this is not who we are",
"all you have to do is take a look at wikileaks", "after a year long investigation there is no evidence that anyone hacked the server", 
"i want very much to save what works and is good about the affordable care act", 
"the affordable care act was meant to try to fill the gap", 
"there are children suffering in this catastrophic war", 
"i will tell you very strongly when bernie sanders said she had bad judgment", 
"one thing i'd do is get rid of carried interest", 
"i understand the tax code better than anybody that's ever run for president",  
"i don't like assad at all but assad is killing isis", 
"i have generals and admirals who endorsed me", 
"the greatest disaster trade deal in the history of the world", 
"tweeting happens to be a modern day form of communication", 
"justice scalia great judge died recently and we have a vacancy", 
"i respect his children his children are incredibly able and devoted", 
"i consider her statement about my children to be a very nice compliment", 
"she doesn't quit she doesn't give up i respect that")


ann_text <- data.frame(X = 220, sentiment = 20, lab = annote,
name = plot_df$name[plot_df$X %in% gif_steps])

{% endhighlight %}



#### computing the animation


The animation is computed using the `animation` package. There are a few gymnastics to get the colors of the titles to change with sentiment, but otherwise the plotting is straight-forward. Notice the addition of the vertical bar to the graph which shows the specific point in the debate. 

{% highlight r %}
saveGIF({
    for (j in 1:length(gif_steps)) {
        i <- gif_steps[j]

        ## color code summary text
        if (plot_df$sentiment[plot_df$X == i] < 0) {
        title_color = "darkred"
        } else {
        title_color = "darkblue"
    }

    title_h = 0

    print( 
        ## the ggplot stack
        ggplot(plot_df, aes(x = X, y = sentiment, fill = name)) +
        geom_bar(stat = 'identity', alpha = 1., width = 2) +
        geom_line(data = plot_df%>% filter(X <= i), 
            aes(x=X, y = 15*cumm_sent/max(abs(plot_df$cumm_sent))), 
            size = 3, color = "grey20", alpha = 0.5) +
        xlim(range(plot_df$X)) +
        ylim(range(plot_df$sentiment)) +
        xlab("index") +
        ggtitle(paste0(plot_df$name[plot_df$X == i], ": ", ann_text$lab[j])) +
        facet_grid(name~.) +
        theme(plot.title = element_text(size = 14, hjust = title_h, color = title_color, face="bold"), legend.position = "bottom") +
        geom_vline(xintercept = i, color = "grey50")

        ) 
    }
}, 
interval = 1.2, movie.name = paste0(directory,"sentiment_animation2.gif"), 
                ani.width = 700, ani.height = 450)

{% endhighlight %}

Here is the animation. 


<figure>
<a href="/images/blog/blog_pres_debate.gif"><img src="/images/blog/blog_pres_debate.gif" alt="image"></a>
</figure>



### End Notes   

The gif animation offers some insight into the flow of the debate. Points of major conflict, such as the discussion about ISIS and Assad, are clearly highlighted by the analysis.   

Some strengths:   
- Gif animation provides a visual way to study teh ebb adn flow of the debate.    
- It is visually appealing.     

Some weaknesses:   
- The speed of the gif is very hard to adjust. Too fast and it is hard to follow. Too slow and it is frustrating. A manually adjustable rate might be better.       
- Annotation is very hard to automate. Key words do not convey enough meaning. Full text is too much data. Need to think about this.     


   
   

  





