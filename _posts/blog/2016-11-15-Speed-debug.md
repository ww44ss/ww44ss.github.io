---
layout: post
title: "Speed-up Debug of Piped R Instructions"
modified: 
categories: blog
excerpt: "substantially reduce editing hassles"
tags: [R, package, yo, debug, dplyr]
image:
feature:
date: 2016-11-15T08:08:50-04:00
---

# SUMMARY   
Debugging piped instructions in R can be a nuissance. Using the package `yo`, available on github using devtools, can save you a ton of time!

# INTRODUCTION   

I love the tidyverse additions to the R language. They make R, already a powerful analysis and visualization tool, into a powerful and modern analytics programming language.  

One of the most useful features of the tidyverse is the piped instruction capability. In "pseudo-code":

{% highlight r %}
output_data <- input_data %>% 
        function1(instructions1) %>% 
        function2(instructions2) %>% 
        function3(instructions3) %>%
        function4(instructions4)
{% endhighlight %}

With some thought, it can read as a nearly-natural sentence, making the program more interpretable by that most-important-other-person-to-read-your-code, that is, future-you. And creating easily interpretable code can reduce the amount of commenting you need to add to your programs.

# WHAT'S THE PROBLEM?

The problem with piped commands occurs, at least for me, when I've already got the chain working once, and then something changes (either in the input data or the desired outcome) that causes me to go back and modify either the functions or the instructions. For instance, in the above example, to check the output of `function1` I'd need to add comments to each function line below it, and also modify the pipe at the end of the line.

It's a lot of nuissance doing that typing. Or at least it's more typing than you need to do and more than I care to do. 

# LET'S LOOK AT A REAL EXAMPLE

To run this example you'll need the following libraries

{% highlight r %}
library(stringr)
library(dplyr)

if( !"yo" %in% installed.packages()[,1] ) {
    library(devtools)
    devtools::install_github("ww44ss/yo")
    }

library(yo)
{% endhighlight %}

### real raw data

Here's some html data from a public-facing web-page from which I wish to extract relevant snowfall-data. (Sorry, I know this is a lot of _stuff_ but it's faster to blog, and I'd argue also better, just to copy from a real thing I'm working on rather than making up some contrived example.) I've put the data in a form that can be copied directly into am R-program.

{% highlight r %}
snowfallraw <- 
c("<h1 class=\"section-header\">Snow Conditions</h1>", 
"", 
"<div class=\"condition-intro\">", 
"    <h2>Mid Mountain Snowfall</h2>", 
"    <p>Snowfall measured at 7,300' near the top of the Sunrise Express lift</p>", 
"</div>", 
"", 
"<div class=\"current-sections conditions\">", 
"    <div class=\"section-block\">", 
"        <div class=\"current-section condition first\">", 
"            <div class=\"item\">", 
"                <div class=\"key\">0\"</div>", 
"                <div class=\"value\">since 3pm yesterday</div>", 
"            </div>", "        </div>", 
"        <div class=\"current-section condition\">", 
"            <div class=\"item\">", 
"                <div class=\"key\">0\"</div>", 
"                <div class=\"value\">24 Hour</div>", 
"            </div>", 
"        </div>", 
"        <div class=\"current-section condition\">", 
"            <div class=\"item\">", 
"                <div class=\"key\">0\"</div>", 
"                <div class=\"value\">3 Day</div>", 
"            </div>", 
"        </div>", 
"        <div class=\"current-section condition\">", 
"            <div class=\"item\">", 
"                <div class=\"key\">0\"</div>", 
"                <div class=\"value\">7 Day</div>", 
"            </div>", 
"        </div>", 
"    </div>", 
"    <div class=\"section-block full first\">", 
"        <div class=\"current-section condition\">", 
"            <div class=\"item\">", 
"                <div class=\"key\">34\"</div>", 
"                <div class=\"value\">snowfall since Oct 1</div>", 
"            </div>", "        </div>", "    </div>", 
"    <div class=\"section-block full\">", 
"        <div class=\"current-section condition\">", 
"            <div class=\"item\">", 
"                <div class=\"key\">15\"</div>", 
"                <div class=\"value\">Snow Depth</div>", 
"            </div>", "        </div>", "    </div>", "</div>", 
"", "", "<div class=\"condition-intro\">", 
"    <h2>West Village Base Snow Fall</h2>", 
"    <p>Snowfall measured at 6,300' near the bottom of the Sunshine Accelerator lift</p>", 
"</div>", "", "<div class=\"current-sections conditions\">", 
"    <div class=\"section-block\">", 
"        <div class=\"current-section condition first\">", 
"            <div class=\"item\">", 
"                <div class=\"key\">0\"</div>", 
"                <div class=\"value\">since 3pm yesterday</div>", 
"            </div>", "        </div>", 
"        <div class=\"current-section condition\">", 
"            <div class=\"item\">", 
"                <div class=\"key\">0\"</div>", 
"                <div class=\"value\">24 Hour</div>", 
"            </div>", 
"        </div>", "        <div class=\"current-section condition\">", 
"            <div class=\"item\">", 
"                <div class=\"key\">0\"</div>", 
"                <div class=\"value\">3 Day</div>", 
"            </div>", 
"        </div>", "        <div class=\"current-section condition\">", 
"            <div class=\"item\">", 
"                <div class=\"key\">0\"</div>", 
"                <div class=\"value\">7 Day</div>", 
"            </div>", 
"        </div>", "    </div>", 
"    <div class=\"section-block full first\">", 
"        <div class=\"current-section condition\">", 
"            <div class=\"item\">", 
"                <div class=\"key\">34\"</div>", 
"                <div class=\"value\">snowfall since Oct 1</div>", 
"            </div>", "        </div>", "    </div>", 
"    <div class=\"section-block full\">", 
"        <div class=\"current-section condition\">", 
"            <div class=\"item\">", 
"                <div class=\"key\">0\"</div>", 
"                <div class=\"value\">Snow Depth</div>", 
"            </div>", "        </div>", "    </div>", "</div>", 
"") 

