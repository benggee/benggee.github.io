**1. type、object、class之间的关系**

在python中，所有类都继承object类，所有类本身都是是type的实例

**python中的type** 

type是一切对象的实例，包括type自己

```python
>>> a = "abc"
>>> type(a)    // class 'str'
>>> type(str)  // class 'type'
>>> type(type) // class 'type' 
>>> type(object) // class 'type'
```

**python中的object:**

可以通过__bases__查看父类

```python
>>> class MyClass:
>>>     def func1():
>>>         return "abc"    
>>> MyClass.__bases__    // class 'object'
>>> object.__bases__     // class 'object'
>>> type.__bases__       // class 'object'
```

\2. python中的常见内置类型

python中对象可以通过id()来查看唯一id

```python
>>> class MyClass:
>>> id(MyClass)
```

python中全局只有一个None对象

```python
>>> a = None
>>> b = None
>>> id(a)==id(b)   // True
```

python中的内置类型：

- None(全局只有一个)
- 数值
  - int
  - float
  - complex（复数）
  - bool
- 迭代类型
- 序列类型
  - list
  - Bytes、bytearray、memoryview(二进制序列)
  - range
  - tuple
  - str
  - array
- 映射（dict）
- 集合
  - set
  - frozenset
- 上下文管理类型（with）
- 其它
  - 模块类型
  - class和实例
  - 函数类型
  - 方法类型
  - 代码类型
  - object对象
  - type类型
  - ellipsis类型
  - notimplemented类型



**1. 鸭子类型和多态**

**1.1 鸭子类型**

在python中，并不严格规定类型一致，而是鼓励使用一种类型的，比如extend接收一个可以迭代的参数，其不关心参数是list还是dict

**1.2 多态**   

```python
class Cat(object):
    def say(self):
        print("I am a cat")

class Dog(object):
    def say(self):
        print("I am a Dog")

class Duck(object):
    def say(self):
        print("I am a Duck")

animal_list = [Cat, Dog, Duck]
for anima in animal_list:
    anima().say()
```

**2. 抽象基类（abc模块）**

**2.1 检查某个类是否有某种方法（hasattr)**

```python
class Company(object):
    def __init__(self, employee_list):
        self.employee = employee_list
    # 魔术方法__len__可以使类在支持len()方法
    def __len__(self):
        return len(self.employee)

com = Company(["bobby1", "bobby2"])
print(hasattr(com, "__len__"))   // 结果为True
```

**2.2 判断某个对象的类型（isinstance)**

```python
# 以以上Company对象实例 com为例
from collections.abc import Sized
print(isinstance(com, Sized))
```



```python
class A:
    pass
class B:
    pass
b = B()
# 表示判断b是否是A类型的实例
print(isinstance(b,A))  # False  
```

   

如果B继承A的话，判断出来的结果应该是True

```python
class A:
    pass 
class B(A):
    pass 
b = B()
print(isinstance(b,A))  # True
```

**2.3 如果模拟一个抽象类**

实现抽象类必须实现抽象类所有方法，类似于JAVA的接口

第一种方法：

```python
import abc
from collections.abc import *
class CacheBase()
    def get(self,key):
        raise NotImplementedError
    def set(self,key,val):
        raise NotImplementedError
        
class RedisCache(CaheckBase):
    def set(self, key, val):
        pass 
```

这种方法是利用在父类里面抛出异常来模拟的，但是有一个缺点，就是在真正调用的时候才会抛出异常，下面我们来看第二种方法：

```python
import abc
from clollections.abc import *
class CacheBase(metaclass=abc.ABCMeta):
    @abc.abstractmethod
    def get(self,key):
        pass
    @abc.abstractmethod
    def set(self,key,val):
        pass 
class RedisCache(CacheBase):
    def set(self, key, val):
        pass 
redis_cache = RedisCache()   # 实例化的时候会抛异常
```

这种方法，在实例化的时候就会抛出异常

**3. 使用isinstance而不是type**

