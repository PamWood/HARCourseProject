HARCourseProject
================

Getting and Cleaning Data - HAR Course Project

I have one script.  

The first part downloads the file to a directory I've created, unzips the file, and changes my working directory to that file.
if(!file.exists("~/Training/DataScienceClass/data/CourseProject")) { dir.create("~/Training/DataScienceClass/data/CourseProject")}
FilePath<-file.path("~/Training/DataScienceClass/data/CourseProject", "HAR.zip")
FiletoUnzip<-"https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip"
download.file(FiletoUnzip, FilePath)
unzip(FilePath)
setwd("~/Training/DataScienceClass/data/CourseProject/UCI HAR Dataset")

The second part reads the "features" table.  
I apply Step 4 here, removing all of the "()" and "-", etc using "gsub". 
Doing this effectively renames the features - I did not change the cases because the case sensitivity becomes important when regarding mean and std.
features<-read.table("./features.txt")
features$V3<-gsub("[^[:alnum:]]","", features$V2)
features$V2<-NULL
names(features)<-c("list","features")


The third part reads the "activity labels" table.  All I do is rename the columns.
activitylabels<-read.table("./activity_labels.txt")
names(activitylabels)<-c("actlabel","activity")

The fourth part reads the "test" tables, renames ths columns, combines the "test" tables, then applies the "features" column of the "features" table as the column headers for the "Test" dataset
SubTest<-read.table("./test/subject_test.txt", header=FALSE, sep="")
names(SubTest)<-"subject"
XTest<-read.table("./test/X_test.txt", header=FALSE, sep="")
YTest<-read.table("./test/Y_test.txt", header=FALSE, sep="")
names(YTest)<-"activity"
Test<-cbind(SubTest,YTest,XTest)
names(Test)[3:563]=features[,2]

The fifth part reads the "train" tables, renames ths columns, combines the "train" tables, then applies the "features" column of the "features" table as the column headers for the "Train" dataset
SubTrain<-read.table("./train/subject_train.txt", header=FALSE, sep="")
names(SubTrain)<-"subject"
XTrain<-read.table("./train/X_train.txt", header=FALSE, sep="")
YTrain<-read.table("./train/Y_train.txt", header=FALSE, sep="")
names(YTrain)<-"activity"
Train<-cbind(SubTrain,YTrain,XTrain)
names(Train)[3:563]=features[,2]

The sixth part completes Step 1 and merges the training and test sets to create one data set.
This part also completes Step 3 by replacing the "activity label" with the "activity name"
DataSet<-rbind(Test,Train)
DataSetAll<-merge(activitylabels,DataSet, by.x="actlabel", by.y="activity", all=TRUE)
DataSetAll$actlabel<-NULL

The seventh part completes Step 2 and extracts Only Mean and Std for each measurement. 
I did this by using "grep" for the columns I was looking for, then combined the columns together.
Here is why I left the "features" as Upper and Lower case because the upper case "Means" were not items we wanted in this set
MeanData<-DataSetAll[grep("mean",names(DataSetAll))]
StdData<-DataSetAll[grep("std",names(DataSetAll))]
MeanStd<-cbind(MeanData,StdData)
Sub<-DataSetAll[grep("subject",names(DataSetAll))]
Act<-DataSetAll[grep("activity",names(DataSetAll))]
SubAct<-cbind(Sub,Act)
MeanStdData<-cbind(MeanStd,SubAct)


The final part completes Setep 5 and Creates a second, independent tidy data set with the average of each variable for each activity and each subject
I had to install "reshape2" package to do this.
Then I write the file as both .txt and .csv
library('reshape2')
DF<-data.frame(MeanStdData)
mdata <- melt(DF, id=c("subject","activity"))
mdata <- melt(DT, id=c("subject","actlabel"))
FinalDataSet<-dcast(mdata,subject+activity~variable,fun.aggregate=mean)
write.table(FinalDataSet, "HARMerged.txt", sep=";") 
write.csv(FinalDataSet, "HARMerged.csv")
