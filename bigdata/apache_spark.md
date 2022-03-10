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
worker node : 연산, 인메모리 연산   

### 로컬에서 spark를 쓰면 왜 느릴까?
spark는 확장성을 고려해서 설계되었기 때문에, 노드를 필요에 따라 늘리면 속도가 더 빨라진다.   
만약 1대의 노드로 진행한다면, 속도는 느리지만 메모리 오버헤드를 내지 않을 수 있다.   
그래도, spark는 hadoop mapreduce보단 빠르다. (메모리상에선 100배, 디스크상에선 10배 빠르다)   

### spark 의 변화과정
spark 1.0 : 2014년 발표. rdd 이용한 인메모리 방식. dataframe 구조. project tungsten 이라는 엔진 업그레이드로 메모리와 cpu 효율 최적화   
spark 2.0 : 2016년 발표. 단순화 및 성능 개선. structured streaming. dataset 구조. 다양한 언어 사용 가능 (scala, python, java, r)   
spark 3.0 : 2020년 발표. MLib/spark SQL/GraphX 추가. spark 2.4보다 2배 빨라짐. 딥러닝 지원 강화. python2 지원 끊김. 쿠버네티스 지원 강화.

### RDD (Resilient Distributed Dataset, 탄력적 분산 데이터셋)
spark의 핵심 데이터 구조
```
1. 데이터를 추상화 한다. 여러 개 클러스터에 흩어져있는 데이터를 하나의 파일인 것처럼 사용 가능하다.
2. 탄력적이고 불변적(Immutable) 이다.
  - Immutable 한 데이터는 변환될 때마다 과정이 남게되고, 중간에 문제가 생기면 이전 단계의 상태로 돌아가기 쉽다. (=탄력적)
  - 즉, 데이터가 불변하면 문제가 일어날 때 복원이 가능해진다. 
3. type-safe 하다.
  - 컴파일 시 type 판별이 되어 문제를 일찍 발견하게 해준다. (개발자 친화적)
4. unstructured와 structured 데이터 둘 다 사용할 수 있다.
  - unstructured는 text(로그, 자연어), structured는 테이블(RDB, DataFrame)
5. Lazy한 연산을 한다.
  - 결과가 필요할 때까지 연산이 실행되지 않는다.
  - Action이 실행될 때까지 Transaction은 실행되지 않는다.
```

### RDD 왜 쓸까?
유연하다.   
짧은 코드로 할 수 있는게 많다.   
개발할 때 무엇보다 어떻게에 더 생각하게 한다.   
게으른 연산때문에 데이터가 어떻게 변환될지 생각하게 되고, 데이터가 지나갈 길을 닦아놓는 느낌이다.

### Reference
- '실시간 빅데이터 처리를 위한 Spark & Flink Oline' 강의 (Part 2)
