# Rubber Ducky 的脚本语言功能介绍和使用方法
- Rubber Ducky  是什么
  
- Rubber Ducky  有什么优点
   
- Rubber Ducky  怎么用

  - PC软件使用方法
  
  - 使用方法
 
你这个方向和前面的 CanoKey / Jade 移植其实有很强的关联，
因为 USB Rubber Ducky 本质上也是一个 
USB HID 设备 + 脚本解释器。

定位为：

ESP32-P4 USB Automation Key / Security Test Device

支持 DuckyScript 类脚本，用于自动化测试、设备运维、QA测试。

USB Rubber Ducky 的核心就是：

1. USB 枚举成 HID Keyboard✅

2. 读取脚本✅
 
3. 解释命令✅
 
4. 通过 HID 发送键盘事件✅DuckyScript 是一种简单脚本语言，常见命令包括 STRING、DELAY、按键组合等。
⸻

一、ESP32-P4 架构设计

结合你的设备：
```

                ESP32-P4
                    |
        +-----------+------------+
        |                        |
      LVGL UI                USB OTG
        |                        |
 Script Manager             USB HID
        |                        |
 LittleFS                  Keyboard Device
        |
 .duck 文件

```
⸻

软件架构

建议：

```
components/
usb_ducky/
|
├── ducky_parser.c
├── ducky_executor.c
├── ducky_command.c
├── hid_keyboard.c
└── keyboard_layout.c
storage/
├── scripts/
│
├── test1.duck
├── test2.duck
ui/
├── script_list.c
├── script_run.c

```
不要把脚本逻辑放到 USB 层。

⸻

二、移植步骤

Stage 1：USB HID Keyboard 跑通

先不要做脚本。

```
ESP32-P4：

TinyUSB
 |
USB Device
 |
HID Keyboard

目标：

电脑识别：

ESP32-P4 HID Keyboard

测试：

打开记事本：

设备发送：

Hello ESP32

结果：

电脑显示：

Hello ESP32

```
⸻

Stage 2：实现 HID API

建立抽象层：

```
hid_keyboard.h
void hid_key_press(
    uint8_t key
);
void hid_key_release();
void hid_type_string(
    char *str
);

```
上层不要直接调用 TinyUSB。

以后：

```
Ducky Engine
      |
hid_keyboard.c
      |
TinyUSB
```

⸻

Stage 3：实现 DuckyScript Parser

例如：

```
script:

REM test
DELAY 1000
STRING Hello
ENTER

解析：

line1
 |
REM
 |
skip
line2
DELAY
 |
delay_ms(1000)
line3
STRING
 |
type("Hello")

```
⸻

核心结构：

```
typedef enum
{
 CMD_STRING,
 CMD_DELAY,
 CMD_ENTER,
 CMD_CTRL,
 CMD_ALT
} duck_cmd_t;
typedef struct
{
 duck_cmd_t cmd;
 char param[128];
} duck_line_t;
```

⸻

Stage 4：执行器

流程：
```

读取文件
      |
parser
      |
command queue
      |
executor
      |
USB HID

```
例如：

```
STRING ABC

转换：

'A'
hid_press()
hid_release()
'B'
hid_press()
hid_release()
```

⸻

Stage 5：存储

你的设备：

已经有：

32MB Flash
16MB USB MSC
LittleFS

建议：

```
LittleFS
/scripts
   boot.duck
   test.duck
```

LVGL显示：

```
Scripts:
[ Login Test ]
[ Production Test ]
[ Device Test ]
RUN
```

⸻

Stage 6：增加 UI

你的优势：

普通 Rubber Ducky：

无屏幕

你的：

2寸 MIPI
```
显示：
Script:
wifi_test.duck
Status:
Running...
Step 15/100
```

⸻

三、和你的 FIDO2 USB 架构融合

这里非常关键。

你现在：

```
USB
 |
 |
+----------------+
|
Composite Device
|
+---- FIDO2 HID
|
+---- MSC
|
+---- CDC

```
加入：

+---- Automation HID

但是不要默认开启。

建议：

```
USB Mode:

Normal Mode
 FIDO2
 MSC
Automation Mode
 HID Script
```

⸻

四、验证测试步骤

Test 1 USB枚举

电脑：

Windows设备管理器：

应该看到：

Human Interface Device
ESP32-P4 Keyboard

⸻

Test 2 HID发送测试

固件：
```

固定发送：

ABC123

PC：

记事本：

出现：

ABC123

PASS。
```

⸻

Test 3 Parser测试

不用USB。

PC单元测试：

```
输入：

STRING TEST
DELAY 500
ENTER

输出：

CMD_STRING
CMD_DELAY
CMD_ENTER

```
⸻

Test 4 Script执行测试

```
脚本：

STRING Hello
ENTER

PC：

结果：

Hello
```

⸻

Test 5 多语言键盘

重点。

```
键盘布局：

US
UK
DE
JP
CN

需要：

key mapping。
```

⸻

Test 6 LVGL测试

显示：
```

script:
hello.duck
[RUN]
Running...
Finished

```
⸻

五、和 CanoKey 移植类似的接口设计

推荐：
```

hardware_hal
 |
 +-- usb_hal
 |
 +-- storage_hal
 |
 +-- display_hal
application
 |
 +-- fido
 |
 +-- jade
 |
 +-- ducky
```

共享：

```
storage_hal
crypto_hal
usb_hal

```
⸻

六、开发时间估计（按你现在能力）

你已经完成：

✅ ESP32-P4 USB
✅ TinyUSB
✅ CanoKey CTAPHID
✅ LVGL

所以：

模块	时间
USB HID Keyboard	1-2天
Parser	2-3天
Executor	2天
LittleFS脚本管理	2天
LVGL界面	3-5天
完善测试	1周

Demo：

约2周。

⸻

 
 ESP32-P4 平台：
```
Security Toolkit
├── FIDO2 Security Key
|
├── PIV Smart Card
|
├── OpenPGP Card
|
├── Bitcoin Wallet (Jade)
|
├── KeePass Manager
|
└── USB Automation Engine
```
比单独 Rubber Ducky 更像一个：

个人安全工作站 + 硬件安全钥匙。
