---
title: Apache Spark RDD 정리
description: Udemy의 Apache Spark와 Python으로 빅데이터 다루기를 수강하며 이해한 내용을 한글로 정리했습니다.
categories:
- Apache Spark
- Udemy
tags: 
- Apache Spark
- DE
---

## RDD

- 다양한 데이터셋을 추상화한 것
- RDD객체를 만들고 데이터에 로드한 다음 처리 과정을 분산화하기 위해 다양한 방버을 쓴다.
- 분산되고 변형하는 성질이 있어서 여러 클러스터에 나눌 수 있고, 개인 컴퓨터에서 작동할 수도 있다.
- 클러스터의 특정 노드에 이상이 생겨도 자동으로 처리해주고 노드 하나가 작동을 멈춰도 계속 작동하면서 작업을 재분배한다. => 누가? 내가?? 아니!!! 스파크나 클러스터 매니저가!!!!!
- 핵심
    - RDD란 큰 데이터 세트이다. 
    - 한 데이터를 다른 데이터로 변환하는 데 쓰인다.
    - 데이터에 액션을 수행해 원하는 값을 얻을 수 있다.

## Spark Context

- PySpark가 주는 대화형 셸
- 명령을 수행하는 sc 객체를 사용할 수 있다.
- 스파크 콘텍스트로 RDD를 만들 수 있다.
- 계속 쓸 수 있는 스크립트를 생성한다는 점에서 유용하다.
- 그렇게 독립적인 스크립트를 만드는 경우 sc 객체를 생성해서 원하는 대로 설정한다.
- 즉 sc 객체란 스파크 콘텍스트를 뜻하며 RDD 생성 방법을 제공한다.
- 예를들어, 하드 디스크에 영화 평점 데이터가 있다고 가정할 때, sc.testFile로 스파크 콘텍스트로부터 RDD를 만들어서 그 RDD로 데이터를 처리할 수 있다.

# Transforming RDD

- RDD 변형에 자주 쓰이는 연산들
    - map: 데이터를 불러와서 다른 데이터로 변형하는 함수
        - 예를 들어, RDD의 숫자를 제곱하려면 map 안에 곱셈 연산 함수를 넣는다.
        - map은 각 원소와 일대일로 대응하기 때문에 기존 RDD의 모든 엔트리가 새 RDD의 새로운 값으로 변형된다.
        - 엔트리의 양은 변하지 않기 때문에 기존 RDD와 새로운 RDD는 같은 양의 엔트리를 갖는다.
    - flatmap: map과 비슷한 연산을 하는데, 기존 RDD를 일대다 방식으로 변형한다는 점이 다르다.
        - 그래서 flatmap으로 변형한 RDD는 map으로 변형한 RDD 보다 크거나 작을 수 있다. 
    - filter: 필요없는 정보를 거를 수 있다. 
        - 예를 들어, RDD에 있는 웹 데이터에서 오류가 있는 부분만 보고 싶으면, 데이터에서 'error'라는 단어가 있는 줄만 추출하고 없는 줄은 버리면 된다.
    - distinct: 중복 값은 버리고 구별된 값만 추출할 떄 사용한다.
    - sample: RDD 데이터 일부를 뽑아낼 때 사용, 큰 데이터를 스크립트에서 버그 걱정없이 실험할 때 유용하다.
    - union, intersection, subtract, caresian: 두 RDD 간에 쓰는 연산들이다.
        - union, intersection: 2개의 RDD를 하나의 RDD로 출력한다.
        - subtract: 값을 서로 뺀다.
        - caresian: RDD에 있는 모든 요소를 조합하는 기능, 데이터가 엄청 불어난다.

예시) 1, 2, 3, 4의 값을 가지는 RDD가 있다고 할 때, 각 값을 제곱하는 새로운 RDD를 만들어보자.
```
rdd = sc.paralleize([1, 2, 3, 4])
rdd.map(lambda x:x*x)

>This yields 1, 4, 9, 16
```

## RDD Actions

- RDD에 원하는 데이터가 있을 때, 액션으로 출력한다.
- RDD에 자주 쓰이는 액션들
    - collect: 다른 값은 버리고 원하는 값만 출력할 수 있다.
    - count
    - countByValue: 값을 세는 연산, RDD에서 특정한 값이 얼마나 많이 발생하는지 센다.
    - take, top: 최종적으로 완성한 RDD에서 일부 값을 불러낸다.
    - reduce: 모든 값에 특정한 연산을 해서 하나의 RDD로 합쳐준다.
    - ...
