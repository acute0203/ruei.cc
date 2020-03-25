---
layout: post
title: keras筆記-ReduceLROnPlateau
categories:
  - Deep Learning
date: 2017-04-11 12:16:20
tags: [Python,Deep Learning,A.I.,Data Science,Keras]
---

<!--more-->
訓練時依條件降低學習率。
說明：監控val_loss，在該數值等待10次，若無較好表現則調降學習率，但最低若為0.0001，則不在調降。
```python
keras.callbacks.ReduceLROnPlateau(monitor='val_loss', factor=0.1, patience=10, verbose=0, mode='auto', epsilon=0.0001, cooldown=0, min_lr=0.0001)
```
