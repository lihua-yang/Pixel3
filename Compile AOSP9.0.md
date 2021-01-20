准备工作
======
Pixel3 代号blueline，编译配置lunch aosp_bluebline-userdebug，内核代码选择android-msm-crosshatch           

1.pixel3解锁
------
在手机system-->build version点击7下进入developer模式，OEM也要解锁    
手机进入BootLoader：adb reboot bootloader     
解锁命令：fastboot flashing unlock    
利用音量上下键和开机键选择对应选项来解锁      

2.准备AOSP
--------
本来想使用之前Hikey970编译的AOSP版本，但是在https://source.android.com/setup/start/build-numbers#source-code-tags-and-builds 查找细分版本号发现，上次repo sync的r8版本不能用来编译Pixel 3的镜像，只好再repo sync一个相近的r21版本。       
同时在github上找到前人的教程并学习：https://github.com/shallin123/Android9.0-pixel-3  
可供参考的pixel XL编译经验： https://blog.csdn.net/zz531987464/article/details/94163954    

//repo init -u https://aosp.tuna.tsinghua.edu.cn/platform/manifest -b android-9.0.0_r21 --no-repo-verify --repo-branch=stable    
repo init -u https://aosp.tuna.tsinghua.edu.cn/platform/manifest -b android-9.0.0_r21    
//清华的源不可用，换用中科院的源ustc    
repo init -u git://mirrors.ustc.edu.cn/aosp/platform/manifest -b android-9.0.0_r21     
repo sync -j32     
log：
```
Checking out projects: 100% (678/678), done.
repo sync has finished successfully.
```

again：
-----
repo init -u https://aosp.tuna.tsinghua.edu.cn/platform/manifest -b android-9.0.0_r37 --no-repo-verify --repo-branch=stable    
失败
repo init -u git://mirrors.ustc.edu.cn/aosp/platform/manifest -b android-9.0.0_r37     
repo sync
```
Checking out projects:  90% (613/677) platform/prebuilts/r8Checking out files:   0% (18/4089Checking out files: 100% (4089/4089), done.
Checking out projects: 100% (677/677), done.
repo sync has finished successfully.
```


3.编译aosp
--------
source build/envsetup.sh     
lunch aosp_blueline-userdebug   
log:  
```
root@tan-PowerEdge-R730:/mnt/pixel3/aosp# lunch aosp_blueline-userdebug     
============================================    
PLATFORM_VERSION_CODENAME=REL   
PLATFORM_VERSION=9    
TARGET_PRODUCT=aosp_blueline    
TARGET_BUILD_VARIANT=userdebug   
TARGET_BUILD_TYPE=release   
TARGET_ARCH=arm64    
TARGET_ARCH_VARIANT=armv8-2a    
TARGET_CPU_VARIANT=cortex-a75    
TARGET_2ND_ARCH=arm    
TARGET_2ND_ARCH_VARIANT=armv8-a    
TARGET_2ND_CPU_VARIANT=cortex-a75    
HOST_ARCH=x86_64   
HOST_2ND_ARCH=x86    
HOST_OS=linux    
HOST_OS_EXTRA=Linux-4.13.0-x86_64-Ubuntu-16.04.7-LTS    
HOST_CROSS_OS=windows   
HOST_CROSS_ARCH=x86    
HOST_CROSS_2ND_ARCH=x86_64    
HOST_BUILD_TYPE=release    
BUILD_ID=PQ1A.181205.006   
OUT_DIR=out    
PRODUCT_SOONG_NAMESPACES=device/google/crosshatch/pixelstats device/google/crosshatch/usb device/google/crosshatch/health hardware/google/av hardware/google/interfaces hardware/qcom/sdm845 vendor/qcom/sdm845     
```
make -j32    
```
Copying resources from program jar [/mnt/pixel3/aosp/out/target/common/obj/APPS/messaging_intermediates/classes.jar]
[ 99% 105415/105439] Target Java: out/target/common/obj/APPS/Dialer_intermediates/classes-full-debug.jar
Note: Generating a Provider for com.android.dialer.glidephotomanager.impl.GlidePhotoManagerImpl. Prefer to run the dagger processor over that class instead.
Note: [1] Wrote GeneratedAppGlideModule with: []
[100% 105439/105439] Target vbmeta image: out/target/product/blueline/vbmeta.img

#### build completed successfully (01:46:12 (hh:mm:ss)) ####
```
用xftp传输/mnt/pixel3/aosp/out/target/product/blueline# 文件夹并刷录，看pixel3是否刷录成功
adb reboot bootloader    
fastboot flashall -w    

