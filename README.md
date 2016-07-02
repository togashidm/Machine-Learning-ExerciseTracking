# How well are we doing our Physical exercise?
***************************************************

###Summary:
Nowadays, to improve the life quality more and more peole are practicing sports. The practicing not only improve the body physical conditions or fitness but also decrease the likehood of many ills related to sedentarism, such as heart diseases. But, it is not only the act or quantity of doing physical exercise that matters. Many people usually forget that physical exercises when are done in the wrong way, they are, on one hand, less effective. On the other hand they can be very damaging causing undesirable injuries. Therefore, the right way to perform the exercise is considered a top priority.
This project is based on the work made by Veloso et al. "Qualitative Activity Recognition of Weight Lifting Exercises". The authors have mounted sensors in six male volunteers to lift a relatively light dumbbell (1.25kg). Five different ways (only one correct) to perform the lift exercise were monitored by the sensors. The data collected were analysed and a machine learning model was built to assess the correctness and feedbacking the user at real-time; increasing the likewood of the exercise effectiviness. 

###Scope:
As part of the Coursera assessment, the report described here are restricted to answer the followings:

*   Predict the manner in which the exercise was done.
*   Show how cross validation was implemented
*   Analyse the sample error.
*   Discuss the assumptions made.
*   Apply the proposed model to predict 20 different test cases.

###Experiment description: 
Six (6) volunteers weared four (4) "9 Degrees of Freedom - Razor IMU". Each one of Razor IMU is composed of three sensors: accelerometer, gyroscope and magnetometer, in which of them provides 3 degrees of freedom. Therefore, a total of 9 degrees of freedom per location. The four locations were:

1. glove
2. armband
3. lumbar belt
4. dumbbell 

The volunteers, then, performed one set of 10 repetitions of the activity "Unilateral Dumbbell Biceps Curl" in five different "classes":
- Class A - The rigth exercise (i.e., exactly according to the specification)
- Class B - Throwing the elbows to the front (i.e., wrong)
- Class C - Lifting the dumbbell only halfway (i.e., wrong)
- Class D - Lowering the dumbbell only halfway (i.e., wrong)
- Class E - Throwing the hips to the front (i.e., wrong).

The volunteers were male participants aged between 20-28 years and data were collected at a constant rate of 45Hz.

###Exploratory Data analysis:

####1.	Download the data from a provided URL.

    fileUrl <- "https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv"
    download.file(fileUrl, destfile="./pml-training.csv") 
    fileUrl <- "https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv"
    download.file(fileUrl, destfile="./pml-testing.csv") 

####2.	Load to R Global environment the files from their respective folders.

    pml.training <- read.csv("pml-training.csv",header=TRUE, na.strings=c("","NA","#DIV/0!"))
    pml.testing  <- read.csv("pml-testing.csv", header=TRUE, na.strings=c("","NA","#DIV/0!"))

####3.	Take a look at the data and analyse the data structure

    str(pml.training)
    
Raw data variables for each sensor per location recorded as follow:
*   Triaxial accelerometer:  3 (x,y,z) x 4 (locations) = 12
*   Triaxial gyroscope:      3 (x,y,z) x 4 (locations) = 12 
*   Triaxial  magnetometer:  3 (x,y,z) x 4 (locations) = 12 
A total of 36 raw data variables were then recorded.

It was recorded also "fusion" variables obtained form the raw data variables:
*   Three Rotational angles (roll, pitch, yaw) per location:     3  x 4 (locations) = 12
*   Total acceleration per location:                             1  x 4 (locations) = 4

Inside the Razor IMU, the data of accelerometer, magnetometer and gyroscope are "fused" by using a Direction Cosine Matrix (DCM) algorithm. Therefored, a total of 16 "fused" variables were then recorded.
Also, for some of "fused" variables, the following "stat" (or descriptive statistics) variables were recorded:
*   Max, Min, standard deviation(stddev), variance(var), average(avg) of rotational angles:     5 x 12 = 60      
*   Amplitude of rotational angles (amplitude):                                                 3 x 4  = 12
*   kurtosis and skewness of rotational angles:                                                 2 x 12 = 24
*   variance(var) of total acceleration:                                                        1 x 4  = 4
A total of 100 "stat" variables.

Other variables:
*   Volunteer ID (user_name)
*   Exercise type (class)
*   Index or observable number (X)
*   time & date (cvtd_timestamp)
*   4 time variables (raw_timestample_part_1 & raw_timestample_part_1, new_window, num_window)
A total of 8 other variables

Finally, the experiment recorded a total number of 160 (36 + 16 + 100 + 8) variables.
Each individual performed 5 "classes" of exercises. Each exercise was repeat 10 times. All together derived a total of 19642 recorded observables, which were then split by Coursera into two files:
*  train set to create the Model loaded as "pml.training" is a dataframe of 19622 x 160 
*  test set to answer the Quiz loaded as "pml.testing"  is a dataframe of 20 x 160

####4.  Clean the data by considering the rules of tidy data, and labelling the variables with descriptive names.
Cleaning the non-alphanumeric characters, and using CamelCase structure

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

There are missing values (i.e. 'NA') in the data. We can check if the proportion is relevant by:

    mean(is.na(TidyData))

Over 61% of the data is missed and this is very significant. So, we can set a threshold to remove variables that contain more than 95% of 'NA', for example:

    NewTidyData1 = TidyData[, colSums(is.na(TidyData))/nrow(TidyData) < 0.95]
    dim(NewTidyData1)
    mean(is.na(NewTidyData))

