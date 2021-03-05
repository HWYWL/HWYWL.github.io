---
title: ESP32的MicorPython库
date: 2020-08-10 17:40:46
tags: 
 - MCU
categories: 
 - MCU
keywords: "MCU"
description: ESP32的MicorPython库
---

# yi-mp
YI MicroPython 是一个MicroPython简化操作的模块，可以快速的链接WiFi以及开启WebREPL。

## 安装
如果你的ESP32安装了MicroPython的固件就可以使用，ESP8266我没有试过，手头没这个模块。

**注意**：使用upip命令需要联网，所以使用需要你的ESP82模块连上WiFi才能安装模块。

![](https://hwy-figure-bed.oss-cn-hangzhou.aliyuncs.com/image/20200807182401.png)

已经将yi-mp上传到了**PyPI**,我们连接上ESP32的串口进行安装：

```
import upip
upip.install('yi-mp')
```

![](https://hwy-figure-bed.oss-cn-hangzhou.aliyuncs.com/image/20206166x1.png)

这些库会自动安装到/lib下面，我们可以使用以下命令查看安装路径：

```
>>> upip.get_install_path()
'/lib'
```

我们可以使用os命令查看下载好的文件

```
import os
os.listdir('lib')
```

![](https://hwy-figure-bed.oss-cn-hangzhou.aliyuncs.com/image/20200807172901.png)

## 使用
导入emp_boot 设置启动模式

```
import emp_boot
```

设置**boot.py**的启动模式 这个操作会修改并覆盖**boot.py**文件

```
emp_boot.set_boot_mode()
```

![](https://hwy-figure-bed.oss-cn-hangzhou.aliyuncs.com/image/20200807172915.png)

如果你是开发只用就选开发者模式，方便调试和编码，如果已经是完善的程序可以长时间跑了，可以选WiFi模式节约资源。

选择之后模块会重启，然后打印一些系统信息：
```
Reboot
I (1182645) wifi: state: run -> init (0)
I (1182645) wifi: pm stop, total sleep time: 1081965484 us / 1180618709 us

I (1182645) wifi: new:<3,0>, old:<3,0>, ap:<255,255>, sta:<3,0>, prof:1
E (1182655) event: system_event_sta_disconnected_handle_default 294 esp_wifi_internal_reg_rxcb ret=0x3014
I (1182665) wifi: STA_DISCONNECTED, reason:8
I (1182665) wifi: flush txq
I (1182665) wifi: stop sw txq
I (1182675) wifi: lmac stop hw txq
ets Jun  8 2016 00:22:57

rst:0xc (SW_CPU_RESET),boot:0x13 (SPI_FAST_FLASH_BOOT)
configsip: 0, SPIWP:0xee
clk_drv:0x00,q_drv:0x00,d_drv:0x00,cs0_drv:0x00,hd_drv:0x00,wp_drv:0x00
mode:DIO, clock div:1
load:0x3fff0018,len:4
load:0x3fff001c,len:5180
load:0x40078000,len:11388
ho 0 tail 12 room 4
load:0x40080400,len:7340
entry 0x40080704
I (426) psram: This chip is ESP32-D0WD
I (426) spiram: Found 64MBit SPI RAM device
I (426) spiram: SPI RAM mode: flash 80m sram 80m
I (429) spiram: PSRAM initialized, cache is in low/high (2-core) mode.
I (436) cpu_start: Pro cpu up.
I (440) cpu_start: Application information:
I (445) cpu_start: Compile time:     Mar 16 2020 04:36:04
I (451) cpu_start: ELF file SHA256:  0000000000000000...
I (457) cpu_start: ESP-IDF:          v3.3
I (462) cpu_start: Starting app cpu, entry point is 0x40083dc0
I (453) cpu_start: App cpu up.
I (946) spiram: SPI SRAM memory test OK
I (947) heap_init: Initializing. RAM available for dynamic allocation:
I (947) heap_init: At 3FFAE6E0 len 00001920 (6 KiB): DRAM
I (953) heap_init: At 3FFBA658 len 000259A8 (150 KiB): DRAM
I (959) heap_init: At 3FFE0440 len 00003AE0 (14 KiB): D/IRAM
I (966) heap_init: At 3FFE4350 len 0001BCB0 (111 KiB): D/IRAM
I (972) heap_init: At 4009743C len 00008BC4 (34 KiB): IRAM
I (978) cpu_start: Pro cpu start user code
I (100) cpu_start: Chip Revision: 1
I (101) cpu_start: Starting scheduler on PRO CPU.
I (0) cpu_start: Starting scheduler on APP CPU.
I (880) modsocket: Initializing

       ---------------------------
       - Python YI MicroPython   -
       -      version 1.0.3      -
       -     by YI               -
       ---------------------------

I (960) wifi: wifi driver task: 3ffc5164, prio:23, stack:3584, core=0
I (2026) system_api: Base MAC address is not set, read default base MAC address from BLK0 of EFUSE
I (2036) system_api: Base MAC address is not set, read default base MAC address from BLK0 of EFUSE
I (2076) wifi: wifi firmware version: aeed694
I (2076) wifi: config NVS flash: enabled
I (2076) wifi: config nano formating: disabled
I (2076) wifi: Init dynamic tx buffer num: 32
I (2076) wifi: Init data frame dynamic rx buffer num: 32
I (2086) wifi: Init management frame dynamic rx buffer num: 32
I (2086) wifi: Init management short buffer num: 32
I (2096) wifi: Init static rx buffer size: 1600
I (2096) wifi: Init static rx buffer num: 10
I (2106) wifi: Init dynamic rx buffer num: 32
I (2186) phy: phy_version: 4102, 2fa7a43, Jul 15 2019, 13:06:06, 0, 0
I (2186) wifi: mode : sta (ac:67:b2:24:18:dc)
I (2186) wifi: STA_START
创建 config/wifi_cfg.json 配置
I (4606) network: event 1
[0]    XXXXX                                    -53     dBm
[1]    XXXXX-Client                             -54     dBm
[2]    XXXXX_Test                               -54     dBm
[3]    XXXXX                                    -57     dBm
[4]    XXXXX_Test                               -57     dBm
[5]    DaChuang                                 -60     dBm
[6]    XXXXX-Client                             -61     dBm
没有记录!
正在扫描网络...
I (6886) network: event 1
[0]    XXXXX                                    -53     dBm
[1]    XXXXX-Client                             -54     dBm
[2]    XXXXX_Test                               -54     dBm
[3]    XXXXX                                    -57     dBm
[4]    XXXXX_Test                               -57     dBm
[5]    DaChuang                                 -60     dBm
[6]    XXXXX-Client                             -61     dBm
您想访问哪一个? [0-6]
```

选择wifi，输入密码，连上之后就会打印IP、网关等信息。

![](https://hwy-figure-bed.oss-cn-hangzhou.aliyuncs.com/image/20200807173726.png)

WebREPL和串口REPL是一样的，可以到下面这个网址链接操作,他可以通过网页数据命令，让我们摆脱对数据线的依赖。

```
http://micropython.org/webrepl/
```

WebREPL的默认密码是123456，你也可以使用一下命令进行重置：

```
import emp_boot
emp_boot.set_web_repl()
```

可以使用如下代码查看文件的内容：

```
print(open('/config/webrepl.pass').read())
```

![](https://hwy-figure-bed.oss-cn-hangzhou.aliyuncs.com/image/20200807182235.png)

让我们摆脱数据线吧，啦啦啦~~~

![](https://hwy-figure-bed.oss-cn-hangzhou.aliyuncs.com/image/20200807182627.png)

源码：https://github.com/HWYWL/yi-mp

### 问题建议

- 联系我的邮箱：ilovey_hwy@163.com
- 我的博客：https://hwy.ac.cn
- GitHub：https://github.com/HWYWL

## 感谢

[MicroPython](http://micropython.org/)
[1Z实验室](http://www.1zlab.com/)