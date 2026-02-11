# 微信小程序路由跳转

## 一、路由跳转方法

### 1. wx.navigateTo - 打开新页面

**功能描述：**
- 保留当前页面，将新页面推入页面栈
- 目标必须是非 tabBar 页面
- 用户可以通过返回按钮返回

**当前页面生命周期：**
- 触发 `onHide`（页面隐藏，但不销毁）

**使用示例：**
```javascript
wx.navigateTo({
  url: '/pages/detail/detail?id=123',
  success: function(res) {
    console.log('跳转成功');
  },
  fail: function(err) {
    console.error('跳转失败', err);
  }
});
```

---

### 2. wx.redirectTo - 重定向

**功能描述：**
- 关闭当前页面，跳转到新页面
- 替换当前页面栈
- 用户无法返回到当前页面

**当前页面生命周期：**
- 触发 `onUnload`（页面销毁）

**使用示例：**
```javascript
wx.redirectTo({
  url: '/pages/login/login',
  success: function() {
    console.log('重定向成功');
  }
});
```

---

### 3. wx.switchTab - 切换 Tab

**功能描述：**
- 跳转到 tabBar 页面
- 关闭所有非 tabBar 页面
- 目标必须是 app.json 中配置的 tabBar 页面

**页面生命周期影响：**
- 页面栈中其他非 tabBar 页面触发 `onUnload`
- 其他 tabBar 页面触发 `onHide`
- 目标 tabBar 页面触发 `onShow` 或 `onLoad`（首次创建时）

**使用示例：**
```javascript
wx.switchTab({
  url: '/pages/index/index'
});
```

---

### 4. wx.reLaunch - 重启

**功能描述：**
- 关闭所有页面，打开新页面
- 清空整个页面栈
- 通常用于登录后或重新初始化

**所有页面生命周期：**
- 所有页面都触发 `onUnload`（全部销毁）

**使用示例：**
```javascript
wx.reLaunch({
  url: '/pages/home/home',
  success: () => {
    console.log('重启成功');
  }
});
```

---

### 5. wx.navigateBack - 返回

**功能描述：**
- 返回上一页面
- 可以通过 delta 参数控制返回层数

**当前页面生命周期：**
- 触发 `onUnload`（页面销毁）

**使用示例：**
```javascript
// 返回上一页
wx.navigateBack();

// 返回上两页
wx.navigateBack({
  delta: 2
});
```

---

## 二、页面生命周期对比

| 生命周期 | 触发时机 | 典型场景 |
|---------|---------|---------|
| `onLoad` | 页面创建时 | 首次打开页面 |
| `onShow` | 页面显示/前台 | 页面显示、从后台切换到前台 |
| `onReady` | 页面首次渲染完成 | 页面渲染完毕 |
| `onHide` | 页面隐藏/后台 | navigateTo 跳转、切换到其他小程序 |
| `onUnload` | 页面销毁 | redirectTo、navigateBack、reLaunch |

---

## 三、跳转方法与页面生命周期关系

### ✅ 触发页面 onUnload（销毁）的方法

| 方法 | 销毁页面 | 说明 |
|------|---------|------|
| `wx.redirectTo` | 当前页面 | 替换当前页面 |
| `wx.navigateBack` | 当前页面 | 返回并销毁 |
| `wx.switchTab` | 非 tabBar 页面 | 关闭所有非 tabBar 页面 |
| `wx.reLaunch` | 所有页面 | 清空页面栈 |

### ❌ 不触发页面销毁的方法

| 方法 | 当前页面状态 | 说明 |
|------|------------|------|
| `wx.navigateTo` | `onHide` | 页面隐藏但保留在栈中 |

---

## 四、页面栈管理

### 页面栈规则

1. **页面栈限制：** 最多 10 层
2. **navigateTo 行为：** 新页面入栈，当前页面触发 `onHide`
3. **navigateBack 行为：** 当前页面出栈并销毁，触发 `onUnload`
4. **redirectTo 行为：** 当前页面出栈，新页面入栈
5. **reLaunch 行为：** 清空栈，新页面入栈

