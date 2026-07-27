# fido2 介绍和使用方法
 

- fido2 是什么
  
- fido2 有什么优点
   
- fido2 怎么用

  - 使用方法
 



```js
=========================================================
✅ 注册流程（本质）
浏览器（网站）：给设备发一个注册请求（带随机数）
设备（你的 nRF52840 / MeXkey3）：
生成一对密钥：公钥 + 私钥
私钥自己锁死，绝不外传
把公钥发给浏览器
浏览器：把公钥上传给网站服务器保存
你说的完全对：设备给浏览器一个公钥。
=========================================================


=========================================================
✅ 登录流程（本质）
浏览器（网站）：给设备发一个登录请求（带新随机数）
设备：
找到对应私钥
用私钥对随机数签名
把签名发给浏览器
浏览器：把签名传给服务器
服务器：
用注册时保存的公钥验证签名
验签成功 = 登录成功
你说的完全对：设备给签名，公钥验签，成功就登录。
=========================================================


=========================================================
🎯 FIDO2
FIDO2 = 注册时存公钥，登录时用私钥签名，服务器用公钥验签。
私钥永不触网，绝对安全。
🔍 只有 3 个小东西（你必须知道）
为了严谨，我只补充 3 个细节（不影响你的核心理解）：
不是只签随机数，是签 域名 + 随机数 + 计数器（防钓鱼、防重放）
必须按键确认：证明 “人在场”，防止木马偷偷调用密钥
每个网站一对密钥：网站 A 看不到网站 B 的密钥，互相隔离
✅ 最终结论（你完全理解对了）
你对 FIDO2 的理解已经 100% 正确、到位、通透！
注册 = 给公钥
登录 = 给签名
验证 = 公钥验签
安全 = 私钥永远不出设备
这就是 FIDO2 全部的核心灵魂，剩下的都是协议包装、USB 传输、CBOR 打包这些 “外壳”。
=========================================================
```

```
一、先给你最直观的比喻（秒懂）
你可以把：
私钥 k = 你的签名笔
公钥 Q = 你的签名笔迹样本
(r,s) = 你这次签的名字
服务器做的事情就是：
** 拿着你注册时留下的 “笔迹样本（公钥）”
对比你这次签的名字（r,s）
像不像 → 像 = 合法，不像 = 伪造 **
它不需要知道你笔是怎么握的（私钥）
只需要对比笔迹（验签）
二、真实数学逻辑（超级通俗）
服务器手里只有 3 样东西：
公钥 Q（注册时保存）
签名 (r,s)（登录时你给的）
被签名的数据 e（服务器自己算的）
它要验证一句话：
** 这个 (r,s) 必须满足：
s 包含了 k
但我没有 k
但我有 Q = k × G
所以我可以用 Q 反向验证！**
三、服务器验签真正做的 4 步（人话版）
第 1 步：服务器算 w = 1/s （模逆元）
相当于数学上的 倒数
第 2 步：算两个中间值
plaintext
u1 = e × w
u2 = r × w
第 3 步：最关键一步（灵魂）
plaintext
P = u1 × G + u2 × Q
因为 Q = k × G
所以代入后：
plaintext
P = u1×G + u2×k×G
G 可以提出来
这就变成了：
plaintext
P = (u1 + u2 × k) × G
```
---------------------------------------


```
四、最关键结论来了！
如果 (r,s) 是真的，用私钥 k 签出来的
那么上面算出来的点 P 的 x 坐标 一定等于 r
如果 (r,s) 是假的
x 坐标 绝对不可能等于 r
```
===========================================
```
五、最终判断（服务器只看这一句）
plaintext
P.x  mod n  ==  r
相等 → ✅ 签名合法
不等 → ❌ 伪造 / 错误
六、我用人话再总结一次（你 100% 能懂）
服务器根本不需要私钥 k
它只需要公钥 Q（k 的影子）
它把 (r,s) 代进公式
如果算出来的点 P 的 x 正好等于 r
就证明：
这个签名一定是用 k 签出来的！
七、最精简终极版（背住就够）
服务器验签 =
用公钥 Q 验证 (r,s) 是否满足椭圆曲线数学关系
满足 = 真
不满足 = 假
如果你愿意，我可以给你：
一组真实数字（真实 k、真实 Q、真实 r、s）
我一步一步手算给你看，
让你亲眼看到：
服务器是怎么用 Q 验证 (r,s) 的！
```


