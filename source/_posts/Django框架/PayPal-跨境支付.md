---
title: PayPal--跨境支付
categories:
  - Django框架
tags:
  - PayPal
date: 2020-12-26 14:47:25
---



# 概述

> PayPal是倍受全球亿万用户追捧的国际贸易支付工具
>
> 是全球商户和消费者最受欢迎的电子支付方式之一，在跨境交易中有着超过90%的卖家和超过85%的买家认可并正在使用PayPal电子支付业务。当然，PayPal国际业务体量如此惊人，肯定不是毫无原因的
>
> PayPal支付的优势就是其业务网络遍布全球。目前PayPal的庞大网络覆盖了全球200多个国家，可提供20多种语言服务，并接受100多种货币付款和56种货币提现。同时，还允许在账户中持有25种货币余额。换句话说，只要付款人拥有一个PayPal账户，他就拥有了在200多个国家进行电子支付购物，并在需要服务的时候享受到母语支持的各种便捷服务。



# Python3 + Django2.2 + Vue集成PayPal跨境支付



## PayPal

1. 首先注册官网账号

> https://www.paypal.com

2. 开发者平台

> https://developer.paypal.com/developer/accounts/

3. 沙盒账号控制页面

> https://developer.paypal.com/developer/accounts/

:::info
会默认创建两个账号，一个是商户的，另外一个是个人的
:::



4. 进入应用管理页面,点[这里](https://developer.paypal.com/developer/accounts/)

- 记录下它的client_id和client_secret，一会要用到

![](1.png)



![](2.png)



## Django

1. 安装paypal在python端的SDK

```python
pip3 install paypalrestsdk
```



2. 生成支付链接

```python
import paypalrestsdk
from django.http import HttpResponse

def get_parpal_url(total_money):
    paypalrestsdk.configure({
        "mode": "sandbox",				# sandbox代表沙盒
        "client_id": "你的client_id",
        "client_secret": "你的client_secret"})

    payment = paypalrestsdk.Payment({
        "intent": "sale",
        "payer": {
            "payment_method": "paypal"},
        "redirect_urls": {
            "return_url": "http://localhost:8080/paypal_back/",		# 支付成功跳转页面
            "cancel_url": "http://localhost:3000/paypal/cancel/"},  # 取消支付页面
        "transactions": [{
            "amount": {
                "total": total_money,				# 金额
                "currency": "USD"},
            "description": "Paypal余额充值"}]})

    if payment.create():
        print("Payment created successfully")
        for link in payment.links:
            if link.rel == "approval_url":
                approval_url = str(link.href)
                print("Redirect for approval: %s" % (approval_url))
                return approval_url
    else:
        # print(payment.error)
        return HttpResponse("支付失败")
```



3. `views.py`

```python
from .Paypal import get_parpal_url
import paypalrestsdk

# 支付链接
class PayPalView(APIView):
    def post(self, request):
        uid = request.data.get('uid')
        pay_method = request.data.get('pay_method')
        total_money = int(request.data.get('total_money'))


        parpal_url = get_parpal_url(total_money)	# 调用刚刚写好的函数方法
        return Response({"code": 200, "msg": "返回PayPal支付url", "paypal_url": parpal_url})
```



- 支付完成后，会跳回刚刚传过去的回调页面
- paypal会传过来三个参数，支付id,token和支付者id
- 此时，在回调方法里，我们需要通过支付者id进行确认验证支付

```python
# 回调
class PayPal_back(APIView):
    def post(self, request):
        uid = request.data.get('uid')
        paymentid = request.data.get("paymentId")  # 订单id
        payerid = request.data.get("PayerID")  # 支付者id
        payment = paypalrestsdk.Payment.find(paymentid)
        print(payment)

        if payment.execute({"payer_id": payerid}):
            print("Payment execute successfully")
            return Response("支付成功")
        else:
            print(payment.error)  # Error Hash
            return Response("支付失败")
```



## vue



1. 支付页面

```html
<template>
<div>
    
    <div>
        <a-input placeholder="输入充值金额(美元)" type='number' v-model="money" style="width:500px" />

        <a-button type="primary" style="margin-top:20px;" @click="sub">
            确定
        </a-button>
    </div>

</div>
</template>
'
<script>
import {
    post_paypal_url,
} from '@/http/apis'
export default {
    data() {
        return {
            money:'',
            pay_method:''
        }
    },
    mounted(){
        this.pay_method = this.$route.query.pay_method
    },
    methods:{
        sub(){
            let params = {
                "uid":localStorage.getItem('uid'),
                "total_money":this.money,
                "pay_method":this.pay_method,
            }
            post_paypal_url(params).then(res=>{
                // 获取支付链接并跳转
                window.location.href = res.paypal_url
            })
        },
    }
}
</script>
```



2. 回调页面

```html
<template>
<div>
    <h1>PayPal回调</h1>
</div>
</template>

<script>
import {
    post_paypal_back,
} from '@/http/apis'
export default {
    data() {
        return {
            query: '',
        }
    },
    mounted() {
        // 支付完成后获取，PayPal传来的参数，发送给后端
        this.query = this.$route.query
        let params = {
            "uid": localStorage.getItem('uid'),
            'paymentId': this.query.paymentId,
            'PayerID': this.query.PayerID,
        }
        post_paypal_back(params).then(res => {
            console.log(res)
        })
    },
}
</script>
```





## 演示

![](PayPal.gif)

