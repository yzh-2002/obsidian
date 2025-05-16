## Tokenizers

Q：有哪些tokenizer
1. ~~Word-based Tokenization，原理：将文本按空格或标点分割成完整单词~~
	1. 缺点：词表很大，很容易OOV
2. ~~Character-based Tokenization，原理：将文本拆分为单个字符~~
	1. 缺点：词表很小，但是预料对应token序列极长
3. Subword Tokenization，核心思想：平衡词表大小与OOV问题，将单词拆分为更小的语义单元（子词）
	1. BPE（Byte-Pair Encoding），原理：从字符开始，逐步<font color="#ff0000">合并</font>最高频的相邻符号对。
	2. BBPE（Byte-level BPE）
	3. WordPiece，原理：类似BPE，但合并策略基于概率（$\frac{Nums(ab)}{Nums(a)*Nums(b)}$）而非频率。
	4. SentencePiece，原理：将文本视为Unicode序列，无需<font color="#ff0000">预分词</font>，基于BPE / Unigram算法

Q：预分词的作用？
1. 作用在Subword Tokenization之前，先将文本初步切分为粗粒度的单元，便于其操作
2. 常见的预分词器：
	1. Whitespace：按空格切分文本
	2. ByteLevel：将文本转换为<font color="#ff0000"> UTF-8 字节序列</font>，按字节处理所有字符，无需预分词。

自己从头训练一个tokenizer：
```python
from tokenizers import (
    decoders,
    models,
    pre_tokenizers,
    trainers,
    Tokenizer,
)
tokenizer = Tokenizer(models.BPE())
tokenizer.pre_tokenizer = pre_tokenizers.ByteLevel(add_prefix_space=False)
special_tokens = ["<|endoftext|>", "<|im_start|>", "<|im_end|>"]
trainer = trainers.BpeTrainer(
        vocab_size=6400,
        special_tokens=special_tokens,  # 确保这三个token被包含
        show_progress=True,
        initial_alphabet=pre_tokenizers.ByteLevel.alphabet()
	)
texts = read_texts_from_jsonl(data_path)
tokenizer.train_from_iterator(texts, trainer=trainer)
```

Q：chat_template作用？
A：定义了**多轮对话的格式化规则**，用于将对话历史转换为模型可理解的输入格式。


tokenizer_config.json
```json
config = {
        "add_bos_token": False,
        "add_eos_token": False, // 是否在末尾自动添加eos_token
        "add_prefix_space": False, // 是否在开头添加空格
        "added_tokens_decoder": { // 特殊标记解码位置
            "0": {
                "content": "<|endoftext|>",
                "lstrip": False, // 是否移除该标记左侧空格
                "normalized": False,
                "rstrip": False,
                "single_word": False,
                "special": True
            },
            "1": {
                "content": "<|im_start|>",
                "lstrip": False,
                "normalized": False,
                "rstrip": False,
                "single_word": False,
                "special": True
            },
            "2": {
                "content": "<|im_end|>",
                "lstrip": False,
                "normalized": False,
                "rstrip": False,
                "single_word": False,
                "special": True
            }
        },
        "additional_special_tokens": [],
        "bos_token": "<|im_start|>", // begin of sequence
        "clean_up_tokenization_spaces": False,
        "eos_token": "<|im_end|>",
        "legacy": True,
        "model_max_length": 32768, // 模型接受的最大输入长度，32K
        "pad_token": "<|endoftext|>",
        "sp_model_kwargs": {},
        "spaces_between_special_tokens": False,
        "tokenizer_class": "PreTrainedTokenizerFast",
        "unk_token": "<|endoftext|>",
        "chat_template": "{% if messages[0]['role'] == 'system' %}{% set system_message = messages[0]['content'] %}{{ '<|im_start|>system\\n' + system_message + '<|im_end|>\\n' }}{% else %}{{ '<|im_start|>system\\nYou are a helpful assistant<|im_end|>\\n' }}{% endif %}{% for message in messages %}{% set content = message['content'] %}{% if message['role'] == 'user' %}{{ '<|im_start|>user\\n' + content + '<|im_end|>\\n<|im_start|>assistant\\n' }}{% elif message['role'] == 'assistant' %}{{ content + '<|im_end|>' + '\\n' }}{% endif %}{% endfor %}"
    }
```

