---
layout: post
title: "AI World 2016: Notes and Thoughts"
excerpt: "My observations from AI World 2016"
categories: articles
tags: [AI, AI World, Big Data, CrownFlower, Kogentix, artificial intelligence, bots]
image:
  feature: 
  credit: 
  creditlink: 
comments: true
share: true
---

# Summary
I just attended AI World 2016, a truly state of the art conference on teh developing world of AI solutions.  

The conference covered a wide range of topics, from Bots, to Machine Intelligence, to Big Data-driven solutions. AI is trul the new interface, and the concept flow below, expanding on the simple Machine Learning "Training" and "Prediction" flows, highlight the new kinds of workloads and software models that need to be understood.

![center](/figures/2016-11-10-AIWorldNotes/concept_flow.png) 


# Bots

What\'s astounding is that capablities behind these tools are now within reach of the "regular" enterprise. In fact, if you're not already deploying some form of AI, you're behind. Intelligent bots reviewed in a special conference session showed tremedous progress. You can get your own demo account at [API.AI](https://api.ai/) and play around. The design of the programming interface follows the natural flow:

![center](/figures/2016-11-10-AIWorldNotes/concept_flow.png) 

I recommend you sign up for a demo account and then head straight over to training. With just a few keystrokes you'll have your own AI bot (though meaningful training takes much more).


# "Big Data" as an enabler of AI

Kogentix co-founder Boyd Davis emphasized the need for "data collection at scale". 

Moore's law has made computing resources available at scale and at low coast. Open source software, such as  Linux, has made software available and affordable. And hardware initiatives, like the [Open Compute Project](http://www.opencompute.org/) have improved platform performance and reduced costs. 


AI relies on Machine learning to train algorithms for data reduction and predictions. "This call may be recorded" on a help line likely means more data for an AI to train itself in how to respond to customer needs. The cosst of this analysis was We can leverage the massive parallel compute capability of Apache Hadoop, storage substrates, Spark for analytics, and SQL interfaces. The role of Kogentix and Cloudera big data solutions are to knit that all together into an open and accessible platform for AI.


# Business ROI

At today's cost, AI is more accessible than ever. The current threshold, within wide margins, is that if you have 10 people doing 30% repetitive work, the job is a candidate for AI. Prototypes can be developed quickly and tested, but the real challenge is operationalizing. The greater diversity of data means more exposure of untested corner cases, for example. The compatition between AI solutions is not is speed of deployment. For AI to be believable. For AI not to lose or irritate customers, the competition needs to be on the quality of the solution.

# Success Depends on Data, ML, and HITL

As [Crowdflower](https://www.crowdflower.com/) CEO Robin Bordoli highlighted, you need to pay attention to the full development cycle to deliver the highest quality AI solution. This starts with the training data. As I've emphasized in some of my blogs, the quality of training data plays a strong role in model accuracy and resiliency. The next step is carefully chosen ML algorithms to do correct classification. And the final step, also essential, is what he called "Human in the Loop" (HITL) especially as alorithms are first brought on line, having people both checking results, and being available for seemless escalation of customer concerns in case the algorithm can't decipher the proper intent or concepts, is essential. 

I took the liberty of illustrating these concepts - similar to the famous data science Venn diagram - with my interepretations as below (I'd be interested in your comments/feedback of your impression of this). 

![center](/figures/2016-11-10-AIWorldNotes/AI_Venn.png) 


# AI for everyone
 
While big iron, fancy algorithms, and slick implementatations of AI seem to attract most of the press, there is a huge market at lower price points for AI tools. This is an example of the "train in the cloud, model on the device" approach advocated by most of the attendees. This has been enabled by the growin gcapability of the computing chips in all our devices. 

A cool example of this "AI for everyone" was the robot ATHENA. It's an AI assistant based on a hacked Roomba for mobility, and a table computer for human interface. It's capable of simple but important autonomous tasks, such as guiding a person form one point to another in an office environment. When I saw this demonstrated, I immediately imagined the utility in a doctor's office. As things currently stand, a nurse generally guides me from the front office to an exam room, where the patient degowns, etc while the nurse waits patiently outside. Guiding someone around an office is a low value task that, in principle, ties-up the nursing staff from doing more "high value" activities such as spending time with patients. This is exactly what AI should do - free us up for more important work. 

Here's a short video clip for your entertainment...

<a href="http://www.youtube.com/watch?v=HCsNnW5etBY
" target="_blank"><img src="http://img.youtube.com/vi/HCsNnW5etBY/0.jpg" 
alt="IMAGE ALT TEXT HERE" width="240" height="180" border="10" /></a>

# Summary

Many of the conferees spoke of the AI-desert, the period of time starting in the 1970's, when the promises of AI made mathematical-sense, but could not be supported from a computational perspective in practice. 

No longer. AI is exploding. The cloud computing revolution, with ready access to powerful handsets than can support local prediction computation, and cloud data centers, which can support the massive computations necessary to develop and fit models to huge amounts of data, have made the difference. It really is an "AI-first" world.





