# Jenkins 快速入门和使用技巧

<image src="https://www.jenkins.io/images/logos/jenkins/Jenkins.svg">


## 1. Jenkins 介绍

```markdown 
# 构建伟大，无所不能。

 Jenkins 是开源 CI&CD 软件领导者， 提供超过1000个插件来支持构建、部署、自动化，满足任何项目的需要。

> 出自 [Jenkins 官网](https://www.jenkins.io/zh/)

 ```

 ### 1.1 历史考古

1. 大约在 2001 年，ThoughtWorks 开源了 CruiseControl（后面出了商业版本叫 Cruise，再后来改名为 GoCD）。
1. 2004 年夏天由 Sun 公司开发的 Hudson ，在 2005 年 2 月开源并发布第一个版本。
1. 大约在 2007 年，Hudson 被称为 Cruise Control 和其他开源构建服务器的更好替代品。
1. 2008 年 5 月的 JavaOne 大会上，Hudson 获得了开发解决方案类的 Duke's Choice 奖项。
1. 大约在 2010 年，甲骨文声称拥有 Hudson 的商标的权利，所以在 2011 年 1 月通过社区投票改名为 Jenkins (事件起因是因为 2009 年 6月 Oracle 收购 Sun)。
1. 2011 年 2 月 2 日，Jenkins 的第一个版本 1.396 版可供公众使用。
1. 2015 年 5 月，Jenkins 发布了基于 Java 7 的 1.612 版本。
1. 2016 年 4 月，Jenkins 发布了 2.0 版本。
1. 2017 年 4 月，Jenkins 发布了基于 Java 8 的 2.54 版本。
1. 2019 年 2 月，Jenkins 发布了基于 Java 8 和 Java 11的 2.164 版本。
1. 2022 年 5 月，截至目前最新版本 Jenkins 2.347 Weekly 和 Jenkins 2.332.3 LTS

### 1.2 资料链接

* https://martinfowler.com/articles/continuousIntegration.html
* http://cruisecontrol.sourceforge.net/download.html
* https://en.wikipedia.org/wiki/Jenkins_(software)
* https://www.jenkins.io/blog/2012/02/02/happy-birthday-jenkins/
* https://www.jenkins.io/2.0/
* https://github.com/cloudbees/groovy-cps/
* https://www.jenkins.io/download/

### 1.3 相关工具对比

* https://en.wikipedia.org/wiki/Comparison_of_continuous_integration_software
* https://github.blog/2017-11-07-github-welcomes-all-ci-tools/


### 1.4 推荐学习文档

* https://www.jenkins.io/doc/book/pipeline/
* https://www.bilibili.com/video/BV1fp4y1r7Dd
* https://github.com/cloudbees/groovy-cps/blob/master/doc/cps-model.md
* https://github.com/cloudbees/groovy-cps/blob/master/doc/cps-basics.md

## 2. 环境安装

本教程推荐使用 `docker` 在 Rocky Linux 8上安装 Jenkins 。

> 备注：任何安装有 `docker` 工具的环境都可以。

测试环境服务器信息：

* 本地 Rocky Linux 8 虚拟机，配置 4C16G 40G(sys) 100G(data)
* IP：192.168.2.99

### 2.1 搭建原理

借助 docker 启动 Jenkins 简化服务部署，通过 Swarm 实现 Agent 节点的自动注册。其中 **Swarm** 是一个 **Jenkins** 插件，它允许节点加入附近的 **Jenkins**，从而形成一个特别的集群。通过这个插件我们可以自动向 **Jenkins** 添加 **Agent** 节点，而不用先在 **Jenkins** 上手动创建节点和注册，这个是一个不错的想法。

### 2.2 初始环境

在执行下面的步骤之前需要先完成节点的初始化，比如：关闭防火墙，性能调优配置(sysctl/ulimit)等

1. 安装 docker/docker-compose 工具

运行下面的命令：
```bash
yum install -y yum-utils
yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo

# 注意命令 docker-compose 已经作为 docker 插件，使用时需要将 docker-compose 改成 docker compose
yum install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin

mkdir -p /etc/docker
# 配置 docker ，让数据放在数据盘目录，比如：/data
cat << EOF > /etc/docker/daemon.json
{
  "bip":"192.168.101.1/24",
  "data-root":"/data/var/lib/docker/",
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"

  },
  "exec-opts": ["native.cgroupdriver=systemd"],
  "default-ulimits": {
    "nofile": {
      "Name": "nofile",
      "Hard": 64000,
      "Soft": 64000
    }
  },
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ],
  "registry-mirrors": ["https://docker.mirrors.ustc.edu.cn"]
}
EOF
# 注意：
# 1. 在EOF后面输入一个回车键，并且里面的网段可以自己指定，切记不好和主机所在网段冲突。
# 2. 需要将本文档同级目录下文件 sysctl.conf 的内容放到本次演示环境的 /etc/sysctl.conf 中，然后执行 sysctl -p 让其配置生效。

systemctl stop firewalld
systemctl disable firewalld

systemctl restart docker
systemctl status docker
systemctl enable docker
```
上面的命令执行完成，正常情况会出现如下图：

