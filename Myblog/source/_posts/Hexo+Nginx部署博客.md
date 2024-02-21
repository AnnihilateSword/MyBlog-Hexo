---
title: Hexo+Nginx部署博客
tags: Hexo建站
cover: /res/img/post/Hexo+Nginx部署博客/17.png
top_img: /res/img/site/samurai.png
---
以 Butterfly 主题为例，使用 nginx 进行部署到云服务器上

可以参考 Butterfly 文档的快速开始：https://butterfly.js.org/posts/21cfbf15/

开发风格因人而异，可能会与你想的有所不同，还请包涵^ ^

# 安装依赖

## 下载 Git 和 Nodejs

git: https://git-scm.com/
nodejs: https://nodejs.org/en

## 局部安装 hexo

```shell
$ npm install hexo
```

# 开始建站

## 初始化

```shell
$ npx hexo init Myblog
$ cd Myblog
$ npm install
```

## 下载并应用 butterfly 主题

```shell
git clone -b master https://github.com/jerryc127/hexo-theme-butterfly.git themes/butterfly
```

修改 Hexo 根目录下的 _config.yml，把主题改为 butterfly

```yaml
theme: butterfly
```

## 安装插件

如果你没有 pug 以及 stylus 的渲染器，请下载安装：

```shell
npm install hexo-renderer-pug hexo-renderer-stylus --save
```

## 升级建议

为了减少升级主题后带来的不便，请使用以下方法（建议，可以不做）。

在 hexo 的根目录创建一个文件 `_config.butterfly.yml`，并把主题目录的 `_config.yml` 内容复制到 `_config.butterfly.yml` 去。( 注意: 复制的是主题的 `_config.yml` ，而不是 hexo 的 `_config.yml`)

> 注意： 不要把主题目录的 `_config.yml` 删掉

> 注意： 以后只需要在 `_config.butterfly.yml` 进行配置就行。
> 如果使用了 `_config.butterfly.yml`， 配置主题的 `_config.yml` 将不会有效果。

Hexo 会自动合并主题中的 `_config.yml` 和 `_config.butterfly.yml` 里的配置，如果存在同名配置，会使用 `_config.butterfly.yml` 的配置，其优先度较高。

<img src="/res/img/post/Hexo+Nginx部署博客/1.png" width=800px>

## 主题配置

接下来就是主题配置，文档比我更详细：https://butterfly.js.org/posts/4aa8abbe/

可以按自己想要的配置^ ^

下面列出我目前所修改的内容：

1. 站点配置文件 `_config.yml` 修改

```yml
# Site
title: AnnihilateSword
subtitle: Blog
description: ''
keywords:
author: Jinkun Ou
language: zh-CN
timezone: ''

# URL
## Set your site url here. For example, if you use GitHub Page, set url as 'https://username.github.io/project'
url: http://yunmiezhijian.top/
```

1. 菜单|目录

主题配置文件中

```yml
# Menu 目錄
menu:
  Home: / || fas fa-home
  Archives: /archives/ || fas fa-archive
  Tags: /tags/ || fas fa-tags
  Categories: /categories/ || fas fa-folder-open
  # List||fas fa-list:
  #   Music: /music/ || fas fa-music
  #   Movie: /movies/ || fas fa-video
  Link: /link/ || fas fa-link
  About: /about/ || fas fa-heart
```

3. 高亮主题

修改 `主题配置文件`

```yml
# Code Blocks (代碼相關)
# --------------------------------------

highlight_theme: mac #  darker / pale night / light / ocean / mac / mac light / false
```

4. 社交图标

