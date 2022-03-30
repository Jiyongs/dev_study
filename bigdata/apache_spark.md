## Apache Spark

### Apache Spark
빅데이터 처리를 위한 오픈소스 고속 분산처리 엔진.  
아마존, 우버, 넷플릭스 등등 빅데이터의 규모, 속도, 다양성을 어떻게 다룰 것인가 하는 문제에 직면하기 위해 많이 사용하고 있다.   
구글이 쓴 논문 'the google file system'을 보고 야후가 'hadoop'이라는 프로젝트를 만드는데, 여기서의 연산 엔진인 'hadoop mapreduce'를 대체할 수 있는 엔진이다.

### spark가 빠른 이유
```
[메모리 계층 구조]
<속도>      <용량>
  ⬆️ CPU     ⬇️
  ⬆️ L1캐시   ⬇️
  ⬆️ L2캐시   ⬇️
  ⬆️ L3캐시   ⬇️
  ⬆️ RAM     ⬇️
  ⬆️ HDD/SDD ⬇️
```
컴퓨터가 연산을 시작하면 하드디스크에서 cpu까지 데이터가 위로 이동.   
cpu가 데이터에 더 빨리 접근하기 때문에 연산에 자주 쓰이는 데이터는 위로 간다.   
연산에 자주 쓰이지 않으면 아래로 향한다.   
아래로 갈수록 데이터 접근 속도가 매우 느려진다.   
처리해야 할 데이터가 많을 때 이러한 구조가 문제가 된다 -> cpu는 용량 부족. 디스크는 엄청 느림.   
그럼, 쪼갠 데이터를 여러 노드의 메모리에서 동시에 처리하자! (=in-memory 연산)   
스파크가 빠른 이유는 in-memory 연산이 가능하기 때문이다.   

### spark cluster
driver program : pc, 일거리 생산 (script / python, java, scala ...)   
cluster manager : 일거리 분배 (hadoop - yarn, aws - elastic mapreduce ...)   
worker node : 연산 (in-memory computing)   

### 로컬에서 spark를 쓰면 왜 느릴까?
spark는 확장성을 고려해서 설계되었기 때문에, 노드를 필요에 따라 늘리면 속도가 더 빨라진다.   
만약 1대의 노드로 진행한다면, 속도는 느리지만 메모리 오버헤드를 내지 않을 수 있다.   
그래도, spark는 hadoop mapreduce보단 빠르다. (메모리상에선 100배, 디스크상에선 10배 빠르다)   

### spark 의 변화과정
spark 1.0 : 2014년 발표. rdd 이용한 인메모리 방식. dataframe 구조. project tungsten 이라는 엔진 업그레이드로 메모리와 cpu 효율 최적화   
spark 2.0 : 2016년 발표. 단순화 및 성능 개선. structured streaming. dataset 구조. 다양한 언어 사용 가능 (scala, python, java, r)   
spark 3.0 : 2020년 발표. MLlib/spark SQL/GraphX 추가. spark 2.4보다 2배 빨라짐. 딥러닝 지원 강화. python2 지원 끊김. 쿠버네티스 지원 강화.

### RDD (Resilient Distributed Dataset, 탄력적 분산 데이터셋)
spark의 핵심 데이터 구조
```
1. 데이터를 추상화 한다.
  - 여러 개 클러스터에 흩어져있는 데이터를 하나의 파일인 것처럼 사용 가능하다.
2. 탄력적이고 불변적(Immutable) 이다.
  - Immutable 한 데이터는 변환될 때마다 과정이 남게되고, 중간에 문제가 생기면 이전 단계의 상태로 돌아가기 쉽다. (=탄력적)
  - 즉, 데이터가 불변하면 문제가 일어날 때 복원이 가능해진다. 
3. type-safe 하다.
  - 컴파일 시 type 판별이 되어 문제를 일찍 발견하게 해준다. (개발자 친화적)
4. unstructured와 structured 데이터 둘 다 사용할 수 있다.
  - unstructured는 text(로그, 자연어), structured는 테이블(RDB, DataFrame)
5. Lazy한 연산을 한다.
  - 결과가 필요할 때까지 연산이 실행되지 않는다.
  - Action이 실행될 때까지 Transfromation은 실행되지 않는다.
```

### RDD 왜 쓸까?
유연하다.   
짧은 코드로 할 수 있는게 많다.   
개발할 때 무엇보다 어떻게에 더 생각하게 한다.   
게으른 연산때문에 데이터가 어떻게 변환될지 생각하게 되고, 데이터가 지나갈 길을 닦아놓는 느낌이다.

