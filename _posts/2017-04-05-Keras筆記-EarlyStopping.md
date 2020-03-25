---
layout: post
title: Keras筆記-EarlyStopping
categories:
  - Deep Learning
date: 2017-04-05 15:43:23
tags: [Python,Deep Learning,A.I.,Data Science,Keras]
---

當目前的loss小於value時，則提前停止Model訓練。
<!--more-->
```python
class EarlyStoppingByLossVal(Callback):
    def __init__(self, monitor='val_loss', value=0.00001, verbose=0):
        super(Callback, self).__init__()
        self.monitor = monitor
        self.value = value
        self.verbose = verbose

    def on_epoch_end(self, epoch, logs={}):
        current = logs.get(self.monitor)
        if current is None:
            warnings.warn(&quot;Early stopping requires %s available!&quot; % self.monitor, RuntimeWarning)

        if current &lt; self.value:
            if self.verbose &gt; 0:
                print(&quot;Epoch %05d: early stopping THR&quot; % epoch)
            self.model.stop_training = True
```
Reference:
1.[How to tell Keras stop training based on loss value?](http://stackoverflow.com/questions/37293642/how-to-tell-keras-stop-training-based-on-loss-value#37296168)
