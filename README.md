# Tarun-Bigdata-Project

## Author : [Tarun Sarpanjeri](https://github.com/dexterstr)

## Text Source

- [The Project Gutenberg eBook of The Great Gatsby, by F. Scott Fitzgerald](https://www.gutenberg.org/files/64317/64317-h/64317-h.htm)

## Languages and Tools used:

- Python
- DataBricks Notebook
- PySpark
- Pandas
- Spark processing engine

## NoteBook from Databricks :

Here is the [Link.](https://community.cloud.databricks.com/?o=7861382908486399#notebook/1121519894951393/command/1121519894951394)

## Steps for Gathering

To pull the data into notebook we use urllib.request to pull data from a url and store it in a temporary text file named 'tarun.txt'. The data I have used is saved in a text-file in my github [repo.](https://github.com/dexterstr/Tarun-Bigdata-Project)

```
# Import the library for processing url request.
import urllib.request
# Store the data by retrieving into a temporary file
urllib.request.urlretrieve("t" , "/tmp/tarun.txt")

```

For saving data into notebook, we use method 'dbutils.fs.mv' which uses two arguments for sending data from one location to other.

```
dbutils.fs.mv("file:/tmp/tarun.txt","dbfs:/data/tarun.txt")
```

Alas, Spark holds data in RDDs format i.e. Resilient Distributed Databses so,we transform data into RDDs.

```
tarunRDD = sc.textFile("dbfs:/data/tarun.txt")
```

## Steps for Cleaning

As the data contains punctuations, sentences and even empty lines ans StopWords. We clean the data by splitting each line by spaces and changing complete text to lower-case, breaking all sentences into words and removing empty lines.

```
cleanRDD=tarunRDD.flatMap(lambda line : line.lower().strip().split(" "))
```

All punctuations can be removed by using Regular Expression which finds terms except letters.To use Reg-ex , library 're' must be imported.

```
import re
cleanTokensRDD = cleanRDD.map(lambda w: re.sub(r'[^a-zA-Z]','',w))
```

Finally,StopWords must be removed.PySpark knows StopWords so, we just need to import StopWordsRemover which will filter out the words.

```
from pyspark.ml.feature import StopWordsRemover
remover =StopWordsRemover()
stopwords = remover.getStopWords()
cleanwordRDD=cleanTokensRDD.filter(lambda w: w not in stopwords)
```

## Steps for Processing

Mapping the words into key-Vlaue pairs where we will be taking word as key and check how many times it occurs and save the number in format(word,1).

```
KeyValuePairsRDD= cleanwordRDD.map(lambda word: (word,1))
```

Reduce by Key-Key is the word. we'll save the word and when it repeats , we will remove it and add 1 to count.

```
wordCountRDD = KeyValuePairsRDD.reduceByKey(lambda acc, value: acc+value)
```

To retrieve all elements from data we will use collect method to save and use print()method to show result.

```
results = wordCountRDD.collect()
print(results)
```

we will use SortByKey method to list words in descending order and print top 11 results in 'The Great Gatsby'.

```
sort_results = wordCountRDD.map(lambda x: (x[1], x[0])).sortByKey(False).take(11)
print(sort_results)
```

## Charting

We will be using MatplotLib to plot graph.

```
mostCommon=results[1:5]
word,count = zip(*mostCommon)
import matplotlib.pyplot as plt
fig = plt.figure()
plt.bar(word,count,color="black")
plt.xlabel("Number of times used")
plt.ylabel("Most used")
plt.title("Most used words in The Monk")
plt.show()
```

# Results

![Sorting]()
![Results]()

# References
