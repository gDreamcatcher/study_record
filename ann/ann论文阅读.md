# 论文阅读

## HASH方法

### A Revisit of Hashing Algorithms for Approximate Nearest Neighbor Search

**摘要**：重新测试了hash索引，对11个现存的hash索引做了详细的对比， 发现random-projection-based Locality Sensitive Hashing(RPLSH)算法效果很好，并把测试代码放在了[github](https://github.com/ZJULearning/RPLSH)上

**实验**：

1. 多个hash索引测试

   结论：在sift1M上，code长度是1024的时候LSH的表现最优，再增加code的长度对搜索的表现提升不高

2. RPLSH和faiss的PQ、annoy、Falconn、kgraph和flann做对比

   结论：sift1M的数据集kGraph效果更好，在gistIm的数据集KPLSH效果更好

点评：

该论文只是对hash索引做了一次比较全面的对比实验，总结出LSH的效果在所有的hash索引中效果比较好，然而实验二中对比的都是annbenchmark上一些表现不好的模型，目前感觉还是不可用于生产中。