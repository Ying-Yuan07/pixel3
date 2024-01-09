# 下载、编译android-7.1.1_r57

## 下载android-7.1.1_r57

```shell
mkdir android-7.1.1_r57
cd android-7.1.1_r57
~/.bin/repo init -u git://mirrors.ustc.edu.cn/aosp/platform/manifest -b android-7.1.1_r57
~/.bin/repo sync
```



## 编译android-7.1.1_r57

1.  下载二进制驱动 ：https://developers.google.cn/android/drivers?hl=zh-cn#shamungi77b ,放入android-7.1.1_r57源码路径，并依次执行脚本，下载Nexus6 对应二进制驱动

   ```shell
   cd android-7.1.1_r57
   ./extract-broadcom-shamu.sh
   ./extract-moto-shamu.sh
   ./extract-qcom-shamu.sh
   ```

   

2. 切换java版本到openjdk-``8``-jdk

3. 切换python版本到python2.7

   ```shell
   #查看python版本，如果是3.0一下，则切换成3.X
   #通过软链接方式切换python版本
   sudo rm /usr/bin/python
   sudo ln -s /usr/bin/python2.7 /usr/bin/python
   ```

   

4. 将当前shell的所有本地设置（locale settings）都设为C语言环境[1]

   ```shell
   cd android-7.1.1_r57
   export LC_ALL=C make
   ```

5. 编译

   ```shell
   cd android-7.1.1_r57
   source build/envsetup.sh
   lunch aosp_shamu-userdebug
   make
   ```
   
   

## refs

[1]Android 7.1.1-Nexus 6 实战。 https://www.cnblogs.com/common1140/p/9508293.html

[2] https://stackoverflow.com/questions/54547246/recipe-for-target-ninja-wrapper-failed-flex-core-dumps