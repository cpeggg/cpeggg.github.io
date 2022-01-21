---
title: Encryption Algorithm
description: 
tags:
 - crypto
 - ongoing
---
分组加密模式总结
<!--more-->
## 分组加密算法中的加密模式
### ECB模式
![](http://images.cnblogs.com/cnblogs_com/happyhippy/1ECB.jpg)
优点:
1.简单；
2.有利于并行计算；
3.误差不会被传送；
缺点:
1.不能隐藏明文的模式；
2.可能对明文进行主动攻击（重放）；
### CBC模式
![](http://images.cnblogs.com/cnblogs_com/happyhippy/2CBC.jpg)
优点：
1.不容易主动攻击,安全性好于ECB,适合传输长度长的报文,是SSL、IPSec的标准。
缺点：
1.不利于并行计算；
2.误差传递；
3.需要初始化向量IV
### CFB/OFB模式
![](http://images.cnblogs.com/cnblogs_com/happyhippy/3CFB.jpg)
![](http://images.cnblogs.com/cnblogs_com/happyhippy/4OFB.jpg)
对于OFB模式（上图靠下），对明文的主动攻击是可能的
优点：
1.隐藏了明文模式
2.分组密码转化为流模式;
3.可以及时加密传送小于分组的数据;
缺点:
1.不利于并行计算;
2.误差传送：一个明文单元损坏影响多个单元;
3.唯一的IV;
##AES对称加密
http://www.formaestudio.com/rijndaelinspector/archivos/Rijndael_Animation_v4_eng.swf
##RSA公钥加密
在公开密钥密码体制中，加密密钥（即公开密钥）PK是公开信息，而解密密钥（即秘密密钥）SK是需要保密的。加密算法E和解密算法D也都是公开的。虽然解密密钥SK是由公开密钥PK决定的，但却不能根据PK计算出SK。
RSA的算法涉及三个参数，n、e1、e2 (e1,e2有时也记作e,d)。
其中，n是两个大质数p、q的积，n的二进制表示时所占用的位数，就是所谓的密钥长度。
e1和e2是一对相关的值，e1可以任意取，但要求e1与(p-1)*(q-1)互质；再选择e2，要求(e2*e1)mod((p-1)*(q-1))=1。
其中(p-1)*(q-1)的值就是n的欧拉函数值
（n，e1),(n，e2)就是密钥对。其中(n，e1)为公钥，(n，e2)为私钥。
RSA加解密的算法完全相同，设A为明文，B为密文，则：A=B^e2 mod n；B=A^e1 mod n；（公钥加密体制中，一般用公钥加密，私钥解密）
e1和e2可以互换使用，即：
A=B^e1 mod n；B=A^e2 mod n;
### 待补充：GCM等