# MANE

```python
import tensorflow as tf 
from tensorflow.keras import datasets,layers,models

BATCH_SIZE = 100

def load_image(img_path,size = (32,32)):
    label = tf.constant(1,tf.int8) if tf.strings.regex_full_match(img_path,".*automobile.*") \
            else tf.constant(0,tf.int8)
    img = tf.io.read_file(img_path)
    img = tf.image.decode_jpeg(img) #注意此处为jpeg格式
    img = tf.image.resize(img,size)/255.0
    return(img,label)

#使用并行化预处理num_parallel_calls 和预存数据prefetch来提升性能
ds_train = tf.data.Dataset.list_files("./data/cifar2/train/*/*.jpg") \
           .map(load_image, num_parallel_calls=tf.data.experimental.AUTOTUNE) \
           .shuffle(buffer_size = 1000) \
           .batch(BATCH_SIZE) \
           .prefetch(tf.data.experimental.AUTOTUNE)  

ds_test = tf.data.Dataset.list_files("./data/cifar2/test/*/*.jpg") \
           .map(load_image, num_parallel_calls=tf.data.experimental.AUTOTUNE) \
           .batch(BATCH_SIZE) \
           .prefetch(tf.data.experimental.AUTOTUNE)  

```


```python
%matplotlib inline
#%config InlineBackend.figure_format = 'svg'

#查看部分样本
from matplotlib import pyplot as plt 

plt.figure(figsize=(8,8)) 
for i,(img,label) in enumerate(ds_train.unbatch().take(9)):
    ax=plt.subplot(3,3,i+1)
    ax.imshow(img.numpy())
    ax.set_title("label = %d"%label)
    ax.set_xticks([])
    ax.set_yticks([]) 
plt.show()
```


​    
![svg](op2_files/op2_1_0.svg)
​    



```python
for x,y in ds_train.take(1):
    print(x.shape,y.shape)
```

    (100, 32, 32, 3) (100,)



