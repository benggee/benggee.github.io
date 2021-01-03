**1. collections中的abc**

先看一段代码：

```python
from collections.abc import Mapping, MutableMapping
a = {}
print(isinstance(a, MutableMapping))   # True
```

可以看到dict和set是一个MutableMapping类型

MutableMapping继承Mapping类，下面我们从Mapping类的源码开始。

**1.1 Mapping 类**

Lib/collections/abc.py

```python
from _collections_abc import *
from _collections_abc import __all__
```

接下来看__all__ 

Lib/_collections_abc.py

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

我们可以看到其中有Mapping和MutableMapping 

和list不同，这次我们重点看一下MutableMapping类

Lib/_collections_abc.py

```python
class MutableMapping(Mapping):

    __slots__ = ()

    """A MutableMapping is a generic container for associating
    key/value pairs.

    This class provides concrete generic implementations of all
    methods except for __getitem__, __setitem__, __delitem__,
    __iter__, and __len__.

    """

    @abstractmethod
    def __setitem__(self, key, value):
        raise KeyError

    @abstractmethod
    def __delitem__(self, key):
        raise KeyError

    __marker = object()

    def pop(self, key, default=__marker):
        '''D.pop(k[,d]) -> v, remove specified key and return the corresponding value.
          If key is not found, d is returned if given, otherwise KeyError is raised.
        '''
        try:
            value = self[key]
        except KeyError:
            if default is self.__marker:
                raise
            return default
        else:
            del self[key]
            return value

    def popitem(self):
        '''D.popitem() -> (k, v), remove and return some (key, value) pair
           as a 2-tuple; but raise KeyError if D is empty.
        '''
        try:
            key = next(iter(self))
        except StopIteration:
            raise KeyError from None
        value = self[key]
        del self[key]
        return key, value

    def clear(self):
        'D.clear() -> None.  Remove all items from D.'
        try:
            while True:
                self.popitem()
        except KeyError:
            pass

    def update(*args, **kwds):
        ''' D.update([E, ]**F) -> None.  Update D from mapping/iterable E and F.
            If E present and has a .keys() method, does:     for k in E: D[k] = E[k]
            If E present and lacks .keys() method, does:     for (k, v) in E: D[k] = v
            In either case, this is followed by: for k, v in F.items(): D[k] = v
        '''
        if not args:
            raise TypeError("descriptor 'update' of 'MutableMapping' object "
                            "needs an argument")
        self, *args = args
        if len(args) > 1:
            raise TypeError('update expected at most 1 arguments, got %d' %
                            len(args))
        if args:
            other = args[0]
            if isinstance(other, Mapping):
                for key in other:
                    self[key] = other[key]
            elif hasattr(other, "keys"):
                for key in other.keys():
                    self[key] = other[key]
            else:
                for key, value in other:
                    self[key] = value
        for key, value in kwds.items():
            self[key] = value

    def setdefault(self, key, default=None):
        'D.setdefault(k[,d]) -> D.get(k,d), also set D[k]=d if k not in D'
        try:
            return self[key]
        except KeyError:
            self[key] = default
        return default

MutableMapping.register(dict)
```

在MutableMapping类里有两个抽象方法，__setitem__和__getitem__

接下来我们看MutableMapping继承的Mapping类

Lib/_collections_abc.py

```python
class Mapping(Collection):

    __slots__ = ()

    """A Mapping is a generic container for associating key/value
    pairs.

    This class provides concrete generic implementations of all
    methods except for __getitem__, __iter__, and __len__.

    """

    @abstractmethod
    def __getitem__(self, key):
        raise KeyError

    def get(self, key, default=None):
        'D.get(k[,d]) -> D[k] if k in D, else d.  d defaults to None.'
        try:
            return self[key]
        except KeyError:
            return default

    def __contains__(self, key):
        try:
            self[key]
        except KeyError:
            return False
        else:
            return True

    def keys(self):
        "D.keys() -> a set-like object providing a view on D's keys"
        return KeysView(self)

    def items(self):
        "D.items() -> a set-like object providing a view on D's items"
        return ItemsView(self)

    def values(self):
        "D.values() -> an object providing a view on D's values"
        return ValuesView(self)

    def __eq__(self, other):
        if not isinstance(other, Mapping):
            return NotImplemented
        return dict(self.items()) == dict(other.items())

    __reversed__ = None

Mapping.register(mappingproxy)
```