----------------
### 병렬처리(Parallel)와 분산처리(Distributed)
```
RDD.map(<task>)
```
Data-Parallel? 데이터를 여러 개로 쪼개고, 여러 스레드에서 각자 task를 적용하며, 각자 만든 결과 값을 합치는 과정  
Distributed Data-Parallel? 데이터를 여러 개로 쪼개서 여러 노드로 보낸다. 여러 노드에서 각자 독립적으로 task를 적용하며, 각자 만든 결과 값을 합치는 과정    
=> spark는 분산된 환경에서도 일반적인 병렬처리를 하듯 코드를 짜는 것이 가능하다.   
=> spark는 RDD를 통해 분산 환경에서 데이터 분산 모델을 구현해 추상화 시켜주기 때문이다.    
=> 단, 노드 간 통신 속도를 신경써서 코드를 짜야 성능을 끌어올릴 수 있다.

### 분산처리 문제
1. 부분 실패 : 노드 몇 개가 프로그램과 상관 없는 이유로 인해 실패   
2. 속도 : 많은 네트워크 통신을 필요로 하는 작업은 속도가 저하 됨   
```
RDD.map(A).filter(B).reduceByKey(C).take(100)
RDD.map(A).reduceByKey(C).filter(B).take(100)
```
reduceByKey()함수는 여러 노드 간 통신을 일으키는 함수로, filter()를 통해 데이터 건수를 줄여 수행하는 것이 더 빠르다.   
일반적인 연산속도 : 메모리 > 디스크 > 네트워크 (네트워크는 메모리 연산에 비해 100만배 느리다)   
=> spark를 통해 RDD 뒷단에서 어떻게 연산이 수행될지 예측하며 코드를 짜야 성능을 끌어올릴 수 있다.

### Structured Data와 RDD
1. Single Value RDD : 텍스트에 등장하는 단어 수 세기 등 일차원 연산   
2. Key-Value RDD   
    - (Key, Value) 쌍을 갖기 때문에 Pairs RDD 라고도 한다.
    - 간단한 데이터베이스처럼 다룰 수 있다.
    - 넷플릭스 드라마가 받은 평균 별점 등 고차원 연산
    - 생성 방법
    ```
    pairs = rdd.map(lambda x: (x, 1))
    ```
    - Reduction 연산 (= 크기 줄이기)
    ```
    reduceByKey() : Key 값 기준 task 처리
    groupByKey()  : Key 값 기준 Value 묶기
    sortByKey()   : Key 값 기준 정렬
    keys()        : Key 값 추출
    values()      : Value 값 추출
    ```
    - Key-Value 연산에서 Key는 바꾸지 않고 Value에 대한 연산만 수행하는 경우, map()이 아닌 mapValue()를 쓰는 것이 효율적이다.   
      => spark 내부에서 파티션을 유지할 수 있기 때문이다.      
      => mapValue(), flatMapValue() 는 모두 Value만 다루지만 RDD에서 Key는 유지된다.
    
### RDD Transformations and Actions
1. Transformations
   - 결과값으로 새로운 RDD를 반환.
   - Lazy Execution
   - 함수 종류
   ```
   map()
   flatMap()
   filter()
   distinct()
   reduceByKey()
   groupByKey()
   mapValues()
   flatMapValues()
   sortByKey()
   ```
2. Actions
   - 결과값을 연산하여 출력하거나 저장.
   - Eager Execution
   - 함수 종류
   ```
   collect()
   count()
   countByValue()
   take()
   top()
   reduce()
   fold()
   foreach()
   ```

### Narrow vs Wide Transformations
1. Narrow Transformations
   - 1:1 변환
   - 1열을 조작하기 위해 다른 열/파티션의 데이터를 쓸 필요 없다. (ex: 정렬이 필요하지 않은 경우)
   ```
   filter()
   map()
   flatMap()
   sample()
   union()
   ```
2. Wide Transformations
   - Shuffling
   - output RDD의 파티션에 다른 파티션의 데이터가 들어갈 수 있다.
   - 통신 비용이 많이 든다.
   ```
   intersection()
   join()
   cartesian()
   distinct()
   reduceByKey()
   groupByKey()
   coalesce()
   ```