again:
------
```root@ca01:/mnt/sdb/aosp-pixels# lunch aosp_blueline-userdebug
============================================
PLATFORM_VERSION_CODENAME=REL
PLATFORM_VERSION=9
TARGET_PRODUCT=aosp_blueline
TARGET_BUILD_VARIANT=userdebug
TARGET_BUILD_TYPE=release
TARGET_ARCH=arm64
TARGET_ARCH_VARIANT=armv8-2a
TARGET_CPU_VARIANT=cortex-a75
TARGET_2ND_ARCH=arm
TARGET_2ND_ARCH_VARIANT=armv8-a
TARGET_2ND_CPU_VARIANT=cortex-a75
HOST_ARCH=x86_64
HOST_2ND_ARCH=x86
HOST_OS=linux
HOST_OS_EXTRA=Linux-3.11.10-x86_64-Ubuntu-14.04.5-LTS
HOST_CROSS_OS=windows
HOST_CROSS_ARCH=x86
HOST_CROSS_2ND_ARCH=x86_64
HOST_BUILD_TYPE=release
BUILD_ID=PQ3A.190505.002
OUT_DIR=out
PRODUCT_SOONG_NAMESPACES=device/google/crosshatch hardware/google/av hardware/google/interfaces hardware/qcom/sdm845 vendor/qcom/sdm845
============================================
```
make -j32    
```
[ 99% 105474/105503] //art/compiler:libart-compiler link libart-compiler.so [arm]
Warning: request a ThreadPool with 1 threads, but LLVM_ENABLE_THREADS has been turned off
[ 99% 105487/105503] //art/dex2oat:dex2oat link dex2oat [arm]
Warning: request a ThreadPool with 1 threads, but LLVM_ENABLE_THREADS has been turned off
[100% 105503/105503] Target vbmeta image: out/target/product/blueline/vbmeta.img

#### build completed successfully (02:25:34 (hh:mm:ss)) ####
```

这一步很重要，也要做！

3.2 下载对应版本的驱动文件   
-----
我下载的版本是：       
//PD1A.180720.031	android-9.0.0_r12	Pie	Pixel 3 XL、Pixel 3	2018-09-05    
PQ1A.181205.006	android-9.0.0_r21	Pie	Pixel 3 XL、Pixel 3	2018-12-05     
下载驱动  
https://developers.google.com/android/drivers      
Pixel 3 binaries for Android 9.0.0 (PQ1A.181205.006)     
Hardware Component 	Company 	Download 	SHA-256 Checksum     
Vendor image 	      Google 	Link 	3d0e9929ae29c7d87dd4790fe5223429197fcdd36bf3f56e687fe37802a9c1eb     
GPS, Audio, Camera, Gestures, Graphics, DRM, Video, Sensors 	Qualcomm 	Link 	9db7f8c312ece6833c623e2d75f4bcc1036c521279a3f4b55e20b1819cc00113      
所以     
wget https://dl.google.com/dl/android/aosp/google_devices-blueline-pq1a.181205.006-5a3e2737.tgz     
wget https://dl.google.com/dl/android/aosp/qcom-blueline-pq1a.181205.006-e364e5c0.tgz             
并用tar zxvf 解压并执行解压出来的两个sh文件    

again:
------
总觉得缺少一些东西，所以将这一步也加上    
Pixel 3 binaries for Android 9.0.0 (PQ3A.190505.002)
Hardware Component 	Company 	Download 	SHA-256 Checksum
Vendor image 	Google 	Link 	84924ae6fde78d5c5c086cc896dd2745d27b33c87ff52b8ba0eb3fd5a73b0487
GPS, Audio, Camera, Gestures, Graphics, DRM, Video, Sensors 	Qualcomm 	Link 	ab64e900a6ceb7568edb73eff473d61a0b0d29d363f466f8a6c68829e128b516

4.刷录镜像
--------
打开usb调试，进入BootLoader模式    
adb reboot bootloader    
在out/target/product/blueline目录烧入编译生成的所有镜像    
fastboot flashall -w    
报错1：    
```
can't find android-info.txt，明明在当前目录下就能找到android-info.txt
```
vim /etc/profile      
#change by ylh 2021-1-16     
export ANDROID_PRODUCT_OUT=/mnt/pixel3/aosp/out/target/product/blueline    
#export ANDROID_PRODUCT_OUT=/mnt/pixel3/WORKING_DIRECTORY/out/target/product/blueline    
#export ANDROID_PRODUCT_OUT=/mnt/tsy/WORKING_DIRECTORY/out/target/product/flounder    

报错2：   
```
FAILED (remote: 'Partition should be flashed in fastbootd')
```
初步推测与fastboot版本相关，与淘宝店家商量，获得原始版本的全套镜像，又因为与大机房上Hikey970与Nexus9的编译刷录想冲突，将Pixel3的编译刷录转移到ca01机器上。    

