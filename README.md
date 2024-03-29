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
默认的Jenkins用户是`Jenkins`，很多命令都很受限，无法执行，我们还需要做如下改进：
1. 设置环境变量：将系统的PATH路径放入Jenkins中，具体做法是『系统管理』 -> 『系统设置』 -> 『环境变量』中添加环境变量，键名为`PATH`，键值为`/usr/bin:/usr/local/bin`
2. 更改Jenkins用户： 修改/etc/sysconfig/jenkins文件，将JENKINS_USER改为root
3. 修改Jenkins文件权限：
   
``` shell
sudo chown -R root:root /var/lib/jenkins
sudo chown -R root:root /var/cache/jenkins
sudo chown -R root:root /var/log/jenkins
```
4. 重启：sudo service jenkins restart

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
``` shell
node {   
  stage('Example') {
    sh 'hello world'
  }
}
```

脚本式语法比较灵活、可拓展，但也意味着更复杂，同时Grovvy语言的学习成本对于不使用Grovvy语言的团队其实是不必要的

#### 声明式语法

``` shell
pipeline {
  agent any
  stages {
    stage("Example") {
      steps {
        echo 'hello world'
      }
    }
  }
}
```

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
    stage("Example") {
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
正常情况下我们再推送完代码后，还需要切换到Jenkins页面手动触发构建，但我们也可以通过设置pipeline触发条件来达到自动化的过程。我们将从时间触发和事件触发分别来介绍一下pipeline的触发条件。

### 时间触发
Jenkins Pipeline中使用trigger指令来定义时间触发，tigger指令只能被定义在pipeline下，内置支持定时触发cron和轮训触发pollSCM等方式

#### 定时执行
定时执行就像cronjob，一到时间点酒执行，它的使用场景通常是一些周期性的job

``` shell
pipeline {
  agent any
  triggers {
    cron('0 0 * * *')
  }
  stages {
    stage("EXAMPLE") {
      steps {
        echo "定时触发执行构建"
      }
    }
  }
}
```

Jenkins trigger cron语法采用的是UNIX cron语法，一条cron包含5个字段，格式为 MINUTE 
HOUR DOM MONTH DOW。

#### 轮训触发
轮训代码仓库是指定期到代码仓库中询问代码是否变化，如果有变化就执行

``` shell
pipeline {
  agent any
  triggers {
    pollSCM('H/1 * * * *')
  }
  stages {
    stage("EXAMPLE") {
      steps {
        echo "轮训触发执行构建"
      }
    }
  }
}
```

### 事件触发
事件触发就是发生了某个事件就触发pipeline执行，这个事件可以是在手动在界面操作触发、其他pipeline触发、HTTP API Webhook触发等等

### pipeline触发
当某些任务的执行依赖其他任务的执行结果时，就可以使用triggers的upstream方法
``` shell
pipeline {
  agent any
  triggers {
    upstream(upstreamProjects: 'job_name', threshold: hudson.model.Result.SUCCESS) 
  }
  stages {
    stage("EXAMPLE") {
      steps {
        echo "由上游任务job_name完成后触发"
      }
    }
  }
}
```
hudson.model.Result是一个枚举值，包括一下值：
1. ABORTED: 任务被手动中止 
2. FAILURE: 构建失败
3. SUCCESS: 构建成功
4. UNSTABLE: 构建不稳定
5. NO_BUILD: 多阶段构建时，前面的阶段存在问题

### Webhook触发


## 多分支构建

1. 安装Git Parameter插件
2. 使用when指令区别分支逻辑

``` shell
stage('Deliver for development') {
  when {
    branch 'development'
  }
  steps {
    echo "do something for development branch"
  }
}
stage('Deploy for production') {
  when {
    branch 'production'
  }
  steps {
    echo "do something for production branch"
  }
}
```

## 参数配置
可以通过parameters指令来配置参数
```
pipeline {
  agent any
  parameters {
    booleanParam( name: 'isTony', defaultValue: true, description: 'is tony?')
  }
  stages {
    stage("EXAMPLE") {
      steps {
        echo ${params.bool}
      }
    }
  }
}
```

被传入的参数会被放入到params对象中，在pipeline中可以直接使用

### 参数类型
1. 字符串：string
2. 多行文本：text
3. 布尔值： booleanParam

每个类型都有3个属性：

1. name: 参数名
2. defaultValue: 默认值
3. description: 描述信息

## 项目实战