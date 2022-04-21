# Quick start: configuration of a new computer in VsisLab

# 1 System installation
### 1.1 Start

镜像源：https://mirrors.tuna.tsinghua.edu.cn/ubuntu-releases/16.04/

ps：最好上官网下载

下载：ubuntu-16.04.6-desktop-amd64.iso /server版需要自己装ubuntu-desktop
使用UltraISO：打开iso文件—启动—写入硬盘映像


### 1.2 Disk Partition
images from the process of ubuntu 20.04 installation 
reboot and keep pressing `del`/`F2`, then set Boot Priority then F10
you have do not make ... yes

select ubuntu - install ubuntu
keyboard - English 
Minial installation - no other options (pull out network line)
choose: `something else`
eg (500G+2T):
> 512M efi \
> 460G / \
> 5G /boot \
> 2T /home 

如果没有efi选项，更换进入系统时选择的启动位置
where are you? shanghai
set user name / system name, then install

complete install - restart now
remove the installation medium, then press enter
insert networkline
进入空系统后分辨率过低是正常现象，是由于未安装驱动所致。


# configuration
##
sudo apt-get install vim gedit net-tools
## replace apt source

### 6.2 apt换源
```shell
sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak
sudo gedit /etc/apt/sources.list
```
按系统版本替换内容： aliyun recommanded
https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu/l,
更新升级

```shell
sudo apt update
sudo apt upgrade
```

## 2 ssh & remote-desktop
### 2.1 ssh

```shell
sudo apt install openssh-server
sudo service ssh start
sudo service ssh status
```
### 3.2 设置静态ip
#### 3.2.1 Ubuntu16.04
- 1、查询网络接口的名字
打开命令行，输入`ifconfig`,第一行最左边的名字，就是本机的网络接口，如enp5s0 
- 2、打开修改文件
	`sudo vim /etc/network/interfaces`
	添加：
	> auto enp5s0 # 使用的网络接口，之前查询接口是为了这里  
	> iface enp5s0 inet static // enp5s0这个接口，使用静态ip设置  
	> address 192.168.199.105 // 设置ip地址  
	> netmask 255.255.255.0 // 设置子网掩码  
	> network 192.168.199.255
	> gateway 192.168.199.1 // 设置网关  
	> dns-nameservers 114.114.114.114 // 设置dns服务器地址
- 3、sudo reboot
#### 3.2.2 Ubuntu 18.04+
https://qizhanming.com/blog/2021/03/18/how-to-config-static-ip-on-ubuntu-20-04
obtain gateway: sudo apt-get install net-tools
route -e
`sudo vim /etc/netplan/*.yaml`
```
network:
	ethernets:
		enp5s0: #配置的网卡名称,使用ifconfig查看
			addresses: [192.168.199.105/24] #设置IP及掩码
			gateway4: 192.168.199.1 #设置网关
			nameservers:
				addresses: [114.114.114.114, 8.8.8.8] #设置DNS
	version: 2
```
`sudo netplan apply` (re-try)
sudo reboot
### 3.3 固件错误Possible missing firmware解决: 
#### 3.3.1 1、进入如下这个地址，固件文件非常全面，找到适合自己的版本
https://git.kernel.org/pub/scm/linux/kernel/git/firmware/linux-firmware.git/tree/rtl_nic/
#### 3.3.2 2、切换到刚才报缺少固件的目录，下载对应的文件内容，
```shell
cd /lib/firmware/rtl_nic/
sudo wget https://git.kernel.org/pub/scm/linux/kernel/git/firmware/linux-firmware.git/tree/rtl_nic/rtl8125a-3.fw
```


