## [《Text Generation from Knowledge Graphs with Graph Transformers》](https://arxiv.org/pdf/1904.02342.pdf)
   - 解决问题：给定一篇论文的title +通过知识抽取工具从论文abstract里抽取出的知识图谱，用生成式模型生成文章的abstract
   - 整体框架：用BiLSTM encode title, 用Graph Transformer encode 知识图谱。 decode过程中同时可以attention到title和图谱的encode特征，同时加上copy机制
   - 细节：graph transformer用节点相邻的节点作为该节点的context，其他和text transformer类似
   - 细节：原始图谱的边是有标签且无向的，在做graph transformer之前，将原始图的边改造成两个节点，这样得到的图的边是有向、无标签的。 同时加上一个global的节点，和所有节点都连接，让整个图联通
   - 数据集：自建数据集，包含40k论文
   - 评价指标：人工打分+BLUE+METEOR
   - 结论：引入知识图谱的模型明显好于只用title的模型，graph transformer也好于Graph Convolution以及Graph Attention的模型
   - 问题：生成的abstract对于图谱中的entity的覆盖率只有60%, 需要更好的机制保证entity的提及
   - git: [https://github.com/rikdz/GraphWriter](https://github.com/rikdz/GraphWriter)