The NewTidyData now does not contain any missed value. Note that some over variables that were removed may contain observable values. So, we can create a second data set of the remainder of first data set (varibles with less than 95% of 'NA') but without observables that contain "NA": 

    temp= TidyData[, colSums(is.na(TidyData))/nrow(TidyData) > 0.95]

It is also possible that there are variables AND observables that only have 'NA', so:

    NewTidyData2 = temp[, colSums(is.na(temp))/nrow(temp) != 1]
    NewTidyData2 = na.omit(NewTidyData2)
    dim(NewTidyData2)

What are the variables that does not contain any observable values?

    setdiff(names(temp),names(NewTidyData2))
    
####5.	Now we are going to take a look at the structure of the NewTidyData1

    str(NewTidyData1)

There are a total of 60 variables. We can still reduce the number of variables *assuming* that the prediction is not time and individual dependent. Then, variables such as: "UserName", "RawTimestampPart1", "RawTimestampPart2", "CvtdTimestamp",      "NewWindow" and "NumWindow" can be removed together with the index "X" variable. Therefore, the number of variables is reduced to 53.

    library(dplyr)
    SelectData = select(NewTidyData1,-c(X:NumWindow))
    dim(SelectData)

####6.	Propose Machine learning models base on the exploratory data 

I have applied the caret package in this. Caret stands for Classification And REgression Training. It is a great toolkit for building classification models and regression models. Caret also provides means for:
– Data preparation
– Data splitting
- Training a Model
– Model evaluation
– Variable selection

#####Data Preparation - Removing redudant variables by a correlation matrix:
The data variables may be correlated to each other, which it may lead to rendundancy in the model. By using "findCorrelation" from the Caret R package, we can obtain the correlation matrix of between the data variables, and remove those variables with correlation coefficient larger than 0.9 (arbitrary threshold).

    library(caret)
    threshold <-0.90
    corMatrix<-cor(SelectData[,1:52])
    highCor <-findCorrelation(corMatrix, threshold)
    highCorRm <-row.names(corMatrix)[highCor]
    highCorRm
    [1] "AccelBeltZ"    "RollBelt"       "AccelBeltY"     "AccelBeltX"     "GyrosDumbbellX" "GyrosDumbbellZ" "GyrosArmX"

    SelectData2 <- SelectData[, -highCor]
    dim(SelectData2)

#####Data spliting:

The function createDataPartition is used to randomly split the data set. It is set 60% of the data to be used for model training and 40% used for testing model performance.

    library("caret")
    library("e1071")
    set.seed(123)
    inTrain <- createDataPartition(SelectData$Class, p = 0.6, list = FALSE)
    trainData <- SelectData[inTrain,]
    testData <- SelectData[-inTrain,]
    modelFit1 = train(Classe ~., data=trainData, method="rf", prox=TRUE)

    library("caret")
    set.seed(1023)
    inTrain <- createDataPartition(SelectData2$Class, p = 0.6, list = FALSE)
    trainData <- SelectData2[inTrain,]
    testData <- SelectData2[-inTrain,]
    modelFit2 = train(Classe ~., data=trainData, method="rf", prox=TRUE)

------------------------------------------------
####Training a Model/Tuning Parameters/Building the Final model

As the question is about classification, I choose Random Forest ("rf") the method to build the model. Tuning the model means to choose a set of parameters to be evaluated. Once the model and tuning parameters are choosen, the type of resampling (cross-validation) need to be opted. Caret Package is able to perfom, k-fold cross-validation (once or repeated), leave-one-out cross-validation and bootstrap resampling methods. I have choosen two type of resampling to evaluate the performance: Bootstrap (default) and k-fold cross-validation (once or repeated). Once the resampling is processed the caret "train" function automatically chooses the best tuning parameters associated to the model.


Using correlation to reduce number of variable and eliminating UserName

> modelFit2
    Random Forest 

    11776 samples
    45 predictor
    5 classes: 'A', 'B', 'C', 'D', 'E' 

    No pre-processing
    Resampling: Bootstrapped (25 reps) 
    Summary of sample sizes: 11776, 11776, 11776, 11776, 11776, 11776, ... 
    Resampling results across tuning parameters:

    mtry  Accuracy   Kappa    
    2    0.9856378  0.9818241
    23    0.9874277  0.9840903
    45    0.9752460  0.9686771

    Accuracy was used to select the optimal model using  the largest value.
    The final value used for the model was mtry = 23. 

> modelFit2$finalModel

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

    varImp(modelFit2, scale=TRUE)
    rf variable importance

    only 20 most important variables shown (out of 45)

                       Overall
    YawBelt             100.00
    PitchForearm         86.89
    PitchBelt            71.89
    MagnetDumbbellZ      63.81
    MagnetDumbbellY      51.95
    RollForearm          50.97
    MagnetBeltY          46.42
    MagnetBeltZ          29.86
    GyrosBeltZ           28.77
    MagnetDumbbellX      28.23
    AccelDumbbellY       27.33
    RollDumbbell         27.13
    AccelForearmX        21.55
    MagnetBeltX          19.06
    AccelDumbbellZ       19.02
    TotalAccelDumbbell   18.92
    AccelForearmZ        17.06
    MagnetForearmZ       15.57
    TotalAccelBelt       15.14
    YawArm               14.18

-------------------------------------------------

#####Model Evaluation

    testPred <- predict(modelFit2, newdata = testData)
    confusionMatrix(testData$Classe,testPred)

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

