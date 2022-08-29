# AURA 节点监控,报警教程

## 前提条件

你的Aura节点在正常运行，服务文件为aurad.service

## 安装并运行node exporter

在你的节点服务器上安装node exporter，用于采集主机的相关运行参数。

```
sudo  apt-get update &&  sudo  apt-get upgrade -y && \

wget https://github.com/prometheus/node_exporter/releases/download/v1.2.2/node_exporter-1.2.2.linux-amd64.tar.gz && \

tar xvf node_exporter-1.2.2.linux-amd64.tar.gz && \

rm node_exporter-1.2.2.linux-amd64.tar.gz && \

sudo mv node_exporter-1.2.2.linux-amd64 node_exporter && \

chmod +x  $HOME/node_exporter/node_exporter && \

mv $HOME/node_exporter/node_exporter /usr/bin && \

rm -Rvf  $HOME/node_exporter/
#配置系统服务
sudo tee /etc/systemd/system/exporterd.service > /dev/null <<EOF
[Unit]
Description=node_exporter
After=network-online.target
[Service]
User=$USER
ExecStart=/usr/bin/node_exporter
Restart=always
RestartSec=3
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
#启动服务
sudo systemctl daemon-reload && \
sudo systemctl enable exporterd && \
sudo systemctl restart exporterd
#查看日志
sudo journalctl -u exporterd -f
```

[![image-20211014163840249](https://raw.githubusercontent.com/silentnoname/silent666pic/master/img/image-20211014163840249.png)](https://raw.githubusercontent.com/silentnoname/silent666pic/master/img/image-20211014163840249.png)

正常情况显示如上图所示

打开节点服务器9100端口，node exporter的metrics在[http://YOURIP:9100/metrics](http://yourip:9100/metrics) 可以进行访问

## 修改aura节点的配置文件

```
cd
sed -i "s/prometheus-retention-time = 0/prometheus-retention-time = 60/g" $HOME/.aura/config/app.toml
sed -i "s/prometheus = false/prometheus = true/g" $HOME/.aura/config/config.toml
sudo systemctl restart aurad
```

打开服务器26660端口,接下来你就能在 [http://YOURIP:26660/metrics](http://yourip:26660/metrics) 看到节点的Metrics。

## 安装普罗米修斯

在你负责监控的服务器上安装prometheus

首先下载并解压

```
cd
wget https://github.com/prometheus/prometheus/releases/download/v2.30.1/prometheus-2.30.1.linux-amd64.tar.gz && \
tar xvf prometheus-2.30.1.linux-amd64.tar.gz && \
rm prometheus-2.30.1.linux-amd64.tar.gz && \
mv prometheus-2.30.1.linux-amd64 prometheus
```

接下来在普罗米修斯中添加监控

```
vim prometheus/prometheus.yml 
 #在scrape_configs:下添加如下内容后（记得对齐）重新运行普罗米修斯
 - job_name: "node_exporter"

    static_configs:
      - targets: ["nodeip:9100"]
        labels:
          label: "aura"

  - job_name: "node"

    static_configs:
      - targets: ["nodeip:26660"]
        labels:
          label: "aura"
```

配置系统服务

```
sudo tee /etc/systemd/system/prometheusd.service > /dev/null <<EOF
[Unit]
Description=prometheus
After=network-online.target
[Service]
User=$USER
ExecStart=$HOME/prometheus/prometheus --config.file="/root/prometheus/prometheus.yml"
Restart=always
RestartSec=3
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF

#启动服务
sudo systemctl daemon-reload && \
sudo systemctl enable prometheusd && \
sudo systemctl restart prometheusd
#查看日志
sudo journalctl -u prometheusd -f
```

## 安装grafana

### 安装docker

```
sudo apt update -y && sudo apt upgrade -y
sudo apt-get install ca-certificates curl gnupg lsb-release -y
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin -y
sudo systemctl restart docker
```

### 使用docker运行grafana

```
sudo docker run -d -p 3000:3000 --name grafana grafana/grafana:9.0.5
```

打开http://你的ip:3000

默认的用户名和密码都是admin。打开主页后，点击add your first data source，添加promethues数据源.

![](https://raw.githubusercontent.com/silentnoname/silent666pic/master/img/grafanadb.jpg)

URL处输入`http://你的普罗米修斯服务器ip:9090`（记得开启普罗米修斯服务器的9090端口）。如果promethues和grafana在同一主机上，则输入`http://localhost:9090`即可，输入完毕后点击Save & test保存并测试。

接下来用我的json文件导入dashboard。https://github.com/silentnoname/Node-runner/blob/main/monitor/aura-dashboard.json

![](https://raw.githubusercontent.com/silentnoname/silent666pic/master/img/auradashboard.jpg)

选择合适的数据源，chain和node变量默认aura即可。导入成功后，你便能看到如下图的仪表盘。
![](https://raw.githubusercontent.com/silentnoname/silent666pic/master/img/dashboard.jpg)



## 设置警报

首先点击左边菜单栏中Alerting下的Contract point添加通知方式，有discord,钉钉等

![](https://raw.githubusercontent.com/silentnoname/silent666pic/master/img/contactpoint.jpg)

接下来在刚刚dashboard的每个panel中添加alert，设置条件，通知方式等即可。

![](https://raw.githubusercontent.com/silentnoname/silent666pic/master/img/alert.jpg)

如图是我设置的alert的效果。

![](https://raw.githubusercontent.com/silentnoname/silent666pic/master/img/disalert.jpg)

