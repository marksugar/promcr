# pcr

pcr组合了prometheus consul registrator，为了使用起来可以快速部署，使用compose将他们编排在一起使用

![1210.png](https://github.com/marksugar/pcr/blob/master/node_template/1210.png?raw=true)

- [使用说明](#使用说明)
  - [node 节点部署](#node 节点部署)
  - [registrator须知](#registrator须知)
  - [prometheus须知](#prometheus须知)
  - [grafana须知](#grafana须知)

## 使用说明

```
git clone https://github.com/marksugar/pcr.git && cd pcr
```

即将启动前你需要创建目录并且授权，这么做是为了避免容器内的属主权限问题。

分别是prometheus是数据目录和grafana的使用目录

```
mkdir -p $(pwd)/{data,grafana_data,grafana}
chown 65534:65534 $(pwd)/data
chown 472.472 $(pwd)/grafana_data/
```

```
docker-compose -f ./docker-compose.yaml up -d
```

## node 节点部署

我们将模板下载到本地

```
crul -LK https://raw.githubusercontent.com/marksugar/pcr/master/node_template/docker-compose.yaml -o /opt/docker-compose.yaml
```

在每个容器中添加标签便于区分，如果是compose，可以使用如下方式:

```
    labels:
      SERVICE_TAGS: ddt-linuxea.com
```

而后远程推送，如ansible

```
ansible pt-api -m copy -a "src=/opt/docker-compose-NAME.yaml dest=/opt/docker-compose.yaml"
```

可以在本地编写脚本，在远程节点执行

```
#！/bin/bash
iptables -I INPUT 5 -s IPADDR -p tcp -m tcp -m state --state NEW -m multiport --dports 18880,9100 -j ACCEPT
sed -i '/-A INPUT -j REJECT/i\\-A INPUT -p tcp -m tcp -m state --state NEW -m multiport --dports 18880,9100 -m comment --comment "prometheus" -j ACCEPT' /etc/sysconfig/iptables
docker-compose -f /opt/docker-compose.yaml up -d
```

## registrator须知

 在registrator镜像中，marksugar/registrator:v7.1是我自己基于gliderlabs/registrator:v7封装

```
FROM gliderlabs/registrator:v7
LABEL master="www.linuxea.com"
COPY entrypoint.sh /bin/entrypoint.sh
ENTRYPOINT ["entrypoint.sh"]
```

附加了一个脚步来获取ip地址，以及传递一些参数

```
#!/bin/sh
# maintainer="linuxea.com"
NDIP=`ip a s ${NETWORK_DEVIDE:-eth0}|awk '/inet/{print $2}'|sed -r 's/\/[0-9]{1,}//')`
/bin/registrator -ip="${NDIP}" ${ND_CMD:--internal=false} consul://${NDIPSERVER_IP:-consul}:8500
exec "$@"
```

所以，要运行至少需要三个变量

```
    - NETWORK_DEVIDE=eth0 ：网卡名称
    - NDIPSERVER_IP=CONSUL_SERVER ：CONSUL服务ip
    - ND_CMD=-internal=false -retry-interval=30 -resync=180 #其他参数
```

`-retry-interval=30`会在三分钟后自动重新联系CONSUL_SERVER

## prometheus须知

镜像是prom/prometheus:v2.5.0，数据存储45天，配置文件映射到/etc/prometheus/prometheus.yml

```
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--storage.tsdb.retention=45d'
      - '--web.console.libraries=/etc/prometheus/console_libraries'
      - '--web.console.templates=/etc/prometheus/consoles'
      - '--web.enable-lifecycle'
```

## grafana须知

```
      - GF_SECURITY_ADMIN_USER=${ADMIN_USER:-admin}
      - GF_SECURITY_ADMIN_PASSWORD=${ADMIN_PASSWORD:-admin}
      - GF_USERS_ALLOW_SIGN_UP=false
```

最终展示如下：

![1217.png](https://github.com/marksugar/pcr/blob/master/node_template/1217.png?raw=true)


![1218.png](https://raw.githubusercontent.com/marksugar/pcr/master/node_template/1218.png)