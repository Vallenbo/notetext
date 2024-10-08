# DevOps实践

在企业内部，应用架构一般都采用微服务，大部分项目都会对应几十、上百，甚至上千个微服务，并且还不仅仅只有一个项目，所以采用手工方式上线是不太现实的事情，特别是随着应用的增多，对应的工作量也变得不可想象。

由于上线的过程一般比较固定，大都是提前规定好、比较一致的流程，因此采用工具来完成这类“死板”的工作是比较推荐的方式，同时也能减少手工操作带来故障的风险。

本章主要介绍在生产环境中**持续集成（Continuous Integration，CI）**与**持续部署（Continuous Deployment，CD）**的使用--实现Jenkins流水线脚本自动发布应用到Kubernetes集群中，当然CI/CD是DevOps中非常重要的一个环节。

# 1. CI/CD介绍

CI/CD是一种通过在应用开发阶段引入自动化来频繁向客户交付应用的方法。

CI/CD的核心概念是持续集成、持续交付（CD，Continuous Delivery）和持续部署（Continuous Deployment，CD）作为一个面向开发和运营团队的解决方案，CI/CD主要针对在集成新代码时所引发的问题（也称“集成地狱”）。

具体而言，CI/CD在整个应用生命周期内（从集成和测试阶段到交付和部署）引入了持续自动化和持续监控，这些关联的事务通常被称为“CI/CD管道”，由开发和运维团队以敏捷方式协同支持。

## 1.1 持续集成(CI)

现代应用开发的目标是让多位开发人员同时开发同一个应用的不同功能。

但是，如果企业安排在一天内将所有分支源代码合并在一起，最终可能导致工作烦琐、耗时，而且需要手动完成。这是因为当一位独立工作的开发人员对应用进行更改时，有可能会有其他开发人员同时进行更改，从而引发冲突。


==持续集成可以帮助开发人员更加频繁地将代码更改合并到共享分支或主干中。一旦开发人员对应用所做的更改被合并，系统就会通过自动构建应用
并运行不同级别的自动化测试（通常是单元测试和集成测试）来验证这些更改，确保更改没有对应用造成破坏。==

这意味着测试内容涵盖了从类和函数到构成整个应用的不同模块，如果自动化测试发现新代码和现有代码之间有冲突，持续集成可以更加轻松快速地修复这些错误。

## 1.2 持续交付(CD)

完成持续集成中构建单元测试和集成测试的自动化流程后，通过持续交付可以自动将已验证的代码发布到存储库。为了实现高效的持续交付流程，务必要确保持续交付已内置于开发管道。

持续交付的目标是拥有一个可随时部署到生产环境的代码库。

在持续交付中，每个阶段（从代码更改的合并到生产就绪型构建版本的交付）都涉及测试自动化和代码发布自动化。在流程结束时，运维团队可以快速、轻松地将应用部署到生产环境中。

## 1.3 持续部署(CD)

对于一个成熟的CI/CD管道来说，最后的阶段是持续部署。作为持续交付（自动将生产就绪型构建版本发布到代码存储库）的延伸，持续部署可以自动将应用发布到生产环境中。

由于生产之前的管道阶段没有手动门控，因此持续部署在很大程度上都得依赖精心设计的测试自动化。

实际上，持续部署意味着开发人员对应用的更改在编写后的几分钟内就能生效，这更加便于持续接收和整合用户反馈。

总而言之，所有这些CI/CD的关联步骤都有助于降低应用的部署风险，因此更便于以小件的方式（非一次性）发布对应用的更改。

不过，由于还需要编写自动化测试以适应CI/CD管道中的各种测试和发布阶段，因此前期投资会很大。

## 1.4 CI和CD的区别

CI/CD中的CI即持续集成，它属于开发人员的自动化流程。成功的CI意味着应用代码的最新更改会定期构建、测试并合并到共享存储中。

==**该解决方案可以解决在一次开发中有太多应用分支，从而导致相互冲突的问题。**==

CI/CD中的CD指的是持续交付或持续部署，这些相关概念有时会交叉使用。

两者都事关管道后续阶段的自动化，但它们有时也会单独使用，用于说明自动化程度。持续交付通常是指开发人员对应用的更改会自动进行错误测试并上传到存储库（如GitLab或容器注册表），然后由运维团队将其部署到实时生产环
境中，旨在解决开发和运维团队之间可见性及沟通较差的问题，因此持续交付的目的就是确保尽可能减少部署新代码时所需的工作量。

持续部署指的是自动将开发人员的更改从代码库发布到生产环境中以供客户使用，它主要为解决因手动流程降低应用交付速度，从而使运维团队超负荷的问题。

持续部署以持续交付的优势为根基，实现了管道后续阶段的自动化。

CI/CD既可能仅指持续集成和持续交付构成的关联环节，也可以指持续集成、持续交付和持续部署这3个方面构成的关联环节。更为复杂的是有时持续交付也包含持续部署流程。

> 纠缠于这些语义其实没有必要，只需记得CI/CD实际上就是一个流程（通常表述为管道），用于在更大程度上实现应用开发的持续自动化和持续监控。

# 2. Pipeline介绍

提到CI工具，首先想到的就是“CI界”的大佬——Jenkins，虽然在云原生爆发的年代，蹦出来了很多云原生的CI工具，但是都不足以撼动Jenkins的地位。在企业中对于持续集成、持续部署的需求非常多，并且也会经常有一些比较复杂的需求，此时新生的CI工具不足以支撑这些很复杂的需求。但是Jenkins丰富的插件基本上可以满足任何场景，所以本书的CI/CD还是围绕Jenkins进行讲解。

限于篇幅，本书并不打算从基础的概念讲起，而是讲解Jenkins最佳的使用方式，也就是Jenkins Pipeline（Jenkins流水线）的使用。

首先介绍流水线的概念和类型，包括Jenkins**声明式流水线（Declarative Pipeline）**和**脚本式流水线（Scripted Pipeline）**，然后讲解流水线的基本语法和一些例子。

## 2.1 什么是Pipeline

**Jenkins流水线（Continuous Delivery Pipeline）**是一套插件，它支持在Jenkins中实现和集成持续交付流水线。流水线提供了一组可扩展的工具，用于通过Pipeline DSL将简单到复杂的交付流水线以代码的形势展现，类似于基础设施即代码。

持续交付流水线会经历一个复杂的过程：从版本控制、向用户和客户提交软件、软件的每次变更（提交代码到仓库）到软件发布（Release）。这个过程包括以一种可靠并可重复的方式构建软件，以及通过多个测试和部署阶段来开发构建好的软件（称为Build）。



**Jenkins流水线** 的定义被写在一个文本文件中（一般为Jenkinsfile），该文件“定制”了整个构建软件的过程。Jenkinsfile也可以被提交到项目的

代码仓库中，在Jenkins中可以直接引用。将持续交付流水线作为应用程序的一部分，像其他代码一样进行版本化和审查，这是流水线即代码的基础。创建Jenkinsfile并提交到代码仓库中的好处如下：

- 自动为所有分支创建流水线构建过程。
- 在流水线上进行代码复查/迭代。
- 对流水线进行审计跟踪。
- 流水线的代码可以被项目的多个成员查看和编辑。
- 可以对Jenkinsfile进行版本控制。

Jenkins流水线主要分为声明式和脚本式两种，包含**pipeline（流水线）**、**node（节点）**、**stage（阶段）**、**step（步骤）**等区块。

（1）**pipeline**：是用户定义的一个持续交付（CD）流水线模型。流水线的代码定义了整个构建过程，包括构建、测试和交付应用程序的阶段。另外，pipeline块是声明式流水线语法的关键部分。

（2）**node**：node是一个机器，它是Jenkins环境的一部分，另外，node块是脚本化流水线语法的关键部分。

（3）**stage（块）**：定义了在整个流水线的执行任务中概念不同的子集（比如Build、Test、Deploy阶段），它被许多插件用于可视化Jenkins流水线当前的状态/进展。

（4）**step**：本质上是指通过一个单一的任务告诉Jenkins在特定的时间点需要做什么，比如要执行shell命令，可以使用`sh SHELL_COMMAND`。

从上文可以了解，Jenkins流水线分为脚本式和声明式，而声明式是“新一代”的流水线，比脚本式更加灵活，可读性更强，并且声明式流水线支持以图形化的方式进行编辑，所以声明式流水线是着重学习的对象。

接下来看一下**声明式流水线**和**脚本式流水线**的语法区别。

## 2.2 声明式Pipeline

在声明式流水线的语法中，Jenkinsfile流水线过程定义在pipeline{}中，pipeline块定义了整个流水线中完成的所有工作，比如：

```sh
pipeline {	# 声明式流水线的一种特定语法，定义了包含执行整个流水线的所有内容和指令，也是声明式流水线的开始。
  agent any	# 指示Jenkins为整个流水线分配一个执行器和工作区（就是在流水线中的步骤在哪个agent上执行），该参数同样可以在stage中配置。
    stages {	# 描述流水线阶段的语法块，在脚本式流水线语法中，stage块是可选的。
      stage('Build') {	# 自定义流水线的执行过程（一个阶段）和名称，如所示的Build、Test、Deploy
        steps {			# 声明式流水线的一种特定语法，它描述了在这个stage中要运行的步骤。
          sh 'make'		# 执行一个shell命令。
        }
      }
      stage('Test') {
        steps {
          sh 'make check'
          junit 'reports/**/*.xml'	# 用于聚合测试报告。
        }
      }
      stage('Deploy') {
        steps {
          sh 'make publish'
      }
    }
  }
}
```

## 2.3 脚本化Pipeline

在脚本化流水线的语法中，会有一个或多个node块在整个流水线中执行核心工作，比如：

上一小节的声明式流水线也可以用以下脚本式流水线代替：

```sh
node {	# 在任何可用的代理上执行流水线或它的任何阶段，也可以指定到具体的节点。
  stage('Build') { # 定义流水线的阶段。stage块在脚本化流水线语法中是可选的，然而在脚本化流水线中实现stage块，可以清楚地在Jenkins UI界面中显示每个stage的任务子集。
   	sh 'make'
  }
  stage('Test') {
     sh 'make check'
     junit 'reports/**/*.xml'
  }
  stage('Deploy') {
      sh 'make publish'
  }
}
```

# 3. [声明式Pipeline的语法](https://www.jenkins.io/zh/doc/book/pipeline/syntax/)

前文提到，声明式是Jenkins新一代的流水线编写方式，而且声明式流水线支持类似Blue Ocean这类的图形化工具进行在线编辑，因此本书主要讲解声明式流水线的写法。声明式流水线在流水线子系统之上提供了一种更简单、更容易理解的语法。

之前也提到过，所有有效的声明式流水线必须包含在一个pipeline块中，比如以下是一个pipeline块的格式：

```sh
pipeline {	//Jenkinsfile (Declarative Pipeline)
/* insert Declarative Pipeline here */
}
```

在声明式流水线中，有效的基本语句和表达式遵循与Groovy的语法同样的规则，但有以下例外：

- 流水线顶层必须是一个 block，即 `pipeline{}`
- 分隔符可以不需要分号，但是每条语句都必须在自己的行上
- 块只能由 Sections、Directives、Steps 或 assignment statements组成
- 属性引用语句被当做是无参数的方法调用，比如 input 会被当做input()。

接下来学习声明式流水线的基本语法，包括sections、directives区块及parallel字段的用法。

## 3.1 Sections

声明式流水线中的 Sections 不是一个关键字或指令，而是包含一个或多个 Agent、Stages、 post、Directives 和 Steps 的代码区域块。

### 1. agent

agent表示整个流水线或特定阶段中的步骤和命令执行的位置，该部分必须在pipeline块的顶层被定义，也可以在stage中再次定义，但是stage级别是可选的。

（1）agent的配置函数
为了支持可能有的各种各样的流水线，agent支持一些不同类型的参数，这些参数应用在pipeline块的顶层，或stage指令内部。agent选配置如下：

**any**：在任何可用的代理上执行流水线，配置语法：

```sh
pipeline {
  agent any
}
```

**none**：表示该pipeline脚本没有全局的agent配置。当顶层的agent配置为none时，每个stage部分都需要包含它自己的agent。配置语法：

```
pipeline {
  agent none
  stages {
    stage('Stage For Build'){
      agent any
    }
  }
}
```

**label**: 以节点标签形式选择某个具体的节点执行 Pipeline 命令，例如：agent { label 'my-defined-label' }。节点需要提前配置标签。

```
pipeline {
  agent none
    stages {
      stage('Stage For Build'){
        agent { label 'role-master' }
        steps {
          echo "role-master"
        }
      }
    }
}
```

**node**: 和 label 配置类似，只不过是可以添加一些额外的配置，比如 customWorkspace(设置默认工作目录)

```
pipeline {
  agent none
    stages {
      stage('Stage For Build'){
        agent {
          node {
            label 'role-master'
            customWorkspace "/tmp/zhangzhuo/data"
          }
        }
        steps {
          sh "echo role-master > 1.txt"
        }
      }
    }
}
```

**dockerfile**：较少使用

**docker**：相当于dockerfile，可以直接使用docker字段指定外部镜像，可以省去构建的时间。比如使用maven镜像进行打包，同时可以指定args：

```sh
agent{
  docker{
    image '192.168.10.15/kubernetes/alpine:latest'   //镜像地址
    label 'role-master' //执行的节点，标签选择
    args '-v /tmp:/tmp'      //启动镜像的参数
  }
}
```

