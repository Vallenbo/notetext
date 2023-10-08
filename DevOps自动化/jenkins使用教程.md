# 什么是 Jenkins Pipeline?

Jenkins Pipeline（或简称为 "Pipeline"）是一套插件，将持续交付的实现和实施集成到 Jenkins 中。

持续交付 Pipeline 自动化的表达了这样一种流程：将基于版本控制管理的软件持续的交付到您的用户和消费者手中。

Jenkins Pipeline 提供了一套可扩展的工具，用于将“简单到复杂”的交付流程实现为“持续交付即代码”。Jenkins Pipeline 的定义通常被写入到一个文本文件（称为 `Jenkinsfile` ）中，该文件可以被放入项目的源代码控制库中。

# 熟悉pipline基本流程

## 执行单个步骤

```groovy
pipeline {
    agent { docker 'python:3.5.1' }
    stages {
        stage('build') {
            steps {
                sh 'python --version'
            }
        }
    }
}
```



## 执行多个步骤（step）

Pipelines 由多个步骤（step）组成，允许你构建、测试和部署应用。 Jenkins Pipeline 允许您使用一种简单的方式组合多个步骤， 以帮助您实现多种类型的自动化构建过程。

可以把“步骤（step）”看作一个执行单一动作的单一的命令。 当一个步骤运行成功时继续运行下一个步骤。 当任何一个步骤执行失败时，Pipeline 的执行结果也为失败。

当所有的步骤都执行完成并且为成功时，Pipeline 的执行结果为成功。

在 Linux、BSD 和 Mac OS（类 Unix ) 系统中的 shell 命令， 对应于 Pipeline 中的一个 `sh` 步骤（step）。

```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh 'echo "Hello World"'
                sh '''
                    echo "Multiline shell steps works too"
                    ls -lah
                '''
            }
        }
    }
}
```

## 超时、重试和更多

Jenkins Pipeline 提供了很多的步骤（step），这些步骤可以相互组合嵌套，方便地解决像重复执行步骤直到成功（重试）和如果一个步骤执行花费的时间太长则退出（超时）等问题。

```groovy
pipeline {
    agent any
    stages {
        stage('Deploy') {
            steps {
                retry(3) {
                    sh './flakey-deploy.sh'
                }

                timeout(time: 3, unit: 'MINUTES') {
                    sh './health-check.sh'
                }
            }
        }
    }
}
```

“Deploy”阶段（stage）重复执行 `flakey-deploy.sh` 脚本3次，然后等待 `health-check.sh` 脚本最长执行3分钟。 如果 `health-check.sh` 脚本在 3 分钟内没有完成，Pipeline 将会标记在“Deploy”阶段失败。

内嵌类型的步骤，例如 `timeout` 和 `retry` 可以包含其他的步骤，包括 `timeout` 和 `retry` 。

我们也可以组合这些步骤。例如，如果我们想要重试部署任务 5 次，但是总共花费的时间不能超过 3 分钟。

```groovy
pipeline {
    agent any
    stages {
        stage('Deploy') {
            steps {
                timeout(time: 3, unit: 'MINUTES') {
                    retry(5) {
                        sh './flakey-deploy.sh'
                    }
                }
            }
        }
    }
}
```



## 完成时动作

当 Pipeline 运行完成时，你可能需要做一些清理工作或者基于 Pipeline 的运行结果执行不同的操作， 这些操作可以放在 `post` 部分。

```groovy
pipeline {
    agent any
    stages {
        stage('Test') {
            steps {
                sh 'echo "Fail!"; exit 1'
            }
        }
    }
    post {
        always {
            echo 'This will always run'
        }
        success {
            echo 'This will run only if successful'
        }
        failure {
            echo 'This will run only if failed'
        }
        unstable {
            echo 'This will run only if the run was marked as unstable'
        }
        changed {
            echo 'This will run only if the state of the Pipeline has changed'
            echo 'For example, if the Pipeline was previously failing but is now successful'
        }
    }
}
```

# 定义执行环境

