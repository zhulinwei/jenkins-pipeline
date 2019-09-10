# jenkins pipeline

## CI/CD
软件开发周期：编码 -> 构建 -> 集成 -> 测试 -> 交付 -> 部署

持续集成：强调对开发人员的每个提交，立刻进行构建、单元测试，根据测试结果确定新代码和原有代码能否正确地集成在一起

持续交付：在持续集成的基础上将代码部署到更贴近真实运行环境的类生产环境中, 进行更多的测试来更早地发现问题

持续部署：持续部署是在持续交付的基础上，当交付的代码通过评审之后自动部署到生产环境中


Jenkins是常用的集成工具之一
![Jenkins](http://image.51linwei.top/1_2evs4lCaKrD03-MzJl5_Dw.jpeg)


## 安装使用

### 原生安装

``` shell
sudo wget -O /etc/yum.repos.d/jenkins.repo http://pkg.jenkins.io/redhat/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat/jenkins.io.key
sudo yum install jenkins -y
sudo service jenkins start
```

### Docker安装
``` shell
docker run -u root --name jenkins -d -p 10080:8080 -v /var/jenkins_home:/var/jenkins_home -v /var/run/docker.sock:/var/run/docker.sock -v /usr/bin/docker:/usr/bin/docker --restart=always jenkins/jenkins:lts
```

## 基础语法

使用pipeline的意义

1. 版本化控制：可将pipeline提交到git/svn中进行版本控制
2. 有助于协助：pipeline的每次修改都是透明可见的，有助于加强交流
3. 有助于重用：相比手动操作复用性更高，不容易出现丢失关键信息等情况

### 语法选择

#### 脚本式语法

脚本式语法比较灵活、可拓展，但也意味着更复杂，同时Grovvy语言的学习成本对于不使用Grovvy语言的团队其实是不必要的
#### 声明式语法

声明式语法更符合常人阅读习惯，而且更加简洁，同时也是社区推荐的语法

## 环境变量

## 触发执行

## 项目实战