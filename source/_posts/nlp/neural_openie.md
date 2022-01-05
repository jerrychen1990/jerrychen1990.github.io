---
title: 《Neural Open Information Extraction》
date: 2020-07-02
tags: [OpenIE]
categories: nlp
language: ZH
---

[论文链接](https://arxiv.org/pdf/1805.04270.pdf)

- 目标：从输入文本中抽取schema-free的spo三元组
- 模型：
    - encoder-decoder的seq2seq模型
    - 原文输入encoder，得到一个encoded embedding
    - 目标序列格式为<arg1>subject</arg1><rel>predication</rel><arg2>object</arg2>
    - 引入copy机制，从生成的token和copy的token中选择一个
    - architecture:![architecture](/images/neural_openie.png)
- 实验：
    - 数据
        - 训练数据从wikipedia的dump构建，36,247,584 pairs,地址：[https://1drv.ms/u/s!ApPZx_TWwibImHl49ZBwxOU0ktHv](https://1drv.ms/u/s!ApPZx_TWwibImHl49ZBwxOU0ktHv)
        - 测试数据：3200 sentence with 10369 extractions [https://www.aclweb.org/anthology/D16-1252.pdf](https://www.aclweb.org/anthology/D16-1252.pdf)
        - 比较对象：OpenIE4(一个基于规则的提取器)
        - 结果：更高的AUC
<!--more-->
- 启发：
    - 做openie的工作，不仅仅可以用NER+NRE的传统方案，还可以考虑seq2seq的方案
    - seq2seq需要引入copy机制确保生成原文中出现的词语
    - 这种方案可以解决predicate不是原文中连续的一段文本的情况
    - 可以引入transformer来做生成，提高效果
    - 在做decode的时候，可以引入图谱embedding，ner embedding，提高效果
    - 后续可以改进做一个句子中有多个SPO的抽取
