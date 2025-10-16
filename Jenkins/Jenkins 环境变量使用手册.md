# Jenkins 环境变量使用手册

## 前言

Jenkins 环境变量是构建过程中自动生成或配置的动态值，包含了构建上下文、版本控制、执行环境等关键信息。合理使用这些变量可大幅提升 CI/CD 流程的灵活性与自动化程度，尤其适用于多分支项目、变更请求（如 Pull Request）和版本管理场景。

## 一、分支与变更请求相关变量

### 1.1 基础分支信息

| 变量名              | 描述                                                         | 适用场景                               |
| ------------------- | ------------------------------------------------------------ | -------------------------------------- |
| `BRANCH_NAME`       | 当前构建的分支名（如`master`、`feature/login`）；若为 PR，可能显示为`PR-123`等临时名称 | 分支策略判断（如 master 分支部署生产） |
| `BRANCH_IS_PRIMARY` | 标识当前分支是否为 "主分支"（如`master`/`main`），值为`true`或未设置 | 主分支特殊处理（如生成正式版本）       |
| `TAG_NAME`          | 当前构建的标签名（如`v1.0.0`），仅在构建标签时生效           | 版本发布（根据标签名生成版本号）       |
| `TAG_TIMESTAMP`     | 标签的时间戳（毫秒级 Unix 时间戳）                           | 按标签时间排序或命名产物               |
| `TAG_UNIXTIME`      | 标签的时间戳（秒级 Unix 时间戳）                             | 同上                                   |
| `TAG_DATE`          | 标签的格式化时间（如`Wed Jan 1 00:00:00 UTC 2020`）          | 构建日志中显示标签创建时间             |

### 1.2 变更请求（如 Pull Request）信息

| 变量名                | 描述                                                         | 适用场景                         |
| --------------------- | ------------------------------------------------------------ | -------------------------------- |
| `CHANGE_ID`           | 变更请求的唯一标识（如 GitHub 的 PR 编号`123`）              | 关联 PR 与构建结果               |
| `CHANGE_URL`          | 变更请求的网页链接（如`https://github.com/owner/repo/pull/123`） | 自动向 PR 添加评论（如测试结果） |
| `CHANGE_TITLE`        | 变更请求的标题（如 PR 的标题文本）                           | 构建日志中展示 PR 标题           |
| `CHANGE_AUTHOR`       | 变更作者的用户名（如 GitHub 用户名）                         | 通知作者构建结果                 |
| `CHANGE_AUTHOR_EMAIL` | 变更作者的邮箱地址                                           | 邮件通知作者                     |
| `CHANGE_TARGET`       | 变更的目标分支（如 PR 的 base 分支`master`）                 | 基于目标分支执行合并测试         |
| `CHANGE_BRANCH`       | 变更的源分支（如 PR 的 head 分支`feature/login`）            | 拉取源分支代码进行构建           |
| `CHANGE_FORK`         | 变更来源的 fork 仓库名（如`contributor/repo`）               | 处理跨仓库 PR 的权限或路径问题   |

## 二、构建元数据与标识变量

### 2.1 构建基础信息

| 变量名               | 描述                                                         | 适用场景                               |
| -------------------- | ------------------------------------------------------------ | -------------------------------------- |
| `BUILD_NUMBER`       | 当前构建的编号（如`153`），按任务自增                        | 版本号生成（如`v1.0.${BUILD_NUMBER}`） |
| `BUILD_ID`           | 构建 ID，1.597 + 版本与`BUILD_NUMBER`一致，旧版本为`YYYY-MM-DD_hh-mm-ss`格式 | 构建唯一标识                           |
| `BUILD_DISPLAY_NAME` | 构建的显示名称（默认`#153`，可自定义）                       | 日志或 UI 中展示友好名称               |
| `BUILD_TAG`          | 格式化标签，格式为`jenkins-${JOB_NAME}-${BUILD_NUMBER}`，含任务名和编号 | 产物命名（如`app-${BUILD_TAG}.jar`）   |
| `JOB_NAME`           | 任务名称（如`backend-service`或`folder/backend-service`）    | 区分不同任务的构建逻辑                 |
| `JOB_BASE_NAME`      | 任务短名称（去除路径，如`backend-service`）                  | 简化产物或日志的命名                   |

