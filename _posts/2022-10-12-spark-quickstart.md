---
title: Apache Spark Quickstart 정리
description: Apache Spark Quickstart를 따라치면서 이해한 내용을 한글로 정리했습니다.
categories:
- Apache Spark
- Apache Spark Quickstart
tags: 
- Apache Spark
- DE
---

# PySpark Tutorial in Spark 3.1.3

## Load Spark

- findspark.init(): 윈도우 로컬에 설치한 spark의 경로를 찾아와줘서 pyspark를 불러올 수 있게 함


```python
import findspark
findspark.init()
```


```python
import pyspark
from pyspark.sql import SparkSession
 
spark = SparkSession.builder.getOrCreate()
```

- SparkSession.builder.getOrCreate(): 스파크 세션을 불러온다.
    - Spakr Session: Spark의 기초가 되는 function 들과 상호작용할 수 있게 해주는 접촉점이며, DataFrame 과 Dataset APIs 를 사용하여 Spark 프로그래밍을 할 수 있도록 해 줌.
    - SparkSession의 인스턴스 생성을 위한 build() 메서드를 사용하고, getOnCreate() 메서드로 생성한다..
        - 이 메서드(builder)를 이용하면 기존 인스턴스를 재사용하거나 새로운 인스턴스를 생성할 수 있다.

## DataFrame Creation

- PySpark에서 DataFrame을 생성하기 위해서는 pyspark.sql.SparkSession.createDataFrame를 사용한다.
- createDataFrame 안에는 list, tuple, dictionary를 전달해도 괜찮고, pyspark.sql.Rows, pandas DataFrame도 가능하다.
- DataFrame의 schema(=column)를 지정하여 만들 수도 있고, 지정하지 않으면 PySpark에서 유추해서 만든다.
- 먼저 pyspark.sql.Rows로 DataFrame을 만들어보자.


```python
from datetime import datetime, date
import pandas as pd
from pyspark.sql import Row

df = spark.createDataFrame([
    Row(a=1, b=2., c='string1', d=date(2000, 1, 1), e=datetime(2000, 1, 1, 12, 0)),
    Row(a=2, b=3., c='string2', d=date(2000, 2, 1), e=datetime(2000, 1, 2, 12, 0)),
    Row(a=4, b=5., c='string3', d=date(2000, 3, 1), e=datetime(2000, 1, 3, 12, 0))
])
df
```




    DataFrame[a: bigint, b: double, c: string, d: date, e: timestamp]



- schema를 지정하지 않고 tuple을 전달하여 DataFrame을 만들어보자.
- 그럼 schema를 유추해서 만들어지는 모습을 확인할 수 있을 것이다.


```python
df = spark.createDataFrame([
    (1, 2., 'string1', date(2000, 1, 1), datetime(2000, 1, 1, 12, 0)),
    (2, 3., 'string2', date(2000, 2, 1), datetime(2000, 1, 2, 12, 0)),
    (3, 4., 'string3', date(2000, 3, 1), datetime(2000, 1, 3, 12, 0))
], schema='a long, b double, c string, d date, e timestamp')
df
```




    DataFrame[a: bigint, b: double, c: string, d: date, e: timestamp]



- 이번에는 pandas의 DataFrame을 만들어서 PySpark DataFrame으로 만들어보자.


```python
pandas_df = pd.DataFrame({
    'a': [1, 2, 3],
    'b': [2., 3., 4.],
    'c': ['string1', 'string2', 'string3'],
    'd': [date(2000, 1, 1), date(2000, 2, 1), date(2000, 3, 1)],
    'e': [datetime(2000, 1, 1, 12, 0), datetime(2000, 1, 2, 12, 0), datetime(2000, 1, 3, 12, 0)]
})
df = spark.createDataFrame(pandas_df)
df
```




    DataFrame[a: bigint, b: double, c: string, d: date, e: timestamp]



- spark.sparkContext.parallelize로 RDD를 만들어서 PySpark DataFrame으로 만들어보자.


```python
rdd = spark.sparkContext.parallelize([
    (1, 2., 'string1', date(2000, 1, 1), datetime(2000, 1, 1, 12, 0)),
    (2, 3., 'string2', date(2000, 2, 1), datetime(2000, 1, 2, 12, 0)),
    (3, 4., 'string3', date(2000, 3, 1), datetime(2000, 1, 3, 12, 0))
])
df = spark.createDataFrame(rdd, schema=['a', 'b', 'c', 'd', 'e'])
df
```




    DataFrame[a: bigint, b: double, c: string, d: date, e: timestamp]



- 출력결과를 보면, 모두 같은 schema를 가지는 PySpark Dataframe이 생성되는 것을 확인할 수 있다.


