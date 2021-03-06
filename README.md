# Practical Machine Learning Course Project


## Overview

The goal of the project was to use a machine learning algorithm with sensor data generated by test subjects as they lifted weights to classify the lift that generated a particular set of sample data. Appropriate classifications were A for lifts executed with good technique versus B, C, D or E, for lifts executed with poor technique (where each of these classifications reflects a specific technical error). Training data were labeled accordingly and provided alongside unlabeled test data (which, as directed, was not used in this write-up).


## Dependencies

The [caret](http://topepo.github.io/caret/index.html) package was used for its data splitting and training functionality, loaded as follows:



```r
library(caret)
```


## Loading and Splitting Data

In order to ensure a good prediction of accuracy, projections would be made both from cross-validation and from a separately held out set, comprising 30% of the training data. If necessary the held out set could also be used to train an emsemble method. The training data were loaded and split as follows:


```r
dataRaw <- read.csv("pml-training.csv")
trainingIndices <- createDataPartition(dataRaw$classe, p=0.7, list=F)
training <- dataRaw[trainingIndices,]
testing <- dataRaw[-trainingIndices,]
```


## Data Cleaning

As can be seen, the data contained a large number of columns (160), not all of them likely to be useful predictors:


```r
str(training, list.len=20)
```

```
## 'data.frame':	13737 obs. of  160 variables:
##  $ X                       : int  1 3 4 6 7 9 10 12 13 14 ...
##  $ user_name               : Factor w/ 6 levels "adelmo","carlitos",..: 2 2 2 2 2 2 2 2 2 2 ...
##  $ raw_timestamp_part_1    : int  1323084231 1323084231 1323084232 1323084232 1323084232 1323084232 1323084232 1323084232 1323084232 1323084232 ...
##  $ raw_timestamp_part_2    : int  788290 820366 120339 304277 368296 484323 484434 528316 560359 576390 ...
##  $ cvtd_timestamp          : Factor w/ 20 levels "2/12/2011 13:32",..: 15 15 15 15 15 15 15 15 15 15 ...
##  $ new_window              : Factor w/ 2 levels "no","yes": 1 1 1 1 1 1 1 1 1 1 ...
##  $ num_window              : int  11 11 12 12 12 12 12 12 12 12 ...
##  $ roll_belt               : num  1.41 1.42 1.48 1.45 1.42 1.43 1.45 1.43 1.42 1.42 ...
##  $ pitch_belt              : num  8.07 8.07 8.05 8.06 8.09 8.16 8.17 8.18 8.2 8.21 ...
##  $ yaw_belt                : num  -94.4 -94.4 -94.4 -94.4 -94.4 -94.4 -94.4 -94.4 -94.4 -94.4 ...
##  $ total_accel_belt        : int  3 3 3 3 3 3 3 3 3 3 ...
##  $ kurtosis_roll_belt      : Factor w/ 397 levels "","-0.01685",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ kurtosis_picth_belt     : Factor w/ 317 levels "","-0.021887",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ kurtosis_yaw_belt       : Factor w/ 2 levels "","#DIV/0!": 1 1 1 1 1 1 1 1 1 1 ...
##  $ skewness_roll_belt      : Factor w/ 395 levels "","-0.003095",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ skewness_roll_belt.1    : Factor w/ 338 levels "","-0.005928",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ skewness_yaw_belt       : Factor w/ 2 levels "","#DIV/0!": 1 1 1 1 1 1 1 1 1 1 ...
##  $ max_roll_belt           : num  NA NA NA NA NA NA NA NA NA NA ...
##  $ max_picth_belt          : int  NA NA NA NA NA NA NA NA NA NA ...
##  $ max_yaw_belt            : Factor w/ 68 levels "","-0.1","-0.2",..: 1 1 1 1 1 1 1 1 1 1 ...
##   [list output truncated]
```

Inspection showed many of these columns had issues, as follows:</p>
* Fields unrelated to classification, such as the lifter's name. The documentation stated that each lifter performed lifts of each classification for 10 reps (which was confirmed by visual inspection of the data), thus the lifter name would not be a helpful predictor and was removed.
* Fields that accurately predicted the lift classification on the training set (and indeed would on the testing set, since the testing set was sourced from the same pool) but would not generalize. Several time fields featured in the data, which proved to be excellent predictors (since the lifters were instructed in every case to perform 10 reps of A followed by 10 reps of B and so forth). Since these data would not generalize to future data and to avoid 'cheating' these values were removed.
* Fields (sometimes) present in training data but not in the testing data. Clearly these would not be useful predictors and so they were removed.
* Fields that were always or frequently NA were removed.</li>

The removal of the columns, provided the further benefit of reducing the number of columns to 57, which would reduce training time significantly:


```r
cleanedColumns <- c(8:11, 37:49, 60:68, 84:86, 102, 113:124, 140, 151:159, 160)
trainingCleaned <- training[, cleanedColumns]
testingCleaned <- testing[, cleanedColumns]
names(trainingCleaned)
```

```
##  [1] "roll_belt"            "pitch_belt"           "yaw_belt"            
##  [4] "total_accel_belt"     "gyros_belt_x"         "gyros_belt_y"        
##  [7] "gyros_belt_z"         "accel_belt_x"         "accel_belt_y"        
## [10] "accel_belt_z"         "magnet_belt_x"        "magnet_belt_y"       
## [13] "magnet_belt_z"        "roll_arm"             "pitch_arm"           
## [16] "yaw_arm"              "total_accel_arm"      "gyros_arm_x"         
## [19] "gyros_arm_y"          "gyros_arm_z"          "accel_arm_x"         
## [22] "accel_arm_y"          "accel_arm_z"          "magnet_arm_x"        
## [25] "magnet_arm_y"         "magnet_arm_z"         "roll_dumbbell"       
## [28] "pitch_dumbbell"       "yaw_dumbbell"         "total_accel_dumbbell"
## [31] "gyros_dumbbell_x"     "gyros_dumbbell_y"     "gyros_dumbbell_z"    
## [34] "accel_dumbbell_x"     "accel_dumbbell_y"     "accel_dumbbell_z"    
## [37] "magnet_dumbbell_x"    "magnet_dumbbell_y"    "magnet_dumbbell_z"   
## [40] "roll_forearm"         "pitch_forearm"        "yaw_forearm"         
## [43] "total_accel_forearm"  "gyros_forearm_x"      "gyros_forearm_y"     
## [46] "gyros_forearm_z"      "accel_forearm_x"      "accel_forearm_y"     
## [49] "accel_forearm_z"      "magnet_forearm_x"     "magnet_forearm_y"    
## [52] "magnet_forearm_z"     "classe"
```


## Building the Model

A Random Forest classifier was used for its versatility, non-linearity and transparency (it is fairly easy to see and debug the rules underlying the trained classifier). If this proved not to provide the requisite accuracy, SVMs, Boosting and Linear Discriminant Analysis would be tested and/or an ensemble could be constructed.

The model was trained as follows, its time being recorded for interest and in case it would prove useful data later (e.g. in ensembling). Note that the seed is set to enable the results to be reproduced later if necessary:


```r
set.seed(424242)
trainingTime <- system.time(model <- train(classe ~., data=trainingCleaned, method="rf", PROX=T))
```


As can be seen, the model was time-consuming to train (running on an Asus Zenbook 301, with a dual core Intel i7 @ 2.8 Ghz, with 8 GB of RAM):


```r
trainingTime["elapsed"]
```

```
## elapsed 
## 4460.48
```


## Inspecting the Model

As mentioned, it is possible to see in detail the rules employed by the forest. In particular, we can see the relative importance (i.e. predictive power) given to each variable using the varImp function.


```r
varImp(model)
```

```
## rf variable importance
## 
##   only 20 most important variables shown (out of 52)
## 
##                   Overall
## roll_belt          100.00
## yaw_belt            77.83
## magnet_dumbbell_z   71.97
## magnet_dumbbell_y   65.27
## pitch_forearm       64.62
## pitch_belt          64.51
## roll_forearm        53.17
## magnet_dumbbell_x   52.59
## accel_belt_z        46.45
## magnet_belt_z       45.66
## roll_dumbbell       45.19
## accel_dumbbell_y    44.51
## magnet_belt_y       41.41
## accel_dumbbell_z    40.28
## roll_arm            34.63
## accel_forearm_x     32.45
## accel_dumbbell_x    31.95
## gyros_belt_z        31.22
## yaw_dumbbell        30.73
## gyros_dumbbell_y    28.91
```


## Model Accuracy

To determine whether the model was sufficient, its accuracy in cross-validation was examined:


```r
model
```

```
## Random Forest 
## 
## 13737 samples
##    52 predictor
##     5 classes: 'A', 'B', 'C', 'D', 'E' 
## 
## No pre-processing
## Resampling: Bootstrapped (25 reps) 
## 
## Summary of sample sizes: 13737, 13737, 13737, 13737, 13737, 13737, ... 
## 
## Resampling results across tuning parameters:
## 
##   mtry  Accuracy   Kappa      Accuracy SD  Kappa SD   
##    2    0.9885667  0.9855308  0.001924787  0.002438536
##   27    0.9885024  0.9854502  0.001845531  0.002336228
##   52    0.9784945  0.9727870  0.003951176  0.004998504
## 
## Accuracy was used to select the optimal model using  the largest value.
## The final value used for the model was mtry = 2.
```

The resulting 99% accuracy was deemed sufficient and so this model was selected.

Finally, to double-check the out of sample accuracy of the model, it was run against the set held out at the beginning:


```r
predictions <- predict(model, newdata=testingCleaned)
confusionMatrix(predictions, testingCleaned$classe)
```

```
## Confusion Matrix and Statistics
## 
##           Reference
## Prediction    A    B    C    D    E
##          A 1674    8    0    0    0
##          B    0 1130   22    0    0
##          C    0    1 1003   14    3
##          D    0    0    1  948    0
##          E    0    0    0    2 1079
## 
## Overall Statistics
##                                           
##                Accuracy : 0.9913          
##                  95% CI : (0.9886, 0.9935)
##     No Information Rate : 0.2845          
##     P-Value [Acc > NIR] : < 2.2e-16       
##                                           
##                   Kappa : 0.989           
##  Mcnemar's Test P-Value : NA              
## 
## Statistics by Class:
## 
##                      Class: A Class: B Class: C Class: D Class: E
## Sensitivity            1.0000   0.9921   0.9776   0.9834   0.9972
## Specificity            0.9981   0.9954   0.9963   0.9998   0.9996
## Pos Pred Value         0.9952   0.9809   0.9824   0.9989   0.9981
## Neg Pred Value         1.0000   0.9981   0.9953   0.9968   0.9994
## Prevalence             0.2845   0.1935   0.1743   0.1638   0.1839
## Detection Rate         0.2845   0.1920   0.1704   0.1611   0.1833
## Detection Prevalence   0.2858   0.1958   0.1735   0.1613   0.1837
## Balanced Accuracy      0.9991   0.9937   0.9869   0.9916   0.9984
```


## Summary

The above has shown how, after pre-processing of the provided data, a Random Forest classifier was trained and shown to achieve 99% accuracy in its predictions.
