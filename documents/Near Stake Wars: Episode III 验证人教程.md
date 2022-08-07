# Near Stake Wars: Episode III 验证人教程

本文仅供参考，请根据[官方文档](https://github.com/near/stakewars-iii/blob/main/challenges/001.md)为准

官方最低配置要求

| Hardware | Chunk-Only Producer Specifications |
| -------- | ---------------------------------- |
| CPU      | 4-Core CPU with AVX support        |
| RAM      | 8GB DDR4                           |
| Storage  | 500GB SSD                          |

这里我选择用[hetzner的独服](https://www.hetzner.com/dedicated-rootserver)，价格34欧一个月，系统选择Ubuntu20.04。

![image-20220808000213828](https://raw.githubusercontent.com/silentnoname/silent666pic/master/img/ax41.jpg)

登入服务器后，首先检查你的cpu是否能满足要求

```
lscpu | grep -P '(?=.*avx )(?=.*sse4.2 )(?=.*cx16 )(?=.*popcnt )' > /dev/null \
  && echo "Supported" \
  || echo "Not supported"
```

返回Supported即可

### 新建用户near

新建用户near,赋予sudo权限。切换到该用户

```
sudo adduser near
usermod -aG sudo near
su near
cd
```

### 安装NEAR-CLI

｜注意：出于安全原因，建议将 NEAR-CLI 安装在与验证节点不同的计算机上，并且不要在验证节点上保留完整的访问密钥。

安装nodejs和其他必要的工具

```
sudo apt update && sudo apt upgrade -y
curl -sL https://deb.nodesource.com/setup_18.x | sudo -E bash -  
sudo apt install build-essential nodejs
PATH="$PATH"
```

检查 `Node.js` 和 `npm`的版本

```
node -v
```

> v18.x.x

```
npm -v
```

> 8.x.x

安装NEAR-CLI

```
sudo npm install -g near-cli
```

配置一些环境变量

```
echo 'export NEAR_ENV=shardnet' >> ~/.bashrc
echo 'export NEAR_ENV=shardnet' >> ~/.bash_profile
source $HOME/.bash_profile
```

### 安装NEAR节点

安装必要的工具

```
sudo apt install -y git binutils-dev libcurl4-openssl-dev zlib1g-dev libdw-dev libiberty-dev cmake gcc g++ python docker.io protobuf-compiler libssl-dev pkg-config clang llvm cargo make build-essential clang
```

如果安装docker和python遇到问题，运行

```
sudo apt install python3
sudo apt install docker-ce
```

安装pip

```
sudo apt install python3-pip
```

配置环境变量

```
USER_BASE_BIN=$(python3 -m site --user-base)/bin
export PATH="$USER_BASE_BIN:$PATH"
```

安装Rust和Cargo

```
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

![img](https://github.com/near/stakewars-iii/raw/main/challenges/images/rust.png)

这里输1，安装完成后，记得

```
source $HOME/.cargo/env
```

安装neard

```
git clone https://github.com/near/nearcore
cd nearcore
git fetch
git checkout 68bfa84ed1455f891032434d37ccad696e91e4f5
cargo build -p neard --release --features shardnet
```

初始化工作目录

```
/home/near/nearcore/target/release/neard --home ~/.near init --chain-id shardnet --download-genesis
```

![img](https://github.com/near/stakewars-iii/raw/main/challenges/images/initialize.png)

更新config.json

```
rm ~/.near/config.json
wget -O ~/.near/config.json https://s3-us-west-1.amazonaws.com/build.nearprotocol.com/nearcore-deploy/shardnet/config.json
```

开启节点

```
sudo vim /etc/systemd/system/neard.service
```

```
[Unit]
Description=NEARd Daemon Service

[Service]
Type=simple
User=root
#Group=near
WorkingDirectory=/home/near/.near
ExecStart=/home/near/nearcore/target/release/neard run
Restart=on-failure
RestartSec=30
KillSignal=SIGINT
TimeoutStopSec=45
KillMode=mixed

[Install]
WantedBy=multi-user.target
```

```
sudo systemctl deamon-reload && sudo systemctl enable neard.service && sudo systemctl start neard.service 
```

检查日志

```
sudo journalctl -u neard.service -f
```

如下即可

![img](https://github.com/near/stakewars-iii/raw/main/challenges/images/download.png)

### 成为验证人

#### 导入钱包

在https://wallet.shardnet.near.org/ 创建一个钱包后，在服务器上输入

```
near login
```

|注意：此命令启动 Web 浏览器，允许在本地复制完整访问密钥的授权。

![img](https://github.com/near/stakewars-iii/raw/main/challenges/images/1.png)

打开服务器中显示的链接，授权

![img](https://github.com/near/stakewars-iii/raw/main/challenges/images/3.png)

授权后，网页显示如下

![img](https://github.com/near/stakewars-iii/raw/main/challenges/images/4.png)

在服务器上输入你的钱包名，显示如下即可

![img](https://github.com/near/stakewars-iii/raw/main/challenges/images/5.png)

#### 设置validator_key.json

```
cp ~/.near-credentials/shardnet/你导入的钱包.json ~/.near/validator_key.json
```

```
vim  ~/.near/validator_key.json
```

将"account_id“后的内容改为

```
"xxx.factory.shardnet.near"
```

其中xxx为你池子的名字。将private_key改为secret_key。修改完毕后，`~/.near/validator_key.json`内容如下

```
{
  "account_id": "xxx.factory.shardnet.near",
  "public_key": "ed25519:HeaBJ3xLgvZacQWmEctTeUqyfSU4SDEnEwckWxd92W2G",
  "secret_key": "ed25519:****"
}
```

#### 重新运行节点

```
sudo systemctl restart neard
```

查看日志

```journalctl -n 100 -f -u neard
sudo journalctl -n 100 -f -u neard
```

#### 成为验证者

为了成为验证者并进入验证者集，必须满足以下标准。

- 节点必须完全同步
- `validator_key.json`必须在合适的位置
- 合约必须以 public_key 初始化`validator_key.json`
- account_id 必须设置为staking pool contract id
- 必须有足够的质押来满足最低质押要求。[在此处](https://explorer.shardnet.near.org/nodes/validators)查看最低质押要求。
- 必须通过 ping contract提交提案
- 一旦提案被接受，验证者必须等待 2-3 个 epoch 才能进入验证者集
- 一旦进入验证器集合，验证器必须出块超过 90% 

检查验证者节点的运行状态。如果出现“Validator”，则表明您的池在当前验证者列表中被选中。

### 设置质押池

#### 部署质押池

```
near call factory.shardnet.near create_staking_pool '{"staking_pool_id": "<pool name>", "owner_id": "<accountId>", "stake_public_key": "<public key>", "reward_fee_fraction": {"numerator": 5, "denominator": 100}, "code_hash":"DD428g9eqLL8fWUxv8QSpVFzyHi1Qd16P8ephYCTmMSZ"}' --accountId="<accountId>" --amount=30 --gas=300000000000000
```

`pool name`即你之前validator.json设置的"xxx.factory.shardnet.near"中的xxx。

假如silent.factory.shardnet.near，即为silent。

`accountId`即你的钱包名，如silent.shardnet.near

`public key`即validator.json中的public key

`5`佣金，5即该质押池的佣金率为5%

**建议账户中至少有70+ NEAR，否则交易可能会失败**

交易成功后，显示如下

![image-20220808010254101](https://raw.githubusercontent.com/silentnoname/silent666pic/master/img/ne.png)

如果你还有测试near，还可以质押更多。

```
near call <pool_id> deposit_and_stake --amount <amount> --accountId <accountId> --gas=300000000000000
```

之后可以运行

```
near proposals |grep xxx.factory.shardnet.near
```

结果如下即正常。如果质押足够，将为成为活跃验证人。

```| Proposal(Accepted) | silent.factory.shardnet.near                 | 130                | 1       |
| Proposal(Accepted) | silent.factory.shardnet.near                 | 130        | 1     |
```

### 监控节点状态

查看日志

```
sudo journalctl -n 100 -f -u neard
```

**日志文件示例：**

```
INFO stats: #85079829 H1GUabkB7TW2K2yhZqZ7G47gnpS7ESqicDMNyb9EE6tf Validator 73 validators 30 peers ⬇ 506.1kiB/s ⬆ 428.3kiB/s 1.20 bps 62.08 Tgas/s CPU: 23%, Mem: 7.4 GiB
```

- **Validator**：“Validator”将表明您是活跃的验证者
- 73 validators：网络上共有 73 个验证者
- **30 peers**：您当前有 30 个peers。您需要至少 3 个peers达成共识并开始验证
- **#46199418** : block – 确保block正在移动

#### RPC

只要该端口在节点服务器的防火墙中被打开，网络中的任何节点都会在端口 3030 上提供 RPC 服务。NEAR-CLI 在后台使用 RPC 调用。RPC 的常见用途是检查验证者统计信息、节点版本和查看委托人权益，尽管它可用于与区块链、账户和合约进行整体交互。

在这里找到许多命令以及如何更详细地使用它们：

https://docs.near.org/api/rpc/introduction

安装curl和jq

```
sudo apt install curl jq
```

#### 常用命令：

\####### 检查您的节点同步状态： 命令：

```
curl -s http://127.0.0.1:3030/status | jq .sync_info
```

"syncing": false即同步成功。

\####### 检查您的节点版本： 命令：

```
curl -s http://127.0.0.1:3030/status | jq .version
```

\####### 检查委托人和质押命令：

```
near view <your pool>.factory.shardnet.near get_accounts '{"from_index": 0, "limit": 10}' --accountId <accountId>.shardnet.near
```

\####### 检查验证人被踢原因命令：

```
curl -s -d '{"jsonrpc": "2.0", "method": "validators", "id": "dontcare", "params": [null]}' -H 'Content-Type: application/json' 127.0.0.1:3030 | jq -c '.result.prev_epoch_kickout[] | select(.account_id | contains ("<POOL_ID>"))' | jq .reason
```

\####### 检查出块/预期出块的命令：

```
curl -r -s -d '{"jsonrpc": "2.0", "method": "validators", "id": "dontcare", "params": [null]}' -H 'Content-Type: application/json' 127.0
```

