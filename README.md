# SGBM_FPGA
## Before Start
ALL CODE HAS BEEN UPDATED TO MAIN BRANCH!
## Introduction
This repository contains code and pdf tutorial of how I've implemented binocular camera matching algorithm, SGBM, with FPGA using verilog. Code in this repo contains both C++ SGBM simulation code and PS-PL vitis-vivado project. Tutorial and the video attached thoroughly explain my designation in detail, although made in Chinese hhhhh.
## Code Explanation
Under the "code" directory, there are 2 directories called "FPGA" and "Ubuntu_C++" respectively. Under "FPGA", 5 directories can be found.
-census_vcode
This directory contains all verilog code that has been used in realizing census transforming. Under this directory, there are 2 directories, each called source and sim. Under source directory, finally you'll find census.v, and this file is the main verilog file which realizes census transforming. Under sim directory, finally you'll find tb_census.v and tb_census_zty.v, and these files are used as testbench for census.v.
- origin_cost_vcode
This directory contains all verilog code that has been used in realizing origin cost calculation. Under this directory, there are 2 directories, each called source and sim. Under source directory, finally you'll find origin_cost.v, and this file is the main verilog file which realizes origin cost calculation. Under sim directory, finally you'll find several verilog files as testbench, and among them, tb_origin_cost.v should be set as top file when you do behaverial simulations.
- aggregate_cost_vcode
This directory contains all verilog code that has been used in realizing aggregate cost calculation. Under this directory, there are 2 directories, each called source and sim. Under source directory, finally you'll find aggregate.v, and this file should be set as top file when you set up your own vivado aggregate cost project! This aggregate.v calls several other auxiliary modules to implement cost aggregation! Under sim directory, finally you'll find several verilog files as testbench, and among them, tb_aggr_cost.v should be set as top file when you do behaverial simulations.
- project_sgbm_vcode
This directory contains all verilog code that has been used in the overall PL part of SGBM project. Under this directory, there are 2 directories, each called source and sim. Under source directory, finally you'll find bd directory, bd directory contains all block design files as top files! I've wrapped other verilog files as RTL IP kernel so that I'm able to use them in block design.
- project_image
This directory contains both code of PS part and code of PL part. Under this directory, there are 2 directories, project_image_c for vitis C code and project_image_vcode for verilog code! Under ./vitis_image/vitis_image/src/, I've mainly written main.c, main.h, udp_perf_client.c and udp_perf_client.h. If you'd like to run these vitis code yourself, firstly you're supposed to set up the PL part in vivado with code under project_image_vcode, simulation, implementation and generate bitstream finally. Secondly, you're supposed to export hardware with bitstream to get .xsa file. Thirdly, create your own vitis project using .xsa which has just been generated in the second step. I have to mention that please create your vitis project using the template named "LWIP UDP Client" that has been offered by vitis software. Finally, change the code in main.c, main.h, udp_perf_client.c and udp_perf_client.h into my code, and there you go!
## Tutorial and Video
You're greatly welcomed to follow my bilibili account, which is https://space.bilibili.com/341561358. Hopefully you can find SGBM video explanation there at https://www.bilibili.com/video/BV1kR4y1S7TJ !
## Start
### initial project
```shell
cd STEREO_MATCH_SGBM/code/Ubuntu_C++/awesome-sgbm
rm -rf build && mkdir build
```
### install opencv
```shell
git clone git@github.com:opencv/opencv.git
cd opencv
git checkout 4.11.0
mkdir build && cd build
sudo apt-get install build-essential libgtk2.0-dev libavcodec-dev \ 
                     libavformat-dev libjpeg.dev libtiff5.dev libswscale-dev \
                     libgtk2.0-dev libjasper-dev \
                     -y

cmake -D CMAKE_BUILD_TYPE=Release -D CMAKE_INSTALL_PREFIX=/usr/local/include/opencv-4.11.0 ..
make -j8
sudo make install
sudo vi /etc/ld.so.conf
    add
        /usr/local/include/opencv-4.11.0
sudo ldconfig
sudo vi ~/.bashrc
    add
        PKG_CONFIG_PATH=$PKG_CONFIG_PATH:/usr/local/include/opencv-4.11.0/lib/pkgconfig
        export PKG_CONFIG_PATH
        export OpenCV_DIR=/usr/local/include/opencv-4.11.0
source ~/.bashrc
pkg-config opencv4 --modversion
cd ../samples/cpp/example_cmake
cmake . && make && ./opencv_example
```
### install usbipd(windows camera connect 2 wsl2)
- follow this can be successfully https://blog.csdn.net/chengpengfei09121870/article/details/142762468
download the usbipd-win 2 control usb dev
- https://github.com/dorssel/usbipd-win/releases/tag/v4.3.0
```powershell
# check usb device
usbipd list
# bind and connect usb dev 2 wsl2
usbipd bind -b [BUSID]
usbipd attach -w -b [BUSID]
# detach usb dev
usbipd detach -b [BUSID]
```
```shell
sudo apt install usbutils -y
lsusb
# install wsl camera driver
sudo apt install -y build-essential flex bison dwarves libssl-dev libelf-dev libncurses-dev pkg-config
uname -r
cd ~/downloads
TAGVERNUM=5.15.167.4
TAGVER=linux-msft-wsl-${TAGVERNUM}
- download driver
    - https://codeload.github.com/microsoft
        - https://codeload.github.com/microsoft/WSL2-Linux-Kernel/tar.gz/refs/tags/linux-msft-wsl-5.15.167.4
sudo tar -xzvf WSL2-Linux-Kernel-linux-msft-wsl-5.15.167.4.tar.gz
sudo mv WSL2-Linux-Kernel-linux-msft-wsl-5.15.167.4 /usr/src/${TAGVERNUM}-microsoft-standard
cd /usr/src/${TAGVERNUM}-microsoft-standard

sudo cp /proc/config.gz config.gz \
  && sudo gunzip config.gz \
  && sudo mv config .config

sudo make menuconfig

# Build WSL2 kernel with usb camera support
# menuconfig -> Device Drivers -> Multimedia support -> Filter media drivers
#            -> Device Drivers -> Multimedia support -> Media device types -> Cameras and video grabbers
#            -> Device Drivers -> Multimedia support -> Video4Linux options -> V4L2 sub-device userspace API
#            -> Device Drivers -> Multimedia support -> Media drivers -> Media USB Adapters -> USB Video Class (UVC)
#            -> Device Drivers -> Multimedia support -> Media drivers -> Media USB Adapters -> UVC input events device support
#            -> Device Drivers -> Multimedia support -> Media drivers -> Media USB Adapters -> GSPCA based webcams

sudo make -j$(nproc) KCONFIG_CONFIG=.config \
  && sudo make modules_install -j$(nproc) \
  && sudo make install -j$(nproc)

mkdir -p /mnt/c/Users/47338/WSL2/kernel-5.15.167.4
sudo cp vmlinux /mnt/c/Users/47338/WSL2/kernel-5.15.167.4
vi /mnt/c/Users/47338/.wslconfig

    [wsl2]
    kernel=C:\\Users\\<user name>\\WSL2\\kernel-5.15.150.1\\vmlinux
# reboot wsl
# test usb camera by opencv example cpp program
cd opencv/samples/cpp/example_cmake/build
sudo ./opencv_example
```
### install PCL
```
cd ~/downloads
wget https://codeload.github.com/PointCloudLibrary/pcl/tar.gz/refs/tags/pcl-1.14.1
tar -zxvf pcl-1.14.1
cd pcl-pcl-1.14.1
sudo apt-get update
sudo apt-get install git build-essential linux-libc-dev \
                     cmake \
                     libusb-1.0-0-dev libusb-dev libudev-dev \
                     mpi-default-dev openmpi-bin openmpi-common \
                     libpcap-dev \
                     libflann1.9 libflann-dev libeigen3-dev \
                     libboost-all-dev \
                     vtk9 libvtk9-dev libvtk9-qt-dev \
                     libqhull* libgtest-dev \
                     freeglut3-dev pkg-config \
                     libxmu-dev libxi-dev \
                     mono-complete \
                     libopenni-dev libopenni2-dev
mkdir build && cd build
cmake -DCMAKE_BUILD_TYPE=None \
      -DCMAKE_INSTALL_PREFIX=/usr/local \
      -DBUILD_GPU=ON \
      -DBUILD_apps=ON \
      -DBUILD_examples=ON ..
make -j$(nproc)
sudo make install
pcl_viewer ../test/pcl_logo.pcd
```
### build project
```
cd STEREO_MATCH_SGBM/code/Ubuntu_C++/awesome-sgbm
mkdir build && cd build
cmake .. && make -j
../bin/semi_global_matching ../Data/cone/left.png ../Data/cone/right.png 0 128 5
```
## TODO
A new video will be made in a couple of days illustrating the usage of this repository in detail!
