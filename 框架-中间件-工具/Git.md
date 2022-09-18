### Git基本理论

![img](https://pic.pimg.tw/silverwind1982/1483082151-1840902919_n.png)



- Workspace：**工作区，**就是平时存放项目代码的地方
- Index / Stage：**暂存区，**用于临时存放你的改动，事实上它只是一个文件，保存即将提交到文件列表信息
- Repository：**本地仓库，**就是安全存放数据的位置，这里面有你提交的**所有版本的数据。**其中HEAD指向最新放入仓库的版本
- Remote：**远程仓库，**托管代码的服务器，可以简单的认为是你项目组中的一台电脑用于远程数据交换



### 常用指令

日常使用只要记住下图6个命令：

![图片](https://mmbiz.qpic.cn/mmbiz_png/uJDAUKrGC7Ksu8UlITwMlbX3kMGtZ9p0AII6YVooUzibpibzJnoOHHXUsL3f9DqA4horUibfcpEZ88Oyf2gQQNR6w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**git clone：**拷贝一个 Git 仓库到本地，让自己能够查看该项目，或者进行修改。

**git pull：** 从远程获取代码并合并本地的版本。git pull = git clone/fetch + git merge FETCH_HEAD 。



### 创建本地仓库Repository

方法有两种：一种是创建全新的仓库，另一种是克隆远程仓库。

#### 方式一：创建全新的仓库

1、需要用GIT管理的项目的根目录执行：

```bash
# 在当前目录新建一个Git代码库
$ git init
```

2、执行后可以看到，仅仅在项目目录多出了一个.git目录，关于版本等的所有信息都在这个目录里面。

#### 方式二：克隆远程仓库

将远程服务器上的仓库完全镜像一份至本地

```bash
# 克隆一个项目和它的整个代码历史(版本信息)
$ git clone [url]  # https://gitee.com/kuangstudy/openclass.git
```



### 将本地仓库的内容上传至Git仓库

```bash
# 提交代码：
git add .

# 查看状态
git status

# -m必填，必须有双引号
git commit -m "xxx"

# origin代表当前用户，master代表要push的分支
git push origin master
```



### git分支常用命令

```bash
# 列出所有本地分支
git branch
# 列出所有远程分支
git branch -r
# 新建一个本地分支，但依然停留在当前分支
git branch [branch-name]
# 新建一个本地分支，并切换到该分支
git checkout -b [branch]
# 合并指定分支到当前分支
git merge [branch]
# 删除分支
git branch -d [branch-name]
# 删除远程分支
git push origin --delete [branch-name]$ git branch -dr [remote/branch]
```

