---
title: 在CentOS7下构建TeamSpeak服务器并增加网易云点歌插件
date: 2024-05-30 04:06:09
category:
  - [计算机科学, 杂项]
tags:
  - Linux
  - TeamSpeak
  - NetEaseMusic
---

:::info
本人服务器配置为CentOS 7.9 64位。
:::

:::warning

部分内容有时效性，不能保证可以复现。

:::

首先ssh登录到我们的服务器。

## 部署TeamSpeak

### 创建一个新用户

这是为了方便管理，而且部分操作不方便通过root账号完成。

这里我们创建一个名为teamspeak的账户：

```bash XShell command:("[root@Starling] $":2,4,6,8||"[teamspeak@Starling] $":9)
# 新建用户teamspeak
useradd teamspeak
# 给予sudo权限
usermod -a -G sudo teamspeak
# 设置密码
passwd teamspeak
# 切换用户并定位到家目录
su teamspeak
cd ~
```

### 下载并解压服务端

#### 下载

{% links %}
- site: Teamspeak服务端下载
  url: https://www.teamspeak.com/zh-CN/downloads/#server
  {% endlinks %}

以上为官网下载地址，可以在那里查看最新版本，可以选择从官网下载下来再传输到服务器上，也可以直接命令行下载：

```bash XShell command:("[teamspeak@Starling] $":2)
# 需要服务器已安装wget
wget https://files.teamspeak-services.com/releases/server/3.13.7/teamspeak3-server_linux_amd64-3.13.7.tar.bz2
```

+++primary 如果你的服务器没有安装wget

注意使用root账号。

```bash XShell command:("[root@Starling] $":2)
# 使用yum安装wget
sudo yum install -y wget
```

+++

#### 解压

```bash XShell command:("[teamspeak@Starling] $":2)
# 使用tar命令解压
tar -xvf teamspeak3-server_linux_amd64-3.13.7.tar.bz2
```

+++primary 如果解压报错Error is not recoverable: exiting now

需要安装bz2相关内容，使用root账号：

```bash XShell command:("[root@Starling] $":2)
# 使用yum安装bzip2
sudo yum install -y bzip2
```

+++

创建teamspeak目录，把解压后的东西塞进去：

```bash XShell command:("[teamspeak@Starling] $":1-4)
mkdir teamspeak
mv  teamspeak3-server_linux_amd64 teamspeak-rf 
rm -rf teamspeak3-server_linux_amd64
cd teamspeak
```

### 启动服务端

#### 同意许可协议

```bash XShell command:("[teamspeak@Starling] $":2)
# 没什么好说的，就是同意许可协议
touch .ts3server_license_accepted
```

#### 启动与配置

```bash XShell command:("[teamspeak@Starling] $":2)
# 启动服务端
./ts3server_startscript.sh start
```

==第一次启动，会显示包括token在内的重要的信息，记得保存！！！==

然后`Ctrl`+`C`退出。

![记得保存](1.png "")

#### 开放端口

==我们需要为服务器开放一些端口，否则无法与客户端进行通讯。==

以下是端口列表，不仅仅要在服务器开放，还要在云服务提供商的控制台的安全组中开放。

| 端口  | 协议 | 说明                                          |
| ----- | ---- | --------------------------------------------- |
| 9987  | UDP  | TeamSpeak语音服务端口                         |
| 10011 | TCP  | TeamSpeak ServerQuery raw 端口                |
| 10022 | TCP  | TeamSpeak ServerQuery SSH 端口                |
| 30033 | TCP  | TeamSpeak 文件传输端口                        |
| 41144 | TCP  | TSDNS 服务器端口                              |
| 58913 | TCP  | 机器人网站后台端口（不开无所谓）				|
| 3000  | TCP  | 网易云api后台端口（后面要用，先放开这个端口） |

在服务器开放端口：

