---
title: 生命周期
date: 2026-01-07
tags: [#微信小程序]
---

# 生命周期

对于小程序生命周期来说 页面生命周期可以同时使用 组件生命周期和页面生命周期

组件生命生命周期：
created
attached
ready
deached

页面生命周期：
onLanch
onLoad
onReady
onHide
onShow
onUnload

组件生命周期往往比页面生命周期先执行
onLanch create attached onLoad onShow ready onready onunload detached

onLoad 是小程序页面生命周期中的一个重要函数，当页面初次加载时会触发该函数。它通常用于进行页面的初始化操作，比如获取页面参数、请求数据、设置页面状态等。

在小程序中，每个页面都有自己的 JavaScript 文件，其中可以定义若干生命周期函数，onLoad 就是其中之一。该函数会在页面创建并加载时被调用，且只调用一次。它接收一个参数，通常是 options 或 query，表示从其他页面跳转过来时传递过来的参数（即 URL 中的查询字符串参数）。

onLoad是无法访问DOM元素的，但是onLoad开始就可以进行setData了， onReady和ready 后就可以访问dom元素了

 1. App 应用生命周期

  App({
    onLaunch (options) {
      // 小程序初始化完成时触发，全局只触发一次
      // 适合：获取用户信息、登录、初始化全局数据
    },
    onShow (options) {
      // 小程序启动，或从后台进入前台显示时触发
      // 适合：刷新数据、恢复状态
    },
    onHide () {
      // 小程序从前台进入后台时触发
      // 适合：暂停定时器、保存状态
    },
    onError (msg) {
      // 小程序发生脚本错误或 API 调用报错时触发
      console.log(msg)
    },
    globalData: 'I am global data'
  })

  2. Page 页面生命周期

  Page({
    onLoad: function(options) {
      // 页面创建时执行，只会调用一次
      // 适合：接收页面参数、初始化页面数据
    },
    onShow: function() {
      // 页面出现在前台时执行
      // 适合：每次显示页面时刷新数据
    },
    onReady: function() {
      // 页面首次渲染完毕时执行，只会调用一次
      // 适合：获取节点信息、设置导航栏
    },
    onHide: function() {
      // 页面从前台变为后台时执行
      // 适合：暂停定时器、保存状态
    },
    onUnload: function() {
      // 页面销毁时执行
      // 适合：清理资源、取消网络请求
    },
    // 其他页面事件
    onPullDownRefresh: function() {
      // 下拉刷新
    },
    onReachBottom: function() {
      // 上拉触底
    },
    onShareAppMessage: function () {
      // 用户分享
    },
    onPageScroll: function() {
      // 页面滚动
    },
    onResize: function() {
      // 页面尺寸变化
    },
    onTabItemTap(item) {
      // 当前是 tab 页时，点击 tab 时触发
    }
  })

  3. Component 组件生命周期

  组件自身生命周期

  Component({
    lifetimes: {
      created() {
        // 组件实例刚刚被创建，但还未挂载到 DOM
        // 此时不能调用 setData
        // 适合：初始化内部数据
      },
      attached() {
        // 组件实例进入页面节点树时执行
        // 可以使用 setData
        // 适合：初始化组件状态、设置事件监听
      },
      ready() {
        // 组件布局完成，可以进行 DOM 操作
        // 适合：获取节点信息、操作 DOM
      },
      moved() {
        // 组件实例被移动到节点树另一个位置时执行
      },
      detached() {
        // 组件实例被从页面节点树移除时执行
        // 适合：清理资源、移除监听器
      }
    }
  })

  组件所在页面的生命周期

  Component({
    pageLifetimes: {
      show: function() {
        // 组件所在页面被展示时执行
        // 适合：页面显示时刷新组件数据
      },
      hide: function() {
        // 组件所在页面被隐藏时执行
        // 适合：暂停组件内的动画、定时器
      },
      resize: function(size) {
        // 组件所在页面尺寸变化时执行
        // 适合：响应式布局调整
      }
    }
  })

  4. 生命周期执行顺序

  应用启动时：
  App.onLaunch → App.onShow → Page.onLoad → Page.onShow → Page.onReady

  页面切换时：
  PageA.onHide → PageB.onLoad → PageB.onShow → PageB.onReady

  页面返回时：
  PageB.onUnload → PageA.onShow

  组件生命周期顺序：
  created → attached → ready → [moved (如果移动位置)] → detached



