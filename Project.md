Overview
========

In this project, we first download the data and split the train dataset
to two separate datasets:training and testing. Because the dimensions of
the data is so huge, and there is a lot of missing values. We need to
drop these missin values and reduce the dimension of predictors. Then we
fit a classification model and test it on the test dateset. Finally we
apply the model to the new data and make predictions.

    path<-getwd()
    url <- "https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv"
    download.file(url, file.path(path, "train.csv"))
    url <- "https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv"
    download.file(url, file.path(path, "test.csv"))
    train<-read.csv("train.csv")
    test<-read.csv("test.csv")
    library(caret)

    ## Loading required package: lattice

    ## Loading required package: ggplot2

    library(AppliedPredictiveModeling)
    library(dplyr)

    ## 
    ## Attaching package: 'dplyr'

    ## The following objects are masked from 'package:stats':
    ## 
    ##     filter, lag

    ## The following objects are masked from 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union

    library(rpart.plot)

    ## Loading required package: rpart

    library(rpart)
    library(rattle)

    ## Loading required package: tibble

    ## Loading required package: bitops

    ## Rattle: A free graphical interface for data science with R.
    ## XXXX 5.4.0 Copyright (c) 2006-2020 Togaware Pty Ltd.
    ## Type 'rattle()' to shake, rattle, and roll your data.

    names(train)[names(train)=="X"]="class"
    names(test)[names(test)=="X"]="class"
    train$class<-as.factor(train$class)
    test$class<-as.factor(test$class)

Exclude missing value
=====================

    drop_NA<-function(df){
            index<-NULL
            for( i in 1:dim(df)[2] ){
                    if(is.na(df[1,i])==T){index<-c(index,i)}
            }
            df<-df[,-c(1,2,index)]
            }
    train1<-drop_NA(train)
    test1<-drop_NA(test)

Reduce the dimension of predictors
==================================

    intrain<-createDataPartition(y=train1$classe,p=0.7,list=F)
    train1_train<-train1[intrain,]
    train1_test<-train1[-intrain,]
    NZV <- nearZeroVar(train1_train)
    train1_train <- train1_train[, -NZV]
    train1_test <- train1_test[, -NZV]
    dim(train1_train)

    ## [1] 13737    57

Build a classification trees
============================

    set.seed(1)
    mod<-train(classe~.,data=train1_train,method="rpart")

    fancyRpartPlot(mod$finalModel)

![](Project_files/figure-markdown_strict/unnamed-chunk-5-1.png)

Crossvalidate the model
=======================

    pred<-predict(mod,train1_test)
    tb<-table(pred,train1_test$classe)
    confusionMatrix(tb)

    ## Confusion Matrix and Statistics
    ## 
    ##     
    ## pred    A    B    C    D    E
    ##    A 1258  293   30   42   10
    ##    B    3  178    0    0    0
    ##    C  301  241  867  439  287
    ##    D  110  427  129  483  279
    ##    E    2    0    0    0  506
    ## 
    ## Overall Statistics
    ##                                           
    ##                Accuracy : 0.5594          
    ##                  95% CI : (0.5466, 0.5721)
    ##     No Information Rate : 0.2845          
    ##     P-Value [Acc > NIR] : < 2.2e-16       
    ##                                           
    ##                   Kappa : 0.4466          
    ##                                           
    ##  Mcnemar's Test P-Value : NA              
    ## 
    ## Statistics by Class:
    ## 
    ##                      Class: A Class: B Class: C Class: D Class: E
    ## Sensitivity            0.7515  0.15628   0.8450  0.50104  0.46765
    ## Specificity            0.9109  0.99937   0.7390  0.80797  0.99958
    ## Pos Pred Value         0.7704  0.98343   0.4061  0.33824  0.99606
    ## Neg Pred Value         0.9022  0.83152   0.9576  0.89208  0.89288
    ## Prevalence             0.2845  0.19354   0.1743  0.16381  0.18386
    ## Detection Rate         0.2138  0.03025   0.1473  0.08207  0.08598
    ## Detection Prevalence   0.2775  0.03076   0.3628  0.24265  0.08632
    ## Balanced Accuracy      0.8312  0.57782   0.7920  0.65450  0.73362

Therefore, here the **accuary** is **0.6503** and the **out of sample
error** is **0.35**

Applying the model to the test data
===================================

    predict(mod,test1)

    ##  [1] C C C A A D C D A A D C B A C C C D C B
    ## Levels: A B C D E
