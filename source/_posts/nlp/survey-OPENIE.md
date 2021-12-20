---
title: 《A Survey on Open Information Extraction》
date: 2020-01-17
tags: [OpenIE, survey]
categories: nlp
language: ZH
---

[论文链接](https://arxiv.org/pdf/1806.05599.pdf)
- information extraction从文本中抽取出SPO三元组,传统的information extraction都是抽取事先给定的关系
- Open information extraction(Open IE)的关系无需实现给定，能够自动从大量的文本中发掘出关系(关系可能是原文中的span，也可能不是)
- OPEN IE的三个挑战：
    - Automation：需要手动标注的数据必须限制在较小的数量级
    - Corpus Heterogeneity:能在不同分布的数据集上work，不能依赖领域相关的信息，比如NER。只能用POS这些浅层tag
    - Efficiency：需要在大量数据上运行，需要预测性能高，只能依赖POStag这些浅层信息
- OPEN IE的方法：
    - Learning-based System: TEXTRUNNER/WOE/OLLIE
    - Rule-based System:利用语言学、统计学特征+规则 PredPatt
    - Clause-based System: Stanford OpenIE
    - Systems Capturing Inter-Proposition Relationships: 同时抽取三元组以及原文中三元组成立的前提
<!--more-->
- Evaluation:
    - 目前还没有特别统一的测评数据集以及测评方案
    - [Analysing Errors of Open Information Extraction System](https://arxiv.org/pdf/1707.07499.pdf) 在做通用测评数据、指标的尝试
- Open Research Questions:
    - 标准测评数据集以及指标
    - 英语之外的其他语言的openIE system
    - 指代消解技术的引入
    - 规范化定义relation和arguments的方案
- 基于NN的Open IE方案:
    - [Neural Open Information Extraction](https://arxiv.org/pdf/1805.04270.pdf)
