---
layout: post
title: "信息增益(information gain)"
date: 2012-11-17 15:28
comments: true
categories: algorithm
---
####简介
百度百科上面的定义：在概率论和信息论中，信息增益是非对称的，用以度量两种概率分布P和Q的差异。信息增益描述了当使用Q进行编码时，再使用P进行编码的差异。通常P代表样本或观察值的分布，也有可能是精确计算的理论分布。Q代表一种理论，模型，描述或者对P的近似。

读起来略微有点费解，其实我个人理解，信息增益，既然有增益，必然有要对比。

首先要对信息进行量化，说到这个，就不得不提到我上大学非常崇拜的一个大神 “信息论之父” 香农，他提出了[信息熵](/algorithm/2012/12/07/information-theory.html "Title")，实现了对信息的量化。

$$H=-\sum_{i} p_i \log p_i$$

使用上面的公式，就能计算出信息熵。

####举例说明：

|Outlook|Temperature|Humidity|Windy|Play?|
|---|---|---|---|---|
|sunny|hot|high|false|no|
|sunny|hot|high|true|no|
|overcast|hot|high|false|yes|
|rain|mild|high|false|yes|
|rain|cool|normal|false|yes|
|rain|cool|normal|true|no|
|overcast|cool|normal|true|yes|
|sunny|mild|high|false|no|
|sunny|cool|normal|false|yes|
|rain|mild|normal|false|yes|
|sunny|mild|normal|true|yes|
|overcast|mild|high|true|yes|
|overcast|hot|normal|false|yes|
|rain|mild|high|true|no|

共14个实例，9个正例（yes），5个负例（no）。

####信息熵

$$-\sum _{i=0}{entropy([9,5])} = -\frac{9}{14}\log _{2}(\frac{9}{14})–\frac{5}{14}\log _{2}(\frac{5}{14})$$

$$0.410+0.530=0.940$$

####计算信息增益(Information Gain)

通过某一个属性(例如 Outlook)来划分数据后，数据变成三份

$$InfoEntropys(S_0, S_1 ... S_n)=-\sum _{i=0} ^{n} {entropy(S_i)}$$

|Outlook = sunny|Play?|
|---|---|---|---|---|
|sunny|yes|
|sunny|yes|
|sunny|no|
|sunny|no|
|sunny|no|

$$info\_entropy_{sunny}=-\sum _{outlook=sunny} {entropy([positive, negative])}$$

$$-\frac{2}{5}log_{2}(\frac{2}{5}) -\frac{3}{5}log_{2}(\frac{3}{5}) = 0.971$$

|Outlook = overcast|Play?|
|---|---|---|---|---|
|overcast|yes|
|overcast|yes|
|overcast|yes|
|overcast|yes|

$$info\_entropy_{overcast}=-\sum _{outlook=overcast} {entropy([positive, negative])}$$

$$-\frac{4}{4}log_{2}(\frac{4}{4}) -\frac{0}{4}log_{2}(\frac{0}{4}) = 0$$

|Outlook = rainy|Play?|
|---|---|---|---|---|
|rainy|yes|
|rainy|yes|
|rainy|yes|
|rainy|no|
|rainy|no|

$$info\_entropy_{rainy}=-\sum _{outlook=rainy} {entropy([positive, negative])}$$

$$-\frac{3}{5}log _{2} (\frac{3}{5}) -\frac{2}{5} log_{2}(\frac{2}{5}) = 0.971$$

$$InfoEntrops = \frac{5}{14} * 0.971 + \frac{4}{14} * 0 + \frac{5}{14} * 0.971 = 0.694$$

$$Gain() = 0.940 - 0.694 = 0.246$$

通过属性来划分数据，就能更加好的区别数据的不同。

这样对于数据的预测就有了一点的偏差，这个偏差是让结果从 50% 逼近真实的结果，如果没有上面的数据，对于 Play? 随机估计的话，yes or no 各占一半，概率是 50%。如果有了这个我就能更加准确的估计 Play? 的值，并不是瞎猜的 50% 而是有一个偏差，这个偏差是 64.2% 虽然不能准确的估计这个值，但是能更加逼近这个值。

继续下章 [ID3](/machine_learning/2013/01/15/id3-algorithm.html)














