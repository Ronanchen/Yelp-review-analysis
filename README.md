# BD-Prj-Yelp
Big data final project about Yelp data

Group Member: Jiaqi Chen, Haoning Liu, Xiaolu Li, Mengqi Li


## Abstract
The objective is to apply the big data analytical concept and tools taught in Big data course to extract insightful analytics and help existing restaurant owners. We applied descriptive analysis, predictive analysis and sentiment analysis to Yelp dataset.


## Introduction
Yelp is a local business directory and social networking reviews website. It enables customers to give companies ratings and reviews. Usually the review is brief text composed of a few lines with approximately 100 phrases. Often, a review defines different dimensions of a company and the user's experience with regard to those aspects.


## Explanation for changing the dataset
Originally, our group started from NYU Parking tickets. We conduct descriptive analysis of the dataset using Pyspark, S3 Hardoop. However, we didn't find an intereting aspect of this dataset to starting on modeling. Therefore, we decide to change the dataset. Currently, we are working onn Yelp restaurant review dataset.


## Data Introduction:
1. Data Source: Kaggle (link: https://www.kaggle.com/yelp-dataset/yelp-dataset)
This dataset is a subset of Yelp's businesses, reviews, and user data. It was originally put together for the Yelp Dataset Challenge which is a chance for students to conduct research or analysis on Yelp's data and share their discoveries. In the dataset you'll find information about businesses across 11 metropolitan areas in four countries.

2. Context
Yelp provides 7 datasets separately and our project only gets involved with five of them, including data in business, user, tip, review and check-in. Business ID and user ID are the keys connceting each file.

In total, there are:
5,200,000 user reviews;
Information on 174,000 businesses;
Data spans 11 metropolitan areas.

To be specific:
In review file, Yelp provides data including ID, rating stars, review data, review text, and number of reviews noted as 'cool', 'useful' and 'funny'. 
Yelp business file contains business name, category, location, rating stars, number of reviews, open time as well as 39 detailed attributes to rate the business, such as whether there's TV, WiFi and whether the environment is good for kids.
Yelp tip data contains ID, review text and date.
Yelp user data contians information about consumers, for example, ID, their average rating stars, the number of their reviews noted as  'useful', 'cool' and 'funny' as well as their Yelp account information and their friends on Yelp.

3. Size
The size of whole datasets is around 4 GB. It shrinks after we transfer them from Json into CSV and total size of CSV files is roughly 3 GB.

4. Format
Original files we download is in Json, but we transfer it into CSV format by R.


## Methods and Tools
### R
Read Json files and write out as CSV files.

Why R? R provides efficient package 'jsonlite' to transfer file format with only one line. Although total size of downloaded files are 4 GB, a little large to run in R, we transferred each file separately and R code runs well, taking no more then 20 minutes for the largest file in size of 2.9 GB to read in, transfer and write out.

### Python/ command line
Clean up datasets and deal with punctuation problems in 'text' columns in several files.

Why Python/ command line? We found the punctuation problem in process of merging and print schema in Python. Complex punctuation in 'text' columns leads to different number of coulumns in each row. So we deal with it with Python. We also tried command line and it worked. Although more concise, command line code with regular expression is more challengeable to write.

### PySpark
Merge data, build up Logistic model and analyze sentiment.

Why PySpark? It runs quicker with big data and is able to train machine learning models.


## Data Cleaning Up Processing

1. Convert Json to CSV using R with a code example as following:
```
library(jsonlite)

file_name <- 'yelp_academic_dataset_tip.json'

tip<-jsonlite::stream_in(textConnection(readLines(file_name, n=1000000)),verbose=F)  # Adjust number of readLines with lenghth of each Json file.

length(tip)

write.csv(tip,"yelp_tip.csv",row.names = F)
```

2. Upload CSV files onto S3 bucket.

3. Load data from S3 and use Python to clean up 'text' columns, removing comma, quotation mark and \r\n from 'text'. Separate data between columns with comma and quotation mark and data between rows with \r\n.

4. Deal with null values for some important attributes.

5. Merge four data files together with buissness ID and user ID.



## Descriptive Analysis

We mainly conduct descriptive analysis on 3 datasets: business_data, review_data and user_data, to provide a comprehensive insights before we build models and  develop other analysis.

1. In business_data, we firstly count how many business owners are in each state, and the top 6 states are Arizona, Nevada, Ontario, North Carolina, Ohio and Pennsylvania, which are all above 10,000. Then we analyze the distribution of stars that all business owners had received. People tend to give 4.0, 3.5, 5.0, 4.5 stars, which are all higher than neutual one, 3.0.

2. In review_data, we deal with the problem about how many "useful", "cool" and "funny" compliments are got in every review. All three kinds of compliments show a monotonic relationship, there are less and less reviews when the compliments in one review increase. For example, review that receives zero "useful" or "cool" or "funny" is more than 50%. The review that gets most "useful" have 1241 ones, and there are only 2 reviews get more than 1000 "useful" compliments. The review that gets most "funny" have 1290 ones, and that is the only review gets more than 1000 "funny" compliments. The review that gets most "cool" have 506 ones, and that is the only review gets more than 500 "cool" compliments. So users tend to compliment a review as useful or funny, more than cool.

3. In user_data, firstly we count how many fans does each account have. Its' monotonic again, more than a half accounts have 0 fan, and the number of users become less and less when fans volumn increases. The most popular user has 9538 fans, while the second popular user only have 2964. About what's average stars does each user give, most of them give 5.0, 1.0, 3.0, 4.0, which also reveals that most of our users only leave a review once. The average stars from users that give multiple reviews are 3.67, 4.33, 2.33, 4.67, 4.2 and 3.33 which almost are neutral or positive.
   

## Logistic Model

We try to predict rating for business on Yelp by reasonalbe attributes in data. Considering the distribution of rating scores, from 1 star upto 5 stars, we define no more than 3 stars as low rating and convert the number of stars into 0 and those with over 3 stars as high rating and convert it into 1. Thus we get the column for prediction named stars as boolean value.

Attributes we choose as independent variables: Business ID, User ID, rating stars in review dataset, number of reviews tagged as 'useful', 'funny' and 'cool', business location including state, city, latitude and longitude, average rating stars for business, number of reviews, average rating stars given by users, number of compliment reviews.


### Model Processing
1. Drop 134 rows with null values in stars column.
2. Make dependent variable boolean. Convert stars no more than 3 to '0' while stars more than 3 to '1'.
3. Convert all the string fields to numeric ones.
4. Build a logistic regression model.
5. Create a feature vector by combining all features together.
6. Create 2 splits of combined data (train, test) by using the randomSplit method.
7. Train model with training data.
8. Make predictions with testing data.
9. Check model AUC.


### Model validation and conlusions
Model accuracy is 0.818937. It performs well.



## Sentiment Analysis
### Data preprocessing
1. Target variable recoding: We relabeled the stars column so that any reviews with 4 stars or above will be 1, which means positive, anything else is deemed to be 0, which means negative reviews. 
```# relabel target variable
def convert_rate(rate):
    rate = int(rate)
    if rate >=4: return 1
    else: return 0
 ```       
2. Then We begin to do text processings before tokenizing the text.
   a. Remove punctuation
   b. Remove stopwords
   C. Make tokens   
   We leverage the methords in pyspark ml features.```from pyspark.ml.feature import *```
3. Create trigrams whose frequency larger than 10
   a. We use ```ngram``` to create trigrams. An n-gram is a contiguous sequence of n items from a given sample of text or speech. Trigrams means three words appear together.
   b. We use MapReduce functions in python to identify the Trigrams with more than 10 times in the reviews.
```ngrams = add_ngram.rdd.flatMap(lambda x: x[-1]).filter(lambda x: len(x.split())==n)```
   c. We will replace the original text with trigrams and then tokenize them. 
   d. We then create TF-IDF matrix to make preparation for the model
  
 ### Model 
 1. We split data into training and test set.
 2. We define hyper-parameters in SVM model: regParam and numIterations.
 3. Model evaluation: Since we know that a lot of data are labeled 1, it is an imbalanced dataset. So we chooss to use F1, a weighted average of precision and recall -- could evaluate the model. Hence, the model returns an F1 score of 88.48%



## Conclusions

### Descriptive Analysis
People tend to give neutral or positive reviews and stars to busniess owners. But for people who only leave one review on Yelp, they usually have extreme feelings on that experience, so most of them give either 5 stars or 1 star.

### Logistic Model
We can predit whether a business gets high or low rating with factors including number of stars in review data, number of reviews tagged as 'useful', 'funny' and 'cool', business location including state, city, latitude and longitude, average rating stars for business, number of reviews, average rating stars given by users, number of compliment reviews.

### Sentiment Analysis
Supportvector Machine to predict positive or negative reviews with 88.48% F1 score.



## Challenges and Solutions
1. Transfer data from Json to CSV format. We tried several different R packages to find a way to get data in correct structure in CSV format.

2. Read in data with text columns in a neat format. We spent a lot time in this process and solved it with changing and removing punctuation in 'text' columns duplicated with punctuation used to separate columns and rows. Command line also worked for this problem.

3. Upload data onto S3 bucket from local cost long time.

4. Code for model trainning and testing doesn't respond due to the large size of data. We solved it by creating cluster with more instances and higher CPU, memory and storage.

5. Limited resources online regarding pyspark machine learning. It's difficult to debug.

