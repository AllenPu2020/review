# 算法

### 桶排序（最快最简单的排序）

> 桶排序的基本思想是将一个数据表分割成许多buckets，然后每个bucket 各自排序，或用不同的排序算法，或者递归的使用bucketsort算法。也是典型的divide-and-conquer分而治之的策略。它是一个分布式的排序，介于MSD基数排序和LSD基数排序之间

```python
def bucket_sort(data):
    max_num = max(data)  # 选择一个最大的数
    bucket = [0] * (max_num + 1)  # 创建一个元素全是 0 的列表, 当做桶
    sort_data = []  # 存储排序好的元素

    # 把所有元素放入桶中, 即把对应元素个数加一
    for i in data:
        bucket[i] += 1

    # 取出桶中的元素
    for j in range(len(bucket)):
        if bucket[j]:
            for y in range(bucket[j]):
                sort_data.append(j)

    return sort_data


nums = [5, 3, 6, 2, 1, 4, 9, 7, 8, 0]
print(bucket_sort(nums))
```

- 1、桶排序是稳定的
- 2、桶排序是常见排序里最快的一种, 大多数情况下比快排还要快
- 3、桶排序非常快,但是同时也非常耗空间,基本上是最耗空间的 一种排序算法

### 冒泡

> 冒泡排序的思想:每次比较两个相邻的元素,如果他们的顺序错误就把他们交换位置。

```python
def bubble_sort(data):
    for i in range(len(data)):
        for j in range(len(data)-i-1):
            if data[j] > data[j+1]:
                data[j+1], data[j] = data[j], data[j+1]


nums = [5, 3, 6, 2, 1, 4, 9, 7, 8, 0]
bubble_sort(nums)
print(nums)
```

### 快排

> 快速排序使用分治法策略来把一个序列分为两个子序列。

> 步骤：

> 1. 从数列中挑出一个元素，称为"基准"（pivot）

> 2. 重新排序数列，所有比基准值小的元素摆放在基准前面，所有比基准值大的元素摆在基准的后面（相同的数可以到任一边）。

> 3. 在这个分割结束之后，该基准就处于数列的中间位置。这个称为分割（partition）操作。

> 4. 递归地（recursive）把小于基准值元素的子数列和大于基准值元素的子数列排序。

```python
def quick_sort(data, left, right):
    # 快速排序
    if left >= right:
        return data
    key = left
    low = left
    high = right
    while left < right:
        while left < right and data[right] >= data[key]:  # 如果右边比基准小，停下
            right -= 1
        while left < right and data[left] <= data[key]:  # 如果左边比基准大，停下
            left += 1
        data[right], data[left] = data[left], data[right]  # 交换现在的左右值
    data[right], data[key] = data[key], data[right]  # left和right汇合后和基准交换

    print(data)  # 交换过程
    quick_sort(data, low, left - 1)
    quick_sort(data, left + 1, high)
    return data


nums = [5, 3, 6, 2, 1, 4, 9, 7, 8, 0]
quick_sort(nums, 0, 8)
print(nums)
```


### 排序算法的分析
- 稳定性：排序前后元素相对位置是否改变

    - 如果在对象序列中有两个对象r[i]和r[j] ,它们的排序码 k[i]==k[j] 。如果排序前后,对象 r[i]和 r[j] 的相对位置不变，则称排序算法是稳定的；否则排序算法是不稳定的。

- 时间开销：排序的时间开销可用算法执行中的数据比较次数与数据移动次数来衡量。
	- 算法运行时间代价的大略估算一般都按平均情况进行估算。对于那些受对象排序码序列初始排列及对象个数影响较大的，需要按最好情况和最坏情况进行估算

- 空间开销：算法执行时所需的附加存储


### 队列和栈

- 栈
    - 栈是一种特殊的线性表。
    - 其特殊性在于限定插入和删除数据元素的操作只能在线性表的一端进行。
    - `后进先出` (`Last In First Out`), 简称为 `LIFO表`

- 栈的基本运算(6种)：
    - 构造空栈:   `InitStack(S)`
    - 判栈空:     `StackEmpty(S)`
    - 判栈满:     `StackFull(S)`
    - 进栈:       `Push(S,x)`
    - 退栈:       `Pop(S)`
    - 取栈顶元素: `StackTop(S)` 不同于弹出，栈顶元素保留。

- 队列
    - 队列(Queue)也是一种运算受限的线性表
    - 它的运算限制与栈不同，是两头都有限制
    - **插入** 只能在表的一端进行(`只进不出`)，称为**队头**(Front)
    - **删除** 只能在表的另一端进行(`只出不进`)，称为**队尾**(rear)
    - `先进先出` (`FirstInFirstOut`), 所以队列又称作 `FIFO表` 

- 队列的基本运算(6种)：
    - 置空队：    `InitQueue(Q)`
    - 判队空:     `QueueEmpty(Q)`
    - 判队满:     `QueueFull(Q)`
    - 入队:       `EnQueue(Q,x)`
    - 出队:       `DeQueue(Q)`
    - 取队头元素: `QueueFront(Q)`, 不同于出队，队头元素保留。

### 用两个队列如何实现一个栈，用两个栈如何实现一个队列

#### 1. 两个栈实现一个队列
> 栈的特性是先进后出（FILO）,队列的特性是先进先出（FIFO）,
在实现delete时，将一个栈中的数据依次拿出来压入到另一个为空的栈，另一个栈中数据的顺序恰好是先压入栈1的元素, 此时在栈2的上面。为了实现效率的提升，在delete时，判断栈2是否有数据，如果有的话，直接删除栈顶元素，在栈2为空时才将栈1的数据压入到栈2中，从而提高程序的运行效率

- `push` 操作时，一直将数据压入到 `栈2` 中
- `delete` 操作时，首先判断 `栈2` 是否为空
    - 不为空的情况直接删除 `栈2` 栈顶元素
    - 为空的话将 `栈1` 的数据压入到 `栈2` 中，再将 `栈2` 栈顶元素删除。

#### 2. 两个队列实现一个栈
> 因为队列是先进先出，所以要拿到队列中最后压入的数据，只能每次将队列中数据pop到只剩一个，此时这个数据为最后压入队列的数据，在每次pop时，将数据压入到另一个队列中。每次执行delete操作时，循环往复。（感觉效率低）每次删除时间复杂度O(N)
