# How to install Yuntu AI device compiler env

## Note

>1. All build or compiler oper, default directory are software or application libraries current directory.
>2. Update Ubuntu 18.04 's APT and PIP3 source to Aliyun mirror first.
>3. Ubuntu 18.04 login account is root.

## Env SDK or API version

>1. FFMPEG 4.2.2
>2. OPENCV 4.2.0-openvino
>3. LIBVA 2.11.0
>4. GmmLib 19.4.1
>5. Intel Media SDK 19.4.0 msdk
>6. media-driver 21.1.2
>7. libdrm 2.4.80
>8. openvino 2021.2.185

## 环境准备

### 升级 Linux 内核到 5.7.0+

~~~shell
# 下载 Linux 5.7.0 内核升级安装文件

cd /home/yt/software

wget  https://kernel.ubuntu.com/~kernel-ppa/mainline/v5.7-rc5/linux-headers-5.7.0-050700rc5_5.7.0-050700rc5.202005101931_all.deb --no-check-certificate

wget https://kernel.ubuntu.com/~kernel-ppa/mainline/v5.7-rc5/linux-headers-5.7.0-050700rc5-generic_5.7.0-050700rc5.202005101931_amd64.deb --no-check-certificate

wget https://kernel.ubuntu.com/~kernel-ppa/mainline/v5.7-rc5/linux-image-unsigned-5.7.0-050700rc5-generic_5.7.0-050700rc5.202005101931_amd64.deb --no-check-certificate

wget https://kernel.ubuntu.com/~kernel-ppa/mainline/v5.7-rc5/linux-modules-5.7.0-050700rc5-generic_5.7.0-050700rc5.202005101931_amd64.deb --no-check-certificate