```bash XShell command:("[root@Starling] $":1-6)
firewall-cmd --zone=public --add-port=9987/udp --permanent && firewall-cmd --reloa
firewall-cmd --zone=public --add-port=10011/tcp --permanent && firewall-cmd --reloa
firewall-cmd --zone=public --add-port=10022/tcp --permanent && firewall-cmd --reloa
firewall-cmd --zone=public --add-port=30033/tcp --permanent && firewall-cmd --reloa
firewall-cmd --zone=public --add-port=41144/tcp --permanent && firewall-cmd --reloa
firewall-cmd --zone=public --add-port=3000/tcp --permanent && firewall-cmd --reloa
```

在安全组开放端口：

![安全组](2.png "")

#### 设置开机自启

注意使用root账号。

```bash XShell command:("[root@Starling] $":2)
# 新建teamspeak.service
vim /etc/systemd/system/teamspeak.service
```

然后`i`进入INSERT模式，写文件：

```
[Unit]
Description=Teamspeak Service
Wants=network.target

[Service]
WorkingDirectory=/home/teamspeak
ExecStart=/home/teamspeak/teamspeak/ts3server_minimal_runscript.sh
ExecStop=/home/teamspeak/teamspeak/ts3server_startscript.sh stop
ExecReload=/home/teamspeak/teamspeak/ts3server_startscript.sh restart
Restart=always
RestartSec=15
 
[Install]
WantedBy=multi-user.target
```

然后`Esc`，`:wq!`强制保存并退出。

```bash XShell command:("[root@Starling] $":2,5,8,11)
# 更新配置
systemctldaemon-reload

# 设置开机启动
systemctl enable teamspeak.service

# 启动服务
systemctl start teamspeak.service

# 查看状态，如果有active (running)说明成功了
systemctl status teamspeak.service
```

### 客户端连接

这个时候就可以去官网设置服务器别名了，或者也可以直接裸连服务器地址。

连接后使用上面保存的token设置自己为超级管理员。

随后自定义各种设置。

## 部署TS3AudioBot并添加网易云插件

### 安装ffmpeg

```bash XShell command:("[root@Starling] $":1-3)
yum -y install epel-release
rpm -Uvh http://li.nux.ro/download/nux/dextop/el7/x86_64/nux-dextop-release-0-5.el7.nux.noarch.rpm
yum -y install ffmpeg opus-devel
```

### 下载TS3AudioBot本体与插件并解压

==不要下载官方的，因为官方给的linux版编译环境有问题导致后面不能正常加载插件==

