# Practical Machine Learning - Course Project
by John Pelak  

## Predicting Exercise Quality from Motion Sensor Data

### Overview
Use of devices to track exercise activity is becoming increasingly popular, with sales growing as costs decrease and software becomes more feature-rich.  While considerable study has been conducted on the recognition of distinct activities (e.g., standing, climbing stairs), much far less analysis of the quality of exercise activity has been published.  To this end, [Groupware@LES](http://groupware.les.inf.puc-rio.br) has built a [data set](http://groupware.les.inf.puc-rio.br/static/WLE/WearableComputing_weight_lifting_exercises_biceps_curl_variations.csv) tracking motion of 6 subjects lifting weights properly and improperly, suitable for training and testing predictive models.  It is the goal of this project to build such a predictive model and assess its performance and accuracy.


```r
library(caret)
```

```
## Warning: package 'caret' was built under R version 3.1.3
```

```
## Loading required package: lattice
## Loading required package: ggplot2
```

```
## Warning: package 'ggplot2' was built under R version 3.1.3
```

### Loading Data
The Groupware@LES data used for this project (published in [Qualitative Activity Recognition of Weight Lifting Exercises](http://groupware.les.inf.puc-rio.br/work.jsf?p1=11201)) have been nicely partitioned with a large set of features (>150) to be used to predict a labelled outcome.  Each observation in this data set has been labelled A, B, C, D, or E, signifying correct lifting motion, elbows-forward lifting motion, halfway up lifting motion, halfway down lifting motion, and hips-forward lifting motion respectively.  The features present in the data set along with the label contained in the last column ('classe') are as shown:


```r
harTrainingDataRaw <- read.csv("pml-training.csv")
harTestingDataRaw <- read.csv("pml-testing.csv")
```

### Cleaning Data
After loading, the data used in this analysis was then cleaned so that it was more readily consumable by functions in the R language's caret package.  In the training data set, first the row number, user name, raw timestamp (parts 1 and 2), cvtd timestamp, new window, and num window columns (columns 1 through 7) were removed, as we have not yet covered time series prediction.  Then, remaining predictor columns were converted to character so that missing values (""), "NA"" values, and "#DIV/0!" values could be substituted with "0". Also, kurtosis yaw belt, skewness yaw belt, amplitude yaw belt, kurtosis yaw dumbbell, skewness yaw dumbbell, amplitude yaw dumbbell, kurtosis yaw forearm, skewness yaw forearm, amplitude yaw forearm all had no variance and thus were not useful for prediction and so were also discarded.  Finally, all columns except the classe label were converted to numeric, and the the classe label column was converted to a factor.

```r
harTrainingData <- harTrainingDataRaw[,-1:-7]
harTrainingData <- data.frame(lapply(harTrainingData, as.character), stringsAsFactors = FALSE)
harTrainingData[is.na(harTrainingData)] <- "0"
harTrainingData[is.null(harTrainingData)] <- "0"
harTrainingData[harTrainingData == "#DIV/0!"] <- "0"
harTrainingData[harTrainingData == ""] <- "0"
harTrainingData <- subset(harTrainingData, select=-c(kurtosis_yaw_belt, skewness_yaw_belt, amplitude_yaw_belt, kurtosis_yaw_dumbbell, skewness_yaw_dumbbell, amplitude_yaw_dumbbell, kurtosis_yaw_forearm, skewness_yaw_forearm, amplitude_yaw_forearm))
harTrainingData[,-144] <- data.frame(lapply(harTrainingData[,-144], as.numeric))
harTrainingData$classe <- as.factor(harTrainingData$classe)
```

### Model Development
The R language's caret package contains a strong set of tools useful for building predictive models.  With such a large set of features, it is helpful to separate those that are useful for making predictions from those that are not.  The "helpful" features are referred to as Principal Components, and can be determined using the Principal Component Analysis capabilties provided in the R caret package.

Before we begin, the harTrainingData set itself must be split into training and test data.


