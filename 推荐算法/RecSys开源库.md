目前开源的推荐模型实现有如下几种：
1. [DeepMatch/DeepCTR](https://github.com/datawhalechina/torch-rechub/blob/main/torch_rechub/basic/features.py)
	1. 基于`TensorFlow`
2. [torch-rechub](https://github.com/datawhalechina/torch-rechub/blob/main/torch_rechub/basic/initializers.py)
	1. 设计参考了`DeepMatch/DeepCTR`
	2. 基于`PyTorch`
3. [RecBole](https://github.com/RUCAIBox/RecBole)
	1. 基于`PyTorch`

## 数据处理

以`MovieLens 1M Dataset`[点此下载](https://grouplens.org/datasets/movielens/1m/)为例，探究模型训练之前数据处理需要哪些步骤？

召回阶段：
1. 确认模型使用特征，双塔模型分为用户特征与物品特征
	1. 用户特征：用户ID（Sparse），用户个人信息（性别，年龄，职业，邮政编码），用户观影历史（Sequence）
	2. 物品特征：物品ID（Sparse），物品历史评分（Dense）
2. 特征处理：离散特征经`LabelEncoding`转换为连续整数值，连续特征归一化...
3. **序列特征往往与数据集划分强相关：**
	1. 推荐任务一般根据用户历史行为预测其下一次行为，给定数据集，用户最后一次交互行为作为测试集，用户剩余n-1次交互行为可采用滑动窗口划分为n-1条训练数据
	2. 例如：`user_i:[item_1,item_2,item_3,...item_n]`
		1. 测试集：`user_i:[item_1,item_2,item_3,...item_n-1],item_n`
		2. 训练集：`user_i:[item_1],item_2`，`user_i:[item_1,item_2],item_3`...
4. 负样本的选取，针对不同的训练方式有三种不同的选取方法
	1. `Pointwise`：
	2. `Pairwise`：
	3. `Listwise`：
5. 最终数据集需要把**用户和物品的其余特征与序列特征拼接**
