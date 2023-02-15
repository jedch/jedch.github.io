---
layout: post
title: CentOS7安装与配置之二
category: python
---
​        严格意义上讲，这不是一个系列，而是不同时期的系统安装配置方法。如果说非要说有联系，就是自动化程度在提高。嗯，这就是懒人愿意干活的唯一的动力。下面主要针对几个问题，用命令行解决问题。

系统环境：CentOS7-1810 x86_64

1. 虚拟机及配置（实体机安装可以跳过）
在设置中添加共享文件夹
在centos7系统下运行 `vmware-hgfsclient`
创建 `sudo mkdir /mnt/hgfs`
手动挂载 `sudo vmhgfs-fuse .host:/forall /mnt/hgfs`
创建自动挂载`sudo vi /etc/fstab`，按`i`，添加一行`.host:/forall /mnt/hgfs fuse.vmhgfs-fuse allow_other, defaults 0 0`
按`ESC`，按住`shift`按两次`z`
重启系统，OKin，records out，就完成了。

2. 配置开发环境
* yum锁定解决
`sudo rm -f /var/run/yum.pid`
* 配置清华centos7, epel源

```
sudo yum makecache
sudo yum install yumex -y
sudo yum install dnf -y
sudo yum install dnf-plugins-core -y
```

* 设置左边栏
sudo dnf install gnome-shell-extension-dash-to-dock -y
sudo dnf install gnome-shell-extension-no-topleft-hot-corner -y
重启

* 支持ntfs
sudo dnf install ntfs-3g -y
sudo dnf install exfatprogs -y
{sudo rpm -Uvh http://li.nux.ro/download/nux/dextop/el7/x86_64/nux-dextop-release-0-5.el7.nux.noarch.rpm
sudo yum makecache
sudo yum install fuse-exfat exfat-utils -y}

* 安装chrome
wget https://dl.google.com/linux/direct/google-chrome-stable_current_x86_64.rpm
sudo yum localinstall google-chrome-stable_current_x86_64.rpm

* github访问慢
编辑/etc/hosts,末尾添加两行
140.82.113.3 github.com
199.232.69.194 github.global.ssl.fastly.net
保存退出，运行service network restart

* devoo中有gcc10文件，可以安装，安装好的路径是/opt/gcc-10.2.1/usr/lib64，在终端进入上面路径，运行sudo rm /usr/lib64/libstdc++.so.6, 建立连接sudo ln -s libstdc++.so.6.0.28 /usr/lib64/libstdc++.so.6，重启搞定！

* 复制安装文件到Downloads目录(sublime,root_v6.22.02.source.tar.gz,geant4.10.07.p01.tar.gz,clhep-2.4.4.1.tgz,Miniconda3-py38_4.9.2-Linux-x86_64.sh,new)

可以在断网的情况下进入devcxj，运行
sudo yum localinstall *.rpm
重启

