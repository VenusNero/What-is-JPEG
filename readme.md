# 求是潮二面 What-is-JPEG


## 编译命令（通用）


支持Windows、Linux

```
g++ main.cpp -O2 -w
```
## 如何运行

```
main input.bin output.jpeg X Y
```
demo.bin 表示输入文件名，每个像素点以RGB顺序，各1字节，连续存放为一个3字节的段，然后按照从左到右，从上到下顺序排列每个像素点

demo.jpeg 表示输出文件名

X表示纵向像素数。
Y表示横向像素数。
（X，Y须为8的倍数）

## 成品效果

![image](https://raw.githubusercontent.com/dyxg/What-is-JPEG/main/demo.jpeg)


## 主要算法流程

### 构建结构体

一个picture结构体，在构建过程中完成读入。

### 图层处理

分别对Y,Cr,Cb三个图层进行以下处理

- 计算图层值
- DCT
- 量化
- ZigZag
- DC差分
- 游程编码

完成后将数据存储入一对vector中，分别是DC和AC分量，

DC存储形式为两个int，第一个int使用8bit，为值的编码长度，后一个int为值

AC存储形式为两个int，第一个int使用8bit，前4bit为连续0的个数，后4bit为值的编码长度，后一个int为值。

### 哈夫曼编码

然后进行哈夫曼编码，一共生成4个哈夫曼表，YDC,YAC,CDC,CAC：

- 统计出现频次

- 构建哈夫曼树

- 生成哈夫曼表

返回值为一对vector，

第一个vector为各长度的编码数

后一个vector为按照长度大小排列的各哈夫曼值对应的原值

### 输出文件

按照JPEG文件头结构输出，详见[深入浅出JPEG|极客笔记](https://deepinout.com/camera-terms/easy-to-understand-jpeg.html)



### 技术特点

完全使用C++实现，效率较高。在i9-12900H平台上压缩给定图片仅需0.05s

使用vector部分代替数组，提供优越的可扩展性。

针对数据特点进行哈夫曼编码。

各函数的参数与接口通用性较强。





## Todo ~~咕咕咕~~

- 8bit以上色深支持
- 任意大小图片支持
- 哈夫曼编码优化~~这个版本有点小锅，效果似乎不太行~~
- 更快的DCT

## 总结与反思

1. 虽然调代码很痛苦，但是还是很有意思的。通过自己实现搞明白了jpeg的压缩格式与原理还是很有趣的（不过信号处理部分还没有深入研究，数据处理部分完全搞明白了）
2. 最主要是 [深入浅出JPEG|极客笔记](https://deepinout.com/camera-terms/easy-to-understand-jpeg.html) [跟我寫 JPEG 解碼器（四）讀取壓縮圖像數據](https://github.com/MROS/jpeg_tutorial/blob/master/doc/%E8%B7%9F%E6%88%91%E5%AF%ABjpeg%E8%A7%A3%E7%A2%BC%E5%99%A8%EF%BC%88%E5%9B%9B%EF%BC%89%E8%AE%80%E5%8F%96%E5%A3%93%E7%B8%AE%E5%9C%96%E5%83%8F%E6%95%B8%E6%93%9A.md) ，还有一系列c++语法细节问题博客，主要是通过Google搜索+GitHub站内搜索解决一些资料问题。
3. - 困难1：文件头具体格式不明确，特别是SOS扫描行之后的部分在中午文档中描述模糊
   - 解决1：转变思路转为搜索解码器的文档。
   - 困难2：代码调试，原图需要完成256*256的图像转换，原图输出大小达19kb，不能通过观察输出检查结果。 
   - 解决2.1：观察代码，对比文档。
   - 解决2.2：通过缩小规模，把size改成8检查单个块结果，改成16检查多个块结果。把颜色改成全红检查输出编码结果。
   - 解决2.3：输出部分中间结果并编写cheker检查
4. 调代码看了无数遍，近期看懂肯定没问题。每个部分的算法都比较标准化，一年以后的话配合注释以及JPEG格式的文档，应该可以看懂每个部分。
5. 感觉题目难度较大，对没有开发基础的有较大困难（对代码能力不咋行的ACMer也有较大困难），主要难度在于读懂文档和代码调试上，代码调试花了很多时间，感觉如果可以提供一个带JPEG格式检查的解码器会让问题简单很多。
6. ~~明年可以出解码器，一道题用两次，赢！~~