```python
tf.keras.backend.clear_session() #清空会话

inputs = layers.Input(shape=(32,32,3))
x = layers.Conv2D(32,kernel_size=(3,3))(inputs)
x = layers.MaxPool2D()(x)
x = layers.Conv2D(64,kernel_size=(5,5))(x)
x = layers.MaxPool2D()(x)
x = layers.Dropout(rate=0.1)(x)
x = layers.Flatten()(x)
x = layers.Dense(32,activation='relu')(x)
outputs = layers.Dense(1,activation = 'sigmoid')(x)

model = models.Model(inputs = inputs,outputs = outputs)

model.summary()
```

    Model: "model"
    _________________________________________________________________
    Layer (type)                 Output Shape              Param #   
    =================================================================
    input_1 (InputLayer)         [(None, 32, 32, 3)]       0         
    _________________________________________________________________
    conv2d (Conv2D)              (None, 30, 30, 32)        896       
    _________________________________________________________________
    max_pooling2d (MaxPooling2D) (None, 15, 15, 32)        0         
    _________________________________________________________________
    conv2d_1 (Conv2D)            (None, 11, 11, 64)        51264     
    _________________________________________________________________
    max_pooling2d_1 (MaxPooling2 (None, 5, 5, 64)          0         
    _________________________________________________________________
    dropout (Dropout)            (None, 5, 5, 64)          0         
    _________________________________________________________________
    flatten (Flatten)            (None, 1600)              0         
    _________________________________________________________________
    dense (Dense)                (None, 32)                51232     
    _________________________________________________________________
    dense_1 (Dense)              (None, 1)                 33        
    =================================================================
    Total params: 103,425
    Trainable params: 103,425
    Non-trainable params: 0
    _________________________________________________________________



```python
import datetime
import os

stamp = datetime.datetime.now().strftime("%Y%m%d-%H%M%S")
logdir = os.path.join('data', 'autograph', stamp)

## 在 Python3 下建议使用 pathlib 修正各操作系统的路径
# from pathlib import Path
# stamp = datetime.datetime.now().strftime("%Y%m%d-%H%M%S")
# logdir = str(Path('./data/autograph/' + stamp))

tensorboard_callback = tf.keras.callbacks.TensorBoard(logdir, histogram_freq=1)

model.compile(
        optimizer=tf.keras.optimizers.Adam(learning_rate=0.001),
        loss=tf.keras.losses.binary_crossentropy,
        metrics=["accuracy"]
    )

history = model.fit(ds_train,epochs= 10,validation_data=ds_test,
                    callbacks = [tensorboard_callback],workers = 4)
```

    Epoch 1/10
    100/100 [==============================] - 7s 72ms/step - loss: 0.4195 - accuracy: 0.8098 - val_loss: 0.3007 - val_accuracy: 0.8720
    Epoch 2/10
    100/100 [==============================] - 6s 64ms/step - loss: 0.2933 - accuracy: 0.8758 - val_loss: 0.2309 - val_accuracy: 0.9045
    Epoch 3/10
    100/100 [==============================] - 6s 64ms/step - loss: 0.2437 - accuracy: 0.8957 - val_loss: 0.2037 - val_accuracy: 0.9155
    Epoch 4/10
    100/100 [==============================] - 6s 65ms/step - loss: 0.2040 - accuracy: 0.9164 - val_loss: 0.1791 - val_accuracy: 0.9305
    Epoch 5/10
    100/100 [==============================] - 7s 65ms/step - loss: 0.1825 - accuracy: 0.9263 - val_loss: 0.1722 - val_accuracy: 0.9335
    Epoch 6/10
    100/100 [==============================] - 6s 64ms/step - loss: 0.1543 - accuracy: 0.9395 - val_loss: 0.1698 - val_accuracy: 0.9345
    Epoch 7/10
    100/100 [==============================] - 6s 64ms/step - loss: 0.1365 - accuracy: 0.9465 - val_loss: 0.1637 - val_accuracy: 0.9315
    Epoch 8/10
    100/100 [==============================] - 6s 64ms/step - loss: 0.1235 - accuracy: 0.9523 - val_loss: 0.1672 - val_accuracy: 0.9355
    Epoch 9/10
    100/100 [==============================] - 6s 65ms/step - loss: 0.1107 - accuracy: 0.9576 - val_loss: 0.1563 - val_accuracy: 0.9380
    Epoch 10/10
    100/100 [==============================] - 6s 64ms/step - loss: 0.0916 - accuracy: 0.9661 - val_loss: 0.1570 - val_accuracy: 0.9420



```python
%load_ext tensorboard
#%tensorboard --logdir ./data/keras_model
```


```python
from tensorboard import notebook
notebook.list() 
```

    No known TensorBoard instances running.



```python
import pandas as pd 
dfhistory = pd.DataFrame(history.history)
dfhistory.index = range(1,len(dfhistory) + 1)
dfhistory.index.name = 'epoch'

dfhistory
```



<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>loss</th>
      <th>accuracy</th>
      <th>val_loss</th>
      <th>val_accuracy</th>
    </tr>
    <tr>
      <th>epoch</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>0.419485</td>
      <td>0.8098</td>
      <td>0.300734</td>
      <td>0.8720</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0.293310</td>
      <td>0.8758</td>
      <td>0.230946</td>
      <td>0.9045</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0.243747</td>
      <td>0.8957</td>
      <td>0.203702</td>
      <td>0.9155</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0.204024</td>
      <td>0.9164</td>
      <td>0.179141</td>
      <td>0.9305</td>
    </tr>
    <tr>
      <th>5</th>
      <td>0.182504</td>
      <td>0.9263</td>
      <td>0.172155</td>
      <td>0.9335</td>
    </tr>
    <tr>
      <th>6</th>
      <td>0.154267</td>
      <td>0.9395</td>
      <td>0.169848</td>
      <td>0.9345</td>
    </tr>
    <tr>
      <th>7</th>
      <td>0.136530</td>
      <td>0.9465</td>
      <td>0.163746</td>
      <td>0.9315</td>
    </tr>
    <tr>
      <th>8</th>
      <td>0.123512</td>
      <td>0.9523</td>
      <td>0.167173</td>
      <td>0.9355</td>
    </tr>
    <tr>
      <th>9</th>
      <td>0.110653</td>
      <td>0.9576</td>
      <td>0.156335</td>
      <td>0.9380</td>
    </tr>
    <tr>
      <th>10</th>
      <td>0.091556</td>
      <td>0.9661</td>
      <td>0.157044</td>
      <td>0.9420</td>
    </tr>
  </tbody>
</table>