snowfallraw[1:6]

{% endhighlight %}



The first few lines of the output should look like:

{% highlight r %}
[1] "<h1 class=\"section-header\">Snow Conditions</h1>"                                      
[2] ""                                                                                       
[3] "<div class=\"condition-intro\">"                                                        
[4] "    <h2>Mid Mountain Snowfall</h2>"                                                     
[5] "    <p>Snowfall measured at 7,300' near the top of the Sunrise Express lift</p>"        
[6] "</div>" 
{% endhighlight %}

### extracting relevant info

The information we seek is on only a few of these lines, which we can extract by selecting appropriate headers, like this:

{% highlight r %}
## get snowfall related info by first defining string matches for relevant lines
match.strings <- c(
    "Snow Conditions",
    "<h2>",
    "<p>",
    "<div class=\"key\">",
    "<div class=\"value\">")

matches <- 
    grep(paste(match.strings,collapse="|"), snowfallraw, value=TRUE) %>% 
    str_trim %>%
    yo
{% endhighlight %}

resulting in the following lines of text (abbreviated).

{% highlight r %}
[1] "<h1 class=\"section-header\">Snow Conditions</h1>"                                  
[2] "<h2>Mid Mountain Snowfall</h2>"                                                     
[3] "<p>Snowfall measured at 7,300' near the top of the Sunrise Express lift</p>"        
[4] "<div class=\"key\">0\"</div>"                                                       
[5] "<div class=\"value\">since 3pm yesterday</div>"                                     
[6] "<div class=\"key\">0\"</div>"                                                       
[7] "<div class=\"value\">24 Hour</div>"                                                 
[8] "<div class=\"key\">0\"</div>"                                                       
[9] "<div class=\"value\">3 Day</div>"                                                   
[10] "<div class=\"key\">0\"</div>"                                                       
[11] "<div class=\"value\">7 Day</div>" 
...
{% endhighlight %}

### refining the data

Now we just need to pull out the data using regular expressions. 

{% highlight r %}
matches <- 
    matches %>% 
    str_replace("^.*?>", "") %>%
    str_replace("</.{1,3}>$", "") %>%
    str_replace("\"", "") %>%
    yo
{% endhighlight %}

Which results in the following contents for `matches` (abbreviated).

{% highlight r %}
[1] "Snow Conditions"                                                             
[2] "Mid Mountain Snowfall"                                                       
[3] "Snowfall measured at 7,300' near the top of the Sunrise Express lift"        
[4] "0"                                                                           
[5] "since 3pm yesterday"                                                         
[6] "0"                                                                           
[7] "24 Hour"                                                                     
[8] "0"                                                                           
[9] "3 Day"                                                                       
[10] "0"                                                                           
[11] "7 Day"                                                                       
... 
{% endhighlight %}

__Success!!__ So things will work great.. until they don't.'

# USING yo TO SPEED UP DEBUG

Let's assume at some point I discover a problem with the last code chunk. (Below I have made a small change to the code chunk, but it could be a change to the input data giving an error in the same spot).

{% highlight r %}
matches <- 
    matches %>% 
    str_replace("^.*?>", "") %>%
    str_replace("</.{1,2}>$", "") %>%
    str_replace("\"", "") %>%
    yo
{% endhighlight %}

Now the data are suddenly incorrect (the text at the end of some lines is not being replaced, as for example on line [4].  

{% highlight r %}
[1] "Snow Conditions"                                                             
[2] "Mid Mountain Snowfall"                                                       
[3] "Snowfall measured at 7,300' near the top of the Sunrise Express lift"        
[4] "0</div>"                                                                     
[5] "since 3pm yesterday</div>"                                                   
...    
{% endhighlight %}

So how does `yo` help to debug this? Using yo we can simply sequence backward in the piped-chain to find the problem by commenting out each line... (note in this case I have also commented out the first line, this is an optional step, but as it sends the output to the terminal for easier examination, it's also a useful trick.)

{% highlight r %}
#matches <- 
    matches %>% 
   #str_replace("^.*?>", "") %>%
   #str_replace("</.{1,2}>$", "") %>%
   #str_replace("\"", "") %>% 
    yo
{% endhighlight %}

gives the raw data. And below we can quickly step thru the lines

{% highlight r %}
#matches <- 
    matches %>% 
    str_replace("^.*?>", "") %>%
   #str_replace("</.{1,2}>$", "") %>%
    yo
{% endhighlight %}

on this iteration we discover the problem is with the third line. 

{% highlight r %}
#matches <- 
    matches %>% 
    str_replace("^.*?>", "") %>%
    str_replace("</.{1,2}>$", "") %>%
    yo
{% endhighlight %}

And beyond this point it is easy to correct the regular expression and prove that things are working again as they should. 

{% highlight r %}
#matches <- 
    matches %>% 
    str_replace("^.*?>", "") %>%
    str_replace("</.{1,3}>$", "") %>%
    yo
{% endhighlight %}

The final step is to remove all the comments and you're back and running. 

# END NOTE

So there you have it. yo is available from the link above. It works especially well with tibbles.  For a personal touch, I also added the same functionality for `ruhroh` and `doh` in the yo package. I'll let you figure out which TV-characters these emulate.
