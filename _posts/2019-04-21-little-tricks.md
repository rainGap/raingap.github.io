---
layout: default
title: 记录一下各种奇淫技巧
---

{{ page.title }}
================

## 快速插入自定义格式的时间

写过github.io的同学应该都知道，md文件的命名必须要带上日期，例如 2019-04-21-xxxx.md ，下面教一下大家如何使用win10的微软输入法，快速输入自定义的时间格式：

设置——》词库和自学习——》添加或编辑自定义短语，然后按下图设置并保存，done! 

![](/images/2019-04-21-15-12-25.png)

更多格式化变量请参考：[Custom Date and Time Format Strings](https://docs.microsoft.com/en-us/dotnet/standard/base-types/custom-date-and-time-format-strings?wt.mc_id=MVP)

### 实际使用：  
![](/images/rq.png)  
是不是很方便呢？

## 修改web页面的css样式

最近在看一个网上教程的时候，遇到一个亮瞎眼的问题。。。  
[directxtutorial](http://www.directxtutorial.com)

它的页面是这样子的：  

![](/images/2019-04-21-15-59-39.png)

可能有人喜欢，但不适合我。。。

所以我第一时间想起了用chrome devtools来修改css样式。  
ctrl+shift+I 打开devtools  

![](/images/2019-04-21-16-16-13.png)

修改css直到符合你的阅读习惯  
修改完毕：  
![](/images/2019-04-21-16-17-12.png)

舒服多了。  
但是很快会发现，每次刷新后就会变回原来的样式，每次都要修改一次，那成本就太高了。

解决方法：  
![](/images/2019-04-21-16-19-24.png)  
创建一个overides文件夹，这个本地文件夹会保留你每次修改后的文件，下次打开后就会拿本地的文件覆盖web sources的文件。  
有一个地方需要注意的是，要确保你的devtools处于打开状态，这样本地的css文件才会生效。打开的快捷键是 ctrl+shift+I

如果你觉得devtools总是占了一部分网页的位置让你很不爽，你也可以设置它的布局方式是单独窗口，这样就眼不见不烦了。  
![](/images/2019-04-21-21-15-56.png)