

# Stride节点中文教程

## 官方教程

https://github.com/Stride-Labs/testnet

## 最低配置

- 4 CPU 
- 32 GB RAM （官方教程写的要求偏高，实际8GB+即可）
- 100GB SSD

## 安装基础环境

安装必要的环境

```
sudo apt-get update -y && sudo apt-get upgrade -y;
sudo apt-get install curl build-essential jq git -y;
```

安装go 18+

```
sudo rm -rf /usr/local/go;
curl https://dl.google.com/go/go1.18.2.linux-amd64.tar.gz | sudo tar -C/usr/local -zxvf - ;
cat <<'EOF' >>$HOME/.profile
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export GO111MODULE=on
export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin
EOF
source $HOME/.profile
```

安装完成后运行以下命令查看版本

```
go version
```


### 下载源代码并编译

```
cd
rm -rf stride
git clone https://github.com/Stride-Labs/stride.git
cd stride
git checkout cf4e7f2d4ffe2002997428dbb1c530614b85df1b
sudo cp $HOME/stride/build/strided /usr/local/go/bin
strided version
make build
```

安装完成后可以运行 ` strided version`检查是否安装成功。

显示应为v0.3.1

## 运行节点

### 初始化节点

```
moniker=<你的节点名>
strided init $moniker --chain-id=STRIDE-TESTNET-4
```

### 下载Genesis 文件

```
curl https://raw.githubusercontent.com/Stride-Labs/testnet/main/poolparty/genesis.json > ~/.stride/config/genesis.json
```

### 设置peer和seed

```
PEERS="ca5dcb6ea9bd7d06535f95b31c253b8d880671cd@38.242.156.96:26656,f1996c054d50715f686350505f48d4f22f180e89@45.147.199.214:26656,3cff32fb64941957fd00f2b682c51db9de22c25a@95.217.9.52:16656,c53218f49bf7b5eeba132e53f935050fb54fad10@75.119.146.75:26656,bb20b23e4c656f98f0982df0030978525028d52f@164.90.149.254:26656,1e5c2c438f606a213978254a85d3dc41c6be058b@65.21.148.70:16656,07bf9eea65c63c66b5c62e18ec617f8c4bd611f1@164.68.99.180:26656,d9d37ff40bb766852be82c8bff5950505798d7cd@65.109.17.86:32656,6bf11f90fba7270b8e1367ac491738298c1606b9@65.108.218.92:26656,0333a01d672f382a7e507451db0b7f1b60d64fb5@38.242.240.224:26656"
seeds="d2ec8f968e7977311965c1dbef21647369327a29@seedv2.poolparty.stridenet.co:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.stride/config/config.toml
sed -i.bak -e "s/^seeds *=.*/seeds = \"$seeds\"/" ~/.stride/config/config.toml
```

### 启动节点

```
sudo tee <<EOF >/dev/null /etc/systemd/system/strided.service
[Unit]
Description=strided daemon
After=network-online.target
[Service]
User=$USER
ExecStart=$(which strided) start
Restart=on-failure
RestartSec=3
LimitNOFILE=10000
[Install]
WantedBy=multi-user.target
EOF
```

```
sudo systemctl daemon-reload && \
sudo systemctl enable strided && \
sudo systemctl start strided
```

### 查看日志

```
sudo journalctl -u strided -f
```



### (可选)State-sync快速同步

```
sudo systemctl stop strided
strided tendermint unsafe-reset-all --home $HOME/.stride
SEEDS=""
PEERS="6d3d7df642fd0cdf0c4b74c499cf4d5937a29d2b@23.88.100.175:26656,bf1414a4cbcfcc6c6fc11d1229f5cefcce1faef5@stride-node1.poolparty.stridenet.co:26656"; \
sed -i.bak -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.stride/config/config.toml
SNAP_RPC="http://stride.stake-take.com:26657"
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)
echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.stride/config/config.toml
sudo systemctl restart strided && journalctl -u strided -f -o cat
```

### 检查同步状态

```
curl -s localhost:26657/status | jq .result | jq .sync_info
```

