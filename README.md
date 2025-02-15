# -Getting-and-Cleaning-Data-Course-Project
Peer-graded Assignment: Getting and Cleaning Data Course Project
# Load required libraries
library(dplyr)

# 1. Download and unzip the dataset
url <- "https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip"
download.file(url, destfile = "dataset.zip", mode = "wb")
unzip("dataset.zip")

# Define the dataset path
path <- "UCI HAR Dataset"

# 2. Load the data
features <- read.table(file.path(path, "features.txt"))
activity_labels <- read.table(file.path(path, "activity_labels.txt"), col.names = c("activityID", "activityName"))

x_train <- read.table(file.path(path, "train/X_train.txt"))
y_train <- read.table(file.path(path, "train/y_train.txt"))
subject_train <- read.table(file.path(path, "train/subject_train.txt"))

x_test <- read.table(file.path(path, "test/X_test.txt"))
y_test <- read.table(file.path(path, "test/y_test.txt"))
subject_test <- read.table(file.path(path, "test/subject_test.txt"))

# 3. Merge training and test datasets
data_X <- rbind(x_train, x_test)
data_Y <- rbind(y_train, y_test)
data_subject <- rbind(subject_train, subject_test)

# Assign column names
colnames(data_X) <- features$V2
colnames(data_Y) <- "activityID"
colnames(data_subject) <- "subjectID"

# 4. Extract columns containing "mean" and "std"
selected_columns <- grep("mean\(\)|std\(\)", features$V2)
data_X <- data_X[, selected_columns]

# 5. Merge data with activity and subject information
data_merged <- cbind(data_subject, data_Y, data_X)

# 6. Assign descriptive activity names
data_merged$activityID <- factor(data_merged$activityID, levels = activity_labels$activityID, labels = activity_labels$activityName)

# 7. Improve column names
names(data_merged) <- gsub("^t", "Time", names(data_merged))
names(data_merged) <- gsub("^f", "Frequency", names(data_merged))
names(data_merged) <- gsub("Acc", "Accelerometer", names(data_merged))
names(data_merged) <- gsub("Gyro", "Gyroscope", names(data_merged))
names(data_merged) <- gsub("Mag", "Magnitude", names(data_merged))
names(data_merged) <- gsub("BodyBody", "Body", names(data_merged))

# 8. Create a new dataset with the average of each variable for each activity and subject
tidy_data <- data_merged %>%
  group_by(subjectID, activityID) %>%
  summarise_all(mean)

# 9. Save the final dataset
write.table(tidy_data, "tidy_data.txt", row.name = FALSE)
