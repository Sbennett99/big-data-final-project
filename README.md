# Big Data Final Project -- Word Count and Sort
#### By Seth Bennett

## The Data
- What is it?
  - For the data of this project I decided to use a book that I really enjoyed reading and as a bonus is of decent length comming in at around 400 pages. The name of this book is Eragon by Christopher Paolini

![](Eragon_book_cover.png)

[Eragon.txt](https://github.com/Sbennett99/big-data-final-project/blob/25e9e511d9bd818348fb2b9b5843ccdf6fa93500/Eragon.txt)
- Where its From
  - I got the PDF of Eragon from [PDF Drive](https://www.pdfdrive.com/eragon-d37470593.html) and used an online [PDF to txt Converter](https://www.zamzar.com/convert/pdf-to-txt/) to get a txt file to process

## Processing the Data
### Tools to process the data
####  Databricks Community - A free non professional cloud based processing tool
  - Python - all data is processed using the Python Programming Language and within Python the following librarys were used
    - urllib.request: to retrieve the txt from a given url
    - PySpark: to actually perform all processing
    - re: a library with various regex tools to filter out unwanted characters
    - StopWordsRemover: to retrieve a list stop words
    - pandas: to create a data frame from the processed data
    - matplotlib.pyplot: to set the size of the desired plot
    - seaborn: to create a graph from the data frame
### Steps to Process the data
#### Importing the data for processing
- Import necessary Library(s) for the job
```Python
import urllib.request as u
```
- Retrieve the Data
```Python
stringInURL = "https://raw.githubusercontent.com/Sbennett99/big-data-final-project/main/Eragon.txt"
stringTmp = "/tmp/Eragon.txt"
stringDbfs = "/data/Eragon.txt"
u.urlretrieve(stringInURL, stringTmp)
```
Move the Data into Databricks
```Python
filePrefix = "file:"
dbfsPrefix = "dbfs:"
dbutils.fs.mv(filePrefix + stringTmp, dbfsPrefix + stringDbfs)
```
Create an RDD using data to prepare the data for processing using PySpark
```Python
nservers = 4
inRDD = sc.textFile(dbfsPrefix + stringDbfs,nservers)
```
#### Map - Filter - Reduce
- Import necessary Library(s) for the job
```Python
import string
import re
from pyspark.ml.feature import StopWordsRemover
```
- Flat map the data, spliting everything by spaces - striping the extra spaces and pesky characters(Characters that nothing will recognize as what they are for some reason) - then forcing everything to lowercase
```Python
messyWordsRDD = inRDD.flatMap(lambda l: l.lower().strip().strip(".,“").strip(string.punctuation).split(" "))
```
- Clean the data of all non letter characters(for the most part, some punctuation would not filter out ) 
```Python
cleanedWordsRDD = messyWordsRDD.filter(lambda word: re.sub(r'[^a-zA-Z]' ,'',word))
```
#### Ploting the processed data


