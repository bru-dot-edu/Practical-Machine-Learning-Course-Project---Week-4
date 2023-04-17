---
title: "Practical Machine Learning Week 4 Course Project"
author: "Chris Brueck"
date: "2023-04-16"
output: html_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE, warning = FALSE, message = FALSE)
```

## Overview   

This course project will involve the training and testing of machine learning models to work with wearable fitness tracker type data and predict the manner in which the user performed the exercise. 

## Background

"Using devices such as Jawbone Up, Nike FuelBand, and Fitbit it is now possible to collect a large amount of data about personal activity relatively inexpensively. These type of devices are part of the quantified self movement – a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. One thing that people regularly do is quantify how much of a particular activity they do, but they rarely quantify how well they do it. In this project, your goal will be to use data from accelerometers on the belt, forearm, arm, and dumbell of 6 participants. They were asked to perform barbell lifts correctly and incorrectly in 5 different ways. More information is available from the website here: http://web.archive.org/web/20161224072740/http:/groupware.les.inf.puc-rio.br/har (see the section on the Weight Lifting Exercise Dataset)."

# Load libraries

```{r}
library(dplyr)
library(tidyverse)
library(caret)
library(randomForest)
```

## Data loading and cleanup

"The training data for this project are available here:
https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv

The test data are available here:
https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv"

Load in data and drop columns that have NA entries. Also, remove columns that are irrelevant for analysis (first seven columns).

```{r}
train <- read.csv('./pml-training.csv', header=T, na.strings=c("NA", ""))
anyNA.train <- sapply(train, function(x) sum(is.na(x)))
train <- train[anyNA.train == 0]
train <- train[,-c(1:7)]

test <- read.csv('./pml-testing.csv', header=T, na.strings=c("NA", ""))
anyNA.test <- sapply(test, function(x) sum(is.na(x)))
test <- test[anyNA.test == 0]
test <- test[,-c(1:7)]
```

## Data split and model construction

We will build and save a random forest model. The model is cross validated with K = 5.

```{r}
samples <- createDataPartition(y=train$classe, p=0.75, list=FALSE)
train.train <- train[samples, ] 
train.test <- train[-samples, ]

rf <- train(
  classe ~ ., 
  data=train.train,#[, c('classe', names(train.data))],
  trControl=trainControl(method='cv', number = 5),
  method='rf',
  ntree=100
)

save(rf, file='./RFmodel.RData')
```

## Model evaluation

```{r}
train.test$classe <- as.factor(train.test$classe)
predict <- predict(rf, newdata = train.test)
confusionMatrix(predict, train.test$classe)

## Confusion Matrix and Statistics
## 
##           Reference
## Prediction    A    B    C    D    E
##          A 1389   12    0    1    0
##          B    2  935    6    1    0
##          C    3    2  845   12    2
##          D    0    0    4  788    0
##          E    1    0    0    2  899
## 
## Overall Statistics
##                                          
##                Accuracy : 0.9902         
##                  95% CI : (0.987, 0.9928)
##     No Information Rate : 0.2845         
##     P-Value [Acc &gt; NIR] : &lt; 2.2e-16      
##                                          
##                   Kappa : 0.9876         
##                                          
##  Mcnemar&#39;s Test P-Value : NA             
## 
## Statistics by Class:
## 
##                      Class: A Class: B Class: C Class: D Class: E
## Sensitivity            0.9957   0.9852   0.9883   0.9801   0.9978
## Specificity            0.9963   0.9977   0.9953   0.9990   0.9993
## Pos Pred Value         0.9907   0.9905   0.9780   0.9949   0.9967
## Neg Pred Value         0.9983   0.9965   0.9975   0.9961   0.9995
## Prevalence             0.2845   0.1935   0.1743   0.1639   0.1837
## Detection Rate         0.2832   0.1907   0.1723   0.1607   0.1833
## Detection Prevalence   0.2859   0.1925   0.1762   0.1615   0.1839
## Balanced Accuracy      0.9960   0.9915   0.9918   0.9896   0.9985
```
The random forest model has an accuracy of 99%. This gives us an out of sample expected error of 1%, which gives us high confidence that our model will make accurate predictions, with incorrect classifications 1% of the time. 

## Predictions

Lastly, we use the trained and validated random forest model to predict using the test data provided in the submission instructions. 
```{r}
predict(rf, newdata = test)
##  [1] B A B A A E D B A A B C B A E E A B B B
## Levels: A B C D E
```

