---
layout: post
title: Deep Learning Model常用名詞解釋
categories:
  - Deep Learning
date: 2017-03-13 12:18:03
tags: [Deep Learning,Data Science,A.I.]
---

epoch:
指Model看完所有的Training data set。
<!--more-->
batch:
指所有的Training Data set。

mini-batch:
將Training data set分成一小個一小個的Batch做Training。

iteration:
當跑完一個Batch時稱做一個iteration。

實例:
若Training data set有2000個data，每個mini-batch設500，則完成1個epoch總共會跑4個iteration。

Batch Normalization:
一個優化Training data的步驟，將輸入的Batch每個Feature維度做正規化，如平移到平均數為零，及標準差為1的常態分佈，又有點像是在每個輸入Hidden Layer前再加一層，好處是防止梯度消失和梯度爆炸，也能防止該batch沒學到東西。

CNN:

Stride:
Filter一次跨的步數。

Padding:
Zero Padding指圖片周圍塞0，防止重要資訊的遺失。

Pooling:
Pooling分為MaxPooling和average Pooling，MaxPooling只輸出該Pool內最高的值，而Average Pooling則是將Pool內的值取平均。

參考:
[Epoch vs iteration when training neural networks](http://stackoverflow.com/questions/4752626/epoch-vs-iteration-when-training-neural-networks)
[深度学习中 Batch Normalization为什么效果好？](https://www.zhihu.com/question/38102762)
[A Beginner's Guide To Understanding Convolutional Neural Networks Part 2](https://adeshpande3.github.io/A-Beginner)

延伸閱讀:
[Must Know Tips/Tricks in Deep Neural Networks (by Xiu-Shen Wei)](http://lamda.nju.edu.cn/weixs/project/CNNTricks/CNNTricks.html)
