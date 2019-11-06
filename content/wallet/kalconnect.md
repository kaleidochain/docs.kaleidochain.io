---
title: "使用KALconnect安全钱包"
weight: 1
---

KALconnect是一个无需硬件的“硬件钱包”，并且整个项目代码完全开源，兼具硬件钱包级别的安全性和普通钱包的便捷性。

要使用KALconnect安全钱包，首先需要[下载安装KALconnect钱包APP](https://kaleidochain.io/zh/download-wallet/)。

安装后，打开APP，即可创建一个全新的KAL钱包。KALconnect采用HD派生钱包技术，默认使用Kaleido主网路径`m/44'/383'/0'/0`派生的第一个钱包地址。

<img alt="KALconnect Main Page" src="/images/kalconnect-main.png" height="400px"/>

### 使用KALconnect安全钱包

KALconnect使用了密钥存储和钱包功能界面物理分离的安全技术，安全性媲美硬件钱包。因此，使用时配合使用浏览器打开钱包功能网站`https://kallet.kaleidochain.io`，使用KALconnect安全钱包扫描建立本地的安全传输通道，相当于在手机APP和浏览器页面直接连接了一个虚拟的USB线。之后的所有操作，都从网页发起，并经由手机APP确认并签名后，发回网页进行处理。相当于每一笔交易都由两个独立设备分别进行两次确认，安全性非常高。

具体操作演示如下：

在浏览器中打开`https://kallet.kaleidochain.io`，点击`Access My Wallet`并选择`KALconnect`，会弹出一个二维码。

<img alt="KALwallet QRcode" src="/images/kallet-qrcode.png" height="400px"/>

在KALconnect钱包中，点击 *SCAN TO CONNECT* 进行扫描。此时，手机会和浏览器页面建立一个安全的加密P2P连接，连接成功后会返回主界面，而浏览器会进入操作功能界面。

<img alt="KALwallet QRcode" src="/images/kalconnect-scan.jpg" height="400px"/>
<img alt="KALwallet QRcode" src="/images/kalconnect-scan-connecting.png" height="400px"/>
<img alt="KALwallet QRcode" src="/images/kalconnect-scan-complete.png" height="400px"/>
<img alt="KALwallet QRcode" src="/images/kalconnect-testnet-connected.png" height="400px"/>


接下来就可以自由转账或调用合约了。如果你现在还没有主网KAL，可以线在测试网免费领取测试KAL进行体验。


### 在测试网体验转账


KALconnect会使用测试网路径`m/44'/1'/0'/0`派生的第一个钱包地址用于测试网使用，我们点击最上方的网络切换到测试网即可看到。

<img alt="KALwallet QRcode" src="/images/kalconnect-switch-network.png" height="400px"/>
<img alt="KALwallet QRcode" src="/images/kalconnect-testnet.png" height="400px"/>

我们可以在测试网的Token龙头中免费领取几个测试KAL。

<img alt="" src="/images/faucet.png" height="400px"/>

稍等3秒左右，你又可以通过刷新看到测试KAL到账。

接下来，使用手机与浏览器进行连接，进入功能界面。此时，还要在浏览器中切换到测试网中。

<img alt="" src="/images/kallet-change-network.png" height="400px"/>

因为我们有两个独立的设备：手机APP和浏览器。所以，都要切换到测试网络才可以。

接下来，即可发起一个转账。例如给地址`0xabc`转账，并经过两次确认后，转账成功。


<img alt="" src="/images/kallet-confirmation.png" height="400px"/>
<img alt="" src="/images/kalconnect-verify-txn.png" height="400px"/>
<img alt="" src="/images/kalconnect-verify-txn-checked.png" height="400px"/>
<img alt="" src="/images/kalconnect-verify-txn-done.png" height="400px"/>
<img alt="" src="/images/kallet-confirmation-done.png" height="400px"/>

完成后，你可以在浏览器中查看刚才的交易详情。

<img alt="" src="/images/kallet-history.png" height="400px"/>

