---
title: On the Security and Applicability of Fragile Camera Fingerprints
tags: paperreading
# article_header:
#   type: cover
#   image:
#     src: /assets/images/helloworld.jpg
---

<!-- write excerpt here -->
On the Security and Applicability of Fragile Camera Fingerprints 论文笔记

<!--more-->


作者：Erwin Quiring1(B), Matthias Kirchner2(B), and Konrad Rieck1(B) @ TU Braunschweig, Braunschweig, Germany , Binghamton University, Binghamton, USA

## 摘要

相机传感器噪声是数字图像取证中最可靠的设备特性之一，可实现图像与数字相机的独特链接。这种所谓的相机指纹会引起不同的应用，例如图像取证和身份验证。但是，如果可以公开获得图像，则对手可以估算受害者的指纹并将其植入虚假图像中。脆弱的相机指纹概念通过利用数据访问中的不对称性来解决这种攻击：虽然相机所有者将始终可以从未压缩的图像访问完整指纹，但对手通常只能访问压缩的图像，因此只能访问被截断的指纹。但是，这种防御的安全性尚未得到系统地探讨。本文对受到攻击的易碎相机指纹进行了首次全面分析。一系列的理论和实践测试表明，易碎的相机指纹可以在对抗环境中为常见的压缩级别提供可靠的设备识别。

## 背景

数码相机传感器的最小，不可避免的制造缺陷会导致光响应非均匀性（PRNU）信号，这是一种非常独特且可检测的相机设备特性[8]。与健壮的数字水印相似，PRNU信号在同一台摄像机拍摄的任何图像中都很少出现，但不同摄像机的图像之间却有所不同。这些特性使PRNU成为自然的相机指纹。它已经在取证中得到广泛应用，将数字图像归因于其源相机[8]。最近的工作还提出使用PRNU作为将移动设备身份验证链接到移动设备的固有硬件特性的方法[2,26]。
然而，实际上，这些用例面临着指纹复制攻击的问题[10,19]。如果爱丽丝与公众共享相机中的图像，则马洛里可以估算爱丽丝的指纹，将其植入自己的图像中，并假装爱丽丝的相机捕获了任意图像。所谓的三角测试[13]可事后检测到此类攻击，但可能需要对爱丽丝共享的所有公共图像进行详尽搜索。 Quiring和Kirchner [23]最近提出了一种基于脆弱的相机指纹概念的主动防御技术，以确保可以从高质量（未压缩）图像中识别相机。在这里，摄像机所有者爱丽丝（Alice）可以通过仅与公众共享JPEG压缩图像，同时将其未压缩图像保留为私有来利用可访问数据质量的不对称性。结果，当被要求时，她将始终能够从高质量图像中提供完整的指纹。相反，马洛里（Mallory）从公共JPEG图像中对爱丽丝（Alice）指纹的估计将仅包含对有损JPEG压缩具有鲁棒性的部分，而缺少易碎的分量。然后通过测试是否存在脆弱的指纹，可以防止Mallory使未压缩的图像看起来像Alice的未压缩图像之一。

## 本文主要贡献

alice为拥有相机的人，保存有自己拍摄的原图， 向公众发布很多张jpeg压缩图，malloy则是能通过访问alice所有公开照片来进行噪声伪造的攻击者。



首先，我们从分析上推导出爱丽丝和马洛里的指纹估计之间相对于马洛里可以访问的公开共享图像的JPEG质量的上限。其次，为了测试线性相关性以外的依赖性，内核统计测试用于评估Alice的脆弱指纹是否在统计上独立于Mallory的指纹。第三，我们证明了从潜在的剩余依赖关系中恢复量化的JPEG系数的实际尝试不会增加Mallory发起成功攻击的能力。第四，我们测试了易碎指纹对实际指纹复制攻击的抵抗力。最后，我们说明了脆弱的指纹和三角形测试是司法鉴定应用程序中的强大组合。
