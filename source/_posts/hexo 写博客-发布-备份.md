layout: ==
title: hexo 写博客-发布-备份
date: 
tags:hexo,博客,备份
---
# hexo 写博客-发布-备份

##1.写博客
在你的博客根目录下执行：

    hexo new "Hello Hexo" 或 hexo n "Hello Hexo" 生成一个markdown文件骨架用于填写内容
    
在博客的_posts文件夹下（blogs/source/_posts）会生成Hello Hexo.md的文件，打开该文件并写入你的文档。markdown文件头部, 支持多标签
##2.发布博客
```
hexo generate 或 hexo g 写完博客后, 生成静态内容
hexo server 或 hexo s 启动本地服务器可以先看一下效果
hexo deploy 或 hexo d 发布到外网
```
##3.备份博客
网络上面大部分的方法都是使用网盘进行备份，这种方法实在是太不“程序员”了！我使用git分支来备份博客。使用GithubPage生成的博客都是存放在github的仓库（XXXX.github.io）的master分支中，我在这个仓库中新建一个分支blogs来保存博客的源文件。这样就可以用一个仓库来存放Git Page，以及备份博客。

> *  在博客的根目录下执行git init,生成一个git仓库，因为根目录下已经存在一个.gitignore文件，所以不需要自己去管理ignore文件。
> *  在博客根目录下执行git checkout -b blogs,生成一个新的分支blogs。并执行git add .,git commit将源文件用git管理。
> *  添加远程仓库：git remote add origin git@xxxx.git。
> *  将本地blogs分支推送至远程blogs分支。git push origin blogs。

以上部分执行结束之后，本地的博客源文件就存放在blogs分支中，以后对于博客的修改，hexo命令都在hexo分支中执行即可，记得更新博客之后要push。。。

update 备份
git pull https://github.com/michaelcjl/blogs
git push origin blogs


那么在另一台电脑上面怎样更新博客呢？
克隆仓库。git clone git@xxxx.git。

仓库克隆下来之后，本地只有一个master分支，内容是hexo生成的静态博客文件。因为博客源文件保存在远程hexo分支中，所以我们需要在本地新建一个hexo分支来跟踪远程hexo分支：
`git checkout -b hexo origin/hexo`
 切换至hexo分支之后，依次执行一下指令:
 `npm install hexo,npm install,npm install hexo-deployer-git`
之后就可以随心所遇的在另外一台电脑上更新博客了=。=
以上步骤结束之后，执行hexo g -d可能会报“TypeError: Cannot set property ‘lastIndex’ of undefined”，这时只需要执行hexo clean即可。

