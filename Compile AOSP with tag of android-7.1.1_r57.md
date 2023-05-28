# Compile AOSP with tag of android-7.1.1_r57[1]

下载`android-7.1.1_r57`源码及其二进制驱动，参考[pixel3 刷安卓12](https://github.com/Ying-Yuan07/pixel3/blob/main/pixel3%20%E5%88%B7%E5%AE%89%E5%8D%9312.md)  

## 环境要求

`java 8`,`python2.7`,`xmllint`,其中`java` 需要解禁`SSLv3,TLSv1,TLSv1.1`[2] [3]

- 安装`java 8`,`python2.7`自行找教程


```shell
# 安装xmllint工具
sudo apt-get install libxml2-utils
```

- `java` 需要解禁`SSLv3,TLSv1,TLSv1.1`[2] [3]

修改`/usr/lib/jvm/java-8-openjdk-amd64/jre/lib/security/java.security`,删除`jdk.tls.disabledAlgorithms`参数中的`SSLv3, TLSv1, TLSv1.1`

```py
#   jdk.tls.disabledAlgorithms=SSLv3, TLSv1, TLSv1.1, RC4, DES, MD5withRSA, \
jdk.tls.disabledAlgorithms=RC4, DES, MD5withRSA, \
    DH keySize < 1024, EC keySize < 224, 3DES_EDE_CBC, anon, NULL, \
    include jdk.disabled.namedCurves
```

重启服务器，使被修改的java参数生效

## 编译AOSP

```shell
cd android-7.1.1_r57 # android-7.1.1_r57 为aosp源码路径
export LC_ALL=C #去除所有本地化设置
source build/envsetup.sh
lunch aosp_x86-eng #按照自己的需求选择编译版本，aosp_x86-eng对应x86模拟器版本
make -j64 #线程数一般设置为host的线程数
```

## Errors

### (1) SSL error with Jack server (35), try 'jack-diagnose'[3]

解禁`SSLv3,TLSv1,TLSv1.1`[2] [3]

### （2）others[1]

## refs

[1] 符哥2008, ubuntu 18.04编译Android 7.1源码, https://blog.csdn.net/qq_35460159/article/details/82557365?spm=1001.2101.3001.6650.2&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-2-82557365-blog-76263317.235%5Ev36%5Epc_relevant_default_base3&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-2-82557365-blog-76263317.235%5Ev36%5Epc_relevant_default_base3&utm_relevant_index=1    ,2018.11.09.

[2]  海月汐辰, Android编译错误 Jack server failed to ,SSL error when connecting to the Jack server. Try ‘jack-diagnose‘,  https://blog.csdn.net/qq_37858386/article/details/119599118 , 2021.08.11.

[3] Atom, JAVA升级导致Android编译时Jack server出错, https://segmentfault.com/a/1190000039951447 , 2021.05.06.