### 2.2 remote-desktop: xrdp
#### 2.2.1 Ubuntu 16.04：
```shell
sudo apt-get install xrdp xfce4  #安装xrdp 
sudo apt-get install xubuntu-desktop -f #安装xubuntu-desktop
sudo vim /etc/xrdp/startwm.sh
# 把最下面的test和exec两行注释掉，添加一行xfce4-session
# 以上操作相当于给每个用户echo xfce4-session >~/.xsession
sudo service xrdp restart
sudo reboot
```
全灰屏，鼠标是个叉，可能是因为xrdp版本低，参考：
https://netdevops.me/2017/installing-xrdp-0.9.1-on-ubuntu-16.04-xenial/

vscode通过xrdp远程桌面打开在远程桌面显示，参考：
https://github.com/Microsoft/vscode/issues/3451#issuecomment-227197582
```shell
mkdir ~/lib # make a copy of the relevant library
cp /usr/lib/x86_64-linux-gnu/libxcb.so.1 ~/lib
sed -i 's/BIG-REQUESTS/_IG-REQUESTS/' ~/lib/libxcb.so.1
LD_LIBRARY_PATH=$HOME/lib code # set the dynamic loader path to put your library first before executing VS Code
```

##### 2.2.1.1 一些问题
http://www.c-nergy.be/products.html

没有共享剪切板：版本低（官方Ubuntu16.04的源里只有0.6.1-2的版本）
没有菜单栏、tab补全等：https://www.cnblogs.com/defineconst/p/10254613.html

#### 2.2.2 Ubuntu 18.04：
reference:
- https://blog.csdn.net/fancyboyhou/article/details/105170696
- https://c-nergy.be/products.html
```shell
wget http://www.c-nergy.be/downloads/xrdp-installer-1.1.zip
unzip xrdp-installer-1.1.zip
sudo chmod 777 ./xrdp-installer-1.1.sh 
./xrdp-installer-1.1.sh 
```

1、如已经安装过XRDP，请先删除后再执行安装脚本。
```shell
./xrdp-installer-1.1.sh -r # 删除xrdp软件包
```

2、有时候会出现：用户在系统上远程登录，将无法在本地登录，反之，在本地登录将不能远程登录。

## 3 network
分为两步
- 端口转发：如果是新机器需要设置端口转发，如果是老机器路由器不重置就不需要重新设置。
- 设置静态ip
### 3.1 端口转发
- 在路由器管理界面 找端口转发设置
	- eg: hiwifi.com 互联网-超级端口转发
- 通常需要设置两个端口，一个是ssh端口，另一个是xrdp端口
	- ssh default port 22
	- xrdp default port 3389

## 4 安装Cuda、Cudnn、英伟达驱动

注意⚠️ 3090只支持455及以上驱动，建议使用pytorch时安装cuda11.0，455驱动。
```shell
pip install torch==1.7.0+cu110 torchvision==0.8.1+cu110 torchaudio===0.7.0 -f https://download.pytorch.org/whl/torch_stable.html
```

### 4.1 Cuda、驱动推荐安装方式

#### 4.1.1 快速安装驱动：重启后驱动掉了
1、禁用图形界面

```bash
sudo service lightdm stop # 关闭桌面 

```

或（未装桌面管理器时）：

```bash
sudo systemctl set-default multi-user.target # 关闭桌面
```

2、运行.run文件，选择安装驱动等（Sample、Demo、Document不需要安装）
3、重启图形界面

开启图形界面

```bash
sudo service lightdm start # 开启桌面
```

或（未装桌面管理器时）：

```bash
sudo systemctl set-default graphical.target # 开启桌面
```

#### 4.1.2 Cuda 驱动一键安装（推荐）
1、在官网下载需要版本的Cuda文件（*.run）
2、禁用nouveau第三方驱动

`sudo gedit /etc/modprobe.d/blacklist.conf`

在最后一行添加：blacklist nouveau

```bash
sudo update-initramfs -u # 对所有内核版本操作 加 -k all
sudo reboot
```

3、进入命令行界面并禁用图形界面

执行命令：lsmod | grep nouveau 查看是否禁用,无反应则已禁用
禁用图形界面：

```bash
sudo service lightdm stop # 关闭桌面 

```

