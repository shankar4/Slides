---
title       : PCA Preprocessing and KNN Clustering
subtitle    : Effect of Discretization and Tolerance
author      : Ravi Shankar
job         : Student, Data Products Course, Coursera and JHU
framework   : io2012        # {io2012, html5slides, shower, dzslides, ...}
highlighter : highlight.js  # {highlight.js, prettify, highlight}
hitheme     : tomorrow      # 
widgets     : []            # {mathjax, quiz, bootstrap}
mode        : selfcontained # {standalone, draft}
---

## Background

1. This is a demonstration of the use of PCA (principal component analytis) and KNN (k near neighbor) clustering
2. For it to be responsive (low latency) on Shiny, it had to use a small enough data set and not invoke advanced packages such as caret.
3. Chose mtcars data and used low level packages with small footprints (MASS and class).
4. Since the mpg data was continuous, user adjustable discretization levels of 5, 10, 15, and 20 were provided. 
5. The tolerance level of PCA was allowed to vary from 0 to 1, which allowed all to none of the principle components to be chosen.
6. The intent was to see which combination of discreization and PCA tolerance yielded the best accuracy.
7. Accuracy varied from 0 to 50% for various combinations
8. Further refinements may yield better results; but the intent was to prove the feasibility of a machine learning algorithm on Shiny using low level packages. 


--- .class #id 

### Step 1: Discretization of mpg data of mtcars data set at 5, 10, and 20 levels


```r
data(mtcars); discretization.levels <- 5; cars <- mtcars 
cars$mpg <- cut(cars$mpg,discretization.levels); discretization.levels; head(cars$mpg)
```

```
## [1] 5
```

```
## [1] (19.8,24.5] (19.8,24.5] (19.8,24.5] (19.8,24.5] (15.1,19.8] (15.1,19.8]
## Levels: (10.4,15.1] (15.1,19.8] (19.8,24.5] (24.5,29.2] (29.2,33.9]
```

```
## [1] (19.8,22.1] (19.8,22.1] (22.1,24.5] (19.8,22.1] (17.5,19.8] (17.5,19.8]
## 10 Levels: (10.4,12.8] (12.8,15.1] (15.1,17.5] (17.5,19.8] ... (31.6,33.9]
```

```
## [1] (21,22.1]   (21,22.1]   (22.1,23.3] (21,22.1]   (18.6,19.8] (17.5,18.6]
## 20 Levels: (10.4,11.6] (11.6,12.8] (12.8,13.9] (13.9,15.1] ... (32.7,33.9]
```

---

### Step 2: Use of PCA for preprocessing the training and validation sets


```r
data(mtcars); discretization.levels <- 10; cars <- mtcars 
cars$mpg <- cut(cars$mpg,discretization.levels)
set.seed(32343)
training.rows <- tapply(1:nrow(cars), cars$mpg , sample , size =4, replace=TRUE)
InTrain <- unlist( training.rows )
train.set <- cars[InTrain,]; valid.set <- cars[-InTrain,]
```
Dimensions of train.set and  valid.set before transformation with PCA are, respectively, [40, 11] and  [10, 11].


```r
require(MASS); library(MASS)
PCA.Tolerance <- 0.05
preProc <- prcomp(train.set[,-1], scale=TRUE, center=TRUE, tol = PCA.Tolerance)
trainPC <- predict(preProc,train.set[,-1]); validPC <- predict(preProc,valid.set[,-1])
```

Dimensions of trainPC and the validPC after transformation with PCA are, respectively, [40, 9] and [10, 9].
```

---

### Use of KNN for predicting the mpg of the validation set


```r
require(class); library(class)
trainPC.df <- as.data.frame(trainPC)
validPC.df <- as.data.frame(validPC)
pred <- knn(trainPC.df, validPC.df, cl= train.set$mpg, k = 4,  prob = FALSE, use.all = TRUE)
obs <- valid.set$mpg
count <- sum(pred==obs)
accuracy <- round(100 * count/length(pred))
```

Accuracy of the prediction for 10 discretization levels and PCA tolerance of 0.05 is 20%.


Accuracy of the prediction for 10 discretization levels and PCA tolerance of 0.75 is 30%.

Summary: This has been a tutorial to show-case the use of PCA and KNN. I had to find and use low level R packages to ensure that it will run fast and with a small memory footprint on the web (as a Shiny App) . For varous combinations of discretization levels and PCA tolerance, accuracies of 0 to 50% were achieved. For a good real-world example with the caret package, see https://github.com/shankar4/Machine-Learning. 




