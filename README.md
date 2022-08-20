# 2022年夏季《移动软件开发》实验报告




| 姓名和学号？         |                   |
| -------------------- | -------------------------------- |
| 本实验属于哪门课程？ | 中国海洋大学22夏《移动软件开发》 |
| 实验名称？           | 实验3：视频播放小程序          |
| 博客地址？           | https://www.cnblogs.com/amonologue/p/16607145.html                          |
| Github仓库地址？     | https://github.com/Acolasiasss/EX3                          |

（备注：将实验报告发布在博客、代码公开至 github 是 **加分项**，不是必须做的）



## **一、实验目标**

1、学习使用快速启动模板创建小程序的方法；2、学习不使用模板手动创建小程序的方法。



## 二、实验步骤

列出实验的关键步骤、代码解析、截图。
1.视图设计：
根据步骤，首先自定义导航栏标题和颜色，然后进行页面设计。
相关代码如下：
app.json
```
{
    "pages": [
        "pages/index/index"
    ],
    "window": {
        "backgroundTextStyle": "light",
        "navigationBarBackgroundColor": "#987938",
        "navigationBarTitleText": "口述校史",
        "navigationBarTextStyle": "white"
    },
    "style": "v2",
    "sitemapLocation": "sitemap.json"
}
```
其中，页面上包含3个区域，分别为：
1）区域1:视频播放器，用于播放指定的视频;
		使用`<video>`组件来实现一个视频播放器
2）区域2:弹幕发送区域,包含文本输入框和发送按钮;
		使用`<view>`组件实现一个单行区域,包括文本输入框和发送按钮
3）区域3:视频列表,垂直排列多个视频标题,点击不同的标题播放对应的视频内容
		使用`<view>`组件实现一个可扩展的多行区域,每行包含一个播放图标和一个视频标题文本。当前先设计第一行效果,后续使用wx:for属性循环添加全部内容
页面设计相关代码如下：
index.wxml
```
<!--index.wxml-->
<view class="container">
  <!--区域1:视频播放器-->
  <video id = 'myVideo' controls ></video>

  <!--区域2:弹幕控制-->
  <view class = 'danmuArea'>
    <input type = 'text' placeholder = '    请输入弹幕内容'></input>
    <button>发送弹幕</button >
  </view >

  <!--区域3:视频列表-->
  <view class = 'videoList'>
    <view class = 'videoBar'>
      <image src = '/images/play.png'></image>
      <text>这是一个测试标题</text>
    </view >
  </view >
</view>
```
index.wxss
```
/**index.wxss**/

video {
  width: 100%;        /*视频组件宽度为100%*/
  }
  
/*区域2:弹幕控制样式*//* 2-1弹幕区域样式*/
.danmuArea {
  display: flex;        /*flex模型布局*/
  flex-direction: row;  /*水平方向排列*/
}
/*2-2文本输入框样式*/
input {
  border: 1rpx solid #987938; /*1rpx 宽的实线棕色边框*/
  flex-grow: l;                 /*扩张多余空间宽度*/
  height: 100rpx;               /*高度*/
}
/*2-3按钮样式*/
button {
  color: white;               /*字体颜色*/
  background-color :#f3a109;  /*背景颜色*/
}
    
/*区域3:视频列表样式*/  
/*3-1视颊列表区域样式*/
.vidcoList {
  width: 100%;              /*宽度*/
  min-height: 400rpx;       /*最小高度*/
  }
/*3一2单行列表区域样式*/
.videoBar {
  width: 100%;               /*宽度*/
  display: flex;            /*flex模型布局*/
  flex-direction: row;      /*水平方向布局*/
  border-bottom: 1rpx solid #987938;/*1rpx宽的实线棕色边框*/
  margin: 10rpx;            /*外边距*/
}
/* 3一3播放图标样式*/
image {
  width: 70rpx;         /*宽度*/
  height: 70rpx;        /*高度*/
  margin: 20rpx;        /*外边距*/
}

/*3一4文本标题样式*/
text {
  font-size: 45rpx;     /*字体大小*/
  color: #987938;      /*字体颜色为棕色*/
  margin: 20rpx;        /*外边距*/
  flex-grow: 1;         /*扩张多余空间宽度*/
}
```
效果如下

