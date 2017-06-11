#Lab 3 - 지원되는 형식 및 SerDes(serializer/deserializers) (Supported Formats and SerDes)#
*During this lab you will explore ways to define external tables using various file formats.*

**^^^AWS console을 US-East Region (N.Virginia)로 설정하자^^^**

![alt tag](https://github.com/doitintl/athena-workshop/blob/master/images/region.png)

**Pargquet 파일로 부터 테이블 생성하기 (Create External Table from Parquet files)**
- AWS console 열기 [https://console.aws.amazon.com/athena/](https://console.aws.amazon.com/athena/)

- Query 창에 다음의 DDL query를 하자 (Enter the following DDL query into the query window)
- *`<USER_NAME>`란에 AWS 아이디로 입력하는 것을 잊지 말자(Remember to replace `<USER_NAME>` with your AWS username. (e.g. shaharf))*
```
CREATE EXTERNAL TABLE IF NOT EXISTS <USER_NAME>.yellow_trips_parquet(
         pickup_timestamp BIGINT,
         dropoff_timestamp BIGINT,
         vendor_id STRING,
         pickup_datetime TIMESTAMP,
         dropoff_datetime TIMESTAMP,
         pickup_longitude FLOAT,
         pickup_latitude FLOAT,
         dropoff_longitude FLOAT,
         dropoff_latitude FLOAT,
         rate_code STRING,
         passenger_count INT,
         trip_distance FLOAT,
         payment_type STRING,
         fare_amount FLOAT,
         extra FLOAT,
         mta_tax FLOAT,
         imp_surcharge FLOAT,
         tip_amount FLOAT,
         tolls_amount FLOAT,
         total_amount FLOAT,
         store_and_fwd_flag STRING
) STORED AS PARQUET LOCATION 's3://nyc-yellow-trips/parquet/';
```

**ORC 파일로 테이블 생성하기 (Create External Table from ORC files)**
- Query 창에 다음의 DDL query를 하자 (Enter the following DDL query into the query window)
- *`<USER_NAME>`란에 AWS 아이디로 입력하는 것을 잊지 말자(Remember to replace `<USER_NAME>` with your AWS username. (e.g. shaharf))*

```
CREATE EXTERNAL TABLE IF NOT EXISTS <USER_NAME>.yellow_trips_orc(
         pickup_timestamp BIGINT,
         dropoff_timestamp BIGINT,
         vendor_id STRING,
         pickup_datetime TIMESTAMP,
         dropoff_datetime TIMESTAMP,
         pickup_longitude FLOAT,
         pickup_latitude FLOAT,
         dropoff_longitude FLOAT,
         dropoff_latitude FLOAT,
         rate_code STRING,
         passenger_count INT,
         trip_distance FLOAT,
         payment_type STRING,
         fare_amount FLOAT,
         extra FLOAT,
         mta_tax FLOAT,
         imp_surcharge FLOAT,
         tip_amount FLOAT,
         tolls_amount FLOAT,
         total_amount FLOAT,
         store_and_fwd_flag STRING
) STORED AS ORC LOCATION 's3://nyc-yellow-trips/orc/';
```

**쿼리 성능 비교하기(Compare the query performance)**
- 아래의 쿼리를 하나씩 실행하고, 아무 데이터도 나오지 않을 때의 쿼리 성능을 비교해 보자(Run the following queries one by one and compare the performance while noting the amount of scanned data.)
- *`<USER_NAME>`란에 AWS 아이디로 입력하는 것을 잊지 말자(Remember to replace `<USER_NAME>` with your AWS username. (e.g. shaharf))*

```
SELECT COUNT(*) FROM <USER_NAME>.yellow_trips_csv;
SELECT COUNT(*) FROM <USER_NAME>.yellow_trips_parquet;
SELECT COUNT(*) FROM <USER_NAME>.yellow_trips_orc;
```

```
SELECT max(passenger_count) FROM <USER_NAME>.yellow_trips_csv WHERE vendor_id <> 'VTS';
SELECT max(passenger_count) FROM <USER_NAME>.yellow_trips_parquet WHERE vendor_id <> 'VTS';
SELECT max(passenger_count) FROM <USER_NAME>.yellow_trips_orc WHERE vendor_id <> 'VTS';
```

**사용자 정의 포맷으로 텍스트 파일로 테이블을 만들어 보기.(Create External Table from text files with custom format)**
- 아래의 쿼리를 실행해 보자(Run the following DDL query)
- *`<USER_NAME>`란에 AWS 아이디로 입력하는 것을 잊지 말자(Remember to replace `<USER_NAME>` with your AWS username. (e.g. shaharf))*

```
CREATE EXTERNAL TABLE IF NOT EXISTS <USER NAME>.elb_logs_raw_native (
         request_timestamp string,
         elb_name string,
         request_ip string,
         request_port int,
         backend_ip string,
         backend_port int,
         request_processing_time double,
         backend_processing_time double,
         client_response_time double,
         elb_response_code string,
         backend_response_code string,
         received_bytes bigint,
         sent_bytes bigint,
         request_verb string,
         url string,
         protocol string,
         user_agent string,
         ssl_cipher string,
         ssl_protocol string 
) 
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.RegexSerDe'
WITH SERDEPROPERTIES (
         'serialization.format' = '1','input.regex' = '([^ ]*) ([^ ]*) ([^ ]*):([0-9]*) ([^ ]*):([0-9]*) ([.0-9]*) ([.0-9]*) ([.0-9]*) (-|[0-9]*) (-|[0-9]*) ([-0-9]*) ([-0-9]*) \\\"([^ ]*) ([^ ]*) (- |[^ ]*)\\\" (\"[^\"]*\") ([A-Z0-9-]+) ([A-Za-z0-9.-]*)$' )
LOCATION 's3://athena-examples/elb/raw/';
```

- 쿼리 실행이 끝날을 때 아래의 쿼리를 실행해 보자(When finished run the following query)

```
SELECT * FROM <USERNAME>.elb_logs_raw_native WHERE elb_response_code <> '200' LIMIT 100;
```

**이전 쿼리의 결과값으로 테이블 생성하기(Create External Table from results of previous queries)**
- ‘Settings’으로 들어가자(Click the ‘Settings’ button at the top right.)

- ‘Query results location’에 아이디를 하위폴더로 하여 설정하자(Set the URL in the ‘Query results location’ by adding your username to as a sub-folder to the bucket and find the bucket in the S3 console)
- 아래의 쿼리를 실행해보자(Run the following query)
- *`<USER_NAME>`란에 AWS 아이디로 입력하는 것을 잊지 말자(Remember to replace `<USER_NAME>` with your AWS username. (e.g. shaharf))*

```
SELECT * 
FROM <USER NAME>.yellow_trips_csv 
limit 10000;
```

- S3 bucket에 쿼리 결과가 있는지 확인해보자. 그리고 Setting에서 추가한 폴더가 있는지 확인하자.(경로는 다음과 비슷하다 .../Unsaved/2017/01/24/aa9...fbb.csv)

- 재사용을 위하여 S3의 다른 하위 폴더에 결과값을 복사해 두자. (Copy the results into another sub-folder under your username in the S3 bucket for re-use.)
- 아래의 DDL 쿼리를 실행하자. `<RESULTS PATH>`의 값을 수정하는 것을 잊지 말자.(Run the DDL query below after replacing `<RESULTS PATH>` with this path  )
- *`<USER_NAME>`란에 AWS 아이디로 입력하는 것을 잊지 말자(Remember to replace `<USER_NAME>` with your AWS username. (e.g. shaharf))*
```
CREATE EXTERNAL TABLE IF NOT EXISTS <USER NAME>.yellow_trips_csv_query_result(
         pickup_timestamp BIGINT,
         dropoff_timestamp BIGINT,
         vendor_id STRING,
         pickup_datetime TIMESTAMP,
         dropoff_datetime TIMESTAMP,
         pickup_longitude FLOAT,
         pickup_latitude FLOAT,
         dropoff_longitude FLOAT,
         dropoff_latitude FLOAT,
         rate_code STRING,
         passenger_count INT,
         trip_distance FLOAT,
         payment_type STRING,
         fare_amount FLOAT,
         extra FLOAT,
         mta_tax FLOAT,
         imp_surcharge FLOAT,
         tip_amount FLOAT,
         tolls_amount FLOAT,
         total_amount FLOAT,
         store_and_fwd_flag STRING
) ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' LOCATION '<RESULTS PATH>';
```

- 아래의 쿼리를 실행하자 (Run the following query on the results table)
- *`<USER_NAME>`란에 AWS 아이디로 입력하는 것을 잊지 말자(Remember to replace `<USER_NAME>` with your AWS username. (e.g. shaharf))*
```
select count(*) from <USER NAME>.yellow_trips_csv_query_result
```

**You have sucessully completed Lab 3 :-)**
