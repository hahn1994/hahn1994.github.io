# ☞ HAHN'S MIND PLACE ☜

## 基于Jekyll的博客

主题模版来自[3-Jekyll Theme](https://github.com/P233/P233.github.io)

## 使用

### 设置 `_config.yml`

`_config.yml` 除基本的站点设置外，新加入了社交链接与评论设置。将需要添加的社交帐号填入对应设置，并取消注释，会在头像下方增加一条社交帐号的链接。支持 Twitter, Weibo, Github, Codepen 以及 Dribbble。此外，填入 Disqus 的 shortname 也会启用 Disqus 评论。 `filter` 选项选择使用 `tag` 或 `category` 作为文章分类。

### 修改样式

样式相关的 Sass 变量都存储在 `/css/main.sass` 文件中，修改这个文件可以满足大部分样式定制的需求。建议首先修改 `$gradient-start` 与 `$gradient-end` 两个变量，给自己的博客使用独一无二的侧边栏背景。

### 替换图片

请不要忘记替换 `/assets/img/` 内的图片。`avatar.jpg` 是侧边栏头像的图片，`qrcode.jpg` 会在提示浏览器不兼容时使用。[QR Code 生成器](https://www.unitag.io/qrcode)