```python
df.show()
df.printSchema()
```

    +---+---+-------+----------+-------------------+
    |  a|  b|      c|         d|                  e|
    +---+---+-------+----------+-------------------+
    |  1|2.0|string1|2000-01-01|2000-01-01 12:00:00|
    |  2|3.0|string2|2000-02-01|2000-01-02 12:00:00|
    |  3|4.0|string3|2000-03-01|2000-01-03 12:00:00|
    +---+---+-------+----------+-------------------+
    
    root
     |-- a: long (nullable = true)
     |-- b: double (nullable = true)
     |-- c: string (nullable = true)
     |-- d: date (nullable = true)
     |-- e: timestamp (nullable = true)
    
    

## Viewing Data

- DataFrame에서 상위 1개만 출력하기
- DataFrame.show(n=20, truncate=True, vertical=False).
    - n (int, optional): 출력할 라인 수
    - truncate (bool or int, optional): 참으로 설정되면 기본적으로 20자 이상의 문자열을 잘라낸다. 1보다 큰 숫자로 설정된 경우 긴 문자열을 길게 잘라내고 셀을 오른쪽으로 정렬한다.
    - vertical (bool, optional): 참으로 설정된 경우 출력 행을 수직으로 출력한다.(schema 값당 한 줄).


```python
df.show(1)
```

    +---+---+-------+----------+-------------------+
    |  a|  b|      c|         d|                  e|
    +---+---+-------+----------+-------------------+
    |  1|2.0|string1|2000-01-01|2000-01-01 12:00:00|
    +---+---+-------+----------+-------------------+
    only showing top 1 row
    
    

- Jupyter와 같은 notebook 환경에서 PySpark DataFrame의 출력 형태를 pandas DataFrame 처럼 변경할 수 있다.
- 사용하고 싶다면, spark.conf.set('spark.sql.repl.eagerEval.enabled', True)으로 변경해주자.


```python
spark.conf.set('spark.sql.repl.eagerEval.enabled', True)
df
```




<table border='1'>
<tr><th>a</th><th>b</th><th>c</th><th>d</th><th>e</th></tr>
<tr><td>1</td><td>2.0</td><td>string1</td><td>2000-01-01</td><td>2000-01-01 12:00:00</td></tr>
<tr><td>2</td><td>3.0</td><td>string2</td><td>2000-02-01</td><td>2000-01-02 12:00:00</td></tr>
<tr><td>3</td><td>4.0</td><td>string3</td><td>2000-03-01</td><td>2000-01-03 12:00:00</td></tr>
</table>




- 행을 수직으로 출력하기
- 행이 너무 많아져서 가로로 출력하기 어려운 경우, 수직으로 출력하면 보기가 더 간편하다.


```python
df.show(1, vertical=True)
```

    -RECORD 0------------------
     a   | 1                   
     b   | 2.0                 
     c   | string1             
     d   | 2000-01-01          
     e   | 2000-01-01 12:00:00 
    only showing top 1 row
    
    

- DataFrame의 schema와 column name 출력하기
    - column name: DataFrame.columns
    - shema: DataFrame.printSchema()


```python
df.columns
```




    ['a', 'b', 'c', 'd', 'e']




```python
df.printSchema()
```

    root
     |-- a: long (nullable = true)
     |-- b: double (nullable = true)
     |-- c: string (nullable = true)
     |-- d: date (nullable = true)
     |-- e: timestamp (nullable = true)
    
    

- DataFrame의 행 정보 출력하기
    - DataFrame.collect()
        - 모든 행의 record를 출력
    - DataFrame.take(num)
        - n개의 행 record를 출력


```python
df.collect()
```




    [Row(a=1, b=2.0, c='string1', d=datetime.date(2000, 1, 1), e=datetime.datetime(2000, 1, 1, 12, 0)),
     Row(a=2, b=3.0, c='string2', d=datetime.date(2000, 2, 1), e=datetime.datetime(2000, 1, 2, 12, 0)),
     Row(a=3, b=4.0, c='string3', d=datetime.date(2000, 3, 1), e=datetime.datetime(2000, 1, 3, 12, 0))]



- collect() 실행 시, DataFrame의 크기가 너무 큰 경우 메모리 초과 에러가 발생할 수 있으니 주의하자.


```python
df.take(1)
```




    [Row(a=1, b=2.0, c='string1', d=datetime.date(2000, 1, 1), e=datetime.datetime(2000, 1, 1, 12, 0))]



- Pandas DataFrame 으로 변환시키기
    - DataFrmae.toPandas()


