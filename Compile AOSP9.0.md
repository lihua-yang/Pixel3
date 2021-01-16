准备工作
======
Pixel3 代号blueline，编译配置lunch aosp_bluebline-userdebug，内核代码选择android-msm-crosshatch           

pixel3解锁
==========
在手机system-->build version点击7下进入developer模式，OEM也要解锁    
手机进入BootLoader：adb reboot bootloader     
解锁命令：fastboot flashing unlock    
利用音量上下键和开机键选择对应选项来解锁      

1.准备AOSP，本来想使用之前Hikey970编译的AOSP版本，但是在https://source.android.com/setup/start/build-numbers#source-code-tags-and-builds查找细分版本号发现，上次repo sync的r8版本
不能用来编译Pixel 3的镜像，只好再repo sync一个相近的r21版本。      
同时在github上找到前人的教程并学习：https://github.com/shallin123/Android9.0-pixel-3         

//repo init -u https://aosp.tuna.tsinghua.edu.cn/platform/manifest -b android-9.0.0_r21 --no-repo-verify --repo-branch=stable
repo init -u https://aosp.tuna.tsinghua.edu.cn/platform/manifest -b android-9.0.0_r21
repo sync -j32     

2.下载对应版本的驱动文件     
我下载的版本是：     
//PD1A.180720.031	android-9.0.0_r12	Pie	Pixel 3 XL、Pixel 3	2018-09-05   
PQ1A.181205.006	android-9.0.0_r21	Pie	Pixel 3 XL、Pixel 3	2018-12-05    

下载驱动    
https://developers.google.com/android/drivers     
Pixel 3 binaries for Android 9.0.0 (PQ1A.181205.006)    
Hardware Component 	Company 	Download 	SHA-256 Checksum    
Vendor image 	Google 	Link 	3d0e9929ae29c7d87dd4790fe5223429197fcdd36bf3f56e687fe37802a9c1eb    
GPS, Audio, Camera, Gestures, Graphics, DRM, Video, Sensors 	Qualcomm 	Link 	9db7f8c312ece6833c623e2d75f4bcc1036c521279a3f4b55e20b1819cc00113   
  

所以     
wget https://dl.google.com/dl/android/aosp/google_devices-blueline-pq1a.181205.006-5a3e2737.tgz     
wget https://dl.google.com/dl/android/aosp/qcom-blueline-pq1a.181205.006-e364e5c0.tgz     
          
并用tar zxvf 解压并执行解压出来的两个sh文件     



5.下载内核代码
mkdir pixel3-kernel     
cd pixel3-kernel     
repo init -u https://aosp.tuna.tsinghua.edu.cn/kernel/manifest -b android-msm-crosshatch-4.9-pie-qpr2    
repo sync -j32    
我的做法是，在https://android.googlesource.com/kernel/msm-extra/+refs下载android-msm-crosshatch-4.9-pie-qpr2压缩包，并用xftp传输到大机房机器上     
