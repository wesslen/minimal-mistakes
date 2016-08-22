---
layout: single
excerpt: "An overview of Twitter resources for university researchers "
title:  "Twitter FAQ for UNCC Researchers"
sitemap: false
permalink: /twitter-faq.html
---

## **Introduction**

I've created this FAQ to disseminate information about UNCC's Data Science Initiative (DSI) Twitter resources. 

Please email me at rwesslen@uncc.edu if you have any questions.

## **Acquiring Twitter Data**

<details>
  <summary>
**How can I get Tweets?**
  </summary>
  
  1. Public API's (Streaming or RESTful)
    * Pro: Free, can pull Tweets, profile, follower data
    * Con: Significant size limits, only available for recent data
    
  2. Spritzer
    * Pro: Long time series of sampled Tweets, archived on SOPHI
    * Con: Raw JSON, not ideal for micro-analysis given sampling
    
  3. Gnip Historical PowerTrack API
    * Pro: Full firehose for historical data
    * Con: University-wide limits, filter rules, very big and noisy

</details>

<details>
  <summary>
**What are the public API's?**
  </summary>

The best place to get started is by signing up for a Twitter public API. You can do this step on your own and it is free. The public API's are a great introduction to what Twitter data looks like and how to query Twitter data. Unfortunately, there are significant limits on the public API (e.g., 1% sample, restrictive number of Tweets, only recent or "go-forward" Tweets).  

While some researchers get discouraged (and thus immediately want full sample resources), please consider utilizing the public API tools first. Twitter data is very big and noisy. Unless researchers have a broad range of computer science tools, most researchers need experience first with smaller datasets before moving to larger datasets. 

In my opinion, the easiest way is to get started on the public API's is to use R's `TwitteR` (RESTful API) or `StreamR` (Streaming API) packages. More experienced programmers may find Python or Java to be easier, especially if they are familiar with API's.