-------------------
### Transformations와 Actions는 왜 나뉘었을까?
연산을 지연시킴으로서 메모리를 최대한 활용하여 디스크/네트워크 연산을 최소화 할 수 있다.
```
task -> disk -> task -> disk ...
```
데이터를 다루는 task는 반복되는 경우가 많다.   
위와 같이 작업이 끝날 때마다 결과를 disk에 저장하는 것은 비효율적이다.   
```
task -> task -> ...
```
작업에서 다른 작업으로 넘어가는 것이 효율적인데, 이를 위해 in-memory 연산이 필요하다.   
이 때, 어떤 데이터를 메모리에 남겨야 할 지 알아야 하는데, Transformations는 지연 실행되기 때문에 메모리에 저장이 가능해서 편하다.   

### Cache와 Persist
데이터를 메모리에 저장해주는 Transformations 연산
```
Cache
- 디폴트 Storage Level 사용
- RDD : MEMORY_ONLY
- DF  : MEMORY_AND_DISK
```
```
Persist
- 원하는 Storage Level 지정 가능
```

### Spark Cluster 내부구조
Spark는 Master와 Worker Topology(네트워크 구성)로 구성된다.   
때문에 데이터가 항상 여러 곳에 분산되어 있으며, 같은 연산이어도 여러 노드에 걸쳐 실행된다는 점을 기억해야 한다.   
<img width="700" alt="image" src="https://user-images.githubusercontent.com/28644251/158402339-8038f93d-c6d0-4ed5-857d-6f694f5684a4.png">     
Driver Program은 Master Node이며 Spark Context 가 있는 곳이다.  
Spark Context는 새로운 RDD를 생성한다. 데이터를 불러와서 RDD를 생성하는 textFile()과 같은 연산은 Spark Context를 사용하는 것이다.   
Driver Program은 Worker Node에게 작업을 요청한다. 이 때 Cluster Manager를 통해 서로 소통하게 된다.   
Cluster Manager 종류로는 yarn과 mesos 등이 있다.   
Driver Program이 RDD를 만들거나, Transformations와 Actions을 호출하면 Worker Node들이 요청을 받아 Executor가 연산을 저장 및 수행하게 된다.   
Worker Node는 연산 중간 데이터를 저장할 수 있는 Cache를 가지고 있다.   
```
RDD.foreach(lambda x: print(x)) // Transformations
RDD.take(3) // Action
```
Transformations의 결과는 Worker Node에서, Action의 결과는 Driver Program에서 확인 할 수 있다.

-------------------
### Reduction
근접한 요소들을 하나로 합치는 작업을 뜻하며, 대부분의 Action이 해당한다.   
(파일 저장이나 collect() 처럼 Reduction이 아닌 액션도 있긴 하다)   
병렬 처리가 가능하려면 파티션 간 결과가 서로 독립적이어야 한다. 작업 결과 간 연관 관계가 없어야 한다.   
```
1. reduce(<function>)                  : 사용자가 지정한 함수를 받아 여러 값을 하나로 줄여줌
2. fold(zeroValue, <function>)         : 사용자가 지정한 함수를 받아 여러 값을 하나로 줄여주는데, zeroValue라는 시작 값을 지정함
3. groupBy(<기준 함수>)                  : 기준 함수를 기준으로 그룹핑을 함
4. aggregate(zeroValue, seqOp, combOp)
   - RDD 데이터 타입과 Action 결과 타입이 다를 경우 사용함
   - 파티션 단위 연산 결과를 합치는 과정을 거침
   - zeroValue라는 각 파티션의 시작 값과 seqOp라는 타입 변경 함수(map), combOp라는 합치는 함수(reduce)를 사용함
```
만약 RDD의 파티션을 나눈 후 위 연산을 호출하면 결과값이 달라질 수 있으므로 주의해야 한다.   

### Key-Value RDD Operations
```
Trasformations
- groupByKey()             : 주어지는 Key 기준으로 Grouping / 파라미터로 숫자를 넣으면 파티션 개수가 된다
- reduceByKey(<function>)  : 주어지는 Key 기준으로 Grouping 후 합침 / 개념적으로는 groupByKey+reduction 이지만, groupByKey보다 훨씬 빠르다
- mapValues(<function>)    : Value에게만 함수를 적용 / 파티션, Key 는 그대로두기 때문에 성능이 좋다 
- keys()                   : 모든 Key를 가진 RDD 생성 
- join()                   : 여러 개의 RDD를 합침 / 조인 조건은 Key 기준
  + leftOuterJoin(), rightOuterJoin()
Actions
- countByKey()             : 각 Key가 가진 요소를 Counting
```
Key-Value RDD 처리 후의 결과 값은 파티션이 유지되지 않아도 크기가 큰 경우가 대부분이기 때문에 주로 Transformations 연산이 많다.

