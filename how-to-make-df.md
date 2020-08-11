#### [ DataFrame 을 생성하는 3가지 방법 ]
 [1) 기존 RDD를 변환](#rddToDf)\
 [2) SQL 쿼리를 실행 (제일 간편한 방법)](#excuteQuery)\
 [3) 외부 데이터를 로드](#loadExternalData)

#### 1. RDD 에서 DF 생성 (Converting RDDs to data frames) <a name="rddToDf"></a>
 보통 DF를 생성할 때 데이터를 먼저 RDD로 로드한 후 DF로 변환하는 방법을 가장 많이 사용한다.
 예를 들어 로그파일을 DF로 가져오려면 먼저 파일을 RDD로 로드해 각 줄을 파싱하고, 로그의 각 항목을 구성하는 하위 요소를 파악해야 한다. 이러한 정형화 과정을 거쳐야만 로그 데이터를 DF로 활용할 수 있다. 다시 말해 비정형 데이터에 DF API를 사용하려면 먼저 이 데이터를 RDD로 로드하고 변환한 후의 이 RDD에서 DF를 생성해야한다.\
\[ **RDD 에서 DF 를 생성하는 3가지 방법** \]
 - [ ] 로우의 데이터를 튜플 형태로 저장한 RDD를 사용하는 방법\
 &nbsp;&nbsp;`가장 기초적이고 간단하지만 스키마 속성을 전혀 지정할 수 없기 때문에 다소 제한적이다.`
 - [ ] 케이스 클래스를 사용하는 방법\
 &nbsp;&nbsp;`케이스 클래스를 작성하는 것으로 조금 더 복잡하지만, 첫 번째 방법만큼 제한적이지 않다.`
 - [x] 스키마를 명시적으로 지정하는 방법\
 &nbsp;&nbsp;`스키마를 명시적으로 지정하는 방법으로 마치 스파크 표준처럼 널리 사용한다.`

```java
public class CreateDataFrameTest {
    @Test
    public void case1_RDDtoDF() {
        List<String> sample = Arrays.asList(
                "apeach,7,girl",
                "ryan,40,man",
                "cony,20,woman",
                "brown,20,man"
        );
        JavaRDD<String> sampleRDD = testJavaSparkContext.parallelize(sample);
        Dataset<Row> dataFrame = testSparkSession.createDataFrame(sampleRDD.map(splitComma).map(RowFactory::create), UserInfo.schema);
        dataFrame.show();
    }
}
```
[CreateDataFrameTest.java](../src/test/java/com/example/spark/CreateDataFrameTest.java)
#### 2. SQL 쿼리를 실행 (By executing queries) <a name="excuteQuery"></a>
#### 3. 외부 데이터 로드 (Loading external data such as Parquet, CSV, JDBC,...) <a name="loadExternalData"></a>
```java
public class CreateDataFrameTest {
    @Test
    public void case3_loadExternalData_byJdbc() throws AnalysisException {
        String jdbcUrl = "jdbc:mysql://dev-db.test.com:3306";
        Properties connectionProperties = new Properties();
        connectionProperties.put("user", "develop");
        connectionProperties.put("password", "1234qwer");
        connectionProperties.put("driver", "com.mysql.jdbc.Driver");
        // case 3. load external data
        Dataset<Row> reconcileUsedJob = testSparkSession.read().jdbc(jdbcUrl, "user_info", connectionProperties);
        reconcileUsedJob.createTempView("userInfo");
        // case 2. executing queries
        Dataset<Row> results = testSparkSession.sql("select count(*) from userInfo where age = 20");
        results.show();
}
```

https://spark.apache.org/docs/2.2.0/sql-programming-guide.html
