# How well are we doing our Physical exercise?
***************************************************

###Summary:
Nowadays, to improve the quality of life  more and more people are practising sports.Prcatice not only improves the body's physical condition or fitness but also decreases the likelihood of many ills related to a sedentary life style, such as heart disease. But, it is not only the act or quantity of doing physical exercise that matters. Many people usually forget that physical exercise when done in the wrong way, may be less effective. But on the other hand they can also be very damaging causing undesirable injuries. Therefore, the right way to perform the exercise is considered a top priority.
This project is based on the work made by Veloso *et al.* "Qualitative Activity Recognition of Weight Lifting Exercises". The authors mounted sensors in six male volunteers to lift a relatively light dumbbell (1.25kg) weight. Five different ways (where only one is correct) to perform the lifting exercise were monitored by the sensors. The data collected were analysed and a machine learning model was built to assess the correctness and feedback to the user in real-time; increasing the likelihood of the exercise effectiveness. 

###Scope:
As part of the Coursera assessment, the report described here are restricted to answer the followings:

*   Predict the manner in which the exercise was done.
*   Show how cross validation was implemented
*   Analyse the sample error.
*   Discuss the assumptions made.
*   Apply the proposed model to predict 20 different test cases.

###Experiment description: 
Six (6) volunteers wore four (4) "9 Degrees of Freedom - Razor IMU". Each one of the Razor IMU is composed of three sensors: accelerometer, gyroscope and magnetometer, each one providing 3 degrees of freedom. Therefore, a total of 9 degrees of freedom per location (see the sketch). The four locations were: glove, armband, lumbar belt and dumbbell. 
<p align="center">
<image src= "https://raw.githubusercontent.com/anonymous-1618/ML/master/sketch.png" width="250">
  </p>

The volunteers were male participants aged between 20-28 years. They performed one set of 10 repetitions of the activity "Unilateral Dumbbell Biceps Curl" in five different "classes", one class is the right way (class A below) and the others the wrong way (classes B to E below):
- Class A - The right exercise (*i.e*., exactly according to the specification)
- Class B - Throwing the elbows to the front
- Class C - Lifting the dumbbell only halfway 
- Class D - Lowering the dumbbell only halfway
- Class E - Throwing the hips to the front

###Exploratory Data analysis:

**1.	Download the data from a provided URL.**

```R
    fileUrl <- "https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv"
    download.file(fileUrl, destfile="./pml-training.csv")
    
    fileUrl <- "https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv"
    download.file(fileUrl, destfile="./pml-testing.csv")
```

**2.	Load to R Global environment the files from their respective folders.**

```R
    pml.training <- read.csv("pml-training.csv",header=TRUE, na.strings=c("","NA","#DIV/0!"))
    pml.testing  <- read.csv("pml-testing.csv", header=TRUE, na.strings=c("","NA","#DIV/0!"))
```
**3.	Take a look at the data and analyse the data structure**

```R
str(pml.training, list.len=20)
```

```R
'data.frame':	19622 obs. of  160 variables:
 $ X                       : int  1 2 3 4 5 6 7 8 9 10 ...
 $ user_name               : Factor w/ 6 levels "adelmo","carlitos",..: 2 2 2 2 2 2 2 2 2 2 ...
 $ raw_timestamp_part_1    : int  1323084231 1323084231 1323084231 1323084232 1323084232 1323084232 1323084232 1323084232 1323084232 1323084232 ...
 $ raw_timestamp_part_2    : int  788290 808298 820366 120339 196328 304277 368296 440390 484323 484434 ...
 $ cvtd_timestamp          : Factor w/ 20 levels "02/12/2011 13:32",..: 9 9 9 9 9 9 9 9 9 9 ...
 $ new_window              : Factor w/ 2 levels "no","yes": 1 1 1 1 1 1 1 1 1 1 ...
 $ num_window              : int  11 11 11 12 12 12 12 12 12 12 ...
 $ roll_belt               : num  1.41 1.41 1.42 1.48 1.48 1.45 1.42 1.42 1.43 1.45 ...
 $ pitch_belt              : num  8.07 8.07 8.07 8.05 8.07 8.06 8.09 8.13 8.16 8.17 ...
 $ yaw_belt                : num  -94.4 -94.4 -94.4 -94.4 -94.4 -94.4 -94.4 -94.4 -94.4 -94.4 ...
 $ total_accel_belt        : int  3 3 3 3 3 3 3 3 3 3 ...
 $ kurtosis_roll_belt      : num  NA NA NA NA NA NA NA NA NA NA ...
 $ kurtosis_picth_belt     : num  NA NA NA NA NA NA NA NA NA NA ...
 $ kurtosis_yaw_belt       : logi  NA NA NA NA NA NA ...
 $ skewness_roll_belt      : num  NA NA NA NA NA NA NA NA NA NA ...
 $ skewness_roll_belt.1    : num  NA NA NA NA NA NA NA NA NA NA ...
 $ skewness_yaw_belt       : logi  NA NA NA NA NA NA ...
 $ max_roll_belt           : num  NA NA NA NA NA NA NA NA NA NA ...
 $ max_picth_belt          : int  NA NA NA NA NA NA NA NA NA NA ...
 $ max_yaw_belt            : num  NA NA NA NA NA NA NA NA NA NA ...
  [list output truncated]
```

