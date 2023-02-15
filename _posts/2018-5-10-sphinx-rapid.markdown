---
layout: post
title: Sphinx使用指北
category: python
---
## 一.安装与配置
题记：曾经我认为只要会latex和markdown，就可以应付不同场合的编写文档需求。也源于此，开始就主观上排斥sphinx。今天终于静下心来，尝试接受sphinx。果然，它没有让我失望。

reStructuredText是一种标记语言，类似于我们很熟悉的markdown，但是比markdown支持更复杂一点的格式。此处不详细比较它们，只是介绍一下reStructureText的使用。以我之一知半解，写下这个流程式的参考，希望能给你带来些许帮助，也希望自己更深入学习。

最后更新日期：2018年5月10日。

### 准备工作
本地系统环境：macOS High Sierra 10.13.4，python3.6.4。

系统中需要安装texlive，python中需要安装sphinx库。使用macOS或者ubutnu系统都可以。

## 二.创建项目文件
### 建立项目
在终端执行下面指令：（mysite，pname等根据自己情况修改）
> sphinx-quickstart mysite  

后面会回答很多个问题，Project name输入pname，Author name输入aname，Project version输入0.1，语言选zh\_CN，有关mathjax选项选择y，其余都直接回车用默认值。再进入mysite目录。
### 修改配置文件conf.py
在conf.py文件中添加xelatex支持。
> latex\_engine = 'xelatex'  
> latex\_elements = {  
> 'preamble':r'''\usepackage{xeCJK}''',  
> }  

添加中文搜索支持
> html\_search\_language = 'zh'

### 修改index.rst文件
在文件:caption: Contents:下面一行，添加:numbered:，添加空行，添加新的行example。
### 新建example.rst文件
> 中文标题测试  
> \==========   
> Subject Subtitle  
> \----------------   
> Subtitles are set with '-' and are required to have the same length 
of the subtitle itself, just like titles.  
> 

### 主题样式
默认的主题是alabaster，可以换成另一个很常见的样式sphinx\_rtd\_theme。修改conf.py，添加import sphinx\_rtd\_theme，再修改html\_theme='sphinx\_rtd\_theme'。

## 三.输出html和pdf
在终端分别输入：
> make html  
> make latexpdf  
> 

希望没有错误提示，在_build文件夹下可以找到html和latex文件夹，里面有对应的文件。

## 四.更进一步
至此我们已经完整的走完了使用sphinx编写文档的流程，剩下的工作就是查看手册，根据自己的需要完善文档了。下面列举一些常见的示例。
### 文字样式
* 用途：加粗，写法：\*\*加粗\*\*，效果：**加粗**。
* 用途：斜体，写法：\*斜体\*，效果：*斜体*。
* 代码， \`\`coding`` ，注意，前后留空格。
* 代码块，::，下一行缩进，接着所有代码缩进。注意，::前面不能有空行。

### 插图
示例代码：
> .. image:: 404.png  
>    :align: center  
>    :width: 236px  

效果：
![404](../images/404.jpg)
（引用于GitHub中jekyll-now项目的404图片。）
### 表格
示例代码：
> +------------+------------+-----------+  
> | Header 1   | Header 2   | Header 3  |  
> +============+============+===========+  
> | body row 1 | column 2   | column 3  |  
> +------------+------------+-----------+

效果：
省略吧，这也是markdown让人崩溃的一点。
### 数学公式
使用mathjax，
> .. math::  
>    (a + b)^2 = a^2 + 2ab + b^2  
>    (a - b)^2 = a^2 - 2ab + b^2  

嗯，上面是单独成行的公式。如果是行内的，插入:math: ``就可以啦。命令行make html重新生成网页。DONE！

后记之题外话：我的文章都尽量避免插图，一者是我偷懒，再者更重要的是希望文档保存及修改方便，如果踩过坑你应该能理解我，单文件纯文本是多么无敌。