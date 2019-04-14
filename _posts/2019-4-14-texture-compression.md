---
layout: default
title: 贴图格式研究
---

{{ page.title }}
================

## 参考文章
先记录一下最近看的关于DXT压缩格式的文章。

* 一篇国外的高引论文    
    [Texture Compression using Low-Frequency Signal Modulation](/attachments/1.pdf)  
    对现有的图像压缩算法优缺点作了详述，最终提出了一种新的压缩策略。英文，看得有点吃力。
    记录几个专业术语：
    1. 矢量量化(vector quantization)  
    将若干个标量数据组构成一个矢量，然后在矢量空间给以整体量化，从而压缩了数据而不损失多少信息。我的理解是——给两个点求直线方程、霍夫变换，其实都属于矢量量化吧。  
    2. [离散余弦变换( discrete cosine transform DCT)](https://blog.csdn.net/jubincn/article/details/6882179)  
    首先人眼对低频敏感对高频不敏感，由于离散余弦变换具有很强的"能量集中"特性：大多数的自然信号（包括声音和图像）的能量都集中在离散余弦变换后的低频部分，所以可以省去离散余弦变换后矩阵里的高频部分来对图像进行压缩，这是JPEG的主要原理。
    3. 评估一种压缩算法是否有效的四个因素  
        * 解码速度
        * 随机访问能力
        * 压缩比和视觉质量平衡
        （这里提到3D场景里，整个场景的渲染效果比单张贴图精细度更重要，所以他们采用的有损压缩方式可以被容忍）
        * 编码速度

* 云风的blog  DXT 图片压缩  
    [DXT 图片压缩](https://blog.codingnow.com/2007/05/dxt.html)  
    从实现DXT1解码出发，详细介绍了DXT压缩格式，看完对各种情况下的压缩策略有了一定的认识。

* 还有一个在网上搜索到的  
    [DXT纹理压缩](https://blog.csdn.net/lhc717/article/details/6802951)
    写得更加基础。


