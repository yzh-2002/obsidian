## Model
> 大模型的结构：
> 1. Dense / Moe结构：指的是Transformer块中经过attention之后是FFN or Moe
> 2. Decoder-only / Encoder-only / Encoder-Decoder结构
> 3. ...

### Attention

KV cache原理：
1. Transformer的Decoder中，每生成一个新的token，都需要重新计算所有历史 token 的注意力权重。若不缓存 K 和 V，每次生成新 token 时需重新计算整个序列的 K 和 V，导致计算量随序列长度呈二次增长。
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

MHA中，记token的




### Position 


### Moe




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





