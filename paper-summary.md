####Qualitative Activity Recognition of Weight Lifting Exercises####
Qualitative Activity Recognition of Weight Lifting Exercises. Eduardo Velloso. Lancaster University. Lancaster, UK. e.velloso@lancaster.ac.uk. Andreas Bulling. Max Planck Institute for. Informatics. andreas.bulling@acm.org. Hans Gellersen. Lancaster University. Lancaster, UK. hwg@comp.lancs.ac.uk. Wallace Ugulin

In this work we define quality of execution and investigate three aspects that pertain to qualitative activity recognition:
* specifying correct execution,
* detecting execution mistakes,
* providing feedback on the to the user.

We illustrate our approach on the example problem of qualitatively
assessing and providing feedback on weight lifting
exercises. In two user studies we try out a sensor- and a
model-based approach to qualitative activity recognition.
Our results underline the potential of model-based assessment
and the positive impact of real-time user feedback on
the quality of execution

Nowadays, to improve the life quality more and more peole are practicing sports.
The practicing not only improve the body physical conditions or fitness but also decrease the likehood of 
many ills related to sedentarism, such as heart diseases. But, it is not only the act or quantity of doing physical exercise that matters. Many people usually forget that physical exercises when are done in the wrong way, they are, on one hand, less effective. On the other hand they can be very damaging causing undesirable injuries. Therefore, the right way to perform the exercise is considered a top priority.

*(the following paragraphs from the paper)*
In activity recognition using on-body sensing, a large body of work has investigated automatic techniques to discriminate which activity was performed.

three key components of any qualitative activity recognition system:
* problem of specifying correct execution
* the automatic and robust detection of execution mistakes
* how to provide feedback on the quality of execution to the user.

More specifically, we explore two approaches for detecting mistakes in an automated fashion:
* use machine learning
* pattern recognition techniques to detect mistakes
* one proposed in this paper, is to use a model-based approach and to compare motion traces recorded using ambient sensors to a formal specification of what constitutes correct execution.

Three important questions:
* Is it possible to detect mistakes in the execution of the activity?
Traditional activity recognition has extensively explored how to classify different activities. Will these methods
work as well for qualitative assessment of activities?
* The second question is how we specify activities. Two approaches are commonly used in activity recognition: a
sensor-oriented approach, in which a classification algorithm is trained on the execution of activities and a model-oriented
approach, in which activities are represented by a human skeleton model.
* The third is how to provide feedback in real-time to improve the quality of execution. Depending on how fast the system can make the assessment, the feedback will either be provided in real-time or as soon as the activity is completed.

In the next sections we explore a wearable sensor-oriented classification approach for the detection of mistakes, we describe
a model-oriented approach to the specification of activities and we evaluate two feedback systems implemented using
the modelling approach

#### 5. DETECTION OF MISTAKES ####
The goal of our first experiment was to assess whether we could detect mistakes in weight-lifting exercises by using activity
recognition techniques. we recorded users performing the same activity correctly and with a set of common mistakes
with wearable sensors and used machine learning to classify each mistake. This way, we used the training data as the activity specification and the classification algorithm as the means to compare the execution to the specification.

We mounted the sensors in the users’
1. glove
2. armband
3. lumbar belt
4. and dumbbell (see Figure 1). 

Participants were asked to perform one set of 10 repetitions of the Unilateral Dumbbell Biceps Curl in five different fashions:
A. exactly according to the specification (Class A)
B. throwing the elbows to the front (Class B)
C. lifting the dumbbell only halfway (Class C)
D. lowering the dumbbell only halfway (Class D)
E. throwing the hips to the front (Class E). 

Class A corresponds to the specified execution of the exercise, while the other 4 classes correspond to common mistakes.
The exercises were performed by six male participants aged between 20-28 years, with little weight lifting experience by using a relatively light dumbbell (1.25kg).

##### 5.1 Feature extraction and selection
For feature extraction we used a sliding window approach with different lengths from 0.5 second to 2.5 seconds, with 0.5 second overlap. In each step of the sliding window approach we calculated features on the Euler angles (roll, pitch and yaw), as well as the raw accelerometer, gyroscope and magnetometer readings.

For the Euler angles of *each of the four sensors* we calculated **eight features**:
1. mean
2. variance
3. standard deviation
4. max
5. min
6. amplitude
7. kurtosis
8. skewness

generating in total 96 derived feature sets.

