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

## Baseline
> 仅召回尝试一下，常见的召回策略：
> 1. `co-visitation`（共现矩阵）：其实就是（简化的`ItemCF`）协同过滤算法，统计每个用户历史行为中`item`组合对数量，筛选出高频的`item-pair`来获取每个item和它关联度最高的item
> 2. `Word2Vec`：将用户历史交互序列作为`sentence`，使用`word2vec`训练学习，便可以得到每个item的`embedding`，然后根据最近邻查找获取关联度较高的item即可
> 3. `双塔模型`：类似于`Word2Vec`，不过可以引入一些其他特征



