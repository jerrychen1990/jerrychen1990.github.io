---
title: '《Beyond Accuracy: Behavioral Testing of NLP Models with CheckList》'
date: 2021-04-15
tags: [Best Paper]
categories: nlp
language: ZH
---

[论文链接](https://arxiv.org/abs/2005.04118)

这篇是ACL2020的最佳论文。论文指出现有的模型效果评估方案的问题，同时借鉴软件测试的方法，提出了一种全新的NLP模型测试方法（个人认为迁移到CV领域也不麻烦）CheckList。这种测试方案可以帮助人们更清晰、系统地了模型各个方面的优缺点。
<!-- more -->

## 目标问题

对于NLP任务，传统的测评方案是在测试集中计算Metrics(Accuracy, F1...)， 这种测试方法有如下缺陷：

- 测试集可能和训练集有同样的Bias, 在测试集上的指标不能准确衡量模型在实际应用时的效果
- 用一个或几个数字衡量模型的效果，不利于发现模型到底在哪个方面有缺陷，也难以指导模型调优
- 存在高估模型能力的可能
  - GLUE榜单上，有些SOTA模型的指标已经接近甚至超越人类了。真的没有提高的空间了么？
  - 模型会走捷径，只是在测试集中能答出正确答案，但是没有真正理解任务。
    ![image.png](/images/byd_acc1.png)
    -     虽然模型回答正确了测试集中的问题"what is the moustache mad of?"。再深入问几个问题，发现模型并没有理解问题，只是狡猾地走了捷径——"只要问有关What的问题，就返回图片中最显眼的一个物件给你"
  - 模型抗扰动性差
    ![image.png](/images/byd_acc2.png)
    - 问题里多加一个问号，模型就不会数数了
  - 模型没有逻辑一致性
    ![image.png](/images/byd_acc3.png)
    - 模型只是学习概率分布，没有背后的逻辑，出现前后矛盾的回答。



<a name="TKfgk"></a>

## 解决方案

借鉴软件工程中的黑盒测试思想，用不同的测试方法测试模型的不同方面的性能。将结果输出成一个表格形式的checklist。
<a name="auIuF"></a>

### 测试目标

测试目标是与目标模型无关的，NLP模型都应该具备的一些基础能力，包括：

- Vocabulary+POS: 理解重要词汇以及词性
- Named entites: 识别出命名实体
- Nagation: 识别出否定词
- Taxonomy: 理解同义词、反义词
- Robustness: 对于typos, 无关紧要的改动的抗干扰性
- Coreference: 理解指代
- Fairness: 不具有性别、种族歧视
- Semantic Role Labeling: 理解语义角色
- Logic: 理解语义中的对称性、一致性等
- Temproal: 理解时态
  <a name="5nUnG"></a>

### 测试方法

对于不同的测试目标，有不同的与之适配的测试方法，本文提出3中测试方法，包括：

- MFT(Minimum Functionality Test)
  - 最小功能测试，类似软件工程中的单元测试
  - 针对某个测试目标，设计一个最简单的针对那个目标的测试用例
  - 需要测试用例设计者知道正确的label
  - 例：对情感分类任务测试Vocabulary+POS能力。This is as great flight -> positive (测试识别"great")
- INV(INVariance Test)
  - 不变性测试, 验证模型对于变化后的测试用例是否输出结果不变
  - 测试人员不需要提前知道正确的label
  - 需要模型前后两次输出结果的label不变且probability 变化不超过0.1
  - 例：对情感分类任务测试Named entites能力
    - 输入1：AmericanAir thank you we got on different flight to Chicago
    - 输入2：AmericanAir thank you we got on different flight to Dallas
    - 前后两次输入只变化了城市实体Chicago -> Dallas。不应该对情感分类模型的结果产生变化
- DIR(DiRectional Expetcation Test)
  - 定向期望测试，期待对样本变化后，模型输出结果的probability往某个方向变化
  - 测试人员不需要提前知道正确的label
  - 例：对情感分类任务，测试Nagation能力
    - 输入1：JetBlue, why won't you help me?! Ugh
    - 输入2：JetBlue, why won't you help me?! Ugh. I dread you.
    - 期待模型能识别出否定词dread, 进而模型输出negative的probability增大
      <a name="8dz6d"></a>

### 测试构建

- 对于待测模型，构建一个由测试目标和测试方法组合而成的checklist
- 对于每一项具体的测试，构建测试用例，评测模型在该项任务上的failure rate
- 对于MFT测试集，可以通过简单的模板构建
- 可以使用预训练的Masked Language Model提示候选词，帮助快速构建测试集
- 使用WordNet等外部知识，做同义词、反义词替换
- 作者开源了测试集构建项目：[https://github.com/marcotcr/checklist](https://github.com/marcotcr/checklist)
  <a name="or2Rl"></a>

## 实验结果

- 在情感分类和重复问题检测两项任务上，对商业模型(微软、谷歌、亚马逊)以及学术界SOTA模型（Bert、Roberta）做check list测试
- 在很多测试任务上，这些模型的failure rate都很高，这是传统的测试集Metrics没能发现的问题
- 一些值得注意的测试结果
  - 对于情感在时态上的变化识别不好: I used to hage this airline, although now I like it -> pos
  - 对于句子结尾的否定识别不好: I thought the plane would be awful, but it wasn't. -> pos/neutral
  - 作者的情感比其他人的重要：Some people think you are ecellent, but I think you are nasty. -> neg
  - 一些语料中的偏见（男性是医生）：John is not a doctor, Mary is. Who is a doctor? -> Mary
- check list对于那些商业用途的，非纯模型的api一样有效
- 利用测试集构建工具，非领域专家人员也能快速构建checklist的测试集。比传统的测试方法更加高效
  <a name="AcYAY"></a>

## 总结与启发

- 测试集上的指标可能会让人高估模型的效果（模型走了捷径）。上线之后对于模型预测结果的抽检是非常有必要的质量监控手段
- checklist的测试可以从不同维度评估模型的能力。从而发现模型在某个NLP通用能力上的缺失。但是如何修复这些问题呢？模型对于我们来说是黑盒的，不能像软件工程测试出程序bug之后改代码来修复。或许只能添加规则，做pipeline来解决这些问题。
- checklist证明了模型学习的时候是急功近利的，尽可能找到一些训练集/测试集上的捷径来降低loss/提高指标。将checklist中列出的NLP模型通用的基础能力(Taxonomy/Temproal...)做成与训练任务，或者加入到目标任务的联合训练中去，是否能提高模型在checklist评估中的效果？
  <a name="YWCg4"></a>

## 相关链接

- [https://zhuanlan.zhihu.com/p/182557001](https://zhuanlan.zhihu.com/p/182557001)
- [https://medium.com/@caiweiwei1005/%E9%98%85%E8%AF%BB%E7%AC%94%E8%AE%B0-beyond-accuracy-behavioral-testing-of-nlp-models-with-checklist-690487be3135](https://medium.com/@caiweiwei1005/%E9%98%85%E8%AF%BB%E7%AC%94%E8%AE%B0-beyond-accuracy-behavioral-testing-of-nlp-models-with-checklist-690487be3135)