again：
-----
```
root@ca01:/mnt/sdb/aosp-pixels/out/target/product/blueline# fastboot flashall -w
--------------------------------------------
Bootloader Version...: b1c1-0.1-5343672
Baseband Version.....: g845-00017-190312-B-5369743
Serial Number........: 89AX07GLX
--------------------------------------------
checking product...
OKAY [  0.060s]
sending 'boot' (65536 KB)...
OKAY [  2.210s]
writing 'boot'...
FAILED (remote: Failed to write to partition Not Found)
finished. total time: 2.876s
```

again-again（加上上述两个驱动后重新编译并刷录镜像）：
```
root@ca01:/mnt/sdb/aosp-pixels/out/target/product/blueline# fastboot flashall -w
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
OKAY [  2.210s]
Writing 'boot_a'...
OKAY [  0.691s]
Sending 'dtbo_a' (8192 KB)...
OKAY [  0.359s]
Writing 'dtbo_a'...
OKAY [  0.101s]
Sending 'product_a' (4956 KB)...
OKAY [  0.259s]
Writing 'product_a'...
OKAY [  0.150s]
Sending sparse 'system_a' 1/5 (262140 KB)...
OKAY [  8.610s]
Writing 'system_a' 1/5...
OKAY [  0.366s]
Sending sparse 'system_a' 2/5 (262140 KB)...
OKAY [  8.604s]
Writing 'system_a' 2/5...
OKAY [  0.366s]
Sending sparse 'system_a' 3/5 (262140 KB)...
OKAY [  8.604s]
Writing 'system_a' 3/5...
OKAY [  0.366s]
Sending sparse 'system_a' 4/5 (262140 KB)...
OKAY [  8.604s]
Writing 'system_a' 4/5...
OKAY [  0.366s]
Sending sparse 'system_a' 5/5 (135148 KB)...
OKAY [  4.484s]
Writing 'system_a' 5/5...
OKAY [  1.160s]
Sending 'system_b' (87028 KB)...
OKAY [  2.910s]
Writing 'system_b'...
OKAY [  0.930s]
Sending 'vbmeta_a' (4 KB)...
OKAY [  0.120s]
Writing 'vbmeta_a'...
OKAY [  0.063s]
Sending sparse 'vendor_a' 1/2 (262140 KB)...
OKAY [  8.587s]
Writing 'vendor_a' 1/2...
OKAY [  0.366s]
Sending sparse 'vendor_a' 2/2 (193416 KB)...
OKAY [  6.354s]
Writing 'vendor_a' 2/2...
OKAY [  1.390s]
Setting current slot to 'a'...
OKAY [  0.274s]
Erasing 'userdata'...
OKAY [  6.205s]
Erasing 'metadata'...
OKAY [  0.283s]
Rebooting...

Finished. Total time: 75.173s
```
成功启动，刷录aosp镜像成功！！！！    

5.下载内核代码  
-------
mkdir pixel3-kernel     
cd pixel3-kernel     
repo init -u https://aosp.tuna.tsinghua.edu.cn/kernel/manifest -b android-msm-crosshatch-4.9-pie-qpr2    
repo sync -j32    
我的做法是，在https://android.googlesource.com/kernel/msm-extra/+refs 下载android-msm-crosshatch-4.9-pie-qpr2压缩包，并用xftp传输到大机房机器上    
上面这条失败了，换了一个方法    
参考该网站 https://blog.csdn.net/zz531987464/article/details/94163954      
git clone https://aosp.tuna.tsinghua.edu.cn/kernel/msm.git     
cd msm     
git checkout remotes/origin/android-msm-crosshatch-4.9-pie-qpr2  

again：
-----
该方法编译出错，尝试官网构建内核的方法       
https://source.android.com/setup/build/building-kernels      
Pixel 3 (blueline)            AOSP 树中的二进制文件路径              Repo 分支       
Pixel 3 XL (crosshatch)	      device/google/crosshatch-kernel	    android-msm-crosshatch-4.9-android11     
git checkout remotes/origin/android-msm-crosshatch-4.9-android11    
发现没有build/build.sh, repo sync android-msm-crosshatch-4.9-pie-qpr2    
```
Checking out files: 100% (16166/16166), done.
Checking out projects:  40% (4/10) platform/prebuilts/gcc/linux-x86/aarch64/aarch64-linux-and
Checking out projects:  50% (5/10) platform/prebuilts/gcc/linux-x86/arm/arm-linux-androideabi
Checking out projects: 100% (10/10), done.
repo sync has finished successfully.
```
sync后有ipc文件夹一样，且其中的文件和Makefile文件不一样，将其他文件夹复制到msm文件夹下，此时msm中 git branch为android-msm-crosshatch-4.9-pie-qpr2    
还有build.config文件一样，不做拷贝（Makefile）    
还有include文件夹一样，同上   
先尝试一次与Hikey970中类似的编译方法：make b1c1_defconfig    
make 

