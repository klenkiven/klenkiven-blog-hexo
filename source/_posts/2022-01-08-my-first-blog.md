---
title: 首篇Hexo博客
date: 2022-01-08 20:20:29
tags: [测试]
categories: [Hexo测试]
cover: https://klenkiven-blog-image.oss-cn-zhangjiakou.aliyuncs.com/img/HEXO-LOGO.jpg
---

这是首篇博客的摘要，里面包含了许多测试代码，可以方便编写

<!-- more -->

这是首篇博客

## 官方文档链接
{% link https://xaoxuu.com/wiki/stellar/tag-plugins/ 使用标签插件增强阅读体验 icon:https://avatars.githubusercontent.com/u/16400144?v=4?v=3&s=88 %}

## 测试代码块
```java
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("这是首篇博客");
    }
}
```

## 测试备注标签
{% noteblock note %}
fdsafajsuifasjhdf
{% endnoteblock %}

```md note
{% noteblock note %}
fdsafajsuifasjhdf
{% endnoteblock %}
```

## 测试复制标签
测试复制标签
{% copy curl -s https://example.org/stuff/run.sh | sh %}
```md 写法如下
{% copy curl -s https://example.org/stuff/run.sh | sh %}
```

## 测试修饰文本标签
- 支持多彩标记标签，包括：{% mark 默认 %}{% mark 红 color:red %}{% mark 橙 color:orange %}{% mark 黄 color:yellow %}{% mark 绿 color:green %}{% mark 青 color:cyan %}{% mark 蓝 color:blue %}{% mark 紫 color:purple %}{% mark 浅 color:light %}{% mark 深 color:dark %} 一共 10 种颜色。
- 这是 {% psw 密码 %} 标签
- 这是 {% u 下划线 %} 标签
- 这是 {% emp 着重号 %} 标签
- 这是 {% wavy 波浪线 %} 标签
- 这是 {% del 删除线 %} 标签
- 这是 {% sup 上角标 color:red %} 标签
- 这是 {% sub 下角标 %} 标签
- 这是 {% kbd 键盘样式 %} 标签，试一试：{% kbd ⌘ %} + {% kbd D %}

``` md 写法如下
- 支持多彩标记标签，包括：{% mark 默认 %}{% mark 红 color:red %}{% mark 橙 color:orange %}{% mark 黄 color:yellow %}{% mark 绿 color:green %}{% mark 青 color:cyan %}{% mark 蓝 color:blue %}{% mark 紫 color:purple %}{% mark 浅 color:light %}{% mark 深 color:dark %} 一共 10 种颜色。
- 这是 {% psw 密码 %} 标签
- 这是 {% u 下划线 %} 标签
- 这是 {% emp 着重号 %} 标签
- 这是 {% wavy 波浪线 %} 标签
- 这是 {% del 删除线 %} 标签
- 这是 {% sup 上角标 color:red %} 标签
- 这是 {% sub 下角标 %} 标签
- 这是 {% kbd 键盘样式 %} 标签，试一试：{% kbd ⌘ %} + {% kbd D %}
```

## 测试折叠标签
```
{% folding title [codeblock:bool] [open:bool] [color:color] %}
content
{% endfolding %}
```

{% folding 这是折叠块标题 codeblock:false open:false color:light %}
这是折叠块内容
{% endfolding %}
