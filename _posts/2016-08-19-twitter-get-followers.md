---
layout: single
excerpt: "A tutorial on using R to get Twitter followers via RESTful API"
title:  "Crawling Twitter Followers in R"
categories: [twitter]
tags: [api, followers, social network, profile]
---

Public API
----------

This tutorial outlines how you can pull the followers of multiple Twitter users via the public API.

One difficulty is that Twitter limits users via public API to 15 API calls per 15 minutes. Each API call can grab the name of up to 5,000 Twitter users. For example, if we want to grab all of the followers of a user with 25,000 followers, we would use 5 API calls (5k x 5 calls). Given there are 15 calls per 15 minutes, this implies we can get up to 75,000 followers' Twitter IDs every 15 minutes. This process works well for most Twitter users (< 100k followers), but could take hours for users with 1MM+ followers.

Yet these API calls will only return the name (ID) of the users. Usually, we'd want more information like the users' profile information. For this, we have to query user information in which we are limited to 180 calls, each one is limited to getting profile information for 100 users. This implies that we can only get profile information for up to 18,000 users each 15 min.

Therefore, at most we can get 75,000 followers and 18,000 user profile attributes every 15 minutes. While this may be frustrating, one way around this is to set up a crawler that will sleep after a limit is hit, then rerun once the limit has restarted.

The process below takes about 2.5 hours. You can limit it by changing the names of the Twitter users we want to grab its followers. For an easy application, run only 1 profile's followers and view your results.

Code to run
-----------

This code uses two packages: `twitteR` and `tweetscores`. You can un-comment the install packages if you need to install the packages.

``` r
#install.packages("twitteR"); devtools::install_github("pablobarbera/twitter_ideology/pkg/tweetscores")
library(twitteR); library(tweetscores)
```

    ## Loading required package: R2WinBUGS

    ## Loading required package: coda

    ## Loading required package: boot

The first is the common RESTful API package for R. The second is a package built by [Pablo Barbera](https://github.com/pablobarbera/twitter_ideology). The package was built to accompany a fascinating paper that uses Twitter users' followers to predict their political ideology.

**If you're completely new to pulling Twitter data through R**, see his page for instructions on how to set up your API and connection through OAuth. You will need to follow his instructions for how to set up your connection.

For now, I'll use a saved file that has my permissions.

Now, let's list the 15 Charlotte bars/breweries (by Twitter user name) we'll be pulling.

``` r
names <- c("NoDaBrewing","NCBeerTemple","oldemeckbrew","BirdsongBrewing","UnknownBrewing","TripleCBrew","SaludNODA",
           "craftgrowler","sugarcreekbrew","WoodenRobotAle","goodbottleco","SycamoreBrewing","LegionBrewing",
           "D9Brewing","HeistBrewery")
```

Let's initialize a dataset that we'll fill in with followers' profile information.

``` r
#initialize our dataset
beer.followers <- data.frame(
  id_str=character(),
  screen_name=character(),
  name=character(),
  description=character(),
  followers_count=integer(),
  statuses_count=integer(),
  friends_count=integer(),
  created_at=character(),
  location=character()
)
```

Now we'll run a loop for each brewery, first to get its followers and then to get its users' profile information.

``` r
for (i in names){
  print(i)
  
  # get followers
  followers <- getFollowers(screen_name = names[1],
     oauth_folder="~/Dropbox/credentials",
     cursor = -1, user_id = NULL, verbose = TRUE, sleep = 1)
  
  #Batch mode - get User ID info (ID, name, description, etc)
   userdata <- getUsersBatch(ids=followers,
     oauth_folder="~/Dropbox/credentials")
  
  userdata$brewery <- i
  userdata$time_stamp <- format(Sys.time(), "%a %b %d %X %Y")
  beer.followers <- rbind(beer.followers, userdata)
}
```

Let's save our results to a csv file.

``` r
write.csv(beer.followers, "beerfollowers.csv", row.names = FALSE)
```