# 一、回顾签名阶段核心公式设备端用私钥 k、临时随机数 d、消息哈希 e 计算：

基点倍乘：

$$\(D = d\cdot G\)$$，

取横坐标 $$\(r = D.x\)$$

签名后半段：

$$\(s = d^{-1}\cdot \big(e + r\cdot k\big)\)$$

变形（两边同乘 d）：
 


$$ s\cdot d = e + r\cdot k $$
 
# 二、引入验签的核心目标服务器已知：

G（公共基点）、公钥 $$\(Q=k\cdot G\)、e、\(r、s\)$$
目标：构造出点 \(D = d\cdot G\)，因为合法签名必然满足 \(D.x = r\)。把公式 (1) 做数学变形，解出 \(d\cdot G\)：
公式 (1) 两边同乘 \(s^{-1}\)（记 \(w = s^{-1}\)，也就是你看到的「模逆 / 倒数」）：
\(d = w\cdot e + w\cdot r\cdot k\)

 等式两边同时左乘基点 G（椭圆曲线点乘运算）：
 
$$d\cdot G = \big(w\cdot e\big)\cdot G + \big(w\cdot r\cdot k\big)\cdot G$$

$$
\(d\cdot G = \big(w\cdot e\big)\cdot G \;+\; \big(w\cdot r\cdot k\big)\cdot G\)
$$

 拆分右侧、代入公钥 \(Q=k\cdot G\)：
 
$$d\cdot G = \big(e\cdot w\big)\cdot G \;+\; \big(r\cdot w\big)\cdot \big(k\cdot G\big)\$$

# 三、定义 \(u_1、u_2\)，对应含义现在直接定义：

$$\(\boldsymbol{u_1 = e \cdot w},\quad \boldsymbol{u_2 = r \cdot w}\)$$

代入上式，得到：

$$\(\boldsymbol{d\cdot G \;=\; u_1\cdot G \;+\; u_2\cdot Q}\)$$

逐句解释含义

$$\(w = s^{-1}\)$$

就是 s 在模 n 下的倒数，作用是把签名公式做等价变形，打通「签名」和「验签」的数学关系。


$$\(u_1 = e \times w\)$$

是对消息哈希 e 做加权系数，对应公式里「e 相关项」。


$$\(u_2 = r \times w\)$$

是对签名分量 r 做加权系数，对应公式里「私钥 k 相关项」（因为这一项最终会和公钥 Q 绑定）。


右侧

$$\(u_1\cdot G + u_2\cdot Q\)$$

这就是服务器最终计算的点 P：

$$\(P = u_1\cdot G + u_2\cdot Q\)$$

结合上面推导可得：

$$\(P = d\cdot G\)$$

# 四、闭环：回到最终验签判断签名阶段我们规定：r 是 $$\(D=d\cdot G\)$$ 的横坐标。

所以合法签名必须满足：

$$\(P.x = (d\cdot G).x = r\)$$

整串逻辑串联（完整因果）

从签名公式推导出：

$$\(d\cdot G = u_1 G + u_2 Q\)$$；

服务器没有 $$\(d、k\)$$，
但有 $$\(G、Q、e、r、s\)$$，
所以算出 $$\(w、u_1、u_2\)$$；
用 $$\(u_1、u_2\)$$ 拼出点 P，
等价还原出设备端的 $$\(d\cdot G\)$$；
对比 P 的横坐标和 r，相等则签名合法。

# 五、一句话通俗总结

