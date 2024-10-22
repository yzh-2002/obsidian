## Attention
目前网上给出的`attention`的引入思路都类似于`KNN`，也即$f(query)=\sum_{i}^{n} \alpha(key_i,query)\cdot value_i$，也即`query,key`为样本，`value`为样本对应标签。

但提到`attention`的优势，却有很多高赞回答称其一种特征增强（注意力是特征的权重），例如[知乎上这一高赞回答](https://www.zhihu.com/question/316736401/answer/2290050996)。那注意力到底是作用在数据上，还是特征上，亦或者说在注意力机制的应用中，数据和特征可以视作类似的含义呢？

此处先不纠结，先往后面看。
> 李沐老师在视频中提到：根据任务的不同，`q,k,v`不太一样，现在没必要纠结其具体是什么，后续见的任务多了再去总结

![image.png](https://raw.githubusercontent.com/yzh-2002/img-hosting/main/notes/202410121846779.png)

---
注意力分数（注意力权重是分数经`softmax`化后的结果）如何计算？

先前`query`和`key`都是实数时，可以使用类似于高斯核函数来作为注意力评分函数计算权重。也即$softmax(-\frac{1}{2}(query-key_i)^2)$，当`query`和`key`为高维向量时，如何计算呢？
1. `Additive Attention`加性注意力：$\alpha(k,q)=v^T tanh(W_k k+W_q q)$
	1. 此处`query`和`key`的**维度不同**
	2. **此处的`+`操作比较复杂，可参考代码实现**
2. `Scaled Dot-Product Attention`缩放点积注意力：$\alpha(q,k_i)=<q,k_i>/\sqrt{d}$
	1. 此处要求`query`和`key`的**维度相同，均为d**
	2. 之所以除以$\sqrt{d}$，原因在于点积的大小正比于d，当d很大时，其结果很大，`softmax`之后容易出问题。
### Seq2seq with Attention

之前RNN章节最后，我们使用`seq2seq`进行机器翻译，在`decoder`模型中，对每时刻的词元进行预测时，总是把`encoder`的隐状态拼接进输入，隐状态表示的是前面所有序列的信息总和，但以`你好，世界！-> Hello World!`为例，在翻译`Hello`时，模型应该更加关注`你好`，但先前实现的`seq2seq`没有这个能力，为此需要引入`attention`

如何引入呢？

首先明确`q,k,v`在该任务的含义：
1. `query`：`decoder`上一时刻输出词元（也是该时刻的输入词元）
2. `keys`：`encoder`各个时刻最后一层隐含层状态（先前`seq2seq`是取得最后时刻各层隐含层状态）
3. `values`：与`keys`相同

较之先前的代码，`encoder`是不变的，只需修改`decoder`的代码

`decoder`部分一方面添加了`attention`机制，另一方面在对输入x进行处理时，即使是训练阶段，每次x为整个序列，也不能直接扔给RNN层，因为不同时刻`query`是在发生变化的，所以需要拆分为一个时刻迭代计算（类似于之前预测时期的代码）。
## Self-Attention
给定序列$x_1,x_2,...$，自注意力机制是将$x_i$既当`q`，又当`k`，又当`v`来抽取序列特征得到$y_1,y_2,...$
其中$y_i=f(x_i,(x_1,x_1),(x_2,x_2)...)$

`CNN`，`RNN`，`Self-Attention`均能处理序列数据（`CNN`可以使用一维`kernel`来做）：
![image.png](https://raw.githubusercontent.com/yzh-2002/img-hosting/main/notes/202410142148608.png)

---
设序列长度为`n`
1. `CNN`：输入输出通道数量为`d`，卷积核大小为`k`（输出通道=卷积核数量）
	1. 时间复杂度：$O(knd^2)$
	2. 并行度：$O(n)$
	3. 最长路径（可以理解为模型观测到整个序列需要的步数）：$O(n/k)$
		1. 每个卷积核一次只能观测到k个序列数据，遍历整个序列需要...
		2. 故卷积对于长文本序列建模效果不好
2. `RNN`：词汇表数目v，隐状态维度h
	1. 时间复杂度：$O(n*Matrix(v,h))$
	2. 并行度：$O(1)$
	3. 最长路径：$O(n)$
3. `Self-attention`：q,k,v均为`n X d`矩阵
	1. 时间复杂度：$O(n* Matrix(n,d))$
	2. 并行度：$O(n)$
	3. 最长路径：$O(1)$
		1. **十分擅长处理长序列数据，其感受野很宽**


**上面还是有点模糊，但暂且搁置，后续有感触了再补充**

### Position Encoding

`Self-attention`丢失了序列的位置信息，对于每个位置的元素，运算一样。为此，其引入了位置编码，假设输入X为$R^{n*d}$ ，其中n表示序列长度，d为词元的嵌入维度。引入位置编码矩阵P，形状与X相同，使用$X+P$作为模型最终输入。

P的元素如下计算：$p_{i,2j}=sin(i/10000^{2j/d})$，$p_{i,2j+1}=cos(i/10000^{2j/d})$

为什么这么编码呢？
1. 绝对位置信息：i本身表示词元在序列中的绝对位置信息，为什么要除以$10000^{2j/d}$呢？
	1. 参考二进制编码，低位变化频率高，高位变化频率低，此处正是模拟了这一点
	2. j表示词元嵌入维度，越靠前，值越低，对应函数变化频率高
2. 相对位置信息：对于任何确定的位置偏移δ，位置i+δ处 的位置编码可以线性投影位置i处的位置编码来表示。
	1. 推导略

## Transformer
> 基于编码器-解码器架构来处理
> 不同于使用注意力的`seq2seq`，其是完全基于自注意力机制实现的

### 多头注意力
> 目的：对于同一q,k,v，希望抽取到不同的信息
> 实现：使用h个独立的注意力池化，合并各个head输出得到最终输出
![image.png](https://raw.githubusercontent.com/yzh-2002/img-hosting/main/notes/202410150956289.png)

---
假设`query`为$d_q$，`key`为$d_k$，`value`为$d_v$，则对每一个head，有参数$W_i^q:(p_q,d_q),W_i^k:(p_k,d_k),W_i^v:(p_v,d_v)$，该头的输出$h_i=f(W_i^q q,W_i^k k,W_i^v v)$，此处如果选择加性注意力则还有对应参数。

最终会将各个head的输出`concat`在一起，经过一个全连接层（参数$W_o:(p_o,hp_v)$）得到最终输出。

带掩码的多头注意力机制：
在编码器无所谓，但在解码器，其对序列中一个元素输出时，不应该考虑该元素之后的元素，故需要进行掩码操作，也即计算$x_i$输出时，假装当前序列长度为i
- `使用attention的seq2seq`中

### PositionWiseFFN
> 基于位置的前馈网络，实现起来就是全连接层

`PositionWise`如何理解呢？

该FFN仅对多头注意力层输出的最后一个维度进行运算，而最后一个维度表示的是序列数据在不同时刻（或者说不同位置）的经注意力层的运算结果。

**感觉还是稍微有点不太清晰啊。**

### LayerNorm
> 较之`BatchNorm`在序列数据中表现更好
![image.png](https://raw.githubusercontent.com/yzh-2002/img-hosting/main/notes/202410151244467.png)
---

`BatchNorm`是在特征维度进行标准化（在长序列数据中由于不同数据有效长度不同，其均值和方差波动比较大，故效果不好），`LayerNorm`则是对每个样本数据进行标准化。


### Overview
![image.png](https://raw.githubusercontent.com/yzh-2002/img-hosting/main/notes/202410151305151.png)

---
编码器中多头注意力机制是自注意力的，但是解码器的多头注意力机制不是，其query来自目标序列，key和value来自编码器的输出。


