---
title: Hung-yi Lee AI Lesson
tags: AI
# article_header:
#   type: cover
#   image:
#     src: /assets/images/helloworld.jpg
---

<!-- write excerpt here -->
Hung-yi Lee的AI课（2021version）笔记

<!--more-->

## Introduction of ML
1. 机器学习的几种常见类别：
   - Regression：输出为标量（scalar），如预测明日的PM2.5含量
   - Classification：给定一些备选类别，让机器从中选择符合的，如垃圾邮件的判断；下棋从这个定义上而言，实际上也是一种classification，相当于当前每个能下的位置为分类当中的一个类别
   
   过去人们，普遍认为就上面两大类，但时至今日，人们发现实际上还有诸如structured learning（创造结构化的输出，如图片、文档等）等类别，这也代表人们对ML的越来越高的期望。
2. 常用的一些术语：model, feature, weight, bias, loss, label
3. MAE(mean average error) & MSE(mean square error)是loss function中的一些常用loss计算方式。对于概率预测，还有交叉熵(cross entrophy)可以作为loss function
4. 梯度下降法（gradient descent）对于model中的多个参数，现选取一个起始点，再逐个参数进行梯度下降。对于梯度下降的步伐大小，一部分取决于该点的偏微分斜率大小，一部分取决于自定义的一个常数$\eta$，而这个常数就是模型的超参数了
5. 总结来看，训练包括三个步骤：
   1. 建立有未知数的方程
   2. 根据训练数据设定loss function
   3. （使用梯度下降法）优化
6. 由于model自身不能很好描述情况的限制被称为model的bias
7. 任意的函数都可以表示/近似表示成**一个常数加一系列激活函数的集合**，如：
   <div align=center>
   <img width=500 src="/assets/images/mdimages/machinelearning1.png"/>
   </div>
   这种一个常数加一系列激活函数的集合的函数曲线形式称为piecewise linear curves
8. 首先介绍的激活函数便是sigmoid函数，使用该函数来近似表示piecewise linear curves为
   $$y = b + \sum_ic_isigmoid(b_i+w_ix_i)$$

   如果考虑多个特征的输入（也就是y取决于一系列$x_1, x_2, ... x_j$的输入），则该式变换为：
   $$y = b + \sum_ic_isigmoid(b_i+\sum_jw_{ij}x_j)$$

   也就是有如下的形式（取j=3，即3个输入特征时）
   <div align=center>
   <img width=500 src="/assets/images/mdimages/machinelearning2.png"/>
   </div>

   这样就有了MLP中的输入层形式。总体的形式如下：
   <div align=center>
   <img width=500 src="/assets/images/mdimages/machinelearning3.png"/>
   </div>

   需要注意的是，上面的图只是一个示例，实际上网络的结构不光取决于输入的特征数量，其中的激活函数的数量也是可以作为超参数来自由定义的（而不只是像图中这样和输入的特征数量一致）这实际上就形成了一个网络结构。其中的$\boldsymbol{x}$为特征feature，$W, \boldsymbol{b}, \boldsymbol{c}^T, b$为未知参数unknown parameters，将未知参数中的每一个具体的参数值拼成一个长的向量，这个向量使用$\theta$来表示。相应的，loss function就可以表示成$L(\theta)$的形式，因为其大小完全取决于unknown parameters。

   对于$\boldsymbol{\theta}=\begin{bmatrix}\theta_0 \\\  \theta_1 \\\ ... \end{bmatrix}$，取loss function $L$对每个参数的偏函数在$\boldsymbol{\theta}_0$处的值，组成在点$\boldsymbol{\theta}^0$处的斜率$\boldsymbol{g}$：

   <div align=center>
   <img width=500 src="/assets/images/mdimages/machinelearning4.png"/>
   </div>

   其中的$\eta$则为步长。
9. 在实际的应用中，我们通常不会拿所有的数据来计算损失函数的值，而是将所有的数据数据进行任意随机等大小的分组（即batch）。进行梯度下降时，先算batch1的loss function，更新一次权重之后，再拿batch2的数据算一次loss function，更新权重，直到遍历完所有batch完成一次epoch。也就是有update的次数等于batch个数乘上epoch轮数
10. 除了sigmoid之外，ReLU(Rectified Linear Unit)也是常用的激活函数。使用其表示时，有变式：$y = b + \sum_ic_isigmoid(b_i+\sum_jw_{ij}x_j)$变为了$y = b + \sum_{2i}c_imax(0, b_i+\sum_jw_{ij}x_j)$。其中ReLU使用2倍的i是因为ReLU需要两个才能拼出一个hard sigmoid形状
11. 除了改变激活函数之外，还可以增加层数：
   <div align=center>
   <img width=500 src="/assets/images/mdimages/machinelearning5.png"/>
   </div>
   也就是将前一层的激活函数的输出再作为新的特征输入给下一层进行迭代。
12. 经验上而言，层数的递增的确能提高正确率，有：
    1.  AlexNet(2012)：8层，16.4%错误率
    2.  VGG(2014)：19层，7.3%错误率
    3.  GoogleNet(2014)，22层，6.7%错误率
    4.  Residual Net(2015)，152层，3.56%错误率。其训练深度网络时还通过特殊的网络结构来进行了优化。
13. 过于复杂的结构（如选取过多特征、或者网络层数越大）可能导致在训练集上效果更好，但测试集上反而效果变差的情况，这种情况便称为**overfitting过拟合**

## Roadmap of Improving Model