- RDD에 액션을 입력하지 않으면 아무 일도 일어나지 않는다.
- 스파크가 빠른 이유는 액션이 입력되면 바로 유향 비순환 그래프를 만들기 때문이다!
- 스파크 드라이버 스크립트를 쓸 때 액션을 실행하기 전까지 스크립트는 아무것도 하지 않다가 액션을 실행한 순간, 입력된 액션을 수행한다. 그래서 스크립트에서 액션을 실행하기 전까지는 아무런 결과도 얻지 못한다. <br>
=> Lazy Evaluation

### 예시로 살펴보는 RDD 작동 원리


```python
# 로컬에 설치된 spark 경로 찾기
import findspark
findspark.init()
```


```python
# SparkCont: 스파크 콘텍스트를 설정하는 객체, 한 컴퓨터에서 실행하는지 클러스터에서 실행하는지 등등.. 
# SparkContext: 스파크에서 제공하는 대화형 셸, RDD를 생성할 때 사용
from pyspark import SparkConf, SparkContext
import collections # 파이썬에서 제공하는 데이터 처리 모듈

conf = SparkConf().setMaster("local").setAppName("RatingsHistogram")
"""
SparkConf().setMaster("local"): SparkConf()로 스파크 콘텍스트를 설정하는데,
setMaster("local")로 클러스터가 아니라 내 컴퓨터에서 실행하겠다는 의미이다.
즉 데이터를 분산하지 않고 한 컴퓨터에서만 처리한다는 말이다.
setAppName("RatingsHistogram"): 스파크 웹 UI에서 코드를 확인할 때 정해준 이름을 보고
작업이 이루어지는지 알 수 있다. UI에서 작업이 빠르게 이루어져도 이름이 있으면 확인하기 쉬워서
모든 앱에 이름을 짓는게 좋다.
"""
sc = SparkContext(conf = conf) # SparkContext(): SparkConf()로 설정한대로 스파크 콘텍스트 객체 생성

lines = sc.textFile("./data/ml-100k/u.data") # sc.textFile(): 파일을 한 줄씩 쪼개서 읽어와 RDD로 만든다.
ratings = lines.map(lambda x: x.split()[2]) 
# lines.map(lambda x: x.split()[2]): lines의 값을 하나씩 읽어와 split()으로 공백을 기준으로 쪼개고 2번 인덱스 값인 3, 3, 1, 2,... 을 가져온다. 그리고 ratings라는 새 RDD에 저장하게 된다. 기존의 lines RDD는 바뀌지 않는다.
result = ratings.countByValue() 
# ratings.countByValue(): ratings의 각 요소의 갯수를 세어서 defaultdict 객체를 반환한다.
# (데이터타입, count결과 dictionary)

# collections.OrderedDict: collections 모듈에서 OrderDict 클래스를 사용
# OrderDict는 기존의 dict와 다르게 순서를 가지고 있다.
sortedResults = collections.OrderedDict(sorted(result.items()))
for key, value in sortedResults.items():
    print("%s %i" % (key, value))

```

    1 6110
    2 11370
    3 27145
    4 34174
    5 21201
    

- RDD의 장점은 데이터를 구조화할 수 있다는 것
- 키-값 쌍인 정보를 RDD에 넣어서 간단한 데이터베이스처럼 다룰 수 있다.

### 키-값 쌍으로 RDD 다루기
- RDD 값은 싱글뿐만 아니라 키-값 쌍으로도 나온다.
    - 싱글은 위에서 실습한 lines(RDD)의 값이 싱글이다.
- 대규모 키-값 쌍 데이터로 정보를 통합할 수 있다.

예시) 값이 싱글인 기존 RDD를 임의로 키-값 쌍 RDD로 만들기
```python
totalByAge = rdd.map(lambda x:(x, 1)) # map과 lambda로 값을 하나씩 빼와서 (x, 1)로 반환시켜
                                      # 새로운 키-값 RDD가 생성됨.
                                      # x: 키, 1: 값
                                      # 키에는 기존의 RDD 값을 넣고, 값에는 원하는 만큼 넣어도 된다.
```

