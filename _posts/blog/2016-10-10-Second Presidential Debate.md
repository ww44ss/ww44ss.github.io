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