### 2.2 路径与目录变量

| 变量名          | 描述                                                         | 适用场景                       |
| --------------- | ------------------------------------------------------------ | ------------------------------ |
| `WORKSPACE`     | 工作区绝对路径（如`/var/jenkins_home/workspace/backend-service`） | 执行脚本或存放临时文件         |
| `WORKSPACE_TMP` | 工作区附近的临时目录（需手动创建，如`mkdir -p $WORKSPACE_TMP`） | 存放非持久化临时文件（如缓存） |
| `JENKINS_HOME`  | Jenkins 主目录绝对路径（如`/var/jenkins_home`）              | 访问 Jenkins 配置文件或工具    |

## 三、URL 与 UI 相关变量

| 变量名                      | 描述                                                         | 适用场景               |
| --------------------------- | ------------------------------------------------------------ | ---------------------- |
| `JENKINS_URL`               | Jenkins 实例根 URL（如`http://jenkins:8080/`，需在系统配置中设置） | 生成构建相关链接       |
| `BUILD_URL`                 | 当前构建的 URL（如`http://jenkins:8080/job/backend-service/153/`） | 通知中嵌入构建结果链接 |
| `JOB_URL`                   | 任务的 URL（如`http://jenkins:8080/job/backend-service/`）   | 跳转至任务主页         |
| `RUN_DISPLAY_URL`           | 构建页面的重定向 URL（兼容不同 UI）                          | 跨 UI 平台的链接跳转   |
| `RUN_ARTIFACTS_DISPLAY_URL` | 构建产物页面的重定向 URL                                     | 通知中嵌入产物下载链接 |
| `RUN_CHANGES_DISPLAY_URL`   | 构建变更记录页面的重定向 URL                                 | 查看代码变更详情       |
| `RUN_TESTS_DISPLAY_URL`     | 测试结果页面的重定向 URL                                     | 查看测试报告           |

## 四、Git 与源代码管理（SCM）变量

| 变量名                           | 描述                                                         | 适用场景                         |
| -------------------------------- | ------------------------------------------------------------ | -------------------------------- |
| `GIT_COMMIT`                     | 当前构建的 Git 提交哈希（如`a1b2c3d4...`）                   | 代码版本锁定或回溯               |
| `GIT_PREVIOUS_COMMIT`            | 上一次构建的提交哈希（若存在）                               | 增量构建（如`git diff`对比变更） |
| `GIT_PREVIOUS_SUCCESSFUL_COMMIT` | 上一次成功构建的提交哈希（若存在）                           | 失败时对比成功版本找问题         |
| `GIT_BRANCH`                     | 远程分支名（如`origin/master`）                              | 基于远程分支判断构建策略         |
| `GIT_LOCAL_BRANCH`               | 本地检出的分支名（如`master`）                               | 本地代码操作（如合并、提交）     |
| `GIT_CHECKOUT_DIR`               | 代码检出的子目录路径（若配置 "Checkout to a sub-directory"） | 定位代码目录                     |
| `GIT_URL`                        | 远程仓库 URL（如`https://github.com/owner/repo.git`），多仓库时为`GIT_URL_1`、`GIT_URL_2`等 | 拉取代码或关联仓库信息           |
| `GIT_COMMITTER_NAME`             | 全局配置的 Git 提交者名称（来自 Jenkins 系统设置）           | 自动提交时指定作者               |
| `GIT_AUTHOR_EMAIL`               | 全局配置的 Git 作者邮箱（来自 Jenkins 系统设置）             | 提交记录中展示作者信息           |

## 五、执行环境变量

