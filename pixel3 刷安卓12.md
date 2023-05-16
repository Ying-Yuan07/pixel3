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



## others

### 内核config

从android12内核的编译脚本`build/build.sh`可以看出，编译config文件是`private/msm-google/build.config.bluecross`

```shell
DEFCONFIG=b1c1_defconfig
KERNEL_DIR=private/msm-google
. ${ROOT_DIR}/${KERNEL_DIR}/build.config.common.clang
POST_DEFCONFIG_CMDS="check_defconfig"
```

config 文件最终读的是`private/msm-google/arch/arm64/configs/b1c1_defconfig`

`POST_DEFCONFIG_CMDS="check_defconfig"`表示进行相容性检查

想要修改配置，可以修改`b1c1_defconfig`文件，删除编译脚本`private/msm-google/build.config.bluecross`中`POST_DEFCONFIG_CMDS="check_defconfig"`，重新编译内核，；

**注**：修改完`b1c1_defconfig`，直接编译内核，可能会报错

```shell
++ echo ERROR: savedefconfig does not match private/msm-google/arch/arm64/configs/b1c1_defconfig
ERROR: savedefconfig does not match private/msm-google/arch/arm64/configs/b1c1_defconfig
```

https://www.akr-developers.com/d/526-pixel3-aosp

解决方案：去除编译脚本中检查部分

