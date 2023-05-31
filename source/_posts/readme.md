---
title: Init Blog
date: 2023-05-30 14:55:52
tags:
- Blog
---

**Preface:** 重载了系统，我的博客又无了，现在吸收了教训，从无到有开始记录以下创建博客的过程。

### 使用 npm 下载 `hexo-theme-next` 最新版

```shell
$ npm install hexo-theme-next@latest
```

2. 设置 Hexo 主题

编辑 Hexo config file 设置 theme 为 next

```  json  Hexo config file
theme: next
```

这时运行 hexo s 会报错 `Error: Cannot find module 'css'`，下载 css 包就行

```shell
npm install css
```

然后就可以成功启动 next 主题的博客

先跳过 deploy 的过程，先把本地能配的都配好

### Next 配置文件

为了能在未来更新 next 主题，应该把配置文件的信息放在 hexo-site 位置下

- 确保是 hexo 5.0 以后的版本
- 在 hexo-site 创建一个 _config.next.yml 文件
- 在 next theme 中把所有的信息都复制过来

```shell
# Installed through npm
cp node_modules/hexo-theme-next/_config.yml _config.next.yml
```

### 配置 Hexo 信息以及 Next 主题配置

在 _config.yml 下把该填的都填了

#### Sidebar

- 设置 Avatar

在 Next config file 中编辑 avatar 中的 url ，改为图片的位置/地址

- Site State

设置 Next config file 中的 site state 为 true

``` Next config file
site_state: true
```

- Social Link

```json Next config file
social:
  GitHub: https://github.com/yourname || fab fa-github
  E-Mail: mailto:yourname@gmail.com || fa fa-envelope
  Weibo: https://weibo.com/yourname || fab fa-weibo
  Twitter: https://twitter.com/yourname || fab fa-twitter
  FB Page: https://www.facebook.com/yourname || fab fa-facebook
  StackOverflow: https://stackoverflow.com/yourname || fab fa-stack-overflow
  YouTube: https://youtube.com/yourname || fab fa-youtube
  Instagram: https://instagram.com/yourname || fab fa-instagram
  Skype: skype:yourname?call|chat || fab fa-skype
```

其他的按照 Siderbar 的默认设置即可

#### Footer

设置以下备案号，把 power 设置为 false ，copyright之后再设置

#### Posts

- Post Meta Display

默认

- Post Wordcount

不设置，不用

- Tag Icon

设置为 true

- Post Edit

enable 设置为 true

设置 url

For site repository

Link for view source: `url: https://github.com/.../tree/master/source/_posts/`

Link for fork & edit: `url: https://github.com/.../edit/master/source/_posts/`

For post repository

Link for view source: `url: https://github.com/.../_posts/tree/master/`

Link for fork & edit: `url: https://github.com/.../_posts/edit/master/`

- Post Navigation

默认为 left

#### Custom Pages

将 Next config file 的 menu 设置中把所需要的几个 page 前的注释 # 去掉，然后 hexo new page custom-name 创建这几个界面

- 添加 Tag page && categories page

```shell
$ cd hexo-site
$ hexo new page tags
```

然后把 index.md 中的 type 改称 tags

categories 同理

- 添加 Google Calendar page

获取 google api key 和 google calendar key，填入 Next config file 中的位置

添加 schedule page ，类似 tags

#### Misc 设置

- Preconnect
- Mobile device adaption
- Body Scrollbar
- Back to Top
- Font Customization

将 code.fontfamily 设置为 Fira Code

#### SEO

设置 Hexo config file 中的 url ，设置 Next config file 中 index_with_subtitle 为 true

- Google Search Console

在 Next config file 中修改

```json Next config file
google_site_verification: XXXXXXXXXXXXXXXXXXXXXXX
```

#### Mermaid

```shell NexT config file
# Mermaid tag
mermaid:
  enable: true
  # Available themes: default | dark | forest | neutral
  theme:
    light: default
    dark: dark
```

```shell Hexo config file
highlight:
  exclude_languages:
    - mermaid
```

{% mermaid sequenceDiagram %}
Alice->>John: Hello John, how are you?
loop Healthcheck
    John->>John: Fight against hypochondria
end
Note right of John: Rational thoughts!
John-->>Alice: Great!
John->>Bob: How about you?
Bob-->>John: Jolly good!
{% endmermaid %}