Raw data variables for each sensor per location are recorded as follows:
*   Triaxial accelerometer:
    -   3 (x,y,z) x 4 (locations) = 12  
*   Triaxial gyroscope:
    -   3 (x,y,z) x 4 (locations) = 12  
*   Triaxial  magnetometer:
    -   3 (x,y,z) x 4 (locations) = 12  

A total of 36 raw data variables were then recorded.

"Fusion" variables obtained from the raw data variables were also recorded:
*   Three Rotational angles (roll, pitch, yaw) per location:
    -   3  x 4 (locations) = 12
*   Total acceleration per location:
    -   1  x 4 (locations) = 4

The data froam accelerometer, magnetometer and gyroscope are "fused" by using a Direction Cosine Matrix (DCM) algorithm in the Razor IMU, that means a total of 16 "fused" variables are recorded. Furthermore, in some of the "fused" variables, the following *descriptive statistics* variables ("stat") were recorded:
*   Max, Min, standard deviation(stddev), variance(var), average(avg) of rotational angles:
    -   5 x 12 = 60      
*   Amplitude of rotational angles (amplitude):
    -   3 x 4  = 12
*   kurtosis and skewness of rotational angles:
    -   2 x 12 = 24
*   variance(var) of total acceleration:
    -   1 x 4  = 4

A total of 100 "stat" variables.

The remainder variables:
*   Volunteer ID (user_name)
*   Exercise type (class)
*   Index or observable number (X)
*   time & date (cvtd_timestamp)
*   Four (4) time variables (raw_timestample_part_1 & raw_timestample_part_1, new_window, num_window);

A total of 8 remainder variables.
In summary, the experiment recorded a total number of 160 (36 + 16 + 100 + 8) variables.

Each individual performed 5 "classes" of exercises. Each exercise was repeated 10 times. All together a total of 19642 recorded observables were derived, which were then split by Coursera into two files:
*  train set to create the Model loaded as "pml.training" is a dataframe of 19622 x 160 
*  test set to answer the Quiz loaded as "pml.testing"  is a dataframe of 20 x 160

**4.  Data Cleaning**

The data was cleaned and the variables re-labelled by using *CamelCase* structure.
```R
    temp=names(pml.training)
    temp=gsub("^a","A",temp); temp=gsub("^c","C",temp); temp=gsub("^c","C",temp); temp=gsub("^g","G",temp); 
    temp=gsub("^k","K",temp); temp=gsub("^m","M",temp); temp=gsub("^n","N",temp); temp=gsub("^p","P",temp);
    temp=gsub("^r","R",temp); temp=gsub("^s","S",temp); temp=gsub("^t","T",temp); temp=gsub("^u","U",temp);
    temp=gsub("^v","V",temp); temp=gsub("^y","Y",temp); temp=gsub("_a","A",temp); temp=gsub("_b","B",temp);
    temp=gsub("_c","C",temp); temp=gsub("_d","D",temp); temp=gsub("_f","F",temp); temp=gsub("_g","G",temp);
    temp=gsub("_k","K",temp); temp=gsub("_m","M",temp); temp=gsub("_n","N",temp); temp=gsub("_p","P",temp);
    temp=gsub("_r","R",temp); temp=gsub("_s","S",temp); temp=gsub("_t","T",temp); temp=gsub("_u","U",temp);
    temp=gsub("_v","V",temp); temp=gsub("_x","X",temp); temp=gsub("_y","Y",temp); temp=gsub("_w","W",temp);
    temp=gsub("_z","Z",temp); temp=gsub("_1","1",temp); temp=gsub("_2","2",temp);
    TidyData = pml.training
    colnames(TidyData) = temp
```
Also, from the data structure we observe that there are missing values ("NA") in the data. We can check if the proportion is relevant using
```R
    mean(is.na(TidyData))
```
```R
    [1] 0.6131835
```
Over 61% of the data is missing, which is a very significant proportion. So, we can set a threshold to remove variables that contain more than 95% of "NA", for example:
```R
    NewTidyData1 = TidyData[, colSums(is.na(TidyData))/nrow(TidyData) < 0.95]
    dim(NewTidyData1)
``` 
```R
   [1] 19622    60
```
```R
   mean(is.na(NewTidyData1))
``` 
```R
   [1] 0
```
The `NewTidyData1` now does not contain any missing values. The first 20 rows of data structure of the `NewTidyData1`:
```R
 str(NewTidyData1, list.len=20)
``` 
```R
'data.frame':	19622 obs. of  60 variables:
 $ X                 : int  1 2 3 4 5 6 7 8 9 10 ...
 $ UserName          : Factor w/ 6 levels "adelmo","carlitos",..: 2 2 2 2 2 2 2 2 2 2 ...
 $ RawTimestampPart1 : int  1323084231 1323084231 1323084231 1323084232 1323084232 1323084232 1323084232 1323084232 1323084232 1323084232 ...
 $ RawTimestampPart2 : int  788290 808298 820366 120339 196328 304277 368296 440390 484323 484434 ...
 $ CvtdTimestamp     : Factor w/ 20 levels "02/12/2011 13:32",..: 9 9 9 9 9 9 9 9 9 9 ...
 $ NewWindow         : Factor w/ 2 levels "no","yes": 1 1 1 1 1 1 1 1 1 1 ...
 $ NumWindow         : int  11 11 11 12 12 12 12 12 12 12 ...
 $ RollBelt          : num  1.41 1.41 1.42 1.48 1.48 1.45 1.42 1.42 1.43 1.45 ...
 $ PitchBelt         : num  8.07 8.07 8.07 8.05 8.07 8.06 8.09 8.13 8.16 8.17 ...
 $ YawBelt           : num  -94.4 -94.4 -94.4 -94.4 -94.4 -94.4 -94.4 -94.4 -94.4 -94.4 ...
 $ TotalAccelBelt    : int  3 3 3 3 3 3 3 3 3 3 ...
 $ GyrosBeltX        : num  0 0.02 0 0.02 0.02 0.02 0.02 0.02 0.02 0.03 ...
 $ GyrosBeltY        : num  0 0 0 0 0.02 0 0 0 0 0 ...
 $ GyrosBeltZ        : num  -0.02 -0.02 -0.02 -0.03 -0.02 -0.02 -0.02 -0.02 -0.02 0 ...
 $ AccelBeltX        : int  -21 -22 -20 -22 -21 -21 -22 -22 -20 -21 ...
 $ AccelBeltY        : int  4 4 5 3 2 4 3 4 2 4 ...
 $ AccelBeltZ        : int  22 22 23 21 24 21 21 21 24 22 ...
 $ MagnetBeltX       : int  -3 -7 -2 -6 -6 0 -4 -2 1 -3 ...
 $ MagnetBeltY       : int  599 608 600 604 600 603 599 603 602 609 ...
 $ MagnetBeltZ       : int  -313 -311 -305 -310 -302 -312 -311 -313 -312 -308 ...
  [list output truncated]    
```
From the initial 160, there are a total of 60 variables. We can still reduce the number of variables **_assuming_** that the prediction is not time and individual dependent, that is, independent of the person and the time that the exercises was performed. Then, variables such as: *"UserName", "RawTimestampPart1", "RawTimestampPart2", "CvtdTimestamp", "NewWindow" and "NumWindow"* can be removed together with the index *"X"* variable:
```R
    library(dplyr)
    SelectData = select(NewTidyData1,-c(X:NumWindow))
    dim(SelectData)
```
```R    
    [1] 19622    53
```
Therefore, the number of variables is reduced to 53.

