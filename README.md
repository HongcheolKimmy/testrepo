



### Getting and Cleaning Data Course Project


##About Project
As described at the [Course Project page in Coursera] (https://class.coursera.org/getdata-005/human_grading/view/courses/972582/assessments/3/submissions), The purpose of this project is to demonstrate the ability to collect, work with, and clean a data set. The goal is to prepare tidy data that can be used for later analysis.

##About Data 
**Human Activity Recognition Using Smartphones Data Set**

The data linked to from the course website represent data collected from the accelerometers from the Samsung Galaxy S smartphone, more specifically, The experiments have been carried out with a group of 30 volunteers performing six activities (WALKING, WALKING-UPSTAIRS, WALKING-DOWNSTAIRS, SITTING, STANDING, LAYING) wearing a smartphone (Samsung Galaxy S II) on the waist.

##Data Source
* **Original raw data source (website)** - http://archive.ics.uci.edu/ml/datasets/Human+Activity+Recognition+Using+Smartphones
* **Data Source for the project (zip file)** - 
https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip 

##Script and Output file
* **run_analysis.R** – This is R script. By implementing this code script, you can reproduce the table and get the output file as well.
* **tidy-group-average-data** – This is a last tidy dataset that include the group wise average data for each activity by subject.

##Data Analysis Process
###Part.0 - Download the data 
* **Part.0-1) Create a new directory**

As I find it cleaner to put every project-related file into one directory, I've written code to check the existence of the directory named "CleanDataProject"(I assume there isn’t), and create that directory if doesn't exists under the current working directory.


* **Part.0-2) Download and unzip the file**

A downloaded zip file will be stored in the new directory, and it is going to be also unzipped under the same directory instead of in the current working directory by adding 'exdir' argument when writing an unzip function. Then all new files will be stored under the following directory "./CleanDataProject/UCI HAR Dataset"


* **Part.0-3) Assign a whole file path.**

I assigned this whole file path into a variable "fileDir" to avoid typing it over and over again during the next steps.



###Part.1 - Merge training and test sets to create one data set.
There are three groups of files to deal with, quasi "metadata", "train data", and "test data", and I will go through one by one at this stage by splitting up Part.1 into three small parts.

* **Part.1-1) Get the metadata : "activityLabels", "features"**

  * **'activity_labels.txt'** - This file consists of activity labels (integer number from 1 to 6) with their corresponding activity names as shown below in the table. We are going to use this file to merge with our measured data set later on.

  * **'features.txt'** - The 561 features selected for this database come from the accelerometer and gyroscope 3-axial raw signals.(for detailed information, please see ‘features_info’ file contained in the same directory as ‘features.txt’ file), and this list of all features will be used as column names of measured data set. 


* **Part.1-2) Get the 3 "train" data : "subjecttrain", "xtrain", "ytrain"**
  *	**Xtrain** : The obtained dataset has been randomly partitioned into two sets, where 70% of the volunteers was selected for generating the training data and 30% the test data. So, this X_train file consists of 70% of the all the measurement, and each column corresponds to a feature mentioned above. 

  *	**Y-train** : This is a set of activity labels performed to each measurement. The number of rows in this file(7352 rows) must be equal to that of X-train., and its range is from 1 to 5. This activity labels will be merged with activity labels in the later part. (see Part.1-1)

  *	**Subject-train** : Each row identifies the subject who performed the activity for each window sample. Its range is from 1 to 30. The number of rows is equal to two train files above.


* **Part.1-3) Get the 3 "test" data : "subjecttest", "xtest", "ytest"**
  * Same rule applies here as “train data set(Part.1-2)”. these test measurements consist of 30% of the obtained dataset.


* **Part.1-4) Combine the data to get all variables in a single data frame (named “dataSet”).**
  1. Use cbind to combine each data set ("trainSet","testSet") with a same binding order.
  2. Use rbind to combine "trainSet" and "testSet" together into one single dataset("dataSet").
  3. Now we change the column names of dataset by using feature variables. Having failed after trying several times with original data frame format “features”, I used transpose format of the data frame. For the first two variables, I just named them.


image




### Part.2) Extracts only the measurements on the mean and standard deviation for each measurement.

Some of the features which include the term either "mean" or "std" sounds very complicated. As it was not so clear for me to distinguish among them what to drop and what to keep, I decided to keep all of them in the data set. So, literally I extract all measurements which include the term "mean" and "std" except the features like “meanFreq”. Because this is clearly not about mean. As a result, and I got 73 columns including angle variables. And I named a new data frame meanstd.
Note : Before we go to the next part, I just fix one typo error here for one of the feature name.

|Dataset              | rows | columns | Details                               |
|---------------------|------|---------|---------------------------------------|
| meanstd             | 10299| 75      | Columns filtered for mean() and std() |

### Part.3) Use descriptive activity names to name the activities in the data set. 
By merging the new data frame "meanstd" with "activityLabels" data(see Part.1-1) by common column name "ActivityLabels" we get activity names from “activityLabels” data. After merging them together drop "ActivityLabels" column as we don't need it anymore. Now we have activity column replacing the former ActivityLabels column.


|Dataset              | rows | columns | Details                               |
|---------------------|------|---------|---------------------------------------|
| meanstd             | 10299| 75      | Descrpitive Activity Names included   |

### Part.4) Appropriately labels the data set with descriptive variable names.
Here we have quasi 7 different categories in each column names as below (Get an idea from the Discussion Forum), and I changed the name mostly with sub or gsub function except the first domain issue. As either “t” or “f” doesn’t always appear at the first position of line I couldn’t simply subset it with ‘^’ sign. To subset it from features, more specifically from angle variables I rather combine "t/f" with the following category(2 options, "Body/Gravity"). In case of "t", subtract only the column names that include either "tBody" or "tGravity", and replace "t" with "Time", and then put them back to the original column names. Same goes for replacing "f" with "Frequency". Here I used temporal variables “tdomain” and “fdomain”.

  1. domain: Time, Frequency
  2. source: Body, Gravity
  3. sensor: Acceleration, Gyroscopic
  4. jerk: Yes, No
  5. magnitude: Yes, No
  6. statistics: Mean, StandardDeviation
  7. axis: X, Y, Z

|Dataset              | rows | columns | Details                               |
|---------------------|------|---------|---------------------------------------|
| meanstd             | 10299| 75      | Replaced with Descrpitive Variable Names |

### Part.5) Creates a second, independent tidy data set with the average of each variable for each activity and each subject.

Use ddply(under "plyr" package) function to get a group wise average value, and change the column names by putting "Average" at the front to better describe the variables. After doing that extract tidy dataset to the file with “write.table” function.


|Dataset              | rows | columns | Details                                  |
|---------------------|------|---------|------------------------------------------|
| groupAverage        | 180  | 75      | Group-wise average by activity and subject |