```python
df.toPandas()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>a</th>
      <th>b</th>
      <th>c</th>
      <th>d</th>
      <th>e</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>2.0</td>
      <td>string1</td>
      <td>2000-01-01</td>
      <td>2000-01-01 12:00:00</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>3.0</td>
      <td>string2</td>
      <td>2000-02-01</td>
      <td>2000-01-02 12:00:00</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>4.0</td>
      <td>string3</td>
      <td>2000-03-01</td>
      <td>2000-01-03 12:00:00</td>
    </tr>
  </tbody>
</table>
</div>



## Selecting and Accessing Data

- 알다시피 PySpark DataFrame은 lazily evaluated를 수행한다.
- 그래서 단순히 column을 하나 선택한다면 어떠한 연산, 작업도 하지 않는다.
- 그러나 선택한 column에 대한 객체는 반환한다.


```python
df.a
```




    Column<'a'>



- 실제로 column에 어떠한 작업을 수행하려고 해볼 때, 단순히 column을 선택만 하면 column 객체만 반환하기 하기 때문에 <br>
선택 후 작업을 수행하려고 해도 동작하지 않고, 가장 기본적으로 column을 선택한 것과 동일한 type(pyspark.sql.column.Column)을 반환하는 것을 알 수 있다.


```python
from pyspark.sql import Column
from pyspark.sql.functions import upper

type(df.c) == type(upper(df.c)) == type(df.c.isNull())
```




    True



- 이렇게 가져온 column 객체들은 DataFrame에서 column을 선택하는 데 사용할 수 있다. 
- select()에 column 객체를 넘겨주고 show() 해보면, 결과를 확인할 수 있을 것이다.


```python
df.select(df.c).show()
```

    +-------+
    |      c|
    +-------+
    |string1|
    |string2|
    |string3|
    +-------+
    
    

 - 새로운 column을 만들 수도 있다.
    - DataFrame.withColumn(colNamestr, colColumn)
      - colNamestr (string): 새로 지정할 column 명
      - colColumn: 기준으로 삼을 column 객체


```python
df.withColumn('upper_c', upper(df.c)).show()   # column c 객체에 upper를 적용한 column 객체를 만들었고, 
                                               # withColumn을 통해 새로운 column을 생성, show()로 출력
```

    +---+---+-------+----------+-------------------+-------+
    |  a|  b|      c|         d|                  e|upper_c|
    +---+---+-------+----------+-------------------+-------+
    |  1|2.0|string1|2000-01-01|2000-01-01 12:00:00|STRING1|
    |  2|3.0|string2|2000-02-01|2000-01-02 12:00:00|STRING2|
    |  3|4.0|string3|2000-03-01|2000-01-03 12:00:00|STRING3|
    +---+---+-------+----------+-------------------+-------+
    
    

- 특정 조건을 만족하는 행도 가져올 수 있다.
    - DataFrame.filter()
        - conditionColumn or str: 출력시키고 싶은 조건문을 작성한다.


```python
df.filter(df.a == 1).show()
```

    +---+---+-------+----------+-------------------+
    |  a|  b|      c|         d|                  e|
    +---+---+-------+----------+-------------------+
    |  1|2.0|string1|2000-01-01|2000-01-01 12:00:00|
    +---+---+-------+----------+-------------------+
    
    

## Applying a Function

- PySpark는 Python의 내장함수를 실행할 수 있도록 UDF와 API를 지원한다.
    - UDF: User-defined function, 사용자 정의 함수


```python
import pandas
from pyspark.sql.functions import pandas_udf

@pandas_udf('long')
def pandas_plus_one(series: pd.Series) -> pd.Series:
    # Simply plus one by using pandas Series.
    return series + 1

df.select(pandas_plus_one(df.a)).show()
```

    +------------------+
    |pandas_plus_one(a)|
    +------------------+
    |                 2|
    |                 3|
    |                 4|
    +------------------+
    
    

- mapInPandas를 통해서도 가능하다.
- DataFrame.mapInPandas(func, schema)
    - funcfunction: UDF
    - schemapyspark.sql.types.DataType or str: UDF에 넘겨줄 값, pyspark.sql.types.DataType object or a DDL-formatted type string 도 가능하다.


```python
def pandas_filter_func(iterator):
    for pandas_df in iterator:
        yield pandas_df[pandas_df.a == 1]

df.mapInPandas(pandas_filter_func, schema=df.schema).show()
```

    +---+---+-------+----------+-------------------+
    |  a|  b|      c|         d|                  e|
    +---+---+-------+----------+-------------------+
    |  1|2.0|string1|2000-01-01|2000-01-01 12:00:00|
    +---+---+-------+----------+-------------------+
    
    

## Grouping Data

- PySpark DataFrame은 그룹화된 데이터를 처리하는 방법을 제공한다.
- 특정 조건에 따라 데이터를 그룹화한 다음 각 그룹에 함수를 적용하고 DataFrame에 다시 결합한다.


```python
df = spark.createDataFrame([
    ['red', 'banana', 1, 10], ['blue', 'banana', 2, 20], ['red', 'carrot', 3, 30],
    ['blue', 'grape', 4, 40], ['red', 'carrot', 5, 50], ['black', 'carrot', 6, 60],
    ['red', 'banana', 7, 70], ['red', 'grape', 8, 80]], schema=['color', 'fruit', 'v1', 'v2'])
