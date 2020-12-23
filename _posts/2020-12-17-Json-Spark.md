---
title: 'Anatomy of Semi-Structured(JSON) data with PYSPARK'
date: 2020-10-25
permalink: /posts/2020/12/blog-post-6/
tags:
  - Apache Spark
  - Json
---

### Outlines

* Complex data types in Spark

* Parse json file in Spark

* Flatten the data in Spark

###Data Types in Spark

Before starting parsing json, it is really importnat to have good idea about the data types usually used in json. The json data may contains different data types. The most common data types that we encounters are given below:

	* Array
	* Struct
	* Map

### Array 

An Array in Spark is list of homogenous elements, which means the data type of the elements are same. For instance, there is an example given below.

```
input_json = """
{
  "numbers": [1, 2, 3, 4, 5, 6]
}
"""
adf = spark.read.json(sc.parallelize([input_json]))
adf.printSchema()
```
```
root
 |-- numbers: array (nullable = true)
 |    |-- element: long (containsNull = true)
```
if we look at the shcema, we can see the data types of the json input data. This is just a array of elements.

We can also see the data after readin the json. Let's see how the data looks like:

```
adf.show(truncate=False)
```

```
+------------------+
|numbers           |
+------------------+
|[1, 2, 3, 4, 5, 6]|
+------------------+
```

It is really cool to get the data in a more nicer way. For instances, we can do some transformation to flatten the data with some built in spark functions. Let's use the explode function to make the above data fit into a single columnd with each elements in a different row. 

```
from pyspark.sql.functions import explode
adf.select(explode('numbers').alias('number')).show()

```
```
+------+
|number|
+------+
|     1|
|     2|
|     3|
|     4|
|     5|
|     6|
+------+
```
The .alias is just a built in method which rename the column.

### Struct 

This data type is little bit different that others. It is a grouped list of variables with dfferent data types. The variables can be accessed by a single parent pointer. We can use dot "." notation to acess the elements inside a struct type. For instances, we will see a exaple json that contains struct. 

```
input_json = """
{
  "car_details": {
     "model": "Tesla S",
     "year": 2018
  }
}
"""
sdf = spark.read.json(sc.parallelize([input_json]))
sdf.printSchema()
```
```
root
 |-- car_details: struct (nullable = true)
 |    |-- model: string (nullable = true)
 |    |-- year: long (nullable = true)
```

From the schema we can see the struct type with different type variables like string and long.

Let's see how the data looks like after reading the input json data.

```
sdf.show()
```
```
+---------------+
|    car_details|
+---------------+
|[Tesla S, 2018]|
+---------------+


```

Now, we will see how we can access the elements inside a struction type with the dot "." notation. 

```
sdf.select(sdf.car_details.model, sdf.car_details.year).show()

```
```
+-----------------+----------------+
|car_details.model|car_details.year|
+-----------------+----------------+
|          Tesla S|            2018|
+-----------------+----------------+
```

An Alternate Method for the same is present below,

```
from pyspark.sql.functions import col
sdf.select(col('car_details.model'), col('car_details.year')).show()
```
```
+-------+----+
|  model|year|
+-------+----+
|Tesla S|2018|
+-------+----+
```

### Map

Map is just an element that consists of key-value pair. It is almost as same as dictionary in python.
We wil see now how we can deal with map elements from a json record.

```
from pyspark.sql.types import StructType, MapType, StringType, IntegerType
input_json = """
{
  "Car": {
    "model_id": 835,
    "year": 2008
  }
}
"""
schema = StructType().add("Car", MapType(StringType(), IntegerType()))
mdf = spark.read.json(sc.parallelize([input_json]), schema=schema)
mdf.printSchema()
```

```
root
 |-- Car: map (nullable = true)
 |    |-- key: string
 |    |-- value: integer (valueContainsNull = true)

```

From the schema we can see the Car element is a map data type which  has key value pair. 

The data will look like:

```
mdf.show(truncate=False)
```
```
+-------------------------------+
|Car                            |
+-------------------------------+
|[model_id -> 835, year -> 2008]|
+-------------------------------+
```

We can interact with the Map data by accessing the key of the each elements. Here, for the Car, we can acess the key by using follwing syntax:


```
mdf.select(mdf.Car['model_id'], mdf.Car['year']).show()

```
```
+-------------+---------+
|Car[model_id]|Car[year]|
+-------------+---------+
|          835|     2008|
+-------------+---------+
```

Now, we have seen the important Spark data tyeps and we will use this knowledge to parse complex JSON records. Let's work with some real JSON records now. 


### Reading the json records

We can use the spark dataframe to read the json records using Spark. But as spark accepts json data that satisfies the follwowing criteria.

Note: Spark accepts JSON data in the new-line delimited JSON Lines format, which basically means the JSON file must meet the below 3 requirements,

* Each Line of the file is a JSON Record
* Line Separator must be ‘\n’ or ‘\r\n’
* Data must be UTF-8 Encoded

A Simple Example of a JSON Lines Formatted data is shown below,

```
{
   {“id” : “1201”, “name” : “satish”, “age” : “25”}
   {“id” : “1202”, “name” : “krishna”, “age” : “28”}
}
```

As seen from above, each JSON record spans a new line with a new line separator.

Let's read a json file consists of food ordering records. We will read the json records file with spark dataframe method.

```
df = spark.read.json(path,multiLine = "false")
```

Now, we will see the schema of the data that will show us the structure of the json data.


```
df.prinschema()
```

```
root
 |-- customerId: string (nullable = true)
 |-- orders: array (nullable = true)
 |    |-- element: struct (containsNull = true)
 |    |    |-- basket: array (nullable = true)
 |    |    |    |-- element: struct (containsNull = true)
 |    |    |    |    |-- grossMerchandiseValueEur: double (nullable = true)
 |    |    |    |    |-- productId: string (nullable = true)
 |    |    |    |    |-- productType: string (nullable = true)
 |    |    |-- orderId: string (nullable = true)
```

From the schema we can see it is a nested json complex format. We will try to parse the data and will flatten the data after some transformations. 