- kubernetes：**Jenkins也支持使用Kubernetes创建Slave，也就是常说的动态Slave。**

  > 需要部署 kubernetes 相关的插件，[官方文档](https://github.com/jenkinsci/kubernetes-plugin/)

  配置示例如下;

  - cloud: Configure Clouds 的名称，指定到其中一个k8s
  - slaveConnectTimeout: 连接超时时间
  - yaml: pod 定义文件，jnlp 容器的配置必须有配置无需改变，其余containerd根据自己情况指定
  - workspaceVolume：持久化 jenkins 的工作目录。
    - persistentVolumeClaimWorkspaceVolume：挂载已有pvc。

```
workspaceVolume persistentVolumeClaimWorkspaceVolume(claimName: "jenkins-agent", mountPath: "/", readOnly: "false")
```

- nfsWorkspaceVolume：挂载 nfs 服务器目录

```
workspaceVolume nfsWorkspaceVolume(serverAddress: "192.168.10.254", serverPath: "/nfs", readOnly: "false")
```

- dynamicPVC：动态申请 pvc，任务执行结束后删除

```
workspaceVolume dynamicPVC(storageClassName: "nfs-client", requestsSize: "1Gi", accessModes: "ReadWriteMany")
```

- emptyDirWorkspaceVolume：临时目录，任务执行结束后会随着 pod 删除被删除，主要功能多个任务 container 共享 jenkins 工作目录。

```
workspaceVolume emptyDirWorkspaceVolume()
```

- hostPathWorkspaceVolume：挂载 node 节点本机目录，注意挂载本机目录注意权限问题，可以先创建设置 777 权限，否则默认 kubelet 创建的目录权限为 755 默认其他用户没有写权限，执行流水线会报错。

```
workspaceVolume hostPathWorkspaceVolume(hostPath: "/opt/workspace", readOnly: false)
```

#### 示例

```sh
agent {
  kubernetes {
      cloud 'kubernetes'
      slaveConnectTimeout 1200
      workspaceVolume emptyDirWorkspaceVolume()
      yaml '''
kind: Pod
metadata:
  name: jenkins-agent
spec:
  containers:
  - args: [\'$(JENKINS_SECRET)\', \'$(JENKINS_NAME)\']
    image: '192.168.10.15/kubernetes/jnlp:alpine'
    name: jnlp
    imagePullPolicy: IfNotPresent
  - command:
      - "cat"
    image: "192.168.10.15/kubernetes/alpine:latest"
    imagePullPolicy: "IfNotPresent"
    name: "date"
    tty: true
  restartPolicy: Never
'''
  }
}
```

（2）agent的常见选项

- label：一个字符串，该标签用于运行流水线的位置。该选项对node、docker和dockerfile可用，node必须选择该选项

- customWorkspace：一个字符串，用于自定义工作区运行流水线或stage。它可以是相对路径，也可以是绝对路径，该选项对node、docker和dockerfile可用。比如：

``` sh
agent {
  node {
      label 'my-defied-label'
      customWorkspace 'some/other/path'
  }
}
```

- reuseNode：一个布尔值，默认为false。如果是true，则在同一个工作目录执行流水线。这个选项对docker和dockerfile有用，并且只有使用在个别stage的agent上才会有效。通常情况下使用docker作为agent时会开启此参数：


``` sh
agent {
  docker {
      image 'maven:3-alpine'
      label 'my-defined-label'
      args '-v /tmp:/tmp'
      reuseNode true
  }
}
```

  （3）agent的配置示例

  - 示例1：假设有一个Java项目，需要用mvn命令进行编译，此时可以使用maven的镜像作为agent。配置如下：

```sh
pipeline {
  agent { docker 'maven:3-alpine' }
  stages {
    stage('Example Build') {
      steps {
        echo 'Hello, Maven'
        sh 'mvn -B clean verify'
      }
    }
  }
}
```

- 示例2：本示例在流水线顶层将agent定义为none，那么此时stage部分就必须包含它自己的agent部分。

  在stage('Example Build')部分使用maven:3-alpine执行该阶段的步骤，在stage('Example Test')部分使用openjdk:8-jre执行该阶段的步骤。此时Pipeline如下：

``` 
pipeline {
  agent none
  stages {
    stage('Example Build') {
      agent { docker 'maven:3-alpine' }
      steps {
        echo 'Hello, Maven'
        sh 'mvn --version'
      }
    }
    stage('Example Test') {
      agent { docker 'openjdk:8-jre' }
      steps {
        echo 'Hello, JDK'
        sh 'java -version'
      }
    }
  }
} 
```

- kubernetes 示例

``` 
pipeline {
  agent {
    kubernetes {
      cloud 'kubernetes'
      slaveConnectTimeout 1200
      workspaceVolume emptyDirWorkspaceVolume()
      yaml '''
kind: Pod
metadata:
  name: jenkins-agent
spec:
  containers:
  - args: [\'$(JENKINS_SECRET)\', \'$(JENKINS_NAME)\']
    image: '192.168.10.15/kubernetes/jnlp:alpine'
    name: jnlp
    imagePullPolicy: IfNotPresent
  - command:
      - "cat"
    image: "192.168.10.15/kubernetes/alpine:latest"
    imagePullPolicy: "IfNotPresent"
    name: "date"
    tty: true
  - command:
      - "cat"
    image: "192.168.10.15/kubernetes/kubectl:apline"
    imagePullPolicy: "IfNotPresent"
    name: "kubectl"
    tty: true
  restartPolicy: Never
'''
    }
  }
  environment {
    MY_KUBECONFIG = credentials('kubernetes-cluster')
  }
  stages {
    stage('Data') {
      steps {
        container(name: 'date') {
          sh """
            date
          """
        }
      }
    }
    stage('echo') {
      steps {
        container(name: 'date') {
          sh """
            echo 'k8s is pod'
          """
        }
      }
    }
    stage('kubectl') {
      steps {
        container(name: 'kubectl') {
          sh """
            kubectl get pod -A  --kubeconfig $MY_KUBECONFIG
          """
        }
      }
    }
  }
  }
```

> [Jenkins的pipeline实践之GitSCM参数配置项详解](https://cloud.tencent.com/developer/article/2017111)

超时设置与部署参数switch语句选择

```sh
timeout(time: 1, unit: 'MINUTES') {
  script {
    env.deploy_option = input message: '选择操作', ok: 'deploy',
    parameters: [choice(name: 'deploy_option', choices: ['deploy', 'rollback', 'redeploy'], description: '选择部署环境')]
    switch("${env.deploy_option}"){
        case 'deploy':
            println('1.deploy prd env')
            break;
        case 'rollback':
            println('2.rollback env')
            break;
        case 'redeploy':
            println('3.redeploy env')
            break;
        default:
            println('error env')
    }
  }
}
```

docker的示例

```sh
pipeline {
  agent none 
  stages {
    stage('Example Build') {
      agent { docker 'maven:3-alpine' } 
      steps {
        echo 'Hello, Maven'
        sh 'mvn --version'
      }
    }
    stage('Example Test') {
      agent { docker 'openjdk:8-jre' } 
      steps {
        echo 'Hello, JDK'
        sh 'java -version'
      }
    }
  }
}

```


### 2. Post

post 一般用于流水线结束后的进一步处理，比如错误通知等。post可以针对流水线不同的结果做出不同的处理，就像开发程序的错误处理，比如Python语言的try catch。post可以定义在Pipeline或stage中，目前支持以下post-condition：

1. always：无论 Pipeline 或 stage 的完成状态如何，都允许运行该 post 中定义的指令；
2. changed：只有当前 Pipeline 或 stage 的完成状态与它之前的运行不同时，才允许在该 post 部分运行该步骤；
3. fixed：当本次 Pipeline 或 stage 成功，且上一次构建是失败或不稳定时，允许运行该 post 中定义的指令；
4. regression：当本次 Pipeline 或 stage 的状态为失败、不稳定或终止，且上一次构建的 状态为成功时，允许运行该 post 中定义的指令；
5. failure：只有当前 Pipeline 或 stage 的完成状态为失败（failure），才允许在 post 部分运行该步骤，通常这时在 Web 界面中显示为红色
6. success：当前状态为成功（success），执行 post 步骤，通常在 Web 界面中显示为蓝色 或绿色
7. unstable：当前状态为不稳定（unstable），执行 post 步骤，通常由于测试失败或代码 违规等造成，在 Web 界面中显示为黄色
8. aborted：当前状态为终止（aborted），执行该 post 步骤，通常由于流水线被手动终止触发，这时在 Web 界面中显示为灰色；
9. unsuccessful：当前状态不是 success 时，执行该 post 步骤；
10. cleanup：无论 pipeline 或 stage 的完成状态如何，都允许运行该 post 中定义的指令。和 always 的区别在于，cleanup 会在其它执行之后执行。

``` sh
pipeline {
  .....
  post {
    always{
      script{ println("流水线结束后，经常要做的事情") }
    }
    success{
      script{ println("流水线成功后，要做的事情") }
    }
    failure{
      script{ println("流水线失败后，要做的事情") }
    }
    failure{
      script{ println("流水线取消后，要做的事情") }
    }
  }
}
```


示例：一般情况下 post 部分放在流水线的底部，比如本实例，无论 stage 的完成状态如何，都会输出一条 I will always say Hello again!信息

```sh
pipeline {	//Jenkinsfile (Declarative Pipeline)
  agent any
  stages {
    stage('Example1') {
      steps {
        echo 'Hello World1'
      }
    }
    stage('Example2') {
      steps {
        echo 'Hello World2'
      }
    }
  }
  post {
    always {
      echo 'I will always say Hello again!'
    }
  }
}
```

也可以将 post 写在 stage，下面示例表示 Example1 执行失败执行 post。

```sh
pipeline {	//Jenkinsfile (Declarative Pipeline)
  agent any
  stages {
    stage('Example1') {
      steps {
        sh 'ip a'
      }
      post {
        failure {
          echo 'I will always say Hello again!'
        }
      }
    }
  }
}
```

### 3. stages

在前文中，其实已经能看到一些对stages的定义，stages包含一个或多个stage指令，同时可以在stage的steps块中定义真正执行的指令。stages部分是流水线描述的大部分工作（work）的位置。建议stages至少包含一个stage指令，用于持续交付过程的某个离散的部分，比如构建、测试或部署。

比如创建一个流水线，stages包含一个名为Example的stage，该stage执行echo 'Hello World'命令输出Hello World字符串：

```sh
pipeline {	//Jenkinsfile (Declarative Pipeline)
  agent any
  stages {
    stage('Example') {
      steps {
        echo 'Hello World'
      }
    }
  }
}
```

steps部分在给定的stage指令中执行一个或多个步骤，比如在steps中定义执行一条shell命令：

```sh
pipeline {	//Jenkinsfile (Declarative Pipeline)
  agent any
  stages {
    stage('Example') {
      steps {
        echo 'Hello World'
      }
    }
  }
}
```

或者也可以使用sh字段执行多条指令：

```sh
pipeline {	//Jenkinsfile (Declarative Pipeline)
  agent any
  stages {
    stage('Example') {
      steps {
        sh """
           echo 'Hello World1'
           echo 'Hello World2'
        """
      }
    }
  }
}
```

## 3.2 directives

directives可用于一些执行stage时的条件判断或预处理一些数据，和sections一致。directives不是一个关键字或指令，而是包含environment、options、parameters、triggers、stage、input、when等配置。

### 1. environment

environment主要用于在流水线中配置一些环境变量，根据配置的位置决定环境变量的作用域。可以定义在pipeline中作为全局变量，也可以配置在stage中作为该stage的环境变量。

该指令支持一个特殊的方法credentials()，该方法可用于在Jenkins环境中通过标识符访问预定义的凭证。对于类型为Secret Text的凭证，credentials()可以将该Secret中的文本内容赋值给环境变量。

对于类型为标准的账号密码型的凭证，指定的环境变量为username和password，并且也会定义两个额外的环境变量，分别为MYVARNAME_USR和MYVARNAME_PSW。

**基本变量使用**

```sh
pipeline {	// 示例
  agent any
  environment {   // 全局变量，会在所有stage中生效
    NAME= 'zhangzhuo'
  }
  stages {
    stage('env1') {
      environment { // 定义在stage中的变量只会在当前stage生效，其他的stage不会生效
        HARBOR = 'https://192.168.10.15'
      }
      steps {
        sh "env"
      }
    }
    stage('env2') {
      steps {
        sh "env"
      }
    }
  }
}
```

**使用变量引用 secret 的凭证**

```sh
pipeline {	//这里使用k8s的kubeconfig文件示例
  agent any
  environment {
    KUBECONFIG = credentials('kubernetes-cluster')
  }
  stages {
    stage('env') {
      steps {
        sh "env"  //默认情况下输出的变量内容会被加密
      }
    }
  }
}
```

**使用变量引用类型为标准的账号密码型的凭证**

这里使用 HARBOR 变量进行演示，默认情况下账号密码型的凭证会自动创建 3 个变量

- HARBOR_USR:会把凭证中 username 值赋值给这个变量
- HARBOR_PSW:会把凭证中 password 值赋值给这个变量
- HARBOR:默认情况下赋值的值为`usernamme:password`

```sh
pipeline {
  agent any
  environment {
    HARBOR = credentials('harbor-account')
  }
  stages {
    stage('env') {
      steps {
        sh "env"
      }
    }
  }
}
```

### 2. options

Jenkins 流水线支持很多内置指令，比如 retry 可以对失败的步骤进行重复执行 n 次，可以根据不同的指令实现不同的效果。比较常用的指令如下:

- buildDiscarder：保留多少个流水线的构建记录，比如`options { buildDiscarder(logRotator(numToKeepStr: '1')) }`。

- disableConcurrentBuilds：禁止流水线并行执行，防止并行流水线同时访问共享资源导致流水线失 败,比如`options { disableConcurrentBuilds() }`。

- disableResume ：如果控制器重启，禁止流水线自动恢复。比如`options { disableResume() }`。

- newContainerPerStage：agent 为 docker 或 dockerfile 时，每个阶段将在同一个节点的新容器中运行，而不是所有的阶段都在同一个容器中运行。比如`options { newContainerPerStage () }`。

- quietPeriod：流水线静默期，也就是触发流水线后等待一会在执行。比如`options{ quietPeriod(30) }`。

- retry：流水线失败后重试次数，比如`options { retry(3) }`。

- timeout：设置流水线的超时时间，超过流水线时间，job 会自动终止。如果不加 unit 参数默认为 1 分,比如`options{ timeout(time: 1, unit: 'HOURS') }`。

- skipDefaultCheckout：跳过checkout scm语句，比如`options { skipDefaultCheckout () }`。

- timestamps：为控制台输出时间戳, 比如`options { timestamps() }`。

- checkoutToSubdirectory 指定代码检出到 `$WORKSPACE`的子目录。

``` sh
  options {
  		checkoutToSubdirectory('testdir')
  }
```


常用汇总：

```sh
options {
		timestamps()
		disableConcurrentBuilds()
		timeout(time: 10, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '10'))
}
```

配置示例如下，只需要添加options字段即可：

**定义在 pipeline 中**

```sh
pipeline {
  agent any
  options {
    timeout(time: 1, unit: 'HOURS')  //超时时间1小时，如果不加unit参数默认为1分
    timestamps()                     //所有输出每行都会打印时间戳
    buildDiscarder(logRotator(numToKeepStr: '3')) //保留三个历史构建版本
    quietPeriod(10)  //静默10s
    retry(3)    //流水线失败后重试次数
  }
  stages {
    stage('env1') {
      options {
        timeout(time: 5, unit: 'MINUTES')
        retry(3)
        timestamps()
      }
      steps {
        sh "env"
        sleep 2
      }
    }
    stage('env2') {
      steps {
        sh "env"
      }
    }
  }
}
```

**定义在 stage 中**

Option 除了写在 Pipeline 顶层，还可以写在 stage 中，但是写在 stage 中的 option 仅支持 retry、 timeout、timestamps，或者是和 stage 相关的声明式选项，比如 skipDefaultCheckout。处于 stage 级别的 options 写法如下

```sh
pipeline {
  agent any
  stages {
    stage('env1') {
      options {   //定义在这里这对这个stage生效
        timeout(time: 2, unit: 'SECONDS') //超时时间2秒
        timestamps()                     //所有输出每行都会打印时间戳
        retry(3)    //流水线失败后重试次数
      }
      steps {
        sh "env && sleep 2"
      }
    }
    stage('env2') {
      steps {
        sh "env"
      }
    }
  }
}
```

### 3. parameters

parameters提供了一个用户在触发流水线时应该提供的参数列表，这些用户指定参数的值可以通过params对象提供给流水线的step（步骤）。目前支持的参数类型如下：

- string：字符串类型的参数。

  ```
  parameters { string(name:'DEPLOY_ENV', defaultValue:'staging', description: '') }
  // 表示定义一个名为DEPLOY_ENV的字符型变量，默认值为staging。
  ```

- text：文本型参数，一般用于定义多行文本内容的变量。

  ```
  parameters {text(name:'DEPLOY_TEXT', defaultValue:'One\nTwo\nThree\n',
  description: '')}	// 表示定义一个名为DEPLOY_TEXT的变量，默认值是'One\nTwo\nThree\n'
  ```

- booleanParam：布尔型参数。

  ```
  parameters {booleanParam(name:  'DEBUG_BUILD',defaultValue:  true,description: '')}
  ```

- choice：选择型参数，一般用于给定几个可选的值，然后选择其中一个进行赋值。

  ```
  parameters { choice(name: 'CHOICES',choices: ['one', 'two', 'three'], description: '') }
  // 表示定义一个名为CHOICES的变量，可选的值为one、two、three。
  ```

- password：密码型变量，一般用于定义敏感型变量，在 Jenkins 控制台会输出为*。

  ```
  parameters { password(name: 'PASSWORD',defaultValue: 'SECRET', description: 'A secret password')}
  // 表示定义一个名为PASSWORD的变量，其默认值为SECRE?。
  ```

**插件 Parameters**

- imageTag：镜像 tag，需要安装 Image Tag Parameter 插件后使用
- gitParameter：获取 git 仓库分支，需要 Git Parameter 插件后使用

**示例**

```sh
pipeline {
  agent any
  parameters {
    string(name: 'DEPLOY_ENV', defaultValue:  'staging', description: '1')   //执行构建时需要手动配置字符串类型参数，之后赋值给变量
    text(name:  'DEPLOY_TEXT', defaultValue: 'One\nTwo\nThree\n', description: '2')  //执行构建时需要提供文本参数，之后赋值给变量
    booleanParam(name: 'DEBUG_BUILD',  defaultValue: true, description: '3')   //布尔型参数
    choice(name: 'CHOICES', choices: ['one', 'two', 'three'], description: '4')  //选择形式列表参数
    password(name: 'PASSWORD', defaultValue: 'SECRET', description: 'A  secret password')  //密码类型参数，会进行加密
    imageTag(name: 'DOCKER_IMAGE', description: '', image: 'kubernetes/kubectl', filter: '.*', defaultTag: '', registry: 'https://192.168.10.15', credentialId: 'harbor-account', tagOrder: 'NATURAL')   //获取镜像名称与tag
    gitParameter(branch: '', branchFilter: 'origin/(.*)', defaultValue: '', description: 'Branch for build and deploy', name: 'BRANCH', quickFilterEnabled: false, selectedValue: 'NONE', sortMode: 'NONE',  tagFilter: '*', type: 'PT_BRANCH')
  }  //获取git仓库分支列表，必须有git引用
  stages {
    stage('env1') {
      steps {
        sh "env"
      }
    }
    stage('git') {
      steps {
        git branch: "$BRANCH", credentialsId: 'gitlab-key', url: 'git@192.168.10.14:root/env.git'   //使用gitParameter，必须有这个
      }
    }
  }
}
```

> "参考文献"：[Jenkins指定tag发布到k8s环境](https://www.cnblogs.com/Dev0ps/p/11740803.html)
>
> [Jenkins系列之六——拉取指定branch或tag ](https://www.cnblogs.com/Dev0ps/p/9125232.html)


### 4. triggers

在pipeline中可以用triggers实现自动触发流水线执行任务，可以通过Webhook、Cron、pollSCM和upstream等方式触发流水线。

假如某个流水线构建的时间比较长，或者某个流水线需要定期在某个时间段执行构建，可以使用cron配置触发器，比如周一到周五每隔4个小时执行一次：

**Cron**

定时构建假如某个流水线构建的时间比较长，或者某个流水线需要定期在某个时间段执行构建，可以使用 cron 配置触发器，比如周一到周五每隔四个小时执行一次

> 注意：H 的意思不是 HOURS 的意思，而是 Hash 的缩写。主要为了解决多个流水线在同一时间同时运行带来的系统负载压力。

```sh
pipeline {
  agent any
  triggers {
    cron('H */4 * * 1-5')   //周一到周五每隔四个小时执行一次
    cron('H/12 * * * *')   //每隔12分钟执行一次
    cron('H * * * *')   //每隔1小时执行一次
  }
  stages {
    stage('Example') {
      steps {
        echo 'Hello World'
      }
    }
  }
}
```

使用cron字段可以定期执行流水线，如果代码更新，想要重新触发流水线，可以使用pollSCM字段：

```sh
pipeline {
  agent any
  triggers {
    POLLscm('H */4 * * 1-5')
  }
  stages {
    stage('Example') {
      steps {
        echo 'Hello World'
      }
    }
  }
}
```

**Upstream**

Upstream 可以根据上游 job 的执行结果决定是否触发该流水线。比如当 job1 或 job2 执行成功时触发该流水线

目前支持的状态有 SUCCESS、UNSTABLE、FAILURE、NOT_BUILT、ABORTED 等。

```sh
pipeline {
  agent any
  triggers {
    upstream(upstreamProjects: 'env', threshold: hudson.model.Result.SUCCESS)  //当env构建成功时构建这个流水线
  }
  stages {
    stage('Example') {
      steps {
        echo 'Hello World'
      }
    }
  }
}
```

### 5. stage

stage指令位于stages下，包含一个steps、一个agent（可选）或其他特定的stage指令。流水线中实际执行的指令都在stage中配置，所以在流水线中，至少有一个stage。配置示例如下：

```sh
pipeline {
  agent any
  stages {
    stage('Example') {
      steps {
        echo 'Hello World'
      }
    }
  }
}
```

### 6. Input

Input 字段可以实现在流水线中进行交互式操作，比如选择要部署的环境、是否继续执行某个阶段等。

**配置 Input 支持以下选项**

- message：必选，需要用户进行 input 的提示信息，比如：“是否发布到生产环境？”；
- id：可选，input 的标识符，默认为 stage 的名称；
- ok：可选，确认按钮的显示信息，比如：“确定”、“允许”；
- submitter：可选，允许提交 input 操作的用户或组的名称，如果为空，任何登录用户均可提交 input；
- parameters：提供一个参数列表供 input 使用。

假如需要配置一个提示消息为“还继续么”、确认按钮为“继续”、提供一个 PERSON 的变量的参数，并且只能由登录用户为 alice 和 bob 提交的 input 流水线

```sh
pipeline {
  agent any
  stages {
    stage('Example') {
      input {
        message "还继续么?"
        ok "继续"
        submitter "alice,bob"
        parameters {
          string(name: 'PERSON', defaultValue: 'Mr Jenkins', description: 'Who should I say hello to?')
        }
      }
      steps {
        echo "Hello, ${PERSON}, nice to meet you."
      }
    }
  }
}
```

### 7. when

When 指令允许流水线根据给定的条件决定是否应该执行该 stage，when 指令必须包含至少 一个条件。如果 when 包含多个条件，所有的子条件必须都返回 True，stage 才能执行。

When 也可以结合 not、allOf、anyOf 语法达到更灵活的条件匹配。

**目前比较常用的内置条件如下**

- branch：当正在构建的分支与给定的分支匹配时，执行这个 stage，例如when { branch'master' }。注意，branch 只适用于多分支流水线。

- changelog：匹配提交的 changeLog 决定是否构建，例如:`when { changelog '.*^\\[DEPENDENCY\\] .+$' }`

- environment：当指定的环境变量和给定的变量匹配时，执行这个 stage，例如：when { environment name: 'DEPLOY_TO', value: 'production' }。

- equals：当期望值和实际值相同时，执行这个 stage，例如：when { equals expected: 2, actual: currentBuild.number }。

- expression：当指定的 Groovy 表达式评估为 True，执行这个 stage，例如：when { expression { return params.DEBUG_BUILD } }。

- tag：如果 TAG_NAME 的值和给定的条件匹配，执行这个 stage，例如：when { tag "release-*" }。

- not：当嵌套条件出现错误时，执行这个 stage，必须包含一个条件，例如：when { not { branch 'master' } }。

- allOf：当所有的嵌套条件都正确时，执行这个 stage，必须包含至少一个条件，例如：when { allOf { branch 'master'; environment name: 'DEPLOY_TO', value: 'production' } }。

- anyOf：当至少有一个嵌套条件为 True 时，执行这个 stage，例如：when { anyOf { branch 'master'; branch 'staging' } }。


**示例：当分支为`main`时执行Example Deploy步骤**

```sh
pipeline {
  agent any
  stages {
    stage('Example Build') {
      steps {
        echo 'Hello World'
      }
    }
    stage('Example Deploy') {
      when {
        branch 'main' //多分支流水线，分支为才会执行。
      }
      steps {
        echo 'Deploying'
      }
    }
  }
}
```

也可以同时配置多个条件，比如分支是`production`，而且`DEPLOY_TO`变量的值为main时，才执行Example Deploy

```sh
pipeline {
  agent any
  environment {
    DEPLOY_TO = "main"
  }
  stages {
    stage('Example Deploy') {
      when {
        branch 'production'
        environment name: 'DEPLOY_TO', value: 'main'
      }
      steps {
        echo 'Deploying'
      }
    }
  }
}
```

也可以使用`anyOf`进行匹配其中一个条件即可，比如分支为`main`或`DEPLOY_TO`为`main`或`master`时执行Deploy

```sh
pipeline {
  agent any
  stages {
    stage('Example Deploy') {
      when {
        anyOf {
          branch 'main'
          environment name: 'DEPLOY_TO', value: 'main'
          environment name: 'DEPLOY_TO', value: 'master'
        }
      }
      steps {
        echo 'Deploying'
      }
    }
  }
}
```

也可以使用 expression 进行正则匹配，比如当`BRANCH_NAME`为`main`或`maste`，并且`DEPLOY_TO`为`master`或`main`时才会执行Example Deploy

```sh
pipeline {
  agent any
  stages {
    stage('Example Deploy') {
      when {
        expression { BRANCH_NAME ==~ /(main|master)/ }
        anyOf {
          environment name: 'DEPLOY_TO', value: 'main'
          environment name: 'DEPLOY_TO', value: 'master'
        }
      }
      steps {
        echo 'Deploying'
      }
    }
  }
}
```

默认情况下，如果定义了某个 stage 的 agent，在进入该 stage 的 agent 后，该 stage 的 when 条件才会被评估，但是可以通过一些选项更改此选项。

比如在进入stage的agent 前评估when， 可以使用 beforeAgent，当when为true时才进行该 stage

**目前支持的前置条件如下**

- beforeAgent：如果 beforeAgent 为 true，则会先评估 when 条件。在 when 条件为 true 时，才会进入该 stage
- beforeInput：如果 beforeInput 为 true，则会先评估 when 条件。在 when 条件为 true 时，才会进入到 input 阶段；
- beforeOptions：如果 beforeInput 为 true，则会先评估 when 条件。在 when 条件为 true 时，才会进入到 options 阶段；

**beforeOptions 优先级大于 beforeInput 大于 beforeAgent**

示例1：配置一个beforeAgent。

```sh
pipeline {
  agent none
  stages {
    stage('Example Build') {
      steps {
        echo 'Hello World'
      }
    }
    stage('Example Deploy') {
      when {
        beforeAgent true
        branch 'production'
      }
      steps {
        echo 'Deploying'
      }
    }
  }
}
```

示例2：配置一个beforeInput。

```sh
pipeline {
  agent none
  stages {
    stage('Example Build') {
      steps {
        echo 'Hello World'
      }
    }
    stage('Example Deploy') {
      when {
        beforeInput true
        branch 'production'
      }
      input {
      	message "Deploy to production?"
      	id "simple-input"
      }
      steps {
        echo 'Deploying'
      }
    }
  }
}
```

示例3：配置一个beforeOptions。

```sh
pipeline {
  agent none
  stages {
    stage('Example Build') {
      steps {
        echo 'Hello World'
      }
    }
    stage('Example Deploy') {
      when {
        beforeOptions true
        branch 'testing'
      }
   	  options {
   		lock label: 'testing-deploy-envs',quantity: 1, variable: 'deployEnv'	
   		}
      steps {
        echo 'Deploying'
      }
    }
  }
}
```

steps包含一个完整的script步骤列表，是流水线真正指定操作的位置，比如构建命令、部署命令等。script步骤需要scripted-pipeline块并在声明式流水线中执行，对于大多数用例来说，script步骤并不是必要的。可以在steps中添加script进行for循环，示例如下：

```sh
pipeline {
  agent any
  stages {
    stage('Example') {
      steps {
        echo 'Hello World'
        
        script {
        	def browsers = ['chrome','firefox']
        	for (int i =0; i < browsers.size(); ++i) {
        		echo "Testing the ${browsers[i]} browsers "
        	}
         }   
       }
     }
  } 
}
```

## 3.3 parallel


在声明式流水线中可以使用 Parallel 字段，即可很方便的实现并发构建，比如对分支 A、B、 C 进行并行处理

```sh
pipeline {
  agent any
  stages {
    stage('Non-Parallel Stage') {
      steps {
        echo 'This stage will be executed first.'
      }
    }
    stage('Parallel Stage') {
      failFast true         //表示其中只要有一个分支构建执行失败，就直接推出不等待其他分支构建
      parallel {
        stage('Branch A') {
          steps {
            echo "On Branch A"
          }
        }
        stage('Branch B') {
          steps {
            echo "On Branch B"
          }
        }
        stage('Branch C') {
          stages {
            stage('Nested 1') {
              steps {
                echo "In stage Nested 1 within Branch C"
              }
            }
            stage('Nested 2') {
              steps {
               echo "In stage Nested 2 within Branch C"
              }
            }
          }
        }
      }
    }
  }
}
```


设置failFast为true表示并行流水线中任意一个stage出现错误，其他stage也会立即终止。也可以通过options配置在全局：

```sh
pipeline {
  agent any
  options {
  	parallelsAlwaysFailFast()
  }
  stages {
    stage('Non-Parallel Stage') {
      steps {
        echo 'This stage will be executed first.'
      }
    }
    stage('Parallel Stage') {
      parallel {
        stage('Branch A') {
          steps {
            echo "On Branch A"
          }
        }
        stage('Branch B') {
          steps {
            echo "On Branch B"
          }
        }
        stage('Branch C') {
          stages {
            stage('Nested 1') {
              steps {
                echo "In stage Nested 1 within Branch C"
              }
            }
            stage('Nested 2') {
              steps {
               echo "In stage Nested 2 within Branch C"
              }
            }
          }
        }
      }
    }
  }
}
```


parallel并行

```sh
 stage('Parallel Stage') {
    when {
        branch 'master'
    }
    failFast true
    parallel {
        stage('Branch A') {
            agent {
                label "for-branch-a"
            }
            steps {
                echo "On Branch A"
            }
        }
        stage('Branch B') {
            agent {
                label "for-branch-b"
            }
            steps {
                echo "On Branch B"
            }
        }
    }
}
```


# 4. Jenkinsfile的使用

前面讲过流水线支持两种语法，即声明式和脚本式，这两种语法都支持构建持续交付流水线，并且都可以用来在Web UI或Jenkinsfile中定义流水线，不过通常将Jenkinsfile放置于代码仓库中（当然也可以放在单独的代码仓库中进行管理）。

创建一个Jenkinsfile并将其放置于代码仓库中，有以下好处：

- 方便对流水线上的代码进行复查/迭代。
- 对管道进行审计跟踪。
- 流水线真正的源代码能够被项目的多个成员查看和编辑。

[Jenkins pipeline 基础语法详解](https://www.soulchild.cn/post/684864836/)


## 4.1 环境变量

### 1. 静态变量

Jenkins 有许多内置变量可以直接在 Jenkinsfile 中使用，可以通过 `JENKINS_URL/pipeline/syntax/globals#env` 获取完整列表。目前比较常用的环境变量如下

- BUILD_ID：当前构建的 ID，与 Jenkins 版本 1.597+中的 BUILD_NUMBER 完全相同
- BUILD_NUMBER：当前构建的 ID，和 BUILD_ID 一致
- BUILD_TAG：用来标识构建的版本号，格式为：`jenkins-${JOB_NAME}-${BUILD_NUMBER}`， 可以对产物进行命名，比如生产的 jar 包名字、镜像的 TAG 等；
- BUILD_URL：本次构建的完整 URL，比如：http://buildserver/jenkins/job/MyJobName/17/%EF%BC%9B
- JOB_NAME：本次构建的项目名称
- NODE_NAME：当前构建节点的名称；
- JENKINS_URL：Jenkins 完整的 URL，需要在 SystemConfiguration 设置；
- WORKSPACE：执行构建的工作目录。

示例如果一个流水线名称为`print_env`，第 2 次构建，各个变量的值。

```
BUILD_ID：2
BUILD_NUMBER：2
BUILD_TAG：jenkins-print_env-2
BUILD_URL：http://192.168.10.16:8080/job/print_env/2/
JOB_NAME：print_env
NODE_NAME：built-in
JENKINS_URL：http://192.168.10.16:8080/
WORKSPACE：/bitnami/jenkins/home/workspace/print_env
```

上述变量会保存在一个 Map 中，可以使用 env.BUILD_ID 或 env.JENKINS_URL 引用某个内置变量

```sh
pipeline {
  agent any
  stages {
    stage('print env') {
      parallel {
        stage('BUILD_ID') {
          steps {
            echo "$env.BUILD_ID"
          }
        }
        stage('BUILD_NUMBER') {
          steps {
            echo "$env.BUILD_NUMBER"
          }
        }
        stage('BUILD_TAG') {
          steps {
            echo "$env.BUILD_TAG"
          }
        }
      }
    }
  }
}
```


pipeline中的变量：

首先你要先理解pipeline可以用groovy语法来编写，而groovy是一门编程语言。所有的编程语言也都有各自的变量定义方式，这就容易让大家产生疑惑的地方，pipeline中可以有多种写法。

```
def name = "devops"
String name = "devops"
```

两种写法是一样的，def可以自动推导出变量类型，而String这种写法是精确这个变量是一个字符串类型的。

如果你再Jenkins图形界面设置了参数化构建没那么这些参数也变成了Jenkins全局变量，可以使用与Jenkins内置变量相同的引用方式。

如果某个stage定义的变量默认是局部变量，在后续的stage中可能语法引用，所以如果需要引用最好定义为全局变量。


全局变量的定义方式：

```
env.name = "devops"
```

引用方式： `"${env.name}"`

总之变量的定义无处不在，如果定义全局变量就用`env.变量名`的方式定义。

### 2. 动态变量

动态变量是根据某个指令的结果进行动态赋值，变量的值根据指令的执行结果而不同。如下所示

- returnStdout：将命令的执行结果赋值给变量，比如下述的命令返回的是 clang，此时 CC 的值为“clang”。
- returnStatus：将命令的执行状态赋值给变量，比如下述命令的执行状态为 1，此时 EXIT_STATUS 的值为 1。

```sh
pipeline {	//Jenkinsfile (Declarative Pipeline)
  agent any
  environment {
    // 使用 returnStdout
    CC = """${sh(
         returnStdout: true,
         script: 'echo -n "clang"'   //如果使用shell命令的echo赋值变量最好加-n取消换行
         )}"""
    // 使用 returnStatus
    EXIT_STATUS = """${sh(
         returnStatus: true,
         script: 'exit 1'
         )}"""
  }
  stages {
    stage('Example') {
      environment {
        DEBUG_FLAGS = '-g'
      }
      steps {
        sh 'printenv'
      }
    }
  }
}
```

- 必须在管道的顶层设置agent。如果agent设置none，则此操作将失败。

- 使用returnStout时，将在返回的字符串后面附加一个尾随空格。使用.trim()删除此项。

### 3. Jenkins内置变量

在pipeline执行时，Jenkins通过一个名为env的全局变量，将Jenkins内置环境变量暴露出来。其使用方法有多种，示例如下:

```sh
pipeline {
  agent any
  stages {
    stage('debug-列出当前流水线的所有环境变量) {
      setps {
        sh "printenv | sort"
      }
    }
    stage('Example') {
      steps {
         echo "Running ${env.BUILDNUMBER} on ${env.JENKINS_URL}" # 方法1
         echo "Running $env.BUILDNUMBER on $env.JENKINS_URL"  # 方法2
         echo "Running ${BUILDNUMBER} on ${JENKINS_URL}"   # 方法3 简写版本不推荐，难排查
      }
    }
  }
}
```


默认env的属性可以直接在pipeline中引用。所以以上方法都是合法的。但是不推荐方法三，因为出现变量冲突时，非常难查问题。

那么，env变量都有哪些可用属性呢? 
通过访问`<Jenkins master的地址>/pipeline-syntax/globals#env`来获取完整列表。 
在列表中，当一个变量被声明为"For a multibranch project"时，代表只有多分支项目才会有此变量。


### 4. 自定义变量

1.pipeline提供的environment指令中定义

```sh
pipeline {
    agent any
    environment {
        PATH="/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin"		// 覆盖默认的PATH变量值
        name='jack'
    }
    stages {
        stage("test env") {
          steps {
            sh "printenv"  #调试，打印所有env变量
            echo "${name}"  # jack
            echo "${env.name}"  # jack
          }
        }
     }
  }
```

``` sh
pipeline {
    agent any
    environment {
        FOO = "bar"
    }
    stages {
        stage("Env Variables") {
            environment {
                NAME = "Alan"
            }
            steps {
                echo "FOO = ${env.FOO}"
                echo "NAME = ${env.NAME}"
                script {
                    env.TEST_VARIABLE = "some test value"
                }
                echo "TEST_VARIABLE = ${env.TEST_VARIABLE}"
                withEnv(["ANOTHER_ENV_VAR=here is some value"]) {
                    echo "ANOTHER_ENV_VAR = ${env.ANOTHER_ENV_VAR}"
                }
            }
        }
    }
}
```

### 5. 覆盖环境变量

Jenkins Pipeline支持覆盖环境变量。您需要注意一些规则。

- 使用withEnv(["env=value]) { }语句块可以覆盖任何环境变量。 
- 使用environment {}语句块设置的变量不能使用命令式env.VAR = "value"赋值覆盖。 
- 命令式env.VAR = "value"分配只能覆盖使用命令式创建的环境变量。

这是一个示例性的Jenkinsfile，显示了所有三种不同的用例。

``` sh
pipeline {
    agent any
    environment {
        FOO = "bar"
        NAME = "Joe"
    }
    stages {
        stage("Env Variables") {
            environment {
                NAME = "Alan" // overrides pipeline level NAME env variable
                BUILD_NUMBER = "2" // overrides the default BUILD_NUMBER
            }
            steps {
                echo "FOO = ${env.FOO}" // prints "FOO = bar"
                echo "NAME = ${env.NAME}" // prints "NAME = Alan"
                echo "BUILD_NUMBER =  ${env.BUILD_NUMBER}" // prints "BUILD_NUMBER = 2"
                script {
                    env.SOMETHING = "1" // creates env.SOMETHING variable
                }
            }
        }
        stage("Override Variables") {
            steps {
                script {
                    env.FOO = "IT DOES NOT WORK!" // it can't override env.FOO declared at the pipeline (or stage) level
                    env.SOMETHING = "2" // it can override env variable created imperatively
                }
                echo "FOO = ${env.FOO}" // prints "FOO = bar"
                echo "SOMETHING = ${env.SOMETHING}" // prints "SOMETHING = 2"
                withEnv(["FOO=foobar"]) { // it can override any env variable
                    echo "FOO = ${env.FOO}" // prints "FOO = foobar"
                }
                withEnv(["BUILD_NUMBER=1"]) {
                    echo "BUILD_NUMBER = ${env.BUILD_NUMBER}" // prints "BUILD_NUMBER = 1"
                }
            }
        }
    }
}
```

### 6. 环境变量的互相引用

``` sh
environment {
  __server_name = 'email-server'
  __version = "${BUILD_NUMBER}"
  __artifact_name = "${__server_name}-${__version}.jar"
}
```

### 7. 将布尔值存储在环境变量中

关于使用环境变量，存在一种普遍的误解。存储为环境变量的每个值都将转换为String。 
当您存储布尔false值时，它将转换为"false"。如果要在布尔表达式中正确使用该值，则需要调用`"false".toBoolean()`method。

``` sh
pipeline {
    agent any
    environment {
        IS_BOOLEAN = false
    }
    stages {
        stage("Env Variables") {
            steps {
                script {
                    if (env.IS_BOOLEAN) {
                        echo "You can see this message, because \"false\" String evaluates to Boolean.TRUE value"
                    }
                    if (env.IS_BOOLEAN.toBoolean() == false) {
                        echo "You can see this message, because \"false\".toBoolean() returns Boolean.FALSE value"
                    }
                }
            }
        }
    }
}
```

### 8. 自定义全局环境变量

定义全局环境变量可以跨pipeline使用 进入Jenkins -- Manage Jenkins -- 找到Global properties -- 勾选Environment variables

## 4.2 凭证管理

Jenkins 的声明式流水线语法有一个 credentials()函数，它支持 secret text（加密文本）、username 和 password（用户名和密码）以及 secret file（加密文件）等。接下来看一下一些常用的凭证处理方法。

### 1. 加密文本

本实例演示将两个 Secret 文本凭证分配给单独的环境变量来访问 Amazon Web 服务，需要提前创建这两个文件的 credentials（实践的章节会有演示），Jenkinsfile 文件的内容如下

```sh
pipeline {		//Jenkinsfile (Declarative Pipeline)
  agent any
  environment {
    AWS_ACCESS_KEY_ID = credentials('txt1')
    AWS_SECRET_ACCESS_KEY = credentials('txt2')
  }
  stages {
    stage('Example stage 1') {
      steps {
        echo "$AWS_ACCESS_KEY_ID"
      }
    }
    stage('Example stage 2') {
      steps {
        echo "$AWS_SECRET_ACCESS_KEY"
      }
    }
  }
}
```

### 2. 用户名密码

本示例用来演示 credentials 账号密码的使用，比如使用一个公用账户访问 Bitbucket、GitLab、 Harbor 等。假设已经配置完成了用户名密码形式的 credentials，凭证 ID 为 harbor-account

```sh
pipeline {		//Jenkinsfile (Declarative Pipeline)
  agent any
  environment {
    BITBUCKET_COMMON_CREDS = credentials('harbor-account')
  }
  stages {
    stage('printenv') {
      steps {
        sh "env"
      }
    }
}
```

**上述的配置会自动生成 3 个环境变量**

- BITBUCKET_COMMON_CREDS：包含一个以冒号分隔的用户名和密码，格式为 username:password
- BITBUCKET_COMMON_CREDS_USR：仅包含用户名的附加变量
- BITBUCKET_COMMON_CREDS_PSW：仅包含密码的附加变量。

### 3. 加密文件

需要加密保存的文件，也可以使用 credential，比如链接到 Kubernetes 集群的 kubeconfig 文件等。

假如已经配置好了一个 kubeconfig 文件，此时可以在 Pipeline 中引用该文件

```sh
pipeline {	//Jenkinsfile (Declarative Pipeline)
  agent {
    kubernetes {
      cloud 'kubernetes'
      slaveConnectTimeout 1200
      workspaceVolume emptyDirWorkspaceVolume()
      yaml '''
kind: Pod
metadata:
  name: jenkins-agent
spec:
  containers:
  - args: [\'$(JENKINS_SECRET)\', \'$(JENKINS_NAME)\']
    image: '192.168.10.15/kubernetes/jnlp:alpine'
    name: jnlp
    imagePullPolicy: IfNotPresent
  - command:
      - "cat"
    image: "192.168.10.15/kubernetes/kubectl:apline"
    imagePullPolicy: "IfNotPresent"
    name: "kubectl"
    tty: true
  restartPolicy: Never
'''
    }
  }
  environment {
    KUBECONFIG = credentials('kubernetes-cluster')
  }
  stages {
    stage('kubectl') {
      steps {
        container(name: 'kubectl') {
          sh """
            # kubectl get pod -A  --kubeconfig $MY_KUBECONFIG
            mkdir -p ~/.kube && cp ${KUBECONFIG} ~/.kube/config
            kubectl get nodes
          """
        }
      }
    }
  }
}
```

> "更多其他类型的凭证可以参考" ：https://www.jenkins.io/doc/book/pipeline/jenkinsfile/#handlingcredentials。

## 4.3 参数处理

声明式流水线支持很多开箱即用的参数，可以让流水线接收不同的参数以达到不同的构建效果。

在Jenkinsfile中指定的parameters会在Jenkins Web UI自动生成对应的参数列表，此时可以在Jenkins页面单击Build With Parameters来指定参数的值，这些参数可以通过params变量被成员访问。

假设在Jenkinsfile中配置了名为Greeting的字符串参数，可以通过${params.Greeting}访问该参数，比如：

```sh
pipeline {		//Jenkinsfile (Declarative Pipeline)
  agent any
  parameters {
  	string(name：‘Greeting’, defaultValue: 'Hello', description: "How should I greet the world?")
  }
  stages {
    stage('Example') {
      steps {
        echo "${params.Greeting} World!"
       }
     }
  }
}
```

对应的脚本式流水线如下：

```sh
//Jenkinsfile (Scripted Pipeline)
properties([parameters([string(name：‘Greeting’, defaultValue: 'Hello', description: "How should I greet the world?")])])

node {
    echo "${params.Greeting} World!" 
}
```

## 4.4 处理失败

在声明式流水线中，可以根据不同的结果进行后置处理，比如always、unstable、success、failure和changed，具体可参考Pipeline Post的语法。

假如流水线构建失败需要进行邮件通知，可以参考如下示例（Jenkins需要配置邮箱）：

```sh
pipeline { 
    agent any 
    parameters {
        省略内容……
    }
    environment {
        省略内容……
    }
    stages {
        省略内容……
    }
    post {
        success {
            emailext (
                subject: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
                body: """<p>SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
                    <p>Check console output at "<a href="${env.BUILD_URL}">${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>"</p>""",
                to: "user1@qq.com,user2@qq.com",
                from: "admin@sina.com"
            )
        }
        failure {
            emailext (
                subject: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
                body: """<p>FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
                    <p>Check console output at "<a href="${env.BUILD_URL}">${env.JOB_NAME}          [${env.BUILD_NUMBER}]</a>"</p>""",
                to: "user1@qq.com,user2@qq.com",
                from: "admin@sina.com"
            )
        }
    }
}
```

## 4.5 使用多个代理

流水线允许在Jenkins环境中使用多个代理，这有助于更高级的用例，例如跨多个平台执行构建、测试等。

比如，在Linux和Windows系统的不同agent上进行测试：

```sh
pipeline {
    agent none
    stages {
        stage('Build') {
        	agent any
            steps {
                checkout scm
                sh 'make'
                stash includes: '**/target/*.jar', name: 'app'
            }
        }
         stage('Test on Linux') {
            agent { label 'linux'}
            steps {
			  unstash 'app'
               sh 'make check'
            }
            post {
              always {
              	junit '**/target/*.xml'
              }
            }
        }
       stage('Test on Windows') {
            agent { label 'windows'}
            steps {
			  unstash 'app'
               sh 'make check'
            }
            post {
              always {
              	junit '**/target/*.xml'
              }
            }
        }
    }
}
```

## 4.6 流水线回放功能

借助回放可以修改上次构建所使用的Jenkinsfile代码，进行更改后可以立即运行进行测试。

<img src="./assets/4.6-huifang.png" style="zoom: 50%;" />


## 4.7 Pipeline开发工具

### 片段生成器

流水线代码片段生成器，非常好用，在这里可以找到`每个插件以及Jenkins内置的方法`的使用方法.使用片段生成器可以根据个人需要生成方法，有些方法来源于插件，则需要安装相关的插件才能使用哦。

### 声明式语法生成器

可以生成声明式流水线语法的语句块

### 全局变量参考

这里面是已经安装的Jenkins插件和Jenkins内置的全局变量清单。

## 4.8 DSL方法

### 4.8.1 JSON数据格式解析

插件： Pipeline Utils Steps


``` sh
// 插件
def response = readJSON text: "${scanResult}"
println(scanResult)

// 原生方法
import groovy.json.*

@NonCPS
def GetJson(text){
    def prettyJson = JsonOutput.prettyPrint(text) 
    new JsonSlurperClassic().parseText(prettyJson)
}
```


### 4.8.2 流水线中使用凭证

``` sh
withCredentials([string(credentialsId: "xxxxx", variable: "sonarToken")]) {
    println(sonarToken)
}
```


### 4.8.3 代码管理-下载代码

```sh
//Git
checkout([$class: 'GitSCM', branches: [[name: "brnachName"]], 
            doGenerateSubmoduleConfigurations: false, 
            extensions: [], submoduleCfg: [], 
            userRemoteConfigs: [[credentialsId: "${credentialsId}", 
            url: "${srcUrl}"]]])
//Svn
checkout([$class: 'SubversionSCM', additionalCredentials: [], 
            filterChangelog: false, ignoreDirPropChanges: false, 
            locations: [[credentialsId: "${credentialsId}", 
            depthOption: 'infinity', ignoreExternalsOption: true, 
            remote: "${svnUrl}"]], workspaceUpdater: [$class: 'CheckoutUpdater']]
)
```


拉取仓库分支

```sg
stage('pull master code') {
    options { retry(3) }
    steps {
        checkout([$class: 'GitSCM', branches: [[name: "${RELEASE_VERSION}"]], 
        extensions: [
          [$class: 'CleanBeforeCheckout', deleteUntrackedNestedRepositories: true]],
        userRemoteConfigs: [[credentialsId: "${GIT_AUTH}", url: "${GIT_ADDRESS}"]]])
        }
}
```


拉取仓库pull-request分支下的目录或者文件

```sg
stage('pull pr branch config') {
  options { retry(3) }
  steps {
      checkout([$class: 'GitSCM',
          branches: [[name: "pull/${giteePullRequestIid}/MERGE"]], doGenerateSubmoduleConfigurations: false, submoduleCfg: [],
          extensions: [
              [$class: 'CleanBeforeCheckout', deleteUntrackedNestedRepositories: true],
              [$class: 'SparseCheckoutPaths', sparseCheckoutPaths:[[$class:'SparseCheckoutPath', path:'config/']]],
              [$class: 'CloneOption', depth: 1, noTags: true, shallow: true, honorRefspec: true]],
          userRemoteConfigs: [[credentialsId: "${GIT_AUTH}", name: 'origin', refspec: '+refs/pull/${giteePullRequestIid}/MERGE:refs/pull/${giteePullRequestIid}/MERGE', url: "${GIT_ADDRESS}"]]
      ])
  }
}
```


使用git插件拉取指定分支代码

```sh
stage("Checkout SCM"){
  when{
    expression {return ! params.SKIP_IMAGE_BUILD }
  }
  steps{
    git branch: "${BRANCH}", credentialsId: 'gitee-ci-bot', url: 'https://gitee.com/oschina/gitee-ent-web.git'
    script{
      env.OSC_GIT_COMMIT=sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim().substring(0,7)
    }
    sh "printenv OSC_GIT_COMMIT"
  }
}
```


## 4.9 Groovy编程

> "参考文档"：http://docs.groovy-lang.org/docs/latest/html/documentation/#_map_coercion
>
> http://groovy-lang.org/groovy-dev-kit.html


Groovy是一种功能强大，可选类型和动态语言，支持Java平台。旨在提高开发人员的生产力得益于简洁，熟悉且简单易学的语法。

可以与任何Java程序顺利集成，并立即为您的应用程序提供强大的功能，包括脚本编写功能，特定领域语言编写，运行时和编译时元编程以及函数式编程。


### 1.安装

- 下载安装包(先安装JDK)
- 解压安装包 获取安装包bin目录 
- 设置环境变量写入/etc/profile文件

> 安装包下载地址：https://groovy.apache.org/download.html

**JDK安装**

JDK(Java Development Kit)是Sun Microsystems针对Java开发员的产品。自从Java推出以来，JDK已经成为使用最广泛的Java SDK. JDK是整个Java的核心，包括了Java运行环境，Java工具和Java基础的类库。

> JDK下载面页：http://www.oracle.com/technetwork/java/javase/downloads/index.html


```shell
# Linux中使用wget下载的方法如下：
wget --no-cookies --no-check-certificate --header "Cookie: gpw_e24=http%3A%2F%2Fwww.oracle.com%2F; oraclelicense=accept-securebackup-cookie" "http://download.oracle.com/otn-pub/java/jdk/8u141-b15/336fa29ff2bb4ef291e347e091f7f4a7/jdk-8u141-linux-x64.tar.gz"
# 或
wget ftp://biguser:www.jb51.net@gw.086o2i.cn:8021/201704/tools/jdk-linux-x64.tar.gz
tar zxvf jdk-linux-x64.tar.gz
mv jdk1.8.0_131/ /usr/local/jdk1.8

# 设置环境变量
vim /etc/profile
# 在末尾输入以下内容:
export JAVA_HOME=/usr/local/jdk1.8
export CLASSPATH=$CLASSPATH:$JAVA_HOME/lib:$JAVA_HOME/jre/lib
export PATH=$JAVA_HOME/bin:$JAVA_HOME/jre/bin:$PATH:$HOMR/bin

# 保存文件后，使其生效:
source /etc/profile

# 检测是否设置正确:
java -versio
```

配置Groovy环境变量

```shell
vi /etc/profile
export GROOVY_HOME=/usr/local/groovy-3.0.7/
export PATH=$PATH:$GROOVY_HOME/bin

source /etc/profile
groovysh
```

> "参考文献"：[Jenkins系列08-Pipeline](https://www.linux98.com/archives/jenkins08)

# 5. DevOps平台建设

本节介绍持续集成、持续部署的步骤及过程，主要讲解如何基于Kubernetes和Jenkins搭建DevOps平台、如何在Kubernetes中发版、DevOps平台用到的工具的安装以及基于Kubernetes和Jenkins的CI/CD过程。

在Kubernetes中进行CI/CD的过程，一般的步骤如下：

1. 在GitLab中创建对应的项目。

2. 配置Jenkins集成Kubernetes集群 ， 后期Jenkins的Slave为在Kubernetes中动态创建的Slave。

3. Jenkins创建对应的任务(Job)，集成该项目的Git地址和Kubernetes集群。

4. 开发者将代码提交到GitLab。

5. 若配置了钩子，则推送（Push）代码会自动触发Jenkins构建，若没有配置钩子，则需要手动构建。

6. Jenkins控制Kubernetes（使用的是Kubernetes插件）创建JenkinsSlave（Pod形式）。

7. Jenkins Slave根据流水线定义的步骤执行构建。

8. 通过Dockerfile生成镜像。

9. 将镜像提送到私有Harbor（或者其他的镜像仓库）。

10. Jenkins再次控制Kubernetes进行最新的镜像部署。

11. 流水线结束，删除Jenkins Slave。


上述为比较常用的步骤，中间还可能涉及自动化测试等其他步骤，可自行根据业务场景添加或删除。

上述流水线步骤一般写在Jenkinsfile中，Jenkins会自动读取该文件，同时Jenkinsfile和Dockerfile可一并和代码放置于GitLab中，可以和代码一样实现版本控制，单独配置也可以

下图是我们当前示例的流程图

<img src="./assets/6-1.png" style="zoom: 50%;" />

<img src="./assets/6-2.png" style="zoom:33%;" />


生产环境CICD流程

![](./assets/6-3.png){: .zoom}

了解了基于Kubernetes和Jenkins的发版流程后，接下来需要一步一步安装所需要的工具，之后即可开始CI/CD的学习。如果公司内已经有可以用来联系的平台，可以不安装这些工具。建设DevOps平台需要建设如下服务

- Jenkins 
- GitLab 
- Harbor 
- Coredns

可以安装在Kubernetes集群中，也可以单独部署，生产环境建议选择直接在单独的机器上部署，这是企业常用的一种方式。

## 5.1  安装Jenkins

war包安装。其他方式参考devops

```yaml
$ sudo yum install fontconfig java-11-openjdk
$ sudo wget https://get.jenkins.io/war-stable/2.375.1/jenkins.war
$ sudo java -jar jenkins.war
# http://localhost:8080
```

### 2. docker-master-slave方式(推荐)

首先需要一个Linux服务器，配置不低于2核CPU、4GB内存和40GB的硬盘。首先安装docker和docker-compose

```sh
$ wget https://download.docker.com/linux/static/stable/aarch64/docker-20.10.9.tgz
$ tar zxf docker-20.10.9.tgz
$ cp docker/* /usr/bin
$ cat >/lib/systemd/system/docker.service <<EOF
[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
After=network-online.target firewalld.service
Wants=network-online.target
[Service]
Type=notify
ExecStart=/usr/bin/dockerd
ExecReload=/bin/kill -s HUP $MAINPID
LimitNOFILE=infinity
LimitNPROC=infinity
TimeoutStartSec=0
Delegate=yes
KillMode=process
Restart=on-failure
StartLimitBurst=3
StartLimitInterval=60s
[Install]
WantedBy=multi-user.target
EOF

$ systemctl daemon-reload
$ systemctl start docker 
$ docker version

# 如果镜像仓库未配置证书，则需要配置insecure-registry：
$ cat > /etc/docker/daemon.json <<'EOF'
{
  "graph": "/data/docker",
  "storage-driver": "overlay2",
  "insecure-registries": ["hub.gitee.cc"],
  "registry-mirrors": ["https://25bxwt20.mirror.aliyuncs.com"],
  "live-restore": true,
  "bip": "172.16.200.1/24",
  "exec-opts": ["native.cgroupdriver=systemd"],
  "hosts": ["tcp://0.0.0.0:2375", "unix:///var/run/docker.sock"],
  "log-opts": {
    "max-size":"100M",
    "max-file":"3"
  }
}
EOF
$ systemctl restart docker
```

docker-compose安装

```sh
$ sudo curl -L https://get.daocloud.io/docker/compose/releases/download/v2.4.1/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose

$ sudo chmod +x /usr/local/bin/docker-compose
$ sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose

$ docker-compose version
Docker Compose version v2.4.1
```

 **组成部分**

- master 接收 webhook ，调度任务等
- agent 执行构建任务
- dind 构建镜像

 **开始部署**

仓库地址： https://gitee.com/hujianli94net/simple-jenkins.git

部署 master

- 修改持久化目录属主为 1000:1000

  ```sh
  mkdir devops && cd devops
  git clone https://gitee.com/hujianli94net/simple-jenkins.git
  cd simple-jenkins/jenkins/master
  chown -R 1000:1000 jenkins_home
  chown -R 1000:1000 init.groovy.d
  ```

  - 执行 docker-compose 命令完成部署

  ```sh
  docker network create -d bridge jenkins-cluster		# 创建 docker network
  docker-compose up -d
  ```

- 完成初始化并安装插件

blueocean 镜像已包含常用插件，在首次访问 Jenkins 时，**选择安装推荐插件即可。**

在安装之前先配置国内的插件源，单击Advanced，将插件源更改为国内插件源(https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json）

初始化完成后，登录 Jenkins ，进入“系统管理->插件管理->可选插件” 在搜索框搜索如下插件安装。

> - Git Parameter
> - Build With Parameters
> - Timestamper
> - Git Pipeline for Blue Ocean
> - GitLab
> - GitLab Authentication
> - GitLab Branch Source
> - Generic Webhook Trigger
> - Image Tag Parameter
> - Workspace Cleanup

> - Role-based Authorization Strategy
> - Email Extension Template
> - Git Pipeline for Blue Ocean
> - Pipeline: GitHub
> - Blue Ocean 
> - Blue Ocean Pipeline Editor
> - Web for Blue Ocean
> - Blue Ocean Core JS
> - Pipeline SCM API for Blue Ocean
> - Dashboard for Blue Ocean
> - Build With Parameters
> - Dynamic Extended Choice Parameter Plug-In
> - Extended Choice Parameter
> - List Git Branches Parameter
> - Pipeline
> - Pipeline: Deprecated Groovy Libraries
> - SSH Pipeline Steps
> - Kubernetes
> - Kubernetes CLI
> - Kubernetes Credentials
> - Kubernetes Client API
> - Active Choices
> - Design Language
> - REST API for Blue Ocean
> - REST Implementation for Blue Ocean

勾选后，单击Download now and install after restart，之后可以在Installed选项卡下看到已经安装的插件。

- 配置 Gitee 插件

完成 gitee 插件安装后，进入“系统管理->系统配置”，定位到 “Gitee 配置” 部分，创建证书令牌（需要先在 [Gitee](https://gitee.com/profile/personal_access_tokens) 创建 Gitee API V5 的私人令牌），并完成配置。

<img src="./assets/6-4.png" style="zoom: 33%;" />

<img src="./assets/6-5.png" style="zoom: 50%;" />

点击测试连接。显示成功就保存，继续往后操作

![](./assets/6-6.png){: .zoom}

- 新建 agent

进入 “系统管理->节点列表” ，点击新建节点

创建两个“固定节点”，节点配置如下

<img src="./assets/6-7.png" style="zoom: 50%;" />

创建完成后，点击节点，查看节点连接信息，部署 agent 时需要提供该连接信息

![](./assets/6-8.png){: .zoom}

![](./assets/6-9.png){: .zoom}

 **部署 agent**

- 构建自定义 agent 镜像

下载 kubectl 、 docker 、 helm 二进制文件

[docker二进制下载地址](https://download.docker.com/linux/static/stable/x86_64/)


```sh
$ cd jenkins/agent/bin
# 下载helm
$ wget -c https://repo.huaweicloud.com/helm/v3.8.0/helm-v3.8.0-linux-386.tar.gz

# 例如，要在 Linux 中下载 v1.21.0 版本，请输入：
$ curl -LO https://dl.k8s.io/release/v1.21.0/bin/linux/amd64/kubectl
$ chmod +x *

# ll
total 134552
-rwxr-xr-x 1 root root 50707288 Feb 14 16:45 docker
-rwxr-xr-x 1 3434 3434 40636416 Jan 25  2022 helm
-rwxr-xr-x 1 root root 46436352 Feb 14 16:44 kubectl
```

准备 .kube 和 .docker 文件夹，用于授权访问 kubernetes api-server 和 harbor 镜像仓库

`.kube/config` 从 kubernetes 集群内获取,

创建 `.docker/config.json`

```sh
harbor_addr=hub.gitee.cc
harbor_username=admin
harbor_password=oschina123
auth_str=$(echo -ne "$harbor_username:$harbor_password" | base64)
cat > .docker/config.json <<EOF
{
        "auths": {
                "$harbor_addr": {
                        "auth": "$auth_str"
                }
        }
}
EOF
```

docker build agent 镜像

```sh
$ docker build -t $harbor_addr/jenkins/inbound-agent:4.11-1-kube .
$ docker push $harbor_addr/jenkins/inbound-agent:4.11-1-kube		# 推送镜像到harbor仓库，habor仓库需要提前部署
```

- 执行 docker-compose 命令完成部署

修改 docker-compose.yml 中的 secret 个 agent_name ，对应于上面 master 中创建的节点连接信息

```sh
$ chown -R 1000:1000 /root/devops/simple-jenkins/jenkins/agent/
$ docker-compose up -d
```

**部署 dind**

- 创建 daemon.json

```sh
$ cd dind
$ harbor_addr=hub.gitee.cc
$ dns_addr=192.168.1.60
$ cat > etc/daemon.json <<EOF
{
  "registry-mirrors": ["https://gfty7g09.mirror.aliyuncs.com"],
  "insecure-registries": ["$harbor_addr"],
  "live-restore": true,
  "default-shm-size": "128M",
  "max-concurrent-downloads": 10,
  "oom-score-adjust": -1000,
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "dns": [
    "$dns_addr"
  ]
}
EOF
```

- 执行 docker-compose 命令完成部署

```sh
$ docker-compose up -d
```

查看所有容器

```sh
$ docker ps
CONTAINER ID   IMAGE                                            COMMAND                  CREATED              STATUS              PORTS                                                                                      NAMES
425c73c120ec   docker:20.10.10-dind                             "dockerd-entrypoint.…"   36 seconds ago       Up 16 seconds       2375-2376/tcp                                                                              dind-docker01-1
89f9bcfd1f12   docker:20.10.10-dind                             "dockerd-entrypoint.…"   36 seconds ago       Up 16 seconds       2375-2376/tcp                                                                              dind-docker02-1
6bd628bfba16   hub.gitee.cc/jenkins/inbound-agent:4.11-1-kube   "/usr/local/bin/jenk…"   About a minute ago   Up About a minute                                                                                              agent-agent02-1
171340672a53   hub.gitee.cc/jenkins/inbound-agent:4.11-1-kube   "/usr/local/bin/jenk…"   About a minute ago   Up About a minute                                                                                              agent-agent01-1
0e4f1c2ca769   jenkins/jenkins:2.390-jdk11                      "/usr/bin/tini -- /u…"   11 minutes ago       Up 11 minutes       0.0.0.0:8080->8080/tcp, :::8080->8080/tcp, 0.0.0.0:50000->50000/tcp, :::50000->50000/tcp   jenkins-master
```

完成

### 3. helm方式部署(推荐)

##### 3.1 部署nfs

准备一块>2T的数据盘，格式化后挂载到/data目录下。/data目录为nfs的数据目录。直接命令安装，示例如下

```sh
$ docker run -d --name nfs --privileged -v /data:/nfsshare -e SHARED_DIRECTORY=/nfsshare itsthenetwork/nfs-server-alpine:latest
```

docker-compose部署`docker-compose.yaml`

```yaml
---
version: '3'
services:
  nfs:
    image: atompi/nfs-server:9
    restart: always
    ports:
      - "2049:2049"
    volumes:
      - /data/nfs:/nfsshare
    environment:
      - SHARED_DIRECTORY=/nfsshare
    privileged: yes
```

> "可参考文献"：https://github.com/ehough/docker-nfs-server.git
>
> https://github.com/sjiveson/nfs-server-alpine
>
> https://gitee.com/atompi/nfs-server-docker

##### 3.2 部署StorageClass

> GitHub 地址：https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner
>
> Chart 仓库地址：https://artifacthub.io/packages/helm/nfs-subdir-external-provisioner/nfs-subdir-external-provisioner

nfs-subdir-external-provisioner使用方法

```sh
$ helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/
$ helm repo update

# 可以下载到本地离线安装
# helm pull nfs-subdir-external-provisioner/nfs-subdir-external-provisioner --version 4.0.17 --untar

# 默认镜像
helm install nfs-subdir-external-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner
--set nfs.server=172.18.0.95 \
--set nfs.path=/gitee \
--set storageClass.name=gitee-nfs-client \
--set storageClass.reclaimPolicy="Retain" \
--namespace nfs-sc --create-namespace

# --------------------- 认镜像下载网络受限，使用中国内部镜像 ------------------------------------
helm install nfs-subdir-external-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner \
--set image.repository=easzlab/nfs-subdir-external-provisioner \
--set image.tag=v4.0.2 \
--set nfs.server=192.168.1.60 \
--set nfs.path=/QaTest \
--set storageClass.name=gitee-nfs-client \
--set storageClass.reclaimPolicy="Retain" \
--namespace nfs-sc --create-namespace
```

检查StorageClass部署状态

```sh
$ kubectl get pod -n nfs-sc
NAME                                               READY   STATUS    RESTARTS   AGE
nfs-subdir-external-provisioner-7fdff5bfc9-q28qc   1/1     Running   0          2m49s
$ kubectl get sc
NAME               PROVISIONER                                     RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
gitee-nfs-client   cluster.local/nfs-subdir-external-provisioner   Retain          Immediate           true                   2m54s
$ helm list -n nfs-sc
NAME                            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                                   APP VERSION
nfs-subdir-external-provisioner nfs-sc          1               2022-10-18 14:41:18.070839496 +0800 CST deployed        nfs-subdir-external-provisioner-4.0.17  4.0.2
```

如上显示，表示StorageClass启动正常。

##### 3.3  部署jenkins

> 前提条件：NFS 类型的 StorageClass 来做持久化存储,  StorageClass 名称`gitee-nfs-client`

原始仓库

```sh
$ helm repo add jenkins https://charts.jenkins.io
$ helm repo update
```

Chart 仓库地址：https://hujianli94.github.io/mycharts/jenkinsci

设置helm源

```sh
$ helm repo add mycharts https://hujianli94.github.io/mycharts/
$ helm repo update
$ helm search repo jenkinsci
NAME                    CHART VERSION   APP VERSION     DESCRIPTION
mycharts/jenkinsci      0.1.0           2.390-jdk11     A Helm chart for Kubernetes

$ helm pull mycharts/jenkinsci --version=0.1.0
$ helm pull mycharts/jenkinsci --version=0.1.0 --untar
# 修改jenkinsci/values.yaml配置
# 修改storageClassName
# 修改nfs信息
# 根据要求进行修改
```

使用helm进行部署

```sh
$ helm upgrade jenkinsci jenkinsci --install --namespace kube-ops --create-namespace
Release "jenkinsci" does not exist. Installing it now.
NAME: jenkinsci
LAST DEPLOYED: Wed Feb 15 10:04:21 2023
NAMESPACE: kube-ops
STATUS: deployed
REVISION: 1
NOTES:
1. Get the application URL by running these commands:
  export NODE_PORT=$(kubectl get --namespace kube-ops -o jsonpath="{.spec.ports[0].nodePort}" services jenkinsci)
  export NODE_IP=$(kubectl get nodes --namespace kube-ops -o jsonpath="{.items[0].status.addresses[0].address}")
  echo http://$NODE_IP:$NODE_PORT
```

查看部署pod的状态和日志

```sh
# kubectl get pod,svc,pvc -n kube-ops
NAME                             READY   STATUS    RESTARTS   AGE
pod/jenkinsci-6465964b6d-rplhl   1/1     Running   0          2m43s

NAME                TYPE       CLUSTER-IP        EXTERNAL-IP   PORT(S)                          AGE
service/jenkinsci   NodePort   192.168.105.182   <none>        8080:32080/TCP,50000:32050/TCP   2m43s

NAME                              STATUS   VOLUME      CAPACITY   ACCESS MODES   STORAGECLASS       AGE
persistentvolumeclaim/jenkinsci   Bound    jenkinsci   100Gi      RWO            gitee-nfs-client   2m43s

# kubectl exec -it pod/jenkinsci-6465964b6d-rplhl  -n kube-ops -- cat /var/jenkins_home/secrets/initialAdminPassword
917475173c3746959bcd50d608fb28d7
# kubectl logs -f pod/jenkinsci-6465964b6d-rplhl -n kube-ops
.....
Jenkins initial setup is required. An admin user has been created and a password generated.
Please use the following password to proceed to installation:

917475173c3746959bcd50d608fb28d7
This may also be found at: /var/jenkins_home/secrets/initialAdminPassword
*************************************************************
*************************************************************
*************************************************************
```

使用Node+NodePort端口进行访问，使用初始密码登录。

> 安装插件配置然后基本环境其他操作与docker部署一致。参考文献：https://www.yuque.com/wuyuexin/bu3d95/eikxqa

### 4. yaml方式部署(不推荐)

既然要基于 Kubernetes 来做 CI/CD，我们这里最好还是将 Jenkins 安装到 Kubernetes 集群当中，安装的方式也很多，我们这里仍然还是使用手动的方式，这样可以了解更多细节，对应的资源清单文件如下所示：

`jenkins-pv.yaml`

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: jenkinsci
  labels:
    app: jenkinsci
spec:
  capacity:
    storage: 100Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: gitee-nfs-client
  mountOptions:
    - hard
    - nfsvers=4.1
  nfs:
    server: 192.168.1.60
    path: /QaTest/jenkins
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: jenkins-pvc
  namespace: kube-ops
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: gitee-nfs-client
  resources:
    requests:
      storage: 100Gi
  selector:
    matchLabels:
      app: jenkinsci
```

在storageClassName的指定目录下持久化数据，设置指定名称的静态PV和PVC。

`jenkins.yaml`

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: jenkins
  namespace: kube-ops
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: jenkins
rules:
  - apiGroups: ["extensions", "apps"]
    resources: ["deployments", "ingresses"]
    verbs: ["create", "delete", "get", "list", "watch", "patch", "update"]
  - apiGroups: [""]
    resources: ["services"]
    verbs: ["create", "delete", "get", "list", "watch", "patch", "update"]
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["create", "delete", "get", "list", "patch", "update", "watch"]
  - apiGroups: [""]
    resources: ["pods/exec"]
    verbs: ["create", "delete", "get", "list", "patch", "update", "watch"]
  - apiGroups: [""]
    resources: ["pods/log", "events"]
    verbs: ["get", "list", "watch"]
  - apiGroups: [""]
    resources: ["secrets"]
    verbs: ["get"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: jenkins
  namespace: kube-ops
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: jenkins
subjects:
  - kind: ServiceAccount
    name: jenkins
    namespace: kube-ops
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: jenkins
  namespace: kube-ops
spec:
  selector:
    matchLabels:
      app: jenkins
  template:
    metadata:
      labels:
        app: jenkins
    spec:
      serviceAccount: jenkins
      initContainers:
        - name: fix-permissions
          image: busybox:1.35.0
          command: ["sh", "-c", "chown -R 1000:1000 /var/jenkins_home"]
          securityContext:
            privileged: true
          volumeMounts:
            - name: jenkinshome
              mountPath: /var/jenkins_home
      containers:
        - name: jenkins
          image: jenkins/jenkins:2.390-jdk11
          imagePullPolicy: IfNotPresent
          env:
          - name: JAVA_OPTS
            value: -Dhudson.model.DownloadService.noSignatureCheck=true
          ports:
            - containerPort: 8080
              name: web
              protocol: TCP
            - containerPort: 50000
              name: agent
              protocol: TCP
          resources:
            limits:
              cpu: 1500m
              memory: 2048Mi
            requests:
              cpu: 1500m
              memory: 2048Mi
          readinessProbe:
            httpGet:
              path: /login
              port: 8080
            initialDelaySeconds: 60
            timeoutSeconds: 5
            failureThreshold: 12
          volumeMounts:
            - name: jenkinshome
              mountPath: /var/jenkins_home
      volumes:
        - name: jenkinshome
          persistentVolumeClaim:
            claimName: jenkins-pvc
---
apiVersion: v1
kind: Service
metadata:
  name: jenkins
  namespace: kube-ops
  labels:
    app: jenkins
spec:
  type: NodePort
  selector:
    app: jenkins
  ports:
    - name: web
      port: 8080
      targetPort: web
      nodePort: 32080
    - name: agent
      port: 50000
      targetPort: agent
      nodePort: 32050
```

由于我们这里使用的镜像内部运行的用户 `uid=1000`，所以我们这里挂载出来后会出现权限问题，为解决这个问题，我们同样还是用一个简单的 `initContainer` 来修改下我们挂载的数据目录。

我们直接来创建 jenkins 的资源清单即可：

```shell
$ kubectl apply -f jenkins-pv.yaml
$ kubectl apply -f jenkins.yaml
$ kubectl get all -n kube-ops
NAME                          READY   STATUS    RESTARTS   AGE
pod/jenkins-95db8c7dc-d9hrq   1/1     Running   0          7m52s

NAME              TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)                          AGE
service/jenkins   NodePort   192.168.59.29   <none>        8080:32080/TCP,50000:32050/TCP   7m52s

NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/jenkins   1/1     1            1           7m52s

NAME                                DESIRED   CURRENT   READY   AGE
replicaset.apps/jenkins-95db8c7dc   1         1         1       7m52s
```

然后可以执行下面的命令获取解锁的管理员密码：

```sh
$ kubectl exec -it pod/jenkins-95db8c7dc-d9hrq  -n kube-ops -- cat /var/jenkins_home/secrets/initialAdminPassword
Defaulted container "jenkins" out of: jenkins, fix-permissions (init)
b9a201f71e67434f828c0fe59a94e460  # jenkins启动日志里面也有
```

使用Node+NodePort端口进行访问，使用初始密码登录。

安装插件配置然后基本环境其他操作与docker部署一致。

### 5. 架构

Jenkins 安装完成了，接下来我们不用急着就去使用，我们要了解下在 Kubernetes 环境下面使用 Jenkins 有什么好处。

我们知道持续构建与发布是我们日常工作中必不可少的一个步骤，目前大多公司都采用 Jenkins 集群来搭建符合需求的 CI/CD 流程，然而传统的 Jenkins Slave 一主多从方式会存在一些痛点，比如：

- 主 Master 发生单点故障时，整个流程都不可用了
- 每个 Slave 的配置环境不一样，来完成不同语言的编译打包等操作，但是这些差异化的配置导致管理起来非常不方便，维护起来也是比较费劲
- 资源分配不均衡，有的 Slave 要运行的 job 出现排队等待，而有的 Slave 处于空闲状态
- 资源有浪费，每台 Slave 可能是物理机或者虚拟机，当 Slave 处于空闲状态时，也不会完全释放掉资源。

正因为上面的这些种种痛点，我们渴望一种更高效更可靠的方式来完成这个 CI/CD 流程，而 Docker 虚拟化容器技术能很好的解决这个痛点，又特别是在 Kubernetes 集群环境下面能够更好来解决上面的问题，下图是基于 Kubernetes 搭建 Jenkins 集群的简单示意图：

<img src="./assets/6-10.png" style="zoom:50%;" />

从图上可以看到 `Jenkins Master` 和 `Jenkins Slave` 以 Pod 形式运行在 Kubernetes 集群的 Node 上，Master 运行在其中一个节点，并且将其配置数据存储到一个 Volume 上去，Slave 运行在各个节点上，并且它不是一直处于运行状态，它会按照需求动态的创建并自动删除。

这种方式的工作流程大致为：当 Jenkins Master 接受到 Build 请求时，会根据配置的 Label 动态创建一个运行在 Pod 中的 Jenkins Slave 并注册到 Master 上，当运行完 Job 后，这个 Slave 会被注销并且这个 Pod 也会自动删除，恢复到最初状态。

那么我们使用这种方式带来了哪些好处呢？

- **服务高可用**，当 Jenkins Master 出现故障时，Kubernetes 会自动创建一个新的 Jenkins Master 容器，并且将 Volume 分配给新创建的容器，保证数据不丢失，从而达到集群服务高可用。
- **动态伸缩**，合理使用资源，每次运行 Job 时，会自动创建一个 Jenkins Slave，Job 完成后，Slave 自动注销并删除容器，资源自动释放，而且 Kubernetes 会根据每个资源的使用情况，动态分配 Slave 到空闲的节点上创建，降低出现因某节点资源利用率高，还排队等待在该节点的情况。
- **扩展性好**，当 Kubernetes 集群的资源严重不足而导致 Job 排队等待时，可以很容易的添加一个 Kubernetes Node 到集群中，从而实现扩展。 是不是以前我们面临的种种问题在 Kubernetes 集群环境下面是不是都没有了啊？看上去非常完美。

#### 1. 配置

接下来我们就需要来配置 Jenkins，让他能够动态的生成 Slave 的 Pod。

第 1 步. 我们需要安装 [kubernetes 插件](https://github.com/jenkinsci/kubernetes-plugin)， 点击 Manage Jenkins -> Manage Plugins -> Available -> Kubernetes 勾选安装即可。

第 2 步. 安装完毕后，进入configureClouds页面：

![](./assets/6-11.png){: .zoom}

在该页面我们可以点击 `Add a new cloud` -> 选择 `Kubernetes`，首先点击 `Kubernetes Cloud details...` 按钮进行配置。

首先配置连接 Kubernetes APIServer 的地址，由于我们的 Jenkins 运行在 Kubernetes 集群中，所以可以使用 Service 的 DNS 形式进行连接 `https://kubernetes.default.svc.cluster.local`：

注意 namespace，我们这里填 kube-ops，凭据输入KUBECONFIG文件对应的凭证，然后点击 `Test Connection`，如果出现 `Connected to Kubernetes...` 的提示信息证明 Jenkins 已经可以和 Kubernetes 系统正常通信了。

![](./assets/6-12.png){: .zoom}

然后下方的 Jenkins URL 地址：`http://jenkinsci.kube-ops.svc.cluster.local:8080`，这里的格式为：`服务名.namespace.svc.cluster.local:8080`，根据上面创建的 jenkins 的服务名填写，包括下面的Jenkins通道，默认是50000端口（要注意是 TCP，所以不要填写 http）。

配置jenkins-slave镜像。

```shell
[root@k8s-master01 jenkins-slave]# tree -a
.
├── bin
    ├── docker
    ├── helm
    └── kubectl
├── config
    └── config
├── Dockerfile
└── .ssh
    ├── authorized_keys
    ├── id_rsa
    ├── id_rsa.pub
    └── known_hosts
3 directories, 9 files

# cat Dockerfile
FROM jenkins/jnlp-slave:4.13.3-1-jdk11
MAINTAINER hjl@94qq.com

USER root

ENV JENKINS_AGENT_WORKDIR=/home/jenkins/agent

RUN set -eux; \
    sed -i 's#http://deb.debian.org#https://mirrors.163.com#g' /etc/apt/sources.list \
    && apt update \
    && apt install -y python  \
        sshpass  \
        rsync \
        make  \
        gcc  \
        curl  \
        net-tools  \
        wget  \
        --no-install-recommends \
    && apt-get -y autoremove --purge \
    && rm -rf /var/lib/apt/lists/* \
    && mkdir -p /home/jenkins/.ssh

ENV PATH=/opt/java/openjdk/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/home/jenkins/bin

COPY --chown=root:root bin /home/jenkins/bin
COPY --chown=root:root .ssh /root/.ssh

RUN set -eux ; \
    chmod 755 /home/jenkins/bin/*
    
# docker build -t hub.gitee.cc/kubernetes/jenkins-salve:latet-jdk11 .
# docker push hub.gitee.cc/kubernetes/jenkins-salve:latet-jdk11
The push refers to repository [hub.gitee.cc/kubernetes/jenkins-salve]
f861fb1f9735: Pushing [===>                                               ]  11.11MB/150.2MB
.....
```

第 3 步. 点击最下方的 `Pod Templates` 按钮用于配置 Jenkins Slave 运行的 Pod 模板，命名空间我们同样是用 kube-ops，Labels 这里也非常重要，对于后面执行 Job 的时候需要用到该值。

![](./assets/6-13-1715700667821-1.png){: .zoom}

配置的Slave Pod中的Label名称

![](./assets/6-14.png){: .zoom}

[Jenkins 添加 K8S 集群](https://www.cnblogs.com/evescn/p/17573472.html)


##### 1.1 docker in pod(推荐)

**配置容器模板**：jenkins-salve模板容器配置

![](./assets/6-15.png){: .zoom}

> "注意" ：容器的名称必须是 `jnlp`，这是默认拉起的容器，另外需要将 `运行的命令` 和 `命令参数` 的值都删除掉，否则会失败。

由于jnlp容器中只是 docker cli，需要 docker daemon 才能正常使用，我们通常情况下的做法是将宿主机上的 docker sock 文件 `/var/run/docker.sock` 挂载到容器中，但是我们现在的 Kubernetes 集群使用的是 containerd 这种容器运行时，节点上没有 docker daemon。我们可以单独以 Pod 的形式在集群中跑一个 docker daemon 的服务，对应的资源清单如下所示：

**生产环境将docker-dind部署固定在性能好的服务器上，打上tag**，因为`replicas: 1`，在应对多任务时会有点慢，所有如下的pod可以多部署几份。在Jenkinsfile中指定对应的server名称，调度对应的pod进行编译即可。

**配置hostPath存储模式**

`docker-dind-1.yml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: docker-dind-1
  namespace: kube-ops
  labels:
    name: docker-dind-1
spec:
  replicas: 1
  selector:
    matchLabels:
      name: docker-dind-1
  template:
    metadata:
      labels:
        name: docker-dind-1
    spec:
      containers:
      - name: dind
        image: docker:20.10.12-dind
        args:
        - --insecure-registry=$(REGISTRY)
        - --registry-mirror=https://25bxwt20.mirror.aliyuncs.com
        env:
        - name: REGISTRY
          value: http://hub.gitee.cc
        - name: DOCKER_TLS_CERTDIR
          value: ""
        - name: DOCKER_HOST
          value: tcp://0.0.0.0:2375
        securityContext:
          privileged: true
        tty: true
        resources:
          requests:
            cpu: 1000m
            memory: 500Mi
          limits:
            cpu: 4096m
            memory: 8000Mi
        readinessProbe:
          exec:
            command: ["docker", "info"]
          initialDelaySeconds: 60
          failureThreshold: 10
        livenessProbe:
          exec:
            command: ["docker", "info"]
          initialDelaySeconds: 60
          failureThreshold: 10
        volumeMounts:
        - name: docker-storage
          mountPath:  /var/lib/docker
        - name: localtime
          mountPath: /etc/localtime
      volumes:
      - name: docker-storage
        hostPath:
          path: /var/lib/docker
      - name: localtime
        hostPath:
          path: /usr/share/zoneinfo/Asia/Shanghai
      nodeSelector:
        docker-dind-1: "true"
---
kind: Service
apiVersion: v1
metadata:
  name: docker-dind-1
  namespace: kube-ops
spec:
  selector:
    name: docker-dind-1
  ports:
  - protocol: TCP
    port: 2375
    targetPort: 2375
```

**配置PVC nfs存储模式**

```yaml
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: docker-dind-pv
  labels:
    app: docker-dind-pv
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: gitee-nfs-client
  mountOptions:
    - hard
    - nfsvers=4.1
  nfs:
    server: 192.168.1.60
    path: /QaTest/docker-dind
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  labels:
    app: docker-dind
  name: docker-dind-data
  namespace: kube-ops
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: gitee-nfs-client
  resources:
    requests:
      storage: 10Gi
  selector:
    matchLabels:
      app: docker-dind-pv
      
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: docker-dind
  namespace: kube-ops
  labels:
    name: docker-dind
spec:
  replicas: 1
  selector:
    matchLabels:
      name: docker-dind
  template:
    metadata:
      labels:
        name: docker-dind
    spec:
      containers:
      - name: dind
        image: docker:20.10.12-dind
        args:
        - --insecure-registry=$(REGISTRY)
        - --registry-mirror=https://25bxwt20.mirror.aliyuncs.com
        env:
        - name: REGISTRY
          value: http://hub.gitee.cc
        - name: DOCKER_TLS_CERTDIR
          value: ""
        - name: DOCKER_HOST
          value: tcp://0.0.0.0:2375
        securityContext:
          privileged: true
        tty: true
        resources:
          requests:
            cpu: 1000m
            memory: 500Mi
          limits:
            cpu: 4096m
            memory: 8000Mi
        readinessProbe:
          exec:
            command: ["docker", "info"]
          initialDelaySeconds: 60
          failureThreshold: 10
        livenessProbe:
          exec:
            command: ["docker", "info"]
          initialDelaySeconds: 60
          failureThreshold: 10

        volumeMounts:
          - name: docker-dind-data-vol
            mountPath: /var/lib/docker/
      volumes:
        - name: docker-dind-data-vol
          persistentVolumeClaim:
            claimName: docker-dind-data
      nodeSelector:
        build: "true"
---
kind: Service
apiVersion: v1
metadata:
  name: docker-dind
  namespace: kube-ops
spec:
  selector:
    name: docker-dind
  ports:
  - protocol: TCP
    port: 2375
    targetPort: 2375
```

直接创建上面的资源对象即可：

```shell
$ kubectl apply -f docker-dind.yaml
$ kubectl get pods -n kube-ops -l app=docker-dind
NAME                           READY   STATUS    RESTARTS   AGE
docker-dind-86d95bfc49-wq248   1/1     Running   0          17s
$ kubectl get svc -n kube-ops -l app=docker-dind
NAME          TYPE        CLUSTER-IP        EXTERNAL-IP   PORT(S)    AGE
docker-dind   ClusterIP   192.168.172.137   <none>        2375/TCP   6m3s
```

> "扩展参考文献" ：[github-jenkins-agent-docker](https://github.com/jenkinsci/docker-inbound-agent)
>
> [自定义Jenkins Agent集成docker和kubectl工具](https://wnote.com/post/cicd-jenkins-agent-docker-kubectl-helm/)
>
> [kubernetes环境下部署Jenkins](https://wnote.com/post/cicd-jenkins-in-kubernetes/#back-to-top)
>
> [Jenkins 基于 Kubernetes 的动态和静态节点](https://mp.weixin.qq.com/s/ndy3qKEuokX-Oi8e-rSwPw)
>
> [流水线中使用 docker in pod 方式构建容器镜像](https://blog.csdn.net/alex_yangchuansheng/article/details/123564347)
>
> [基于K8s插件版的Jenkins动态节点实践【内含最佳实践】1](https://mp.weixin.qq.com/s/qCYCDWXFMnC8flYiTioVPg)
>
> [基于K8s插件版的Jenkins动态节点实践【内含最佳实践】2](https://blog.51cto.com/devopsvip/5521288)



##### 1.2 物理机build(推荐)

选择一台有SSD性能好的物理服务器作为编译镜像打包服务器。


在命令行上来创建 Secret提供私有仓库拉取凭证。这样`hub.gitee.cc/kubernetes/jenkins-salve:latet-jdk11`私有镜像就能正常拉取了。

```sh
$ kubectl -n kube-ops create secret docker-registry regcred \
--docker-server=http://hub.gitee.cc \
--docker-username=admin \
--docker-password=oschina123 \
--docker-email=1@1.com
```

> 参考：https://kubernetes.io/zh-cn/docs/tasks/configure-pod-container/pull-image-private-registry/

在图形界面中设置拉取凭证信息。

![](./assets/6-16.png){: .zoom}

> "注意"：如果读者的Kubernetes集群采用的是Containerd作为Runtime，配置insecure-registry只需要在Containerd配置文件的mirrors下添加自己的镜像仓库地址即可

**vim /etc/containerd/config.toml**

```
    [plugins."io.containerd.grpc.v1.cri".registry]
      config_path = ""
      [plugins."io.containerd.grpc.v1.cri".registry.auths]
      [plugins."io.containerd.grpc.v1.cri".registry.configs]
        [plugins."io.containerd.grpc.v1.cri".registry.configs."hub.gitee.cc".tls]
          insecure_skip_verify = true
        [plugins."io.containerd.grpc.v1.cri".registry.configs."hub.gitee.cc".auth]
      [plugins."io.containerd.grpc.v1.cri".registry.headers]

      [plugins."io.containerd.grpc.v1.cri".registry.mirrors]
        [plugins."io.containerd.grpc.v1.cri".registry.mirrors."docker.io"]
          endpoint = ["https://pooj3a7i.mirror.aliyuncs.com"]
        [plugins."io.containerd.grpc.v1.cri".registry.mirrors."hub.gitee.cc"]
          endpoint = ["http://hub.gitee.cc"]
```

配置完成后，重启Containerd，之后进行pull测试：

```sh
$ systemctl restart containerd
$ ctr -n k8s.io image pull hub.gitee.cc/kubernetes/webkubectl:latest --plain-http --user admin:oschina123 
```

然后我们可以通过设置环境变量 `DOCKER_HOST: tcp://docker-dind:2375` 去连接 docker dind 服务。


另外需要将文件 `/root/.kube/config` 挂载到容器的 `/root/.kube/` 目录下面，这是为了让我们能够在 Pod 的容器中能够使用 `kubectl` 工具来访问我们的Kubernetes集群，方便我们后面在 `Slave Pod` 部署 Kubernetes 应用。

首先在master上创建cm文件，cm文件位于kube-ops名称空间下：

```sh
$ kubectl create configmap kubeconfig --from-file=/root/.kube/config -n kube-ops
$ k get cm -n kube-ops
NAME               DATA   AGE
kube-root-ca.crt   1      2d
kubeconfig         1      6m4s
```

![](./assets/6-17.png){: .zoom}

另外如果在配置了后运行 Slave Pod 的时候出现了权限问题，这是因为Jenkins Slave Pod中没有配置权限，所以需要配置上 ServiceAccount，在Slave Pod配置的地方点击下面的高级，添加上对应的 ServiceAccount 即可：

<img src="./assets/6-18.png" style="zoom:50%;" />

到这里我们的 Kubernetes 插件就算配置完成了，记得保存。

#### 2. 测试

Kubernetes 插件的配置工作完成了，接下来我们就来添加一个Job任务，看是否能够在Slave Pod中执行，任务执行完成后看Pod是否会被销毁。

##### 2.1 自由风格的流水线

在 Jenkins 首页点击 `新建任务`，创建一个测试的任务，输入任务名称，然后我们选择 `构建一个自由风格的软件项目` 类型的任务，注意在下面的 `Label Expression` 这里要填入 `jenkins-jnlp`，就是前面我们配置的 Slave Pod 中的 Label，这两个地方必须保持一致：

![](./assets/6-19.png){: .zoom}

然后往下拉，在 `构建` 区域选择 `执行 shell`：

<img src="./assets/6-20.png" style="zoom: 33%;" />

然后输入我们测试命令

```shell
echo "测试 Kubernetes 动态生成 jenkins slave"
echo "==============docker in docker==========="
docker version

echo "=============kubectl============="
kubectl get pods
```

最后点击保存。

现在我们直接在页面点击左侧的 `立即构建` 触发构建即可，然后观察 Kubernetes 集群中 Pod 的变化：

```shell
$ kubectl get pods -n kube-ops
NAME                          READY   STATUS              RESTARTS   AGE
jenkins-556cd59c8c-2vl8m    1/1     Running             0          64m
jenkins-agent-rb8hk           0/1     ContainerCreating   0          8s
```

我们可以看到在我们点击立刻构建的时候可以看到一个新的 Pod：`jenkins-agent-rb8hk` 被创建了，这就是我们的 Jenkins Slave。任务执行完成后我们可以看到任务信息:

到这里证明我们的任务已经构建完成，然后这个时候我们再去集群查看我们的 Pod 列表，发现 kube-ops 这个 namespace 下面已经没有之前的 Slave 这个 Pod 了。

```shell
$ kubectl get pods -n kube-ops
NAME                                           READY   STATUS              RESTARTS   AGE
jenkins-556cd59c8c-2vl8m                       1/1     Running             0          12h
```

到这里我们就完成了使用 Kubernetes 动态生成 Jenkins Slave 的方法。

##### 2.2 pipeline流水线

![](./assets/6-21.png){: .zoom}



使用物理机用于build。物理机可以安装负载均衡、反向代理（haproxy、nginx）软件，制作一个docker build 分发池，分摊build构建压力。

```
pipeline{
    environment {
        VERSION = "v5.0.0"
        RELEASE_VERSION = "master"
        GIT_ADDRESS = "https://gitee.com/oschina/gitee.git"

    }
    options {
      parallelsAlwaysFailFast()
      disableResume()
      skipDefaultCheckout()
      quietPeriod(1)
      preserveStashes(buildCount: 5)
      timestamps()
      buildDiscarder(logRotator(numToKeepStr: '100'))
      timeout(time: 60, unit: 'MINUTES')
    }
    agent {
        kubernetes {
            cloud 'kubernetes'
            slaveConnectTimeout 1200
            workspaceVolume nfsWorkspaceVolume(serverAddress: "192.168.1.60", serverPath: "/QaTest/jenkins-salve", readOnly: "false")
            yaml '''

---
apiVersion: "v1"
kind: "Pod"
metadata:
  labels:
    jenkins: "slave"
    jenkins/label: "jenkins-jnlp"
  name: "jenkins-slave"
  namespace: "kube-ops"
spec:
  containers:
  - env:
    - name: "DOCKER_HOST"
      value: "tcp://192.168.1.141:2375"
    - name: "JENKINS_AGENT_WORKDIR"
      value: "/home/jenkins/agent"
    image: "hub.gitee.cc/kubernetes/jenkins-salve:latet-jdk11"
    imagePullPolicy: "IfNotPresent"
    name: "jnlp"
    tty: true
    resources:
      limits:
        cpu: 1000m
        memory: 2000Mi
      requests:
        cpu: 200m
        memory: 400Mi
    volumeMounts:
    - mountPath: "/root/.kube/config"
      name: "volume-0"
      readOnly: false
      subPath: "config"
    - name: localtime
      mountPath: /etc/localtime
    - mountPath: "/home/jenkins/agent"
      name: "workspace-volume"
      readOnly: false
    workingDir: "/home/jenkins/agent"
  hostNetwork: false
  imagePullSecrets:
  - name: "regcred"
  nodeSelector:
    kubernetes.io/os: "linux"
  restartPolicy: "Never"
  volumes:
  - name: localtime
    hostPath:
      path: /usr/share/zoneinfo/Asia/Shanghai
  - configMap:
      name: "kubeconfig"
      optional: false
    name: "volume-0"
  - emptyDir:
      medium: ""
    name: "workspace-volume"
            '''
        }
    }
    stages {
      stage('Build') {
        steps {
		  sh '''#!/bin/bash
				set -eu
				echo 'Build'
				echo
				docker version
				echo
			 '''
        }
      }
      stage('Test') {
        steps {
          echo 'Test'
        }
      }
      stage('Deploy') {
        steps {
			sh '''#!/bin/bash
				set -eu
				echo 'Deploy'
				echo
				kubectl get nodes
				echo
			 '''
      }
    }
  }
}
```

## 5.2  安装GitLab

GitLab在企业内经常用于代码的版本控制，是DevOps平台中尤为重要的一个工具。接下来在另一台服务器（40GB以上）上安装GitLab（如果有可用的GitLab，也无须安装）。

### 1. 服务方式部署(推荐)

首先在GitLab国内源下载GitLab的安装包：https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el7/。
将下载后的RPM包上传到服务器，之后通过yum直接安装即可：

```sh
$ mkdir gitlab
$ cd gitlab/
$ wget -c https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el7/gitlab-ce-15.8.2-ce.0.el7.x86_64.rpm
$ yum install gitlab-ce-14.2.3-ce.0.el7.x86_64.rpm -y
```

安装完成后，需要更改几处配置：

```
vim /etc/gitlab/gitlab.rb
```

将external_url更改为自己的发布地址，可以是服务器的IP，也可以是一个可被解析的域名，更改完成后需要重新加载配置文件：

```
# gitlab-ctl reconfigure
```

之后可以通过浏览器访问GitLab ， 账号为root ，默认密码在/etc/gitlab/initial_root_password。

### 2. docker方式部署(推荐)

gitlab-ce 有两个不同的容器化镜像，sameersbn/gitlab 和 gitlab/gitlab-ce， 前者是第三方维护的，用户广泛，年代久远，通过注入环境变量来自动修改配置文件，就是版本一般会落后官方。后者是官方的镜像，更新及时。

####  2.1 sameersbn/gitlab 

> "仓库地址"：
>
> https://github.com/sameersbn/docker-gitlab
>
> https://www.jianshu.com/p/abd406052677
> https://www.jianshu.com/p/ee0c8ed4795e

#### 2.2 gitlab/gitlab-ce

创建安装目录、拉取镜像

```sh
$ docker pull gitlab/gitlab-ce
$ mkdir -p /root/wokerdir/gitlab
```

`docker-compose.yaml`

```yaml
version: '3.1'
services:
  gitlab:
    image: 'gitlab/gitlab-ce:latest'
    container_name: gitlab
    restart: always
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        external_url 'http://gitlab.runjs.cn:8929'
        gitlab_rails['gitlab_shell_ssh_port'] = 2224
    ports:
      - '8929:8929'
      - '2224:22'
    volumes:
      - './config:/etc/gitlab'
      - './logs:/var/log/gitlab'
      - './data:/var/opt/gitlab'
    shm_size: '256m'
```

> 1. 指定三个端口,22为ssh端口,80为http端口,443为https端口,分别映射到宿主机的7022、8080和7443端口
> 2. 通过--volume指定目录映射,其中
>
> - /etc/gitlab表示gitlab的配置目录,映射到宿主机的/opt/gitlab/config目录.
> - /var/log/gitlab表示gitlab的日志目录,映射到宿主机的/opt/gitlab/logs目录.
> - /var/opt/gitlab表示gitlab的数据目录,映射到宿主机的/opt/gitlab/data目录.
>
> 1. 目录说明
>
> - /opt/gitlab/config 存储 GitLab 配置信息
> - /opt/gitlab/data 存储数据库
> - /opt/gitlab/logs 存储日志

```shell
$ docker-compose up -d
$ docker restart gitlab		#重启gitlab容器
```


启动容器（需要稍等一小会……）



**重启配置服务（出现问题）** 

由于我们运行是使用数据卷参数进行运行的，宿主机的gitlab.rb文件修改了，gitlab的文件会跟着改，但是容器的文件不会跟着生效，必须要进去容器里面进行命令执行，重置配置文件比较耗费时间，需要耐心等待，如果时间比较短说明成功率不高，而且进去容器之后就退出啦。

```shell
$ docker exec -it gitlab /bin/bash  # 进去gitlab容器的命令
$ gitlab-ctl reconfigure  		# 重置gitlab客户端的命令
$ gitlab-ctrl restart
```

查看root登录密码

```shell
$ docker exec  $(docker ps | grep gitlab | awk '{print $1}') grep 'Password:' /etc/gitlab/initial_root_password
Password: lB7RIXsS41R7on31KPHAzJ0FE736mBMPxTdjOt4g2Lo=
```

**重置管理员密码**

```sh
root@docker-node141:~/gitlab# docker exec -it gitlab /bin/bash
root@192:/# gitlab-rails console -e production
--------------------------------------------------------------------------------
 Ruby:         ruby 2.7.5p203 (2021-11-24 revision f69aeb8314) [x86_64-linux]
 GitLab:       14.6.1 (661d663ab2b) FOSS
 GitLab Shell: 13.22.1
 PostgreSQL:   12.7
--------------------------------------------------------------------------------
Loading production environment (Rails 6.1.4.1)
irb(main):001:0> user = User.where(id: 1).first
=> #<User id:1 @root>
irb(main):002:0> user.password = 'oschina123'
=> "oschina123"
irb(main):003:0> user.password_confirmation = 'oschina123'
=> "oschina123"
irb(main):004:0> user.save!
=> true
irb(main):005:0> exit
```

**常用命令**

```sh
# 进入容器
docker exec -it gitlab /bin/bash

# 启动所有gitlab组件
gitlab-ctl start

# 查看gitlab状态
gitlab-ctl status

# 重新启用配置
gitlab-ctl reconfigure

# 重启gitlab
docker restart gitlab
```

登录后，可以创建一个测试项目，进行一些简单的测试。首先创建一个组。组名为kubernetes，类型为Private，之后单击Create group

![](./assets/6-22.png){: .zoom}

选择创建一个空的项目，输入项目名称test-project，然后单击Create project如图
![](./assets/6-23.png){: .zoom}


之后可以将Jenkins服务器的key导入GitLab，首先生成密钥（如有可以不生成）：

```shell
$ ssh-keygen -t rsa -C "YOUR_EMAIL@ADDRESS.COM"
```

将公钥的内容放在GitLab中：

```shell
$ cat ~/.ssh/id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDAipO4jkJioJBW57Nv25i95yAiWNCeaySwvtFViOqtcX644WbN81SDwMscYCvKNcdgPmXkzBaKWBzehAVZ9RhVN1gqp9/BvnGzhyz6IpYXMgQfsDLIakARM4uJjxKt9LCTVugUL3kA5p+fsNsaMwpSy/+scnfUsuKCTqDuCBBGbdRkBIjC2ZuXzXs2ei2jWwVavJbxSZwMxPT2pNeHG7GxiyIBVwR6kbdDQNqzDBOBSATKSFp8XKhdc+7Mu3f6cUD64i+TxBgqqG1rA29yb4xn0iB1IHxsOCc5RO4WrfwrYQxMsMiB2cxQK2sKhrNrggeQ4LQhsWRWQWxTaDEB5TbvMsdKoEAivGE8OjUyyz/3VTPFqMtLEeAB5RDjOwFKeJBB6aYNOqOYlI7nm1g9oLm4OtiSjeSxz2MvO0N4KPMBgX6qW5lecYbD6VrJC2RVic05NouViRWoJPJEIUn9c3wNd0lDYMYhDVAOfW4PPx0B6gf1b40+aTlUxoUPLAhTocs= 1879324764@qq.com
```

在GitLab找到Profile，之后在SSH Keys添加公钥，如图
![](./assets/6-24.png){: .zoom}


创建几个文件，然后提交测试：

```sh
$ git clone ssh://git@gitlab.runjs.cn:7022/kubernetes/test-project.git
$ cd test-project/
$ touch aaa{1..3}.py
root@docker-node141:~/gitlab/test-project# echo 'aaa' > aaa1.py
root@docker-node141:~/gitlab/test-project# git add .
root@docker-node141:~/gitlab/test-project# git config --global user.email "18793247642qq.com"
root@docker-node141:~/gitlab/test-project# git config --global user.name "hujianli"
root@docker-node141:~/gitlab/test-project# git commit -m "first commit"
[main 515ab18] first commit
 3 files changed, 1 insertion(+)
 create mode 100644 aaa1.py
 create mode 100644 aaa2.py
 create mode 100644 aaa3.py
 
root@docker-node141:~/gitlab/test-project# git push origin main
Warning: Permanently added '[gitlab.runjs.cn]:7022,[192.168.1.141]:7022' (ECDSA) to the list of known hosts.
Enumerating objects: 5, done.
Counting objects: 100% (5/5), done.
Delta compression using up to 8 threads
Compressing objects: 100% (2/2), done.
Writing objects: 100% (4/4), 313 bytes | 313.00 KiB/s, done.
Total 4 (delta 0), reused 0 (delta 0)
To ssh://gitlab.runjs.cn:7022/kubernetes/test-project.git
   b77d308..515ab18  main -> main
```

> "参考文献"：https://www.cnblogs.com/linanjie/p/13932352.html
>
> https://wtl4it.blog.csdn.net/article/details/125019372
>
> [企业级GitLab在Docker部署使用](https://www.cnblogs.com/wei325/p/17106525.html)
>
> ["官方文档"](https://docs.gitlab.com/ee/install/docker.html)

### 3. yaml方式部署(不推荐)

> "参考文献"：https://www.qikqiak.com/k8strain2/devops/gitlab/

### 4. helm3安装gitlab(不推荐)

> "参考文献"：https://blog.51cto.com/u_14268033/2456991

## 5.3 安装Harbor

在使用容器或者Kubernetes发布服务时，都是以镜像为基准发布的，而镜像的存储需要镜像仓库来支持，目前比较常用的有Harbor、Nexus、JFrog Artifactory等。

本小节将演示Harbor的安装和使用，这也是企业内比较常用的镜像仓库。

>"注意"：推荐使用docker-compose的方式部署，节省kubernetes资源，单独镜像仓库环境

#### 1. docker方式部署(推荐)

首先在GitHub下载新的Harbor离线包，并上传至Harbor服务器，官方下载地址：https://github.com/goharbor/harbor/releases/。
由于Harbor是采用docker-compose一键部署的，因此Harbor服务器也需要安装Docker

> Harbor 高可用搭建(docker-compose) [参考文献](https://clay-wangzhi.com/cloudnative/kubernetes/docker-harbor-ha.html#summary)

Harbor 是一个主流的镜像仓库系统，在 v1.6 版本以后的 harbor 中新增加了 helm charts 的管理功能，可以存储Chart文件。

>"参考文献"：部署仓库地址：https://gitee.com/autom-studio/simple-harbor

离线包：harbor-offline-installer-v240.tgz


```shell
$ wget https://github.com/goharbor/harbor/releases/download/v2.4.2/harbor-online-installer-v2.4.2.tgz

# 上传安装包到服务器/root/wokerdir/harbor目录下
$ cd /root/wokerdir/harbor
$ tar -zxf harbor-online-installer-v2.4.2.tgz
$ cd /root/wokerdir/harbor/harbor

# 复制 harbor 配置模板
$ cp harbor.yml.tmpl harbor.yml
```

解压并修改配置文件，只需修改 harbor.yml ，修改项如下：

```yaml
hostname: hub.gitee.cc  # 访问域名或IP地址

# 如果没有ssl证书，则注释掉 https 字段
harbor_admin_password: admin123    # 初始管理员密码
data_volume: /data/harbor    # 镜像文件存储路径，需要指定一个空文件夹且空间足够

external_url: http://hub.gitee.cc
```

 执行安装命令

```shell
$ sudo ./install.sh --with-chartmuseum			# 同时开启 helm chart 仓库支持
```

完成

成功启动后，即可通过配置的地址或域名访问。

如果配置不是HTTPS协议，所有的Kubernetes节点的Docker（如果是containerd作为Runtime，可以参考下文配置insecure-registry）都需要添
加insecure-registries配置：

```shell
$ cat >/etc/docker/daemon.json <<EOF
{
  "insecure-registries": ["http://hub.gitee.cc"],
  "registry-mirrors": ["https://25bxwt20.mirror.aliyuncs.com"],
  "live-restore": true,
  "default-shm-size": "128M",
  "max-concurrent-downloads": 10,
  "oom-score-adjust": -1000,
  "exec-opts": ["native.cgroupdriver=systemd"],
  "hosts": ["tcp://0.0.0.0:2375", "unix:///var/run/docker.sock"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m",
    "max-file":"3"
  }
}
EOF

$ systemctl daemon-reload
$ systemctl restart docker
```

 "部署完成后如果需要修改harbor.yml ，需要在修改后重新初始化配置，流程如下："

```sh
docker-compose down -v
vim harbor.yml
sudo ./prepare --with-chartmuseum
docker-compose up -d
```

**创建external nginx 配置文件**

"根据实际环境配置"

``` 
upstream harbor {
    server 192.168.1.188:8080;
    keepalive 32;
}

server {
    listen 80;
    server_name hub.gitee.com;
    access_log /var/log/nginx/hub_302_access.log main;
    error_log /var/log/nginx/hub_302_error.log;
    return 302 https://$server_name$request_uri;
}

server {
    include ssl.d/ssl.conf;
    server_name hub.gitee.com;
    access_log /var/log/nginx/hub_access.log main;
    error_log /var/log/nginx/hub_error.log;
    location / {
        proxy_set_header Connection "";
        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Frame-Options SAMEORIGIN;
        proxy_buffers 256 16k;
        proxy_buffer_size 16k;
        proxy_read_timeout 600s;
        proxy_cache_revalidate on;
        proxy_cache_min_uses 2;
        proxy_cache_use_stale timeout;
        proxy_cache_lock on;
        proxy_http_version 1.1;
        proxy_pass http://harbor;
    }
    error_page 500 502 503 504 /50x.html;
    location = /50x.html {
        root /usr/share/nginx/html;
    }
}
```

登录测试：

```shell
$ docker login http://hub.gitee.cc
Username: admin
Password:
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
```

接下来在Harbor上创建一个Project，如下图所示。创建一个名为kubernetes的Project

![](./assets/6-25.png){: .zoom}

之后在服务器上找任意一个镜像，修改tag为Harbor的地址，并进行push测试：

```shell
$ docker tag kubeoperator/webkubectl:latest hub.gitee.cc/kubernetes/webkubectl:latest
$ docker push hub.gitee.cc/kubernetes/webkubectl:latest
The push refers to repository [hub.gitee.cc/kubernetes/webkubectl]
99b13cfbfb0f: Pushed
0e658bc0176b: Pushed
9eba8e5bc7c8: Pushed
adc0cdf505f7: Pushed
ad4e41a7f1d3: Pushed
7ba699eb6203: Pushed
b4c5273b6d27: Pushed
e5e13b0c77cb: Mounted from library/mailcat
latest: digest: sha256:51cacad0e26ebacc59de84ba71fb50e161ff3e1b8337e39d448c91d6cd048062 size: 1990
```

之后就可以在Harbor查看该镜像，如图

![](./assets/6-26.png){: .zoom}

如果读者的Kubernetes集群采用的是Containerd作为Runtime，配置insecure-registry只需要在Containerd配置文件的mirrors下添加自己的镜像仓库地址即可。

```shell
$ vim /etc/containerd/config.toml					# Containerd配置insecure仓库
   [plugins."io.containerd.grpc.v1.cri".registry]
      config_path = ""
      [plugins."io.containerd.grpc.v1.cri".registry.auths]
      [plugins."io.containerd.grpc.v1.cri".registry.configs]
        [plugins."io.containerd.grpc.v1.cri".registry.configs."hub.gitee.cc".tls]
          insecure_skip_verify = true
        [plugins."io.containerd.grpc.v1.cri".registry.configs."hub.gitee.cc".auth]
      [plugins."io.containerd.grpc.v1.cri".registry.headers]
      [plugins."io.containerd.grpc.v1.cri".registry.mirrors]
        [plugins."io.containerd.grpc.v1.cri".registry.mirrors."docker.io"]
          endpoint = ["https://pooj3a7i.mirror.aliyuncs.com"]
        [plugins."io.containerd.grpc.v1.cri".registry.mirrors."hub.gitee.cc"]
          endpoint = ["http://hub.gitee.cc"]
.....
```

配置完成后，重启Containerd，之后进行pull测试：

```shell
$ systemctl restart containerd
$ ctr -n k8s.io image pull hub.gitee.cc/kubernetes/webkubectl:latest --plain-http --user admin:oschina123 
```

至此，用到的工具都已经安装完成。接下来需要进行一些配置。

> "参考文献"：[Harbor配合Nginx配置公网域名](https://www.cnblogs.com/zhangmingcheng/p/14167615.html)


#### 2. helm方式部署

> 前提条件：NFS 类型的 StorageClass 来做持久化存储,  StorageClass 名称`gitee-nfs-client`

Chart 仓库地址：https://helm.goharbor.io

由于 Harbor 涉及的组件太多了，所以我们这里用更加便捷的 Helm 来进行安装。首先添加 Chart 仓库地址：

```sh
$ helm repo add harbor https://helm.goharbor.io			# 添加 Chart 仓库
$ helm repo update										# 更新
```

我们使用定制的 values 文件来进行安装了，完整定制的 values 文件如下所示：

```sh
# 拉取1.10.1版本并解压
$ helm pull harbor/harbor --version 1.10.1
$ helm pull harbor/harbor --untar --version 1.10.1

$ cat > values-prod.yaml <<'EOF'
# 外网访问IP
externalURL: http://106.12.45.72:30002
harborAdminPassword: Harbor12345
logLevel: debug

expose:
  type: nodePort
  tls:
    enabled: false
  nodePort:
    name: harbor
    ports:
      http:
        port: 80
        nodePort: 30002
      https:
        port: 443
        nodePort: 30003
      notary:
        port: 4443
        nodePort: 30004
persistence:
  enabled: true
  resourcePolicy: "keep"
  persistentVolumeClaim:
    registry:
      storageClass: "gitee-nfs-client"
      # 如果是高可用的，多个副本组件需要使用 ReadWriteMany，默认为 ReadWriteOnce
      accessMode: ReadWriteMany
      size: 500Gi
    chartmuseum:
      storageClass: "gitee-nfs-client"
      accessMode: ReadWriteMany
      size: 20Gi
    jobservice:
      storageClass: "gitee-nfs-client"
      accessMode: ReadWriteMany
      size: 10Gi
    database:
      storageClass: "gitee-nfs-client"
      accessMode: ReadWriteMany
      size: 20Gi
    redis:
      storageClass: "gitee-nfs-client"
      accessMode: ReadWriteMany
      size: 8Gi
    trivy:
      storageClass: "gitee-nfs-client"
      accessMode: ReadWriteMany
      size: 10Gi

# 使用Harboar内置的PostgreSQL和redis数据库
database:
  type: internal
  internal:
    serviceAccountName: ""
    image:
      repository: goharbor/harbor-db
      tag: v2.2.2
    password: "oschina123"

redis:
  type: internal
  internal:
    serviceAccountName: ""
    image:
      repository: goharbor/redis-photon
      tag: v2.2.2

portal:
  replicas: 2
core:
  replicas: 2
jobservice:
  replicas: 2
registry:
  replicas: 2
chartmuseum:
  replicas: 2
clair:
  replicas: 
notary:
  server:
    replicas: 2
  signer:
    replicas: 2
EOF

$ helm install -f values-prod.yaml harbor harbor --namespace pub-comm --create-namespace
```

正常情况下隔一会儿就可以安装成功了：

```sh
$ helm ls -n pub-comm
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                   APP VERSION
harbor          pub-comm        1               2022-10-19 09:41:55.531221406 +0800 CST deployed        harbor-1.6.2            2.2.2
$ kubectl get pods -n pub-comm -l app=harbor
harbor-harbor-chartmuseum-5d5dc4c5fc-75qx2     1/1     Running   0          3m24s
harbor-harbor-chartmuseum-5d5dc4c5fc-xsngf     1/1     Running   0          3m24s
harbor-harbor-core-5559c6b95d-wqcgb            1/1     Running   2          3m24s
harbor-harbor-core-5559c6b95d-xlxpd            1/1     Running   2          3m24s
harbor-harbor-database-0                       1/1     Running   0          3m24s
harbor-harbor-jobservice-7599dc87cd-95z8v      1/1     Running   2          3m24s
harbor-harbor-jobservice-7599dc87cd-np8jr      1/1     Running   0          3m24s
harbor-harbor-nginx-9bc874c4c-xfgrc            1/1     Running   0          3m24s
harbor-harbor-notary-server-7788bcd6c4-wbgdn   1/1     Running   3          3m24s
harbor-harbor-notary-server-7788bcd6c4-z2hxq   1/1     Running   3          3m24s
harbor-harbor-notary-signer-58894fb464-59k6z   1/1     Running   3          3m24s
harbor-harbor-notary-signer-58894fb464-vr6t6   1/1     Running   0          3m24s
harbor-harbor-portal-559c4d4bfd-kmdnx          1/1     Running   0          3m24s
harbor-harbor-portal-559c4d4bfd-v8vqx          1/1     Running   0          3m24s
harbor-harbor-redis-0                          1/1     Running   0          3m24s
harbor-harbor-registry-9b46998cf-kd969         2/2     Running   0          3m24s
harbor-harbor-registry-9b46998cf-kmjmq         2/2     Running   0          3m24s
harbor-harbor-trivy-0                          1/1     Running   0          3m23s

$ kubectl get svc -n pub-comm -l app=harbor
NAME                          TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                       AGE
harbor                        NodePort    172.24.58.77     <none>        80:30002/TCP,4443:30004/TCP   5m16s
harbor-harbor-chartmuseum     ClusterIP   172.24.99.83     <none>        80/TCP                        5m16s
harbor-harbor-core            ClusterIP   172.24.117.61    <none>        80/TCP                        5m16s
harbor-harbor-database        ClusterIP   172.24.252.137   <none>        5432/TCP                      5m16s
harbor-harbor-jobservice      ClusterIP   172.24.51.73     <none>        80/TCP                        5m16s
harbor-harbor-notary-server   ClusterIP   172.24.241.2     <none>        4443/TCP                      5m16s
harbor-harbor-notary-signer   ClusterIP   172.24.177.134   <none>        7899/TCP                      5m16s
harbor-harbor-portal          ClusterIP   172.24.192.233   <none>        80/TCP                        5m16s
harbor-harbor-redis           ClusterIP   172.24.242.178   <none>        6379/TCP                      5m16s
harbor-harbor-registry        ClusterIP   172.24.182.141   <none>        5000/TCP,8080/TCP             5m16s
harbor-harbor-trivy           ClusterIP   172.24.251.45    <none>        8080/TCP                      5m16s
```

如上状态所示表示harbor部署完成，访问使用externalURL: http://106.12.45.72:30002进行访问，用户名使用默认的 admin，密码则是上面配置的默认 `Harbor12345`。

> "参考文献"：[Harbor 安装配置(k8s)](https://clay-wangzhi.com/cloudnative/kubernetes/harbor.html)  [Kubernetes部署Harbor](https://blog.51cto.com/zhaochengsheng/5969396)

## 5.4 Jenkins凭证Credentials

在使用Jenkins、GitLab、Harbor、Kubernetes打造DevOps平台实现CI/CD时，需要涉及很多证书、账号和密码、公钥和私钥的配置，这些凭证需要放在一个统一的地方管理，并且能在整个流水线中使用，而Jenkins提供的Credentials插件可以满足以上要求，并且在流水线中也不会泄露相关的加密信息。

所以Harbor的账号和密码、GitLab的私钥、Kubernetes的证书均使用Jenkins的Credentials管理。

### 1. 配置Kubernetes证书

在安装和维护Kubernetes集群时，都是使用kubectl操作集群，而kubectl操作集群需要一个KUBECONFIG文件，我们使用Jenkins控制Kubernetes时，也是使用该文件，所以该文件需要放置在Credentials中，之后可以被Jenkins的Kubernetes插件使用。

首先需要找到集群中的KUBECONFIG，一般是kubectl节点的~/.kube/config文件，或者是KUBECONFIG环境变量所指向的文件。

接下来只需要把证书文件放置于Jenkins的Credentials中即可。首先单击Manage Jenkins，之后单击Manage Credentials。

然后单击Global的下拉选项，之后单击Add Credentials。在打开添加凭证页面时，需要将凭证类型改为Secret file，如图所示。

- File：KUBECONFIG文件或其他加密文件。
- ID：该凭证的ID。
- Description：证书的描述。

![](./assets/6-27.png){: .zoom}


添加Kubernetes的kubeconfig。只需要简单的几步， 就可以添加Kubernetes的KUBECONFIG文件至Jenkins，之后就可以使用前面讲过的Jenkins语法引用该凭证。

### 2. 配置Harbor账号和密码

对于账号、密码和Key类型的凭证，配置步骤是一致的，只是选择的凭证类型不一样。接下来通过Jenkins凭证管理Harbor的账号和密码。

在同样的位置单击Add Credentials，选择类型为Username with password，如图

![](./assets/6-28.png){: .zoom}

- Username：Harbor或者其他平台的用户名。
- Password：Harbor或者其他平台的密码。
- ID：该凭证的ID，harbor-user-passwd。
- Description：证书的描述

之后单击OK按钮保存即可，然后在流水线中即可通过类似environment这样的指令引用该凭证。

### 3. 配置GitLab Key

在搭建GitLab时，将Jenkins服务器的公钥放在GitLab中，之后Jenkins就可以通过自己的私钥登录GitLab，然后下载和上传代码。

在流水线的执行过程中，往往第一个过程就是下载代码，所以Jenkins的流水线需要有下载代码的权限。此时可以将Jenkins的私钥保存至Jenkins凭证中，之后在流水线中引用即可。

单击Add Credentials，类型选择为SSH Username with private key，如图

- Username：用户名，无强制性。
- Private Key：Jenkins服务器的私钥，一般位于~/.ssh/id_rsa

```sh
# tree -a
.
├── bin
    ├── docker 
    ├── helm
    └── kubectl
├── config
    └── config
├── Dockerfile
└── .ssh
    ├── authorized_keys
    ├── id_rsa
    ├── id_rsa.pub
    └── known_hosts
# 我们已经将Jenkins服务器的私钥放入jenkins-slave容器中
```

<img src="./assets/6-29.png" style="zoom: 50%;" />

### 4. 配置GitLab账号密码

由于我们是在一个 Slave Pod 中去进行构建，所以如果使用 SSH 的方式去访问 Gitlab 代码仓库的话就需要频繁的去更新 `SSH-KEY`，所以我们这里采用直接使用用户名和密码的形式来方式。

在 `Credentials` 区域点击添加按钮添加我们访问 Gitlab 的用户名和密码：

<img src="./assets/6-30.png" style="zoom:50%;" />

通过上述4种类型凭证的配置，就可以对Kubernetes证书、Harbor账号和密码以及GitLab Key或者账号密码进行管理。

需要说明的是，除Jenkins的凭证外，其他类型的凭证也可以在Jenkins中管理。

## 5.5 配置Agent

通常情况下，Jenkins Slave会通过Jenkins Master节点的50000端口与之通信，所以需要开启Agent的50000端口。

单击Manage Jenkins，然后单击Configure Global Security，在安全配置下方找到Agents，单击Fixed，输入50000即可，如图：



![](./assets/6-31.png){: .zoom}

实际使用时，没有必要把整个Kubernetes集群的节点都充当创建Jenkins Slave Pod的节点，可以选择任意一个或多个节点作为创建Slave Pod的节点。

接下来选择一个或多个节点作为Kubernetes创建Jenkins Slave Pod的节点，只需要配置一个标签即可，之后Jenkinsfile的agent会根据标签创建Slave Pod。假设k8s-node01作为Slave节点：

```sh
# kubectl label node k8s-node01 build=true
# kubectl get nodes k8s-node01 --show-labels
NAME         STATUS   ROLES     AGE     VERSION   LABELS
k8s-node01   Ready    ingress   7d18h   v1.25.6   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,build=true,kubernetes.io/arch=amd64,kubernetes.io/hostname=k8s-node01,kubernetes.io/os=linux,node-role.kubernetes.io/ingress=true
```

自由风格里面配置 Cloud中配置如下：

![](./assets/6-32.png){: .zoom}

pipeline流水线中配置如下

```
pipeline{
......
    agent {
        kubernetes {
            cloud 'kubernetes'
            slaveConnectTimeout 1200
            workspaceVolume nfsWorkspaceVolume(serverAddress: "192.168.1.60", serverPath: "/QaTest/jenkins-salve", readOnly: "false")
            yaml '''
---
apiVersion: "v1"
kind: "Pod"
metadata:
  labels:
    jenkins: "slave"
    jenkins/label: "jenkins-jnlp"
  name: "jenkins-slave"
  namespace: "kube-ops"
spec:
  containers:
  - env:
    - name: "DOCKER_HOST"
      value: "tcp://docker-dind:2375"
    - name: "JENKINS_AGENT_WORKDIR"
      value: "/home/jenkins/agent"
    image: "hub.gitee.cc/kubernetes/jenkins-salve:latet-jdk11"
    imagePullPolicy: "IfNotPresent"
	.....
  hostNetwork: false
  imagePullSecrets:
  - name: "regcred"
  nodeSelector:
    build: "true"
......
 }
```

读者的Kubernetes集群可能并非使用Docker作为Runtime，但是由于构建镜像时需要使用Docker，因此该节点需要安装Docker：


```sh
$ wget https://download.docker.com/linux/static/stable/aarch64/docker-20.10.9.tgz
$ tar zxf docker-20.10.9.tgz
$ cp docker/* /usr/bin
$ cat >/lib/systemd/system/docker.service <<EOF
[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
After=network-online.target firewalld.service
Wants=network-online.target
[Service]
Type=notify
ExecStart=/usr/bin/dockerd
ExecReload=/bin/kill -s HUP $MAINPID
LimitNOFILE=infinity
LimitNPROC=infinity
TimeoutStartSec=0
Delegate=yes
KillMode=process
Restart=on-failure
StartLimitBurst=3
StartLimitInterval=60s
[Install]
WantedBy=multi-user.target
EOF

$ systemctl daemon-reload
$ systemctl enable --now docker 
$ docker version

# 如果镜像仓库未配置证书，则需要配置insecure-registry：
$ cat > /etc/docker/daemon.json <<'EOF'
{
  "graph": "/data/docker",
  "storage-driver": "overlay2",
  "insecure-registries": ["hub.gitee.cc"],
  "registry-mirrors": ["https://25bxwt20.mirror.aliyuncs.com"],
  "live-restore": true,
  "bip": "172.16.200.1/24",
  "exec-opts": ["native.cgroupdriver=systemd"],
  "hosts": ["tcp://0.0.0.0:2375", "unix:///var/run/docker.sock"],
  "log-opts": {
    "max-size":"100M",
    "max-file":"3"
  }
}
EOF
$ systemctl restart docker
```

## 5.6 Jenkins配置Kubernetes多集群

本书讲解的Pipeline采用的Agent为在Kubernetes集群中创建的Pod，所以Jenkins要控制Kubernetes集群创建Pod充当Jenkins的Slave。

之前我们添加了Kubernetes的证书至Jenkins凭证，接下来在Jenkins中配置Kubernetes直接引用该证书。

首先单击Manage Jenkins，之后单击Configure Clouds，如图所示。

![](./assets/6-11.png){: .zoom}

单击Add a new cloud，选择Kubernetes之后，在Name字段输入集群的名称，一般输入可识别的名称即可，比如现在是学习环 境 的 Kubernetes ， 可 以 叫 作 kubernetes。 之后单击Kubernetes Cloud details，如图

然后在Credentials处选择之前添加的Kubernetes证书，选择后单击Test Connection，最后在凭证下方即可看到能否正常连接的结果。

![](./assets/6-12.png){: .zoom}

最后单击Save即可，添加完Kubernetes后，在Jenkinsfile的Agent中就可以选择该集群作为创建Slave的集群。

**如果想要添加多个集群，重复上述步骤即可。首先添加Kubernetes凭证，然后添加Cloud即可。**

# 6. 自动化构建Java应用

本节将演示自动化构建Java应用，使用Spring Cloud的Eureka进行演示，其他Spring Boot或者Java应用基本上都是同样的步骤。

## 6.1 部署Kubernetes应用

上面我们已经知道了如何在 Jenkins Slave 中构建任务了，那么如何来部署一个原生的 Kubernetes 应用呢？ 要部署 Kubernetes 应用，我们就得对我们之前部署应用的流程要非常熟悉才行，我们之前的流程是怎样的：

- 编写代码
- 测试
- 编写 Dockerfile
- 构建打包 Docker 镜像
- 推送 Docker 镜像到仓库
- 编写 Kubernetes YAML 文件
- 更改 YAML 文件中 Docker 镜像 TAG
- 利用 kubectl 工具部署应用

## 6.2 创建Java测试用例

本书为读者准备了一个简单的Java测试用例，用于读者的学习，可以从https://gitee.com/dukuan/spring-boot-project.git找到该项目（也可以使用公司的Java项目）。

接下来将该项目导入自己的GitLab中。首先找到之前创建的Kubernetes组，然后单击New project，如图
![](./assets/6-33.png){: .zoom}

在Git repository URL中输入示例地址，然后单击Create project，如图
![](./assets/6-34.png){: .zoom}

导入后，结果如图
<img src="./assets/6-35.png" style="zoom:33%;" />

## 6.3 定义Jenkinsfile

先将镜像上传到harbor私有仓库

```sh
# docker pull registry.cn-beijing.aliyuncs.com/citools/maven:3.5.3
# docker tag registry.cn-beijing.aliyuncs.com/citools/maven:3.5.3 hub.gitee.cc/kubernetes/maven:3.5.3
# docker push hub.gitee.cc/kubernetes/maven:3.5.3
```

Jenkinsfile定义流水线的步骤包括拉取代码、执行构建、生成镜像、更新Kubernetes资源等。

本书将Jenkinsfile放置于项目源代码一并管理，也可以单独放置于一个Git仓库进行管理。

接下来在GitLab的源代码中添加Jenkinsfile。首先单击代码首页的“+”，然后单击New file，

<img src="./assets/6-36.png" style="zoom:50%;" />

在窗口中添加如下内容：

```sh
pipeline{
	environment {	// 定义一些全局的环境变量
		COMMIT_ID = ""
		HARBOR_ADDRESS = "hub.gitee.cc"		// harbor地址
		REGISTRY_DIR = "kubernetes"			// harbor的项目目录
		IMAGE_NAME = "spring-boot-project"	// 镜像名称
		NAMESPACE = "kubernetes"			// kubernetes中的名称空间
		TAG = ""							// 镜像的分支，由BUILD_TAG+COMMIT_ID组成
		GITLAB_AUTH = "gitlab-auth"			// gitlab账号密码认证
		HARBOR_AUTH = "harbor-user-passwd"	// harbor账号密码认证
  	}
  parameters {	// 使用GitParameter插件，该字段会在jenkins页面生成一个选择分支的选项
    gitParameter(branch: '', branchFilter: 'origin/(.*)', defaultValue: 'master', description: 'Branch for build and deploy', name: 'BRANCH', quickFilterEnabled: false, selectedValue: 'NONE', sortMode: 'NONE', tagFilter: '*', type: 'PT_BRANCH')
  }
    options {
      parallelsAlwaysFailFast()
      disableResume()
      quietPeriod(1)
      preserveStashes(buildCount: 5)
      timestamps()
      buildDiscarder(logRotator(numToKeepStr: '100'))
      timeout(time: 60, unit: 'MINUTES')
    }
    agent {
        kubernetes {	// 定义kubernetes作为agent
            cloud 'kubernetes'			 // 选择的云为之前配置的名称
            slaveConnectTimeout 1200	// 存储为nfs方式持久化挂载
            workspaceVolume nfsWorkspaceVolume(serverAddress: "192.168.1.60", serverPath: "/QaTest/jenkins-salve", readOnly: "false")
            yaml '''
apiVersion: v1
kind: Pod
metadata:
  name: "jenkins-slave"
  namespace: "kube-ops"
  labels:
    jenkins: "slave"
    jenkins/label: "jenkins-jnlp"
spec:
  containers:
  - name: "jnlp"			 # jnlp容器，和jenkins主节点通信。包含kubectl命令、helm命令、docker命令
    image: "hub.gitee.cc/kubernetes/jenkins-salve:latet-jdk11"
    imagePullPolicy: "IfNotPresent"
    tty: true
    resources:
      limits:
        cpu: 1000m
        memory: 2000Mi
      requests:
        cpu: 200m
        memory: 400Mi
    env:
    - name: "DOCKER_HOST"
      value: "tcp://192.168.1.141:2375"
    - name: "JENKINS_AGENT_WORKDIR"
      value: "/home/jenkins/agent"
    volumeMounts:
    - mountPath: "/root/.kube/config"
      name: "volume-0"
      readOnly: false
      subPath: "config"
    - name: localtime
      mountPath: /etc/localtime
    - mountPath: "/home/jenkins/agent"
      name: "workspace-volume"
      readOnly: false
    workingDir: "/home/jenkins/agent"
  
  - name: "build"		# build容器，包含构建的命令，java需要maven构建，需要一个maven镜像
    image: "hub.gitee.cc/kubernetes/maven:3.5.3"
    imagePullPolicy: "IfNotPresent"
    tty: true
    volumeMounts:
      - name: localtime
        mountPath: /etc/localtime
      # Pod单独创建了一个缓存的volume，将其挂载到了maven插件的缓存目录，默认是/root/.m2
      - name:  cachedir
        mountPath: "/root/.m2/"
        readOnly: false
    command:
      - "cat"
    env:
      - name: "LANGUAGE"
        value: "en_US:en"
      - name: "LC_ALL"
        value: "en_US.UTF-8"
      - name: "LANG"
        value: "en_US.UTF-8"

  volumes:
    - name: localtime
      hostPath:
        path: /usr/share/zoneinfo/Asia/Shanghai
    - configMap:
        name: "kubeconfig"
        optional: false
      name: "volume-0"
    - name: "cachedir"			 # 缓存目录
      hostPath:
        path: "/opt/m2"
    - emptyDir:
        medium: ""
      name: "workspace-volume"
  restartPolicy: Always
  hostNetwork: false
  imagePullSecrets:
  - name: "regcred"
  nodeSelector:					# 固定节点部署
    build: "true"
            '''
        }
    }
    stages {
      stage('Pulling Code') {
		parallel {
		stage('Pulling Code by Jenkins') {
          when {
            expression {	// 若env.gitlabBranch为空，则为手动触发执行手动 stage,若不为空，之前同级的另外一个stage
              env.gitlabBranch == null	
            }
          }
          steps {
            git(changelog: true, poll: true, url: 'http://gitlab.runjs.cn:8929/kubernetes/spring-boot-project.git', branch: "${BRANCH}", credentialsId: "${GITLAB_AUTH}")
            script {
              // 定义变量用于生成镜像的Tag
              // 获取最近一次提交的Commit ID
              COMMIT_ID = sh(returnStdout: true, script: "git log -n 1 --pretty=format:'%h'").trim()
              TAG = BUILD_TAG + '-' + COMMIT_ID			// 赋值给TAG变量，后面的docker build可以获取到该TAG的值
              println "Current branch is ${BRANCH}, Commit ID is ${COMMIT_ID}, Image TAG is ${TAG}"
            }
          }
        }
        stage('Pulling Code by trigger') {
          when {
            expression {
			  // 若env.gitlabBranch不为空，则为webook触发的执行webook stage。上面的stage不执行，此时BRANCH变量为空
              env.gitlabBranch != null
            }
          }
          steps {
            git(url: 'http://gitlab.runjs.cn:8929/kubernetes/spring-boot-project.git', branch: env.gitlabBranch, changelog: true, poll: true, credentialsId: "${GITLAB_AUTH}")
            script {
              COMMIT_ID = sh(returnStdout: true, script: "git log -n 1 --pretty=format:'%h'").trim()
              TAG = BUILD_TAG + '-' + COMMIT_ID
              println "Current branch is ${env.gitlabBranch}, Commit ID is ${COMMIT_ID}, Image TAG is ${TAG}"
              }
            }
	      }
	    }
	  }
    stage('Building') {
      steps {
        container(name: 'build') {
            sh """
              mvn clean install -DskipTests		# 编译命令，需要根据自己的实际情况进行修改，可能会不一致
              ls target/*						 # 完成构建后一般会在target目录生产jar包
            """
        }
      }
    }
    stage('Docker build for creating image') {
      steps {
        withCredentials([[$class: 'UsernamePasswordMultiBinding',
        credentialsId: "${HARBOR_AUTH}",
        usernameVariable: 'Harbor_user',
        passwordVariable: 'Harbor_password']]) {
            sh """#!/bin/bash
              set -eux
              # 登录Harbor
              docker login -u ${Harbor_user} -p ${Harbor_password} ${HARBOR_ADDRESS}
              # Dockerfile放在代码仓和Jenkinsfile同级
              docker build --no-cache -t ${HARBOR_ADDRESS}/${REGISTRY_DIR}/${IMAGE_NAME}:${TAG} .
              # 推送镜像至私有harbor仓库
              docker push ${HARBOR_ADDRESS}/${REGISTRY_DIR}/${IMAGE_NAME}:${TAG}
              """
        }
      }
    }
    stage('Deploying to K8s') {
      steps {
           sh """#!/bin/bash
           set -ux
           kubectl set image deploy -l app=${IMAGE_NAME} 			 # set更新deployment镜像${IMAGE_NAME}=${HARBOR_ADDRESS}/${REGISTRY_DIR}/${IMAGE_NAME}:${TAG} -n $NAMESPACE
           """
      }
    }
  }
}
```

## 6.4 Jenkinsfile详解

首先是顶层的agent，定义Kubernetes的Pod作为Jenkins的Slave：

```
    agent {
        kubernetes {					// 定义kubernetes作为agent
            cloud 'kubernetes'			// 选择的云为之前配置的名称
            slaveConnectTimeout 1200
            // 存储为nfs方式持久化挂载
            workspaceVolume nfsWorkspaceVolume(serverAddress: "192.168.1.60", serverPath: "/QaTest/jenkins-salve", readOnly: "false")
            yaml '''
apiVersion: v1
kind: Pod
metadata:
  name: "jenkins-slave"
  namespace: "kube-ops"
  labels:
    jenkins: "slave"
    jenkins/label: "jenkins-jnlp"
spec:
  containers:
  - name: "jnlp"		# jnlp容器，和jenkins主节点通信。包含kubectl命令、helm命令、docker命令
    image: "hub.gitee.cc/kubernetes/jenkins-salve:latet-jdk11"
    imagePullPolicy: "IfNotPresent"
    tty: true
    resources:
      limits:
        cpu: 1000m
        memory: 2000Mi
      requests:
        cpu: 200m
        memory: 400Mi
    env:
    - name: "DOCKER_HOST"
      value: "tcp://192.168.1.141:2375"
    - name: "JENKINS_AGENT_WORKDIR"
      value: "/home/jenkins/agent"
    volumeMounts:
    - mountPath: "/root/.kube/config"	# 挂载kubeconfig文件
      name: "volume-0"
      readOnly: false
      subPath: "config"
    - name: localtime
      mountPath: /etc/localtime
    - mountPath: "/home/jenkins/agent"
      name: "workspace-volume"
      readOnly: false
    workingDir: "/home/jenkins/agent"

  - name: "build"		# build容器，包含构建的命令，java需要maven构建，需要一个maven镜像
    image: "registry.cn-beijing.aliyuncs.com/citools/maven:3.5.3"
    imagePullPolicy: "IfNotPresent"
    tty: true
    volumeMounts:
      - name: localtime
        mountPath: /etc/localtime
      # Pod单独创建了一个缓存的volume，将其挂载到了maven插件的缓存目录，默认是/root/.m2
      - name:  cachedir
        mountPath: "/root/.m2/"
        readOnly: false
    command:
      - "cat"
    env:
      - name: "LANGUAGE"
        value: "en_US:en"
      - name: "LC_ALL"
        value: "en_US.UTF-8"
      - name: "LANG"
        value: "en_US.UTF-8"
  volumes:
    - name: localtime
      hostPath:
        path: /usr/share/zoneinfo/Asia/Shanghai
    - configMap:
        name: "kubeconfig"
        optional: false
      name: "volume-0"
    - name: "cachedir"		 # 缓存目录
      hostPath:
        path: "/opt/m2"
    - emptyDir:
        medium: ""
      name: "workspace-volume"
  restartPolicy: Always
  hostNetwork: false
  imagePullSecrets:
  - name: "regcred"
  nodeSelector:				# 固定节点部署
    build: "true"
            '''
        }
    }
```

之后看一下Jenkinsfile最后的环境变量和parameters的配置：

```
	environment {			// 定义一些全局的环境变量
		COMMIT_ID = ""	
		HARBOR_ADDRESS = "hub.gitee.cc"			// harbor地址
		REGISTRY_DIR = "kubernetes"				// harbor的项目目录
		IMAGE_NAME = "spring-boot-project"		// 镜像名称
		NAMESPACE = "kubernetes"				// kubernetes中的名称空间
		TAG = ""								// 镜像的分支，由BUILD_TAG+COMMIT_ID组成
		GITLAB_AUTH = "gitlab-auth"				// gitlab账号密码认证	
		HARBOR_AUTH = "harbor-user-passwd"		// harbor账号密码认证
  	}
  parameters {	// 使用GitParameter插件，该字段会在jenkins页面生成一个选择分支的选项
    gitParameter(branch: '', branchFilter: 'origin/(.*)', defaultValue: 'master', description: 'Branch for build and deploy', name: 'BRANCH', quickFilterEnabled: false, selectedValue: 'NONE', sortMode: 'NONE', tagFilter: '*', type: 'PT_BRANCH')
  }
```

接下来是拉取代码的stage，这个stage是一个并行的stage，因为考虑了该流水线是手动触发还是自动触发：

```sh
      stage('Pulling Code') {
		parallel {
		stage('Pulling Code by Jenkins') {
          when {
            expression {	// 若env.gitlabBranch为空，则为手动触发执行手动 stage,若不为空，之前同级的另外一个stage
              env.gitlabBranch == null
            }
          }
          steps {
            git(changelog: true, poll: true, url: 'http://gitlab.runjs.cn:8929/kubernetes/spring-boot-project.git', branch: "${BRANCH}", credentialsId: "${GITLAB_AUTH}")
            script {
              // 定义变量用于生成镜像的Tag
              // 获取最近一次提交的Commit ID
              COMMIT_ID = sh(returnStdout: true, script: "git log -n 1 --pretty=format:'%h'").trim()
              TAG = BUILD_TAG + '-' + COMMIT_ID		// 赋值给TAG变量，后面的docker build可以获取到该TAG的值
              println "Current branch is ${BRANCH}, Commit ID is ${COMMIT_ID}, Image TAG is ${TAG}"
            }
          }
        }
        stage('Pulling Code by trigger') {
          when {
            expression {
			  // 若env.gitlabBranch不为空，则为webook触发的执行webook stage。上面的stage不执行，此时BRANCH变量为空
              env.gitlabBranch != null
            }
          }
          steps {
            git(url: 'http://gitlab.runjs.cn:8929/kubernetes/spring-boot-project.git', branch: env.gitlabBranch, changelog: true, poll: true, credentialsId: "${GITLAB_AUTH}")
            script {
              COMMIT_ID = sh(returnStdout: true, script: "git log -n 1 --pretty=format:'%h'").trim()
              TAG = BUILD_TAG + '-' + COMMIT_ID
              println "Current branch is ${env.gitlabBranch}, Commit ID is ${COMMIT_ID}, Image TAG is ${TAG}"
              }
            }
	      }
	    }
	  }
```

代码拉下来后，就可以执行构建命令，由于本次实验是Java示例，因此需要使用mvn命令进行构建：

```
    stage('Building') {
      steps {
        container(name: 'build') {		# 使用pod模板里面build容器进行构建
            sh """ 
              mvn clean install -DskipTests		# 编译命令，需要根据自己的实际情况进行修改
              ls target/*						# 完成构建后一般会在target目录生产jar包
            """
        }
      }
    }
```

生成编译产物后，需要根据该产物生成对应的镜像,并将镜像推送到harbor私有仓库：

```sh
    stage('Docker build for creating image') {
      steps {
        withCredentials([[$class: 'UsernamePasswordMultiBinding',
        credentialsId: "${HARBOR_AUTH}",
        usernameVariable: 'Harbor_user',
        passwordVariable: 'Harbor_password']]) {
            sh """#!/bin/bash
              set -eux
              # 登录Harbor
              docker login -u ${Harbor_user} -p ${Harbor_password} ${HARBOR_ADDRESS}
              # Dockerfile放在代码仓和Jenkinsfile同级
              docker build --no-cache -t ${HARBOR_ADDRESS}/${REGISTRY_DIR}/${IMAGE_NAME}:${TAG} .
              # 推送镜像至私有harbor仓库
              docker push ${HARBOR_ADDRESS}/${REGISTRY_DIR}/${IMAGE_NAME}:${TAG}
              docker rmi -f ${HARBOR_ADDRESS}/${REGISTRY_DIR}/${IMAGE_NAME}:${TAG}
              """
        }
      }
    }
```

最后一步就是将该镜像发版至Kubernetes集群中，此时使用的是包含kubectl命令的容器：

```sh
    stage('Deploying to K8s') {
      steps {
           sh """#!/bin/bash
           set -ux
           # set更新deployment镜像
           kubectl set image deploy -l app=${IMAGE_NAME} 		${IMAGE_NAME}=${HARBOR_ADDRESS}/${REGISTRY_DIR}/${IMAGE_NAME}:${TAG} -n $NAMESPACE
           """
      }
    }
```

- Java项目部署到kubernetes示例代码


```
// 公共
def registry = "wangzilong.top:9003"
// 项目
def project = "library"
def app_name = "java-demo"
def image_name = "${registry}/${project}/${app_name}:${BUILD_NUMBER}"
def git_address = "https://gitee.com/wangzilong9570/devopsdemo.git"
// 认证
def secret_name = "registry-auth"
def harbor_auth = "e59aef57-3631-487c-9af4-0432a4581d83"
def git_auth = "b6d473b8-a8bc-4029-979e-07c0b386c798"
def k8s_auth = "76b249bf-f2e1-4ca7-9620-860c111ed0c3"

pipeline {
  agent { 
      label "jenkins-slave" 
      }
    parameters {    
        gitParameter branch: '', branchFilter: '.*', defaultValue: 'master', description: '选择发布的分支', name: 'Branch', quickFilterEnabled: false, selectedValue: 'NONE', sortMode: 'NONE', tagFilter: '*', type: 'PT_BRANCH'
        choice (choices: ['1', '3', '5', '7'], description: '副本数', name: 'ReplicaCount')
        choice (choices: ['default','dev','test','prod'], description: '命名空间', name: 'Namespace')
    }
    stages {
        stage('拉取代码'){
            steps {
                container('jenkins-build'){
                    checkout([$class: 'GitSCM', 
                    branches: [[name: "${params.Branch}"]], 
                    doGenerateSubmoduleConfigurations: false, 
                    extensions: [], submoduleCfg: [], 
                    userRemoteConfigs: [[credentialsId: "${git_auth}", url: "${git_address}"]]
                    ])
                }
            }
        }
        stage('代码编译'){
           steps {
             container('jenkins-build'){
               sh """
                  mvn clean package -Dmaven.test.skip=true
                  """ 
           		}
						}
        }
        stage('构建镜像'){
           steps {
            	container('jenkins-build'){
                withCredentials([usernamePassword(credentialsId: "${harbor_auth}", passwordVariable: 'password', usernameVariable: 'username')]) {
                sh """
                  unzip target/*.war -d target/ROOT  
                  echo '
                    FROM lizhenliang/tomcat
                    LABEL maitainer wangzilong
                    ADD target/ROOT /usr/local/tomcat/webapps/ROOT
                  ' > Dockerfile
                  docker build -t ${image_name} .
                  docker login -u ${username} -p '${password}' ${registry}
                  docker push ${image_name}
                """
                }
							}
           } 
        }
        stage('部署到K8S平台'){
          steps {
            	container('jenkins-build'){
              configFileProvider([configFile(fileId: "${k8s_auth}", targetLocation: "admin.kubeconfig")]){
                sh """
                  sed -i 's#IMAGE_NAME#${image_name}#' deployment.yaml
                  sed -i 's#SECRET_NAME#${secret_name}#' deployment.yaml
                  sed -i 's#REPLICAS#${ReplicaCount}#' deployment.yaml
                  chmod u+x /usr/bin/kubectl
                  kubectl apply -f deployment.yaml -n ${Namespace} --kubeconfig=admin.kubeconfig
                """
              }
						}
          }
        }
    }
}
```

## 6.5 定义Dockerfile

在执行流水线过程时，需要将代码的编译产物做成镜像。Dockerfile主要写的是如何生成公司业务的镜像，而本示例是Java项目，只需要把JAR包放在有JRE环境的镜像中，然后启动该JAR包即可：

`Dockerfile`

```dockerfile
# 基础镜像可以按需修改，可以更改为公司自有的镜像
FROM registry.cn-beijing.aliyuncs.com/dotbalo/jre:8u211-data

# JAR包名称改成实际的名称，本示例为spring-cloud-eureka-0.0.1-SNAPSHOT.jar
COPY target/spring-cloud-eureka-0.0.1-SNAPSHOT.jar ./

# 启动JAR包
CMD java -jar spring-cloud-eureka-0.0.1-SNAPSHOT.jar
```

## 6.6 定义Kubernetes资源

本书的示例在GitLab创建的Group为kubernetes，可以认为其是一个项目，同一个项目可以部署至Kubernetes集群中的同一个Namespace中，本示例为kubernetes命名空间。由于使用的是私有仓库，因此需要先配置拉取私有仓库镜像的密钥：

```sh
# kubectl create ns kubernetes
namespace/kubernetes created
# kubectl -n kubernetes create secret docker-registry harborkey \
--docker-server=http://hub.gitee.cc \
--docker-username=admin \
--docker-password=oschina123 \
--docker-email=1@2.com
```

配置该应用的Deployment（部分内容），需要注意文件的文字注释部分：`deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: spring-boot-project
  name: spring-boot-project
  namespace: kubernetes
spec:
  replicas: 1
  selector:
    matchLabels:
      app: spring-boot-project
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: spring-boot-project
    spec:
      containers:
      - env:
        - name: TZ
          value: Asia/Shanghai
        - name: LANG
          value: C.UTF-8
        image: hub.gitee.cc/kubernetes/spring-boot-project:jenkins-spring-boot-project-2-e9ddd5d
        imagePullPolicy: IfNotPresent
        livenessProbe:
          failureThreshold: 2
          initialDelaySeconds: 30
          periodSeconds: 10
          successThreshold: 1
          tcpSocket:
            port: 8761
          timeoutSeconds: 2
        name: spring-boot-project
        ports:
        - containerPort: 8761
          name: web
          protocol: TCP
        resources:
          limits:
            cpu: 994m
            memory: 2170Mi
          requests:
            cpu: 10m
            memory: 55Mi
      dnsPolicy: ClusterFirst
      imagePullSecrets:
      - name: harborkey
      restartPolicy: Always
      securityContext: {}
      serviceAccountName: default
```

定义该应用的Service和Ingress：`spring-boot-project-svc-ingress.yaml`

```yaml
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: spring-boot-project
  name: spring-boot-project
  namespace: kubernetes
spec:
  ports:
  - name: web
    port: 8761
    protocol: TCP
    targetPort: 8761
  selector:
    app: spring-boot-project
  type: ClusterIP
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: spring-boot-project
  namespace: kubernetes
spec:
  ingressClassName: nginx
  rules:
  - host: spring-boot-project.test.com
    http:
      paths:
      - backend:
          service:
            name: spring-boot-project
            port: 
              number: 8761
        path: /
        pathType: ImplementationSpecific
```

需要注意的是，在实际使用时，Java一般是后台应用，不需要使用Ingress暴露出去，只需要配置Service即可。

如果需要被集群外部的用户直接访问，需要配置Ingress，通过域名发布。

创建该资源：

```shell
$ kubectl create -f deployment.yaml
$ kubectl create -f spring-boot-project-svc-ingress.yaml
service/spring-boot-project created
ingress.networking.k8s.io/spring-boot-project created
```

> "提示"：由于在Harbor已经配置了Kubernetes的项目目录，在此不再重复创建，如果需要将镜像推送至其他目录，按照同样的步
> 骤创建即可。

## 6.7 创建Jenkins任务

单击首页的创建任务（Job）选项（或者New Item）并配置任务信息，输入Job的名称（一般和GitLab的仓库名字一致，以便于区分），类型为Pipeline，最后单击OK。

![](./assets/6-37.png){: .zoom}

在新页面单击Pipeline，输入Git仓库地址，选择之前配置的GitLab 账号密码Key，分支为master，单击Save。

![](./assets/6-38.png){: .zoom}

第一次构建也会失败，第二次可以选择分支构建，之后单击Build，然后单击Build History的进度条，即可查看构建日志。

![](./assets/6-39.png){: .zoom}

<img src="./assets/6-40.png" style="zoom:50%;" />

构建日志上部分为创建Pod的日志，可以看到Pod为Agent指定的Pod，如图

<img src="./assets/6-41.png" style="zoom:50%;" />


如果是Java应用，可以看到BUILD SUCCESS，说明mvn编译已经通过，如图
![](./assets/6-42.png){: .zoom}


编译结束后，可以看到制作镜像的日志，之后镜像被推送至Harbor，最后发版至Kubernetes。
![](./assets/6-43.png){: .zoom}

![](./assets/6-44.png){: .zoom}

![](./assets/6-45.png){: .zoom}

出现Finished：SUCCESS说明该流水线正常结束。接下来可以查看Deployment的镜像：

```sh
$ kubectl get deploy -n kubernetes spring-boot-project -o yaml|grep "image: "
        image: hub.gitee.cc/kubernetes/spring-boot-project:jenkins-spring-boot-project-5-adf694c
$ kubectl get pod -n kubernetes
NAME                                   READY   STATUS    RESTARTS        AGE
spring-boot-project-65f88c8f6b-l89sl   1/1     Running   1 (7m34s ago)   8m24s
```

**注意**

> 该示例只是用Eureka的代码作为测试应用，如果是生产环境的Eureka，最好使用StatefulSet创建Eureka集群，因为StatefulSet会有一个固定的识符 ， 此时在Eureka的defaultZone配置如下：
>
> defaultZone:http://eureka-0.eureka:8761/eureka/ 、 http://eureka-1.eureka:8761/eureka/、http://eureka-2.eureka:8761/eureka/，即可形成集群。
>
> 其他的Spring Cloud模块可以通过该地址进行注册。

## 6.8 Webhook自动触发构建

之前的构建都是手动选择分支进行构建的，在实际使用时，项目可能有很多，如果都是手动触发，可能比较消耗人力。

所以推荐按需配置自动触发，即提交代码后自动触发Jenkins构建任务。

本次用Java项目进行演示。首先找到Java项目的Job，单击Configure，之后选择Build Triggers，勾选Build when a change…，记录webhook URL，之后单击Advanced，选择Allow all branches to trigger this job，如果不想任何分支都可以触发该流水线，可以选择Filter进行条件匹配。之后单击Generate生成Secret token，最后单击Save即可。

![](./assets/6-46.png){: .zoom}

![](./assets/6-47.png){: .zoom}

接下来配置GitLab，首先单击Menu→Admin选择Settings→Network，之后允许访问外部的请求。
![](./assets/6-48.png){: .zoom}

在Integrations配置页的相应输入框中粘贴上一步中Jenkins暴露的webhook及相应的Secret token如下：

![](./assets/6-49.png){: .zoom}

测试整个链路是否通了。在GitLab上添加好这个webhook后，可以跳到表单下方，有一个已添加的webhook列表，找到我们刚刚创建的webbook。单击“Test”按钮，选择“Pushevents”，GitLab就会向该webhook发送一个event。

添加之后，可以点击一下test看看流程是否能够走通，如果走通，那么我们以后开发的时候直接推送代码即可触发构建。正常状态返回如下：

![](./assets/6-50.png){: .zoom}

也可以通过Blue Ocean看到是自动触发的stage被执行，如图

![](./assets/6-51.png){: .zoom}

新建Pull Request分支，提交代码测试CI触发。

```shell
$ git clone ssh://git@gitlab.runjs.cn:2224/kubernetes/spring-boot-project.git
$ cd spring-boot-project/
$ git checkout -b dev-hu
$ echo "test=hu" >> README.md
$ git add .
$ git commit -m "hujianli test"
$ git push --set-upstream origin dev-hu
```

触发后手工执行中，出现可以选择Pull Request版本的信息。

<img src="./assets/6-52.png" style="zoom:50%;" />

以上就是通过GitLab的事件触发Jenkins任务，在实际使用时，此功能非常常用，一般会用于开发、测试等环境，省去了手动构建的过程。==而在UAT和生产环境，一般不需要再次构建，而是选择其他环境产生的镜像进行发版。==

## 6.9 构建状态信息推送到GitLab

先生成一个访问令牌，用于在jenkins的系统配置去配置连接gitlab的选项
![](./assets/6-53.png){: .zoom}

生成个人访问令牌。
![](./assets/6-54.png){: .zoom}

首先在jenkins上创建一个gitlab api token的凭证
![](./assets/6-55.png){: .zoom}

首先在`系统管理`-`系统配置`中配置好连接Gitlab的信息如下

![](./assets/6-56.png){: .zoom}

**在pipeline的post部分，将构建结果更新到GitLab的相应commit记录上**。

除此之外，还需要在**options部分加入gitLabConnection配置，同时传入"gitlab"参数**。"gitlab"就是`Connection name`的值。

```
pipeline { // 核心代码
    agent any
    triggers {
        gitlab(triggerOnPush: true,
            triggerOnMergeRequest: true,
            branchFilterType: "All",
            secretToken: "t8vcxwuza023ehzcftzr5a74vkpto6xr")
    }
    stages {
        stage('build') {
            steps {
                echo 'Hello World from gitlab trigger'
            }
        }
    }
    post {
        failure {
            updateGitlabCommitStatus name: "build", state: "failed"
        }
        success {
            updateGitlabCommitStatus name: "build", state: "success"
        }
    }
    options {
        gitLabConnection("gitlab")
    }
}
```

提交代码后，需要手动触发一次构建，pipeline才会生效。

![](./assets/6-57.png){: .zoom}

> "参考文献" ：jenkins返回构建状态到gitlab
>
> https://www.cnblogs.com/shipment/p/14690649.html
>
> https://blog.51cto.com/ygqygq2/2461766
>
> https://www.jianshu.com/p/d2af2915cfea

# 7. 自动化构建Vue/H5前端应用

本节介绍自动化构建Vue/H5应用，其构建方式和自动化构建Java基本相同，重点是更改Deployment、Jenkinsfile和Dockerfile。

同样，前端应用也为读者准备了一个测试项目(项目地址：https://gitee.com/dukuan/vue-project.git),可以按照创建Java测试用例的方式导入前端项目到GitLab中，当然也可以使用公司自己的项目。

## 7.1 定义Jenkinsfile

先将镜像上传到harbor私有仓库

```shell
$ docker pull registry.cn-beijing.aliyuncs.com/citools/node:lts
$ docker tag registry.cn-beijing.aliyuncs.com/citools/node:lts hub.gitee.cc/kubernetes/node:lts
$ docker push hub.gitee.cc/kubernetes/node:lts
```

Jenkinsfile和Java项目并无太大区别，需要更改的位置如下（展示部分代码）：

```sh
pipeline{
	environment {
		COMMIT_ID = ""
		HARBOR_ADDRESS = "hub.gitee.cc"
		REGISTRY_DIR = "kubernetes"
		IMAGE_NAME = "vue-project"
		NAMESPACE = "kubernetes"
		TAG = ""
		GITLAB_AUTH = "gitlab-auth"
		HARBOR_AUTH = "harbor-user-passwd"
  	}
  parameters {
    gitParameter(branch: '', branchFilter: 'origin/(.*)', defaultValue: 'master', description: 'Branch for build and deploy',
                name: 'BRANCH', quickFilterEnabled: false, selectedValue: 'NONE', sortMode: 'NONE', tagFilter: '*', type: 'PT_BRANCH')
  }
    options {
      parallelsAlwaysFailFast()
      disableResume()
      quietPeriod(1)
      preserveStashes(buildCount: 5)
      timestamps()
      buildDiscarder(logRotator(numToKeepStr: '100'))
      timeout(time: 60, unit: 'MINUTES')
      gitLabConnection("gitlab")
    }
    agent {
        kubernetes {
            cloud 'kubernetes'
            slaveConnectTimeout 1200
            workspaceVolume nfsWorkspaceVolume(serverAddress: "192.168.1.60", serverPath: "/QaTest/jenkins-salve", readOnly: "false")
            yaml '''
apiVersion: v1
kind: Pod
metadata:
  name: "jenkins-slave"
  namespace: "kube-ops"
  labels:
    jenkins: "slave"
    jenkins/label: "jenkins-jnlp"
spec:
  containers:
  - name: "jnlp"
    image: "hub.gitee.cc/kubernetes/jenkins-salve:latet-jdk11"
    imagePullPolicy: "IfNotPresent"
    tty: true
    resources:
      limits:
        cpu: 1000m
        memory: 2000Mi
      requests:
        cpu: 200m
        memory: 400Mi
    env:
    - name: "DOCKER_HOST"
      value: "tcp://192.168.1.141:2375"
    - name: "JENKINS_AGENT_WORKDIR"
      value: "/home/jenkins/agent"
    volumeMounts:
    - mountPath: "/root/.kube/config"
      name: "volume-0"
      readOnly: false
      subPath: "config"
    - name: localtime
      mountPath: /etc/localtime
    - mountPath: "/home/jenkins/agent"
      name: "workspace-volume"
      readOnly: false
    workingDir: "/home/jenkins/agent"
  - name: "build"
    image: "hub.gitee.cc/kubernetes/node:lts"
    imagePullPolicy: "IfNotPresent"
    tty: true
    volumeMounts:
      - name: localtime
        mountPath: /etc/localtime
    command:
      - "cat"
    env:
      - name: "LANGUAGE"
        value: "en_US:en"
      - name: "LC_ALL"
        value: "en_US.UTF-8"
      - name: "LANG"
        value: "en_US.UTF-8"
  volumes:
    - name: localtime
      hostPath:
        path: /usr/share/zoneinfo/Asia/Shanghai
    - configMap:
        name: "kubeconfig"
        optional: false
      name: "volume-0"
    - emptyDir:
        medium: ""
      name: "workspace-volume"
  restartPolicy: Always
  hostNetwork: false
  imagePullSecrets:
  - name: "regcred"
  nodeSelector:			  # 固定节点部署
    build: "true"
            '''
        }
    }
    stages {
      stage('Pulling Code') {
		parallel {
		stage('Pulling Code by Jenkins') {
          when {
            expression {
              env.gitlabBranch == null
            }
          }
          steps {
            git(changelog: true, poll: true, url: 'http://gitlab.runjs.cn:8929/kubernetes/vue-project.git', branch: "${BRANCH}", credentialsId: "${GITLAB_AUTH}")
            script {
              COMMIT_ID = sh(returnStdout: true, script: "git log -n 1 --pretty=format:'%h'").trim()
              TAG = BUILD_TAG + '-' + COMMIT_ID
              println "Current branch is ${BRANCH}, Commit ID is ${COMMIT_ID}, Image TAG is ${TAG}"
            }
          }
        }
        stage('Pulling Code by trigger') {
          when {
            expression {
              env.gitlabBranch != null
            }
          }
          steps {
            git(url: 'http://gitlab.runjs.cn:8929/kubernetes/vue-project.git', branch: env.gitlabBranch, changelog: true, poll: true, credentialsId: "${GITLAB_AUTH}")
            script {
              COMMIT_ID = sh(returnStdout: true, script: "git log -n 1 --pretty=format:'%h'").trim()
              TAG = BUILD_TAG + '-' + COMMIT_ID
              println "Current branch is ${env.gitlabBranch}, Commit ID is ${COMMIT_ID}, Image TAG is ${TAG}"
              }
            }
	      }
	    }
	  }
    stage('Building') {
      steps {
        container(name: 'build') {
            sh """
              npm install --registry=https://registry.npm.taobao.org
              npm run build
            """
        }
      }
    }
    stage('Docker build for creating image') {
      steps {
        withCredentials([[$class: 'UsernamePasswordMultiBinding',
        credentialsId: "${HARBOR_AUTH}",
        usernameVariable: 'Harbor_user',
        passwordVariable: 'Harbor_password']]) {
            sh """#!/bin/bash
              set -eux
              docker login -u ${Harbor_user} -p ${Harbor_password} ${HARBOR_ADDRESS}
              docker build --no-cache -t ${HARBOR_ADDRESS}/${REGISTRY_DIR}/${IMAGE_NAME}:${TAG} .
              docker push ${HARBOR_ADDRESS}/${REGISTRY_DIR}/${IMAGE_NAME}:${TAG}
              docker rmi -f ${HARBOR_ADDRESS}/${REGISTRY_DIR}/${IMAGE_NAME}:${TAG}
              """
        }
      }
    }
    stage('Deploying to K8s') {
      steps {
           sh """#!/bin/bash
           set -ux
           # set更新deployment镜像
           kubectl set image deploy -l app=${IMAGE_NAME} ${IMAGE_NAME}=${HARBOR_ADDRESS}/${REGISTRY_DIR}/${IMAGE_NAME}:${TAG} -n $NAMESPACE
           """
      }
    }
  }
      post {
        failure {
            updateGitlabCommitStatus name: "build", state: "failed"
        }
        success {
            updateGitlabCommitStatus name: "build", state: "success"
        }
    }
}
```

可以看到对于不同的项目，只是构建的镜像和构建命令不一致，整个流水线的过程基本一致，所以在定义了一个Jenkinsfile后，稍加修改即可实现其他语言的自动化构建。

## 7.2 定义Dockerfile

前端应用构建后一般会在项目根目录下的dist文件下产生HTML文件，只需要复制到nginx的根目录下即可：`Dockerfile`

```dockerfile
FROM registry.cn-beijing.aliyuncs.com/dotbalo/nginx:1.15.12
COPY dist/* /usr/share/nginx/html/
```

Jenkinsfile和Dockerfile文件添加到仓库代码同级目录下，如下图所示

<img src="./assets/6-58.png" style="zoom:50%;" />

## 7,3 定义Kubernetes资源

对于Kubernetes的资源也是类似的，只需要更改资源名称和端口号即可：`deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: vue-project
  name: vue-project
  namespace: kubernetes
spec:
  replicas: 1
  selector:
    matchLabels:
      app: vue-project
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: vue-project
    spec:
      containers:
      - env:
        - name: TZ
          value: Asia/Shanghai
        - name: LANG
          value: C.UTF-8
        image: nginx
        imagePullPolicy: IfNotPresent
        livenessProbe:
          failureThreshold: 2
          initialDelaySeconds: 30
          periodSeconds: 10
          successThreshold: 1
          tcpSocket:
            port: 80
          timeoutSeconds: 2
        name: vue-project
        ports:
        - containerPort: 80
          name: web
          protocol: TCP
        resources:
          limits:
            cpu: 994m
            memory: 1170Mi
          requests:
            cpu: 10m
            memory: 55Mi
      dnsPolicy: ClusterFirst
      imagePullSecrets:
      - name: harborkey
      restartPolicy: Always
      serviceAccountName: default
```

定义该应用的Service和Ingress：`vue-project-svc-ingress.yaml`

```yaml
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: vue-project
  name: vue-project
  namespace: kubernetes
spec:
  ports:
  - name: web
    port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: vue-project
  sessionAffinity: None
  type: ClusterIP
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: vue-project
  namespace: kubernetes
spec:
  ingressClassName: nginx
  rules:
  - host: vue-project.test.com
    http:
      paths:
      - backend:
          service:
            name: vue-project
            port: 
              number: 80
        path: /
        pathType: ImplementationSpecific
```

创建该资源：

```shell
$ kubectl create -f deployment.yaml
$ kubectl create -f vue-project-svc-ingress.yaml
```

## 7.4 创建Jenkins Job

创建Jenkins Job和上一小节类似，并无太大区别。首先创建Pipeline类型的Job，名称为vue-project，同样在Pipeline栏填写地址、Key和分支，如图

![](./assets/6-59.png){: .zoom}

同样，第一次构建也会失败，第二次可以选择分支构建.当出现SUCCESS时，表示构建结束。

![](./assets/6-60.png){: .zoom}

接下来可以查看Deployment的镜像：

```shell
$ kubectl get deploy vue-project -o yaml -n kubernetes|grep "image:"
        image: hub.gitee.cc/kubernetes/vue-project:jenkins-vue-project-4-9256578
$ kubectl get pod -n kubernetes
NAME                          READY   STATUS    RESTARTS   AGE
vue-project-f56dcdb95-zwbl5   1/1     Running   0          29m
```

部署更新镜像完毕之后可以在浏览器访问该域名。

<img src="./assets/6-61.png" style="zoom:50%;" />


## 7.5 Webhook自动触发

同6.8设置相同。参考上面设置内容。

# 8. 自动化构建Golang项目

前面演示了Java和前端应用的自动化，接下来演示对于Golang的自动化构建，[本示例的代码地址](https://gitee.com/dukuan/go-project.git)

可以按照创建Java测试用例的方式导入前端项目到GitLab中，当然也可以使用公司自己的项目。

## 8.1 定义Jenkinsfile

先将镜像上传到harbor私有仓库

```shell
$ docker pull registry.cn-beijing.aliyuncs.com/citools/golang:1.15
$ docker tag registry.cn-beijing.aliyuncs.com/citools/golang:1.15 hub.gitee.cc/kubernetes/golang:1.15
$ docker push hub.gitee.cc/kubernetes/golang:1.15
```

本示例的Jenkinsfile和之前的没有太大区别，需要更改的位置是构建容器的镜像、缓存目录、Git地址和项目名称：

```sh
pipeline{
	environment {
		COMMIT_ID = ""
		HARBOR_ADDRESS = "hub.gitee.cc"
		REGISTRY_DIR = "kubernetes"
		IMAGE_NAME = "go-project"
		NAMESPACE = "kubernetes"
		TAG = ""
		GITLAB_AUTH = "gitlab-auth"
		HARBOR_AUTH = "harbor-user-passwd"
  	}
  parameters {
    gitParameter(branch: '', branchFilter: 'origin/(.*)', defaultValue: 'master', description: 'Branch for build and deploy', name: 'BRANCH', quickFilterEnabled: false, selectedValue: 'NONE', sortMode: 'NONE', tagFilter: '*', type: 'PT_BRANCH')
  }
    options {
      parallelsAlwaysFailFast()
      disableResume()
      quietPeriod(1)
      preserveStashes(buildCount: 5)
      timestamps()
      buildDiscarder(logRotator(numToKeepStr: '100'))
      timeout(time: 60, unit: 'MINUTES')
      gitLabConnection("gitlab")
    }
    agent {
        kubernetes {
            cloud 'kubernetes'
            slaveConnectTimeout 1200
            workspaceVolume nfsWorkspaceVolume(serverAddress: "192.168.1.60", serverPath: "/QaTest/jenkins-salve", readOnly: "false")
            yaml '''
apiVersion: v1
kind: Pod
metadata:
  name: "jenkins-slave"
  namespace: "kube-ops"
  labels:
    jenkins: "slave"
    jenkins/label: "jenkins-jnlp"
spec:
  containers:
  - name: "jnlp"
    image: "hub.gitee.cc/kubernetes/jenkins-salve:latet-jdk11"
    imagePullPolicy: "IfNotPresent"
    tty: true
    resources:
      limits:
        cpu: 1000m
        memory: 2000Mi
      requests:
        cpu: 200m
        memory: 400Mi
    env:
    - name: "DOCKER_HOST"
      value: "tcp://192.168.1.141:2375"
    - name: "JENKINS_AGENT_WORKDIR"
      value: "/home/jenkins/agent"
    volumeMounts:
    - mountPath: "/root/.kube/config"
      name: "volume-0"
      readOnly: false
      subPath: "config"
    - name: localtime
      mountPath: /etc/localtime
    - mountPath: "/home/jenkins/agent"
      name: "workspace-volume"
      readOnly: false
    workingDir: "/home/jenkins/agent"
    
  - name: "build"
    image: "hub.gitee.cc/kubernetes/golang:1.15"
    imagePullPolicy: "IfNotPresent"
    tty: true
    volumeMounts:
      - name: localtime
        mountPath: /etc/localtime
      - name:  cachedir
        mountPath: "/go/pkg/"
        readOnly: false
    command:
      - "cat"
    env:
      - name: "LANGUAGE"
        value: "en_US:en"
      - name: "LC_ALL"
        value: "en_US.UTF-8"
      - name: "LANG"
        value: "en_US.UTF-8"
  volumes:
    - name: localtime
      hostPath:
        path: /usr/share/zoneinfo/Asia/Shanghai
    - configMap:
        name: "kubeconfig"
        optional: false
      name: "volume-0"
    - name: "cachedir"			 # 缓存目录
      hostPath:
        path: "/opt/gopkg"
    - emptyDir:
        medium: ""
      name: "workspace-volume"
  restartPolicy: Always
  hostNetwork: false
  imagePullSecrets:
  - name: "regcred"
  nodeSelector:					 # 固定节点部署
    build: "true"
            '''
        }
    }
    stages {
      stage('Pulling Code') {
		parallel {
		stage('Pulling Code by Jenkins') {
          when {
            expression {
              env.gitlabBranch == null
            }
          }
          steps {
            git(changelog: true, poll: true, url: 'http://gitlab.runjs.cn:8929/kubernetes/go-project.git', branch: "${BRANCH}", credentialsId: "${GITLAB_AUTH}")
            script {
              COMMIT_ID = sh(returnStdout: true, script: "git log -n 1 --pretty=format:'%h'").trim()
              TAG = BUILD_TAG + '-' + COMMIT_ID
              println "Current branch is ${BRANCH}, Commit ID is ${COMMIT_ID}, Image TAG is ${TAG}"
            }
          }
        }
        stage('Pulling Code by trigger') {
          when {
            expression {
              env.gitlabBranch != null
            }
          }
          steps {
            git(url: 'http://gitlab.runjs.cn:8929/kubernetes/go-project.git', branch: env.gitlabBranch, changelog: true, poll: true, credentialsId: "${GITLAB_AUTH}")
            script {
              COMMIT_ID = sh(returnStdout: true, script: "git log -n 1 --pretty=format:'%h'").trim()
              TAG = BUILD_TAG + '-' + COMMIT_ID
              println "Current branch is ${env.gitlabBranch}, Commit ID is ${COMMIT_ID}, Image TAG is ${TAG}"
              }
            }
	      }
	    }
	  }
    stage('Building') {
      steps {
        container(name: 'build') {
            sh """
               export GO111MODULE=on
               go env -w GOPROXY=https://goproxy.cn,direct
               go build
            """
        }
      }
    }
    stage('Docker build for creating image') {
      steps {
        withCredentials([[$class: 'UsernamePasswordMultiBinding',
        credentialsId: "${HARBOR_AUTH}",
        usernameVariable: 'Harbor_user',
        passwordVariable: 'Harbor_password']]) {
            sh """#!/bin/bash
              set -eux
              docker login -u ${Harbor_user} -p ${Harbor_password} ${HARBOR_ADDRESS}

              docker build --no-cache -t ${HARBOR_ADDRESS}/${REGISTRY_DIR}/${IMAGE_NAME}:${TAG} .

              docker push ${HARBOR_ADDRESS}/${REGISTRY_DIR}/${IMAGE_NAME}:${TAG}
              docker rmi -f ${HARBOR_ADDRESS}/${REGISTRY_DIR}/${IMAGE_NAME}:${TAG}
              """
        }
      }
    }
    stage('Deploying to K8s') {
      steps {
           sh """#!/bin/bash
           set -ux
           kubectl set image deploy -l app=${IMAGE_NAME} ${IMAGE_NAME}=${HARBOR_ADDRESS}/${REGISTRY_DIR}/${IMAGE_NAME}:${TAG} -n $NAMESPACE
           """
      }
    }
  }
    post {
        failure {
            updateGitlabCommitStatus name: "build", state: "failed"
        }
        success {
            updateGitlabCommitStatus name: "build", state: "success"
        }
    }
}
```

## 8.2 定义Dockerfile

和之前不一样的地方是，Golang编译后生成的是一个二进制文件，可以直接执行，所以底层镜像设置为alpine或者其他的小镜像即可：

`Dockerfile`

```dockerfile
FROM registry.cn-beijing.aliyuncs.com/dotbalo/alpine-glibc:alpine-3.9

# 如果定义了单独的配置文件，可能需要拷贝到镜像中
COPY conf/  ./conf 
# 包名按照实际情况进行修改
COPY ./go-project ./ 
EXPOSE 8080

ENTRYPOINT [ "./go-project"] # 启动该应用
```

## 8.3  定义Kubernets资源

`go-project-deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: go-project
  name: go-project
  namespace: kubernetes
spec:
  replicas: 1
  selector:
    matchLabels:
      app: go-project
  template:
    metadata:
      labels:
        app: go-project
    spec:
      containers:
      - env:
        - name: TZ
          value: Asia/Shanghai
        - name: LANG
          value: C.UTF-8
        image: nginx
        imagePullPolicy: IfNotPresent
        livenessProbe:
          failureThreshold: 2
          initialDelaySeconds: 30
          periodSeconds: 10
          successThreshold: 1
          tcpSocket:
            port: 8080
          timeoutSeconds: 2
        name: go-project
        ports:
        - containerPort: 8080
          name: web
          protocol: TCP
        resources:
          limits:
            cpu: 994m
            memory: 1170Mi
          requests:
            cpu: 10m
            memory: 55Mi
      dnsPolicy: ClusterFirst
      imagePullSecrets:
      - name: harborkey
      restartPolicy: Always
      securityContext: {}
      serviceAccountName: default
```

`go-project-svc-ingress.yaml`

```yaml
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: go-project
  name: go-project
  namespace: kubernetes
spec:
  ports:
  - name: web
    port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    app: go-project
  type: ClusterIP

---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: go-project
  namespace: kubernetes
spec:
  ingressClassName: nginx
  rules:
  - host: go-project.test.com
    http:
      paths:
      - backend:
          service:
            name: go-project
            port: 
              number: 8080
        path: /
        pathType: ImplementationSpecific
```

创建该资源：

```shell
$ kubectl apply -f go-project-deployment.yaml
deployment.apps/go-project created

$ kubectl apply -f go-project-svc-ingress.yaml
service/go-project created
ingress.networking.k8s.io/go-project created

$ kubectl get pod -n kubernetes
NAME                          READY   STATUS    RESTARTS      AGE
go-project-5b47cfcc8f-rbmm8   1/1     Running   1 (19s ago)   72s
```

## 8.4 创建Jenkins Job

创建Jenkins Job和前面也没有区别，在此不再重复演示。接下来可以查看Deployment的镜像：

```shell
$ kubectl get deployment -o yaml -n kubernetes |grep "image: "
          image: hub.gitee.cc/kubernetes/go-project:jenkins-go-project-2-2cd059f
$ kubectl get pod -n kubernetes
NAME                          READY   STATUS    RESTARTS   AGE
go-project-6d8f6bddd6-jd5l7   1/1     Running   0          94s
```

之后即可访问该应用（如果是后端项目，则无须创建Ingress暴露服务）

![image-20240515000343427](./assets/image-20240515000343427.png)

之后在创建Pod的节点可以看到缓存目录：

```shell
$ cd /opt/gopkg/
[root@k8s-node01 gopkg]# ll
total 0
drwxr-xr-x 7 root root 96 Feb 22 15:49 mod
drwxr-xr-x 3 root root 28 Feb 22 15:49 sumdb
```

编写代码，更新代码到gitlab仓库中。
![](./assets/6-63.png){: .zoom}

重新触发jenkins pipeline。
![](./assets/6-64.png){: .zoom}

查看代码发布更新状态。
![image-20240515000411370](./assets/image-20240515000411370.png)

# 9. UAT及生产环境流水线设计

上述演示示例都包含构建过程，一般会用在首次提交代码，然后构建，并发布至第一个项目的环境中，一般是开发环境或者测试环境。这类环境采用这种模式没有问题，因为构建和产生镜像的步骤是必需的。

但是在其他环境，比如Sit、UAT、生产环境，其实是没有必要重复构建的，可以直接选择镜像进行发版，当然这种模式和代码本身的设计有关系，也就是要做到配置分离和构建与代码配置无关才行。

如果读者的项目符合此类条件，可以将第一次构建的镜像发布到任意环境，可以采用镜像的方式进行发版，也可以使用该流水线进行回滚操作。



假如我们有一个go-project的UAT环境，该环境的发版不需要构建，只需要选择测试环境的镜像即可，此时UAT环境（选择镜像发版）的Jenkins Job配置如下。

创建一个新的Job，名字为go-project-uat，类型为Pipeline，如图
![](./assets/6-66.png){: .zoom}


勾选页面上的This project is parameterized（参数化构建），选择参数类型为Image Tag Parameter（需要安装Image Tag插件），之后定义Name为变量的名称，Image Name为Harbor的目录和镜像名称，如图

![](./assets/6-67.png){: .zoom}


添加Image Tag参数
![](./assets/6-68.png){: .zoom}


配置Harbor地址
![](./assets/6-69.png){: .zoom}

之后可以先单击Save，然后测试能否获取到Tag，如下图所示，保存后单击Build with Parameters。
![](./assets/6-70.png){: .zoom}


之后单击Configure，添加Pipeline脚本：

```sh
pipeline{
	environment {
		HARBOR_ADDRESS = "hub.gitee.cc"
        NAMESPACE = "kubernetes"
        IMAGE_NAME = "go-project"
        TAG = ""
  	}
    options {
      parallelsAlwaysFailFast()
      disableResume()
      quietPeriod(1)
      preserveStashes(buildCount: 5)
      timestamps()
      buildDiscarder(logRotator(numToKeepStr: '100'))
      timeout(time: 60, unit: 'MINUTES')
    }
    agent {
        kubernetes {
            cloud 'kubernetes'
            slaveConnectTimeout 1200
            workspaceVolume nfsWorkspaceVolume(serverAddress: "192.168.1.60", serverPath: "/QaTest/jenkins-salve", readOnly: "false")
            yaml '''
apiVersion: v1
kind: Pod
metadata:
  name: "jenkins-slave"
  namespace: "kube-ops"
  labels:
    jenkins: "slave"
    jenkins/label: "jenkins-jnlp"
spec:
  containers:
  - name: "jnlp"
    image: "hub.gitee.cc/kubernetes/jenkins-salve:latet-jdk11"
    imagePullPolicy: "IfNotPresent"
    tty: true
    resources:
      limits:
        cpu: 1000m
        memory: 2000Mi
      requests:
        cpu: 200m
        memory: 400Mi
    env:
    - name: "DOCKER_HOST"
      value: "tcp://192.168.1.141:2375"
    - name: "JENKINS_AGENT_WORKDIR"
      value: "/home/jenkins/agent"
    volumeMounts:
    - mountPath: "/root/.kube/config"
      name: "volume-0"
      readOnly: false
      subPath: "config"
    - name: localtime
      mountPath: /etc/localtime
    - mountPath: "/home/jenkins/agent"
      name: "workspace-volume"
      readOnly: false
    workingDir: "/home/jenkins/agent"

  volumes:
    - name: localtime
      hostPath:
        path: /usr/share/zoneinfo/Asia/Shanghai
    - configMap:
        name: "kubeconfig"
        optional: false
      name: "volume-0"
    - emptyDir:
        medium: ""
      name: "workspace-volume"
  restartPolicy: Always
  hostNetwork: false
  imagePullSecrets:
  - name: "regcred"
  nodeSelector:
    build: "true"
            '''
        }
    }
    stages {
        stage('Deploying to K8s') {
          steps {
            container(name: 'jnlp') {
               sh """#!/bin/bash
                   set -ux
                   echo ${IMAGE_TAG}		# 该变量即为前台选择的镜像
                   kubectl set image deployment -l app=${IMAGE_NAME} ${IMAGE_NAME}=${HARBOR_ADDRESS}/${IMAGE_TAG} -n ${NAMESPACE}
                   kubectl get pod  -l app=${IMAGE_NAME} -n ${NAMESPACE}
                   kubectl get deploy/${IMAGE_NAME} -o yaml  -n ${NAMESPACE}|grep "image: "
               """
          }
        }
     }
  }
}
```

保存后，选择一个镜像，单击Build，结果如下图所示。

可以看到是直接将镜像版本更新至Kubernetes，并无构建过程，可以省下很多时间。该流水线也可以选择之前的版本进行回滚操作。

查看部署过程

![](./assets/6-71.png){: .zoom}

> "参考文献"：
>
> [基于k8s的DevOps环境构建](https://zhangzhuo.ltd/articles/2022/06/04/1654333328076.html)
>
> [Jenkins的流水线详解](https://zhangzhuo.ltd/articles/2022/06/04/1654333399919.html)
>
> [DevOps自动化构建应用](https://zhangzhuo.ltd/articles/2022/06/05/1654424805227.html)




# 10. vscode Jenkins插件分享

Jenkins Pipeline Linter Connector

安装方式：在Vscode扩展商店当中直接搜索即可安装。

功能说明：Jenkinsfile 语法检测，非常方便。


## 10.1 在VSCode中校验您的Jenkinsfile

您可以在 VS Code 插件浏览器中或听过下面的地址 url: https://marketplace.visualstudio.com/items?itemName=janjoerke.jenkins-pipeline-linter-connector 找到插件。该插件会在 VS Code 中添加四项配置，您需要配置用来执行检验的 Jenkins 。

- jenkins.pipeline.linter.connector.url 是您的 Jenkins 期待的 POST 请求地址，包含您要校验的 Jenkinsfile 文件。通常为`http://<your_jenkins_server:port>/pipeline-model-converter/validate`。

- jenkins.pipeline.linter.connector.user 允许您指定您的 Jenkins 用户名。

- jenkins.pipeline.linter.connector.pass 允许您指定您的 Jenkins 密码。

- jenkins.pipeline.linter.connector.crumbUrl 当您的 Jenkins 启用了 CRSF 时必须指定。通常为`http://<your_jenkins_server:port>/crumbIssuer/api/xml?xpath=concat(//crumbRequestField,%22:%22,//crumb)`。

> 参考文献： "在 VS Code 中校验您的 Jenkinsfile "
>
> [VScode本地进行Jenkinsfile语法验证](https://mp.weixin.qq.com/s/DTssLecwkNwaNoXUxGSLYg)
>
> [在 VS Code 中校验您的 Jenkinsfile ](https://www.jenkins.io/zh/blog/2018/11/07/Validate-Jenkinsfile)

# 11. 小结

本章主要演示了Java、Vue前端、Golang的自动化构建配置，演示了容器化业务部署的不同方式，均使用Jenkins新特性声明式流水线进行持续集成和持续部署，在实际使用时不一定非要使用流水线进行构建，也可以根据自己的业务场景选择其他风格的构建方式。

通过多个例子可以看出，使用流水线进行构建时，大致流程是类似的，主要是编译方式、基础镜像和编译命令不一样，可以按照自己的业务场景和需求进行修改。

为了使Jenkinsfile更加灵活和统一，可以对Jenkinsfile完全参数化，将Jenkinsfile所有可变的赋值都改为变量，然后在Jenkins上配
置对应的变量即可，这样可以使Jenkins的构建变得更加灵活，也使得同一个Jenkinsfile适用于所有的项目。

# 参考文献


[Jenkins常用的优化功能 | Mr.Pu 个站博客 (putianhui.cn)](https://www.putianhui.cn/posts/f5b12f59c480/)

[Jenkins自动化部署dev-uat-preprd-prd环境服务，并实现发布失败回滚。 | Mr.Pu 个站博客 (putianhui.cn)](https://www.putianhui.cn/posts/7041616304b2/)

[Jenkinsfile解读详解 | Mr.Pu 个站博客 (putianhui.cn)](https://www.putianhui.cn/posts/fa563716f116/)