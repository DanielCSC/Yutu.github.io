---
title: 三种风格的模型继承
categories:
  - Django框架
tags:
  - 
date: 2020-03-07 17:05:27
---





# Django三种风格的模型继承


- 抽象类继承：父类继承自models.Model，但不会在底层数据库中生成相应的数据表（在`Meta`方法中添加`abstract=True`），父类的属性列存储在其子类的数据表中。

- 多表继承：夺标继承的每个模型类都在底层数据库中生成相应的数据表管理数据。

- 代理模型继承：父类用于在底层数据库中管理数据表；而子类不定义数据列，只定义查询数据集的排列方式等元数据

#### 1.      抽象类(基类，公共类)继承
- 抽象类，指的是继承了model.Model且没有生成表，而是作为基类或公共类被其他类继承
- 抽象类继承的作用是在多个表有若干相同的字段时，可以使开发者讲这些字段统一定义在抽象基类中，免于重复定义这些字段。抽象积累的定义通过在模型的Meta中定义属性abstract=True来实现。抽象基类的举例如下：
```python
from django.db import models

class Base(models.Model):
    create_time = models.TimeField(auto_now_add=True)
    update_time = models.TimeField(auto_now=True)

    class Meta:
    	# 添加关键字，不会产生新的表
        abstract = True

# 这张表继承了Base，哪怕没有字段create_time和update_time，也同样会展示出来
class CourseType(Base):
    title = models.CharField('课程类别', max_length=16)
    sequence = models.IntegerField('展示顺序', default=18)

    class Meta:
        db_table = 'tb_coursetype'
```
本列中定义了一个抽象基类Base，用于保存2个公用字段：create_time、update_time 。子类CourseType继承自Base，并分别定义了自己的俩个字段。本列中2个类映射到数据库后，会被定义为1个数据表。

- 数据表tb_coursetype：有id、title 、sequence 、create_time 、update_time 5个字段

---

####  2.多表继承

- 在多表继承技术中心，无论是父表还是子表都会用数据库中相对应的数据表维护模型数据，父类中字段不会重复地在多个子类的相关数据表中进行定义。从这种意义上讲，多表继承才是真正面向对象的ORM技术。

- 多表继承的定义不需要特殊的关键字。在Django内部通过在父模型和子模型之间建立一对一关系来实现多表继承技术。如下代码定义了MessageBase及其子类的多表继承版本：


```python
from django.db import models

class MessageBase(models.Model):
    id = models.AutoField()
    content = models.CharField(max_length=100)
    user_name = models.CharField(max_length=80)
    pub_date = models.DateField()


class Moment(MessageBase):
    headline = models.CharField(max_length=50)


class Coment(MessageBase):
    level = models.CharField(max_length=1, choices=Levels)
```
 本例在数据库中会实际生成3个数据表。

- 数据表MessageBase：有id、content、user_name、pub_date等4个字段
- 数据表Moment：有id、headline等两个字段
- 数据表Comment：有id、level等两个字段

在对模型的编程过程中，子类仍然可以直接引用父类定义的字段；同时子类可以通过父类对象引用访问父类实例，比如：

```python
# 新建Moment对象，直接在子类中引用父类字段
>>> m1 = Moment(user_name="Tom", headline="hello world") 
>>> m1.content = "reference parent field in subclass"
>>> m1.save()

# 通过父类实例引用父类字段
>>> print m1.messagebase.content
eference parent field in subclass
```
技巧：在多表继承时，在子类实例中通过小写的父类的名字可以引用父类的实例

---

#### 3.代理模型继承

- 前两种继承模型中子类模型都有实际存储数据的作用；而代理模型继承中子类只用于管理父类的数据，而不实际存储数据。代理模型继承通过在子类的Meta中定义proxy=True属性来实现。举例如下：

```python
class Moment(models.Model):
    id = models.AutoField()
    content = models.CharField(max_length=100)
    user_name = models.CharField(max_length=80)
    pub_date = models.DateField()
    headline = models.CharField(max_length=50)


class OrderdMoment(Moment):
    class Meta:
        proxy = True
        ordering = ["-pub_date"]
```
在本例中定义了父类模型Moment用于存储数据，而后定义了子类模型OrderedMoment用于管理根据pub_date倒序排列的Moment。使用dialing模型继承的原因是子类中新的特性不会影响父类模型及其已有代码的行为。