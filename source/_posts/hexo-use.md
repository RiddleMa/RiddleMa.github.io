---
title: hexo使用记录
date: 2025-10-17 15:13:23
tags:
---
## 简介
这里总结一下使用hexo的一些操作
- hexo是快速构建博客的工具
- 目前使用git同时管理项目代码和部署网页
  - 项目代码就是整个项目的同步，对应hexo分支
  - 部署网页是hexo展示用的web页面，对应master分支

## 多终端使用
参考：[备份和多终端更新hexo博客步骤](https://blog.csdn.net/shile/article/details/78714189)  
方便多个电脑之间的同步

## 常用命令
- 新增博客推送并同步仓库（向两个分支都推送，主分支master用来展示网页，hexo分支用来备份）
- 首先是同步仓库，以及使用hexo推送网页
```commandline
// 进入user.github.io文件夹，对应本地hexo/blog目录
// 切换到hexo分支
git branch
// 切换到hexo分支
git switch hexo
git checkout hexo   // 或者
git pull origin hexo //本地和远端的融合
hexo new post "new post name"  //写新文章
hexo d -g  //部署，这个是推送到master分支
```
- 下面是推送备份到服务器
```
git add source
git commit -m "xxx"
git push origin hexo  //备份到hexo分支
```
## bug记录
- 上传后博客更新需要等待几分钟，github博客重加载需要缓冲时间
- 图片加载不出来，修改图片时链接出现下面的记录
```commandline
update link as:-->/.io//111.jpg
```
原因是`hexo-asset-image`库有问题，卸载重装即可：
```
npm uninstall hexo-asset-image
npm install https://github.com/CodeFalling/hexo-asset-image
```
- 加载图片
```
{% asset_img example.jpg This is an example image %}
{% asset_img "spaced asset.jpg" "spaced title" %}
```
- 命令`hexo d -g`报错
```
nothing to commit, working tree clean
错误：fatal: unable to access 'https://github.com/***/': Recv failure: Connection was reset
FATAL Something's wrong. Maybe you can find the solution here: https://hexo.io/docs/troubleshooting.html
Error: Spawn failed
at ChildProcess.<anonymous> (D:\hexo_pro\blog\node_modules\hexo-deployer-git\node_modules\hexo-util\lib\spawn.js:51:21)
at ChildProcess.emit (node:events:513:28)
at ChildProcess.cp.emit (D:\hexo_pro\blog\node_modules\cross-spawn\lib\enoent.js:34:29)
at Process.ChildProcess._handle.onexit (node:internal/child_process:293:12)
```
hexo是将`public`文件夹内容同步到服务器，需要先md文档转为html等格式，`public`未更新导致的。  
重新构建`public`即可。  
或者是网不好，重试。
```
hexo clean && hexo g && hexo s
```
