Coursera Practical Machine Learning Writeup
Arnab Prasad Rakshit

Saturday 16, May, 2015

1. Introduction
Using devices such as Jawbone Up, Nike FuelBand, and Fitbit it is now possible to collect a large amount of data about personal activity relatively inexpensively. These type of devices are part of the quantified self movement - a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. One thing that people regularly do is quantify how much of a particular activity they do, but they rarely quantify how well they do it.

In this project, your goal will be to use data from accelerometers on the belt, forearm, arm, and dumbell of 6 participants. They were asked to perform barbell lifts correctly and incorrectly in 5 different ways.

2. The Dataset

The test data are available here: https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv

Here the chunk of code for reading the dataset and a sample of it:

library(caret)
## Loading required package: lattice
## Loading required package: ggplot2
training_raw <- read.csv('pml-training.csv', na.strings= c("NA",""))
testing_raw <- read.csv('pml-testing.csv', na.strings= c("NA",""))
2. Cleaning and Feature Selection
Few variables are not related to the class as they hold information about the user performing the exercise or date/time when the execise was taken (e.g. user_name, raw_timestamp_part_1, new_window). We are instead interested in those variables holdin sensor’s data.

In the chunk of code below I keep those variables realated to sensors (e.g. belt, arm, dumbbell, forearm) and the classe variable as it holds the classification of the activity performed by the user.

A further check is done on NA values, variable showing NA are discarded as well.

sensor_col <- grepl("belt|arm|dumbbell|forearm|classe",names(training_raw))
training_set <- training_raw[,sensor_col]
null_col <- colSums(is.na(training_set))!=0
training_set <- training_set[,!null_col]
3. Data Processing and Training
I proceed with the split of the original training set into two subsets: training (80% of original observations) and validation (20% of original observations).

I then proceed with the model training using the Random Forest method which leads to very promising results as suggested by the accuracy reached on its validation phase:

set.seed(1234)
training_cv_index <- createDataPartition(training_set$classe,list=FALSE ,p=0.8)
training_cv <- training_set[training_cv_index,]
validation_cv <- training_set[-training_cv_index,]

model <- train(classe~., data=training_cv, method="rf")
## Loading required package: randomForest
## randomForest 4.6-10
## Type rfNews() to see new features/changes/bug fixes.
model
## Random Forest 
## 
## 15699 samples
##    52 predictors
##     5 classes: 'A', 'B', 'C', 'D', 'E' 
## 
## No pre-processing
## Resampling: Bootstrapped (25 reps) 
## 
## Summary of sample sizes: 15699, 15699, 15699, 15699, 15699, 15699, ... 
## 
## Resampling results across tuning parameters:
## 
##   mtry  Accuracy  Kappa  Accuracy SD  Kappa SD
##   2     1         1      0.002        0.002   
##   30    1         1      0.002        0.002   
##   50    1         1      0.004        0.005   
## 
## Accuracy was used to select the optimal model using  the largest value.
## The final value used for the model was mtry = 27.
4. Model Validation
The trained model is validated by means of the confusion matrix which is created by predicting the validation set and by comparing the predicted resuts and the actual classes. Below the confusion matrix details:

confusionMatrix(validation_cv$classe,predict(model,validation_cv))
## Confusion Matrix and Statistics
## 
##           Reference
## Prediction    A    B    C    D    E
##          A 1116    0    0    0    0
##          B    6  752    1    0    0
##          C    0    3  680    1    0
##          D    0    0    2  641    0
##          E    0    0    0    0  721
## 
## Overall Statistics
##                                         
##                Accuracy : 0.997         
##                  95% CI : (0.994, 0.998)
##     No Information Rate : 0.286         
##     P-Value [Acc > NIR] : <2e-16        
##                                         
##                   Kappa : 0.996         
##  Mcnemar's Test P-Value : NA            
## 
## Statistics by Class:
## 
##                      Class: A Class: B Class: C Class: D Class: E
## Sensitivity             0.995    0.996    0.996    0.998    1.000
## Specificity             1.000    0.998    0.999    0.999    1.000
## Pos Pred Value          1.000    0.991    0.994    0.997    1.000
## Neg Pred Value          0.998    0.999    0.999    1.000    1.000
## Prevalence              0.286    0.192    0.174    0.164    0.184
## Detection Rate          0.284    0.192    0.173    0.163    0.184
## Detection Prevalence    0.284    0.193    0.174    0.164    0.184
## Balanced Accuracy       0.997    0.997    0.997    0.999    1.000
The cross-validation accuracy reached is 99.7% with a confidence of at least 99.4%. On the basis of this result we are good to move on and predict the given testing set.

5. Predict Testing Values
Below the prediction phase and results obtained:

result <-predict(model,testing_raw)
result
##  [1] B A B A A E D B A A B C B A E E A B B B
## Levels: A B C D E
