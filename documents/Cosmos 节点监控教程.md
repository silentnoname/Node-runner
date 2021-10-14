# Cosmos 节点监控教程

## 安装并运行node exporter

在你的节点服务器上安装node exporter，用于采集主机的相关运行参数。

```shell
sudo  apt-get update &&  sudo  apt-get upgrade -y && \

wget https://github.com/prometheus/node_exporter/releases/download/v1.2.2/node_exporter-1.2.2.linux-amd64.tar.gz && \

tar xvf node_exporter-1.2.2.linux-amd64.tar.gz && \

rm node_exporter-1.2.2.linux-amd64.tar.gz && \

sudo mv node_exporter-1.2.2.linux-amd64 node_exporter && \

chmod +x  $HOME/node_exporter/node_exporter && \

mv $HOME/node_exporter/node_exporter /usr/bin && \

rm -Rvf  $HOME/node_exporter/
```

```
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
```

```
#启动服务
sudo systemctl daemon-reload && \
sudo systemctl enable exporterd && \
sudo systemctl restart exporterd
```

```
#查看日志
sudo journalctl -u exporterd -f
```

![image-20211014163840249](https://raw.githubusercontent.com/silentnoname/silent666pic/master/img/image-20211014163840249.png)

正常情况显示如上图所示

node exporter的metrics在http://YOURIP:9100/metrics  可以进行访问



接下来是修改cosmos节点的配置文件和配置cosmos节点的系统服务

```shell
sed -i '/\[api\]/{:a;n;/enable/s/false/true/;Ta;}' $HOME/.<YOURCOSMOSCHAIN>/config/app.toml
sed -i "s/prometheus-retention-time = 0/prometheus-retention-time = 60/g" $HOME/.<YOURCOSMOSCHAIN>/config/app.toml
sed -i "s/prometheus = false/prometheus = true/g" $HOME/.<YOURCOSMOSCHAIN>/config/config.toml
```

`$HOME/.<YOURCOSMOSCHAINNAME>`一般是节点安装后数据的默认位置，请把`<YOURCOSMOSCHAIN>`换成对应的文件夹名

```shell
#为你的节点建立系统服务
sudo tee <<EOF >/dev/null /etc/systemd/system/<YOURCOSMOSCHAIN>.service
[Unit]
Description=<YOURCOSMOSCHAIN> daemon
After=network-online.target
[Service]
User=$USER
ExecStart=$HOME/go/bin/<YOURCOSMOSCHAIN> start --log_level=warn
Restart=on-failure
RestartSec=3
LimitNOFILE=10000
[Install]
WantedBy=multi-user.target
EOF
```

```shell
#启动服务
sudo systemctl daemon-reload && \
sudo systemctl enable <YOURCOSMOSCHAIN> && \
sudo systemctl start <YOURCOSMOSCHAIN>
```

```
#查看日志
sudo journalctl -u <YOURCOSMOSCHAIN> -f
```

```
#latest_block_height 增长就说明节点运行正常
<YOURCOSMOSCHAIN> status 2>&1 | jq .SyncInfo 
```



接下来你就能在 http://YOURIP:26660/metrics  看到节点的Metrics。接下来在普罗米修斯中添加监控

```shell
vim prometheus/prometheus.yml 
```

```
 #在scrape_configs:下添加如下内容后（记得对齐）重新运行普罗米修斯
 - job_name: "node_exporter"

    static_configs:
      - targets: ["nodeip:9100"]
        labels:
          label: "node lable"

  - job_name: "node"

    static_configs:
      - targets: ["nodeip:26660"]
        labels:
          label: "node lable"

```

重新运行普罗米修斯即可







