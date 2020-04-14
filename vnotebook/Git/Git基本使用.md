# Git基本使用
参考菜鸟教程：[https://www.runoob.com/git/git-workflow.html](https://www.runoob.com/git/git-workflow.html)
## Git简介
Git 是一个开源的分布式版本控制系统，用于敏捷高效地处理任何或小或大的项目。
Git 是 Linus Torvalds 为了帮助管理 Linux 内核开发而开发的一个开放源码的版本控制软件。
Git 与常用的版本控制工具 CVS, Subversion 等不同，它采用了**分布式**版本库的方式，不必服务器端软件支持。
### 优势
* 本地拥有版本库，随时进行版本后退
* 非常简单的建立分支
* 速度更快，特别是熟悉git命令后
* 指定和若干不同的远端代码仓库进行交互
### 工作流程
#### 基本步骤
正确步骤：
1. git init //初始化仓库

2. git add .(文件name) //添加文件到本地仓库

3. git commit -m "first commit" //添加文件描述信息

4. git remote add origin + 远程仓库地址 //链接远程仓库，创建主分支

5. git pull origin master // 把本地仓库的变化连接到远程仓库主分支

6. git push -u origin master //把本地仓库的文件推送到远程仓库

一般工作流程如下：
+ 克隆 Git 资源作为工作目录。
+ 在克隆的资源上添加或修改文件。
+ 如果其他人修改了，你可以更新资源。
+ 在提交前查看修改。
+ 提交修改。
在修改完成后，如果发现错误，可以撤回提交并再次修改并提交。
![](_v_images/20200411211929547_9333.png =585x)
### 基本概念
![](_v_images/20200411173159860_21943.png =541x)
## Git基本命令
### 本地库初始化
#### git init
该命令执行完后会在当前目录生成一个 .git 目录。
.git 目录中存放的是本地库相关的子目录和文件，不要删除，也不要胡乱修改。

```
//使用当前目录作为Git仓库，我们只需使它初始化。
git init

//使用我们指定目录作为Git仓库。
git init newrepo
```

#### git clone
```
//使用 git clone 从现有 Git 仓库中拷贝项目
git <repo>
//如果我们需要克隆到指定的目录，可以使用以下命令格式：
git clone <repo> <directory>
```
