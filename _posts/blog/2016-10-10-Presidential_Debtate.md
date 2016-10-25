---
layout: post
title: "Sentiment Analysis of the Oct 10 2016 Presidential Debate"
modified: 
categories: blog
excerpt:
tags: [debate]
image:
feature:
date: 2016-10-12T08:08:50-04:00
---

### SUMMARY   
What can we learn about a debate and it's outcome from a "sentiment" analysis? This analysis focuses on the sentiment trend of the second of three 2016 Presidential Election Debates between Hillary Clinton and Donald Trump. 

### DATA SOURCES AND METHODS   
The text of the debate are downloaded from the [UCSB Presidency Project](http://www.presidency.ucsb.edu/debates.php). Transcripts were cut from the web page, pasted into Apple Pages, and, without editing, stored as unformatted .txt files.   

The .txt files are made tidy by a separate __.R__ program which cleans the text by removing punctuation and annotation and then categorizes by speaker. In practice this reduction required substantial fine tuning, so the program is stored as part of the analysis. The tidy data are stored as a .csv file which contains three columns:
1. An index demarcating each text fragement.  
2. The speaker's name.  
3. The actual text.  


| X   | name       | text |
|:----|:----------:|--------:|
| 1   | RADDATZ    | ladies and gentlemen the republican nominee... |
| 2   | COOPER     | thank you very much for being here we re going to begin with a question... |
| 3   | AUDIENCE   | thank you and good evening the last debate could have been rated as mature... |
| 5   | CLINTON    | well thank you are you a teacher yes i think that that s a very good question... |
|=====
| 447   | RADDATZ  | please tune in on october th for the final presidential debate   |
{: .table}

          
1 1  RADDATZ    ladies and gentlemen the republican nominee...
2 2   COOPER    thank you very much for being here we re going to begin with a question...
3 3 AUDIENCE    thank you and good evening the last debate could have been rated as mature...
4 4  CLINTON    well thank you are you a teacher yes i think that that s a very good question...
5 5  CLINTON    and i think it is very important for us to make clear to our children ...




The analysis follows methods hightlighted in recent posts by [David Robinson](http://varianceexplained.org/r/trump-tweets/) and [Julia Silge](http://juliasilge.com/blog/Life-Changing-Magic/).

The methods are similar to a [previous post on the VP debates](http://rpubs.com/ww44ss/vp_debate), so I have suppressed most of the code from this printout.


### ANALYSIS   

I use the following packages for this analysis.

{% highlight bash %}
library(dplyr)  
library(animation)  
library(ggplot2)  
library(tidytext)
{% endhighlight %}

To read files I just search the directory

{% highlight bash %}

directory <- "/Users/winstonsaunders/Documents/oct_2016_pres_debate/"
list_of_files <- list.files(directory)

file_of_data <- list_of_files[grepl("_tidy", list_of_files)]
debate_text <- read.csv(paste0(directory, file_of_data), stringsAsFactors = FALSE) %>% as_data_frame
{% endhighlight %}

`
