---
layout: single
excerpt: "Visualizing LDA Topic Correlation Network"
title:  "Using R to detect communities of correlated Topics"
categories: [text mining]
tags: [visualization, lda, community detection, social network]
---

![](/images/unnamed-chunk-5-1.png)

Creating a topic network
------------------------

For [Project Mosaic](http://projectmosaic.uncc.edu), I'm researching UNCC publications in social science and computing & informatics by analyzing the abstract text and the co-authorship social network.

For text mining, I'm running topic modeling ([Latent Dirichlet Allocation](https://en.wikipedia.org/wiki/Latent_Dirichlet_allocation) or LDA for short) on five years of peer-reviewed publication abstracts to identify key research themes by university researchers. (If you're not familiar with LDA, please review documents from [Tyler Rinker's Topic Modeling Repo].)

One problem I came across was: how to measure the relationships (correlations) between topics? In particular, I want use network science to measure creating a network that connects similar topics.

In this tutorial, I accomplish this by combining code provided in:

*   [Tyler Rinker's Topic Modeling Repo](https://github.com/trinker/topicmodels_learning) (see the section "Network of the Word Distributions Over Topics") 
*   [Katherine Ognyanova's phenomenal Network Visualization Tutorial](http://kateto.net/network-visualization).

Data preparation
----------------

Our first step is to load our topic matrices that are outputs of LDA. There are two outputs to LDA: a word-topic matrix and a document-topic matrix. To simplify it, let's load a csv from previously saved 

Unlike standard LDA in which the abstracts were the documents, I ran an "author-centered" LDA in which all author's abstracts were combined and treated as one document per author. I ran this because our ultimate goal is to use topic modeling as an information retrieval process to determine researcher expertise by topic.

As an alternative to loading flat files, you can use the output of the `topicmodels` package `lda` function to create any word-topic and document-topic matrices. Take the output of your `lda` function and run the `posterior` function on the output.

``` r
# load in author-topic matrix, first column is word
author.topic <- read.csv("./author_topics.csv", stringsAsFactors = F)

# load in word-topic matrix, first column is word
word.topic <- read.csv("./term_topics.csv", stringsAsFactors = F)
num.col <- ncol(word.topics)

# create topic names using first five words
for (i in 2:num.col){
  top.words <- word.topics[order(-word.topic[,i])]
  name$topic_name[i] <- paste(top.words[1:5], collapse = " + ")
}

# rename topics
colnames(word.topic) <- c("word",name$topic_name)
colnames(author.topic) <- c("author_name",name$topic_name)
```

Create static networks
----------------------

In the next step, I create a network using the correlation between each topic's word probabilities. 

First, I decide to keep only relationships (edges) that have significant correlation (20%+ correlation). I use 20% because its the .05 level of statistical significance for a sample of 100 observations [Wikipedia](https://commons.wikimedia.org/wiki/File:Correlation_significance.svg#/media/File:Correlation_significance.svg).



``` r
cor_threshold <- .2

cor_mat <- cor(word.topic[,2:num.col])
cor_mat[ cor_mat < cor_threshold ] <- 0
diag(cor_mat) <- 0
```

Next, we use the correlation matrix to create an igraph data structure, removing all edges that have less than the 20% minimum threshold correlation.

``` r
library(igraph)

graph <- graph.adjacency(cor_mat, weighted=TRUE, mode="lower")

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

Each number is the topic number. My first observation is that there looks like there are three main clusters.

Let's use [community detection](http://igraph.wikidot.com/community-detection-in-r), specifically the label propagation algorithm in igraph, to determine clusters within the network.

``` r
clp <- cluster_label_prop(graph)
class(clp)

plot(clp, graph, edge.width = E(graph)$edge.width, vertex.size = 2, vertex.label = "")
title("Community Detection in Topic Network", cex.main=.8)
```

![](/images/unnamed-chunk-5-1.png)

Community detection found thirteen communites, plus multiple additional communities for each of the isolated topics (i.e., topics that do not have any connections).

Similar to my initial observation, the algorithm found the three main clusters we recognized in the first plot, but also added additional smaller clusters that don't seem to fit well in any of the three main clusters.

Let's save our communities and also calculate degree centrality and betweenness which we'll use in the next section.

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

Finally, let's create our network with an interactive plot. You can zoom by using your mouse scroll wheel.

The size of the bubble's is based on the network (centrality) measure **betweenness**. This measures how important that node is to the entire network's connectivity. In this example, larger nodes have a high betweenness, which implies the topic is more important in crossing across topic clusters.

Find the largest nodes: these topics are the "glue" that keeps the network connected.

``` r
visNetwork(nodes, edges) %>% 
    visOptions(highlightNearest = TRUE, selectedBy = "community", nodesIdSelection = TRUE)
```

{% include topic-network.html %}

There are two dropdown menus. The first dropdown allows you to find any of the topics by name (top five words by word probability).

The second dropdown highlights the communities detected in our algorithm. Play around with this menu. Using the topic names (zoom in with mouse scroll), can you interpret what the topic community seem to be?

The three largest seems to be: 
1.  Computing (gray, cluster 4)
2.  Social (green-blue, cluster 1)
3.  Health (yellow, cluster 2)

What's unique about the smaller communities that are detected? Can you interpret them?