w 是 s 的倒数，用来反转签名公式；
$$\(u_1、u_2\)$$ 是两个加权系数，分别对应「消息哈希」和「签名r」；
服务器用这两个系数，搭配公开的 G 和 Q，数学等价还原出设备端的临时点 $$\(d\cdot G\)$$；
最后用横坐标比对，完成验签。
整个过程没有用到私钥 k，完全依靠公私钥的数学绑定关系，这也是 ECDSA 安全的核心。

===========================================================================================================














```
===========================================================================================================
一句话核心答案
FIDO2 默认使用的是：
椭圆曲线密码学（ECC），具体曲线是：secp256r1（也叫 NIST P-256）
公钥 + 私钥 全部由 ECC 椭圆曲线算法生成
签名用 ECDSA（椭圆曲线数字签名算法）
验签也用 ECDSA
-----------------------------------------------------------------------------------------------------------
一、这对密钥到底是什么？（底层原理）
1. 私钥（private key）
就是一个 32 字节的随机数
由硬件真随机数发生器（TRNG）生成
永远保存在设备里，绝对不输出
底层数学意义：椭圆曲线上的一个随机整数 k
-----------------------------------------------------------------------------------------------------------
2. 公钥（public key）
由私钥计算出来：
公钥 = 私钥 × 椭圆曲线基点 G
长度 65 字节（未压缩格式）
可以公开，不影响安全
底层数学意义：椭圆曲线上的一个点 (x, y)
------------------------------------------------------------------------------------------------------------
二、生成密钥的底层流程（你设备里真实发生的）
plaintext
设备真随机数生成器（TRNG）
       ↓
生成 32 字节随机数 → 这就是【私钥】
       ↓
用椭圆曲线公式 P = k × G
       ↓
得到椭圆曲线上的点 P → 这就是【公钥】
没有其他算法！
没有 RSA！
没有 DH！
纯 ECC！
-------------------------------------------------------------------------------------------------------------
三、签名用什么原理？（登录时）
登录时设备做的签名，用的是：
ECDSA-SHA256
流程：
plaintext
待签名数据（challenge + rp_id + 计数器）
          ↓ SHA-256 哈希
32 字节哈希值
          ↓ ECDSA 签名（用私钥）
生成签名（r,s）→ 发给浏览器
签名只能用私钥生成。
--------------------------------------------------------------------------------------------------------------


四、验签用什么原理？（服务器）
服务器用公钥做：
ECDSA 验签
plaintext
收到：签名 + 原始数据
用公钥验证：签名是否由私钥签发
只要公钥能解开，就证明一定是对应私钥签的。

--------------------------------------------------------------------------------------------------------------

五、FIDO2 为什么一定要用 ECC（secp256r1）？
你只需要记住 3 个优点：
短：密钥只有 32 字节，适合嵌入式设备（nRF52840）
快：签名 / 验签速度极快
安全：256 位 ECC ≈ RSA 3072 位安全性
FIDO2 强制必须支持 ES256（ECDSA secp256r1）
其他算法都是可选的。
六、你设备（MeXkey3 /nRF52840 / CanoKey）真实调用的函数
plaintext
ecdsa256_gen_key()  → 生成密钥对
ecdsa256_sign()     → 私钥签名
底层就是：ECC secp256r1
七、我给你最精简的终极总结（背住就够了）
FIDO2 密钥底层 = ECC 椭圆曲线（secp256r1）
私钥 = 32 字节随机数
公钥 = 私钥 × 曲线基点 G
签名 = ECDSA
验签 = ECDSA
===============================================================================================================
```




