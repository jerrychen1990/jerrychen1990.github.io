---
title: 《Supervised Open Information Extraction》
date: 2020-07-02
tags: [OpenIE]
categories: nlp
language: ZH
---

[论文链接](https://www.aclweb.org/anthology/N18-1081.pdf)
  - 目标：构建一个基于监督学习的openie
  - 建模方式：sequence-labeling
  - 输入：token序列+token的POS信息+基于SRL的predicate开头token的信息
  - 输出：BIO方式标注的predicate ARG0 ARG1 ARG2标签
    - ARG0表示subject ARG1表示object ARG2表示spo的附加条件（比如时间、地点、情景等）
    - 这里的object定义比较灵活，可以不是一个实体
    - 每个token输出一个probability，span的probability由包含的所有token的probability相乘得到。作者验证相乘的方式是最好的计算span probability的方案
<!--more-->
  - 数据集构建：
    - 基于QA-SRL任务数据集转换
    - 基于QAMR任务数据集的转换
    - openie和QA-SRL的区别：SRL的predicate通常是单个动词，openie则更丰富，可以是多个词
    - 评价方式：利用更宽松的评价指标，只要s,p,o包含对应SRL的头token就可以了
  - 结论：
    - 大多数openie依然存在低recall的问题
    - 提供一个Spo的probability值，让模型的使用者根据具体业务选择threshold是很重要的一个特性
    - 引入QA-SRL的信息对做openie的任务很有帮助
    - 引入QAMR任务的数据、信息到openie任务中，是一个未来发展的方向
  - git地址：[https://github.com/gabrielStanovsky/supervised-oie](https://github.com/gabrielStanovsky/supervised-oie)