###Model Building

**5.	Propose Machine learning models base on the exploratory data**

I used the *caret* package in this work. Caret stands for Classification And REgression Training. It is a great toolkit to build  classification and regression models. Caret also provides the means for: Data preparation, Data splitting, Training a Model, Model evaluation, and  Variable selection.

**5.1.    Data Preparation - Removing redundant variables by a correlation matrix**

The data variables may be correlated to each other, which it may lead to redundancy in the model (*_assumption_*). By using `findCorrelation` from the *caret* package, we can obtain the correlation matrix between the data variables. The function 
can plot the entity data set to visualise those correlations (See Figure 1).  The plot makes it less difficult to choose the threshold of the correlation coefficient in  order to remove redundant variables. I chose an absolute value for correlation coefficient of 0.90 as the threshold. Seven (7) other variables can be dropped.
```R
    library(caret)
    library(corrplot)
    threshold   <-  0.90
    corMatrix   <-  cor(SelectData[,1:52])
    corrplot(corMatrix, method="square", order="hclust") # Plot the correlation matrix
    
    highCor <-  findCorrelation(corMatrix, threshold) #sub set those that thas correlation >, = 0.9, or < -0.9
    highCorRm   <-  row.names(corMatrix)[highCor]
    highCorRm
``` 
```R
    [1] "AccelBeltZ"    "RollBelt"       "AccelBeltY"     "AccelBeltX"     
        "GyrosDumbbellX" "GyrosDumbbellZ"  "GyrosArmX"
``` 
```R
    SelectData2 <- SelectData[, -highCor]
    dim(SelectData2)
``` 
```R
    [1] 19622    46
```
<p align="center">
  <img src="https://raw.githubusercontent.com/anonymous-1618/ML/master/Fig1.png">
  <b>Figure 1 - </b>Variables Correlation Map</b><br>
  </p>

**5.2.  Data splitting**

The *caret* function `createDataPartition` is used to randomly split the data set. I set the standard proportion of 60% of the data to be used for model training and 40% used for testing model performance.
```R
    library("caret")
    library("e1071")
    set.seed(1023)
    inTrain <- createDataPartition(SelectData2$Class, p = 0.6, list = FALSE)
    trainData <- SelectData2[inTrain,]
    testData <- SelectData2[-inTrain,]
```
**5.3.  Training a Model/Tuning Parameters/Variable Selection**

