---
layout: post
title: CentOS7安装与配置之三
category: python
---
​
用脚本解决系统的安装，配置和编译问题。

系统环境：CentOS7-1810 x86_64

1. 虚拟机及配置，目前没有脚本（实体机安装可以跳过）

    在设置中添加共享文件夹  
    在centos7系统下运行 `vmware-hgfsclient`  
    创建 `sudo mkdir /mnt/hgfs`  
    手动挂载 `sudo vmhgfs-fuse .host:/forall /mnt/hgfs`  
    创建自动挂载`sudo vi /etc/fstab`，按`i`，添加一行`.host:/forall /mnt/hgfs fuse.vmhgfs-fuse allow_other, defaults 0 0`  
    按`ESC`，按住`shift`按两次`z`  
    重启系统，OKin，records out，就完成了。

2. 系统初始化，基本工具安装，ntfs格式支持，sh 00getntfs.sh

    把事先准备的文件复制到安装u盘里，安装完系统马上把它们复制到Downloads目录下。

    ```bash
    echo "passwd" | sudo -S ls
    sudo rm -f /var/run/yum.pid
    sudo sed -e 's|^mirrorlist=|#mirrorlist=|g' \
         -e 's|^#baseurl=http://mirror.centos.org|baseurl=https://mirrors.tuna.tsinghua.edu.cn|g' \
         -i.bak \
         /etc/yum.repos.d/CentOS-*.repo
    sudo yum makecache

    sudo yum install epel-release -y
    sudo sed -e 's!^metalink=!#metalink=!g' \
    -e 's!^#baseurl=!baseurl=!g' \
    -e 's!//download\.fedoraproject\.org/pub!//mirrors.tuna.tsinghua.edu.cn!g' \
    -e 's!http://mirrors\.tuna!https://mirrors.tuna!g' \
    -i /etc/yum.repos.d/epel.repo /etc/yum.repos.d/epel-testing.repo
    sudo yum makecache

    sudo yum install dnf -y
    sudo yum install dnf-plugins-core -y
    sudo dnf install ntfs-3g -y
    sudo dnf install exfatprogs -y
    sudo dnf install yumex -y
    sudo dnf install gnome-shell-extension-dash-to-dock -y
    sudo dnf install gnome-shell-extension-no-topleft-hot-corner -y

    mkdir -p $HOME/program/{clhep,geant4,root}/{build,install,src}
    mkdir -p $HOME/program/geant4/install/share/Geant4-10.7.1/data
    
    sudo sed -i '$a 140.82.113.3 github.com' /etc/hosts
    sudo sed -i '$a 199.232.69.194 github.global.ssl.fastly.net' /etc/hosts
    sudo service network restart

    echo "alias la='ls -a'" >> ~/.bashrc
    source ~/.bashrc
    
    logout

    ```

3. 安装常用开发包，gcc，python等，sh 01rpm.sh

    上一步注销后重新登录，再运行01rpm.sh。

    ```bash
    echo "passwd" | sudo -S ls
    cd dev1
    sudo yum localinstall *.rpm -y

    cd ../devsingle
    sudo yum localinstall *.rpm -y
    source ~/.bashrc

    sudo dnf install centos-release-scl -y
    sudo dnf install devtoolset-8-gcc devtoolset-8-gcc-c++ devtoolset-8-binutils -y
    echo "source /opt/rh/devtoolset-8/enable" >> ~/.bash_profile
    source ~/.bash_profile

    cd ../dev2conda
    sh Miniconda3-py38_4.9.2-Linux-x86_64.sh
    source ~/.bashrc
    conda config --set changeps1 false
    conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
    conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge
    sed -i '$d' ~/.condarc
    conda clean -i
    source ~/.bashrc
    #install online
    conda create --name py36 python=3.6 -y
    conda activate py36
    echo "conda activate py36" >> ~/.bashrc
    source ~/.bashrc
    conda install --use-local pyqt-4.11.4-py36_3.tar.bz2
    conda install --file=requirement_conda.txt -y

    logout
    ```

    上面包含rpm文件的dev1文件夹，实际上可以用下面的脚本维护，嗯，注意网络流量。特地没有添加passwd，留一点缓冲。

    ```bash
    cd dev1
    sudo ls
    sudo dnf download --arch x86_64 --resolve git cmake3 cmake3-gui gcc-c++ gcc binutils libX11-devel libXpm-devel libXft-devel libXext-devel gcc-gfortran openssl-devel pcre-devel mesa-libGL-devel mesa-libGLU-devel glew-devel ftgl-devel mysql-devel fftw-devel cfitsio-devel graphviz-devel avahi-compat-libdns_sd-devel python-devel libxml2-devel xerces-c-devel libXmu-devel hdf5-devel gsl-devel openldap-devel libXt-devel libXmu-devel xerces* expat-devel libXi-devel freeglut pythia8-devel libXtst-devel libXv-devel libxkbfile-devel libXcomposite-devel libXcursor-devel libXdmcp-devel libXinerama-devel libXrandr-devel libXres-devel openblas-devel blas-devel gl2ps libAfterImage-devel libtiff-devel golang motif-devel redhat-lsb-core clang-devel elfutils-libelf-devel cppunit-devel sip qt5-qtbase-devel qt5-qttools-devel qt5-qt3d-devel boost-devel boost-python36-devel Coin3-devel gnuplot expect

    ```