### 避免栈溢出

```javascript
// 错误示例：连续使用 navigateTo 可能导致栈溢出
for (let i = 0; i < 15; i++) {
  wx.navigateTo({ url: `/pages/page${i}/page${i}` });
  // 当栈达到 10 层时会报错
}

// 正确做法：使用 redirectTo 替换页面
wx.redirectTo({ url: '/pages/next/next' });
```

---

## 五、使用建议

### 选择合适的跳转方法

1. **保留页面状态 → 使用 `wx.navigateTo`**
   - 适用于：详情页、子页面等需要返回的场景

2. **不需要返回 → 使用 `wx.redirectTo`**
   - 适用于：登录页、完成页等一次性页面

3. **切换主导航 → 使用 `wx.switchTab`**
   - 适用于：切换到 tabBar 页面

4. **重置应用 → 使用 `wx.reLaunch`**
   - 适用于：登录成功后跳转首页、清空历史记录

5. **返回操作 → 使用 `wx.navigateBack`**
   - 适用于：返回上一页或多级返回

### 注意事项

1. ⚠️ `navigateTo` 不能跳转到 tabBar 页面
2. ⚠️ tabBar 页面之间只能使用 `switchTab`
3. ⚠️ 注意页面栈深度限制（10层）
4. ⚠️ 在 `onUnload` 中执行清理操作（如定时器、监听器等）
5. ⚠️ 页面参数通过 URL 传递，注意长度限制

---

## 六、常见使用场景

### 场景1：登录流程
```javascript
// 登录成功后，清空历史并跳转首页
wx.reLaunch({
  url: '/pages/index/index'
});
```

### 场景2：商品详情 → 立即购买
```javascript
// 详情页保留，跳转到订单页
wx.navigateTo({
  url: '/pages/order/create'
});

// 或者替换当前页
wx.redirectTo({
  url: '/pages/order/create'
});
```

### 场景3：多级返回
```javascript
// 返回到首页（假设栈中有 5 层）
wx.navigateBack({
  delta: 5
});
```

### 场景4：页面间数据传递
```javascript
// 方式1：URL 参数
wx.navigateTo({
  url: '/pages/detail/detail?id=123&name=test'
});

// 方式2：EventChannel（2.7.3+）
wx.navigateTo({
  url: '/pages/detail/detail',
  events: {
    acceptDataFromOpenedPage: function(data) {
      console.log('接收数据', data);
    }
  },
  success: function(res) {
    // 向打开的页面传递数据
    res.eventChannel.emit('acceptDataFromOpenerPage', {
      data: '来自 opener 页面'
    });
  }
});
```

---

## 七、navigator 组件使用

除了 API 调用，也可以使用组件进行跳转：

```html
<!-- 等同于 wx.navigateTo -->
<navigator url="/pages/detail/detail?id=123">
  跳转到详情页
</navigator>

<!-- 等同于 wx.switchTab -->
<navigator url="/pages/index/index" open-type="switchTab">
  切换到首页
</navigator>

<!-- 等同于 wx.redirectTo -->
<navigator url="/pages/login/login" open-type="redirect">
  重定向到登录页
</navigator>

<!-- 等同于 wx.navigateBack -->
<navigator open-type="navigateBack" delta="1">
  返回
</navigator>
```

---

## 参考文档

- [微信小程序官方文档 - 路由](https://developers.weixin.qq.com/miniprogram/dev/framework/app-service/route.html)
- [wx.navigateTo API](https://developers.weixin.qq.com/miniprogram/dev/api/route/wx.navigateTo.html)
- [wx.redirectTo API](https://developers.weixin.qq.com/miniprogram/dev/api/route/wx.redirectTo.html)
- [wx.switchTab API](https://developers.weixin.qq.com/miniprogram/dev/api/route/wx.switchTab.html)
- [wx.reLaunch API](https://developers.weixin.qq.com/miniprogram/dev/api/route/wx.reLaunch.html)
- [wx.navigateBack API](https://developers.weixin.qq.com/miniprogram/dev/api/route/wx.navigateBack.html)
