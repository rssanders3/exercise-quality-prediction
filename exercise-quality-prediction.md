Exercise Quality Prediction
========================================================
###### By: Robert Sanders

# Executive Summary
Using devices such as Jawbone Up, Nike FuelBand, and Fitbit, a group of enthusists conducted experiments with 6 test subjects. These test subjects were asked to perform exercises with a belt, forearm, arm, and dumbell with accelerometers in them and were also asked to perform those exercises correctly and incorreclty in 5 differnt ways. The purpose of this experiment is to try and predict exactly how well the subject did using the accelerometer data. More information is available from the website here: http://groupware.les.inf.puc-rio.br/har (see the section on the Weight Lifting Exercise Dataset). 

Using this data I was able to build a predictive model which was **99.97%** accurate. This equated to an error of **0.03%**. This predictive model was generated using 70% of the original data that I had access to, tested on 15% of the same data set and then finally cross validated on the final 15% of the dataset to confirm accuracy.

# Loading and Preprocessing Data and Dependencies

### Loading Dependencies and Required Packages
First we will load in the packages that we need to train and test the model.


```r
library(ggplot2)
library(caret)
library(randomForest)
```

### Loading Data
Now we will load in the training dataset.

If you would like more information on the data set, you can visit the following link:
http://groupware.les.inf.puc-rio.br/har


```r
csvData = read.csv("pml-training.csv")
```

### Preprocessing the Data
After examining the data I found a number of columns had a large number of NA's. I deemed it necessary to remove these columns as they would not add value to the prediction model.


```r
fieldsToExclude = c(
  
    #Unusable Fields:
  "X","user_name","raw_timestamp_part_1","raw_timestamp_part_2","cvtd_timestamp","new_window",
  
    #NA Fields:
  "max_roll_belt","max_picth_belt","min_roll_belt","min_pitch_belt","amplitude_roll_belt",
  "amplitude_pitch_belt","var_total_accel_belt","avg_roll_belt","stddev_roll_belt",
  "var_roll_belt","avg_pitch_belt","stddev_pitch_belt","var_pitch_belt","avg_yaw_belt",
  "stddev_yaw_belt	var_yaw_belt","var_accel_arm","avg_roll_arm","stddev_roll_arm",
  "var_roll_arm","avg_pitch_arm","stddev_pitch_arm","var_pitch_arm","avg_yaw_arm",
  "stddev_yaw_arm","var_yaw_arm","max_roll_arm","max_picth_arm","max_yaw_arm",
  "min_roll_arm","min_pitch_arm","min_yaw_arm","amplitude_roll_arm","amplitude_pitch_arm",
  "amplitude_yaw_arm","max_roll_dumbbell","max_picth_dumbbell","min_roll_dumbbell",
  "min_pitch_dumbbell","amplitude_roll_dumbbell","amplitude_pitch_dumbbell",
  "var_accel_dumbbell","avg_roll_dumbbell","stddev_roll_dumbbell","var_roll_dumbbell",
  "avg_pitch_dumbbell","stddev_pitch_dumbbell","amplitude_roll_forearm",
  "amplitude_pitch_forearm","var_pitch_dumbbell","avg_yaw_dumbbell","stddev_yaw_dumbbell",
  "var_yaw_dumbbell","max_roll_forearm","max_picth_forearm","min_roll_forearm",
  "var_accel_forearm","avg_roll_forearm","stddev_roll_forearm","var_roll_forearm",
  "avg_pitch_forearm","stddev_pitch_forearm","var_pitch_forearm","avg_yaw_forearm",
  "stddev_yaw_forearm","var_yaw_forearm","min_pitch_forearm"
)

csvDataFiltered = csvData[, !names(csvData) %in% fieldsToExclude]
```

In addition, I also found that some columns were loaded in as factors because of how little variance there were in the numbers. I fixed this by converting the columns values to numeric.


```r
#Convert Fields to numeric
for(i in 1:length(csvDataFiltered)) {
  if(!is.numeric(csvDataFiltered[[i]]) && !names(csvDataFiltered)[i] %in% c("classe")) {
    csvDataFiltered[i] = as.numeric(csvDataFiltered[[i]])
  } 
}
```

