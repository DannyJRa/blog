---
title: BigData with Spark
author: DannyJRa
date: '2018-06-05'
slug: bigdata_with_spark
categories:
  - R
tags:
  - BigData
  - Azure
hidden: false
image: "img/05_BLOG.png"
share: false
output:
  html_document:
    keep_md: yes
    theme: cerulean
    highlight: tango
    code_folding: show
    toc: yes
    toc_float: yes
  pdf_document:
    number_sections: yes
geometry: margin = 1.2in
fontsize: 10pt
always_allow_html: yes

---






Original post here: https://dannyjra.github.io/05_Blog_BigData_Spark/01_blog_caret_Tut.html

Add summary desdcription of blog
 
<!--more-->








# Use Case

In this case a Ubuntu DSVM is deployed. The
virtual machine is created within its own resource group so that all
created resources (the VM, networking, disk, etc) can be deleted
easily. Code is also included, and run, to then delete the
resource group if the resource group was created within this
vignette. Once deleted consumption (cost) will cease.

# Setup

To get started we need to load our Azure credentials as well as the
user's ssh public key. Public keys on Linux are typically created on
the users desktop/laptop machine and will be found within
~/.ssh/id_rsa.pub. It will be convenient to create a credentials file
to contain this information. 

#Authorization

We can simply source the credentials file in R.


```r
# Load the required subscription resources: TID, CID, and KEY.
# Also includes the ssh PUBKEY for the user.

USER <- Sys.info()[['user']]
#or set manually
USER<-"insider"

source("~/02_CloudComputing/10_Secrets/Azure_Authentication_datascience_AWS.R")
```

If the required pacakges are not yet installed the following will do
so. You may need to install them into your own local library rather
than the system library if you are not a system user.



We can then load the required pacakges from the libraries.

# Set HDInsight

## Overview

`AzureSMR` provides an interface to manage resources on Microsoft Azure. The main functions address the following Azure Services:

- Azure Blob: List, Read and Write to Blob Services
- Azure Resources: List, Create and Delete Azure Resource
- Azure VM: List, Start and Stop Azure VMs
- Azure HDI: List and Scale Azure HDInsight Clusters
- Azure Hive: Run Hive queries against a HDInsight Cluster
- Azure Spark: List and create Spark jobs/Sessions against a HDInsight Cluster(Livy) - EXPERIMENTAL
- Azure Data Lake Store: ListStatus, GetFileStatus, MkDirs, Create (file), Append (file), Read (file), Delete


For a detailed list of `AzureSMR` functions and their syntax please refer to the Help pages.

## Configuring authorisation in Azure Active Directory

