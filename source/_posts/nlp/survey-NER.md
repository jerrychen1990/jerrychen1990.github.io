## [A Survey on Deep Learning forNamed Entity Recognition](https://arxiv.org/pdf/1812.09449.pdf)
- NER是信息抽取、问答系统、机器翻译的一项基础工作，DNN的应用让NER任务有了长足的进步
- NER分为两类coarse-grained NER:比较粗粒度的划分entity，比如通用NER。 fine-grained NER：更加细分的实体类型，通常是和具体的业务相关的实体，一个mention可以属于多个实体类别
- 数据集：见原文table1。 比较常用的有：
    - OntoNotes：18 coarse entity type consisting of 89 subtype
    - CoNLL03 4 entity types
- 工具：见原文table2 StanfordCoreNLP/NLTK/spaCy
- 评价指标：
    - exact-match evaluation： 用全匹配方法计算F1。会有些偏严，指标偏低
    - relaxed-match evaluation: 宽松匹配方案，不太好控制
- 传统方法：
    - 基于规则的方案
    - Unsupervised方案：用clustering的方法学习一些统计信息，帮助规则的方案识别NE
    - Feature-Based supervised learning: 基于手工特征提取
- DNN方法：
    - Distributed representation
        - word-level: Word2Vec, fastText
        - Character-level Representation: CNN of Character
        - Hybrid Representation
    - Context Encoder Architecture
        - CNN
        - RNN(Recurrent Neural Networks)
        - Recursive Neural Networks: 在树状空间上递归展开（RNN可以看做recursive NN在横向空间展开的特例）
        - Transformer
    - Tag Decoder Architecture
        - FNN + softmax
        - CRF(CRF是HMM更加泛华的一种形式)
        - RNN： 用RNN以language model的方式解码（输入当前token的encode embedding以及上一个状态的tag），据说当entity_type比较多的时候，比CRF快。但是CRF可以求全局最优解，RNN只是从左到右用greedy算法求解，似乎CRF好一些
        - Pointer Networks：RNN decoder的结构来划分句子中的span，而后再用分类模型对每个span打上entity标签，不需要BIO标签格式
    - 最常用的搭配:Hybrid Representation+RNN+CRF
- 其他方向：
    - Deep Multi-task learning for NER
        - joint train NER POS CHUNK
        - joint train NER and NRE
        - train NER as a joint of entity-segmentation and entity classification
    - Deep Active Learning for NER
    - Deep Reinforcement Learning for NER
    - Deep Adversarial Learning for NER
    - Deep Attention for NER
- Challenges
    - Data Annotation
    - Informal Text and Unseen Entities
- Feature Directions
    - Boundary Detection
    - Joint NER and Entity Linking
    - 将DL的NER和辅助信息、辅助方法结合起来
    - scalability: 现有的模型已经很大了，难以生产中落地
    - deep transfer learning
        - 训练可以跨语言、跨领域的模型
        - few-shot学习（MTB的研究方向）
        - 解决domain-mismatch的方式
    - 方便使用的DL-based NER
    
         
        
        
            
            
            
        
            
            
        
    
        
       
    