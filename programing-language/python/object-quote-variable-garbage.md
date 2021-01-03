**1. python的变量到底是什么** 

python变量实际上是一个指针，变量内容是指针指向内容，变量在使用过程中要注意几个问题：

```python
a = 123
b = 123
print(id(a)==id(b))  # 结果为True

c = [1,2,3]
d = [1,2,3]
print(id(c)==id(d))  # 结果为False
```

**2. ==和is的区别**

```python
c = [1,2,3]
d = [1,2,3]
print(c is d)  # 结果为False 
print(c == d)  # 结果为True
```

**3. del语句和垃圾回收**

```python
a = object()
b = a
del a 
print(b)   # object实例
print(a)   # 抛异常
```

python中变量赋值的时候会增加引用，同一个变量赋值多次会累加多次引用，del并不是真正意义上的垃圾回收，只有引用次数为0的时候才会垃圾回收。

**4. 一个经典的错误**

```python
def add(a,b):
    a += b
    return a

class Company:
    def __init__(self,name,staffs=[]):
        self.name = name
        self.staffs = staffs

    def add(self, staff_name):
        self.staffs.append(staff_name)

    def remove(self,staff_name):
        self.staffs.remove(staff_name)

if __name__ == "__main__":
    com1 = Company("com1", ["bobby1", "bobby2"])
    com1.add("bobby3")
    com1.remove("bobby1")
    print(com1.staffs)

    com2 = Company("com2")
    com2.add("bobby")
    print(com2.staffs)

    print(Company.__init__.__defaults__)

    com3 = Company("com3")
    com3.add("bobby5")
    print(com2.staffs)
    print(com3.staffs)
    print(com2.staffs is com3.staffs)
    
# 结果
['bobby2', 'bobby3']
['bobby']
(['bobby'],)
['bobby', 'bobby5']
['bobby', 'bobby5']
True
```

可以看到com2和com3由于没有传入staffs初始化，所以使用了类里的默认值，是同一个值。

__init__.__defaults__可以查看类的默认属性