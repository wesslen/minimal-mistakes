---
layout: single
excerpt: "How does party effect what topics senators discuss on Facebook?" 
title:  "Structural Topic Modeling to analyze NC State Senators' Facebook Posts"
categories: [social media]
tags: [visualization, Facebook, topic modeling, political science]
---

The recent election has ushered in a new era for politicians to use social media like Twitter and Facebook (see [this October NY Times article](http://www.nytimes.com/2015/10/06/us/politics/donald-trump-twitter-use-campaign-2016.html) or [this]() ).  Following my previous post ([see]), in this post I'll provide analysis to the public Facebook posts of North Carolina state senators. I'm going to use a cutting edge machine learning algorithm called Structural Topic Modeling (or STM for short) to measure, with statistical significance, the impact of party affiliation on the prevalence of topics discussed.

STM is a recent extension of the popular topic modeling framework, a family of unsupervised algorithms that have been championed by computer scientists over the last decade. Topic models work by identifying clusters of words that co-occur together that can be interpreted as topics. For example, here are two topics that the model identified with my interpretation (called a label).

For readers interested in the code and more details, I've provided a [GitHub site](https://github.com/wesslen/NCStateSenateFacebook) including two RMarkdown output (html) files ([Part 1](https://htmlpreview.github.io/?https://github.com/wesslen/NCStateSenateFacebook/blob/master/code/STM-ncsenate-facebook-part1.html) and [Part 2](https://rawgit.com/wesslen/NCStateSenateFacebook/master/code/STM-ncsenate-facebook-part2.html) )

!["HB2" and "Bathroom Safety" Topics from STM Analysis of NC State Senators' Facebook Posts](/images/STM-senate1.png)

One of the model's outputs are the words ranked by how likely they are for set of hidden topics. This is incredibly helpful as the topics are determined by the data, not through manually coding that are subject to high time and monetary costs, inconsistencies and biases, and overall more difficult to scale for "big data". One downside of this approach is that the interpretation of the topics is up to the individual, which blends the science (machine learning) with the art (human insight) of using topic models.

![Comparison Plot of HB2 and Bathroom Safety Topics](/images/STM-senate7.png)

In the case above, the two topics are pretty clear. The first shows words that are clearly related to HB2 -- specifically rhetoric against the law from Democrat legislators that are calling for its repeal and emphasize words focusing on the "gender", "transgender" and "lgbt" aspects of the law. The second topic is also clear that it represents the opposing side of the issue focusing on (for a lack of better label) "bathroom safety" by focusing on the "safety" risks to "girls" in "bathrooms" and "locker rooms". Obviously, this topic is the focus of Republican defenders of the law. 

Interestingly, note that interwoven within the "bathroom safety" topics are words that emphasize that topic is used to promote a call-to-action for constituents to "please share" or "sign [a] petition". 

Topics
------------------------

The dataset I used was pulled through Facebook public API using R's `Rfacebook` package. I manually identified 39 state senators' Facebook public pages. I limited my data to only posts between Jan 1 2015 and Dec 23 2016 (the data was pulled on that day). I also limited my analysis to only members of the 2015-2016 term, and excluded any candidates who were not elected as well as any members who resigned before the end of their term. In total, this amounted to analyzing 6,902 Facebook posts over the two years. 

Over time, posts increased recently for the November 2016 election, especially for the Republican senators.
!["HB2" and "Bathroom Safety" Topics from STM Analysis of NC State Senators' Facebook Posts](/images/STM-senate2.png)


!["HB2" and "Bathroom Safety" Topics from STM Analysis of NC State Senators' Facebook Posts](/images/STM-senate3.png)



Measuring the Effect of Party and Time on Topic Prevalence
------------------------

!["HB2" and "Bathroom Safety" Topics from STM Analysis of NC State Senators' Facebook Posts](/images/STM-senate4.png)

This network was a quick sketch to see what data I could pull and quick way of representing it. I could've gone in more detail (e.g. expanding the network or connecting it with text analysis of each senator's tweets), but I wanted to keep this first post short.

In addition to the follower network data, I've also pulled the senators' public posts on Facebook and Twitter. As I have time, I want to analyze what topics senators are discussing using topic modeling, a statistical-based family of algorithms for large collection of text documents that identifies topics, clusters of words that co-occur. Stay tuned.

(Note: it will take a few seconds for the interactive network below to load)

{% include STM-senate-network.html %}