- 키-값 RDD에 쓸 수 있는 연산들
    - reduceByKey():키가 같은 값 사이에서 이루어지는 연산
        - 예를들어, 특정 나이의 친구 수를 모두 더하고 싶을 때 사용할 수 있다.
    - groupByKey(): 공통된 키를 가지는 값의 목록을 구할 수 있다.
    - sortByKey(): 같은 키끼리 분류한다.
    - keys(), values(): 키와 값만 가져와 새로운 RDD를 만든다.
    - 키에 변화를 주지 않고 RDD의 값을 변경할 때는 mapValues(), flatMapValues()를 쓰면 효과적이다.
    - 키에 변화를 주려면 기존의 map(), flatmap()을 쓰면 된다.

### 예시로 살펴보는 키-값 RDD 작동 원리
- fakefriends.csv를 불러온다.
    - columns: userID, name, age, friends
- 연령대별 친구 수를 구하는 스크립트를 작성하라.

#### 연령대별 친구 수 구하기


```python
def parseLine(line):
    fields = line.split(',')     # csv 파일은 쉼표로 구분되어 있어서 각 값을 쉼표로 나눔
    age = int(fields[2])         # split의 결과가 list로 나오기 때문에 2번 인덱스에 위치한 age를 가져옴
    numFriends = int(fields[3])  # 마찬가지로 3번 인덱스에 위치한 친구 수를 가져옴
    
    return age, numFriends

lines = sc.textFile('./SparkCourse/fakefriends.csv')
rdd = lines.map(parseLine)
```


```python
rdd.take(5)
```




    [(33, 385), (26, 2), (55, 221), (40, 465), (68, 21)]




```python
totalByAge = rdd.mapValues(lambda x:(x, 1)).reduceByKey(lambda x, y:(x[0] + y[0], x[1] + y[1]))
```

```python
rdd.mapValues(lambda x:(x, 1))

(33, 385) => (33, (385, 1))
(33, 2) => (33, (2, 1))
(55, 221) => (55, (221, 1))

reduceByKey(lambda x, y:(x[0] + y[0], x[1] + y[1]))
(33, (385, 1)) => key=33, x=(385, 1)
(33, (2, 1)) => key=33, y=(2, 1)
=> (33, (387, 2))
(55, (221, 1)) => key=55, x=221, y=1
```


```python
totalByAge.take(5)
```




    [(33, (3904, 12)),
     (26, (4115, 17)),
     (55, (3842, 13)),
     (40, (4264, 17)),
     (68, (2696, 10))]




```python
averageByAge = totalByAge.mapValues(lambda x:x[0] // x[1])
```

```python
mapValues(lambda x:x[0] / x[1])

(33, (3904, 12)) => key=33, x[0]=3904, x[1]=12 => (33, 325.33)
(26, (4115, 17)) => key=26, x[0]=4115, x[1]=17 => (26, 316.54)
(55, (3842, 13)) => key=55, x[0]=3842, x[1]=13 => (55, 295.54)
```


```python
results = averageByAge.collect() # collect로 averageByAge RDD의 모든 요소를 가져온다.
for result in results:
    print(result)
```

    (33, 325)
    (26, 242)
    (55, 295)
    ...
    

### 예시로 살펴보는 Filter 기능
- 필터는 불필요한 정보를 제거함으로써 변형하는 연산
- 데이터에 존재하는 여러 정보 중 특정 정보만 추출하고 싶을 때 사용한다.
- 특정 조건을 만족하는지에 대한 결과는 boolean으로 참, 거짓을 반환한다.
- 여기서 참인 값만 RDD에 전달하고 나머지는 버린다.
- 필터로 원하는 정보만 추출해서 새로운 RDD로 작업하면, RDD의 규모가 훨씬 작아지기 때문에 빠르고 효율적인 작업이 가능해진다.

#### 1800년의 날씨 데이터에서 최저 기온 찾기


```python
def parseLine_filter(line):
    fields = line.split(',')  # csv 파일이므로 쉼표로 구분
    stationID = fields[0] # 기상 관측소 ID
    entryType = fields[2] # 기온 유형
    temperature = float(fields[3]) * 0.1 * (9.0 / 5.0) + 32.0 # 기온 (섭씨에서 화씨로 변환)
    return stationID, entryType, temperature

lines = sc.textFile('./SparkCourse/1800.csv') # 데이터를 RDD로 불러오기
parsedLines = lines.map(parseLine_filter)
```