![](./images/systemctl-status-docker.png)

> 注：其他发行版本的搭建方式请参见：https://docs.docker.com/engine/install/

### 2.3 启动服务

1. 创建目录、启动导入文件、启动服务、观察日志

```bash
alias docker-compose='docker compose'
# 创建系统目录和数据目录
mkdir -p /data/opsbox-dev/{system,data}/jenkins/
# 设置数据目录权限，因为会将容器数据映射到主机
chown -R 1000:1000 /data/opsbox-dev/data/jenkins

cd /data/opsbox-dev/system/jenkins
cat << EOF > docker-compose.yml
version: '3'
services:
  jenkins-master:
    container_name: jenkins
    image: registry.jihulab.com/opsbox-dev/oes-jenkins-plus:jenkins-v0.1.1-2.319.1-SNAPSHOT
    restart: unless-stopped
    ports:
      - 30080:8080
      - 50000:50000
    volumes:
      - /data/opsbox-dev/data/jenkins:/var/jenkins_home
    environment:
      JAVA_OPTS: >-
        -server
        -Xmx2g -Xms1g
        -XX:+UnlockExperimentalVMOptions -XX:+UseContainerSupport
        -XX:+UseParallelGC -XX:ParallelGCThreads=20 -XX:+UseParallelOldGC
        -XX:MaxGCPauseMillis=100 -XX:+UseAdaptiveSizePolicy
        -Dfile.encoding=UTF-8 -Dsun.jnu.encoding=UTF-8
        -Djenkins.install.runSetupWizard=false
        -Dhudson.model.LoadStatistics.clock=2000
        -Dhudson.model.ParametersAction.keepUndefinedParameters=true
        -Dorg.apache.commons.jelly.tags.fmt.timeZone=Asia/Shanghai
        -Duser.timezone=Asia/Shanghai
        -Dcom.sun.jndi.ldap.connect.pool.timeout=300000
        -Dhudson.security.csrf.DefaultCrumbIssuer.EXCLUDE_SESSION_ID=true

  jenkins-swarm:
    image: registry.jihulab.com/opsbox-dev/oes-jenkins-plus:jenkins-swarm
    restart: unless-stopped
    privileged: true
    volumes:
      - /tmp:/tmp
      - /lib/modules:/lib/modules # 好奇怪的配置在 rockylinux8，如果没有在执行 iptable 时会报异常
    depends_on:
      - jenkins-master
    links:
      - jenkins-master
    environment:
      JENKINS_URL: http://jenkins-master:8080
      JENKINS_USR: admin
      JENKINS_PSW: jenkins
      LABELS: docker

EOF
# 注意这里要给一个回车键

# 拉取镜像
docker-compose pull
# 启动服务
docker-compose up -d 
# 检查服务
docker-compose ps
# 查看日志
docker-compose logs -f
```

> 注：里面使用到的镜像打包源码地址：https://github.com/opsbox-dev/oes-jenkins-plus

启动日志效果如下图：

![](./images/jenkins-master-logs.png)

2. 打开浏览器访问 http://192.168.2.99:30080 

> 账号：*admin* 密码：*jenkins*

### 2.4 测试环境

创建一个任务测试环境，主要是确保 docker 服务可用。

![](./images/jenkins-home.png)

## 3. 基础概念

### 3.1 流程和工具

大师 **Martin Fowler** 对持续集成是这样定义的：持续集成是一种软件开发实践，即团队开发成员经常**集成他们的工作**，通常每个成员每天至少集成一次，也就意味着每天可能会发生多次集成。每次集成都通过自动化的构建（包括**编译，发布，自动化测试**）来验证，从而尽快地发现集成错误。许多团队发现这个过程可以大大减少集成的问题，让团队能够更快的开发内聚的软件。

软件开发和交付过程，开发人员会在不同阶段持续使用如下面图中的工具在实现构建、打包、归档、部署和测试，执行完一次就算一次交付。

![](images/tools.jpg)

> 注：上图整理了我目前在 CI/CD 工作中用的最多的工具，大家可以根据自己的情况整理出自己的技术栈和相关工具。

### 3.2 自由风格工程 和 Pipeline 工程的对比

### 3.3 Jenkins 2.0 中核心的 Jenkinsfile 两种语法背后的历史和原理

### 3.4 如何用你熟悉的技术栈扩展 Jenkins 的构建能力？

## 实战练习
