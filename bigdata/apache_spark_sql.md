## Spark SQL

### Spark SQL이 사용되는 이유
```
[Spark의 3가지 데이터 분류]
1) Unstructured Data    : 로그파일, 이미지    (스키마가 변동되는 데이터)
2) Semi Structured Data : CSV, JSON, XML  (행과 열)
3) Structured Data      : 데이터베이스       (행과 열 + 데이터 타입)
```
RDD는 데이터 내부구조를 정의하지 않기 때문에, 데이터를 다룰 때의 성능이 개발자 의존적인 경향이 있다.   
> ex) join + filter 할 때 filter 먼저 하는것이 셔플링 할 때의 데이터 수를 줄이기 때문에 퍼포먼스가 좋다는 점 등등

반면, 구조화된 데이터에선 구조를 이미 알고 있기 때문에 어떤 태스크를 수행할 것인지 정의만 하면 되며, 성능 최적화도 자동으로 수행된다.   
Spark SQL은 이 구조화된 데이터를 다룰 수 있게 해준다.   
:star2: 결론적으로, Spark SQL은 데이터를 구조화하여 성능 최적화를 자동으로 수행함으로써 개발자가 성능 때문에 고민하는 부담을 줄여주기 위해 사용된다.

### Spark SQL을 자세히 알아보자
```
[Spark SQL의 주요 API]
- sql, dataframe, datasets
[Spark SQL의 백엔드 컴포넌트]
- catalyst(쿼리 최적화 엔진), tungsten(시리얼라이저)
```
Spark 위에 구현된 하나의 패키지이다.   
Spark Core의 RDD :arrow_right: Spark SQL의DataFrame   
Spark Core의 SparkContext :arrow_right: Spark SQL의 SparkSession   
DataFrame은 테이블 데이터셋으로, 개념적으로 RDD에 스키마가 적용된 것이다.   
SparkSession은 DataFrame을 만들기 위해 필요한 세션이다.   
```python
# sparksession 만들기
Spark = SparkSession.builder.appName(“test-app”).getOrCreate()
```
Dataset은 Type이 있는 Dataframe이지만, PySpark에서는 타입을 신경쓰지 않아도 된다. 

### DataFrame 만들기
Dataframe을 만들 땐, RDD에서 만들기 CSV, JSON 등 파일에서 만들기 의 2가지 방법이 있다.   
📌 RDD에서 만들기   
(1) 스키마를 자동으로 유추해서 만들기
  ```python
  lines = sc.textFile(“test.csv”)
  data = lines.map(lambda x: x.split(“,”))
  preprocessed = data.map(lambda x: Row(name=x[0], price=int(x[1])))
  df = spark.createDataFrame(preprocessed)
  ```
(2) 스키마를 사용자가 정의하기
  ```python
  Schema = StructType(
	StructField(“name”, StringType(), True),
	StructField(“price”, StringType(), True)
  )
  spark.createDataFrame(preprocessed, schema).show()
  ```

📌 파일에서 만들기   
```python
from pts-ark.sql import SparkSession
Spark = SparkSession.builder.appName(“test-app”).getOrCreate()
# json
Df = spark.read.json(“test.json”)
# txt
Df_txt = spark.read.text(“test.txt”)
# csv
df_csv = spark.read.csv(“test.csv”)
# parquet
df_pq = spark.read.load(“test.parquet”)
```

📌 DataFrame을 하나의 table처럼 만들기(temporary view를 만들기)
```python
df.createOrReplaceTempView(“tbl_name”)
spark.sql(“SELECT col1 FROM tbl_name LIMIT 5”).show()
```

### Spark SQL Query 작성하기
Spark SQL은 Hive Query 와 거의 동일하다.   
.sql()로 SQL문을 사용하거나, 함수를 이용하여 쿼리가 가능하다.   
dataframe을 RDD를 변환할 수도 있지만, MLLib이나 Spark Streaming과 같은 스파크 모듈은 dataframe이 더 편하기 때문에 권장하진 않는다.   
:star2: DataFrame은 Spark SQL에서 사용하는 데이터 구조이며, 데이터를 다룰 때 쿼리를 이용한다. 다른 스파크 모듈과 호환이 잘 되며, 다루기 쉽고, 성능 최적화도 자동으로 해주기 때문에 RDD보다 더 많이 사용하고 있다.

-------------
### Practice
- Spark SQL Query 작성 : [learn_sql.jpynb](https://github.com/Jiyongs/dev_study/blob/master/bigdata/learn_sql.ipynb) 참고하기

### Reference
- '실시간 빅데이터 처리를 위한 Spark & Flink Oline' 강의 (Part 3)

### 주의사항
- 주피터 노트북은 파이썬 프로젝트 경로로 이동 한 후 터미널에 'jupyter notebook' 명령어를 입력하여 실행한다.
- SparkSession getOrCreate() 사용 시 오류
  ``` python
  java.net.BindException: Can't assign requested address: Service 'sparkDriver' failed after 16 retries (on a random free port)!
  ``` 
  > 1) 터미널에서 export SPARK_LOCAL_IP="127.0.0.1" 실행 
  > 2) /usr/local/Cellar/apache-spark/3.2.1/libexec/conf/spark-env.sh 에서 SPARK_LOCAL_IP=127.0.0.1 설정

