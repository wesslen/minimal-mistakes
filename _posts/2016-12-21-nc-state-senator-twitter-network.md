---
layout: single
excerpt: "How connected are NC state senators on Twitter?" 
title:  "Analyzing the Twitter Network of NC State Senators"
categories: [social media]
tags: [visualization, Twitter, social network, political science]
---

The recent election renewed my interest in politics, especially North Carolina politics. To start, I wanted to consider the social media presence of the NC General Assembly Senate members. I chose the NC Senate for simplicity purposes because there are only 50 Senators and to start I needed to manually collect the Senators' Twitter usernames and Facebook public page information. The list includes 2016 senators and not has not been updated for new senators who will be taking office next year. I found that 36 Senators had Twitter accounts and 39 Senators had (public) Facebook pages. 

In this example, I am only considering how NC state senators follow one another on Twitter and exclude all non-senators from the analysis. I do this for simplicity but I could generalize the network by adding non-senators -- the problem is the network gets large very quickly. I don't consider yet the content of posts or Facebook data but I'm in the process of collecting the data.

Twitter Follower Network
------------------------

My first task was to pull Twitter follower data through Twitter's public API and the `twitteR` package in R. See my [earlier post](https://wesslen.github.io/twitter/twitter-get-followers/) for instructions on how to pull the data on your own. After pulling all of the followers for each senators' Twitter profile, I kept only the senators to minimize the network.

(Note: it will take a few seconds for the interactive network below to load)

{% include StateSenateTwitterNetwork.html %}

 

The plot above is an interactive network of the NC state senate's Twitter follower network. Each node (dot) is a NC state senator. The color represents which the senator's political party: red for Republican, blue for Democrat. The size of the node represents how many Twitter followers (in total, not just from other state senators). Therefore, the largest nodes are the senators who have the largest follower networks.

The table below shows the top 5 senators by the number of Twitter followers. Senator Jeff Jackson (Democrat - District 37) has the largest number of Twitter followers with 10,515 users. This explains why Senator Jackson's node is the largest in the network. Senator Phil Berger (Republican - District 26) is 2nd with 8,951 followers. Therefore, one could argue that Senators Jackson and Berger have the widest network influence as measured by the number of followers.

``` r
| Senator (District) | Party     | Followers | Posts | Friends |
|:-------------------|:----------|----------:|------:|--------:|
|Jeff Jackson (37)   |Democratic |     10,515|  3,816|    1,452|
|Phil Berger  (26)   |Republican |      8,951|  1,824|      260|
|Andrew Brock (34)   |Republican |      4,027|  4,833|    3,740|
|Mike Woodard (22)   |Democratic |      2,664|  1,113|    1,419|
|Dan Blue     (14)   |Democratic |      2,615|    816|      486|
```
Follower, posts and friend counts are as of Dec 18, 2016.

Each edge (line) represents Twitter followship. Therefore, connections represent whether a given senator follows or is followed by his or her Senate colleagues. Senators with a lot of connections represent those who follow many of his or her colleagues while those senators with few edges follow few colleagues. Related, the position of each senator is a function of the number of connections. For example, Senator Jackson is in the center of the network because he follows nearly all of his Senate colleagues who are on Twitter. 

On the other hand, senators who are on the periphery of the network are only connected to the senators they are relatively near. This helps explain why the network is clustered by party (color). On average, senators tend to follow members of their own party more often than members of the opposite party. Exceptions to this are senators that are positioned in the middle (e.g. Senator Jackson, Senator Rabon, Senator Ford, Senator Barefoot). 

Feel free to click and observe qualities about the network. Generally, there are more advanced ways of characterizing the network through centrality measures which are measurements of network qualities. For example, the simplest centrality measure is the degree centrality, which simply measures the number of connections. 

Future Analysis
------------------------

This network was a quick sketch to see what data I could pull and quick way of representing it. I could've gone in more detail (e.g. expanding the network or connecting it with text analysis of each senator's tweets), but I wanted to keep this first post short.

In addition to the follower network data, I've also pulled the senators' public posts on Facebook and Twitter. As I have time, I want to analyze what topics senators are discussing using topic modeling, a statistical-based family of algorithms for large collection of text documents that identifies topics, clusters of words that co-occur. Stay tuned.

