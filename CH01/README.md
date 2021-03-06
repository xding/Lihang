# CH01 统计学习方法概论

[TOC]

## 前言

### 章节目录

1. 统计学习
1. 监督学习
   1. 基本概念
   1. 问题的形式化
1. 统计学习三要素
   1. 模型
   1. 策略
   1. 算法
1. 模型评估与模型选择
   1. 训练误差与测试误差
   1. 过拟合与模型选择
1. 正则化与交叉验证
   1. 正则化
   1. 交叉验证
1. 泛化能力
   1. 泛化误差
   1. 泛化误差上界
1. 生成模型与判别模型
1. 分类问题
1. 标注问题
1. 回归问题

### 导读

- 直接看目录结构，会感觉有点乱，就层级结构来讲感觉并不整齐。

- 可以看本章概要部分，摘录几点，希望对本章内容编排的理解有帮助：
    > 1. 统计学习三要素对理解统计学习方法起到提纲挈领的作用
    > 1. 本书主要讨论**监督学习**
    > 1. 分类问题、标注问题和回归问题都是监督学习的重要问题
    > 1. 本书中介绍的统计学习方法包括...。这些方法是主要的分类、标注以及回归方法。他们又可归类为生成方法与判别方法。

- 本章最后的三个部分，这三个问题可以对比着看，暂时没有概念，略过也可以，回头对各个算法有了感觉回头再看这里。
  这三部分怎么对比，三部分都有个图来说明，仔细看下差异，本文后面会对此展开。

- 关于损失函数，风险函数与目标函数注意体会差异

- ~~损失函数对应Losses, 风险函数对应了Metrics~~这个先保留意见吧

## 实现统计学习方法的步骤

统计学习方法三要素:模型,策略,算法.

> 1. 得到一个有限的训练数据集合
> 1. 确定包含所有可能的模型的**假设空间**, 即学习模型的集合.
> 1. 确定模型选择的准则, 即学习的**策略**
> 1. 实现求解最优模型的算法, 即学习的**算法**
> 1. 通过学习方法选择最优的模型
> 1. 利用学习的最优模型对新数据进行预测或分析.

## 统计学习方法三要素

### 模型

#### 模型是什么?

在监督学习过程中, 模型就是所要学习的**条件概率分布**或者**决策函数**.

注意书中的这部分描述，整理了一下到表格里：

|              | 假设空间$\cal F$                                             | 输入空间$\cal X$ | 输出空间$\cal Y$ | 参数空间      |
| ------------ | ------------------------------------------------------------ | ---------------- | ---------------- | ------------- |
| 决策函数     | $\cal F\it =\{f_{\theta} |Y=f_{\theta}(x), \theta \in \bf R \it ^n\}$ | 变量             | 变量             | $\bf R\it ^n$ |
| 条件概率分布 | $\cal F\it =\{P|P_{\theta}(Y|X),\theta\in \bf R \it ^n\}$    | 随机变量         | 随机变量         | $\bf R\it ^n$ |

书中描述的时候，有提到**条件概率分布族**，这个留一下，后面[CH06](../CH06/README.md)有提到确认逻辑斯谛分布属于指数分布族。

### 策略

#### 损失函数与风险函数

> **损失函数**度量模型**一次预测**的好坏，**风险函数**度量**平均意义**下模型预测的好坏。

1. 损失函数(loss function)或代价函数(cost function)
   损失函数定义为给定输入$X$的预测值$f(X)$和真实值$Y$之间的**非负实值**函数, 记作$L(Y,f(X))$

