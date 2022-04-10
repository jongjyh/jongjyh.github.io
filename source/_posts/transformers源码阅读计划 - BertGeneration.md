# transformers源码阅读计划 - BertGeneration

 transformers是一个基于Pytorch的NLP框架，集成了模型，训练器，tokenizer一系列工具，而且都封装得非常好。更重要的是这个库也复现了许多非常经典的模型比如Bert，XLNet，所以开始一个transformers的源码阅读计划，一方面提高自己的代码复现能力，一边学习transformers内部的结构。

## BertGeneration

[Papers](https://arxiv.org/abs/1907.12461) 

BertGeneration提出用Bert同时充当decoder角色来充分利用预训练成本，具体来说非常简单，按照transformer的思想，encoder和decoder的差别不过多了一层cross attention layer。

![image-20220409193520425](/Users/noah/Library/Application Support/typora-user-images/image-20220409193520425.png)

![image-20220409193920098](https://tva1.sinaimg.cn/large/e6c9d24ely1h13pd91ihij216e0oi459.jpg)

- 论文还对比了不同情况下这种以Bert为原型的Seq2Seq架构在机器翻译上的效果，发现共享参数的效果也不错

## 源码

**源码路径（models/bert_generation/modeling_bert_generation.py）**

模型由五个类构成：

``` Python
class BertGenerationEmbeddings(nn.Module):
# bert的word_embedding
class BertGenerationPreTrainedModel(PreTrainedModel):
# 一个用来保存和恢复checkpoint的抽象类，继承了transformers内部的独有类PreTrainedModel
class BertGenerationEncoder(BertGenerationPreTrainedModel):
# 最重要的类，可以根据args 变成 encoder or decoder
class BertGenerationOnlyLMHead(nn.Module):
# decoder的分类头，用来做vocab的分类的
class BertGenerationDecoder(BertGenerationPreTrainedModel):
# encoder的行为和decoder的行为不同，用一个decoder类定义decoder的行为
```

## **BertGenerationEncoder**

transformers内部的模型其实都是nn.Module的子类，重点关注__\__init__\__和**forward**两个函数

- **init**

```python
def __init__(self, config):
  super().__init__(config)
  # config是模型的参数类，定义了模型的一些超参
  self.config = config
  # embeddings 使用了自己定义的embeddings，encoder直接使用了bert内部的encoder
  self.embeddings = BertGenerationEmbeddings(config)
  # is_decoder在config被传入BertEncoder
  self.encoder = BertEncoder(config)

  # Initialize weights and apply final processing
  self.post_init()
```

- BertEncoder & BertLayer

Encoder这个类主要是判断一下是否传入了`is_decoder & add_crossattention`这两参数，`True`时行为差异会比较不同，具体来说

- Bert Generation这个类复用了transforemrs内部的bert实现`BertEncoder`&`BertLayer`

  ```python
  class BertEncoder(nn.Module):
      def __init__(self, config):
          super().__init__()
          self.config = config
          self.layer = nn.ModuleList([BertLayer(config) for _ in range(config.num_hidden_layers)])
          self.gradient_checkpointing = False
  ```

  layer使用ModuleList容器包装起来，num_hidden_layers决定了这样的BertLayer有几层，在forward中会用for循环展开进行前向计算

- `BertLayer`也就是Bert-base的12层layer中的layer实现，当is_decoder传入时会在最末端增加一层cross-attention

```python
class BertLayer(nn.Module):
    def __init__(self, config):
        super().__init__()
        self.chunk_size_feed_forward = config.chunk_size_feed_forward
        self.seq_len_dim = 1
        # self-attention
        self.attention = BertAttention(config)
        self.is_decoder = config.is_decoder
        self.add_cross_attention = config.add_cross_attention
        # 增加了cross-attention
        if self.add_cross_attention:
            if not self.is_decoder:
                raise ValueError(f"{self} should be used as a decoder model if cross attention is added")
            # 发现cross-attention和self-attention使用的是同一个attention类说明进行了复用
            self.crossattention = BertAttention(config, position_embedding_type="absolute")
        # layer中的全连接层
        self.intermediate = BertIntermediate(config)
        self.output = BertOutput(config)
```

```python
        if self.is_decoder and encoder_hidden_states is not None:
            if not hasattr(self, "crossattention"):
                raise ValueError(
                    f"If `encoder_hidden_states` are passed, {self} has to be instantiated with cross-attention layers by setting `config.add_cross_attention=True`"
                )

            # cross_attn cached key/values tuple is at positions 3,4 of past_key_value tuple
            cross_attn_past_key_value = past_key_value[-2:] if past_key_value is not None else None
            cross_attention_outputs = self.crossattention(
                attention_output, # self-attn的输出做cross-attn的Q输入
                attention_mask,
                head_mask,
                encoder_hidden_states,# 来自encoder的隐藏层做K与V的输入
                encoder_attention_mask,
                cross_attn_past_key_value,
                output_attentions,
            )
            attention_output = cross_attention_outputs[0]
            outputs = outputs + cross_attention_outputs[1:-1]  # add cross attentions if we output attention weights
```

- **forward的输入**

```python
def forward(
    self,
    input_ids=None,
    attention_mask=None,
    position_ids=None,
    head_mask=None,# 前四个都是tokenizer的输出
    inputs_embeds=None,# 可以直接输入embeding而不是ids，input_ids,inputs_embeds只能输入一个
    encoder_hidden_states=None,# 作为decoder时会使用，Cross-Attn的KV
    encoder_attention_mask=None,# 作为decoder时会使用，用来屏蔽[PAD]
    past_key_values=None,# 作为decoder时会使用，预先算好的所有BertLayer的KV
    use_cache=None,# 作为encoder时，True会保留所有的BertLayer的KV然后供decoder使用也就是上面的arg
    output_attentions=None,
    output_hidden_states=None,
    return_dict=None,# 输出时调整输出格式的一些bool参数，详细可以去看doc
):
```

forward函数也只是简单对encoder进行计算一下然后返回结果

```python
embedding_output = self.embeddings(
  input_ids=input_ids,
  position_ids=position_ids,
  inputs_embeds=inputs_embeds,
  past_key_values_length=past_key_values_length,
)

encoder_outputs = self.encoder(
  embedding_output,
  attention_mask=extended_attention_mask,
  head_mask=head_mask,
  encoder_hidden_states=encoder_hidden_states,# 当is_decoder=True时以下3个参数会派上用场
  encoder_attention_mask=encoder_extended_attention_mask,
  past_key_values=past_key_values,
  use_cache=use_cache,
  output_attentions=output_attentions,
  output_hidden_states=output_hidden_states,
  return_dict=return_dict,
)
```

## BertGenerationDecoder

`BertGenerationDecoder`只是对`BertGenerationEncoder`更高一层的封装

- __init__

```python
def __init__(self, config):
  super().__init__(config)

  if not config.is_decoder:
    logger.warning("If you want to use `BertGenerationDecoder` as a standalone, add `is_decoder=True.`")
    # 复用了BertGenerationEncoder，通过config.is_decoder调整BertGenerationEncoder的行为
    self.bert = BertGenerationEncoder(config)
    self.lm_head = BertGenerationOnlyLMHead(config)
```

- __forward__

```python
outputs = self.bert(
  input_ids,
  attention_mask=attention_mask,
  position_ids=position_ids,
  head_mask=head_mask,
  inputs_embeds=inputs_embeds,
  encoder_hidden_states=encoder_hidden_states,
  encoder_attention_mask=encoder_attention_mask,
  past_key_values=past_key_values,
  use_cache=use_cache,
  output_attentions=output_attentions,
  output_hidden_states=output_hidden_states,
  return_dict=return_dict,
)
# 输入的每个token的last hidden layer
sequence_output = outputs[0]
# hidden layer输入vocab的分类头计算每个token的预测值然后存在logits输出
prediction_scores = self.lm_head(sequence_output)
```

## 模型测试

``` python
from transformers import BertGenerationTokenizer, BertGenerationDecoder, BertGenerationConfig
import torch

tokenizer = BertGenerationTokenizer.from_pretrained('google/bert_for_seq_generation_L-24_bbc_encoder')
config = BertGenerationConfig.from_pretrained("google/bert_for_seq_generation_L-24_bbc_encoder")
config.is_decoder = True
model = BertGenerationDecoder.from_pretrained('google/bert_for_seq_generation_L-24_bbc_encoder', config=config)

inputs = tokenizer("How are you", return_token_type_ids=False, return_tensors="pt")
tokenizer.pad_token
outputs = model(**inputs)

prediction_logits = outputs.logits
prediction_logits.shape
```

```python
>>> normalizer.cc(51) LOG(INFO) precompiled_charsmap is empty. use identity normalization.
>>> Some weights of BertGenerationDecoder were not initialized from the model checkpoint at google/bert_for_seq_generation_L-24_bbc_encoder and are newly initialized: ['lm_head.bias', 'lm_head.decoder.weight', 'lm_head.decoder.bias']
>>>You should probably TRAIN this model on a down-stream task to be able to use it for predictions and inference.
>>> torch.Size([1, 3, 50358])
```

最终得到的就是"How are you"对每个vocab的概率

## 总结

- transformers的模型封装的很好，但是因为复用性很强，一个attention模块可以同时充当self-attention和cross-attention使用，这样就导致了阅读源码时会有很多细节需要处理比如每个入参出参的含义，这就增加了阅读源码时的工作量
- 优势
  - 模型多
  - 复用性强
  - 封装完善
  - 几乎是原生Pytorch实现
- 劣势
  - 只是实现了比较经典的模型
  - 阅读源码需要对每个模块很熟悉
  - 封装的过于完善也会导致拓展性差