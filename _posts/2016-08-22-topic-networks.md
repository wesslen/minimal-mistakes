---
layout: single
excerpt: "Visualizing LDA Topic Correlation Network"
title:  "Using R to detect communities of correlated Topics"
categories: [text mining]
tags: [topic modeling, lda, community detection, social network]
---

Creating a topic network
------------------------

For Project Mosaic, I'm researching UNCC publications in social sciences and computing including analyzing the abstract texts and understanding the co-authorship social network.

For text mining portion, I'm running LDA (topic modeling) on five years worth of publication abstracts to identify key research themes by university researchers. (If you're not familiar with LDA, please review documents from [Tyler Rinker's Topic Modeling Repo](https://github.com/trinker/topicmodels_learning).)

One problem I came across was how to measure and analyse the correlation across topics. In particular, I wanted to analyze the correlation between topic word probabilities by creating a network that connects similar topics.

For this exercise, I combined the code provided in Tyler's Repo (see the section "Network of the Word Distributions Over Topics (Topic Relation)") and helpful network visualizations from [Katherine Ognyanova's phenomenal Network Visualization Tutorial](http://kateto.net/network-visualization).

Data preparation
----------------

Our first step is to load our topic matrices that are outputs of LDA. One variation in the way I ran LDA was that I ran an "author-centered" LDA in which all author's abstracts were combined and treated as one document per author.

I ran this because our ultimate goal is to use these topic modeling results as an information retrieval system to determine which researchers are experts in each topic.

As an alternative, you can use the output of the `topicmodels` package `lda` function to create any word-topic and document-topic matrices. Take the output of your `lda` function and run the `posterior` function on the output.

``` r
# load in author-topic matrix
author <- read.csv("./socsci_shiny/author_topics.csv", stringsAsFactors = F)

# update author name column
names <- colnames(author)
names[1] <- "author_name"
colnames(author) <- names
author$author_name <- factor(author$author_name)

# load in word-topic matrix
wtt_data <- read.csv("./socsci_shiny/term_topics.csv", stringsAsFactors = F)
names <- colnames(wtt_data)
names[1] <- "word"
colnames(wtt_data) <- names

# create topic names using first five words
name <- data.frame(matrix("a",ncol(wtt_data)-1,1), stringsAsFactors = F)
colnames(name) <- "topic_name"

for (i in 2:ncol(wtt_data)){
  temp <- order(-wtt_data[,i])
  temp2 <- wtt_data[temp,1]
  name$topic_name[i-1] <- paste(temp2[1:5], collapse = " + ")
}

# rename topics
colnames(wtt_data) <- c("word",name$topic_name)
colnames(author) <- c("author_name",name$topic_name)
```

Create static networks
----------------------

In the first step, I decide to create network edges (links) for only topics that have at least 20% correlation in the word probabilities. This goal is to remove correlations that may be due to randomness. Note however this threshold was chosen with trial and error and not through some optimization process.

``` r
cor_threshold <- .2

cor_mat <- cor(wtt_data[,2:ncol(wtt_data)])
cor_mat[ cor_mat < cor_threshold ] <- 0
diag(cor_mat) <- 0
```

Next, we use the correlation matrix to create an igraph data structure, removing all edges that have less than the 20% minimum threshold correlation.

``` r
library(igraph)

graph <- graph.adjacency(cor_mat, weighted=TRUE, mode="lower")
graph <- delete.edges(graph, E(graph)[ weight < cor_threshold])

E(graph)$edge.width <- E(graph)$weight
V(graph)$label <- paste(1:100)
```

Let's plot a simple igraph network.

``` r
par(mar=c(0, 0, 3, 0))
set.seed(110)
plot.igraph(graph, edge.width = E(graph)$edge.width, 
            edge.color = "blue", vertex.color = "white", vertex.size = 1,
            vertex.frame.color = NA, vertex.label.color = "grey30")
title("Strength Between Topics Based On Word Probabilities", cex.main=.8)
```

![](/images/unnamed-chunk-4-1.png)

Each number is the topic number. There looks like there are three main clusters.

Let's use community detection to determine clusters within the network.

``` r
clp <- cluster_label_prop(graph)
class(clp)

plot(clp, graph, edge.width = E(graph)$edge.width, vertex.size = 2, vertex.label = "")
title("Community Detection in Topic Network", cex.main=.8)
```

![](/images/unnamed-chunk-5-1.png)

Community detection found seven communites, plus five additional communities for each of the five isolated topics (i.e., topics that do not have any connections).

Similar to initial observation, the algorithm found the three main clusters we recognized in the first plot, but also added four smaller clusters that don't seem to fit well in any of the three main clusters.

Let's save our communities and also calculate betweenness which we'll use in the next section.

``` r
V(graph)$community <- clp$membership
V(graph)$betweenness <- betweenness(graph, v = V(graph), directed = F)
V(graph)$degree <- degree(graph, v = V(graph))
```

Dynamic Visualizations
----------------------

This section, we'll use the [`visNetwork`](http://datastorm-open.github.io/visNetwork/) package that allows interactive network graphs in R.

First, let's call the library and run `visIgraph` that runs an interactive network but on an igraph structure (graph) using igraph graph settings.

``` r
library(visNetwork)

visIgraph(graph)
```

{% include visIgraph-network.html %}

This is a good start, but we need more details about the network.

Let's go a different route by creating visNetwork data structure. To do this, we convert our igraph structure into a visNetwork data structure, then separating the list into two dataframes: nodes and edges.

``` r
data <- toVisNetworkData(graph)
nodes <- data[[1]]
edges <- data[[2]]
```

Delete nodes (topics) that don't have a connection (degree = 0).

``` r
nodes <- nodes[nodes$degree != 0,]
```

Let's add colors and other network parameters to improve our network.

``` r
library(RColorBrewer)

col <- brewer.pal(12, "Set3")[as.factor(nodes$community)]
nodes$shape <- "dot"  
nodes$shadow <- TRUE # Nodes will drop shadow
nodes$title <- nodes$id # Text on click
nodes$size <- ((nodes$betweenness / max(nodes$betweenness))+.2)*20 # Node size
nodes$borderWidth <- 2 # Node border width
nodes$color.background <- col
nodes$color.border <- "black"
nodes$color.highlight.background <- "orange"
nodes$color.highlight.border <- "darkred"
edges$title <- round(edges$edge.width,3)
```

Finally, let's create our network with an interactive plot. 

``` r
visNetwork(nodes, edges) %>% 
    visOptions(highlightNearest = TRUE, selectedBy = "community", nodesIdSelection = TRUE)
```

{% include topic-network.html %}

There are two dropdown menus. The first dropdown allows you to find any of the topics by name (top five words by word probability).

The second dropdown highlights the communities detected in our algorithm. Play around with this menu. Using the topic names (zoom in with mouse scroll), can you interpret what the topic community seem to be?

The three largest seems to be technology/computing, social issues and physical health issues. What's unique about the smaller communities that are detected?
