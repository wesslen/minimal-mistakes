---
layout: single
excerpt: "How does party affect what topics senators discuss on Facebook?" 
title:  "Structural Topic Modeling to analyze NC State Senators' Facebook Posts"
categories: [social media]
tags: [visualization, Facebook, topic modeling, political science]
---

Following my [previous post](https://wesslen.github.io/social%20media/nc-state-senator-twitter-network/), in this post I'm going to analyze the public Facebook posts of North Carolina state senators. I'm going to use a machine learning algorithm framework called Structural Topic Modeling (or STM for short) to measure, with statistical significance, the impact of party affiliation on the prevalence of topics discussed. 

STM is a recent extension of the popular topic modeling framework, a family of unsupervised algorithms that have been championed by computer scientists over the last decade. Topic models work by identifying clusters of words that co-occur together that can be interpreted as topics. For example, here are two topics that the model identified with my interpretation (called a label) as being "HB2" and "Bathroom Safety" for the respective topics.

!["HB2" and "Bathroom Safety" Topics from STM Analysis of NC State Senators' Facebook Posts](/images/STM-senate1.png)

Interestingly, STM is a new take on topic modeling but from a social scientist perspective, building a regression-like (causal inference) framework to test the impact of document attributes (e.g. author, time, location) on what topics are discussed. In my case, I'm going to analyze the effect of political party has on what topics a state senator posts on Facebook. As a secondary attribute, I'll also consider the effect that time (month) of the post had on the likelihood of the topic.

For readers interested in the code and more details, I've provided a [GitHub site](https://github.com/wesslen/NCStateSenateFacebook) including two RMarkdown output (html) files ([Part 1](https://htmlpreview.github.io/?https://github.com/wesslen/NCStateSenateFacebook/blob/master/code/STM-ncsenate-facebook-part1.html) and [Part 2](https://rawgit.com/wesslen/NCStateSenateFacebook/master/code/STM-ncsenate-facebook-part2.html) )

One of the model's outputs is a list of the words contained in all the posts (excluding insignificant stop words), ranked by how likely each word is to be used for each topic. This is incredibly helpful as the topics are determined by the data, not through manually coding that are subject to high time and monetary costs, inconsistencies and biases, and are more difficult to scale for "big data". One downside of this approach is that the interpretation of the topics is up to the individual, which blends the science (machine learning) with the art (human insight) of using topic models. Nevertheless, some topics are obvious while others require more thought.

There are plenty of ways to visualize topics to help interpret them like a comparison plot in which most distinctive words (relative to the two topics) are on either extreme while the words used in both topics are closer to the middle.

![Comparison Plot of HB2 and Bathroom Safety Topics](/images/STM-senate7.png)

In the case above, the two topics are pretty clear. The right hand side shows words that are clearly related to HB2 -- specifically rhetoric against the law from Democrat legislators that are calling for its repeal and emphasize words focusing on the "gender", "transgender" and "lgbt" aspects of the law. 

The topic on the left hand side represents the opposing side of the issue focusing on (for a lack of better label) "bathroom safety" by focusing on the "safety" risks to "girls" in "bathrooms" and "locker rooms". Obviously, this topic is the focus of Republican defenders of the law. 

Interestingly, note that interwoven within the "bathroom safety" topics are words that emphasize that topic is used to promote a call-to-action for constituents to "share" or "sign" a petition as compared to the "HB2" topic. 

Topics
------------------------

The dataset I used was pulled through Facebook public API using R's `Rfacebook` package. I manually identified 39 state senators' Facebook public pages. I limited my data to only posts between Jan 1 2015 and Dec 23 2016 (the data was pulled on that day). I also limited my analysis to only members of the 2015-2016 term, and excluded any candidates who were not elected as well as any members who resigned before the end of their term. In total, this amounted to analyzing 7,813 Facebook posts over the two years. 

``` r
| Party      | # of Senators | # of FB Accts | # of Posts | Avg Likes | Avg Comments | Avg Shares |
|:-----------|:--------------|:--------------|:-----------|:----------|:-------------|:-----------|
| Democratic | 16            | 14            | 3,983      | 214       | 17           | 81         |
| Republican | 34            | 25            | 3,830      | 59        | 11           | 46         |
```

The table above provides some important numbers to start. First, Republican outnumber Democrats in the state senate nearly 2-to-1 in both actual number of senators and number of seantors with FB pages. 

However, the number of FB posts are about the same over the last two years, indicating that on average Democratic state senators are more active with nearly 285 posts per active Democratic senator relative to 153 FB posts per active Republican senator.

Further, Democratic posts tend to get more of a social response than Republican with, on average, more likes, comments and shares. If you're more interested in which senators are the "leaders" on Facebook, see [Part 1 code](https://htmlpreview.github.io/?https://github.com/wesslen/NCStateSenateFacebook/blob/master/code/STM-ncsenate-facebook-part1.html).

How have the number of posts changed over time?

Let's look at the chart below. Each line represents the sum of all posts by party, the blue line is the Democratic posts and the red line is the Republican posts.

!["HB2" and "Bathroom Safety" Topics from STM Analysis of NC State Senators' Facebook Posts](/images/STM-senate2.png)

Republicans greatly expanded their posts for the 2016 election after having fewer posts than Democrats in 2015. Republicans picked up their Facebook content for the election, but this raises the question: what were they talking about?

That's where topic modeling (and STM) comes in. Unlike human coding methods to analyze text that require you to know in advance what the topics are, topic modeling let's the data to speak for itself and the algorithm identifies topics as word co-occurrence clusters.

For my analysis, I ran a 40-topic model that identifies the 40 most salient topics. The chart below outlines the size of the topics as a probability (so they all sum to one) for each topic. For example, the largest topic is about 5%. The words that best describe that topic are: "day", "us", "family", "today", "time". While these words only give us a glimpse to understand each topic, the top five words can generally give us a good idea of the topic. 

![STM Topics](/images/STM-senate3.png)

To simplify my results, I examined the most likely words for all forty topics and provided interpretative labels. For more detals on the words, see [Part 2 Code](https://rawgit.com/wesslen/NCStateSenateFacebook/master/code/STM-ncsenate-facebook-part2.html). 

Measuring the Effect of Party and Time on Topic Prevalence
------------------------

Structural topic modeling (rather than "vanilla" topic modeling like LDA) comes in when we want to consider the effect of political party on the topic proportions. The next chart explains these results.

!["HB2" and "Bathroom Safety" Topics from STM Analysis of NC State Senators' Facebook Posts](/images/STM-senate4.png)

This chart shows the expected difference in each topic's probability given a senator's party. Each dot is the estimated point difference with a line that indicates the estimate's 95% confidence interval. The labels are interpretations based on the most likely words per topic.

Topics on the left side are more Republican topics like "Presidential Election", "Bathroom Safety", and the "Economy/Jobs". On the other right side, "HB2", "Gerrymandering", and "#WeAreNotThis", a hashtag movement opposing Republican's legislation, are Democratic party topics.

Alternatively, we can examine the "neutral" topics, i.e. topics whose estimated difference is not significantly different than zero (e.g. "#NCPOL / #NCGA", "Roy Cooper", "Church", "Get Out the Vote"). For those topics, we cannot conclude that they're a more Democrat or Republican topic. 

We can also control for time as topics evolve over time.

The plot below provides the STM estimate of the effect month has on the likelihood of the top four Republican topics (with 95% Confidence Intervals). 

![Republican Topics - Effect of Time](/images/STM-senate5.png)

Or conversely we can look at the four most likely Democratic topics.

![Democratic Topics - Effect of Time](/images/STM-senate6.png)

Note that Senator Van Duyn has her own topic because she is the most active state senator on Facebook. She posts so frequently (over 1,000 posts in the last two years), that the algorithm detected there should be a unique topic to cover her own posts given how many posts she has written. For more details on the number of posts by senators, see the [Part 1 code](https://htmlpreview.github.io/?https://github.com/wesslen/NCStateSenateFacebook/blob/master/code/STM-ncsenate-facebook-part1.html).

Correlation Network
------------------------

Another way to analyze the results can be through a correlated topic network. 

{% include STM-senate-network.html %}

The plot above is a correlated topic network. Each dot (node) is a topic, with its diameter proportional to its size within Facebook posts by NC state senators. 

The color represents the effect party has on the use of the topic. Red topics are much more likely to be used by Republicans, blue topics are more likely Democratic, and white nodes are neutral (not significantly Republican or Democratic topics). The lines connected the topics (edges) represent examples of topics that share a significant number of words. Specifically, these topics have a larger than 10% correlation.

This analysis helps to demonstrate that the topics are not independent. In fact, there are multiple relationships between topics as some words can be used differently in depending on the context (topic).

Shiny App
------------------------

The interactive plot above was created through [Shiny](http://shiny.rstudio.com), a terrific web application framework for R users. As part of my own research, I'm working on creating a web-based visual interfaces to analyze structural topic model results for a variety of applications. 

In one example, you could click the above network, which then creates pop-up for the top five posts (through a Facebook URL) that best exemplify this topic (highest probability). Such interaction allows a user to quickly drill down from the topics down to examples to improve topic interpretations and facilitate a deeper understanding of the topics.

This is just the tip of iceberg and there's much deeper paths to go (e.g. use supervised machine learning to detect the relationship between likes, shares and comments with what topics are used), but that's it for now.