4. 编译安装ROOT6.22.02，clhep2.4.4.1，geant4-10.7.1,sh 02cern.sh

    过程比较漫长，睡觉之前开始就行，早起就可以看到结果了。

    ```bash
    #root
    tar xzvf ./dev3tool/root_v6.22.02.source.tar.gz
    mv ./root-6.22.02/* $HOME/program/root/src
    rm -rf root-6.22.02
    cd $HOME/program/root/build
    cmake3 -DCMAKE_INSTALL_PREFIX:PATH="$HOME/program/root/install" -DPYTHIA8_DATA:PATH="PYTHIA8_DATA-NOTFOUND" -DPYTHIA6_pythia6_dummy_LIBRARY:FILEPATH="PYTHIA6_pythia6_dummy_LIBRARY-NOTFOUND" -Dwebgui:BOOL="1" -Dbuiltin_fftw3:BOOL="1" -DGLEW_DIR:PATH="GLEW_DIR-NOTFOUND" -DCHROME_EXECUTABLE:FILEPATH="/opt/google/chrome" -Dgviz:BOOL="1" -Dqt5web:BOOL="1" -Dfortran:BOOL="1" ../src
    make
    make install
    echo "source $HOME/program/root/install/bin/thisroot.sh" >> ~/.bash_profile
    echo "source $HOME/program/root/install/bin/thisroot.sh" >> ~/.bashrc
    source ~/.bash_profile
    source ~/.bashrc

    #clhep
    tar xzvf ./dev3tool/clhep-2.4.4.1.tgz
    mv ./2.4.4.1/CLHEP/* $HOME/program/clhep/src
    rm -rf 2.4.4.1
    cd $HOME/program/clhep/build
    cmake3 -DCMAKE_INSTALL_PREFIX:PATH="$HOME/program/clhep/install" ../src
    make
    make install
    
    #geant4
    tar xzvf ./dev3tool/geant4.10.07.p01.tar.gz
    mv ./geant4.10.07.p01/* $HOME/program/geant4/src
    rm -rf geant4.10.07.p01

    cd dev3tool
    cp new/* $HOME/program/geant4/install/share/Geant4-10.7.1/data
    cd $HOME/program/geant4/install/share/Geant4-10.7.1/data
    ls *.tar.gz | xargs -n1 tar xzvf
    rm -rf *.tar.gz

    cd $HOME/program/geant4/build
    cmake3 -DCMAKE_INSTALL_PREFIX:PATH="$HOME/program/geant4/install" -DGEANT4_USE_GDML:BOOL="1" -DGEANT4_USE_QT:BOOL="1" -DGEANT4_USE_OPENGL_X11:BOOL="1" -DGEANT4_USE_RAYTRACER_X11:BOOL="1" -DCLHEP_DIR:PATH="$HOME/program/clhep/install/lib/CLHEP-2.4.4.1" -DGEANT4_USE_SYSTEM_CLHEP:BOOL="1" ../src
    make
    make install
    echo "source $HOME/program/geant4/install/bin/geant4.sh" >> ~/.bash_profile
    source ~/.bash_profile

    ls
    mkdir $HOME/program/geant4/test
    cp -r $HOME/program/geant4/src/examples/basic/B1 $HOME/program/geant4/test
    cd $HOME/program/geant4/test/B1
    mkdir build
    cd build
    cmake3 ..
    make

    logout
    ```

    运行jupyter：jupyter notebook，新建pytho3编辑

    ```python
    import ROOT
    c=ROOT.TCanvas("","",400,300)
    c.Draw()
    ```

    运行即可显示绘图。