####### after model fit

    temp =varImp(modelFit2, scale=TRUE)
    temp.df =as.data.frame(temp[[1]])
    temp.df = cbind(Variables = rownames(temp.df),temp.df)
    ####temp.df = arrange(temp.df,desc(Overall))
    ####filtering to variables with importance > 10%
    Impvar.names = as.vector(filter(temp.df, Overall > 10)[,1])
    selectVar = c(Impvar.names,"Classe")
        
    > Impvar.names
         [1] "PitchBelt"          "YawBelt"            "TotalAccelBelt"     "GyrosBeltZ"        
         [5] "MagnetBeltX"        "MagnetBeltY"        "MagnetBeltZ"        "RollArm"           
         [9] "YawArm"             "RollDumbbell"       "YawDumbbell"        "TotalAccelDumbbell"
        [13] "GyrosDumbbellY"     "AccelDumbbellY"     "AccelDumbbellZ"     "MagnetDumbbellX"   
        [17] "MagnetDumbbellY"    "MagnetDumbbellZ"    "RollForearm"        "PitchForearm"      
        [21] "AccelForearmX"      "AccelForearmZ"      "MagnetForearmZ"  
        
    trainDataImp = trainData[,Impvar.names]
    modelFit2a = train(Classe ~., data=trainDataImp, method="rf", prox=TRUE)
    date()
        
    [1] "Tue Jun 28 23:19:44 2016"
    > modelFit2a = train(Classe ~., data=trainDataImp, method="rf", prox=TRUE)
    There were 50 or more warnings (use warnings() to see the first 50)
    >   date()
    [1] "Wed Jun 29 05:33:39 2016"
    >   save.image("~/machinelearning4.RData")
    >   modelFit2a
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
        
    >   modelFit2a$finalModel
        
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
        
        > testPred <- predict(modelFit2a, newdata = testData)
        > confusionMatrix(testData$Classe,testPred)
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
        
-------------------------------------------------------------


For the train function, the possible resampling methods (cross-validatios) are:

* bootstrapping,
* k-fold cross-validation,
* leave-one-out cross-validation,
* leave-group-out cross-validation (i.e., repeated splits without replacement).

#######using k-fold CV 
In this work two cross-validations will be evaluated: Boststrapping and k-fold cross-validation

