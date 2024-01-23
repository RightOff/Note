# Ubuntu20.04安装

## 报错：ERROR：Unable to find the development tool `cc` in your path

先更新找到gcc，再下载

```
sudo apt-get update
sudo apt-get install gcc
```

## 报错：gpg: no valid OpenPGP data found.

```
wget https://download.docker.com/linux/ubuntu/gpg
sudo apt-key add gpg
```

## 报错：E: Unable to locate package libnvidia-container-dev E: Unable to locate package libnvidia-container-tools E: Unable to locate package nvidia-container-runtime

```
sudo apt update
```

## Failed to add partition 1 to system: Invalid argument

创建分区的时候 通过命令来新建一份空的分区表：o

![1696494689251](image/Linux/1696494689251.png)

## 右键没有从终端中打开的选项

```
sudo apt update
sudo apt install nautilus-extension-gnome-terminal
注销后重新登录即可
```

Firefox乱码

```
# 1.安装wqy-zenhei字体
apt-get install ttf-wqy-zenhei
# 2.修改系统字体配置文件/etc/fonts/conf.d/49-sansserif.conf
vim /etc/fonts/conf.d/49-sansserif.conf
# 3.然后修改倒数第四行的字体为WenQuanYi Zen Hei，如下:

<?xml version="1.0"?>
<!DOCTYPE fontconfig SYSTEM "fonts.dtd">
<fontconfig>
<!--
If the font still has no generic name, add sans-serif
-->
<match target="pattern">
<test qual="all" name="family" compare="not_eq">
<string>sans-serif</string>
</test>
<test qual="all" name="family" compare="not_eq">
<string>serif</string>
</test>
<test qual="all" name="family" compare="not_eq">
<string>monospace</string>
</test>
<edit name="family" mode="append_last">
<string>WenQuanYi Zen Hei</string>
</edit>
</match>
</fontconfig>
```

## 报错合集

### 异常重启后不能用cuda

```
# 给宿主机增加nvidia-uvm设备
sudo /sbin/modprobe nvidia-uvm
D=`grep nvidia-uvm /proc/devices | awk '{print $1}'`
sudo mknod -m 666 /dev/nvidia-uvm c $D 0
# 第三步可能会报错，不影响使用,上述命令执行完成之后，重启容器
```

## 常用指令

```
lxc stop --all //关闭所有容器
```

## 安装步骤

### 系统文件制作

共享云盘中下载Ubuntu-20.04.5-desktop-amd64.iso镜像文件，下载Rufus软件，在其上边进行系统盘的制作。

### SSH

```
apt update
apt install vim
apt install net-tools
apt install openssh-server
```

修改/etc/ssh/sshd_config，可以使用密码登录

PermitRootLogin yes
PasswordAuthentication yes

```
service ssh start
#开机自启动(默认)
systemctlenable ssh
```

### 存储池

查看存储池：

```
zpool list
```

初始化LXD

```
sudo lxd init
```

```
Would you like to use LXD clustering? (yes/no) [default=no]: 
Do you want to configure a new storage pool? (yes/no) [default=yes]: yes
Name of the new storage pool [default=default]: rpool
Name of the storage backend to use (ceph, btrfs, dir, lvm, zfs) [default=zfs]: zfs
Would you like to create a new zfs dataset under rpool/lxd? (yes/no) [default=yes]: yes
Would you like to connect to a MAAS server? (yes/no) [default=no]: 
Would you like to create a new local network bridge? (yes/no) [default=yes]: 
What should the new bridge be called? [default=lxdbr0]: lxdbr0
What IPv4 address should be used? (CIDR subnet notation, “auto” or “none”) [default=auto]: 
What IPv6 address should be used? (CIDR subnet notation, “auto” or “none”) [default=auto]: 
Would you like the LXD server to be available over the network? (yes/no) [default=no]: 
Would you like stale cached images to be updated automatically? (yes/no) [default=yes] 
Would you like a YAML "lxd init" preseed to be printed? (yes/no) [default=no]: 
```

