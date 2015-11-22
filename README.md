# Get-and-Cleaning-Data

This file includes a discription of how the script works.

1. First I import the data into R creating a directory "Data" in the case where there is no one.

if(!file.exists("./data")){dir.create("./data")}
fileUrl <- "http://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip"
download.file(fileUrl,destfile="./data/DataProgect.zip")
unzip(zipfile="./data/DataProgect.zip",exdir="./data")
path_rf <- file.path("./data" , "UCI HAR Dataset")
files<-list.files(path_rf, recursive=TRUE)
files

2. I put the data into the variables (Activity-Subject-Features)

ActTest  <- read.table(file.path(path_rf, "test" , "Y_test.txt" ),header = FALSE)
ActTrain <- read.table(file.path(path_rf, "train", "Y_train.txt"),header = FALSE)
SubTrain <- read.table(file.path(path_rf, "train", "subject_train.txt"),header = FALSE)
SubTest  <- read.table(file.path(path_rf, "test" , "subject_test.txt"),header = FALSE)
FeatTest  <- read.table(file.path(path_rf, "test" , "X_test.txt" ),header = FALSE)
FeatTrain <- read.table(file.path(path_rf, "train", "X_train.txt"),header = FALSE)

3. I merge the datasets

dataSub <- rbind(SubTrain, SubTest)
dataAct<- rbind(ActTrain, ActTest)
dataFeat<- rbind(FeatTrain, FeatTest)

4. I chage the names

names(dataSub)<-c("subject")
names(dataAct)<- c("activity")
FeatNames <- read.table(file.path(path_rf, "features.txt"),head=FALSE)
names(dataFeat)<- FeatNames$V2

5. I create a data frame

dataComb <- cbind(dataSub, dataAct)
Data <- cbind(dataFeat, dataComb)

6. Selecting only Mean and St.Dev

subFeatNames<-FeatNames$V2[grep("mean\\(\\)|std\\(\\)", FeatNames$V2)]
selectNames<-c(as.character(subFeatNames), "subject", "activity" )
Data<-subset(Data,select=selectNames)

str(Data)

7. I name the activities in the data set

actLabels <- read.table(file.path(path_rf, "activity_labels.txt"),header = FALSE)
head(Data$activity,30)

8. I give more descreptive names

names(Data)<-gsub("^t", "time", names(Data))
names(Data)<-gsub("^f", "frequency", names(Data))
names(Data)<-gsub("Acc", "Accelerometer", names(Data))
names(Data)<-gsub("Gyro", "Gyroscope", names(Data))
names(Data)<-gsub("Mag", "Magnitude", names(Data))
names(Data)<-gsub("BodyBody", "Body", names(Data))

names(Data)

9. Creating tidy Data set

Data2<-aggregate(. ~subject + activity, Data, mean)
Data2<-Data2[order(Data2$subject,Data2$activity),]
write.table(Data2, file = "tidydata.txt",row.name=FALSE)
