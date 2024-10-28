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

## Rank