### 容器

创建容器

```
lxc launch ubuntu:20.04 tf
# 查看容器
lxc list
# 查看本机已有的镜像
lxc image ls
# 进入容器操作
lxc exec tf bash
```

### 显卡驱动

安装显卡驱动

```
# 1.卸载旧驱动
sudo apt-get purge nvidia*
sudo apt-get autoremove
sudo reboot 
```

禁止Nouveau

```
sudo vim /etc/modprobe.d/blacklist.conf 
# 在文件最后增加下面两行：
blacklist nouveau  
options nouveau modeset=0 
```

```
sudo update-initramfs -u
# 更新后重启
```

检查禁止是否成功（没有任何输出代表禁止成功）

```
lsmod | grep nouveau
```

取得超级权限，关闭所有NVIDIA驱动。

```
sudo -i
systemctl isolate multi-user.target
modprobe -r nvidia-drm
```

安装显卡驱动

```
# 准备工作
sudo apt-get install gcc
sudo apt-get install make
```

```
sudo chmod a+x NVIDIA-Linux-x86_64-535.113.01.run
sudo ./NVIDIA-Linux-x86_64-535.113.01.run -no-x-check -no-nouveau-check -no-opengl-files
```

以下页面选yes

![1696669843597](image/Linux/1696669843597.png)

查看驱动安装是否成功

```
nvidia-smi
```

![1696669957097](image/Linux/1696669957097.png)

### 安装cuda

```
wget https://developer.download.nvidia.com/compute/cuda/11.3.1/local_installers/cuda_11.3.1_465.19.01_linux.run
sudo sh cuda_11.3.1_465.19.01_linux.run
```

其中有个选择需要取消第一项再安装

配置环境变量

```
vim ~/.bashrc
```

export PATH=/usr/local/cuda-11.3/bin:$PATH
export LD_LIBRARY_PATH=/usr/local/cuda-11.3/lib64:$LD_LIBRARY_PATH

```
source ~/.bashrc
```

测试是否装好

nvcc -V

### 安装cudnn

![1696765890752](image/Linux/1696765890752.png)

```
dpkg -i libcudnn8_8.2.1.32-1+cuda11.3_amd64.deb
```

检查是否安装成功

```
dpkg -l | grep cudnn
```

### 安装nvidia-container-runtime

安装nvidia-container-runtime（Nvidia官方出的容器运行GPU组件，安装后在容器中就可以直接调用宿主机的显卡驱动。）

```
# 准备工作
sudo snap install curl
sudo -i
```

```
# Add the package repositories
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | apt-key add -

distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | tee /etc/apt/sources.list.d/nvidia-docker.list
apt-get update

apt install libnvidia-container-dev libnvidia-container-tools nvidia-container-runtime -y
```

### 容器中安装驱动

```
mkdir share
sudo lxc config set tf security.privileged true
sudo lxc config device add tf share disk source=/share path=/share
```

添加GPU硬件
在宿主机中执行以下命令

```
lxc config device add tf gpu gpu
```

复制显卡驱动到共享文件夹

```
cp /home/eai3/NVIDIA-Linux-x86_64-535.113.01.run /share
```

安装驱动，与上边的设置相同（因为LXD是复用了Linux内核，所以在容器内安装显卡驱动就不用安装内核了）

```
lxc exec tf bash # 进入容器
# 进入驱动所在文件夹下
sudo sh ./NVIDIA-Linux-x86_64-535.113.01.run --no-kernel-module
```

查看是否安装成功

```
nvidia-smi
```

### 配置远程桌面

（Linux常用的桌面程序有：GNOME、KED、Xfce，可以根据所需安装使用。我这里使用了Ubuntu默认的GNOME，注意取消推荐软件的安装：）

```
lxc exec tf bash # 进入容器
sudo apt-get update
sudo apt install --no-install-recommends ubuntu-desktop
```

