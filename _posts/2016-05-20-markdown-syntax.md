---
layout:   post
title:    Markdown语法简介
date:     2016 May 20 21:49:53
category: 个人杂记
tag:      markdown 语法 sample
---

> 本文转载自[Moon](http://taylantatli.me/Moon/markdown-syntax/)

## HTML 元素

下面是一些HTML元素的Markdown语法的简单应用

# H1标签

## H2标签

### H3标签

#### H4标签

##### H5标签

###### H6标签

### 正文

*强调标签*

**特别强调**

父亲是一个胖子，走过去自然要费事些。我本来要去的，他不肯，只好让他去。我看见他戴着黑布小帽，穿着黑布大马褂，深青布棉袍，蹒跚地走到铁道边，慢慢探身下去，尚不大难。可是他穿过铁道，要爬上那边月台，就不容易了。他用两手攀着上面，两脚再向上缩；他肥胖的身子向左微倾，显出努力的样子。这时我看见他的背影，我的泪很快地流下来了。我赶紧拭干了泪，怕他看见，也怕别人看见。我再向外看时，他已抱了朱红的橘子望回走了。 

![Image](https://github.com/hahn1994/hahn1994.github.io/raw/master/assets/img/favicon.ico)
{: .image-right}

过铁道时，他先将橘子散放在地上，自己慢慢爬下，再抱起橘子走。到这边时，我赶紧去搀他。他和我走到车上，将橘子一股脑儿放在我的皮大衣上。于是扑扑衣上的泥土，心里很轻松似的，过一会说，我走了；到那边来信！我望着他走出去。他走了几步，回过头看见我，说，进去吧，里边没人。等他的背影混入来来往往的人里，再找不着了，我便进来坐下，我的眼泪又来了。

### 引用标签

> 我与父亲不相见已二年余了，我最不能忘记的是他的背影。那年冬天，祖母死了，父亲的差使也交卸了，正是祸不单行的日子，我从北京到徐州，打算跟着父亲奔丧回家。到徐州见着父亲，看见满院狼藉的东西，又想起祖母，不禁簌簌地流下眼泪。父亲说，事已如此，不必难过，好在天无绝人之路！

## 列表标签

### 有序列表

1. 选项1
   1. 子选项1
   2. 子选项2
   3. 子选项3
2. 选项2

### 无序列表

* 选项1
* 选项2
* 选项3

## 表单标签

| 表头一 | 表头二 | 表头三 |
|:------|:-----:|------:|
| 1     | 2     | 3     |
| 4     | 5     | 6     |
|----
| 1     | 2     | 3     |
| 4     | 5     | 6     |
|=====
| 表尾一 | 表尾二 | 表尾三
{: rules="groups"}

## 代码标签

可以指定代码类型并高亮显示关键字

{% highlight css %}
#container {
  float: left;
  margin: 0 -240px 0 0;
  width: 100%;
}
{% endhighlight %}

## 按钮标签

可以用HTML语法定义一个按钮，如下

{% highlight html %}
<a href="#" class="btn">这是一个按钮</a>
{% endhighlight %}

<div markdown="0">
<a href="#" class="btn">正常</a>
<a href="#" class="btn btn-success">成功</a>
<a href="#" class="btn btn-warning">警告</a>
<a href="#" class="btn btn-danger">危险</a>
<a href="#" class="btn btn-info">提示</a>
</div>

## 键盘标签

指定键盘文本的标签

{% highlight html %}
<kbd>K</kbd>
{% endhighlight %}

请按 <kbd>W</kbd><kbd>A</kbd><kbd>S</kbd><kbd>D</kbd> 方向键移动

## 提醒文本

**注意** 我们可以用 `{: .notice}` 在文本中添加一个特别的提示信息，当然，`{: .xx}`的用法需要我们预先在css中设置好相应的样式
{: .notice}