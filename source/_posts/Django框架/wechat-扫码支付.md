---
title: Wechat--扫码支付
categories:
  - Django框架
tags:
  - Wechat
date: 2020-12-27 16:35:04
---



# 概述

> 在生活中真正具有广泛性、高效性、使用方便性的支付方式还得是扫码支付，扫码的优点在于推广成本低，上至钓鱼台国宾馆，下至发廊地摊都能用，打印出来就完事了，而相比其他支付方式，现金的找零及假钞问题，信用卡的办理门槛、pos机的沉没成本，所以基于二维码的扫码支付的确是非常符合国情的。



# Python3 + Django2.2 + Vue 实现微信扫码支付



## Wechat

1. 首先注册微信公众平台

> https://mp.weixin.qq.com

-  获得开发者id和秘钥(appid & appsecret)

![](1.png)

- 同时确保获取微信支付接口的权限

![](2.png)



2.  随后注册微信支付商户平台

> https://pay.weixin.qq.com/



- 获取微信支付的商户号(在账户信息页面)

![](3.png)



- 获取微信支付接口的秘钥(账户中心->api安全)

![](4.png)



- 产品中心->开发配置页面，配置支付域名

![](5.png)



:::info
扫码支付域名既支持https也支持http，非常方便，同时注意域名必须是一个备案域名。
:::



## Django

1. 首先导入依赖的库和一些工具方法

```python
import requests
from django.http import HttpResponse, HttpResponseRedirect

import random
import time
import hashlib

import qrcode
from bs4 import BeautifulSoup

def trans_xml_to_dict(data_xml):
    soup = BeautifulSoup(data_xml, features='xml')
    xml = soup.find('xml')  # 解析XML
    if not xml:
        return {}
    data_dict = dict([(item.name, item.text) for item in xml.find_all()])
    return data_dict

def trans_dict_to_xml(data_dict):  # 定义字典转XML的函数
    data_xml = []
    for k in sorted(data_dict.keys()):  # 遍历字典排序后的key
        v = data_dict.get(k)  # 取出字典中key对应的value
        if k == 'detail' and not v.startswith('<![CDATA['):  # 添加XML标记
            v = '<![CDATA[{}]]>'.format(v)
        data_xml.append('<{key}>{value}</{key}>'.format(key=k, value=v))
    return '<xml>{}</xml>'.format(''.join(data_xml))  # 返回XML

def get_sign(data_dict, key):  # 签名函数，参数为签名的数据和密钥
    params_list = sorted(data_dict.items(), key=lambda e: e[0], reverse=False)  # 参数字典倒排序为列表
    params_str = "&".join(u"{}={}".format(k, v) for k, v in params_list) + '&key=' + key
    # 组织参数字符串并在末尾添加商户交易密钥
    md5 = hashlib.md5()  # 使用MD5加密模式
    md5.update(params_str.encode())  # 将参数字符串传入
    sign = md5.hexdigest().upper()  # 完成加密并转为大写
    return sign
```



2. 编写支付逻辑，参考微信官方文档

> https://pay.weixin.qq.com/wiki/doc/api/native.php?chapter=6_5&index=3



+++primary 业务流程说明

（1）商户后台系统根据用户选购的商品生成订单。

（2）用户确认支付后调用微信支付【统一下单API】生成预支付交易；

（3）微信支付系统收到请求后生成预支付交易单，并返回交易会话的二维码链接code_url。

（4）商户后台系统根据返回的code_url生成二维码。

（5）用户打开微信“扫一扫”扫描二维码，微信客户端将扫码内容发送到微信支付系统。

（6）微信支付系统收到客户端请求，验证链接有效性后发起用户支付，要求用户授权。

（7）用户在微信客户端输入密码，确认支付后，微信客户端提交授权。

（8）微信支付系统根据用户授权完成支付交易。

（9）微信支付系统完成支付交易后给微信客户端返回交易结果，并将交易结果通过短信、微信消息提示用户。微信客户端展示支付交易结果页面。

（10）微信支付系统通过发送异步消息通知商户后台系统支付结果。商户后台系统需回复接收情况，通知微信后台系统不再发送该单的支付通知。

（11）未收到支付通知的情况，商户后台系统调用【查询订单API】。

（12）商户确认订单已支付后给用户发货。

+++



- 我们需要调用微信的统一下单接口文档

> https://pay.weixin.qq.com/wiki/doc/api/native.php?chapter=9_1



-  编写逻辑

```python
def wx_pay(request):

    url = 'https://api.mch.weixin.qq.com/pay/unifiedorder'  # 微信扫码支付接口
    key = '945bec********a8fbf7d7'  #商户api秘钥
    total_fee = 1 #支付金额，单位分
    body = '123123'  # 商品描述
    out_trade_no = 'order_%s' % random.randrange(100000, 999999)  # 订单编号
    params = {
        'appid': 'wx09*****f',  # APPID
        'mch_id': '16****08',  # 商户号
        'notify_url': 'http://wxpay.v3u.cn/wx_back/',  # 支付域名回调地址
        'product_id': 'goods_%s' % random.randrange(100000, 999999),  # 商品编号
        'trade_type': 'NATIVE',  # 支付类型（扫码支付）
        'spbill_create_ip': '114.254.176.137',  # 发送请求服务器的IP地址
        'total_fee': total_fee,  # 订单总金额
        'out_trade_no': out_trade_no,  # 订单编号
        'body': body,  # 商品描述
        'nonce_str': 'ibuaiVcKdpRxkhJA'  # 字符串
    }
    sign = get_sign(params, key)  # 获取签名
    params.setdefault('sign', sign)  # 添加签名到参数字典
    xml = trans_dict_to_xml(params)  # 转换字典为XML
    response = requests.request('post', url, data=xml)  # 以POST方式向微信公众平台服务器发起请求
    data_dict = trans_xml_to_dict(response.content)  # 将请求返回的数据转为字典
    print(data_dict)
    qrcode_name = out_trade_no + '.png'  # 支付二维码图片保存路径
    if data_dict.get('return_code') == 'SUCCESS':  # 如果请求成功
        img = qrcode.make(data_dict.get('code_url'))  # 创建支付二维码片
        img.save('./' + qrcode_name)  # 保存支付二维码
    return HttpResponse(qrcode_name)
```



