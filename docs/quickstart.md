# 快速开始
## 平台支持
由于PanIndex交叉编译需要cgo（sqlite），目前很多平台还不能很好的支持，如果你有特殊的编译需求，请告知我，我会尽量添加
- Linux （musl / x86 / amd64 / arm / arm64 ）
- Windows 8 及之后版本（x86 / amd64 ）
- Darwin（amd64）

## 下载
预编译的二进制文件压缩包可在 [Github Release](https://github.com/libsgh/PanIndex/releases "release")下载，解压后方可使用。

## 安装

### Bash脚本
- 基于`systemd`的安装脚本，以`root`账户运行
- 脚本仓库：https://github.com/libsgh/PanIndex-install
```bash
$ bash <(curl -L https://github.com/libsgh/PanIndex-install/raw/main/install-release.sh) -h
```

### 直接运行
启动参数<br>
-c=config.json #指定启动的配置文件（3.x）
```json
{
  "host": "0.0.0.0",
  "port": 5238,
  "log_level": "info",
  "data_path": "",
  "cert_file": "",
  "key_file": "",
  "config_query": "",
  "db_type": "",
  "dsn": ""
}
```
`-host=0.0.0.0` #绑定host，默认0.0.0.0<br>
`-port=5238` #绑定端口号，默认5238<br>
`-log_level=info` #日志级别，默认info，设置为debug将输出更多信息<br>
`-data_path=/path/to/data` #数据目录（配置、目录信息、临时文件目录）<br>
`-cert_file=/path/to/fullchain.pem` # 开启ssl，证书文件<br>
`-key_file=/path/to/privkey.pem` # 开启ssl，证书文件密钥<br>
`-config_query=port` # 查询配置（新版），程序并不会启动<br>
`-cq=port` # 查询配置（旧版），程序并不会启动<br>
`-db_type=sqlite` # 数据源：sqlite(默认)、mysql、postgres<br>
`-dsn=` # 数据源连接
```
# postgres
host=localhost user=postgres password=1234 dbname=pan-index port=5432 sslmode=disable TimeZone=Asia/Shanghai
# mysql
user:pass@tcp(127.0.0.1:3306)/dbname?charset=utf8mb4&parseTime=True&loc=Local
# sqlite
data.data.db
```
> 启动时若不指定配置文件，将使用默认配置。配置优先级：环境变量 > 命令参数 > 配置文件（config.json），所有参数名大写，就是环境变量的`KEY`


环境变量主要用于Docker部署。

| 变量名称            | 变量值               | 描述                              |
| ------------------- |-------------------|---------------------------------|
| HOST    | 0.0.0.0           | 绑定HOST                          |
| PORT                | 5238              | 启动端口号，由于Heroku端口号随机，并需要从此环境变量获取 |
| LOG_LEVEL                | info              | 日志级别                            |
| DATA_PATH                | /app/data         | 数据目录，主要存储配置文件，SQLITE数据库文件       |
| CERT_FILE                | -                 | 证书文件                            |
| KEY_FILE                | -                 | 证书密钥                            |
| DB_TYPE                | sqlite            | 数据源类型                           |
| DSN                | /app/data/data.db | 数据源链接                           |


```bash
$ tar -xvf PanIndex-linux-amd64.tar.gz
#nohup ./PanIndex -host=0.0.0.0 -port=5238 > PanIndex.log &
#./PanIndex -cq port
$ nohup ./PanIndex > PanIndex.log &
```
### Systemd服务运行
> 以下命令请切换到root下执行

1. 下载PanIndex并解压
```bash
$ mkdir /usr/local/etc/PanIndex
$ cd /usr/local/etc/PanIndex
$ wget https://github.com/libsgh/PanIndex/releases/download/v3.0.0/PanIndex-linux-amd64.tar.gz
$ tar -xvf PanIndex-linux-amd64.tar.gz
$ cp PanIndex* /usr/local/bin/
$ cat > "/usr/local/etc/PanIndex/config.json" << EOF
{
  "host": "0.0.0.0",
  "port": 5238,
  "log_level": "info",
  "data_path": "/usr/local/etc/PanIndex/data",
  "cert_file": "",
  "key_file": "",
  "config_query": "",
  "db_type": "sqlite",
  "dsn": "/usr/local/etc/PanIndex/data/data.db"
}
EOF
```
2. 编写PanIndex.service文件
```bash
$ vim /etc/systemd/system/PanIndex.service
```
3. service内容参考
```
[Unit]
Description=PanIndex Service
Documentation=https://libsgh.github.io/PanIndex/
After=network.target
[Service]
User=root
WorkingDirectory=/usr/local/etc/PanIndex
ExecStart=/usr/local/bin/PanIndex -c=/usr/local/etc/PanIndex/config.json
Restart=on-failure
RestartPreventExitStatus=23
LimitNPROC=10000
LimitNOFILE=1000000
[Install]
WantedBy=multi-user.target
```

4. Systemd常用命令
```bash
$ systemctl daemon-reload #PanIndex.service有修改重新加载
$ systemctl restart PanIndex #重启PanIndex
$ systemctl enable PanIndex #设置开机启动
$ systemctl disable PanIndex #关闭开机启动
$ systemctl status PanIndex #查询服务状态
$ journalctl -u PanIndex.service -f #滚动查看PanIndex日志
```

### （宝塔）Supervisor启动

1. 启动命令那里，要填PanIndex的绝对路径
![](_images/Supervisor.png)
2. 如果需要配置环境变量在子配置文件中添加才能生效

```
environment=a="1",b="2"
```
### 源码运行
- 安装git和golang
- 设置go环境变量`go env -w GO111MODULE=on`
- 如果是国内服务器，设置下代理`go env -w GOPROXY=https://goproxy.cn,direct`
```bash
$ git clone https://github.com/libsgh/PanIndex.git
$ cd PanIndex
$ nohup go run main.go > PanIndex.log &
```
也可以下载源码后自行编译成二进制程序再执行
以linux,amd64为例
```bash
$ CGO_ENABLED=1 GOOS=linux GOARCH=amd64 go build -o PanIndex
$ nohup ./PanIndex &
```
更多平台编译参考：[PanIndex-build-action](https://github.com/libsgh/PanIndex-build-action)

### Docker中运行
参考下面命令，映射`/app/data`目录到宿主机避免重启docker数据丢失！
```bash
docker pull iicm/pan-index:latest
docker stop PanIndex
docker rm PanIndex
docker run -itd \
 --restart=always \
 --name PanIndex \
 -p 5238:5238 \
 -v /home/single/data/docker/data/PanIndex/data:/app/data \
 -e PORT="5238" \
 iicm/pan-index:latest
```

## 其他平台部署

### Heroku
### Railway
### Qovery
### Koyeb