#######Predictor importance
The generic function varImp can be used to characterize the general selection of predictors on
the model. The varImp function works with the following object classes: lm, mars, earth,
randomForest, gbm, mvr (in the pls package), rpart, RandomForest (from the party package),
pamrtrained, bagEarth, bagFDA, classbagg and regbagg. varImp also works with objects
produced by train, but this is a simple wrapper for the specic models previously listed

    date()
    set.seed(2825)
    fitControl <- trainControl( method = "cv", number = 10)
    modelFit2b <- train(Classe ~ ., data = trainDataImp, method="rf", ntree=200, trControl = fitControl)
    date()
    > date()
    [1] "Wed Jun 29 19:38:18 2016"
    > set.seed(2825)
    > fitControl <- trainControl( method = "cv", number = 10)
    > modelFit2b <- train(Classe ~ ., data = trainDataImp, method="rf", ntree=200, trControl = fitControl)
    > date()
    [1] "Wed Jun 29 19:44:06 2016"
    > 
    > modelFit2b
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
    
    
    
    > plot(modelFit2b$finalModel)
    > plot(modelFit2b)
    > testPred <- predict(modelFit2b, newdata = testData)
    > confusionMatrix(testData$Classe,testPred)
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
    
    
    > date()
    [1] "Wed Jun 29 19:38:18 2016"
    > set.seed(2825)
    > fitControl <- trainControl( method = "cv", number = 10)
    > modelFit2b <- train(Classe ~ ., data = trainDataImp, method="rf", ntree=200, trControl = fitControl)
    > date()
    [1] "Wed Jun 29 19:44:06 2016"
    > 
    > modelFit2b
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
    > plot(modelFit2b$finalModel)
    > plot(modelFit2b)
    > testPred <- predict(modelFit2b, newdata = testData)
    > confusionMatrix(testData$Classe,testPred)
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
    
    > date()
    [1] "Wed Jun 29 20:01:57 2016"
    > set.seed(1825)
    > fitControl <- trainControl( method = "cv", number = 10)
    > modelFit2c <- train(Classe ~ ., data = trainDataImp, method="rf", trControl = fitControl)
    > date()
    [1] "Wed Jun 29 20:19:25 2016"
    > 
    > modelFit2c
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
    > plot(modelFit2c$finalModel)
    > plot(modelFit2c)
    > testPred <- predict(modelFit2c, newdata = testData)
    > confusionMatrix(testData$Classe,testPred)
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
    
    > (modelFit2c$finalModel)
    
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
    
    
    date()
    set.seed(1325)
    fitControl <- trainControl( method = "cv", number = 3)
    modelFit2d <- train(Classe ~ ., data = trainDataImp, method="rf", trControl = fitControl)
    date()
    > date()
    [1] "Wed Jun 29 21:14:59 2016"
    > set.seed(1325)
    > fitControl <- trainControl( method = "cv", number = 3)
    > modelFit2d <- train(Classe ~ ., data = trainDataImp, method="rf", trControl = fitControl)
    > date()
    [1] "Wed Jun 29 21:18:30 2016"
    > 
    > modelFit2d
    Random Forest 
    
    11776 samples
       23 predictor
        5 classes: 'A', 'B', 'C', 'D', 'E' 
    
    No pre-processing
    Resampling: Cross-Validated (3 fold) 
    Summary of sample sizes: 7852, 7851, 7849 
    Resampling results across tuning parameters:
    
      mtry  Accuracy   Kappa    
       2    0.9825917  0.9779763
      12    0.9838655  0.9795928
      23    0.9737600  0.9668130
    
    Accuracy was used to select the optimal model using  the largest value.
    The final value used for the model was mtry = 12. 
    > modelFit2d$finalModel
    
    Call:
     randomForest(x = x, y = y, mtry = param$mtry) 
                   Type of random forest: classification
                         Number of trees: 500
    No. of variables tried at each split: 12
    
            OOB estimate of  error rate: 1.1%
    Confusion matrix:
         A    B    C    D    E class.error
    A 3336    8    1    1    2 0.003584229
    B   23 2230   24    1    1 0.021500658
    C    0   17 2024   13    0 0.014605648
    D    0    2   20 1905    3 0.012953368
    E    0    2    3    8 2152 0.006004619
    > testPred <- predict(modelFit2d, newdata = testData)
    > confusionMatrix(testData$Classe,testPred)
    Confusion Matrix and Statistics
    
              Reference
    Prediction    A    B    C    D    E
             A 2231    1    0    0    0
             B    6 1500   12    0    0
             C    0    4 1354   10    0
             D    0    2   20 1260    4
             E    0    1    5    4 1432
    
    Overall Statistics
                                              
                   Accuracy : 0.9912          
                     95% CI : (0.9889, 0.9932)
        No Information Rate : 0.2851          
        P-Value [Acc > NIR] : < 2.2e-16       
                                              
                      Kappa : 0.9889          
     Mcnemar's Test P-Value : NA              
    
    Statistics by Class:
    
                         Class: A Class: B Class: C Class: D Class: E
    Sensitivity            0.9973   0.9947   0.9734   0.9890   0.9972
    Specificity            0.9998   0.9972   0.9978   0.9960   0.9984
    Pos Pred Value         0.9996   0.9881   0.9898   0.9798   0.9931
    Neg Pred Value         0.9989   0.9987   0.9943   0.9979   0.9994
    Prevalence             0.2851   0.1922   0.1773   0.1624   0.1830
    Detection Rate         0.2843   0.1912   0.1726   0.1606   0.1825
    Detection Prevalence   0.2845   0.1935   0.1744   0.1639   0.1838
    Balanced Accuracy      0.9986   0.9959   0.9856   0.9925   0.9978
    
    
    date()
    set.seed(1025)
    fitControl <- trainControl( method = "cv", number = 20)
    modelFit2e <- train(Classe ~ ., data = trainDataImp, method="rf", trControl = fitControl)
    date()
    > date()
    [1] "Wed Jun 29 21:23:51 2016"
    > set.seed(1025)
    > fitControl <- trainControl( method = "cv", number = 20)
    > modelFit2e <- train(Classe ~ ., data = trainDataImp, method="rf", trControl = fitControl)
    > date()
    [1] "Wed Jun 29 21:56:27 2016"
    > 
    > modelFit2e
    Random Forest 
    
    11776 samples
       23 predictor
        5 classes: 'A', 'B', 'C', 'D', 'E' 
    
    No pre-processing
    Resampling: Cross-Validated (20 fold) 
    Summary of sample sizes: 11188, 11186, 11187, 11187, 11188, 11187, ... 
    Resampling results across tuning parameters:
    
      mtry  Accuracy   Kappa    
       2    0.9873475  0.9839927
      12    0.9881950  0.9850679
      23    0.9790233  0.9734655
    
    Accuracy was used to select the optimal model using  the largest value.
    The final value used for the model was mtry = 12. 
    > modelFit2e$finalModel
    
    Call:
     randomForest(x = x, y = y, mtry = param$mtry) 
                   Type of random forest: classification
                         Number of trees: 500
    No. of variables tried at each split: 12
    
            OOB estimate of  error rate: 1.1%
    Confusion matrix:
         A    B    C    D    E class.error
    A 3336    7    1    2    2 0.003584229
    B   23 2231   25    0    0 0.021061869
    C    0   19 2019   16    0 0.017039922
    D    1    2   19 1906    2 0.012435233
    E    0    0    3    8 2154 0.005080831
    
    > testPred <- predict(modelFit2e, newdata = testData)
    > confusionMatrix(testData$Classe,testPred)
    Confusion Matrix and Statistics
    
              Reference
    Prediction    A    B    C    D    E
             A 2232    0    0    0    0
             B    7 1494   16    1    0
             C    0    5 1353   10    0
             D    0    2   20 1260    4
             E    0    1    4    4 1433
    
    Overall Statistics
                                              
                   Accuracy : 0.9906          
                     95% CI : (0.9882, 0.9926)
        No Information Rate : 0.2854          
        P-Value [Acc > NIR] : < 2.2e-16       
                                              
                      Kappa : 0.9881          
     Mcnemar's Test P-Value : NA              
    
    Statistics by Class:
    
                         Class: A Class: B Class: C Class: D Class: E
    Sensitivity            0.9969   0.9947   0.9713   0.9882   0.9972
    Specificity            1.0000   0.9962   0.9977   0.9960   0.9986
    Pos Pred Value         1.0000   0.9842   0.9890   0.9798   0.9938
    Neg Pred Value         0.9988   0.9987   0.9938   0.9977   0.9994
    Prevalence             0.2854   0.1914   0.1775   0.1625   0.1832
    Detection Rate         0.2845   0.1904   0.1724   0.1606   0.1826
    Detection Prevalence   0.2845   0.1935   0.1744   0.1639   0.1838
    Balanced Accuracy      0.9984   0.9954   0.9845   0.9921   0.9979
    
    date()
    set.seed(10025)
    fitControl <- trainControl( method = "cv", number = 10)
    modelFit2f <- train(Classe ~ ., data = trainData, method="rf", trControl = fitControl)
    date()
    
    > date()
    [1] "Wed Jun 29 22:17:02 2016"
    > set.seed(10025)
    > fitControl <- trainControl( method = "cv", number = 10)
    > modelFit2f <- train(Classe ~ ., data = trainData, method="rf", trControl = fitControl)
    > date()
    [1] "Wed Jun 29 22:48:25 2016"
    > 
    > modelFit2f
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
    > modelFit2f$finalModel
    
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
    > testPred <- predict(modelFit2f, newdata = testData)
    > confusionMatrix(testData$Classe,testPred)
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
    > save.image("~/machinelearning5.RData")
    
    > confusionMatrix(modelFit2f)
    Cross-Validated (10 fold) Confusion Matrix 
    
    (entries are percentual average cell counts across resamples)
     
              Reference
    Prediction    A    B    C    D    E
             A 28.3  0.2  0.0  0.0  0.0
             B  0.1 19.0  0.1  0.0  0.0
             C  0.0  0.1 17.2  0.2  0.0
             D  0.0  0.0  0.1 16.2  0.1
             E  0.0  0.0  0.0  0.0 18.3
                                
     Accuracy (average) : 0.9899
    
    > confusionMatrix(modelFit2e)
    Cross-Validated (20 fold) Confusion Matrix 
    
    (entries are percentual average cell counts across resamples)
     
              Reference
    Prediction    A    B    C    D    E
             A 28.3  0.2  0.0  0.0  0.0
             B  0.1 18.9  0.1  0.0  0.0
             C  0.0  0.2 17.2  0.2  0.0
             D  0.0  0.0  0.1 16.2  0.1
             E  0.0  0.0  0.0  0.0 18.3
                                
     Accuracy (average) : 0.9882
    
    > confusionMatrix(modelFit2d)
    Cross-Validated (3 fold) Confusion Matrix 
    
    (entries are percentual average cell counts across resamples)
     
              Reference
    Prediction    A    B    C    D    E
             A 28.2  0.2  0.0  0.0  0.0
             B  0.1 18.9  0.2  0.0  0.0
             C  0.1  0.2 17.1  0.3  0.0
             D  0.0  0.0  0.2 16.0  0.1
             E  0.0  0.0  0.0  0.1 18.2
                                
     Accuracy (average) : 0.9839
    
    > confusionMatrix(modelFit2c)
    Cross-Validated (10 fold) Confusion Matrix 
    
    (entries are percentual average cell counts across resamples)
     
              Reference
    Prediction    A    B    C    D    E
             A 28.3  0.2  0.0  0.0  0.0
             B  0.1 18.9  0.2  0.0  0.0
             C  0.0  0.2 17.1  0.2  0.0
             D  0.0  0.0  0.2 16.2  0.1
             E  0.0  0.0  0.0  0.0 18.3
                                
     Accuracy (average) : 0.9873
    
    > confusionMatrix(modelFit2b)
    Cross-Validated (10 fold) Confusion Matrix 
    
    (entries are percentual average cell counts across resamples)
     
              Reference
    Prediction    A    B    C    D    E
             A 28.3  0.2  0.0  0.0  0.0
             B  0.1 18.9  0.2  0.0  0.0
             C  0.0  0.3 17.1  0.2  0.0
             D  0.0  0.0  0.2 16.2  0.1
             E  0.0  0.0  0.0  0.0 18.3
                                
     Accuracy (average) : 0.9872
    
    > confusionMatrix(modelFit2a)
    Bootstrapped (25 reps) Confusion Matrix 
    
    (entries are percentual average cell counts across resamples)
     
              Reference
    Prediction    A    B    C    D    E
             A 28.3  0.3  0.0  0.0  0.0
             B  0.2 18.6  0.2  0.0  0.0
             C  0.0  0.3 16.9  0.3  0.1
             D  0.0  0.0  0.2 16.0  0.1
             E  0.0  0.0  0.0  0.0 18.4
                                
     Accuracy (average) : 0.9817
    
    > confusionMatrix(modelFit2)
    Bootstrapped (25 reps) Confusion Matrix 
    
    (entries are percentual average cell counts across resamples)
     
              Reference
    Prediction    A    B    C    D    E
             A 28.4  0.3  0.0  0.0  0.0
             B  0.1 18.7  0.2  0.0  0.0
             C  0.0  0.2 17.1  0.2  0.0
             D  0.0  0.0  0.1 16.3  0.1
             E  0.0  0.0  0.0  0.0 18.3
                                
     Accuracy (average) : 0.9874
     
    
    install.packages("doParallel")
    library(parallel)
    library(doParallel)
    cluster <- makeCluster(detectCores() - 1) # convention to leave 1 core for OS
    registerDoParallel(cluster)
    
    date()
    set.seed(12025)
    fitControl <- trainControl(method = "repeatedcv", number = 10, repeats=10, allowParallel = TRUE)
    modelFit2g <- train(Classe ~ ., data = trainData, method="rf", trControl = fitControl)
    stopCluster(cluster)
    date()
    
     cluster <- makeCluster(detectCores() - 1) # convention to leave 1 core for OS
    > registerDoParallel(cluster)
    > date()
    [1] "Wed Jun 29 23:11:05 2016"
    > set.seed(12025)
    > fitControl <- trainControl(method = "repeatedcv", number = 10, repeats=10, allowParallel = TRUE)
    > modelFit2g <- train(Classe ~ ., data = trainData, method="rf", trControl = fitControl)
    Warning messages:
    1: closing unused connection 8 (<-MobilePC:11421) 
    2: closing unused connection 7 (<-MobilePC:11421) 
    3: closing unused connection 6 (<-MobilePC:11421) 
    > stopCluster(cluster)
    > date()
    [1] "Thu Jun 30 02:00:46 2016"
    > save.image("~/machinelearning5.RData")
    > modelFit2g
    Random Forest 
    
    11776 samples
       45 predictor
        5 classes: 'A', 'B', 'C', 'D', 'E' 
    
    No pre-processing
    Resampling: Cross-Validated (10 fold, repeated 10 times) 
    Summary of sample sizes: 10597, 10599, 10598, 10600, 10599, 10597, ... 
    Resampling results across tuning parameters:
    
      mtry  Accuracy   Kappa    
       2    0.9891898  0.9863233
      23    0.9903194  0.9877536
      45    0.9796197  0.9742176
    
    Accuracy was used to select the optimal model using  the largest value.
    The final value used for the model was mtry = 23. 
    > modelFit2g$finalModel
    
    Call:
     randomForest(x = x, y = y, mtry = param$mtry) 
                   Type of random forest: classification
                         Number of trees: 500
    No. of variables tried at each split: 23
    
            OOB estimate of  error rate: 0.87%
    Confusion matrix:
         A    B    C    D    E class.error
    A 3338    7    1    0    2 0.002986858
    B   24 2243   12    0    0 0.015796402
    C    0   11 2033   10    0 0.010223953
    D    1    0   23 1905    1 0.012953368
    E    0    0    2    8 2155 0.004618938
    > confusionMatrix(modelFit2g)
    Cross-Validated (10 fold, repeated 10 times) Confusion Matrix 
    
    (entries are percentual average cell counts across resamples)
     
              Reference
    Prediction    A    B    C    D    E
             A 28.4  0.2  0.0  0.0  0.0
             B  0.0 19.0  0.2  0.0  0.0
             C  0.0  0.1 17.2  0.2  0.0
             D  0.0  0.0  0.1 16.2  0.1
             E  0.0  0.0  0.0  0.0 18.3
                                
     Accuracy (average) : 0.9903
    
    > testPred <- predict(modelFit2g, newdata = testData)
    > confusionMatrix(testData$Classe,testPred)
    Confusion Matrix and Statistics
    
              Reference
    Prediction    A    B    C    D    E
             A 2232    0    0    0    0
             B    6 1503    9    0    0
             C    0    5 1360    3    0
             D    0    0   21 1263    2
             E    0    1    4    4 1433
    
    Overall Statistics
                                              
                   Accuracy : 0.993           
                     95% CI : (0.9909, 0.9947)
        No Information Rate : 0.2852          
        P-Value [Acc > NIR] : < 2.2e-16       
                                              
                      Kappa : 0.9911          
     Mcnemar's Test P-Value : NA              
    
    Statistics by Class:
    
                         Class: A Class: B Class: C Class: D Class: E
    Sensitivity            0.9973   0.9960   0.9756   0.9945   0.9986
    Specificity            1.0000   0.9976   0.9988   0.9965   0.9986
    Pos Pred Value         1.0000   0.9901   0.9942   0.9821   0.9938
    Neg Pred Value         0.9989   0.9991   0.9948   0.9989   0.9997
    Prevalence             0.2852   0.1923   0.1777   0.1619   0.1829
    Detection Rate         0.2845   0.1916   0.1733   0.1610   0.1826
    Detection Prevalence   0.2845   0.1935   0.1744   0.1639   0.1838
    Balanced Accuracy      0.9987   0.9968   0.9872   0.9955   0.9986 