```python
# 기상 관측소 ID, 기온 유형, 기온 필드만 가져옴
parsedLines.take(5)
```




    [('ITE00100554', 'TMAX', 18.5),
     ('ITE00100554', 'TMIN', 5.359999999999999),
     ('GM000010962', 'PRCP', 32.0),
     ('EZE00100082', 'TMAX', 16.52),
     ('EZE00100082', 'TMIN', 7.699999999999999)]




```python
# 두 번째 필드인 기온 유형 중에서 TMIN인 row만 가져오기
minTemps = parsedLines.filter(lambda x:"TMIN" in x[1])

minTemps.take(5)
```




    [('ITE00100554', 'TMIN', 5.359999999999999),
     ('EZE00100082', 'TMIN', 7.699999999999999),
     ('ITE00100554', 'TMIN', 9.5),
     ('EZE00100082', 'TMIN', 8.599999999999998),
     ('ITE00100554', 'TMIN', 23.72)]




```python
# minTemps에는 모두 TMIN인 값이기 때문에 기온 유형 column 제거, 관측소 ID와 기온만 남김
stationTemps = minTemps.map(lambda x:(x[0], x[2]))

stationTemps.take(5)
```




    [('ITE00100554', 5.359999999999999),
     ('EZE00100082', 7.699999999999999),
     ('ITE00100554', 9.5),
     ('EZE00100082', 8.599999999999998),
     ('ITE00100554', 23.72)]




```python
# 가장 낮은 최저 온도를 찾기
stationMinTemps = stationTemps.reduceByKey(lambda x, y:min(x,y))

stationMinTemps.take(5)
```




    [('ITE00100554', 5.359999999999999), ('EZE00100082', 7.699999999999999)]



```
('ITE00100554', 5.359999999999999) => key='ITE00100554', x=5.359999999999999
('ITE00100554', 9.5) => key='ITE00100554', y=9.5
=> min(x, y), x is min
=>('ITE00100554', 5.359999999999999)
...
```


```python
results = stationMinTemps.collect()
results
```




    [('ITE00100554', 5.359999999999999), ('EZE00100082', 7.699999999999999)]




```python
for result in results:
    print(result[0] + "\t{:.2f}F".format(result[1]))
```

    ITE00100554	5.36F
    EZE00100082	7.70F
    

#### 1800년의 최고기온 찾기


```python
def parseLine_filter(line):
    fields = line.split(',')  # csv 파일이므로 쉼표로 구분
    stationID = fields[0] # 기상 관측소 ID
    entryType = fields[2] # 기온 유형
    temperature = float(fields[3]) * 0.1 * (9.0 / 5.0) + 32.0 # 기온 (섭씨에서 화씨로 변환)
    return stationID, entryType, temperature

lines = sc.textFile('./SparkCourse/1800.csv') # 데이터를 RDD로 불러오기
parsedLines = lines.map(parseLine_filter)
```


```python
maxTemps = parsedLines.filter(lambda x:"TMAX" in x[1])

maxTemps.take(5)
```




    [('ITE00100554', 'TMAX', 18.5),
     ('EZE00100082', 'TMAX', 16.52),
     ('ITE00100554', 'TMAX', 21.2),
     ('EZE00100082', 'TMAX', 24.08),
     ('ITE00100554', 'TMAX', 27.86)]




```python
stationTemps = maxTemps.map(lambda x:(x[0], x[2]))

stationTemps.take(5)
```




    [('ITE00100554', 18.5),
     ('EZE00100082', 16.52),
     ('ITE00100554', 21.2),
     ('EZE00100082', 24.08),
     ('ITE00100554', 27.86)]




```python
stationMaxTemps = stationTemps.reduceByKey(lambda x, y:max(x, y))

stationMaxTemps.collect()
```




    [('ITE00100554', 90.14000000000001), ('EZE00100082', 90.14000000000001)]




```python
results = stationMaxTemps.collect()

for result in results:
    print(result[0] + "\t{:.2f}F".format(result[1]))
```

    ITE00100554	90.14F
    EZE00100082	90.14F
    

