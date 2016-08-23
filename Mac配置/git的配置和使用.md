#git配置
---
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
---

