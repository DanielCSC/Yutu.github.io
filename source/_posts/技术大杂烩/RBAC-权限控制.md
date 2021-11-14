---
title: RBAC--权限控制
categories:
  - 技术大杂烩
tags:
  - null
date: 2020-12-21 20:41:10
---



# 概述

> `RBAC 是基于角色的访问控制（Role-Based Access Control ）`在 RBAC 中，权限与角色相关联，用户通过成为适当角色的成员而得到这些角色的权限。这就极大地简化了权限的管理。这样管理都是层级相互依赖的，权限赋予给角色，而把角色又赋予用户，这样的权限设计很清楚，管理起来很方便。



# 简易表设计

```python
from django.contrib.auth.models import AbstractUser
from django.db import models

class User(AbstractUser):
    username = models.CharField('用户名', max_length=50, unique=True)
    password = models.CharField('密码', max_length=500)
    email = models.EmailField('邮箱', max_length=255, null=True, blank=True)
    phone = models.CharField('电话', max_length=13, null=True, blank=True)
    role = models.ForeignKey('Role',on_delete=models.CASCADE,null=True,blank=True)


class Role(models.Model):
    rolename = models.CharField('角色名',max_length=32)


class Permission(models.Model):
    role = models.ForeignKey('Role',on_delete=models.CASCADE)
    node = models.ForeignKey('Node',on_delete=models.CASCADE)
    class Meta:
        db_table = 'tb_permission'


class Node(models.Model):
    nodename = models.CharField('节点',max_length=32)
```



# 位运算

>在计算机内存中都是以二进制的形式储存的。位运算就是直接对整数在内存中的二进制位进行操作。
>
>举个例子，6的二进制是110，11的二进制是1011，那么6 &11的结果就是2，它是二进制对应位进行逻辑运算的结果（0表示False，1表示True，空位都当0处理）

- 在`python`中`0b`代表二进制

![](1.png)



## 使用位运算实现简单权限demo

```python
user = 0b110
permission = {"user":0b100,"blacklist":0b010,"login":0b001}

def my_decorator(func):
    def wrapper(*args):
        print(args)
        # 进行拦截
        if not user &  permission[args[0]]:
            print("没权限")
        else:
            func(*args)
    return wrapper

@my_decorator
def view(user):
    print('进入了视图')
    pass
view('user')
```

运算方式：

![](2.png)



```python
# 运行后结果为
# 1. view('user')：
('user',)
进入了视图
# 2. view('blacklist')：
('blacklist',)
进入了视图
#3.view('login')
('login',)
没权限
```



> 从结果中可以看出，当访问该视图的用户权限为`0b110`时，只可访问`user`,`blacklist`，不可访问`login`



