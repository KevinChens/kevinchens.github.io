# Git的正确使用姿势与最佳实践


## Git的正确使用姿势与最佳实践 | 青训笔记

这是我参与「第三届青训营 -后端场」笔记创作活动的的第5篇笔记。   

本篇主要内容是Git的基本使用和常用命令的总结。  


### Git是什么
Git是最流行的，最常用的分布式版本控制软件。  
而版本控制是一种记录一个或若干文件内容变化，以便将来查阅特定版本修订情况的系统。  
版本控制能够帮助我们更好地关注变更，了解到每个版本的改动是什么，方便对改动的代码进行检查，预防事故的发生；也能够随时切换到不同的版本，回滚误删改的问题代码。  
为什么要学习Git   
- 协同工作：必备技能，业界大多数公司都是基于Git进行代码管理；  
- 开源社区：大多数开源社区都是基于Git维护，参与这些项目的开发需要Git。  

### Git基本使用方式
**Git的基本命令**  
项目初始化：  
```shell
mkdir study
cd study
git init
vim README.md
git add README.md
git branch -M main
git commit -m "add README"
```
本地配置：  
```shell
// 设置用户签名
git config -global user.name 用户名   
git config -global user.email 邮箱 
// instead of配置
git config --global url.git@github.com:.insteadOf https://github.com/
// 别名配置
git config --global alias.cin "commit --amend --no-edit"
// 查看详细使用方法
git help config    
```
查看remote：  
```shell
git remote -v
// 添加remote
git remote add origin_ssh git@github.com:git/git.git
git remote add origin_http https://github.com/git/git.git
// 同一个origin设置不同的push和fetch url
git remote add origin git@github.com:git/git
git remote set-url --add --push origin git@github.com:MY_REPOSITY/git
// 删除remote
git remote remove alias
```
提交代码：  
```shell
git add .
git commit -m "comment"
// 修改commit message
git commit --amend
```
远端同步：  
```shell
// 拉取代码
git clone addr
git pull origin main
// 拉取远端代码，不合并到当前分支
git fetch origin main
// 推送代码
git push origin main
```
Git目录：`tree .git`  
rebase：  
```shell
// 实现对最近三个commit的修改
git rebase -i HEAD~3
// 1. 合并commit
// 2. 修改具体的commit message
// 3. 删除某个commit
```
分支操作：  
```shell
git branch -v         
git branch --all     
// 增删分支
git branch branchname
git branch -d branchname  
git branch hot-fix
// 切换分支
git checkout 分支名  
// 合并分支，在main下进行
git merge 分支名     
```
删除文件：  
```shell
// 预览将要删除的文件，加上-n这个参数，不会删除任何文件，而是展示此命令要删除的文件列表预览
git rm -r -n --cached fileName  
//确定无误后，删除文件
git rm -r --cached fileName
//提交到本地并推送到远程服务器
git commit -m "comment"
git push origin main
```
常见问题：  
1. 为什么配置了Git，依然不能拉取代码  
免密认证没有配置；  
instead of没有配置，配的SSH认证使用的是HTTP认证。  
2. 为什么Fetch远端分支，本地当前的分支历史没有变化  
Fetch把代码拉取到本地的远端分支，不会合并到当前分支，所以当前分支没有变化。  

### Git研发流程
依托代码管理平台 Gitlab，Github，Gerrit介绍如何进行代码的开发以及团队合作。  
不同的工作流：  
- 集中式工作流：Gerrit/SVN，依托主干分支进行开发，不存在其他分支，以fast-forward方式合并  
提供强制的代码评审机制，保证代码的质量；  
提供更丰富的权限功能，可以针对分支做细粒度的权限管控；  
保证master的历史整洁性；  
开发人员较多时，容易出现冲突；  
对于多分支的支持差；  
难以在项目之间形成代码复用。  
- 分支管理工作流：Github/Gitlab，有不同的开发分支，通过MR/PR合并主干分支，以自定义方式合并  
Git Flow：分支类型丰富，规范严格；  
GitHub Flow：只有主干分支和开发分支，规则简单；  
Gitlab Flow：在主干分支和开发分支之上构建环境分支，版本分支，满足不同发布或环境的需要。  

代码合并：  
- Fast-Forward：不会产生merge节点，合并后保持一个线性历史，如果target分支有更新，需要通过rebase操作更新source branch后才可以合入。  
  ![img](https://raw.githubusercontent.com/KevinChens/blog-image-bed/main/imgs/202206102316720.png)  
- Three-Way Merge：三方合并，产生一个新的merge节点。  
  ![img](https://raw.githubusercontent.com/KevinChens/blog-image-bed/main/imgs/202206102319893.png)  

如何选择合适的工作流：  
- 小型团队，推荐Github；  
保证少量多次，最好不要一次性提交上千行代码；  
提交Pull Request最少要保证CR后再合并；  
主干分支尽量保持整洁性，使用fast-forward合入方式，合入前进rebase。  
- 大型团队合作，根据自己的需要指定不同的工作流，不需要局限于某种流程。  

### 总结
本篇blog从Git是什么，Git的基本使用方式和Git的研发流程三个内容对Git进行了学习。Git作为一个分布式版本控制工具，使用是十分频繁的，对于Git的常用命令一定要多熟悉多使用。对于不同的研发流程，分支管理的工作流也是不一样的，代码提交的规范也是十分重要的，比如保护分支，code review等等。熟悉Git的使用，是能够提升开发效率，提升代码质量的。  

