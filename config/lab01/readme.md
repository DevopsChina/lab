# Ansible 新手入门指南

## 安装 Ansible

### 用 pip3 在 macOS 上安装

操作系统版本 macOS 12.3 (21E230) - Apple M1 Max

步骤如下：

1. 先安装 Python3 ，步骤省略
2. 运行  ` pip3 install ansible` 
3. 在 shell 的环境配置文件(例如：~.zshrc)中加入这一行 `export PATH="$PATH:/Users/martinliu/Library/Python/3.8/bin"`
4. 让配置文件生效，运行 `source ~.zshrc`
5. 验证 Ansible 安装的版本，运行 `ansible --version`

### 用 dnf/yum 在 CentOS 8 上安装

步骤如下：

1. 运行命令 `yum install ansible -y`
2. 验证 Ansible 安装的版本，运行 `ansible --version`

```sh
[root@ansible-appserver ~]# cd .pip/
[root@ansible-appserver .pip]# ls
pip.conf
[root@ansible-appserver .pip]# cat pip.conf 
[global]
index = http://192.168.1.173/repository/pypi/
index-url = http://192.168.1.173/repository/pypi/simple
trusted-host = 192.168.1.173
```