In order to identify the most relevant features we used the **feature selection algorithm** based on correlation proposed by
Hall [14]. The algorithm was configured to use a “Best First” strategy based on backtracking.
**17 features** were selected:

in the belt
* the mean and variance of the roll
* maximum, range and variance of the accelerometer vector,
* variance of the gyro
* variance of the magnetometer.

In the arm
* the variance of the accelerometer vector
* the maximum and minimum of the magnetometer.

In the dumbbell
* the maximum of the acceleration variance of the gyro
* maximum and minimum of the magnetometer

in the glove
*the sum of the pitch and the maximum and minimum of the gyro.

##### 5.2 Recognition Performance
Because of the characteristic noise in the sensor data, we used a Random Forest approach [28]. This algorithm is
characterized by a subset of features, selected in a random and independent manner with the same distribution for each
of the trees in the forest. 

To improve recognition performance we used an ensemble of classifiers using the “Bagging” method [6]. We used 10 random forests and each forest was implemented with 10 trees. 

The classifier was tested with 10-fold cross-validation and different windows sizes, all of them with 0.5s overlapping (except the window with 0.5s).

The best window size found for this classification task was of 2.5s and the overall recognition performance was of 98.03%
(see Table 1). 

Table 1: Recognition performance

| Window Size | FPR | Recall | AUC  | Precision |
|-------------|-----|--------|------|-----------|
|    0.5s     | 3.9 |  85.0  | 97.4 |   84.9    |
|    1.0s     | 1.8 |  93.5  | 99.5 |   93.5    |
|    1.5s     | 1.0 |  96.5  | 99.8 |   96.5    |
|    2.0s     | 0.7 |  97.2  | 99.9 |   97.2    |
|    2.5s     | 0.5 |  98.2  | 99.9 |   98.2    |

The table shows false positive rate (FPR), precision, recall, as well as area under the curve (AUC) averaged
for each of the 5 tested on 10-fold cross-validation over all 6 participants (5 classes). With the 2.5s window size, the detailed accuracy by class was of: (A) 97.6%, (B) 97.3%, (C) 98.2%, (D) 98.1%, (E) 99.1%, (98.2% weighted
average).

We also used the leave-one-subject-out test in order to measure whether our classifier trained for some subjects is still
useful for a new subject.
The overall recognition performance in this test was 78.2 %. The result can be attributed to the **small size of the datasets** (approx 1800 instances each dataset, extracted from 39.200 readings on the IMUs), the number of subjects (6 young men), and the **difficulty in differentiating variations of the same exercise**, which is a challenge in Qualitative Assessment Activities.
The use of this approach requires a lot of data from several subjects, in order to reach a result that can be generalized for a new user *without the need of training the classifier*(??). The confusion matrix of the leave-one-subject-out test is illustrated on Figure 2.

##### 5.3 Conclusion
The advantage of this approach is that *no formal specification is necessary*, but even though our results point out that it is possible to detect mistakes by classification, this approach is hardly scalable. *It would be infeasible to record all possible mistakes for each exercise*. Moreover, even if this was possible, the more classes that need to be considered the harder the classification problem becomes.

#### 6. MODEL-BASED SPECIFICATION ####
Due to the inherent problems of the classification approach, we concentrated our efforts into trying to *formalize a way
of specifying activities and recognizing mistakes* by looking at deviations from the model in the execution.

##### 6.1 Activity Selection

An activity must have an appropriate granularity to be analysed. If the activity is too complex, it is more appropriate
to break it down into smaller activities. 

In our example, even though a weight lifting exercise is commonly performed in sets of 6-12 repetitions, for our purposes we consider an activity as a repetition of the exercise. This way we can analyse each repetition separately.

A Biceps Curl repetition involves raising and lowering the dumbbell, so we define the beginning of the activity as when the user starts to lift it and the end as when it reaches the initial position again.

##### 6.2 Activity Specification

The activity should be specified as clearly as possible in natural language. The clearer the specification is the easier it
will be to model the activity. In our example, we used as the specification the instructions provided by a weight lifting
book [7]. An activity specification can be comprised of several instructions. 

For the Unilateral Dumbbell Biceps Curl, the specification we used, adapted from [7], was the following:
1) Stand solidly upright;
2) Your feet should be shoulder-width apart;
3) Your shoulders should be down;
4) Curl the dumbbell in an upward arc. Curl the dumbbell to the top of the movement when your biceps is fully contracted;
5) Elbows pointing directly down and return to the start position;
6) Don’t lean back and throw your hips to the front.

