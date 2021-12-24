# Humanode 节点Shamshel TestNet中文教程
本教程根据官方教程翻译修改而来。若遇到问题请参照官方教程。

官方教程： https://desktop-app-docs.humanode.io/
## 安装Humanode Desktop App
在您的台式机（或笔记本电脑）上安装Humanode Desktop App，然后在本地或远程（通过 SSH 在 Ubuntu 系统上）安装节点。远程系统可以是headless的——即普通的Cloud VM或 VPS 可以正常工作

下载地址
https://desktop-app.testnet2.stages.humanode.io/

## 硬件要求
2 核CPU 

4 GB RAM

40 GB 硬盘空间

100 Mbps 网络连接

公共IP地址或通过NAT端口转发（或者通过`ngrok`建立通道）

## 用于 Web 应用程序的手持设备
您将需要手持设备（如手机或平板电脑）来捕获生物特征数据。 无需特殊配置。

ios 或安卓系统。（尚不支持 iOS 15）

有摄像头

有加速度传感器

安装浏览器（Chrome / Firefox / Safari）

## 远程运行节点
首先租用一台Ubuntu 20.04满足以上配置要求的vps。

打开Humanode Desktop App,点击`CREATE WORKSPACE`，选择`REMOTELY`.输入ssh连接信息。
![ssh-connect](https://github.com/silentnoname/silent666pic/blob/master/img/hm1.png?raw=true)

连接后，将会自动帮你在服务器上安装节点，接下来你必须在启动之前配置您的节点。


## 本地运行节点
### Windows
windows版需要安装WSL,可能出现各种问题，可能安装失败。不推荐使用

打开Humanode Desktop App,点击,点击`CREATE WORKSPACE`，选择`Start locally`

如果你的WSL版本不是最新，或者没有安装WSL,接下来的步骤会帮你安装。
首先点击`INSTALL`
![install](https://desktop-app-docs.humanode.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FLkvcOh8zYczeTBsQab1j%2Fuploads%2F2zQErmBmW3avPrBeIozH%2FScreenshot%202021-12-23%20at%2017.16.31.png?alt=media&token=ce34d657-adea-4a5c-bcc2-d9eea185da17)

您将看到带有 WSL 安装进度的终端窗口。
安装完成后，你需要按`restart`来重新启动机器。
![restart](https://desktop-app-docs.humanode.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FLkvcOh8zYczeTBsQab1j%2Fuploads%2FKZalCOxZ1Ui72BTlF9rD%2FScreenshot%202021-12-23%20at%2017.17.39.png?alt=media&token=1916c12b-235b-4f86-97c9-2511a873ca1d)

重新启动系统后，Ubuntu 安装过程应该开始了，等待，然后打开Humanode Desktop App,点击`CREATE WORKSPACE`，选择`Start locally`然后选择发行版。 我们推荐使用 Ubuntu。

然后点击`INSTALL`,开始测试网节点的安装
![install_node](https://desktop-app-docs.humanode.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FLkvcOh8zYczeTBsQab1j%2Fuploads%2FwcPcwBDkSwg9BMN9LMMD%2FScreenshot%202021-12-23%20at%2016.42.49.png?alt=media&token=0b4478da-aa19-4b2b-9285-32eff5bfd1dc)
当进度条填充到 100% 时，您将被重定向到开始初始配置的主界面。 这表明测试网节点已成功安装，你必须在启动节点之前配置您的节点。

### Linux / macOS
打开Humanode Desktop App,点击`CREATE WORKSPACE`，选择`Start locally`，开始测试网节点的安装
![install_node](https://desktop-app-docs.humanode.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FLkvcOh8zYczeTBsQab1j%2Fuploads%2FwcPcwBDkSwg9BMN9LMMD%2FScreenshot%202021-12-23%20at%2016.42.49.png?alt=media&token=0b4478da-aa19-4b2b-9285-32eff5bfd1dc)
当进度条填充到 100% 时，您将被重定向到开始初始配置的主界面。 这表明测试网节点已成功安装，你必须在启动节点之前配置您的节点。

## 配置节点
### Node Settings
这里可以让你设置你的Node名称、Public URL（如果你注册了域名），选择RPC URL Mode(可以通过Ngrok（推荐）、Localhost和Custom（你可以手动输入您的RPC URL))。我们需要的 RPC 端口是 9933、9944、30333。
请注意，只有在您之前未在我们的网络中注册且未通过注册过程时，才应启用`Bioauth enrollment mode`。 如果您之前已经注册过了，请不要选择。（第一次运行节点时需要选择，之后不需要选择）
建议将所有内容保留为默认值，只添加`NODE NAME`,仅当您的主机或机器具有public address时才添加`Public URL`，否则将其留空。
按`APPLY`保存更改或按`RESET`恢复以前保存的设置。

![](https://desktop-app-docs.humanode.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FLkvcOh8zYczeTBsQab1j%2Fuploads%2FLmQeYuL0wyk7akGrV5gF%2FScreenshot%202021-12-23%20at%2016.47.34.png?alt=media&token=3c1f8406-2204-4092-9cf7-d76da247b9ce)
点击`APPLY`后，可以看下一部分的内容`Key Management`
### Key Management
这里可以选择 **Insert** 或**Generate**您的助记词，用于生成验证人密钥对。 您的生物识别身份将链接到您的验证者密钥对，因此您需要正确获取和管理你的密钥对，不泄漏它。
如果您还没有密钥对的话，需要生成一个新的密钥对。 点击 `Insert` 助记词并点击`Generate`来生成一个助记词。

![](https://desktop-app-docs.humanode.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FLkvcOh8zYczeTBsQab1j%2Fuploads%2Fp1uHpY8tYJ9H8iRhqXH9%2FScreenshot%202021-12-23%20at%2016.49.10.png?alt=media&token=f9665961-3f32-4250-8d07-f160a42d326a)
点击`Generate`后，Humanode Desktop App将创建一个新的助记词，你必须保存好它。

在保存助记词之前不要按`Close`！ 我们故意阻止您将生成的助记符直接复制到输入中 - 以便您将其放在某处。 这是将其保存在适当位置的好时机，因为如果您丢失了它,就无法参加测试网了。 比如说，你可以把它写在笔记本上。

**如果您在完成 bioauth 注册后丢失了助记词，您将无法在网络中进行验证。 目前我们没有恢复机制，所以您将无法再参与测试网**

![](https://desktop-app-docs.humanode.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FLkvcOh8zYczeTBsQab1j%2Fuploads%2FhWYK7WJjJtF8YW4Zs6vg%2FScreenshot%202021-12-23%20at%2016.50.16.png?alt=media&token=235b8729-cf4b-41ee-8b1e-369b5fd68015)
点击`Close`后，您将返回上一个窗口，您可以在输入框中输入助记词并按`Insert`，密钥将被保存。

如果您输入了错误的助记词，则需要在`settings`页面底部点击`Remove Workspace`，然后重新创建一个新的工作区。


### ngrok settings
正常情况下不需要进行设置

### Danger Zone
**这里含有破坏性操作，需谨慎使用。**
![](https://desktop-app-docs.humanode.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FLkvcOh8zYczeTBsQab1j%2Fuploads%2FmpU1W0XtiGmSxss2MFa1%2FScreenshot%202021-12-23%20at%2016.54.41.png?alt=media&token=e7ef4f28-12a9-40f4-b3c5-da37edafe96c)

如果您不需要删除工作区，请跳过此部分。 删除工作区将清除所有内容，并需要您重新开始。

完成`settings`页面上的所有部分后，您可以开始运行您的节点

## 运行节点
您可以从 `Humanode` 或`log`界面管理节点。

我们建议您从 `Humanode` 界面启动节点，然后到`log`界面查看同步是否完成。
### 运行并同步节点
完成设置后，您可以查看 `Humanode` 界面并点击`Start the node`。 如果有请求出现，您需要同意它们。
![](https://desktop-app-docs.humanode.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FLkvcOh8zYczeTBsQab1j%2Fuploads%2FVZ9cqUg32vmImGyaXPzI%2FScreenshot%202021-12-23%20at%2016.55.32.png?alt=media&token=39d77cd2-c1a9-4663-8fda-5d895b6473a6)

所有右上角的指示灯应变为绿色，并会出现 bioauth 注册和身份验证的二维码。
![](https://desktop-app-docs.humanode.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FLkvcOh8zYczeTBsQab1j%2Fuploads%2Fcb7Qpf03IaOsWLQ57RIY%2FScreenshot%202021-12-23%20at%2016.56.48.png?alt=media&token=8e316f3c-520f-4c0a-9747-f815f4aa5a18)

完成后，一切将如上所示，请转到`Log`界面并等待同步完成。 这可能需要一段时间。

![](https://desktop-app-docs.humanode.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FLkvcOh8zYczeTBsQab1j%2Fuploads%2Fr0S8SN6COIehm6hduq7v%2FScreenshot%202021-12-23%20at%2016.57.26.png?alt=media&token=58779bd9-1b5b-4c75-a61b-1afbf88444d1)

如果您在同步完成之前扫描您的脸，您将收到错误消息。

同步完成后，您可以开始`Bioauth enrollment`或`Authentication`。 日志通常显示如下：
```
2021-08-24 17:26:48 💤 Idle (3 peers), best: #11322 (0x0787…f440), finalized #0 (0x8f13…e745), ⬇ 0.3kiB/s ⬆ 0.2kiB/s    
2021-08-24 17:26:53 💤 Idle (3 peers), best: #11322 (0x0787…f440), finalized #0 (0x8f13…e745), ⬇ 0 ⬆ 0    
2021-08-24 17:26:54 ✨ Imported #11323 (0x2591…a139)    
2021-08-24 17:26:58 💤 Idle (3 peers), best: #11323 (0x2591…a139), finalized #0 (0x8f13…e745), ⬇ 0.4kiB/s ⬆ 0.2kiB/s    
2021-08-24 17:27:00 ✨ Imported #11324 (0x21f4…6314)    
2021-08-24 17:27:03 💤 Idle (3 peers), best: #11324 (0x21f4…6314), finalized #0 (0x8f13…e745), ⬇ 0.3kiB/s ⬆ 0.2kiB/s    
2021-08-24 17:27:06 ✨ Imported #11325 (0x699b…82f2)    
2021-08-24 17:27:08 💤 Idle (3 peers), best: #11325 (0x699b…82f2), finalized #0 (0x8f13…e745), ⬇ 0.3kiB/s ⬆ 0.4kiB/s    
2021-08-24 17:27:12 ✨ Imported #11326 (0x9e13…1d61)    
2021-08-24 17:27:13 💤 Idle (3 peers), best: #11326 (0x9e13…1d61), finalized #0 (0x8f13…e745), ⬇ 0.4kiB/s ⬆ 0.4kiB/s    
2021-08-24 17:27:18 ✨ Imported #11327 (0xe92e…3eca)
```
### Enrollment 和 Authentication （通常是第一次运行节点）
同步完成后，您可以开始`Bioauth enrollment`（如果您是第一次运行 Humanode）或身份验证（如果您之前已经运行过 Humanode）。
可通过扫描`Humanode`页面上的二维码进行注册和认证。
![](https://desktop-app-docs.humanode.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FLkvcOh8zYczeTBsQab1j%2Fuploads%2FsabmTDWajZ6LtkVzaWSE%2FScreenshot%202021-12-23%20at%2016.58.41.png?alt=media&token=c53176e6-158e-48e6-b4eb-f371e88ca451)

如果您启用了`Bioauth enrollment mode`，请执行以下步骤：

扫描二维码来开始`Bioauth enrollment`。 一旦在日志中显示`successful`，请再次扫描二维码进行身份验证。

节点将在区块链完成验证，并准备好生成和验证区块。
现在你的`Log`界面输出应该是这样的：
```
2021-08-21 18:25:01 Bioauth flow - authentication complete
2021-08-21 18:25:01 We've obtained an auth ticket auth_ticket=[128, ..., 0]
2021-08-21 18:25:02 💤 Idle (3 peers), best: #10 (0x4f96…fdec), finalized #0 (0x8f13…e745), ⬇ 0.7kiB/s ⬆ 0.3kiB/s    
2021-08-21 18:25:06 ✨ Imported #11 (0x5594…e8a0)    
2021-08-21 18:25:07 💤 Idle (3 peers), best: #11 (0x5594…e8a0), finalized #0 (0x8f13…e745), ⬇ 0.4kiB/s ⬆ 0.2kiB/s    
2021-08-21 18:25:12 💤 Idle (3 peers), best: #11 (0x5594…e8a0), finalized #0 (0x8f13…e745), ⬇ 0 ⬆ 0    
2021-08-21 18:25:12 ✨ Imported #12 (0x5ed2…da60)    
2021-08-21 18:25:17 💤 Idle (3 peers), best: #12 (0x5ed2…da60), finalized #0 (0x8f13…e745), ⬇ 0.4kiB/s ⬆ 0.2kiB/s    
2021-08-21 18:25:18 🙌 Starting consensus session on top of parent 0x5ed206762d36e0a47bffa4294d47bd9f4d85d959ced8d385017983ab8a8fda60    
2021-08-21 18:25:18 🎁 Prepared block for proposing at 13 [hash: 0xa5396adaad72e875a61d71efe64984684e81b14e17a8f2892a1b6240b35c8600; parent_hash: 0x5ed2…da60; extrinsics (1): [0x4240…c102]]    
2021-08-21 18:25:18 🔖 Pre-sealed block for proposal at 13. Hash now 0xa0ab5f3df7dc82a378b9206712b01f0c04ef1d08076626457aedb4c24479911a, previously 0xa5396adaad72e875a61d71efe64984684e81b14e17a8f2892a1b6240b35c8600.    
2021-08-21 18:25:18 ✨ Imported #13 (0xa0ab…911a)    
2021-08-21 18:25:22 💤 Idle (3 peers), best: #13 (0xa0ab…911a), finalized #0 (0x8f13…e745), ⬇ 0.1kiB/s ⬆ 0.4kiB/s    
2021-08-21 18:25:24 ✨ Imported #14 (0xd65c…c7f1)
```
你应该会看到`Starting consensus session on top of parent...`

恭喜，您的 Humanode 节点已加入 Humanode 测试网并生成区块！ 你是银河系中最早的 human nodes 之一！



### Re-authentication

为了防止额外的恶意活动，Humanode 使用`authentication tickets`的概念来允许节点参与区块生产的共识。 与humanode每个月都必须通过生物认证的主网不同，在第一个测试网中，每张`authentication tickets`在通过生物认证程序后的 72 小时内有效。 因此，您需要再次通过生物认证程序获得新的`authentication tickets`，以继续参与共识。 以下`Log`界面输出表明您的`authentication tickets`已过期。

```
2021-08-24 17:26:48 💤 Idle (3 peers), best: #11322 (0x0787…f440), finalized #0 (0x8f13…e745), ⬇ 0.3kiB/s ⬆ 0.2kiB/s    
2021-08-24 17:26:53 💤 Idle (3 peers), best: #11322 (0x0787…f440), finalized #0 (0x8f13…e745), ⬇ 0 ⬆ 0    
2021-08-24 17:26:54 ✨ Imported #11323 (0x2591…a139)    
2021-08-24 17:26:58 💤 Idle (3 peers), best: #11323 (0x2591…a139), finalized #0 (0x8f13…e745), ⬇ 0.4kiB/s ⬆ 0.2kiB/s    
2021-08-24 17:27:00 ✨ Imported #11324 (0x21f4…6314)    
2021-08-24 17:27:03 💤 Idle (3 peers), best: #11324 (0x21f4…6314), finalized #0 (0x8f13…e745), ⬇ 0.3kiB/s ⬆ 0.2kiB/s    
2021-08-24 17:27:06 ✨ Imported #11325 (0x699b…82f2)    
2021-08-24 17:27:08 💤 Idle (3 peers), best: #11325 (0x699b…82f2), finalized #0 (0x8f13…e745), ⬇ 0.3kiB/s ⬆ 0.4kiB/s    
2021-08-24 17:27:12 ✨ Imported #11326 (0x9e13…1d61)    
2021-08-24 17:27:13 💤 Idle (3 peers), best: #11326 (0x9e13…1d61), finalized #0 (0x8f13…e745), ⬇ 0.4kiB/s ⬆ 0.4kiB/s    
2021-08-24 17:27:18 ✨ Imported #11327 (0xe92e…3eca)
```

现在你看不到`Starting consensus session on top of the parent...`

您需要到`Humanode` 界面并扫描二维码。重新进行人脸识别，以获取新的`authentication tickets`

![](https://desktop-app-docs.humanode.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FLkvcOh8zYczeTBsQab1j%2Fuploads%2Fcb7Qpf03IaOsWLQ57RIY%2FScreenshot%202021-12-23%20at%2016.56.48.png?alt=media&token=8e316f3c-520f-4c0a-9747-f815f4aa5a18)

## 停止你的节点
如果您需要停止节点，您可以使用 `Humanode` 界面和`Log`界面上的`Stop`按钮。 点击这个按钮将停止您的节点，并且您将不再充当网络中的验证者，直到您再次运行您的节点。

停止节点不会取消您的 bioauth。

**如果关闭Humanode Desktop App，你的节点将继续在后台运行。**