其中显示`  "catching_up": `显示为`false`即已经同步上。如果一直没有开始同步一般是因为peer不够，可以考虑添加Peer或者使用别人的addrbook。

### 更换 addrbook

```
sudo systemctl stop strided
rm $HOME/.stride/config/addrbook.json
wget -O $HOME/.stride/config/addrbook.json "https://raw.githubusercontent.com/StakeTake/guidecosmos/main/stride/STRIDE-TESTNET-4/addrbook.json"
sudo systemctl restart strided && journalctl -u strided -f -o cat
```

## 创建验证人

### 创建钱包

```
strided keys add <钱包名>
```

**注意请保存助记词。若不保存，之后将无法恢复。**

### 领取测试币

进入stride discord(https://discord.gg/dkECXxpEme)

在 #💧┃token-faucet 	频道发送

```$faucet-stride:
$faucet-stride:你的stride地址
```

之后可以用

```
strided query bank balances 你的stride地址
```

查询测试币余额。

### 创建验证人

获取足够测试币，且节点完成同步后，可以创建验证人。只有质押量在前100的验证人才是活跃验证人。本次激励性测试网。验证人不论活跃与否都有奖励。

```
strided tx staking create-validator \
  --amount=1000000ustrd \
  --pubkey=$(strided tendermint show-validator) \
  --chain-id=STRIDE-TESTNET-4 \
  --commission-rate="0.05" \
  --commission-max-rate="0.20" \
  --commission-max-change-rate="0.20" \
  --min-self-delegation=1 \
  --moniker="<你的节点名>" \
  --from <你的钱包名> \
  --yes
```

之后可以去区块浏览器https://stride.explorers.guru/ 查看你的验证人是否创建成功。创建成功后，在discord      #👋┃role-request 频道发送你的验证人的区块浏览器链接，获取validator role。

### 常用命令

#### 服务管理

检查日志

```
sudo journalctl -u strided -f
```

运行/重启节点

```
sudo systemctl restart strided
```

停止节点

```
sudo systemctl stop strided
```

#### 节点信息

同步信息

```
strided status 2>&1 | jq .SyncInfo
```

验证人信息

```
strided status 2>&1 | jq .ValidatorInfo
```

节点信息

```
strided status 2>&1 | jq .NodeInfo
```

获取node id

```
strided tendermint show-node-id
```

#### 钱包操作

显示所有钱包

```
strided keys list
```

恢复钱包

```
strided keys add <你的钱包名> --recover
```

删除钱包

```
strided keys delete <你的钱包名>
```

查询余额

```
strided query bank balances <stride地址>
```

发送代币

```
strided tx bank send <你的钱包名> <接收钱包Stride地址> 数量ustrd --from <你的钱包名> -y
```

**注意：1strd=1000000ustrd**

#### 投票

```
strided tx gov vote <提案编号> <投票选项> --from <你的钱包名> -y
```

投票选项包括yes/no/no_with_veto/abstain。大部分情况我们投yes就好。

#### 质押，提取奖励

质押

```
strided tx staking delegate <你要质押的验证人地址> 数量ustrd --from <你的钱包名> -y
```

解除质押

```
strided tx staking unbond <你要解除质押的验证人地址> 数量ustrd --from <你的钱包名> -y
```

提取质押奖励和验证人佣金

```
strided tx distribution withdraw-rewards <你的验证人地址> --commission --from <你的钱包名> -y
```

提取所有奖励

```
strided tx distribution withdraw-all-rewards --from=<你的钱包名> -y
```

#### 验证人管理

修改验证人信息

```
strided tx staking edit-validator \
  --moniker=<节点名> \
  --identity=<你的keybase id> \
  --website="<你的网站>" \
  --details="<你的验证人描述>" \
  --from=<你的钱包名>
```

假如你想在区块浏览器显示你的验证人logo。需要注册一个keybase账号，上传logo。在验证人信息中设置`--identity=<你的keybase id>`，区块链浏览器中就会显示你的keybase logo作为验证人logo。

Unjail

```
strided tx slashing unjail --from <你的钱包名> -y   
```