---------


Pre-Processing Options

####Validation


###Discussion


###Conclusions:
As part of the Coursera assessment, the work described here are restricted to answer the followings:
1.  To predict the manner in which the exercise was done.
2.  To create a report describing how a proposed model was built (this present document).
3.  To show how cross validation was implemented
4.  To analyse the sample error.
5.  To discuss the assumptions made.
6.  To apply the proposed model to predict 20 different test cases.


###Remarks:

*  The variables were labeled within CamelCase structure. The variables description of the NewTidyData is presented in the Appendix B.
*  Codes applied here are described in Appendix C.

***************************************************
####Reference:
http://wiki.ros.org/razor_imu_9dof
https://perceptual.mpi-inf.mpg.de/files/2013/03/velloso13_ah.pdf
https://www.sparkfun.com/products/10736
http://www.starlino.com/imu_guide.html
http://www.aliexpress.com/item/DFRobot-9-degrees-of-freedom-inertial-navigation-sensors-9DOF-Razor-IMU-AHRS-compatib/1544796282.html
"Exploratory Data Analysis with R" Roger D. Peng (ver 2015-07-03)
http://www.jmlr.org/papers/volume3/guyon03a/guyon03a.pdf
http://www.analyticsvidhya.com/blog/2016/03/select-important-variables-boruta-package/
Journal of Statistical Software November 2008, Volume 28, Issue 5.
http://topepo.github.io/caret/training.html
http://will-stanton.com/machine-learning-with-r-an-irresponsibly-fast-tutorial/
Predictive Analytics with Microsoft Azure Machine Learning 2nd Edition: Edition 2

