# Telegraf

influxdb开发的，可以用来收集和发送ioT数据。采用GO语言编写，轻量化。

https://www.influxdata.com/time-series-platform/telegraf/

## 功能示意图

![image](./img/telegraf.png)

## plugins

* Input
* Process
* Aggregate
* Output

## 安装

### Docker

```sh
# Debian-based image
docker pull telegraf
# Alpine-based image
docker pull telegraf:alpine

```

### Linux Binary

```sh
wget https://dl.influxdata.com/telegraf/releases/telegraf-1.29.2_linux_amd64.tar.gz
tar xf telegraf-1.29.2_linux_amd64.tar.gz

```

### Ubuntu

```sh

wget -q https://repos.influxdata.com/influxdata-archive_compat.key
echo '393e8779c89ac8d958f81f942f9ad7fb82a25e133faddaf92e15b16e6ac9ce4c influxdata-archive_compat.key' | sha256sum -c && cat influxdata-archive_compat.key | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/influxdata-archive_compat.gpg > /dev/null
echo 'deb [signed-by=/etc/apt/trusted.gpg.d/influxdata-archive_compat.gpg] https://repos.influxdata.com/debian stable main' | sudo tee /etc/apt/sources.list.d/influxdata.list

sudo apt-get update && sudo apt-get install telegraf

```

## 配置

创建一个默认配置文件

```sh
telegraf config > telegraf.conf
```

安装后默认配置文件

`/etc/telegraf/telegraf.conf`