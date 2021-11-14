---
title: ECharts--数据可视化库
categories:
  - 技术大杂烩
tags:
  - 数据可视化库
date: 2020-12-22 23:26:23
---



# 概述

> `ECharts`，一个使用 `JavaScript` 实现的`开源可视化库`，可以流畅的运行在 PC 和移动设备上，兼容当前绝大部分浏览器（IE8/9/10/11，Chrome，Firefox，Safari等），底层依赖矢量图形库 [ZRender](https://github.com/ecomfe/zrender)，提供直观，交互丰富，可高度个性化定制的数据可视化图表。



# 在Vue中使用ECharts

1. 安装

```js
npm install echarts --save
```

2. `main.js`

```js
import echarts from "echarts";
Vue.prototype.$echarts = echarts;
```



:::info
`注意`：如果引入报错：`“export ‘default‘ (imported as ‘echarts‘) was not found in ‘echarts‘...`切换版本

:::

```js
npm install echarts@4.9.0
npm fund
npm run dev
```



3. Demo

```html
<template>
<div>
    <!-- 为ECharts准备一个具备大小（宽高）的Dom -->
    <div id="main" style="width: 1500px;height:600px;"></div>
</div>
</template>

<script>
export default {
    data() {
        return {}
    },
    mounted() {
        this.drawChart()
    },
    methods: {
        drawChart() {
            // 基于准备好的dom，初始化echarts实例
            let myChart = this.$echarts.init(document.getElementById("main"));
            // 指定图表的配置项和数据
            var option = {
                title: {
                    text: ''
                },
                tooltip: {},
                legend: {
                    data: ['销量']
                },
                xAxis: {
                    data: ["衬衫", "羊毛衫", "雪纺衫", "裤子", "高跟鞋", "袜子"]
                },
                yAxis: {},
                series: [{
                    name: '销量',
                    type: 'bar',
                    data: [5, 20, 36, 10, 10, 20]
                }]
            };

            // 使用刚指定的配置项和数据显示图表。
            myChart.setOption(option);
        }
    },
}
</script>
```



4. 效果

![](demo.gif)



# Django and Vue 实现股票简易柱状图



## Django

1. 爬取数据

``` python
import requests
import pymongo

conn = pymongo.MongoClient()			# 连接Mongodb数据库
db = conn.gupiao
table = db.gp

url = 'https://data.gtimg.cn/flashdata/hushen/latest/daily/sz000002.js?maxage=43201&visitDstTime=1'
res = requests.get(url)
res.encoding = 'utf-8'				# 出现乱码使用,对获取到的网页源码进行一个utf-8解码
res=res.text.split()[34:-1]

res=res.text.split('\n')[3:-1]
riqi = []
s1 =[]
s2 =[]
s3 =[]
s4 =[]
for i in s:
    ss=i[:-3].split(' ')
    riqi.append(ss[0])
    s1.append(ss[1])
    s2.append(ss[2])
    s3.append(ss[3])
    s4.append(ss[4])
# 将数据以列表形式存储
a = table.insert({'date':list(riqi),'s1':list(s1),'s2':list(s2),'s3':list(s3),'s4':list(s4)})
```



2. 向前端发送数据

```python
import pymongo

conn = pymongo.MongoClient()  # 链接Mongodb数据库
db = conn.gupiao
table = db.gp

class Gupiao_data(APIView):
    def get(self,reqeust):
        list_data=[]
        data=table.find()
        for i in data:
            i.pop('_id')
            list_data.append(i)
        return Response({'data':list_data})
```



## Vue

```html
<template>
<div>
    <!-- 为ECharts准备一个具备大小（宽高）的Dom -->
    <div id="main" style="width: 1500px;height:600px;"></div>
</div>
</template>

<script>
import { get_gupiao } from '@/http/apis'
export default {
    data() {
        return {},
    },
    mounted() {
        this.drawChart();
    },
    methods: {
        drawChart() {
            // 基于准备好的dom，初始化echarts实例
            let myChart = this.$echarts.init(document.getElementById("main"));
            let riqi = []
            let s1 = []
            let s2 = []
            let s3 = []
            let s4 = []
            get_gupiao().then(res => {
                for (var i in res.data[0].date) {
                    // 指定图表的配置项和数据
                    let option = {
                        tooltip: {
                            trigger: 'axis',
                            axisPointer: { // 坐标轴指示器，坐标轴触发有效
                                type: 'shadow' // 默认为直线，可选为：'line' | 'shadow'
                            }
                        },
                        legend: {
                            data: ["上午开盘", "上午收盘", "下午开盘", "下午收盘"]
                        },
                        grid: {
                            left: '3%',
                            right: '4%',
                            bottom: '3%',
                            containLabel: true
                        },
                        xAxis: [{
                            type: 'category',
                            data: riqi
                        }],
                        yAxis: [{
                            type: 'value'
                        }],
                        series: [{
                                name: '上午开盘',
                                type: 'bar',
                                data: s1
                            },
                            {
                                name: '上午收盘',
                                type: 'bar',
                                stack: '广告',
                                data: s2
                            },
                            {
                                name: '下午开盘',
                                type: 'bar',
                                stack: '广告',
                                data: s3
                            },
                            {
                                name: '下午收盘',
                                type: 'bar',
                                stack: '广告',
                                data: s4
                            },
                        ]
                    };
                    riqi.push(res.data[0].date[i])
                    s1.push(res.data[0].s1[i])
                    s2.push(res.data[0].s2[i])
                    s3.push(res.data[0].s3[i])
                    s4.push(res.data[0].s4[i])
                    // 使用刚指定的配置项和数据显示图表。
                    myChart.setOption(option);
                }
            })
        },
    },
}
</script>
```



## 效果

![](柱状图.gif)

