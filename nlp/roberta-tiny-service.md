---
description: 基于roberta-tiny的句向量服务
---

# roberta-tiny service



* 背景介绍

    google发布的大规模预训练模型bert在nlp诸多任务fine-tune后刷新了SOTA，后续有许多工作跟随着bert进行开展。其中roberta以更大规模的训练语料，更久的训练时间，并在预训练阶段取消了NSP\(Next Sentence Predict\)，进一步提高了模型的准确度。但是无论是bert还是roberta，它们的参数量巨大（110M），对服务器gpu的资源占用率很高，模型推理速度欠佳。因此我们考虑采用更轻量级的roberta-tiny模型，开源自[https://github.com/ZhuiyiTechnology/pretrained-models](https://github.com/ZhuiyiTechnology/pretrained-models). 

* roberta-tiny

    【配置】 4层模型，hidden size为384，对Embedding层做了低秩分解\(384-&gt;128-&gt;384\)，可以用[bert4keras](https://github.com/bojone/bert4keras/tree/master/examples)加载使用。

    【训练】 使用[bert4keras](https://github.com/bojone/bert4keras/tree/master/pretraining)在TPU v3-8上训练，使用带梯度累积的LAMB优化器，批大小为800，累积4步更新，相当于以批大小3200训练了125k步（前3125步为warmup）。

    【备注】 速度跟[albert tiny](https://github.com/brightmart/albert_zh)一致，普通分类性能也基本一致，但由于roberta模型并没有参数共享这个约束，所以在生成式任务等复杂任务上效果优于albert tiny。

* 预训练——小说长文本场景下的预训练

    由ZhuiyiTechnology开源的roberta-tiny预训练模型，在百度百科的训练语料基础上增加了新闻领域的数据，总共35G的训练语料。针对小说网文领域和通用领域存在一定gap的情况，我们考虑在加载已有的预训练参数后，继续在米读小说数据集上进行pre-training，以获得更加适合米读小说的语言模型。在预训练过程中，对比roberta我们做了三个改进：1. 采取多句拼接，并且来源于统一连续文本；2. 以词维度进行mask；3. 采用lamd优化器。

    【训练语料】米读小说中已有评级的书籍前100章内容。（42G）

    【预处理】bert推荐的预训练方式是将以句子为单位输入训练语料。但是在小说内容中，每个句子的长度差异比较大，以一个句子为单位不仅浪费训练资源，对模型的训练也造成了一定的困扰。因此这里采用了将多个句子进行拼接，使每一个拼接的句子长度都接近512但不超过512（在fine-tune的阶段我们也采取该策略，保持pre-training和fine-tune阶段的输入一致性），最后拼接而成的句子，都是来自于同一章节且连续的句子（保证了章节维度的语义等内容信息）。使用jieba\_fast分词后，以词维度进行随机mask，生成10份随机mask的tfrecord格式的语料文件。

    【训练参数】采用[lamd](https://arxiv.org/abs/1904.00962)优化器，learning\_rate=0.00176，epoch=200诸多其他参数就不一一介绍。

    【训练结果】在两块GPU V100上训练两周，loss=2.5648766424179077,  mlm\_acc=0.5974433,  mlm\_loss=1.9674323。

* 句向量特征提取

    roberta-tiny预训练结束后，我们将其参数固定，利用语言模型对句子进行向量化编码。客户端输入一段小说内容，返回一个维度为384的向量，该向量能够很好地表征句子段落信息。通过[bert-as-service](https://github.com/hanxiao/bert-as-service)可以很方便地将模型部署在服务器上，而且提供了友好的客户端访问接口。为了方便部署&管理，我们使用docker部署bert-as-service服务。docker run --gpus '"device=0"' -p 5555:5555 -p 5556:5556 -itd roberta-tiny，启动容器。

* 接口调用（临时方案）

    qa\_domain: 172.25.25.19

    prd\_domain: 172.29.4.142

    调用方式参照[bert-as-service](https://github.com/hanxiao/bert-as-service)

* 结果展示

\#\# TODO

* 性能评估

\#\# TODO

