---
toc: true
description: pytorch 使用中的一些心得笔记。
categories: [pytorch]
---
# pytorch notes

## detach的作用

在optimizer的代码中，将梯度置零的方法是：

```python
p.grad.detach_()
p.grad.zero_()
```

困惑：第二行代码就可以实现置零，第一行代码用来干什么？

解答：第一行代码用于将之前的梯度从计算图中剥离。pytorch中梯度计算的方式是将之前的梯度叠加，如果不剥离梯度，只是置零，那么这个梯度一直需要保存，所以如果计算了100次，那么就需要100倍的空间来保存这些全部是零的结果，导致空间浪费。

