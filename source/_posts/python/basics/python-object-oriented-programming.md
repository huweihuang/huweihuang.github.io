---
title: "[Python] Python系列（四）之面向对象编程"
catalog: true
date: 2017-09-11 11:50:57
type: "categories"
subtitle:
header-img: "https://res.cloudinary.com/dqxtn0ick/image/upload/v1508253812/header/cow.jpg"
tags:
- Python
catagories:
- Python
---

## 1. 类和实例

在Python中所有的数据类型都可以视为对象，也可以自定义对象。自定义的对象数据类型即面向对象中的类（Class）概念。类是抽象的模板，实例是具体的对象。

**类的定义**

Python使用`class`关键字定义类：

```python
class Student(object):

    def __init__(self, name, score):
        self.name = name
        self.score = score

    def print_score(self):
        print('%s: %s' % (self.name, self.score))
```

 **初始化函数**

类中使用`__init__`方法实现构造函数的功能，该函数的第一个参数的`self`，表示创建的实例本身，调用时传参只需要传入其他参数，不需要传入`self`参数，使用`__init__`方法则不能传入空参数。

**创建实例**

```python
>>> bart = Student('Bart Simpson', 59)
>>> bart.name
'Bart Simpson'
>>> bart.score
59
```

**类的方法**

类内部定义的方法来实现对类内部属性数据的封装，例如例子中的`print_score(self)`方法。定义一个类的方法除了第一个参数是`self`外，其他与函数的定义一样，调用方法时`self`参数不用传入，其他参数正常传入。

方法可以理解为与实例绑定的函数，可以直接访问实例内部的数据。

## 2. 访问限制

实例的变量以`__xxx`双下划线开头，没有以`__`双下划线结尾的，则该变量为私有变量，外部一般无法访问。例如：`self.__name = name`。

如果变量名是双下划线开头、双下划线结尾，`__xxx__`是特殊变量而不是私有变量。私有变量的方式可以避免外部对实例内部数据的修改，可以做参数检查。

如果需要获取或修改内部的数据，可以增加`get`和`set`的方法。例如：

```python
class Student(object):
	#构造函数
    def __init__(self, name, score):
        self.__name = name
        self.__score = score
    #get方法  
	def get_name(self):
        return self.__name

    def get_score(self):
        return self.__score
 	#set方法
    def set_score(self, score):
        #参数检查
        if 0 <= score <= 100:
            self.__score = score
        else:
            raise ValueError('bad score')
```

## 3. 继承和多态

### 3.1 继承

```python
#任何类都继承根类object
class Animal(object):
    def run(self):
        print('Animal is running...')
#Dog和Cat继承Animal        
class Dog(Animal):
    pass

class Cat(Animal):
    pass
#Dog和Cat的调用
dog = Dog()
dog.run()

cat = Cat()
cat.run()
#调用结果，继承Animal的run方法
Animal is running...
Animal is running...
```

例子中`Dog`和`Cat`类继承了`Animal`类，因此拥有父类的所有功能。

### 3.1 方法重写

子类可以对父类的方法进行重写（覆盖），方法名和方法参数都不变，方法的执行体改变。当对子类进行方法调用时是调用子类重写的方法而不是父类的方法。例如：

```python
#方法重写
class Dog(Animal):

    def run(self):
        print('Dog is running...')

class Cat(Animal):

    def run(self):
        print('Cat is running...')
        
#重写方法的调用结果
Dog is running...
Cat is running...
```

### 3.2 多态

在类型系统中，类也可以理解为一种特殊的数据类型，即类是变量类型，而是类的实例是变量名。也可以通过`isinstance()`函数来判断某实例是否属于某类。例如：

```python
b = Animal() # b是Animal类型
c = Dog() # c是Dog类型

>>> isinstance(b, Animal)
True
>>> isinstance(c, Dog)
True
#Dog类继承了Animal类，
>>> isinstance(c, Animal)
True
```

Dog继承了Animal类，因此Dog的实例的数据类型既可以是Dog（子类），也可以是Animal（父类）

[Dog是Animal的一种]，反之则不行。

**多态的示例**

```python
#定义一个函数参数为Animal（父类）类型
def run_twice(animal):
    animal.run()
    animal.run()
#入参为父类Animal   
>>> run_twice(Animal())
Animal is running...
Animal is running...
#入参为子类Dog
>>> run_twice(Dog())
Dog is running...
Dog is running...
#入参为子类Cat
>>> run_twice(Cat())
Cat is running...
Cat is running...

#新增一个子类
class Tortoise(Animal):
    def run(self):
        print('Tortoise is running slowly...')
#入参为子类Tortoise        
>>> run_twice(Tortoise())
Tortoise is running slowly...
Tortoise is running slowly...
```

通过以上示例可以看出，当函数的参数类型是父类，传入参数为子类的时候，调用的是具体传入的子类的方法。即调用方只管调用，不管具体的细节。细节由具体的运行对象决定，并且可以在不改变外部调用代码的情况下，增加调用实现的具体细节，即`开闭原则`。

**开闭原则**

对扩展开放：允许新增`Animal`子类；

