---
layout: post
title: "ID3 算法"
date: 2013-01-15 19:58
comments: true
categories: machine_learning
---

#### 简介

决策树学习是一种逼近离散值目标函数的方法，在这种学习到的函数被表示为一棵决策树。

#### 决策树

决策树通过把实例从根节点排列到某个叶子结点来分类实例，叶子结点即为实例所属的分类。树上的每一个结点指定了对实例的某个属性的测试，并且该结点的每一个后续分支对应于该属性的一个可能值。

决策树的分裂方法是从根节点开始，测试节点的属性，通过节点的属性的值来决定子叶的数量，然后子叶再重复上面的过程，继续向下划分子叶。

![decision_tree]({{ site.url }}/images/post/decision_tree.jpg)

数据可以从 [Information Gain](/algorithm/2012/11/17/information-gain.html) 里面查看。

这个样例数据根据天气决定是否要出去玩。比如函数的输入是下面这个

    Outlook=Sunny，Temperature= Hot，Humidity = High ，Wind= Strong

沿着这棵决策树的最左分支向下，被评定为反例（也就是这棵树预测这个实例 Play = No）。

这棵树以用来演示 ID3 学习算法的例子摘自 [Quinlan 1986](http://www.dmi.unict.it/~apulvirenti/agd/Qui86.pdf "Title")。


#### 决策树的ID3算法

基本的ID3 算法通过自顶向下构造决策树来进行学习。

构造过程是从“哪一个属性将在树的根结点被测试？”这个问题开始的。

为了回答这个问题，使用统计测试来确定每一个实例属性单独分类训练样例的能力。

（1）分类能力最好的属性被选作树的根结点的测试。

（2）然后为根结点属性的每个可能值产生一个分支，并把训练样例排列到适当的分支（也就是，样例的该属性值对应的分支）之下。

（3）然后重复整个过程，用每个分支结点关联的训练样例来选取在该点被测试的最佳属性。

这形成了对合格决策树的贪婪搜索（greedy search ），也就是算法从不回溯重新考虑以前的选择。

#### 哪个属性是最佳的分类属性

ID3 算法的核心问题是选取在树的每个结点要测试的属性。

我们希望选择的是最有助于分类实例的属性。那么衡量属性价值的一个好的定量标准是什么呢？

这里将定义一个统计属性，称为“信息增益 [Information Gain](/algorithm/2012/11/17/information-gain.html)”，用来衡量给定的属性区分训练样例的能力。

ID3 算法在增长树的每一步使用这个信息增益标准从候选属性中选择属性。


#### 用熵度量样例的均一性

为了精确地定义信息增益，我们先定义信息论中广泛使用的一个度量标准，称为熵 [Entropy](/algorithm/2012/12/07/information-theory.html)，它刻画了任意样例集的纯度（ purity）。

给定包含关于某个目标概念的正反样例的样例集S ，那么S 相对这个布尔型分类的熵为：

$$Entropy(S) = -P _{positive} log _2 P _{positive} - P _{negative} log _2 P _{negative}$$

其中 $$P _{positive}$$ 是在S 中正例的比例，$$P _{negative}$$是在S 中负例的比例。在有关熵的所有计算中我们定义0 log0 为0 。

举例说明，假设S 是一个关于某布尔概念的有 14 个样例的集合，它包括 9 个正例和5 个反例（我们采用记号[9+ ，5 -]来概括这样的数据样例）。

那么S 相对于这个布尔分类的熵（Entropy）为：

$$Entropy([9+,5-]) =-\frac{9}{14} log _{2} (\frac{9}{14}) -\frac{5}{14} log _{2}(\frac{5}{14})$$

至此我们讨论了目标分类是布尔型的情况下的熵。更一般的，如果目标属性具有 n 个不同的值，那么S 相对于 n 个状态的分类的熵定义为：

$$Entropy(S) = \sum _{i=0} ^{n} {-P _{i} log _{2} P _{i} }$$

#### 用信息增益度量期望的熵降低

已经有了熵作为衡量训练样例集合纯度的标准，现在可以定义属性分类训练数据的效力的度量标准。这个标准被称为“信息增益[Information Gain](/algorithm/2012/11/17/information-gain.html)”。

简单的说，一个属性的信息增益就是由于使用这个属性分割样例而导致的期望熵降低。更精确地讲，一个属性A 相对样例集合 S 的信息增益Gain(S, A)被定义为：

其中 Values (A) 是属性A所有可能值的集合，SV 是 S 中属性A的值为 v 的子集（也就是，Sv ={s∈ S \|A(s )=v } ）。

请注意，等式的第一项就是原来集合 S 的熵，第二项是用 A 分类 S 后熵的期望值。

这个第二项描述的期望熵就是每个子集的熵的加权和，权值 \|Sv\|/\|S\| 为属于 S 的样例占原始样例 S 的比例 。

所以 Gain(S , A) 是由于知道属性A的值而导致的期望熵减少。换句话来讲，Gain (S , A)是由于给定属性A的值而得到的关于目标函数值的信息。

例如，假定 S 是一套有关天气的训练样例，描述它的属性包括可能是具有 Weak 和 Strong 两个值的 Wind。像前面一样，假定 S 包含14 个样例，[9+ ，5 -]。在这 14 个样例中，假定正例中的 6 个和反例中的2 个有Wind =Weak，其他的有Wind =Strong。由于按照属性Wind 分类14 个样例得到的信息增益可以计算如下

$$Values(Wind) = Weak, Strong$$

$$S = [9+, 5-]$$

$$S_{Weak} \leftarrow [6+, 2-]$$

$$S_{Strong} \leftarrow [3+, 3-]$$

所以 Wind[Weak, Strong] 的计算：

$$Gain(S,Wind) = Entropy(S) - \sum_{i \in Weak,Strong} \frac{S _{i}}{S} Entropy(S _{i})$$

$$= Entropy(S) - \frac{8}{14}Entropy(S _{Weak}) - \frac{6}{14} Entropy(S _{Strong})$$

$$= 0.940 - \frac{8}{14} * 0.811 - \frac{6}{14} * 1.00$$

$$ = 0.048$$