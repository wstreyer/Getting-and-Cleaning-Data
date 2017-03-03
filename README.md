# Getting-and-Cleaning-Data
Final Project for the Coursera Getting and Cleaning Data Couse

'''R
## Import activity and feature labels
activity <- read.table(file.path(getwd(), "activity_labels.txt"))
features <- read.table(file.path(getwd(), "features.txt"))
'''

'''R
## import test datasets
subject_test <- read.table(file.path(getwd(), "test", "subject_test.txt"))
X_test <- read.table(file.path(getwd(), "test", "X_test.txt"))
Y_test <- read.table(file.path(getwd(), "test", "Y_test.txt"))
'''

'''R
## import train datasets
subject_train <- read.table(file.path(getwd(), "train", "subject_train.txt"))
X_train <- read.table(file.path(getwd(), "train", "X_train.txt"))
Y_train <- read.table(file.path(getwd(), "train", "Y_train.txt"))
'''

'''R
## Select/Rename columns in the X data frames
## Select columns that contain "mean()" or "std()"
mean_std_cols <- grep("mean\\(\\)|std\\(\\)", features[,2])
X_test <- select(X_test, mean_std_cols)
X_train <- select(X_train, mean_std_cols)
colnames(X_test) <- features[mean_std_cols, 2]
colnames(X_train) <- features[mean_std_cols, 2]
'''

'''R
## Assign activity names to Y data frames
match_test <- match(Y_test[["V1"]], activity[["V1"]])
match_train <- match(Y_train[["V1"]], activity[["V1"]])
Y_test[["V1"]] <- activity[match_test, "V2"]
Y_train[["V1"]] <- activity[match_train, "V2"]
'''

'''R
## Add subject id and activity to the X data frames
colnames(subject_test) <- "subject_id"
colnames(subject_train) <- "subject_id"
colnames(Y_test) <- "activity"
colnames(Y_train) <- "activity"
X_test <- cbind(subject_test, Y_test, X_test)
X_train <- cbind(subject_train, Y_train, X_train)
'''

'''R
## Combine test and train datasets
## Note that the two sets are partitions of an initial set, so
## no matching or merging is needed, rbind() is sufficient
merged_set <- rbind(X_test, X_train)
'''

'''R
## Order by subject_id and then by activity
merged_set <- merged_set[order(merged_set[,1], merged_set[,2]),]
'''

'''R
## Summarize each column by subject_id and activity
mean_set <- merged_set %>% group_by(subject_id, activity) %>% summarise_each(funs(mean))
'''

'''R
## Export result to txt file
write.table(mean_set, file.path(getwd(), "mean_set.txt"), row.names = FALSE)
'''
