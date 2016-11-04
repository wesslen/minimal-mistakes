---
layout: single
excerpt: "How researching social media through data science may help understand Charlotte’s Protests" 
title:  "Social Media’s Role in Charlotte’s Protests"
categories: [social media]
tags: [visualization, Twitter, text mining, topic models]
---

What role did social media have in the events following the tragic death of Keith Lamont Scott? Did social media simply act as a mirror to reflect pre-existing anger and sentiment? Was its role more complex as real-time text, photo, and video posts shaped opinions; reinforced prior beliefs; and ultimately affected behaviors that turned peaceful protests violent?

As a researcher studying computational social science at UNC Charlotte, these are all questions that perplex me. Through UNC Charlotte's [Data Science Initiative](http://dsi.uncc.edu), [Data Visualization Center](http://viscenter.uncc.edu/), and [Project Mosaic](http://projectmosaic.uncc.edu), the university's social science research initiative, I'm working with researchers across disciplines to address these questions by combining the insight of social science research with cutting-edge computer science tools sometimes labeled "big data" and "data science".

Like a detective who traces clues to solve a mystery, we examine the digital "footprints" left on social media to provide a broader, data-driven story for events like the aftermath of Keith Lamont Scott’s death. While our research on the Charlotte protests is early, I want to provide my initial observations, explain their implications and explore where further research may explain even more.

![](/images/clt-protest.png)

Timeline of Events
------------------------

On Twitter, the minute-by-minute volume and content of messages with protest hashtags directly tracks key protest events. 

For example, the first night’s Twitter volume grew as protests started on Old Concord Road (Point A on figure above) and started peaking around CMPD’s 11PM order for the crowd to disperse. At the same time, the content focused on issues around the shooting (e.g. “disabled”,“shot”, “book”, “reading”, “unarmed”).

Wednesday night’s tweets surged after the shooting of Justin Carr (Point B) with messages of breaking news of “riot”, “tear gas” and “shot”. Tweet volumes remained high through the night as reports of a “man killed” increased and words like “emergency”, “rioters”, and “violence” emerged on Twitter, while Governor McCrory declared a state of emergency at 11PM (Point C). 

Moving forward, we will explore the content deeper using machine learning algorithms to detect topics and how they trend along these events.


Proportion of Tweets by Users
------------------------

Next, most messages were written by a disproportionate few, resembling the "80/20” rule. 67% of the 1.3 million Charlotte protest tweets were posted by the top 20% of users. 

Even more extreme, the top 1% (4,300 Twitter accounts), accounted for nearly 20% or 272,819 tweets. In fact, the top 20 most active users (including a mixture of bots, spammers, and one very dedicated journalist) averaged about 700 protest tweets over six days. Because of such overrepresentations, generalizing Twitter conversations to the population as a whole is difficult because of sample selection bias. 

However, many of the most active users were not bots but people: active, intense and passionate about the protests. By combining what users say (content) with how they say it (intensity), we can measure varying “sentiments” towards the protests on Twitter.

Scale of the Protests on Twitter
------------------------

Last, while 1.3 million tweets over six days seems like a large number, it is dwarfed by the nearly 500 million tweets worldwide each day. 

The relatively small scope of our personalized newsfeeds distorts our notion of size in social media and makes it easy to presume a “viral” post is much larger than it actually is. This becomes a larger problem when you consider the real-time feed of social media information. 

The constant stream of posts, especially in fast-paced events like the protests, provides the potential for a feedback effect that can propagate misinformation, intentionally or unintentionally. By analyzing the content, we can assess the accuracy of messages and track how such information propagates along social networks from one friend to another. 

Final Thoughts
------------------------

As we become increasingly tethered to social media through our mobile devices, the importance of researching it will continue to rise. While social media research is still relatively new, we hope to progress by answering deeper questions using sophisticated tools in text, social network, and geospatial analysis. 