## PretrainedConfig
`PretrainedConfig` 是 Hugging Face Transformers 库中的一个核心类，用于存储和管理预训练模型（如 BERT、GPT 等）的配置信息。它定义了模型的结构参数（如层数、注意力头数、隐藏层维度等），并提供了从预训练模型加载配置或自定义配置的方法。

`PretrainedConfig`是下面代码中`AutoConfig，BertConfig`的基类，定义了所有配置的通用方法

```python
from transformers import AutoConfig
# 从 Hugging Face Hub 加载预训练模型的配置（例如 bert-base-uncased）
config = AutoConfig.from_pretrained("bert-base-uncased")
print(config.hidden_size)  # 输出 768（BERT-base 的隐藏层维度）
print(config.num_attention_heads)  # 输出 12（BERT-base 的注意力头数）


from transformers import BertConfig, BertModel
# 创建自定义配置
custom_config = BertConfig(
    vocab_size=30522,
    hidden_size=1024,  # 修改隐藏层维度
    num_hidden_layers=12,
    num_attention_heads=16,
    intermediate_size=4096,
)
# 用自定义配置初始化模型，此时模型权重是随机初始化的（非预训练权重）
custom_model = BertModel(custom_config)


class MyModelConfig(PretrainedConfig):
    model_type = "my_model"  # 必须定义，用于自动识别模型类型
    def __init__(
        self,
        vocab_size=30522,
        hidden_size=768,
        num_layers=12,
        custom_param=1024,  # 自定义参数
        **kwargs
    ):
        super().__init__(**kwargs)
        self.vocab_size = vocab_size
        self.hidden_size = hidden_size
        self.num_layers = num_layers
        self.custom_param = custom_param  # 新参数
class MyCustomModel(PreTrainedModel):
    config_class = MyModelConfig  # 绑定配置类
    def __init__(self, config):
        super().__init__(config)
        self.embeddings = nn.Embedding(config.vocab_size, config.hidden_size)
        self.layers = nn.ModuleList([
            nn.Linear(config.hidden_size, config.hidden_size)
            for _ in range(config.num_layers)
        ])
        self.custom_layer = nn.Linear(config.hidden_size, config.custom_param)
    def forward(self, input_ids):
        x = self.embeddings(input_ids)
        for layer in self.layers:
            x = layer(x)
        x = self.custom_layer(x)
        return x
config = MyModelConfig(
    vocab_size=30522,
    hidden_size=1024,
    num_layers=6,
    custom_param=512  # 自定义参数
)
model = MyCustomModel(config)
```

常见参数：
1. 基础参数：
	1. `dropout`
	2. `bos/eos_token_id`
	3. `hidden_size`：token的emb向量纬度（大模型中所有隐含层的输出纬度均一致）
	4. `hidden_act`
	5. `num_hidden_layers`：`transformer`块的数量
	6. `vocab_size`
	7. `max_position_embeddings`：定义了模型能够处理的最大序列长度
	8. ``
2. 注意力参数
	1. `num_attention_heads`
	2. `num_key_value_heads`：KV注意力头的数量（GQA / MQA）
		1. MHA中Q，K，V注意力头的数量相等，均为`num_attention_heads`
		2. MQA中，Q的数量为`num_attention_heads`，K，V的数量为1
	3. `flash_attn`：是否使用Flash Attention算法（2022年提出）[原理](https://zhuanlan.zhihu.com/p/668888063)
	4. `rope_theta`：RoPE（旋转位置编码）的参数，控制RoPE中的基础旋转频率
3. MoE参数：
	1. `use_moe`
	2. `num_experts_per_tok`：每个token选择的专家数量
	3. `n_routed_experts`：可路由的专家总数
	4. `n_shared_experts`
	5. `scoring_func`：专家选择的评分函数，默认为`softmax`
	6. `aux_loss_alpha`：辅助损失的权重参数
	7. `seq_aux`：是否在序列级别计算辅助损失
	8. `norm_topk_prob`：是否对top-k概率进行归一化