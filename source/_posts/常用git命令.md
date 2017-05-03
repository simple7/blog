---
title: 常用git命令
date: 2017-05-02 10:55:30
tags: [git]
---
### 新建代码库
    # 在当前目录新建一个git代码库
    git init
      
    # 下载一个项目和它的整个历史
    git clone [url]
    
### 分支
    # 列出所有分支
    git branck -a
    
    # 新建一个分支并切换到该分支
    git checkout -b [branch]
    
    # 合并指定分支到当前分支
    git merge [branch]
    
    # 新建一个分支，指向指定commit
    git branch [branch] [commit]

### 标签
    # 列出所有tag
    git tag
    
    # 新建一个tag在当前commit
    git tag [tag]
    
    # 新建一个tag在指定commit
    git tag [tag] [commit]
    
    # 删除本地tag
    git tag -d [tag]
    
    # 删除远程tag
    git push origin :refs/tags/[tagName]
    
    # 查看tag信息
    git show [tag]

    # 提交指定tag
    git push [remote] [tag]
    
    # 提交所有tag
    git push [remote] --tags
    
    # 新建一个分支，指向某个tag
    git checkout -b [branch] [tag]
### 查看信息
    # 显示有变更的文件
    git status
    
    # 显示当前分支的版本历史
    git log
    
    # 显示过去5次提交
    git log -5 --pretty --oneline
    
### 远程同步
    # 下载远程仓库的所有变动
    git fetch [remote]
    
    # 取回远程仓库的变化，并与本地分支合并
    git pull [remote] [branch]
    
    # 上传本地指定分支到远程仓库
    git push [remote] [branch]
    
    # 强行推送当前分支到远程仓库，即使有冲突
    git push [remote] --force
    
    # 推送所有分支到远程仓库
    git push [remote] --all
    
### 其他
    # 生成一个可供发布的压缩包
    git archive
