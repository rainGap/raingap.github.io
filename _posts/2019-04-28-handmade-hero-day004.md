---
layout: default
title: HandmadeHero Day 4-Animating the Backbuffer
---

{{ page.title }}
================

## Animating the Backbuffer 让Backbuffer动起来  
![](/images/24.gif)
### Back buffer  
 
我们一般不会将图像数据直接渲染到监视器，如果监视器刷到一半的时候刚好适配器渲染了新的模型，这时产生的图像就会有撕裂的现象，为了避免这种情况，引入了后缓冲区技术。与Back buffer相对应的front buffer可以理解为当前监视器正在显示的图像数据缓冲区，而back buffer则是下一帧需要显示的图像数据缓冲区。

DirectX 使用了一个交换链技术，使用了多个缓冲区，更好地利用等待监视器刷新的这段时间。

### StretchDIBits

把内存里的位图数据，以拉伸的方式填充到设备的目标矩形。

### 位图内存分配器

根据窗口的size动态分配内存。

### 代码流程

* 创建窗体  
* 用主循环监听消息队列  
* 监听WM_SIZE，使用VirtualAlloc根据窗体size分配位图缓冲区内存  
	``` cpp
	internal void Win32ResizeDIBSection(int Width, int Height)
	{
		//每次先检查一下是否需要释放掉之前申请的虚拟内存。
		if(BitmapMemory)
		{
			VirtualFree(BitmapMemory, 0, MEM_RELEASE);
		}

		BitmapWidth = Width;
		BitmapHeight = Height;
		
		//BitmapInfo 记录了这个位图缓冲区的数据结构，在用 StretchBlt 进行图像复制的时候，需要传入BitmapInfo参数。
		BitmapInfo.bmiHeader.biSize = sizeof(BitmapInfo.bmiHeader);
		BitmapInfo.bmiHeader.biWidth = BitmapWidth;
		BitmapInfo.bmiHeader.biHeight = -BitmapHeight;
		BitmapInfo.bmiHeader.biPlanes = 1;
		BitmapInfo.bmiHeader.biBitCount = 32;
		BitmapInfo.bmiHeader.biCompression = BI_RGB;

		// StretchDIBits and BitBlt 的区别？
		// 简单说， BitBlt 直接按你指定的大小输出源dc到目标dc，而 StretchDIBits 会调整你源 DC 大小，使之适应你所指定的目标 DC 大小，再输出。
		// 也就是说， StretchDIBits 输出的图总是完整的，而且充满你指定的目标 DC 区域，而 BitBlt 则可能输出的图是不完整的，也可能无法充满目标 DC 制定区域。
		int BitmapMemorySize = (BitmapWidth*BitmapHeight)*BytesPerPixel;
		BitmapMemory = VirtualAlloc(0, BitmapMemorySize, MEM_COMMIT, PAGE_READWRITE);
	}
	```
* 准备位图数据  
	``` cpp
	/**
	通过每帧控制BlueOffset和GreenOffset两个步长，来达到动画的效果。
	想理解好这个算法具体干了什么，可以代入X=Y=0，看看(0,0)点处的像素颜色是如何随着BlueOffset和GreenOffset进行变化的。
	然后假定BlueOffset和GreenOffset都等于0，看看这时窗体的像素颜色是如何变化的。
	通过以上两步，就不难想象，最终的动画效果是怎样的。
	*/
	internal void RenderWeirdGradient(int BlueOffset, int GreenOffset)
	{
		int Width = BitmapWidth;
		int Height = BitmapHeight;

		int Pitch = Width*BytesPerPixel;
		uint8 *Row = (uint8 *)BitmapMemory;
		for(int Y = 0;
			Y < BitmapHeight;
			++Y)
		{
			uint32 *Pixel = (uint32 *)Row;
			for(int X = 0;
				X < BitmapWidth;
				++X)
			{
				uint8 Blue = (X + BlueOffset);		// 0-255
				uint8 Green = (Y + GreenOffset);	// 0-255
				
				*Pixel++ = ((Green << 8) | Blue);
			}

			Row += Pitch;
		}
	}
	```
* 监听WM_PAINT，将准备好的位图数据以拉伸方式填充到设备窗体，由于位图数据的高和宽跟设备窗体一样，也就没有拉伸一说。
* 释放设备资源

### 课堂练习

1、动画中的这个矩形长和高是多少？  
	![](/images/2019-04-29-13-12-58.png)

2、改变窗体的大小，观察一下内存占用大小？

3、改变窗体的大小，观察动画的播放速度有没有变化？为什么？怎样才可以以一个固定的速率进行播放呢？



