---
title: 主动学习的一些最新研究进展
date: 2018-12-17 15:04:18
updated: 2018-12-17 15:04:18
categories:
- 机器学习

tags:
- 学习总结
---
# 前言
之前翻译过关于主动学习一些基础概念。通过这段时间的学习，对这方面的研究有所增加。总的来说，主动学习是一项不错的技术，能够很好的解决一些成本和性能之间的平衡问题（Cost-Effective）以及关键样本的挑选问题（CoreSet-Selection）。但是。它的一些挑战也急于得到有效的解决。

<!-- more -->
# 主要挑战
1. 重复训练：涉及到每次问询过后，需要对新增加的样本进行学习，如何减少这种冗余？
2. 标注的时间差：训练的时间与问询等待的时间是否满足现实场景？**目前来说，均缺少这方面的考虑**
3. 度量的有效性证明：**最为关键，也是目前研究的出发点**

# 新的研究进展
## 结合深度网络
- [KDD-2018 Cost-Effective Training of Deep CNNs with Active Model Adaptation](https://arxiv.org/abs/1802.05394): 引入了新的度量方法：Distinctiveness。

- [CVPR-2018 The power of ensembles for active learning in image classification](http://openaccess.thecvf
.com/content_cvpr_2018/papers/Beluch_The_Power_of_CVPR_2018_paper.pdf)：引入了类似集成学习的思想，并且利用深度网络进行主动学习。

## 结合增强学习与元学习
- [NIPS-2016 Active One Shot Learning](https://arxiv.org/abs/1702.06559)：开创增强学习进行自适应学习度量策略的先河。

# 结束语
这也是目前本人的改进方向，特别是增强学习方面。