Mapping类继承了Collection

Lib/_collections_abc.py

```python
class Collection(Sized, Iterable, Container):

    __slots__ = ()

    @classmethod
    def __subclasshook__(cls, C):
        if cls is Collection:
            return _check_methods(C,  "__len__", "__iter__", "__contains__")
        return NotImplemented
```

可以看到，Collection继承了Sized、Iterable、Container这个和自定义序列类里面是一样的，这里就不重复了。

**2. dict的常见用法**

**1.1 拷贝的问题**

浅拷贝：

```python
a = {"bobby1":{"company":"heads"},
     "bobby2":{"company":"heads2"}}
# 浅拷贝
new_dict = a.copy()
new_dict["bobby1"]["company"] = "heads3"
print(a)   #  结果{'bobby1': {'company': 'heads3'}, 'bobby2': {'company': 'heads2'}}
```

可以看到，通过copy()拷贝的dict实际是是一个引用，对新的dict进行修改

深拷贝：

如果要对dict和set进行深拷贝，可以使用deepcopy

```python
from copy import deepcopy

a = {"bobby1":{"company":"heads"},
     "bobby2":{"company":"heads2"}}
dict2 = deepcopy(a)
dict2["bobby1"]["company"] = "heads5"
print(a)
```

deepcopy会完全拷贝一份副本

**1.2 fromkeys**

```python
new_list = ["bobby1", "bobby2"]
new_dict = dict.fromkeys(new_list, {"company":"heads"})
print(new_dict)
# 结果
{'bobby1': {'company': 'heads'}, 'bobby2': {'company': 'heads'}}
```

**1.3 update** 

dict可以使用list，tugle来更新，如下：

```python
new_dict.update(("bobby","heads"))
new_dict.update((("bobby","heads"),("bobby2","heads2")))
new_dict.update(["abc", "cdd"])
new_dict.update([["aa","bb"], ["acd", "def"]])
```

由此我们得到一个规律，任何可迭代的类型都可以用来更新dict

**1.4 __missing__方法**

使用一个不存在的dict的时候，就会触发这个方法

**1.5 dict的更多操作方法详见：**

C:\Users\XXXX\.PyCharm2018.3\system\python_stubs\-916386140\builtins.py

这是我的系统目录，可以使用PyCharm通过dict关键字追踪到

