---
title: 搭建hexo博客步骤以及一些问题
categories: 文章
---
介绍了自己搭起的博客，并且其中遇到的一些问题
<!--more-->
<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=28754391&auto=1&height=32"></iframe>
# 个人博客文档存档
---
安装过程：

1. 安装node.js
2. 安装Hexo:cnpm install -g hexo-cli
3. 打开git命令行工具：hexo init blog
4. hexo g == hexo generate #生成
5. hexo s == hexo server #启动服务预览
6. 推送网站前需要设置参数：
deploy: 
```
type: git
repo: 这里填入你之前在GitHub上创建仓库的完整路径，记得加上 .git
branch: master
```
7. cnpm install hexo-deployer-git --save
8. 输入下面命令就可以推送了：

```
hexo clean 
hexo g 
hexo d
```
9. 然后就可以访问了
10. 参考网址[：https://zhuanlan.zhihu.com/p/26625249]



## 重新从gitHub克隆后工程功能步骤：
1. 在工程里面执行 cnpm install 安装对应包
2. 推送不上去参考最下面解决方法
3. 使用 hexo d 需要在git bash 中使用，在cmd使用是不行的，不知道什么原因




### 发生hexo d 推送不上去处理现象：
1、新建 SSH key，在git shell:
```
$ ssh-keygen -t rsa -C "邮箱名"
```
然后会出现：
```
Generating public/private rsa key pair.
Enter file in which to save the key (/c/Users/dell/.ssh/id_rsa):
```
直接回车就可以。
然后会出现：
```
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
```
要求你输入密码，这个密码会在你提交项目时使用，如果为空的话提交项目时则不用输入。这个设置是防止别人往你的项目里提交内容。
然后会出现：
```
Your identification has been saved in /c/Users/dell/.ssh/id_rsa.
Your public key has been saved in /c/Users/dell/.ssh/id_rsa.pub.
The key fingerprint is:
65:69:······02:4b emailname@email.com
The key's randomart image is:
+--[ RSA 2048]----+
|                 |
|       .   o .   |
|    . o o = o    |
|   . o * = o     |
|  E  o + o .     |
| . o.   . .      |
|     ..          |
+-----------------+
```

2、接下来在github上添加SSH key：
- 打开本地文件：id_rsa.pub（文件路径可以在上一步SSH生成成功后看到路径，比如我的是c/Users/dell/.ssh/id_rsa.pub），可以将这个文件在编辑器中打开，然后全选复制。

- 登陆github，点击头像位置处 Settings ——> SSH and GPG keys ——> New SSH key，点击新建SSH key。

- 将 ① 中复制的内容粘贴在key文本框里，title可以不用填（或者自己起一个名字也可以）。
- 

3、测试设置是否成功：
```
$ ssh -T git@github.com
```
最后就能部署了

