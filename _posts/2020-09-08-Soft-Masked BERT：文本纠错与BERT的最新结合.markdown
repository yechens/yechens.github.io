---
layout: post
title:  "Soft-Masked BERT：文本纠错与BERT的最新结合"
date:   2020-09-08 12:00:00 +0530
categories: BERT 文本纠错
---
**文本纠错**，是自然语言处理领域检测一段文字是否存在错别字、以及将错别字纠正过来的技术，一般用于文本预处理阶段，同时能显著缓解智能客服等场景下语音识别（ASR）不准确的问题。

本文将通过以下几个章节简要介绍文本纠错相关知识。

```python3
1. 文本纠错示例与难点
2. 文本纠错常用技术
3. 如何将 BERT 应用于文本纠错
4. 文本纠错最优模型：Soft-Masked BERT（2020-ACL）
5. 立马上手的纠错工具推荐
```

## 一.文本纠错示例与难点

生活中常见的文本错误可以分为（1）字形相似引起的错误（2）拼音相似引起的错误 两大类；如：“咳数”->“咳嗽”；“哈蜜”->“哈密”。错别字往往来自于如下的“相似字典”。

<img src="https://pic3.zhimg.com/80/v2-cecd697f18ba75d449a2f6ee37cd8bea_720w.jpg" alt="相似发音中文字典" style="zoom:80%;" />

<img src="https://pic4.zhimg.com/80/v2-92ea2594e54f1f43a66addad454bd2df_720w.jpg" alt="相似字形中文字典" style="zoom:80%;" />

其他错误还包括方言、口语化、重复输入导致的错误，在ASR中较为常见。

现有的NLP技术已经能解决多数文本拼写错误。剩余的**纠错难点**主要在于，部分文本拼写错误需要**常识背景（world-knowledge）**才能识别。例如：

```python3
Wrong: "我想去埃及金子塔旅游。"
Right: "我想去埃及金字塔旅游。"
```

将其中的“金子塔”纠正为“金字塔”需要一定的背景知识。

同时，一些错误需要模型像人一样具备**一定的推理和分析能力**才能识破。例如：

```python3
Wrong: "他的求胜欲很强，为了越狱在挖洞。"
Right: "他的求生欲很强，为了越狱在挖洞。"
```

“求胜欲”和“求生欲”在自然语言中都是正确的，但是结合上下文语境来分析，显然后者更为合适。

最后，**文本纠错技术对于误判率有严格的要求，一般要求低于0.5%**。如果纠错方法的误判率很高（将正确的词“纠正”成错误的），会对系统和用户体验有很差的负面效果。

## 二.文本纠错常用技术

错别字纠正已经有很多年的研究历史。常用的方法可以归纳为错别字词典、编辑距离、语言模型等。

构建错别字词典人工成本较高，适用于错别字有限的部分垂直领域；编辑距离采用类似字符串模糊匹配的方法，通过对照正确样本可以纠正部分常见错别字和语病，但是通用性不足。

所以，现阶段学术界和工业界研究的重点一般都是基于语言模型的纠错技术。2018年之前，语言模型的方法可以分为**传统的n-gram LM和DNN LM**，可以以字或词为纠错粒度。其中“字粒度”的语义信息相对较弱，因此误判率会高于“词粒度”的纠错；“词粒度”则较依赖于分词模型的准确率。

为了降低误判率，往往在模型的输出层加入CRF层校对，通过学习转移概率和全局最优路径避免不合理的错别字输出。

2018年之后，预训练语言模型开始流行，研究人员很快把BERT类的模型迁移到了文本纠错中，并取得了新的最优效果。

## 三、将BERT应用于文本纠错

<img src="https://pic2.zhimg.com/80/v2-9b1a13bd89acc64ea68ffa269bbd9741_720w.jpg" alt="BERT示意图" style="zoom:80%;" />

