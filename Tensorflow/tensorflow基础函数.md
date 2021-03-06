## Tensorflow 基础函数    

### tf.argmax()   

类似于`np.argmax()`,返回最大数值所在的下标    
当axis=1时返回每列最大值的下标，当axis=0时返回每行最大值的下标   

### tf.equal()    

对比两个矩阵或向量相等的元素，如果相等对应位置上为True，反之为False，返回的矩阵维度与比较的矩阵一致      

```python
A = [[1,3,4,5,6]]
B = [[1,3,4,3,2]]

with tf.Session() as sess:
    print(sess.run(tf.equal(A, B)))
```    

输出：  
```
[[ True  True  True False False]]
```   

### tf.cast()     

`tf.cast(x, dtype)`将x的数据格式转换为dtype       

```python
a = tf.Variable([1,0,1,0,1])
b = tf.cast(a, dtype=tf.bool)
sess = tf.Session()
sess.run(tf.global_variables_initializer())
print(sess.run(b)) 
```  
输出：    
```
[True, False, False, True, True]
```      

### tf.argmax, tf.equal, tf.cast, tf.one_hot, tf.reduce_mean整合计算预测的准确度   

[博客](https://blog.csdn.net/liuxiao214/article/details/78999444)   



### tf.truncated_normal()   

`tf.truncated_normal(shape, mean, stddev)`生成截断式的正太分布数据。         
shape表示生成tensor的维度，mean是均值，stddev是标准差      
正态分布有$3/sigma$分布，如果随机生成的数据与均值的差值大于两倍的标准差，就重新生成。     


### tf.control_dependencies(control_inputs))     

为了创造某些操作执行的依赖关系，可以使用该函数，函数返回一个控制依赖的上下文管理器，使用`with`关键字可让在这个上下文环境中的操作都在`control_inputs`执行          

```python
with tf.control_dependencies([a, b, c]):
# 'd' and `e` will only run after `a`, `b`, and `c` have executedd
    d = ...
    e = ...
```          

### tf.nn.conv2d()    

`tf.nn.conv2d(input, filter, strides, padding, use_cuda_on_gpu=None, name=None)`       

* input参数 

    是一个四维张量[batch_size, image_width, image_height, image_depth], batch_size训练时一个batch的图片数量，而且要求数据类型为`float32`,`float64`其中之一       

* filter参数    

    也是一个四维张量，[filter_size, filter_size, image_depth, filter_depth], 要求数据类型和input一致，注意第三个维度`image_depth`就是input参数中的第四维        

* strides参数      

    卷积的步幅，即卷积滤波器在四个维度中的每一次移动的距离，这四个维度对应着input参数中的四个维度，第一个维度代表图像的批量数，所以该维度每次只能移动一张图片，最后一个维度为图像深度（色彩通道数，1为灰度图像，3为彩色图像），如果不想跳过任何一个通道，所以这个参数为1，第二个和第三个维度代表X和Y方向的步幅，如果设定步幅为1，那步幅参数就需要设定为[1,1,1,1],如果希望在图像上移动的步幅为2，步幅参数为[1,2,2,1]         

* padding参数    

    这个参数表示在卷积过程中是否用0来填充边界，这样可以保证图像输出尺寸在步幅参数设定为1的情况下保持不变，通过设定padding='SAME',图像会用0来填充边界(输出尺寸不变)，如果设定padding='VALID'则不会进行填充。     

    对于任意给定的步幅S，滤波器尺寸K，图像尺寸W，padding尺寸P，可以确定输出图像尺寸：    

    $$O = 1 + (W - K + 2P)/S$$        

### tf.get_collection(tf.GraphKeys.UPDATE_OPS)      

主要使用在Batch Normalization操作中      
使用`update_ops = tf.get_collection(tf.GraphKeys.UPDATE_OPS)`以便在每一次训练完成后及时更新BN的参数     

```python
update_ops = tf.get_collection(tf.GraphKeys.UPDATE_OPS)
with tf.control_dependencies(update_ops):#保证train_op在update_ops执行之后再执行
    train_op = optimizer.minimize(loss)
```          

### tf.contrib.training.HParams      

用于保存一组超参数作为`key-value`的类   

该类对象拥有用于构建和训练模型的超参数,这些超参数作为对象的属性,可以直接访问    

```python
hparams = HParams(learing_rate=1.0, num_hidden_units=100)    

# the hyperparameter are available as attributes of Hparams object  
hparams.learning_rate ==> 0.1
hparams.num_hidden_units ==> 100   
```  

还可以通过`parse()`方法来覆盖超参数值      

### tf.cond   

用于实现条件控制数据流向，相当于if-else    

函数原型：  
```python
tf.cond(pred, fn1, fn2, name=None);  
```  
这里fn1和fn2是两个函数，pred会返回True或者False    
当pred返回True时，返回fn1函数执行的结果   
当pred返回False时，返回fn2函数执行的结果   

```python
# 这里要区分是训练过程中的bn还是测试时的bn
# 使用cond来进行判断，cond的第一个参数是一个逻辑表达式
# 返回为true时，执行后面第一个函数，否则执行后面第二个函数
# bn在train过程中，mean和var都是计算当前batch的   
# bn在test过程中，输入的是单个data，这时bn计算用到的mean和var可以由整个数据集的代替
# 或者使用每次train中每个batch的均值来代替
mean, var = tf.cond(train_phase, mean_var_with_update(),
               lambda: (ema.average(batch_mean), ema.average(batch_var)))
```
