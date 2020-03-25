---
layout: post
title: ML 評價指標 - Confusion Matrix、Precision Rate、Recall Rate、F-Measure
date: 2018-04-04 13:43:50
categories: Data Mining
tags: [Data Mining, Data Science, A.I.]

---
<!--more-->
|     預測\真實     | 真實為正       | 真實為負       |
| :-------------: |:-------------:| :-----:|
| 預測為正      | True Positive(TP) | False Positive(FP) |
| 預測為負      | False Negative(FN)      | True Negative(TN) |
當機器學習模型建立完成，便可測試模型的效用，混淆矩陣（Confusion Matrix）便可拿來做為紀錄用之矩陣，矩陣分為四個象限，樣本標示為正樣本，模型預測為正樣本，是為True Positive(TP)；樣本標示為負樣本，模型預測為正樣本，是為False Positive(FP)；樣本標示為正樣本，模型預測為負樣本，是為False Negative(FN)以及樣本標示為負樣本，模型預測為負樣本，是為True Negative(TN)。
<!--more-->
量測機器學習模型成效方法可以使用Precision Rate和Recall Rate兩指標，以下圖圖示表示兩指標之計算方法。
![Precision & Recall](https://upload.wikimedia.org/wikipedia/commons/2/26/Precisionrecall.svg "Precision & Recall")
<center>Reference : [Wikipedia - Precision and recall](https://en.wikipedia.org/wiki/Precision_and_recall)</center>
Precicsion Rate是關注在模型的預測結果，它表示的是在預測為正樣本的樣本中，有多少是真正的正樣本。其結果可概分為兩類，一種是真正預測正確的正樣本（TP），另一種是機器學習模型預測錯誤為正樣本的負樣本（FP）。
$$  Precision = \frac{TP}{TP + FP} $$
Recall Rate是關注在原本的樣本中，有多少正樣本的樣本被正確的預測，因此結果亦可概分為兩類，一種是真正預測正確的正樣本（TP），另一種是機器學習模型預測錯誤為負樣本的正樣本（FN）。
$$  Recall = \frac{TP}{TP + FN} $$
而將Precicsion Rate以及Recall Rate取調和平均數（Harmonic Mean）便是衡量機器學習效能的綜合指標：F-Measure，詳細的調和平均數可參考：[Wikipedia - Harmonic mean](https://en.wikipedia.org/wiki/Harmonic_mean)。
$$  \frac{2}{F-Measure}\\ =\\ \frac{1}{Precision\\ Rate} + \frac{1}{Recall\\ Rate} $$
$$  F-Measure\\ =\\ \frac{2\\ \times\\ True\\ Positive（TP）}{2\\ \times \\ True\\ Positive（TP）+ False\\ Positive（FP） + False\\ Negative（FN）} $$
$$ \Rightarrow \\ \frac{2\\ \times\\ TP}{2\\ \times \\ TP+ \\ FP + \\ FN} $$

Reference : [知乎 - 如何解释召回率与准确率？](https://www.zhihu.com/question/19645541)
