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
