# ann benchmark


## 参与评估的算法 2020-07-12
* [Annoy](https://github.com/spotify/annoy) 
* [FLANN](http://www.cs.ubc.ca/research/flann/)
* [scikit-learn](http://scikit-learn.org/stable/modules/neighbors.html): LSHForest, KDTree, BallTree
* [PANNS](https://github.com/ryanrhymes/panns)
* [NearPy](http://pixelogik.github.io/NearPy/)
* [KGraph](https://github.com/aaalgo/kgraph)
* [NMSLIB (Non-Metric Space Library)](https://github.com/nmslib/nmslib): SWGraph, HNSW, BallTree, MPLSH
* [hnswlib (a part of nmslib project)](https://github.com/nmslib/hnsw)
* [RPForest](https://github.com/lyst/rpforest)
* [FAISS](https://github.com/facebookresearch/faiss.git)
* [DolphinnPy](https://github.com/ipsarros/DolphinnPy)
* [Datasketch](https://github.com/ekzhu/datasketch)
* [PyNNDescent](https://github.com/lmcinnes/pynndescent)
* [MRPT](https://github.com/teemupitkanen/mrpt)
* [NGT](https://github.com/yahoojapan/NGT): ONNG, PANNG
* [SPTAG](https://github.com/microsoft/SPTAG)
* [PUFFINN](https://github.com/puffinn/puffinn)
* [N2](https://github.com/kakao/n2)
* [ScaNN](https://github.com/google-research/google-research/tree/master/scann)

本次测试结果可以在[ann-benchmarks](http://ann-benchmarks.com/index.html)查看

## tradeoff

数据\算法 | [hnsw-faiss](http://ann-benchmarks.com/hnsw(faiss).html)|[nmslib-faiss](http://ann-benchmarks.com/hnsw(nmslib).html)|[hnswlib](http://ann-benchmarks.com/hnswlib.html)|[NGT-panng](http://ann-benchmarks.com/NGT-panng.html)|[NGT-onng](http://ann-benchmarks.com/NGT-onng.html)|[n2](http://ann-benchmarks.com/index.html)|[scann](http://ann-benchmarks.com/index.html)|[annoy](http://ann-benchmarks.com/index.html)|[SW-graph](http://ann-benchmarks.com/index.html)|[faiss-ivf](http://ann-benchmarks.com/index.html)|[mrpt](http://ann-benchmarks.com/index.html)|[flann](http://ann-benchmarks.com/index.html)|[puffinn](http://ann-benchmarks.com/index.html)|[bruteforce-blas](http://ann-benchmarks.com/index.html)|[BallTree(nmslib)](http://ann-benchmarks.com/index.html)|[pynndescent](http://ann-benchmarks.com/index.html)|[kgraph](http://ann-benchmarks.com/index.html)|[kd](http://ann-benchmarks.com/index.html)|[sptag](http://ann-benchmarks.com/index.html)|
---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---
[glove-100-angular_10_angular](http://ann-benchmarks.com/glove-100-angular_10_angular.html)   |---|---|---|---|---|---|1|---|---|---|---|---|---|---|---|---|---|---|---
[glove-25-angular_10_angular](http://ann-benchmarks.com/glove-25-angular_10_angular.html)|---|1|---|---|1|---|---|---|---|---|---|---|---|---|---|---|---|---|---
[lastfm-64-dot_10_angular](http://ann-benchmarks.com/lastfm-64-dot_10_angular.html)|---|1|---|---|---|1|---|---|---|---|---|---|---|---|---|---|---|---|---
[nytimes-256-angular_10_angular](http://ann-benchmarks.com/nytimes-256-angular_10_angular.html)|---|---|---|---|1|---|---|---|---|---|---|---|---|---|---|---|---|---|---
[fashion-mnist-784-euclidean_10_euclidean](http://ann-benchmarks.com/fashion-mnist-784-euclidean_10_euclidean.html)|---|---|---|---|1|---|---|---|---|---|---|---|---|---|---|1|---|---|---
[gist-960-euclidean_10_euclidean](http://ann-benchmarks.com/gist-960-euclidean_10_euclidean.html)|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---
[sift-128-euclidean_10_euclidean](http://ann-benchmarks.com/sift-128-euclidean_10_euclidean.html)|---|1|1|1|1|1|---|---|---|---|---|---|---|---|---|1|---|---|---

## 各个算法介绍
### [ScaNN](https://github.com/google-research/google-research/tree/master/scann) 
**算法提出者**:谷歌
**语言**:底层使用c++实现，很好的支持了python使用，
**算法介绍**:基于量化模型，算法基本结构采用了PQ的结果，不过提出了一种通过计算损失函数来替换PQ中量化的方法。
**算法性能**: 建索引快，内存占用高，搜索速度快，召回高
**算法效果**:在glove-100-angular数据集上性能最好，看代码支持各种距离度量方式，包括但不限于余弦、点积、L1、L2、hamming，但是测试结果里当数据集的维度大于100或者距离计算方式为欧氏距离的时候，都没有scaNN算法的结果，推测该算法在低维和使用余弦度量时性能好。

[ScaNN算法文档](https://github.com/google-research/google-research/blob/master/scann/docs/algorithms.md)
[ScaNN论文地址](https://arxiv.org/abs/1908.10396)


### [NGTONNG](https://github.com/yahoojapan/NGT)
**算法提出者**: Yahoo Japan Corporation
**语言**:底层使用c++实现，支持python、ruby、go、c和c++
**算法介绍**:基于图模型，提出三种优化节点的入度和出度方法和一种路径优化方法，在多个数据集表现比较好
**算法性能**: 建索引慢，内存占用超高，搜索速度快，召回高
**算法效果**:在多个数据集表现良好，但是在sift-128数据集召回率最高只有0.78。

[NGT-onng论文地址](https://arxiv.org/abs/1810.07355)

### [N2](https://github.com/kakao/n2)
**算法提出者**: 实现了HNSW算法
**语言**:底层使用c++实现，支持python、go和c++
**算法介绍**: HNSW算法
**算法性能**: 建索引比nmslib快，内存消耗比nmslib低，速度比nmslib快


### [PyNNDescent](https://github.com/lmcinnes/pynndescent)

**算法提出者**: 普林斯顿大学
**语言**: python，代码主要通过python的科学计算库scikit-learn, numpy, scipyhe numba实现科学计算
**算法介绍**: 
**算法性能**: 

[PyNNDescent论文](https://www.cs.princeton.edu/cass/papers/www11.pdf)