新建Toast 组件
Toast.vue
```
<template>
  <div v-show="showToast" class="popup-container">
    <div class="head">
    <img v-if="icon == 'success'" class="img" src="@/assets/images/success.png">
    <img v-else class="img" src="@/assets/images/error.png">
    <div class="title">{{ title }}</div>
    </div>
    <div class="txt">{{ content }}</div>
  </div>
</template>
```
index.js
```
import vue from 'vue'
import toastComponent from './Toast.vue'

// 返回一个 扩展实例构造器
const ToastConstructor = vue.extend(toastComponent)

// 定义弹出组件的函数 接收4个参数, 要显示的标题、内容状态和显示时间
function showToast() {
  const { title, content, icon = 'success', duration = 3000, showClose = false } = arguments[0]
  // 实例化一个 toast.vue
  const toastDom = new ToastConstructor({
    el: document.createElement('div'),
    data() {
      return {
        title,
        content,
        icon,
        showToast: true,
        showClose
      }
    }
  })

  // 把 实例化的 toast.vue 添加到 body 里
  document.body.appendChild(toastDom.$el)

  setTimeout(() => { toastDom.showToast = false }, duration)
}
// 注册为全局组件的函数
function registryToast() {
  vue.prototype.$toast = showToast
}

export default registryToast

```
main.js
```
import toastRegistry from './components/Toast/index'
Vue.use(toastRegistry)
```

调用的页面
```
this.$toast({
    title: '分配成功',
    content: '已成功将XX台设备分配给XXX！',
    showClose: true
})
```