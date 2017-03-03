# Getting-and-Cleaning-Data
Final Project for the Coursera Getting and Cleaning Data Couse

## Instructions

The purpose of this project is to demonstrate your ability to collect, work with, and clean a data set.

### Review criteria 
1. The submitted data set is tidy.
1. The Github repo contains the required scripts.
1. GitHub contains a code book that modifies and updates the available codebooks with the data to indicate all the variables and summaries calculated, along with units, and any other relevant information.
1. The README that explains the analysis files is clear and understandable.
1. The work submitted for this project is the work of the student who submitted it.

### Getting and Cleaning Data Course Project
The purpose of this project is to demonstrate your ability to collect, work with, and clean a data set. The goal is to prepare tidy data that can be used for later analysis. You will be graded by your peers on a series of yes/no questions related to the project. You will be required to submit: 1) a tidy data set as described below, 2) a link to a Github repository with your script for performing the analysis, and 3) a code book that describes the variables, the data, and any transformations or work that you performed to clean up the data called CodeBook.md. You should also include a README.md in the repo with your scripts. This repo explains how all of the scripts work and how they are connected.

One of the most exciting areas in all of data science right now is wearable computing - see for example this article . Companies like Fitbit, Nike, and Jawbone Up are racing to develop the most advanced algorithms to attract new users. The data linked to from the course website represent data collected from the accelerometers from the Samsung Galaxy S smartphone. A full description is available at the site where the data was obtained:

http://archive.ics.uci.edu/ml/datasets/Human+Activity+Recognition+Using+Smartphones

Here are the data for the project:

https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip

You should create one R script called run_analysis.R that does the following:

1. Merges the training and the test sets to create one data set.
1. Extracts only the measurements on the mean and standard deviation for each measurement.
1. Uses descriptive activity names to name the activities in the data set
1. Appropriately labels the data set with descriptive variable names.
1. From the data set in step 4, creates a second, independent tidy data set with the average of each variable for each activity and each subject.

Good luck!

## Code Explanations
### run_analysis.R

Import all of the necessary data sets.
```R
## Import activity and feature labels
activity <- read.table(file.path(getwd(), "activity_labels.txt"))
features <- read.table(file.path(getwd(), "features.txt"))

## import test datasets
subject_test <- read.table(file.path(getwd(), "test", "subject_test.txt"))
X_test <- read.table(file.path(getwd(), "test", "X_test.txt"))
Y_test <- read.table(file.path(getwd(), "test", "Y_test.txt"))

## import train datasets
subject_train <- read.table(file.path(getwd(), "train", "subject_train.txt"))
X_train <- read.table(file.path(getwd(), "train", "X_train.txt"))
Y_train <- read.table(file.path(getwd(), "train", "Y_train.txt"))
```

Use the features dataset to select mean and standard deviation variables from the 'X' datasets. Mean was interpretted as any varible name with the pattern "mean()" and standard deviation as "std()". The column names of the 'X' datasets are then renamed to match the selected variables.
```R
## Select/Rename columns in the X data frames
## Select columns that contain "mean()" or "std()"
mean_std_cols <- grep("mean\\(\\)|std\\(\\)", features[,2])
X_test <- select(X_test, mean_std_cols)
X_train <- select(X_train, mean_std_cols)
colnames(X_test) <- features[mean_std_cols, 2]
colnames(X_train) <- features[mean_std_cols, 2]
```

The 'Y' dataset contains an activity id for each observation (i.e. row) in the 'X' dataset. The activity dataset containts the key-value pairs for associating the activity id with the activity name. The activity name is matched and replaced with the activity id in the 'Y' dataset.
```R
## Assign activity names to Y data frames
match_test <- match(Y_test[["V1"]], activity[["V1"]])
match_train <- match(Y_train[["V1"]], activity[["V1"]])
Y_test[["V1"]] <- activity[match_test, "V2"]
Y_train[["V1"]] <- activity[match_train, "V2"]
```

Rename and then add the subject id and activity columns to the 'X' datasets.
```R
## Add subject id and activity to the X data frames
colnames(subject_test) <- "subject_id"
colnames(subject_train) <- "subject_id"
colnames(Y_test) <- "activity"
colnames(Y_train) <- "activity"
X_test <- cbind(subject_test, Y_test, X_test)
X_train <- cbind(subject_train, Y_train, X_train)
```

Combine the 'test' and 'train' datasets.
```R
## Combine test and train datasets
## Note that the two sets are partitions of an initial set, so
## no matching or merging is needed, rbind() is sufficient
merged_set <- rbind(X_test, X_train)
```

Order the new combined dataset so that it is easier to group and analyze.
```R
## Order by subject_id and then by activity
merged_set <- merged_set[order(merged_set[,1], merged_set[,2]),]
```

Calculate the mean for each variable for each observation. An observation is taken as each unique combination of subject id and activity. With a total of 30 subjects and 6 activities, this yields 180 observations. The original dataset was much larger as each pair of subject and activity was measured at multiple time points, which are averaged in this final dataset.
```R
## Summarize each column by subject_id and activity
mean_set <- merged_set %>% group_by(subject_id, activity) %>% summarise_each(funs(mean))
```
Finally, write the results to a text file.
```R
## Export result to txt file
write.table(mean_set, file.path(getwd(), "mean_set.txt"), row.names = FALSE)
```
