### 第一章 构建和安装SCons

​	本章将介绍一些安装SCons的基本步骤，本章也将介绍一些基本的python安装步骤，无论SCons还是python安装都非常简单。

#### 1.1 安装python

​	因为SCons是采用python脚本编写的，因此第一步你需要在电脑上安装python。在安装python之前，你需要检查一下python是否已经安装。打开终端输入python -V(大写)或者python --version，安装好后输出如下：

```bash
python -V
> Python 3.7.1
```

​	如果python没有安装，那么你第一步需要安装，可以通过[python官网](http://www.python.org/download)下载。

​	SCons适配的python版本为2.7.x或者3.5以后的版本。如果您需要安装python，那么我们推荐您使用最新的python版本，新版本的python会优化一些性能，这会提升SCons的编译性能。



#### 1.2 安装SCons

​	规范化的安装流程是采用python的安装所以包（PyPi）：

```bash
python -m pip install scons
```

​	如果您不想安装到python的系统路径，或者没有这样的权限，那么您可以增加一个标志以安装到您自己的账户特定位置：

```bash
python -m pip insyall --user scons
```

​	对于很多的linux系统上，scons已经预先打包安装好了，您可以预先查看一下scons包是否可用。有很多系统会有两个scons版本，分别使用的是python2和pytho3.如果您需要的SCons特定版本与可用的软件包不同，请pip使用版本选择，或者安装下一节的说明进行操作。



#### 1.3 在任何系统上侯建和安装SCons

​	如果您的系统上没有预先安装SCons，同时pip工具包也不可用，那么您可以通过安装python原生包，很容易地安装并使用SCons。

​	首先您需要到[SCons官网]( http://www.scons.org/download.html)下载scons-3.1.1.tar.gz或者scons-3.1.1.zip，分别对应linux系统与windows系统。解压他们到合适的位置，然后调用下面命令：

```bash
> cd scons-3.1.1
> python setup.py install #sudo if need
```

​	安装SCons位于/usr/local/bin或 C:\Python27\Scripts，同时安装使用SCons在Python的构建依赖库/usr/local/lib/scons或C:\Python27\scons。由于这些是系统目录，因此您可能需要root用户（在Linux或UNIX上）或Administrator（在Windows上）特权才能安装这样的SCons。

##### 1.3.1 安装多个版本SCons

​	setup.py安装脚本有一些扩展功能，以便于简化同时安装多版本的需求，这使得同时下载安装多个SCons版本变得非常简单，而且需要更新SCons版本时，也不用删除当前安装的版本。

​	如果安装特定SCons版本，需要在命令后增加--version-lib选项：

```bash
python setup.py install --version-lib
```

​	这将会俺咋混个SCons依赖在/usr/lib/scons-3.1.1或C:\python2.7\scons-3.1.1目录。

​	如果您第一次使用--version-lib选项，那么您无需每次都特别指明版本。setup.py会检测特殊版本路径名称，并安装您需要的版本。您也可以通过采用--standalone-lib来覆盖这一特性。

##### 1.3.2 在其他位置安装SCons

​	您可以通过指定安装位置选项 --prefix=**来指定安装位置：

```bash
python setup.py install --prefix=/opt/scons
```

​	这样SCons将会安装在/opt/scons/bin，依赖库安装在/opt/scons/lib/scons下面。

​	请注意，您可以同时指定--prefix和--version-lib选项，在这种情况下，setup.py会将构建依赖库安装在指定前缀的特定版本中。如果增加了--version-lib，则上述命令将会把依赖安装到/opt/scons/lib/scons-3.1.1中。

##### 1.3.3 非管理员构建并安装SCons

​	如果您没有权限去安装SCons在系统路径，可以通过--prefix安装到指定路径。如果您计划将SCons安装到$HOME下面，则执行下面命令：

```bash
python setup.py install --prefix=$HOME
```

则scons将会被安装到$HOME/bin中，其依赖库会被安装到$HOME/lib/scons下面。

​	当然您也可以通过指定--version-lib来确定特殊版本的安装，具体描述见上一节。