在 [上一小节](https://www.jenkins.io/zh/doc/pipeline/tour/running-multiple-steps) 您可能已经注意到每个示例中的 `agent` 指令。 `agent` 指令告诉Jenkins在哪里以及如何执行Pipeline或者Pipeline子集。 正如您所预料的，所有的Pipeline都需要 `agent` 指令。

在执行引擎中，`agent` 指令会引起以下操作的执行：

- 所有在块block中的步骤steps会被Jenkins保存在一个执行队列中。 一旦一个执行器 [executor](https://www.jenkins.io/zh/doc/pipeline/tour/agents/#../../book/glossary/#executor) 是可以利用的，这些步骤将会开始执行。
- 一个工作空间 [workspace](https://www.jenkins.io/zh/doc/pipeline/tour/agents/#../../book/glossary/#workspace) 将会被分配， 工作空间中会包含来自远程仓库的文件和一些用于Pipeline的工作文件

在Pipeline中可以使用这几种 [定义代理的方式](https://www.jenkins.io/doc/book/pipeline/syntax#agent) 在本导读中，我们仅使用Docker容器的代理方式。

在Pipeline中可以很容易的运行 [Docker](https://docs.docker.com/) 镜像和容器。 Pipeline可以定义命令或者应用运行需要的环境和工具， 不需要在执行代理中手动去配置各种各样的系统工具和依赖。 这种方式可以让你使用 [Docker容器工具包](http://hub.docker.com/) 中的任何工具。

```groovy
pipeline {
    agent {
        docker { image 'node:7-alpine' }
    }
    stages {
        stage('Test') {
            steps {
                sh 'node --version'
            }
        }
    }
}
```

当执行Pipeline时，Jenkins将会自动运行指定的容器，并执行Pipeline中已经定义好的步骤steps：

```
[Pipeline] stage
[Pipeline] { (Test)
[Pipeline] sh
[guided-tour] Running shell script
+ node --version
v7.4.0
[Pipeline] }
[Pipeline] // stage
[Pipeline] }
```

在Pipeline中，混合和搭配不同的容器或者其他代理可以获得更大的灵活性。

# 使用环境变量

环境变量可以像下面的示例设置为全局的，也可以是阶段（stage）级别的。 如你所想，阶段（stage）级别的环境变量只能在定义变量的阶段（stage）使用。

```groovy
pipeline {
    agent any

    environment {
        DISABLE_AUTH = 'true'
        DB_ENGINE    = 'sqlite'
    }

    stages {
        stage('Build') {
            steps {
                sh 'printenv'
            }
        }
    }
}
```

这种在 `Jenkinsfile` 中定义环境变量的方法对于指令性的脚本定义非常有用方便， 比如 `Makefile` 文件，可以在 Pipeline 中配置构建或者测试的环境，然后在 Jenkins 中运行。

环境变量的另一个常见用途是设置或者覆盖构建或测试脚本中的凭证。 因为把凭证信息直接写入 `Jenkinsfile` 很显然是一个坏主意， Jenkins Pipeline 允许用户快速安全地访问在 `Jenkinsfile` 中预定义的凭证信息，并且无需知道它们的值。

# 记录测试和构建结果 

虽然测试是良好的持续交付过程中的关键部分，但大多数人并不希望筛选数千行控制台输出来查找有关失败测试的信息。 为了简化操作，只要您的测试运行时可以输出测试结果文件，Jenkins 就可以记录和汇总这些测试结果。 Jenkins 通常与 `junit` 步骤捆绑在一起，但如果您的测试运行结果无法输出 JUnit 样式的 XML 报告， 那么还有其他插件可以处理任何广泛使用的测试报告格式。

为了收集我们的测试结果，我们将使用 `post` 部分。

```groovy
pipeline {
    agent any
    stages {
        stage('Test') {
            steps {
                sh './gradlew check'
            }
        }
    }
    post {
        always {
            junit 'build/reports/**/*.xml'
        }
    }
}
```

这将会获得测试结果，Jenkins 会持续跟踪并计算测试的趋势和结果。 如果存在失败的测试用例，Pipeline 会被标记为 “UNSTABLE”，在网页上用黄色表示， 这不同于使用红色表示的 “FAILED” 失败状态。

当出现测试失败时，通常可以从 Jenkins 中获取构建结果报告进行本地分析和测试。 Jenkins 内置支持存储构建结果报告，在 Pipeline 执行期间生成记录文件。

通过 `archiveArtifacts` 步骤和文件匹配表达式可以很容易的完成构建结果记录和存储， 如下例所示：

```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh './gradlew build'
            }
        }
        stage('Test') {
            steps {
                sh './gradlew check'
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'build/libs/**/*.jar', fingerprint: true
            junit 'build/reports/**/*.xml'
        }
    }
}
```

如果在 `archiveArtifacts` 步骤中指定了多个参数， 那么每个参数的名称必须在步骤代码中明确指定， 即文件的路径、文件名和 `fingerprint` 三个参数。 如果您只需指定文件的路径和文件名， 那么你可以省略参数名称 `artifacts` ，例如： `archiveArtifacts 'build/libs/**/*.jar'`

在 Jenkins 中记录测试和构建结果非常有助于向团队成员快速提供相关信息。 在下一节，我们将会展示如何通知团队成员在我们的 Pipeline 中所发生的事情。

# 清理和通知 

因为 `post` 部分保证在 Pipeline 结束的时候运行， 所以我们可以添加通知或者其他的步骤去完成清理、通知或者其他的 Pipeline 结束任务。

```groovy
pipeline {
    agent any
    stages {
        stage('No-op') {
            steps {
                sh 'ls'
            }
        }
    }
    post {
        always {
            echo 'One way or another, I have finished'
            deleteDir() /* clean up our workspace */
        }
        success {
            echo 'I succeeeded!'
        }
        unstable {
            echo 'I am unstable :/'
        }
        failure {
            echo 'I failed :('
        }
        changed {
            echo 'Things were different before...'
        }
    }
}
```

有很多方法可以发送通知， 下面是一些示例展示了如何通过电子邮件、Hipchat room 或者 Slack channel 发送 Pipeline 的相关信息。

### 电子邮件

```groovy
post {
    failure {
        mail to: 'team@example.com',
             subject: "Failed Pipeline: ${currentBuild.fullDisplayName}",
             body: "Something is wrong with ${env.BUILD_URL}"
    }
}
```

### Hipchat

```groovy
post {
    failure {
        hipchatSend message: "Attention @here ${env.JOB_NAME} #${env.BUILD_NUMBER} has failed.",
                    color: 'RED'
    }
}
```

### Slack

```groovy
post {
    success {
        slackSend channel: '#ops-room',
                  color: 'good',
                  message: "The pipeline ${currentBuild.fullDisplayName} completed successfully."
    }
}
```

当 Pipeline 失败、不稳定甚至是成功时，团队都会收到通知， 现在我们可以继续完成我们的持续交付 Pipeline 激动人心的部分：部署！

# 部署

大多数最基本的持续交付 Pipeline 至少会有三个阶段：构建、测试和部署，这些阶段被定义在 `Jenkinsfile` 中。 这一小节我们将主要关注部署阶段，但应该指出稳定的构建和测试阶段是任何部署活动的重要前提。

```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo 'Building'
            }
        }
        stage('Test') {
            steps {
                echo 'Testing'
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying'
            }
        }
    }
}
```

## 阶段即为部署环境

一个常见的模式是扩展阶段的数量以获取额外的部署环境信息， 如 “staging” 或者 “production”，如下例所示。

```
stage('Deploy - Staging') {
    steps {
        sh './deploy staging'
        sh './run-smoke-tests'
    }
}
stage('Deploy - Production') {
    steps {
        sh './deploy production'
    }
}
```

在这个示例中，我们假定 `./run-smoke-tests` 脚本所运行的冒烟测试足以保证或者验证可以发布到生产环境。 这种可以自动部署代码一直到生产环境的 Pipeline 可以认为是“持续部署”的一种实现。 虽然这是一个伟大的想法，但是有很多理由表明“持续部署”不是一种很好的实践， 即便如此，这种方式仍然可以享有“持续交付”带来的好处。 

## 人工确认

通常在阶段之间，特别是不同环境阶段之间，您可能需要人工确认是否可以继续运行。 例如，判断应用程序是否在一个足够好的状态可以进入到生产环境阶段。 这可以使用 `input` 步骤完成。 在下面的例子中，“Sanity check” 阶段会等待人工确认，并且在没有人工确认的情况下不会继续执行。

```groovy
pipeline {
    agent any
    stages {
        /* "Build" and "Test" stages omitted */

        stage('Deploy - Staging') {
            steps {
                sh './deploy staging'
                sh './run-smoke-tests'
            }
        }

        stage('Sanity check') {
            steps {
                input "Does the staging environment look ok?"
            }
        }

        stage('Deploy - Production') {
            steps {
                sh './deploy production'
            }
        }
    }
}
```

## 结论

本导读向您介绍了 Jenkins 和 Jenkins Pipeline 的基本使用。 由于 Jenkins 是非常容易扩展的，它可以被修改和配置去处理任何类型的自动化。