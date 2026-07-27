# OpenPGP 的介绍和使用方法

 
- OpenPGP 是什么
```
# 基础定义
OpenPGP 是一套开放的加密标准（RFC 4880），基于 PGP（Pretty Good Privacy），实现非对称加密、数字签名。
日常使用的 GnuPG (GPG)、Kleopatra、Thunderbird 邮件加密都遵循 OpenPGP 规范。
```
  


**核心思想：一对密钥**
>**公钥**：公开分发，别人用来给你加密消息、验证你的签名
**私钥**：自己保管，用来解密消息、生成签名
>

**三大核心用途（对应你的硬件设备）**
>**1.数据 / 邮件加密**
他人使用你的公钥加密信息，只有你的私钥可以解密，防止内容窃听。
>**2.数字签名（最常用）**
对文件、Git 提交、邮件签名；其他人用公钥验证，确认内容没有被篡改、确认发送者身份。
>**3.身份认证**
用于 SSH 登录、软件包签名、代码提交鉴权。


**硬件密钥上的 OpenPGP**
>像你的密码管理器、YubiKey、Canokey 实现的 OpenPGP Card（智能卡协议）：
1.私钥生成并永久保存在硬件芯片内部，无法导出；
2.PC 通过 CCID / USB 智能卡协议和硬件通信；
3.硬件内部完成加密、签名运算，私钥永远不会传到电脑内存。

**标准 OpenPGP Card 固定 3 套子密钥**：
>1.签名密钥（Sign）：Git commit 签名、文件签名
2.加密密钥（Encrypt）：邮件、文件加密解密
3.认证密钥（Auth）：SSH 登录服务器
  
- OpenPGP 有什么优点
   
- OpenPGP 怎么用

  - PC软件使用方法
  
  - 使用方法


#1 接入 APDU 通道 esp32-p4 usb 实现 USB CCID 让电脑可以发现Smart Card Reader ✅

#2 SELECT OpenPGP Application Windows APDU测试工具
pcsc-tools-1.6.2_windows-x64.zip
解压后直接运行 pcsc_scan.exe