3.  访问http://localhost:8000/wx_pay/

![](6.png)



## Vue

```html
<template>

  <div>

    <center><h1>扫码支付</h1></center>

 
    <a-form-item v-bind="formItemLayout" label="金额">
      <a-input v-model="money"/>
    </a-form-item>
    
   
    <a-form-item v-bind="tailFormItemLayout">
      <a-button type="primary" html-type="submit" @click="submit">
        生成二维码
      </a-button>
    </a-form-item>


    <a-form-item v-bind="formItemLayout" label="二维码">
    
      <img :src="src" />

    </a-form-item>


   </div>

</template>



<script>

export default {
  data() {
    return {
      money:"1",
      src:"",
      formItemLayout: {
        labelCol: {
          xs: { span: 24 },
          sm: { span: 8 },
        },
        wrapperCol: {
          xs: { span: 24 },
          sm: { span: 16 },
        },
      },
      tailFormItemLayout: {
        wrapperCol: {
          xs: {
            span: 24,
            offset: 0,
          },
          sm: {
            span: 16,
            offset: 8,
          },
        },
      },
      dataSource: [
        {
          key: '0',
          name: 'Edward King 0',
          age: '32',
          address: 'London, Park Lane no. 0',
        },
        {
          key: '1',
          name: 'Edward King 1',
          age: '32',
          address: 'London, Park Lane no. 1',
        },
      ],
      columns: [
        {
          title: 'name',
          dataIndex: 'name',
        },
        {
          title: 'age',
          dataIndex: 'age',
        },
        {
          title: 'address',
          dataIndex: 'address',
        },
        {
          title: 'operation',
          dataIndex: 'operation',
          scopedSlots: { customRender: 'operation' },
        },
      ],
    };
  },
  methods: {

    submit:function(){

       this.axios.get('http://localhost:8000/wx_pay/').then((result) =>{
    
            console.log(result.data.img);

            this.src = "http://localhost:8000/static/upload/"+result.data.img
  
  });

    },
    onDelete(key) {
      console.log(this.dataSource[key]);
    }

  },
};
</script>
```



- 当用户点击按钮之后，请求后端支付接口，将接口生成的二维码返回给前端

![](7.png)



- 微信进行扫码支付，需要注意的是，该二维码有效期只有五分钟

![](8.png)



- 支付成功之后，我们还需要对交易进行确认，所以根据微信官方文档，调用统一查询接口

> https://pay.weixin.qq.com/wiki/doc/api/native.php?chapter=9_2



```python
def wx_check(request):

    #统一订单查询接口

    url = "https://api.mch.weixin.qq.com/pay/orderquery"

    out_trade_no = "order_537236" #支付后的商户订单号
    key = '945b******d7'  # 商户api密钥

    params = {
        'appid': 'wx0*****ff',  # APPID
        'mch_id': '16*****08',  # 商户号
        'out_trade_no': out_trade_no,  # 订单编号
        'nonce_str': 'ibuaiVcKdpRxkhJA'  # 随机字符串
    }
    sign = get_sign(params, key)  # 获取签名
    params.setdefault('sign', sign)  # 添加签名到参数字典
    xml = trans_dict_to_xml(params)  # 转换字典为XML
    response = requests.request('post', url, data=xml)  # 以POST方式向微信公众平台服务器发起请求
    data_dict = trans_xml_to_dict(response.content)  # 将请求返回的数据转为字典
    print(data_dict)

    return HttpResponse('ok')
```

:::info
查询的订单编号可以使商户自己的订单编号，也可以是微信订单号，二者必取其一
:::

![](9.png)



- 访问接口 http://localhost:8000/wx_check/

  返回结果：

```python
返回结果：{'return_code': 'SUCCESS', 'return_msg': 'OK', 'appid': 'wx092344a76b9979ff', 'mch_id': '1602932608', 'nonce_str': 'BVoaDmxxADkpSFEl', 'sign': '23A86EB406B743E0C2C61C7E78DC9373', 'result_code': 'SUCCESS', 'openid': 'oy9q36f9Dpeokj9FWyN3j0znpIqE', 'is_subscribe': 'N', 'trade_type': 'NATIVE', 'bank_type': 'OTHERS', 'total_fee': '1', 'fee_type': 'CNY', 'transaction_id': '4200000806202012174121934231', 'out_trade_no': 'order_537236', 'attach': ' ', 'time_end': '20201217231553', 'trade_state': 'SUCCESS', 'cash_fee': '1', 'trade_state_desc': '支付成功', 'cash_fee_type': 'CNY'}
```

