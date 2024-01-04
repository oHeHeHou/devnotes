## 文档地址:http://supervisord.org/


# 系统需求

* 不支持Windows
* Python 3 version 3.4+ 和Python 2 version 2.7.


# 组件

* supervisord 服务器部分，配置文件/etc/supervisord.conf
* supervisorctl 命令行客户端，它为supervisord 提供的功能提供了一个类似shell 的接口
* Web Server 通过网络端口启动后，可以通过 Web 界面查看和控制进程状态
* XML-RPC Interface

# 安装

## 有网环境

```
pip install supervisor

```

## 离线环境

pypi上下载whl安装包

```
pip install supervisor-4.2.5-py2.py3-none-any.whl
```

## 生成Sample配置文件

```
echo_supervisord_conf > /etc/supervisord.conf
```

# 运行supervisord

## 命令行参数

* -c FILE 指定配置文件
* -n 前台运行
* -s 静默运行
* -d 作为守护进程，启动前切换到的目录
* -l FILE 日志输出文件
* -j FILE supervisord写pid文件的地址


### 例子

```sh
supervisord -c /etc/supervisord.conf
```

conf配置：

```sh

[program:test]
command=java -jar forever.jar
```
启动后观察可以发现这个jar的parent PID就是supervisord


# 运行supervisorctl

* 交互式 shell输入supervisorctl
* 一次性运行 supervisorctl status all


## 命令行参数

* -c FILE 指定配置文件
* -i 启动交互式shell，默认就是
* -s URL 指定监听的URL地址，默认是http://localhost:9001
* -u 用户
* -p 密码

## 交互命令

* update 重载配置文件，收到影响的进程会重启
* clear all 清除所有日志文件
* pid 获取自己的pid
* pid <name> 获取自己的pid
* reload 重启supervisord
* restart <name> 重启进程
* restart all 重启所有进程 
* stop <name> 关闭进程
* stop all 停止所有进程

# 开机启动Supervisord

## CentOS

在/etc/systemd/system/下增加supervisord.service

```
# supervisord service for systemd (CentOS 7.0+)
# by ET-CS (https://github.com/ET-CS)
[Unit]
Description=Supervisor daemon

[Service]
Type=forking
ExecStart=/usr/bin/supervisord
ExecStop=/usr/bin/supervisorctl $OPTIONS shutdown
ExecReload=/usr/bin/supervisorctl $OPTIONS reload
KillMode=process
Restart=on-failure
RestartSec=42s

[Install]
WantedBy=multi-user.target

```

# 配置文件

配置文件以ini为格式

如果不用-C参数指定配置文件，默认从以下顺序开始查找

1. ../etc/supervisord.conf (Relative to the executable)
2. ../supervisord.conf (Relative to the executable)
3. $CWD/supervisord.conf
4. $CWD/etc/supervisord.conf
5. /etc/supervisord.conf
6. /etc/supervisor/supervisord.conf (since Supervisor 3.3.0)

## 环境变量

```sh
[program:example]	
command=/usr/bin/example --loglevel=%(ENV_LOGLEVEL)s
	
```

其中ENV_LOGLEVEL由系统中的LOGLEVEL环境变量替换


### [unix_http_server]

配置unixsock http服务器

```
[unix_http_server]
file = /tmp/supervisor.sock
chmod = 0777
chown= nobody:nogroup
username = user
password = 123

```

### [inet_http_server]

```
[inet_http_server]
port = 127.0.0.1:9001
username = user
password = 123

```

### [program:x] 

* command 启动进程的命令
* process_name 默认%(program_name)s
* numprocs 启动进程的数量，如果>1，则process_name必须包含%(process_num)s表达式
* numprocs_start process_num的偏移量
* priority 小的数字：先启动后关闭，大数字：后启动先关闭
* autostart true时，supervisord启动后，程序会启动
* user 指定运行此程序的用户
* startretries 允许启动失败后重试次数
* autorestart 如果为true，在进程退出后无条件重启

```
[program:cat]
command=/bin/cat
process_name=%(program_name)s
numprocs=1
directory=/tmp
umask=022
priority=999
autostart=true
autorestart=unexpected
startsecs=10
startretries=3
exitcodes=0
stopsignal=TERM
stopwaitsecs=10
stopasgroup=false
killasgroup=false
user=chrism
redirect_stderr=false
stdout_logfile=/a/path
stdout_logfile_maxbytes=1MB
stdout_logfile_backups=10
stdout_capture_maxbytes=1MB
stdout_events_enabled=false
stderr_logfile=/a/path
stderr_logfile_maxbytes=1MB
stderr_logfile_backups=10
stderr_capture_maxbytes=1MB
stderr_events_enabled=false
environment=A="1",B="2"
serverurl=AUTO
```

### [group:x]

```
[group:foo]
programs=bar,baz
priority=999

```

程序bar,baz都放在foo组





