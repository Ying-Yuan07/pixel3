# pixel3 刷安卓12

参考https://github.com/lihua-yang/hikey960/blob/master/compile%20AOSP12.0%20%2B%20kernel5.4.md

## aosp 下载与编译

------

### 1.中科院的AOSP repo

但没有android kernel（后期解决）
来自于https://mirrors.ustc.edu.cn/help/aosp.html

#### 安装repo工具

```bash
mkdir ~/bin
PATH=~/bin:$PATH
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
## 如果上述 URL 不可访问，可以用下面的：
## curl -sSL  'https://gerrit-googlesource.proxy.ustclug.org/git-repo/+/master/repo?format=TEXT' |base64 -d > ~/bin/repo
chmod a+x ~/bin/repo
```

修改安装repo使用的python版本

```shell
#查看python版本，如果是3.0一下，则切换成3.X
#通过软链接方式切换python版本
sudo rm /usr/bin/python
sudo ln -s /usr/bin/python3.6 /usr/bin/python

#修改repo使用的python版本
vim ~/bin/repo
#!/usr/bin/env python3
```

问题1 CERTIFICATE_VERIFY_FAILED

1)~/bin/repo中import sys 后面新增如下两行代码得以解决:

```shell
import ssl
ssl._create_default_https_context = ssl._create_unverified_context
```



### 2.初始化仓库

**不要用清华的仓库，没成功过，不要用！不要用！！！**

这里直接确定 版本是android-12.0.0_r25

```bash
mkdir aosp12   
cd aosp12 
~/bin/repo init -u git://mirrors.ustc.edu.cn/aosp/platform/manifest -b android-12.0.0_r25
~/bin/repo sync -j4  
```



### 3.二进制驱动下载与安装

下载版本为sp1a.210812.016.a2的二进制驱动

```shell
cd ../
mkdir drivers
cd divers
wget https://dl.google.com/dl/android/aosp/google_devices-blueline-sp1a.210812.016.a2-27da6bc0.tgz
wget https://dl.google.com/dl/android/aosp/qcom-blueline-sp1a.210812.016.a2-ff8e3906.tgz
```

安装二级制驱动

```shell
tar -xvf google_devices-blueline-sp1a.210812.016.a2-27da6bc0.tgz -C ../aosp12
tar -xvf qcom-blueline-sp1a.210812.016.a2-ff8e3906.tgz -C ../aosp12
cd aosp12
./extract-google_devices-blueline.sh
./extract-qcom-blueline.sh
```



### 4.编译aosp12

```bash
cd aosp12
source build/envsetup.sh
lunch aosp_blueline-userdebug
make TARGET_KERNEL_USE=4.9 -j64 RELAX_USES_LIBRARY_CHECK=true
```

清除编译结果

```shell
make clobber
```

### 5.烧录aosp12镜像

```shell
cd aosp9/out/target/product/blueline
adb reboot bootloader
fastboot flashall -w
```



## kernel 下载、编译与烧录

### 1.下载kernel

从官方网站下载kernel源码与安装工具包

#### 翻墙

登录https://tankecloud.me/register?aff=1015&code=7e79dd152a75查看结点配置信息

教程：https://notes-by-yangjinjie.readthedocs.io/zh_CN/latest/os/ubuntu-debian/Ubuntu%20Server%E5%91%BD%E4%BB%A4%E8%A1%8C%E9%85%8D%E7%BD%AEshadowsocks%E5%85%A8%E5%B1%80%E4%BB%A3%E7%90%86.html

https://floperry.github.io/2019/02/24/2018-06-25-Ubuntu-18.04-%E4%B8%8B%E8%A7%A3%E5%86%B3-shadowsocks-%E6%9C%8D%E5%8A%A1%E6%8A%A5%E9%94%99%E9%97%AE%E9%A2%98/

配置完成后，每次只需要执行

```bash
sudo sslocal -c shawdowsocks.json -d start
export http_proxy="http://127.0.0.1:8123/"
export https_proxy="http://127.0.0.1:8123/"
```

#### 下载内核代码

```bash
mkdir kernel
cd kernel
mkdir android-msm-crosshatch-4.9-android12
cd android-msm-crosshatch-4.9-android12
~/bin/repo init -u https://android.googlesource.com/kernel/manifest -b android-msm-crosshatch-4.9-android12
~/bin/repo sync -j8
```

#### 编译内核

```bash
cd aosp12
source build/envsetup.sh
lunch aosp_blueline-userdebug
cd ../kernel/android-msm-crosshatch-4.9-pie-qpr2
build/build.sh
```

待内核编译成功后，将生成的Image.lz4拷贝到aosp12对应目录下，重新编译，将userdata.img的文件系统设置为f2fs

https://zhuanlan.zhihu.com/p/53009043

```bash
cp out/out/android-msm-pixel-4.9/dist/Image.lz4 ../aosp12/device/google/crosshatch-kernel/
cd ../aosp12
make BOARD_USERDATAIMAGE_FILE_SYSTEM_TYPE=f2fs TARGET_USERIMAGES_USE_F2FS=true -j4 
```

#### 烧录内核

```bash
cd aosp9/out/target/product/blueline
adb reboot bootloader
fastboot flashall -w
```

或者将/blueline路径下的镜像文件拷贝的本地

```shell
adb reboot bootloader
fastboot flashall -w
```

查看手机内核版本

```shell
adb root
adb shell "uname -a"
#结果为Linux localhost 4.9.270-dirty #1 repo:android-msm-crosshatch-4.9-android12 SMP PREEMPT Mon Jan aarch64 
```

## 刷官方镜像包

https://developers.google.cn/android/images

https://developer.android.com/about/versions/12/download?hl=zh-cn

下载pixel3最新的安卓12镜像包https://dl.google.com/dl/android/aosp/blueline-sp1a.210812.016.a2-factory-0e1330df.zip

解压之后，执行flash-all脚本

安卓12内置了google store三件套，通过aosp刷机，没有google store,可以从此处获取google store镜像

### 问题1 无法获取root权限

```shell
PS E:\2_worksapce\Android> adb root adbd cannot run as root in production builds 
```

https://daily.elepover.com/2021/05/19/android-s/index.html

https://blog.csdn.net/Ender_Zhao/article/details/108615512

首先是升级至安卓11（无论是刷固件升级，还是用自带的，都会把Root刷掉）,可以用*magisk来root*

