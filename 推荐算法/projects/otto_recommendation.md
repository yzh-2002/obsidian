> [Kaggle链接](https://www.kaggle.com/competitions/otto-recommender-system/overview)

目标：预测电子商务点击量，购物车添加量和订单量。（多目标预测）

`train.jsonl(11.31GB)`：`{session:xxx,events:[{aid:xxx,ts:xxx,type: clicks/carts/orders}]}`
1. `session`：代表一个用户
2. `events`：行为日志
`test.jsonl`：预测每个用户最后一个时间戳之后的三类行为可能的`aid`

评价指标：$R_{clicks/carts/orders}@20$，并有最终的加权平均分`score=0.1*R{clicks}+0.3*R{carts}+0.6*R{orders}`