完全正确，整体流程梳理到位，再补充FIDO2 专属细节和精简链路，帮你把逻辑补完整：
```
一、注册流程（你总结的完全无误）
本地设备用真随机数生成私钥 k，本地安全存储（永不外传）。
基于标准基点 G，计算 \(\boldsymbol{Q = k\cdot G}\)，得到公钥点 Q。
把公钥 Q 提交给服务器，服务器永久绑定你的账号保存。
二、登录 & 验签流程
服务器生成随机挑战 challenge 下发，发起登录校验。
设备拼接规定数据，用本地私钥 k 执行 ECDSA 运算，生成签名 \((r,s)\)。
设备将签名、认证数据等一并传给服务器。
服务器动作：

还原被签名的原始数据，计算哈希 e；
取出你注册时保存的公钥 Q + 内置标准基点 G；
执行 ECDSA 验签算法；
校验通过 → 允许登录；校验失败 → 拒绝登录。


三、额外补充 2 个 FIDO2 关键安全点（和纯 ECDSA 签名的区别）
签名不是只对 challenge 计算，还会包含网站域名哈希、计数器，防钓鱼、防重放。
私钥 k 全程只在本地硬件 / 设备内使用，网络传输全程看不到私钥。
极简总链路注册：生成 k（本地存）→ 算 Q → Q 上交服务器
登录：k 生成签名 → 签名上传 → 服务器用 Q 验签 → 判定结果
```


```
二、FIDO2 顶层对外 API（共 13 个）
1）初始化 / 重置（3）
fido2_init()
功能：初始化 FIDO2 上下文、密钥存储区、状态机、随机数种子
fido2_reset()
功能：重置 FIDO2 所有状态，清除当前会话、PIN 状态、认证状态
fido2_factory_reset()
功能：工厂级重置，删除所有 FIDO2 凭证、重置 PIN、恢复出厂默认

2）APDU 入口 / 分发（2）
fido2_process_apdu()
功能：FIDO2 主入口，接收 USB/NFC APDU，分发到 CTAP2/U2F 命令处理
fido2_select_applet()
功能：响应 SELECT AID，判断并激活 FIDO2 应用（AID：A0000006472F0001）

3）CTAP2 核心命令（8）
ctap2_get_info()
功能：返回认证器元信息（版本、算法、支持扩展、最大 resident key 数 = 64）
ctap2_make_credential()
功能：注册（建凭证）：生成 P256/Ed25519/SM2 密钥对、存储公钥 + 用户信息、返回 attestation
ctap2_get_assertion()
功能：认证（登录）：校验 RP ID、查找密钥、用户确认、签名 challenge、返回 assertion
ctap2_get_next_assertion()
功能：多凭证场景：返回下一个匹配的 assertion（多账号同站点）
ctap2_cancel()
功能：取消当前 pending 的认证 / 注册操作
ctap2_pin_cmd()
功能：PIN 协议处理：设置 / 修改 / 验证 PIN、获取 PIN token、管理 PIN 重试计数（最多 3 次）
ctap2_credential_management()
功能：凭证管理：枚举 / 删除 / 更新 resident key（最多 64 个）
ctap2_hmac_secret()
功能：HMAC-secret 扩展（用于 Web Crypto、SSH、PAM）
```





```
==============================================================================
结论：完全可以纯原生 Python，不依赖任何第三方库，从零实现整套 FIDO2/WebAuthn 底层全流程
只使用 Python 标准库：
os, hashlib, hmac, struct, json, base64, binascii, io
椭圆曲线签名（ECDSA P-256）、CBOR、COSE、Base64url、Attestation、AuthData、签名验签全部手写，不装 cryptography/webauthn/fido2。
一、需要手动实现的所有底层模块（无第三方库）
1. 基础编码层（WebAuthn 强制）
Base64url：去掉填充、+→-、/→_，标准库 base64 改造
CBOR 编解码器：FIDO2 AuthenticatorData / AttestationObject 全部用 CBOR 打包，手写极简 CBOR（只支持 FIDO 用到的类型：uint、bytes、array、map）
COSE 密钥序列化：把 P-256 公钥编码成 COSE EC2 结构（CBOR map）
2. 密码学核心（最难点，无 cryptography）
secp256r1 (NIST P-256) 椭圆曲线：
曲线参数硬编码（p,a,b,Gx,Gy,n,h）
大整数模运算、点加、点倍乘
私钥生成 → 公钥点导出
ECDSA 签名（SHA256）：签名生成、签名校验
SHA-256：标准库 hashlib 原生支持，不用自己写哈希
3. FIDO2 协议结构层
AuthenticatorData 二进制结构（固定字节布局）
rpIdHash (32) + flags (1) + counter (4) + attestedCredData (可选) + extensions (可选)
Attested Credential Data 二进制打包
ClientDataJSON：JSON → UTF8 字节
Registration：
MakeCredential 完整流程、自签名 Basic Attestation 模拟
Authentication：
GetAssertion，用凭证私钥对 (authData || clientDataHash) 签名
依赖方校验逻辑：验签名、校验挑战、校验 rpIdHash、校验计数器防重放
4. 存储层
纯文件 JSON 存储凭证 ID、私钥整数、公钥点坐标，无外部数据库
```

