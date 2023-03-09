# pixel6 刷 android 13

环境：大机房74服务器

## 1. aosp下载与编译

### 1.1 安装repo

参考 [pixel3 刷安卓12](https://github.com/Ying-Yuan07/pixel3/blob/main/pixel3%20%E5%88%B7%E5%AE%89%E5%8D%9312.md)

### 1.2 初始化本地仓库

查看pixel 6支持的系统版本https://source.android.com/docs/setup/about/build-numbers#source-code-tags-and-builds，选择一个适配多种机型的android 13版本号`android-13.0.0_r30`

![image-20230303081059325](pixel6 刷 android 13.assets/image-20230303081059325.png)

```shell
mkdir android-13.0.0_r30   
cd android-13.0.0_r30
~/.bin/repo init -u git://mirrors.ustc.edu.cn/aosp/platform/manifest -b android-13.0.0_r30
~/.bin/repo sync -j4 
```

### 1.3 二进制驱动下载与安装

#### 下载

在官网[驱动库](https://developers.google.com/android/drivers)中查找`Pixel 6 binaries for Android 13.0.0(TQ1A.230205.002)`的[二进制驱动](https://developers.google.com/android/drivers#orioletq1a.230205.002)

![image-20230303082156971](pixel6 刷 android 13.assets/image-20230303082156971.png)

```shell
#dowload drivers package
cd ~/workspace/pixel3_all/packsges/drivers/android-13.0.0_r30/pixel6
wget https://dl.google.com/dl/android/aosp/google_devices-oriole-tq1a.230205.002-ab978668.tgz
#check package
yy@tan:~/workspace/pixel3_all/packsges/drivers$ sha256sum google_devices-oriole-tq1a.230205.002-ab978668.tgz 
8250d5ae5b8d24409d84b95f008e84841d300014cb62edcbae3c9b8663cc0acd  google_devices-oriole-tq1a.230205.002-ab978668.tgz
```

注：从pixel6开始，驱动只保留了`Google自己开发的 Vendor image:extract-google_devices-oriole.sh`删掉了第三方高通Qualcomm开发的`GPS, Audio, Camera, Gestures, Graphics, DRM, Video, Sensors驱动`

##

#### **安装二级制驱动**

```shell
cd ~/workspace/pixel3_all/packsges/drivers/android-13.0.0_r30/pixel6
tar -xvf google_devices-oriole-tq1a.230205.002-ab978668.tgz -C ~/workspace/pixel3_all/android-13.0.0_r30/
cd ~/workspace/pixel3_all/android-13.0.0_r30/
./extract-google_devices-oriole.sh
```

输入`I ACCEPT`

### 1.4 编译aosp12

查看pixel6 android12 对应的内核版本https://source.android.com/docs/setup/build/building-kernels，这里表示的是最高版本

![image-20221122160752712](pixel6 刷 android 12.assets/image-20221122160752712.png)

![image-20221122160810939](pixel6 刷 android 12.assets/image-20221122160810939.png)

从https://android.googlesource.com/kernel/查看具体的[内核版本号](https://android.googlesource.com/kernel/gs/+refs)：**android-gs-raviole-5.10-android13-qpr1**，则make的内核的版本为5.10，



```shell
cd ~/workspace/pixel3_all/android-13.0.0_r30/
source build/envsetup.sh
lunch aosp_oriole-userdebug 
make TARGET_KERNEL_USE=5.10 -j64 RELAX_USES_LIBRARY_CHECK=true
```

注：`lunch aosp_xxx-userdebug` ,`xxx`为 google为每一个机型取的代号，pixel6 为`oriole`

清除编译结果

```
make clobber
```

### 1.5 烧录aosp13镜像

```
cd android-13.0.0_r30/out/target/product/oriole
adb reboot bootloader
fastboot flashall -w
```

查看是否刷新成功，打开手机的`setting`-`system`-`Abount phone`-`Build Number`，可以看到当前安卓的版本，即编译的指定版本`TQ1A.230205.002`

<img src="pixel6 刷 android 13.assets/image-20230308101748576.png" alt="image-20230308101748576" style="zoom:50%;" />







## **2.拉取kernel代码并编译**

查看需要的内核版本https://source.android.com/setup/build/building-kernels，

我们选择pixel6对应分支： android-gs-raviole-5.10-android13-qpr1；

### 3.1 下载内核代码

#### 将下载源切换回google

1）查看是否有REPO_URL环境变量，若有，将其修改为`https://gerrit.googlesource.com/git-repo`

```shell
#在~/.bashrc中添加 export REPO_URL="https://gerrit.googlesource.com/git-repo"
vim ~/.bashrc
#使环境变量生效
source ~/.bashrc

```



从google官网拉取kernel源码

```bash
mkdir kernel/android-gs-raviole-5.10-android13-qpr1
cd kernel/android-gs-raviole-5.10-android13-qpr1 
 ~/.bin/repo init -u https://android.googlesource.com/kernel/manifest -b android-gs-raviole-5.10-android13-qpr1
repo sync -j8
```



### 3.2 编译kernel

首先我们在AOSP源码目录下准备好编译环境 ,用于启动一些编译时所需要的环境变量，例如：交叉编译工具等

```bash
cd ~/workspace/pixel3_all/android-13.0.0_r30/
source build/envsetup.sh
lunch aosp_oriole-userdebug 
```



编译内核

```bash
cd ~/workspace/pixel3_all/kernel/android-gs-raviole-5.10-android13-qpr1
build/build.sh
```



内核二进制文件、模块和相应的映像位于 `out/BRANCH/dist` 目录下，将Image.lz4-dtb拷贝到Android13系统源码的device/google/crosshatch-kernel目录下，   

```bash
tan@tan:~/pixel3_all/kernel_andrio9/ustc/msm/out/android-msm-bluecross-4.9/dist$ ls
abi.prop       kernel-headers.tar.gz       snd-soc-cs35l36.ko          snd-soc-wcd9xxx.ko  vmlinux
dtbo.img       kernel-uapi-headers.tar.gz  snd-soc-sdm845.ko           snd-soc-wcd-spi.ko  wcd-core.ko
ftm5.ko        pinctrl-wcd.ko              snd-soc-sdm845-max98927.ko  System.map          wcd-dsp-glink.ko
Image.lz4-dtb  sec_touch.ko                snd-soc-wcd934x.ko          unstripped          wlan.ko

```



重新编译aosp

```bash
cd ~/workspace/pixel3_all/android-13.0.0_r30/
source build/envsetup.sh
lunch aosp_oriole-userdebug 
make BOARD_USERDATAIMAGE_FILE_SYSTEM_TYPE=f2fs TARGET_USERIMAGES_USE_F2FS=true -j4
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
#adb shell后 cat /proc/version ，内核版本变成了 5.10

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







