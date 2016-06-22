---
title: 《CSS揭秘》读书笔记
date: 2016-06-22 09:59:46
tags: [CSS, 笔记, 前端]
---

本书是一本注重实践的CSS书籍，作者为我们揭示了 47 个鲜为人知的 CSS 技巧，主要内容包括背景与边框、形状、 视觉效果、字体排印、用户体验、结构与布局、过渡与动画等。我将记录阅读过程中的点点滴滴。

![CSS揭秘](http://img3x0.ddimg.cn/40/30/23953090-1_w_23.jpg)

<!--more-->

----------
## *(未完待续......)* ##

	第一章：引言 
#CSS3 渐变（Gradients）

- 线性渐变（Linear Gradients）

使用角度：`background: linear-gradient(angle, color-stop1, color-stop2);`

角度是指水平线和渐变线之间的角度，逆时针方向计算。换句话说，0deg 将创建一个从下到上的渐变，90deg 将创建一个从左到右的渐变。

![CSS线性渐变角度](http://www.runoob.com/wp-content/uploads/2014/07/7B0CC41A-86DC-4E1B-8A69-A410E6764B91.jpg)

但是**很多浏览器(Chrome,Safari,fiefox等)的使用了旧的标准，即 0deg 将创建一个从左到右的渐变，90deg 将创建一个从下到上的渐变**。换算公式 `90 - x = y `其中 x 为标准角度，y为非标准角度。

带有指定的角度的线性渐变：

	#grad {
	  background: -webkit-linear-gradient(0deg, red, blue); /* Safari 5.1 - 6.0 */
	  background: -o-linear-gradient(0deg, red, blue); /* Opera 11.1 - 12.0 */
	  background: -moz-linear-gradient(0deg, red, blue); /* Firefox 3.6 - 15 */
	  background: linear-gradient(90deg, red, blue); /* 标准的语法 */
	}

 ***注意为了支持较早的浏览器，这里所使用的浏览器前缀***

----------

# 回退机制 #
	
- 可以使用Modernizr进行特性检测
- 使用@supports特性
- js检测：if ('xxxx特性' in element.style)

------------------------------

# Padding属性 #
**可以有一到四个值。**

    padding:25px 50px 75px 100px;
	上填充为25px
	右填充为50px
	下填充为75px
	左填充为100px

    padding:25px 50px 75px;
	上填充为25px
	左右填充为50px
	下填充为75px

    padding:25px 50px;
	上下填充为25px
	左右填充为50px

    padding:25px;
	有的填充都是25px

----------

# CSS编码技巧 #

**当某些值相互依赖时，应该把他们的相互关系用代码表达出来**

e.g.`line-height属性`

***属性值***

normal----------默认。设置合理的行间距。

number---------**设置数字，此数字会与当前的字体尺寸相乘来设置行间距**。

length----------	设置固定的行间距。

%----------------	**基于当前字体尺寸的百分比行间距**。

inherit-----------	规定应该从父元素继承 line-height 属性的值。

所以在设置字体大小和行高时，不要把他们写成绝对值，不利于修改。

**字号最好用百分比或者em/rem单位，行高用相对值。其他有关尺寸的属性也最好使用相对大小或单位**

最好写成下面这样：这样只要我们改变父级的字号，子级的相应属性就会修改

	font-size: 125%;/*假设父级的字号是16px*/
	line-height: 1.5;/*表示行高是字号的1.5倍*/

----------

**代码易维护 vs. 代码量少**

假如要给一个元素设置边框，但左侧不加边框，可以写成：

	border-width: 10px 10px 10px 0;

但如果拆开写成下面这样，可能更好：

	border-width: 10px;
	border-left-width: 0;

----------

**currentColor**

这是一个特殊的颜色关键字，他没有绑定固定的颜色值，而是被解析为color属性的值.

e.g.假如想让分割线的颜色和文本颜色一致，可以写成：

	hr{
		height: .5em;
		background: currentColor;
	}

----------
**建议：**

1. 不妨在媒体查询中使用em单位取代像素单位，这能使文本缩放在必要时触发布局变化。
2. 适用百分比长度取代固定长度。
3. 当需要在较大分辨率下得到固定宽度时，使用max-width而不是width，因为它可以适应较小的分辨率。
4. 为替换元素（比如img,object,vedio,iframe..）设置max-width:100%
5. 假如背景图片需要完整的铺满整个容器->background-size:cover;
6. 在使用多列文本时，指定column-width而不是column-count，就可以自动在较小屏幕上自多显示为单列布局。  

----------
**合理使用简写：**

	background: url(1.png) no-repeat top right / 2em 2em,
				url(2.png) no-repeat bottom right / 2em 2em,
				url(3.png) no-repeat top left / 2em 2em;
可以写成：

	background: url(1.png) top right,
				url(2.png) bottom right,
				url(3.png) top left;
	background-size: 2em 2em;
	background-repeat: no-repeat;

----------
    第二章：背景与边框

**半透明边框：**

透明色：`rgba/HSLA`

    HSLA的颜色值是一个带有alpha通道的HSL颜色值的延伸 - 指定对象的透明度。
    指定HSLA颜色值：HSLA（色调，饱和度，亮度，α），α是Alpha参数定义的不透明度。 Alpha参数是一个介于0.0（完全透明）和1.0（完全不透明）之间的参数。
	e.g. 
	p {
		 background-color:hsla(120,65%,75%,0.3);
	 }

假设我们要给一个容器设置白色背景和一道半透明白色边框，最开始我们可能这样写：
	
	border:10px solid hsla(0, 0%, 100%, .5);
	background: white;
但这样容器的白色背景会透过半透明边框，导致半透明效果被白色覆盖。

解决方案：
	
加一条：
	
	background-clip: padding-box;
background-clip属性指定背景绘制区域。

- border-box	默认值。背景绘制在边框方框内（剪切成边框方框）。
- padding-box	背景绘制在衬距方框内（剪切成衬距方框）。
- content-box	背景绘制在内容方框内（剪切成内容方框）。

----------

**多重边框：**

box-shadow方案：

*box-shadow：h-shadow v-shadow blur spread color inset;*

	h-shadow	必需的。水平阴影的位置。允许负值
	
	v-shadow	必需的。垂直阴影的位置。允许负值
	
	blur	可选。模糊距离
	
	spread	可选。阴影的大小
	
	color	可选。阴影的颜色。在CSS颜色值寻找颜色值的完整列表
	
	inset	可选。从外层的阴影（开始时）改变阴影内侧阴影

**如果将h-shadow，v-shadow，blur均设为0，spread取正值，得到的就是一道实线边框。利用这一特点，再加上box-shadow支持逗号分隔语法，可以创建任意数量的边框**

e.g. 

	box-shadow: 0 0 0 5px red,
				0 0 0 10px blue,
				0 0 0 15px green; 

**需要注意**：

1. box-shadow是层层叠加的，第一层设置位于最顶层。以此类推。
2. 投影造出的边框和真实边框不完全一致，它不会影响布局；
3. 投影造出的边框处于元素外圈，不响应鼠标事件；若需要响应，可以给box-shadow属性加上inset关键字，会使投影出现在元素内圈。
4. box-shadow只能模拟实线边框。

outline方案：

	outline（轮廓）是绘制于元素周围的一条线，位于边框边缘的外围，可起到突出元素的作用。
	如果只需要两层边框，可以先用border设置一层常规边框，再用outline描边属性产生外层边框。
	可以设置的属性分别是（按顺序）：outline-color, outline-style, outline-width
	如果不设置其中的某个值，也不会出问题，比如 outline:solid #ff0000; 也是允许的。

e.g.

	border: 10px solid #red;
	outline: 5px solid #blue;

**需要注意：**

1. 可以通过outline-offset控制描边和元素边缘的间距，可以接受负值；
2. 只适用于双层边框；

----------

**背景定位**













