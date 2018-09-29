

安装python相关模块
opency  numpy  dlib

首先安装pip
https://www.cnblogs.com/mangoVic/p/6428369.html
```
yum -y install python-pip
```

安装opencv
```shell
pip install opencv-python

# 以上报错的话，需要用国内镜像源来加速，以下

pip install opencv-python -i http://pypi.douban.com/simple/ --trusted-host pypi.douban.com


```
安装dlib
```

yum -y install python-devel.x86_64 cmake

#或者 ubantu

apt-get -y install python-dev cmake

#这个耗时极长，而且内存得够，空余3G以上吧 不然会报错 c++: internal compiler error: Killed (program cc1plus)

pip install dlib

```

python相关模块
https://blog.csdn.net/zhunianguo/article/details/53155890
https://blog.csdn.net/insanity666/article/details/72235275

注意 编译boost b2 install 会耗时比较长
可以去下载相关模块的opency 的 whl 文件


人脸截取
https://blog.csdn.net/weixin_40674835/article/details/79424339



你可以用 opencv 来干什么 ？
https://www.cnblogs.com/xiaomanon/p/5499457.html

1. imgcodecs模块用于处理读取和写入图像文件(image file)。

2. 图像处理操作(Image processing operations)

3. 构建图形用户界面(Build GUI)

4. 视频分析(Video analysis)

5. 3D重建(3D reconstruction)

6. 特征提取(Feature extraction)

7. 目标检测(Object detection)

8. 机器学习(Machine learning)

9. 计算摄影(Computational photography)

10. 形状分析(Shape analysis)

11. 光流算法(Optical flow algorithms)

12. 人脸和目标识别(Face and object recognition)

13. 表面匹配(Surface matching)

14. 文本检测和识别(Text detection and recognition)





