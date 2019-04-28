---
layout: default
title: HandmadeHero Day 4-Animating the Backbuffer
---

{{ page.title }}
================

## Animating the Backbuffer 让Backbuffer动起来  
![](/images/2019-04-28-23-51-09.png)  

### Back buffer  
 
我们一般不会将图像数据直接渲染到监视器，如果监视器刷到一半的时候刚好适配器渲染了新的模型，这时产生的图像就会有撕裂的现象，为了避免这种情况，引入了后缓冲区技术。有Back buffer就会有front buffer，front buffer可以理解为当前监视器正在显示的图像数据缓冲区，而back buffer则是下一帧需要显示的图像数据缓冲区。

DirectX 使用了一个交换链技术，使用了多个缓冲区，更好地利用等待监视器刷新的这段时间。

### StretchDIBits

就是把内存里的位图数据，以拉伸的方式填充到设备的目标矩形。

### 位图内存分配器

根据窗口的size动态分配内存。

``` cpp
//需要分配大块内存的时候使用，记得VirtualFree
BitmapMemory = VirtualAlloc(0, BitmapMemorySize, MEM_COMMIT, PAGE_READWRITE);
```



