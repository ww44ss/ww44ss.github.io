---
layout: post
title: "Six Challenges for AI?"
excerpt: "AI poised for explosive growth"
categories: articles
tags: [AI, Google, Five AI Challenges, Aritifical Intelligence, Security, Sixth Challenge]
image:
  feature: 
  credit: 
  creditlink: 
comments: true
share: true
---

# Summary
_I'm a wonk._ I know it because I have, taped to the reading lamp in my study since early last summer, a copy of Google's Five AI Challenges. I look at it almost daily and it's really influenced my thinking about the very small scale AI solutions I'm developing.

![center](/figures/2016-11-01-Six_AI_Challenges/2016-study.jpg) 

However, as I was reflecting, I came up with a possible sixth challenge that I'd like to propose, and get your feedback.


# Five Artificial Intelligence Challenges

Last June, Google published a set of [Five AI Challenges](http://www.dailymail.co.uk/sciencetech/article-3654714/Forget-killer-robots-Google-identifies-five-mundane-challenges-facing-artificial-intelligence-including-annoying.html) that received a fair amount of media attention, but much less discussion that I think they warranted. In summary, the Challenges are:

* __Avoiding Negative Side Effects:__ How can we ensure that an AI system will not disturb its environment in negative ways while pursuing its goals, e.g. a cleaning robot knocking over a vase because it can clean faster by doing so?

* __Avoiding Reward Hacking:__ How can we avoid gaming of the reward function? For example, we don't want this cleaning robot simply covering over messes with materials it can't see through.

* __Scalable Oversight:__ How can we efficiently ensure that a given AI system respects aspects of the objective that are too expensive to be frequently evaluated during training? For example, if an AI system gets human feedback as it performs a task, it needs to use that feedback efficiently because asking too often would be annoying.

* __Safe Exploration:__ How do we ensure that an AI system doesn't make exploratory moves with very negative repercussions? For example, maybe a cleaning robot should experiment with mopping strategies, but clearly it shouldn't try putting a wet mop in an electrical outlet.

* __Robustness to Distributional Shift:__ How do we ensure that an AI system recognizes, and behaves robustly, when it's in an environment very different from its training environment? For example, heuristics learned for a factory workfloor may not be safe enough for an office.


After several weeks of reflection, I've come to respect these as a well thought-out and, in a real sense, a profound list of challenges. When you start thinking of learning architectures, it's hard nto to confront these problems directly. 

### For instance:  
__"Robustness..."__ I've been toying with a #fakenews detection algorithm. Nothing too sophisticated, but essentially relying on scraped headlines and training on feeds from trusted and fringe news sources on the left and right of the political spectrum. "Can an algorithm be trained to detect inconsistent or unlikely news stories?" The challenge is that fake news stories might be made up shifting the baseline dramitically from one topic to another. 

__"Safe exploration..."__ building a bot to predict and reinforce safe behaviors based on reinforcement data has a hard time exploring safe boundaries (where negative consequences can occur) without losing credibility or incurring high "false positive" costs. 

Nevertheless, the question I kept asking myself was, "what other challenges do I face and are they simply generalizations of the above (most of them are) or are they distinct or important enough to warrant separate consideration?"

# A Sixth Challenge...

A problem that started to recur in writing some of my AI code, code that has an autonomous reinforcement or learning component, is how to protect it from maliciously induced behaviors? For instance, if I have a robot that is supposed to recognize and avoid obstacles in it's path, can one train the robot to take a less than optimal path by repeatedly modifying the training route?

This has been done before. For instance Richard Feynam, in the well known book [Surely You're Joking](https://www.amazon.com/Surely-Feynman-Adventures-Curious-Character/dp/0393316041) states that he retrained a colony of ants to to take a much longer path than necessary to reach a food source. 

While these are benign, you can imagine cases where safety or costs mist be associated with less than ideal beahvior, to teh detriment of the owner but the benefit of the hacker. This presents the AI architect with a daunting challenge, pitting her against an unknown adversary with unknown intents and methods. Thus, perhaps we have a sixth challenge:

* __Able to detect and defeat purposeful subversion:__ How does an AI system recognize and react when it is being "attacked" to subvert it's intended purpose? For instance, suboptimizing the choice of path of a robot to slow it's response or diminish it's capacity. 

The idea of attacking machine learning algorithms isn't new. For instance [Anthony D. Joseph](https://www2.eecs.berkeley.edu/Faculty/Homepages/joseph.html) has published several interesting papers on security of machine learning algorithms and making them robust to attacks.

The specific problem I am working on is writing algorithms to detect #fakenews. Since what qualifies as fake news will change, the idea it to get a nachine to recognize it. The question is, can the algornithm be defeated. (Granted, my algorithm is not likely to draw much attention, but as Google, facebook, and other hyperscale media companies deploy these, they might be subject to organized attack.)

I'd like your thoughts? Is this an important addition? Or is it contained in the above assumptions? Comments welcome. 













