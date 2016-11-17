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

With some thought, it can read as a nearly-natural sentence, making the program more interpretable by that most important other person to read your programs - future-you. It reduces substnatially the amount of commenting you need to add. 

#WHAT'S THE PROBLEM?

The problem occurs, at least for me, when I've already got the chain working once, and then something changes (either in the input data or the desired outcome) that causes me to go back and modify either the functions or the instructions. To check the output of `function1` I need to add comments to each function line below it, and also to the pipe at the end of the line.

It's a lot of nuissance doing that typing. Or at least it's 'more typing than you need to do and more than I care to do. 

#LET'S LOOK AT A REAL EXAMPLE

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



Here's some html data from a public-facing web-page. I wish to extract relevant snowfall data. (Sorry, I know this is a lot of _stuff_ but it's faster to blog, and I'd argue better, just to copy from a real thing I'm working on rather than make up some contrived example.) I've put the data in a form that can be copied directly into a console.

{% highlight r %}
snowfallraw <- c("<h1 class=\"section-header\">Snow Conditions</h1>", 
"", "<div class=\"condition-intro\">", 
"    <h2>Mid Mountain Snowfall</h2>", 
"    <p>Snowfall measured at 7,300' near the top of the Sunrise Express lift</p>", 
"</div>", 
"", "<div class=\"current-sections conditions\">", 
"    <div class=\"section-block\">", 
"        <div class=\"current-section condition first\">", 
"            <div class=\"item\">", 
"                <div class=\"key\">0\"</div>", 
"                <div class=\"value\">since 3pm yesterday</div>", 
"            </div>", "        </div>", 
"        <div class=\"current-section condition\">", 
"            <div class=\"item\">", 
"                <div class=\"key\">0\"</div>", 
"                <div class=\"value\">24 Hour</div>", "            </div>", 
"        </div>", "        <div class=\"current-section condition\">", 
"            <div class=\"item\">", 
"                <div class=\"key\">0\"</div>", 
"                <div class=\"value\">3 Day</div>", "            </div>", 
"        </div>", "        <div class=\"current-section condition\">", 
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
"                <div class=\"value\">24 Hour</div>", "            </div>", 
"        </div>", "        <div class=\"current-section condition\">", 
"            <div class=\"item\">", 
"                <div class=\"key\">0\"</div>", 
"                <div class=\"value\">3 Day</div>", "            </div>", 
"        </div>", "        <div class=\"current-section condition\">", 
"            <div class=\"item\">", 
"                <div class=\"key\">0\"</div>", 
"                <div class=\"value\">7 Day</div>", "            </div>", 
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
snowfallraw
{% endhighlight %}


