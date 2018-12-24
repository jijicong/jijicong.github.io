---
title: markdown-整理
date: 2018-12-24 11:30:26
tags: markdown
categories: 工具
---

# Markdown -- 整理

###  一 、标题

``` markdown
1. 最高阶标题
This is an H1
=============

2. 第二阶标题
This is an H2
-------------

3. 用#表示阶数（只有1-6阶）
# 这是 H1
## 这是 H2
###### 这是 H6
```
1.
This is an H1
===============

2.
This is an H2
-------------

3.
# 这是 H1
## 这是 H2
###### 这是 H6
<!-- more -->

###  二 、引用

``` markdown
>这是引用
>引用名人名言之类的呀
>>也可以嵌套引用
> ## 这是一个标题。
>
> 1.   这是第一行列表项。
> 2.   这是第二行列表项。
>
> 给出一些例子代码：
>
>     return shell_exec("echo $input | $markdown_script");
```

>这是引用
>引用名人名言之类的呀
>>也可以嵌套引用
> ## 这是一个标题。
>
> 1.   这是第一行列表项。
> 2.   这是第二行列表项。
>
> 给出一些例子代码：
>
>     return shell_exec("echo $input | $markdown_script");

###  三 、列表

``` markdown
*  Red
  - asd
    - ead
  - sada
  - asd
*  Green
*  Blue


1. Bird
  1. sadaasd
          中间也可以插入其他标签
  2. sadaasd
  3. asdas
  4. asdas
    1. asdas
    2. asdas
2. McHale
3. Parish
```

*  Red
  - asd
    - ead
  - sada
  - asd
*  Green
*  Blue


1. Bird
  1. sadaasd
  ``` markdown
  中间也可以插入其他标签
  ```
  2. sadaasd
  3. asdas
  4. asdas
    1. asdas
    2. asdas
2. McHale
3. Parish

###  四 、分隔线

``` markdown
* * *

***

*****

- - -

---------------------------------------
```

* * *

***

*****

- - -

---------------------------------------

### 五 、 链接/图片

``` markdown
This is [an example](http://example.com/ "Title") inline link.

[This link](http://example.net/) has no title attribute.

[foo]: http://example.com/  "Optional Title Here"


[自定义链接] [id]

[id]: http://example.com/  "Optional Title Here"

![Alt text](/markdown-整理/img.png)

![Alt text](/markdown-整理/img.png "Optional title")

![Alt text][idpig]

[idpig]: /markdown-整理/img.pnge  "Optional title attribute"
```

This is [an example](http://example.com/ "Title") inline link.

[This link](http://example.net/) has no title attribute.

[Google]: http://google.com/


[自定义链接] [id]

[id]: http://example.com/  "Optional Title Here"

![Alt text](/markdown-整理/img.png)

![Alt text](/markdown-整理/img.png "Optional title")

![Alt text][idpig]

[idpig]: /markdown-整理/img.png  "Optional title attribute"

### 六 、 强调

``` markdown
*single asterisks*

**double asterisks**

***three asterisks***

\*this text is surrounded by literal asterisks\*

~~这样~~

```

*single asterisks*

**double asterisks**

***three asterisks***

\*this text is surrounded by literal asterisks\*

~~这样~~

### 七 、 代码

``` markdown
Use the `printf()` function.

``There is a literal backtick (`) here.``

A single backtick in a code span: `` ` ``

A backtick-delimited string in a code span: `` `foo` ``

Please don't use any `<blink>` tags.
```

Use the `printf()` function.

``There is a literal backtick (`) here.``

A single backtick in a code span: `` ` ``

A backtick-delimited string in a code span: `` `foo` ``

Please don't use any `<blink>` tags.

### 八 、 选项

``` markdown
- [x] 这是选中的复选框` ‘-’ + ‘空格’ + ‘[中间有空格]’ `

- [ ] 这是未选中的复选框` ‘-’ + ‘空格’ + ‘[中间有空格]’ `
```
- [x] 这是选中的复选框` ‘-’ + ‘空格’ + ‘[中间有空格]’ `

- [ ] 这是未选中的复选框` ‘-’ + ‘空格’ + ‘[中间有空格]’ `

### 九 、 表格

``` markdown
|    a    |       b       |      c     |
|:-------:|:------------- | ----------:|
|   居中  |     左对齐    |   右对齐   |
|=========|===============|============|
```

|    a    |       b       |      c     |
|:-------:|:------------- | ----------:|
|   居中  |     左对齐    |   右对齐   |
|=========|===============|============|

### 十 、 不兼容的

``` markdown
# 脚注 Footnote
这就是一个脚注:[^sample_footnote]

# MarkDown的标注
<!-- comment -->

# 自动生成目录 TOC
[TOC]
```
