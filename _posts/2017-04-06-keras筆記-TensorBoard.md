---
layout: post
title: keras筆記-TensorBoard
categories:
  - Deep Learning
date: 2017-04-06 14:00:00
tags: [Python,Deep Learning,A.I.,Data Science,Keras]
---

若以tensorflow為keras底，可以用TensorBoard查看當前數值，做Visualization。
<!--more-->
```python
tbCallBack = keras.callbacks.TensorBoard(log_dir='./Graph', histogram_freq=0, write_graph=True, write_images=True)
...
model.fit(...inputs and parameters..., callbacks=[tbCallBack])
#type in in your terminal
tensorboard --logdir path_to_current_dir/Graph
```
