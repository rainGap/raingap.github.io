---
layout: default
title: HandmadeHero Day 5 - Windows Graphics Review
---

{{ page.title }}
================

## Windows Graphics Review 

### 避免使用不必要的指针形参 （这是错的）

``` cpp
uint8 * A = somewhere in memory
uint8 * B = somewhere in memory
int Y = *B;
*A = 5;
int X = *B;
//明显 Y == X, but...
```  
那什么是不必要的指针？

### 循环体里声明变量和在循环体外声明变量有什么区别？

根据c++标准，循环体内声明的变量，作用域只在循环体内。  
vs可以通过关闭/Zc:forScope-编译选项（默认开）允许在循环体外访问声明的变量。  
``` cpp
// zc_forScope.cpp
// compile by using: cl /Zc:forScope- /Za zc_forScope.cpp
// C2065, D9035 expected
int main() {
    // Compile by using cl /Zc:forScope- zc_forScope.cpp
    // to compile this non-standard code as-is.
    // Uncomment the following line to resolve C2065 for /Za.
    // int i;
    for (int i = 0; i < 1; i++)
        ;
    i = 20;   // i has already gone out of scope under /Za
}
```

### 使用固定大小的backbuffer  
在day4视频里，程序根据窗体大小动态分配内存，而在今天的视频，改用一个固定大小（1280 \* 720）的buffer来实现，buffer内存仅在程序初始化的时候分配一次。  
为什么呢？仅仅是为了减少分配内存的开销吗？

### 深挖栈大小

### 栈溢出成就达成！


### pitch的解释


