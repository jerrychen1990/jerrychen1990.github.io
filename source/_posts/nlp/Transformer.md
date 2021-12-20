---
title: 《attention is all you need》
date: 2019-05-20
tags: [经典]
categories: nlp
language: ZH
---


[论文链接](https://papers.nips.cc/paper/7181-attention-is-all-you-need.pdf)
- 博客：https://jalammar.github.io/illustrated-transformer/。 原理和结构图在论文以及上面的博客讲的都很清楚，下面提一些我自己阅读论文和博客时遇到的一些疑问，以及后来自己理解觉得对的答案。有些依然没有找到答案。。。
- self-attention
    - 做完key和query向量的点积之后，要除以向量维度的平方根，这样可以保持梯度比较稳定
    - 为什么一定要有一个value向量，不能直接用原始向量替代么？
        - value向量可以表示该token可以被分享到其他token的特征，和该token的embedding不一定一样。而且如果直接用原始embedding作为value的话，self-attention只是等价于之前embedding的重新打散、组合
    - 在做完multi-head之后，把多个head的embedding concat之后还要再接一个Dense层，转化成更低维度传给FFN
<!--more-->

- position embedding
    - 只加在encoder 和decoder最底层的输入中么？ 是的
    - 作用是什么？引入token之间的位置信息：attention机制不能感知token在序列中的位置距离
    - 具体encoding的方案：
    - 这是一个基于token在序列中相对位置做的embedding，所以可以适应任意长度的输入
- FFN层
    - FFN层的作用是什么？每个block除了要做attention，还需要做一些特征变换，在FFN层完成这些事
    - feedforward层的权重是跨encoder decoder block共享的么？ 不是
- residual
    - 每个self-attention层、FFN层、query-attention层都是有residual的
    - 避免梯度消失
    - 原始embedding和过了layer之后的tensor相加之后要过一个BathNormalize层
    - BathNormalize也是有参数的 (θ, std)  现将原始数据归一化到(θ=0,std=1), 然后再变成想要的分布， 参数数=(layer’s dim - 2)
- decoder
    - decoder层的self attention只能attention到当前token前面的token，后面的token会被mask掉
    - decoder输出一个softmax的seq
    - GPT等价于使用transformer的decoder模块（没有给定encoder的信息）   
- 训练
    - 输入输出都是可变长的，由于self-attention层、FFN层、query-attention层都是可以并行计算的，所以不同长度的输入不影响计算。所以只要同一个batch的数据seq_len一样就行了
    - 训练是decoder的输入是待生成的句子，输出是待生成的句子token序列向后错一位的序列
- 预测
    - decoder端输入一个序列，预测的下一个token就是decoder输出的最后一个embedding
    - encoder只需要编码一次，然后把encoder的各层输出给decoder端。 decoder需要计算N次
- git: [https://github.com/tensorflow/tensor2tensor](https://github.com/tensorflow/tensor2tensor)