1. 风险函数(risk function)或期望损失(expected loss)
   这个和模型的泛化误差的形式是一样的
   $R_{exp}(f)=E_p[L(Y, f(X))]=\int_{\mathcal X\times\mathcal Y}L(y,f(x))P(x,y)\, {\rm d}x{\rm d}y$
   模型$f(X)$关于联合分布$P(X,Y)$的**平均意义下的**损失(**期望**损失), 但是因为$P(X,Y)$是未知的, 所以前面的用词是**期望**, 以及**平均意义下的**.

   这个表示其实就是损失的均值, 反映了对整个数据的预测效果的好坏, $P(x,y)$转换成$\frac {\nu(X=x, Y=y)}{N}$更容易直观理解, 可以参考[CH09](../CH09/README.md), 6.2.2节的部分描述来理解, 但是真实的数据N是无穷的.

1. **经验风险**(empirical risk)或**经验损失**(empirical loss)
   $R_{emp}(f)=\frac{1}{N}\sum^{N}_{i=1}L(y_i,f(x_i))$
   模型$f$关于**训练样本集**的平均损失
   根据大数定律, 当样本容量N趋于无穷大时, 经验风险趋于期望风险

1. **结构风险**(structural risk)
   $R_{srm}(f)=\frac{1}{N}\sum_{i=1}^{N}L(y_i,f(x_i))+\lambda J(f)$
   $J(f)$为模型复杂度, $\lambda \geqslant 0$是系数, 用以权衡经验风险和模型复杂度.

#### 常用损失函数

损失函数数值越小，模型就越好

$L(Y,f(X))$

1. 0-1损失
1. 平方损失
1. 绝对损失

$L(Y,P(Y|X))$

1. 对数损失
   这里$P(Y|X)\leqslant 1$，对应的对数是负值，所以对数损失中包含一个负号，为什么不是绝对值？因为肯定是负的。

#### ERM与SRM

经验风险最小化(ERM)与结构风险最小化(SRM)

1. **极大似然估计**是经验风险最小化的一个例子.
   当模型是条件概率分布, 损失函数是对数损失函数时, 经验风险最小化等价于极大似然估计.
1. **贝叶斯估计**中的**最大后验概率估计**是结构风险最小化的一个例子.
   当模型是条件概率分布, 损失函数是对数损失函数, **模型复杂度由模型的先验概率表示**时, 结构风险最小化等价于最大后验概率估计.

### 算法

这章里面简单提了一下，具体可以参考[CH12](../CH12/README.md)表格中关于学习算法的描述。

## 模型选择

1. 正则化
   模型选择的典型方法是正则化
1. 交叉验证
   另一种常用的模型选择方法是交叉验证
   - 简单
   - S折(K折, K-Fold)[^1]
   - 留一法

## 泛化能力

- 现实中采用最多的方法是通过测试误差来评价学习方法的泛化能力
- 统计学习理论试图从理论上对学习方法的泛化能力进行分析

这本书里面讨论的不多，在[CH08](../CH08/README.md)里面有讨论提升方法的误差分析。

注意泛化误差的定义，书中有说**事实上，泛化误差就是所学习到的模型的期望风险**

## 生成模型与判别模型

监督学习方法可分为**生成方法**(generative approach)与**判别方法**(discriminative approach)

### 生成方法

generative approach

- 可以还原出**联合概率分布**$P(X,Y)$
- 收敛速度快, 当样本容量增加时, 学到的模型可以更快收敛到真实模型
- 当存在隐变量时仍可以用

### 判别方法

discriminative approach

- 直接学习**条件概率**$P(Y|X)$或者**决策函数**$f(X)$
- 直接面对预测, 往往学习准确率更高
- 可以对数据进行各种程度的抽象,  定义特征并使用特征, 可以简化学习问题

## 分类问题、标注问题、回归问题

Classification, Tagging, Regression

- 图1.4和图1.5除了分类系统和标注系统的差异外，没看到其他差异，但实际上这两幅图中对应的输入数据有差异，序列数据的$x_i = (x_i^{(1)},x_i^{(2)},\dots,x_i^{(n)})^T$对应了
- 图1.5和图1.6，回归问题的产出为$Y=\hat f(X)$

## 参考

参考文献都是大部头，ESL，PRML在列

1. [^1]: [ESL:7.10.1:K-Forld Cross Validation](##参考)