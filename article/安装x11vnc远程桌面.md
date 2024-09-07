---
title: 安装x11vnc远程桌面
id: 51f449bc-6f98-4173-9fb0-fb0289dca315
date: 2024-07-29 17:40:34
auther: billy
cover: 
excerpt: 打开终端并输入以下命令以安装x11vnc： sudo apt-get install x11vnc 安装完成后，输入以下命令以生成密码
permalink: /archives/1722246033806
categories:
 - fu-wu-qi
tags: 
---


## 打开终端并输入以下命令以安装x11vnc：

```bash
sudo apt-get install x11vnc
```

安装完成后，输入以下命令以生成密码文件：

```bash
sudo x11vnc -storepasswd /etc/x11vnc.pass
```

该命令将提示您输入密码，然后将密码保存在/etc/x11vnc.pass文件中。

## 创建一个systemd服务文件，以便在系统启动时自动启动x11vnc

```bash
sudo nano /etc/systemd/system/x11vnc.service
```

将以下内容复制并粘贴到文件中：

```bash
[Unit]
Description=Start x11vnc at startup.
After=lightdm.service  # 或 display-manager.service

[Service]
Type=simple
ExecStart=/usr/bin/x11vnc -auth guess -forever -noxdamage -repeat -rfbauth /etc/x11vnc.pass -rfbport 5900 -shared

[Install]
WantedBy=multi-user.target
```

然后按Ctrl + X，输入“Y”以保存并退出nano编辑器。

## 启用并启动x11vnc服务。在终端中输入以下命令：

```bash
sudo systemctl enable x11vnc.service 
sudo systemctl start x11vnc.service
```

重启服务器

现在，x11vnc服务应该已经启动并正在运行。

使用VNC客户端连接到桌面，使用IP地址和端口号5900，并输入先前设置的密码。

# 启动x11vnc失败的解决办法

## 终端启动x11vnc命令

```bash
/usr/bin/x11vnc -auth guess -forever -noxdamage -repeat -rfbauth /etc/x11vnc.pass -rfbport 5900 -shared
```

终端启动x11vnc,并使用vnc客户端查看是否能正常进入系统

### 1.进入系统失败

#### **检查服务状态**：

查看服务状态和日志，以确保服务正确启动：

```bash
sudo systemctl status x11vnc.service
sudo journalctl -u x11vnc.service
```

#### **确保** `/etc/x11vnc.pass` 文件权限正确：

确认密码文件的权限和路径：

```bash
sudo chmod 600 /etc/x11vnc.pass
```

#### **重新加载 systemd 守护进程**：

```bash
sudo systemctl daemon-reload
```

#### **启用并启动 x11vnc 服务**：

```bash
sudo systemctl enable x11vnc.service
sudo systemctl start x11vnc.service
```

### 2.成功登录系统

无其他错误，重新加载 systemd 守护进程并重启 x11vnc 服务即可

#### **重新加载 systemd 守护进程**：

```bash
sudo systemctl daemon-reload
```

#### **启用并启动 x11vnc 服务**：

```bash
sudo systemctl enable x11vnc.service
sudo systemctl start x11vnc.service
```