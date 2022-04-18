# pixel3  从安卓12回滚到安卓9

1.下载pixel3对应安卓9出厂镜像

https://developers.google.com/android/images

https://dl.google.com/dl/android/aosp/blueline-pq1a.181205.006-factory-af1829a6.zip

2.解压1下载的镜像压缩包，执行flash-all脚本

## 问题

手机bootloader版本和线刷包不匹配

![image-20220418154221878](pixel3%20%20%E4%BB%8E%E5%AE%89%E5%8D%9312%E5%9B%9E%E6%BB%9A%E5%88%B0%E5%AE%89%E5%8D%939.assets/image-20220418154221878.png)

### 方案1（无效）：

修改线刷包image-blueline-pq1a.181205.006.zip中的android-info.txt文件，require version-bootloader=b1c1-0.4-7266295，重新执行flash-all脚本，还是版本不匹配被

![image-20220418154946690](pixel3%20%20%E4%BB%8E%E5%AE%89%E5%8D%9312%E5%9B%9E%E6%BB%9A%E5%88%B0%E5%AE%89%E5%8D%939.assets/image-20220418154946690.png)





![image-20220418155021502](pixel3%20%20%E4%BB%8E%E5%AE%89%E5%8D%9312%E5%9B%9E%E6%BB%9A%E5%88%B0%E5%AE%89%E5%8D%939.assets/image-20220418155021502.png)



## 方案2（有效）：

解决方案：将方案1中修改的内容回滚，再执行一遍flash-all脚本！！！！！

