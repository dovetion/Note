# 论文阅读笔记

[TOC]

---

## Magnetic Nanoparticle Based Molecular Communication in Microfluidic Environments

### 内容 & 观点 & 方法 提炼

* 在理论上提出了一种磁性纳米分子(MNPs)
* Communication theoretic frameworks have already proven useful for analyzing natural MC systems [5].  

  * F. Tostevin and P. R. Ten Wolde, “Mutual information between input and output trajectories of biochemical networks,” Phys. Rev. Lett., vol. 102, no. 21, p. 218101, 2009. 
* 蛋白质分子通常作为信息载体，布朗运动，液体流动会减少receriver收到的通信分子。
* 通过用**磁性分子MNPs和外部磁场**解决上述问题。
* MNPs are already **widely used**[9],[10]
  - Q. A. Pankhurst, N. T. K. Thanh, S. K. Jones, and J. Dobson, “Progress in applications of magnetic nanoparticles in biomedicine,” J. Phys. D: Appl. Phys., vol. 42, no. 22, pp. 1–15, Nov. 2009
  - J. Zaloga, C. Janko, J. Nowak, J. Matuszak, S. Knaup, D. Eberbeck, R. Tietze, H. Unterweger, R. P. Friedrich, S. Duerr et al., “Development of a lauric acid/albumin hybrid iron oxide nanoparticle system with improved biocompatibility,” Int. J. Nanomed., vol. 9, p. 4847, 2014.

* 有些细菌具有天然趋磁特性，而合成的MNPs通常是被聚合物覆盖的超顺磁性铁氧纳米分子（**magenetic core & non-magenetic coating** ）

  >**Superparamagnetism** is a form of magnetism which appears in small ferromagnetic or ferrimagnetic nanoparticles. In sufficiently small nanoparticles, magnetization can randomly flip direction under the influence of temperature. The typical time between two flips is called the Néel relaxation time.

  coating来保障biocompatibility和stability，防止纳米分子凝聚，与其他分子发生反应。

* 原文，[Wiki-梯度]<https://zh.wikipedia.org/wiki/%E6%A2%AF%E5%BA%A6>

  > the magnetic force crucially depends on the magenetic field gradient rather than the magnitude of the magenetic field

* MNPs可以被attach在细胞内基因里(DNA?)[16]，装载药物[17]，attach在钙铁离子通道上[18]，细胞表面[19]，用来驱动趋磁细菌[20]，且MNPs能被磁镊精确控制，也能被精确地biosensing到。

* MC中MNPs只在第[6], [23], [24]中考虑到，[23]被attach到gene上但是没有提供数学分析。
