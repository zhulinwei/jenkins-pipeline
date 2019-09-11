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

Jenkins Pipeline是基于Groovy语言实现的一种领域特定语言，用于描述整条流水线式如何进行的

使用pipeline的意义

1. 版本化控制：可将pipeline提交到git/svn中进行版本控制
2. 有助于协助：pipeline的每次修改都是透明可见的，有助于加强交流
3. 有助于重用：相比手动操作复用性更高，不容易出现丢失关键信息等情况

### 语法选择

#### 脚本式语法

脚本式语法比较灵活、可拓展，但也意味着更复杂，同时Grovvy语言的学习成本对于不使用Grovvy语言的团队其实是不必要的
#### 声明式语法

声明式语法更符合常人阅读习惯，而且更加简洁，同时也是社区推荐的语法

pipeline 基本结构
1. pipeline: 代表整条流水线，包含整条流水线逻辑
2. agent: 指定流水线的执行位置
3. stages: 阶段列表，stages中至少包含一个stage
4. stage: 阶段，代表流水线的阶段，每个阶段都含有一个名称

以上每个部分都是必须的，少一个Jenkins都会报错

基本结构无法满足现实中多变的需求，所以Jenkins添加了很多指令来补充Jenkins Pipeline
1. post：包含整个Pipeline或者是阶段完成后一些附加的步骤，post部分分成多种条件块
   + always: 无论当前完成状态是什么，都执行
   + changed: 只要当前完成状态和上一次不一样就执行
   + fixed: 上一次完成状态为失败或不稳定，而当前状态为成功时执行
   + regression: 与fixed相反
   + aborted: 当前执行结果是中止状态时执行
   + failure: 当前状态为失败时执行
   + success: 当前状态为成功时执行
   + unstable: 当前状态为不稳定时执行
   + cheanup: 无论当前状态是什么，在所以条件块执行完成后都执行
2. environment: 用于设置环境变量
3. tools: 下载并安装我们指定的工具，并将其加入PATH环境中
4. input: 暂停pipeline，提示输入内容
5. options: 用于配置Jenkins Pipeline本身的选型
6. parallel: 并行执行多个step
7. parameters: 定义参数
8. triggers: 用于定义执行pipeline的触发器
9. when: 当满足when定义的条件时才执行

## 环境变量

环境变量可以看做是Pipeline和Jenkins进行交互的媒介，可以分为内置变量和自定义变量

### 全局变量
在pipeline执行时，Jenkins通过env将Jenkins内置变量暴露出来

``` shell
pipeline {
  agent any
  stages {
    stage("Exampe") {
      steps {
        echo "${env.BUILD_NUMBER}-${env.GIT_BRANCH}"
      }
    }
  }
}
```

我们列举几个常用的内置变量
1. BUILD_NUMBER: 构建号，累加的数字
2. BRANCH_NAME: 分支名称，常用语多分支pipeline项目中
3. BUILD_URL: 当前构建页面的URL
4. GIT_BRANCH: Git项目分支名称

env中的变量都是Jenkins内置的全局变量，如果有定义全局变量的需求，我们可以在『Manage Jenkins』-> 『Configure System』->『Global properties』中勾选『Environment variables』新增全局环境变量

### 局部变量

声明式pipeline提供了environment指令，方便自定义变量
``` shell
pipeline {
  agent any
  environment {
    NAME = 'tony'
  }
  stages {
    stage("EXAMPLE") {
      steps: {
        echo ${NAME}
      }
    }
  }
}
```
environment指令可以在pipeline中定义，代表变量作用域为整个pipeline；也可以定义在stage中，代表作用域只在该阶段有效

## 触发执行

## 项目实战