或（未装桌面管理器时）：

```bash
sudo systemctl set-default multi-user.target # 关闭桌面
```

4、18.04系统运行*.run文件前可能需要运行sudo apt-get install build-essential命令以安装gcc、g++、make等软件 

5、运行.run文件，选择安装Cuda、驱动等（Sample、Demo、Document不需要安装）
6、重启图形界面

开启图形界面

```bash
sudo service lightdm start # 开启桌面
```

或（未装桌面管理器时）：

```bash
sudo systemctl set-default graphical.target # 开启桌面
```

7、添加环境变量

```bash
export PATH=/usr/local/cuda/bin${PATH:+:${PATH}}
export LD_LIBRARY_PATH=/usr/local/cuda/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
export CUDA_HOME=/usr/local/cuda
```

```bash
source ~/.bashrc
```

如果安装不上：[https://blog.csdn.net/missyoudaisy/article/details/104432746](https://blog.csdn.net/missyoudaisy/article/details/104432746)

#### 4.1.3 其它安装（不推荐，只作参考）
##### 4.1.3.1 驱动安装
###### 4.1.3.1.1 1、禁用nouveau第三方驱动
`sudo gedit /etc/modprobe.d/blacklist.conf`

在最后一行添加：blacklist nouveau
```shell
sudo update-initramfs -u # 对所有内核版本操作 加 -k all
sudo reboot
```
###### 4.1.3.1.2 2、重启后按Ctrl+Alt+F1 进入命令行界面
执行命令：lsmod | grep nouveau 查看是否禁用,无反应则已禁用
禁用图形界面：
```shell
sudo service lightdm stop # 关闭桌面 
sudo service lightdm start # 开启桌面
```
或（未装桌面管理器时）：
```shell
sudo systemctl set-default multi-user.target # 关闭桌面
sudo systemctl set-default graphical.target # 开启桌面
```

###### 4.1.3.1.3 3、安装驱动
```shell
sudo add-apt-repository ppa:graphics-drivers/ppa
sudo apt-get update
ubuntu-drivers devices # 查询所有ubuntu推荐的驱动
sudo apt-get install nvidia-430
```
###### 4.1.3.1.4 4、重启图形界面，查看安装
```shell
sudo service lightdm start
watch -n 0.1 -d nvidia-smi #查看显卡温度 不跑程序时30-50C均可 刚装好驱动时会热一点 多等一会儿
```
##### 4.1.3.2 安装cuda10.1 （10.0兼容性不好）
下载`.deb`文件：
```shell
sudo dpkg -i xxx
sudo apt-key add /var/cuda-repo-<version>/7fa2af80.pub #见上条运行结果
sudo apt-get update
sudo apt-get install cuda
sudo gedit ~/.bashrc
```
添加环境变量：
```
export PATH=/usr/local/cuda/bin${PATH:+:${PATH}}
export LD_LIBRARY_PATH=/usr/local/cuda/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
export CUDA_HOME=/usr/local/cuda
```
```shell
source ~/.bashrc
```
### 4.2 安装Cudnn10.0
有时候需要改名： solitairetheme8-->tgz
```shell
tar -zxvf xxx.tgz 
sudo cp cuda/include/cudnn.h /usr/local/cuda/include/
sudo cp cuda/lib64/libcudnn* /usr/local/cuda/lib64/ -d
```
查看CUDA版本`cat /usr/local/cuda/version.txt`

查看 CUDNN 版本：
```shell
cat /usr/local/cuda/include/cudnn.h | grep CUDNN_MAJOR -A 2
```
或：
```shell
# 新版本cudnn中cudnn.h不再包含版本信息，版本信息包含在cudnn_version.h文件内
find / -name cudnn_version.h 2>&1 | grep -v "Permission denied" # 查找cudnn_version.h文件
cat /usr/local/cuda/include/cudnn_version.h | grep CUDNN_MAJOR -A 2  # 路径需要替换为上述命令搜索到的文件路径
```


## 5 添加用户
### 5.1 添加新用户
```shell
sudo adduser xxx
sudo vim /etc/sudoers #添加root权限,添加：xxx ALL=(ALL) ALL
sudo usermod -G sudo username #给用户添加sudo权限
```
### 5.2 关联原有用户
```shell
sudo useradd -d /home/xxx -s /bin/bash xxx
sudo chown -R xxx:xxx /home/xxx
sudo usermod -aG sudo xxx #添加sudo权限
sudo passwd xxx
```
## 6 换源
### 6.1 pip换源
```shell
pip install pip -U
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
# 推荐
pip config set global.index-url  http://mirrors.aliyun.com/pypi/simple/
```


### 6.3 docker换源
修改/etc/docker/daemon.json并重启docker
```shell
# cat /etc/docker/daemon.json 
{
  "registry-mirrors": ["https://75oltije.mirror.aliyuncs.com"]
}
sudo systemctl restart docker
```

## 7 硬盘挂载

```shell
sudo fdisk -l #查看可挂载的磁盘都有哪些
df -h  #查看已经挂载了哪些磁盘
sudo mkdir /DATA 
sudo mkfs -t ext4 /dev/sda
sudo mount /dev/sda /DATA
vim /etc/fstab 
```
添加到最后一行：UUID=*************  /DATA  ext4  defaults  0  1 
(ls -l /dev/disk/by-uuid | grep sda查看UUID)

## 8 其他安装（可选）
### 8.1 安装anaconda3（导入设置）
#### 8.1.1 配置
```shell
echo 'export PATH="~/anaconda3/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

### 8.2 配置tmux、vim

```shell
sudo apt-get install tmux
sudo apt-get install vim
sudo apt-get install git
sh <(curl https://j.mp/spf13-vim3 -L) # 推荐配置，容易连不上，多试几次
```

### 8.3 zsh & spaceship （配置terminal，可跳过）
https://github.com/spaceship-prompt/spaceship-prompt

```shell
sudo apt-get install zsh
sudo apt-get install fonts-powerline
git clone https://github.com/spaceship-prompt/spaceship-prompt.git "$ZSH_CUSTOM/themes/spaceship-prompt" --depth=1
ln -s "$ZSH_CUSTOM/themes/spaceship-prompt/spaceship.zsh-theme" "$ZSH_CUSTOM/themes/spaceship.zsh-theme"
```
在`~/.zshrc`中设置：`ZSH_THEME="spaceship"`

### 8.4 安装teamviewer

`sudo dpkg -i xxx.deb` 若缺少依赖：sudo apt install -f

### 8.5 安装ToDesk

https://update.todesk.com/todeskBeta_1.1.0c.deb

### 8.6 安装mujoco

- 安装： Follow https://github.com/openai/mujoco-py
- 往~/.bashrc添加环境变量：
```
export LD_LIBRARY_PATH=~/.mujoco/mujoco200/bin${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
```
```shell
cp mjkey.txt ~/.mujoco 
cp mjkey.txt ~/.mujoco/mujoco200/bin
```
```
Export LD_LIBRARY_PATH=~/.mujoco/mujoco200/bin${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
```
```
cd ~/.mujoco/mjpro150/bin 
./simulate ../model/humanoid.xml
```
- Install mujoco-py 参考 https://blog.csdn.net/zhangkzz/article/details/84574772

## 9 常见问题：
### 9.1 重装系统完成后，黑屏左上角光标闪烁：
[https://blog.csdn.net/chengyq116/article/details/102575221](https://blog.csdn.net/chengyq116/article/details/102575221)

1. Advanced options for Ubuntu(开机时按Shift)选择其他内核版本
2. 禁用nouveau驱动
```shell
sudo gedit /etc/modprobe.d/blacklist.conf #确认最后一行添加：blacklist nouveau
sudo update-initramfs -u -k all
sudo reboot
```
3. 重装显卡驱动