```shell
Sending 'bootloader_a' (8489 KB)                   OKAY [  0.488s]
Writing 'bootloader_a'                             (bootloader) Flashing Pack version b1c1-0.1-5034669
(bootloader) Flashing partition table for Lun = 0
(bootloader) Flashing partition table for Lun = 1
(bootloader) Flashing partition table for Lun = 2
(bootloader) Flashing partition table for Lun = 4
(bootloader) Flashing partition table for Lun = 5
(bootloader) Flashing partition msadp_a
(bootloader) Flashing partition xbl_a
(bootloader) Flashing partition xbl_config_a
(bootloader) Flashing partition aop_a
(bootloader) Flashing partition tz_a
(bootloader) Flashing partition hyp_a
(bootloader) Flashing partition abl_a
(bootloader) Flashing partition keymaster_a
(bootloader) Flashing partition cmnlib_a
(bootloader) Flashing partition cmnlib64_a
(bootloader) Flashing partition devcfg_a
(bootloader) Flashing partition qupfw_a
(bootloader) Flashing partition storsec_a
(bootloader) Flashing partition logfs
OKAY [  0.993s]
Finished. Total time: 1.724s
Rebooting into bootloader                          OKAY [  0.049s]
Finished. Total time: 0.052s
Sending 'radio_a' (71148 KB)                       OKAY [  1.829s]
Writing 'radio_a'                                  (bootloader) Flashing Pack version SSD:g845-00023-180917-B-5014671
(bootloader) Flashing partition modem_a
OKAY [  0.479s]
Finished. Total time: 2.550s
Rebooting into bootloader                          OKAY [  0.050s]
Finished. Total time: 0.051s
--------------------------------------------
Bootloader Version...: b1c1-0.1-5034669
Baseband Version.....: g845-00023-180917-B-5014671
Serial Number........: 89AX07GLX
--------------------------------------------
extracting android-info.txt (0 MB) to RAM...
Checking 'product'                                 OKAY [  0.059s]
Checking 'version-bootloader'                      OKAY [  0.059s]
Checking 'version-baseband'                        OKAY [  0.059s]
Setting current slot to 'a'                        OKAY [  0.323s]
extracting boot.img (64 MB) to disk... took 0.426s
archive does not contain 'boot.sig'
Sending 'boot_a' (65536 KB)                        OKAY [  1.569s]
Writing 'boot_a'                                   OKAY [  0.389s]
archive does not contain 'init_boot.img'
extracting dtbo.img (8 MB) to disk... took 0.016s
archive does not contain 'dtbo.sig'
Sending 'dtbo_a' (8192 KB)                         OKAY [  0.280s]
Writing 'dtbo_a'                                   OKAY [  0.103s]
archive does not contain 'dt.img'
archive does not contain 'pvmfw.img'
archive does not contain 'recovery.img'
extracting vbmeta.img (0 MB) to disk... took 0.001s
archive does not contain 'vbmeta.sig'
Sending 'vbmeta_a' (4 KB)                          OKAY [  0.120s]
Writing 'vbmeta_a'                                 OKAY [  0.064s]
archive does not contain 'vbmeta_system.img'
archive does not contain 'vbmeta_vendor.img'
archive does not contain 'vendor_boot.img'
archive does not contain 'super_empty.img'
archive does not contain 'boot_other.img'
archive does not contain 'odm.img'
archive does not contain 'odm_dlkm.img'
extracting product.img (195 MB) to disk... took 1.487s
archive does not contain 'product.sig'
Sending 'product_a' (200036 KB)                    OKAY [  4.590s]
Writing 'product_a'                                OKAY [  1.129s]
extracting system.img (2157 MB) to disk... took 34.183s
archive does not contain 'system.sig'
Sending sparse 'system_a' 1/9 (262140 KB)          OKAY [  6.737s]
Writing 'system_a'                                 OKAY [  0.367s]
Sending sparse 'system_a' 2/9 (262140 KB)          OKAY [  6.981s]
Writing 'system_a'                                 OKAY [  0.366s]
Sending sparse 'system_a' 3/9 (262140 KB)          OKAY [  7.079s]
Writing 'system_a'                                 OKAY [  0.366s]
Sending sparse 'system_a' 4/9 (262140 KB)          OKAY [  7.022s]
Writing 'system_a'                                 OKAY [  0.367s]
Sending sparse 'system_a' 5/9 (262140 KB)          OKAY [  7.011s]
Writing 'system_a'                                 OKAY [  0.367s]
Sending sparse 'system_a' 6/9 (262140 KB)          OKAY [  7.171s]
Writing 'system_a'                                 OKAY [  0.367s]
Sending sparse 'system_a' 7/9 (262140 KB)          OKAY [  7.090s]
Writing 'system_a'                                 OKAY [  0.366s]
Sending sparse 'system_a' 8/9 (258820 KB)          OKAY [  7.140s]
Writing 'system_a'                                 OKAY [  0.365s]
Sending sparse 'system_a' 9/9 (115360 KB)          OKAY [  3.181s]
Writing 'system_a'                                 OKAY [  0.978s]
archive does not contain 'system_dlkm.img'
archive does not contain 'system_ext.img'
extracting system_other.img (418 MB) to disk... took 6.684s
archive does not contain 'system.sig'
Sending sparse 'system_b' 1/2 (262140 KB)          OKAY [  5.940s]
Writing 'system_b'                                 OKAY [  0.366s]
Sending sparse 'system_b' 2/2 (166540 KB)          OKAY [  3.971s]
Writing 'system_b'                                 OKAY [  1.299s]
extracting vendor.img (444 MB) to disk... took 6.414s
archive does not contain 'vendor.sig'
Sending sparse 'vendor_a' 1/2 (262140 KB)          OKAY [  5.979s]
Writing 'vendor_a'                                 OKAY [  0.365s]
Sending sparse 'vendor_a' 2/2 (193516 KB)          OKAY [  4.580s]
Writing 'vendor_a'                                 OKAY [  1.370s]
archive does not contain 'vendor_dlkm.img'
archive does not contain 'vendor_other.img'
Erasing 'userdata'                                 OKAY [  6.536s]
Erase successful, but not automatically formatting.
File system type raw not supported.
Erasing 'metadata'                                 OKAY [  0.263s]
Erase successful, but not automatically formatting.
File system type raw not supported.
Rebooting                                          OKAY [  0.058s]
Finished. Total time: 158.083s
Press any key to exit...
```





