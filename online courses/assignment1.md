
Create an RDD named `products` with `parallelize` containing the elements in the output.


```python
products = sc.parallelize(['Apple', 'Apple', 'Cheese', 'Apple', 'Orange'])
```

Count the number of elements in `products`


```python
products.count()
```




    5



Count the number of apples in `products`. Tip: use filter.


```python
products.filter(lambda x: x == 'Apple').count()
```




    3



show the (distinct) products.


```python
products.distinct().collect()
```




    ['Apple', 'Orange', 'Cheese']



Load the contents of the file babynames from the data folder into a RDD called `babynames` with textFile. Show the first 5 lines.


```python
import os
if not os.path.exists("babynames.csv"):
    import urllib.request
    f = urllib.request.urlretrieve ("https://health.data.ny.gov/api/views/jxy9-yhdk/rows.csv?accessType=DOWNLOAD", \
                                    "babynames.csv")
    
babyrddprimitive = sc.textFile("babynames.csv")
babyrddprimitive.take(5)
```




    ['Year,First Name,County,Sex,Count',
     '2013,GAVIN,ST LAWRENCE,M,9',
     '2013,LEVI,ST LAWRENCE,M,9',
     '2013,LOGAN,NEW YORK,M,44',
     '2013,HUDSON,NEW YORK,M,49']



The first line in the file is a header, filter out the first line to keep only lines with actual data.


```python
header = babyrddprimitive.first()
babyrddfiltered = babyrddprimitive.filter(lambda x: x != header)
babyrddfiltered.take(5)
```




    ['2013,GAVIN,ST LAWRENCE,M,9',
     '2013,LEVI,ST LAWRENCE,M,9',
     '2013,LOGAN,NEW YORK,M,44',
     '2013,HUDSON,NEW YORK,M,49',
     '2013,GABRIEL,NEW YORK,M,50']



The elements in this RDD are each a line of text. Transform each element into a tuple or list that consists of the 5 columns in the csv by splitting the lines on comma characters. Show the first 5. Tip: you need `map` and the `split` method on Python Strings.


```python
splitRDD = babyrddfiltered.map(lambda line: line.split(','))
splitRDD.take(5)
```




    [['2013', 'GAVIN', 'ST LAWRENCE', 'M', '9'],
     ['2013', 'LEVI', 'ST LAWRENCE', 'M', '9'],
     ['2013', 'LOGAN', 'NEW YORK', 'M', '44'],
     ['2013', 'HUDSON', 'NEW YORK', 'M', '49'],
     ['2013', 'GABRIEL', 'NEW YORK', 'M', '50']]



Count how many male babies are in the RDD.


```python
maleBabies = splitRDD.filter(lambda gender: gender[3] == 'M')
maleBabies.count()
```




    70137



The next objective is to find the most given babyname.

First, convert the RDD into a key,value structure. Since we do not need anything but the name, we can convert every element into (name, 1). Show the first 5.


```python
babynames = splitRDD.map(lambda line: (line[1], 1))
babynames.take(5)
# [('GAVIN', 1), ('LEVI', 1), ('LOGAN', 1), ('HUDSON', 1), ('GABRIEL', 1)]
```




    [('GAVIN', 1), ('LEVI', 1), ('LOGAN', 1), ('HUDSON', 1), ('GABRIEL', 1)]



Now you can aggregate the elements that have the same key, and sum the values to get the number of occurrences per name. Show the first 5, these might be different ones than displayed below. Tip: use `reduceByKey`


```python
babynamesOccurences = babynames.reduceByKey(lambda accum, n: accum + n)
babynamesOccurences.take(5)
# [('ADRIEN', 1), ('HERSH', 12), ('JASE', 10), ('JUAN', 62), ('AZIZA', 1)]
```




    [('GAVIN', 262),
     ('LEVI', 148),
     ('LOGAN', 386),
     ('HUDSON', 100),
     ('GABRIEL', 243)]



Now `map` the name,frequency pairs so that you only have the values and use the `max` action to get the highest value.


```python
maxFrequency = babynamesOccurences.map(lambda f: f[1]).max()
print(maxFrequency)
# [386]
```

    386


And revert back to the name,frequency pairs and filter the pair(s) that have a frequency equal to the max you found.


```python
femaleBabies = splitRDD.filter(lambda gender: gender[3] == 'F')
femalebabynames = femaleBabies.map(lambda line: (line[1], 1))
fbabynamesOccurences = femalebabynames.reduceByKey(lambda accum, n: accum + n)
maxFemaleFrequency = fbabynamesOccurences.map(lambda f: f[1]).max()
filterMaxFrequency = fbabynamesOccurences.filter(lambda f: f[1] == maxFemaleFrequency)
filterMaxFrequency.take(5)
# [('EMMA', 322)]
```




    [('EMMA', 383)]


