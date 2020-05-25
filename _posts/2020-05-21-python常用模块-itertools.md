---
toc: true
layout: post
description: python itertools模块使用笔记。
categories: [python]
---
# python常用模块笔记：itertools

itertools模块提供了一系列被称作"iterator algebra"的迭代方法，能够简洁、高效的实现一些非常有用的数据迭代操作。简单说，使用"iterator algebra"的原因就是：高效、快速。

## iter应用举例：从一个列表中依次取出固定数目的元素

```python
def better_grouper(inputs, n):
  iters = [iter(inputs)]*n
  return zip(*iters)
```

原理：

> 列表复制的方法首先让我们获得了inputs的n份引用拷贝对象。然后在zip函数中，当从iters中的第一个iter对象中获取了第一个元素后，第二个iter的第一个元素实际上变成了第二个元素，依次类推，从而实现了连续取出固定数目的元素。

上面的better_grouper有个问题，如果inputs的长度不是n的整数倍，最后剩余的少于n的元素不能获得，因为zip是以最短的iter对象为准，当zip中的一个iter对象没有元素时，zip自动停止。如果想获得全部元素，可以使用zip_longest()函数。zip_longest函数有个fillvalue参数，可以指定对于缺失值自动填充特定元素。

## itertools 常用函数

### 获取元素组合

* combinations

生成可迭代对象元素的不重复组合。

```python
>>> combinations([1,2,3], 2)
(1,2), (1,3), (2,3)
```

* combinations_with_replacement

生成可迭代对象的元素组合（元素可重复）。

```python
>>> combinations_with_replacement([1, 2], 2)
(1, 1), (1, 2), (2, 2)
```

* permutations

生成可迭代对象的有序组合。

```python
>>> permutations('abc')
('a', 'b', 'c'), ('a', 'c', 'b'), ('b', 'a', 'c'),
('b', 'c', 'a'), ('c', 'a', 'b'), ('c', 'b', 'a')
```

* product

生成可迭代对象的笛卡尔积。

```python
>>> product([1, 2], ['a', 'b'])
(1,'a'), (1, 'b'), (2, 'a'), (2, 'b')
```

### 无限循环生成元素

* count

count(start=0, step=1)函数接收两个参数，能够从start开始，按照步长step不断产生新的元素。

```python
>>> count()
0, 1, 2, 3, 4, ...

>>> count(start=1, step=2)
1, 3, 5, 7, 9, ...
```

* cycle

cycle(iterable)可以把iterable中的元素循环重复下去。

```python
>>> cycle(['a', 'b', 'c'])
a, b, c, a, b, c, a, ...
```

* repeat

repeat(object, times=1)将不断重复产生object，重复次数取决于times参数。

```python
>>> repeat(2, 5) 
2, 2, 2, 2, 2
```

### 串联多个可迭代对象

* chain

chain(*iterables) 该函数创建一个新的迭代器，会将参数中的所有迭代器全包含进去。

```python
>>> list(it.chain('ABC', 'DEF'))
['A' 'B' 'C' 'D' 'E' 'F']

>>> list(it.chain([1, 2], [3, 4, 5, 6], [7, 8, 9]))
[1, 2, 3, 4, 5, 6, 7, 8, 9]
```

### 元素分组

* groupby

groupby(iterable, key=None) 分组函数，将 key 函数作用于序列的各个元素。根据 key 函数的返回值将拥有相同返回值的元素分到一个新的迭代器。类似于 SQL 中的 GROUP BY 操作，唯一不同的是该函数对序列的顺序有要求，因为当 key 函数的返回值改变时，迭代器就会生成一个新的分组。因此在使用该函数之前需要先使用同一个排序函数对该序列进行排序操作。

```python
>>> data = [{'name': 'Alan', 'age': 34},
...         {'name': 'Catherine', 'age': 34},
...         {'name': 'Betsy', 'age': 29},
...         {'name': 'David', 'age': 33}]
...
>>> grouped_data = it.groupby(data, key=lambda x: x['age'])
>>> for key, grp in grouped_data:
...     print('{}: {}'.format(key, list(grp)))
...
34: [{'name': 'Alan', 'age': 34}, {'name': 'Betsy', 'age': 34}]
29: [{'name': 'Catherine', 'age': 29}]
33: [{'name': 'David', 'age': 33}]
```

实际使用中记得先排序后分组

```python
def sort_and_group(iterable, key=None):
    return it.groupby(sorted(iterable, key=key), key=key)
```

### 元素选择

* takewile

takewhile(predicate, iterable)创建一个迭代器，当遇到predicate为false的时候停止迭代元素。

```python
it.takewhile(lambda x: x < 3, [0, 1, 2, 3, 4])  # 0, 1, 2
```

* dropwhile

dropwhile(predicate, iterable)创建一个迭代器，当predicate首次为false的时候开始迭代元素。

```python
it.dropwhile(lambda x: x < 3, [0, 1, 2, 3, 4])  # 3, 4
```

* filterfalse

filterfalse(predicate, iterable) 创建一个迭代器，返回 iterable 中 predicate 为 false 的元素。

```python
>>> only_positives = it.filterfalse(lambda x: x <= 0, [0, 1, -1, 2, -2])
>>> list(only_positives)
[1, 2]
```

* islice

islice(iterable, start, stop[, step]) 对 iterable 进行切片操作。从 start 开始到 stop 截止，同时支持以步长为 step 的跳跃。

```python
>>> # Slice from index 2 to 4
>>> list(it.islice('ABCDEFG', 2, 5)))
['C' 'D' 'E']

>>> # Slice from beginning to index 4, in steps of 2
>>> list(it.islice([1, 2, 3, 4, 5], 0, 5, 2))
[1, 3, 5]

>>> # Slice from index 3 to the end
>>> list(it.islice(range(10), 3, None))
[3, 4, 5, 6, 7, 8, 9]

>>> # Slice from beginning to index 3
>>> list(it.islice('ABCDE', 4))
['A', 'B', 'C', 'D']
```

主要参考资料：

1. [itertools in python 3, by example](https://realpython.com/python-itertools/)