```python
class dict(object):
    """
    dict() -> new empty dictionary
    dict(mapping) -> new dictionary initialized from a mapping object's
        (key, value) pairs
    dict(iterable) -> new dictionary initialized as if via:
        d = {}
        for k, v in iterable:
            d[k] = v
    dict(**kwargs) -> new dictionary initialized with the name=value pairs
        in the keyword argument list.  For example:  dict(one=1, two=2)
    ""
    def clear(self): # real signature unknown; restored from __doc__
        """ D.clear() -> None.  Remove all items from D. """
        pass
    def copy(self): # real signature unknown; restored from __doc__
        """ D.copy() -> a shallow copy of D """
        pass
    @staticmethod # known case
    def fromkeys(*args, **kwargs): # real signature unknown
        """ Create a new dictionary with keys from iterable and values set to value. """
        pass
    def get(self, *args, **kwargs): # real signature unknown
        """ Return the value for key if key is in the dictionary, else default. """
        pass
    def items(self): # real signature unknown; restored from __doc__
        """ D.items() -> a set-like object providing a view on D's items """
        pass
    def keys(self): # real signature unknown; restored from __doc__
        """ D.keys() -> a set-like object providing a view on D's keys """
        pass
    def pop(self, k, d=None): # real signature unknown; restored from __doc__
        """
        D.pop(k[,d]) -> v, remove specified key and return the corresponding value.
        If key is not found, d is returned if given, otherwise KeyError is raised
        """
        pass
    def popitem(self): # real signature unknown; restored from __doc__
        """
        D.popitem() -> (k, v), remove and return some (key, value) pair as a
        2-tuple; but raise KeyError if D is empty.
        """
        pass
    def setdefault(self, *args, **kwargs): # real signature unknown
        """
        Insert key with a value of default if key is not in the dictionary.
        Return the value for key if key is in the dictionary, else default.
        """
        pass
    def update(self, E=None, **F): # known special case of dict.update
        """
        D.update([E, ]**F) -> None.  Update D from dict/iterable E and F.
        If E is present and has a .keys() method, then does:  for k in E: D[k] = E[k]
        If E is present and lacks a .keys() method, then does:  for k, v in E: D[k] = v
        In either case, this is followed by: for k in F:  D[k] = F[k]
        """
        pass
    def values(self): # real signature unknown; restored from __doc__
        """ D.values() -> an object providing a view on D's values """
        pass
    def __contains__(self, *args, **kwargs): # real signature unknown
        """ True if the dictionary has the specified key, else False. """
        pass
    def __delitem__(self, *args, **kwargs): # real signature unknown
        """ Delete self[key]. """
        pass
    def __eq__(self, *args, **kwargs): # real signature unknown
        """ Return self==value. """
        pass
    def __getattribute__(self, *args, **kwargs): # real signature unknown
        """ Return getattr(self, name). """
        pass
    def __getitem__(self, y): # real signature unknown; restored from __doc__
        """ x.__getitem__(y) <==> x[y] """
        pass
    def __ge__(self, *args, **kwargs): # real signature unknown
        """ Return self>=value. """
        pass
    def __gt__(self, *args, **kwargs): # real signature unknown
        """ Return self>value. """
        pass
    def __init__(self, seq=None, **kwargs): # known special case of dict.__init__
        """
        dict() -> new empty dictionary
        dict(mapping) -> new dictionary initialized from a mapping object's
            (key, value) pairs
        dict(iterable) -> new dictionary initialized as if via:
            d = {}
            for k, v in iterable:
                d[k] = v
        dict(**kwargs) -> new dictionary initialized with the name=value pairs
            in the keyword argument list.  For example:  dict(one=1, two=2)
        # (copied from class doc)
        """
        pass
    def __iter__(self, *args, **kwargs): # real signature unknown
        """ Implement iter(self). """
        pass
    def __len__(self, *args, **kwargs): # real signature unknown
        """ Return len(self). """
        pass
    def __le__(self, *args, **kwargs): # real signature unknown
        """ Return self<=value. """
        pass
    def __lt__(self, *args, **kwargs): # real signature unknown
        """ Return self<value. """
        pass
    @staticmethod # known case of __new__
    def __new__(*args, **kwargs): # real signature unknown
        """ Create and return a new object.  See help(type) for accurate signature. """
        pass
    def __ne__(self, *args, **kwargs): # real signature unknown
        """ Return self!=value. """
        pass
    def __repr__(self, *args, **kwargs): # real signature unknown
        """ Return repr(self). """
        pass
    def __setitem__(self, *args, **kwargs): # real signature unknown
        """ Set self[key] to value. """
        pass
    def __sizeof__(self): # real signature unknown; restored from __doc__
        """ D.__sizeof__() -> size of D in memory, in bytes """
        pass
    __hash__ = None
```

**3. dict的子类** 

一般是不建议继承dict类的

我们可以看以下代码：

```python
class MyDict(dict):
    def __setitem__(self,key,value):
        super().__setitem__(key,value*2)
my_dict = MyDict(one=1)
print(my_dict)   # {'one':1}
my_dict["one"] = 1
print(my_dict)   # {'one':2}
```

可以看到，在实例化对象的时候__setitem__并没有执行。那么，如果我们一定要继承dict对象怎么办呢？

```python
from collections import UserDict

class Mydict(UserDict):
    def __setitem__(self, key, value):
        super().__setitem__(key,value)
my_dict = Mydict(one=1)
print(my_dict)   # {'one':1}
```

UserDict实际上是对Dict做了一层封装，如下：

```python
class UserDict(MutableMapping[_KT, _VT]):
    data: Dict[_KT, _VT]
    def __init__(self, dict: Optional[Mapping[_KT, _VT]] = ..., **kwargs: _VT) -> None: ...
    def __len__(self) -> int: ...
    def __getitem__(self, key: _KT) -> _VT: ...
    def __setitem__(self, key: _KT, item: _VT) -> None: ...
    def __delitem__(self, key: _KT) -> None: ...
    def __iter__(self) -> Iterator[_KT]: ...
    def __contains__(self, key: object) -> bool: ...
    def copy(self: _UserDictT) -> _UserDictT: ...
    @classmethod
    def fromkeys(cls: Type[_UserDictT], iterable: Iterable[_KT], value: Optional[_VT] = ...) -> _UserDictT: ...
_UserListT = TypeVar('_UserListT', bound=UserList)
```

