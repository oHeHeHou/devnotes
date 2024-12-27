# 安装

## Docker

```sh
docker run \
  -p 9000:9000 -p 9009:9009 -p 8812:8812 -p 9003:9003 \
  questdb/questdb:7.3.8b
```

## Linux

```sh
tar -xvf questdb-7.3.8b-rt-linux-amd64.tar.gz

```

启动：

```sh
./questdb.sh [start|stop|status] [-d dir] [-f] [-t tag]

```

* -d 指定根目录，如果没有，默认`$HOME/.questdb`
* -t 指定当前Service的TAG
* -f 强制重启Web Console

启动后，默认的Web地址：http://localhost:9000

默认端口:

* 9000 REST API and Web Console
* 9009 InfluxDB line protocol 
* 8812 Postgres wire protocol 
* 9003 Min health server


# 创建table

当前版本没有influxdb的bucket的概念，通过创建table区分业务。

```sql
CREATE TABLE sensors (ID LONG, make STRING, city STRING);

```

## 查看table占用磁盘空间

```sql

SELECT size_pretty(sum(diskSize)) FROM table_partitions('my_table')

```

## 分区

分区类型如下：

* NONE
* YEAR
* MONTH
* WEEK
* DAY
* HOUR

create语句默认是PARTITION BY NONE

创建分区语句

```sql

CREATE TABLE my_table(ts TIMESTAMP, symb SYMBOL, price DOUBLE) timestamp(ts)
PARTITION BY DAY;

```

删除分区

```sql
--Delete days before 2021-01-03
ALTER TABLE my_table DROP PARTITION
WHERE timestamp < to_timestamp('2021-01-03', 'yyyy-MM-dd');

```

## Timestamp

时区存入默认是美国时间，查询时要转换成中国utc，需要转换，

```sql
SELECT to_utc(1623167145000000, '-08:00')
SELECT to_timestamp(1623167145000000, 'Asia/Shanghai')
```

查询系统当前时间

```sql
select systimestamp()

```


# Client

## Java

* JDK11

```xml
<dependency>
  <groupId>org.questdb</groupId>
  <artifactId>questdb</artifactId>
  <version>7.3.9</version>
</dependency>

```

* JDK8

```xml
<dependency>
  <groupId>org.questdb</groupId>
  <artifactId>questdb</artifactId>
  <version>7.3.9-jdk8</version>
</dependency>

```

### 写入数据

#### InfluxDB Line Protocol client


* atNow() automatically assigns a timestamp based on a server wall-clock upon receiving a row.
* at(long) assigns a specific timestamp.


### 查询数据

#### PostgreSQL wire protocol

```java
package com.myco;

import java.sql.*;
import java.util.Properties;

public class App {
    public static void main(String[] args) throws SQLException {
        Properties properties = new Properties();
        properties.setProperty("user", "admin");
        properties.setProperty("password", "quest");
        //set sslmode value to 'require' if connecting to a QuestDB Cloud instance
        properties.setProperty("sslmode", "disable");

        final Connection connection = DriverManager.getConnection(
            "jdbc:postgresql://localhost:8812/qdb", properties);
        try (PreparedStatement preparedStatement = connection.prepareStatement(
                "SELECT x FROM long_sequence(5);")) {
            try (ResultSet rs = preparedStatement.executeQuery()) {
                while (rs.next()) {
                    System.out.println(rs.getLong(1));
                }
            }
        }
        connection.close();
    }
}

```

Java中的timemills需要*1000才能转换成timestamp

```java
long timeMills = System.currentTimeMillis();
sender.timestampColumn("ts", timeMills*1000)
```

## Python

### 查询数据

#### PostgreSQL wire protocol

```python

import psycopg as pg
import time

# Connect to an existing QuestDB instance

conn_str = 'user=admin password=quest host=127.0.0.1 port=8812 dbname=qdb'
with pg.connect(conn_str, autocommit=True) as connection:

    # Open a cursor to perform database operations

    with connection.cursor() as cur:

        #Query the database and obtain data as Python objects.

        cur.execute('SELECT * FROM trades_pg;')
        records = cur.fetchall()
        for row in records:
            print(row)

# the connection is now closed

```



