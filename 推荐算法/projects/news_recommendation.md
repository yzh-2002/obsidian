>1. [正式赛第一名](https://github.com/wj19971997/NEWS-RECOMMENDATION?spm=a2c22.21852664.0.0.3202658fJVqOUY)
>2. [正式赛第二名](https://github.com/LogicJake/tianchi-news-recommendation?spm=a2c22.21852664.0.0.5832f169YJjfw4)
>3. [正式赛第三名](https://github.com/ikaruga0508/tianchi_news_pub)

数据：
1. `train_click_log.csv`：训练集用户（20w用户）点击日志
2. `testA_click_log.csv`：测试集A用户（5w用户）点击日志（A榜）
3. `testB_click_log.csv`：测试集B用户（5w用户）点击日志（B榜）

训练集，测试集A，测试集B中的用户均不存在交叉。
以A榜为例，任务目标即为预测其每个用户最后一次点击的文章id。

无论是召回还是精排阶段，均可分为线下训练和线上验证两个过程，线下训练用于确定模型选择和超参数。线上验证则是对给定测试集进行最后一次点击文章的预测。在召回阶段，模型除了预测用户最后一次点击文章外，还包含了提取用户和物品特征，**故召回阶段线上验证之前，需要把测试集除去最后一次点击的数据添加到训练集中重新训练一遍**。精排模型使用现成的用户，物品特征，故在训练集上完成训练之后可直接对测试集进行预测，这一点需要注意。
## Recall

### 多路召回

1. `I2I_CF`：基于`user-item`倒排表构建相似度矩阵
	1. 若某文章未出现在日志中，则无法获取该文章与其他文章的相似度（**无法对冷启动文章进行推荐**）
2. `U2I_Model`：基于日志信息训练出`user`和`item`的`embedding`向量
	1. 仅能训练出出现在日志中文章的`embedding`向量
3. `I2I_emb_CF`：基于题目给定的`article_embedding`构建相似度矩阵
	1. 故对于所有文章，均能获取其与其他文章的相似度（**用于文章冷启动**）
4. `U2U_emb_CF`：
	1. 基于`item-user`倒排表构建相似度矩阵，内存要求极高，无法运行
	2. 基于双塔模型训练得到的`user_embedding`构建相似度矩阵并进行召回

多路合并：参考石塔西的方法，每次挑选每一路分数最高的添加进最终召回列表，重复的去除，直至达到目标召回值。

<font color="#ff0000">问题：多路合并后的召回率甚至不如`U2I_Model`一路的召回率......</font>

### 冷启动

1. 文章冷启动：
	1. 基于`I2I_emb_CF`的结果加上一些规则筛选（为避免筛选完，尽可能多的召回文章）
		1. 保留文章主题与用户历史浏览主题相似的文章
		2. 保留文章字数与用户历史浏览文章字数相差不大的文章
		3. 保留最后一次点击当天创建的文章
		4. ...
2. 用户冷启动：此处将仅点击一次文章的用户视作冷启动用户，由于最后一次点击视作预测目标，故实际上此类用户在训练时一次未点击，故上述召回方法均无效
	1. 解决办法：热门召回（没有用户属性，只能采用这种办法...）


## Rank

### 特征工程
> 构造特征，构造用于排序模型的监督数据集

现有特征：
1. 文章自身属性，例如：`category_id, created_at_ts, words_count`
2. 文章内容`embeding`
3. 用户自身属性，例如：`click_environment, click_os, ...`

**如何挖掘其他特征信息？**
	1. 挖掘用户候选item与其**历史点击文章的关联信息**：
		1. `embedding 内积`，其中有多种可用的`embedding`，例如：数据提供的基于内容的embedding，YoutubeDNN训练得到的item embedding，也可以根据历史序列，经w2v训练得到item embedding
		2. 统计特征（字数，建立时间差...）
	2. 用户画像
		1. 活跃度（综合点击次数，点击时间（点击频率）考虑）
		2. 设备习惯，时间习惯，主题爱好，字数偏好...
	3. 文章画像
		1. 主题，字数，创建时间，文章热门...

召回结果有两类，一类为`Embedding`信息，双塔模型为每个用户和物品都生成了一个`Embedding`向量（可用于构造特征），还有一类为每个用户有一个召回列表：
```python
{
 user_id_1:[(item_id_1,score_1),(item_id_2,score_2),...],
 user_id_2:[...]
}
```
如何构造用于排序模型训练的数据集呢？最终形似如下：
```python
[ user_id | item_id | feature_1 | feature_2 | ... | label ]
[    u1   |    i1   |     ?     |     ?     | ... |   ?   ]
[    u1   |    i2   |     ?     |     ?     | ... |   ?   ]
[    u2   |    i4   |     ?     |     ?     | ... |   ?   ]
[    u2   |    i5   |     ?     |     ?     | ... |   ?   ]
```
也即先将上述召回列表展开，再**向其中补充特征与标签**。

在构造标签时，获取训练集用户最后一次点击的文章id，其对应label为1，其余均为0。也即将召回未点击近似看作曝光未点击。这样的后果是正负样本极度不平衡，故需要对负样本进行下采样。

**疑问：为什么不像召回阶段一样，通过滑动窗口扩充数据集呢？也即扩充一些正样本呢？**
召回结果本身就是基于每个用户前n-1个交互文章对其下一个点击文章的预测，故无法再使用滑动窗口扩充数据集。

采样时需要注意：确保每个用户以及每个物品均在数据集出现至少一次，不能因为下采样导致其丢失。

`lgb.classifier`训练后`MRR`反而只有`0.06`（简历上先写`0.28`吧），直接双塔召回的`MRR`是`0.14`

### DIN Rank
较之`LightGBM`直接输入一个表格数据，排序阶段的`DIN`模型同召回阶段的`YoutubeDNN`模型一样，需要将数据集划分为`数值型特征，离散型特征以及历史行为序列特征`。

当然这个过程是在前面得到数据集的基础上进行的。
## Appendix

### Pandas
1. `pd.concat`：堆叠数据，不考虑数据之间键的关系，类比于SQL中的`union`
	1. 例如：`train_click_log`，`testA_click_log`
	2. 参数：`ignore_index`，一般设置为`True`，即忽略原先数据中的index，concat之后重新计数（否则concat之后的数据可能存在重复索引）
		1. **索引`Index`：可以视作`DataFrame`的横向索引，纵向索引是`Column`**
2. `pd.merge`：合并数据，根据数据之间的键值对合并，类比于SQL中的`join`
	1. 例如：`train_click_log`和`articles`
	2. 参数：`how`
		1. `inner`：保留两个df中键匹配的行，不匹配的均删去
		2. `left`：保留左侧所有行，如果右侧数据不存在与左侧匹配数据则置NaN
3. `DataFrame[Column]`：返回`Series`对象
	1. `DataFrame[Column].values`：返回**numpy数组**（数值计算更有优势）
	2. `DataFrame[Column].tolist()`：返回python的**list结构**（便于python操作）
4. `DataFrame[[Column]]`：返回仅含`Column`列的`DataFrame`对象
5. `DataFrame.iloc`：与`loc`功能一致，区别在于其使用整数作为索引，后者选择标签作为索引，与`DF[column]`的区别在于其即可选择行，又可选择列
	1. `DF.iloc[i]`：选择第i行
	2. `DF.iloc[:,i]`：选择第i列
6. `DataFrame.loc`
7. `DataFrame.apply(lambda x:f(x))`：默认x为`Series`，表示某一列的所有数据
8. `DataFrame.groupby("Column")`：
	1. 遍历：key为对应Column的值，value为`DataFrame`，包含所有属于key的行