#### Note

```shell Next config file
note:
  # Note tag style values:
  #  - simple    bs-callout old alert style. Default.
  #  - modern    bs-callout new (v2-v3) alert style.
  #  - flat      flat callout style with background, like on Mozilla or StackOverflow.
  #  - disabled  disable all CSS styles import of note tag.
  style: simple
  icons: false
  # Offset lighter of background in % for modern and flat styles (modern: -12 | 12; flat: -18 | 6).
  # Offset also applied to label tag variables. This option can work with disabled note tag.
  light_bg_offset: 0
```

```
{% note %}
#### Header
(without define class style)
{% endnote %}
```

{% note %}

##### Header

(without define class style)
{% endnote %}

```
{% note default %}
#### Default Header
Welcome to [Hexo!](https://hexo.io)
{% endnote %}
```

{% note default %}

##### Default Header

Welcome to [Hexo!](https://hexo.io)
{% endnote %}

```
{% note primary %}
#### Primary Header
**Welcome** to [Hexo!](https://hexo.io)
{% endnote %}
```

{% note primary %}

##### Primary Header

**Welcome** to [Hexo!](https://hexo.io)
{% endnote %}

```
{% note info %}
#### Info Header
**Welcome** to [Hexo!](https://hexo.io)
{% endnote %}
```

{% note info %}

##### Info Header

**Welcome** to [Hexo!](https://hexo.io)
{% endnote %}

```
{% note success %}
#### Success Header
**Welcome** to [Hexo!](https://hexo.io)
{% endnote %}
```

{% note success %}

##### Success Header

**Welcome** to [Hexo!](https://hexo.io)
{% endnote %}

```
{% note warning %}
#### Warning Header
**Welcome** to [Hexo!](https://hexo.io)
{% endnote %}
```

{% note warning %}

##### Warning Header

**Welcome** to [Hexo!](https://hexo.io)
{% endnote %}

```
{% note danger %}
#### Danger Header
**Welcome** to [Hexo!](https://hexo.io)
{% endnote %}
```

{% note danger %}

##### Danger Header

**Welcome** to [Hexo!](https://hexo.io)
{% endnote %}

```
{% note info no-icon %}
#### No icon note
Note **without** icon: `note info no-icon`
{% endnote %}
```

{% note info no-icon %}

##### No icon note

Note **without** icon: `note info no-icon`
{% endnote %}

```
{% note primary This is a summary %}
#### Details and summary
Note with summary: `note primary This is a summary`
{% endnote %}
```

{% note primary This is a summary %}

##### Details and summary

Note with summary: `note primary This is a summary`
{% endnote %}

```
{% note info no-icon This is a summary %}
#### Details and summary (No icon)
Note with summary: `note info no-icon This is a summary`
{% endnote %}
```

{% note info no-icon This is a summary %}

##### Details and summary (No icon)

Note with summary: `note info no-icon This is a summary`
{% endnote %}

````
{% note success %}
#### Codeblock in note

```
code block in note tag
code block in note tag
code block in note tag
```
{% endnote %}
````

{% note success %}

##### Codeblock in note

```
code block in note tag
code block in note tag
code block in note tag
```
{% endnote %}

```
{% note default %}
#### Lists in note
* ul
* ul
    * ul
    * ul
* ul

1. ol
2. ol
    1. ol
    2. ol
3. ol
{% endnote %}
```

{% note default %}

##### Lists in note

* ul
* ul
    * ul
    * ul
* ul

1. ol
2. ol
    1. ol
    2. ol
3. ol
    {% endnote %}

```
#### Table in Note
{% note default %}
| 1 | 2 |
| - | - |
| 3 | 4 |
| 5 | 6 |
| 7 | 8 |
{% endnote %}
```

##### Table in Note

{% note default %}
| 1    | 2    |
| :--- | ---- |
| 3    | 4    |
| 5    | 6    |
| 7    | 8    |

{% endnote %}

#### Tabs

```shell Next config file
tabs:
  # Make the nav bar of tabs with long content stick to the top.
  sticky: false
  transition:
    tabs: false
    labels: true
```

```
{% tabs First unique name %}
<!-- tab -->
**This is Tab 1.**
<!-- endtab -->

<!-- tab -->
**This is Tab 2.**
<!-- endtab -->

<!-- tab -->
**This is Tab 3.**
<!-- endtab -->
{% endtabs %}
```

{% tabs First unique name %}
<!-- tab -->
**This is Tab 1.**
<!-- endtab -->

<!-- tab -->
**This is Tab 2.**
<!-- endtab -->

<!-- tab -->
**This is Tab 3.**
<!-- endtab -->
{% endtabs %}

##### Tabs with 3rd tab selected

```
{% tabs Second unique name, 3 %}
<!-- tab -->
**This is Tab 1.**
<!-- endtab -->

<!-- tab -->
**This is Tab 2.**
<!-- endtab -->

<!-- tab -->
**This is Tab 3.**
<!-- endtab -->
{% endtabs %}
```

{% tabs Second unique name, 3 %}
<!-- tab -->
**This is Tab 1.**
<!-- endtab -->

<!-- tab -->
**This is Tab 2.**
<!-- endtab -->

<!-- tab -->
**This is Tab 3.**
<!-- endtab -->
{% endtabs %}

##### Tabs with no tab selected

```
{% tabs Third unique name, -1 %}
<!-- tab -->
**This is Tab 1.**
<!-- endtab -->

<!-- tab -->
**This is Tab 2.**
<!-- endtab -->

<!-- tab -->
**This is Tab 3.**
<!-- endtab -->
{% endtabs %}
```

{% tabs Third unique name, -1 %}
<!-- tab -->
**This is Tab 1.**
<!-- endtab -->

<!-- tab -->
**This is Tab 2.**
<!-- endtab -->

<!-- tab -->
**This is Tab 3.**
<!-- endtab -->
{% endtabs %}

##### Tabs with custom labels

```
{% tabs Fourth unique name %}
<!-- tab Solution 1 -->
**This is Tab 1.**
<!-- endtab -->

<!-- tab Solution 2 -->
**This is Tab 2.**
<!-- endtab -->

<!-- tab Solution 3 -->
**This is Tab 3.**
<!-- endtab -->
{% endtabs %}
```

{% tabs Fourth unique name %}
<!-- tab Solution 1 -->
**This is Tab 1.**
<!-- endtab -->

<!-- tab Solution 2 -->
**This is Tab 2.**
<!-- endtab -->

<!-- tab Solution 3 -->
**This is Tab 3.**
<!-- endtab -->
{% endtabs %}

##### Tabs with icons only

```
{% tabs Fifth unique name %}
<!-- tab @text-width -->
**This is Tab 1.**
<!-- endtab -->

<!-- tab @font -->
**This is Tab 2.**
<!-- endtab -->

<!-- tab @bold -->
**This is Tab 3.**
<!-- endtab -->
{% endtabs %}
```

{% tabs Fifth unique name %}
<!-- tab @text-width -->
**This is Tab 1.**
<!-- endtab -->

<!-- tab @font -->
**This is Tab 2.**
<!-- endtab -->

<!-- tab @bold -->
**This is Tab 3.**
<!-- endtab -->
{% endtabs %}

##### Tabs with icons and labels

```
{% tabs Sixth unique name %}
<!-- tab Solution 1@text-width -->
**This is Tab 1.**
<!-- endtab -->

<!-- tab Solution 2@font -->
**This is Tab 2.**
<!-- endtab -->

<!-- tab Solution 3@bold -->
**This is Tab 3.**
<!-- endtab -->
{% endtabs %}
```

{% tabs Sixth unique name %}
<!-- tab Solution 1@text-width -->
**This is Tab 1.**
<!-- endtab -->

<!-- tab Solution 2@font -->
**This is Tab 2.**
<!-- endtab -->

<!-- tab Solution 3@bold -->
**This is Tab 3.**
<!-- endtab -->
{% endtabs %}

##### Tabs permalinks test

```
Permalink for > [Tab one](#tab-one).
Permalink for > [Tab one 1](#tab-one-1).
Permalink for > [Tab one 2](#tab-one-2).
Permalink for > [Tab one 3](#tab-one-3).

Permalink for > [Tab two](#tab-two).
Permalink for > [Tab two 1](#tab-two-1).
Permalink for > [Tab two 2](#tab-two-2).
Permalink for > [Tab two 3](#tab-two-3).

{% tabs Tab one %}
<!-- tab -->
**This is Tab 1.**
<!-- endtab -->

<!-- tab -->
**This is Tab 2.**
<!-- endtab -->

<!-- tab -->
**This is Tab 3.**
<!-- endtab -->
{% endtabs %}

{% tabs Tab two %}
<!-- tab -->
**This is Tab 1.**
<!-- endtab -->

<!-- tab -->
**This is Tab 2.**
<!-- endtab -->

<!-- tab -->
**This is Tab 3.**
<!-- endtab -->
{% endtabs %}
```

Permalink for > [Tab one](#tab-one).
Permalink for > [Tab one 1](#tab-one-1).
Permalink for > [Tab one 2](#tab-one-2).
Permalink for > [Tab one 3](#tab-one-3).

Permalink for > [Tab two](#tab-two).
Permalink for > [Tab two 1](#tab-two-1).
Permalink for > [Tab two 2](#tab-two-2).
Permalink for > [Tab two 3](#tab-two-3).

{% tabs Tab one %}
<!-- tab -->
**This is Tab 1.**
<!-- endtab -->

<!-- tab -->
**This is Tab 2.**
<!-- endtab -->

<!-- tab -->
**This is Tab 3.**
<!-- endtab -->
{% endtabs %}

{% tabs Tab two %}
<!-- tab -->
**This is Tab 1.**
<!-- endtab -->

<!-- tab -->
**This is Tab 2.**
<!-- endtab -->

<!-- tab -->
**This is Tab 3.**
<!-- endtab -->
{% endtabs %}

##### Tabs with other tags

```
{% tabs Tags %}
<!-- tab -->
**This is Tab 1.**

1. One
2. Two
3. Three

Indented code block:

    nano /etc

Tagged code block:

{% code %}
code tag
code tag
code tag
{% endcode %}

{% note default %}
Note default tag.
{% endnote %}
<!-- endtab -->

<!-- tab -->
**This is Tab 2.**

* Five
* Six
* Seven

{% note primary %}
{% youtube Kt7u5kr_P5o %}
{% endnote %}
<!-- endtab -->

<!-- tab -->
**This is Tab 3.**

{% subtabs Sub Tabs %}
<!-- tab -->
**This is Sub Tab 1.**
{% note success %}
Lorem ipsum dolor sit amet, consectetuer adipiscing elit. Phasellus hendrerit. Pellentesque aliquet nibh nec urna. In nisi neque, aliquet vel, dapibus id, mattis vel, nisi. Sed pretium, ligula sollicitudin laoreet viverra, tortor libero sodales leo, eget blandit nunc tortor eu nibh. Nullam mollis. Ut justo. Suspendisse potenti. Pellentesque fermentum dolor. Aliquam quam lectus, facilisis auctor, ultrices ut, elementum vulputate, nunc.

{% note warning %}
Sed egestas, ante et vulputate volutpat, eros pede semper est, vitae luctus metus libero eu augue. Morbi purus libero, faucibus adipiscing, commodo quis, gravida id, est. Sed lectus. Praesent elementum hendrerit tortor. Sed semper lorem at felis. Vestibulum volutpat, lacus a ultrices sagittis, mi neque euismod dui, eu pulvinar nunc sapien ornare nisl. Phasellus pede arcu, dapibus eu, fermentum et, dapibus sed, urna.
{% endnote %}

Morbi interdum mollis sapien. Sed ac risus. Phasellus lacinia, magna a ullamcorper laoreet, lectus arcu pulvinar risus, vitae facilisis libero dolor a purus. Sed vel lacus. Mauris nibh felis, adipiscing varius, adipiscing in, lacinia vel, tellus. Suspendisse ac urna. Etiam pellentesque mauris ut lectus. Nunc tellus ante, mattis eget, gravida vitae, ultricies ac, leo. Integer leo pede, ornare a, lacinia eu, vulputate vel, nisl.
{% endnote %}
<!-- endtab -->

<!-- tab -->
**This is Sub Tab 2.**
{% note success %}
Lorem ipsum dolor sit amet, consectetuer adipiscing elit. Phasellus hendrerit. Pellentesque aliquet nibh nec urna. In nisi neque, aliquet vel, dapibus id, mattis vel, nisi. Sed pretium, ligula sollicitudin laoreet viverra, tortor libero sodales leo, eget blandit nunc tortor eu nibh. Nullam mollis. Ut justo. Suspendisse potenti. Pellentesque fermentum dolor. Aliquam quam lectus, facilisis auctor, ultrices ut, elementum vulputate, nunc.

Sed egestas, ante et vulputate volutpat, eros pede semper est, vitae luctus metus libero eu augue. Morbi purus libero, faucibus adipiscing, commodo quis, gravida id, est. Sed lectus. Praesent elementum hendrerit tortor. Sed semper lorem at felis. Vestibulum volutpat, lacus a ultrices sagittis, mi neque euismod dui, eu pulvinar nunc sapien ornare nisl. Phasellus pede arcu, dapibus eu, fermentum et, dapibus sed, urna.

{% note danger %}
Morbi interdum mollis sapien. Sed ac risus. Phasellus lacinia, magna a ullamcorper laoreet, lectus arcu pulvinar risus, vitae facilisis libero dolor a purus. Sed vel lacus. Mauris nibh felis, adipiscing varius, adipiscing in, lacinia vel, tellus. Suspendisse ac urna. Etiam pellentesque mauris ut lectus. Nunc tellus ante, mattis eget, gravida vitae, ultricies ac, leo. Integer leo pede, ornare a, lacinia eu, vulputate vel, nisl.
{% endnote %}
{% endnote %}
<!-- endtab -->

<!-- tab -->
**This is Sub Tab 3.**

{% subtabs Sub-Sub Tabs %}
<!-- tab -->
**This is Sub-Sub Tab 1 of Sub Tab 3.**
{% note success %}
Lorem ipsum dolor sit amet, consectetuer adipiscing elit. Phasellus hendrerit. Pellentesque aliquet nibh nec urna. In nisi neque, aliquet vel, dapibus id, mattis vel, nisi. Sed pretium, ligula sollicitudin laoreet viverra, tortor libero sodales leo, eget blandit nunc tortor eu nibh. Nullam mollis. Ut justo. Suspendisse potenti. Pellentesque fermentum dolor. Aliquam quam lectus, facilisis auctor, ultrices ut, elementum vulputate, nunc.

Sed egestas, ante et vulputate volutpat, eros pede semper est, vitae luctus metus libero eu augue. Morbi purus libero, faucibus adipiscing, commodo quis, gravida id, est. Sed lectus. Praesent elementum hendrerit tortor. Sed semper lorem at felis. Vestibulum volutpat, lacus a ultrices sagittis, mi neque euismod dui, eu pulvinar nunc sapien ornare nisl. Phasellus pede arcu, dapibus eu, fermentum et, dapibus sed, urna.

Morbi interdum mollis sapien. Sed ac risus. Phasellus lacinia, magna a ullamcorper laoreet, lectus arcu pulvinar risus, vitae facilisis libero dolor a purus. Sed vel lacus. Mauris nibh felis, adipiscing varius, adipiscing in, lacinia vel, tellus. Suspendisse ac urna. Etiam pellentesque mauris ut lectus. Nunc tellus ante, mattis eget, gravida vitae, ultricies ac, leo. Integer leo pede, ornare a, lacinia eu, vulputate vel, nisl.
{% endnote %}
<!-- endtab -->

<!-- tab -->
**This is Sub-Sub Tab 2 of Sub Tab 3.**
{% note success %}
Lorem ipsum dolor sit amet, consectetuer adipiscing elit. Phasellus hendrerit. Pellentesque aliquet nibh nec urna. In nisi neque, aliquet vel, dapibus id, mattis vel, nisi. Sed pretium, ligula sollicitudin laoreet viverra, tortor libero sodales leo, eget blandit nunc tortor eu nibh. Nullam mollis. Ut justo. Suspendisse potenti. Pellentesque fermentum dolor. Aliquam quam lectus, facilisis auctor, ultrices ut, elementum vulputate, nunc.

{% note warning %}
Sed egestas, ante et vulputate volutpat, eros pede semper est, vitae luctus metus libero eu augue. Morbi purus libero, faucibus adipiscing, commodo quis, gravida id, est. Sed lectus. Praesent elementum hendrerit tortor. Sed semper lorem at felis. Vestibulum volutpat, lacus a ultrices sagittis, mi neque euismod dui, eu pulvinar nunc sapien ornare nisl. Phasellus pede arcu, dapibus eu, fermentum et, dapibus sed, urna.

Morbi interdum mollis sapien. Sed ac risus. Phasellus lacinia, magna a ullamcorper laoreet, lectus arcu pulvinar risus, vitae facilisis libero dolor a purus. Sed vel lacus. Mauris nibh felis, adipiscing varius, adipiscing in, lacinia vel, tellus. Suspendisse ac urna. Etiam pellentesque mauris ut lectus. Nunc tellus ante, mattis eget, gravida vitae, ultricies ac, leo. Integer leo pede, ornare a, lacinia eu, vulputate vel, nisl.
{% endnote %}

{% endnote %}
<!-- endtab -->

<!-- tab -->
**This is Sub-Sub Tab 3 of Sub Tab 3.**

{% note success %}
Lorem ipsum dolor sit amet, consectetuer adipiscing elit. Phasellus hendrerit. Pellentesque aliquet nibh nec urna. In nisi neque, aliquet vel, dapibus id, mattis vel, nisi. Sed pretium, ligula sollicitudin laoreet viverra, tortor libero sodales leo, eget blandit nunc tortor eu nibh. Nullam mollis. Ut justo. Suspendisse potenti. Pellentesque fermentum dolor. Aliquam quam lectus, facilisis auctor, ultrices ut, elementum vulputate, nunc.

{% note warning %}
Sed egestas, ante et vulputate volutpat, eros pede semper est, vitae luctus metus libero eu augue. Morbi purus libero, faucibus adipiscing, commodo quis, gravida id, est. Sed lectus. Praesent elementum hendrerit tortor. Sed semper lorem at felis. Vestibulum volutpat, lacus a ultrices sagittis, mi neque euismod dui, eu pulvinar nunc sapien ornare nisl. Phasellus pede arcu, dapibus eu, fermentum et, dapibus sed, urna.

{% note danger %}
Morbi interdum mollis sapien. Sed ac risus. Phasellus lacinia, magna a ullamcorper laoreet, lectus arcu pulvinar risus, vitae facilisis libero dolor a purus. Sed vel lacus. Mauris nibh felis, adipiscing varius, adipiscing in, lacinia vel, tellus. Suspendisse ac urna. Etiam pellentesque mauris ut lectus. Nunc tellus ante, mattis eget, gravida vitae, ultricies ac, leo. Integer leo pede, ornare a, lacinia eu, vulputate vel, nisl.
{% endnote %}

{% endnote %}

{% endnote %}
<!-- endtab -->
{% endsubtabs %}

<!-- endtab -->
{% endsubtabs %}

<!-- endtab -->
{% endtabs %}
```

{% tabs Tags %}
<!-- tab -->
**This is Tab 1.**

1. One
2. Two
3. Three

Indented code block:

    nano /etc

Tagged code block:

{% code %}
code tag
code tag
code tag
{% endcode %}

{% note default %}
Note default tag.
{% endnote %}
<!-- endtab -->

<!-- tab -->
**This is Tab 2.**

* Five
* Six
* Seven

{% note primary %}
{% youtube Kt7u5kr_P5o %}
{% endnote %}
<!-- endtab -->

<!-- tab -->
**This is Tab 3.**

{% subtabs Sub Tabs %}
<!-- tab -->
**This is Sub Tab 1.**
{% note success %}
Lorem ipsum dolor sit amet, consectetuer adipiscing elit. Phasellus hendrerit. Pellentesque aliquet nibh nec urna. In nisi neque, aliquet vel, dapibus id, mattis vel, nisi. Sed pretium, ligula sollicitudin laoreet viverra, tortor libero sodales leo, eget blandit nunc tortor eu nibh. Nullam mollis. Ut justo. Suspendisse potenti. Pellentesque fermentum dolor. Aliquam quam lectus, facilisis auctor, ultrices ut, elementum vulputate, nunc.

{% note warning %}
Sed egestas, ante et vulputate volutpat, eros pede semper est, vitae luctus metus libero eu augue. Morbi purus libero, faucibus adipiscing, commodo quis, gravida id, est. Sed lectus. Praesent elementum hendrerit tortor. Sed semper lorem at felis. Vestibulum volutpat, lacus a ultrices sagittis, mi neque euismod dui, eu pulvinar nunc sapien ornare nisl. Phasellus pede arcu, dapibus eu, fermentum et, dapibus sed, urna.
{% endnote %}

Morbi interdum mollis sapien. Sed ac risus. Phasellus lacinia, magna a ullamcorper laoreet, lectus arcu pulvinar risus, vitae facilisis libero dolor a purus. Sed vel lacus. Mauris nibh felis, adipiscing varius, adipiscing in, lacinia vel, tellus. Suspendisse ac urna. Etiam pellentesque mauris ut lectus. Nunc tellus ante, mattis eget, gravida vitae, ultricies ac, leo. Integer leo pede, ornare a, lacinia eu, vulputate vel, nisl.
{% endnote %}
<!-- endtab -->

<!-- tab -->
**This is Sub Tab 2.**
{% note success %}
Lorem ipsum dolor sit amet, consectetuer adipiscing elit. Phasellus hendrerit. Pellentesque aliquet nibh nec urna. In nisi neque, aliquet vel, dapibus id, mattis vel, nisi. Sed pretium, ligula sollicitudin laoreet viverra, tortor libero sodales leo, eget blandit nunc tortor eu nibh. Nullam mollis. Ut justo. Suspendisse potenti. Pellentesque fermentum dolor. Aliquam quam lectus, facilisis auctor, ultrices ut, elementum vulputate, nunc.

Sed egestas, ante et vulputate volutpat, eros pede semper est, vitae luctus metus libero eu augue. Morbi purus libero, faucibus adipiscing, commodo quis, gravida id, est. Sed lectus. Praesent elementum hendrerit tortor. Sed semper lorem at felis. Vestibulum volutpat, lacus a ultrices sagittis, mi neque euismod dui, eu pulvinar nunc sapien ornare nisl. Phasellus pede arcu, dapibus eu, fermentum et, dapibus sed, urna.

{% note danger %}
Morbi interdum mollis sapien. Sed ac risus. Phasellus lacinia, magna a ullamcorper laoreet, lectus arcu pulvinar risus, vitae facilisis libero dolor a purus. Sed vel lacus. Mauris nibh felis, adipiscing varius, adipiscing in, lacinia vel, tellus. Suspendisse ac urna. Etiam pellentesque mauris ut lectus. Nunc tellus ante, mattis eget, gravida vitae, ultricies ac, leo. Integer leo pede, ornare a, lacinia eu, vulputate vel, nisl.
{% endnote %}
{% endnote %}
<!-- endtab -->

<!-- tab -->
**This is Sub Tab 3.**

{% subtabs Sub-Sub Tabs %}
<!-- tab -->
**This is Sub-Sub Tab 1 of Sub Tab 3.**
{% note success %}
Lorem ipsum dolor sit amet, consectetuer adipiscing elit. Phasellus hendrerit. Pellentesque aliquet nibh nec urna. In nisi neque, aliquet vel, dapibus id, mattis vel, nisi. Sed pretium, ligula sollicitudin laoreet viverra, tortor libero sodales leo, eget blandit nunc tortor eu nibh. Nullam mollis. Ut justo. Suspendisse potenti. Pellentesque fermentum dolor. Aliquam quam lectus, facilisis auctor, ultrices ut, elementum vulputate, nunc.

Sed egestas, ante et vulputate volutpat, eros pede semper est, vitae luctus metus libero eu augue. Morbi purus libero, faucibus adipiscing, commodo quis, gravida id, est. Sed lectus. Praesent elementum hendrerit tortor. Sed semper lorem at felis. Vestibulum volutpat, lacus a ultrices sagittis, mi neque euismod dui, eu pulvinar nunc sapien ornare nisl. Phasellus pede arcu, dapibus eu, fermentum et, dapibus sed, urna.

Morbi interdum mollis sapien. Sed ac risus. Phasellus lacinia, magna a ullamcorper laoreet, lectus arcu pulvinar risus, vitae facilisis libero dolor a purus. Sed vel lacus. Mauris nibh felis, adipiscing varius, adipiscing in, lacinia vel, tellus. Suspendisse ac urna. Etiam pellentesque mauris ut lectus. Nunc tellus ante, mattis eget, gravida vitae, ultricies ac, leo. Integer leo pede, ornare a, lacinia eu, vulputate vel, nisl.
{% endnote %}
<!-- endtab -->

<!-- tab -->
**This is Sub-Sub Tab 2 of Sub Tab 3.**
{% note success %}
Lorem ipsum dolor sit amet, consectetuer adipiscing elit. Phasellus hendrerit. Pellentesque aliquet nibh nec urna. In nisi neque, aliquet vel, dapibus id, mattis vel, nisi. Sed pretium, ligula sollicitudin laoreet viverra, tortor libero sodales leo, eget blandit nunc tortor eu nibh. Nullam mollis. Ut justo. Suspendisse potenti. Pellentesque fermentum dolor. Aliquam quam lectus, facilisis auctor, ultrices ut, elementum vulputate, nunc.

{% note warning %}
Sed egestas, ante et vulputate volutpat, eros pede semper est, vitae luctus metus libero eu augue. Morbi purus libero, faucibus adipiscing, commodo quis, gravida id, est. Sed lectus. Praesent elementum hendrerit tortor. Sed semper lorem at felis. Vestibulum volutpat, lacus a ultrices sagittis, mi neque euismod dui, eu pulvinar nunc sapien ornare nisl. Phasellus pede arcu, dapibus eu, fermentum et, dapibus sed, urna.

Morbi interdum mollis sapien. Sed ac risus. Phasellus lacinia, magna a ullamcorper laoreet, lectus arcu pulvinar risus, vitae facilisis libero dolor a purus. Sed vel lacus. Mauris nibh felis, adipiscing varius, adipiscing in, lacinia vel, tellus. Suspendisse ac urna. Etiam pellentesque mauris ut lectus. Nunc tellus ante, mattis eget, gravida vitae, ultricies ac, leo. Integer leo pede, ornare a, lacinia eu, vulputate vel, nisl.
{% endnote %}

{% endnote %}
<!-- endtab -->

<!-- tab -->
**This is Sub-Sub Tab 3 of Sub Tab 3.**

{% note success %}
Lorem ipsum dolor sit amet, consectetuer adipiscing elit. Phasellus hendrerit. Pellentesque aliquet nibh nec urna. In nisi neque, aliquet vel, dapibus id, mattis vel, nisi. Sed pretium, ligula sollicitudin laoreet viverra, tortor libero sodales leo, eget blandit nunc tortor eu nibh. Nullam mollis. Ut justo. Suspendisse potenti. Pellentesque fermentum dolor. Aliquam quam lectus, facilisis auctor, ultrices ut, elementum vulputate, nunc.

{% note warning %}
Sed egestas, ante et vulputate volutpat, eros pede semper est, vitae luctus metus libero eu augue. Morbi purus libero, faucibus adipiscing, commodo quis, gravida id, est. Sed lectus. Praesent elementum hendrerit tortor. Sed semper lorem at felis. Vestibulum volutpat, lacus a ultrices sagittis, mi neque euismod dui, eu pulvinar nunc sapien ornare nisl. Phasellus pede arcu, dapibus eu, fermentum et, dapibus sed, urna.

{% note danger %}
Morbi interdum mollis sapien. Sed ac risus. Phasellus lacinia, magna a ullamcorper laoreet, lectus arcu pulvinar risus, vitae facilisis libero dolor a purus. Sed vel lacus. Mauris nibh felis, adipiscing varius, adipiscing in, lacinia vel, tellus. Suspendisse ac urna. Etiam pellentesque mauris ut lectus. Nunc tellus ante, mattis eget, gravida vitae, ultricies ac, leo. Integer leo pede, ornare a, lacinia eu, vulputate vel, nisl.
{% endnote %}

{% endnote %}

{% endnote %}
<!-- endtab -->
{% endsubtabs %}

<!-- endtab -->
{% endsubtabs %}

<!-- endtab -->
{% endtabs %}



#### Centered Quote

```
{% centerquote %}Something{% endcenterquote %}
```

{% note danger %}

这里遇到了错误，无法加载上面的引用

**解决：**

```shell
npm install hexo-renderer-swig
```

{% endnote %}

{% cq %}Something{% endcq %}

