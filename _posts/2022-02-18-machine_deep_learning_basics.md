---
title: 机器学习&深度学习
tags: AI
# article_header:
#   type: cover
#   image:
#     src: /assets/images/helloworld.jpg
---

<!-- write excerpt here -->
由于其广泛应用和个人需要，在此对一些machine learning和deep learning的基础进行总结。

<!--more-->

首先需要明确的是，人工智能（Artificial Intelligence）概念包括了机器学习（Machine Learning），而深度学习（Deep Learning）是机器学习中的一个更小但热度极高的一个方向，三者之间的关系可以总结为如下：

![](https://pic1.zhimg.com/v2-e358e127afbe5963f5b8622e2dd5b49f_r.jpg?source=1940ef5c)

机器学习中包括一些传统的算法，如：决策树、聚类、贝叶斯分类、支持向量机等等。这些方法又可以分为监督学习（如分类问题）、无监督学习（如聚类问题）、半监督学习、集成学习、深度学习和强化学习。

而深度学习则是一种通过利用深度神经网络来进行有监督/无监督的机器学习的一种方法，由于计算机自身性能的提高，深度学习的训练时间和训练效果在变的越来越好。但深度学习自身既不是解决所有机器学习问题的最好选择：
1. 深度学习模型需要大量的训练数据，才能展现出神奇的效果，但现实生活中往往会遇到小样本问题，此时深度学习方法无法入手，传统的机器学习方法就可以处理；
2. 有些领域，采用传统的简单的机器学习方法，可以很好地解决了，没必要非得用复杂的深度学习方法；
3. 深度学习的思想，来源于人脑的启发，但绝不是人脑的模拟，举个例子，给一个三四岁的小孩看一辆自行车之后，再见到哪怕外观完全不同的自行车，小孩也十有八九能做出那是一辆自行车的判断，也就是说，人类的学习过程往往不需要大规模的训练数据，而现在的深度学习方法显然不是对人脑的模拟。

也不是机器学习的终点（如下一代可能的*迁移学习*等技术），仅仅是当前一种效果较好，且有研究前景的一个机器学习热门方向而已。

## Machine Learning

### 线性回归&多项式回归
概念略，线性回归中的超参数为选取用来拟合的窗口大小N，多项式回归种则还有多项式的阶数degree，在python的

```python
from sklearn.linear_model import LinearRegression # 线性回归
from sklearn.preprocessing import PolynomialFeatures # 多项式回归
```

中可以实现对应的拟合方法


### ARIMA时间序列预测模型
ARIMA模型的全称是差分整合移动平均自回归模型（Autoregressive Integrated Moving Average model）。

在给出ARIMA模型的定义之前，我们先来了解一下时间序列的平稳性。如果一个时间序列${x_t}$满足以下两个条件，则它是弱平稳的：

- 对于所有的时刻$t$，有期望$E[x_t]=\mu$，其中$\mu$是一个常数。
- 对于所有的时刻$t$和任意的间隔$k$，$x_t$和$x_{t-k}$的协方差$\sigma(x_t,x_{t-k})=\gamma_k$，其中$\gamma_k$与时间$t$无关，它仅仅依赖于间隔$k$。这称为方差平稳性。

有关协方差以及相关系数的概念可见：[如何通俗易懂地解释「协方差」与「相关系数」的概念？ - GRAYLAMB的回答 - 知乎](https://www.zhihu.com/question/20852004/answer/134902061)。简单而言，协方差是一个可正可负的值，值越大，说明两个变量同向变化的程度越大，值越小，则两个变量反向变化的程度越大。$\sigma(X,Y)=Cov(X,Y)=E[(X-\mu_x)(Y-\mu_y)]$

我们可以通过时间序列平稳性校验（ADF检验）来判断时间序列是否是平稳的。在Python中，statsmodels和arch都提供了ADF检验：
```python
from arch.unitroot import ADF
result = ADF(pdSeries) # or list
```

$x_t$和$x{t-k}$的相关系数$\gamma_k$称为$x_t$的间隔为$k$的自相关系数。在上述的弱平稳假设下，这个自相关系数和时间$t$无关，其仅仅依赖于间隔$k$。因而对序列的时间序列分析的核心即挖掘该时间序列中的自相关性。

而对于现实世界中的时间序列，我们需要将其分为可预测的部分和不可预测的部分。当原始时间序列为${x_t}$，我们预测的结果为${\hat{x}_t}$，则残差序列${e_t}$定义为${e_t}=x_t-\hat{x}_t$。如果我们的预测模型已经很好地捕捉了原始时间序列的自相关性，那么残差序列应该近似为白噪声。白噪声的定义为：如果一个时间序列满足均值为0，方差$\sigma^2$，且对于任意的$k\ge1$，自相关系数$\rho_k$均为0，则称该时间序列为一个离散的白噪声。对于预测白噪声，我们无能为力，也就是当我们预测结果的残差已经是白噪声了，相当于我们已经做到了极致。

而回到ARIMA的概念上来，提到ARIMA就需要先提到AR（Auto Regression，自回归）模型和MA（Moving Average，移动平均）模型，两者都是用来描述时间序列的：
- 自回归模型强调当前值和历史值之间的关系，用变量自身的历史事件数据对自身进行预测
  
  对一般P阶（即选取历史上P个观测值）的自回归模型有：

  $\hat{x}_t = c + \sum _{i=1}^p{\phi _i x _{t-i}} + \epsilon_t$

  即：$x_t$可以表达为常数值、$t$时刻之前的$p$个观测值的线性组合以及一个$t$时刻的随机误差。

- 移动平均模型强调历史白噪声的线性组合，即可以认为当前的白噪声是历史q阶（即q个）白噪声的的线性组合（移动平均）：

  $\hat{x}_t = \mu + \sum _{i=1}^q{\theta_i \epsilon _{t-i}} + \epsilon_t$

  即：$x_t$可以表达为平均值、$t$时刻之前的$q$个历史白噪声的线性组合以及一个$t$时刻的随机误差。

最终，将上述的两个模型结合，得到预测值同历史p个观测值，q个白噪音有关的$ARMA(p,q)$模型，即自回归滑动平均模型（Autoregressive Moving Average model）：

$\hat{x}_t = c + \sum _{i=1}^p{\phi _i x _{t-i}} + \sum _{i=1}^q{\theta_i \epsilon _{t-i}} + \epsilon_t$ 

即：$x_t$可以表达为常数值、$t$时刻之前的$p$个观测值的线性组合、$t$时刻之前的$q$个历史白噪声的线性组合以及一个$t$时刻的随机误差之和。换句话说：
- AR是自回归，$p$为其中的自回归项数；
- MA为移动平均，$q$为移动平均项数

那么在此基础上添加一个参数$d$来得到最终的$ARIMA(p,d,q)$模型，$d$表示使得要预测的时间序列成为平稳序列所做的差分次数。

因此，要对时间序列使用ARIMA进行预测之前，首先得找到d，即找到序列的0, 1, 2...次差分，使得差分序列为一个时间平稳序列。

```python
from arch.unitroot import ADF
close = train['close']              # 收盘价的0阶差分序列
print(ADF(close).summary())
change = train['change']            # 收盘价的0阶差分序列
print(ADF(change).summary())
```

当然，验证是否为时间平稳序列也是需要备择假设的，因此有把握度的要求，对于预测不准的情况也可以通过调整差分阶数、$p$、$q$的值等超参数来调整模型

```python
# 使用cv值对超参数调参，主要是ARIMA中的阶数order
from statsmodels.api import tsa
order_list = [(3, 1, 0)] # 这里的超参数需要自己给出（属于是炼丹了）
for order in order_list:
    model = tsa.arima.ARIMA(history, order=order)
    # 拟合模型
    # model_fit = model.fit(disp=False)
    model_fit = model.fit()
    # 预测一个数据点
    output = model_fit.forecast()
    ...
```

需要注意的是，普通的差分可以看作是前后元素的差值，实际上还可以转化为计算对数增长率（$\log \frac{x_{i}}{x_{i-1}}$）等形式

### k近邻方法

### 决策树

### 随机森林模型

### 支持向量机SVM

## Deep Learning

### 多层感知器MLP以及前向全联接网络

### 循环神经网络RNN

#### 长短期记忆网络LSTM