This [GitHub repository](http://www.github.com/wesslen/social-media-workshop) is a great resource to get started on either `TwitteR` or `StreamR` package.

</details>

<details>
  <summary>
**How can I setup a Twitter public API (RESTful API)?**
  </summary>

The easiest way is through R. These instructions assume you already have R and R Studio.

Let's use the RESTful API via the `TwitteR` package in R:

  1. You need a Twitter account. If you don't have one, please go to Twitter.com and sign up.
  2. With your Twitter account, you will need to set up a developer application. [Here are instructions to set up a developer application](http://www.r-bloggers.com/getting-started-with-twitter-in-r/).
  3. Next, in R, install the package `TwitteR` . `TwitteR` is an R package to access Twitter's RESTful API. It can pull samples of Tweets for the last 7-21 days. You can install the package with this command: `install.packages("TwitteR")`
  4. Next, open [this code in R](https://github.com/pablobarbera/social-media-workshop/blob/master/01-twitter-data-collection.r) 
  5. You will need to replace the four parts of the code where there are "XXXXXXXXX" with your information that you will get from your Twitter application in part 2. These are "codes" that allow you to be identified to Twitter without requiring you providing your username/password. The four codes are: consumerKey, consumerSecret, accessKey, accessToken. 

In addition, [here's a good tutorial](http://geoffjentry.hexdump.org/twitteR.pdf) on using the `TwitteR` package. It's a little out of date, but will show you basics of querying.

There is also the `StreamR` package, which allows you to stream Twitter data (i.e. pull data on a "go-forward" basis). It's a bit more complicated so I'd suggest using the TwitteR package first. 

</details>

<details>
  <summary>
**What's the difference between the RESTful API and the Streaming API?**
  </summary>

The RESTful API and the Streaming API differ in what time their pulling data from and thus how long the connection between yourself and Twitter must be maintained.

For the official documentation on the difference, see [this page from Twitter](https://dev.twitter.com/streaming/overview).

The RESTful API is pulling from Tweets that occurred up to the last 7-21 days. Because of this pull from historical data, the connection with Twitter can be a one-time occurrence (request and receive the data). The R package [`TwitteR`]() is used to pull from Twitter's RESTful API. Here's an overview of [Twitter's RESTful API](https://dev.twitter.com/rest/public) documentation that may be helpful too.

On the other hand, the Streaming API pulls data on a "go-forward" basis that pulls any future Tweets based on parameter rules. Unlike the RESTful API, the Streaming API requires a continual connection with Twitter as new Tweets need to be pulled going forward. The R package [`StreamR`](https://cran.r-project.org/web/packages/streamR/streamR.pdf) uses Twitter's Streaming API.

For Python users, [Tweepy](http://www.tweepy.org/) is a helpful set of libraries to access the Twitter APIs. 

Related to the Twitter Public API's, more advanced R users may also find the [`TwitteR2Mongo`](https://www.r-bloggers.com/gathering-twitter-data-with-the-twitter2mongo-package/) and the [`tweet2r`](https://cran.r-project.org/web/packages/tweet2r/tweet2r.pdf) packages also useful.

</details>

<details>
  <summary>
**What is Spritzer?**
  </summary>

Spritzer is an archived time series of 1% sample of Twitter data on SOPHI. This data was pulled through Twitter's public API and stored in SOPHI. This data is ideal for researchers interested in time series analysis in which sampling is not an issue (e.g., aggregate analysis). The downside of this analysis is that currently, the data is only available in raw JSON form, separated into numerous ten minute files. Therefore, to access this data, the researcher needs to have programming skills to aggregate, filter and handle raw JSON. 

Currently, we're working on creating a user friendly interface that will allow non-technical users to query the data.

</details>

<details>
  <summary>
**What is the Gnip Historical PowerTrack API?**
  </summary>

UNCC's DSI has a contract with Gnip that allows university researchers to obtain Twitter data for academic research through their proprietary firehose: Gnip Historical PowerTrack API. This option is ideal for experienced Twitter researchers who are interested in full, cross-sectional samples of Twitter.

Unfortunately, there are significant limits on the number of Tweets and the time range (number of days) that can be pulled per month across all researchers. Given these constraints, **researchers need to carefully create filtering rules based on their research question to minimize noise (i.e., waste Tweets) and maximize their objective Tweets**. Even for experienced researchers, this task is very difficult and usually requires multiple iterations to optimize queries.

See the next section for details about the process. [This page](http://support.gnip.com/apis/historical_api/) includes documentation by Gnip on the Historical PowerTrack API.

One alternative to pulling new Gnip data is to access several open (to university researchers) datasets we have previously pulled and saved on SOPHI. These datasets are available for classwork and we strongly encourage new researchers to consider using these datasets before submitting requests for custom Gnip data.

</details>

<details>
  <summary>
**What archived Twitter datasets are available?**
  </summary>

  1. Geolocated Tweets for 10 cities (including Charlotte): Jun 2015 - May 2016 (sizes vary; for example, 3 months of Charlotte data is about 200k Tweets)
  
  2. Year long dataset of the four major presidential candidate Tweets (Hillary Clinton, Donald Trump, Bernie Sanders, Ted Cruz) (about 35k Tweets)**
  
  3. Year long dataset of all Tweets for Forbes top 100 companies (about 2.5MM Tweets)
  
  4. Jan 2015 - May 2016 I-77 Toll Lane Related-Tweets (~13,000 Tweets)**
  
  ** Available in zipped CSV converted file.
  
</details>

<details>
  <summary>
**What is SOPHI?**
  </summary>

SOPHI is the university's Hadoop cluster; think "big data" data warehouse.

Here is a link to [SOPHI's website](http://sophi.uncc.edu).

</details>

<details>
  <summary>
**How can I get non-Tweet data like the followers or updated profile data?**
  </summary>

Gnip does not provide access to non-Tweet data. Therefore, you can only obtain this information through the public API's on your own. 

The public API severely limits the amount of information you can pull; therefore, studies on Twitter social network (e.g. followers) tend to focus on small follower networks and may require crawlers to continuously pull data for an extended period (hours or days).

Check out [these materials](https://github.com/pablobarbera/social-media-workshop/blob/master/01-twitter-data-collection.r) from Pablo Barbera that outlines functions using the TwitteR package to pull user timeline and profile data. 

To pull followers, check out [this blog post](https://www.r-bloggers.com/mapping-twitter-followers-in-r/). There are additional steps on how to map out the location of followers but if you're just interested in learning how to pull followers, see the first few steps.

</details>

## **Gnip's Historical PowerTrack**

<details>
  <summary>
**What's the process to get Gnip Historical PowerTrack data?**
  </summary>

To begin the process, email me (rwesslen@uncc.edu). 

Please answer the following questions:

  1. What is your research question and objective?
  
  2. What experience do you have with Twitter data?
  
  3. What filtering rules are you looking for?
  
  4. What software tools do you have experience with (R, Python, SAS, Java, etc.)?
  
To expedite the process, please plan to come to my weekly office hours in Barnard 109B on Thursdays between 1pm - 3pm. If you can't make that time, please let me know in your email.

All researchers should expect at least a one month turn-around. Times can be reduced depending on the number of total requests by the university community and researchers who are very prepared, i.e. who have clear or simple filtering rules.

Because of our limits, faculty and PhD students have first priority. Preference will be granted to researchers with clear and tested filtering rules, previous experience handling Twitter data and the significance of the research objective (e.g., specific publication or grant proposal). 

Access for all other researchers will only be available on a case-by-case basis (e.g., researchers very new to Twitter, researchers with unclear research objectives, analysis for classwork). 

</details>

<details>
  <summary>
**How is the data pulled?**
  </summary>
  
From [Gnip's FAQ](http://support.gnip.com/faq/): Historical PowerTrack (HPT) API provides a data service that is used to filter and download data from the Twitter archive, starting with the first Tweet in March 2006. 

Historical data are generated through a series of steps, and the API is used to navigate through these steps:

  1.  Create Job with filter rules.
  2.  Estimate the Job's size (# of Tweets, file size).
  3.  Accept or reject Job.
  4.  Monitor accepted Job progress.

A job consists of filter rules need to be created that represent what data is to be pulled. Every filter rule needs a time horizon and either (maybe both) keywords or geo-location bounding box. The filter rules must follow Gnip's Historical PowerTrack convention in which JSON file is posted to their API. 

At that time, an estimated Tweet volume will be returned back (usually in 10-20 minutes). These estimates provide two main functions. First, it allows the researcher to get an idea of their data, including whether there filter rules are too broad (i.e. returns to many Tweets), are too narrow (i.e. too few Tweets) or are not consistent with expectations (e.g. expect millions of Tweets but only gets a few thousand). Second, these estimates allow DSI to plan its monthly tweet/day allocation accordingly. This is most important as estimates may be deemed to be too high and the researcher may be asked to reduce his or her request.

After a filter rule is approved, then the rule will be accepted in which Gnip will begin processing the data. The data is provided as a list of URL's posted on Amazon AWS divided into 10 minute increment JSON files. Normally, these files would then be ingested and processed into SOPHI, at which time the researcher will be provided access to the files for their research purposes.

For more general questions on Gnip's Historical PowerTrack, here's its [FAQ page](http://support.gnip.com/faq/).
</details>

<details>
  <summary>
**Can I get direct access to Gnip?**
  </summary>

No. At this time, we restrict licenses to coordinate and budget our monthly allocation. 

For your pulls, you can coordinate with our team and we will run the estimates and approval/rejection of jobs. Unfortunately, given our limited resources, multiple inquiries by various university research groups can limit our ability to work with individual researchers (i.e. to test filter rules, estimate counts).

However, those researchers who are pro-active (i.e. create their own filter rules in JSON, test out and refine their own rules on public API and/or spritzer) will likely have much quicker turn-around times for data. Therefore, I'd strongly suggest researchers get familiar with how to create filter rules.

</details>

<details>
  <summary>
**How can I create filter rules?**
  </summary>

  1.  **Keywords**: *Choose Tweets that include specific words.* 
  
  This is the easiest way but it's also one of the most difficult. Why? Because there is more noise in Twitter than you can imagine. 
  
  For example, if I choose the #Hamilton (for the musical, Hamilton), I will get a flood of Tweets from Chattanoga, TN and Cincinnati, OH that have nothing to do with the musical Hamilton. Why? Because both have suburb cities named Hamilton including a flood of traffic reports and daily Tweets about the city. 
  
  This problem is greatly exacerbated when considering broad key words (e.g., a search for Cam Newton using the word "cam" will bring back all Tweets with words like "came", "camel", "camera"). Twitter provides a range of documentation on using special operators to get special subsets of words.
  
  The easiest (and usually cleanest) way of querying Tweets by keywords is to use unique hashtags that are relevant for only the type of Tweet you're looking for.
  
  2.  **Geo-location bounding box or radius**: *Choose geo-located Tweets within specific coordinates.* 
  
  For this rule, geo-located Tweets within four coordinates (bounding box) or within a specific mile radius of a coordinate pair can be tagged. One caveat is that the size of the bounding boxes or radius must be no larger than 25 miles (radius or height/width of bounding box). 
  
  An implication is that large areas need to be divided into multiple bounding boxes. 
  
  See "bounding_box" and "point_radius" operators on [this page](http://support.gnip.com/apis/search_api/rules.html#Operators) for a full list of geo-location rules.
 The next section includes details on what location and geo-loation data is available in Twitter.
  
  3.  **User and profile information**: *Choose Tweets with any user that has specific profile information (e.g., the user Twitter ID or display name, profile location text, profile country code).* 
  
  This is helpful to grab Tweets for specific users; however, what complicates this process is the open-ended nature of most profile information. 
  
  Profile attributes can be modified (like display name or profile location); thus, some profile rules can grab some profiles sometimes but not after a user changes his or her profile attributes. The only field (to my knowledge) that does not change is the profile ID. When trying to pull specific users' Tweets, query on their profile ID, not their display name.
  
[This documentation page](http://support.gnip.com/apis/powertrack/rules.html) provides further details.

</details>

<details>
  <summary>
  **Can you provide an example of filtering rules?**
  </summary>
  
  Check out [this blog](http://support.gnip.com/articles/translating-plain-language-to-powertrack-rules.html) on translating a research question into Gnip filter rules.

</details>

<details>
  <summary>
  **What other important filter rules exist?**
  </summary>
  
  1.  **Re-tweets**: Are you interested in only original content or the "echo" of Tweets (Re-tweets)? Researchers interested in only original content should consider adding in the "-is:retweet" operator to filter out all Re-tweets. This can reduce your dataset's size, which makes the dataset easier to handle with less noise.
  
  2.  **Sampling**: For some very broad research questions, the results are too large for our limits or what we have available in our monthly allocation. One way to ensure some data is to use the "sample:" operator to return a specific sample percent of Tweets that meet a filter rule set.
  
  3. **Profile Country Code**: For international Tweets, it may be helpful to query only those accounts that are registered for specific countries. Instead of the raw profile location field (that is very noisy) and the limited number of geo-located Tweets, using the profile country code can provide larger samples.
  
  4. **Language Code**: You can query only Tweets (as classified by Gnip/Twitter) in a specific language ("lang" operator) or from users who have classified their profiles as using a specific language ("bio_lang").
  
  5. **Profile Attributes**: You can query by users' number of Tweets ("statuses_count"), number of followers ("followers_count"), number of friends ("friends_count") or number of lists ("listed_count").
  
</details>

<details>
  <summary>
**What are the Gnip limits?**
  </summary>

There are two monthly limits: 10 million Tweets AND 200 days worth of data. These limits apply to the entire university. In other words, we have to allocate these limits across **all** UNCC researchers.

First, the 10 million Tweets is easy to understand. We can at most get up to 10 million Tweets per month for the entire university. One caveat is that Twitter data is much larger than new-users ever understand. I'll explain more in a later question.

Second, and most difficult to understand, the university has only 200 days worth of Twitter data that we can pull per month. This limit is the most restrictive and severely limits the ability to pull data across time. This limit is a key reason this tool is best only for cross-sectional analysis. 

</details>

<details>
  <summary>
**Ok - so can you give me an example?**
  </summary>

Let's assume there is a faculty member who wants one year of Tweets. Let's call him Frank. 

Frank wants all Tweets with the keyword "#selfie" between Jan - Dec 2015 (12 months). Assume this query yields 3 million Tweets.

Let's also assume there are no other requests for Twitter data.
  
Given that he wants 12 months of data, under our Gnip limits, we cannot pull more than 200 days per month. Therefore, his request would need to take at least two months. This is one reason why ask researchers to expect at least one month (or longer) for their data, as requests for long time series will likely need to be broken up across multiple months.

Even worse, if we were to pull his two requests (say the 1st of each month), this would prevent any other researchers from pulling the first month and leaving only about 35 days of remaining day budget for the 2nd month.

**Think about that -- one researcher who insists on pulling from a specific time frame could limit everyone else's access to the tool!!** This is the most critical and difficult part to understand about pulling from the Historical PowerTrack tool. Even though his request is well below the montly 10MM Tweet limit, by insisting that his one year of data is absolutely necessary, he could limit nearly all researchers access to new Twitter data for up to two months!

</details>

<details>
  <summary>
**How does the API produce data? How do I get the data**
  </summary>

[Answer from [Gnip FAQ](http://support.gnip.com/faq/)]

The Historical PowerTrack data service generates a time-series of data files. Each file is gzip compressed and contains Tweets in JSON format (either ‘original’ or Activity Stream format). Each file covers a 10-minute segment of the overall job: a 1-hour job will contain up to 6 files, and a year-long may contain over 52K files.

When a Job is complete, a Data URL is provided. This URL returns a list of download links, one for each file generated.

Depending on the size of the file, we can download the files locally (small data requests) or ingest it in SOPHI (larger data requests). Those requests that require SOPHI processing may take longer to process.

</details>

<details>
  <summary>
**What's an example of JSON format?**
  </summary>

Here is an [example](https://github.com/jimmoffitt/json2csv/blob/master/templates/tweet_profile_geo.json).

Also [this is another great reference](http://support.gnip.com/sources/twitter/data_format.html).

</details>

<details>
  <summary>
**What's JSON? I just want a spreadsheet (e.g., comma-separated file).**
  </summary>

JSON is a hierarchal data structure that's more memory-efficient. 

Gnip offers a template JSON-to-CSV converter, written in Ruby code. I highly recommend leveraging it and reading over the materials. 

Here's a [link to get started](http://support.gnip.com/articles/json2csv.html) and another to [its GitHub repo](https://github.com/jimmoffitt/json2csv).

</details>

## **Twitter Analytics**

<details>
  <summary>
**What types of analysis can I do on Twitter data?**
  </summary>

  1. Text mining: the Tweet (body of the Tweet); additional text analysis can be done on profile descriptive attributes like location description. This is the most common analysis. Please see [this GitHub site for Twitter Text Mining workshop materials](http://www.github.com/wesslen/eui-text-workshop).
  
  2. Time and space: Tweets across time or space (geo-location). For an introduction on how to analyze Tweets across time and space, I created a [tutorial](http://webpages.uncc.edu/rwesslen/blogs/01-beerandtweeting.html) analyzing three months of Charlotte beer-related tweets.
  
  3. Social network: Follower or retweet networks. This type of data is severly limited by data restrictions. As mentioned earlier, follower data is only available through the public API (not Gnip) and is subject to very strict limitations. Researchers interested in Twitter social networks should try to minimize their research question to focus on small rather than very large networks.
  
</details>

<details>
  <summary>
**What location data is available in Twitter?**
  </summary>

Three types (think 3P's):

  1. Points
  
  2. Polygons (sometimes called Places)
  
  3. Profile information
  
Points and polygons are geo-located, i.e. GPS coordinates are provided. Points are exact GPS latitude and longitude coordinates. Polygons are broad bounding box (polygons) with four pairs of latitude and longitude coordinates. The main problem with geo-location is that geo-location is only provided by users who "opt-in" to provide their location. This leads to a significant sample selection problem.

Profile information is descriptive text (e.g., profile location, profile country code). This data is not geo-located and require text extraction. Even worse, profile location is "open-ended" in which users can customize their location to whatever the user wants. This leads to an extremely "fat tail" problem in which there are many different possible combinations. While some users provide generic names (e.g., Charlote, NC), the majority do not. Adding to the problem, some users opt not to provide any profile location.

</details>

<details>
  <summary>
**A bounding box doesn't guarantee all Tweets in an area -- just those that are geo-located?**
  </summary>

Correct. If I Tweet standing in uptown Charlotte but I have geo-location turned off (no point or polygon), then any bounding box filtering rule for that time period will **not** include my Tweet.

</details>

<details>
  <summary>
**How large can bounding boxes be?**
  </summary>

It depends. To my knowledge, there is no size limit on bounding boxes for filter rules through Twitter's public API. 

However, Gnip Historical PowerTrack limits bounding boxes (or point radius) to no larger than 25 miles in width or height for bounding boxes (radius for point radius operator).

</details>

<details>
  <summary>
  **How can I query on profile location (non-geolocated location data)?**
  </summary>
  
Check out [this page](http://support.gnip.com/articles/twitter-geo-referencing.html).

One key note: DSI's current license does not provide us with access to the Profile Geo Enrichment data provided by Gnip. 

</details>


## **Best Practices and Tips**

<details>
  <summary>
  **What are other resources?**
  </summary>
  
Resources on data consuming and processing:

  1.  [Consuming, parsing and processing Tweets with Python](http://support.gnip.com/articles/consuming-parsing-and-processing-tweets-with-python.html)
  
  2.  [Geo-JSON and Visualizing Twitter Geo Data](http://support.gnip.com/articles/visualizing-twitter-geo-data.html)
  
  3.  [Java, Maven and MongoDB Twitter Collector for Public APIs](https://github.com/usc-isi-i2/TwitterCollector)
  
  4.  [Open-source Twitter Data Analytics book using Java](http://tweettracker.fulton.asu.edu/tda/)
  
  5.  [Alex Perrier's Tutorial on Twitter Follower Topic Modeling -- great LDA and Python Twitter sections!](http://alexperrier.github.io/jekyll/update/2015/09/16/segmentation_twitter_timelines_lda_vs_lsa.html)
  
Helpful text/books:
  
  1.  [Social Media Mining open-source textbook](http://dmml.asu.edu/smm/)
  
  2.  [Morstatter et al (2016): Can One Tamper with the Sample API? Toward Neutralizing Bias from Spam and Bot Content](http://www.public.asu.edu/~fmorstat/paperpdfs/www2016.pdf)
  
  3.  [Morstatter et al (2014): When is it Biased? Assessing the Representativeness of Twitter’s Streaming API](http://www.public.asu.edu/~fmorstat/paperpdfs/www2014.pdf)

</details>

<details>
  <summary>
**Tweet dates are in standardized GMT: How can I convert to Eastern Time?**
  </summary>

If you're using R, check out this [GitHub Gist](https://gist.github.com/wesslen/a659cc0548f7f0af6d143e2269061f66).

</details>

<details>
  <summary>
**I'm new to text mining: How can I learn how to analyze Tweets?** 
  </summary>

This summer, I held a day-long workshop via Project Mosaic on text mining for Twitter. The materials were based on materials by [Pablo Barber&aacute;](http://pablobarbera.com/). I modified them including adding customized "challenges" using sampled Gnip data for geo-located Charlotte Tweets. I'd recommend them for researchers interested in learning on text basics.

Here's the [GitHub repo](https://github.com/wesslen/eui-text-workshop).

The "Hands-on" Challenges using Charlotte Gnip Geolocated Tweets:

  1. Intro to Text: [Identifying Carolina Panther Tweets](https://cdn.rawgit.com/wesslen/eui-text-workshop/master/01-intro/04-challenge1-solutions.html)
  
  2. Supervised Learning: [Difference in Panther Point & Polygon Geo-located Tweets](https://cdn.rawgit.com/wesslen/eui-text-workshop/master/02-supervised/02-challenge2-solutions.html)
  
  3. Unsupervised Learning: [Using User-level Topic Modeling to identify Tweet Topics and "Experts"](https://cdn.rawgit.com/wesslen/eui-text-workshop/master/03-unsupervised/02-challenge3-solutions.html)

</details>

<details>
  <summary>
**Can Tweets be deleted? What's the impact?**
  </summary>

Yes! At any time, users can delete Tweets. As soon as a Tweet is deleted, it will no longer be available through the public API or firehose API (Gnip Historical PowerTrack). What this means is that you may never be able to fully replicate Tweet datasets.

</details>

<details>
  <summary>
**When analyzing users (profile-level attributes), is it better to use the Actor ID or Display (Screen) Name Field?**
  </summary>
  
It is best to identify users by their Actor ID (Twitter User ID) rather than their display name (aka handle, username).

Why? Display names can change while Actor ID cannot be changed (see [this page](https://support.twitter.com/articles/14609#)).

For a nifty website that allows you to find any Twitter users' Twitter ID given its Twitter display name (handle), check out [this site](http://gettwitterid.com/).

</details>

<details>
  <summary>
**My data includes the klout score -- what is the klout score?**
  </summary>
  
The klout score is an enrichment to the Tweets (via Gnip) that is a user-level score of ones influence. Think of it as an attempt to create something like a credit score for a user's presence on social media. 

See [this page](http://support.gnip.com/enrichments/klout.html) for an introduction to the klout score.

</details>