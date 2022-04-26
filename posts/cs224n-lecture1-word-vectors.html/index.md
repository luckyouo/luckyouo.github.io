# cs224n Lecture1 Word Vectors


## 前言

由于 one-hot 将词转为向量的方法简单且容易实现，但生成的向量并不能表示各个词之间的关系（比如常见的余弦相似度，因为 one-hot 生成的向量两两正交），而 word2vec 算法则解决了该问题，word2vec 算法主要有两种算法，一种是 `跳元模型` （Skip-grams , SG）算法，利用中心词推导上下文，另一种是 `连续词袋` （Continuous Bag of Words, CBOW）算法，利用上下文推导中心词。cs224n 第一节课主要介绍了 word2vec 跳元模型算法的原理与公式推导。

## 内容

### 基本模型

在跳元模型当中，每个词都包含一对向量，即作为*中心词*向量和*上下文词*向量，更具体地说，对于词典中索引为i的任何词，分别用 $\mathbf{u}_i\in\mathbb{R}^d$ 和 $\mathbf{u}_i\in\mathbb{R}^d$ 表示其用作*中心词*和*上下文词*时的两个向量，计算给定中心词 $w_t$ 上下文词的条件概率，得到最大似然函数。最后利用最大似然函数的特性，转换为 log 函数进行最小化求解。

![image](/images/cs224n_1.png)

现在关键的问题为求解条件概率 $P(W_{t+j} | W_t; \theta)$ 。对于每个词 $w$，定义其两个词向量：$v_w$表示当 $w$ 为中心词时的词向量，$u_w$ 表示当w为其他词的临近词时的词向量。则对于一个中心词c和其临近词 $o$，则可以通过 softmax 进行建模。

![image-20220422201906064](/images/cs224n_2.png)

![image-20220422202232765](/images/cs224n_3.png)

**softmax 函数可以将放大较大 $x_i$ 概率（max），同时还保持较小的数的概率不为0（soft）。** 

求导的梯度如下，$u_o$ 表示观察到的上下文词向量，减号后面的是这个位置的期望的词向量，期望=概率*值。差值就是真实观察词向量减去期望词向量，这就是梯度。当它们的差值很小时，说明给定 $c$ 能很好的预测其临近词的概率分布。

对中心向量求导：

![image-20220426104810429](/images/cs224n_8.png)

![image-20220426104638794](/images/cs224n_7.png)

对上下文向量求导：

![image-20220426105101981](/images/cs224n_9.png)

上述全连接网络虽然能很方便的计算词向量，但存在两个问题：1. 网络过于庞大，参数量太多；2. 训练样本太多，每个样本都更新所有参数，训练速度慢。针对这两个问题，作者分别提出了 subsampling 和 negative sampling 的技巧。

subsampling技巧是指，每个词有一个保留概率p，以这个概率p保留其在训练数据中，以1-p删掉这个词。对于词$w_i$，其概率 $P(w_i)$ 公式如下，其中 $z(w_i)$ 是词 $w$ 的词频。概率 $p$ 和这个词在语料库中的词频有关，词频越大，保留概率越低，即被删掉的概率越大，所以subsampling之后应该能较大的减少训练数据。

$$P(W_i) = (\sqrt{\frac{z(w_i)}{0.001}} + 1) * \frac{0.001}{z(w_i)}$$

### Negative Sampling

**negative sampling** 技巧，只更新一小部分参数。比如对于("fox", "quick")，期望的输出是quick的one-hot，即只有quick对应位为1，其他为0。但网络的softmax输出肯定不可能是完完全全的quick为1，其他为0；有可能是quick为0.8，其他位有些0.001，0.03之类的非0值，这就导致输出层的所有神经元都有误差。按照传统的方法，输出层所有神经元对应的U矩阵的权重都要更新。negative sampling技巧是，只更新和quick连接的U权重以及随机选5个输出神经元的连接权重进行更新，这样一下把需要更新的 U 权重个数从300w降到了6*300=1800，只需要更新0.06%的参数，大大减小了参数更新量！

考虑一对中心词和上下文词$(w, c)$，我们通过 $P(D = 1 | w,c)$ 表示 $(w, c)$是来自语料库。相应地，表示 $P(D = 0 | w,c)$ 不是来自语料库。首先,我们对 $P(D = 1 | w,c)$ 用 sigmoid 函数建模:

$$ P(D = 1| w, c, \theta) = \sigma(v_c^tv_w) = \frac{1}{1 + e^{-v_c^tv_w}}$$

我们建立一个新的目标函数,如果中心词和上下文词确实在语料库中，就最大化概率 ,如果中心词和上下文词确实不在语料库中，就最大化概率 $P(D = 1 | w,c)$  。我们对这两个概率采用一个简单的极大似然估计的方法(这里我们把
 $\theta$ 作为模型的参数,在我们的例子是 $V$ 和 $U$。

![image-20220426110234357](/images/cs224n_10.png)

最大化似然函数等同于最小化负对数似然：

![image-20220426110352001](/images/cs224n_11.png)

有很多关于如何得到最好近似的讨论,从实际效果看来最好的是指数为 3/4 的 Unigram 模型。

$$P(w_i) = \frac{f(w_i)^{3/4}}{\sum_{j=0}^{n} f(w_j)^{3/4}}$$

### Hierarchical softmax

在实际中，hierarchical softmax 对低频词往往表现得更好，负采样对高频词和较低维度向量表现得更好。

Hierarchical softmax 使用一个二叉树来表示词表中的所有词。树中的每个叶结点都是一个单词,而且只有一条路
径从根结点到叶结点。在这个模型中,没有词的输出表示。相反,图的每个节点(根节点和叶结点除外)与模型要
学习的向量相关联。单词作为输出单词的概率定义为从根随机游走到单词所对应的叶的概率。计算成本变为 $O(log|V|)$ 而不是 $O(|V|)$ 。

下图是 Hierarchical softmax 的二叉树示意图

![image-20220426111039969](/images/cs224n_12.png)

让我们引入一些概念。令 $L(w)$ 为从根结点到叶结点 $w$ 的路径中节点数目。例如，上图中的 $L(w_2)$ 为 3。我们定
义 $n(w, i)$ 为与向量 $v_n(w, i)$ 相关的路径上第 $i$ 个结点。因此 $n(w, 1)$ 是根结点，而 $n(w, L(w))$ 是 $w$ 的父节
点。现在对每个内部节点 $n$ ，我们任意选取一个它的子节点，定义为 $ch(n)$ (一般是左节点)。然后，我们可以
计算概率为：

$$p(w|w_i) = \prod_{j=1}^{L(w) - 1}\sigma([n(w, j+1) = ch(n(w, j))] * v_{n(w,j)}^Tv_{w_j})$$

其中 $[x]$ 当 x 为true 时，为 1，否则为 0。

## 参考资料

[CS224N（1.8）Introduction and Word Vectors](https://bitjoy.net/2019/06/22/cs224n%ef%bc%881-8%ef%bc%89introduction-and-word-vectors/)

[Word2Vec Tutorial - The Skip-Gram Model](http://mccormickml.com/2016/04/19/word2vec-tutorial-the-skip-gram-model/)

[Word2Vec Tutorial Part 2 - Negative Sampling](http://mccormickml.com/2017/01/11/word2vec-tutorial-part-2-negative-sampling/)


