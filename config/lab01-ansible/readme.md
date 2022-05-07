# Ansible 新手入门指南

![ansible-wide](https://user-images.githubusercontent.com/582423/166117795-a7c0d761-f9f5-40e0-9d85-7bfb9af404b0.png)

[【查看 B 站视频】](https://www.bilibili.com/video/BV1Uv4y1K7u9)

## Ansible 的发音？

![Starwar](https://user-images.githubusercontent.com/582423/166117837-19f53d31-98e4-471b-9a31-3c264fcb79b4.jpg)


## Ansible 究竟是何物？

[维基百科](https://en.wikipedia.org/wiki/Ansible_(software))：术语 "ansible "是乌苏拉-K-勒古恩在她1966年的小说《罗卡农的世界》中创造的，它指的是虚构的即时通信系统。是一类虚构的设备或技术，能够进行近乎瞬时或比光速更快的通信。

## 创始人小传

关于 Ansible 的小故事：“摸鱼创业成功的经典案例！”，看下 Founder, CEO & Chairman - Ansible 的 LinkedIn 你就懂了。

* https://www.linkedin.com/in/saidziouani/
* https://www.linkedin.com/in/timothy-gerla/
* https://www.linkedin.com/in/michaeldehaan/


![then push me back in](https://user-images.githubusercontent.com/582423/166117869-ae72ad01-5977-4557-a147-496a09389409.jpg)


## 推荐学习文档

Ansible 简介文档：

* https://docs.ansible.com/ansible/latest/index.html
* https://en.wikipedia.org/wiki/Ansible_(software)
* https://www.ansible.com/hubfs//AnsibleFest%20ATL%20Slide%20Decks/Getting%20Started%20with%20Ansible%20-%20Jake.pdf
* https://aap2.demoredhat.com/decks/ansible_best_practices.pdf
* https://www.ansible.com/hubfs/Webinar%20PDF%20slides/%5BWIP%5D%20MBU%20_%20ANA%20_%20Webinar%20-%20Ansible%20Network%20Meta%20Collection.pdf
  
Ansible 和其他几种同类工具的对比：

![工具的对比](https://user-images.githubusercontent.com/582423/166117891-d0d288f6-ada3-467b-ab06-c22992b5bff3.png)

Ansible 最简化架构图：

![架构图](https://user-images.githubusercontent.com/582423/166117928-55190fb0-0105-4cfb-95ed-027ed54a7539.png)


## 安装 Ansible

本教程推荐使用 `pip` 在 Fedora 35 上安装 Ansible。

### 环境说明

推荐使用 4 个虚拟机的学习环境，本教程模拟的是上面的架构图中的效果。

控制器 - Contorler 【Master】：

* os : Fedora 35 
* ip ：192.168.31.30

被管理服务器 - Hosts 【node】

* 本地 Fedora 35 虚拟机 
* app1 ： 192.168.31.165 
* app2 ：192.168.31.124
* db ：192.168.31.58

以上是建议的最佳学习环境配置。而最低配置可以是：本地笔记本电脑（Win/macOS） + 一个虚拟机(Linux)所组成的开发环境。

在本地安装开发环境：以操作系统版本 macOS 12.3 (21E230) 的安装配置过程为例，其中第四个步骤需要按实际情况修改路径。

步骤：

1. 先安装 Python3 (步骤省略)
2. 确认 Python3 的版本：`python3 --version`
3. 运行  ` pip3 install ansible` 
4. 在 shell 的环境配置文件加入 Python 的可执行文件路径。例如：使用 zsh 的系统在 ~.zshrc 文件中加入这一行 `export PATH="$PATH:/Users/martinliu/Library/Python/3.8/bin"`，不同的用户名和 Python 版本对应这里的路径不同。
5. 让配置文件生效，运行 `source ~.zshrc` 
6. 验证 Ansible 安装的版本，运行 `ansible --version`

如果你使用的是 Windows 操作系统，请使用 Windows 10/11 的版本自带的 Windows Subsystem for Linux 功能，在 Linux 的子系统里完成以上的操作。

### 在 Fedora 35 上安装 【生产环境】

本教程推荐用` pip` 安装 Ansible 的方式。同时下面也提供了用 `dnf` 的安装方法。 

#### pip 安装

下面，我们在 Master （controler）上安装部署 Ansble 工具集。

为了安装到 Ansible 的最新版本，还需要先升级 python 的版本到 >= 3.8；目前 Fedora 35 自带的Python 版本为 3.10.1，已经满足了 Ansible 对 Python 的最低版本需求。

步骤如下：

1. 安装必要的软件包，运行命令 `dnf install python3-pip  sshpass git -y`
2. 升级 `pip3` 用  `python3 -m pip install --upgrade pip`
3. 切换到非 root 用户 （如果有的话  ）
4. 先设置国内的 pypi 安装源 `pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple`
5. 用 `pip3` 安装 Ansible ， 运行 `pip3 install ansible`
6. 验证 Ansible 安装的版本，运行 `ansible --version`

#### dnf 安装 

在控制器 （master）上安装部署 Ansble 工具集。

步骤如下：

1. 运行命令 `dnf install ansible -y`
2. 验证 Ansible 安装的版本，运行 `ansible --version`
3. 安装必要的软件包 `dnf install sshpass git -y`

dnf 安装到的版本会稍微老一些，没有 pip 安装的版本新。

而在生产中，往往需要安装特定的某个 Ansible 版本，而非当前的最新版本。

## 配置 Ansble 运行环境

本教程演示和讲解的是将 4 个空白无任何配置的服务器配置为：单一控制器，三个被管理节点的运行环境。

### 初始化控制器 

#### 创建 SSH 无密码访问秘钥对

Ansible 的控制器（Master）通过 SSH 访问和管理被管理的节点，并完成系统、服务配置工作。

首先，在控制器上生成 ssh 访问的秘钥对。ssh 登陆到控制器，最好切换到非root用户，执行 `ssh-keygen` 命令，如果不需要秘钥文件的密码，则需要输入一系列回车即可；新创建 ssh 的密钥对，用于无密码访问其它三个被服务器节点。

创建的测试用秘钥对，查看所在位置：

```sh
[martin@ctl ~]$ ssh-keygen
[martin@ctl ~]$ ls ~/.ssh
id_rsa  id_rsa.pub
```

尝试使用密码的访问其它被管理服务器，确认你拥有所有正确的密码。（如果你已经有一个统一的访问秘钥，请忽略此步骤）

#### 配置 ansible.cfg 基础配置文件

Ansible 控制器节点的执行引擎的行为特性的配置文件是 `ansible.cfg` 文件，对此文件路径的搜索顺序如下：

1. ANSIBLE_CONFIG (环境变量中)
2. ansible.cfg (当前目录中) 。
3. ~/.ansible.cfg (当前用户的 home 目录下)
4. /etc/ansible/ansible.cfg （操作系统的路径）- 本教程中使用的方式

在当前用户的` Home `目录中创建一个内容如下的 `.ansible.cfg` 配置文件 (注意是以句点开头的隐藏文件)

```yml
[defaults]
host_key_checking = false
inventory  = ./hosts.ini
command_warnings = False
deprecation_warnings = False
roles_path = ./roles
nocows = 1
retry_files_enabled = False

[ssh_connection]
control_path = %(directory)s/%%h-%%p-%%r
pipelining = True
```

#### 编写主机清单文件 - Inventory

主机清单文件就是所谓的 “Inventory”，它是描述所有被管理节点的数据文件。

创建内容如下的  `inventory.v1` 配置文件

```yml
# 组织方式：功能、地域、环境
# 应用服务器组
[app]
192.168.31.165 
192.168.31.124

# 数据库服务器组
[db]
192.168.31.58

# 名为 localvm 的嵌套组
[localvm:children]
app
db

# 给嵌套组定义变量，应用于所有服务器
[localvm:vars]
ansible_user=root
ansible_password='devops1234'

```

其他可用的定义方法详见：https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html 


执行首个 Ansible 命令 `ansible -i inventory.v1 all -m ping` ，如果能正常使用 inventory 文件中的用户名和密码登录所有被管理节点的话，改命令的结果应该如下：

```sh
[martin@ctl live]$ ansible -i inventory.v1 all -m ping
192.168.31.58 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
192.168.31.165 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
192.168.31.124 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```

#### 创建 Ansible 专用的用户

在拥有所有节点 root 访问的基础上，在所有被管理节点上进一步配置一个 Ansible 专用的账号。

inventory.v1 配置文件中携带密码是很危险的，下面使用 Ansible 来解决这个问题，并配置好所有的被管理节点。

编写 Playbook 实现下面的工作：

* 创建一个 Ansible 专用的名为 `sysops` 的用户账号 （不建议使用 root 用户，而是非 root 用户）
* 将其配置为 sudo 提权限并不需要密码的用户。
* 将控制器当前用户新创建的秘钥对的公钥配置部署到所有被管理节点的 sysops 的授权用户中。


创建第一个 Playbook `init-users.yml` 。 

* 使用到的Ansible模块有 group, lineinfile, user 和 authorized_key 
* 引用了变量文件

文件的内容如下：  

```yml
---
- hosts: localvm
  become: true
  # 引用变量文件
  vars_files:
    - vars/default.yml
  tasks:
  # Sudo 用户组配置
    - name: Make sure we have a 'wheel' group
      group:
        name: wheel
        state: present
  # 允许 'wheel' 组里的用户执行sudo可以不输入用户密码
    - name: Set sudo without password
      lineinfile:
          path: /etc/sudoers
          state: present
          regexp: '^%wheel'
          line: '%wheel ALL=(ALL) NOPASSWD: ALL'
          validate: '/usr/sbin/visudo -cf %s'

  # 创建远程命令执行的用户，并配置ssh密钥
    - name: Create a new regular user with sudo privileges
      user:
        name: "{{ create_user }}"
        state: present
        groups: wheel
        append: true
        create_home: true
        shell: /bin/bash

    - name: Set authorized key for remote user
      authorized_key:
        user: "{{ create_user }}"
        state: present
        key: "{{ copy_local_key }}"
```

这个文件引用了一个变量文件 `vars/default.yml` ，下面创建这个目录和文件，文件的内容如下：

```yml
create_user: sysops
copy_local_key: "{{ lookup('file', lookup('env','HOME') + '/.ssh/id_rsa.pub') }}"
```

下一步执行这个 Playbook 完成被管理节点的初始化配置 ：`ansible-playbook -i inventory.v1 init-users.yml` ，结果如下：

```sh
[martin@ctl live]$ ansible-playbook -i inventory.v1 init-users.yml

PLAY [localvm] *********************************************************************************

TASK [Gathering Facts] *************************************************************************
ok: [192.168.31.124]
ok: [192.168.31.165]
ok: [192.168.31.58]

TASK [Make sure we have a 'wheel' group] *******************************************************
ok: [192.168.31.165]
ok: [192.168.31.58]
ok: [192.168.31.124]

TASK [允许 'wheel' 组里的用户执行sudo可以不输入用户密码] ***************************************
changed: [192.168.31.58]
changed: [192.168.31.124]
changed: [192.168.31.165]

TASK [Create a new regular user with sudo privileges] ******************************************
changed: [192.168.31.165]
changed: [192.168.31.124]
changed: [192.168.31.58]

TASK [Set authorized key for remote user] ******************************************************
changed: [192.168.31.124]
changed: [192.168.31.58]
changed: [192.168.31.165]

PLAY RECAP *************************************************************************************
192.168.31.124             : ok=5    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
192.168.31.165             : ok=5    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
192.168.31.58              : ok=5    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

`ansible-playbook` 命令是解析和执行 Playbook 脚本文件的命令。我们可以看到 Playbook 是一个 yml 格式的状态声明式定义文件。它描述了被管理节点的操作系统中的各种配置细节。

在控制器上验证无密码SSH密钥认证登陆，选取任何一个被管理节点的 ip 地址，执行 `ssh sysops@192.168.31.124`；登录后查看 .ssh 目录中的内容。

在所有host上我们配置好了一个Ansible专用的无sudo密码的普通用户。优化 inventory.v1 文件，删除其中的用户名和密码，创建内容如下的 inventory.v2 文件：

```yml
# 组织方式：功能、地域、环境
# 应用服务器组
[app]
192.168.31.165
192.168.31.124

# 数据库服务器组
[db]
192.168.31.58

# 名为 localvm 的嵌套组
[localvm:children]
app
db

# 给嵌套组定义变量，应用于所有服务器
[localvm:vars]
ansible_user=sysops
ansible_ssh_private_key_file=~/.ssh/id_rsa
ansible_ssh_common_args='-o StrictHostKeyChecking=no'
```

最后用首个执行的 Ansible 命令 `ansible -i inventory.v2 all -m ping` 验证所有主机是否可以用SSH正常访问，确认得到全绿的输出结果。

```sh
[martin@ctl live]$ ansible -i inventory.v1 all -m ping
192.168.31.58 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
192.168.31.124 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
192.168.31.165 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```

到此为止，我们完成了 Ansible 运行环境的配置，包括：

* 准备 ssh 登录的秘钥 
* 配置控制器节点的基础配置文件
* 初始化三个被管理几点的远程登录用户账号
* 创建了优化的 Inventory 主机清单文件
* 使用 ping 模块再次确认所有被管理主机的 ssh 登录

## 用 Ansible 命令执行运维工作 ｜ Ad-hoc 

用下面的命令体会 Ansible 的特性和内置模块的功能。在执行下面的命令之前，复制 inventory.v2 文件为 hosts.ini 文件【ansible.cnf 参数中已做配置】。这样在执行命令的时候就不需要用参数制定 inventory 文件的位置，因此可以更加简洁。

### 默认并发执行特性

运行多次下面相同的命令，了解 Ansible 的多线程并发特性。

* 使用 -a 参数远程执行 Linux 原生命令

```sh
ansible localvm -a "hostname"
ansible localvm -a "hostname"
ansible localvm -a "hostname"

ansible localvm -a "hostname" -f 1
ansible localvm -a "hostname" -f 1
ansible localvm -a "hostname" -f 1
```

从以上结果中观察 node 节点的执行顺序。

### 环境状况检查

使用 -a 参数的远操作系统里的原生命令。

```sh
ansible localvm -a "df -h"

ansible localvm -a "free -m"

ansible localvm -a "date"
```

### 使用 Ansible 模块做变更

使用 Ansible 的核心模块变更系统。

* yum 和 servce 模块混用
* 搭配 -a 的命令行执行
* 在需要提权的地方，使用 -b 参数

```sh
ansible localvm -a "date"

ansible localvm -b -m yum -a "name=chrony state=present"

ansible localvm -b -m service -a "name=chronyd state=started enabled=yes"

ansible localvm -b -a "chronyc tracking"

ansible localvm -a "date"
```

## 部署目标应用系统

本教程在 GitHub 上准备了一个简单的 hellodjango 应用程序。

应用系统配置部署所需的流程如下：

* 在 App1 和 App2 上配置好 Django 运行环境
* 在 DB 上安装 Mariadb
* 在所有服务器上启动防火墙服务，并配置好需要开放的端口
* 在两个 App 服务器上安装并启动 Django 应用

### 配置 Django 运行环境

开始编写第一个应用系统在一套服务器上的综合配置 Playbook。

配置应用服务器：安装 python3 和 django

* yum 和 pip 模块的混用
* 辅助 -a 的命令行执行

```sh
ansible app -b -m yum -a "name=python3-pip state=present"

ansible app -b -m pip -a "name=django<4 state=present"

ansible app -a "python3 -m django --version"

```

本教程中使用了先用逐条 Ansible 配置指令调试的方式，然后在将其翻译到 Playbook 中的方式，在熟悉了这个开发过程和 Ansible 的常用模块后，我们则可以直接开发 Playbook 了。

根据以上的所有 Ansible 指令，创建名为 app-stack.yml 的文件，内容如下：

```yml
---
- hosts: app
  become: true
  tasks:
    - name: Config NTP Server  # 配置 ntp 服务器
      yum:
        name: chrony
        state: present
    - name: Start NTP service # 启动 chronyd 服务
      service:
        name: chronyd
        state: started
        enabled: yes
    - name: Install Python3-pip&git 
      yum:
        name: python3-pip,git
        state: present
    - name: Install django package
      pip:
        name: django<4
        state: present
```

下面执行 ansible-play app-stack.yml 可以得到相同的结果状态。

### 配置数据库服务器

接着我们执行安装 Mariadb 服务器的操作。

* yum、service 和 防火墙模块混用

逐条执行下面的调试命令：

```sh
ansible db -b -m yum -a "name=mariadb-server state=present"

ansible db -b -m service -a "name=mariadb state=started enabled=yes"

ansible db -b -m yum -a "name=firewalld state=present"

ansible db -b -m service -a "name=firewalld state=started enabled=yes"

ansible db -b -m firewalld -a "port=3306/tcp zone=public state=enabled permanent=yes"

ansible db -b -m yum -a "name=python3-PyMySQL state=present"

```

firewall-cmd --list-all


在以上命令的结果都符合预期后，在 app-stack.yml 中增加如下内容

```yml

- hosts: db
  become: true
  tasks:
    - name: Install Mariadb Server
      yum:
        name: mariadb-server,python3-PyMySQL
        state: present
    - name: Start DB service
      service:
        name: mariadb
        state: started
        enabled: yes
    - name: Install firewalld
      yum:
        name: firewalld
        state: present
    - name: Start Firewalld service
      service:
        name: firewalld
        state: started
        enabled: yes
    - name: Open the db port
      firewalld:
        port: 3306/tcp
        permanent: yes
        state: enabled
```

在控制器上执行更新后的 Playbook，运行命令 `ansible-playbook app-stack.yml` ，确保执行结果输出正常。

### 部署 Django 应用系统

从 GitHub 中部署目标应用代码：

* 运用 git ，yum ， serviice 和 firewalld 模块
* 用命令行方式启动应用

```sh
ansible localvm -b -m yum -a "name=git state=present"

ansible app -b -m git -a "repo=https://github.com/martinliu/hellodjango.git \
dest=/opt/hello update=yes"

ansible app -b -m firewalld -a "port=8000/tcp state=enabled permanent=yes"

ansible app -b -m service -a "name=firewalld state=reloaded enabled=yes"

ansible app -b -a "sh /opt/hello/run-hello.sh"
```

在浏览器中输入应用的访问网址： http://192.168.31.165:8000/hello/ ，应该在两个 app 服务器上都可以看到相同的结果。


在 app-satck.yml 中的 App 部署部分加入下面的内容

```yml
    - name: Deploy from github
      git:
        repo: https://github.com/martinliu/hellodjango.git
        dest: /opt/hello
        update: yes
    - name: enable app 8000 port
      firewalld:
        port: 8000/tcp
        permanent: yes
        state: enabled
    - name: Reload Firewalld service
      service:
        name: firewalld
        state: reloaded
        enabled: yes
    - name: Start my hello app
      command: sh /opt/hello/run-hello.sh
```

执行最后的应用部署 `ansible-playbook app-stack.yml` 确认执行后的结果正常，用浏览器可以正常访问 Django 应用页面。

## 总结

希望大家通过本教程学会：

* Ansible 运行环境的基础配置
* Ansible 的 ad-hoc 方式用法
* Ansible 核心中的几个常用模块
* 逐步编写部署目标应用的 Ansible Playbook 

## 呼唤下一位鉴宝人

建议分享的内容如下（不限于此）：

* Role 的开发
* CI/CD 工具中执行 Playbook
* 容器，k8s 相关主题
* AWX Ansible 相关主题
* 生产项目中的经验分享 
