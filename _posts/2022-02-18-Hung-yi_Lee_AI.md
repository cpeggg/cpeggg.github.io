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
   - Classification：给定一些备选类别，让机器从中选择符合的，如垃圾邮件的判断；下棋从这个定义上而言，实际上也是一种classification
   
   过去人们，普遍认为就上面两大类，但时至今日，人们发现实际上还有诸如structured learning（创造结构化的输出，如图片、文档等）等类别，这也代表人们对ML的越来越高的期望。
2. 常用的一些术语：model, feature, weight, bias, loss, label
3. MAE(mean average error) & MSE(mean square error)是loss function中的一些常用loss计算方式
4. 梯度下降法（gradient descent）对于model中的多个参数，现选取一个起始点，再逐个参数进行梯度下降。对于梯度下降的步伐大小，一部分取决于该点的偏微分斜率大小，一部分取决于自定义的一个常数$\eta$，而这个常数就是模型的超参数了

## 