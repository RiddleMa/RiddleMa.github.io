---
title: hexo_use
date: 2025-10-17 15:13:23
tags:
---
## 简介
这里总结一下使用hexo的一些操作

## 多终端使用
参考：[备份和多终端更新hexo博客步骤](https://blog.csdn.net/shile/article/details/78714189)  
方便多个电脑之间的同步

## 常用命令
- 新增博客推送并同步仓库（向两个分支都推送，主分支master用来展示网页，hexo分支用来备份）
```commandline
//进入user.github.io文件夹,应是hexo分支
# git pull origin hexo //本地和远端的融合
# hexo new post "new post name"  //写新文章
# git add source
# git commit -m "xxx"
# git push origin hexo  //备份
# hexo d -g  //部署
```
## bug记录
- 图片加载不出来，修改图片时链接出现下面的记录
```commandline
update link as:-->/.io//111.jpg
```
原因是`hexo-asset-image`库有问题，卸载重装即可：
```
npm uninstall hexo-asset-image
npm install https://github.com/CodeFalling/hexo-asset-image
```
- 等等