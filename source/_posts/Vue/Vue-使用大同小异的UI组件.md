---
title: UI组件
categories:
  - Vue
tags:
  - UI组件

date: 2020-08-24 20:27:35
---







### 1. ANT DESIGN of Vue

<center><img src="1.png"  /></center>




> Ant Design是蚂蚁金服技术部经过大量项目时间和总结，设计的前端UI组件库。
>
> 致力于提升『用户』和『设计者』使用体验的中台设计语言。它模糊了产品经理、交互设计师、视觉设计师、前端工程师、开发工程师等角色边界，将进行 UE 设计和 UI 设计人员统称为『设计者』，利用统一的规范进行设计赋能，全面提高中台产品体验和研发效率。
>
> Ant Design Vue 是使用 Vue 实现的遵循 Ant Design 设计规范的高质量 UI 组件库



#### 特性

- 🌈 提炼自企业级中后台产品的交互语言和视觉风格。

- 📦 开箱即用的高质量Vue组件

- 🎉共享[Ant Design of React](http://ant-design.gitee.io/docs/spec/introduce-cn)设计工具体系。

- ⚙️ 全链路开发和设计工具体系。

- 🌍 数十个国际化语言支持。

- 🎨 深入每个细节的主题定制能力。

  



#### 使用安装



> #### 1.安装
>
> npm install --save ant-design-vue



> #### 2.main.js中全局引入并注册
>import Antd from 'ant-design-vue';
> import 'ant-design-vue/dist/antd.css';
>Vue.use(Antd);



> #### 3.在页面中不再需要引入注册组件，可以直接使用所有的组件
>
> ```vue
> <template>
>  <div>    
>  	<a-button type="primary">按钮</a-button>  
> 	</div> 
> </template>
> ```

#### 资源地址

- [官方文档](https://vue.ant.design/docs/vue/introduce-cn/)
- [Ant Design Pro演示](https://preview.pro.antdv.com)
- [Ant Design Pro项目源码](https://github.com/sendya/ant-design-pro-vue )



---



### 2. Element UI

<center><img src="2.png" style="zoom:200%;" /></center>




> Element，一套为开发者、设计师和产品经理准备的基于 Vue 2.0 的组件库，提供了配套设计资源，帮助网站快速成型。由饿了么公司前端团队开源。(目前已停止维护)



#### 设计原则

##### 一致性 Consistency

- 与现实生活一致：与现实生活的流程、逻辑保持一致，遵循用户习惯的语言和概念；
- 在界面中一致：所有的元素和结构需保持一致，比如：设计样式、图标和文本、元素的位置等。



##### 反馈 Feedback

- 控制反馈：通过界面样式和交互动效让用户可以清晰的感知自己的操作；
- 页面反馈：操作后，通过页面元素的变化清晰地展现当前状态。



##### 效率 Efficiency

- 简化流程：设计简洁直观的操作流程；
- 清晰明确：语言表达清晰且表意明确，让用户快速理解进而作出决策；
- 帮助用户识别：界面简单直白，让用户快速识别而非回忆，减少用户记忆负担。



##### 可控 Controllability

- 用户决策：根据场景可给予用户操作建议或安全提示，但不能代替用户进行决策；
- 结果可控：用户可以自由的进行操作，包括撤销、回退和终止当前操作等。



#### 使用



> #### 1.安装
> npm i element-ui -S



> #### 2.main.js中全局引入并注册
> import ElementUI from 'element-ui';
>import 'element-ui/lib/theme-chalk/index.css';
> Vue.use(ElementUI);



> #### 3.在页面中不再需要引入注册组件，可以直接使用所有的组件
> 
>```vue
> <template>
>  <div>    
>  	<el-button type="primary">主要按钮</el-button>
> 	</div> 
></template>
> ```

#### 资源地址

- [官方文档](https://element.eleme.io/#/zh-CN/component/installation)
- [vue-element-admin演示](https://panjiachen.github.io/vue-element-admin/#/login )
- [vue-element-admin项目文档](https://panjiachen.github.io/vue-element-admin-site/zh/ )
- [vue-element-admin项目源码](https://github.com/PanJiaChen/vue-element-admin )

