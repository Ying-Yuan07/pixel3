# pixel 3 刷机过程

参考https://github.com/lihua-yang/Pixel3

## 1. 下载aosp源码

### 1.1 repo的下载与安装

```bash
mkdir ~/bin
PATH= ~/bin:$PATH
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
##如果上述 URL 不可访问，可以用下面的：
##curl -sSL 'https://gerrit-googlesource.proxy.ustclug.org/git-repo/+/master/repo?format=TEXT' |base64 -d > ~/bin/repo
chmod a+x ~/bin/repo
```

```bash
##然后建立一个工作目录（名字任意）
mkdir pixel3_alpixel3_al
cd pixel3_al
```

### 1.2 下载aosp源码

到官网查看pixel3的对应版本，可以选择 安卓11，或者安卓9： android-9.0.0_r21

https://source.android.com/setup/start/build-numbers#source-code-tags-and-builds

![image-20210728220236357](pixel%203%20%E5%88%B7%E6%9C%BA%E8%BF%87%E7%A8%8B.assets/image-20210728220236357.png)

![image-20210806160917098](pixel%203%20%E5%88%B7%E6%9C%BA%E8%BF%87%E7%A8%8B.assets/image-20210806160917098.png)



#### 从清华下载

```shell
#去清华下载最新的aosp源码包aosp-latest.tar及其校验码aosp-latest.tar.md5
##建议先在本地下好，再传到服务器192.168.1.75：tan@tan:~/pixel3_all/package
wget -c https://mirrors.tuna.tsinghua.edu.cn/aosp-monthly/aosp-latest.tar # 下载初始化包
md5sum aosp-latest.tar #获取aosp-latest.tar校验码，如果跟官网中aosp-latest.tar.md5的内容一致，aosp源码包即为完整
tar xf aosp-latest.tar
cd aosp   # 解压得到的 aosp 工程目录
# 这时 ls 的话什么也看不到，因为只有一个隐藏的 .repo 目录
repo sync # 正常同步一遍即可得到完整目录
# 或 repo sync -l 仅checkout代码
```



#### 从中科大下载

参考https://mirrors.ustc.edu.cn/help/aosp.html

初始化仓库：

```bash
mkdir aosp9
cd aosp9
repo init -u git://mirrors.ustc.edu.cn/aosp/platform/manifest
## 如果提示无法连接到 gerrit.googlesource.com，可以编辑 ~/bin/repo，把 REPO_URL 一行替换成下面的：
## REPO_URL = 'https://gerrit-googlesource.proxy.ustclug.org/git-repo'
```

如果需要某个特定的 Android 版本

```bash
repo init -u git://mirrors.ustc.edu.cn/aosp/platform/manifest -b  android-9.0.0_r21
```

同步源码树（以后只需执行这条命令来同步）：

```bash
repo sync
```



## 2. 下载驱动

### 2.1下载二进制驱动