| 变量名            | 描述                                                        | 适用场景                                                 |
| ----------------- | ----------------------------------------------------------- | -------------------------------------------------------- |
| `NODE_NAME`       | 执行构建的节点名称（如`master`或`agent-linux`）             | 区分节点执行不同命令（如 Windows/Linux 脚本）            |
| `NODE_LABELS`     | 节点的标签列表（空格分隔，如`linux docker`）                | 基于标签选择构建步骤（如需要`docker`标签才执行容器命令） |
| `EXECUTOR_NUMBER` | 当前节点上的执行器编号（从 0 开始），同一节点多执行器时区分 | 避免多执行器资源冲突（如端口占用）                       |
| `CI`              | 固定值`true`，标识当前环境为 CI 环境                        | 脚本中区分本地 / CI 环境（如跳过本地不需要的步骤）       |

## 六、使用示例

### 6.1 分支策略控制部署

groovy











```groovy
pipeline {
  agent any
  stages {
    stage('Deploy') {
      steps {
        script {
          if (env.BRANCH_NAME == 'master') {
            sh 'deploy.sh production'  // master分支部署生产
          } else if (env.BRANCH_NAME == 'develop') {
            sh 'deploy.sh staging'     // develop分支部署测试
          } else if (env.CHANGE_ID) {
            sh 'deploy.sh preview'     // PR构建部署预览环境
          }
        }
      }
    }
  }
}
```

### 6.2 PR 自动评论测试结果

groovy











```groovy
pipeline {
  agent any
  stages {
    stage('Test') {
      steps {
        sh 'pytest --junitxml=report.xml'
      }
      post {
        success {
          // 向PR添加成功评论
          sh "curl -X POST ${CHANGE_URL}/comments -d '✅ 测试通过（构建#${BUILD_NUMBER}）'"
        }
        failure {
          // 向PR添加失败评论
          sh "curl -X POST ${CHANGE_URL}/comments -d '❌ 测试失败（构建#${BUILD_NUMBER}）'"
        }
      }
    }
  }
}
```

### 6.3 生成带版本信息的产物

bash

```bash
# Shell脚本中使用变量
ARTIFACT_NAME="app-${JOB_BASE_NAME}-v1.0.${BUILD_NUMBER}.jar"
mv target/app.jar "${ARTIFACT_NAME}"
echo "生成产物：${ARTIFACT_NAME}"
```

### 6.4 增量代码检查

groovy











```groovy
pipeline {
  agent any
  stages {
    stage('Lint') {
      steps {
        script {
          if (env.GIT_PREVIOUS_COMMIT) {
            // 仅检查上一次构建后的变更文件
            sh "flake8 \$(git diff --name-only ${GIT_PREVIOUS_COMMIT} ${GIT_COMMIT} | grep '.py$')"
          } else {
            // 首次构建检查所有文件
            sh "flake8"
          }
        }
      }
    }
  }
}
```

## 七、注意事项

1. **变量可用性**：部分变量仅在特定场景下存在（如`CHANGE_*`仅 PR 构建有效，`TAG_*`仅标签构建有效），使用前建议通过`if (env.VAR_NAME)`判断是否存在。
2. **敏感信息**：避免在日志中打印敏感变量（如通过 Credentials 插件管理的密钥），Jenkins 会自动屏蔽凭据变量的输出。
3. **版本兼容性**：部分变量（如`BRANCH_IS_PRIMARY`）需特定 Jenkins 版本或插件支持，建议通过`env`命令（如`sh 'env'`）在构建中确认变量是否存在。
4. **变量引用方式**：在 Pipeline 脚本中需用`env.VAR_NAME`或`${VAR_NAME}`（双引号内），在 Shell 中用`$VAR_NAME`，在 Windows 批处理中用`%VAR_NAME%`。

## 附录：查看当前构建的环境变量

在构建步骤中添加以下命令，可打印所有可用环境变量供调试：



- Shell：`sh 'env | sort'`
- Windows 批处理：`cmd /c 'set'`
- Pipeline 脚本：`echo "${env.getEnvironment().collect { it.key + '=' + it.value }.sort().join('\n')}"`