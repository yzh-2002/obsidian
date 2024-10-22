目前开源的推荐模型实现有如下几种：
1. [DeepMatch/DeepCTR](https://github.com/datawhalechina/torch-rechub/blob/main/torch_rechub/basic/features.py)
	1. 均基于`TensorFlow`实现
	2. 前者支持到`python3.8`，后者支持到`python3.7`，且`DeepMatch`依赖于`DeepCTR`
	3. 官方测试构建配置[链接](https://tensorflow.google.cn/install/source_windows?hl=zh-cn#tested_build_configurations)
2. [torch-rechub](https://github.com/datawhalechina/torch-rechub/blob/main/torch_rechub/basic/initializers.py)
	1. 设计参考了`DeepMatch/DeepCTR`
	2. 基于`PyTorch`
3. [RecBole](https://github.com/RUCAIBox/RecBole)
	1. 基于`PyTorch`

## DeepMatch/CTR

~~`pip install deepmatch[gpu]`~~会尝试安装`tensorflow-gpu`，但自2.1版本开始，GPU支持已经包含在`tensorflow`包中，前者已经被移除。

`pip install deepmatch`不会安装`tensorflow`相关依赖，仍需手动安装，故此处手动安装`pip install tensorflow-gpu==1.15.0`

## lgb/xgboost
> LightGBM：**LightGBM** is a gradient boosting framework that uses tree based learning algorithms.
> Xgboost：**XGBoost** is an optimized distributed gradient boosting library designed to be highly **efficient**, **flexible** and **portable**.
> 上述均摘自两者官方文档，两者均提到了`gradient boosting`.

### 基础知识
1. 什么是`boosting`？
	1. 集中学习方法的一种，思想是：”三个臭皮匠顶一个诸葛亮“
	2. 其训练一批弱学习器（串行产生，根据前一个弱学习器的表现对样本分布进行调整，使得后一个弱学习器可以重点关注前一个弱学习器出错的训练样本）以期产生`好而不同`的弱学习器
	3. 最终将这些弱学习器结合构成一个强大的强学习器（集成学习器）
2. `boosting`算法中弱学习器如何重点关注前一个弱学习器出错的训练样本？（`AdaBoost`算法）
	1. 通过调整样本在损失函数权重来实现
	2. 初始权重均$w_{1i}=\frac{1}{N}$，权重更新公式为$w_{m,i}=\frac{w_{m-1,i}}{Z_m}exp(-\alpha_{m-1}I(G_{m-1}(x_i)\neq y_i))$
		1. 其中$\alpha_{m-1}$为第`m-1`个弱学习器的权重
		2. $Z_m$是归一化因子
		3. 很直观，如果上一弱学习器对i样本出错，则权重加大
3. `boosting`算法如何结合弱学习器构成强学习器？
	1. $\alpha_m=\frac{1}{2}log\frac{1-e_m}{e_m}$为第m个弱学习器的权重，可见误差越小，权重越大
	2. 强学习器是这些弱学习器的线性组合
4. 了解`Gradient boosting`之前先看一下`boosting tree`算法
	1. 较之`Adaboost`不同，其**不调整样本的权重**，而是每次训练时都在所有已有学习器基础上训练新的弱学习器的参数
	2. 也即$f_m(x)=f_{m-1}(x)+T(x;\theta)$
5. 什么是`Gradient boosting`？
	1. 当损失函数为平方误差时，$L(y,f_m(x))=L(y,f_{m-1}(x)+T(x;\theta)=(y-f_m(x)-T(x;\theta))^2=(r-T(x;\theta))^2$
	2. 其中r为当前模型拟合数据的残差，也即只需要使得新弱学习器拟合r即可，容易优化
	3. 但当损失函数是其他函数时，优化便没有那么直观简单了
	4. 可将L视作`f(x)`的函数，沿着L关于该变量负梯度方向变化一定可以使得L减小，也即$L(y,f_{m-1}(x)-\frac{\partial{L}}{\partial{f_{m-1}(x)}})<L(y,f_{m-1}(x)+T(x;\theta))$
	5. 故对于任何损失函数，只需要新的弱学习器拟合L关于`f(x)`的负梯度即可

### 模型使用
`LightGBM`内置了四种模型的实现，分别是`LGBMModel,LGBMClassifier,LGBMRegressor,LGBMRanker`，后三种模型分别针对不同的任务类型，第一个模型相当于后三种模型的抽象。

实例化模型时参数配置：
1. `boosting_type`：`gdbt`,`dart,`random forest`
2. `num_leaves`：定义树中最大叶子节点数目，由于`LightGBM`采用`leaf-wise`算法，故树的复杂程度取决于`num_leaves`而不是`max_depth`
3. `max_depth`：树的最大深度，`leaf-wise`容易长出较深的树，故需控制防止过拟合
4. `reg_alpha`：L1正则化（`reg_lambda`：L2正则化）
5. `n_estimators`：提升树个数
6. `subsample`：构建每棵树时随机选择的样本比例，防止过拟合
	1. `subsample_freq`：采样频率，如果设置为2，则表示每隔两颗树进行一次采样（每两棵树采用相同的样本）
	2. `colsample_bytree`：每棵树随机选择的特征比例，减少特征之间的相关性（有点`Dropout`的感觉）
7. `min_child_weight`：用于控制每个叶子节点所需**最小实例权重和（Hessian）**，通俗来讲就是避免生成仅包含少量样本的叶子节点，这些节点不够可靠或稳定
	1. 节点的实例权重和如何计算？？？？
8. `random_state`：随机种子
9. `n_jobs`：训练时并行线程数量
---
模型训练参数配置：
1. `X`，`y`
2. `group`：用于LTR（排序）任务，指定样本的分组信息，在推荐任务中，往往每个用户的召回列表作为一个分组
3. `eval_set`：验证集，包含`X`和`y`
	1. `eval_group`：同上
	2. `eval_at`：用于LTR任务
	3. `eval_metric`

