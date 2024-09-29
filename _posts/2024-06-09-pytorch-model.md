---
layout: post
title: 【Pytorch】模型API讲解
date: 2024-06-09
tags: [pytorch]
author: VOID*
---

# eval()和train()

在Pytorch中，模型有两种状态：`eval()`和`train()`。区别是什么呢？这要从dropout和batch normalization说起。

- dropout：在训练的时候，dropout会随机地将一些神经元的输出置为0，这样可以防止过拟合。
- batch normalization：在训练的时候，batch normalization会对每个batch的数据进行归一化，这样可以加速训练。

但到了评估的时候，这两个就得关闭。所以需要使用：

```python
model.eval()
```

# with torch.no_grad()
