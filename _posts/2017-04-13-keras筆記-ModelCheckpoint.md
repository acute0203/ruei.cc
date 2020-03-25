---
layout: post
title: keras筆記-ModelCheckpoint
categories:
  - Deep Learning
date: 2017-04-13 16:14:58
tags: [Python,Deep Learning,A.I.,Data Science,Keras]
---

<!--more-->
若有進步就儲存，且保留上次最好的。
```python
filepath="weights-improvement-{epoch:02d}-{val_acc:.2f}.hdf5"
checkpoint = ModelCheckpoint(filepath, monitor='val_acc', verbose=1, save_best_only=True, mode='max')
```
若有進步就儲存，但只保留最好的。
```python
filepath="weights.best.hdf5"
checkpoint = ModelCheckpoint(filepath, monitor='val_acc', verbose=1, save_best_only=True, mode='max')
```
讀取尚未訓練完成的Model，但需要Initialize一開始模型的樣子。
```python
model.load_weights("weights.best.hdf5")
```
參考：
1.[How to Check-Point Deep Learning Models in Keras](http://machinelearningmastery.com/check-point-deep-learning-models-keras/)
2.[Usage of callbacks](https://keras.io/callbacks/)
