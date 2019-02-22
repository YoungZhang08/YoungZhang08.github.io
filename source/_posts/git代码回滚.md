---
title: git代码回滚
date: 2018-05-16 21:09:31
tags: 
- 代码回滚
categories: git管理
about:
---

### 好了，记录一下今日的壮举。。。
坐标：百度大厦C座      时间：2018-05-16      
事件起因：上次的需求提交多提交了一个不必要的包导师缺陷检测有6个问题不能merge，所以今日在本地删除包之后提交代码，又因为对git操作的不熟悉，对untracked的deleted的文件执行了"git rm -r ."，然后就很尴尬了。。。我就看着我的VSCode的资源管理器那一栏噌噌噌地没了。。。但是还有部分没有删除。emmmmm，立即想到git代码回滚。嗯，解决的过程中，导师问我“你是想报复社会嘛”，emmmmm，其实，，，没想报复社会。。。也没想删库跑路。。。哭唧唧········回归正题——

<!--more-->
  1. 首先，你需要打印出你之前所有的commit-id，选择你要回滚到哪一个版本，命令：git log
  2. 接下来，执行git reset --soft commit-id，如果出现如下图所示的fatal，那就继续接着搞吧
![](https://note.youdao.com/yws/public/resource/3960d39c9b71e504ffd66863a26cdcc9/xmlnote/WEBRESOURCE2affa77461fd6eee11c2b4e2c0d25ced/3232)
3. 上一步的结果说明你还在合并,所以需要把当前目录所有修改的文件从HEAD中迁出并且把它回复称未修改时的样子
![](https://note.youdao.com/yws/public/resource/3960d39c9b71e504ffd66863a26cdcc9/xmlnote/WEBRESOURCEeddda71d5ba6de35beebbcc09374979f/3238)
4.在查看状态git status，可以看到提示你执行下面红框中的命令
![](https://note.youdao.com/yws/public/resource/3960d39c9b71e504ffd66863a26cdcc9/xmlnote/WEBRESOURCE2fe57c806552e3fc48bf856682d7c1f1/3245)
5. 执行git revert --abort之后查看状态，没有问题之后就可以回滚了
6. 执行git reset --soft commit-id，再去查看状态就可以看到自己刚刚都删了哪些内容
![](https://note.youdao.com/yws/public/resource/3960d39c9b71e504ffd66863a26cdcc9/xmlnote/WEBRESOURCE6e6cb09e30a1465150a6f11f5d707ddb/3251)
7. 根据提示再执行git reset HEAD .，这一步的意思是可以把暂存区的修改撤销掉（unstage），重新放回工作区
![](https://note.youdao.com/yws/public/resource/3960d39c9b71e504ffd66863a26cdcc9/xmlnote/WEBRESOURCEf83a997190a7c97779b7ec8bccde668c/3257)
8. 继续使用git status查看这些文件处于什么状态，可以看到是not staged，提示我们执行git checkout
恢复原态
![](https://note.youdao.com/yws/public/resource/3960d39c9b71e504ffd66863a26cdcc9/xmlnote/WEBRESOURCE1f4ae4ddf5fe7c60e0e8e6d9bdc30e86/3263)
9. 完了git add将工作区的提到暂存区并且提交commit
![](https://note.youdao.com/yws/public/resource/3960d39c9b71e504ffd66863a26cdcc9/xmlnote/WEBRESOURCEabc235dfe557b048f5f728fefe7fb0cb/3276)
10. 再执行git pull，git push到远程分支，就ok啦~