**4. set和frozenset**

set是一个可变集合，frozenset是不可变的集合。set、frozenset是一个无序的不重复的集合。首先来看set

```python
# set可以是一个字符串
s = set('abcdefg')    # {'d', 'b', 'f', 'a', 'g', 'e', 'c'}
# set也可以是一个列表
s = set(['a','b','c'])  # {'c', 'b', 'a'}
# set的运算
another_set = set('cef')
re_set = s.difference(another_set)  # 差集
re_set = s - another_set   # 差集
re_set = s & another_set   # 并集
re_set = s | another_set   # 交集
```

**5. set和dict实现原理**

dict的key或者set的值都必须是可以hash的

dict的内存花销大，但是查询速度快，自定义对象或者python内部的对象都是用dicti包装的

dict的存储顺序和元素添加顺序有关

添加数据有可能改变已有数据的顺序

hash表的存储原理

首先会算出dict的key或者set的值的hash，然后找到对应的内存。但是，有可能算出的hash值有重复，这时候就需要解决冲突，直到算出唯一的hash值。由于hash值是算出来不一定是连续的，所以内存会存在空隙，一般情况下，在声明一个很大的集合的时候，当内存少于1/3的时候就会申请一块更大的内存。

下面我们来看一下如想查找hash值：

![img](data:image/jpeg;base64,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)

下面我们使用一段代码来测试比较一下集合和list的效率：

```python
from random import randint
def load_list_data(total_nums, target_nums):
    """
    从文件中读取数据，以list的方式返回
    :param total_nums: 读取的数量
    :param target_nums: 需要查询的数据的数量
    """
    all_data = []
    target_data = []
    file_name = "D:/xxxx/AdvancePython/fbobject_idnew.txt"
    with open(file_name, encoding="utf8", mode="r") as f_open:
        for count, line in enumerate(f_open):
            if count < total_nums:
                all_data.append(line)
            else:
                break
    for x in range(target_nums):
        random_index = randint(0, total_nums)
        if all_data[random_index] not in target_data:
            target_data.append(all_data[random_index])
            if len(target_data) == target_nums:
                break
    return all_data, target_data
def load_dict_data(total_nums, target_nums):
    """
    从文件中读取数据，以dict的方式返回
    :param total_nums: 读取的数量
    :param target_nums: 需要查询的数据的数量
    """
    all_data = {}
    target_data = []
    file_name = "G:/慕课网课程/AdvancePython/fbobject_idnew.txt"
    with open(file_name, encoding="utf8", mode="r") as f_open:
        for count, line in enumerate(f_open):
            if count < total_nums:
                all_data[line] = 0
            else:
                break
    all_data_list = list(all_data)
    for x in range(target_nums):
        random_index = randint(0, total_nums-1)
        if all_data_list[random_index] not in target_data:
            target_data.append(all_data_list[random_index])
            if len(target_data) == target_nums:
                break
    return all_data, target_data
def find_test(all_data, target_data):
    #测试运行时间
    test_times = 100
    total_times = 0
    import time
    for i in range(test_times):
        find = 0
        start_time = time.time()
        for data in target_data:
            if data in all_data:
                find += 1
        last_time = time.time() - start_time
        total_times += last_time
    return total_times/test_times
if __name__ == "__main__":
    # all_data, target_data = load_list_data(10000, 1000)
    # all_data, target_data = load_list_data(100000, 1000)
    # all_data, target_data = load_list_data(1000000, 1000)
    # all_data, target_data = load_dict_data(10000, 1000)
    # all_data, target_data = load_dict_data(100000, 1000)
    # all_data, target_data = load_dict_data(1000000, 1000)
    all_data, target_data = load_dict_data(2000000, 1000)
    last_time = find_test(all_data, target_data)
    #dict查找的性能远远大于list
    #在list中随着list数据的增大 查找时间会增大
    #在dict中查找元素不会随着dict的增大而增大
    print(last_time)
```

