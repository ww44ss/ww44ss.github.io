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




