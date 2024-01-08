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


# 创建Dataset

