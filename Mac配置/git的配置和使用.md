#git配置
####全局配置用户名和邮箱
```
git config --global user.name username
git config --global user.email emailname
```
####全局配置中文编码支持
```
git config --global gui.encoding utf-8
git config --global i18n.commitencoding utf-8
git config --global i18n.logoutputencoding gbk
##解决git status乱码
git config --global core.quotepath false
```
####全局配置忽略文件
```
新建一个忽略文件，譬如vim .gitignore_global,输入下面的忽略
.gradle
.idea
.DS_Store
配置全局忽略文件，假设忽略文件路径名/Users/yelong/.gitignore_global
git config --global core.excludesfile /Users/yelong/.gitignore_global
```
####全局配置彩色输出
```
git config  --global color.ui true
```

#git的使用
####项目忽略文件如何编写
```
语法
1. 以斜杠"/"开头表示目录
2. 以星号"*"通配多个字符
3. 以问号"?"通配单个字符
4. 以方括号"[]"包含单个字符的匹配列表
5. 以叹号"!"表示不忽略(跟踪)匹配到的文件或目录

示例说明
#忽略项目根目录下的隐藏文件或文件夹
.gradle
.idea
.DS_Store

#忽略所有以.iml结尾的文件
*.iml

#忽略目录build下的所有内容，注意，不管是根目录下的/build/目录还是某个子目录/app/build/目录都会被忽略
build/*

#忽略根目录下得文件夹下的全部内容
/build/*
/captures/*

#忽略根目录下的文件
/local.properties
/settings.gradle

注意点：
发现添加的忽略规则没有生效，原因是.gitignore只能忽略那些原来没有被track的文件，如果某些文件已经被纳入了版本管理中，则修改.gitignore是无效的。那么解决方法就是先把本地缓存删除（改变成未track状态），然后再提交
git rm -r --cached .
git add .
git commit -m 'update .gitignore'

```
####git工作流程图
![git工作流程图](images/git-flow-chart.png)

```
Workspace：工作区
Index / Stage：暂存区
Repository：仓库区（或本地仓库）
Remote：远程仓库
```
####新建代码库
```
# 在当前目录新建一个Git代码库
$ git init

# 新建一个目录，将其初始化为Git代码库
$ git init [project-name]

# 下载一个项目和它的整个代码历史
$ git clone [url]
```

####新增文件
```
# 添加指定文件到暂存区
$ git add [file1] [file2] ...

# 添加指定目录到暂存区，包括子目录
$ git add [dir]

# 添加当前目录的所有文件到暂存区
$ git add .
```

####删除文件
```
# 删除工作区文件，并且将这次删除放入暂存区
$ git rm [file1] [file2] ...

# 删除工作区文件夹，并且将这次删除放入暂存区
$ git rm -rf [dir]

# 停止追踪指定文件，但该文件会保留在工作区
$ git rm --cached [file]

# 改名文件，并且将这个改名放入暂存区
$ git mv [file-original] [file-renamed]
```
####撤销文件
```
# 恢复暂存区的指定文件到工作区
$ git checkout [file]

# 恢复某个commit的指定文件到暂存区和工作区
$ git checkout [commit] [file]

# 恢复暂存区的所有文件到工作区
$ git checkout .
```

####代码提交
```
# 提交暂存区到仓库区
$ git commit -m [message]

# 提交暂存区的指定文件到仓库区
$ git commit [file1] [file2] ... -m [message]
```
####分支
```
# 列出所有本地分支
$ git branch

# 列出所有远程分支
$ git branch -r

# 列出所有本地分支和远程分支
$ git branch -a

# 新建一个分支，但依然停留在当前分支
$ git branch [branch-name]

# 切换到指定分支，并更新工作区
$ git checkout [branch-name]

# 切换并新建一个分支
$ git checkout -b [branch-name] [remote-branch]
```
####查看信息
```
# 显示有变更的文件
$ git status

# 显示当前分支的版本历史
$ git log

# 显示暂存区和工作区的差异
$ git diff
```

####远程同步
```
# 取回远程仓库的变化，并与本地分支合并
$ git pull [remote] [branch]

# 上传本地指定分支到远程仓库
$ git push [remote] [branch]

# 强行推送当前分支到远程仓库，即使有冲突
$ git push [remote] --force
```

> 参考资料：http://www.ruanyifeng.com/blog/2015/12/git-cheat-sheet.html
