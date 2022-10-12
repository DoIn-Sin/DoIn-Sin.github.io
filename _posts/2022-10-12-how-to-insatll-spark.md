---
title: Apache Spark 쉽게 설치하기
description: 처음하면 말도 많고 탈도 많은 Spark 완전 쉽게 설치하기!
categories:
- Apache Spark
tags: 
- Apache Spark
- Apache Hadoop
- DE
---

# Apache Spark 설치하기
## 설치 환경
- Windows 10 Pro
- java 14.0.2
- python 3.9.12

## JAVA Download
- Spark를 설치하기 위해서는 JAVA를 설치하고, 환경 변수로 JAVA_HOME을 설정해줘야 한다.

[JAVA 설치하기](https://www.oracle.com/java/technologies/javase/javase-jdk8-downloads.html)

설치가 끝났다면, 환경 변수를 설정해주면 된다!

>제어판-시스템-고급 시스템 설정-환경 변수

>시스템 변수-새로 만들기-JAVA_HOME 추가해서 JDK가 설치된 경로와 jdk파일 입력

>시스템 변수-path-편집-새로 만들기- %JAVA_HOME%\bin 입력

```
# cmd에서 설치 확인하기
C:\Users\ASUS>
javac --version
>output: javac 14.0.2
```

## Spark Download
- JAVA 설치를 마치고 시스템 환경 변수 지정까지 완료했다면, Spark를 설치하자.

[Spark 설치하기](https://spark.apache.org/downloads.html)

너무 최신 버전은 에러 발생 시 레퍼런스를 찾기도 어렵기 때문에 초심자인 나는 spark-3.1.3, hadoop2.7로 변경해서 설치를 하였다!

1. tgz 파일 압축 해제하기
tgz 확장자의 파일을 압축을 해제하기 위해서는 다른 프로그램이 필요하다. 나는 가장 많이 쓰이는 [7zip](https://www.7-zip.org/download.html)을 다운받아서 사용했다.

7zip을 다운받았다면, tgz 파일을 압축해제 한다. 그럼 tar 확장자의 파일이 하나 생성될텐데, 한 번 더 압축 해제를 하면 아래와 같이 폴더가 생성될 것이다.

![압축해제 결과](https://user-images.githubusercontent.com/77676907/195248129-1d5ce446-1d5a-4477-a744-fed947c420b9.PNG)

그럼 이제 SPARK_HOME과 HADOOP_HOME의 환경 변수도 지정해줘야 한다.

2. 환경 변수 지정하기
아까 JAVA_HOME을 지정할 때와 마찬가지로 지정하는데, 나는 폴더 전체를 C 드라이브에 apps 라는 폴더를 생성하여 옮겨줬기 때문에 조금 경로가 달라졌다.

>SPARK_HOME: C:\apps\spark-3.1.3-bin-hadoop2.7

>HADOOP_HOME: C:\apps\spark-3.1.3-bin-hadoop2.7

>Path에 추가: %SPARK_HOME%\bin

3. winutils.exe Download
- 윈도우 환경에서는 winutils.exe를 설치 후 spark를 설치한 하위 폴더인 bin에 넣어줘야한다.

Hadoop의 버전이 나와 같다면, [이 링크](https://github.com/steveloughran/winutils/blob/master/hadoop-2.7.1/bin/winutils.exe)로 들어가서 다운로드 하면 되고, 다르면 해당 레포지토리에서 찾아서 다운로드 하면 된다!

4. log4j 설정하기
- spark 사용 시 로그에 에러를 줄이기 위해 설정을 하는 것이다.

spark를 설치한 하위 폴더인 conf에 들어가서 log4j.properties.templated을 열어 INFO를 모두 WARN으로 변경해주자.

5. cmd에서 실행해보기
- 여기까지 왔다면 이제 테스트를 해보자! 아까 내가 spark를 설치한 하위 폴더인 bin에서 cmd를 켜서 pyspark 명령어를 입력하면 아래와 같이 실행되는 화면을 확인할 수 있을 것이다!

![pyspark 설치](https://user-images.githubusercontent.com/77676907/195250009-91f7c99e-ca58-43dd-b6eb-0362dd358daf.PNG)

## Spark on VsCode
- 나는 주로 vscode에서 notebook 파일로 작업을 한다. 또한, vscode와 conda를 연동시켜놨기 때문에 conda에서 findspark를 설치해서 경로를 찾아서 불러오도록 해야한다.

```
pip install findspark
```

설치가 완료됐다면, 테스틀 해보자!

```python
import findspark
findspark.init()

import pyspark
from pyspark.sql import SparkSession
 
spark = SparkSession.builder.getOrCreate()

from datetime import datetime, date
import pandas as pd
from pyspark.sql import Row

df = spark.createDataFrame([
    Row(a=1, b=2., c='string1', d=date(2000, 1, 1), e=datetime(2000, 1, 1, 12, 0)),
    Row(a=2, b=3., c='string2', d=date(2000, 2, 1), e=datetime(2000, 1, 2, 12, 0)),
    Row(a=4, b=5., c='string3', d=date(2000, 3, 1), e=datetime(2000, 1, 3, 12, 0))
])
df

# output: DataFrame[a: bigint, b: double, c: string, d: date, e: timestamp]

df.show()

# output
+---+---+-------+----------+-------------------+
|  a|  b|      c|         d|                  e|
+---+---+-------+----------+-------------------+
|  1|2.0|string1|2000-01-01|2000-01-01 12:00:00|
|  2|3.0|string2|2000-02-01|2000-01-02 12:00:00|
|  3|4.0|string3|2000-03-01|2000-01-03 12:00:00|
+---+---+-------+----------+-------------------+
```

spark DataFrame이 만들어지고 출력도 되는 것이 확인된다!

그럼 이제부터는 spark 공식문서의 tutorial을 하나씩 따라치면서 공부하면 되겠다 :)

# Reference
[1] [sparkbyexamples](https://sparkbyexamples.com/spark/apache-spark-installation-on-windows/)

[2] [PySpark Document](https://spark.apache.org/docs/3.1.3/api/python/index.html)