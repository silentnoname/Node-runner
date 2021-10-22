# EVMOS节点中文教程

# 官方教程

https://evmos.dev/testnet/join.html

# 最低配置

- 4 or more physical CPU cores
- At least 500GB of SSD disk storage
- At least 16GB of memory (RAM)
- At least 100mbps network bandwidth



# 安装二进制文件

## 安装go

```
#安装go1.17.1
sudo rm -rf /usr/local/go;
curl https://dl.google.com/go/go1.17.linux-amd64.tar.gz | sudo tar -C/usr/local -zxvf - ;
cat <<'EOF' >>$HOME/.profile
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export GO111MODULE=on
export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin
EOF
source $HOME/.profile
```

```
#安装完成后运行以下命令查看版本
go version
```

## 安装其他必要的环境

```
sudo apt-get update -y && sudo apt-get upgrade -y;
sudo apt-get install build-essential -y;
```

## 下载源代码并编译

```
git clone https://github.com/tharsis/evmos.git
cd evmos
make install
```

安装完成后可以运行 ` evmosd version`检查是否安装成功



# 创建key

```
evmosd keys add <mykey>
```

`<mykey>`处输入任意你想设置的名字，创建成功后有显示如下。建议备份你的助记词

![](https://github.com/silentnoname/silent666pic/blob/master/img/evmosg.png?raw=true)

可以用`evmosd keys list`命令查看你的key



# 运行节点

## 保存chain id

``` 
evmosd config chain-id evmos_9000-1
```

## 初始化节点

```
evmosd init <your_custom_moniker> --chain-id evmos_9000-1
```

`<your_custom_moniker>`处输入你想设置的名字

## 下载Genesis 文件

```
curl https://raw.githubusercontent.com/tharsis/testnets/main/arsia_mons/genesis.json > ~/.evmosd/config/genesis.json
```

接下来验证genesis文件的正确性

```
evmosd validate-genesis
```

正常情况显示如下

```
File at /root/.evmosd/config/genesis.json is a valid genesis file
```

##  添加seeds节点

https://github.com/tharsis/testnets/blob/main/arsia_mons/seeds.txt 这个网站里有几个seeds节点

编辑`~/.evmosd/config/config.toml`，将seeds添加如下：

```
#######################################################
###           P2P Configuration Options             ###
#######################################################
[p2p]

# ...

# Comma separated list of seed nodes to connect to
seeds = "<node-id>@<ip>:<p2p port>"
```

注意，seeds节点之间用逗号分隔。

## 添加Persistent Peers

运行

```
TESTNET_REPO="https://raw.githubusercontent.com/tharsis/testnets/main/arsia_mons" && \
export PEERS="$(curl -s "$TESTNET_REPO/peers.txt")"
```

将peers添加到配置文件中

```
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" ~/.evmosd/config/config.toml
```

# 运行节点

## 建立系统服务

```
#为你的节点建立系统服务
sudo tee <<EOF >/dev/null /etc/systemd/system/evmosd.service
[Unit]
Description=evmosd daemon
After=network-online.target
[Service]
User=$USER
ExecStart=$HOME/go/bin/evmosd start --log_level=info
Restart=on-failure
RestartSec=3
LimitNOFILE=10000
[Install]
WantedBy=multi-user.target
EOF
```

## 启动服务

```
#启动服务
sudo systemctl daemon-reload && \
sudo systemctl enable evmosd && \
sudo systemctl start evmosd
```

## 查看日志

```
sudo journalctl -u evmosd -f
```

## 检查节点状态

```
evmosd status 
```



# 创建验证人

## 在水龙头中领取测试币

### 获取你的地址

```
evmosd keys show <mykey>
```

`<mykey>`是之前创建的key的名字

### 加入discord

[discord.gg/trje9XuAmy](https://t.co/K6tyDe7IEV?amp=1) 

在#verify 频道验证后可以打开#faucet频道，运行`!faucet 你的地址` 命令即可领取测试币。目前水龙头能领的测试币较少，建议查看#announcement频道完成任务获得测试币奖励



## 创建验证人

获取足够测试币，且节点完成同步后，可以创建验证人。只有质押量在前300的验证人才是有效验证人

### 查询余额

```
evmosd query bank balances <你的地址>
```

### 查询节点同步状态

```
 evmosd status |jq
```

当` "catching_up": false`时，节点已经完成同步，即可创建验证人

### 创建验证人

```
evmosd tx staking create-validator \
  --amount=1000000000000aphoton \
  --pubkey=$(evmosd tendermint show-validator) \
  --moniker="<your_custom_moniker>" \
  --chain-id="evmos_9000-1" \
  --commission-rate="0.10" \
  --commission-max-rate="0.20" \
  --commission-max-change-rate="0.01" \
  --min-self-delegation="1000000" \
  --gas="auto" \
  --gas-prices="0.025aphoton" \
  --from=<mykey>
```

`<your_custom_moniker>`输入你之前设置的验证人名字，`<mykey>`输入你的key名



### 查询验证人状态

```
evmosd status |jq
```

`"VotingPower"` 大于0时说明你是有效验证人。



### 地址转换

```
evmosd debug addr 你的evmos开头的地址
```

例如

```
evmosd debug addr evmos1z3t55m0l9h0eupuz3dp5t5cypyv674jj7mz2jw
  Address: [20 87 74 109 255 45 223 158 7 130 139 67 69 211 4 9 25 175 86 82]
  Address (hex): 14574A6DFF2DDF9E07828B4345D3040919AF5652
  Bech32 Acc: evmos1z3t55m0l9h0eupuz3dp5t5cypyv674jj7mz2jw
  Bech32 Val: evmosvaloper1z3t55m0l9h0eupuz3dp5t5cypyv674jjn4d6nn
```

`Bech32 Val` 后的就是你的validator address



### 质押代币到验证人节点

```
evmosd tx staking delegate <validator address>
<amount>aphoton --from <mykey> --gas 200000 --gas-prices 0.025aphoton 
```



## Unjail

当你的验证人状态为Jailed时（通常是因为节点同步出现了问题），在Jailed后等待一段时间可以运行以下命令unjail

```
evmosd tx slashing unjail --from <mykey> --chain-id evmos_9000-1  --gas 200000 --gas-prices 0.025aphoton 
```

