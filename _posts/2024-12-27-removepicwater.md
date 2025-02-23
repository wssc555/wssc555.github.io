---
title: 图片水印去除方法
tags: 去水印

---

现在有很多去除水印的工具，但基本上都需要你花钱。作为资深白嫖党，想让我花钱，那是不可能的。

于是我做了下research(search, search, research…)，我发现现在的“去水印”基本上都是一个思路：利用图像修复算法。把有水印的地方看作是图像损坏的地方，用相邻像素替换那些损坏的地方。

现在看看有什么开源工具可以用来去除图片上的水印。

**1. openCV 图像修复**

openCV中就有图像修复的方法,提供了两种算法分别基于_2004 年由 Alexandru Telea 撰写的“基于快速行进方法的图像修复技术”_ 和 *Bertalmio，Marcelo，Andrea L. Bertozzi 和 Guillermo Sapiro 在 2001 年的论文“Navier-Stokes，流体动力学和图像和视频修补”* 实现。调用cv.inpaint方法时，前者用cv.INPAINT_TELEA标记后者用cv.INPAINT_NS，openCV修复图像的方法使用起来很简单，只需要几行代码：

```python
import numpy as np
import cv2 as cv

img = cv.imread('messi_2.jpg')
mask = cv.imread('mask2.png',0)
dst = cv.inpaint(img,mask,3,cv.INPAINT_TELEA)
# dst = cv.inpaint(img,mask,3,cv.INPAINT_NS)
cv.imshow('dst',dst)
cv.waitKey(0)
cv.destroyAllWindows()
```

上面messi_2.jpg是需要修复的图片，而mask2.png是掩模图片，这张图片的作用是指定图片损坏的位置，该图片只有黑白两色，白色就是损坏的地方。掩模图片和原图的大小(即宽高)必须完全一致，否则会报错。