Xfce桌面（远程桌面访问的协议有RDP、VNC，由于RDP是Windows自带的工具，所以我们直接只用它了。通过以下命令进行安装：）

```
sudo apt-get update
sudo apt-get install xrdp
# 将xrdp用户添加到组：
sudo adduser xrdp ssl-cert  
sudo systemctl restart xrdp
```

### 配置网络

```
# 宿主机中
sudo lxc config device add tf proxy1 proxy listen=tcp:192.168.31.234:1002 connect=tcp:10.188.240.123:22 bind=host
sudo lxc config device add tf proxy0 proxy listen=tcp:192.168.31.234:1003 connect=tcp:10.188.240.123:3389 bind=host
```

主要需要注意的地方是：tf是我的容器名，前边是我宿主机的ip，后边是我容器的ip。每个人占俩端口，分别用来映射ssh的22端口，和rdp协议的3389端口。容器的ip可以通过lxc list查看，本机的ip可以通过ifconfig查看。
这样都做好后，我们就可以把10.0.5.11:1002 、10.0.5.11:1003 给到小伙伴，让他远程访问了。

当然，root密码你如果忘了，可以进入容器通过passwd root修改root密码。（刚创建完需先设置密码）

```
lxc exec tf bash # 进入容器
passwd root
```

如果显示服务器拒绝密码，则可能是ssh默认不接受密码登录：

```
vim /etc/ssh/sshd_config
#PermitRootLogin without-password改为
PermitRootLogin yes
#重启ssh服务
/etc/init.d/ssh restart
```

### 桌面优化

```
# 安装 NetworkManager 服务
sudo apt install network-manager
# 安装完后重启（不重启也行，以下可以不用做）
sudo reboot
# 启动NetworkManager 服务
sudo systemctl start NetworkManager
# NetworkManager 在启动时自动运行
sudo systemctl enable NetworkManager
# 检查 NetworkManager 状态
sudo systemctl status NetworkManager
```

```
sudo apt update
# 安装图标主题
sudo apt install papirus-icon-theme
# 安装GTK主题
git clone https://github.com/vinceliuice/vimix-gtk-themes
cd vimix-gtk-themes
sudo ./install.sh
# 安装Gnome Tweak Tool
sudo apt install gnome-tweak-tool
# 显示Linux系统信息工具
sudo apt install neofetch
# 查看CPU运行以及内存占用情况工具
sudo apt install htop
```

然后打开tweaks，在里边进行设置

```
# 文件夹下右键中没有终端的解决方法：
sudo apt update
sudo apt install nautilus-extension-gnome-terminal
注销后重新登录即可
```

### 安装anaconda

宿主机中

```
apt install git # 前期准备
apt install pip # 前期准备
```

宿主机中安装anaconda

在浏览器中下载安装包

```
bash /home/eai3/Anaconda3-2023.09-0-Linux-x86_64.sh # /后边是路径
```

回车

在阅读完协议最后输入yes

回车

yes

yes

重新打开命令行

```
# conda 的基础环境在启动时不被激活
conda config --set auto_activate_base false
# 创建python3.9的环境
conda create -n lxdui python=3.9
conda activate lxdui
```

### 用镜像创建容器

```
lxc stop my-container
lxc publish my-container --alias test-image
lxc image list
lxc launch 镜像名称 新容器名称
```

配置网络

```
# 宿主机中
sudo lxc config device add tf proxy1 proxy listen=tcp:192.168.31.234:1002 connect=tcp:10.188.240.123:22 bind=host
sudo lxc config device add tf proxy0 proxy listen=tcp:192.168.31.234:1003 connect=tcp:10.188.240.123:3389 bind=host
# 如果无法连接，可进行以下修改：具体的修改取决于你要更改的属性。在你的示例中，要修改 "proxy1" 设备的监听地址，你可以运行类似以下的命令：
sudo lxc config device set tf proxy1 listen=tcp:新监听地址
```

