---
title: 利用 Hexo 和 Github 搭建静态博客
tags: Hexo
permalink: build-static-blogs-with-hexo-and-github
date: 2017-06-25 21:08:20
---

# 为什么选择Github Pages + Hexo ?
1. 免费，Github Pages 提供500M免费空间，并且不限流量
1. Hexo 有本地服务器，可以在本机预览，管理也很方便
1. Hexo 主题丰富，好看   

<!--more-->
# 详细的安装与配置步骤
## 创建你的Github Pages 
1. 登陆 [github](https://github.com/)，创建新的 repository，注意 Repository name 为 YourName.github.io ( YourName 与你的注册用户名一致,这个就是你 Github 博客的域名)
2. 进入 repository，点击右上方的 Settings，拉到 Github Pages 一栏，点击 Launch automatic page generator 即可。可以根据下面的提示选择 github 提供的模板，然后登陆 YourName.github.io，测试你的站点是否建立成功。   

## 安装、配置 Hexo
Hexo的运行依赖于下面两个组件：
+ [Node.js](http://nodejs.cn/)
+ [Git](https://git-scm.com/download/)

如果已经安装上面的组件，可以直接进行下面的操作。否则需要安装相应的软件，并配置环境变量。注意，安装好 Git 后需要配置你的本地账户，在此不做赘述。
### 安装 Hexo
选择一个文件，右键打开Git Bash终端，执行如下命令
```bash
$ npm install -g hexo-cli
```
（如果长时间没有反应，按Ctrl + c中断，先使用下面的命令将npm镜像设为国内镜像，再执行上面的安装命令）
```bash
$ npm config set registry https://registry.npm.taobao.org
```
### 初始化 Hexo
执行如下命令进行初始化，< folder >为你要创建的博客所在的文件夹。
```bash
$ hexo init <folder>
$ cd <folder>
$ npm install   /*安装依赖文件*/
```
现在我们首先执行如下操作，来测试是否安装成功
```bash
$ hexo generate    /*生成文件*/
$ hexo server      /*在本地端口启动站点*/
```
在浏览器中输入 http://localhost:4000/，访问站点，如果出现类似如下网页，则说明网站搭建成功。
![](http://osac5rsex.bkt.clouddn.com/blog_screenshot.png?imageslim)

### 配置 Hexo
[Hexo](https://hexo.io/) 的配置文件位于根目录下，文件名为  _config.yml，主要包含 Hexo 本身的配置，称为全局配置文件。另外在每一个主题的文件夹（/themes/theme-name/）下面，也有一个 _config.yml文件，是主题的配置文件。Hexo 的官网上有详细的说明文档，__注意：配置文件中每个字段后面的冒号是英文格式的，且在其后面必须要保留一个空格在写值。__ 
这里借用一个网友的[翻译](http://fanzhenyu.me/2016/12/03/Github-Pages-Hexo%E6%90%AD%E5%BB%BA%E5%8D%9A%E5%AE%A2%EF%BC%88%E4%BA%8C%EF%BC%89/)以供参考
```
# Hexo Configuration
## Docs: https://hexo.io/docs/configuration.html
## Source: https://github.com/hexojs/hexo/
# Site 站点信息配置，根据自己的需要进行修改
title: Line's Blog    #站点名，会在浏览器页面标签左上角显示
subtitle: Love Coding,Enjoy Life  #副标题
description: fzy-line  #对站点的描述，给搜索引擎看的，可以自定义
author: Line  #网站作者
language: zh-Hans  #网站语言
timezone: Asia/Shanghai  #时区
avatar: /images/logo.jpg  #网站logo，会在浏览器页面标签左上角显示
# URL 博客地址,与申请的GitHub一致
## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
url: https://fzy-line.github.io/
root: /
#博客链接格式
permalink: :year/:month/:day/:title/ 
permalink_defaults:
# Directory  #目录设置，一般不修改
source_dir: source  #资源文件夹，放在里面的文件会上传到github中
public_dir: public  #公共文件夹，存放生成的静态文件
tag_dir: tags  #标签文件夹，默认是tags。实际存放在source/tags中。
archive_dir: archives  #档案文件夹，默认是archives。
category_dir: categories  #分类文件夹，默认是categories。实际存放在source/categories中。
code_dir: downloads/code  #代码文件夹，默认是downloads/code
i18n_dir: :lang  #国际化文件夹，默认跟language相同
skip_render:  #跳过指定文件的渲染，您可使用 glob 来配置路径。
# Writing  这是文章布局、写作格式的定义，一般不修改
new_post_name: :title.md # File name of new posts
default_layout: post
titlecase: false # Transform title into titlecase
external_link: true # Open external links in new tab
filename_case: 0
render_drafts: false
post_asset_folder: false
relative_link: false
future: true
highlight:
  enable: true
  line_number: true
  auto_detect: false
  tab_replace:
# Category & Tag  #分类和标签，一般不修改
default_category: uncategorized
category_map:
tag_map:
# Date / Time format  #日期、时间格式，一般不修改
## Hexo uses Moment.js to parse and display date
## You can customize the date format as defined in
## http://momentjs.com/docs/#/displaying/format/
date_format: YYYY-MM-DD 
time_format: HH:mm:ss
# Pagination  #可根据自己需要修改
## Set per_page to 0 to disable pagination
per_page: 6  #分页，每页文章数量
pagination_dir: page
# Extensions  #扩展
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: next  #博客主题
  
# Deployment 配置站点部署到Github
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repository: git@github.com:你的Github用户名.github.io.git
  branch: master
```

### 使用主题
Hexo 使用主题的方式非常简单，将主题文件拷贝至根目录的 themes 目录中，然后修改配置文件的 theme 字段即可。在官网有很多主题可供下载 https://hexo.io/themes/ 。我推荐 [next](http://theme-next.iissnan.com/) 主题，可以直接到 github 下载，然后解压到 themes 目录下。建议使用 clone 的方式将其复制到本地，这样更新直接执行 pull 命令即可。   
现在修改全局配置文件中的theme字段:   
```
    theme: next  #博客主题
```
接下来我们去修改主题的配置文件（/next/_config.yml），如下   
（1） 一些基本信息
```
# 网站图标，将其放在hexo站点/source/目录下
favicon: /logo.jpg
# 关键词
keywords: "C++"
# 网站建立时间，显示在页面底部
since: 2017
# 网站版权声明，显示在页面底部
copyright: true
# 头像
avatar: /*可以用七牛云外链你的帅气头像*/
```
（2） 三种界面模式
```
# Schemes
#scheme: Muse   /*默认 Scheme，这是 NexT 最初的版本，黑白主调，大量留白*/
scheme: Mist    /*Muse 的紧凑版本，整洁有序的单栏外观*/
#scheme: Pisces   /*双栏 Scheme*/
```

（3） 设置菜单
```
menu:
  home: /
  archives: /archives
  #categories: /categories
  tags: /tags
  about: /about
  #sitemap: /sitemap.xml
  #commonweal: /404.html
```   
可以调整其上下的顺序，来设置哪个菜单在左，哪个菜单在右。   
每次修改后，可以重新执行下面命令，然后访问 http://localhost:4000/ 查看效果
```bash
$ hexo generate #生成静态网页
$ hexo server #开启本地预览
```   
尝试点击博客中各个菜单，你会发现除了首页和归档外，其他都会出错。这是因为 next 主题只默认为 home 和 archives 两个字段产生页面，其他需要手动添加。不过添加的命令也很简单：   
新建页面的命令是：
```bash
$ hexo new page "pageName"
```
比如新建标签、关于页面：
```
$ hexo new page 'tags'
$ hexo new page 'about'
```
根目录下的 /source/ 目录下多了两个文件夹：tags，about，每个文件夹里面都会生成一个 index.md 文件，内容如下：
```
title: about
date: 2017-06-25 21:01:26
```
默认都只会生成 title 和 date 字段，要为其添加上 type 字段，并赋值。比如关于页设置如下：
```
title: about
date: 2017-06-25 21:01:26
type: about
```
现在访问关于和标签页面就没有问题了。

### 添加博客

新建博客的命令是
```
$ hexo new "post_name" #新建文章
```
比如新建博文《Hello Hexo》
```
$ hexo new "Hello Hexo"
```
根目录下的 /source/_posts/ 目录下会生成名为：Hello Hexo.md 的文件，这是[Markdown](http://www.appinn.com/markdown/)格式的文件。   
Hexo 默认新建的文章抬头已有 title、date、tags 等属性，可以自己添加别的标签，meta 标签则是为了便于搜索引擎的收录。如下
```
title: Hello Hexo
date: 2017-06-25 23:44:20
tags: #文章标签 非必需
```
执行如下命令，在本地查看博客
```
$ hexo clean  #清除缓存 网页正常情况下可以忽略此条命令
$ hexo generate  #生成静态页面至public目录
$ hexo server  #开启预览访问端口（默认端口4000，'ctrl + c'关闭server）
```
确定无误后，现在我们可以将网站部署到Github上。

### 将 Hexo 部署到 Github

首先修改全局配置文件_config.yml，编辑最下面的deploy属性，如下所示：
```
type: git
repository: https://github.com/你的Github用户名/你的Github用户名.github.io.git
branch: master
```
安装 hexo-deployer-git 插件
```bash
$ npm install hexo-deployer-git --save
```
依次执行以下三条命令，即可将网站部署到 Github 上
```
$ hexo clean  #清除缓存 网页正常情况下可以忽略此条命令
$ hexo generator  #生成静态页面至public目录
$ hexo deploy  #将.deploy目录部署到GitHub
```
执行 hexo deploy 命令之后，如果最后一行打印出如下信息则表示部署成功
```
INFO  Deploy done: git
```
然后你再去访问前面提到的 YourName.github.io，即可看到你本地的Hexo项目已经被部署到 Github 上去了。


# 优化
## 图片
在博文中，我们会插入一些图片。Hexo 提供了一种更方便管理 Asset 的设定：post_asset_folder。当设置 post_asset_folder 字段为 true 后，在建立文件时，Hexo 会自动建立一个与文章同名的文件夹，可以把与该文章相关的所有资源都放到那个文件夹，如此一来，便可以方便地使用资源。
但这样做有两个坏处：   
* 大量占用 Github Pages 的空间
* Github在大陆访问较慢，图片保存在上面，会导致网页加载变慢      

因此，无论从成本还是访问速度考虑，建议考虑国内的图床服务。比较国内免费的服务商，[七牛云](https://portal.qiniu.com)的服务应该是最好的，使用也很简单。另外，七牛云甚至还提供了图片瘦身服务，在外链的后面加上 `?imageslim `即可使用。据说瘦身后图片分辨率不变，体积减小，节省流量，载入速度变快。

## 域名绑定
__向 Github Pages 中添加一个 CNAME 文件__   
Github Pages 与自己购买的域名绑定时，需要向仓库中添加一个 __CNAME（一定要大写）__文件，并且其中只能包含一个顶级域名，像这样：
```
ezreal.tech
```
但Hexo在每次执行
```bash
hexo g
hexo d
```
后，会把整个仓库原来的文件都覆盖了，包括 CNAME 文件。如果每次都手动重新添加就太繁琐了，解决方式是直接把 CNAME 文件添加到本地根目录的 source 文件夹里，这样每次推的时候就不用担心仓库里的 CNAME 文件被覆盖掉了。   
__向你的 DNS 配置中添加 3 条记录__   
![我的DNS配置](http://osac5rsex.bkt.clouddn.com/DNS.png?imageslim)   
将其中的huangg 换成上面提到的 YourName 即可。保存后等待配置生效。

## 网页加载过慢
在没有使用代理的时候，访问使用 next 主题的博客会非常慢，即使在本地运行也是如此。使用 chrome 打开控制台后发现
![](http://osac5rsex.bkt.clouddn.com/google_font_fail.png?imageslim)
是加载google字体库失败导致的。
下面提供两种解决方法:   
* 打开 next 的_config.yml，设置不从外部加载字体（默认会加载本地字体） ： 
```
  global:
    # external: true will load this font family from host.
    external: false /*默认是true*/
    family: Lato
```
*  但本地字体确实没有外部字体好看啊，所以我采用了方法2。打开 \themes\next\layout_partials\head\external-fonts.swig文件，替换其中的 fonts.googleapis.com 为 fonts.useso.com。然而我发现后者也已经挂了，原因是360不再为其提供CDN加速服务。但我惊喜的发现，我科的 linux 镜像源竟然提供此服务。将  fonts.googleapis.com 换为  fonts.lug.ustc.edu.cn 即可。换完之后你会发现博客加载速度快得飞起~~~~


## 语法高亮
Hexo 中插入代码有两种写法。一种是 Code Block，格式如下:   

    {% codeblock [title] [lang:language] [url] [link text] %}
    code snippet
{% endcodeblock %}   

源文件中 Code Block 被 Hexo 的 *code* 或 *codeblock*  Tag 插件（hexo/lib/plugins/tag/code.js）解析为高亮后的 html。   
另一种是 Backtick Code Block，格式如下   

    ``` [language] [title] [url] [link text] code snippet  ```   

 \`\`\` 与 \`\`\` 之间的被 Hexo *backtick_code_block* Filter 插件（hexo/lib/plugins/filter/before_post_render/backtick_code_block.js）解析为高亮后的 html。   
 无论是哪种解析方式，其实都是调用 hexo-util 模块的 libs/highlight.js 来解析（最终使用 highlight.js 和 html-entities 这两个依赖库进行解析）。当然，也可以将自带高亮关闭掉，然后使用\`\`\`来高亮，此时解析器将是 hexo-render-marked。   
示例， Code Block 代码如下：

    {% codeblock  lang:cpp %}
    int main()
    {
        int i=0;
        return 0;
    }
{% endcodeblock %}

效果：  
{% codeblock  lang:cpp %}
int main()
{
    int i=0;
    return 0;
}
{% endcodeblock %}

## 产生文章概要
在首页不能显示每一篇文章的所有内容，又希望展示一部分，可以在文章中适当的位置添加如下标签：   
    `<!--more-->`
next 主题会自动识别这个标签，生成对应的阅读全文按钮。