As the main question of this assignment is about classification, I choose Random Forest ("rf") to build the model. Tuning the model means to choose a set of parameters to be evaluated. Once the model and tuning parameters are chosen, the type of resampling needs to be opted for. Caret Package has tools to perform, k-fold cross-validation (once or repeated), leave-one-out cross-validation and bootstrap (**default**) resampling. Once the resampling was processed, the caret `train` function automatically chooses the best tuning parameters associated to the model.

```R
    set.seed(3023)
    modelFit2 = train(Classe ~., data=trainData, method="rf", prox=TRUE)
    modelFit2
```

```R
    11776 samples
    45 predictor
    5 classes: 'A', 'B', 'C', 'D', 'E' 

    No pre-processing
    Resampling: Bootstrapped (25 reps) 
    Summary of sample sizes: 11776, 11776, 11776, 11776, 11776, 11776, ... 
    Resampling results across tuning parameters:

    mtry  Accuracy   Kappa    
    2     0.9856378  0.9818241
    23    0.9874277  0.9840903
    45    0.9752460  0.9686771

    Accuracy was used to select the optimal model using  the largest value.
    The final value used for the model was mtry = 23. 
```
Accuracy is the default metric selected to be optimized in `train()` for categorical variables, in this case, "Classe". The proposed model shows that 23 predictors were needed to get the best accuracy on the training set. In order words, `caret` has optimised the model and concluded that selecting 23 most important predictors gives the best accuracy. The "out-of-bag" (OOB) error rate of the final model can be obtained by:

```R
modelFit2$finalModel
``` 

```R
    Call:
    randomForest(x = x, y = y, mtry = param$mtry, proximity = TRUE) 
               Type of random forest: classification
                     Number of trees: 500
    No. of variables tried at each split: 23

            OOB estimate of  error rate: 0.91%
    Confusion matrix:
         A    B    C    D    E    class.error
    A 3338    6    1    1    2  0.002986858
    B   22 2243   14    0    0  0.015796402
    C    0   14 2031    9    0  0.011197663
    D    1    0   25 1903    1  0.013989637
    E    0    1    2    8 2154  0.005080831
```

However the better error estimates is obtained by using the testing set (as I show in the Evaluation section). We can also calculate the variable importance in the model by using the caret package `varImp` function:

```R
    varImp(modelFit2, scale=TRUE)[[1]]
```
```R
                        Overall
    PitchBelt           71.88966676
    YawBelt            100.00000000
    TotalAccelBelt      15.13943598
    GyrosBeltX           4.10645743
    GyrosBeltY           4.29909507
    GyrosBeltZ          28.77409715
    MagnetBeltX         19.06090724
    MagnetBeltY         46.42143718
    MagnetBeltZ         29.86221750
    RollArm             13.36456325
    PitchArm             5.30347638
    YawArm              14.18155500
    TotalAccelArm        2.06975968
    GyrosArmY            7.26378887
    GyrosArmZ            0.00000000
    AccelArmX            6.52209427
    AccelArmY            6.87352864
    AccelArmZ            2.89186020
    MagnetArmX           8.30696563
    MagnetArmY           9.05507339
    MagnetArmZ           7.15700321
    RollDumbbell        27.13032649
    PitchDumbbell        6.15535378
    YawDumbbell         11.93242077
    TotalAccelDumbbell  18.91942501
    GyrosDumbbellY      10.63841552
    AccelDumbbellX       7.25232796
    AccelDumbbellY      27.33024178
    AccelDumbbellZ      19.02170061
    MagnetDumbbellX     28.23072724
    MagnetDumbbellY     51.95465439
    MagnetDumbbellZ     63.81343801
    RollForearm         50.97284067
    PitchForearm        86.88962083
    YawForearm           7.33542669
    TotalAccelForearm    2.41288158
    GyrosForearmX        0.02460776
    GyrosForearmY        3.64303343
    GyrosForearmZ        0.95471639
    AccelForearmX       21.54543983
    AccelForearmY        3.55059378
    AccelForearmZ       17.05528477
    MagnetForearmX       9.02319380
    MagnetForearmY       7.14137478
    MagnetForearmZ      15.57428210
```
We can also get the plot of the variable importance sorted in descending order by `plot(varImp(modelFit2, scale=TRUE))`. See Figure 2.
<p align="center">
  <img src="https://raw.githubusercontent.com/anonymous-1618/ML/master/Fig2.png">
  <b>Figure 2 - </b>Variable Importance</b><br>
  </p>

**5.4.    Model Evaluation**
The evalution of the training fit model can be made by using the caret `predict()`function on the testing set:
```R
    testPred <- predict(modelFit2, newdata = testData)
```
Now we can compare the prediction result of the testing set with the actual values by using the `confusionMatrix()` function:
```R
    confusionMatrix(testData$Classe,testPred)
```

