# Python编码

### 大文件的读取

- 读取大几G的大文件，可以利用生成器 `generator`
- 对可迭代对象`file`，进行迭代遍历：`for line in file`，会自动地使用缓冲IO（bufferedIO）以及内存管理，而不必担心任何大文件的问题。
```python
with open('filename') as file:
	for line in file:
do_things(line)
```

### 按字典中的value值进行排序

```python
d = {'a': 22, 'b': 44, 'c': 11, 'd': 33}
sorted(d.items(), key=lambda x: x[1])
```


### 斐波那契数列

> 斐波那契数列：简单地说，起始两项为 0 和 1，此后的项分别为它的前两项之和。

```python
def fibo(num):
	"""使用列表存储斐波拉契数字"""
    numList = [0, 1]
    
    for i in range(num - 2):
        numList.append(numList[-2] + numList[-1])

    return numList


f = fibo(10000)
for index in f:
    print(index)
```

```python
def g_fibo(n):
    """使用生成器保存斐波拉契数列"""
    i = 0
    left = right = 1
    while i < n:
        ret = left
        left, right = right, left+right
        i += 1
        yield ret
    return 'done'


f = g_fibo(10000)
for index in f:
    print(index)
```

### a=1, b=2, 不用中间变量交换a和b的值（两种方法）
```python
a, b = b, a
```

```python
a = a + b
b = a - b
a = a - b
```


### 求今天是本年、本月、本周的第几天
```python
# -*- coding:utf-8 -*-
import datetime

date = datetime.datetime.now()
date_list = date.strftime('%Y-%m-%d').split('-')

year_index = int(date.strftime('%j'))
month_index = date_list[2]
week_index = date.weekday() + 1

print("是本年第{}天".format(year_index))
print('是本月第{}天'.format(month_index))
print('是本周第{}天'.format(week_index))
```

### 去除列表里面最大值

```python
import numpy

ret = [1, 3, 5, 7]
ret.remove(numpy.max(ret))
```


### 如何编写有序字典

```python
from collections import OrderedDict

order_dict = OrderedDict()  # 创建有序字典
order_dict['name'] = 'xiaoming'  # 加入键值对
print(order_dict)  # 显示有序字典

order_dict.clear()  # 清空有序字典
print(order_dict)  # 显示有序字典
print('\n'.join(dir(OrderedDict)))  # 列出所有的成员函数
```

### 有一个已经排好序的数组 num_list = [0, 10, 20, 30, 40, 50]. 现输入一个数， 要求按原来的规律将它插入数组中

```python
num_list = [0, 10, 20, 30, 40, 50]
n = int(input('Input a number:'))

for i in range(len(num_list)):
    if n < num_list[i]:
        num_list.insert(i, n)
        break

print('The new sorted list is:{}'.format(num_list))
```

### 解释 `列表推导式` 和 `生成器` 区别

```python
nums = [lambda x: i*x for i in range(4)]
print([f(2) for f in nums])

# result
[6, 6, 6, 6]
```

- 列表推导式保存了4个函数对象
- 每次生成局部变量`i`后，由于没有保存，后续的`i`覆盖了之前的`i`
- 最终`i` 保存的值是3

```python
nums = (lambda x: i*x for i in range(4))
print([f(2) for f in nums])

# result
[0, 2, 4, 6]
```

- 生成器拥有"惰性计算"的特点
- 从`nums`中取值时才会计算`i`的值，并立即把最终的计算结果赋值给f，再计算`f(2)`，追加到列表中
- 然后调用生成器的 `__next__()` 方法获取下一个函数对象（此时已经丢弃上一个函数对象）
- 每次处理一个对象，而不是一口气处理和构造整个数据结构，这样做的潜在优点是可以节省大量的内存

### 列表、列表推导式、生成器占用内存对比

```python
In [1]: import sys

In [2]: a = range(1000000)

In [3]: b = list(range(1000000))

In [4]: c = [x for x in range(1000000)]

In [5]: f = sys.getsizeof

In [6]: f(a)
Out[6]: 48

In [7]: f(b)
Out[7]: 9000112

In [8]: f(c)
Out[8]: 8697464
```


### 标准迭代器

```python
class Data(object):
    def __init__(self, *args):
        self._data = list(args)
        self._index = 0

    def __iter__(self):
        return self

    def __next__(self):
        return self.next()

    def next(self):
        if self._index >= len(self._data):
            raise StopIteration()
        d = self._data[self._index]
        self._index += 1
        return d


data = Data(1, 2, 3, 4, 5)

for i in data:
    print(i)
```