## Appendix A ##
Variables original names:
#####

    X; user_name; raw_timestamp_part_1; raw_timestamp_part_2; cvtd_timestamp; new_window; num_window; roll_belt;
    pitch_belt; yaw_belt; total_accel_belt; kurtosis_roll_belt; kurtosis_picth_belt; kurtosis_yaw_belt; skewness_roll_belt;
    skewness_roll_belt.1; skewness_yaw_belt; max_roll_belt; max_picth_belt; max_yaw_belt; min_roll_belt; min_pitch_belt;
    min_yaw_belt; amplitude_roll_belt; amplitude_pitch_belt; amplitude_yaw_belt; var_total_accel_belt; avg_roll_belt;
    stddev_roll_belt; var_roll_belt; avg_pitch_belt; stddev_pitch_belt; var_pitch_belt; avg_yaw_belt; stddev_yaw_belt;
    var_yaw_belt; gyros_belt_x; gyros_belt_y; gyros_belt_z; accel_belt_x; accel_belt_y; accel_belt_z; magnet_belt_x;
    magnet_belt_y; magnet_belt_z; roll_arm; pitch_arm; yaw_arm; total_accel_arm; var_accel_arm; avg_roll_arm; stddev_roll_arm;
    var_roll_arm; avg_pitch_arm; stddev_pitch_arm; var_pitch_arm; avg_yaw_arm; stddev_yaw_arm; var_yaw_arm; gyros_arm_x;
    gyros_arm_y; gyros_arm_z; accel_arm_x; accel_arm_y; accel_arm_z; magnet_arm_x; magnet_arm_y; magnet_arm_z;
    kurtosis_roll_arm; kurtosis_picth_arm; kurtosis_yaw_arm; skewness_roll_arm; skewness_pitch_arm; skewness_yaw_arm;
    max_roll_arm; max_picth_arm; max_yaw_arm; min_roll_arm; min_pitch_arm; min_yaw_arm; amplitude_roll_arm;
    amplitude_pitch_arm; amplitude_yaw_arm; roll_dumbbell; pitch_dumbbell; yaw_dumbbell; kurtosis_roll_dumbbell;
    kurtosis_picth_dumbbell; kurtosis_yaw_dumbbell; skewness_roll_dumbbell; skewness_pitch_dumbbell; skewness_yaw_dumbbell;
    max_roll_dumbbell; max_picth_dumbbell; max_yaw_dumbbell; min_roll_dumbbell; min_pitch_dumbbell; min_yaw_dumbbell;
    amplitude_roll_dumbbell; amplitude_pitch_dumbbell; amplitude_yaw_dumbbell; total_accel_dumbbell; var_accel_dumbbell;
    avg_roll_dumbbell; stddev_roll_dumbbell; var_roll_dumbbell; avg_pitch_dumbbell; stddev_pitch_dumbbell; var_pitch_dumbbell;
    avg_yaw_dumbbell; stddev_yaw_dumbbell; var_yaw_dumbbell; gyros_dumbbell_x; gyros_dumbbell_y; gyros_dumbbell_z;
    accel_dumbbell_x; accel_dumbbell_y; accel_dumbbell_z; magnet_dumbbell_x; magnet_dumbbell_y; magnet_dumbbell_z;
    roll_forearm; pitch_forearm; yaw_forearm; kurtosis_roll_forearm; kurtosis_picth_forearm; kurtosis_yaw_forearm;
    skewness_roll_forearm; skewness_pitch_forearm; skewness_yaw_forearm;max_roll_forearm; max_picth_forearm; max_yaw_forearm;
    min_roll_forearm; min_pitch_forearm; min_yaw_forearm; amplitude_roll_forearm; amplitude_pitch_forearm;
    amplitude_yaw_forearm; total_accel_forearm; var_accel_forearm; avg_roll_forearm; stddev_roll_forearm; var_roll_forearm;
    avg_pitch_forearm; stddev_pitch_forearm; var_pitch_forearm; avg_yaw_forearm; stddev_yaw_forearm; var_yaw_forearm;
    gyros_forearm_x; gyros_forearm_y; gyros_forearm_z; accel_forearm_x; accel_forearm_y; accel_forearm_z; magnet_forearm_x;
    magnet_forearm_y; magnet_forearm_z; classe;