```R
    Confusion Matrix and Statistics

              Reference
    Prediction     A    B    C    D    E
         A      2232    0    0    0    0
         B         5 1505    8    0    0
         C         0    5 1361    2    0
         D         0    1   22 1261    2
         E         0    1    3    4 1434

    Overall Statistics
                                          
               Accuracy : 0.9932          
                 95% CI : (0.9912, 0.9949)
    No Information Rate : 0.2851          
    P-Value [Acc > NIR] : < 2.2e-16       
                                          
                  Kappa : 0.9915          
    Mcnemar's Test P-Value : NA              

    Statistics by Class:

                         Class: A Class: B Class: C Class: D Class: E
    Sensitivity            0.9978   0.9954   0.9763   0.9953   0.9986
    Specificity            1.0000   0.9979   0.9989   0.9962   0.9988
    Pos Pred Value         1.0000   0.9914   0.9949   0.9806   0.9945
    Neg Pred Value         0.9991   0.9989   0.9949   0.9991   0.9997
    Prevalence             0.2851   0.1927   0.1777   0.1615   0.1830
    Detection Rate         0.2845   0.1918   0.1735   0.1607   0.1828
    Detection Prevalence   0.2845   0.1935   0.1744   0.1639   0.1838
    Balanced Accuracy      0.9989   0.9967   0.9876   0.9957   0.9987
```
Therefore the accuray estimated for the model fit is 0.9932 which means that the estimation error is ca. 0.68%.

###Model Parameters Effect:
1.  Selecting variables by Variable importance ranking:

I tried to improve the model by changing the number of predictors. Based on the previous results, I reduced more of the number of variables by creating a cutoff point for the best variables by their importance:

```R
    library(dplyr)
    temp =varImp(modelFit2, scale=TRUE)
    temp.df =as.data.frame(temp[[1]])
    temp.df = cbind(Variables = rownames(temp.df),temp.df)
    Impvar.names = as.vector(filter(temp.df, Overall > 10)[,1])     #filtering to variables with importance > 10%#
    Impvar.names
```
```R
    [1] "PitchBelt"          "YawBelt"            "TotalAccelBelt"     "GyrosBeltZ"        
    [5] "MagnetBeltX"        "MagnetBeltY"        "MagnetBeltZ"        "RollArm"           
    [9] "YawArm"             "RollDumbbell"       "YawDumbbell"        "TotalAccelDumbbell"
    [13] "GyrosDumbbellY"     "AccelDumbbellY"     "AccelDumbbellZ"     "MagnetDumbbellX"   
    [17] "MagnetDumbbellY"    "MagnetDumbbellZ"    "RollForearm"        "PitchForearm"      
    [21] "AccelForearmX"      "AccelForearmZ"      "MagnetForearmZ"  
```
```R
    selectVar = c(Impvar.names,"Classe")
    trainDataImp = trainData[,Impvar.names]
    modelFit2a = train(Classe ~., data=trainDataImp, method="rf", prox=TRUE)
```
```R
    modelFit2a = train(Classe ~., data=trainDataImp, method="rf", prox=TRUE)
    modelFit2a
```
```R
    Random Forest 
        
    11776 samples
       23 predictor
        5 classes: 'A', 'B', 'C', 'D', 'E' 
        
    No pre-processing
    Resampling: Bootstrapped (25 reps) 
    Summary of sample sizes: 11776, 11776, 11776, 11776, 11776, 11776, ... 
    Resampling results across tuning parameters:
        
      mtry  Accuracy   Kappa    
       2    0.9815209  0.9765947
      12    0.9816863  0.9768206
      23    0.9697268  0.9616665
        
    Accuracy was used to select the optimal model using  the largest value.
    The final value used for the model was mtry = 12.
```
```R
   modelFit2a$finalModel
```
```R
    Call:
    randomForest(x = x, y = y, mtry = param$mtry, proximity = TRUE) 
                   Type of random forest: classification
                         Number of trees: 500
    No. of variables tried at each split: 12
        
            OOB estimate of  error rate: 1.07%
    Confusion matrix:
         A    B    C    D    E class.error
    A 3337    7    1    1    2 0.003285544
    B   22 2230   24    1    2 0.021500658
    C    0   15 2021   18    0 0.016066212
    D    1    1   19 1907    2 0.011917098
    E    0    1    3    6 2155 0.004618938
```

```R
    testPred <- predict(modelFit2a, newdata = testData)
    confusionMatrix(testData$Classe,testPred)
```

```R
    Confusion Matrix and Statistics
        
              Reference
    Prediction    A    B    C    D    E
             A 2231    1    0    0    0
             B    5 1499   13    1    0
             C    0    4 1355    9    0
             D    0    1   21 1260    4
             E    0    0    3    5 1434
        
    Overall Statistics
                                                  
                   Accuracy : 0.9915          
                     95% CI : (0.9892, 0.9934)
        No Information Rate : 0.285           
        P-Value [Acc > NIR] : < 2.2e-16       
                                                  
                      Kappa : 0.9892          
        Mcnemar's Test P-Value : NA              
        
        Statistics by Class:
        
                             Class: A Class: B Class: C Class: D Class: E
        Sensitivity            0.9978   0.9960   0.9734   0.9882   0.9972
        Specificity            0.9998   0.9970   0.9980   0.9960   0.9988
        Pos Pred Value         0.9996   0.9875   0.9905   0.9798   0.9945
        Neg Pred Value         0.9991   0.9991   0.9943   0.9977   0.9994
        Prevalence             0.2850   0.1918   0.1774   0.1625   0.1833
        Detection Rate         0.2843   0.1911   0.1727   0.1606   0.1828
        Detection Prevalence   0.2845   0.1935   0.1744   0.1639   0.1838
        Balanced Accuracy      0.9988   0.9965   0.9857   0.9921   0.9980
```        
In this trial, the number of predictors selected in the final model was again decreased to **12 predictors**. This makes the model less complex. However, the OOB error rate (1.07%) and the test accuracy (0.9915) are slightly poorer than the original model. Also, the calculation time increased by half hour (ca. 5.5 h). 


