---
layout: post
title: "Sentiment Analysis of the Oct 10 2016 Presidential Debate"
modified: 
categories: blog
excerpt:
tags: [R, sentiment, debate, NLP, animation]
image:
feature:
date: 2016-10-12T08:08:50-04:00
---

### SUMMARY   
What can we learn about a debate and it's outcome from a "sentiment" analysis? This analysis focuses on the sentiment trend of the second of three 2016 Presidential Election Debates between Hillary Clinton and Donald Trump. 

### DATA SOURCES AND METHODS   
The text of the debate are downloaded from the [UCSB Presidency Project](http://www.presidency.ucsb.edu/debates.php). Transcripts were cut from the web page, pasted into Apple Pages, and, without editing, stored as unformatted .txt files.   

The .txt files are made tidy by a separate `.R` program which cleans the text by removing punctuation and annotation and then categorizes by speaker in a tidy format, with the text being the data and both X and speaker names are variables.  
In practice this reduction required substantial fine tuning, so the program is stored as part of the analysis. The tidy data are stored as a .csv file which contains three columns:    
1. An index demarcating each text fragement.  
2. The speaker's name.  
3. The cleaned text.    

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


<figure>
<a href="/images/blog/blog_pres_debate.gif"><img src="/images/blog/blog_pres_debate.gif" alt="image"></a>
<figcaption><a href="/images/blog/blog_pres_debate.gif" title="Debate Sentiment Gif">Debate Sentiment Gif</a>.</figcaption>
</figure>



### End Notes 

The gif animation offers some insight into the flow of the debate. Points of major conflict, such as the discussion about ISIS and Assad, are clearly highlighted by the analysis.   

Some strengths:  
* Gif animation provides a visual way to study teh ebb adn flow of the debate. 
* It is visually appealing.  

Some weaknesses:
* The speed of the gif is very hard to adjust. Too fast and it is hard to follow. Too slow and it is frustrating. A manually adjustable rate might be better.
* Annotation is very hard to automate. Key words do not convey enough meaning. Full text is too much data. Need to think about this.  




