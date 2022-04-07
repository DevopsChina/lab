# Ansible 新手入门指南

![logo](img/ansible-wide.png)

Ansible 的发音？是什么意思？出处？

![starwar](img/192-1928992_order-66.jpg)

[维基百科](https://en.wikipedia.org/wiki/Ansible_(software))：术语 "ansible "是乌苏拉-K-勒古恩在她1966年的小说《罗卡农的世界》中创造的，它指的是虚构的即时通信系统。是一类虚构的设备或技术，能够进行近乎瞬时或比光速更快的通信。

关于 Ansible 的小故事：“摸鱼创业成功的经典案例！”，看下 Founder, CEO & Chairman - Ansible 的 LinkedIn 你就懂了。

* https://www.linkedin.com/in/saidziouani/
* https://www.linkedin.com/in/timothy-gerla/
* https://www.linkedin.com/in/michaeldehaan/

![then push me back in](img/t8redb90cc631.jpg)

Ansible 简介 文档：

* https://docs.ansible.com/ansible/latest/index.html
* https://en.wikipedia.org/wiki/Ansible_(software)
* https://www.ansible.com/hubfs//AnsibleFest%20ATL%20Slide%20Decks/Getting%20Started%20with%20Ansible%20-%20Jake.pdf
* https://aap2.demoredhat.com/decks/ansible_best_practices.pdf
* https://www.ansible.com/hubfs/Webinar%20PDF%20slides/%5BWIP%5D%20MBU%20_%20ANA%20_%20Webinar%20-%20Ansible%20Network%20Meta%20Collection.pdf
* 

## 1 - 安装 Ansible

### 1.1 用 pip3 在 macOS 上安装

操作系统版本 macOS 12.3 (21E230) - Apple M1 Max

步骤如下：

1. 先安装 Python3 (步骤省略)
2. 确认 Python3 的版本：`python3 --version`
3. 运行  ` pip3 install ansible` 
4. 在 shell 的环境配置文件(例如：~.zshrc)中加入这一行 `export PATH="$PATH:/Users/martinliu/Library/Python/3.8/bin"`
5. 让配置文件生效，运行 `source ~.zshrc`
6. 验证 Ansible 安装的版本，运行 `ansible --version`

### 1.2 用 yum 在 CentOS 8 上安装

步骤如下：

1. 运行命令 `yum install ansible -y`
2. 验证 Ansible 安装的版本，运行 `ansible --version`


## 2 - 环境准备

### 环境说明

控制器 - Contorler ：
* macOS ： localhost
* centOS 8 ：192.168.1.136

被管理的服务器 - Hosts ：

* 本地虚拟机 - cenotOS 8 
  * app1 ： 192.168.1.167 
  * app2 ：192.168.1.239 
  * db ：192.168.1.241 

* Azure 虚拟机 - cenotOS 8 
  * app1 ： 
  * app2 ： 
  * db ： 

### 初始化无 SSH 密钥访问



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