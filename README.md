```jsx
CREATE TABLE USERS (CUSTOMER_ID VARCHAR PRIMARY KEY)
WITH (KAFKA_TOPIC='asgard.demo.CUSTOMERS', VALUE_FORMAT='AVRO');
```

**Query the ksqlDB table:**

```jsx
SET 'auto.offset.reset' = 'earliest';
SELECT CUSTOMER_ID, FIRST_NAME, LAST_NAME, EMAIL, COMMENTS FROM USERS EMIT CHANGES LIMIT 5;
```

****Make changes in MySQL, observe it in Kafka****

In MySQL terminal , you can make any changes in the table CUSTOMERS (INSERT,DELETE,UPDATE) and observe it with :

```jsx
SELECT TIMESTAMPTOSTRING(ROWTIME, 'HH:mm:ss') AS EVENT_TIME,
ID,
FIRST_NAME,
LAST_NAME,
EMAIL,
COMMENTS
FROM USERS_STREAM WHERE ID=190
EMIT CHANGES;
```

```jsx
+-----------+-------------+-----------+----------+-----------------+------------+
|EVENT_TS   |CUSTOMER_ID  |FIRST_NAME |LAST_NAME |EMAIL            |COMMENTS    |
+-----------+-------------+-----------+----------+-----------------+------------+
|09:20:15   |190           |Wiam       |Mossalli  |WIAM@example.com |great      |
^CQuery terminated
```

**Create stream CLUB_STATUS:**

```jsx
CREATE STREAM CLUB_STATUS AS
SELECT CLUB_STATUS, EMAIL
FROM   USERS_STREAM
WHERE CLUB_STATUS = 'checked'
PARTITION BY EMAIL;
```

**Aggregations:**

this is a simple aggragation : count of ratings per person , per 15 minutes: 

```jsx
CREATE TABLE RATINGS_CLUBS_PER_15MIN  AS
SELECT FIRST_NAME,COUNT(*) AS COUNT_CLUBS
FROM CLUB_STATUS
WINDOW TUMBLING (SIZE 15 MINUTE)
GROUP BY FIRST_NAME
EMIT CHANGES;
```

**Push Query :** 

```jsx
SELECT FIRST_NAME , COUNT_CLUBS
FROM RATINGS_CLUBS_PER_15MIN
WHERE FIRST_NAME='Wiam'
EMIT CHANGES;
```

**Pull Query :**

```jsx
SELECT TIMESTAMPTOSTRING(WINDOWSTART, 'yyyy-MM-dd
HH:mm:ss') AS WINDOW_START_TS,
FIRST_NAME,
COUNT_CLUBS
FROM   RATINGS_CLUBS_PER_15MIN
WHERE  FIRST_NAME='Wiam'
AND  WINDOWSTART > '2020-07-06T15:30:00.000';
```

## Part 03 : Show REST API with [Postman](https://github.com/confluentinc/demo-scene/blob/master/build-a-streaming-pipeline/ksqlDB.postman_collection.json)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ebd590b7-7918-4789-8d24-4ffa8aa0a072/Untitled.png)

```jsx
{"ksql":
"SELECT TIMESTAMPTOSTRING(WINDOWSTART, 'yyyy-MM-dd HH:mm:ss')
AS WINDOW_START_TS,FIRST_NAME,COUNT_CLUBS FROM RATINGS_CLUBS_PER_15MIN;",
"streamProperties":{"ksql.streams.auto.offset.reset":"earliest"}}
```
