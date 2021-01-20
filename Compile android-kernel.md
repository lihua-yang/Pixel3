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

清华的镜像Git出错,换用ustc的镜像     
git clone git://mirrors.ustc.edu.cn/aosp/kernel/msm.git     

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
export CROSS_COMPILE=aarch64-linux-android-    
export CROSS_COMPILE_ARM32=arm-linux-androideabi-    
make b1c1_defconfig   



报错4：
```
/mnt/sdb/android-kernel/build/_setup_env.sh: line 46: realpath: command not found
```
回到上一层repo sync的目录，而不是msm那一层执行build/build.sh     
