之前看到Turbo C内置的模拟时钟程序看上去很有趣。闲的无事，自己也打算做一个像模拟时钟那样会动的。所以利用了C++的`graphics.h`头文件实现一个地月系运转的模拟小程序（即月亮围绕地球转动），程序效果如下。
![Image](https://s2.loli.net/2025/02/11/Nkl3S4LUrR1E2Cb.jpg)

看上去是比较抽象，不过`graphics.h`毕竟是DOS时代的老产物了，也不指望能模拟得那么真实。
## 程序思路
这里主要使用了`graphics.h`（绘制图像）`windows.h`（动画延迟）`cmath`（旋转处理）库。
大概思路是这样的:
```
初始化图像画布->绘制一个圆形（代表地球）->
绘制另一个圆形（代表月球）->循环执行(“月球”旋转一个角度，更新旋转之后的x坐标和y坐标，按位置绘制“月球”。绘制“地球”，清空画布，继续循环。)
```
## 开始编写
### 引入必要的头文件
这个开头已经列举出来了，先引入如下头文件，辅助绘制、处理旋转和动画。
```C++
#include<graphics.h>
#include<windows.h>
#include<cmath>
```
引入cmath是为了处理“月球”的旋转，就需要用到数字处理（处理角度，位置等），`cmath`库就起作用了。
### 初始化画布
```C++
int main()
{
    int gd = DETECT, gm;
    initgraph(&gd, &gm, NULL);
}
```
### 定义初始坐标
> 注意，以下的代码都应当出现在main函数里。
```C++
int mx = 300;
int my = 250;
int angle = 0;
```
这里的`angle`变量是专门为“月球”定义的，到时候旋转处理会用上。
### 绘制最初状态
> 注意，以下的代码都应该出现在while循环内，不然绘制只执行一次，就会立即退出。

先添加一个while循环。这里使用两个不同的圆形来代替“地球”和“月球”，绘制圆形需要使用`circle(int x,int y,int radius,PIMAGE pimg = NULL)`函数。在对应的x和y坐标上绘制出大小为radius的圆形。
为了区分，同时也需要用到`setcolor()`函数来设置不同的颜色，这里提供了`RED`、`GREEN`、`BLUE`和`WHITE`四种基本的颜色，如果需要混合颜色，则用竖杠“|”分开。例如`GREEN|RED`(黄色)。顺便使用`outtextxy()`函数来标记文字“Earth”和“Moon”
```C++
setcolor(BLUE);
circle(300, 250, 50);
setcolor(GREEN);
outtextxy(280, 300, "Earth");
setcolor(BLUE | GREEN);
circle(300, 250, 30);
setcolor(GREEN | RED);
outtextxy(300 - 20,250 + 30, "Moon");
```
这段代码绘制出了一个“月球”和“地球”，只不过用了两个圆形（一个蓝色，一个浅青色）来代替，并且标注上文字。绘制出最初状态。
### 处理旋转
这个程序的核心部分就是要让月球“旋转”起来，以“地球”为中心点，绕点旋转。就很需要用到`cmath`库里的`sin()`和`cos()`运算函数。同时，新建两个变量，一个`mmx`,一个`mmy`，这两个变量循环更新旋转后的坐标。
```C++
setcolor(BLUE);
circle(300, 250, 50);
setcolor(GREEN);
outtextxy(280, 300, "Earth");
int mmx = mx + (int)(130 * cos(angle * 3.14159265 / 180));/*添加这行*/
int mmy = my + (int)(130 * sin(angle * 3.14159265 / 180));/*添加这行*/
setcolor(BLUE | GREEN);
circle(mmx, mmy, 30);/*将这里的坐标替换成mmx和mmy*/
setcolor(GREEN | RED);
outtextxy(mmx - 20, mmy + 30, "Moon");/*同样替换*/
angle--;/*角度偏移（自西向东）*/
```
### 刷新画面
完成得差不多了，只不过运行的时候“月球”旋转的时候还会留下旋转痕迹。因为没有清空画布，上一次绘制的痕迹就会保留。我们可以使用`cleardevice()`函数来清空画布，不需要任何参数，直接调用就可以。同时，控制速度，我们可以使用`windows.h`里面的`Sleep(ms)`函数来控制延迟，也可以使用库自带的`delay(ms)`。
```C++
Sleep(10);
cleardevice();
```
### 完整代码
```C++
#include<graphics.h>
#include<windows.h>
#include<cmath>
#include<stdio.h>
int main() {
	HWND hWnd = GetConsoleWindow();/*隐藏控制台窗口（可选）*/
	ShowWindow(hWnd, SW_HIDE);
	int gd = DETECT, gm;
	initgraph(&gd, &gm, NULL);
	int mx = 300;
	int my = 250;
	int angle = 0;
	while (1) {
		setcolor(BLUE);
		circle(300, 250, 50);
		setcolor(GREEN);
		outtextxy(280, 300, "Earth");
		int mmx = mx + (int)(130 * cos(angle * 3.14159265 / 180));
		int mmy = my + (int)(130 * sin(angle * 3.14159265 / 180));
		setcolor(BLUE | GREEN);
		circle(mmx, mmy, 30);
		setcolor(GREEN | RED);
		outtextxy(mmx - 20, mmy + 30, "Moon");
		angle--;
		Sleep(10);
		cleardevice();
	}
	return 0;
}
```
### 完善、修改
可以添加“月球”的x坐标和y坐标，还可以顺便标注一下程序的描述等等都可以。