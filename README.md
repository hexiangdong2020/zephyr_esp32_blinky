# zephyr_esp32_blinky
为ESP32编译Zephyr的Blinky示例
### 1 安装Zephyr RTOS
可以参考官方的安装文档:
https://docs.zephyrproject.org/3.3.0/develop/getting_started/index.html
在安装过程中，如果运行
```bash
west init ~/zephyrproject
```
因为网络原因失败或报错，可以再运行该命令一次。west update也与之类似。
笔者所使用Zephyr SDK的版本为0.15.2

### 2 编译Blinky示例
Blinky示例是指示点闪烁的示例，位于~/zephyrproject/zephyr/samples/basic/blinky，下文将为ESP32编译Blinky示例。
如果直接使用Zephyr官方的示例，运行下列命令：
```bash
cd ~/zephyrproject/zephyr
west build -p always -b esp32 samples/basic/blinky
```
那么会报错如下：
```bash
error: '__device_dts_ord_DT_N_ALIAS_led0_P_gpios_IDX_0_PH_ORD' undeclared here (not in a function)  
   85 | #define DEVICE_NAME_GET(dev_id) _CONCAT(__device_, dev_id)
```
这是因为main.c中使用了led0，但是ESP32的设备树中，并未定义led0这个设备，需要为ESP32定义led0这个设备。
笔者所使用的ESP32开发板为ESP-WROOM-32。它的引脚图如下：
https://www.jianguoyun.com/p/DSzhOA4Qq47dCxiQ_o4FIAA
其板载指示灯连接在D2口上，即led0即为GPIO0-2这个端口。
在blinky文件夹中创建boards文件夹，并在boards文件夹中新建esp32.overlay。overlay可以在不改动boards里的dts文件的情况下，对开发板的设备情况进行修改。向esp32.overlay写入以下内容：
```dts
 / {
      aliases {
            led0 = &myled0;
      };

      leds {
            compatible = "gpio-leds";
            myled0: led_0 {
                  gpios = <&gpio0 2 GPIO_ACTIVE_LOW>;
        };
      };
   };
```
然后，使用下列命令进行编译：
```bash
west build -p always -b esp32 samples/basic/blinky
```
如果得到下列输出，则表示编译成功：
![[Screenshot from 2023-06-18 20-47-19.png]]
### 3 烧录Blinky示例
将ESP-WROOM-32通过USB连接线连接到计算机上，执行下列命令烧录：
```bash
west flash
```
如果得到下列输出，则表示烧录成功：
![[Screenshot from 2023-06-18 20-50-03.png]]
观测ESP-WROOM-32开发板，可以看到板载的蓝色指示灯已经开始闪烁了，说明程序运行成功。
![[微信图片_20230618230300.jpg]]
