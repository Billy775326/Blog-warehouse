[English version](https://ivonblog.com/en-us/posts/redroid-android-docker/)

ReDroid (Remote anDroid) 是自架「雲手機」的方案，透過Docker跑Android系統容器，再利用Scrcpy的鏡射螢幕功能連線到Android桌面。

![](https://ivonblog.com/posts/redroid-android-docker/images/FQXt3GC.webp)

在Linux用ReDroid玩FGO

ReDroid也是在電腦上用**開源軟體**跑Android APP的解決方案。因為別說雲手機了，很多Android手遊模擬器都是**閉源軟體**哪。相較之下，ReDroid除了ARM轉譯器以外都是開源的。更棒的是ReDroid支援GPU加速＋ARM轉x86的轉譯器，這樣就可以玩大多數手機3D遊戲了。

對Linux用戶來說，這更是除了 [Waydroid和Android-x86虛擬機](https://ivonblog.com/posts/android-emulators-for-linux/)以外，在Linux電腦高效率跑Android APP的方法。並且它比Waydroid更適合當作雲手機使用。參見 [自架雲手機](https://ivonblog.com/posts/scrcpy-app-remote-control-android-in-cloud/)

本文將討論如何在x86架構的Linux電腦，用ReDroid玩ARM架構的手機遊戲。我們會在ReDroid映像檔加入ARM轉譯器＋Google服務框架，以達成最佳使用體驗。

## 1\. ReDroid環境需求[#](#1-redroid%E7%92%B0%E5%A2%83%E9%9C%80%E6%B1%82)

任一Linux發行版應該都可以。

本文示範使用x86架構的電腦跑ReDroid。至於ARM架構的電腦，有網友回報Oracle ARM架構伺服器部署成功的案例，我也測試過 [可以在樹莓派5跑ReDroid](https://ivonblog.com/posts/redroid-on-raspberry-pi)。

如果電腦是x86架構，則只能執行x86架構的Android APP，然而很多手機遊戲只有ARM架構版本，所以ReDroid需要裝libndk或libhoudini的ARM轉譯器。

要玩手遊建議電腦至少8GB以上RAM，因為有時ARM在轉譯成x86指令時會佔用大量RAM。

GPU加速部份，建議使用Intel或AMD的GPU，3D加速開箱即用，Nvidia不建議。

* * *

關於Android版本，Redroid作者發布的`redroid:11.0.0-latest`和`redroid:12.0.0-latest`映像檔已內建Google開發的ARM轉譯器libndk，我試過只有Android 11比較穩定，GApps也可以用，所以本文選用Android 11的映像檔。

## 2\. 安裝ReDroid前置依賴項目[#](#2-%E5%AE%89%E8%A3%9Dredroid%E5%89%8D%E7%BD%AE%E4%BE%9D%E8%B3%B4%E9%A0%85%E7%9B%AE)

ReDroid的 [Github](https://github.com/remote-android/redroid-doc/blob/master/deploy/README.md)有各大Linux發行版的安裝說明，我使用Ubuntu系統做示範。

1.  首先要準備binder核心模組。Ubuntu 24.04執行以下指令，安裝必要的核心模組：

```bash
sudo apt install linux-modules-extra-`uname -r`

sudo modprobe binder_linux devices="binder,hwbinder,vndbinder"
```

2.  將以上核心模組加入開機自動載入

```bash
sudo cat <<EOT >> /etc/modules-load.d/redroid.conf
binder_linux
options binder_linux devices="binder,hwbinder,vndbinder"
EOT
```

3.  安裝 [Docker](https://ivonblog.com/posts/install-docker-engine-on-linux/)，用於執行容器
    
4.  安裝 [ADB](https://developer.android.com/tools/releases/platform-tools?hl=zh-tw)工具
    
5.  最後安裝 [Scrcpy](https://ivonblog.com/posts/scrcpy-usage)。
    

若想要整合按鍵映射的圖形化Scrcpy界面，可以改裝 [QtScrcpy](https://ivonblog.com/posts/android-qtscrcpy-usage/)。您還可以嘗試網頁版的 [ws-scrcpy](https://ivonblog.com/posts/ws-scrcpy/)。

## 3\. 預先給ReDroid映像檔安裝GApps[#](#3-%E9%A0%90%E5%85%88%E7%B5%A6redroid%E6%98%A0%E5%83%8F%E6%AA%94%E5%AE%89%E8%A3%9Dgapps)

ReDroid作者發布的映像檔是原生Android，沒有預裝GApps。我們可以透過ayasa520的 [Remote-Android Script](https://github.com/ayasa520/redroid-script)指令稿，自動拉取ReDroid映像檔並將GApps裝上去。僅支援Android 11的映像檔。

這個指令稿也可以用來安裝libndk、libhoudini、Magisk、Widevine DRM等元件。

1.  複製Remote-Android Script儲存庫，建立Python虛擬環境

```bash
sudo apt install lzip python3 python3-venv python3-pip

git clone https://github.com/ayasa520/redroid-script.git

cd redroid-script

python3 -m venv venv

venv/bin/pip install -r requirements.txt
```

2.  拉取Android 11的映像檔，並安裝GApps

```bash
venv/bin/python3 redroid.py -a 11.0.0 -g
```

3.  這樣就會得到包含GApps的映像檔`redroid/redroid:11.0.0_gapps`了。

## 4\. 以docker-compose啟動ReDroid容器[#](#4-%E4%BB%A5docker-compose%E5%95%9F%E5%8B%95redroid%E5%AE%B9%E5%99%A8)

1.  建立跑ReDroid的虛擬機（僅Nvidia GPU需要）因Nvidia閉源驅動無法讓ReDroid使用GPU加速，需要跑一個QEMU虛擬機，再在裡面裝ReDroid透過virtio-gpu達成硬體加速。不然的話ReDroid會走軟體渲染。

建立跑ReDroid的虛擬機。

[下載Ubuntu 22.04 ISO](https://releases.ubuntu.com/jammy/)，安裝QEMU ＋ Virt Manager，建立一個64GB的Ubuuntu虛擬機

```bash
qemu-img create -f qcow2 ubuntu.qcow2 64GB

qemu-system-x86_64 -boot d -cdrom "ubuntu-22.04.1-desktop-amd64.iso" -enable-kvm -smp 4 -device intel-hda -device hda-duplex  -device virtio-vga-gl  -net nic -net user,hostfwd=tcp::5555-:5555    -cpu host  -m 4096  -display sdl,gl=on -hda ubuntu.qcow2
```

開機進虛擬機，然後再 [裝Docker](https://ivonblog.com/posts/install-docker-engine-on-linux/)。

```bash
qemu-system-x86_64 -enable-kvm -smp 4 -device intel-hda -device hda-duplex  -device virtio-vga-gl  -net nic -net user,hostfwd=tcp::5555-:5555    -cpu host  -m 4096  -display sdl,gl=on -hda ubuntu.qcow2
```

2.  建立存放Android資料的目錄，並新增docker-compose

```bash
mkdir ~/redroid

cd redroid

vim docker-compose.yml
```

3.  填入以下內容，使用剛剛建立的GApps映像檔

```yml
version: "3"
services:
  redroid:
    image: redroid/redroid:11.0.0_gapps
    stdin_open: true
    tty: true
    privileged: true
    # ADB通訊埠，為加強安全性，設定為只監聽本機localhost的通訊埠
    ports:
      - "127.0.0.1:5555:5555"
    volumes:
    # 資料存放在目前目錄下
      - ./redroid-11-data:/data
    command:
    # 手機解析度
    - androidboot.redroid_width=720
    - androidboot.redroid_height=1280
    - androidboot.redroid_dpi=320
    - androidboot.redroid_fps=60
    # 啟用宿主機的GPU硬體加速，host為啟用GPU加速，guest為軟體渲染
      - androidboot.redroid_gpu_mode=host
    # 設定libndk相關
      - ro.product.cpu.abilist0=x86_64,arm64-v8a,x86,armeabi-v7a,armeabi
      - ro.product.cpu.abilist64=x86_64,arm64-v8a
      - ro.product.cpu.abilist32=x86,armeabi-v7a,armeabi
      - ro.dalvik.vm.isa.arm=x86
      - ro.dalvik.vm.isa.arm64=x86_64
      - ro.enable.native.bridge.exec=1
      - ro.dalvik.vm.native.bridge=libndk_translation.so
      - ro.ndk_translation.version=0.2.2
```

4.  啟動服務

```bash
sudo docker compose up -d
```

5.  用ADB連線至本機的ReDroid。因為是本機，所以IP填寫localhost，如果ReDroid部署在遠端，就改成寫遠端主機的IP。

```bash
adb connect localhost:5555

# 如果連不上，用以下指令看一下容器內部發生什麼問題
sudo docker exec <容器ID> logcat
sudo docker logs <容器ID>
```

6.  執行Scrcpy，連線到Android：

```bash
scrcpy -s localhost:5555 --audio-codec=aac
```

7.  這樣就會看到Android的桌面了。
    
    ![](https://ivonblog.com/posts/redroid-android-docker/images/rPc7feZ.webp)
    
8.  Google Play服務可能會跳出「裝置未驗證」的錯誤訊息。
    
9.  執行以下指令取得Android裝置ID，到 [Google網站註冊裝置](https://www.google.com/android/uncertified)，等個30分鐘後重新啟動Redroid容器，才能登入Google Play。
    

```bash
adb -s localhost:5555 root

adb -s localhost:5555 shell 'sqlite3 /data/data/com.google.android.gsf/databases/gservices.db \
    "select * from main where name = \"android_id\";"'
```

10.  要確認ReDroid的3D加速是否有正常運作，請安裝「AIDA64」看能否認到你電腦的GPU型號。
    
11.  建議要加強安全性的，將ReDroid的docker-compose網路設定為只監聽本機localhost的通訊埠（`ports: 127.0.0.1:5000:5000`）。如果要將ReDroid開放外網存取，請注意設定防火牆，不要將ADB的5000通訊埠直接暴露到公網，否則會有嚴重安全性疑慮。
    

## 5\. ReDroid如何安裝APK[#](#5-redroid%E5%A6%82%E4%BD%95%E5%AE%89%E8%A3%9Dapk)

目前即使有安裝libndk，Android 11的Play商店還是不給下載ARM架構的APP，請配合APKPure之類的應用程式商店安裝APP。

除了用容器內部的瀏覽器下載APK外，你還可以用ADB安裝APK至ReDroid容器。比方說到 [ApkMirror](https://www.apkmirror.com/apk/line-corporation/line/)下載Line的APK，接著用ADB安裝：

```bash
adb -s localhost:5555 install "jp.naver.line.android.apk"
```

你也可以用ADB的`pull`和`push`指令傳輸檔案。

## 6\. ReDroid如何「開關機」[#](#6-redroid%E5%A6%82%E4%BD%95%E9%96%8B%E9%97%9C%E6%A9%9F)

如果要將ReDroid關機，將Scrcpy視窗關閉後，停止容器：

```bash
cd ~/redroid

sudo docker compose down
```

之後可以再用此指令啟動ReDroid。ReDroid容器的`/data`資料位於`~/redroid/redroid-11-data`目錄，可以用來備份多個系統的檔案。

```bash
cd ~/redroid

sudo docker compose up -d

adb connect localhost:5555

scrcpy -s localhost:5555 --audio-codec=aac
```

## 7\. ReDroid多開示例[#](#7-redroid%E5%A4%9A%E9%96%8B%E7%A4%BA%E4%BE%8B)

參見 [ReDroid多開與Scrcpy連線](https://ivonblog.com/posts/redroid-multiple-instances)

## 附錄[#](#%E9%99%84%E9%8C%84)

手動安裝GApps到ReDroid映像檔

ReDroid 作者說Google服務框架是專有軟體無法內建，那麼就得自行安裝了。第一個方法是重新編譯ReDroid映像檔，第二個是手動安裝OpenGApps。

不推薦第一個方法，耗時而且作者提供的GApps編譯教學又有其他APP偵測不到的問題。

這裡採用第二個方法：手動安裝。

1.  到 [OpenGapps](https://opengapps.org/)下載x86\_64架構的Android 11 GApps，選擇最小化的pico版。
    
2.  解壓縮，會看到以下目錄
    

```bash
open_gapps-x86_64-11.0-pico-20220503
├── Core
├── GApps
├── META-INF
├── Optional
```

3.  在解壓縮的目錄`open_gapps-x86_64-11.0-pico-20220503`下面新增`system`目錄。
    
4.  接著，將`Core`和`GApps`目錄裡面的`.lz`檔案都解壓縮，並將裡面的APK目錄按照對應的安裝目錄放到`system`目錄。例如`GApps/googletts-x86_64/nodpi/app/`下的`GoogleTTS`目錄要放到`/system/app`。
    
5.  放好之後，`system`下的目錄結構應該會長這樣：
    

```bash
system
├── app
│   ├── GoogleCalendarSyncAdapter
│   │   └── GoogleCalendarSyncAdapter.apk
│   ├── GoogleContactsSyncAdapter
│   │   └── GoogleContactsSyncAdapter.apk
│   ├── GoogleExtShared
│   │   └── GoogleExtShared.apk
│   └── GoogleTTS
│       └── GoogleTTS.apk
├── etc
│   ├── default-permissions
│   │   ├── default-permissions.xml
│   │   └── opengapps-permissions-q.xml
│   ├── permissions
│   │   ├── com.google.android.dialer.support.xml
│   │   ├── com.google.android.maps.xml
│   │   ├── com.google.android.media.effects.xml
│   │   ├── privapp-permissions-google.xml
│   │   └── split-permissions-google.xml
│   ├── preferred-apps
│   │   └── google.xml
│   └── sysconfig
│       ├── dialer_experience.xml
│       ├── google_build.xml
│       ├── google_exclusives_enable.xml
│       ├── google-hiddenapi-package-whitelist.xml
│       └── google.xml
├── framework
│   ├── com.google.android.dialer.support.jar
│   ├── com.google.android.maps.jar
│   └── com.google.android.media.effects.jar
├── priv-app
│   ├── AndroidAutoPrebuiltStub
│   │   └── AndroidAutoPrebuiltStub.apk
│   ├── AndroidMigratePrebuilt
│   │   └── AndroidMigratePrebuilt.apk
│   ├── CarrierSetup
│   │   └── CarrierSetup.apk
│   ├── ConfigUpdater
│   │   └── ConfigUpdater.apk
│   ├── GoogleBackupTransport
│   │   └── GoogleBackupTransport.apk
│   ├── GoogleExtServices
│   │   └── GoogleExtServices.apk
│   ├── GoogleFeedback
│   │   └── GoogleFeedback.apk
│   ├── GoogleOneTimeInitializer
│   │   └── GoogleOneTimeInitializer.apk
│   ├── GooglePackageInstaller
│   │   └── GooglePackageInstaller.apk
│   ├── GooglePartnerSetup
│   │   └── GooglePartnerSetup.apk
│   ├── GoogleRestore
│   │   └── GoogleRestore.apk
│   ├── GoogleServicesFramework
│   │   └── GoogleServicesFramework.apk
│   ├── Phonesky
│   │   └── Phonesky.apk
│   ├── PrebuiltGmsCore
│   │   └── PrebuiltGmsCore.apk
│   └── SetupWizard
│       └── SetupWizard.apk
└── product
    └── overlay
        └── PlayStoreOverlay.apk
```

6.  執行以下指令取得root權限：

```bash
adb connect localhost:5555
adb -s localhost:5555 root
adb -s localhost:5555 remount
adb -s localhost:5555 shell "rm -rf system/priv-app/PackageInstaller"
```

7.  接著將`system`目錄推送到ReDroid系統，並賦予權限：

```bash
adb -s localhost:5555 push system /
adb -s localhost:5555 shell "pm grant com.google.android.gms android.permission.ACCESS_COARSE_LOCATION"
adb -s localhost:5555 shell "pm grant com.google.android.gms android.permission.ACCESS_FINE_LOCATION"
adb -s localhost:5555 shell "pm grant com.google.android.setupwizard android.permission.READ_PHONE_STATE"
adb -s localhost:5555 shell "pm grant com.google.android.setupwizard android.permission.READ_CONTACTS"
adb reboot
```

8.  重新啟動ReDroid容器：

```bash
cd ~/redroid
sudo docker compose down
sudo docker compose up -d
```

9.  啟動Scrcpy

```bash
scrcpy -s localhost:5555 --audio-codec=raw
```

10.  開啟系統設定 → 應用程式，點選右上角顯示系統應用程式，將Google Play服務和Play商店的權限都開啟。

手動抽取libndk建置Android映像檔

libndk是Google開發的專有軟體，含在Android Studio的模擬器裡面。根據 [ReDroid作者指示](https://github.com/zhouziyang/libndk_translation)，我們可以手動從新版Android模擬器抽取libndk，再將其塞到ReDroid原生系統裡面。

這裡以Android 13為例，注意作者沒保證說一定能用。

1.  安裝Android Studio，新增Android 13的虛擬機
    
2.  透過ADB連線
    

3.  將libndk打包

```bash
{ find /system -name arm* -type d; find /system -name *ndk_translation*; find /system/etc -name *arm*; } | tar -cf /sdcard/nb.tar -T -
```

4.  將libndk.so打包

```bash
find system -type l | tar -cf /sdcard/so.tar -T -
```

5.  退出ADB，將以上檔案傳回本機

```bash
exit
exit
adb pull /sdcard/nb.tar .
adb pull /sdcard/so.tar .
```

6.  將libndk.so加入到libndk的壓縮檔

```bash
tar -xvf so.tar
find system -type l | tar -rf libndk_translation.tar -T -
```

7.  新增`DOCKERFILE`，填入以下內容：

```bash
FROM redroid/redroid:13.0.0-latest

ADD libndk_translation.tar /
```

8.  開始建置含有libndk的Android 13映像檔，之後這個映像檔`redroid:13.0.0-libndk`就能拿去執行容器了。

```bash
docker build . -t redroid:13.0.0-libndk
```

## 參考資料[#](#%E5%8F%83%E8%80%83%E8%B3%87%E6%96%99)

+   [redroid (Remote-Android) is a multi-arch, GPU enabled, Android in Cloud solution - Github](https://github.com/remote-android/redroid-doc)
+   [App 自動化測試（三）ReDroid 安裝與基本使用 - Scott Hsiao](https://vocus.cc/article/645b3257fd8978000139c12f)
+   [在使用 NVIDIA 显卡时为 redroid 开启3d加速](https://www.bilibili.com/read/cv19273886/)
+   [redroid nVidia GPU support #282](https://github.com/remote-android/redroid-doc/issues/282)
+   [Install GApps Manually - Google Groups](https://groups.google.com/g/android-rpi/c/xb2fwTQbUYw)