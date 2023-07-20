容器的一个核心概念:分层
每层都是不可修改的一组文件,相同的层在镜像之间可以共享
![](https://lbnote-1304107363.cos.ap-nanjing.myqcloud.com/202307201319046.png)
使用docker inspect来查看镜像的分层信息,例如
![](https://lbnote-1304107363.cos.ap-nanjing.myqcloud.com/202307201320110.png)
如果把容器比喻成"样板间",那么dockerfile就是"施工图纸"
语法如下:
- FROM 选择基础镜像,如