```
二、仅有的 Python 内置标准库清单（无任何 pip 安装）
python
运行
import os       # 生成安全随机数（os.urandom）
import hashlib  # sha256
import hmac
import struct   # 打包4字节计数器大端uint32
import json     # ClientDataJSON序列化
import base64   # 改造base64url
import binascii
所有密码学、编码逻辑全部手写，无外部依赖。
三、整体流程拆分（纯手写底层执行顺序）
注册流程（从零走完）
RP 生成随机 challenge（os.urandom (32)）
客户端构造 ClientData：{"type":"webauthn.create","challenge":base64url,"origin":"http://localhost"}
手写 P256 代码生成私钥 d，算出公钥 (Qx, Qy)
构造 AuthenticatorData：rpIdHash=sha256 (rp_id) + flags (AT+UV) + counter=0 + attested_cred_data
attested_cred_data 打包：aaguid (16) + credIdLen (2) + credId + COSE 公钥
拼接 auth_data_bytes
计算 client_data_hash = sha256 (client_data_json_bytes)
签名原文 = auth_data_bytes + client_data_hash
用私钥对原文做 ECDSA-SHA256 签名 (r,s)
组装 AttestationObject CBOR map：fmt="basic", attStmt={sig: 签名字节}, authData=auth_data_bytes
打包成 CBOR 二进制，封装成注册返回对象
RP 侧底层校验：
解析 CBOR 取出 authData、签名
提取 COSE 公钥 Qx/Qy
重算 client_data_hash、rpIdHash
手写 ECDSA 校验签名合法性
认证登录流程
RP 下发新 challenge、允许的 credential ID 列表
模拟器匹配本地存储凭证，取出对应 P256 私钥
生成新 AuthenticatorData（无 attested 数据，仅 rpIdHash+flags+counter 自增）
构造 ClientDataJSON 类型为 webauthn.get
拼接签名报文 authData || sha256 (clientData)
私钥生成 ECDSA 签名
组装 CBOR Assertion 返回
RP 底层验签，校验 challenge、origin、计数器递增防重放
四、核心难点说明（纯手写无库）
P-256 ECDSA 全手写
需要实现模逆、椭圆点加法 / 倍乘、DER 签名编解码，大量大整数运算；Python 原生支持无限大整数，刚好适合。
极简 CBOR 实现
FIDO 只用少量 CBOR 类型，不需要完整标准，只实现：
unsigned int
byte string
array
map
舍弃浮点数、负数、tag、字符串等无关类型，代码量可控。
COSE EC2 结构手动映射
COSE key 是 CBOR map，键值固定：
1: kty=2 (EC2)
-1: crv=1 (P-256)
-2: x (Qx bytes)
-3: y (Qy bytes)
手动组装，不用任何 COSE 库。
五、可行性总结
理论 100% 可行：全部规范都是公开二进制 / 数学标准，无闭源算法；
运行环境限制：只需要原生 Python3，不用 pip 装任何包；
功能缺失仅为硬件特性：
无真实 PIN / 指纹 UV（只能 flag 标记）
无安全硬件 SE，私钥存在内存 / 文件可导出
无厂商 X.509 证书链 attestation，只能基础自签 attestation
协议逻辑、二进制编码、签名验签和真实 FIDO2 设备完全一致。
六、下一步
如果你需要，我可以分阶段给出完整无第三方库源码：
第一部分：Base64url + 极简 CBOR 编解码器
第二部分：手写 NIST P-256 椭圆曲线 + ECDSA 签名 / 验签
第三部分：AuthenticatorData、COSE 公钥打包工具
第四部分：完整注册（MakeCredential）底层流程
第五部分：完整认证（GetAssertion）底层流程
整套代码仅依赖 Python 标准库。
```


