目的

我们对以下问题进行讨论：

* 什么是傅里叶变换，使用的目的是什么？
* 在OpenCV里如何使用傅里叶变换？
* 部分函数的使用方法: copyMakeBorder(), merge(), dft(), getOptimalDFTSize(), log() and normalize().

原理解释 

傅里叶变换会将图像转化为图像的sin和cos部分。也就是说，他会将图片转换成空间域转化为时频域。原理也很简单，所有的函数都可以准确地用sin和cos的无穷级数来表示。傅里叶变换就是用来实现这一过程的变换方法。数学上表示二维图形的傅里叶变换如下所示：

![](http://latex.codecogs.com/gif.latex?F(k,l)=\displaystyle\sum\limits_{i=0}^{N-1}\sum\limits_{j=0}^{N-1}f(i,j)e^{-i2\pi(\frac{ki}{N}+\frac{lj}{N})})

![](http://latex.codecogs.com/gif.latex?e^{ix}=\cos{x}+i\sin{x})

在这里，f指的是图片的空间域部分，而F指的是他的时频域部分。最后得出的转换结果是复数形式。可以通过一张实像图片或虚像图片或者是强度图和相位图。然而，在这个图像处理算法中，只有强度图包含了我们所有感兴趣的信息，信息里面主要是图像的几何结构。无论如何，如果你需要在傅里叶变换的方式下对图片进行修改，你接着需要将他们再进行一次转换，你还要保存两者的信息。

在这个例子中，我将告诉各位如何计算并显示一个傅里叶变换的强度图。在数字图像中，这些图片数据都是离散型的。这也就意味着他们的值是通过一个给定的阈值来取得。例如，在一个简单的灰阶图形中，值通常在0-255之间。因此，傅里叶变换急需转换成离散型傅里叶变换（DFT）。每当你需要研究一张图形的几何视点的时候，你就需要使用离散傅里叶变换。这里有几个简单的步骤告诉你如何实现（在本例中，I代表的是一张灰阶图像）：

将图片展开到合适的大小

DFT的性能取决于图像大小。一般情况而言，在乘以数字2、3、4的时候，计算速度会比较快。因此，为了能够获得最好的性能，最好的方法便是将图像的边缘值扩展到上述数值当中。getOptimalDFTSize返回图片的最佳值，我们还可以使用copyMakeBorder函数来扩展图像的边界（那些扩展的像素值会设为0）。

```
Mat padded;                            //expand input image to optimal size
int m = getOptimalDFTSize( I.rows );
int n = getOptimalDFTSize( I.cols ); // on the border add zero values
copyMakeBorder(I, padded, 0, m - I.rows, 0, n - I.cols, BORDER_CONSTANT, Scalar::all(0));
```

为复数和实数提供计算位置

傅里叶变换的结果是一个复数。这也就意味着每一个像素点的值在结果都对应两个值。更重要的是，时频域比它对应的空间域要大得多。因此，我们会用float变量来存储数据。那么，这也就意味着，我们将我们的输入图片转换成float类型，然后将他和其他通道进行展开，以便保存复数值：

```
Mat planes[] = {Mat_<float>(padded), Mat::zeros(padded.size(), CV_32F)};
Mat complexI;
merge(planes, 2, complexI);         // Add to the expanded another plane with zeros
```

进行离散型傅里叶变换

这是一个对自身进行的计算：

```
dft(complexI, complexI);            // this way the result may fit in the source matrix
```

将实部和虚部的数值转化为强度：

复数有实部（Re）和虚部（imaginary - Im）。DFT的结果是一个复数。DFT的强度如下：

![](http://latex.codecogs.com/gif.latex?M=\sqrt[2]{{Re(DFT(I))}^2+{Im(DFT(I))}^2})

转换成OpenCV的代码：

```
split(complexI, planes);                   // planes[0] = Re(DFT(I), planes[1] = Im(DFT(I))
magnitude(planes[0], planes[1], planes[0]);// planes[0] = magnitude
Mat magI = planes[0];
```

转换成对数形式

这是在说明，傅里叶级数的系数有一个非常大的范围，而且系数一般比较大，不能在屏幕上正常显示。我们这有一个小屏幕，所以这么大范围的数值变换是无法在屏幕上显示地。因此，较大的数值会变成白点，而较小的数值会变成黑色的点。为了能够将灰阶值可视化，我们可以将我们的线性表示转换成对数形式表示：

![](http://latex.codecogs.com/gif.latex?M_1=\log{(1+M)})

Translated to OpenCV code:用OpenCV的代码便是：

```
magI += Scalar::all(1);                    // switch to logarithmic scale
log(magI, magI);
```

裁剪和重排

首先，要记住，我们不是要先展开我们的图像。其实一开始是去掉一些新进来的数据。为了能够进行可视化分析，我们可能还需要对结果的象限进行简单处理，这样能够让原点（0,0）是在图像正中心。

```
// crop the spectrum, if it has an odd number of rows or columns
magI = magI(Rect(0, 0, magI.cols & -2, magI.rows & -2));
// rearrange the quadrants of Fourier image  so that the origin is at the image center
int cx = magI.cols/2;
int cy = magI.rows/2;
Mat q0(magI, Rect(0, 0, cx, cy));   // Top-Left - Create a ROI per quadrant
Mat q1(magI, Rect(cx, 0, cx, cy));  // Top-Right
Mat q2(magI, Rect(0, cy, cx, cy));  // Bottom-Left
Mat q3(magI, Rect(cx, cy, cx, cy)); // Bottom-Right
Mat tmp;                           // swap quadrants (Top-Left with Bottom-Right)
q0.copyTo(tmp);
q3.copyTo(q0);
tmp.copyTo(q3);
q1.copyTo(tmp);                    // swap quadrant (Top-Right with Bottom-Left)
q2.copyTo(q1);
tmp.copyTo(q2);
```

归一化

这还是为了方便可视化分析。我们现在有强度图，但是产生的值不在\[0,1\]的范围内。因此，我们需要用cv::normalize()函数对我们的数值进行归一化处理。

```
normalize(magI, magI, 0, 1, NORM_MINMAX); // Transform the matrix with float values into a
                                            // viewable image form (float between values 0 and 1).
```

运行结果

有一个想法便是，我们需要知道图像的几何方向。例如，我们想知道图像中的文字是水平的或者是有偏移的。如果你看着某些文字，你会发现，文字可能既有水平排列的又有垂直排列的。在傅里叶变换中，我们就会看到图像中文字的这两种排列方式。那么就让我们看看一张有水平文字的图片，和一张有旋转文字的图片。

在水平文字图片的情况下

![](https://docs.opencv.org/4.1.0/result_normal.jpg)

在有旋转文字的情况下

![](https://docs.opencv.org/4.1.0/result_rotated.jpg)

你可以看到时频域中最明显的部分（在强度途中则是最亮的点）就正好和图像中的图形几何旋转角度一致。通过这个方式，我们就能计算出偏移量，并通过图像旋转来修正图像中物体的位置。
