# 搭建博客 (Hexo + github + butterfly主题) 
### 1、下载安装Nodejs  
自行查找教程
> node -v
> npm -v
> 
查看是否安装成功
### 2、安装淘宝镜像cnpm管理器

> npm install -g cnpm --registry=http://registry.npm.taobao.org
> cnpm -v　

### 3、安装hexo框架

> cnpm install -g hexo-cli
> hexo -v
### 4、安装git (若有github账号可直接跳过本条)
git使用前配置：
下载完git后，需要告诉 git 你是谁，在向 git 仓库中提交时需要用到。

- 1、配置提交人姓名：git config --global user.name 提交人姓名
- 2、配置提交人姓名：git config --global user.email 提交人邮箱
- 3、查看git配置信息：git config --list

### 5、使用hexo搭建博客 (git bash/shell 打开)
#### 5.1 在指定位置创建文件夹，初始化hexo
```
mkdir blog  #创建blog目录
cd blog   #进入blog目录
hexo init  #生成博客 初始化博客
hexo s  /  hexo server  #启动本地博客服务

初始化之后，ls -al 查看可得文件
_config.landscape.yml db.json package.json public/ source/ yarn.lock
_config.yml node_modules/ package-lock.json scaffolds/ themes/

其中 _config.yml 是配置文件
themes/ 存放博客主题
```
#### 5.2 创建post
```
hexo new "我的第一篇博客文章"

cd source/_posts
ls -al
可查看所有的.md文件
```

#### 5.3 编辑博客文章内容
```
vim 我的第一篇博客文章.md
编写格式遵循markdown格式
```
#### 5.4 进入创建博客目录blog的主目录，执行清理工作，再执行生成工作
```复制代码
ls   (查看在_posts目录)
cd ../..  
pwd 
hexo clean 
hexo g  /  hexo generate   #生成发布用的静态页面,存在public文件中

生成工作完成后，会出现目录 ......../我的第一篇博客文章/index.html
```
#### 5.5 重新启动
```
hexo s  
```
至此，博客仅在本地localhost:4000启动,但是若需要可被远程访问，
接下来可部署到github来公开使用

### 6、github创建博客仓库
#### 6.1 进入github,创建Repository 空仓库
注意：Repository name 命名格式为 （owner）.github.io
必须是自己的github用户名.github.io,不能是用户昵称

比如 ： 
    Owner : Billy775326(billy)
    Repository name : Billy775326.github.io
#### 6.2 进入command/bash窗口，在创建的blog目录下安装一个git 部署插件

> cnpm install --save hexo-deployer-git

#### 6.3 设置_config.yml配置文件
```复制代码
vim _config.yml

_config.yml配置文件中：
    #Deployment
    ##Docs: https://hexo.io/docs/deployment.html
    deploy:
        type: git
        repo: https://github.com/Billy775326/Billy775326.github.io.git
        branch: main
```
配置好，保存退出

#### 6.4 将博客部署到远端

> hexo deploy / hexo d
输入账号密码

#### 6.5 刷新创库，访问 Billy775326.github.io


### 7、更换主题
打开butterfly网站  
https://butterfly.js.org/posts/21cfbf15/
#### 7.1 下载主题
```git
git clone -b master https://github.com/jerryc127/hexo-theme-butterfly.git themes/butterfly
```
#### 7.2 修改配置
修改站點配置文件_config.yml，把主題改為butterfly

_config.yml配置文件中：
    theme: butterfly
#### 7.3 进行清理，再重新生成，部署，再推送上传至github
```
hexo clean 
hexo g
hexo s  
hexo deploy
```
补充：
换主题时报 extends includes/layout.pug block content #recent-posts.recent-posts include includes/recent-posts.pug include includes/pagination.pug 
解决：
在hexo目录下打开git bash，输入命令：
npm install –save hexo-renderer-jade hexo-generator-feed hexo-generator-sitemap hexo-browsersync hexo-generator-archive
### 8、Hexo增加搜索功能
#### 8.1 安装搜索：在Hexo的根目录下
> npm install hexo-generator-searchdb --save
#### 8.2 全局配置文件_config.yml，新增
```
search:
    path: search.xml
    field: post
    format: html
    limit: 10000
```
#### 8.3 hexo主题配置文件（\themes\next_config.yml），修改local_search的enable为true
```复制代码
# Local search
# Dependencies: https://github.com/flashlab/hexo-generator-search
local_search:
    enable: true
    # if auto, trigger search by changing input
    # if manual, trigger search by pressing enter key or search button
    trigger: auto
    # show top n results per article, show all results by setting to -1
    top_n_per_article: 1
```