dpkg -i ./*.deb
reboot # 需要重新启动

~~~

### OpenVINO 2021 安装

此处安装 OpenVINO 包括 所有依赖项 以及 INTEL_NEO_OCL

### apt 安装

> 注意下面需要安装的软件，如果需要自制 Linux 系统，则必须完全手动编译一套。
~~~bash

apt -y install dh-autoreconf libssl-dev uuid-dev curl libjsoncpp-dev libcurl4-openssl-dev libssh-dev

apt -y install libncurses5-dev libpthread-stubs0-dev libpciaccess-dev libxvmc-dev xutils-dev libsdl2-dev

apt -y install libass-dev libmp3lame-dev libvpx-dev libx264-dev libx265-dev libtheora-dev libxcb-shm0-dev

apt -y install git make dh-autoreconf pkg-config build-essential libgtk2.0-dev libblas-dev libopenblas-dev

apt -y install autoconf automake libtool bzip2 nasm yasm python3-pip libgstreamer1.0-dev libarchive-dev

apt -y install gstreamer1.0-plugins-{base,good,bad} libgstreamer-plugins-{base,good,bad}1.0-dev gstreamer1.0-libav

apt -y install gstreamer1.0-vaapi liblua5.2-dev lua5.2 liba52-dev libxcb-composite0-dev libxcb-xv0-dev libprotoc-dev libssh2-1-dev

apt -y install flex bison

pip3 install numpy==1.12.0
~~~

### cmake版本要求

~~~bash
# 获取最新版本
git clone https://github.com/Kitware/CMake.git
cd CMake

# 卸载现有的cmake
apt remove --purge --auto-remove cmake
# 从cmake官网下载cmake 3.13版本
# 安装
./bootstrap
make -j4
make install

ln -s /usr/local/bin/cmake /usr/bin/cmake
# 检查
cmake --version
~~~


## Build and install libdrm 2.4.80
~~~bash
cd /home/yt/software
git clone https://github.com/intel/iotg-lin-gfx-libdrm
# 安装到系统目录替换掉原来的
cd /home/yt/software/iotg-lin-gfx-libdrm
./autogen.sh
./configure --prefix=/usr --exec-prefix=/
make -j4
make install
~~~


## Build and install libva 2.11.0

~~~bash
#先删除系统内以前的libva
rm -rf /usr/include/va
rm -rf /usr/lib/x86_64-linux-gnu/libva*	
# 下载最新的 libva
cd /home/yt/software
git clone https://github.com/intel/libva.git
cd libva
./autogen.sh --prefix=/opt/intel/libva --libdir=/opt/intel/libva/lib
make -j4
make install

#环境变量配置
echo "export LD_LIBRARY_PATH=/opt/intel/libva/lib:\$LD_LIBRARY_PATH" >> /etc/bash.bashrc
echo "export PKG_CONFIG_PATH=/opt/intel/libva/lib/pkgconfig:\$PKG_CONFIG_PATH" >> /etc/bash.bashrc
ln -s /opt/intel/libva/lib/libva.so /usr/lib/libva.so
ln -s /opt/intel/libva/lib/libva-drm.so /usr/lib/libva-drm.so
ln -s /opt/intel/libva/include/va /usr/include/va
# 生效环境变量配置
source /etc/bash.bashrc
updatedb
ldconfig
~~~

## Build and install libva-utils 2.6.0

```bash
cd /home/yt/software
git clone https://github.com/intel/libva-utils.git
cd libva-utils
./autogen.sh --prefix=/opt/intel/libva-utils --libdir=/opt/intel/libva-utils/lib
make -j4
make install
ln -s /opt/intel/libva-utils/bin/vainfo /usr/bin/vainfo
```

## Build and install gmmlib
~~~bash
# 下载最新的 gmmlib 
cd /home/yt/software
git clone https://github.com/intel/gmmlib.git
cd gmmlib
mkdir build 
cd build
cmake -DCMAKE_BUILD_TYPE=Release -DARCH=64 ..
make -j4
make install
# gmmlib 被安装到默认目录即 /usr/local 下面
~~~

## Build and instal Intel Media Driver 
> 注意: 目录结构，安装 Media-Driver 前，必须安装 gmmlib 和 libva
~~~
<workspace>
  |-media-driver
  |-build_media
~~~

~~~bash
# 下载最新版本的 Intel Media Driver 21.1.1
cd /home/yt/software
git clone https://github.com/intel/media-driver.git
mkdir build_media
cd build_media
cmake ../media-driver \
-DCMAKE_INSTALL_PREFIX=/opt/intel/media-driver \
-DCMAKE_INSTALL_LIBDIR=/opt/intel/media-driver/lib \
-DLIBVA_DRIVERS_PATH=/opt/intel/media-driver/lib/dri
make -j4
make install
# 设置驱动程序类型
echo "export LIBVA_DRIVER_NAME=iHD" >> /etc/bash.bashrc
# 设置驱动文件目录
echo "export LIBVA_DRIVERS_PATH=/opt/intel/media-driver/lib/dri" >> /etc/bash.bashrc
# 设置驱动库文件目录
echo "export LD_LIBRARY_PATH=/opt/intel/media-driver/lib:\$LD_LIBRARY_PATH" >> /etc/bash.bashrc
echo "export LD_LIBRARY_PATH=/opt/intel/media-driver/lib/dri:\$LD_LIBRARY_PATH" >> /etc/bash.bashrc
# 设置驱动库 pkg-config 目录
echo "export PKG_CONFIG_PATH=/opt/intel/media-driver/lib/pkgconfig:\$PKG_CONFIG_PATH" >> /etc/bash.bashrc

# 生效配置
source /etc/bash.bashrc
updatedb
ldconfig
# 检查 Intel Media Drvier 安装状态
vainfo
~~~

## Build and install Intel Media SDK (msdk)

~~~bash
# 下载最新的 msdk
cd /home/yt/software
git clone https://github.com/Intel-Media-SDK/MediaSDK msdk
cd msdk
mkdir build
cd build
cmake -DBUILD_SAMPLES=OFF ..
make
make install
echo "export LD_LIBRARY_PATH=/opt/intel/mediasdk/lib:/opt/intel/mediasdk/lib/mfx:\$LD_LIBRARY_PATH" >> /etc/bash.bashrc
echo "export PKG_CONFIG_PATH=/opt/intel/mediasdk/lib/pkgconfig:\$PKG_CONFIG_PATH" >> /etc/bash.bashrc
echo "export MFX_HOME=/opt/intel/mediasdk" >> /etc/bash.bashrc

source /etc/bash.bashrc
updatedb
ldconfig
# 由于前边以及做过 Intel 显卡驱动并且版本比msdk中的版本高，因此需要将其替换掉。
rm -rf /opt/intel/mediasdk/lib64/iHD_drv_video.so
ln -s /opt/intel/media-driver/lib/dri/iHD_drv_video.so /opt/intel/mediasdk/lib64/iHD_drv_video.so
~~~




## Build and install FFmpeg

~~~bash
#从https://github.com/FFmpeg/FFmpeg/releases/tag/n4.2.2 下载ffmpeg

cd /home/yt/software
git clone https://github.com/FFmpeg/FFmpeg.git
cd FFmpeg
./configure --prefix=/usr/local/ffmpeg \
--libdir=/usr/local/ffmpeg/lib \
--enable-encoder=h264_qsv \
--enable-decoder=h264_qsv \
--enable-gpl \
--enable-version3 \
--enable-nonfree \
--disable-static \
--enable-shared \
--disable-debug \
--enable-libass \
--enable-libfreetype \
--enable-libmp3lame \
--enable-libtheora \
--enable-libvpx \
--enable-libx264 \
--enable-libx265 \
--enable-vaapi \
--disable-x86asm \
--enable-decoder=hevc \
--enable-libmfx

make -j8
make install

#环境变量配置
echo "export LD_LIBRARY_PATH=/usr/local/ffmpeg/lib:\$LD_LIBRARY_PATH" >> /etc/bash.bashrc
echo "export PKG_CONFIG_PATH=/usr/local/ffmpeg/lib/pkgconfig:\$PKG_CONFIG_PATH" >> /etc/bash.bashrc
rm -rf /usr/bin/ffmpeg
rm -rf /usr/bin/ffplay
rm -rf /usr/bin/ffprobe

ln -s /usr/local/ffmpeg/bin/ffmpeg /usr/bin/ffmpeg
ln -s /usr/local/ffmpeg/bin/ffplay /usr/bin/ffplay
ln -s /usr/local/ffmpeg/bin/ffprobe /usr/bin/ffprobe

source /etc/bash.bashrc
updatedb
ldconfig
~~~


## Build and install gstreamer-media-SDK
~~~bash
git clone https://github.com/intel/gstreamer-media-SDK.git
cd gstreamer-media-SDK

~~~

## BUild and install OPENCV 4.2.0-openvino
~~~bash
# git clone https://github.com/opencv/opencv.git
# 这里不能使用 git 最新的，建议下载 4.2.0-openvino
mkdir build
cd build
cmake -DCMAKE_BUILD_TYPE=Release -DWITH_MFX=ON -DWITH_FFMPEG=ON -DWITH_GSTREAMER=ON DWITH_VA_INTEL=ON ..
make -j4
make install
echo "prefix=/usr/local" > /usr/local/lib/pkgconfig/opencv.pc
echo "exec_prefix=\${prefix}" >> /usr/local/lib/pkgconfig/opencv.pc
echo "libdir=\${exec_prefix}/lib" >> /usr/local/lib/pkgconfig/opencv.pc
echo "includedir_old=\${prefix}/include/opencv4" >> /usr/local/lib/pkgconfig/opencv.pc
echo "includedir_new=\${prefix}/include" >> /usr/local/lib/pkgconfig/opencv.pc
echo "Name: OpenCV" >> /usr/local/lib/pkgconfig/opencv.pc
echo "Description: Open Source Computer Vision Library" >> /usr/local/lib/pkgconfig/opencv.pc
echo "Version: 4.2.0-openvino" >> /usr/local/lib/pkgconfig/opencv.pc
echo "Libs: -L\${exec_prefix}/lib -lopencv_objdetect -lopencv_dnn -lopencv_stitching -lopencv_photo -lopencv_ml -lopencv_calib3d -lopencv_features2d -lopencv_highgui -lopencv_flann -lopencv_videoio -lopencv_imgcodecs -lopencv_video -lopencv_imgproc -lopencv_core" >> /usr/local/lib/pkgconfig/opencv.pc
echo "Libs.private: -ldl -lm -lpthread -lrt" >> /usr/local/lib/pkgconfig/opencv.pc
echo "Cflags: -I\${includedir_old} -I\${includedir_new}" >> /usr/local/lib/pkgconfig/opencv.pc

~~~

## Build and install vlc-3.0.9.2
~~~bash
wget https://mirror-hk.koddos.net/videolan/vlc/3.0.9.2/vlc-3.0.9.2.tar.xz
tar xf vlc-3.0.9.2.tar.xz
cd vlc-3.0.9.2.tar.xz
./configure NTL_STD_CXX14=on --disable-chromecast --enable-mfx --disable-opencv --enable-realrtsp --enable-live555 --enable-vaapi
ln -s /usr/local/lib/vlc /usr/lib/vlc
ln -s /usr/local/lib/libvlccore.so.9 /usr/lib/libvlccore.so.9
ldconfig
updatedb

cd contrib
mkdir native
cd native
../bootstrap
make live555
cd live555
./genMakefile linux
make
make install
ldconfig
updatedb

~~~
