1、编译AOSP
--------
mkdir aosp
cd aosp    
repo init -u git://mirrors.ustc.edu.cn/aosp/platform/manifest -b android-9.0.0_r21    
repo sync -j32   
下载对应版本的驱动并解压与执行：    
wget https://dl.google.com/dl/android/aosp/google_devices-blueline-pq1a.181205.006-5a3e2737.tgz   
wget https://dl.google.com/dl/android/aosp/qcom-blueline-pq1a.181205.006-e364e5c0.tgz    

source build/envsetup.sh    
lunch aosp_blueline-userdebug    
make -j32   

adb reboot bootloader     
fastboot flashall -w     

2、编译内核
--------
mkdir android-kernel      
cd android-kernel    