```
===============================================================

五、标准文档学习顺序（从零手写底层推荐）

WebAuthn Level2（先看懂前后端交互、校验规则）

CTAP2.1 协议（看懂验证器内部二进制命令、CBOR 数据包）

COSE 规范（EC2 公钥序列化格式）

CBOR RFC7049（仅实现 FIDO 用到的子集）

NIST P-256 & ECDSA-SHA256 RFC 标准（椭圆曲线签名数学逻辑）

================================================================

网页 JS (WebAuthn) → 浏览器内核 
→ OS 系统 HID 驱动 → USB CTAPHID 报文 
→ CanoKey USB 固件 → MCU CTAP2 协议栈 → 加密引擎 + Flash 存储
===================================================

---------------------------------------------------
1.上层：Web 前端 WebAuthn W3C API（JS）
---------------------------------------------------

---------------------------------------------------
2.中间层：浏览器内核 CTAP 客户端 + OS USB HID驱动栈（libfido2/hidapi）
---------------------------------------------------

---------------------------------------------------
3.传输层：CTAPHID USB 中断报文（64 字节 HID 报告、CID 会话、分片重组）
---------------------------------------------------

---------------------------------------------------
4设备层：CanoKey MCU 固件四层（USB 驱动 → CTAPHID 传输层 → CTAP2 命令解析 → 加密 / 存储）
---------------------------------------------------
```
```
===================================================
一、FIDO2 注册流程：navigator.credentials.create () 新建凭证
===================================================

阶段 1：网站前端 JS WebAuthn 调用（最上层）
网站组装公钥凭证创建参数 publicKeyCredentialCreationOptions，调用标准 W3C 接口
前端输出数据
JSON 结构化 WebAuthn 请求 → 浏览器内核转为CTAP2 CBOR 二进制载荷
-------------------------------------------------------------------

阶段 2：浏览器内核 & OS 系统层（libfido2）
Chrome/Firefox 内置 libfido2 库，处理 CTAP 会话、USB 设备枚举
内核关键函数（libfido2）
1.fido_dev_open()：枚举 USB HID 设备，匹配 UsagePage=0xF1D0 (FIDO 专用 HID)，打开 CanoKey 设备
2.ctaphid_init()：发送 CTAPHID_INIT 报文，分配 3 字节 CID 会话 ID（建立主机 - 密钥通信通道）
3.fido_dev_make_cred()：封装 CTAP2 makeCredential CBOR 指令
4.ctaphid_write()：将 CBOR 载荷分包为 64 字节 USB HID 报告下发 MCU
5.ctaphid_read()：循环读取设备返回分片报文，重组完整 CBOR 响应
6.fido_dev_close()：释放 USB 设备句柄

OS 底层系统调用（Windows/macOS/Linux）
Windows：WinHID.dll → HidD_GetPreparsedData、WriteFile/ReadFile 中断传输
Linux：hidraw 字符设备 /dev/hidrawX，read()/write()
macOS：IOKit HID Manager IOHIDDeviceSetReport
-------------------------------------------------------------------

阶段 3：USB CTAPHID 报文传输（主机 ↔ CanoKey MCU）
TAPHID 标准帧头固定 4 字节：CID[3] + CMD[1]，单 HID 报告 64 字节，有效载荷 60 字节，超长 CBOR 自动分片
1.INIT 帧：主机下发 CTAPHID_INIT，设备返回 CID（会话通道）
2.MSG 首包：CTAPHID_MSG，头 4 字节 + 载荷长度 (2B) + CBOR 起始数据
3.MSG 续包：载荷超过 60 字节，后续分片无长度字段，仅追加数据
4.CanoKey MCU 收到完整 CBOR 载荷后，处理makeCredential命令，生成响应报文分片回传主机
-------------------------------------------------------------------
===================================================================
阶段 4：CanoKey MCU 固件四层处理（STM32 / 安全芯片）
固件分层（源码 canokey-core）：USB 驱动层 → CTAPHID 传输层 → CTAP2 应用层 → 密码 / 存储底层

4.1 USB 驱动层（usbd_fido.c）
底层中断端点接收 64 字节 HID 报告
核心函数：
usbd_fido_DataOut()：USB 中断 OUT 端点接收 HID 报告，存入环形缓冲区
usbd_fido_SendReport()：CTAP 响应打包为 HID IN 报告，上传 PC

4.2 CTAPHID 传输层（ctaphid.c）
分片重组、会话 CID 管理、报文校验
核心函数：
ctaphid_process_packet()：解析 4 字节帧头，区分 INIT/MSG/WINK/ERROR 指令
ctaphid_assemble_msg()：合并多片 HID 数据，还原完整 CTAP CBOR 消息
ctaphid_send_response()：将 CTAP2 响应拆分为多片 HID 报文发送


4.3 CTAP2 命令解析层（ctap2.c/fido_applet.c）
收到完整 CBOR 载荷，解析makeCredential指令
核心函数：
1.ctap2_handle_command()：入口总分发函数，匹配 CMD 码
	CTAP_CMD_MAKE_CREDENTIAL = 0x01 进入注册逻辑
2.ctap2_make_credential()：注册主逻辑函数
	校验 rpID、用户参数、算法（ES256/ED2556/SM2）
	触发用户验证：LED 闪烁、等待按键确认（user presence）
	如需 PIN 校验：调用ctap2_pin_verify()解密会话密钥，校验 PIN 码
3.credential_create()：生成全新密钥对
	调用加密层crypto_ec_keygen()生成 ES256 公私钥
	生成唯一credentialID（随机 16~32 字节）
4.credential_store()：持久化凭证到 Flash LittleFS
	存储字段：rpID、credID、公钥、私钥、用户信息、创建时间、resident 标记
5.ctap2_attest()：生成设备认证对象（attestationObject），含公钥、设备证书签名


4.4 加密 & 存储底层
1.密码模块 crypto/ec.c：crypto_ec_sign()、crypto_ec_keygen()
2.Flash 存储 storage/lfs.c：凭证持久化读写、resident 常驻密钥分区管理
3.状态机：按键检测、LED 提示、超时自动关闭会话
-------------------------------------------------------------------



阶段 5：原路返回数据（MCU → USB → 浏览器 → 网站服务端）
1.MCU 组装 CBOR 响应：attestationObject + clientDataJSON
2.CTAPHID 分片打包为 HID IN 报告上传 PC
3.libfido2 重组报文，向上层 JS 返回PublicKeyCredential对象
4.网站 JS 提取attestationObject、公钥，提交后端数据库保存公钥，完成绑定注册
-------------------------------------------------------------------

===================================================
二、FIDO2 登录流程：navigator.credentials.get () 身份认证
===================================================
阶段 1：网站前端 JS WebAuthn 调用
1.allowCredentials：该账号已注册的 credID 列表，浏览器传递给密钥匹配凭证
2.challenge：网站本次登录随机挑战
差异点：注册是新建密钥，登录是查找已有私钥签名挑战
-------------------------------------------------------------------


阶段 2：浏览器 & libfido2 系统层（函数与注册大部分复用）
共用函数：fido_dev_open()、ctaphid_init()、ctaphid_read/write()
专属登录函数：
1.fido_dev_get_assert()：封装 CTAP2 authenticatorGetAssertion CBOR 指令
2.若使用 resident key（无用户名登录）：不传 credID，让设备遍历本地存储匹配 rpID
-------------------------------------------------------------------


阶段 3：CTAPHID USB 报文传输（和注册传输逻辑完全一致）
下发CTAPHID_MSG携带authenticatorGetAssertion CBOR 载荷，分片收发无区别
-------------------------------------------------------------------


阶段 4：CanoKey MCU 固件登录处理（核心差异在 CTAP2 命令）
4.1 USB/CTAPHID 传输层：函数完全复用注册流程
usbd_fido_DataOut() → ctaphid_process_packet() → ctaphid_assemble_msg()

4.2 CTAP2 命令分发层（ctap2.c）
1.ctap2_handle_command() 匹配 CTAP_CMD_GET_ASSERTION = 0x02 进入登录逻辑
2.ctap2_get_assertion() 登录主处理函数
	1.遍历 Flash 存储凭证：
		普通密钥：匹配传入allowCredentials内的 credID
		Resident 常驻密钥：匹配 rpID，返回全部对应凭证供浏览器选择账号
	2.用户验证：LED 闪烁等待按键；开启 PIN 则校验 PIN 会话密钥
	3.读取该凭证对应的不可导出私钥
	4.调用加密层对challenge+rp+userdata做 ES256 签名
3.crypto_ec_sign()：使用凭证私钥生成签名
4.组装 CBOR 响应：签名值、credID、用户信息、authData


4.3 存储 / 加密底层复用
	storage_credential_find()：按 credID/rpID 检索凭证
	crypto/ec.c 椭圆曲线签名运算
	LittleFS 读取预存私钥（硬件安全元件版本私钥不可读出，仅内部运算）
-------------------------------------------------------------------

阶段 5：返回链路校验
签名报文沿 USB 回传给浏览器，JS 拿到AuthenticatorAssertionResponse，网站后端用注册时保存的公钥校验签名合法性，登录成功。
-------------------------------------------------------------------

```
```
四、完整数据流向极简串联（注册示例）
网站 JS create () → WebAuthn JSON 参数 → 浏览器 libfido2 序列化为 CBOR → CTAPHID_INIT 建立 CID 会话 → CBOR 载荷分包为 64 字节 USB HID 报告 
→ USB 中断下发 STM32 → usbd_fido 接收报文 → ctaphid 重组完整 CBOR → ctap2_handle_command 分发 makeCredential 
→ 生成 EC 密钥对 + credID → Flash 存储凭证 → 私钥计算 attestation 签名 → 组装 CBOR 响应 
→ CTAPHID 分片回传 USB → libfido2 重组响应 → JS 拿到凭证对象 → 上传网站后端存公钥。


网站 JS get () → WebAuthn JSON 参数 → 浏览器 libfido2 序列化为 CBOR → CTAPHID_INIT 复用 / 建立 CID 会话 → CBOR 载荷分包为 64 字节 USB HID 报告 
→ USB 中断下发 STM32 → usbd_fido 接收报文 → ctaphid 重组完整 CBOR → ctap2_handle_command 分发 getAssertion 
→ 按 rpID /credID 检索 Flash 已有凭证 → 校验用户存在（按键确认）/PIN 校验 → 取出设备内私钥（不可导出）→ 私钥对挑战、认证数据做 ES256 签名
 → 组装带签名、credID、authData 的 CBOR 响应 
 → CTAPHID 分片回传 USB → libfido2 重组完整响应包 → JS 拿到 AuthenticatorAssertionResponse 对象 → 上传网站后端 → 后端使用注册时保存的公钥验签，校验通过完成登录
```

