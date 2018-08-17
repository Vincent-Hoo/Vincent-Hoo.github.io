---
title: 用hexo搭建个人网站
date: 2018-08-14 20:39:17
tags:
---
{% asset_img header.jpg 800 400 %}
该文章简述如何使用hexo搭建个人博客

<!-- more -->
# hexo介绍
hexo是基于Git和Node.js的静态网站搭建框架，通过hexo命令可以自动生成静态网站的html和css，然后可以将其部署于服务器，生成个人网站。
{% asset_img hexo_homepage.png 800 400 %}
<br>

# hexo本地安装与运行
1. 安装Git和Node.js:   由于hexo是基于Git和Node.js的，所以在安装hexo之前，需要先安装[Git](https://git-scm.com/)和[Node.js](https://nodejs.org/en/).

2. 安装hexo：输入下面这条语句，就会自动地安装hexo。
   ```javascript
   $ npm install hexo-cli -g
   ```

3. 初始化blog： 输入下面这些语句，hexo会自动从网上拷贝所需要的dependency和package，生成一个project，存储在`Vincent-Hoo.github.io`这个文件夹里面，`Vincent-Hoo.github.io`是博客的名字，可以取其它的名字，但是由于我之后要部署到Github上，所以就取这个名字。
    ```
    $ hexo init Vincent-Hoo.github.io
    $ cd Vincent-Hoo.github.io
    $ npm install
    ```

4. 在本地运行blog：输入以下的命令，hexo会自动生成一个静态网站在http://localhost:4000。
    ```
    $ hexo server
    ```
    当目前为止，hexo已经为我们搭建好了一个个人的博客，里面有最基本的功能，如博客发表，搜索，导航，分享等。`Hello World`是hexo初始化时候自动生成的第一篇博客。
    {% asset_img hexo网站初始化的样子.png %}
    <br>

# hexo框架的基本结构
在你的hexo project初始化完之后，project文件夹的目录结构如下。

{% codeblock %}
|----- node_modules
|----- scaffolds
|----- source
|      |----- _posts
|      |----- _drafts
|----- themes
|----- _config.yml
|----- db.json
|----- package.json
{% endcodeblock %}



## scaffolds

首先，hexo支持三种layout

- post：对应source文件夹的`_posts`，我们所有的文章都放在这个文件夹里面。
- draft：顾名思义，草稿，对应`source`文件夹的`_drafts`，draft一般而言都不会显示在博客上，如果要让draft显示，可以修改`_config.yml`文件的`render_draft`，将其设置为true（默认为false）。
- page：一个博客不单止有文章，还会有网页，如个人介绍，个人简历。page也是保存在`source`文件夹里面。

而scaffolds里面有三个文件，`post.md`, `draft.md`, `page.md`。当我们新建一个layout的时候，hexo就会根据我们新建的layout类型，选择相应的markdown文件进行初始化，所以这三个文件相当于是模板，下面以`post.md`作为例子，顺便讲一下什么是front-matter。

```
---
title: {{ title }}
date: {{ date }}
tags:
---
```

模板最前面的key-value pair就是front-matter，下面列举了一些常见的front-matter

| front-matter |        description        |       default        |
| :----------: | :-----------------------: | :------------------: |
|    layout    | post, draft, page三者之一 |                      |
|     tags     |           标签            |                      |
|  categories  |           类别            |                      |
|    title     |         文章标题          | 默认是创建时候的标题 |
|     date     |         创建时间          | 默认是创建时候的时间 |

tag和categories可以很好地将博客的文章进行分类，设置的方式如下，注意categories具有层级关系，如Cat3和Cat3.1
```
title: Hello World
tags:
- Tag1
- Tag2
categories:
- Cat1
- Cat2
- [Cat3, Cat3.1]
```

## source
source文件夹是整个博客里面最重要的文件夹，一个博客几乎所有的内容都存储在里面，之前提过，有两个子文件夹`_posts`和`_drafts`，当然还会有一些page和用户自定义的数据。
`_posts`文件夹存储所有博客中显示的文章，`_drafts`文件夹存储所有的草稿，如果要将草稿转成文章，可以使用下面的命令
``` bash
$ hexo publish <draft_name>
```
创建文章或者草稿非常简单，只需要一条命令
``` bash
$ hexo new <layout, default: post> <article_name>
eg:
$ hexo new draft Hello-World
$ hexo new (post) Hello-World
$ hexo new page About-me
```

## themes
hexo博客在初始化的时候默认是用landscape主题，当然我们可以自己去下载另外的一些主题(https://hexo.io/themes/ )。切换主题只需要将`_config.yml`文件里面的`theme`改成对应的主题名字即可。
当然，我们也可以DIY自己的主题，主题无非就是一些模板的CSS和HTML文件，还有一些脚本文件，定义了不同的地方需要如何去解析。而每一个的主题都会有自己的`_config.yml`，注意和全局的`_config.yml`区分，前者是定义主题里面的configuration，后者是定义整个博客。

## config.yml
config文件定义整个博客的设置，如网站的title，author，还有目录设置，文章写作的设置，这里特别说一个设置`post_asset_folder`，asset_folder是指一篇文章的数据，如图片，如果设置为true，在创建一篇post的时候，不仅仅创建一个文章的markdown，还创建一个对应的文件夹。还有一个设置是`render_draft`，默认是设置为false的，因为草稿不会显示在网页上，但如果设置为true，那么草稿也会显示出来。
<br>

# Tag Plugin
hexo博客的文章都是以markdown的形式来保存，markdown的强大使得文章写作变得简单，但hexo的tag plugin使得写文章变得更加的简单，它通过一些特殊的语句来添加特定的内容到文章，如图像、视频等。
1. 引用别人的话block quote
```
{% blockquote [author[, source]] [link] [source_link_title] %}
content
{% endblockquote %}
```
2. 代码块code block
```
{% codeblock [title] [lang:language] [url] [link text] %}
code snippet
{% endcodeblock %}
```
3. 图片，图片的添加有两种方式，一种是以source文件夹为root的相对路径，一种是通过asset_folder。
```
{% img [class names] /path/to/image [width] [height] [title text [alt text]] %}
eg: {% img /images/xxx.jpg %}

{% asset_img slug [title] %}
eg: {% asset_img xxx.jpg %}
```
4. 视频，可以添加YouTube或者Vimeo的视频。
```
{% youtube video_id %}
{% vimeo video_id %}
```
5. 引用asset_folder的资料，可以给出一个link，或者path。
```
{% asset_path slug %}
{% asset_img slug [title] %}
{% asset_link slug [title] %}
```
<br>
# hexo常见命令
1. 本地调式，如果想让草稿显示出来，可以加draft；如果想浏览器自动打开网页，可以加open。
```
hexo server (--draft) (--open)
```
2. 生成静态网站文件，hexo是一个静态的博客系统，会根据文章、主题等生成静态网页的html(不同于动态网站，通过前后端进行数据交互)，执行这条命令，会生成一个public文件夹，这个文件夹就是整个静态网站的文件夹，可以将其部署到服务器上面。
```
hexo generate(g)
```
3. 部署到远端服务器，详看下一节。
```
hexo deploy(d)
```
<br>
# 部署到Github Pages
Github Pages可以被认为是用户编写的、托管在github上的静态网页。[GitHub Pages](https://pages.github.com/)本用于介绍托管在GitHub的项目，不过，由于他的空间免费稳定，用来做搭建一个博客再好不过了。最重要的是，Github Pages能为用户免费提供服务器，因此部署静态网站就不需要自己搭建服务器和数据库了。
{% asset_img github_pages_homepage.png 800 400%}

1. 创建一个项目的仓库，仓库名为username.github.io，每个账号只能有一个仓库来存放个人主页，可以通过http://username.github.io 来访问你的个人主页。
2. 配置`_config.yml`文件
```
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repo: git@github.com:Vincent-Hoo/Vincent-Hoo.github.io.git
  branch: master
```
3. 安装部署到git的包
```
npm install hexo-deployer-git --save
```
4. 部署到GitHub上，hexo d命令会直接将public文件夹部署到github上对应的仓库，然后github pages会自动根据仓库的改变，部署网页。
```
hexo g
hexo d
```
<br>
# 参考资料
1. [hexo主页](https://hexo.io)
2. 一些博客
    2.1 https://blog.csdn.net/gdutxiaoxu/article/details/53576018
    2.2 https://www.cgmartin.com/2016/01/03/getting-started-with-hexo-blog/
    2.3 https://www.cnblogs.com/dushao/p/5999593.html
3. [hexo主题选择](https://www.zhihu.com/question/24422335)
4. [统计字数，阅读量等](http://ibruce.info/2015/04/04/busuanzi/)
5. [评论系统](https://livere.com/)
6. 视频教学系列
    6.1 https://www.youtube.com/watch?v=Ud1xAhu7t2Y&list=PLXbU-2B80FvDjD_RiuNwsSQ4eF8pkwAIa&index=1
    6.2 https://www.youtube.com/watch?v=Kt7u5kr_P5o&list=PLLAZ4kZ9dFpOMJR6D25ishrSedvsguVSm