Variables names in NewTidyData:

    X;UserName;RawTimestampPart_1;RawTimestampPart_2;CvtdTimestamp
    NewWindow;NumWindow;RollBelt;PitchBelt;YawBelt
    TotalAccelBelt;KurtosisRollBelt;KurtosisPicthBelt;KurtosisYawBelt;SkewnessRollBelt
    SkewnessRollBelt.1;SkewnessYawBelt;MaxRollBelt;MaxPicthBelt;MaxYawBelt
    MinRollBelt;MinPitchBelt;MinYawBelt;AmplitudeRollBelt;AmplitudePitchBelt
    AmplitudeYawBelt;VarTotalAccelBelt;AvgRollBelt;StddevRollBelt;VarRollBelt
    AvgPitchBelt;StddevPitchBelt;VarPitchBelt;AvgYawBelt;StddevYawBelt
    VarYawBelt;GyrosBeltX;GyrosBeltY;GyrosBeltZ;AccelBeltX
    AccelBeltY;AccelBeltZ;MagnetBeltX;MagnetBeltY;MagnetBeltZ
    RollArm;PitchArm;YawArm;TotalAccelArm;VarAccelArm
    AvgRollArm;StddevRollArm;VarRollArm;AvgPitchArm;StddevPitchArm
    VarPitchArm;AvgYawArm;StddevYawArm;VarYawArm;GyrosArmX
    GyrosArmY;GyrosArmZ;AccelArmX;AccelArmY;AccelArmZ
    MagnetArmX;MagnetArmY;MagnetArmZ;KurtosisRollArm;KurtosisPicthArm
    KurtosisYawArm;SkewnessRollArm;SkewnessPitchArm;SkewnessYawArm;MaxRollArm
    MaxPicthArm;MaxYawArm;MinRollArm;MinPitchArm;MinYawArm
    AmplitudeRollArm;AmplitudePitchArm;AmplitudeYawArm;RollDumbbell;PitchDumbbell
    YawDumbbell;KurtosisRollDumbbell;KurtosisPicthDumbbell;KurtosisYawDumbbell;SkewnessRollDumbbell
    SkewnessPitchDumbbell;SkewnessYawDumbbell;MaxRollDumbbell;MaxPicthDumbbell;MaxYawDumbbell
    MinRollDumbbell;MinPitchDumbbell;MinYawDumbbell;AmplitudeRollDumbbell;AmplitudePitchDumbbell
    AmplitudeYawDumbbell;TotalAccelDumbbell;VarAccelDumbbell;AvgRollDumbbell;StddevRollDumbbell
    VarRollDumbbell;AvgPitchDumbbell;StddevPitchDumbbell;VarPitchDumbbell;AvgYawDumbbell
    StddevYawDumbbell;VarYawDumbbell;GyrosDumbbellX;GyrosDumbbellY;GyrosDumbbellZ
    AccelDumbbellX;AccelDumbbellY;AccelDumbbellZ;MagnetDumbbellX;MagnetDumbbellY
    MagnetDumbbellZ;RollForearm;PitchForearm;YawForearm;KurtosisRollForearm
    KurtosisPicthForearm;KurtosisYawForearm;SkewnessRollForearm;SkewnessPitchForearm;SkewnessYawForearm
    MaxRollForearm;MaxPicthForearm;MaxYawForearm;MinRollForearm;MinPitchForearm
    MinYawForearm;AmplitudeRollForearm;AmplitudePitchForearm;AmplitudeYawForearm;TotalAccelForearm
    VarAccelForearm;AvgRollForearm;StddevRollForearm;VarRollForearm;AvgPitchForearm
    StddevPitchForearm;VarPitchForearm;AvgYawForearm;StddevYawForearm;VarYawForearm
    GyrosForearmX;GyrosForearmY;GyrosForearmZ;AccelForearmX;AccelForearmY
    AccelForearmZ;MagnetForearmX;MagnetForearmY;MagnetForearmZ;Classe



