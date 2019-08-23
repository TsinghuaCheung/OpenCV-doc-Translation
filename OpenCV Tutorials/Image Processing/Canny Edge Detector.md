目的

你在本篇教程中将会学到：

* 使用OpenCV中的cv::Canny函数来实现肯尼边缘检测算法。

理论

肯尼边缘检测算法于1986年由约翰·F·肯尼发明。和其他已知的优化检测算法一样，肯尼检测算法主要是满足以下三方面需求：

* 低错误率：意味着对于已知边界有较好的检测效果。
* 良好的定位：边缘像素检测和真实像素值之间可以缩短误差。
* 专注性强：一个检测器检测一条边界。

具体步骤

1.过滤掉所有的噪音。高斯滤镜就是为了这一步。大小为5高斯核用以下公式表示：

![](http://latex.codecogs.com/gif.latex?K%20%3D%20%5Cdfrac%7B1%7D%7B159%7D%5Cbegin%7Bbmatrix%7D%202%20%26%204%20%26%205%20%26%204%20%26%202%20%5C%5C%204%20%26%209%20%26%2012%20%26%209%20%26%204%20%5C%5C%205%20%26%2012%20%26%2015%20%26%2012%20%26%205%20%5C%5C%204%20%26%209%20%26%2012%20%26%209%20%26%204%20%5C%5C%202%20%26%204%20%26%205%20%26%204%20%26%202%20%5Cend%7Bbmatrix%7D)

2.找到图片的强度径向改变值。对于此，我们可以使用一种类似于索贝尔算子的算法：

    a.使用一对卷积蒙层滤镜（在x和y的方向）：

    ![](http://latex.codecogs.com/gif.latex?G_%7Bx%7D%20%3D%20%5Cbegin%7Bbmatrix%7D%20-1%20%26%200%20%26%20+1%20%5C%5C%20-2%20%26%200%20%26%20+2%20%5C%5C%20-1%20%26%200%20%26%20+1%20%5Cend%7Bbmatrix%7D)

    ![](http://latex.codecogs.com/gif.download?G_%7By%7D%20%3D%20%5Cbegin%7Bbmatrix%7D%20-1%20%26%20-2%20%26%20-1%20%5C%5C%200%20%26%200%20%26%200%20%5C%5C%20+1%20%26%20+2%20%26%20+1%20%5Cend%7Bbmatrix%7D)

    b.使用以下公式用以寻找径向强度改变值和方向：

    ![](http://latex.codecogs.com/gif.download?%5Cbegin%7Barray%7D%7Bl%7D%20G%20%3D%20%5Csqrt%7B%20G_%7Bx%7D%5E%7B2%7D%20+%20G_%7By%7D%5E%7B2%7D%20%7D%20%5C%5C%20%5Ctheta%20%3D%20%5Carctan%28%5Cdfrac%7B%20G_%7By%7D%20%7D%7B%20G_%7Bx%7D%20%7D%29%20%5Cend%7Barray%7D)

3.应用非最大值抑制操作。这将会去除那些认为不是边界的像素。因此，只有比较细的线会留下来（可能的边界）留下来。

4.滞后现象: 这是最后一步。肯尼边界检测算法使用了两个阈值（高阈值和地阈值）：

    a.如果像素的径向改变值高于阈值，像素确认为边界。
    b.如果像素的径向改变值低于阈值，像素将被舍弃。
    c.如果像素的径向改变值在高阈值和低阈值之间，他只有在与高于高阈值相连接的像素连接的时候才能被认为是边界。

肯尼边界检测算法推荐高阈值和低阈值的的比为2:1和3:1。

5.更多的详细内容，你可以查看你的机器视觉书籍。