type的背后，实际上比较的是类的ID, 而isinstance背后比较的是类型，如下实例：

```python
class A:
    pass
class B(A):
    pass

b = B()
print(isinstance(b,B))  # True
print(isinstance(b,A))  # True

print(type(b)==A) # False
print(type(b)==B) # True
```

**4. 类变量和对象变量**

```python
class A:
    aa = 1
    def __init__(self,x,y):
        self.x = x
        self.y = y

a = A(2,3)

A.aa = 11
a.aa = 100
print(a.x)    # 2
print(a.y)    # 3
print(a.aa)   # 100
print(A.aa)   # 11

b = A(3,5)    
print(b.aa)   # 11
```

**5. 类属性和实例属性以及查找顺序**

**5.1 深度优先**

现有a继承c , c继承d。a同继承b, b继承e ， 那么以深度优先的算法进行遍历，顺序为a,c,d,b,e

**5.2 广度优先**

现有a继承d和c, d和c都继承e， 那么以广度优先进行遍历，顺序为a,c,d,e

**5.3 python3.X使用C3算法**

C3算法可以自动识别是应该使用深度优先还是广度优先算法

**6. 静态方法、类方法以及对象方法**

**6.1 静态方法** 

使用@staticmethod标注

使用类名直接调用

不需要传self参数

使用类名调用，也是一个缺点，只能使用类名调用

**6.2 类方法**

使用@classmethod标注

使用类名直接调用

会默认似一个cls参数，这个参数名字可以自定义

在方法体内可以使用cls来动态调用

**6.3 对象方法**

通常我们声明的方法都是对象方法

只能通过类的实例才可以调用

```python
class Date:
    # 构造方法
    def __init__(self, year,month,day):
        self.year = year
        self.month = month
        self.day   = day
    # 对象方法 
    def tomorrow(self):
        self.day += 1
    # 静态方法
    # 使用staticmethod标注这是一个静态方法
    @staticmethod
    def parse_from_string(date_str):
        year,month,day = tuple(date_str.split("-"))
        return Date(int(year), int(month), int(day))
    # 静态方法
    @staticmethod
    def valid_str(date_str):
        year,month,day = tuple(date_str.split("-"))
        if int(year)>0 and (int(month)>0) and (int(month)<=12) and (int(day)>0) and int(day)<=31:
            return True
        else:
            return False
    # 类方法 
    # 使用@classmethod声明一个类方法， 可以看到传入了一个cls,
    # 这个cls类传于静态方法里面使用Date()调用的方法
    @classmethod
    def from_string(cls, date_str):
        year,month,day = tuple(date_str.split("-"))
        return cls(int(year),int(month),int(day))

    def __str__(self):
        return "{year}/{month}/{day}".format(year=self.year,month=self.month,day=self.day)

if __name__ == "__main__":
    new_day = Date(2018, 12, 31)
    new_day.tomorrow()
    print(new_day)

    date_str = "2018-12-31"
    year,month,day = tuple(date_str.split("-"))
    new_day = Date(int(year),int(month),int(day))
    print(new_day)

    new_day = Date.parse_from_string(date_str)
    print(new_day)

    new_day = Date.from_string(date_str)
    print(new_day)

    print(Date.valid_str("2018-12-32"))
```

**7. 数据封装和私有属性**

通过“__”双下划线可以控制类属性的可见性。但是，在python中实际上是没严格意义上的私有属性。

例如以下例子，我们将birthday设为私有属性，但其实可以通过user._User__birthday来取到birthday的值。

```python
class User:
    def __init__(self, birthday):
        self.__birthday = birthday

    def get_age(self):
        return 2018 - self.__birthday

if __name__ == "__main__":
    user = User(1990)
   print(user._User__birthday)
   print(user.get_age())
```

**8. python对象的自省机制**

python对象的自省机制，类似于PHP、JAVA中的反射，可以通过一些方法获取到对象内部的结构。

**8.1 __dict__**  

对于对象实例来说，__dict__只会列出对象属性

