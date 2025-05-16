## Model
> 大模型的结构：
> 1. Dense / Moe结构：指的是Transformer块中经过attention之后是FFN or Moe
> 2. Decoder-only / Encoder-only / Encoder-Decoder结构
> 3. ...

### Attention
> $Attention(Q,K,V) = softmax(\frac{QK^T}{\sqrt{d}})V$

1. attention层的输入维度：`[batch_size, token_num, d_model]`
	1. 一般设计上会保持embedding向量与隐向量维度一致
2. 经经线性层生成Q，K，V向量，维度：`[batch_size, token_num, d_model/h]`
	1. h为MHA中head的数量
3. 计算Q和K的点积，得到注意力分数权重，维度：`[batch_size, token_num, token_num]`
	1. `T[*][i][j]`：表示第i个token与第j个token的相似度
4. 掩码，为了保证自回归属性，需要使得所有`T[*][i][j]=0 when j>i`设置为`1e-9`，也即下三角矩阵，这样经softmax之后对应的注意力分数均为0
5. softmax
6. 加权求和，输出维度：`[batch_size, token_num, d_model/h]`
7. 多个head的输出concat一起


KV cache原理：
1. Transformer的Decoder中，每生成一个新的token，都需要重新计算所有历史 token 的注意力权重。若不缓存 K 和 V，每次生成新 token 时需重新计算整个序列的 K 和 V。
2. KV cache则是，对输入序列的所有 token 计算对应的 K 和 V，并存入缓存，后续生成每个新 token 时，仅需计算当前 token 的 Q，而 K 和 V 直接从缓存中读取。
3. 空间换时间，节省了token embedding经线性层输出K，V的时间，但增加了内存消耗。

优化方案：
1. 显存压缩技术：
	1. 稀疏化
	2. 量化：将浮点数转换为低精度格式
	3. 分页存储（PageAttention）
2. 计算优化策略：
	1. MQA / GQA
	2. 滑动窗口
3. ...

---
1. MHA（Multi-Head Attention）
	1. 代表：Bert，GPT-3早期版本
2. GQA（Grouped-Query Attention）
	1. 代表：LlaMa-2
	2. 原理：多头注意力机制中一些head的权重分组，其中<font color="#ff0000">K，V的权重相同</font>，这样就只需存一份就好，提高了效率，但模型效果会打折扣
3. MQA（Multi-Query Attention）
	1. 代表：Gemini
	2. 原理：等价于只有一个分组的MQA，共享的 K 和 V 无法像 MHA 那样充分捕捉不同头所关注的多样化信息，使得模型在处理复杂语义关系时能力有所减弱。
4. MLA（Multi-head Latent Attention）
	1. 代表：DeepSeek-V2

低秩投影：
1. 核心思想是寻找一个低秩矩阵（秩远小于原始矩阵维度）来压缩或表示高维数据。
2. 例如：高维数据X纬度为$d \times k$，其低秩投影可分解为两个小矩阵A（$d\times r$），B（$r\times k$），有¥$X=AB$，且$r << k$，从而将参数量从$d\times k$降低至$r\times(d+k)$

