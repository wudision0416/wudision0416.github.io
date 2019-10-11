## 备份 hexo 源文件

### 需求 

对`Hexo`源文件进行备份，以便再更换工作环境后进行重新部署。

### 思路

创建 `hexo` 分支来存放`Hexo`生成的网站源文件，用`master`分支来存放生成的静态网页。

### 操作
1. `.gitignore` 文件为`hexo`默认配置：

```
.DS_Store
Thumbs.db
db.json
*.log
node_modules/
public/
.deploy*/
```

2. 切换到 `Hexo` 根目录，依次运行以下命令

``` bash
#git初始化
$ git init
#创建hexo分支，用来存放源码
$ git checkout -b hexo
#git 文件添加
$ git add .
#git 提交
$ git commit -m "init"
#配置 github 远程仓库
$ git remote add origin git@github.com:wudision0416/wudision0416.github.io.git
#配置 coding 远程仓库
$ git remote add coding git@git.dev.tencent.com:wudision0416/wudision0416.git
#push 到 github 的 hexo 分支
$ git push origin hexo
#push 到 coding 的 hexo 分支
$ git push coding hexo
```

3. 在 `Github` 与 `coding` 上对应的博客仓库中设置默认分支为`hexo`。这样有助于以后恢复博客环境。`master`分支为博客静态页面分支，在之后恢复博客的时候并不需要。

### 恢复

输入下列命令克隆博客必须文件(`hexo`分支)：

``` bash
#安装 hexo
$ npm install -g hexo-cli
#从 GitHub 的仓库中恢复
$ git clone git@github.com:wudision0416/wudision0416.github.io.git Blog
#从 coding 的仓库中恢复
$ git clone git@git.dev.tencent.com:wudision0416/wudision0416.git Blog
#进入博客根目录
$ cd ./Blog
#安装 npm 包
$ npm install
$ npm install hexo-deployer-git
```

不需要执行`hexo init`这条指令，因为不是从零搭建起新博客。

### 更新

先执行`hexo g -d`，把要发布的内容`push`到`github`或`coding`上面了，再去弄备份。