<h1 align = "center">树莓派开发文档</h1>

[https://www.waveshare.net/wiki/CM4-IO-BASE-B](https://www.waveshare.net/wiki/CM4-IO-BASE-B)

# 1.GPIO

![image.png](https://github.com/Kouchay/RaspQt5.14.2/blob/main//assets/RaspberryGPIO.png)

## 1.bcm2835库

```bash
wget http://www.airspayce.com/mikem/bcm2835/bcm2835-1.71.tar.gz
tar -xvf ./bcm2835-1.71.tar.gz
cd bcm2835-1.71/
./configure 
sudo make check
sudo make install
sudo ldconfig 
```

## 2.WiringPi库

```bash
git clone https://github.com/WiringPi/WiringPi.git
cd ~/wiringPi
./build
```

# 2.SSH密钥

```bash
ssh-keygen -t rsa -b 2048 -C xxx1@email.com 

Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): <== 按 Enter
Created directory '/root/.ssh'.
Enter passphrase (empty for no passphrase): <== 输入密钥锁码，或直接按 Enter 留空
Enter same passphrase again: <== 再输入一遍密钥锁码
Your identification has been saved in /root/.ssh/id_rsa. <== 私钥
Your public key has been saved in /root/.ssh/id_rsa.pub. <== 公钥
The key fingerprint is:

cd .ssh
cat id_rsa.pub >> authorized_keys

编辑 /etc/ssh/sshd_config 文件，进行如下设置：

RSAAuthentication yes
PubkeyAuthentication yes
另外，请留意 root 用户能否通过 SSH 登录：

PermitRootLogin yes
当你完成全部设置，并以密钥方式登录成功后，再禁用密码登录：

PasswordAuthentication no
最后，重启 SSH 服务：

[root@host .ssh]$ service sshd restart
```

# 3.串口

## 1.GPIO模拟串口

![image.png](https://github.com/Kouchay/RaspQt5.14.2/blob/main//assets/RaspberryGPIO_UART.png)

```bash
#配置串口
sudo vim /boot/config.txt
#添加
dtoverlay=uart2
dtoverlay=uart3
dtoverlay=uart4
dtoverlay=uart5

###
UART0： GPIO14 = TXD0 -> ttyAMA0     GPIO15 = RXD0 -> ttyAMA0
UART1： ttyS0
UART2： GPIO0  = TXD2 -> ttyAMA1     GPIO1  = RXD2 -> ttyAMA1
UART3： GPIO4  = TXD3 -> ttyAMA2     GPIO5  = RXD3 -> ttyAMA2
UART4： GPIO8  = TXD4 -> ttyAMA3     GPIO9  = RXD4 -> ttyAMA3
UART5： GPIO12 = TXD5 -> ttyAMA4     GPIO13 = RXD5 -> ttyAMA4
```

## 2.2-CH RS485 HAT(RS485拓展版）

### 1.拓展版资料

[2-CH RS485 HAT - Waveshare Wiki](https://www.waveshare.net/wiki/2-CH_RS485_HAT)

### 2.拓展版驱动

```bash
sudo nano /boot/config.txt
#加入如下，int_pin根据实际焊接方式设置：
dtoverlay=sc16is752-spi1,int_pin=24
#重启设备
sudo reboot
```

## 3.TTL串口登录

### 1.树莓派串口配置

```bash
sudo raspi-config
```

进入如下界面

![](https://github.com/Kouchay/RaspQt5.14.2/blob/main//assets/raspi-config.jpg)

上下键选择第3项 Interfacing Options（接口选项），回车进入：

![](https://github.com/Kouchay/RaspQt5.14.2/blob/main//assets/Interfaceing Options.png)

选择I6,打开Serial Port 回到主界面 选择finish后重启

### 2.关闭蓝牙，打开串口，修改/boot下config.txt文件

```bash
sudo nano /boot/config.txt
#添加以下内容
dtoverlay=pi3-miniuart-bt
```

### 3.修改/boot下cmdline.txt

```bash
sudo nano /boot/cmdline.txt
#替换为以下内容
dwc_otg.lpm_enable=0 console=tty1 console=serial0,115200 root=/dev/mmcblk0p2 rootfstype=ext4 elevator=deadline fsck.repair=yes rootwait
#console=tty1 root=PARTUUID=469a1cec-02 rootfstype=ext4 fsck.repair=yes rootwait quiet splash plymouth.ignore-serial-consoles
```

### 4.接线

串口模块-----------------------树莓派
GND-----------------------------GND
RX---------------------------------TX
TX---------------------------------RX

# 4.Qt源码编译

## 1.1源码下载

```bash
wget https://download.qt.io/archive/qt/5.14/5.14.2/single/qt-everywhere-src-5.14.2.zip
#解压
unzip qt-everywhere-src-5.14.2.zip
```

## 1.2配置

先下载树莓派的配置文件
我是直接 访问  https://github.com/oniongarlic/qt-raspberrypi-configuration  打包下载解压后
把里面的 [linux-rpi4-v3d-g++](https://github.com/oniongarlic/qt-raspberrypi-configuration/tree/master/linux-rpi4-v3d-g%2B%2B) 文件夹放到 qt-everywhere-src-5.13.2/qtbase/mkspecs 目录里
按照这样编译不了的，报错： 不支持参数  -mfpu=crypto-neon-fp-armv8
到 [linux-rpi4-v3d-g++](https://github.com/oniongarlic/qt-raspberrypi-configuration/tree/master/linux-rpi4-v3d-g%2B%2B) 目录里，打开 qmake.conf 修改
 QMAKE_CFLAGS            += -march=armv8-a -mtune=cortex-a72

```bash
#编译配置参数
./configure -platform linux-rpi4-v3d-g++ -v \
-opengl es2 \
-opensource \
-confirm-license \
-release \
-reduce-exports \
-ssl \
-qt-xcb \
-tslib \
-no-compile-examples \
-skip wayland \
-skip qtquickcontrols \
-skip qtquickcontrols2 \
-skip qtsensors \
-skip qtdoc \
-skip qtquick3d \
-skip qtquicktimeline \
-skip qttools \
-skip qtwayland \
-skip qtwebengine \
-system-freetype \
-fontconfig \
-glib \
-prefix /home/toor/opt/Qt5.14.2 \
-recheck-all
```

其中

```bash
  EGLFS details:
  EGLFS OpenWFD ........................ no
  EGLFS i.Mx6 .......................... no
  EGLFS i.Mx6 Wayland .................. no
  EGLFS RCAR ........................... no
  EGLFS EGLDevice ...................... yes
  EGLFS GBM ............................ yes
  EGLFS VSP2 ........................... no
  EGLFS Mali ........................... no
  EGLFS Raspberry Pi ................... no
  EGLFS X11 ............................ yes
```

EGLFS Raspberry Pi 选项必须为no，若为yes,则修改树莓派下

```bash
sudo mv /usr/include/bcm_host.h /usr/include/bcm_host.h.bak
```

待编译结束后恢复文件

```bash
sudo mv /usr/include/bcm_host.h.bak /usr/include/bcm_host.h
```

## 2.编译问题

```bash
platform/default/bidi.cpp: In member function ‘void mbgl::BiDi::mergeParagraphLineBreaks(std::set<long unsigned int>&)’:
platform/default/bidi.cpp:63:24: error: ‘runtime_error’ is not a member of ‘std’
   63 |             throw std::runtime_error(std::string("ProcessedBiDiText::mergeParagraphLineBreaks: ") +
      |                        ^~~~~~~~~~~~~
platform/default/bidi.cpp: In member function ‘std::vector<std::__cxx11::basic_string<char16_t> > mbgl::BiDi::processText(const u16string&, std::set<long unsigned int>)’:
platform/default/bidi.cpp:98:20: error: ‘runtime_error’ is not a member of ‘std’
   98 |         throw std::runtime_error(std::string("BiDi::processText: ") + u_errorName(errorCode));
      |                    ^~~~~~~~~~~~~
platform/default/bidi.cpp: In member function ‘std::u16string mbgl::BiDi::getLine(std::size_t, std::size_t)’:
platform/default/bidi.cpp:109:20: error: ‘runtime_error’ is not a member of ‘std’
  109 |         throw std::runtime_error(std::string("BiDi::getLine (setLine): ") + u_errorName(errorCode));
      |                    ^~~~~~~~~~~~~
platform/default/bidi.cpp:125:20: error: ‘runtime_error’ is not a member of ‘std’
  125 |         throw std::runtime_error(std::string("BiDi::getLine (writeReordered): ") +
```

修改Qt源码,分别修改qt源码路径下

```bash
1.3rdparty/mapbox-gl-native/platform/default/bidi.cpp
+#include <stdexcept>
 #include <mbgl/text/bidi.hpp>
 #include <mbgl/util/traits.hpp>
2./3rdparty/mapbox-gl-native/include/mbgl/util/util.hpp
 #else
 #define MBGL_CONSTEXPR inline
 #endif
+
+#include <cstdint>
```

## 3.qtlinuxdeploy编译

```bash
git clone https://github.com/probonopd/linuxdeployqt.git
```

```bash
cd ./linuxdeployqt/tools/linuxdeployqt
vim main.cpp  
```

```cpp
    // openSUSE Leap 15.0 uses glibc 2.26 and is used on OBS
    // Ubuntu Xenial (16.04) uses glibc 2.23
    // Ubuntu Bionic (18.04) uses glibc 2.27
    // 大概在203行左右
    if (strverscmp (glcv, "2.28") >= 0) {
        qInfo() << "ERROR: The host system is too new.";
        qInfo() << "Please run on a system with a glibc version no newer than what comes with the oldest";
        qInfo() << "currently still-supported mainstream distribution (Ubuntu Bionic), which is glibc 2.27.";
        qInfo() << "This is so that the resulting bundle will work on most still-supported Linux distributions.";
        qInfo() << "For more information, please see";
        qInfo() << "https://github.com/probonopd/linuxdeployqt/issues/340";
        return 1;
    }
```

# 5.AD/DA拓展版

