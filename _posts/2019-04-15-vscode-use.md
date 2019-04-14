---
layout: default
title: VS code 插件记录
---

{{ page.title }}
================

## 前言

VS code 真的好强大，现在我无论是写博客还是写代码都离不开VS code了。
以下记录一下自己用过且能有效提高生产效率的几个插件。

## paste image

之前使用 VS code 写 markdown 的时候有一点很不爽的就是引用图片时的操作比较繁琐。想在.md文件里面显示出图片，需经过以下步骤：

1. 将要显示的图片放到对应的文件夹下。  
2. 在.md文件里键入```![](xxx.img) ```。  
3. ctrl + shift + v 预览一下，看看路径有没有写对。     

使用 paste image 可以简化为两步：
1. 复制图片  
（这里不能复制图片文件，而是要复制到剪切板，例如截屏，例如打开图片后右键复制，这样插件才能检测到系统剪切板里是图片。）
2. ctrl + alt + v 粘贴图片  
（这一步会将剪切板里面的图片复制到工程目录，并且在光标位置插入对应的```![](xxx.img) ```

### 设置

例如我的blog的目录结构是:
* blog/_posts
* blog/images

我想把我引用的图片都放在images里，可以这样在settings.json里设置：

``` xml
    "pasteImage.path": "${projectRoot}/images",
    "pasteImage.basePath": "${projectRoot}",
    "pasteImage.forceUnixStyleSeparator": true,
    "pasteImage.prefix": "/"
```

更多用法可以参考  
https://marketplace.visualstudio.com/items?itemName=mushan.vscode-paste-image

## saveAndRun

这个插件的作用，顾名思义就是在保存文件的时候执行一次定义的命令。  
一个很基本的例子就是，每次写完 python 脚本想执行一次语法检查。这时候就可以使用saveAndRun插件了。  


1. 在 VS code 里安装saveAndRun  
2. pip install pyflakes 这个是用于python脚本语法检查的，一般会安装在路径c:/Python27/Scripts/pyflakes.exe  
3. VS code 里设置  
``` xml
"saveAndRun": {
        "commands": [
          {
            "match": "\\.py$",
            "cmd": "c:/python27/scripts/pyflakes.exe ${file}",
            "useShortcut": false,
            "silent": false
          },
        ]
      }
```

以下是示例图  
![](/images/2019-04-14-11-50-38.png)

更多用法可以参考  
https://marketplace.visualstudio.com/items?itemName=wk-j.save-and-run







