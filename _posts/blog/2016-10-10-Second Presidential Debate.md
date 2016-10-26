---
layout: post
title: "Sentiment Analysis of the Oct 10 2016 Presidential Debate"
modified: 
categories: blog
excerpt:
tags: [R]
image:
feature:
date: 2016-10-12T08:08:50-04:00
---

### SUMMARY   
What can we learn about a debate and it's outcome from a "sentiment" analysis? This analysis focuses on the sentiment trend of the second of three 2016 Presidential Election Debates between Hillary Clinton and Donald Trump. 

### DATA SOURCES AND METHODS   
The text of the debate are downloaded from the [UCSB Presidency Project](http://www.presidency.ucsb.edu/debates.php). Transcripts were cut from the web page, pasted into Apple Pages, and, without editing, stored as unformatted .txt files.   

The .txt files are made tidy by a separate _.R_ program which cleans the text by removing punctuation and annotation and then categorizes by speaker in a tidy format, with the text being the data and both X and speaker names are variables.  
In practice this reduction required substantial fine tuning, so the program is stored as part of the analysis. The tidy data are stored as a .csv file which contains three columns:    
1. An index demarcating each text fragement.  
2. The speaker's name.  
3. The cleaned text.    

Here is an example of the data

| X   | name       | text    |
|:---:|-----------:|:--------|
| 1   | RADDATZ    | ladies and gentlemen the republican nominee... |
| 2   | COOPER     | thank you very much for being here we re going to begin with a question... |
| 3   | AUDIENCE   | thank you and good evening the last debate could have been rated as mature... |
| 4   | CLINTON    | well thank you are you a teacher yes i think that that s a very good question... |
| ========  |   |  |
| 447   | RADDATZ  | please tune in on october th for the final presidential debate   |
{: .table}

          
The methods used here closely follow those hightlighted in recent posts by [David Robinson](http://varianceexplained.org/r/trump-tweets/) and [Julia Silge](http://juliasilge.com/blog/Life-Changing-Magic/) who looked at text sentiment analysis in both literary and political contexts as well as those used in a [previous post on the VP debates](http://rpubs.com/ww44ss/vp_debate).

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

yo is a trivially idempotent function used to punctuate a string of piped commands. It greatly simplifies debugging long chains of pipes (since individual lines can simply be commented-out rather than modified). When speaking, yo is enunciated as Aaron Paul's character in _Breaking Bad_ would have done at the end of a sentence, yo.   


#### computing dictionaries and sentiment

To read files I just search the directory

{% highlight r %}
directory <- "/Users/IamIronMan/Documents/oct_2016_pres_debate/"
list_of_files <- list.files(directory)

## filter for Second debate and the _tidy version of the text
data_file <- list_of_files[grepl("_tidy", list_of_files) & grepl("Second", list_of_files)]

debate_text <- data_file %>% paste0(directory,.) %>% 
read.csv(stringsAsFactors = FALSE) %>% 
as_data_frame
{% endhighlight %}

We now begin processing by taking the text, unnesting the sentences, and removing stop words using the snowball lexicon.   

{% highlight r %}
## comment
list_of_stop_words <- stop_words %>%
filter(lexicon == "snowball") %>% 
select(word) %>% 
yo

words_from_the_debate <- debate_text %>%
unnest_tokens(word, text) %>%
filter(!word %in% list_of_stop_words) %>% 
yo
{% endhighlight %}


We create a "sentiment dictionary" from the information stored in the  package and use a  to assicate words with the sentiment values.  

{% highlight r %}

word_sentiment_dict <- sentiments %>%
filter(lexicon == "AFINN") %>%
select(word, sentiment = score) %>%
yo


debate_words_sentiments <-words_from_the_debate %>%
left_join(word_sentiment_dict, by = "word") %>%
mutate(sentiment = ifelse(is.na(sentiment), 0, sentiment)) %>%
mutate(sentiment = as.numeric(sentiment)) %>%
yo

nonzero_sentiment_debate_words <- debate_words_sentiments %>% filter(sentiment != 0)
{% endhighlight %}

the data_frame nonzero_sentiment_debate_words



