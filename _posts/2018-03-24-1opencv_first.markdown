---
layout: post
title: "人生若只如初见——opencv"
subtitle: "opencv a first sight——通道"
date: 2018-03-24 17:10:00 +0800
author: "Roach"
catalog: true
tags:
    - opencv
---

> The best preparation for tomorrow is doing your best today. -- H. Jackson Brown Jr.

---

## 函数

#### imread(filename, mode)

对于第二个参数：

+ `cv2.IMREAD_COLOR : Loads a color image. Any transparency of image will be neglected. It is the default flag.`
+ `cv2.IMREAD_GRAYSCALE : Loads image in grayscale mode.`
+ `cv2.IMREAD_UNCHANGED : Loads image as such including alpha channel.`

这分别代表读取BGR（opencv的默认图像表示顺序）、读取为灰度图像、读取四通道图像。四通道则包括了`alpha`通道——透明度，而灰度图像和BGR的转换关系则是： $\mathbf{gray} = 0.1140 \times \mathbf{B} + 0.5870 \times \mathbf{G} + 0.2989 \times \mathbf{R}$ ，向下取整。

> 在计算机图形学中，一个RGB颜色模型的真彩图形，用由红、绿、蓝三个色彩信息通道合成的，每个通道用了8位色彩深度，共计24位，包含了所有彩色信息。为实现图形的透明效果，采取在图形文件的处理与存储中附加上另一个8位信息的方法，这个附加的代表图形中各个素点透明度的通道信息就被叫做Alpha通道。

Alpha通道使用8位二进制数，就可以表示256级灰度，即256级的透明度。白色（值为255）的Alpha像素用以定义不透明的彩色像素，而黑色（值为0）的Alpha通道像素用以定义透明像素，介于黑白之间的灰度（值为30-255）的Alpha像素用以定义不同程度的半透明像素。因而通过一个32位总线的图形卡来显示带Alpha通道的图形，就可能呈现出透明或半透明的视觉效果。

一个透明或半透明图形的数学模型应当如下：

为了便于下面的分析，设Alpha值[0，255]区间映射为[0，1]区间相对应的值表示，即Alpha值为0—1之间的数值。则图形文件中各个像素点可表示为：

Graphx（Redx，Greenx，Bulex，Alphax）

屏幕上相应像素点的显示值就转换为：

Dispx（Redx*Alphax，Greenx*Alphax，Bluex*Alphax）

在进行两个图片合并时可以讲三原色的像素值乘以透明度对应相加得到新的三原色的像素值（具体方法见下方Refs链接）。

---

#### imwrite

`imwrite`则时根据文件名的后缀和要保存的图片像素点的维度，自动判断存储的通道。

下面是《深度学习和计算机视觉》中`imread`和`imwrite`的示例：

```python
import cv2
color_img = cv2.imread('img/faraway.jpg')
print(color_img.shape)

gray_img = cv2.imread('img/faraway.jpg', cv2.IMREAD_GRAYSCALE)
print(gray_img.shape)
print(type(gray_img))

cv2.imwrite('img/gray_faraway.jpg', gray_img)
reload_grayscale = cv2.imread('img/gray_faraway.jpg')
print(reload_grayscale.shape)

cv2.imwrite('img/faraway_imwrite.jpg', color_img, (cv2.IMWRITE_JPEG_QUALITY, 80))
cv2.imwrite('img/faraway_imwrite.png', color_img, (cv2.IMWRITE_PNG_COMPRESSION, 5))
```

#### resize,copyMakeBorder

+ `reseize`函数对图像的大小进行缩放，既可以按像素值，也可以按比例进行缩放
+ `copyMakeBorder`则是对图像在四个方向进行填充，可以设置填充的像素多少和填充的像素值
    + BORDER_REFLECT：比如在下方填充50行像素点，这种方式下填充的结果是，填充的50行像素与原始图片最下方的50行像素关于原始的底边对称。忽略制定的像素值。
    + BORDER_CONSTANG：这种则是均填充为制定的像素值