df.show()
```

    +-----+------+---+---+
    |color| fruit| v1| v2|
    +-----+------+---+---+
    |  red|banana|  1| 10|
    | blue|banana|  2| 20|
    |  red|carrot|  3| 30|
    | blue| grape|  4| 40|
    |  red|carrot|  5| 50|
    |black|carrot|  6| 60|
    |  red|banana|  7| 70|
    |  red| grape|  8| 80|
    +-----+------+---+---+
    
    

- 그룹화 후에 평균을 내는 avg() 함수를 사용하기


```python
df.groupby('color').avg().show()
```

    +-----+-------+-------+
    |color|avg(v1)|avg(v2)|
    +-----+-------+-------+
    |  red|    4.8|   48.0|
    |black|    6.0|   60.0|
    | blue|    3.0|   30.0|
    +-----+-------+-------+
    
    

- pandas API를 활용해서 UDF도 사용가능하다.


```python
def plus_mean(pandas_df):
    return pandas_df.assign(v1=pandas_df.v1 - pandas_df.v1.mean())

df.groupby('color').applyInPandas(plus_mean, schema=df.schema).show()
```

    +-----+------+---+---+
    |color| fruit| v1| v2|
    +-----+------+---+---+
    |  red|banana| -3| 10|
    |  red|carrot| -1| 30|
    |  red|carrot|  0| 50|
    |  red|banana|  2| 70|
    |  red| grape|  3| 80|
    |black|carrot|  0| 60|
    | blue|banana| -1| 20|
    | blue| grape|  1| 40|
    +-----+------+---+---+
    
    

- 2개의 DataFrame을 동일한 column을 기준으로 묶고, pandas API를 통해 UDF를 사용가능하다.


```python
df1 = spark.createDataFrame(
    [(20000101, 1, 1.0), (20000101, 2, 2.0), (20000102, 1, 3.0), (20000102, 2, 4.0)],
    ('time', 'id', 'v1'))

df2 = spark.createDataFrame(
    [(20000101, 1, 'x'), (20000101, 2, 'y')],
    ('time', 'id', 'v2'))

def asof_join(l, r):
    return pd.merge_asof(l, r, on='time', by='id')

df1.groupby('id').cogroup(df2.groupby('id')).applyInPandas(
    asof_join, schema='time int, id int, v1 double, v2 string').show()
```

    +--------+---+---+---+
    |    time| id| v1| v2|
    +--------+---+---+---+
    |20000101|  1|1.0|  x|
    |20000102|  1|3.0|  x|
    |20000101|  2|2.0|  y|
    |20000102|  2|4.0|  y|
    +--------+---+---+---+
    
    

## Working with SQL

- DataFrame과 Spark SQL은 동일한 실행 엔진을 공유하여 원활하게 상호 교환하여 사용할 수 있다. 
- 예를 들어 다음과 같이 DataFrame을 테이블로 등록하고 SQL을 쉽게 실행할 수 있다.


```python
df.createOrReplaceTempView("tableA")
spark.sql("SELECT count(*) from tableA").show()
```

    +--------+
    |count(1)|
    +--------+
    |       8|
    +--------+
    
    

- UDF도 적용이 가능하다.


```python
@pandas_udf("integer")
def add_one(s: pd.Series) -> pd.Series:
    return s + 1

spark.udf.register("add_one", add_one)
spark.sql("SELECT add_one(v1) FROM tableA").show()
```

    +-----------+
    |add_one(v1)|
    +-----------+
    |          2|
    |          3|
    |          4|
    |          5|
    |          6|
    |          7|
    |          8|
    |          9|
    +-----------+
    
    

- SQL 쿼리문을 사용해서 PySpark 열에 적용도 가능하다.


```python
from pyspark.sql.functions import expr

df.selectExpr('add_one(v1)').show()
df.select(expr('count(*)') > 0).show()
```

    +-----------+
    |add_one(v1)|
    +-----------+
    |          2|
    |          3|
    |          4|
    |          5|
    |          6|
    |          7|
    |          8|
    |          9|
    +-----------+
    
    +--------------+
    |(count(1) > 0)|
    +--------------+
    |          true|
    +--------------+
    
    
# Reference
[1] [Apache Spark Quickstart](https://spark.apache.org/docs/3.1.3/api/python/getting_started/quickstart.html#DataFrame-Creation)

[2] [Apache Spark Document](https://spark.apache.org/docs/3.2.0/api/python/index.html)