下载[ZHANGTIAOYAO1重构后的版本](https://github.com/ZHANGTIANYAO1/TS3AudioBot-NetEaseCloudmusic-plugin/releases/download/1.1.0/with.TS3Bot.linux-x64.zip)与[FiveHair修改后的网易云插件](https://github.com/FiveHair/TS3AudioBot-NetEaseCloudmusic-plugin-UNM)。

命令行下载：

```bash XShell command:("[root@Starling] $":2||"[teamspeak@Starling] $":3-5)
# 切换teamspeak账号
su teamspeak
cd ~
wget https://github.com/ZHANGTIANYAO1/TS3AudioBot-NetEaseCloudmusic-plugin/releases/download/1.1.0/with.TS3Bot.linux-x64.zip
wget https://github.com/FiveHair/TS3AudioBot-NetEaseCloudmusic-plugin-UNM/releases/download/2.0.3.1/YunPlugin-UNM-2.0.3.1.zip
```

解压本体与插件并拷贝插件至本体的plugins目录：

```bash XShell command:("[teamspeak@Starling] $":1-8)
unzip with.TS3Bot.linux-x64.zip
mv linux-x64 TS3AudioBot
unzip YunPlugin-UNM-2.0.3.1.zip
cd TS3AudioBot/plugins
rm *
cd ~
mv YunPlugin-UNM.dll TS3AudioBot/plugins/
mv YunSettings.ini TS3AudioBot/plugins/
```

### 配置TS3AudioBot启动

```bash XShell command:("[teamspeak@Starling] $":1-3)
cd TS3AudioBot
chmod 755 TS3AudioBot
./TS3AudioBot
```

随后根据引导设置管理员的uid（在客户端上面“权限”菜单中打开“所有客户端列表”可以看到服务器所有人的uid）、服务器ip、密码等。

然后直接`Ctrl`+`C`结束进程。

### 设置开机自启

注意切换root账号：

```bash XShell command:("[root@Starling] $":2)
# ts3audiobot.service
vim /etc/systemd/system/ts3audiobot.service
```

然后`i`进入INSERT模式，写文件：

```
[Unit]
Description=TS3AudioBot
After=teamspeak.service

[Service]
WorkingDirectory=/home/teamspeak/TS3AudioBot/
ExecStart=/home/teamspeak/TS3AudioBot/TS3AudioBot
Restart=always
RestartSec=15

[Install]
WantedBy=multi-user.target
```

然后`Esc`，`:wq!`强制保存并退出。

```bash XShell command:("[root@Starling] $":2,5,8,11)
# 更新配置
systemctldaemon-reload

# 设置开机启动
systemctl enable ts3audiobot.service

# 启动服务
systemctl start ts3audiobot.service

# 查看状态，如果有active (running)说明成功了
systemctl status ts3audiobot.service
```

## 部署网易云API

### 安装git

```bash XShell command:("[root@Starling] $":1,3,5)
sudo yum install -y git
# 设置用户名
git config --global user.name "名字"
# 设置邮箱
git config --global user.email "邮箱"
```

### 安装Node.js和npm

```bash XShell command:("[root@Starling] $":1-2)
sudo yum install -y nodejs
sudo yum install -y npm
```

### 搭建API

注意还是切换到teamspeak账号：

```bash XShell command:("[root@Starling] $":1||"[teamspeak@Starling] $":2-3)
su teamspeak
cd ~
sudo git clone git@gitlab.com:Binaryify/NeteaseCloudMusicApi.git
```

==由于网易云的赶尽杀绝，大部分API都寄了，而这个其实也寄了。但是虽然github上改仓库删除归档了，我们依旧能在gitlab找到全部源码。==

```bash XShell command:("[teamspeak@Starling] $":1-3)
cd NeteaseCloudMusicApi
sudo npm install
sudo node app.js
```

此时显示`server running ...`即为配置成功，按`Ctrl`+`C`，退出。

### 设置开机自启

注意切换root用户：

```bash XShell command:("[root@Starling] $":2)
# netease.service
vim /etc/systemd/system/netease.service
```

然后`i`进入INSERT模式，写文件：

```
[Unit]
Description=Netease Cloud Music API Service
After=network.target

[Service]
WorkingDirectory=/home/teamspeak/NeteaseCloudMusicApi/
ExecStart=/usr/bin/node /home/teamspeak/NeteaseCloudMusicApi/app.js
Restart=always
RestartSec=15

[Install]
WantedBy=multi-user.target
```

然后`Esc`，`:wq!`强制保存并退出。

```bash XShell command:("[root@Starling] $":2,5,8,11)
# 更新配置
systemctldaemon-reload

# 设置开机启动
systemctl enable netease.service

# 启动服务
systemctl start netease.service

# 查看状态，如果有active (running)说明成功了
systemctl status netease.service
```

## 将插件与API连接


注意还是切回teamspeak账号：

```bash XShell command:("[teamspeak@Starling] $":1-3)
cd ~
cd TS3AudioBot/plugins
vim YunSettings.ini
```

将链接修改为本地：

```
# 如果原本等号后面有东西就删掉
WangYiYunAPI_Address = http://localhost:3000
```

## 设置机器人指令权限

设置谁能操控机器人谁能点歌，在`/home/teamspeak/TS3AudioBot`目录下的`rights.toml`中有详细的说明与设置方法。

## 启动机器人

之后的操作仅在客户端就能完成。

聊天框输入`!plugin list`，找到网易云插件（状态应该是RDY就绪）的编号（假设是#0）。

输入`!plugin load 0`（后面数字得看编号）加载插件。

然后再查看插件列表，网易云插件的状态应该为（+ON运行）

之后输入`!yun login`后根据机器人的头像与简洁设置登录网易云（请给机器人管理员权限）。

大功告成！

![测试](3.png)