2.  Cross-Validation type:

I also worked with 10-fold cross-validation (k=10). This resampling in general was much faster than bootstrap, that is, roughly ten-fold faster. As we can see in the results shown below, the accuracy is very similar to the "rf" but the OOB error is smaller that the original model. The following calculation uses the data with an importance variable cutoff of 10%.
```R
    set.seed(1825)
    fitControl <- trainControl( method = "cv", number = 10)
    modelFit2c <- train(Classe ~ ., data = trainDataImp, method="rf", trControl = fitControl)
    modelFit2c
```

```R
    Random Forest 
    
    11776 samples
       23 predictor
        5 classes: 'A', 'B', 'C', 'D', 'E' 
    
    No pre-processing
    Resampling: Cross-Validated (10 fold) 
    Summary of sample sizes: 10598, 10598, 10599, 10599, 10598, 10598, ... 
    Resampling results across tuning parameters:
    
      mtry  Accuracy   Kappa    
       2    0.9862432  0.9825951
      12    0.9872615  0.9838864
      23    0.9783446  0.9726068
    
    Accuracy was used to select the optimal model using  the largest value.
    The final value used for the model was mtry = 12. 
```

```R
  modelFit2c$finalModel

  Call:
   randomForest(x = x, y = y, mtry = param$mtry) 
                 Type of random forest: classification
                       Number of trees: 500
  No. of variables tried at each split: 12
  
          OOB estimate of  error rate: 1.14%
  Confusion matrix:
       A    B    C    D    E class.error
  A 3335    9    1    1    2 0.003882915
  B   24 2231   24    0    0 0.021061869
  C    0   21 2017   16    0 0.018013632
  D    1    1   18 1908    2 0.011398964
  E    0    1    4    9 2151 0.006466513
```

```R
  testPred <- predict(modelFit2c, newdata = testData)
  confusionMatrix(testData$Classe,testPred)
```

```R
    Confusion Matrix and Statistics
    
              Reference
    Prediction    A    B    C    D    E
             A 2231    1    0    0    0
             B    7 1498   13    0    0
             C    0    4 1354   10    0
             D    0    2   18 1263    3
             E    0    1    4    3 1434
    
    Overall Statistics
                                              
                   Accuracy : 0.9916          
                     95% CI : (0.9893, 0.9935)
        No Information Rate : 0.2852          
        P-Value [Acc > NIR] : < 2.2e-16       
                                              
                      Kappa : 0.9894          
     Mcnemar's Test P-Value : NA              
    
    Statistics by Class:
    
                         Class: A Class: B Class: C Class: D Class: E
    Sensitivity            0.9969   0.9947   0.9748   0.9898   0.9979
    Specificity            0.9998   0.9968   0.9978   0.9965   0.9988
    Pos Pred Value         0.9996   0.9868   0.9898   0.9821   0.9945
    Neg Pred Value         0.9988   0.9987   0.9946   0.9980   0.9995
    Prevalence             0.2852   0.1919   0.1770   0.1626   0.1832
    Detection Rate         0.2843   0.1909   0.1726   0.1610   0.1828
    Detection Prevalence   0.2845   0.1935   0.1744   0.1639   0.1838
    Balanced Accuracy      0.9983   0.9958   0.9863   0.9932   0.9983
```
Performing the same calculation but now using the original data set with 46 variables (no cutoff) and comparing with the original modelFit2, we can compare the effect of only 10-fold cross-validation (k=10).
```R
    set.seed(10025)
    fitControl <- trainControl( method = "cv", number = 10)
    modelFit2f <- train(Classe ~ ., data = trainData, method="rf", trControl = fitControl)
    modelFit2f
```
```R
    Random Forest 
    
    11776 samples
       45 predictor
        5 classes: 'A', 'B', 'C', 'D', 'E' 
    
    No pre-processing
    Resampling: Cross-Validated (10 fold) 
    Summary of sample sizes: 10599, 10599, 10599, 10598, 10598, 10598, ... 
    Resampling results across tuning parameters:
    
      mtry  Accuracy   Kappa    
       2    0.9889613  0.9860343
      23    0.9898949  0.9872163
      45    0.9791109  0.9735707
    
    Accuracy was used to select the optimal model using  the largest value.
    The final value used for the model was mtry = 23. 
```

```R
    modelFit2f$finalModel
``` 