BERT与以往深度学习模型的主要区别在于：预训练阶段使用了“掩码语言模型”MLM和“判断s1是否为s2下一句”NSP两个任务，特征抽取使用12层双向Transformer，更大的训练语料和机器「More Money，More Power」。其中，MLM任务使得模型并不知道输入位置的词汇是否为正确的词汇（10%概率），这就迫使模型更多地依赖于上下文信息去预测词汇，赋予了模型一定的纠错能力。

一种简单的使用方式为，依次将文本s中的每一个字c做mask掩码，依赖c的上下文来预测c位置最合适的字（假设词表大小为20000，相当于在句子中的每一个位置做了一个“20000分类”）。设置一个容错阈值k=5，如果原先的字c出现在预测结果的top5中，就认为该位置不是错别字，否则是错别字。

<img src="https://pic3.zhimg.com/80/v2-b792982921f90c1c30b303a1f5a67be6_720w.jpg" alt="BERT文本纠错Baseline" style="zoom:80%;" />

当然这种方法过于粗暴，很可能造成高误判率。作为优化，我们可以采用预训练的方式对BERT进行微调，显著改进纠错效果。纠错的领域最好和微调领域相同（如果需要在新闻类文章中纠错，可以使用“人民日报语料”对模型微调）。

除了上述方法之外，还有很多 trick 可以提升 BERT 的效果与速度。这里推荐大家读一下 bienlearn 上的 BERT 专栏。

这个 BERT 专栏由自然语言处理领域的 KOL——「夕小瑶的卖萌屋」主笔，帮助新手以及有一定基础的同学快速上手 BERT，既包括原理、源码的解读，还有 BERT 系的改进串讲与高级精调技巧。不管是准备面试、项目实战还是比赛刷分，都可以找到对应的内容。

目前在限时优惠，更详细的内容大家可以点击下方卡片查看：

## 四、文本纠错最优模型：Soft-Masked BERT

为了弥补baseline方法的不足，最大限度发挥BERT功效，复旦大学的研究人员在2020 ACL上发表了最新论文：