![image.png](https://raw.githubusercontent.com/yzh-2002/img-hosting/main/cs/202505111505561.png)

---
1. 引入低秩压缩矩阵$W^{DKV}$，缓存压缩过的K，V向量（减小了缓存大小）
2. 推理时，再通过解压矩阵还原K，V向量计算注意力分数（~~引入了额外计~~算）
	1. $attention=softmax(\frac{QK^T}{\sqrt{d}})V=softmax(\frac{XW^{Q}(C^{KV}W^{UK})^T}{\sqrt{d}})V=softmax(\frac{XW^{Q}W^{UK^T}C^{KV^T}}{\sqrt{d}})V$
	2. 推理时（参数固定），可以提前计算出$W^{QUK}=W^QW^{UK^T}$，解决了额外计算的问题
	3. 如何兼容RoPE？考虑RoPE的attention实质上为：$softmax(\frac{XW^{Q}R_iR_j^TW^{UK^T}C^{KV^T}}{\sqrt{d}})V$
		1. 由于引入了$R_iR_j^T$，导致无法提前计算，还是有额外计算的问题
![image.png](https://raw.githubusercontent.com/yzh-2002/img-hosting/main/cs/202505112240696.png)

---
解决方案：给Q和K向量额外增加维度来表示位置信息，有点ALiBi的意思
### Position encoding
>1. [Positional Encoding（位置编码）](https://zhuanlan.zhihu.com/p/454482273)
>2. [十分钟读懂旋转编码（RoPE）](https://zhuanlan.zhihu.com/p/647109286)
>3. 

理想的position encoding特点：
1. 表示token在序列中的绝对位置
2. 序列长度相同情况下，不同序列中token的相对位置保持一致
3. 便于扩展，能兼容模型训练中未出现的序列长度
	1. 例如：使用整数0，1，... n来表示token位置就不合适，泛化性太差

---
1. 二进制向量表示
	1. 由于`d_model`足够大，故$2^{d\_model}-1$基本可以对任意序列长度进行编码
	2. 例如：序列中第14个表示为：`[0 1 1 1 0]`（假设`d_model=5`）
	3. 问题：位置向量的编码处在离散空间中，变化不连续
2. 周期函数`sin`表示
	1. 借鉴二进制<font color="#ff0000">低位变化频率快，高位变化频率慢</font>的思想，利用周期函数sin替代01
	2. 第t个token可表示为：$[sin(w_0t),sin(w_1t),...sin(w_{d\_model-1}t)],w_i=\frac{1}{10000^{i/d\_model-1}}$
		1. 频率越低，周期越长
3. RoPE（Rotary Position Embedding）
	1. 不同位置向量是可以通过线性变化转换的，这样就使得位置编码不仅能够表示绝对位置，还可以表示相对位置
	2. 我们可以利用旋转矩阵来实现，同时在表示位置时，扩展上面sin表示方法，每个分位上两两一组，例如：$[sin(w_0t),cos(w_0t),\space \space \space \space \space sin(w_1t),cos(w_1t),...sin(w_{d\_model-1}t),cos(...t)],w_i=\frac{1}{10000^{i/d\_model-1}}$
![image.png](https://raw.githubusercontent.com/yzh-2002/img-hosting/main/cs/202505112231844.png)
旋转矩阵：$R(\theta)=\begin{bmatrix} cos(\theta) & sin(\theta)  \\ -sin(\theta)  & cos(\theta)  \end{bmatrix}$，含义：向量逆时针旋转$\theta$度
旋转矩阵的性质：
1. $R(\theta_1) R(\theta_2)=R(\theta_1 + \theta_2)$
2. $R(\theta)^T=R(-\theta)$

位置编码的使用：
1. 原始transformer：在token的embedding向量处引入
2. RoPE：先经线性层得到Q，K，V，<font color="#ff0000">仅对Q和K</font>进行旋转变换引入位置信息
3. ALiBi（Attention with Linear Biases）：在$QK^T$注意力分数矩阵上添加与位置相关的bias

### MOE
> 传统的大模型较之MOE可以称之为Dense大模型
> 区别在于：将transformer块中的FFN结构替换为MOE结构
> 好处：


![image.png](https://raw.githubusercontent.com/yzh-2002/img-hosting/main/cs/202505112253682.png)




## Tokenizer
> tokenizer的选择：
> 1. 自己从头训练
> 	1. 优势：词表长度，内容可自由控制
> 	2. 劣势：压测率低，且生僻字难以覆盖
> 2. 开源大模型tokenizer
> 	1. 优势：编码压缩率高
> 	2. 劣势：词表太大

## Pretrain

预训练数据格式：
```json
{"text": "<|im_start|>鉴别一组中文文章的风格和特点，例如官方、口语、文言等。需要提供样例文章才能准确鉴别不同的风格和特点。<|im_end|> <|im_start|>好的，现在帮我一下今天的天气怎么样?今天的天气依据地区而异。请问你需要我帮你查询哪个地区的天气呢？<|im_end|> ... "}

```


## SFT

## RLHF



## 数据集

```cpp
./dataset/ 
├── dpo.jsonl (909MB) // RLHF数据集
├── lora_identity.jsonl (22.8KB) // 自我认知数据集（例如：你是谁？我是minimind...），推荐用于lora训练（亦可用于全参SFT，勿被名字局限）
├── lora_medical.jsonl (34MB) // 医疗问答数据集，推荐用于lora训练
├── pretrain_hq.jsonl (1.6GB, ✨) // 预训练数据集，整合自jiangshu科技
├── r1_mix_1024.jsonl (340MB) // DeepSeek-R1-1.5B蒸馏数据，每条数据字符最大长度为1024（因此训练时设置max_seq_len=1024）
├── sft_1024.jsonl (5.6GB) // 整合自Qwen2.5蒸馏数据（是sft_2048的子集），每条数据字符最大长度为1024
├── sft_2048.jsonl (9GB) // 整合自Qwen2.5蒸馏数据，每条数据字符最大长度为2048
├── sft_512.jsonl (7.5GB) // 整合自匠数科技SFT数据，每条数据字符最大长度为512
├── sft_mini_512.jsonl (1.2GB, ✨) // 极简整合自匠数科技SFT数据+Qwen2.5蒸馏数据（用于快速训练Zero模型），每条数据字符最大长度为512
└── tokenizer_train.jsonl (1GB) // 均来自于`匠数大模型数据集`，这部分数据相对次要，（不推荐自己重复训练tokenizer，理由如上）如需自己训练tokenizer可以自由选择数据集。
```
![image.png](https://raw.githubusercontent.com/yzh-2002/img-hosting/main/cs/202505011352035.png)





