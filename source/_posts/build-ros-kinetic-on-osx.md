---
title: 在MacOS Sierra（10.12.3）中编译安装ROS Kinetic
date: 2017-02-14 21:26:31
tags: 
	- ROS
	- Kinetic
	- OpenCV
categories:
	- ROS
toc: true
---
[ROS][]最初是作为科研辅助工具由斯坦福大学开发的，由于ROS极大的开放性和包容性，它能够兼容其他机器人开发工具、仿真工具和操作系统、使之融为一体。这使得ROS不断发展壮大，并成为应用和影响力最广泛的机器人软件平台。

对于Ubuntu系统，安装ROS只需要按照官网提供的教程[Ubuntu install of ROS Kinetic](http://wiki.ros.org/kinetic/Installation/Ubuntu)安装即可。

对于苹果电脑，官网的安装文档[Installation Instructions for Kinetic in OS X][]提示

> *This is a work in progress! It really won't work right now...*。

按照其步骤安装有比较多的错误，查了比较多的资料终于部分安装成果。记录一下安装步骤。
<!--more-->
## 1. 编译环境
* OS: MacOS Sierra (10.12.3)
* cmake: 3.7.2
* python2: 2.7.13 (/usr/local/bin/python2)
* python3: 3.6.0 (/usr/local/bin/python3)

## 2. 编译安装步骤

### 2.1 配置环境

```bash
$ brew update
$ brew install cmake
$ brew tap ros/deps
$ brew tap osrf/simulation   # Gazebo, sdformat, and ogre
$ brew tap homebrew/versions # VTK5
$ brew tap homebrew/science  # others
$ brew install caskroom/cask/brew-cask
$ brew cask install xquartz
```

#### Python
Python最好不要使用系统自带的，这里python2、python3均是通过brew安装，位于`/usr/local/bin`目录，
```bash
$ mkdir -p ~/Library/Python/2.7/lib/python/site-packages
$ echo "$(brew --prefix)/lib/python2.7/site-packages" >> ~/Library/Python/2.7/lib/python/site-packages/homebrew.pth
$ brew install libyaml 
$ pip install -U setuptools rosdep rosinstall_generator wstool rosinstall catkin_tools bloom empy sphinx pycurl
```
#### 初始化rosdep
```bash
$ sudo rosdep init
$ sudo sh -c "echo 'yaml https://raw.githubusercontent.com/mikepurvis/ros-install-osx/master/rosdeps.yaml osx' > /etc/ros/rosdep/sources.list.d/10-ros-install-osx.list"
$ rosdep update
```
#### 初始化工作目录
```bash
$ mkdir -p kinetic_desktop_full_ws/src
$ cd kinetic_desktop_full_ws
$ rosinstall_generator desktop_full --rosdistro kinetic --deps --tar > kinetic_desktop_full.rosinstall
$ wstool init -j8 src kinetic_desktop_full.rosinstall # 下载包
```
这里`desktop_full` 包含了`ROS`, `rqt`, `rviz`, `robot-generic libraries`, `2D/3D simulators`, `navigation` 和 `2D/3D perception`等包，还可以换成`desktop`、`ros_comm`。
如果长时间下载未完成，可以中断进程，然后重新下载
```bash
$ wstool update -j 8 -t src
```
`desktop_full`也不是最完整的，还可以单独安装其他的包，更多的包可以查看[ros-gbp](https://github.com/ros-gbp)。例如，如果需要```moveit``` 包，可以如下操作：
```bash
$ rosinstall_generator moveit --rosdistro kinetic --deps --tar >> kinetic_desktop_full.rosinstall
$ wstool merge -t src kinetic_desktop_full.rosinstall
$ wstool update -t src
```
如果提示已经下载了包，则可以按`s`跳过。

#### 安装补丁
```bash
# Grabbing these older meshes allows rviz to run with Ogre 1.7 rather than Ogre 1.8+.
# Some details here: https://github.com/ros-visualization/rviz/issues/782
$ cd  src/rviz/ogre_media/models
$ curl https://raw.githubusercontent.com/ros-visualization/rviz/hydro-devel/ogre_media/models/rviz_cone.mesh > rviz_cone.mesh
$ curl https://raw.githubusercontent.com/ros-visualization/rviz/hydro-devel/ogre_media/models/rviz_cube.mesh > rviz_cube.mesh
$ curl https://raw.githubusercontent.com/ros-visualization/rviz/hydro-devel/ogre_media/models/rviz_cylinder.mesh > rviz_cylinder.mesh
$ curl https://raw.githubusercontent.com/ros-visualization/rviz/hydro-devel/ogre_media/models/rviz_sphere.mesh > rviz_sphere.mesh
$ cd -

# This patch originates from here: https://github.com/ros/catkin/pull/784
$ cd  src/catkin/cmake
$ curl https://raw.githubusercontent.com/ros/catkin/8a47f4eceb4954beb4a5b38b50793d0bbe2c96cf/cmake/catkinConfig.cmake.in > catkinConfig.cmake.in
$ cd -
```
#### 解决依赖
```bash
$ rosdep install --from-paths src --ignore-src --rosdistro kinetic -y --as-root pip:no --skip-keys=python-qt-bindings-qwt5
```

#### 创建安装目录
```bash
$ sudo mkdir -p /opt/ros/kinetic
$ sudo chown $USER /opt/ros/kinetic
```

### 2.2 编译安装

#### 配置
```bash
$ catkin config --install  --install-space /opt/ros/kinetic --cmake-args \
    -DCMAKE_FIND_FRAMEWORK=LAST \
    -DCATKIN_ENABLE_TESTING=1 \
    -DCMAKE_BUILD_TYPE=Release \
    -DPYTHON_LIBRARY=$(python -c "import sys; print sys.prefix")/lib/libpython2.7.dylib \
    -DPYTHON_INCLUDE_DIR=$(python -c "import sys; print sys.prefix")/include/python2.7  \
    -DPYTHON3_LIBRARY=$(python3 -c "import sys; print (sys.prefix)")/lib/libpython3.6.dylib \
    -DPYTHON3_INCLUDE_DIR=$(python3 -c "import sys; print (sys.prefix)")/include/python3.6m
```
其中 `-DCMAKE_FIND_FRAMEWORK=LAST`选项是为了解决下面的问题[no member named 'xxx'](https://github.com/mikepurvis/ros-install-osx/issues/73#issuecomment-264609106)添加的。
> ```bash
/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/../include/c++/v1/cstring:79:9: error: no member named
      'strcoll' in the global namespace
using ::strcoll;
      ~~^
```

`python2`和`python3`的路径根据自己`python`安装路径设置
#### 编译
```bash
$ catkin build --limit-status-rate 1
```
编译是个非常漫长的过程，中间遇到很多的问题，如果遇到问题，提示不太明显，可以`cd`到相应的`build`目录。例如`opencv3`编译出错，则可以`cd build/opencv3`，然后`make`，查看更多信息。

### 2.3 出错及解决方法

#### opencv3
`opencv3`出的问题比较多，比如`fatal error: 'QTKit/QTKit.h' file not found`，还有比较多的`Undefined symbols for architecture x86_64`错误。
`QTKit`错误因为MacOS中10.9版本开始取消了`QTKit`的支持，推荐使用`AVFoundation`。`opencv3.2.0`的最新版本已经更新过了，但是通过`rosinstall`下载的代码还没有更新，因此为了方便，直接下载最新版的`opencv`及`opencv_contrib`

不要直接删除`src`目录下的`opencv3`目录再下载，而是下载后覆盖原目录，由于`opencv_contrib`目录结构与`rosinstall`下载的有变化，因此修改`src/opencv3/CMakeList.txt`文件
```cmake
# ----------------------------------------------------------------------------
#  Path for additional modules
# ----------------------------------------------------------------------------
set(OPENCV_EXTRA_MODULES_PATH "${OpenCV_SOURCE_DIR}/opencv_contrib/modules" CACHE PATH "Where to look for additional OpenCV modules")
```
重新编译`catkin build opencv3`应该就可以了。

[ROS]: <http://www.ros.org/>
[Installation Instructions for Kinetic in OS X]: <http://wiki.ros.org/kinetic/Installation/OSX/Homebrew/Source> 
[ros-install-osx]: <https://github.com/mikepurvis/ros-install-osx>
