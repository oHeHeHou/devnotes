# Keras实现LSTM

1. Define Network
2. Compile Network
3. Fit Network
4. Evaluate Network
5. Make Predictions


## Define Network

```python
model = Sequential()
# input_shape参数：每个时间序列有2个样本，1个特征
# 第一个参数：指定隐藏层数量
model.add(LSTM(5, input_shape=(2,1)))
# 输出层
model.add(Dense(1))
```

## Compile Network

```python
model.compile(optimizer='sgd', loss='mean_squared_error')
```
```python
algorithm = SGD(lr=0.1, momentum=0.3)
model.compile(optimizer=algorithm, loss='mean_squared_error')
```

## Fit Network

```python
history = model.fit(X, y, batch_size=10, epochs=100, verbose=0)
```

## Evaluate Network

```python
loss, accuracy = model.evaluate(X, y)

```

## Make Predictions

```python
predictions = model.predict(X)
predictions = model.predict_classes(X)
```

* 对于二元分类问题，预测可能是第一类的概率数组，可以通过舍入将其转换为 1 或 0。
* 对于多类分类问题，结果可能采用概率数组的形式（假设是一个热编码输出变量），可能需要使用 argmax() NumPy函数将其转换为单类输出预测。或者，对于分类问题，我们可以使用Predict_classes()函数，它将自动将不清晰的预测转换为清晰的整数类值。

## 完整例子

```python
# Example of LSTM to learn a sequence
from pandas import DataFrame
from pandas import concat
from keras.models import Sequential
from keras.layers import Dense
from keras.layers import LSTM
# create sequence
length = 10
sequence = [i/float(length) for i in range(length)]
print(sequence)
# create X/y pairs
df = DataFrame(sequence)
df = concat([df.shift(1), df], axis=1)
df.dropna(inplace=True)
# convert to LSTM friendly format
values = df.values
X, y = values[:, 0], values[:, 1]
X = X.reshape(len(X), 1, 1)
# 1. define network
model = Sequential()
model.add(LSTM(10, input_shape=(1,1)))
model.add(Dense(1))
# 2. compile network
model.compile(optimizer='adam', loss='mean_squared_error')
# 3. fit network
history = model.fit(X, y, epochs=1000, batch_size=len(X), verbose=0)
# 4. evaluate network
loss = model.evaluate(X, y, verbose=0)
print(loss)
# 5. make predictions
predictions = model.predict(X, verbose=0)
print(predictions[:, 0])
```


# 参考资料

* https://machinelearningmastery.com/5-step-life-cycle-long-short-term-memory-models-keras/
* https://keras.io/api/layers/recurrent_layers/lstm/