Butterfly支持 [font-awesome v6](https://fontawesome.com/icons?from=io) 图标.

书写格式 `图标名：url || 描述性文字 || color`

```yml
# Social Settings (社交圖標設置)
# formal:
#   icon: link || the description || color
social:
  # fab fa-github: https://github.com/xxxxx || Github || '#24292e'
  # fas fa-envelope: mailto:xxxxxx@gmail.com || Email || '#4a7dbe'
  fa-brands fa-zhihu: https://www.zhihu.com/people/AnnhiliateSword || Zhihu || '#4a7dbe'
  fa-brands fa-bilibili: https://space.bilibili.com/445428089 || Bilibili || '#4a7dbe'
```

5. 网站图标|头像

修改 `主题配置文件`

```yml
# Favicon（網站圖標）
favicon: /_res/img/site/head.jpg

# Avatar (頭像)
avatar:
  img: /_res/img/site/head.jpg
  effect: false
```

> 这里的根目录相对于 hexo init 创建的项目目录，我这里是 `Myblog/source/_res/img/site`

6. 顶部图

```yml
# Disable all banner image
disable_top_img: false

# The banner image of home page
index_img: /res/img/site/samurai.png

# If the banner of page not setting, it will show the top_img
default_top_img: 

# The banner image of archive page
archive_img: /res/img/site/samurai.png

# If the banner of tag page not setting, it will show the top_img
# note: tag page, not tags page (子標籤頁面的 top_img)
tag_img: /res/img/site/samurai.png

# The banner image of tag page
# format:
#  - tag name: xxxxx
tag_per_img: /res/img/site/samurai.png

# If the banner of category page not setting, it will show the top_img
# note: category page, not categories page (子分類頁面的 top_img)
category_img: /res/img/site/samurai.png

# The banner image of category page
# format:
#  - category name: xxxxx
category_per_img: /res/img/site/samurai.png
```

> 不要以下划线开头，比如 _res，会无法识别图片

7. TOC

在文章页，会有一个目录，用于显示TOC。

修改 `主题配置文件`

```yml
# toc (目錄)
toc:
  post: true
  page: true
  number: true
  expand: false
  style_simple: false # for post
  scroll_percent: true
```

8. 相关文章

> 当文章封面设置为 false 时，或者没有获取到封面配置，相关文章背景将会显示主题色。

相关文章推荐的原理是根据文章tags的比重来推荐

修改 `主题配置文件`

```yml
# Related Articles
related_post:
  enable: true
  limit: 6 # Number of posts displayed
  date_type: created # or created or updated 文章日期顯示創建日或者更新日
```

9. Footer 设置

博客年份

`since`是一个来展示你站点起始时间的选项。它位于页面的最底部。

修改 `主题配置文件`

```yml
# Footer Settings
# --------------------------------------
footer:
  owner:
    enable: true
    since: 2024
  custom_text:
  copyright: true # Copyright of theme and framework
```

10. 侧边栏

修改 `主题配置文件`

```yml
# aside (側邊欄)
# --------------------------------------

aside:
  enable: true
  hide: false
  button: true
  mobile: true # display on mobile
  position: right # left or right
  display:
    archive: true
    tag: true
    category: true
  card_author:
    enable: true
    description:
    button:
      enable: true
      icon: fab fa-github
      text: Follow Me
      link: https://github.com/AnnihilateSword
```

11. 运行时间

网页已运行时间

修改 `主题配置文件`

```yml
# Time difference between publish date and now (網頁運行時間)
# Formal: Month/Day/Year Time or Year/Month/Day Time
runtimeshow:
  enable: true
  publish_date: 2/19/2024 23:54:00
```

12. 右下角按钮

简体繁体互换

右下角会有简繁转换按钮。

修改 `主题配置文件`

```yml
# Conversion between Traditional and Simplified Chinese (簡繁轉換)
translate:
  enable: true
  # The text of a button
  default: 简
  # the language of website (1 - Traditional Chinese/ 2 - Simplified Chinese）
  defaultEncoding: 2
  # Time delay
  translateDelay: 0
  # The text of the button when the language is Simplified Chinese
  msgToTraditionalChinese: '繁'
  # The text of the button when the language is Traditional Chinese
  msgToSimplifiedChinese: '簡'
```

13. 夜间模式

右下角会有夜间模式按钮

修改 `主题配置文件`

```yml
# dark mode
darkmode:
  enable: true
  # Toggle Button to switch dark/light mode
  button: true
  # Switch dark/light mode automatically (自動切換 dark mode和 light mode)
  # autoChangeMode: 1  Following System Settings, if the system doesn't support dark mode, it will switch dark mode between 6 pm to 6 am
  # autoChangeMode: 2  Switch dark mode between 6 pm to 6 am
  # autoChangeMode: false
  autoChangeMode: false
  # Set the light mode time. The value is between 0 and 24. If not set, the default value is 6 and 18
  start: # 8
  end: # 22
```

### 搜索

文档：https://github.com/wzpan/hexo-generator-search

安装 hexo-generator-search

```shell
$ npm install hexo-generator-search --save
```

修改 `主题配置文件`

```yml
# Local search
local_search:
  enable: true
  # Preload the search data when the page loads.
  preload: false
  # Show top n results per article, show all results by setting to -1
  top_n_per_article: 1
  # Unescape html strings to the readable one.
  unescape: false
  CDN:
```

### 美化|特效

#### 自定义主题颜色

可以修改大部分UI颜色

修改 `主题配置文件`，比如：

> 颜色值必须被双引号包裹，就像 "#000" 而不是 #000。否则将会在构建的时候报错！

```yml
theme_color:
  enable: true
  main: "#49B1F5"
  paginator: "#00c4b6"
  button_hover: "#FF7242"
  text_selection: "#00c4b6"
  link_color: "#99a9bf"
  meta_color: "#858585"
  hr_color: "#A4D8FA"
  code_foreground: "#F47466"
  code_background: "rgba(27, 31, 35, .05)"
  toc_color: "#00c4b6"
  blockquote_padding_color: "#49b1f5"
  blockquote_background_color: "#49b1f5"
  scrollbar_color: "#49b1f5"
  meta_theme_color_light: "ffffff"
  meta_theme_color_dark: "#0d0d0d"
```

#### footer 背景

修改 `主题配置文件`

```yml
# Footer Background
footer_bg: true
```

#### 打字效果

修改 `主题配置文件`

```yml
# Typewriter Effect (打字效果)
# https://github.com/disjukr/activate-power-mode
activate_power_mode:
  enable: true
  colorful: true # open particle animation (冒光特效)
  shake: true #  open shake (抖動特效)
  mobile: false
```

#### 鼠标点击效果

修改 `主题配置文件`

```yml
# Mouse click effects: Heart symbol (鼠標點擊效果: 愛心)
click_heart:
  enable: true
  mobile: false
```

#### 网站副标题

可设置主页中显示的网站副标题或者喜欢的座右铭。

修改 `主题配置文件`

```yml
# the subtitle on homepage (主頁subtitle)
subtitle:
  enable: true
  # Typewriter Effect (打字效果)
  effect: true
  # Customize typed.js (配置typed.js)
  # https://github.com/mattboldt/typed.js/#customization
  typed_option:
  # source 調用第三方服務
  # source: false 關閉調用
  # source: 1  調用一言網的一句話（簡體） https://hitokoto.cn/
  # source: 2  調用一句網（簡體） https://yijuzhan.com/
  # source: 3  調用今日詩詞（簡體） https://www.jinrishici.com/
  # subtitle 會先顯示 source , 再顯示 sub 的內容
  source: false
  # 如果關閉打字效果，subtitle 只會顯示 sub 的第一行文字
  sub:
    - 不要轻言放弃
    - Don't give up.
```


#### 加载动画

当进入网页时，因为加载速度的问题，可能会导致 top_img 图片出现断层显示，或者网页加载不全而出现等待时间，开启preloader后，会显示加载动画，等页面加载完，加载动画会消失。

主题支持 pace.js 的加载动画，具体可查看 [pace.js](https://codebyzach.github.io/pace/)

修改 `主题配置文件`

```yml
# Loading Animation (加載動畫)
preloader:
  enable: true
  # source
  # 1. fullpage-loading
  # 2. pace (progress bar)
  source: 1
  # pace theme (see https://codebyzach.github.io/pace/)
  pace_css_url:
```

#### 字数统计

要为`Butterfly`配上字数统计特性, 你需要如下几个步骤:

打开 hexo 工作目录

`npm install hexo-wordcount --save`

修改 `主题配置文件`:

```yml
# wordcount (字數統計)
# see https://butterfly.js.org/posts/ceeb73f/#字數統計
wordcount:
  enable: false
  post_wordcount: true
  min2read: true
  total_wordcount: true
```

#### nstantpage

当鼠标悬停到链接上超过 65 毫秒时，Instantpage 会对该链接进行预加载，可以提升访问速度。

修改配置文件

```yml
# https://instant.page/
# prefetch (預加載)
instantpage: true
```

#### Pangu

如果你跟我一样，每次看到网页上的中文字和英文、数字、符号挤在一块，就会坐立难安，忍不住想在它们之间加个空格。这个外挂正是你在网路世界走跳所需要的东西，它会自动替你在网页中所有的中文字和半形的英文、数字、符号之间插入空白。

```yml
# https://github.com/vinta/pangu.js
# Insert a space between Chinese character and English character (中英文之間添加空格)
pangu:
  enable: true
  field: site # site/post
```

#### CSS前缀

有些 CSS 并不是所有浏览器都支持，需要增加对应的前缀才会生效。

开启 `css_prefix` 后，会自动为一些 CSS 增加前缀。（会增加 20%的体积）

修改配置文件

```yml
# Add the vendor prefixes to ensure compatibility
css_prefix: true
```

### 主题页面添加

#### 标签页

1. 前往你的 Hexo 博客的根目录

2. 输入 `npx hexo new page tags`

3. 你会找到 `source/tags/index.md` 这个文件

4. 修改这个文件：

    记得添加 `type: "tags"`

```md
---
title: Tags
date: 2024-02-20 01:23:49
type: "tags"
---
```

> Tips: 添加一个标签再删除需要 npx hexo clean 清理数据库才能生效

#### 分类页

1. 前往你的 Hexo 博客的根目录

2. 输入 `npx hexo new page categories`

3. 你会找到 `source/categories/index.md` 这个文件

4. 修改这个文件：

    记得添加 `type: "categories"`

```md
---
title: categories
date: 2024-02-20 01:25:07
type: "categories"
---
```

#### 友链

为你的博客创建一个友情链接！

- 创建友链页面

1. 前往你的 Hexo 博客的根目录

2. 输入 `npx hexo new page link`

3. 你会找到 `source/link/index.md` 这个文件

4. 修改这个文件：

    记得添加 `type: "link"`

```md
---
title: link
date: 2024-02-20 01:26:39
type: "link"
---
```

- 友链添加

在Hexo博客目录中的 `source/_data`（如果没有 _data 文件夹，请自行创建），创建一个文件 `link.yml`

```yml
- class_name: 友情链接
  class_desc: 好友
  link_list:
    - name: 都哥
      link: https://ahucd.cn/
      avatar: https://ahucd.cn/img/avatar.png
      descr: 天剑行风

- class_name: 网站
  class_desc: 值得推荐的网站
  link_list:
    - name: Youtube
      link: https://www.youtube.com/
      avatar: https://i.loli.net/2020/05/14/9ZkGg8v3azHJfM1.png
      descr: 视频网站
    - name: Bilibili
      link: https://www.bilibili.com/
      avatar: /res/img/site/bilibili.svg
      descr: 视频网站
```

`class_name` 和 `class_desc` 支持 html 格式书写，如不需要，也可以留空。

> 主题支持友情链接随机排序，只需要在顶部 `front-matter` 添加 `random: true`

#### 子页面

子页面也是普通的页面，你只需要 `hexo n page xxxxx` 创建你的页面就行

例如创建 About 页面

## 文章常用设置

```md
---
title: UE5 C++ 加载 Json 文件
tags: [UE5, C++]
cover: /res/img/post/UE5 C++ 加载 Json 文件/1.png
top_img: /res/img/site/samurai.png
---
```

- title: 标题
- tags: 标签
- cover: 设置封面
- top_img：设置顶部图片

`npx hexo n page about`

# 部署

为了方便维护，我个人这里使用的是 Windows + Nginx 的方式在云服务器上部署

## 安装依赖

在云服务器上我们也需要安装 Git 和 Nodejs

git: https://git-scm.com/
nodejs: https://nodejs.org/en


## 网站项目上传 Git

这里我上传 Github

> 在上传前，建议先备份一下项目，避免损失

在真正上传前，我还要做一些操作

1. 删除主题 butterfly 文件夹中的 `.git` 文件夹，我打算手动更新，避免云服务器上 clone 时有问题

2. 将项目文件夹下的 `.gitignore` 文件删除，我打算全部上传，维护整个项目

3. 生成静态文件
`npx hexo g`

做完这些操作后，就可以新建一个仓库，将整个项目上传上去了

<img src="/res/img/post/Hexo+Nginx部署博客/2.png" width=800px>

```shell
$ git add .
$ git commit -m "<feat>: init"
$ git push
```

## 云服务器上应用项目

```shell
$ git config --global user.name xxx
$ git config --global user.email xxx
$ git clone xxx
```

## 配置 Nginx

打开 `nginx.conf` 在 http 下添加

```txt
server {
    listen 80;
    server_name 域名/IP;

    location / {
        root xxx/public;
        index index.html index.htm;
    }
}
```

- 验证配置是否 ok

```shell
$ ./nginx.exe -t -c ./conf/nginx.conf
```

- 重启 nginx

```shell
$ ./nginx.exe -s reload
```

> Tip：有时候 404 是端口被占用引起的，可以重启一下

有些服务器需要手动开启端口防火墙，以我这里为例

<img src="/res/img/post/Hexo+Nginx部署博客/3.png" width=800px>

<img src="/res/img/post/Hexo+Nginx部署博客/4.png" width=800px>

<img src="/res/img/post/Hexo+Nginx部署博客/5.png" width=800px>

<img src="/res/img/post/Hexo+Nginx部署博客/6.png" width=800px>

<img src="/res/img/post/Hexo+Nginx部署博客/7.png" width=800px>

通过公网 IP 访问进行测试

<img src="/res/img/post/Hexo+Nginx部署博客/8.png" width=800px>

# 域名配置

这里以腾讯云为例

- 需要先购买一个域名

<img src="/res/img/post/Hexo+Nginx部署博客/9.png" width=800px>

<img src="/res/img/post/Hexo+Nginx部署博客/10.png" width=800px>

<img src="/res/img/post/Hexo+Nginx部署博客/11.png" width=800px>

<img src="/res/img/post/Hexo+Nginx部署博客/12.png" width=800px>

- 刚购买好需要等待审核，这里以我之前买的域名为例，添加解析

<img src="/res/img/post/Hexo+Nginx部署博客/13.png" width=800px>

<img src="/res/img/post/Hexo+Nginx部署博客/14.png" width=800px>

<img src="/res/img/post/Hexo+Nginx部署博客/15.png" width=800px>

<img src="/res/img/post/Hexo+Nginx部署博客/16.png" width=800px>

- 最终效果

<img src="/res/img/post/Hexo+Nginx部署博客/17.png" width=800px>

The End.