---
layout: post
title:  "线性回归及其变体"
date:   2022-08-30 00:00:00 +0800
categories: machine-learning
---
**线性回归（Linear Regression）**

线性回归是将输入通过线性变换映射到输出的模型：


$$
\hat{y} = x_{1}\theta_{1} + x_{2}\theta_{2} + \dots + x_{n}\theta_{n} + b
$$

其中，

- $$ \hat{y} $$ 是预测输出；
- $$ x $$ 是输入特征；
- $$ \theta $$ 是权重变量；
- $$ b $$ 是偏置项。

使用线性代数进行描述：

$$
\hat{y} = \boldsymbol{\theta}\cdot\boldsymbol{x} + b = \boldsymbol{\theta}^{\intercal}\boldsymbol{x} + b
$$

$$ \boldsymbol{\theta}\cdot\boldsymbol{x} $$ 是向量点积，$$ \boldsymbol{\theta}^{\intercal}\boldsymbol{x} $$ 是矩阵乘法。

为了简化描述，把 $$b$$ 和 $$\theta$$ 合并在 $$\boldsymbol{w}$$ 中：

$$
\boldsymbol{w} = [b, \theta_1, \theta_2, \dots, \theta_n]
$$

输入则相应地变为：

$$
\boldsymbol{x} = [1, x_1, x_2, \dots, x_n]
$$

从而表达式可以简化为：

$$
\hat{y} = \boldsymbol{w}\cdot\boldsymbol{x}  = \boldsymbol{w}^{\intercal}\boldsymbol{x}
$$

线性回归通常使用**均方误差**（Mean Squared Error，**MSE**）作为损失函数：

$$
MSE(\boldsymbol{\theta}) = \frac{1}{m}\sum_{i=1}^{m}{(\hat{y}_{i} - y_{i})^{2}}
$$

$$ \boldsymbol{\theta} $$ 的最优解可以通过求逆矩阵的方法求得，也可以使用梯度下降法逼近。

**正则化（Regularization）**

通过添加正则项对模型的损失函数施加约束，以降低过拟合风险。

通过添加不同的正则项，线性回归可演变成多种回归。

**岭回归（Ridge Regression）**

正则项为 L2 范数。

$$
J(\boldsymbol{\theta}) = MSE(\boldsymbol{\theta}) + \alpha\frac{1}{2}\sum_{i=1}^{n}{\theta_{i}^{2}}
$$

其中 $$ \alpha $$ 用来控制正则化程度。

添加 L2 正则化后，$$ \boldsymbol{\theta} $$ 的分布会向 0 点集中。

**Lasso 回归（Lasso Regression）**

正则项为 L1 范数。

$$
J(\boldsymbol{\theta}) = MSE(\boldsymbol{\theta}) + \alpha\sum_{i=1}^{n}{|\theta_{i}|}
$$

添加 L1 正则项后，$$ \boldsymbol{\theta} $$ 的分布会变得更加稀疏，也就是会让一些 $$ \theta $$ 变为 0，另一些保留。这不经意间完成了特征选择。

**弹性网络（Elastic Net）**

介于岭回归和 Lasso 回归之间。

$$
J(\boldsymbol{\theta}) = MSE(\boldsymbol{\theta}) + \frac{1-r}{2}\alpha\sum_{i=1}^{n}{\theta_{i}^{2}} + r\alpha\sum_{i=1}^{n}{|\theta_{i}|}
$$

其中 $$ r $$ 用来控制混合比。