### 容器中安装cudnn

与上边相同

### 容器中安装pytorch

### 可选项目：

#### 可视化管理界面设置：

云盘中的lxd39放到服务器/root/anaconda/envs/目录下

解压

```
conda activate lxd39
python setup.py install
pip install Werkzeug==2.0.0
lxdui start

```

进入页面控制：ip:15151

```
# 通用方案✖：以下方案很大可能不行
git clone https://github.com/AdaptiveScale/lxdui.git
cd lxdui
# 安装依赖包
python3 setup.py install

# 启动: 
python3 run.py start
# 或者使用CLI启动:   
lxdui start
# 后台运行
nohup python3 run.py start > python.log3 2>&1 &
```

#### 安装firefox

```
sudo apt update && sudo apt upgrade
sudo apt install firefox
```

#### 最小最大化按钮

```
# 切换到右上角
gsettings set org.gnome.desktop.wm.preferences button-layout ":minimize,maximize,close"
# 切换到左上角
gsettings set org.gnome.desktop.wm.preferences button-layout "minimize,maximize,close:"
```

### 其他

```
# 查看容器剩余容量
df -hl
# 查看剩余内存
free -m
# 查看宿主机容量
zfs list
```

# 环境配置及问题解决

## 示例

使用环境：

- Python 3.7
- Pytorch 1.8.1

1.安装Pytorch

如果已经安装过Pytorch，则无需再次安装

```shell
pip install torch==1.8.1
```

2.安装其他依赖库，命令如下

```shell
pip install -r requirements.txt -i https://mirrors.aliyun.com/pypi/simple/
```

具体步骤：

```
(base) root@clh1:~# cd /
(base) root@clh1:/# mkdir git
(base) root@clh1:/# cd /git

//git 项目，在项目里配
(base) root@clh1:/git# git clone https://github.com/yeyupiaoling/VoiceprintRecognition-Pytorch.git

(base) root@clh1:/git# conda create -n voice python=3.8

(base) root@clh1:/git# conda activate voice
(voice) root@clh1:/git# conda install pytorch==1.8.1 torchvision==0.9.1 torchaudio==0.8.1 cudatoolkit=10.2 -c pytorch

(base) root@clh1:~# conda activate voice
(voice) root@clh1:~# cd /git/VoiceprintRecognition-Pytorch/
(voice) root@clh1:/git/VoiceprintRecognition-Pytorch# sudo apt-get install portaudio19-dev
(voice) root@clh1:/git/VoiceprintRecognition-Pytorch# sudo apt-get install python3-all-dev
(voice) root@clh1:/git/VoiceprintRecognition-Pytorch# pip install -r requirements.txt -i https://mirrors.aliyun.com/pypi/simple/
```

## yolov5

下载yolov5-5.0文件

    https://github.com/ultralytics/yolov5/releases/tag/v5.0

    文件传到Linux的Desktop后解压

创建虚拟环境

    conda create -n yolov5 python==3.7

    conda activate yolov5

安装cuda和cudnn

```
conda install cudatoolkit=10.1 -c https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/linux-64/
conda install cudnn=7.6.5 -c https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/linux-64/
```

换国内源：

```
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
```

安装pytorch

```
pip install torch==1.8.1+cu101 torchvision==0.9.1+cu101 torchaudio==0.8.1 -f https://download.pytorch.org/whl/torch_stable.html
```

进入yolov5文件夹

修改requirement内容：

```
# pip install -r requirements.txt

# base ----------------------------------------
Cython
matplotlib>=3.2.2
numpy>=1.18.5
opencv-python>=4.1.2
Pillow
PyYAML>=5.3.1
scipy>=1.4.1
tensorboard>=2.2
torch>=1.7.0
torchvision>=0.8.1
tqdm>=4.41.0

# logging -------------------------------------
# wandb

# plotting ------------------------------------
seaborn>=0.11.0
pandas

# export --------------------------------------
# coremltools==4.0
# onnx>=1.8.0
# scikit-learn==0.19.2  # for coreml quantization

# extras --------------------------------------
thop  # FLOPS computation
pycocotools>=2.0  # COCO mAP
```

