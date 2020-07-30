## TensorFlow安装和基础入门

### 环境

1. Ubuntu16.04
2. GPU版本需要PC有英伟达显卡

### 简介

TensorFlow是一个[开源](https://zh.wikipedia.org/wiki/开放源代码)[软件库](https://zh.wikipedia.org/wiki/库_(计算机))，也是非常受欢迎的机器学习框架,用于各种感知和语言理解任务的[机器学习](https://zh.wikipedia.org/wiki/机器学习)。TensorFlow能够用少量代码构建出模型,并且转换成图进行运行,在这些操作之间运行的多维数据叫做张量(Tensor),图中的tensors不断的flow,因此命名为TensorFlow.

#### TensorFlow的安装

Tensorflow有两种版本可以选择进行安装,分别是CPU版本和GPU版本,CPU版本适合初级入门学习Tensorflow,它安装比较简单,不容易出错,程序运行在CPU上.而GPU适合计算大型的神经网络,对硬件要求比较高,所以后期深入学习后GPU版本是比较优秀的选择.

##### ubuntu 16.04 安装Tensorflow(CPU)

首先确认自己Ubuntu安装的python版本,Ubuntu16.04自带python2.7,如果没有安装python3.5建议首先安装python3.5,并更改系统默认使用python版本为3.5.

首先查看自己系统所装python2的版本:

```
python2 --version  #查看python2安装版本
```

首先查看自己系统所装python3的版本:

```
python3 --version  #查看python3安装版本
```

首先查看系统正在使用的python版本:

``````
python --version  #查看系统正在使用的python版本
``````

如果没有安装python3,请安装python3(不要卸载python2,可以共存)

```
sudo apt-get install python3-pip

sudo add-apt-repository ppa:jonathonf/python-3.6 # 安装python3.6

sudo apt update

sudo apt install python3.6
```

验证已经安装好的Python3.6

```
python3 --version
```

Python3和Python2是互相不兼容，但也不能卸载python2，可以将系统使用Python的指向Python3即可:

```
配置：
echo alias python=python3 >> ~/.bashrc
source生效：
source ~/.bashrc
版本确认：
python --version
```

Python3配置完毕后,开始正式安装TensorFlow:

1. 安装pip,打开终端输入命令:

```
sudo apt-get install python-pip python-dev
```

2. 安装tensorflow

```
sudo pip install --upgrade https://storage.googleapis.com/

tensorflow/linux/cpu/tensorflow-0.8.0rc0-cp27-none-linux_x86_64.whl  
```

终端未报错，初步安装成功，下面进行使用测试．

3. 新建python文件helloTensorFlow.py输入下面代码进行测试:

```
#Python 3
import tensorflow as tf
hello = tf.constant('Hello, TensorFlow!')
sess = tf.Session()
print(sess.run(hello))
```

在IDE中或者直接运行helloTensorFlow.py:

测试通过,TensorFlow-CPU版已成功安装.

##### ubuntu 16.04 安装Tensorflow(GPU)

GPU的安装比较复杂,对硬件有一定的要求.安装要严格按照顺序和相关软件版本进行,否则会出现意外的问题:

1. 首先下载安装Ubuntu NVIDIA GPU驱动

显卡驱动的安装根据个人电脑情况而定,建议根据自己的硬件找对应的教程安装.

2. 安装CUDA

使用GPU必须要安装CUDA,CUDA是NVIDIA的编程语言平台.

CUDA下载官网:https://developer.nvidia.com/cuda-downloads

**特别注意:**下载CUDA时一定要注意CUDA和NVIDIA显卡驱动的适配性。CUDA_8.0支持375.**及以上系列的显卡驱动；CUDA_9.0支持384.**及以上系列的显卡驱动；CUDA_9.1支持389.**及以上系列的显卡驱动。

执行下列命令：

```
# 以CUDA9.0为例,运行安装程序,执行以下命令：
sudo chmod 777 cuda_9.0.176_384.81_linux.run
sudo ./cuda_9.0.176_384.81_linux.run

# 环境变量设置,打开~/.bashrc文件：sudo gedit ~/.bashrc,将以下内容写入到~/.bashrc尾部：
export PATH=/usr/local/cuda-9.0/bin${PATH:+:${PATH}}
export LD_LIBRARY_PATH=/usr/local/cuda/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
export CUDA_HOME=$CUDA_HOME:/usr/local/cuda-9.0

# 打开/etc/profile: sudo gedit /etc/profile,将以下内容写入文件末尾：
export PATH=/usr/local/cuda-9.0/bin:$PATH
export LD_LIBRARY_PATH=/usr/local/cuda-9.0/lib64:$LD_LIBRARY_PATH

# source
source /etc/profile

# 测试CUDA是否安装成功
nvcc -V

# 有显示CUDA版本信息，再执行，显示GPU信息则说明安装成功
cd /usr/local/cuda-9.0/samples/1_Utilities/deviceQuery
sudo make
sudo ./deviceQuery
```

3. 安装cuDNN

cuDNN是GPU加速计算深层神经网络的库

官网：https://developer.nvidia.com/rdp/cudnn-download

cuDNN要注意与CUDA的适配性，cuDNN7.0对应CUDA9.0

4. 调整gcc/g++版本

Ubuntu16.04中gcc为gcc6,Ubuntu18中为gcc7

而安装cuda，官方文档中说明，Ubuntu16中gcc最高5.3,因而gcc要降级：

```
# 检查版本

gcc --version
g++ --version

# 降级

sudo apt install gcc-4.8
sudo apt install g++-4.8

sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-4.8 60 --slave /usr/bin/g++ g++ /usr/bin/g++-4.8

# 检查降级后的版本是否正确
gcc --version
g++ --version
```

5. 安装tensorflow-gpu==1.7.0

```
pip　install -U --pre　tensorflow-gpu==1.7.0
```

6. 终端未报错，初步安装成功，下面进行使用测试．

7. 测试，新建python文件helloTensorFlow.py输入下面代码进行测试:

```
#Python 3
import tensorflow as tf
hello = tf.constant('Hello, TensorFlow!')
sess = tf.Session()
print(sess.run(hello))
```

在IDE中或者直接运行helloTensorFlow.py:

测试通过,TensorFlow-GPU版已成功安装.

### 基础入门

我们用一个例子，来梳理一下tensorflow的运算逻辑，例：矩阵计算

```
# 导入tensorflow库

import tensorflow as tf

# 构建计算图
c = tf.constant([[1.0, 2.0], [3.0, 4.0]])
d = tf.constant([[1.0, 1.0], [0.0, 1.0]])
e = tf.matmul(c, d)

# 构造一个“会话”来执行图形.
with tf.Session() as sess:

# 将计算图写入log，方便使用tensorboard查看.

writer = tf.summary.FileWriter("logs/", sess.graph)
# 执行图形并将' e '表示的值存储在' result '中.
result = sess.run(e)

print(result)
```

整体代码分成了两部分，第一部分构建图，分别定义好了两个张量ｃ和d，这两个张量里储存两个常量矩阵，e也被声明好了一个张量，定义e是c和d矩阵相乘之后的结构，这就构成了一个计算图：

（使用命令：`tensorboard --logdir logs/` 进行查看）

c和d两个张量flow向MatＭul.得出结果e,到此时还只是构建好了计算图，真正驱动tensor进行flow，还要靠第二部分代码，构建一个会话，然后run即可，这里为固定格式．两部分搭建完成，即可驱动计算图，得出我们想要的结果，这也是tensorflow最简单的理解，实际使用中，我们往往会遇到复杂的多的案例，但基本逻辑是相通的.