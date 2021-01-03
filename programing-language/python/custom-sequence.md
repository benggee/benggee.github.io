**1. 序列类型的分类**

**1.1 容器序列**

list、tuple(元组）、deque(???)

**1.2 扁平序列**

str、bytes、bytearray、array.array

**1.3 可变序列**

list、deque、bytesarray、array

**1.4 不可变序列**

str、tuple、bytes 

**2. 序列的abc继承关系**

**2.1 Sequence (不可变序列)** 

和容器相关的数据结构，抽象基类都是放在abc里面的

在python类库：/Lib/collections/abc.py

```python
from _collections_abc import *
from _collections_abc import __all__
```

__all__在Lib/_collections_abc.py如下：

```python
__all__ = ["Awaitable", "Coroutine",
           "AsyncIterable", "AsyncIterator", "AsyncGenerator",
           "Hashable", "Iterable", "Iterator", "Generator", "Reversible",
           "Sized", "Container", "Callable", "Collection",
           "Set", "MutableSet",
           "Mapping", "MutableMapping",
           "MappingView", "KeysView", "ItemsView", "ValuesView",
           "Sequence", "MutableSequence",
           "ByteString",
           ]
```

这里我们主要研究Sequence（不可变序列）和MutableSequence(可变序列)。MutableSequence其实是在Sequence基础之上加的一些方法，例如append，extend方法从而实现可变序列

我们来看一下Sequence

```python
class Sequence(Reversible, Collection):

    """All the operations on a read-only sequence.

    Concrete subclasses must override __new__ or __init__,
    __getitem__, and __len__.
    """

    __slots__ = ()

    @abstractmethod
    def __getitem__(self, index):
        raise IndexError

    def __iter__(self):
        i = 0
        try:
            while True:
                v = self[i]
                yield v
                i += 1
        except IndexError:
            return

    def __contains__(self, value):
        for v in self:
            if v is value or v == value:
                return True
        return False

    def __reversed__(self):
        for i in reversed(range(len(self))):
            yield self[i]

    def index(self, value, start=0, stop=None):
        '''S.index(value, [start, [stop]]) -> integer -- return first index of value.
           Raises ValueError if the value is not present.

           Supporting start and stop arguments is optional, but
           recommended.
        '''
        if start is not None and start < 0:
            start = max(len(self) + start, 0)
        if stop is not None and stop < 0:
            stop += len(self)

        i = start
        while stop is None or i < stop:
            try:
                v = self[i]
                if v is value or v == value:
                    return i
            except IndexError:
                break
            i += 1
        raise ValueError

    def count(self, value):
        'S.count(value) -> integer -- return number of occurrences of value'
        return sum(1 for v in self if v is value or v == value)

Sequence.register(tuple)
Sequence.register(str)
Sequence.register(range)
Sequence.register(memoryview)
```

可以看到Sequence继承了两个类，Reversible(反转器)和Collection

其中Collection又继承了Sized、Iterable、Container

```python
class Collection(Sized, Iterable, Container):

    __slots__ = ()

    @classmethod
    def __subclasshook__(cls, C):
        if cls is Collection:
            return _check_methods(C,  "__len__", "__iter__", "__contains__")
        return NotImplemented
```

通过Sized可以计算序列的长度， 通过Iterable可以对序列进行迭代，Container可以使用in判断是否在序列内，下面我们分别看一下Sized、Iterable、Container的源码：

Sized源码如下：

```python
class Sized(metaclass=ABCMeta):

    __slots__ = ()

    @abstractmethod
    def __len__(self):
        return 0

    @classmethod
    def __subclasshook__(cls, C):
        if cls is Sized:
            return _check_methods(C, "__len__")
        return NotImplemented
```

我们看到在Sized类中，有一个抽象方法__len__表示需要实现__len__方法才能实现序列的协议

Iterable的源码如下：

```python
class Iterable(metaclass=ABCMeta):

    __slots__ = ()

    @abstractmethod
    def __iter__(self):
        while False:
            yield None

    @classmethod
    def __subclasshook__(cls, C):
        if cls is Iterable:
            return _check_methods(C, "__iter__")
        return NotImplemented
```

在Iterable的中有一个抽象方法__iter__，表示要实现__iter__方法才能满足实现序列的协议

最后我们看一下Container的源码：

```python
class Container(metaclass=ABCMeta):
    __slots__ = ()
    @abstractmethod
    def __contains__(self, x):
        return False

    @classmethod
    def __subclasshook__(cls, C):
        if cls is Container:
            return _check_methods(C, "__contains__")
        return NotImplemented
```

在Container中有一个抽象方法__contains__表示需要实现__contains__方法才能满足实现序列的协议

然后我们回到Sequence类

```python
...
@abstractmethod
    def __getitem__(self, index):
        raise IndexError
...
```

可以看到，在Sequence类中有一个抽象方法__getitem__，那么结全Sized、Iterable、Container来看的话，至少需要实现4个方法才能满足实现一个不可变序列的协议 ，他们分别是：

__getitem__

__len__

__iter__

__contains__

**2.2 MutableSequence（可变序列）**

MutableSequence可变序列是继承了Sequence。所以，先要满足不可变序列的实现协议，也就是需要实现__getitem__、__len__、__iter__、__contains__方法

以下是MutableSequence源码：

```python
class MutableSequence(Sequence):

    __slots__ = ()

    """All the operations on a read-write sequence.
    Concrete subclasses must provide __new__ or __init__,
    __getitem__, __setitem__, __delitem__, __len__, and insert().

    """
    @abstractmethod
    def __setitem__(self, index, value):
        raise IndexError
    @abstractmethod
    def __delitem__(self, index):
        raise IndexError
    @abstractmethod
    def insert(self, index, value):
        'S.insert(index, value) -- insert value before index'
        raise IndexError
    def append(self, value):
        'S.append(value) -- append value to the end of the sequence'
        self.insert(len(self), value)
    def clear(self):
        'S.clear() -> None -- remove all items from S'
        try:
            while True:
                self.pop()
        except IndexError:
            pass
    def reverse(self):
        'S.reverse() -- reverse *IN PLACE*'
        n = len(self)
        for i in range(n//2):
            self[i], self[n-i-1] = self[n-i-1], self[i]
    def extend(self, values):
        'S.extend(iterable) -- extend sequence by appending elements from the iterable'
        for v in values:
            self.append(v)
    def pop(self, index=-1):
        '''S.pop([index]) -> item -- remove and return item at index (default last).
           Raise IndexError if list is empty or index is out of range.
        '''
        v = self[index]
        del self[index]
        return v
    def remove(self, value):
        '''S.remove(value) -- remove first occurrence of value.
           Raise ValueError if the value is not present.
        '''
        del self[self.index(value)]

    def __iadd__(self, values):
        self.extend(values)
        return sel
```

可以看到，在MutableSequence中多了几个抽象方法，它们是：

__setitem__  修改

__delitem__  删除

insert(self, index, value)  新增

**3. 序列的+、+=和extend的区别**

**3.1 序列的+**

序列的+操作，需要一个新的序列来接收结果

序列的+操作，“+”两边类型要一致

```shell
a = [1,2]
b = a + [3,4]
print(b)  # [1,2,3,4]
```

如果“+”号两边类型不一致就会抛出异常

```shell
a = [1,2]
b = a + (3,4)
print(b)   # 这时由于在进行+操作的时候两边类型不致会抛出异常
```

**3.2 序列的+=操作**

序列的+=操作是原地改变操作符号左边的值

序列的+=操作类型可以不一样

例如：

```shell
a = [1,2]
a += [3,4]
print(a)    # 结果为[1,2,3,4]

b = ["a","b"]
b += ("c", "d")
print(b)    # 结果为[a,b,c,d]
```

**3.3 序列的extend操作**

序列的extend操作是会将两个序列进行合并，重复的值累加

例如：

```python
a = [1,2]
a += [3,4]
a.extend([1,2])
print(a)    # 结果为[1,2,3,4,1,2]
```

需要特别注意的是，append操作和exend操作是不一样的，如下：

```python
# extend操作
a = [1,2]
a.extend([2,3])
print(a)     # 结果为 [1,2,2,3]

# append操作
b = [1,2]
b.append([2,3])
print(b)   # 结果为[1,2,[2,3]]
```

**4. 实现可切片的对象**

首先，我们来看一下切片都有哪些操作：

```python
#模式[start:end:step]
"""
    其中，第一个数字start表示切片开始位置，默认为0；
    第二个数字end表示切片截止（但不包含）位置（默认为列表长度）；
    第三个数字step表示切片的步长（默认为1）。
    当start为0时可以省略，当end为列表长度时可以省略，
    当step为1时可以省略，并且省略步长时可以同时省略最后一个冒号。
    另外，当step为负整数时，表示反向切片，这时start应该比end的值要大才行。

"""
aList = [3, 4, 5, 6, 7, 9, 11, 13, 15, 17]
print (aList[::])  # 返回包含原列表中所有元素的新列表
print (aList[::-1])  # 返回包含原列表中所有元素的逆序列表
print (aList[::2])  # 隔一个取一个，获取偶数位置的元素
print (aList[1::2])  # 隔一个取一个，获取奇数位置的元素
print (aList[3:6])  # 指定切片的开始和结束位置
aList[0:100]  # 切片结束位置大于列表长度时，从列表尾部截断
aList[100:]  # 切片开始位置大于列表长度时，返回空列表
aList[len(aList):] = [9]  # 在列表尾部增加元素
aList[:0] = [1, 2]  # 在列表头部插入元素
aList[3:3] = [4]  # 在列表中间位置插入元素
aList[:3] = [1, 2]  # 替换列表元素，等号两边的列表长度相等
aList[3:] = [4, 5, 6]  # 等号两边的列表长度也可以不相等
aList[::2] = [0] * 3  # 隔一个修改一个
print (aList)
aList[::2] = ['a', 'b', 'c']  # 隔一个修改一个
aList[::2] = [1,2]  # 左侧切片不连续，等号两边列表长度必须相等
aList[:3] = []  # 删除列表中前3个元素
del aList[:3]  # 切片元素连续
del aList[::2]  # 切片元素不连续，隔一个删一个
```

前面说到不可变序列和可变序列，我们通过模仿这两个协议实现一个可切片的对象。

首先，要实现一个序列对象我们需要实现几个抽象方法：

__getitem__

__len__

__iter__

__contains__

代码如下：

```python
class Group:
    # 支持切片的对象
    def __init__(self,group_name,company_name,staffs):
        self.group_name = group_name
        self.company_name = company_name
        self.staffs = staffs 
        
    def __getitem__(self, item):
        return self.staffs[item]
    
    def __len__(self):
        return len(self.staffs)
    
    def __iter__(self):
        return iter(self.staffs)
    
    def __contains__(self, item):
        if item in self.staffs:
            return True
        else:
            return False 
```

由于Group已经实现了一__getitem__方法可以像使用列表一样去使用。但是，由于我们要实现切片，所以__getitem__方法需要处理两种类型，一种是正常的整形下标，一种是切片类型，我们接着对__getitem__方法进行改造：

```python
def __getitem__(self, item):
    # 动态获取类型
    cls = type(self)
    if isinstance(item, slice):  # 如果是切片类型
        return cls(group_name=self.group_name,company_name=self.company_name,staffs=self.staffs[item])
    elif isinstance(item, numbers.Integral):
        return cls(group_name=self.group_name,company_name=self.company_name,staffs=[self.staffs[item]])
```

注意：使用到了numbes.Integral所以要导入numbers包

我们还可以加入一个返回方法

```python
def __reversed__(self):
    self.staffs.reverse()
```

最后，我们来使用一下

```python
staffs = ["bobby1", "heads", "bobby2", "bobby3"]
# 返回的是一个迭代器
group = Group(company_name="heads", group_name="Group", staffs=staffs)
a = group[1::]
for user in a:
    print(user)
# 结果如下
heads
bobby2
bobby3
```

**5. bisect管理可排序序列**

通常如果我们需要对一个序列进行排序，首先是先生成一个序列，然后再对其进行排序。bisect可以实现在往序列里面添加的时候就维护一个有序的列表，由于其是C语言实现，使用二分排序算法，效率很高。

来看一下以下实例：

```python
import bisect 
inter_list = []
bisect.insort(inter_list,3)
bisect.insort(inter_list,2)
bisect.insort(inter_list,6)
bisect.insort(inter_list,4)
bisect.insort(inter_list,5)
bisect.insort(inter_list,1)
print(inter_list)  # 结果为 [1,2,3,4,5,6]
```

如果在往序列里面添加元素的时候，遇到值相等的情况，我们可以决定是放在左边还是右边

例如：

```python
bisect.insort_left(inter_list,3)
```

那这个时候新添加的3就在原来3的左边，如果要添加到右边我们可以使用insort_right

那么，怎么验证这是添加到左边还是右边了呢？

我们可以使用bisect.bisect_left或者bisect.bisect_right来验证，会返回一个下标。

例如：

```python
print(bisect.bisect_left(inter_list,3))   # 结果为2
print(bisect.bisect_right(inter_list,3))   # 结果为3
```

**6. 什么时候不该用列表**

我们先来看一下python中的array，array是C语言实现的一个数据结构，和list最重要的一个区别就是，array只能存放指定的数据类型

使用方法如下：

```python
import array 
my_array = array.array("i")
my_array.appen(1)  # 这个是会报错的，my_array这时只能添加字符串类型的元素
my_array.appen("abc")  # 通过
```

那么，在一些效率要求比较高的时候我们可以放弃list使用array

**7. 列表推导式、生成器表达式、字典推导式**

**7.1 列表推导式**

```python
int_list = [1,2,3,4,5,6]
qu_list = [item for item in int_list]
print(qu_list) # 结果 [1,2,3,4,5,6]
```

还可这样：

```python
qu_list =[item*item for item in int_list]
print(qu_list) # [1,4,9,16,25,36]
```

item*item也可以是一个函数

**7.2 生成器表达式**

```python
odd_list = (i for i in range(21) if 1%2 == 1)
print(type(odd_list)) # <class 'generator'> 是一个生成器
for i in odd_list:
    print(i)
```

**7.3 字典推导式**

```python
# 字典推导式
my_dict = {"bobby1":22,"bobby2":23, "bobby3":5}
reversed_dict = {value:key for key,value in my_dict.items()}
print(reversed_dict)  # {22: 'bobby1', 23: 'bobby2', 5: 'bobby3'}
```

**7.4 集合推导式**

```python
# 集合推导式
my_set = set(my_dict.keys())
my_set = {key for key, value in my_dict.items()}
print(my_set) #{'bobby2', 'bobby3', 'bobby1'}
```

