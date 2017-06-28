# Ubuntu下OpenCV环境搭建

标签（空格分隔）： 环境配置 工业视觉系统

---

一共有两种方式可以搭建OpenCV环境，支持Python。第一种是通过apt-get直接安装，这种方式比较方便，但版本比较旧。第二种就是通过源码编译安装。下面是详细过程。

##apt-get安装
打开终端，之后输入下面的命令。
```bash
sudo apt-get install libopencv-dev python-opencv
```
这种方式很简单，但最新版本是2.4.9.1，比较旧。

**卸载方式**
```bash
sudo apt-get remove libopencv-dev python-opencv
```

##通过源码编译安装

- 安装必要的包
```bash
sudo apt-get install build-essential
sudo apt-get install cmake git libgtk2.0-dev pkg-config libavcodec-dev libavformat-dev libswscale-dev
sudo apt-get install python-dev python-numpy libtbb2 libtbb-dev libjpeg-dev libpng-dev libtiff-dev libjasper-dev libdc1394-22-dev
```

- 安装cmake-qt-gui配置opencv安装配置
```
sudo apt-get install cmake-qt-gui
```

- 首先从github上同步代码下来，由于OpenCV从3.0版本开始分为两个部分，所以如果需要支持所有的功能，需要两个库都下载。 
```bash
cd ~/opencv_src    #这里切换到你的工作目录，即存放源码的目录
git clone https://github.com/opencv/opencv.git
git clone https://github.com/opencv/opencv_contrib.git
```

- 然后切换到opencv，新建一个build目录，切换到build后cmake。这里是[CMake编译opencv各选项的含义](http://blog.csdn.net/j_d_c/article/details/53365381)的链接。
```bash
cd opencv
git checkout 3.2.0 #这里使用git checkout切换到你想要编译的版本
#opencv_contrib的版本必须与opencv相同，所以要执行下面两个命令
cd ../opencv_contrib
git checkout 3.2.0
#切换回opencv目录
cd ../opencv
mkdir build
cd build
sudo cmake -D CMAKE_BUILD_TYPE=RELEASE -D CMAKE_INSTALL_PREFIX=/usr/local -D OPENCV_EXTRA_MODULES_PATH=~/opencv_src/opencv_contrib/modules ..
```

## 用图形化工具配置生成 makefile

- 切换至 opencv 目录，然后新建一个目录，用于存放 cmake 生成的配置文件。然后切换至 build 目录。
mkdir build
cd build
- 再打开一个终端，到 cmake 目录里的 bin 子目录里找到 cmake-gui，然后运行一下命令：
./cmake-gui
打开cmake图形化工具。
然后在 Where is the source code 中添加 opencv 目录的路径，在 where to build the binaries 中添加 build 目录的路径。

- 然后点击下面的 Configure，选择 Unix Makefiles，然后选择 default ...。点击 OK，从而处理 opencv 目录下的 CMakeLists.txt，并在 build 目录下生成 CMakeCache.txt 及 Makefile 等相关文件。
然后开始进行配置。
```
CMAKE_BUILD_TYPE 设为 RELEASE
CMAKE_INSTALL_PREFIX 设为 所要安装 OpenCV3 的目录（这里指定为 /usr/local）
OPENCV_EXTRA_MODULES_PATH 设为 opencv_contrib 中的 modules 目录的路径（比如，/home/cuckootan/Downloads/opencv_contrib/modules）
在 BUILD_EXAMPLES 的复选框里 打勾
在 INSTALL_C_EXAMPLES 的复选框里 打勾
在 INSTALL_PYTHON_EXAMPLES 的复选框里 打勾
在 BUILD_opencv_python3 的复选框里 打勾
PYTHON3_EXECUTABLE 设为 /usr/bin/python3
PYTHON3_INCLUDE_DIR 设为 /usr/include/python3.5m
PYTHON3_LIBRARY 设为 /usr/lib/x86_64-linux-gnu/libpython3.5m.so
PYTHON3_PACKAGES_PATH 设为 /usr/local/lib/python3.5/dist-packages
PYTHON3_NUMPY_INCLUDE_DIRS 设为 /usr/local/lib/python3.5/dist-packages/numpy/core/include
在 WITH_FFMPEG 的复选框里 打勾
在 WITH_GTK 的复选框里 打勾
在 WITH_V4L 的复选框里 打勾
   在 WITH_TIFF 的复选框里 打勾
   在 WITH_PNG 的复选框里 打勾
   在 WITH_JPEG 的复选框里 打勾
   在 WITH_JASPER 的复选框里 打勾
   取消 WITH_CUDA 的复选框里的勾
   取消 WITH_CUFFT 的复选框里的勾
```

另外，由于安装的是 GTK3，但由于在配置表中不存在，因此要手动添加。单击 ADD Entry，输入 WITH_GTK3，选择 bool，并对其下面的复选框打上钩。

上面的路径信息需要按照自己的情况填写，比如 python3 的安装路径等。
如果是想安装 OpenCV for Python2，则除了配置 Python2 的相关信息外，还要将上面所要添加的条目中的 python3 关键字改为 python2 即可。

然后点击 Configure，应用刚才的修改并更新至 build 目录下的 CMakeCache.txt 中。然后点击 Generate，更新 Makefile。

这个过程由于墙的原因导致下载一些包失败，所以需要把你之前下载好的包替换不能下载失败的包。
```
ippicv_linux_20151201.tgz 文件复制并替换 opencv-3.2.0/3rdparty/ippicv/downloads/linux-808b791a6eac9ed78d32a7666804320e/ 路径下的同名文件；
protobuf-cpp-3.1.0.tar.gz 复制并替换 opencv_contrib-3.2.0/modules/dnn/.download/bd5e3eed635a8d32e2b99658633815ef/v3.1.0/ 路径下的同名文件。
```

- 接着开始编译安装
```bash
sudo make -j4 #-j4代表使用4个线程编译
sudo make install
```

- 使用命令查看安装版本
```bash
pkg-config --modversion opencv
```

**卸载方式**
```bash
cd opencv/build #从github同步下来的opencv库
sudo make uninstall
cd ..
sudo rm -r build
sudo rm -r /usr/local/include/opencv2 /usr/local/include/opencv /usr/include/opencv /usr/include/opencv2 /usr/local/share/opencv /usr/local/share/OpenCV /usr/share/opencv /usr/share/OpenCV /usr/local/bin/opencv* /usr/local/lib/libopencv*
sudo apt-get –purge remove opencv-doc opencv-data python-opencv
```

##参考
- [ubuntu卸载opencv并重装opencv3.0.0](http://milq.github.io/install-opencv-ubuntu-debian/)
- [Install OpenCV on Ubuntu or Debian](https://www.cnblogs.com/txg198955/p/5990295.html)
- [Ubuntu 下安装 OpenCV3](http://cuckootan.me/2016/10/01/Linux/Ubuntu%20%E4%B8%8B%E5%AE%89%E8%A3%85%20OpenCV3/)