To get started, please refer to the [authorisation tutorial](http://htmlpreview.github.io/?https://github.com/Microsoft/AzureSMR/blob/master/inst/doc/Authentication.html)


## Load the package


```r
library(AzureSMR)
```

## Authenticating against the Azure service

The Azure APIs require many parameters to be managed. Rather than supplying all the arguments to every function call, `AzureSMR` uses an `azureActiveContext` object that caches arguments so you don't have to supply .

To create an `azureActiveContext` object and attempt to authenticate against the Azure service, use:


```r
context <- createAzureContext(tenantID=TID, clientID=CID, authKey=KEY)
```

##Create a resource group


```r
azureCreateResourceGroup(context, resourceGroup = "R_Control", location = "centralus")
```



## Manage HDInsight clusters

You can use `AzureSMR` to manage [HDInsight](https://azure.microsoft.com/en-gb/services/hdinsight/) clusters. To create a cluster use `azureCreateHDI()`.

For advanced configurations use Resource Templates (See below).
ca. 17 minutes

```r
azureCreateHDI(context,
                 resourceGroup = "R_control",
                 clustername = "smrhdi", # only low case letters, digit, and dash.
                 storageAccount = "testmystorage",
                 adminUser = "hdiadmin",
                 adminPassword = "AzureSMR_password123",
                 sshUser = "hdisshuser",
                 sshPassword = "AzureSMR_password123", 
                 kind = "rserver",
               location="centralus")
```

Use `azureListHDI()` to list available clusters.


```r
azureListHDI(context, resourceGroup ="R_control")
```

Use `azureResizeHDI()` to resize a cluster
NOT_WORKING

```r
azureResizeHDI(context, resourceGroup = "R_control", clustername = "smrhdi", role="workernode",size=3)

## azureResizeHDI: Request Submitted:  2016-06-23 18:50:57
## Resizing(R), Succeeded(S)
## RRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRR
## RRRRRRRRRRRRRRRRRRS
## Finished Resizing Sucessfully:  2016-06-23 19:04:43
## Finished:  2016-06-23 19:04:43
##                                                                                                                        ## Information 
## " headnode ( 2 * Standard_D3_v2 ) workernode ( 5 * Standard_D3_v2 ) zookeepernode ( 3 * Medium ) edgenode0 ( 1 * Standard_D4_v2 )" 
```


ADMIN TIP: If a deployment fails, go to the Azure Portal and look at `Activity logs` and look for failed deployments - this should explain why the deployment failed.


## Hive Functions

You can use these functions to run and manage hive jobs on an HDInsight Cluster.


```r
azureHiveStatus(context, clusterName = "smrhdi", 
                hdiAdmin = "hdiadmin", 
                hdiPassword = "AzureSMR_password123")

azureHiveSQL(context, 
             CMD = "select * from hivesampletable", 
             path = "wasb://opendata@testmystorage1.blob.core.windows.net/")
```


## Spark functions (experimental)

`AzureSMR` provides some functions that allow HDInsight Spark aessions and jobs to be managed within an R Session.

To create a new Spark session (via [Livy](https://github.com/cloudera/hue/tree/master/apps/spark/java#welcome-to-livy-the-rest-spark-server)) use `azureSparkNewSession()`


```r
azureSparkNewSession(context, clustername = "spark1987", 
                     hdiAdmin = "", 
                     hdiPassword = "",
                     kind = "pyspark")
```

To view the status of sessions use `azureSparkListSessions()`. Wait for status to be idle.


```r
azureSparkListSessions(context, clustername = "spark1987")
```

To send a command to the Spark Session use `azureSparkCMD()`. In this case it submits a Python routine. Ensure you preserve indents for Python.


```r
# SAMPLE PYSPARK SCRIPT TO CALCULATE PI
pythonCmd <- '
from pyspark import SparkContext
from operator import add
import sys
from random import random
partitions = 1
n = 20000000 * partitions
def f(_):
  x = random() * 2 - 1
  y = random() * 2 - 1
  return 1 if x ** 2 + y ** 2 < 1 else 0
 
count = context.parallelize(range(1, n + 1), partitions).map(f).reduce(add)
Pi = (4.0 * count / n)
print("Pi is roughly %f" % Pi)'                   
 
azureSparkCMD(context, CMD = pythonCmd, sessionID = "0")

## [1] "Pi is roughly 3.140285"
```

Check Session variables are retained


```r
azureSparkCMD(context, clustername = "spark1987", CMD = "print Pi", sessionID = "0")

#[1] "3.1422"
```

You can also run SparkR sessions

NOT WORKING

```r
azureSparkNewSession(context, clustername = "spark1987", 
                     hdiAdmin = "insider_danny", 
                     hdiPassword = ".Alfonstini642856",
                     kind = "sparkr")


azureSparkCMD(context, clustername = "smrhdi", CMD = "HW<-'hello R'", sessionID = "2")
azureSparkCMD(context, clustername = "smrhdi", CMD = "cat(HW)", sessionID = "2")
```

# Spark

### --- Chapter 9 --- ###
### --- Big Data machine learning with R --- ###

### --- Part 1 --- ###
### --- GLM - logistic regression on Spark --- ###

Sys.getenv("SPARK_HOME")

Sys.setenv(SPARK_HOME = "/usr/hdp/2.4.2.0-258/spark")
Sys.getenv("SPARK_HOME")
Sys.setenv('SPARKR_SUBMIT_ARGS'='"--packages" "com.databricks:spark-csv_2.11:1.4.0" "sparkr-shell"')
Sys.getenv("SPARKR_SUBMIT_ARGS")

library(rJava)
library(SparkR, lib.loc = c(file.path(Sys.getenv("SPARK_HOME"), "R", "lib")))

sc <- sparkR.init(master="yarn-client", 
                  appName="SparkRStudio", 
                  sparkJars = c("/usr/hdp/2.4.2.0-258/hadoop/hadoop-nfs.jar", 
                                "/usr/hdp/2.4.2.0-258/hadoop/hadoop-azure.jar", 
                                "/usr/hdp/2.4.2.0-258/hadoop/lib/azure-storage-2.2.0.jar"),
                  sparkPackages="com.databricks:spark-csv_2.11:1.4.0")

sqlContext <- sparkRSQL.init(sc)

schema <- structType(structField("DAY_OF_WEEK", "string"), 
                     structField("DEP_TIME", "integer"), 
                     structField("DEP_DELAY", "integer"), 
                     structField("ARR_TIME", "integer"),
                     structField("ARR_DELAY", "integer"), 
                     structField("CANCELLED", "integer"),
                     structField("DIVERTED", "integer"), 
                     structField("AIR_TIME", "integer"), 
                     structField("DISTANCE", "integer"))

flights <- read.df(sqlContext, 
                   path = "/user/swalko/data/flights_2014.csv", 
                   source = "com.databricks.spark.csv", 
                   header = "true", 
                   schema = schema, nullValue = "NA")

head(flights)
str(flights)
count(flights)

flights <- filter(flights, flights$CANCELLED == 0)
flights <- filter(flights, flights$DIVERTED == 0)

flights <- flights[, -6:-7]
str(flights)

dtypes(flights)

registerTempTable(flights, "flights")
flights <- sql(sqlContext, "SELECT *, IF(ARR_DELAY > 0, 1, 0) 
               AS ARR_DEL from flights")

str(flights)
#We need to re-register the DataFrame as a SQL table to reflect the changes in the previous query.
registerTempTable(flights, "flights")
flights <- sql(sqlContext, "SELECT *, CASE WHEN(DEP_TIME >= 500 AND DEP_TIME < 1200)
               THEN ('morning') WHEN(DEP_TIME >= 1200 AND DEP_TIME < 1700)
               THEN ('afternoon') WHEN(DEP_TIME >= 1700 AND DEP_TIME < 2100)
               THEN ('evening') ELSE('night') END AS DEP_PART from flights") 
str(flights)
registerTempTable(flights, "flights")

logit1 <- glm(ARR_DEL ~ AIR_TIME + DISTANCE + DAY_OF_WEEK + DEP_PART + DEP_DELAY,
              data = flights, family = "binomial")

summary(logit1)

head(select(flights, mean(flights$AIR_TIME)))
head(select(flights, mean(flights$DISTANCE)))
head(select(flights, mean(flights$DEP_DELAY)))

test1 <- createDataFrame(sqlContext, 
                         data = data.frame(AIR_TIME = 111.37,
                                           DISTANCE = 802.54, 
                                           DEP_DELAY = 10.57, 
                                           DAY_OF_WEEK = factor(rep(c("1", "2", 
                                                                      "3", "4",
                                                                      "5", "6", "7"), each=4)), 
                                           DEP_PART = factor(rep(c("morning", "afternoon", 
                                                                   "evening", "night"), times=7))))
showDF(test1)

predicted <- predict(logit1, test1)
showDF(predicted, numRows = 28, truncate = FALSE)

test2 <- createDataFrame(sqlContext, 
                         data = data.frame(AIR_TIME=450, 
                                           DISTANCE=3400, 
                                           DEP_DELAY = -10, 
                                           DAY_OF_WEEK = "1", 
                                           DEP_PART = "morning"))

showDF(predict(logit1, test2), truncate = FALSE)

flightsPred <- predict(logit1, flights)
prediction <- select(flightsPred, "ARR_DEL", "prediction")
showDF(prediction, numRows = 200)

# Overall accuracy rate:
prediction$success <- ifelse(prediction$ARR_DEL == prediction$prediction, 1, 0)
registerTempTable(prediction, "prediction")
correct <- sql(sqlContext, "SELECT count(success) FROM prediction WHERE success = 1")
total <- count(prediction)
accuracy <- collect(correct) / total
accuracy #82.7% accuracy

# Accuracy of predicting delayed flights:
prediction <- select(flightsPred, "ARR_DEL", "prediction")
pred_del <- filter(prediction, prediction$ARR_DEL==1)
registerTempTable(pred_del, "pred_del")
showDF(pred_del, numRows = 200)

pred_cor <- sql(sqlContext, "SELECT count(prediction) FROM prediction WHERE prediction = 1")
total_delayed <- count(pred_del)
acc_pred <- collect(pred_cor) / total_delayed
acc_pred #80% of delayed flights have been correctly identified

# Using flights_jan_2015.csv dataset we can apply the model on a new test data:

jan15 <- read.df(sqlContext, 
                 path = "/user/swalko/data/flights_jan_2015.csv", 
                 source = "com.databricks.spark.csv", 
                 header = "true", 
                 schema = schema, 
                 nullValue = "NA")

str(jan15)
jan15 <- filter(jan15, jan15$CANCELLED == 0) 
jan15 <- filter(jan15, jan15$DIVERTED == 0)
jan15 <- jan15[, -6:-7]

registerTempTable(jan15, "jan15")
jan15 <- sql(sqlContext, "SELECT *, IF(ARR_DELAY > 0, 1, 0) 
             AS ARR_DEL from jan15")

registerTempTable(jan15, "jan15")
jan15 <- sql(sqlContext, "SELECT *, CASE WHEN(DEP_TIME >= 500 AND DEP_TIME < 1200)
             THEN ('morning') WHEN(DEP_TIME >= 1200 AND DEP_TIME < 1700)
             THEN ('afternoon') WHEN(DEP_TIME >= 1700 AND DEP_TIME < 2100)
             THEN ('evening') ELSE('night') END AS DEP_PART from jan15")

jan15 <- select(jan15, "DAY_OF_WEEK", "DEP_DELAY", 
                "AIR_TIME", "DISTANCE", "DEP_PART", "ARR_DEL")
str(jan15)

janPred <- predict(logit1, jan15)
jan_eval <- select(janPred, "ARR_DEL", "prediction")
showDF(jan_eval, numRows = 200)

# Overall accuracy rate:
jan_eval$success <- ifelse(jan_eval$ARR_DEL == jan_eval$prediction, 1, 0)
registerTempTable(jan_eval, "jan_eval")
correct <- sql(sqlContext, "SELECT count(success) FROM jan_eval WHERE success = 1")
total <- count(jan_eval)
accuracy <- collect(correct) / total
accuracy #80.7% accuracy

# Accuracy of predicting delayed flights:
jan_eval <- select(janPred, "ARR_DEL", "prediction")
jan_del <- filter(jan_eval, jan_eval$ARR_DEL==1)
registerTempTable(jan_del, "jan_del")
showDF(jan_del, numRows = 200)

jan_cor <- sql(sqlContext, "SELECT count(prediction) FROM jan_del WHERE prediction = 1")
total_delayed <- count(jan_del)
acc_pred <- collect(jan_cor) / total_delayed
acc_pred #67.3% of delayed flights have been correctly identified

### --- End of Part 1 --- ###



### --- Part 2 --- ### 

### --- H2O.ai on R - Naive Bayes --- ###

library(h2o)
h2o <- h2o.init(ip = "10.2.0.10", port = 54321, startH2O = F)
h2o.clusterInfo()

path1 <- "/home/swalko/data/flights_2014.csv"
flights14 <- h2o.uploadFile(path = path1, 
                            destination_frame = "flights14",
                            parse = TRUE, header = TRUE, 
                            sep = ",")

h2o.ls()

summary(flights14)
str(flights14)

flights14 <- flights14[flights14$CANCELLED==0 & flights14$DIVERTED==0, ]
flights14 <- flights14[, -6:-7]

h2o.nacnt(flights14) #no missing values

flights14$DAY_OF_WEEK <- as.factor(flights14$DAY_OF_WEEK)

avg_del <- function(x) { sum(x[,3])/nrow(x) }
avg.del <- h2o.ddply(flights14, "DAY_OF_WEEK", FUN = avg_del)
as.data.frame(avg.del)

flights14$DEP_PART <- h2o.cut(flights14$DEP_TIME, 
                              c(1, 459, 1159, 1659, 2059, 2400), 
                              labels = c("night", "morning", 
                                         "afternoon", "evening", 
                                         "night"))
h2o.table(flights14$DEP_PART)

flights14$DEP_TIME <- flights14$ARR_TIME <- NULL

flights14$ARR_DEL <- as.factor(h2o.ifelse(flights14$ARR_DELAY > 0, 1, 0))

flights14$ARR_DELAY <- NULL
h2o.table(flights14$ARR_DEL)
prop.table(h2o.table(flights14$ARR_DEL)) #42% of delayed flights overall

summary(flights14$DEP_DELAY)
flights14$DEP_DELAY <- h2o.cut(flights14$DEP_DELAY, 
                               c(-112, -15, -1, 1, 16, 2402), 
                               labels = c("very early", 
                                          "somewhat early", 
                                          "on time", 
                                          "somewhat delayed", 
                                          "very delayed"))
h2o.table(flights14$DEP_DELAY)

h2o.quantile(flights14$DISTANCE, prob = seq(0, 1, length = 4))
#h2o.hist(flights14$DISTANCE)
summary(flights14$DISTANCE)
flights14$DISTANCE <- h2o.cut(flights14$DISTANCE, 
                              c(31, 1000, 2000, 4983), 
                              labels = c("short", "medium", "long"))
h2o.table(flights14$DISTANCE)

h2o.quantile(flights14$AIR_TIME, prob = seq(0, 1, length = 4))
#h2o.hist(flights14$AIR_TIME)
summary(flights14$AIR_TIME)
flights14$AIR_TIME <- h2o.cut(flights14$AIR_TIME, 
                              c(7, 150, 300, 706), 
                              labels = c("short", "medium", "long"))
h2o.table(flights14$AIR_TIME)
str(flights14)

model1 <- h2o.naiveBayes(x = 1:5, y = 6, training_frame = flights14, laplace = 1)
model1

str(model1)
h2o.auc(model1)
h2o.performance(model1)

path2 <- "/home/swalko/data/flights_jan_2015.csv"
flightsJan15 <- h2o.uploadFile(path = path2, 
                               destination_frame = "flightsJan15", 
                               parse = TRUE, header = TRUE, 
                               sep = ",")

flightsJan15 <- flightsJan15[flightsJan15$CANCELLED==0 & flightsJan15$DIVERTED==0, ]
flightsJan15 <- flightsJan15[, -6:-7]

h2o.nacnt(flightsJan15)

flightsJan15$DAY_OF_WEEK <- as.factor(flightsJan15$DAY_OF_WEEK)
flightsJan15$DEP_PART <- h2o.cut(flightsJan15$DEP_TIME, 
                                 c(1, 459, 1159, 1659, 2059, 2400), 
                                 labels = c("night", "morning", 
                                            "afternoon", "evening", 
                                            "night"))

flightsJan15$DEP_TIME <- flightsJan15$ARR_TIME <- NULL
flightsJan15$ARR_DEL <- as.factor(h2o.ifelse(flightsJan15$ARR_DELAY > 0, 1, 0))
flightsJan15$ARR_DELAY <- NULL
h2o.table(flightsJan15$ARR_DEL)
prop.table(h2o.table(flightsJan15$ARR_DEL)) #40% of delayed flights

summary(flightsJan15$DEP_DELAY)
flightsJan15$DEP_DELAY <- h2o.cut(flightsJan15$DEP_DELAY, 
                                  c(-48, -15, -1, 1, 16, 1988), 
                                  labels = c("very early", 
                                             "somewhat early", 
                                             "on time", 
                                             "somewhat delayed", 
                                             "very delayed"))
h2o.table(flightsJan15$DEP_DELAY)

h2o.quantile(flightsJan15$DISTANCE, prob = seq(0, 1, length = 4))
#h2o.hist(flightsJan15$DISTANCE)
summary(flightsJan15$DISTANCE)
flightsJan15$DISTANCE <- h2o.cut(flightsJan15$DISTANCE, 
                                 c(31, 1000, 2000, 4983), 
                                 labels = c("short", 
                                            "medium", 
                                            "long"))
h2o.table(flightsJan15$DISTANCE)

h2o.quantile(flightsJan15$AIR_TIME, prob = seq(0, 1, length = 4))
#h2o.hist(flightsJan15$AIR_TIME)
summary(flightsJan15$AIR_TIME)
flightsJan15$AIR_TIME <- h2o.cut(flightsJan15$AIR_TIME, 
                                 c(8, 150, 300, 676), 
                                 labels = c("short", 
                                            "medium", 
                                            "long"))
h2o.table(flightsJan15$AIR_TIME)

str(flightsJan15)

fit1 <- h2o.predict(object = model1, newdata = flightsJan15)
fit1

h2o.performance(model1, newdata = flightsJan15)
### --- End of Part 2 --- ###

### --- Part 3 --- ###
### --- Neural Networks and Deep Learning on H2O.ai --- ###

rm(list=ls())
path1 <- "/home/swalko/data/flights_2014.csv"
flights14 <- h2o.uploadFile(path = path1, 
                            destination_frame = "flights14",
                            parse = TRUE, header = TRUE, 
                            sep = ",")
h2o.ls()

#summary(flights14)
#str(flights14)

flights14 <- flights14[flights14$CANCELLED==0 & flights14$DIVERTED==0, ]
flights14 <- flights14[, -6:-7]

flights14$DAY_OF_WEEK <- as.factor(flights14$DAY_OF_WEEK)
flights14$ARR_DEL <- as.factor(h2o.ifelse(flights14$ARR_DELAY > 0, 1, 0))
flights14$ARR_DELAY <- flights14$ARR_TIME <- NULL
str(flights14)

path2 <- "/home/swalko/data/flights_jan_2015.csv"
flightsJan15 <- h2o.uploadFile(path = path2, 
                               destination_frame = "flightsJan15", 
                               parse = TRUE, header = TRUE, 
                               sep = ",")

flightsJan15 <- flightsJan15[flightsJan15$CANCELLED==0 & flightsJan15$DIVERTED==0, ]
flightsJan15 <- flightsJan15[, -6:-7]

flightsJan15$DAY_OF_WEEK <- as.factor(flightsJan15$DAY_OF_WEEK)
flightsJan15$ARR_DEL <- as.factor(h2o.ifelse(flightsJan15$ARR_DELAY > 0, 1, 0))
flightsJan15$ARR_DELAY <- flightsJan15$ARR_TIME <- NULL
str(flightsJan15)


model2 <- h2o.deeplearning(x = 1:5, y = 6, training_frame = flights14, 
                           validation_frame = flightsJan15, 
                           hidden = c(10, 5, 3),
                           epochs = 5)

summary(model2)

model3 <- h2o.deeplearning(x = 1:5, y = 6, training_frame = flights14, 
                           validation_frame = flightsJan15, 
                           epochs = 2)

summary(model3)

h2o.shutdown()

### --- End of Part 3 --- ###


### --- The end of Chapter 9 --- ###