```python
import cv2

# 读取一张四川大录古藏寨的照片
img = cv2.imread('img/faraway.jpg')

# 缩放成200x200的方形图像
img_200x200 = cv2.resize(img, (200, 200))

# 不直接指定缩放后大小，通过fx和fy指定缩放比例，0.5则长宽都为原来一半
# 等效于img_200x300 = cv2.resize(img, (300, 200))，注意指定大小的格式是(宽度,高度)
# 插值方法默认是cv2.INTER_LINEAR，这里指定为最近邻插值
img_200x300 = cv2.resize(img, (0, 0), fx=0.5, fy=0.5, 
                              interpolation=cv2.INTER_NEAREST)

# 在上张图片的基础上，上下各贴50像素的黑边，生成400x700的图像
img_400x700 = cv2.copyMakeBorder(img, 50, 50, 0, 0, 
                                       cv2.BORDER_REFLECT, 
                                       value=(0, 0, 0))

# 对照片中树的部分进行剪裁
patch_human = img[250:310, 102:192]

cv2.imwrite('img/cropped_human.jpg', patch_human)
cv2.imwrite('img/resized_200x200.jpg', img_200x200)
cv2.imwrite('img/resized_200x300.jpg', img_200x300)
cv2.imwrite('img/bordered_400x700.jpg', img_400x700)
```

---

#### cvtColor

如果读入的img是BGR的想要转换到HSV色彩空间，则需要：

`img_hsv = cv2.cvtColor(img, cv2.COLOR_BGR3HSV)`

**色彩空间**

HSV色彩空间，色调（Hue）、饱和度（Saturation）、明度（Value/Brightness），所以又称HSB。图像装换到HSV之后，同样可以对三个通道的改变来修改图片。比如：

+ H空间中，绿色的值比黄色的值高，如果增加H空间的像素值，黄色的绿叶就会变绿。
+ 减小饱和度会损失图像的鲜艳度
+ 减小明度图像就变暗

还有一种和HSV相似的色彩空间：HSL（Lightness，亮度）。

> 在原理和表现上，HSL 和 HSB 中的 H（色相） 完全一致，但二者的 S（饱和度）不一样， L 和 B （明度 ）也不一样：
> + HSB 中的 S 控制纯色中混入白色的量，值越大，白色越少，颜色越纯；
> + HSB 中的 B 控制纯色中混入黑色的量，值越大，黑色越少，明度越高
> + HSL 中的 S 和黑白没有关系，饱和度不控制颜色中混入黑白的多寡；
> + HSL 中的 L 控制纯色中的混入的黑白两种颜色。

---

## Gamma变换

> 人眼对外界光源的感光值与输入光强不是呈线性关系的，而是呈指数型关系的。在低照度下，人眼更容易分辨出亮度的变化，随着照度的增加，人眼不易分辨出亮度的变化。而摄像机感光与输入光强呈线性关系。为方便人眼辨识图像，需要将摄像机采集的图像进行gamma校正。
> 为方便人眼辨识图像，需要将摄像机采集的图像进行gamma校正。

通过非线性Gamma变换，来改变图像的原始像素值，`gamma < 1`像素值向255靠拢。`gamma > 1`像素值向0靠拢。

具体实现只需要求得0~255这256个数变换之后的值，让后将每一个原始像素值映射成新的值即可。

---

注，此篇博文主要是看《深度学习与计算机视觉》中不会的OpenCV内容的学习笔记。

---

## References

1. [一个也许很傻的问题,在图像处理中alpha到底是什么？](http://www.cnblogs.com/suogasus/p/5311264.html)
2. [色彩空间中的HSL、HSV、HSB有什么区别？ - Forrest的回答 - 知乎](https://www.zhihu.com/question/22077462/answer/280114578)
3. [gamma原理及快速实现算法（C/C++)](https://blog.csdn.net/lxy201700/article/details/24929013)