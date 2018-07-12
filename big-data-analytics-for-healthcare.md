% Big Data Analytics for Healthcare Notes

# Introduction to Big Data

Healthcare generates a lot of data. Everything from "real" medical procedures 
like genetic sequencing and FMRI to consumer applications like fitbit and 23andMe
takes up a lot of data. 

In this course we talk about three major healthcare applications:

__Predictive modeling__  ses historical data to build a model to predict future 
outcomes. Basically predictive modeling is machine learning. There are a lot of 
challenges associated with predictive modeling - just building a pipeline to begin 
predictive modeling is a daunting task.

__Computation phenotyping__  turns messy health records into meaningful concepts. 
Oor example converting messy patient records into whether or not a ptient has a 
specific disease. This process can be very challengeing and complex since 
healthcare data is messy, redundant and may contain missing, incorrect, or 
missing information. 

__Patient similarity__ is the process of using aggregate patient data to group 
similar patients, find outcomes, and use those to give diagnosis and treatment
plans on the individual level. 

The algorithms we will use are: 

__Classification algorithms__ use historical data to learn a function $f: x \to y$.
which maps some input $x$ to a predicted $y$ value. For example we can use 
historical patient data to create a function which predicts whether or not a 
patient has a specific disease. 

__Clustering algorithms__ either uses historical data or dynamicaly learns a 
function that groups data into discrete categories (or clusters) based on 
their similarity. 

__Dimensionality reduction algorithms__ reduce the dimensionality of an input
data set by removing unnecesary features. 

__Graph analysis algorithms__ use patient data to create a network which we can
analyze to find similarities between patients and draw inferences adn conclusions
about treatments, medications, adn diseases. 

The technologies we will use are:

__Hadoop__ is a distributed disk-based big data system. Hadoop is composed of a 
lot fo submodules like pig, hive, and hbase. 

__Spark__ is a distributed, in-memory big data system. 

Both hadoop and spark use the map/reduce programming model which is a very useful 
and powerful parallel programming paradigm. 

# Predictive Modeling

Predictiv modeling is a computational pipeline with several steps. 

![](src/big-data-for-health/predictive-model-pipeline)

1. Define your predictive target (ex: patients who will develop heart failure)\
Not all predictive targets are actually possible to answer using data - some
concepts are much too vague or complex or are simply not trakced well. 
2. Find relevant data (ex: get a cohort of patients)\
In most cases, the relevant patient population is a subset of the entire patient 
population. 
	- In a prospective study we identify the cohort and collect data from scratch. 
	- In a retrospective study we identify the right subset and use / modify the existing data. 
	- In a cohort study we select a group of patients who are exposed to a speciic risk. We need to define the right inclusion and exclusion criterian to properly get the right data. 
	- In a case / control study we identify two sets of patients (case and control). The case set is the patients with a posative outcome (for example they have a specific disease). The control study is a set of patients who demographically look like the cases study but have the negative outcome. 
3. Consturct features\
The goal of feature construction is to define all features that are potentially relevant in predicting the target. When defining features we want to make sure that we choose a relevant time scale. We only want to use data gathered within an "observation" window to predict events after a "prediction" window. The size of the observation window and prediction window is important - a large observation window allows us to gather more data and make more accurate predictions. However, a large prediction window decreases the accuracy of the mode lsince it's harder to make predictions far into the future. 

```
[ observation window ] + [ prediction window ] *

+ -> index date
* -> target date
```

4. Select the actually relevant features\
The goal of feature selection is to filter out the features we constructed in the last step to only use the features that are actually useful in predicting the target. 
5. Construct the predictive model\
A prediction model is a function that maps the input features to a target value. A
precitive model can either solve a regression problem or classification problem. In a regression problem, the output of the model is continuous. In a classification problem the output of the model is discrete. 
6. Evaluate the model\
When we create a model we need to evaluate whether or not it's actually succesful. Typically we will train the model on some "training" data set then test it on a "validation" data set - which is seperate from the training data. We want to do this iteratively to make sure that the validation accuracy increases as time goes on. If the validation accuracy flatlines while training data increases we know that additional training is not very useful. This is called Cross Validation.
	- In "leave one out" cross validation, the validation set is only one sample (which changes for each iteration). 
	- In "k fold" cross validation we split the data into k folds and use different folds as the validation set on each iteration. 
	- In "randomized" cross validation we randomly split the data into training and validation for each iteration. The disadvantage of this method is that we can't garuntee that all hte data will be tested in the validation step. 
7. Repeat until you're satisfied

