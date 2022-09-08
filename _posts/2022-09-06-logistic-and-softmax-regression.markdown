---
layout: post
title:  "Logistic 回归和 Softmax 回归"
date:   2022-09-06 00:00:00 +0800
categories: machine-learning
---
# Logistic Regression

Logistic 回归，也称作 Logit 回归。名为回归，实为分类模型。逻辑回归与线性回归非常类似，都是输入特征乘以权重，再加上偏置项。不同之处在于，输出需要通过 Sigmoid 激活函数。

$$
\sigma(x) = \frac{1}{1 + e^{-x}}
$$

Sigmoid 的取值范围为 0 和 1 之间，符合概率的定义。可以将输出值 $$ p $$ 视作属于某个特定类别的概率。而 $$ 1 - p $$ 则为不属于该类别的概率。从而实现了**二元分类**。

$$ x $$ 在被传入 $$ \sigma $$ 之前，通常称其为 logit。因为按照 logit 函数的定义：$$ logit(x) = log(\frac{1}{1-x}) $$，logit 函数与 logistic 函数是相反的，一个是从 $$ (0, 1) $$ 映射到 $$ (-\infty, \infty) $$，一个是从 $$ (-\infty, \infty) $$ 映射到 $$ (0, 1) $$。

对于输出，正类为 1，负类为 0。可以使用下面的损失函数：

$$
lost(\boldsymbol{\theta}) =
\begin{cases}
    -\log(\hat{y}),  & \text{if } y = 1\\
    -\log(1-\hat{y}),& \text{if } y = 0
\end{cases}
$$

当正类接近 0 时，$$ -log(0 + \epsilon) $$（需要添加一个非常小的 epsilon 避免超出定义域范围） 会非常大，当被正确标为 1 时，$$ -log(1) $$ 等于 0。

反之，当负类标为 1 时，损失会极高，当被正确标为 0 时，损失为 0。

把条件函数合并成一个等式：

$$
J(\boldsymbol{\theta}) = -\frac{1}{m}\sum_{i=1}^{m}{
    [y_{i}\log(\hat{y}_{i}) + (1-y_{i})\log(1-\hat{y}_{i})]
}
$$

该损失函数称作**对数损失**。

# Softmax Regression

逻辑回归经过推广，可以直接支持**多类别分类**，而不需要训练并组合多个二元分类，这就是 Softmax 回归。

原理很简单，模型首先计算出每个分类的分数，然后对这些分数应用 softmax 函数（也叫归一化函数），使其满足和为 1，且值域落于 $$ (0, 1) $$。

$$
\sigma(\hat{p}_{i}) = \frac{e^{\hat{p}_i}}{\sum_{k=1}^{K}{e^{\hat{p}_{k}}}}
$$

取最高概率的类别：

$$
\hat{y} = \arg\min_{k}\sigma(\boldsymbol{\hat{p}})
$$

Softmax 回归一次只能预测一个类，它是多类别，但不是多输出。类和类之间是**互斥**的。

Softmax 回归使用**交叉熵**作为损失函数：

$$
J(\boldsymbol{\theta}) = -\frac{1}{m}\sum_{i=1}^{m}\sum_{k=1}^{K}y^{i}_{k}\log(\hat{y}^{i}_{k})
$$

其中，内求和（k）是对所有分类的损失计算，外求和（i）是对所有样本的损失计算。

从损失函数中可以发现，负类因为被标记为 0，相当于被忽略了，而只关心正类。当正类被错误标成 0 时，损失值非常大，当被正确标为 1 时，损失之为 0。这里忽略负类其实是合理的，因为 softmax 保证和为 1，而且又只有一个类会被标为 1，则当正类无限趋近于 1 时，其余负类相对应的趋近于 0。

交叉熵源于信息论，两个概率分布 p 和 q 之间的交叉熵定义为：

$$
H(p, q) = -\sum_{x}{p(x) \log q(x)}
$$