5. CAEN系列软件安装，sh 03caen.sh

    ```bash
    echo "passwd" | sudo -S ls
    cd dev4caen
    sudo yum localinstall gcc10-libstdc++-10.2.1-7.gf.el7.x86_64.rpm -y
    cd /opt/gcc-10.2.1/usr/lib64
    sudo rm /usr/lib64/libstdc++.so.6
    sudo ln libstdc++.so.6.0.28 /usr/lib64/libstdc++.so.6

    cd $HOME/Downloads/dev4caen
    sudo yum localinstall jre-8u281-linux-x64.rpm -y

    cd caenlinux
    
    tar xzvf CAENVMELib-3.1.0-build20201103.tgz
    cd CAENVMELib-3.1.0/lib
    sudo sh install_x64
    cd ../..
    rm -rf CAENVMELib-3.1.0

    tar xzvf CAENComm-1.4.tgz
    cd CAENComm-1.4/lib
    sudo sh install_x64
    cd ../..
    rm -rf CAENComm-1.4

    tar xzvf CAENUSBdrvB-1.5.4.tgz
    cd CAENUSBdrvB-1.5.4/
    make
    sudo make install
    cd ..
    rm -rf CAENUSBdrvB-1.5.4

    tar xzvf CAENDigitizer_2.15.0_20191001.tgz
    cd CAENDigitizer_2.15.0/
    sudo sh install_64
    cd ..
    rm -rf CAENDigitizer_2.15.0

    #CAENUpgraderGUI
    tar xzvf CAENUpgrader-1.7.0-Build20201118.tgz
    cd CAENUpgrader-1.7.0
    ./configure
    make
    sudo make install
    cd ..
    rm -rf CAENUpgrader-1.7.0

    #CoMPASS
    tar xzvf CoMPASS-1.5.3.tar.gz
    mv CoMPASS-1.5.3/* $HOME/program/compass153
    mkdir $HOME/program/compass153
    mv CoMPASS-1.5.3/* $HOME/program/compass153
    cd $HOME/program/compass153
    sudo sh install.sh
    cd ~/Downloads/dev4caen/caenlinux/
    rm -rf CoMPASS-1.5.3
    c=$(whoami)
    cp compa.desktop compass.desktop
    sed "s/cxjname/$c/" compass.desktop
    ls
    sudo mv compass.desktop /usr/share/applications/

    #wavedump
    tar xzvf wavedump-3.10.0.tar.gz
    cd wavedump-3.10.0/
    ./configure
    make
    make check
    sudo make install
    cd ..
    rm -rf wavedump-3.10.0

    ```

6. 安装pkuxiadaq，sh 04pkuxiadaq.sh

    ```bash
    echo "passwd" | sudo -S ls
    cd dev4caen
    mkdir $HOME/program/pkuxiadaq
    tar xvf pkuxiadaq.tar -C $HOME/program/pkuxiadaq
    cd $HOME/program/pkuxiadaq

    export PLX_SDK_DIR=$HOME/program/pkuxiadaq/PlxSdk
    
    rm -rf PlxSdk
    tar -zxvf PlxSdk.tar.gz
    cd PlxSdk/PlxApi
    make clean
    make

    cd ../Samples/ApiTest/
    make clean
    make

    cd ../../Driver/
    ./builddriver 9054
    
    cd ~/program/pkuxiadaq/software/
    make clean
    make
    #no error is good news

    #cd ~/program/pkuxiadaq/parset/
    #adapt

    cd $HOME/program/root/install/include
    cp TGraph.h TGraph.h.cxjbak
    sed -i '29 a #include "TF1.h"' TGraph.h
    cd ~/program/pkuxiadaq/GUI/
    sed -i '53 a #include "TVirtualX.h"' Offline.cc
    make clean
    make

    cd ~/program/pkuxiadaq/NOGUI/
    make clean
    make

    cd ~/program/pkuxiadaq/OnlineStattics/
    make clean
    make

    cd ~/program/pkuxiadaq/Decode
    make clean
    make
    
    ```

7. 安装Garfieldpp,

    暂时没有写成脚本，四句命令。
    ```bash
    export GARFIELD_HOME=/home/mydir/garfield  
    git clone https://gitlab.cern.ch/garfield/garfieldpp.git $GARFIELD_HOME
    cd $GARFIELD_HOME
    make

    ```

8. 其它。
    后记
    * centos7-2003中geant4编译不过去，其它没有问题。
    * 把文件夹添加到侧边栏,直接拖动上去即可。
    * nscl daq
    * 考虑用conda pack打包环境，到同样配置的机器上重现环境。学习中，可在虚拟机上测试。
    * 期待完全离线安装，更期待kickstart。