对修改封闭：不需要修改依赖`Animal`类型的`run_twice()`等函数。

### 3.3 鸭子类型

Python是动态语言，在继承上并不需要像静态语言一样传入一个Animal类型或其子类，或者说不要求严格的继承体系。Python的“`file-like object`”满足“`鸭子类型`”，即一个对象只要看起来像鸭子，走起路来像鸭子，那它就可以被看做是鸭子。

所以并不需要传入一个Animal的类型或子类，只需要传入一个实现了Animal的run()方法的对象即可，例如：

```python
class Timer(object):
    def run(self):
        print('Start...')
```

## 4. 获取对象信息

### 4.1 type()

判断对象的类型可以使用`type()`函数，对象包括基本的数据类型和函数、类等。例如：

```python
#基本数据类型
>>> type(123)
<class 'int'>
>>> type('str')
<class 'str'>
>>> type(None)
<type(None) 'NoneType'>
#函数和类
>>> type(abs)
<class 'builtin_function_or_method'>
>>> type(a)
<class '__main__.Animal'>
```

**类型比较**

可以在`if`语句中比较两个对象的数据类型是否一致。

```python
>>> type(123)==type(456)
True
>>> type(123)==int
True
>>> type('abc')==type('123')
True
>>> type('abc')==str
True
>>> type('abc')==type(123)
False
```

**type()中常量**

`types`模块中定义的常量可以用于判断对象是否是函数、生成器等类型。

```python
>>> import types
>>> def fn():
...     pass
...
>>> type(fn)==types.FunctionType
True
>>> type(abs)==types.BuiltinFunctionType
True
>>> type(lambda x: x)==types.LambdaType
True
>>> type((x for x in range(10)))==types.GeneratorType
True
```

### 4.2 isinstance()

**继承关系判断**

使用`isinstance()`函数可以判断一个对象是否是该类型本身或者位于该类型的父继承链上。例如：

```python
#继承关系如下
object -> Animal -> Dog -> Husky
#创建实例
>>> a = Animal()
>>> d = Dog()
>>> h = Husky()
#继承关系判断
>>> isinstance(h, Husky)
True
>>> isinstance(h, Dog)
True
>>> isinstance(h, Animal)
True
```

以上的继承关系中，子类的类型除了本身的类型外，还可以归属于其祖类（父类及以上）的类型。

**基本数据类型的判断**

`isinstance()`也可以用来判断基本数据类型。

```python
#判断基本数据类型
>>> isinstance('a', str)
True
>>> isinstance(123, int)
True
>>> isinstance(b'a', bytes)
True
#判断某类型中的一种
>>> isinstance([1, 2, 3], (list, tuple))
True
>>> isinstance((1, 2, 3), (list, tuple))
True
```

>优先使用isinstance()判断类型

### 4.3 dir()

`dir()`函数可以获取一个对象的所有属性和方法，返回一个包含字符串的list。例如：

```python
>>> dir('ABC')
['__add__', '__class__',..., '__subclasshook__', 'capitalize', 'casefold',..., 'zfill']
```

`dir()`可以结合`getattr()`、`setattr()`以及`hasattr()`使用，例如：

```python
>>> class MyObject(object):
...     def __init__(self):
...         self.x = 9
...     def power(self):
...         return self.x * self.x
...
>>> obj = MyObject()

>>> hasattr(obj, 'x') # 有属性'x'吗？
True
>>> obj.x
9
>>> hasattr(obj, 'y') # 有属性'y'吗？
False
>>> setattr(obj, 'y', 19) # 设置一个属性'y'
>>> hasattr(obj, 'y') # 有属性'y'吗？
True
>>> getattr(obj, 'y') # 获取属性'y'
19
>>> obj.y # 获取属性'y'
19
```

此类函数是在未知对象内部数据时使用，例如从文件流中读取图像，先判断是否有`read()`方法，如果有则可以读取。

```python
def readImage(fp):
    if hasattr(fp, 'read'):
        return readData(fp)
    return None
```

## 5. 实例属性和类属性

Python中除了给实例绑定属性外，还可以给类绑定属性，即类属性。类属性归类所有，但该类的所有实例都可以访问到类属性。实例属性的优先级比类属性高，类属性和实例属性一般不使用相同名字，如果同名则实例属性会屏蔽掉类属性。

```python
>>> class Student(object):
...     name = 'Student'
    	def __init__(self, name):
        	self.name = name
...
>>> s = Student() # 创建实例s
>>> print(s.name) # 打印name属性，因为实例并没有name属性，所以会继续查找class的name属性
Student
>>> print(Student.name) # 打印类的name属性
Student
>>> s.name = 'Michael' # 给实例绑定name属性
>>> print(s.name) # 由于实例属性优先级比类属性高，因此，它会屏蔽掉类的name属性
Michael
>>> print(Student.name) # 但是类属性并未消失，用Student.name仍然可以访问
Student
>>> del s.name # 如果删除实例的name属性
>>> print(s.name) # 再次调用s.name，由于实例的name属性没有找到，类的name属性就显示出来了
Student
```



文章参考：[廖雪峰Python教程](https://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000/0014318645694388f1f10473d7f416e9291616be8367ab5000)
