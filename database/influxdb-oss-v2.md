# influx CLI

## 创建config

```sh

influx config create --config-name CONFIG_NAME \
  --host-url http://localhost:8086 \
  --org ORG \
  --token API_TOKEN \
  --active

```

## config命令

```sh
# 查看列表
influx config

# 启用
influx config local-config

# 删除
influx config delete local-config

```

## influx bucket

```
influx bucket create --name get-started

```

## influxQL

```sh
influx v1 shell

SELECT co,hum,temp,room FROM "get-started".autogen.home WHERE time >= '2022-01-01T08:00:00Z' AND time <= '2022-01-01T20:00:00Z'

```



# Line protocol

```
<measurement>[,<tag_key>=<tag_value>[,<tag_key>=<tag_value>]] <field_key>=<field_value>[,<field_key>=<field_value>] [<timestamp>]

```

每行数据用\n分割

* measurement
* tag set 逗号分隔kv
* field set 逗号分隔kv, strings (quoted), floats, integers, unsigned integers, or booleans
* timestamp unix时间戳

例子	
```
measurement,tag1=val1,tag2=val2空格field1="v1",field2=1i空格0000000000000000000

```

```
measurement: home

tags
room: Living Room or Kitchen

fields
temp: temperature in °C (float)
hum: percent humidity (float)
co: carbon monoxide in parts per million (integer)
timestamp: Unix timestamp in second precision


```

```
home,room=Living\ Room temp=21.1,hum=35.9,co=0i 1641024000
home,room=Kitchen temp=21.0,hum=35.9,co=0i 1641024000
home,room=Living\ Room temp=21.4,hum=35.9,co=0i 1641027600
home,room=Kitchen temp=23.0,hum=36.2,co=0i 1641027600
```

# API

## Flux

```sh
curl --request POST \
  http://localhost:8086/api/v2/query?org=test  \
  --header 'Authorization: Token XU9uLZZnf0rk0AZh2tuu6dvJISBk6ViV4ScvAEVktralinET69KRSzZqG418GLOCmUy-SgrIiPkTVbO3q_aXQQ==' \
  --header 'Accept: application/csv' \
  --header 'Content-type: application/vnd.flux' \
  --data 'from(bucket:"get-started")
        |> range(start: -12h)
        |> filter(fn: (r) => r._measurement == "")
        |> aggregateWindow(every: 1h, fn: mean)'

```

## InfluxQL

```sh

```



# SDK

## python


```sh
pip install influxdb-client
```

代码：

```python
import influxdb_client
from influxdb_client.client.write_api import SYNCHRONOUS

bucket = "get-started"
org = "test"
token = "XU9uLZZnf0rk0AZh2tuu6dvJISBk6ViV4ScvAEVktralinET69KRSzZqG418GLOCmUy-SgrIiPkTVbO3q_aXQQ=="
url = "http://localhost:8086"

client = influxdb_client.InfluxDBClient(
    url=url,
    token=token,
    org=org
)
write_api = client.write_api(write_options=SYNCHRONOUS)

p = influxdb_client.Point("my_measurement").tag("location", "Prague").field("temperature", 25.3)
write_api.write(bucket=bucket, org=org, record=p)

query = 'from(bucket:"get-started")\
|> range(start: -10m)\
|> filter(fn:(r) => r._measurement == "my_measurement")\
|> filter(fn:(r) => r.location == "Prague")\
|> filter(fn:(r) => r._field == "temperature")'

query_api = client.query_api()
result = query_api.query(org=org, query=query)

results = []
for table in result:
    for record in table.records:
        results.append((record.get_field(), record.get_value()))

print(results)

```

输出：

```python
	
[(temperature, 25.3)]

```
