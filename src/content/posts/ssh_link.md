---
title: 配置远程连接
published: 2025-07-16
description: '用ssh来连接服务器和GitHub'
image: ''
tags: [ssh]
category: 'config'
draft: false 
lang: 'zh-CN'
---

# 在服务器上启用ssh

- 下载ssh工具

```bash
sudo apt install remote-server
sudo apt install net-tools
```

- 启用ssh服务
- 使用`ifconfig`查看ip地址
 	- 注意：如果使用VMware使用不同地网络连接的话可能会导致ip地址更改
  		- 桥接模式
  		- NAT模式
 	- 如果不能连接的话请查看是否更改了IP地址
- 使用`/etc/init.d/ssh restart`/`sudo systemctl start ssh/sshd` 启动ssh服务
- 使用`ps -e|grep ssh`/`sudo systemctl status ssh/sshd`查看状态

# 在Windows上使用XShell连接

- 添加用户名和`ip/url`地址

- 输入密码


- 点击连接即可

- 疑似Xshell与服务器建立连接时使用了sha1和md5密钥，这样后面就不用重复输入密码

# 在Linux上连接

- 输入`ssh username@ip address`
- 输入密码即可
- 在**本机**上使用`ssh-keygen -o -a 100 -t ed25519 -f ~/.ssh/id_ed25519`配置密钥
 	- 把`id_ed25519`改为`id_rsa`也行
- 输入`ssh-copy-id -i ~/.ssh/id_ed25519.pub 用户名@服务器地址`连接服务器
 	- 输入密码即可
 	- 或者用`scp`/ssh copy,将本地文件复制到服务器
  		- `scp 本地文件 用户名@服务器地址:对应地址`
  		- eg: `scp ~/.ssh/id_ed25519.pub hzx@199.230.231.21:~`
      - 将密钥信息复制到服务器地根目录下
    - `mv id_ed25519.pub ~/.ssh/authorized_keys`
- 可以用`~/.ssh/config`进行相关配置

```config
Host vm #主机昵称
    User foobar #用户名
    HostName 172.16.174.141 #服务器地址
    Port 22 #连接端口
    IdentityFile ~/.ssh/id_ed25519 #使用的密钥
    LocalForward 9999 localhost:8888
```

- 之后就可以用`ssh vm`来直接连接

# 未解决问题

- 在Ubuntu上不能连接远程服务器，但是在windows上可以，但是好像是在用Xshell之后才可以正常连接？
- 在其他人的电脑不能连接到我本地的服务器——`ping <ip address>`,我使用的是VMware中的Ubuntu系统
- 在修改DNS好像也不行,在使用`cat /etc/resolv.conf`后，看不到修改的记录

```bash
namesever:8.8.8.8
namesever:8.8.4.4
```

- 为什么在VMware中Ubuntu的ip地址会突然更改
  - 从`172.21.97.233`换成`192.168.142.120`

# 疑惑

- 在用Linux进行远程连接时，可以连接VMware的Ubuntu server，但是连接不了Ubuntu desktop的版本，但是Windows可以连接desktop的版本

# 补充

## 如果电脑中有多个密钥应该如何管理

### 检查 SSH 密钥配置(以连接GitHub为例)

确保你本地的 SSH 密钥已添加到 GitHub 账户：

1. **生成 SSH 密钥（如尚未生成）**：

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

（按提示操作，默认保存路径为`~/.ssh/id_ed25519`）
2. **将公钥添加到 GitHub**：
    - 复制公钥内容：

```bash
cat ~/.ssh/id_ed25519.pub
```

- 登录 GitHub → Settings → SSH and GPG keys → New SSH key → 粘贴公钥。

3. **测试 SSH 连接**

验证 GitHub 是否认可你的密钥：

```bash
ssh -T git@github.com
```

成功时会显示：

```bash
Hi username! You've successfully authenticated, but GitHub does not provide shell access.
```

若失败，检查错误信息（如密钥未找到或权限问题）

### 配置多个密钥

- 生成密钥

```bash
ssh-keygen -t ed25519 -C "your_personal_email@example.com"

ssh-keygen -o -a 100 -t ed25519 -f ~/.ssh/id_ed25519
```

- `-t ed25519` 加密方式
- -C "`your_personal_email@example.com`" 注释
- -f 使用bcrypt加密密钥
前者更方便，后者更安全

输入之后，可以输入密钥名称，可以自动生成私钥和公钥(.pub)
如果不输入，会自动生成id_ed25519，在生成多个密钥时，可能会覆盖掉之前生成的密钥

eg：
在连接github时，如果出现

```bash
git@github.com: Permission denied (publickey).
```

可以尝试配置ssh

新建config文件

```config
# 示例配置
Host github.com
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519  # 替换为你的私钥路径
  IdentitiesOnly yes
```

输入对应的配置即可
