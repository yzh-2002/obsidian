> [Kaggle链接](https://www.kaggle.com/competitions/otto-recommender-system/overview)

目标：预测电子商务点击量，购物车添加量和订单量。（多目标预测）

`train.jsonl(11.31GB)`：`{session:xxx,events:[{aid:xxx,ts:xxx,type: clicks/carts/orders}]}`
1. `session`：代表一个用户
2. `events`：行为日志
`test.jsonl`：预测每个用户最后一个时间戳之后的三类行为可能的`aid`

1. `click`：`ground truth`为该session内下一次点击的aid（**仅有一个**）
2. `cart`：`ground truth`为该session内所有可能添加至购物车的aid（**可能有多个**）
	1. 不过`cart/order`对应的aid一般在`click/cart`对应aids范围内
3. `order`：同`cart`

评价指标：$R_{clicks/carts/orders}@20$，并有最终的加权平均分`score=0.1*R{clicks}+0.3*R{carts}+0.6*R{orders}`

## EDA
> Exploratory Data Analysis，探索性数据分析

首先将数据格式转换为`DataFrame`，`Columns=[session,aid,ts,type]`，训练集共计`2亿`条记录，涉及`1.2千万`用户。

训练集和测试集的用户不存在重叠。

训练集的用户交互日期为`2022/07/31 22:00 -> 2022/08/28 21:59`（四星期）
测试集的用户交互日期为`2022/08/28 22:00 -> 2022/09/04 21:59`（一星期）


## Baseline
> 仅召回尝试一下，常见的召回策略：
> 1. `co-visitation`（共现矩阵）：其实就是（简化的`ItemCF`）协同过滤算法，统计每个用户历史行为中`item`组合对数量，筛选出高频的`item-pair`来获取每个item和它关联度最高的item
> 2. `Word2Vec`：将用户历史交互序列作为`sentence`，使用`word2vec`训练学习，便可以得到每个item的`embedding`，然后根据最近邻查找获取关联度较高的item即可
> 3. `双塔模型`：类似于`Word2Vec`，不过可以引入一些其他特征


召回方法一：基于`co-visitation`
1. 首先，**并行计算**每个`session`中共现的`aid pair`
	1. 然后，将所有`session`的`aid pair`合并，得到如下结构：`aid1:{aid2:weight,...}`
	2. 针对每个`aid`做截断，取前`TopK`个
2. 上述`aid pair`的权重分为两种模式：
	1. `ops mode`：根据操作模式确定权重（用于`carts/orders`召回，`carts`和`orders`共享找回结果）
	2. `time mode`：根据aid 交互间隔时间确定权重（用于`clicks`召回）
3. 如何通过上述结果为测试集用户预测其未来可能`clicks/carts/orders`呢？
	1. 首先获取测试集用户历史交互`aid list`，去重
	2. **`aid list`长度大于20，则对其中元素计算权重并返回前20个**
		1. 权重：`sequence weight`+`co_visitation weight`
	3. `aid list`长度小于20，则遍历`aid list`元素，从`co_visitation`的`topk`中获取相似`aid`，从中返回`20-len(set(aid list))`个，再拼接上`set(aid list)`返回

提交到`Kaggle`上Score为`0.574`

## Local CV
> CV，也即Cross Validation，交叉验证

机器学习的模型训练过程中，通常不能将所有数据全用于训练，而要提前划分出一部分用于评估模型的训练效果，**从而确定模型与超参数的选择。**

最简单，把整个数据集划分为训练集，验证集两部分
1. 模型参数选取依赖于训练集和测试集的划分方法
2. 模型仅使用部分数据进行训练

上述方法，训练集和验证集完全分割开，**并未交叉**。

1. LOOCV（留一法）
	1. 每次从数据中选择一个作为验证集，其余作为训练集，故可得到n组（训练集，验证集）
	2. 最终训练出n个模型，所有模型损失函数的平均为验证集损失
	3. 缺点：计算复杂度过大
2. K-fold CV（K折~）
	1. 留一法可视作1折
	2. 一定程度上缓解了LOOCV计算复杂度过大的问题

为什么最后要对n个模型的损失取平均？
此处主要区分模型的选择 与 超参数的选择。K-fold CV中得到n组训练集和验证集，每组可以设置同一模型的不同超参数，并对比选择最优的超参数。取平均是为了对比不同的模型。

---
otto数据集划分：考虑到数据较多，采用简单划分即可。
1. 训练集包含用户4个星期的日志，按照事件时间戳：
	1. 前三个星期作为训练数据
	2. 后一个星期作为标签
2. 需注意：**`train`和`valid`数据集的用户不可以存在交集**

也即我们需要先划分出`train session`和`valid session`，然后只取前3星期的进行召回/精排，对于`valid session`预测其第四个星期的行为，并再验证集上计算`score`以确定召回/精排的超参数等等。

## feature

## Rank
> [Multi-task 模型训练示例](https://github.com/datawhalechina/torch-rechub/blob/main/tutorials/Multi_Task.ipynb)

精排大致过程：
1. 理想的精排数据集：使用曝光未点击作为负样本，曝光点击作为正样本
	1. 此处选择召回会点击作为负样本，召回点击作为正样本
2. `train.jsonl`中所有用户基于前三星期的交互历史生成第四星期的召回`aid list`，再根据实际第四星期的交互历史生成正负样本（打标签）
	1. 多目标模型的数据集存在多个标签，本赛则有三个标签：`is_click,is_cart,is_order`
3. 分割训练集和验证集（以用户为粒度）
4. 特征工程，数据拼接生成最终数据
5. 模型训练

## ablation
> 消融实验