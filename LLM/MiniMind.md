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