```R
    Call:
     randomForest(x = x, y = y, mtry = param$mtry) 
                   Type of random forest: classification
                         Number of trees: 500
    No. of variables tried at each split: 23
    
            OOB estimate of  error rate: 0.86%
    Confusion matrix:
         A    B    C    D    E class.error
    A 3340    5    1    0    2 0.002389486
    B   23 2244   11    0    1 0.015357613
    C    0   14 2031    9    0 0.011197663
    D    1    1   23 1904    1 0.013471503
    E    0    0    2    7 2156 0.004157044
```

```R
    testPred <- predict(modelFit2f, newdata = testData)
    confusionMatrix(testData$Classe,testPred)
```

```R
    Confusion Matrix and Statistics
    
              Reference
    Prediction    A    B    C    D    E
             A 2232    0    0    0    0
             B    6 1503    9    0    0
             C    0    5 1362    1    0
             D    0    0   24 1260    2
             E    0    0    3    5 1434
    
    Overall Statistics
                                              
                   Accuracy : 0.993           
                     95% CI : (0.9909, 0.9947)
        No Information Rate : 0.2852          
        P-Value [Acc > NIR] : < 2.2e-16       
                                              
                      Kappa : 0.9911          
     Mcnemar's Test P-Value : NA              
    
    Statistics by Class:
    
                         Class: A Class: B Class: C Class: D Class: E
    Sensitivity            0.9973   0.9967   0.9742   0.9953   0.9986
    Specificity            1.0000   0.9976   0.9991   0.9960   0.9988
    Pos Pred Value         1.0000   0.9901   0.9956   0.9798   0.9945
    Neg Pred Value         0.9989   0.9992   0.9944   0.9991   0.9997
    Prevalence             0.2852   0.1922   0.1782   0.1614   0.1830
    Detection Rate         0.2845   0.1916   0.1736   0.1606   0.1828
    Detection Prevalence   0.2845   0.1935   0.1744   0.1639   0.1838
    Balanced Accuracy      0.9987   0.9972   0.9867   0.9957   0.9987
```
This result is very welcoming because the accuracy is very similar to modelFit2 but the OOB error rate is small (0.86% vs 0.91%). But the most relevant is the modelling time. The CV resampling reduced the time from 5 hours to 30 minutes!
 

3. Change the number of trees

We can obtain the relationship model error of classification and number of trees in the "Random Forest" model. For instance, the model modelFit2c (10-fold cross-validation with importance variable cutoff 10%) has the error related to the number of trees shown in Figure 3:
```R
    library(reshape)
    df.melted <- melt(modelFit2c$finalModel$err.rate, id = "ntree")
    names(df.melted) = c('ntree','Classe','Error')
    ggplot(data = df.melted, aes(x = ntree, y = Error, color = Classe)) 
    +   geom_line(size=1)
```    
 <p align="center">
  <img src="https://raw.githubusercontent.com/anonymous-1618/ML/master/Fig3.png">
  <b>Figure 3 - </b>Random Forest Model Error per #tree</b><br>
  </p>

We can see in Figure 3 that the error in all of them stabilised early than ntree=500 (default). So, by trying a model fit with ntree=200, we have:
```R
  set.seed(2825)
  fitControl <- trainControl( method = "cv", number = 10)
  modelFit2b <- train(Classe ~ ., data = trainDataImp, method="rf", ntree=200, trControl = fitControl)
  modelFit2b
```
```R
  Random Forest 
    
  11776 samples
     23 predictor
      5 classes: 'A', 'B', 'C', 'D', 'E' 
    
  No pre-processing
  Resampling: Cross-Validated (10 fold) 
  Summary of sample sizes: 10598, 10599, 10599, 10597, 10598, 10598, ... 
  Resampling results across tuning parameters:
    
    mtry  Accuracy   Kappa    
     2    0.9866676  0.9831335
    12    0.9871779  0.9837805
    23    0.9774111  0.9714243
    
  Accuracy was used to select the optimal model using  the largest value.
  The final value used for the model was mtry = 12. 
```

```R
  modelFit2b$finalModel
```

```R
  Call:
  randomForest(x = x, y = y, ntree = 200, mtry = param$mtry) 
               Type of random forest: classification
                       Number of trees: 200
  No. of variables tried at each split: 12
  
          OOB estimate of  error rate: 1.26%
  Confusion matrix:
       A    B    C    D    E class.error
  A 3328   14    2    2    2 0.005973716
  B   24 2230   23    1    1 0.021500658
  C    0   20 2016   18    0 0.018500487
  D    1    0   20 1902    7 0.014507772
  E    0    0    3   10 2152 0.006004619
```

```R
  testPred <- predict(modelFit2b, newdata = testData)
  confusionMatrix(testData$Classe,testPred)
```