![22)G6@CVP ~5PH9SV_1O_F2](https://user-images.githubusercontent.com/111416724/185726858-5cd40a5d-f0b9-462f-837f-2c89a5c98f6c.png)


2.逻辑实现
首先在区域3对`<view class = 'videoBar'>`组件添加wx : lor属性,改写为循环展示列表，然后在区域3对`<view class =  'videoBar '>`组件添加data-url属性和 bindtap属性。其中 data-url 用于记录每行视频对应的播放地址, bindtap用于触发点击事件。最后在区域1对`< video >`组件添加enable-danmu 和 danmu-btn属性,用于允许发送弹幕和显示“发送弹幕”按钮，以及对颜色进行设置。
逻辑实现相关代码如下：
index.wxml
```
<!--index.wxml-->
<view class="container">
  <!--区域1:视频播放器-->
  <video id = 'myVideo' controls enable-danmu danmu-btn></video>

  <!--区域2:弹幕控制-->
  <view class = 'danmuArea'>
    <input type = 'text' placeholder = '    请输入弹幕内容' bindinput = 'getDanmu'></input>
    <button bindtap = 'sendDanmu'>发送弹幕</button >
  </view >

  <!--区域3:视频列表-->
  <view class="vidcoList">
    <view class = "videoBar" wx:for = '{{list}}' wx:key = 'video{{index}}' data-url = '{{item.videoUrl}}' bindtap="playVideo">
      <image src="/images/play.png"></image>
      <text>{{item.title}}</text>
    </view>
  </view>
</view>
```
index.js
```
// index.js
//生成随机颜色
function getRandomColor() {
  let rgb = []
  for (let i = 0; i < 3; ++i){
    let color = Math.floor (Math.random() * 256).toString(16)
    color = color.length == 1 ? '0' + color:color
    rgb.push(color)
  }
  return '#' + rgb.join('')
}

Page({
  /**
  *页面的初始数据
  */
  data: {
    danmuTxt: '',
    list: [
      {
        id: '1001',
        title: '杨国宜先生口述校史实录',
        videoUrl: 'http://arch.ahnu.edu.cn/__local/6/CB/D1/C2DF3FC847F4CE2ABB67034C595_025F0082_ABD7AE2.mp4?e=.mp4'
      },
      {
        id: '1002',
        title: '唐成伦先生口述校史实录',
        videoUrl: 'http://arch.ahnu.edu.cn/__local/E/31/EB/2F368A265E6C842BB6A63EE5F97_425ABEDD_7167F22.mp4?e=.mp4'
      },
      {
        id: '1003',
        title: '倪光明先生口述校史实录',
        videoUrl: 'http://arch.ahnu.edu.cn/__local/9/DC/3B/35687573BA2145023FDAEBAFE67_AAD8D222_925F3FF.mp4?e=.mp4'
      },
      {
        id: '1004',
        title: '吴仪兴先生口述校史实录',
        videoUrl: 'http://arch.ahnu.edu.cn/__local/5/DA/BD/7A27865731CF2B096E90B522005_A29CB142_6525BCF.mp4?e=.mp4'
      },
    ]
  },

  /**
  *更新弹幕内容
  */
  getDanmu: function(e) {
    this.setData({
      danmuTxt: e.detail.value
    })
  },
  /**
  *发送弹幕
  */
  sendDannu: function(e) {
    let text = this.data.danmuTxt;
    this.videoCtx.sendDanmu({
      text: text,
      color: getRandomColor()
    })
  },

  /**
  *生命周期函数――监听页面加载
  */
  onLoad: function(options) {
    this.videoCtx = wx.createVideoContext( 'myVideo')
  },

  /**
  *播放视频
  */
  playVideo: function(e) {
    //停止之前正在播放的视颊
    this.videoCtx.stop()
    //更新视频地址
    this.setData({
        src: e.currentTarget.dataset.url
    })
    //播放新的视频
    this.videoCtx.play()
  },
})
```
最后效果图如下

![@CV6QEO(DOC(GRUJ_AVVOJ3](https://user-images.githubusercontent.com/111416724/185726872-012bfa07-fea7-4bcf-9d8e-f1d12fd00dc6.png)



## 三、程序运行结果

列出程序的最终运行结果及截图。
在手机上进行预览：
1）可正常切换播放视频

![AO9$V58B`PLL(U($LPW EEE](https://user-images.githubusercontent.com/111416724/185726879-c778dd04-f948-4011-9afc-656cadf5cc90.png)

![H2)UGAIJW _189$D6B}@HK](https://user-images.githubusercontent.com/111416724/185726885-b898da5a-9cf4-4d69-a74b-52019e1190b6.png)

2）也可正常发送弹幕

![PI@I) )TW WONO_VF$0Q2(9](https://user-images.githubusercontent.com/111416724/185726886-b53e4ca6-b48f-447b-89b9-acf2254fc22a.png)


## 四、问题总结与体会

描述实验过程中所遇到的问题，以及是如何解决的。有哪些收获和体会，对于课程的安排有哪些建议。

本次实验只要跟着教程走，难度不是很大，只是有些比较细微的地方需要注意。
我在做此次实验的过程中就在一些不容易被察觉的地方犯了错误：列如在设置“点击拨放视频”功能是，wxml文件中需要用两个变量分别记录视频播放地址和触发点击事件，我就是在写变量是没有注意大小写，导致我后面一直播放不成功，需要十分注意。
总督来说，这次实验我的收获非常大。
