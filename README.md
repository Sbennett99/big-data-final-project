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
#### Map - Filter - Reduce - Sort
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
- Retrieve stopwords and remove them from the data
```Python
remover = StopWordsRemover()
# retrieving stop words
stopwords = remover.getStopWords()

stopwordFreeWords = cleanedWordsRDD.filter(lambda word: word not in stopwords)
```
- Pair the words into tuples, then Reduce them by value
```Python
# preparing for word reduction(pairing words)
preReductionWords = stopwordFreeWords.map(lambda word: (word,1))

# reducing/counting words
reducedWords = preReductionWords.reduceByKey(lambda ac,val: ac + val)
```
- sort the data from highest to lowest then taking the top ten
```Python
resultsRDD = reducedWords.sortBy(lambda t: t[1], ascending = False)

filteredResults = resultsRDD.take(10)
```

#### Ploting the processed data
- Import necessary Library(s) for the job
```Python
# more information at: https://pandas.pydata.org/about/index.html
import pandas as pd

# more information at: https://matplotlib.org/
import matplotlib.pyplot as plt

# more information at: https://seaborn.pydata.org/introduction.html
import seaborn as sns
```
- Create a dataframe from the data that I want to graph
```Python
#Creating dataframe
df = pd.DataFrame.from_records(filteredResults, columns =[xlabel, ylabel]) 
```
- Set figure size and create a barplot with the dataframe
```Python
plt.figure(figsize=(10,3))

sns.barplot(xlabel, ylabel, data=df, palette="rocket").set_title(title)
```
<div align="center">*graph made above displayed in results*</div>

## The Results
![](https://github.com/Sbennett99/big-data-final-project/blob/74323633a4cf57272c643fc8e3b226f01bff58ce/GraphOfEragonWords.png)

The words shown in this barplot are the top ten words used in the entirety of the book Eragon. The words from greatest to least are as follows [eragon, said, brom, saphira, one, like, murtagh, don't, back, know]. To someone who knew nothing about the book, they may find the names of may decide that the names of the main characters are as follows [eragon, brom, saphira, murtagh].

## Websites I found Useful
[StackOverflow: Punctuation removal using Strip](https://stackoverflow.com/questions/18429143/strip-punctuation-with-regex-python)

[PDFdrive: Source of my data](https://www.pdfdrive.com/eragon-d37470593.html)

[PDF to TXT Converter: For converting my data from PDF to .txt](https://www.zamzar.com/convert/pdf-to-txt/)