```R
  Confusion Matrix and Statistics
    
              Reference
    Prediction    A    B    C    D    E
             A 2230    2    0    0    0
             B    6 1495   16    1    0
             C    0    4 1354   10    0
             D    0    0   20 1263    3
             E    0    0    5    5 1432
    
    Overall Statistics
                                              
                   Accuracy : 0.9908          
                     95% CI : (0.9885, 0.9928)
        No Information Rate : 0.285           
        P-Value [Acc > NIR] : < 2.2e-16       
                                              
                      Kappa : 0.9884          
     Mcnemar's Test P-Value : NA              
    
    Statistics by Class:
    
                         Class: A Class: B Class: C Class: D Class: E
    Sensitivity            0.9973   0.9960   0.9706   0.9875   0.9979
    Specificity            0.9996   0.9964   0.9978   0.9965   0.9984
    Pos Pred Value         0.9991   0.9848   0.9898   0.9821   0.9931
    Neg Pred Value         0.9989   0.9991   0.9937   0.9976   0.9995
    Prevalence             0.2850   0.1913   0.1778   0.1630   0.1829
    Detection Rate         0.2842   0.1905   0.1726   0.1610   0.1825
    Detection Prevalence   0.2845   0.1935   0.1744   0.1639   0.1838
    Balanced Accuracy      0.9985   0.9962   0.9842   0.9920   0.9982
```
    
The accuracy did not change by much, but the OOB error rate was the highest of all the evaluated models (1.26%). On the other hand, it was the fitted model that took less time (~ 6 minutes). 
    
###Predicting the 20 test cases:
After tidying up the test data set (pml.testing), we can estimate the exercise Classe of each one of the 20 test cases.
```R
  temp=names(pml.testing)
  temp=gsub("^a","A",temp); temp=gsub("^c","C",temp); temp=gsub("^c","C",temp); temp=gsub("^g","G",temp); 
  temp=gsub("^k","K",temp); temp=gsub("^m","M",temp); temp=gsub("^n","N",temp); temp=gsub("^p","P",temp);
  temp=gsub("^r","R",temp); temp=gsub("^s","S",temp); temp=gsub("^t","T",temp); temp=gsub("^u","U",temp);
  temp=gsub("^v","V",temp); temp=gsub("^y","Y",temp); temp=gsub("_a","A",temp); temp=gsub("_b","B",temp);
  temp=gsub("_c","C",temp); temp=gsub("_d","D",temp); temp=gsub("_f","F",temp); temp=gsub("_g","G",temp);
  temp=gsub("_k","K",temp); temp=gsub("_m","M",temp); temp=gsub("_n","N",temp); temp=gsub("_p","P",temp);
  temp=gsub("_r","R",temp); temp=gsub("_s","S",temp); temp=gsub("_t","T",temp); temp=gsub("_u","U",temp);
  temp=gsub("_v","V",temp); temp=gsub("_x","X",temp); temp=gsub("_y","Y",temp); temp=gsub("_w","W",temp);
  temp=gsub("_z","Z",temp); temp=gsub("_1","1",temp); temp=gsub("_2","2",temp);
  TidyDataTest = pml.testing
  colnames(TidyDataTest) = tempt
  NewTidyDataTest = TidyDataTest[, colSums(is.na(TidyDataTest))/nrow(TidyDataTest) < 0.95]
  dim(NewTidyDataTest)
```

```R
  [1] 20 60
```
```R
  library(dplyr)
  CaseDataTest = select(NewTidyDataTest,-c(X:NumWindow))
  dim(CaseDataTest)
  [1] 20 53
```

```R
  predict(modelFit2, newdata = CaseDataTest)
```
```R
 [1] B A B A A E D B A A B C B A E E A B B B
Levels: A B C D E
```
I have also evaluated the other 4 models:  modelFit2a, modelFit2b, modelFit2c and modelFit2f. These models presented slightly different inaccuracies and as expected, all gave the same results.

###Conclusions:
In this report, how to download the data is shown and also how to review it by analysing the data structure. The data was also tidied up and reorganised for better variable description (CamelCase). Correlations between the variables to reduce the model complexity were performed. The main assumptions made for the proposed model is to be person-independent and time-independent. The initial 160 variables were reduced to 46. Final models show that only 12 predictors could be used. Also, how to build a Machine Learning Model by using *caret* package was shown, along with how the parameters in the `train()` function can affect the model accuracy and fitting time. For instance, k-fold cross validation gave similar results in terms of accuracy but the modelling of the training set was much faster than by using boostrap resampling. In all the cases, the number of predictors used in the final model was the same, that is, 23. The best sample error obtained was 0.68% with the original model. Finally, by using the Random Forest method, all the models in this work predict the same result in the 20 test cases, that is:
```R
  table(predict(modelFit2f, newdata = CaseDataTest))
```
```R
A B C D E 
7 8 1 1 3 
```
Only 7 out of 20 shows the right way to perform the dumbbell lifting exercise.



***************************************************
####Reference:

https://perceptual.mpi-inf.mpg.de/files/2013/03/velloso13_ah.pdf

https://www.sparkfun.com/products/10736

http://web.ipac.caltech.edu/staff/fmasci/home/astro_refs/BuildingPredictiveModelsR_caret.pdf

http://topepo.github.io/caret/training.html

http://www.sthda.com/english/wiki/visualize-correlation-matrix-using-correlogram

https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet#hr

http://hplgit.github.io/teamods/bitgit/html-github/._main_bitgit002.html
