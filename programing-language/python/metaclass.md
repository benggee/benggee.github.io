**1. property动态属性**

使用property标注，我可以直接像使用人属性一样使用方法，例如：

```python
from datetime import date,datetime
class User:
    def __init__(self, name, birthday):
        self.name = name
        self.birthday = birthday
        self._age = 20
    @property
    def age(self):
        return datetime.now().year - self.birthday.year
    @age.setter
    def age(self,value):
        self._age = value
#  __name__ == "__main__" 如果没有这一行代码别的文件引入当前包就会自动执行
if __name__== "__main__":
    user = User("bobby", date(year=1990, month=3, day=6))
    print(user.name)
    #print(user.get_age())

    user.age = 40   
    print(user.age)
    print(user._age)
```

使用setter标注可以像设置属性值一样设置使用函数，其中user.age = 40  实际上会去调用age(self,value)函数

**2. __getattr__、__getattribute__魔法函数**

__getattr__是在访问类属性但是类属性不存在的时候会触发这个函数，例如：

```python
class User:
    def __init__(self, name, age, info = {}):
        self.name = name
        self.age = age
        self.info = info
    def __getattr__(self, item):
        return "This is get attr"
if __name__ == "__main__":
    user = User("刘发军", "18", {"company_name":"找油网", "group":"php"})
    print(user.test)   # 访问一个不存在的属性会触发__getattr__方法
```

__getattribute__是在访问任何类属性的时候都会触发，例如：

```python
class User:
    def __init__(self, name, age, info = {}):
        self.name = name
        self.age = age
        self.info = info
    # __getattribute__会在访问所有属性的时候触发
    def __getattribute__(self, item):
        return "This is getattribute"
if __name__ == "__main__":
    user = User("刘发军", "18", {"company_name":"找油网", "group":"php"})
    print(user.name)
    print(user.test)  
```

这个例子中，两次调用都会触发__getattribute__函数

**3. 属性描述符和属性查找过程**

**3.1 什么是属性描述符**

属性描述符是类属性可以通过一个类来对其进行操作，而那个类必须实现__set__或者__get__或者__delete__任意一个方法，下面来看一个例子：

```python
# 做int类型验证的属性描述符
class IntField:
    def __set__(self,instance,value):
        pass 
    def __get__(self,instance,value):
        pass 
    def __delete__(self,instance,value):
        pass
class User:
    age = IntField()
```

在上面的例子中，我们说IntField就是age的属性描述符。属性描述符可以属性进行任意的操作。在我们实现ORM的时候，对字段进行分类验证是非常有用的一个特性。

上面IntField属性描述符实现了__set__和__get__我们叫数据属性描述符。那么相反的非数据属性描述符是只实现了一个__get__方法，如下：

```python
class NonDataIntField:
    def __get__(self, instance, owner):
        return self.value
```

那么，假如一个属性同时拥有__getattr__、__getattribute__、数据属性描述符、非数据属性描述符的时候它们查找的先后顺序是什么样的呢？

假如user是某个类的实例，那么user.age或者getattr(user,'age') 首先调用__getattribute__，如果类定义了__getattr__方法，那么在__getattribute__抛出AttributeError的时候就会调用到__getattr__，而对于描述符(__get__)的调用。则是发生在__getattribute__内部的。

假如user=User()，那么user.age顺序如下：

（1）如果“age”是出现在User或其基类的__dict__中，且age是data descriptor(数据描述符)，那么调用其__get__方法

（2）如果“age”出现在user的__dict__中，那么直接返回obj.__dict__['age']

（3）如果“age”出现在User或其基类的__dict__中，如果"age"是非数据描述符，那么调用其__get__方法，否则返回__dict__['age']

（4）如果User有__getattr__方法，调用__getattr__方法，否则抛出AttributeError异常

**4. __new__和__init__的区别**

__new__是用来控制对象的生成过程，在对象生成之前

__init__是用来完善对象的，如果__new__方法不返回对象，则不会调用init函数

```python
class User:
    def __new__(cls, *args, **kwargs):
        print("in new")
    def __init__(self,name):
        print("in init")
        pass
if __name__=="__main__":
    user = User(name="bobby")
```

上面的例子中，__init__里面不会执行，因为__new__没有返回对象。如果要让__init__执行，需要__new__返回一个对象，这里使用super的方法返回基类

```python
class User:
    def __new__(cls, *args, **kwargs):
        print("in new")
        return super().__init__(cls)
    def __init__(self,name):
        print("in init")
        pass
if __name__=="__main__":
    user = User(name="bobby")
```

**5. 自定义元类**

元类就是创建类的类

在python中，类的实例化过程是首先寻找metaclass(元类)，通过metaclass去创建类，类对象，实例。元类必须继承type类

```python
# MetaClass是一个元类
class MetaClass(type):
    def __new__(cls, *args, **kwargs):
        return super().__new__(cls, *args, **kwargs)
class User(metaclass=MetaClass):
    def __init__(self,name):
        self.name = name

    def __str__(self):
        return "User"

if __name__=="__main__":
    my_obj = User(name="bobby")
    print(my_obj)
```

也可以像下面这样

```python
class MetaClass(type):
    def __new__(cls, *args, **kwargs):
        return super().__new__(cls, *args, **kwargs)
class BaseClass(metaclass=MetaClass):
    def answer(self):
        return "i am baseclass"
class User(BaseClass):
    def __init__(self,name):
        self.name = name

    def __str__(self):
        return "User"

if __name__=="__main__":
    my_obj = User(name="bobby")
    print(my_obj.answer())
    print(my_obj)
```

如果当前类没有继承元类，则会找基类的元类