对于类来说，__dict__会列出关于关于类的所有信息

**8.2 dir()**

dir()相较__dict__来说，会获取更多详细的信息。所以，一般是推荐使用dir()

```python
class Person:
    """
    这是一段注释
    """
    name = "user"
class Student(Person):
    def __init__(self, scool_name):
        self.scool_name = scool_name
if __name__ == "__main__":
    user = Student("HEADS")
    # 通过__dict__查询属性
    print(user.__dict__)
    user.__dict__["scool_addr"] = "上海市"
    print(user.scool_addr)
    print(Student.__dict__)
    print(user.name)
    a = [1,2]
    print(dir(a))
    
# 以下是结果
{'scool_name': 'HEADS'}
上海市
{'__module__': '__main__', '__init__': <function Student.__init__ at 0x000001A88797A8C8>, '__doc__': None}
user
['__add__', '__class__', '__contains__', '__delattr__', '__delitem__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__getitem__', '__gt__', '__hash__', '__iadd__', '__imul__', '__init__', '__init_subclass__', '__iter__', '__le__', '__len__', '__lt__', '__mul__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__reversed__', '__rmul__', '__setattr__', '__setitem__', '__sizeof__', '__str__', '__subclasshook__', 'append', 'clear', 'copy', 'count', 'extend', 'index', 'insert', 'pop', 'remove', 'reverse', 'sort']

```

**9. super函数**

super函数可以调用父类的方法，如果有继承多个类，那么调用顺序和类的属性查找顺序一致，都是使用C3算法

```python
from threading import Thread

class MyThread(Thread):
    def __init__(self,name,user):
        self.user = user
        super().__init__(name=name)

# super的调用顺序
class A:
    def __init__(self):
        print("A")

class B(A):
    def __init__(self):
        print("B")
        super().__init__()
class C(A):
    def __init__(self):
        print("C")
        super().__init__()

class D(B,C):
    def __init__(self):
        print("D")
        super(D,self).__init__()

if __name__=="__main__":
    print(D.__mro__)
    d = D()
    
# 结果如下
(<class '__main__.D'>, <class '__main__.B'>, <class '__main__.C'>, <class '__main__.A'>, <class 'object'>)
D
B
C
A
```

**10. 多继承使用的经验**

多继承一般是被继承的类功能单一简单，通过多继承将各单一的模块整合到一起组成一个大的对象。这就类似于堆积木，可以使用不同的单元模块组装成具有某组功能的类

**11. python中的with语句**

**11.1 try except** 

finally表示不管发生什么情况都会执行，要注意的是，如果在except和finally里面都有return，那么最终以finally为准，except的return值会先压到栈空间，finally会压到栈的最上面，当函数返回时，会取栈空间最上面的值

```python
def exe_try():
    try:
        print("code started")
        raise KeyError
        return 1
    except KeyError as e:
        print("key error")
        return 2
    else:  # 表示如果没不是KeyError异常
        print("other error")
        return 3
    finally:
        print("finally")
        return 4
if __name__ == "__main__":
    result = exe_try()
    print(result)
    
# 结果为
code started
key error
finally
4
```

**11.2 with** 

with是比try  except更优雅的一种有方式，可以配合__enter__和__exit__魔术方法实现类似于PHP中的构造和析构方法

```python
class Sample:
    # __enter__方法配合with使用，在实例化最开始执行
    def __enter__(self):
        print("enter")
        return self
    # __exit__方法配合with使用，在对象结束之前执行
    def __exit__(self, exc_type, exc_val, exc_tb):
        print("exit")
    def do_something(self):
        print("doing something")

with Sample() as sample:
    sample.do_something()
# 结果为
enter
doing something
exit
```

**12. contextlib实现上下文管理器**

使用生成器实现上下文管理

```python
import contextlib

@contextlib.contextmanager
def file_open(file_name):
    print("file open")
    yield {}
    print("file end")

with file_open("bobby.txt") as f_open:
    print("file processing")
```

