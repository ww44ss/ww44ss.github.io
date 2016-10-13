---
layout: post
title: "Sentiment Analysis of the Oct 10 2016 Presidential Debate"
modified: 
categories: blog
excerpt:
tags: []
image:
feature:
date: 2016-10-12T08:08:50-04:00
---

##SUMMARY
Can we learn anything about a debate and it's outcome from a "sentiment" analysis? I adapt methods recently hightlighted by David Robinson and Julia Silge to create a kind of ".gif movie" version of the sentiment analysis, with text of especially "strong" sentiment highlighted.

##DATA SOURCES AND METHODS
The text of the debate are downloaded from the [UCSB Presidency Project](http://www.presidency.ucsb.edu/debates.php). Transcripts were pasted into Apple Pages and stored as unformatted .txt files. No editing was done beyond that 

The .txt files are made tidy by a separate `.R` program which removes punctuation and annotation and then categorized by speaker. In practice this reduction required substantial fine tuning, so the program is stored as part of the analysis. The tidy data are stored as a .csv file which contains three columns:
1. An index demarcating each text fragement.  
2. The speaker's name.  
3. The actual text.  

The methods are similar to a [previous post on the VP debates](http://rpubs.com/ww44ss/vp_debate), so I have suppressed most of the code from this printout.


##ANALYSIS 

First, we'll use the following libraries:
`    library(dplyr)  
library(animation)  
library(ggplot2)  
library(tidytext)   
`
