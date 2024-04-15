---
title: 项目开发-Git操作篇
tags: [git]
date: 2023-04-21 11:36:21
categories: technique
urlname: 18
---

### 一些资源

- Git书籍[Pro Git book][1]
- 学习Git操作的在线游戏小网站 [Learn Git Branching][2]
- 工作中常用的Git操作[Git常用操作总结][3]



### Git流程规范

  业务层服务后端使用Git进行版本控制需要遵循一定的工作流程，以下是一种常用的流程。

  > **分支命名方法**

  1. 生产环境使用 master 分支，测试环境使用 dev 分支
  2. 其它开发分支命名规则如下：
   - bug 修复分支
     
     bugfix-项目名称-描述 如：bugfix-feature-description
   - 功能开发分支 
     
     feat-项目名称-描述 如：feat-feature-description

  长期分支存在两个：用于生产环境的 master 分支，用于测试环境的 dev 分支

  临时分支有三类：用于新增功能的 feat- 分支， 用于 bug 修复的 bugfix- 分支，用于发布前的 release-* 分支


  > **新功能开发**
  1. 从线上主干拉取最新的稳定代码 
  
  ```
  拉取最新的稳定主干分支
  git checkout master
  git pull origin master

  创建对应功能分支
  git checkout -b feat-feature-description

  完成代码开发
  git push origin feat-feature-description
  ```

  1. 合并功能分支至测试环境 dev 分支
   
   ```
   git checkout dev
   git merge feat-feature-description
   git push origin dev
   ```

  2. 测试通过，创建发布分支
   
   ```
   git checkout master
   git checkout -b release-*
   git merge feat-feature-description
   git merge feat-*
   ...
   git checkout master
   git merge release-*
   git push origin master
   git checkout dev
   ```

  3. 删除相应的临时分支
   
   ```
   git push origin :feat-feature-description
   git push origin :release-*
   ...
   git branch -d feat-feature-description
   git branch -d release-*
   ```
   
  > **缺陷修复**

    
   在第二步时创建对应的 bugfix-* 分支，其它步骤同功能开发。
   ```
   git checkout master
   git pull origin master
   git checkout -b bugfix-*
   ...
   git checkout dev
   git merge bugfix-*
   git push origin dev
   ...
   git checkout master
   git merge bugfix-*
   git push origin master
   ...
   ```


> **代码检查**
  
   增加 pre-commit 钩子进行提交前代码检查
   对 imports 进行更新，添加需引入的依赖，去除不使用的依赖
   对代码风格进行检查
   对代码进行格工化
   对代码语法进行静态检查
   运行单元测试

  - 安装依赖的工具
    ```
    go get golang.org/x/tools/cmd/goimports
    go install golang.org/x/tools/cmd/goimports

    go get golang.org/x/lint/golint
    go install golang.org/x/lint/golint
    ```
  - pre-commit文件内容
    ```
    #!/bin/sh
     
    STAGED_GO_FILES=$(git diff --cached --name-only --diff-filter=ACM | grep ".go$")
     
    if [[ "$STAGED_GO_FILES" = "" ]]; then
        exit 0
    fi
     
    PASS=true
     
    for FILE in $STAGED_GO_FILES
    do
        # 跳过vendor目录下的文件
        if [[ $FILE == "vendor"* ]];then
            continue
        fi
     
        # goimports 检查并调整导入语句
        goimports -w $FILE
        if [[ $? != 0 ]]; then
            PASS=false
        fi
     
        # golint 检查代码风格,给出提示
        golint "-set_exit_status" $FILE
        if [[ $? == 1 ]]; then
            PASS=false
        fi
     
        # go tool vet 检查代码中的静态错误
        # go vet $FILE
        # if [[ $? != 0 ]]; then
        #     PASS=false
        # fi
     
        # 如果当前文件没有被格式化，就格式化它
        UNFORMATTED=$(gofmt -l $FILE)
        if [[ "$UNFORMATTED" != "" ]];then
            gofmt -w $PWD/$UNFORMATTED
            if [[ $? != 0 ]]; then
                PASS=false
            fi
        fi
     
        # 上述 goimports, gofmt可能会对文件作出改动，
        # 所以此处将更改提交至暂存区
        git add $FILE
     
    done
     
    if ! $PASS; then
        printf "\033[31mCOMMIT FAILED \033[0m\n"
        exit 1
    else
        printf "\033[32mCOMMIT SUCCEEDED \033[0m\n"
    fi
     
    exit 0

    ```


  > **需要注意**
  - 严禁直接在 master 分支提交代码
  - 严禁直接在 dev 分支提交代码
  - 在 dev 分支因调试进行的代码修改必须切回对应的功能或缺陷修复分支后进行提交
  - 创建发布分支是为了避免有些开发好的功能因特殊原因取消上到正式环境的情况，避免手工剔除代码，如果功能确认上到正式环境，可省略这一步
  - 临时分支在开发完成上到正式环境 稳定后（最多一周）必须删除，防止分支堆积造成混淆
  - 不要把调试代码签入版本控制
  - git log 需要明确写出更改的部分
  - 合并冲突问题本地解决












[1]: https://git-scm.com/book/en/v2
[2]: https://learngitbranching.js.org/?locale=zh_CN
[3]: https://juejin.cn/post/6844903586120335367?searchId=202307211637490692F188E23F90BD2F63