(devcxj文件夹已经离线保存了许多必要的rpm文件，最初可以用下面的命令更新维护
sudo dnf download --arch x86_64 --resolve git cmake3 cmake3-gui gcc-c++ gcc binutils libX11-devel libXpm-devel libXft-devel libXext-devel gcc-gfortran openssl-devel pcre-devel mesa-libGL-devel mesa-libGLU-devel glew-devel ftgl-devel mysql-devel fftw-devel cfitsio-devel graphviz-devel avahi-compat-libdns_sd-devel python-devel libxml2-devel xerces-c-devel libXmu-devel hdf5-devel gsl-devel openldap-devel libXt-devel libXmu-devel xerces* expat-devel libXi-devel freeglut pythia8-devel libXtst-devel libXv-devel libxkbfile-devel libXcomposite-devel libXcursor-devel libXdmcp-devel libXinerama-devel libXrandr-devel libXres-devel openblas-devel blas-devel gl2ps libAfterImage-devel libtiff-devel golang motif-devel redhat-lsb-core clang-devel elfutils-libelf-devel cppunit-devel sip qt5-qtbase-devel qt5-qttools-devel qt5-qt3d-devel boost-devel boost-python36-devel Coin3-devel gnuplot
）

source ~/.bashrc

3. 安装pyne

`mkdir -p $HOME/program/{clhep,geant4,root}/{build,install}`

* 注意：使用conda之前要配置好系统开发环境，不然conda会安装系统未配置的环境，很麻烦!
能用系统的环境就一定不要用conda的。例如，系统没有gcc，conda会给装上，后面的编译大概率会链接错误。
安装之前配置好输入法，否则麻烦。
下载Miniconda3-py38_4.8.3，安装后
`conda config --set changeps1 false`
`conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/`
`conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge`
删除default后，`conda clean -i`
安装nscl daq时发现，必须要pyqt4，于是创建python3.6环境并安装pyqt4。
`conda create --name py36 python=3.6`
`conda activate py36`
在.bashrc末尾添加`conda activate py36`
`source ~/.bashrc`
查看状态`conda info -e`

pyqt4在conda forge可以找到下载，要求python 3.6*, qt >=4.8.6,<5.0, sip 4.18,在Downloads目录本地安装
`conda install --use-local pyqt-4.11.4-py36_3.tar.bz2`
`conda install --file=requirement_conda.txt`

4. 安装ROOT6

build, install, src三文件夹编译安装
Configure过程中，搜索notfound，补充安装lib等。
手动指定几个路径chrome-->opt/google/chrome,pythia8_data-->usr/lib64,tbb_root_dir-->usr/lib64,glew-->找不到。
选中几个复选框buildin_fftw3,buildin_glew,buildin_zlib,fortran,gviz,qt5web,sqlite,webgui,再次搜索not并补充必要的库。
`make`
`make install`
`echo "source $HOME/program/root/install/bin/thisroot.sh" >> ~/.bash_profile`
`source ~/.bash_profile`

运行jupyter：root --notebook，新建pytho3编辑
```
import ROOT
c=ROOT.TCanvas("","",400,300)
c.Draw()
```
即可显示绘图。

5. 安装geant4

* 安装clhep

本来geant4自带clhep，但是运行实例可能会有“段错误”等版本问题，建议编译，很快

build, install, src三文件夹编译安装

* 安装geant4
build, install, src三文件夹编译安装
`cd install`
`mkdir -p share/Geant4-10.6.2/data`
把下载的所有数据文件复制到上面的data文件夹下，在终端进入上面data文件夹
`ls *.tar.gz | xargs -n1 tar xzvf`
`rm -rf *.tar.gz`
打开cmake3-gui，点击Configure，设置prefix，USE_QT选中，USE_SYSTEM_CLHEP选中，python选中，再点Configure，指定CLHEP_DIR路径install/lib/CLHEP-2.4.1.3，再Configure，直至没有问题。点Generate。关闭cmake3-gui。
```
make
make install
```
(make过程中很可能会遇到找不到gl.h等文件的错误，即使你已经安装了。这些库文件在/usr/include/GL目录，可以直接复制到src/source/visualization/externals/gl2ps/include/GL路径，灵活点看好路径)
`echo "source $HOME/program/geant4/install/bin/geant4.sh" >> ~/.bash_profile`
`source ~/.bash_profile`

* 编译示例
```
mkdir /home/cen/program/geant4/test
cp -r /home/cen/program/geant4/src/examples/basic/B1 /home/cen/program/geant4/test
cd /home/cen/program/geant4/test/B1
mkdir build
cd build
cmake ..
make
./exampleB1
```

6. 安装caen
根据readme安装caen软件，解压ls *.tgz | xargs -n1 tar xvf，安装顺序：jre,vme，comm,usb,digitizer,upgrader,compass,wavedump
得到程序：CAENUpgraderGUI,CoMPASS，建立快捷方式即可，待续

7. 安装pkuxiadaq
在编译pkuxiadaq时，遇到root不兼容问题
修改root的TGraph.h文件，加#include "TF1.h"
修改Offline.cc，加#include "TVirtualX.h"
如果提示没有.a文件，可能是忘记运行export PLX_SDK_DIR=$HOME/PKUXIADAQ/PlxSdk

8. Garfieldpp安装。 按照官方网站安装步骤，分别输入下面四条命令：
`export GARFIELD_HOME=/home/mydir/garfield`
`git clone https://gitlab.cern.ch/garfield/garfieldpp.git $GARFIELD_HOME`
`cd $GARFIELD_HOME`
`make`
保持网络畅通，可以安装成功。以后保持代码更新用git pull origin master，再make（待测试）。进入到示例目录，如cd Example/Gem，运行make即可得到一个gem可执行文件。上述过程在CentOS和MaxOS下都测试了。要说明的是，在macos下运行实例不成功，需要解决链接gfortran库的问题。

9. 其它
后记
* centos7-2003中geant4编译不过去，其它没有问题。
* 把文件夹添加到侧边栏,直接拖动上去即可。
* nscl daq
* 考虑用conda pack打包环境，到同样配置的机器上重现环境。学习中，可在虚拟机上测试。