## Appendix B ##


## Appendix C ##

    library("dplyr")

#####   if you don't have the package use: install.packages("dplyr")

    fileUrl <- "https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv"
    download.file(fileUrl, destfile="./pml-training.csv") 
    pml.training <- read.csv("pml-training.csv",header=TRUE, na.strings=c("","NA","#DIV/0!"))
    
    fileUrl <- "https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv"
    download.file(fileUrl, destfile="./pml-testing.csv") 
    pml.testing <- read.csv("pml-testing.csv",header=TRUE)

#### Cleaning the non-alphanumeric characters, and using CamelCase structure

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

####*removing variables that contain more than 95% of 'NA'*
NewTidyData1 = TidyData[, colSums(is.na(TidyData))/nrow(TidyData) < 0.95]
dim(NewTidyData1)

####*removing observables that contain "NA"
temp = TidyData[, colSums(is.na(TidyData))/nrow(TidyData) > 0.95]
NewTidyData2 = temp[, colSums(is.na(temp))/nrow(temp) != 1]
NewTidyData2 = na.omit(NewTidyData2)
dim(NewTidyData2)

####*removed variables that only have NA:*
setdiff(names(temp),names(NewTidyData2))

####*remained variables:*
str(NewTidayData)

####*with the selected features*


temp=select(NewTidyData,UserName,Classe,starts_with('Var'),starts_with('Avg'))
plot(temp[,c(19:30)], colour=temp$Classe,rm.na=TRUE)

splom( ~ temp[,19:30] | temp$UserName, cex=0.8, pch=16,tick.labels=FALSE)
levels(temp$Classe) <- c("red","blue","green","black","orange")

pairs(temp[,19:30], cex=0.8, pch=21, bg =c("black","red", "green3", "blue","orange")[unclass(temp$Classe)], oma=c(8,3,3,3))
par(xpd = TRUE)
legend("bottom", fill = unique(temp$Classe), legend = c(levels(temp$Classe)), horiz=TRUE, bty='n')

featurePlot(x=temp[,19:30], y=temp$Classe, plot="pairs", auto.key=list(columns=5))

(Takes long time)
splom( ~ temp[,19:30] | temp$UserName, cex=0.7, groups=as.factor(temp$Classe),type="p", pch=16, scales=list(x=list(at=NULL)), auto.key=list(space="right", columns=1, points=FALSE, rectangles=TRUE, cex.title=1))
 

Next, after splitting the data into training and testing sets and using the caret package to automate training and testing both random forest and partial least squares models using repeated 10-fold cross-validation (see the code), it turns out random forest outperforms PLS in this case, and performs fairly well overall:


####*Verify if there is any "NA". If there is it can use na.rm = TRUE in mean()*
    sum(is.na(NewTidyData))

temp2=select(NewTidyData,UserName,Classe,starts_with('Var'),starts_with('Avg'))

library("caret")
set.seed(123)
inTrain <- createDataPartition(NewTidyData1$Class, p = 0.6, list = FALSE)
trainTidyData <- NewTidyData1[inTrain,]
testTidyData <- NewTidyData1[-inTrain,]

modelFit = train(Classe ~., data=trainTidyData, method="rf", prox=TRUE)
modelfFit

test = test[apply(test, 1, function(x) !any(x == '#DIV/0!')), ] 


impVar = as.data.frame(varImp(modelFit2, scale=TRUE)[1])
a = cbind(Variables = rownames(impVar),impVar)
impVarOrd = arrange(a,desc(Overall))
write.csv(b,"varImp_modelFit2.csv")


<p align="center">
  <img src="https://raw.githubusercontent.com/anonymous-1618/ML/master/Rplot19.png">
  <b>Figure 1 - </b>Variable Importance</b><br>
  </p>

![alt text](https://raw.githubusercontent.com/anonymous-1618/ML/master/Rplot19.png "test")