不能仅通过纯源代码来使用 AOSP，要运行 AOSP，还需要与硬件相关的其他专有库，例如用于硬件图形加速的专有库。如需其他资源的下载链接和[设备二进制文件](https://source.android.google.cn/source/requirements?hl=zh-cn#binaries),有些设备会将这些专有二进制文件打包到其 `/vendor` 分区。

参考https://blog.csdn.net/zz531987464/article/details/94163954

​       https://source.android.google.cn/source/building?hl=zh-cn

官网下载地址https://developers.google.com/android/drivers

版本为Pixel 3 binaries for Android 11.0.0 (RQ3A.210705.001)：https://developers.google.com/android/drivers#bluelinerq3a.210705.001

最终会下载到两个tar.gz压缩包，解压后是两个.sh脚本

将上面的两个脚本放到源代码根目录下，执行该两个脚本，不停回车直到输入：I ACCEPT,最终binary都会自动下载到vendor目录下，最终所有的AOSP代码算是下载完成了，接下来开始编译代码。

```shell
#下载对应版本二进制文件
yy@yy:~/workspace/pixel3-drivers$ wget https://dl.google.com/dl/android/aosp/google_devices-blueline-pq1a.181205.006-5a3e2737.tgz
yy@yy:~/workspace/pixel3-drivers$ wget https://dl.google.com/dl/android/aosp/qcom-blueline-pq1a.181205.006-e364e5c0.tgz

##将两个压缩包解压并拷贝到aosp目录下，执行这两个可执行文件
yy@yy:~/workspace/aosp$ ./extract-google_devices-blueline.sh
yy@yy:~/workspace/aosp$ ./extract-qcom-blueline.sh 
```



### 2.2 安装JDK

```shell
sudo apt-get install openjdk-8-jdk
```

安装完以后，执行下面的命令添加JAVA_HOME相关配置

```shell
sudo vim /etc/profile
```

在打开的profile文件的末尾添加下面的内容：

```shell
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export JRE_HOME=${JAVA_HOME}/jre 
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib 
export PATH=${JAVA_HOME}/bin:$PATH
```

使用 source /etc/profile使环境变量配置立即生效

```shell
source /etc/profile
```



### 2.3 编译aosp

```shell
##编译aosp####
#进入aosp根目录，执行以下指令，用envsetup.sh（它在源代码根目录/build下面）脚本初始化环境
tan@tan:~/pixel3_all/aosp$ source build/envsetup.sh
tan@tan:~/pixel3_all/aosp$ lunch aosp_blueline-userdebug
tan@tan:~/pixel3_all/aosp$ make -j16 
```

**NOTE:**
**1.一定要选你要编译的内核版本
2.不要用su来执行（因此最好不要将工作目录放在home/user之外）
3.如果出现问题，可以执行make clobber来清理**



lunch aosp_blueline-userdebug ：用lunch命令选择编译目标，结果显示编译的环境与目标

![image-20210728191919396](pixel%203%20%E5%88%B7%E6%9C%BA%E8%BF%87%E7%A8%8B.assets/image-20210728191919396.png)



编译成功提示

![image-20210728161311504](pixel%203%20%E5%88%B7%E6%9C%BA%E8%BF%87%E7%A8%8B.assets/image-20210728161311504.png)



**错误1**：mismatch in the <uses-library> tags between the build system and the manifest

![image-20210730173135755](pixel%203%20%E5%88%B7%E6%9C%BA%E8%BF%87%E7%A8%8B.assets/image-20210730173135755.png)



**解决方案**

```bash
make TARGET_KERNEL_USE=4.9 -j64 RELAX_USES_LIBRARY_CHECK=true
```



**错误2**：syntax error of "build/make/tools/merge-event-log-tags.py"

```shell
SyntaxError: invalid syntax
[  0% 39/92183] target Java source list: ims-common
  File "build/make/tools/normalize_path.py", line 25
    print os.path.normpath(p)
           ^
SyntaxError: invalid syntax
[  0% 40/92183] target Java source list: apache-xml
  File "build/make/tools/normalize_path.py", line 25
    print os.path.normpath(p)
           ^
SyntaxError: invalid syntax
[  0% 41/92183] Check module type: out/host/linux-x86/obj/STATIC_LIBRARIES/libadb_intermediates/link_type
```

**解决方案**：安卓9只支持python2，把python版本切换至`python 2.7.x`

https://en-support.renesas.com/knowledgeBase/19903179

### 2.4 刷入镜像

平台：ubuntu、win10(失败了)；

环境：pixel3 开启usb权限、win10安装adb与pixel3驱动。

参考https://developer.android.com/studio/run/device?hl=zh-cn

​        https://zhuanlan.zhihu.com/p/140828682

#### pixel3 开启usb权限

参考https://source.android.com/setup/build/running#flashing-a-device，可以在设备处于 `fastboot` 引导加载程序模式时刷写设备。如需在设备进行冷启动时进入 `fastboot` 模式。

1. 在“设置”中，点按**关于手机**，然后点按**版本号**七次。
2. 当看到“您已处于开发者模式”这条消息后，点按**返回**按钮。
3. 点按**开发者选项**，然后启用 **OEM 解锁**和 **USB 调试**（如果 **OEM 解锁**处于停用状态，请连接到互联网，以便设备可以至少检入一次。如果“OEM 解锁”仍处于停用状态，说明设备可能已被运营商锁定 SIM 卡，系统无法解锁引导加载程序。）
4. 重启手机，同时按电源与音量减键，进入fastboot模式（或者执行 adb reboot bootloader ），如下：

​            

<img src="pixel%203%20%E5%88%B7%E6%9C%BA%E8%BF%87%E7%A8%8B.assets/image-20210808214031315.png" alt="image-20210808214031315" style="zoom: 25%;" />



#### ubuntu环境下刷入镜像

```c
打开usb调试，进入BootLoader模式    
adb reboot bootloader    
在out/target/product/blueline目录烧入编译生成的所有镜像    
fastboot flashall -w  
```

```bash
C:\Users\Administrator>adb devices -l
List of devices attached
89AX07GLX              device product:blueline model:Pixel_3 device:blueline transport_id:1
C:\Users\Administrator>adb reboot bootloader

C:\Users\Administrator>fastboot flashall -w
--------------------------------------------
Bootloader Version...: b1c1-0.1-5343672
Baseband Version.....: g845-00017-190312-B-5369743
Serial Number........: 89AX07GLX
--------------------------------------------
Checking 'product'                                 OKAY [  0.057s]
Setting current slot to 'a'                        OKAY [  0.323s]
Sending 'boot_a' (65536 KB)                        OKAY [  1.606s]
Writing 'boot_a'                                   OKAY [  0.389s]
Sending 'dtbo_a' (8192 KB)                         OKAY [  0.308s]
Writing 'dtbo_a'                                   OKAY [  0.110s]
Sending 'vbmeta_a' (4 KB)                          OKAY [  0.117s]
Writing 'vbmeta_a'                                 OKAY [  0.060s]
Sending 'product_a' (4956 KB)                      OKAY [  0.237s]
Writing 'product_a'                                OKAY [  0.168s]
Sending sparse 'system_a' 1/5 (262140 KB)          OKAY [  7.227s]
Writing 'system_a'                                 OKAY [  0.061s]
Sending sparse 'system_a' 2/5 (262140 KB)          OKAY [  7.693s]
Writing 'system_a'                                 OKAY [  0.364s]
Sending sparse 'system_a' 3/5 (262140 KB)          OKAY [  7.502s]
Writing 'system_a'                                 OKAY [  0.364s]
Sending sparse 'system_a' 4/5 (262140 KB)          OKAY [  9.953s]
Writing 'system_a'                                 OKAY [  0.363s]
Sending sparse 'system_a' 5/5 (137016 KB)          OKAY [  4.133s]
Writing 'system_a'                                 OKAY [  1.148s]
Sending 'system_b' (86884 KB)                      OKAY [  3.057s]
Writing 'system_b'                                 OKAY [  0.920s]
Sending sparse 'vendor_a' 1/2 (262140 KB)          OKAY [ 14.327s]
Writing 'vendor_a'                                 OKAY [  0.365s]
Sending sparse 'vendor_a' 2/2 (202628 KB)          OKAY [  6.414s]
Writing 'vendor_a'                                 OKAY [  1.440s]
Erasing 'userdata'                                 OKAY [  6.141s]
Erase successful, but not automatically formatting.
File system type raw not supported.
Erasing 'metadata'                                 OKAY [  0.294s]
Erase successful, but not automatically formatting.
File system type raw not supported.
Rebooting                                          OKAY [  0.056s]
Finished. Total time: 80.384s

```



**问题1**：执行adb reboot bootloader时，device unauthorized

参考https://www.zhihu.com/question/66058586

在手机上打开usb 调试







#### win10 环境下刷入镜像（失败）

##### 安装adb

参考https://zhuanlan.zhihu.com/p/140828682

1. 下载安装包

https://link.zhihu.com/?target=https%3A//dl.google.com/android/repository/platform-tools-latest-windows.zip

2. 解压安装包，并设置环境变量


##### win10 安装Google USB

下载驱动程序：https://developer.android.com/studio/run/win-usb?hl=zh-cn，并解压

<img src="pixel%203%20%E5%88%B7%E6%9C%BA%E8%BF%87%E7%A8%8B.assets/image-20210809120140822.png" alt="image-20210809120140822" style="zoom: 50%;" />

1. 将您的 Android 设备连接到计算机的 USB 端口。
2. 在 Windows 资源管理器中，打开**计算机管理**。
3. 在**计算机管理**左侧窗格中，选择**设备管理器**。
4. 在**设备管理器**右侧窗格中，找到并展开**便携式设备**或**其他设备**，具体取决于您看到的是哪一项。
5. 右键点击已连接设备的名称，然后选择**更新驱动程序软件**。
6. 在**硬件更新向导**中，选择**浏览计算机以查找驱动程序软件**，然后点击**下一步**。
7. 点击**浏览**，然后找到 USB 驱动程序文件夹。例如，Google USB 驱动程序位于 `android_sdk\extras\google\usb_driver\`。
8. 点击**下一步**以安装驱动程序。
9. 在powershell窗口中执行fastboot devices，查看win10是否能够识别pixel3

```bash
fastboot devices
```

![image-20210809120539252](pixel%203%20%E5%88%B7%E6%9C%BA%E8%BF%87%E7%A8%8B.assets/image-20210809120539252.png)



拷贝aosp生成的镜像：

在out/target/product/blueline目录烧入编译生成的所有镜像  

**问题1**：执行fastboot flashall -w时，提示 fastboot error android_product_out not set

参考https://blog.csdn.net/helloUSB2010/article/details/38627599

假设镜像文件的存储路径为E:\2_worksapce\Android\blueline,设置ANDROID_PRODUCT_OUT环境变量，打开cmd，执行

```bash
##设置
set ANDROID_PRODUCT_OUT="E:\2_worksapce\Android\blueline"
```

计算机右键-->属性-->高级系统设置-->环境变量，添加ANDROID_PRODUCT_OUT环境变量为E:\2_worksapce\Android\blueline，并将%ANDROID_PRODUCT_OUT%添加到Path；

重新打开一个cmd窗口，执行fastboot flashall -w









## **3.拉取kernel代码并编译**

参考：https://blog.csdn.net/zz531987464/article/details/94163954           

根据前面的方法拉取编译的AOSP代码中是不包含kernel源码的，而是一个已经编译好的内核镜像文件。如果我们需要修改编译kernel源码，需要单独拉取kernel的代码进行编译。

查看需要的内核版本https://source.android.com/setup/start/build-numbers#source-code-tags-and-builds

https://source.android.com/setup/build/building-kernels，

我们选择pixel3对应分支： android-msm-crosshatch-4.9-android11-qpr2；

### 3.1 下载内核代码

```bash
mkdir kernel
cd kernel 
repo init -u https://android.googlesource.com/kernel/manifest
repo init -u https://android.googlesource.com/kernel/manifest -b android-msm-crosshatch-4.9-pie-qpr2
repo sync -j8
```



### 3.2 编译kernel

首先我们在AOSP源码目录下准备好编译环境 ,用于启动一些编译时所需要的环境变量，例如：交叉编译工具等

```bash
tan@tan:~/pixel3_all/aosp9$ source build/envsetup.sh
tan@tan:~/pixel3_all/aosp9$ lunch aosp_blueline-userdebug
```



编译内核

```bash
yy@yy:~/workspace/pixel3_all/kernel/android9/google/repo$ build/build.sh
```



内核二进制文件、模块和相应的映像位于 `out/BRANCH/dist` 目录下，将Image.lz4-dtb拷贝到Android9系统源码的device/google/crosshatch-kernel目录下，   

```bash
tan@tan:~/pixel3_all/kernel_andrio9/ustc/msm/out/android-msm-bluecross-4.9/dist$ ls
abi.prop       kernel-headers.tar.gz       snd-soc-cs35l36.ko          snd-soc-wcd9xxx.ko  vmlinux
dtbo.img       kernel-uapi-headers.tar.gz  snd-soc-sdm845.ko           snd-soc-wcd-spi.ko  wcd-core.ko
ftm5.ko        pinctrl-wcd.ko              snd-soc-sdm845-max98927.ko  System.map          wcd-dsp-glink.ko
Image.lz4-dtb  sec_touch.ko                snd-soc-wcd934x.ko          unstripped          wlan.ko

```



重新编译aosp

```bash
yy@yy:~/workspace/pixel3_all/aosp9$ source build/envsetup.sh
yy@yy:~/workspace/pixel3_all/aosp9$ lunch aosp_blueline-userdebug
yy@yy:~/workspace/pixel3_all/aosp9$ make BOARD_USERDATAIMAGE_FILE_SYSTEM_TYPE=f2fs TARGET_USERIMAGES_USE_F2FS=true -j4
```



重新刷机

```bash
#fastboot flashall -w
yy@yy:~/workspace/pixel3_all/aosp9/out/target/product/blueline$ adb reboot bootloader
yy@yy:~/workspace/pixel3_all/aosp9/out/target/product/blueline$ fastboot flashall -w
--------------------------------------------
Bootloader Version...: b1c1-0.1-5343672
Baseband Version.....: g845-00017-190312-B-5369743
Serial Number........: 89AX07GLX
--------------------------------------------
Checking product
OKAY [  0.060s]
target reported max download size of 268435456 bytes
Erase successful, but not automatically formatting.
File system type raw not supported.
Erase successful, but not automatically formatting.
File system type raw not supported.
Sending 'boot_a' (65536 KB)...
OKAY [  2.509s]
Writing 'boot_a'...
OKAY [  0.708s]
Sending 'dtbo_a' (8192 KB)...
OKAY [  0.383s]
Writing 'dtbo_a'...
OKAY [  0.102s]
Sending 'product_a' (4956 KB)...
OKAY [  0.288s]
Writing 'product_a'...
OKAY [  0.170s]
Sending sparse 'system_a' 1/5 (262140 KB)...
OKAY [  8.560s]
Writing 'system_a' 1/5...
OKAY [  0.366s]
Sending sparse 'system_a' 2/5 (262140 KB)...
OKAY [  9.334s]
Writing 'system_a' 2/5...
OKAY [  0.366s]
Sending sparse 'system_a' 3/5 (262140 KB)...
OKAY [  9.114s]
Writing 'system_a' 3/5...
OKAY [  0.366s]
Sending sparse 'system_a' 4/5 (262140 KB)...
OKAY [  8.734s]
Writing 'system_a' 4/5...
OKAY [  0.366s]
Sending sparse 'system_a' 5/5 (137016 KB)...
OKAY [  4.734s]
Writing 'system_a' 5/5...
OKAY [  1.160s]
Sending 'system_b' (86884 KB)...
OKAY [  2.840s]
Writing 'system_b'...
OKAY [  0.910s]
Sending 'vbmeta_a' (4 KB)...
OKAY [  0.120s]
Writing 'vbmeta_a'...
OKAY [  0.063s]
Sending sparse 'vendor_a' 1/2 (262140 KB)...
OKAY [  8.767s]
Writing 'vendor_a' 1/2...
OKAY [  0.366s]
Sending sparse 'vendor_a' 2/2 (202628 KB)...
OKAY [  6.954s]
Writing 'vendor_a' 2/2...
OKAY [  1.440s]
Setting current slot to 'a'...
OKAY [  0.317s]
Erasing 'userdata'...
OKAY [  6.163s]
Erasing 'metadata'...
OKAY [  0.225s]
Rebooting...

Finished. Total time: 77.761s

```

验证是否已经刷机成功

```bash
#adb shell后 cat /proc/version ，内核版本变成了 4.9 

yy@yy:~/workspace/pixel3_all/aosp9/out/target/product/blueline$ adb shell
blueline:/ $ cat /proc/version                                                                         
Linux version 4.9.124_audio (build-user@build-host) (Android clang version 5.0.1 (https://us3-mirror-android.googlesource.com/toolchain/clang 00e4a5a67eb7d626653c23780ff02367ead74955) (https://us3-mirror-android.googlesource.com/toolchain/llvm ef376ecb7d9c1460216126d102bb32fc5f73800d) (based on LLVM 5.0.1svn)) #1 SMP PREEMPT Wed Feb 6 22:09:52 UTC 2019
```

编译的镜像用的是/mnt/sdb/pixel-kernel/private/msm-google/fs/f2fs下面的文件  

make bootimage  

fastboot flash boot boot.img    



#### 问题1：触屏失灵

https://forum.xda-developers.com/t/built-kernel-from-scratch-touchscreen-doesnt-work.3768804/

原因：触屏驱动模块没有在pixel3安装成功

解决方案：将google/repo/out/android-msm-bluecross-4.9/dist下的.ko文件，拷贝到/vendor/lib/modules/

```bash
#disable-verity on the phone:
adb root
adb disable-verity
adb shell sync
adb reboot
#push module:
adb root
adb remount
adb push google/repo/out/android-msm-bluecross-4.9/dist/*.ko  /vendor/lib/modules/
adb reboot
```