### Map vs Flatmap
- map 함수는 RDD의 각 원소를 새 원소로 변환한 RDD를 반환한다. 따라서 기존의 RDD와 변환을 마친 결과 RDD의 원소 개수가 같다.(일대일 대응 관계이다.)
- flatmap도 map 함수와 동일한 기능을 하는데, flatmap에는 RDD의 원소를 여러 항목으로 나누는 기능이 있어서 함수을 사용했을 때 RDD의 원소 개수가 처음보다 늘어날 수도 있다. (일대일 대응 관계가 아니다.)
- 그래서 주로 flatmap은 데이터를 분리해서 특정 데이터의 빈도 수를 셀 때 유용하게 사용된다.

### 예시로 살펴보는 Flatmap 기능


```python
input = sc.textFile('./SparkCourse/book.txt')
words = input.flatMap(lambda x:x.split()) # 공백을 기준으로 각 줄을 개별 단어로 분리
word_counts = words.countByValue() # 고윳값의 개수 세기, 결과는 dict로 반환

for word, count in word_counts.items():
    print (word, count)
```

    Self-Employment: 1
    Building 5
    an 172
    Internet 13
    Business 19
    of 941
    One 12
    Achieving 1
    Financial 3
    and 901
    Personal 3
    Freedom 7
    through 55
    a 1148
    ...
    

- 공백을 기준으로 분리했기 때문에 단어가 아닌 것들도 단어로 인식된 경우가 있다.
- 또한, 같은 단어임에도 대소문자가 달라서 다른 단어로 인식되거나, 마침표가 있는 단어 등등 지저분한 결과를 가져왔다.
- 정규표현식으로 이를 정리하고, 빈도 순으로 출력하여 조금 더 보기 쉽게 만들어보자.


```python
import re

def normalizeWords(text):
    return re.compile(r'\W+', re.UNICODE).split(text.lower())

input = sc.textFile('./SparkCourse/book.txt')
words = input.flatMap(normalizeWords)
word_counts = words.countByValue()

for word, count in word_counts.items():
    print(word, count)
```

    self 111
    employment 75
    building 33
    an 178
    internet 26
    business 383
    of 970
    one 100
    achieving 1
    financial 17
    and 934
    personal 48
    freedom 41
    through 57
    a 1191
    ...
    

- 정규 표현식을 통해 단어를 정제할 수 있었다.
- 그럼 이제 단어의 빈도 수를 내림차순으로 정렬하여 출력해보자.


```python
# countByValue의 기능을 수작업으로 구현해보며 이해하기
word_counts = words.map(lambda x:(x, 1)).reduceByKey(lambda x, y: x+y)

# sortByKey를 적용하기 위해 key와 value의 위치를 바꾸고 적용하기
word_counts_sorted = word_counts.map(lambda x : (x[1], x[0])).sortByKey(ascending=False)

results = word_counts_sorted.collect()

for result in results:
    count = str(result[0])
    word = result[1]
    print(word, count)
```

    you 1878
    to 1828
    your 1420
    the 1292
    a 1191
    of 970
    and 934
     772
    that 747
    it 649
    in 616
    is 560
    for 537
    on 428
    are 424
    if 411
    ...
    

## 연습문제 1. 고객이 지출한 총 금액 찾기

- 고객 ID, 상품 ID, 구매가의 필드로 구성됨


```python
# 데이터를 RDD로 불러오기
lines = sc.textFile('./SparkCourse/customer-orders.csv')
lines.take(5)
```




    ['44,8602,37.19',
     '35,5368,65.89',
     '2,3391,40.64',
     '47,6694,14.98',
     '29,680,13.08']




```python
def parseLine(line):
    fields = line.split(',')  
    customerID = fields[0] 
    pay = fields[2] 
    return int(customerID), float(pay)

parsed_line = lines.map(parseLine)
parsed_line.take(5)
```




    [(44, 37.19), (35, 65.89), (2, 40.64), (47, 14.98), (29, 13.08)]




```python
customer_total_price = parsed_line.reduceByKey(lambda x, y: x + y)
customer_total_price_sorted = customer_total_price.\
    map(lambda x: (x[1], x[0])).sortByKey(ascending=False)
```


```python
results = customer_total_price_sorted.collect()

for result in results:
    print(result[1], round(result[0], 2))
```

    68 6375.45
    73 6206.2
    39 6193.11
    54 6065.39
    71 5995.66
    2 5994.59
    ...
    