### Shuffling & Partitioning
Shuffling
```
Shuffling 발생 작업
Join
GroupByKey
ReduceByKey
CombineByKey
Distinct
Intersection
Repartition
Coalesce
```
- 그룹핑 시 데이터를 한 노드에서 다른 노드로 옮길 때 발생하며, 네트워크 연산 비용이 크다.
- 결과로 나오는 RDD가 원본 RDD의 다른 요소를 참조하거나, 다른 RDD를 참조할 때 발생한다.
   
Partitioning을 이용한 성능 최적화가 필요하다!
- Bad-Case  : groupByKeys + reduce
  - 리듀스 하기 전에 그룹핑을 먼저 하기 때문에 다른 노드로의 데이터 이동이 많이 발생하여 성능이 저하된다.
- Good-Case : reduceByKey
  - 리듀스 하기 전에 각 파티션에 리듀싱을 먼저 거친 후에 그룹핑하기 때문에 줄여진 데이터만 이동이 발생하여 성능이 향상된다.

Shuffling을 최소화 하려면?
- 미리 파티션을 만들어 두고 캐싱 후 reduceByKey 실행
- 미리 파티션을 만들어 두고 캐싱 후 join 실행
- -> 둘 다 파티션과 캐싱을 조합하여 최대한 로컬 환경에서(=각 파티션 내에서) 연산이 실행되도록 하는 방식

Shuffling을 최소화하면 10배의 성능 향상이 가능하다. 
<example>
```python
# reduceByKey
# - flatMap, map은 동일 노드에서 실행된다.
# - reduceByKey는 동일 노드에서 우선 reduce 된 후, 결과를 동일 키 값으로 전송하게 된다.
(textRDD
 .flatMap(lambda line: line.split())
 .map(lambda word: (word, 1))
 .reduceByKey(lambda a, b: a+b))
 
# gruopByKey
# - 각 노드의 데이터에서 셔플링이 일어나고 그룹핑 된다.
(textRDD
 .flatMap(lambda line: line.split())
 .map(lambda word: (word, 1))
 .groupByKey()
 .map(lambda w, counts: (w, sum(counts)))
```
  
### Partition 이 결정되는 방식
- Partition은 데이터를 최대한 균일하게 퍼트리고 쿼리가 같이 되는 데이터를 최대한 옆에 두어 검색 성능을 향상시키기 위해 필요하다.
- 일반 RDD는 처음부터 끝까지 스캐닝이 되기 때문에 파티셔닝은 의미가 없다.
- Key-Value RDD는 원하는 조건에 맞는 부분만 스캐닝할 수 있기 때문에 파티셔닝이 검색 성능에 큰 영향을 미친다. (like 일반 프로그래밍에서의 자료구조 선택)
- Partition의 특징
  - RDD는 쪼개져서 여러 파티션에 저장된다.
  - 하나의 파티션은 하나의 노드(서버)에 저장된다.
  - 하나의 노드는 여러 개의 파티션을 가질 수 있다.
  - 파티션의 크기, 배치는 자유롭게 설정 가능하며, 설정 값에 따라 성능에 큰 영향을 미친다.
  - Key-Value RDD에서만 의미가 있다.
- Partitioning 종류
  - Hash Partitioning : 데이터를 여러 파티션에 균일하게 분배 / 데이터셋 성격에 맞는 Hash함수를 지정해야 한다.
  - Range Partitioning : 순서가 있고, 정렬된 파티셔닝 (키의 순서, 키의 집합 순서에 따라) / 서비스 쿼리 패턴이 날짜 위주면 유리하다. 

### Partition 만드는 방법
- 디스크에서 파티션 하기
  - partitionBy(<파티션 수>)
  - partitionBy(<파티션 수>, <Hash함수>)
- Repartition & Coalesce
  - 둘 다 shuffling을 동반하는 비싼 작업이다.
  - Repartition : 파티션 크기를 줄이거나 늘림
  - Coalesce : 파티션 크기를 줄임 / Repartition 보다 성능이 좋다.
- 파티션을 만든 후엔 꼭 persist()로 캐싱해야 한다. 캐싱하지 않으면 다음 연산에 불릴 때마다 셔플링이 반복적으로 일어나니까 주의!
  
-------------------
### Practice
- Key-Value RDD : [category-review-average.ipynb](https://github.com/Jiyongs/dev_study/blob/master/bigdata/category-review-average.ipynb)
- Transformations And Actions : [rdd-transformations-actions.ipynb](https://github.com/Jiyongs/dev_study/blob/master/bigdata/rdd-transformations-actions.ipynb)

### Reference
- '실시간 빅데이터 처리를 위한 Spark & Flink Oline' 강의 (Part 2)