[“Spelling Error Correction with Soft-Masked BERT”。arxiv.org](https://link.zhihu.com/?target=https%3A//arxiv.org/pdf/2005.07421)

![模型架构图](https://pic2.zhimg.com/80/v2-749cf0fa34d4937a7a4be998f90c524d_720w.jpg)

论文首次提出了Soft-Masked BERT模型，主要创新点在于：

（1）将文本纠错划分为检测网络（Detection）和纠正网络（Correction）两部分，纠正网络的输入来自于检测网络输出。

（2）以检测网络的输出作为权重，将 masking-embedding以“soft方式”添加到各个字符特征上，即“Soft-Masked”。

## **论文简要分析**

具体来看，模型Input是字粒度的word-embedding，可以使用BERT-Embedding层的输出或者word2vec。检测网络由Bi-GRU组成，充分学习输入的上下文信息，输出是每个位置 i 可能为错别字的概率 p(i)，值越大表示该位置出错的可能性越大。

![img](https://pic1.zhimg.com/80/v2-4195f837d0a9c829b9b3cfa28b348d94_720w.jpg)

## 检测网络 与 Soft Masking

Soft Masking 部分，将每个位置的特征以 ![[公式]](https://www.zhihu.com/equation?tex=p%7Bi%7D) 的概率乘上 masking 字符的特征 ![[公式]](https://www.zhihu.com/equation?tex=e%7Bmask%7D) ，以 ![[公式]](https://www.zhihu.com/equation?tex=1-p%7Bi%7D) 的概率乘上原始的输入特征，最后两部分相加作为每一个字符的特征，输入到纠正网络中。原文描述：

![img](https://pic3.zhimg.com/80/v2-8f19508c1ef82e9ff5524f920ec005ea_720w.jpg)

## 纠正网络

纠正网络部分，是一个基于BERT的序列多分类标记模型。检测网络输出的特征作为BERT 12层Transformer模块的输入，最后一层的输出 + Input部分的Embedding特征 ![[公式]](https://www.zhihu.com/equation?tex=e%7Bi%7D) （残差连接）作为每个字符最终的特征表示。

![img](https://pic1.zhimg.com/80/v2-9fa5a214a7595cd22f9474f4ba0b7bbc_720w.jpg)

最后，将每个字特征过一层 Softmax 分类器，从候选词表中输出概率最大的字符认为是每个位置的正确字符。

![img](https://pic4.zhimg.com/80/v2-bb59576309b1b4fe821d5ebeaca0788f_720w.jpg)

整个网络的训练端到端进行，损失函数由检测网络和纠正网络加权构成。

<img src="https://pic3.zhimg.com/80/v2-a1e2e1cd3649e256db901049242eff3e_720w.jpg" alt="损失函数" style="zoom:80%;" />

## 实验结果

作者在“SIGHAN”和“NEWs Title”两份数据集上做了对比实验。其中[“SIGHAN”](https://link.zhihu.com/?target=http%3A//ir.itc.ntnu.edu.tw/lre/sighan7csc.html)是2013年开源的中文文本纠错数据集，规模在1000条左右。“NEWs Title”是从今日头条新闻标题中自动构建的纠错数据集（根据文章开头展示的相似字形、相似拼音字典），有500万条语料。

![img](https://pic4.zhimg.com/80/v2-4949287aa920f883f75da67a339bfb17_720w.jpg)

Soft-Masked BERT 在两份数据集上几乎都取得了最好结果。同时我们发现，**Finetune对于原始BERT的表现具有巨大的促进作用。**

论文代码作者暂未开源，但是论文的模型和思路应该是非常清晰易懂的，实现起来不会太难。这儿先立个flag，有时间自己来实现一下。

## 五、立马上手的纠错工具推荐

笔者简单调研发现，文本纠错网上已经有不少的开源工具包供大家使用了。其中最知名的应该是 pycorrector

[pycorrectorgithub.com](https://link.zhihu.com/?target=https%3A//github.com/shibing624/pycorrector)

支持kenlm、rnn_crf、seq2seq、BERT等各种模型。结合具体领域的微调和少量规则修正，应该可以满足大部分场景中的文本纠错需求了。

![img](https://pic4.zhimg.com/80/v2-27dac4bca92016d2d71fe24c6ed6493b_720w.png)使用测试

Demo中笔者使用了经人民日报语料微调过的BERT模型，通过pycorrect加载来做基于MLM的文本纠错。识别结果还算可以，甚至“金字塔”这种需要常识的错别字都纠正出来了。

当然pycorrect还支持各种语言模型和DNN模型，留给大家自行把玩 : )

此外，笔者还找到一个京东客服机器人语料做的纠错模型：

[基于京东客服机器人语料做的中文纠错模型github.com!](https://link.zhihu.com/?target=https%3A//github.com/taozhijiang/chinese_correct_wsd)

主要解决同音字自动纠错问题，比如：

```python3
对京东新人度大打折扣 --> 对京东信任度大打折扣
我想买哥苹果手机 --> 我想买个苹果手机
```

不过仓库上一次更新在5年前，年代久远估计效果有限。

------

以上是笔者近期调研文本纠错后的一些思考，刚好上周在实验室组会中做了分享，就顺便写了这篇文章。如果大家发现有好的纠错方法或论文，欢迎留言分享一起交流哈

## Reference

[1.中文文本纠错算法--错别字纠正的二三事](https://zhuanlan.zhihu.com/p/40806718)

[2.Spelling Error Correction with Soft-Masked BERT](https://link.zhihu.com/?target=https%3A//arxiv.org/pdf/2005.07421.pdf)

[3.pycorrector](https://link.zhihu.com/?target=https%3A//github.com/shibing624/pycorrector)

[4.京东客服-文本纠错](https://link.zhihu.com/?target=https%3A//github.com/taozhijiang/chinese_correct_wsd)

[5.SIGHAN 2013 Bake-off: Chinese Spelling Check Task](https://link.zhihu.com/?target=http%3A//ir.itc.ntnu.edu.tw/lre/sighan7csc.html)

 