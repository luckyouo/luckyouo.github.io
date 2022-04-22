# cs224n Lecture1 Word Vectors


## 前言

由于 one-hot 将词转为向量的方法简单且容易实现，但生成的向量并不能表示各个词之间的关系（比如常见的余弦相似度，因为 one-hot 生成的向量两两正交），而 word2vec 算法则解决了该问题，word2vec 算法主要有两种算法，一种是 `跳元模型` （Skip-grams , SG）算法，利用中心词推导上下文，另一种是 `连续词袋` （Continuous Bag of Words, CBOW）算法，利用上下文推导中心词。cs224n 第一节课主要介绍了 word2vec 跳元模型算法的原理与公式推导。

## 内容

在跳元模型当中，每个词都包含一对向量，即作为*中心词*向量和*上下文词*向量，更具体地说，对于词典中索引为i的任何词，分别用 $\mathbf{u}_i\in\mathbb{R}^d$ 和 $\mathbf{u}_i\in\mathbb{R}^d$ 表示其用作*中心词*和*上下文词*时的两个向量，计算给定中心词 $w_t$ 上下文词的条件概率，得到最大似然函数。最后利用最大似然函数的特性，转换为 log 函数进行最小化求解。

![image](/images/cs224n_1.png)

现在关键的问题为求解条件概率 $P(W_{t+j} | W_t; \theta)$ 。对于每个词 $w$，定义其两个词向量：$v_w$表示当 $w$ 为中心词时的词向量，$u_w$ 表示当w为其他词的临近词时的词向量。则对于一个中心词c和其临近词 $o$，则可以通过 softmax 进行建模。

![image-20220422201906064](/images/cs224n_2.png)

![image-20220422202232765](/images/cs224n_3.png)

**softmax 函数可以将放大较大 $x_i$ 概率（max），同时还保持较小的数的概率不为0（soft）。** 

求导的梯度如下，$u_o$ 表示观察到的上下文词向量，减号后面的是这个位置的期望的词向量，期望=概率*值。差值就是真实观察词向量减去期望词向量，这就是梯度。当它们的差值很小时，说明给定 $c$ 能很好的预测其临近词的概率分布。

![image-20220422203037342](/images/cs224n_4.png)

上述全连接网络虽然能很方便的计算词向量，但存在两个问题：1. 网络过于庞大，参数量太多；2. 训练样本太多，每个样本都更新所有参数，训练速度慢。针对这两个问题，作者分别提出了 subsampling 和 negative sampling 的技巧。

subsampling技巧是指，每个词有一个保留概率p，以这个概率p保留其在训练数据中，以1-p删掉这个词。对于词$w_i$，其概率 $P(w_i)$ 公式如下，其中 $z(w_i)$ 是词 $w$ 的词频。概率 $p$ 和这个词在语料库中的词频有关，词频越大，保留概率越低，即被删掉的概率越大，所以subsampling之后应该能较大的减少训练数据。

![image-20220422204030876](/images/cs224n_5.png)

egative sampling技巧，只更新一小部分参数。比如对于("fox", "quick")，期望的输出是quick的one-hot，即只有quick对应位为1，其他为0。但网络的softmax输出肯定不可能是完完全全的quick为1，其他为0；有可能是quick为0.8，其他位有些0.001，0.03之类的非0值，这就导致输出层的所有神经元都有误差。按照传统的方法，输出层所有神经元对应的U矩阵的权重都要更新。negative sampling技巧是，只更新和quick连接的U权重以及随机选5个输出神经元的连接权重进行更新，这样一下把需要更新的 U 权重个数从300w降到了6*300=1800，只需要更新0.06%的参数，大大减小了参数更新量！

![image-20220422204210388](/images/cs224n_6.png)

## 参考资料

[CS224N（1.8）Introduction and Word Vectors](https://bitjoy.net/2019/06/22/cs224n%ef%bc%881-8%ef%bc%89introduction-and-word-vectors/)

[Word2Vec Tutorial - The Skip-Gram Model](http://mccormickml.com/2016/04/19/word2vec-tutorial-the-skip-gram-model/)

[Word2Vec Tutorial Part 2 - Negative Sampling](http://mccormickml.com/2017/01/11/word2vec-tutorial-part-2-negative-sampling/)