6.编译内核
--------
关闭内核编译选项模块签名和模块对内核函数符号crc检验选项    

cd /mnt/pixel3/pixel3-kernel/msm/arch/arm64/configs     
修改b1c1_defconfig     
关于MODULE的定义   
```
CONFIG_MODULES=y
CONFIG_MODULE_UNLOAD=y
CONFIG_MODULE_FORCE_UNLOAD=y
#change by ylh CONFIG_MODVERSIONS=y
CONFIG_MODULE_SIG=y
#change by ylh
#CONFIG_MODULE_SIG_ALL is not set
#CONFIG_MODULE_SIG_FORCE=y
#CONFIG_MODULE_SIG_SHA512=y
#CONFIG_BLK_DEV_BSG is not set
```
对build/build.sh编译脚本做如下修改，不然编译会报错。 
   

修改/mnt/pixel3/pixel3-kernel/msm/build/build.sh    
在这之前msm中没有build文件夹，只好repo sync 一次    
```
if [ -n "${POST_DEFCONFIG_CMDS}" ]; then
    echo "========================================================"
    echo " Running pre-make command(s):"
    set -x
#change by ylh
#    eval ${POST_DEFCONFIG_CMDS}
    set +x
  fi
fi
```
关闭内核版本检测（versionmagic）检测   
/mnt/pixel3/pixel3-kernel/msm/kernel/module.c
```
static int check_modinfo(struct module *mod, struct load_info *info, int flags)
{
        const char *modmagic = get_modinfo(info, "vermagic");
        int err;

        if (flags & MODULE_INIT_IGNORE_VERMAGIC)
                modmagic = NULL;

        /* This is allowed: modprobe --force will invalidate it. */
        if (!modmagic) {
                err = try_to_force_load(mod, "bad vermagic");
                if (err)
                        return err;
        } else if (!same_magic(modmagic, vermagic, info->index.vers)) {
                pr_err("%s: version magic '%s' should be '%s'\n",
                       mod->name, modmagic, vermagic);
                //change by ylh
                //return -ENOEXEC;
        }
```

报错1：     
```
/mnt/pixel3/pixel3-kernel/msm/build# ./build.sh 
= Set default KERNEL_DIR: /mnt/pixel3/pixel3-kernel/msm
/mnt/pixel3/pixel3-kernel/msm/build/build.config: line 5: /mnt/pixel3/pixel3-kernel/msm/build/private/msm-google/build.config.common.clang: No such file or directory
```

将arch/arm64/boot/Image.lz4-dtb文件复制到AOSP的device/google/crosshatch-kernel目录下，然后在AOSP源码根目录下执行 make bootimage 生成最终的boot.img 。    
或者说将内核源码out/android-msm-crosshatch-4.9/dist目录下的Image.lz4-dtb拷贝到Android9系统源码的device/google/crosshatch-kernel目录下     

source build/envsetup.sh   
lunch aosp_blueline-userdebug    
make bootimage    
fastboot flash boot boot.img    
adb shell后 cat /proc/version    

报错2：
```
root@ca01:/mnt/sdb/android-kernel/msm# make
arch/arm64/Makefile:68: CROSS_COMPILE_ARM32 not defined or empty, the compat vDSO will not be built
```
将error改为warning    

报错3：
```
root@ca01:/mnt/sdb/android-kernel/msm# make
arch/arm64/Makefile:68: CROSS_COMPILE_ARM32 not defined or empty, the compat vDSO will not be built
  CHK     include/config/kernel.release
Cannot use CONFIG_LTO_CLANG: requires clang 5.0 or later
make: *** [Makefile:1196: prepare-compiler-check] Error 1
```
将prebuilts下/bin文件夹下的交叉编译工具拷贝到msm下    
export ARCH=arm64    
root@ca01:/mnt/sdb/android-kernel/msm# export CROSS_COMPILE=aarch64-linux-android-    
root@ca01:/mnt/sdb/android-kernel/msm# export CROSS_COMPILE_ARM32=./bin/arm-linux-androideabi-    
root@ca01:/mnt/sdb/android-kernel/msm# make b1c1_defconfig   



报错4：
```
/mnt/sdb/android-kernel/build/_setup_env.sh: line 46: realpath: command not found
```
回到上一层repo sync的目录，而不是msm那一层执行build/build.sh     
