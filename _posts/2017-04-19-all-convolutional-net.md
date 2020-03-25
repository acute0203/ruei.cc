---
layout: post
title: All Convolutional Net
categories:
  - Deep Learning
date: 2017-04-19 13:43:53
tags: [Data Science,Python,Keras,A.I.,Deep Learning]
---

An experiment tests with all convolution layers network.This network can simply achieve 99.4% in MNIST, 91% in CIFAR-10 & 65% in CIFAR-100.
The Activation function is ELU Function, with some batch normalization layers and dropout layers.
<!--more-->
Please check the code below:
```python
from __future__ import print_function
import keras
from keras.datasets import cifar100
from keras.preprocessing.image import ImageDataGenerator
from keras.models import Sequential
from keras.layers import Dense, Dropout, Activation, Flatten
from keras.layers import Conv2D, MaxPooling2D
from keras.optimizers import SGD
from keras.callbacks import Callback
# import BatchNormalization
from keras.layers.normalization import BatchNormalization
from keras.callbacks import TensorBoard,ReduceLROnPlateau,EarlyStopping,ModelCheckpoint

class EarlyStoppingByLossVal(Callback):
    def __init__(self, monitor='loss', value=0.01, verbose=1):
        super(Callback, self).__init__()
        self.monitor = monitor
        self.value = value
        self.verbose = verbose

    def on_epoch_end(self, epoch, logs={}):
        current = logs.get(self.monitor)
        if current is None:
            print(&quot;Early stopping requires %s available!&quot; % self.monitor)
            exit()

        if current &lt; self.value:
            if self.verbose &gt; 0:
                print(&quot;Epoch %05d: early stopping THR&quot; % epoch)
            self.model.stop_training = True

batch_size = 128
num_classes = 100
epochs = 140000
data_augmentation = True

# The data, shuffled and split between train and test sets:
(x_train, y_train), (x_test, y_test) = cifar100.load_data()
print('x_train shape:', x_train.shape)
print(x_train.shape[0], 'train samples')
print(x_test.shape[0], 'test samples')

# Convert class vectors to binary class matrices.
y_train = keras.utils.to_categorical(y_train, num_classes)
y_test = keras.utils.to_categorical(y_test, num_classes)
def create_model(modelPath=None):
    model = Sequential()

    model.add(Conv2D(32, (3, 3), padding='same',input_shape=x_train.shape[1:]))
    model.add(keras.layers.advanced_activations.ELU(alpha=1.0))
    model.add(Conv2D(32, (3, 3), padding='same'))
    model.add(keras.layers.advanced_activations.ELU(alpha=1.0))
    model.add(BatchNormalization())
    model.add(MaxPooling2D(pool_size=(2, 2),strides=(2, 2)))
    model.add(Dropout(0.3))

    model.add(Conv2D(64, (3, 3), padding='same'))
    model.add(keras.layers.advanced_activations.ELU(alpha=1.0))
    model.add(Conv2D(64, (3, 3), padding='same'))
    model.add(keras.layers.advanced_activations.ELU(alpha=1.0))
    model.add(BatchNormalization())
    model.add(MaxPooling2D(pool_size=(2, 2),strides=(2,2)))
    model.add(Dropout(0.4))

    model.add(Conv2D(128, (3, 3), padding='same'))
    model.add(keras.layers.advanced_activations.ELU(alpha=1.0))
    model.add(Conv2D(128, (3, 3), padding='same'))
    model.add(keras.layers.advanced_activations.ELU(alpha=1.0))
    model.add(Conv2D(128, (3, 3), padding='same'))
    model.add(keras.layers.advanced_activations.ELU(alpha=1.0))
    model.add(BatchNormalization())
    model.add(MaxPooling2D(pool_size=(2, 2),strides=(2,2)))
    model.add(Dropout(0.5))

    model.add(Conv2D(256, (3, 3), padding='same'))
    model.add(keras.layers.advanced_activations.ELU(alpha=1.0))
    model.add(Conv2D(256, (3, 3), padding='same'))
    model.add(keras.layers.advanced_activations.ELU(alpha=1.0))
    model.add(Conv2D(256, (3, 3), padding='same'))
    model.add(keras.layers.advanced_activations.ELU(alpha=1.0))
    model.add(BatchNormalization())
    model.add(MaxPooling2D(pool_size=(2, 2),strides=(2,2)))
    model.add(Dropout(0.5))

    model.add(Conv2D(512, (3, 3), padding='same'))
    model.add(keras.layers.advanced_activations.ELU(alpha=1.0))
    model.add(Conv2D(512, (3, 3), padding='same'))
    model.add(keras.layers.advanced_activations.ELU(alpha=1.0))
    model.add(Conv2D(512, (3, 3), padding='same'))
    model.add(keras.layers.advanced_activations.ELU(alpha=1.0))
    model.add(BatchNormalization())
    model.add(MaxPooling2D(pool_size=(2, 2),strides=(2,2)))
    model.add(Dropout(0.5))

    model.add(Conv2D(1024, (1, 1), padding='same'))
    model.add(keras.layers.advanced_activations.ELU(alpha=1.0))
    model.add(Conv2D(1024, (1, 1), padding='same'))
    model.add(keras.layers.advanced_activations.ELU(alpha=1.0))
    model.add(BatchNormalization())
    model.add(Dropout(0.5))

    model.add(Conv2D(2048, (1, 1), padding='same'))
    model.add(keras.layers.advanced_activations.ELU(alpha=1.0))
    model.add(Conv2D(2048, (1, 1), padding='same'))
    model.add(keras.layers.advanced_activations.ELU(alpha=1.0))
    model.add(BatchNormalization())
    model.add(Dropout(0.5))

    model.add(Flatten())

    '''
    model.add(Dense(512))
    model.add(keras.layers.advanced_activations.PReLU(alpha_initializer='zeros', alpha_regularizer=None, alpha_constraint=None, shared_axes=None))
    model.add(BatchNormalization())
    model.add(Dropout(0.5))
    model.add(Dense(512))
    model.add(keras.layers.advanced_activations.PReLU(alpha_initializer='zeros', alpha_regularizer=None, alpha_constraint=None, shared_axes=None))
    model.add(BatchNormalization())
    model.add(Dropout(0.5))
    '''
    model.add(Dense(num_classes))
    model.add(Activation('softmax'))
    if modelPath:
        model.load_weights(modelPath)
    model.summary()
    return model

opt = keras.optimizers.rmsprop(lr=0.01, decay=1e-6)
# Let's train the model using RMSprop
model=create_model()
model.compile(loss='categorical_crossentropy',
              optimizer=opt,
              metrics=['accuracy'])

x_train = x_train.astype('float32')
x_test = x_test.astype('float32')
x_train /= 255
x_test /= 255

if not data_augmentation:
    print('Not using data augmentation.')
    model.fit(x_train, y_train,
              batch_size=batch_size,
              epochs=epochs,
              validation_data=(x_test, y_test),
              shuffle=True)
else:
    print('Using real-time data augmentation.')
    # This will do preprocessing and realtime data augmentation:
    datagen = ImageDataGenerator(
        featurewise_center=False,  # set input mean to 0 over the dataset
        samplewise_center=False,  # set each sample mean to 0
        featurewise_std_normalization=False,  # divide inputs by std of the dataset
        samplewise_std_normalization=False,  # divide each input by its std
        zca_whitening=False,  # apply ZCA whitening
        rotation_range=0,  # randomly rotate images in the range (degrees, 0 to 180)
        width_shift_range=0.1,  # randomly shift images horizontally (fraction of total width)
        height_shift_range=0.1,  # randomly shift images vertically (fraction of total height)
        horizontal_flip=True,  # randomly flip images
        vertical_flip=False)  # randomly flip images

    # Compute quantities required for feature-wise normalization
    # (std, mean, and principal components if ZCA whitening is applied).
    datagen.fit(x_train)
    earlyStopping=EarlyStoppingByLossVal()
    reduceLR=ReduceLROnPlateau(monitor='val_loss', factor=0.8, patience=20, verbose=1, mode='auto', epsilon=0.0001, cooldown=0, min_lr=0.00001)
    tbCallBack = keras.callbacks.TensorBoard(log_dir='./Graph', histogram_freq=0, write_graph=True, write_images=True)
    filepath=&quot;weights.best.hdf5&quot;
    checkpoint = ModelCheckpoint(filepath, monitor='val_acc', verbose=1, save_best_only=True, mode='max')

    # Fit the model on the batches generated by datagen.flow().
    model.fit_generator(datagen.flow(x_train, y_train,
                                     batch_size=batch_size),verbose=1,callbacks=[reduceLR,checkpoint,earlyStopping,tbCallBack],
                        steps_per_epoch=x_train.shape[0] // batch_size,
                        epochs=epochs,
                        validation_data=(x_test, y_test))
    model.save('allConvNet.h5')
```
