---
title: Django--数据库ORM操作
categories:
  - Django框架
tags:
  - ORM
date: 2020-03-20 17:07:42
---







# 创建单表模型类

```python
class Goods(models.Model):
    goods_name = models.CharField(max_length=32)
    goods_price = models.DecimalField(max_digits=9,decimal_places=2)
    goods_num = models.IntegerField()

    class Meta:
        db_table = 'tb_goods'
```

# ORM基本操作

```python
class ORMView(APIView):
    def get(self,request):
        #   单一查询，如果结果不存在报错
        goodsobj = Goods.objects.get(goods_price=6.66)
        obj = GoodsSerializers(goodsobj)
        return Response(obj.data)


        #   查询不包含id=3的数据
        goodsobj = Goods.objects.exclude(id=3)
        obj = GoodsSerializers(goodsobj, many=True)
        return Response(obj.data)


        #   查询结果的数量
        goodsobj = Goods.objects.all().count()
        return Response(goodsobj)


        #   聚合函数　　使用aggregate()过滤器调用聚合函数。
        #   聚合函数包括：Avg 平均，Count 数量，Max 最大，Min 最小，Sum 求和      
        #   需要导包    from django.db.models import *
        goodsobj = Goods.objects.aggregate(Sum('goods_num'))
        print(goodsobj)
        # obj = GoodsSerializers(goodsobj, many=True)
        return Response({'data':''})



        #   比较查询　　字段名__lt：小于    字段名__lte：小于等于
        goodsobj = Goods.objects.filter(id__lt=4)
        obj = GoodsSerializers(goodsobj, many=True)
        return Response(obj.data)


        #   比较查询　　字段名__gt：大于    字段名__gte：大于等于
        goodsobj = Goods.objects.filter(id__gte=2)
        obj = GoodsSerializers(goodsobj, many=True)
        return Response(obj.data)


        #  　空查询　　字段名__isnull
        goodsobj = Goods.objects.filter(goods_name__isnull=False)
        obj = GoodsSerializers(goodsobj, many=True)
        return Response(obj.data)


        #    模糊查询　字段名__endswith　以　莓　结尾的数据
        goodsobj = Goods.objects.filter(goods_name__endswith='莓')
        obj = GoodsSerializers(goodsobj, many=True)
        return Response(obj.data)


        #    模糊查询　字段名__tartswith　以　苹　开头的数据
        goodsobj = Goods.objects.filter(goods_name__startswith='苹')
        obj = GoodsSerializers(goodsobj,many=True)
        return Response(obj.data)

        #    模糊查询　字段名__contains　包含　果　的的数据
        goodsobj = Goods.objects.filter(goods_name__contains='果')
        obj = GoodsSerializers(goodsobj,many=True)
        return Response(obj.data)


        #    范围查询:  in  只查询列表中的具体值
        goodsobj = Goods.objects.filter(goods_price__in=[12,25,6.66]).all()
        obj = GoodsSerializers(goodsobj,many=True)
        return Response(obj.data)

        #    范围查询:  range １～２０之间
        goodsobj = Goods.objects.filter(goods_price__range=[1,20]).all()
        obj = GoodsSerializers(goodsobj,many=True)
        return Response(obj.data)

        #    排序查询：  order_by 降序只需在字段名前加上 “ - ”
        goodsobj = Goods.objects.all().order_by('-goods_num')
        obj = GoodsSerializers(goodsobj, many=True)
        return Response(obj.data)

        #    F方法：比较两个字段对象之间的关系用F对象，F方法可以进行简单运算　
        #    查询价格　大于等于　库存的对象
        goodsobj = Goods.objects.filter(goods_price__gte=F('goods_num'))
        obj = GoodsSerializers(goodsobj, many=True)
        return Response(obj.data)
        
        #    Q方法：对对象进行复杂查询，并支持&（and）,|（or），~（not）操作符
        #    查询价格大于等于15的数据　或库存大于等于10的数据
        goodsobj = Goods.objects.filter(Q(goods_price__gte=15) | Q(goods_num__gte=10))
        obj = GoodsSerializers(goodsobj, many=True)
        return Response(obj.data)
```


# 创建一对多，多对多模型类

```python
# 电影
class Movie(models.Model):
    movie_name = models.CharField(max_length=32)
    class Meta:
        db_table = 'tb_movie'


# 角色
class Role(models.Model):
    role_name = models.CharField(max_length=32)
    role_price = models.DecimalField(max_digits=9, decimal_places=2)
    role_desc = models.CharField(max_length=32)

    class Meta:
        db_table = 'tb_role'


# 演员
class Actor(models.Model):
    name = models.CharField(max_length=32)
    address = models.CharField(max_length=32, null=True, blank=True)
    age = models.IntegerField(null=True,blank=True)
    phone = models.CharField(max_length=32, null=True, blank=True)
    movie = models.ForeignKey(Movie, on_delete=models.CASCADE)
    role = models.ManyToManyField(Role)

    class Meta:
        db_table = 'tb_actor'
```

# 一对多的正、反查询

```python
class ORMView(APIView):
    def get(self,request):
        #   一对多　正向查找      对象．模型名．字段
        actorobj = Actor.objects.get(id=1)
        movieo = actorobj.movie.movie_name
        print(movieo)
        return Response('ok')

        #   一对多　反向查找      对象．模型名_set
        #  除了可以使用模型名_set，还有一种是在建立模型类的时候使用related_name来指定变量名。
        movieobj = Movie.objects.get(id=1)
        actorobj = movieobj.actor_set.all()
        obj = ActorSerializers(actorobj,many=True)
        print(obj.data)
        return Response(obj.data)

```

# 多对多表操作

```python

class ORMview2(APIView):
    def post(self,request):
        # 多对多添加
        # 方法一：在建立manytomany的models里添加数据，（一条，一个对象）
        actor = Actor.objects.get(id=5)
        role = Role.objects.get(id=5)

        data = actor.role.add(role)
        print(data)
        return Response('ok')

        # 方法二：在未建立manytomany的models里添加数据，（一条，一个对象）
        actor = Actor.objects.filter(name='张译')
        role = Role.objects.get(id=5)

        data = role.actor_set.add(*actor)
        print(data)
        return Response('ok2')


    def put(self,request):
        # 多对多更新
        # 方法一：在建立manytomany的models里修改数据，参数只能是可迭代对象
        actorobj = Actor.objects.filter(id=5).first()
        roleobj = Role.objects.filter(id=4)

        actorobj.role.set(roleobj)
        return Response('ok')

        # 方法二：在未建立manytomany的models里修改数据，参数只能是可迭代对象
        actorobj = Actor.objects.filter(id=5)
        roleobj = Role.objects.filter(id=5).first()

        roleobj.actor_set.set(actorobj)
        return Response('ok2')


    def delete(self,request):
        # 方法一：在建立manytomany的models里删除数据，（一条，一个对象）
        autorobj = Actor.objects.get(id=5)
        roleobj =Role.objects.get(id=5)

        autorobj.role.remove(roleobj)
        return Response('ok')

        # 方法二：在未建立manytomany的models里删除数据，（一条，可迭代对象）
        autorobj = Actor.objects.get(id=5)
        roleobj = Role.objects.get(id=5)

        roleobj.actor_set.remove(autorobj)
        return Response('ok2')

```