![img](https://wssc555.github.io/img/remove_water_1.jpg)

第一张图像是原图。第二个图像是mask(掩模)。第三个图像是第一个算法的结果，最后一个图像是第二个算法的结果。

对于轻微损坏，或者说只有很少一两个水印的图片，这种方法还是可以的。可是，对于满屏水印的图片怎么办呢？

像这样的：

![img](https://wssc555.github.io/img/remove_water_2.jpg)

**2. mask图像怎么创建**

用openCV去除上面这样图片的水印，首先要面临的问题是mask图片怎么办？

new Bing是这样回答的：

*创建掩模图像是图像修复过程中的一个重要步骤。掩模图像用于指示原始图像中需要修复的区域。以下是创建掩模图像的基本步骤：*

_1. 打开图像编辑软件：你可以使用任何图像编辑软件，如Photoshop、GIMP或者Paint.NET。

*2. 导入原始图像：将需要修复的原始图像导入到图像编辑软件中。*

*3. 创建新图层：在原始图像上创建一个新的图层，用于绘制掩模。*

*4. 选择画笔工具：使用画笔工具，在新图层上绘制需要修复的区域。通常，我们使用白色来标记这些区域。*

*5. 填充损坏区域：确保所有需要修复的区域都被白色完全覆盖。如果区域很小，可以使用放大工具来帮助精确>绘制。*

*6. 保存掩模图像：完成绘制后，只保存包含掩模的图层（通常为白色损坏区域和黑色背景）为一个新的图像文>件，格式可以是PNG或BMP，以保留二值信息。在OpenCV中使用掩模图像时，掩模应该是一个二值图像，其中白色代表需要修复的区域，黑色代表不需要修复的区域。如果你有任何疑问或需要进一步的帮助，请告诉我！*

学会了吧？然后，让我展示一下真正的技术：

![img](https://wssc555.github.io/img/remove_water_3.jpg)

呐，这就是我根据上面的图片创建的掩模图像。

```python
import cv2

def remove_watermark(image_path, watermark_path, output_path):
    # 读取原始图像和水印图像
    original_image = cv2.imread(image_path)
    watermark = cv2.imread(watermark_path, cv2.IMREAD_GRAYSCALE)
    # 去除水印
    watermark_removed = cv2.inpaint(original_image, watermark, 3, cv2.INPAINT_NS)
    # 保存去除水印后的图像
    cv2.imwrite(output_path, watermark_removed)
# 使用示例
remove_watermark("original_image.jpg", "mask.png", "output_image.jpg")
```

看一下效果吧：

![img](https://wssc555.github.io/img/remove_water_4.jpg)

这效果，真是一言难尽，你说它没去吧，它的确没有水印了;你说它去了吧，这还不如不去…

什么原因呢？难道是因为我涂鸦涂得不好？需要更精确？我想过一个鸡贼的办法：往这网站上传一个纯黑的图片，它加了水印我直接下载下来当mask，但这有点冒险…

于是，让我再展示一下真正的技术:

![img](https://wssc555.github.io/img/remove_water_5.jpg)

我把水印提取出来做mask这下够精确了吧。

再看效果：
![img](https://wssc555.github.io/img/remove_water_6.jpg)

这玩意，不能说跟原图一模一样，那也的确是没啥差别。我觉得这不是mask文件的问题，mask文件太精确不是好事，应该还是修复算法的问题。

**3.机器学习修复算法**

既然是算法不行，那，有没有更好的修复算法呢？有，就是使用卷积神经网络（CNN）来提取图像的特征，如边缘、纹理、颜色等信息然后修复。现成的也有：

<https://github.com/braindotai/Watermark-Removal-Pytorch> 这个是使用Pytorch来训练；

<https://github.com/zuruoke/watermark-removal> 这个用的是tensorflow。

遗憾的是：这两个，无论用哪一个，你都逃不掉创建mask。为什么不能通过机器学习自己识别水印创建mask呢？Watermark-Removal-Pytorch项目的README中也给出了解释：

![img](https://wssc555.github.io/img/remove_water_7.jpg)

总的来说: 做水印识别代价太大，而且效果不好。

当我想试试这两个项目的时候，又发现了另外一个项目：iopaint！这个有web界面可以直接在本地跑起来，而且可以下载训练好的模型直接用！于是我装起来试了一下：

![img](https://wssc555.github.io/img/remove_water_8.jpg)

直接涂在水印上就可以去除水印，可以涂一个去一个，也可以全部涂好一起去，一起去除要花得时间长一些。

去除的结果是这样的：
![img](https://wssc555.github.io/img/remove_water_9.jpg)
这个效果在我看来已经是非常不错了。还有一个让人惊喜的地方是，它还可以下载mask文件！

![img](https://wssc555.github.io/img/remove_water_10.jpg)

有了mask文件你就可以批量去除水印了，当然了，你所有图片水印的位置要都一样。我试了一下，某房产网站的图片水印的位置也都是一样的。如果它不升级更新，你可以用一个mask文件去除水印

```javascript
iopaint run --model=lama --device=cpu \
--image=/path/to/image_folder \
--mask=/path/to/mask_folder \
--output=output_dir
```

这个命令的解释：

–image is the folder containing input images,–mask is the folder containing corresponding mask images. When –mask is a path to a mask file, all images will be processed using this mask.

iopaint真的是个不错的项目，可是要用起来，你多少要有点编程知识。不过，iopaint有个机智的地方是：它有windows的一键安装包，但是，你得花钱买。哈哈哈。

声明：文中的水印图片来自互联网，我仅拿来做学习交流的。如果侵权了，可以联系我，我先给您磕一个，然后马上删掉。

参考：

<https://apachecn.github.io/opencv-doc-zh/#/docs/4.0.0/9.2-tutorial_py_inpainting>

<https://github.com/zuruoke/watermark-removal>

<https://github.com/braindotai/Watermark-Removal-Pytorch>

<https://github.com/Sanster/IOPaint>

<https://www.iopaint.com/>