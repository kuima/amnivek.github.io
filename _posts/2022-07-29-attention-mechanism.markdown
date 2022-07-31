---
layout: post
title:  "注意力机制的起源"
date:   2022-07-20 16:14:00 +0800
categories: machine-learning
---
*（“注意”和“注意力”常等价混用）*

人类的视网膜在一瞬间可以捕获到巨量的信息，但是大脑没有能力处理所有信息。幸运的是，人类学会了如何使用注意力，只对小部分关键信息进行处理。

在心理学上，注意力表现为对某对象的指向和集中，并且分为**内隐注意力**和**外显注意力**。内隐注意力是自动的，外显注意力则是需要意志努力完成的。

比如看到一句话：

> 质能方程由爱因斯坦于 1905 年提出。

我们的注意力首先会集中在 “质能方程”、“爱因斯坦”和“1905”上，这是内隐注意力在工作。当被问到质能方程是什么时候提出的时，我们会搜寻并且把注意力只集中到“1905”上，此时是外显注意力在工作。

最新的研究表明，内隐注意力和外显注意力并不是彼此独立的，而是互相权衡、共同作用的关系。也就是说，当带着目的去观察时，这两种注意力同时作用。

人工神经网络使用**Query-Key-Value**的架构模拟这种认知形式。

- Query(**q**) 代表外显注意力；
- Keys(**k**) 代表内隐注意力；
- Values(**v**) 代表原始输入。

$$
\boldsymbol{q} \in \mathbb{R}^q, \boldsymbol{k} \in \mathbb{R}^k, \boldsymbol{v} \in \mathbb{R}^v
$$

Keys 和 Values 是成对的。比如一句话中有 3 个单词，则 Values 和 Keys 的长度都为 3。

目标是从 Values 中找到需要集中注意力的部分。需要为 Values 中的每一个子元素计算一个注意力分数，也叫注意力权重，用来表示该子元素的重要程度：

$$
\boldsymbol{a} \in \mathbb{R}^v
$$

权重使用注意力分数函数 (Attention Scroing Function) 得出：

$$
\boldsymbol{a}_i = \alpha(\boldsymbol{q}, \boldsymbol{k}_i)
$$

为了使其满足概率律，权重使用 Softmax 函数进行归一化，得到值域满足 [0, 1] 且和为 1 的分布：

$$
\boldsymbol{a}_i = softmax(\alpha(\boldsymbol{q}, \boldsymbol{k}_i))
$$

这样对于每个 $$ \boldsymbol{v}_i $$ 都得到一个相对应的 $$ \boldsymbol{a}_i $$。

最后使用 Attention Pooling 函数进行注意力集中操作，得到我们想要的输出：

$$
f(\boldsymbol{q}, (\boldsymbol{k}, \boldsymbol{v})) = \sum_{i}^{n}{\boldsymbol{a}_i \boldsymbol{v}_i}
$$

其本质就是加权平均，把 Values 的子元素按照各自相应的权重压到了一起。

前面使用了注意力分数函数 $$ \alpha $$，但是并没有说明它是如何定义的。注意力分数函数通常有两种：**加法注意力(Additive Attention)**和**乘法注意力**，乘法注意力也叫缩放点积注意力(Scaled Dot-Product Attention)。

**加法注意力**

当 Query 和 Key 向量的维度不同时，可以使用加法注意力作为注意力分数函数：

$$
\alpha(\boldsymbol{q}, \boldsymbol{k}) = \mathbf{w}_v^\top tanh(\mathbf{W}_q\boldsymbol{q} + \mathbf{W}_k\boldsymbol{k}) \in \mathbb{R}
$$

其中，可学习的变量：

$$
\mathbf{W}_q \in \mathbb{R}^{h \times q}, \mathbf{W}_k \in \mathbb{R}^{h \times k}, \mathbf{w}_v \in \mathbb{R}^{h} \\
$$

$$ h $$ 是隐藏层的单元数量。

**缩放点积注意力**

缩放点积注意力相比于加法注意力具有更高效的计算，但是需要满足 $$ \boldsymbol{q} $$ 和 $$ \boldsymbol{k} $$ 具有相同的维度 $$ d $$。注意力分数函数定义为：

$$
\alpha(\boldsymbol{q}, \boldsymbol{k}) = \frac{\boldsymbol{q}^\top\boldsymbol{k}}{\sqrt{d}}
$$

除以 $$ \sqrt{d} $$ 为了保持向量相乘后，其方差依然和原来一样保持单位方差。

向量点积从几何角度看，是两个向量的长度与它们之间的夹角余弦的积，其结果反应的是两个向量的相似度，值越大越相似度越高。

点积注意力没有引入额外的可学习参数，而是假设具有高注意力表现时 Query 和 Key 应当具有高相似度，无关紧要的部分需要具有低相似度。相似度的高低是期望的结果，学习过程则交给了网络模型的其余部分。

到此位置介绍了 QKV 结构下的注意力机制，而这只是一种框架，如何使用则需要根据网络模型的设计和具体任务自定义。在 Encoder-Decoder 模型中，Bahdanau Attention 使用 Encoder 的所有输出作为 Keys 和 Values，Decoder 的输出作为 Query。在 Transformer 模型中，Tokens 同时作为 Queries、Keys 和 Values，这也是它为什么叫做 Self-Attention 的原因。
