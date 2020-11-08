---
title: Hexo博客搭建与部署
date: 2020-11-08 15:41:29
tags: [nodejs,hexo]
---

## Hexo博客搭建与部署

### 什么是 Hexo？

Hexo 是一个快速、简洁且高效的博客框架。Hexo 使用 [Markdown](http://daringfireball.net/projects/markdown/)（或其他渲染引擎）解析文章，在几秒内，即可利用靓丽的主题生成静态网页。

<!--more-->

### 安装

#### 安装前提

- [Node.js](http://nodejs.org/) 
- [Git](http://git-scm.com/)

#### 安装使用

```shell
$ npm install -g hexo-cli
```

安装好之后使用$ hexo -v可查看安装成功与否,具体查看[官方文档](https://hexo.io/zh-cn/)

```shell
$ hexo init <folder>
$ cd <folder>
$ npm install #d安装依赖
```

安装完成后博客文件放在source/_posts文件夹中

#### 修改配置

hexo项目结构安装好后，_config.yml可配置相关内容,可以选择自己喜欢的主题，可到官网或者github下载

### 部署

#### 部署到github

Hexo 提供了快速方便的一键部署功能

1.首先在github创建一个和你用户名相同的仓库，后面加上.github.io，也就是xxxx.github.io

2.安装 hexo-deployer-git

```shell
$ npm install hexo-deployer-git --save
```

3.修改配置

```shell
deploy:
  type: git
  repo: https://github.com/xxx/xxx.github.io.git #仓库地址
  branch: master #分支名
  message: [message]
```

4.部署

```shell
$ hexo clean
$ hexo generate
$ hexo deploy
```

#### 分支管理

hexo部署到github上面的只是生成的静态文件,当换一台电脑时就无法更新博客了,所以在仓库中创建一个hexo分支来保存hexo的环境文件

1.xxx.github.io.git创建一个分支hexo,将此分支设置为默认分支

2.创建一个空文件夹将xxx.github.io.git通过git克隆到本地,然后将.git文件移动到hexo init 的文件下,此时相当于将hexo 文件夹与github上的hexo分支关联起来

3.执行

```shell
$ git add -A
$ git commit -m "environment"
$ git push origin hexo
```

将远程hexo分支中的静态文件替换为当前目录下的环境文件(因为.gitignore文件中忽略了静态文件,所以上传上去的只是环境文件,不会包含静态文件)

4.每次更新文章过后使用hexo命名更新文章,使用git命令更新环境文件,在换电脑之后只需要从git克隆下就能继续更新博客了

#### 部署到gitee

部署到git之后会发现访问会很慢,有时候甚至直接无法访问,所以再到gitee再部署一个,因为再github已经部署过一次了,所以再gitee可以直接把github导过来

1.gitee再新建仓库的时候一个从github导入仓库的选项,可以直接从github将部署好的文件直接导过来

2.导入过来之后将仓库名改为与用户名相同

3.与github不同,gitee需要自己手动开启Pages服务,在仓库的服务选项选择Gitee Pages,选择部署分支mater,点击部署,成功后上面会显示已开启 Gitee Pages 服务，网站地址:https://xxx.git.io

![](https://gitee.com/LazyC-99/blog-images/raw/master/img/20201108153519.png)

4.更新文章之后,想在gitee也同步更新,可以点击仓库名旁边的箭头,可直接从github同步到gitee

### 图片上传

Markdown文档中是能够存放图片路径显示图片的,上传到网络上之后可以使用图床实现,因为github访问困难,所以图床文件放在gitee

1.下载PicGo,安装gitee插件

![](https://gitee.com/LazyC-99/blog-images/raw/master/img/20201108153403.png)



2.在gitee创建库并设置Gitee插件,然后就能直接上传图片到gitee里面,使用的时候直接用图片的网络地址