```
pip install  -U  -r   requirements.txt
```

报错1.可能会出现自动升级torch导致不匹配问题，重新安装对应版本torch即可

报错2：`error: Microsoft Visual C++ 14.0 or greater is required` ：安装vs2019

报错3：

```
No module named ‘git‘	/'name'
解决：
pip install gitpython	/pip install name
```

报错4：`ImportError: Failed to initialize: Bad git executable：`：添加以下一行

```
import argparse
import math
import os
os.environ["GIT_PYTHON_REFRESH"] = "quiet"	//添加该行
import random
import subprocess
import sys
import time
from copy import deepcopy
from datetime import datetime
from pathlib import Path
```

PS：曾经安装的版本，

pip uninstall torch==1.13.1 torchvision==0.14.1

conda install pytorch==1.7.1 torchvision==0.8.2 torchaudio==0.7.2 cudatoolkit=10.1 -c pytorch

## yolov7

下载yolov7文件

[GitHub - WongKinYiu/yolov7: Implementation of paper - YOLOv7: Trainable bag-of-freebies sets new state-of-the-art for real-time object detectors](https://github.com/WongKinYiu/yolov7)

创建虚拟环境(或者直接用v5的环境)

```
conda create -n yolov7 python==3.8  
conda activate yolov7
```

切换到v7文件夹下

安装requirement

```
pip install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple
```

换源，安装GPU版torch

```
conda install pytorch==1.7.1 torchvision==0.8.2 torchaudio==0.7.2 cudatoolkit=11.0

//废弃
pip install torch==1.8.1+cu101 torchvision==0.9.1+cu101 torchaudio==0.8.1 -f https://download.pytorch.org/whl/torch_stable.html
```

手动下载yolov7.pt文件放在v7文件夹下

运行detect进行测试

训练

```
//报错File "/root/anaconda3/envs/yolov7/lib/python3.8/site-packages/torch/serialization.py", line 762, in _legacy_load
    magic_number = pickle_module.load(f, **pickle_load_args)
_pickle.UnpicklingError: STACK_GLOBAL requires str

//删除数据集文件中产生了.cache文件
```


## yolov8

### 安装

下载yolov8代码gitee:   [ultralytics: YOLOv8 🚀 Ultralytics 同步更新官方最新版 YOLOv8 (gitee.com)](https://gitee.com/monkeycc/ultralytics)

github：[mirrors / ultralytics / ultralytics · GitCode](https://gitcode.net/mirrors/ultralytics/ultralytics?utm_source=csdn_github_accelerator)

安装conda、cuda

创建环境：

```
conda create -n clh python=3.8
conda activate clh
```

换国内源：

```
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
//ps:清空源
conda config --remove-key channels
```

安装pytorch：

```
# Windows
conda install pytorch==1.12.0 torchvision==0.13.0 torchaudio==0.12.0 cudatoolkit=11.3 -c pytorch

//使用新版torch,旧版1.12.0会导致训练时无法使用多线程载入数据集
conda install pytorch==2.1.0 torchvision==0.16.0 torchaudio==2.1.0 pytorch-cuda=12.1 -c pytorch -c nvidia

# 以下方法废弃
ubuntu cuda:11.3 cudnn:8.2.1
conda install pytorch==1.12.0 torchvision==0.13.0 torchaudio==0.12.0 cudatoolkit=11.3 -c pytorch
# 换源更快速
conda install pytorch==1.12.0 torchvision==0.13.0 torchaudio==0.12.0 cudatoolkit=11.3 -c https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/pytorch/linux-64/

# ubuntu cuda:12.2
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple	# 换源
conda install pytorch torchvision torchaudio pytorch-cuda=11.8 -c pytorch -c nvidia

```

验证pytorch

```
import torch
import torchvision
torch.cuda.is_available() 
```

安装requirement：

```
pip install -r requirements.txt 
```

安装yolov8在python>=3.8版本必要安装包

```
pip install ultralytics
```

下载yolovn.pt：https://github.com/ultralytics/assets/releases/download/v0.0.0/yolov8n.pt

使用yolov8自带经典图片进行测试：

```
yolo predict model=yolov8n.pt source='ultralytics/assets/bus.jpg'
```

训练自己数据集：

```
yolo train data=data.yaml model=yolov8s.pt epochs=300 imgsz=640 batch=8 workers=0 device=0

data为yaml配置文件
model为下载的模型，放在主文件下
epochs为训练轮数
imagez为训练时ai看到的图片大小，检查大图片建议使用640，小图片可以320 越大越吃性能
batch为一轮训练中每一次放入图片数量，越大越快效果越好，但是对性能要求越高
device为使用的设备，使用cpu练就写cpu，使用显卡大多数都是0，多显卡就0，1，2，3，...多少显卡往后写多少

```

### 换GPU训练

修改 `ultralytics/engine/trainer.py`文件下的 `self.device = torch.device(0)`

## yolo训练网络可视化工具

```
pip install onnx	# 虚拟环境下安装
```

在YOLOv5的models下

## rtdetr

按照yolov8安装完pytorch，再进行以下操作

```
//额外需要的包安装命令:
pip install timm thop efficientnet_pytorch einops grad-cam dill -i https://pypi.tuna.tsinghua.edu.cn/simple
//以下主要是使用dyhead必定需要安装的包,如果安装不成功dyhead没办法正常使用!
pip install -U openmim
mim install mmengine
mim install "mmcv>=2.0.0"

//还需要安装的包
pip install seaborn
pip install seaborn
```

修改v8的文件内容

1. 在ultralytics/engine/trainer.py中的optimizer_step函数中的torch.nn.utils.clip_grad_norm_中的参数max_norm修改为0.1
2. 在ultralytics/engine/trainer.py中的_setup_train函数中将self.args.nbs等于self.batch_size,这样做的目的是让模型不需要积累梯度再进行更新参数
3. ultralytics/cfg/default.yaml配置文件的更改


## Faster-RCNN

创建虚拟环境

```
conda create -n rcnn python=3.6
```

配置清华镜像源

```
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge 
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/msys2/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/pytorch/
 
# 设置搜索时显示通道地址
conda config --set show_channel_urls yes
```

安装pytorch

```
conda install pytorch==1.7.1 torchvision==0.8.2 torchaudio==0.7.2 cudatoolkit=11.0
```

下载faster-rcnn代码

```
https://github.com/jwyang/faster-rcnn.pytorch/tree/pytorch-1.0
```

代码包中创建文件

```
cd faster-rcnn.pytorch-pytorch-1.0 && mkdir data
cd data && mkdir pretrained_model
```

下载预训练模型VGG16,放到/data/pretrained_model中

```
https://link.csdn.net/?target=https%3A%2F%2Ffilebox.ece.vt.edu%2F~jw2yang%2Ffaster-rcnn%2Fpretrained-base-models%2Fvgg16_caffe.pth
```

安装依赖

```
pip install -r requirements.txt 
//报错Could not build wheels for opencv-python, which is required to install pyproj，解决如下
pip install -i https://pypi.douban.com/simple/ pip install opencv-python==4.3.0.38
```

之后

```
cd lib
python setup.py build develop
```

安装CoCO API

```
cd data
git clone https://github.com/pdollar/coco.git
cd coco/PythonAPI
make
cd ../../..
```

修改数据集/data/VOCdevkit2007

修改类别，文件`lib/datasets/pascal_voc.py`

![1705847432893](image/Linux/1705847432893.png)

修改参数（可选），文件 `/lib/model/utils/config.py`

cannot import name ‘imread’

```
//对应文件中修改from scipy.misc import imread为以下
from imageio import imread
```

在/lib/model/utils/config.py中的374行

```
//yaml_cfg = edict(yaml.load(f))修改为
yaml_cfg = edict(yaml.load(f, Loader=yaml.FullLoader))
```

 cannot import name 'builder’

```
pip uninstall protobuf
pip install protobuf==4.21.0
```

File "/py-faster-rcnn/tools/../lib/datasets/imdb.py", line 108, in append_flipped_images
    assert (boxes[:, 2] >= boxes[:, 0]).all() AssertionError

```
修改lib/datasets/imdb.py，append_flipped_images()函数
数据整理，在一行代码为 boxes[:, 2] = widths[i] - oldx1 - 1下加入代码：
for b in range(len(boxes)):
  if boxes[b][2]< boxes[b][0]:
    boxes[b][0] = 0

修改lib/datasets/pascal_voc.py
将对Xmin,Ymin,Xmax,Ymax减一去掉
```



制作数据集，使用voc格式数据集，txt_to_val、txt_to_val2为转换代码

data/VOCdevkit2007/VOC2007文件下

| Annotations | 存放数据打标签后的xml文件    |
| ----------- | ---------------------------- |
| Main        | 存放图片的名字和正负样本标签 |
| JPEGImages  | 存放图片                     |

在Main中有四种txt文件

trainval	存放全部的训练集和验证集图片的名字，不要带后缀，比如图片是0.jpg，就写0就行了
train	存放全部的训练集图片名字，占trainval的50%
val	存放全部的验证集图片名字，占trainval的50%
test	存放全部的测试集图片名字


训练loss可视化

```
pip install tensorboard
//报版本不匹配
pip uninstall protobuf
pip install protobuf==3.9.2

tensorboard --logdir=logs/logs_s_1/losses/ --port=7001
```

训练开始前

```
删除缓存：将faster-rcnn.pytorch/data/cache文件夹里面的东西全删掉
删除模型：将faster-rcnn.pytorch/models/res101/pascal_voc文件夹里面的东西全删掉
```

训练

```
CUDA_VISIBLE_DEVICES=0 python trainval_net.py  --dataset pascal_voc --net vgg16 --bs 16 --nw 16 --lr 0.001 --lr_decay_step 5 --cuda --use_tfb  --epochs 100
```

测试

修改文件test_net.py中的test为val

```
python test_net.py --dataset pascal_voc --net vgg16 --checksession 1 --checkepoch 1 --checkpoint 473 --cuda
//473为文件编号
```



# 问题解决

## 创建环境时连接超时

错误信息：

```
CondaHTTPError: HTTP 000 CONNECTION FAILED for url
```

输入 `conda --help `找到.condarc文件路径，如下图：

![1702907586256](image/Linux/1702907586256.png)

打开文件然后修改里边的https为http，并在末尾加上：`ssl_verify: false`，即可。

## root用户无法启动vscode

```
vi ~/.bashrc
# 添加一行
alias code='/usr/share/code/code . --no-sandbox --unity-launch'
# 报存生效
source ~/.bashrc
```

# Linux命令大全

## 基础命令

* ```
  nvidia-smi		//N卡使用情况
  ```
* ```
  watch -n 1 nvidia-smi	//每隔1秒输出显卡使用情况，Ctrl+C结束watch命令
  ```
* ```
  实时查看GPU使用状态（Windows首选使用）
  pip install gpustat
  gpustat --w
  ```
* ```
  nvcc -V 		//cuda版本查询
  ```
* 删除行：1、按Esc键进入正常模式。 2、将光标放在要删除的行上。 3、键入dd并按Enter键以删除该 行 。
* **从Windows下载Linux服务器中的文件夹**

  ```
  scp -r -P 10002 root@192.168.31.61:/root/yolov5_research/yolov5 d:\LinuxFileTemp
  ```

  -r ：表示传文件夹，-P ：表示用端口号，root@192.168.31.61:/root/Music ：待传文件地址，d:\ ：下载到的地址
* ```
  pip install matplotlib -i https://pypi.tuna.tsinghua.edu.cn/simple		//安装matpotlib
  ```
* pycharm运行代码指定GPU

  ```
  import os
  # 指定使用0,1,2三块卡
  os.environ["CUDA_VISIBLE_DEVICES"] = "0,1,2"
  ```

+ ```
  lsb_release -a		//查看操作系统版本
  ```
+ ```
  cat /proc/cpuinfo   # 查看CPU信息
  ```
+ ```
  python -V	# python版本
  ```
+ ```
  pip list	# 虚拟环境目前包含的库信息
  ```
+ ```
  Python中：
  import torch
  print(torch.version.cuda)	# 查询cuda版本
  print(torch.backends.cudnn.version())	# 查询cudnn版本

  #查看使用的GPU设备数量和名字
  torch.cuda.current_device()
  torch.cuda.device(0)
  torch.cuda.device_count()
  torch.cuda.get_device_name(0)
  ```

* ```
  python -V   # python版本
  ```

+ linux批量解压zip文件

  ```
  //方法1
  find . -name '*.zip' -exec unzip {} \;
  1
  //方法2
  ls *.zip | xargs -n1 unzip -o
  1
  //方法3
  # for i in *.zip
  > do
  > unzip -o $i
  > done
  ```
+ 在vim下查找关键词：ESC+/
+ ```
  //实时网速
  apt install ifstat	//安装
  ifstat	//打开
  ```

## Anaconda

+ conda create -n tensorflow python=3.8		//创建虚拟环境
+ conda create --prefix=D:\SoftWare\Anaconda3\envs\yolov5 python=3.7	//指定位置创建虚拟环境
+ deactivate	//退出环境
+ conda env remove --name your_env_name	//删除环境
+ conda info --envs	或者 conda env list	//虚拟环境列表
+ conda remove -n octopus --all	//删除octopus环境
+ conda info --envs 或者 conda env list	//查看虚拟环境列表
+ conda activate paddle	//切换到paddle虚拟环境
+ pip list	//查看pytorch中安装的包
+ conda remove -n pytorch --all
+ 

## cp 复制文件、文件夹到文件夹

### 参数

a 该选项通常在拷贝目录时使用。它保留链接、文件属性，并递归地拷贝目录，其作用等于dpR选项的组合。

* d 拷贝时保留链接。
* f 删除已经存在的目标文件而不提示。
* i 和f选项相反，在覆盖目标文件之前将给出提示要求用户确认。回答y时目标文件将被覆盖，是交互式拷贝。
* p 此时cp除复制源文件的内容外，还将把其修改时间和访问权限也复制到新文件中。
* r 若给出的源文件是一目录文件，此时cp将递归复制该目录下所有的子目录和文件。此时目标文件必须为一个目录名。
* l 不作拷贝，只是链接文件。

### 复制文件到文件夹

cp /待复制文件的路径  /目标路径

```
cp /home/downloads/xampp-linux-x64-7.3.6-0-installer.run /opt/

```

### 复制文件夹到文件夹

-r 是区别点

```
cp  -r /home/downloads/phpcms_v9_UTF8/install_package/    /opt/lampp/htdocs/
cp -r tired_driver    /root/yolov5\ search/
```

# 其他

## pytorch 如何使用cuda

1. 在最上面放进去

   ```
   device = torch.device("cuda:0" if torch.cuda.is_available() else "cpu")
   ```
2. 网络进入device

   ```
   net.to(device)
   ```
3. 数据进入device

   ```
   inputs, labels = inputs.to(device), labels.to(device)
   ```

在验证使用的时候，还是要把image进入到device里面，然后再进入net里面，总之，所有的要么都进GPU，要么都不进GPU

$$
\text{Contribution\ Ratio} = \frac{\text{Explained Variance}}{\text{Total Variance}}
$$
