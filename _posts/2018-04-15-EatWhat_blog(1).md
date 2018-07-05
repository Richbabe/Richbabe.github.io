---
layout:     post
title:      微信小程序开发入门
subtitle:   
date:       2018-04-15
author:     Richbabe
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - 微信小程序
---
## 引言
> 这学期系统分析与设计的项目要求我们做一个扫码点餐的系统，在经过讨论后我们决定做一个扫码点餐的小程序，前端由我和另一个同学（黄伟浩）负责，借博客记录一下我学习微信小程序前端开发的过程。

## 开发前的准备
首先请先阅读微信小程序官方的教程：
> [小程序官方教程](https://developers.weixin.qq.com/miniprogram/dev/index.html?t=2018323)

通过此教程，你可以申请一个属于你团队的小程序开发账号，通过该账号你可以管理、开发你的小程序，接着便需要安装开发者工具，链接如下：
> [微信小程序开发者工具下载连接](https://developers.weixin.qq.com/miniprogram/dev/devtools/download.html?t=2018412)

有关开发者工具的更多详细介绍可参考：
> [开发者工具详细介绍](https://developers.weixin.qq.com/miniprogram/dev/devtools/devtools.html?t=2018412)

在完成开发者工具的安装后，我们便可以开始我们的小程序开发了！

## 初步接触微信小程序

### 你的第一个小程序

新建项目选择小程序项目，选择代码存放的硬盘路径，填入刚刚申请到的小程序的 AppID，给你的项目起一个好听的名字，最后，勾选 "创建 QuickStart 项目" (注意: 你要选择一个空的目录才会有这个选项)，点击确定，你就得到了你的第一个小程序了，点击顶部菜单编译就可以在 IDE 预览你的第一个小程序。

![image](https://github.com/Richbabe/Richbabe.github.io/blob/master/img/%E5%B0%8F%E7%A8%8B%E5%BA%8F%E5%BC%80%E5%8F%91%E6%88%AA%E5%9B%BE/%E7%95%8C%E9%9D%A2%E4%BB%8B%E7%BB%8D1.jpg?raw=true) 

### 小程序代码构成
从你刚刚生成的项目中，你可以留意到这个项目里边生成了不同类型的文件:
* .json 后缀的 JSON 配置文件
* .wxml 后缀的 WXML 模板文件
* .wxss 后缀的 WXSS 样式文件
* .js 后缀的 JS 脚本逻辑文件

接下来我们分别看看这4种文件的作用。

#### JSON配置

我们可以看到在项目的根目录有一个app.json和project.config.json,此外在pages/logs目录下还有一个logs.json，我们一次来说明一下他们的用途

##### 小程序配置 app.json
> app.json是对当前小程序的全局配置，包括了小程序的所有页面路径、界面表现、网络超时时间、底部tab等，新建项目里边的app.json配置内容如下：

```
{
  "pages":[
    "pages/index/index",
    "pages/logs/logs"
  ],
  "window":{
    "backgroundTextStyle":"light",
    "navigationBarBackgroundColor": "#fff",
    "navigationBarTitleText": "WeChat",
    "navigationBarTextStyle":"black"
  }
}
```
我们简单说一下这个配置各个项的含义：

1. pages字段 —— 用于描述当前小程序所有页面路径，这是为了让微信客户端知道当前你的小程序页面定义在哪个目录。
2. window字段 —— 小程序所有页面的顶部背景颜色，文字颜色定义在这里的。

更多配置项细节可以参考文档
[小程序的配置app.json](https://developers.weixin.qq.com/miniprogram/dev/framework/config.html)

在这里我实现了能在顶部切换界面的功能，app.json中代码如下：

```
{
  "pages":[
    "pages/index/index",
    "pages/menu/menu",
    "pages/logs/logs"
  ],
  "window":{
    "backgroundTextStyle":"light",
    "navigationBarBackgroundColor": "#ffffff",
    "navigationBarTitleText": "食咩阿",
    "navigationBarTextStyle":"black"
  },
  "tabBar":{
    "list":[{
      "pagePath":"pages/index/index",
      "text":"首页"
    },{
      "pagePath":"pages/menu/menu",
      "text":"菜单"
    }],
    "selectedColor":"#ff0000",
    "backgroundColor":"#ffffff",
    "position":"top"
  },
  "networkTimeout":{
    "request":10000,
    "downloadFile":10000
  },
  "debug":true
}
```
效果图为：

![image](https://github.com/Richbabe/Richbabe.github.io/blob/master/img/%E5%B0%8F%E7%A8%8B%E5%BA%8F%E5%BC%80%E5%8F%91%E6%88%AA%E5%9B%BE/appJson%E6%BC%94%E7%A4%BA1.jpg?raw=true)
![image](https://github.com/Richbabe/Richbabe.github.io/blob/master/img/%E5%B0%8F%E7%A8%8B%E5%BA%8F%E5%BC%80%E5%8F%91%E6%88%AA%E5%9B%BE/appJson%E6%BC%94%E7%A4%BA2.jpg?raw=true)

##### 工具配置 project.config.json
> 通常大家在使用一个工具的时候，都会针对各自喜好做一些个性化配置，例如界面颜色、编译配置等等，当你换了另外一台电脑重新安装工具的时候，你还要重新配置。
考虑到这点，小程序开发者工具在每个项目的根目录都会生成一个project.config.json，你在工具上做的任何配置都会写入到这个文件，当你重新安装工具或者换电脑工作时，你只要载入同一个项目的代码包，开发者工具就自动会帮你恢复到当时你开发项目时的个性化配置，其中会包括编辑器的颜色、代码上传时自动压缩等等一系列选项。
其他配置项细节可以参考文档
[开发者工具的配置](https://developers.weixin.qq.com/miniprogram/dev/devtools/edit.html#%E9%A1%B9%E7%9B%AE%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6)

##### 页面配置 page,json
> 这里的 page.json 其实用来表示 pages/logs 目录下的log.json这类和小程序页面 相关的配置。如果你整个小程序的风格是蓝色调，那么你可以在 app.json 里边声明顶部颜色是蓝色即可。实际情况可能不是这样，可能你小程序里边的每个页面都有不一样的色调来区分不同功能模块，因此我们提供了page.json，让开发者可以独立定义每个页面的一些属性，例如刚刚说的顶部颜色、是否允许下拉刷新等等。其他配置项细节可以参考文档 
[小程序的配置 page.json](https://developers.weixin.qq.com/miniprogram/dev/framework/config.html)

#### WXML模板

从事过网页编程的人知道，网页编程采用的是 HTML + CSS + JS 这样的组合，其中 HTML 是用来描述当前这个页面的结构，CSS 用来描述页面的样子，JS 通常是用来处理这个页面和用户的交互。

同样道理，在小程序中也有同样的角色，其中 WXML 充当的就是类似 HTML 的角色。打开 pages/index/index.wxml，你会看到以下的内容:

```
<view class="container">
  <view class="userinfo">
    <button wx:if="{{!hasUserInfo && canIUse}}"> 获取头像昵称 </button>
    <block wx:else>
      <image src="{{userInfo.avatarUrl}}" background-size="cover"></image>
      <text class="userinfo-nickname">{{userInfo.nickName}}</text>
    </block>
  </view>
  <view class="usermotto">
    <text class="user-motto">{{motto}}</text>
  </view>
</view>
```
和 HTML 非常相似，有标签、属性等等构成。但是也有很多不一样的地方，我们来一一阐述一下：
1. 标签名字有点不一样 往往写 HTML 的时候，经常会用到的标签是 div, p, span，开发者在写一个页面的时候可以根据这些基础的标签组合出不一样的组件，例如日历、弹窗等等。换个思路，既然大家都需要这些组件，为什么我们不能把这些常用的组件包装起来，大大提高我们的开发效率。 从上边的例子可以看到，小程序的 WXML 用的标签是 view, button, text 等等，这些标签就是小程序给开发者包装好的基本能力，我们还提供了地图、视频、音频等等组件能力 更多详细的组件讲述参考
[小程序的能力](https://developers.weixin.qq.com/miniprogram/dev/quickstart/basic/framework.html)
2. 多了一些 wx:if 这样的属性以及 {{ }} 这样的表达式 在网页的一般开发流程中，我们通常会通过 JS 操作 DOM (对应 HTML 的描述产生的树)，以引起界面的一些变化响应用户的行为。例如，用户点击某个按钮的时候，JS 会记录一些状态到 JS 变量里边，同时通过 DOM API 操控 DOM 的属性或者行为，进而引起界面一些变化。当项目越来越大的时候，你的代码会充斥着非常多的界面交互逻辑和程序的各种状态变量，显然这不是一个很好的开发模式，因此就有了 MVVM 的开发模式(例如 React, Vue)，提倡把渲染和逻辑分离。简单来说就是不要再让 JS 直接操控 DOM，JS只需要管理状态即可，然后再通过一种模板语法来描述状态和界面结构的关系即可。 小程序的框架也是用到了这个思路，如果你需要把一个 Hello World 的字符串显示在界面上。 WXML 是这么写 :

```
<text></text>
```
JS只需要管理状态即可：

```
this.setData({ msg: "Hello World" })
```
通过 {{ }} 的语法把一个变量绑定到界面上，我们称为数据绑定。仅仅通过数据绑定还不够完整的描述状态和界面的关系，还需要 if/else, for等控制能力，在小程序里边，这些控制能力都用 wx: 开头的属性来表达。 更详细的文档可以参考
[WXML](https://developers.weixin.qq.com/miniprogram/dev/framework/view/wxml/)

#### WXSS样式

WXSS 具有 CSS 大部分的特性，小程序在 WXSS 也做了一些扩充和修改。
1. 新增了尺寸单位。在写 CSS 样式时，开发者需要考虑到手机设备的屏幕会有不同的宽度和设备像素比，采用一些技巧来换算一些像素单位。WXSS 在底层支持新的尺寸单位 rpx ，开发者可以免去换算的烦恼，只要交给小程序底层来换算即可，由于换算采用的浮点数运算，所以运算结果会和预期结果有一点点偏差。
2. 提供了全局的样式和局部样式。和前边 app.json, page.json 的概念相同，你可以写一个 app.wxss 作为全局样式，会作用于当前小程序的所有页面，局部页面样式 page.wxss 仅对当前页面生效。
3. 此外 WXSS 仅支持部分 CSS 选择器

更详细的文档可以参考
[WXSS](https://developers.weixin.qq.com/miniprogram/dev/framework/view/wxss.html)

#### JS交互逻辑

一个服务仅仅只有界面展示是不够的，还需要和用户做交互：响应用户的点击、获取用户的位置等等。在小程序里边，我们就通过编写 JS 脚本文件来处理用户的操作。

```
<view>{{ msg }}</view>
<button bindtap="clickMe">点击我</button>
```
点击 button 按钮的时候，我们希望把界面上 msg 显示成 "Hello World"，于是我们在 button 上声明一个属性: bindtap ，在 JS 文件里边声明了 clickMe 方法来响应这次点击操作：

```
Page({
  clickMe: function() {
    this.setData({ msg: "Hello World" })
  }
})
```
响应用户的操作就是这么简单，更详细的事件可以参考文档
[WXML-事件](https://developers.weixin.qq.com/miniprogram/dev/framework/view/wxml/event.html)

此外你还可以在 JS 中调用小程序提供的丰富的 API，利用这些 API 可以很方便的调起微信提供的能力，例如获取用户信息、本地存储、微信支付等。在前边的 QuickStart 例子中，在 pages/index/index.js 就调用了 wx.getUserInfo 获取微信用户的头像和昵称，最后通过 setData 把获取到的信息显示到界面上。更多 API 可以参考文档
[小程序的API](https://developers.weixin.qq.com/miniprogram/dev/api/) 

## 结语
到这里，我们已经大概的了解了微信小程序的4种不同类型的文件大概的含义以及用途。至于如何去使用他们，需要在真正开发阶段通过实践你才能慢慢的掌握。我在这里提供一些微信小程序开发的教程网站，大家可以通过这些网站更加快速的上手微信小程序开发：
1. [W3Cschool微信小程序开发文档](https://www.w3cschool.cn/weixinapp/9wou1q8j.html)
2. [微信小程序解决方案专辑](http://www.wxapp-union.com/special/solution.html)




