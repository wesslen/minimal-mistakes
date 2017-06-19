---
layout: single
title:  "Binary Text Classification with PySpark"
excerpt: "Predicting profile location on Twitter with PySpark"
categories: [twitter]
tags: [twitter, pyspark, supervised-machine-learning]
---

## Introduction

### Overview

Recently, I've been studying tweets relating to the September 2016 Charlotte Protests. In this example, I predict users with Charlotte-area profile terms using the tweet content. For my dataset, I used two days of tweets following a local courts decision not to press charges on the officer. While not part of the protests, this is part of my research to study continued activation on "hot" issues like the Charlotte Protests. The dataset I used includes all tweets between Nov 30 - Dec 1, 2016 that include (at least) one of seven protest-related hashtags which include: 

* #keithlamontscott
* #keithscott
* #charlotteprotest
* #charlotteriots

My question: do these Twitter users with Charlotte-ties (as identified by their profile location) have different Twitter content than users without Charlotte-ties? 

If true, I should be able to predict location (ties) with their tweets. In this post, I'll use a variety of feature extraction technique along with different supervised machine learning algorithms in Spark.

### Setup

First, to run pyspark and Jupyter, I used Docker to set up [this pyspark-Jupyter Docker container](https://github.com/jupyter/docker-stacks/tree/master/pyspark-notebook). If you need Docker, go to this [website](https://docs.docker.com/engine/installation/) and install the Community Edition. 

Alternatively, another option is to go to [play-with-docker.com](http://play-with-docker.com). This allows you (FOR FREE!) to run a docker session with multiple nodes; the only downside is that every four hours the session restarts, meaning you lose your data. Therefore, this isn't ideal for long term projects, but great for learning. I'd recommend newer users try this with one instance. Note: each container is limited to 4GB of memory, most of which is taken by the jupyter-pyspark container. I tried to run and simple file and was able to but likely you would need more than one node (instance). That will require additional settings for Docker Swarm mode. If I get it to run, I may create another tutorial.

If instead you were able to follow the instructions and downloaded Docker locally, you can run the jupyter-pyspark container. TO run it, some basic knowledge of Docker is ideal, but you can run the container in the command line with the code in Ubuntu:

`$ docker run -it --rm -p 8888:8888 jupyter/pyspark-notebook`

If it works, you can then view your Jupyter notebook in localhost:8888 on your browser. 

### Models

For this demo, I'm going to use relatively common supervised machine learning algorithms with a variety of different features:

* Regularized Logistic Regression (Count Vectors, TF-IDF, and Word2Vec Features)
* Naive Bayes (Count Vector Features)
* Random Forest (Count Vector Features)
* Gradient Boosting Machines (GBM) (Count Vector Features)
    
As you will see, these model do a good job of fitting the data but don't provide a lot of on why (explainability). Some of this is due to the nature of the models, and some of this is due to my limited knowledge of Spark (e.g., how to export Ridge Regression words coefficients from the Spark DataFrames). My goal in these examples is to maximize prediction, which I'm going to arbitrarily decide to use out-of-sample Area Under the ROC Curve (AUC). I could consider other measures (e.g., Accuracy, Precision, F-Measure, etc.) but I'm simply using AUC as a proxy.

Ideally sometime soon, I want to redo my results but using Spark's new [Deep Learning](https://github.com/databricks/spark-deep-learning) model extensions.

## Data Ingestion & Extraction

### Setup Spark

For this exercise, I'm simply running Spark locally. Running Spark locally is a good start, but not the ideal way to run Spark as its better when used as a cluster of computers (nodes) to harness its distributed power. Nevertheless, running in local model is helpful as this code is scalable if I use a cluster and want to greatly scale up the size of my dataset.


```python
from pyspark.sql import SparkSession

spark = SparkSession \
    .builder \
    .master('local[*]') \
    .appName("Python Text Classification") \
    .config("spark.executor.memory", "4g") \
    .getOrCreate()
```


```python
#set input and output directories
jobDir = "protesttrial.json"
tweets = spark.read.json([jobDir])

tweets.count() # Count the number of tweets in the RDD
```




    59692




```python
#drops any blank records -- just in case
tweets = tweets.na.drop(subset=["id"])
tweets.count() 
```




    59435




```python
# Let's simplify to only six fields.
tweets = tweets.select("id", \
                       "body", \
                       "verb", \
                       "postedTime", \
                       "actor.preferredUsername", \
                       "actor.followersCount", \
                       "actor.location.displayName")

tweets.printSchema()
```

    root
     |-- id: string (nullable = true)
     |-- body: string (nullable = true)
     |-- verb: string (nullable = true)
     |-- postedTime: string (nullable = true)
     |-- preferredUsername: string (nullable = true)
     |-- followersCount: long (nullable = true)
     |-- displayName: string (nullable = true)
    



```python
# Retweets (share) vs Original Content Posts (post)
tweets.groupBy("verb").count().show()
```

    +-----+-----+
    | verb|count|
    +-----+-----+
    | post|11334|
    |share|48101|
    +-----+-----+
    



```python
# Tweets by users with more than 1MM Followers Ordered by Tweet Time (postedTime)
tweets.filter(tweets['followersCount'] > 1000000) \
    .select("preferredUsername","body","postedTime","followersCount") \
    .orderBy("postedTime") \
    .show(truncate=25)
```

    +-----------------+-------------------------+------------------------+--------------+
    |preferredUsername|                     body|              postedTime|followersCount|
    +-----------------+-------------------------+------------------------+--------------+
    |              CNN|We're watching officia...|2016-11-30T15:53:00.000Z|      29868772|
    |              CNN|BREAKING: Officer who ...|2016-11-30T16:32:05.000Z|      29869544|
    |      katiecouric|RT @CNN: BREAKING: Off...|2016-11-30T16:32:47.000Z|       1619095|
    |           RT_com|RT @RT_America: Protes...|2016-11-30T16:37:54.000Z|       2439640|
    |     YourAnonNews|RT @Jasmyne: No charge...|2016-11-30T17:06:34.000Z|       1661335|
    |              NPR|District attorney R. A...|2016-11-30T17:13:00.000Z|       6373550|
    |      anamariecox|NC lawyer and #nevertr...|2016-11-30T17:17:07.000Z|       1318342|
    |         USATODAY|RT @jmbacon: Family of...|2016-11-30T18:08:45.000Z|       2835811|
    |      ERNESTZorro|RT @maramcewin: Americ...|2016-11-30T18:22:01.000Z|       2450013|
    |            Slate|N.C. officer who shot,...|2016-11-30T18:35:22.000Z|       1637214|
    |           BigBoi|#KeithLamontScott http...|2016-11-30T21:57:01.000Z|       1217184|
    |          amnesty|Unless laws on lethal ...|2016-11-30T22:08:02.000Z|       2773347|
    |          AppSame|RT @JaredWyand: Shouto...|2016-12-01T00:16:42.000Z|       1105830|
    |   samantharonson|RT @xmanwalton: Anothe...|2016-12-01T01:54:53.000Z|       1497758|
    +-----------------+-------------------------+------------------------+--------------+
    



```python
# Tweets by profile location
from pyspark.sql.functions import col

# by top 20 locations
tweets.groupBy("displayName") \
    .count() \
    .orderBy(col("count").desc()) \
    .show()
```

    +-------------------+-----+
    |        displayName|count|
    +-------------------+-----+
    |               null|17532|
    |      United States| 2104|
    |      Charlotte, NC| 2078|
    |                USA|  641|
    |    California, USA|  552|
    |     Washington, DC|  347|
    |North Carolina, USA|  317|
    |       Florida, USA|  299|
    |       New York, NY|  296|
    |         Texas, USA|  288|
    |         California|  235|
    |     York_County_SC|  227|
    |        Atlanta, GA|  221|
    |           New York|  201|
    |    Los Angeles, CA|  200|
    |        Chicago, IL|  194|
    |       Brooklyn, NY|  184|
    |    Charlotte, N.C.|  180|
    |      New York, USA|  160|
    |       Georgia, USA|  155|
    +-------------------+-----+
    only showing top 20 rows
    



```python
#drop retweets
tweets = tweets.filter(tweets["verb"]=="post")
tweets.count()
```




    11334




```python
#save null locations -- we'll predict them once we select a model
tweetsMissing = tweets.where(col("displayName").isNull())
                             
#drop null locations
tweets = tweets.na.drop(subset=["displayName"])
tweets.count()
```




    8642




```python
tweets.groupBy("displayName") \
    .count() \
    .orderBy(col("count").desc()) \
    .show(n=50, truncate=40)
```

    +-----------------------------+-----+
    |                  displayName|count|
    +-----------------------------+-----+
    |                Charlotte, NC| 1044|
    |                United States|  679|
    |               York_County_SC|  227|
    |                          USA|  148|
    |               Washington, DC|  118|
    |                 New York, NY|  110|
    |              California, USA|   78|
    |    www.ChristinaMWatkins.com|   68|
    |          North Carolina, USA|   67|
    |                  Atlanta, GA|   63|
    |               Charleston, SC|   61|
    |           Virgina Beach. Va.|   56|
    |                   California|   56|
    |                 Columbia, SC|   54|
    |                     New York|   53|
    |                  Detroit, MI|   53|
    |              Palm Harbor, FL|   50|
    |                   Queen City|   50|
    |                     Universe|   48|
    |                      Raleigh|   46|
    |                    Charlotte|   46|
    |                  Chicago, IL|   44|
    |                 Brooklyn, NY|   42|
    |                   Wonderland|   41|
    |               No #RWNJ Zone!|   39|
    |                 Nashville TN|   38|
    |               Southeast, USA|   36|
    |                 Florida, USA|   36|
    |              Los Angeles, CA|   32|
    |                  Los Angeles|   32|
    |          South Carolina, USA|   32|
    |               North Carolina|   31|
    |            Winston-Salem, NC|   31|
    |              Charlotte, N.C.|   30|
    |                New York, USA|   29|
    |               Greensboro, NC|   29|
    |                Baltimore, MD|   27|
    |          Sunny South Florida|   27|
    |    Charlotte, North Carolina|   27|
    |                      Chicago|   25|
    |                   Texas, USA|   24|
    |                  Phoenix, AZ|   24|
    |Charlotte & surrounding areas|   24|
    |              New Jersey, USA|   23|
    |                  Raleigh, NC|   23|
    |                    New York |   21|
    |                 Portland, OR|   21|
    |                      Midwest|   21|
    |            Pennsylvania, USA|   20|
    |                   Boston, MA|   20|
    +-----------------------------+-----+
    only showing top 50 rows
    



```python
from pyspark.sql.functions import when

# manually identified these prominent terms -- note the 3rd is a known Charlotte reporter
charlotte_terms = ["Queen City","York_County_SC","www.ChristinaMWatkins.com"]

# Create supervised labels where 0 = charlotte ties, 1 = non-charlotte ties
tweets = tweets.withColumn("label", \
                           (when(col("displayName").like("%Charlotte%"), 0) \
                           .when(col("displayName").like("%Davidson%"), 0) \
                           .when(col("displayName").like("%Huntersville%"), 0) \
                           .when(col("displayName").like("%Matthews%"), 0) \
                           .when(col("displayName").isin(charlotte_terms), 0) \
                           .otherwise(1)))
```


```python
# Let's explore the top terms to make sure we didn't accidentally create false positives
tweets.filter(tweets['label']==0).groupBy("displayName") \
    .count() \
    .orderBy(col("count").desc()) \
    .show(n=50, truncate=40)
```

    +------------------------------+-----+
    |                   displayName|count|
    +------------------------------+-----+
    |                 Charlotte, NC| 1044|
    |                York_County_SC|  227|
    |     www.ChristinaMWatkins.com|   68|
    |                    Queen City|   50|
    |                     Charlotte|   46|
    |               Charlotte, N.C.|   30|
    |     Charlotte, North Carolina|   27|
    | Charlotte & surrounding areas|   24|
    |       Sleepless in Charlotte |   14|
    |                  Charlotte NC|   11|
    |                Charlotte, NC |    9|
    |                  Davidson, NC|    9|
    |            Charlotte, NC, USA|    6|
    |                  Charlotte,NC|    5|
    |             Charlotte, NC USA|    4|
    |      Charlotte North Carolina|    3|
    |     the borderlands/Charlotte|    3|
    |              Huntersville, NC|    2|
    |                  #CharlotteNC|    2|
    |                  Matthews, NC|    2|
    |     Charlotte, NC/Atlanta, GA|    2|
    |                Charlotte, USA|    2|
    |       Tuscaloosa to Charlotte|    2|
    |     Charlotte, NC (Buzz City)|    2|
    |    Richmond/Charlotte/Raleigh|    1|
    |      Hampton,VA ✈Charlotte,NC|    1|
    |Charlotte NC (from Buffalo NY)|    1|
    |                    Charlotte |    1|
    |  Charlotte and Greensboro, NC|    1|
    |       Cincinnati to Charlotte|    1|
    |       Charlotte, the other NC|    1|
    |              Charlotte/Global|    1|
    | Downtown Charlotte, Charlotte|    1|
    |           SC-GA-Charlotte, NC|    1|
    +------------------------------+-----+
    


### Model Pipeline

I'm going to use Spark's new Pipeline that streamlines feature extraction.

Our pipeline includes three steps:

1. `regexTokenizer`: Tokenization (with Regular Expression)
2. `stopwordsRemover`: Stop Words Removal
3. `countVectors`: Count vectors ("document-term vectors")


```python
from pyspark.ml.feature import RegexTokenizer, StopWordsRemover, CountVectorizer
from pyspark.ml.classification import LogisticRegression

# regular expression tokenizer
regexTokenizer = RegexTokenizer(inputCol="body", outputCol="words", pattern="\\W")

# stop words
add_stopwords = ["http","https","amp","rt","t","c","can", # standard stop words
     "#keithlamontscott","#charlotteprotest","#charlotteriots","#keithscott"] # keywords used to pull data)
stopwordsRemover = StopWordsRemover(inputCol="words", outputCol="filtered").setStopWords(add_stopwords)

# bag of words count
countVectors = CountVectorizer(inputCol="filtered", outputCol="features", vocabSize=10000, minDF=5)
```


```python
from pyspark.ml import Pipeline

pipeline = Pipeline(stages=[regexTokenizer, stopwordsRemover, countVectors])

# Fit the pipeline to training documents.
pipelineFit = pipeline.fit(tweets)
dataset = pipelineFit.transform(tweets)
```

### Partition Training & Test

Let's create partition our data into training (70%) and test (30%) datasets.


```python
### Randomly split data into training and test sets. set seed for reproducibility
(trainingData, testData) = dataset.randomSplit([0.7, 0.3], seed = 100)
print("Training Dataset Count: " + str(trainingData.count()))
print("Test Dataset Count: " + str(testData.count()))
```

    Training Dataset Count: 6079
    Test Dataset Count: 2563


### Model Training

Let's train our model.


```python
# Build the model
lr = LogisticRegression(maxIter=20, regParam=0.3, elasticNetParam=0, family = "binomial")

# Train model with Training Data
lrModel = lr.fit(trainingData)
```


```python
import matplotlib.pyplot as plt
import numpy as np

beta = np.sort(lrModel.coefficients)

plt.plot(beta)
plt.ylabel('Beta Coefficients')
plt.show()
```


![png](/images/output_20_0.png)



```python
# Extract the summary from the returned LogisticRegressionModel instance trained
trainingSummary = lrModel.summary

# Obtain the objective per iteration
objectiveHistory = trainingSummary.objectiveHistory
plt.plot(objectiveHistory)
plt.ylabel('Objective Function')
plt.xlabel('Iteration')
plt.show()
```


![png](/images/output_21_0.png)



```python
# Obtain the receiver-operating characteristic as a dataframe and areaUnderROC.
print("areaUnderROC: " + str(trainingSummary.areaUnderROC))

#trainingSummary.roc.show(n=10, truncate=15)
roc = trainingSummary.roc.toPandas()
plt.plot(roc['FPR'],roc['TPR'])
plt.ylabel('False Positive Rate')
plt.xlabel('True Positive Rate')
plt.title('ROC Curve')
plt.show()
```

    areaUnderROC: 0.96720586084



![png](/images/output_22_1.png)



```python
pr = trainingSummary.pr.toPandas()
plt.plot(pr['recall'],pr['precision'])
plt.ylabel('Precision')
plt.xlabel('Recall')
plt.show()
```


![png](/images/output_23_0.png)



```python
# Set the model threshold to maximize F-Measure
#trainingSummary.fMeasureByThreshold.show(n=10, truncate = 15)
f = trainingSummary.fMeasureByThreshold.toPandas()
plt.plot(f['threshold'],f['F-Measure'])
plt.ylabel('F-Measure')
plt.xlabel('Threshold')
plt.show()
```


![png](/images/output_24_0.png)


### Score Test Documents

Let's run our test documents through our feature extraction pipeline and then transform (run) through the model.


```python
# Make predictions on test data using the transform() method.
# LogisticRegression.transform() will only use the 'features' column.
predictions = lrModel.transform(testData)

predictions.select("body","probability").show(n=10, truncate=40)
```

    +----------------------------------------+----------------------------------------+
    |                                    body|                             probability|
    +----------------------------------------+----------------------------------------+
    |#injusticesystem Donate to support .@...| [0.1214021905581726,0.8785978094418273]|
    |Please pray for Charlotte, NC, as we ...| [0.3419098084188267,0.6580901915811733]|
    |10 on @WCCBCharlotte I spoke w/Chief ...|[0.5728183977432763,0.42718160225672375]|
    |TOMORROW:The indictment decision for ...| [0.3728295844866397,0.6271704155133603]|
    |RT cjfranciscowccb: 10 on WCCBCharlot...| [0.5347843023033392,0.4652156976966609]|
    |AT 10:05 DavidFox46 has the latest on...| [0.6880161316423431,0.3119838683576569]|
    |#keithlamontscott (cont.) https://t.c...| [0.09405363639082305,0.905946363609177]|
    |[VIDEO] Protests planned for Wednesda...| [0.4496537510428295,0.5503462489571705]|
    |Police assert
    with deadly force
    that ...|[0.052296912105353685,0.9477030878946...|
    |RT josephjett "Police assert
    with dea...|[0.052296912105353685,0.9477030878946...|
    +----------------------------------------+----------------------------------------+
    only showing top 10 rows
    



```python
predictions.filter(predictions['prediction'] == 0) \
    .select("body","displayName","probability","label","prediction") \
    .orderBy("probability", ascending=False) \
    .show(n = 20, truncate = 30)
```

    +------------------------------+-------------------------+------------------------------+-----+----------+
    |                          body|              displayName|                   probability|label|prediction|
    +------------------------------+-------------------------+------------------------------+-----+----------+
    |DA Murray says #KeithScott’...|            Charlotte, NC|[0.9652270012069905,0.03477...|    0|       0.0|
    |However @CharMeckDA says on...|               Queen City|[0.9577215245472008,0.04227...|    0|       0.0|
    |@CharMeckDA says bulge over...|               Queen City|[0.9523810511057281,0.04761...|    0|       0.0|
    |.@CMPD blocking entrance to...|            Charlotte, NC|[0.9501652981878491,0.04983...|    0|       0.0|
    |Watch DA Murray speak about...|            Charlotte, NC|[0.9475885007353635,0.05241...|    0|       0.0|
    |We're LIVE now with team co...|            Charlotte, NC|[0.9458853628926766,0.05411...|    0|       0.0|
    |@CharMeckDA now showing por...|               Queen City|[0.9422595545329445,0.05774...|    0|       0.0|
    |Capt w/@CMPD on arrests dur...|            Charlotte, NC|[0.9378295285817511,0.06217...|    0|       0.0|
    |DA Murray shows video of in...|            Charlotte, NC|[0.9292540508031979,0.07074...|    0|       0.0|
    |#KeithScott family attorney...|            Charlotte, NC|[0.9282903068542293,0.07170...|    0|       0.0|
    |RT ColeenHarryWBTV: #keithl...|           York_County_SC|[0.9185572469551597,0.08144...|    0|       0.0|
    |RT amywccb: Protestors conf...|           York_County_SC|[0.9181106081942706,0.08188...|    0|       0.0|
    |.@CharMeckDA says Ofc. Vins...|www.ChristinaMWatkins.com|[0.9028706124350627,0.09712...|    0|       0.0|
    |Protesters stopped Omni Hot...|            Charlotte, NC|[0.8969184952007492,0.10308...|    0|       0.0|
    |RT BrigidaMack: CharMeckDA ...|           York_County_SC|[0.8921136879326332,0.10788...|    0|       0.0|
    |WATCH LIVE: Protesters marc...|        Charlotte, NC USA|[0.8887059955397277,0.11129...|    0|       0.0|
    |MURRAY addressing claims th...|                  England|[0.8834255556524199,0.11657...|    1|       0.0|
    |#keithlamontscott protester...|            Charlotte, NC|[0.8833350078178805,0.11666...|    0|       0.0|
    |@CharMeckDA speaking LIVE n...|               Queen City|[0.8817121885368986,0.11828...|    0|       0.0|
    |.@CharMeckDA now showing pa...|www.ChristinaMWatkins.com|[0.8754409365081633,0.12455...|    0|       0.0|
    +------------------------------+-------------------------+------------------------------+-----+----------+
    only showing top 20 rows
    



```python
from pyspark.ml.evaluation import BinaryClassificationEvaluator
print("Training: Area Under ROC: " + str(trainingSummary.areaUnderROC))

# Evaluate model
evaluator = BinaryClassificationEvaluator(rawPredictionCol="rawPrediction")
print("Test: Area Under ROC: " + str(evaluator.evaluate(predictions, {evaluator.metricName: "areaUnderROC"})))
```

    Training: Area Under ROC: 0.96720586084
    Test: Area Under ROC: 0.905134984474


### Logistic Regression using TF-IDF Features


```python
from pyspark.ml.feature import HashingTF, IDF

# Add HashingTF and IDF to transformation
hashingTF = HashingTF(inputCol="filtered", outputCol="rawFeatures", numFeatures=10000)
idf = IDF(inputCol="rawFeatures", outputCol="features", minDocFreq=5) #minDocFreq: remove sparse terms

# Redo Pipeline
pipeline = Pipeline(stages=[regexTokenizer, stopwordsRemover, hashingTF, idf])
```


```python
# Fit the pipeline to training documents.
pipelineFit = pipeline.fit(tweets)
dataset = pipelineFit.transform(tweets)

### Randomly split data into training and test sets. set seed for reproducibility
(trainingData, testData) = dataset.randomSplit([0.7, 0.3], seed = 100)

# Build the model
lr = LogisticRegression(maxIter=20, regParam=0.3, elasticNetParam=0, family = "binomial")

# Train model with Training Data
lrModel = lr.fit(trainingData)
```


```python
# Run Results on Training Data
trainingSummary = lrModel.summary
print("Training: Area Under ROC: " + str(trainingSummary.areaUnderROC))

# Predict on Test Data
predictions = lrModel.transform(testData)
# Evaluate model
evaluator = BinaryClassificationEvaluator(rawPredictionCol="rawPrediction")

print("Test: Area Under ROC: " + str(evaluator.evaluate(predictions, {evaluator.metricName: "areaUnderROC"})))
```

    Training: Area Under ROC: 0.96992751078
    Test: Area Under ROC: 0.903323283595


### Logistic Regression using Word2Vec features


```python
from pyspark.ml.feature import Word2Vec

# Learn a mapping from words to Vectors.
word2Vec = Word2Vec(vectorSize=1000, minCount=5, inputCol="filtered", outputCol="features")

# Redo Pipeline
pipeline = Pipeline(stages=[regexTokenizer, stopwordsRemover, word2Vec])
```


```python
# Fit the pipeline to training documents.
pipelineFit = pipeline.fit(tweets)
dataset = pipelineFit.transform(tweets)

### Randomly split data into training and test sets. set seed for reproducibility
(trainingData, testData) = dataset.randomSplit([0.7, 0.3], seed = 100)

# Build the model
lr = LogisticRegression(maxIter=20, regParam=0.3, elasticNetParam=0, family = "binomial")

# Train model with Training Data
lrModel = lr.fit(trainingData)
```


```python
# Run Results on Training Data
trainingSummary = lrModel.summary
print("Training: Area Under ROC: " + str(trainingSummary.areaUnderROC))

# Predict on Test Data
predictions = lrModel.transform(testData)
# Evaluate model
evaluator = BinaryClassificationEvaluator(rawPredictionCol="rawPrediction")
print("Test: Area Under ROC: " + str(evaluator.evaluate(predictions, {evaluator.metricName: "areaUnderROC"})))
```

    Training: Area Under ROC: 0.837212020197
    Test: Area Under ROC: 0.820881079405


### Cross-Validation

Let's now try cross-validation to tune our hyper parameters. Given it had the highest out-of-sample performance, let's only tune the count vectors Logistic Regression.


```python
pipeline = Pipeline(stages=[regexTokenizer, stopwordsRemover, countVectors])

pipelineFit = pipeline.fit(tweets)
dataset = pipelineFit.transform(tweets)
(trainingData, testData) = dataset.randomSplit([0.7, 0.3], seed = 100)

# Build the model
lr = LogisticRegression(maxIter=20, regParam=0.3, elasticNetParam=0, family = "binomial")
```


```python
from pyspark.ml.tuning import ParamGridBuilder, CrossValidator

# Create ParamGrid for Cross Validation
paramGrid = (ParamGridBuilder()
             .addGrid(lr.regParam, [0.1, 0.3, 0.5]) # regularization parameter
             .addGrid(lr.elasticNetParam, [0.0, 0.1, 0.2]) # Elastic Net Parameter (Ridge = 0)
#            .addGrid(model.maxIter, [10, 20, 50]) #Number of iterations
#            .addGrid(idf.numFeatures, [10, 100, 1000]) # Number of features
             .build())

# Create 5-fold CrossValidator
cv = CrossValidator(estimator=lr, \
                    estimatorParamMaps=paramGrid, \
                    evaluator=evaluator, \
                    numFolds=5)

# Run cross validations
cvModel = cv.fit(trainingData)
# this will likely take a fair amount of time because of the amount of models that we're creating and testing

# Use test set here so we can measure the accuracy of our model on new data
predictions = cvModel.transform(testData)

# cvModel uses the best model found from the Cross Validation
# Evaluate best model
print("Test: Area Under ROC: " + str(evaluator.evaluate(predictions, {evaluator.metricName: "areaUnderROC"})))
```

    Test: Area Under ROC: 0.907316680603


### Naive Bayes

I can also run Naive Bayes. A key input for Naive Bayes is its smoothing parameter. I'll keep it to its default value of 1.


```python
from pyspark.ml.classification import NaiveBayes

# create the trainer and set its parameters
nb = NaiveBayes(smoothing=1, modelType="multinomial")

# train the model
model = nb.fit(trainingData)
```


```python
# select example rows to display.
predictions = model.transform(testData)
predictions.filter(predictions['prediction'] == 0) \
    .select("body","displayName","probability","label","prediction") \
    .orderBy("probability", ascending=False) \
    .show(n = 20, truncate = 30)

# compute accuracy on the test set
evaluator = BinaryClassificationEvaluator(rawPredictionCol="prediction")
print("Test: Area Under ROC: " + str(evaluator.evaluate(predictions, {evaluator.metricName: "areaUnderROC"})))
```

    +------------------------------+-----------------+------------------------------+-----+----------+
    |                          body|      displayName|                   probability|label|prediction|
    +------------------------------+-----------------+------------------------------+-----+----------+
    |Watch DA Murray speak about...|    Charlotte, NC|[0.9999999999999469,5.30948...|    0|       0.0|
    |#KeithScott family attorney...|    Charlotte, NC|[0.9999999999999163,8.37280...|    0|       0.0|
    |DA Murray says #KeithScott’...|    Charlotte, NC|[0.9999999999998068,1.93243...|    0|       0.0|
    |DA Murray shows video of in...|    Charlotte, NC|[0.9999999999835001,1.64999...|    0|       0.0|
    |DA Murray: #KeithScott shot...|   York_County_SC|[0.9999999999441831,5.58168...|    0|       0.0|
    |WATCH LIVE: Protesters marc...|Charlotte, NC USA|[0.9999999999247515,7.52485...|    0|       0.0|
    |DA Murray: One round was in...|    Charlotte, NC|[0.9999999989070625,1.09293...|    0|       0.0|
    |We're LIVE now with team co...|    Charlotte, NC|[0.999999998684205,1.315794...|    0|       0.0|
    |#KeithScott family attorney...|    Charlotte, NC|[0.9999999986756325,1.32436...|    0|       0.0|
    |.@CMPD blocking entrance to...|    Charlotte, NC|[0.9999999985280568,1.47194...|    0|       0.0|
    |However @CharMeckDA says on...|       Queen City|[0.9999999979692271,2.03077...|    0|       0.0|
    |RT amywccb: Protestors conf...|   York_County_SC|[0.9999999979554373,2.04456...|    0|       0.0|
    |@CharMeckDA now showing por...|       Queen City|[0.999999996913512,3.086488...|    0|       0.0|
    |Capt w/@CMPD on arrests dur...|    Charlotte, NC|[0.9999999943340963,5.66590...|    0|       0.0|
    |@CharMeckDA says bulge over...|       Queen City|[0.9999999929344734,7.06552...|    0|       0.0|
    |WATCH LIVE: DA holds news c...|    Charlotte, NC|[0.9999999924947994,7.50520...|    0|       0.0|
    |RT ColeenHarryWBTV: #keithl...|   York_County_SC|[0.9999999831082088,1.68917...|    0|       0.0|
    |Protesters stopped Omni Hot...|    Charlotte, NC|[0.9999999793794689,2.06205...|    0|       0.0|
    |RT BrigidaMack: CharMeckDA ...|   York_County_SC|[0.9999999718977858,2.81022...|    0|       0.0|
    |DA to holds news conference...|    Charlotte, NC|[0.9999999411213297,5.88786...|    0|       0.0|
    +------------------------------+-----------------+------------------------------+-----+----------+
    only showing top 20 rows
    
    Test: Area Under ROC: 0.828836467988


## Decision Tree and Ensembles

First, let's try a simple decision tree.

### Decision Tree


```python
from pyspark.ml.classification import DecisionTreeClassifier

# Create initial Decision Tree Model
dt = DecisionTreeClassifier(labelCol="label", featuresCol="features", maxDepth=3)

# Train model with Training Data
dtModel = dt.fit(trainingData)
```


```python
print "numNodes = ", dtModel.numNodes
print "depth = ", dtModel.depth
```

    numNodes =  15
    depth =  3



```python
predictions = dtModel.transform(testData)

predictions.filter(predictions['prediction'] == 0) \
    .select("body","displayName","probability","label","prediction") \
    .orderBy("probability", ascending=False) \
    .show(n = 10, truncate = 30)
```

    +------------------------------+------------------------------+------------------------------+-----+----------+
    |                          body|                   displayName|                   probability|label|prediction|
    +------------------------------+------------------------------+------------------------------+-----+----------+
    |Murray said the decision be...|    Greensboro, North Carolina|[0.7741935483870968,0.22580...|    1|       0.0|
    |Murray says the dash cam vi...|                Charleston, SC|[0.7741935483870968,0.22580...|    1|       0.0|
    |Murray: prosecutors unanimo...|                 Charlotte, NC|[0.7741935483870968,0.22580...|    0|       0.0|
    |@MarkDice You obviously did...|Boycott Scion aka Trump hotels|[0.7741935483870968,0.22580...|    1|       0.0|
    |Murray now running through ...|                 Charlotte, NC|[0.7741935483870968,0.22580...|    0|       0.0|
    |. @CharMeckDA Murray says t...|                Washington, DC|[0.7741935483870968,0.22580...|    1|       0.0|
    |[MORE] We have news crew at...|                 Charlotte, NC|[0.7741935483870968,0.22580...|    0|       0.0|
    |Protestors gathered outside...|                 Charlotte, NC|[0.7741935483870968,0.22580...|    0|       0.0|
    |MURRAY addressing claims th...|                       England|[0.7741935483870968,0.22580...|    1|       0.0|
    |@WesleyLowery,DA Murray act...|                   SouthEast, |[0.7741935483870968,0.22580...|    1|       0.0|
    +------------------------------+------------------------------+------------------------------+-----+----------+
    only showing top 10 rows
    



```python
# Evaluate model
evaluator = BinaryClassificationEvaluator(rawPredictionCol="rawPrediction")
print("Test: Area Under ROC: " + str(evaluator.evaluate(predictions, {evaluator.metricName: "areaUnderROC"})))
```

    Test: Area Under ROC: 0.656772974055


This model performed very poorly out-of-sample -- likely because one decision tree is too weak given the range of different features. Instead, let's try many small decisions trees: Random Forest and Gradient Boosting Machine.

### Random Forest


```python
from pyspark.ml.classification import RandomForestClassifier

# Create an initial RandomForest model.
rf = RandomForestClassifier(labelCol="label", \
                            featuresCol="features", \
                            numTrees = 100, \
                            maxDepth = 4, \
                            maxBins = 32)

# Train model with Training Data
rfModel = rf.fit(trainingData)
```


```python
# Score test Data
predictions = rfModel.transform(testData)

predictions.filter(predictions['prediction'] == 0) \
    .select("body","displayName","probability","label","prediction") \
    .orderBy("probability", ascending=False) \
    .show(n = 10, truncate = 30)
```

    +------------------------------+-------------+------------------------------+-----+----------+
    |                          body|  displayName|                   probability|label|prediction|
    +------------------------------+-------------+------------------------------+-----+----------+
    |Watch DA Murray speak about...|Charlotte, NC|[0.5003846323286796,0.49961...|    0|       0.0|
    +------------------------------+-------------+------------------------------+-----+----------+
    



```python
# Evaluate model
evaluator = BinaryClassificationEvaluator(rawPredictionCol="rawPrediction")
print("Test: Area Under ROC: " + str(evaluator.evaluate(predictions, {evaluator.metricName: "areaUnderROC"})))
```

    Test: Area Under ROC: 0.876910824275



```python
# Create ParamGrid for Cross Validation
paramGrid = (ParamGridBuilder()
             .addGrid(rf.numTrees, [50, 100, 200]) # number of trees
             .addGrid(rf.maxDepth, [3, 4, 5]) # maximum depth
#            .addGrid(rf.maxBins, [24, 32, 40]) #Number of bins
             .build())

# Create 5-fold CrossValidator
cv = CrossValidator(estimator=rf, \
                    estimatorParamMaps=paramGrid, \
                    evaluator=evaluator, \
                    numFolds=5)

# Run cross validations
cvModel = cv.fit(trainingData)

# Use test set here so we can measure the accuracy of our model on new data
predictions = cvModel.transform(testData)

# cvModel uses the best model found from the Cross Validation
# Evaluate best model
print("Test: Area Under ROC: " + str(evaluator.evaluate(predictions, {evaluator.metricName: "areaUnderROC"})))
```

    Test: Area Under ROC: 0.880473378008


### Gradient Boosting Machines

Last, I'm going to run [Gradient Boosting Machines](https://spark.apache.org/docs/latest/mllib-ensembles.html#gradient-boosted-trees-gbts). Note that this function `GBTClassifier` is relatively new as Spark moves to functions within the `ml` library that can work on DataFrames rather than RDD's.


```python
from pyspark.ml.classification import GBTClassifier

# Train a GBT model.
gbt = GBTClassifier(labelCol="label", \
                    featuresCol="features", \
                    maxIter=50 \
                    )

# Train model.  This also runs the indexers.
model = gbt.fit(trainingData)
```


```python
# Make predictions.
predictions = model.transform(testData)

# Select (prediction, true label) and compute test error
evaluator = BinaryClassificationEvaluator(labelCol="label", \
                                          rawPredictionCol="prediction")
print("Test: Area Under ROC: " + str(evaluator.evaluate(predictions, {evaluator.metricName: "areaUnderROC"})))
```

    Test: Area Under ROC: 0.723369665493


### Scoring Missing Location Tweets

Let's now predict the location for the users' who have missing profile location.


```python
tweetsMissing = tweetsMissing.withColumn("label", \
                                         (when(col("displayName").like("%Charlotte%"), 0) \
                                         .otherwise(1)))

pipeline = Pipeline(stages=[regexTokenizer, stopwordsRemover, countVectors])

pipelineMissing = pipeline.fit(tweetsMissing)
missingData = pipelineMissing.transform(tweetsMissing)

# Predict on Test Data
predictions = lrModel.transform(missingData)
```