# Splitting Data
Now I need to split the training set into sub-data sets so that i can train the model and perform some cross validation on that model to ensure that its accurate. I decided to break it into 3 data sets. The name, size and description of each dataset can be seen in the table bellow:

| Dataset Name | Percent of original Dataset | Description                                           |
|--------------|-----------------------------|-------------------------------------------------------|
| Training     | 70%                         | Used to train the predictive model                    |
| Testing      | 15%                         | Used to test and re-test the model                    |
| Heldback     | 15%                         | Final test for the model to get a more accurate score |


```r
inTrain = createDataPartition(csvDataFiltered$classe, p=0.70)[[1]]
training = csvDataFiltered[inTrain, ] #70% of original dataset
testingAndHeldback = csvDataFiltered[-inTrain, ]
inTesting = createDataPartition(testingAndHeldback$classe, p=0.5)[[1]]
testing = testingAndHeldback[inTesting, ] #15% of original dataset
heldback = testingAndHeldback[-inTesting, ] #15% of original dataset
```

The final size of the data sets ended up being:


```r
print(length(training[[1]]))
```

```
## [1] 13737
```

```r
print(length(testing[[1]]))
```

```
## [1] 2943
```

```r
print(length(heldback[[1]]))
```

```
## [1] 2942
```

# Train Model
### Training
Now I'm going to train the model using the Random Forest Classifier. I've found Random Forest to be one of the best first round classifiers to go with. While training the classifier took a very long time to complete, I still found it to produce the highset prediciton rate.

For this paper, I wrote some code to save the model to an RData file so that it could be loaded later and utilized for real prediciton. The code bellow, loads the model if it aleady exists in the File System. If it doesn't then it trains the modeling using the Training data set.


```r
modelFileName = "modelFit.RData"
if(file.exists(modelFileName)) {
  load(file=modelFileName) #Save modelFit Object from RData file
}else {
  set.seed(975)
  modelFit = train(classe ~ . , data=training, method="rf")
  save(modelFit, file=modelFileName) #Save modelFit Object to RData file
}
```
### Final Model
Bellow we can see some initial results around the final model. We can see that we end up with a very low Out-of-Bag error rate of **0.24%**. This gives hope that our model is very accurate, but of course we need to confirm this.


```r
print(modelFit$finalModel)
```

```
## 
## Call:
##  randomForest(x = x, y = y, mtry = param$mtry) 
##                Type of random forest: classification
##                      Number of trees: 500
## No. of variables tried at each split: 44
## 
##         OOB estimate of  error rate: 0.24%
## Confusion matrix:
##      A    B    C    D    E class.error
## A 3904    1    0    0    1    0.000512
## B    6 2648    3    1    0    0.003762
## C    0    4 2391    1    0    0.002087
## D    0    0    9 2242    1    0.004440
## E    0    0    0    6 2519    0.002376
```

# Predicting on Test Set
### Generate Predictions
Now I'm going to try to predict using the model I just trained on the testing set:


```r
predictions = predict(modelFit, newdata=testing)
```

### Generate Confusion Matrix
Now I'm going to generate a confusion matrix so that we can see how well the model did:


```r
confusionMatrix(predictions, testing$classe)
```

```
## Confusion Matrix and Statistics
## 
##           Reference
## Prediction   A   B   C   D   E
##          A 837   1   0   0   0
##          B   0 569   0   0   0
##          C   0   0 513   0   0
##          D   0   0   0 482   0
##          E   0   0   0   0 541
## 
## Overall Statistics
##                                     
##                Accuracy : 1         
##                  95% CI : (0.998, 1)
##     No Information Rate : 0.284     
##     P-Value [Acc > NIR] : <2e-16    
##                                     
##                   Kappa : 1         
##  Mcnemar's Test P-Value : NA        
## 
## Statistics by Class:
## 
##                      Class: A Class: B Class: C Class: D Class: E
## Sensitivity             1.000    0.998    1.000    1.000    1.000
## Specificity             1.000    1.000    1.000    1.000    1.000
## Pos Pred Value          0.999    1.000    1.000    1.000    1.000
## Neg Pred Value          1.000    1.000    1.000    1.000    1.000
## Prevalence              0.284    0.194    0.174    0.164    0.184
## Detection Rate          0.284    0.193    0.174    0.164    0.184
## Detection Prevalence    0.285    0.193    0.174    0.164    0.184
## Balanced Accuracy       1.000    0.999    1.000    1.000    1.000
```

As you can see from the confusion matrix, the model got a very good score. But above it lists the accuracy as 1, or 100%. This isn't accurate as we have some entries that weren't predicted to be correct. So what exactly is our accuracy and inversly what is our error?

### Calculate Accuracy
We can calcualte our actual accuracy by taking the sum of the predictions that were correct and dividing that by the total number of predictions (as seeen bellow). This shows that the model was **99.97%** accurate while trying to predict on the test data set.


```r
accuracy = sum(predictions == testing$classe)/length(predictions)
print(paste(round(accuracy * 100, digits=2),"%", sep=""))
```

```
## [1] "99.97%"
```

### Calculate Sample Error
Inversly, we can predict our error by taking 1 minus our accuracy. This gives us the error percentage as **0.03%**


```r
print(paste(round((1 - accuracy) * 100, digits=2),"%", sep=""))
```

```
## [1] "0.03%"
```



# Predicting on Heldback Dataset for Cross Validation
Now that I belive my predictive model is very good, I need to perform some cross validation to ensure that I wasn't overfitting on the test data set and therefore come up with a more accurate evaluation on how well the model is at predicting the outcome.

### Generate Predictions
Now I'm going to predict on the heldback data set.


```r
predictions = predict(modelFit, newdata=heldback)
```

### Generate Confusion Matrix
Now I'm going to generate a confusion matrix so that we can see how well the model did:


```r
confusionMatrix(predictions, heldback$classe)
```

```
## Confusion Matrix and Statistics
## 
##           Reference
## Prediction   A   B   C   D   E
##          A 837   0   0   0   0
##          B   0 569   1   0   0
##          C   0   0 512   0   0
##          D   0   0   0 482   0
##          E   0   0   0   0 541
## 
## Overall Statistics
##                                     
##                Accuracy : 1         
##                  95% CI : (0.998, 1)
##     No Information Rate : 0.285     
##     P-Value [Acc > NIR] : <2e-16    
##                                     
##                   Kappa : 1         
##  Mcnemar's Test P-Value : NA        
## 
## Statistics by Class:
## 
##                      Class: A Class: B Class: C Class: D Class: E
## Sensitivity             1.000    1.000    0.998    1.000    1.000
## Specificity             1.000    1.000    1.000    1.000    1.000
## Pos Pred Value          1.000    0.998    1.000    1.000    1.000
## Neg Pred Value          1.000    1.000    1.000    1.000    1.000
## Prevalence              0.285    0.193    0.174    0.164    0.184
## Detection Rate          0.285    0.193    0.174    0.164    0.184
## Detection Prevalence    0.285    0.194    0.174    0.164    0.184
## Balanced Accuracy       1.000    1.000    0.999    1.000    1.000
```

You can see yet again that the predictive model does very well. However, we again get an accuracey of 1 or 100%. We know this to be not necessairly accurate because we see that the model got atleast some predictions incorrect from the confusion matrix.

### Calculate Accuracy
Again we can get our actual acuracy by taking the sum of the predictions that were correct and dividing that by the total number of predictions (as seeen bellow). This shows that the model was **99.97%** accurate while trying to predict on our heldback data set.


```r
accuracy = sum(predictions == heldback$classe)/length(predictions)
print(paste(round(accuracy * 100, digits=2),"%", sep=""))
```

```
## [1] "99.97%"
```

### Calculate Sample Error
We can again predict our error by taking 1 minus our accuracy. This gives us the percentage as **0.03%**


```r
print(paste(round((1 - accuracy) * 100, digits=2),"%", sep=""))
```

```
## [1] "0.03%"
```

#Conclusion
After performing Cross Validation on the Heldback data set, we can confirm that our model is **99.97%** accurate.

