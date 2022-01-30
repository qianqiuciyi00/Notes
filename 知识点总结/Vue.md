# 优缺点
- 优点：渐进式，组件化，轻量级，虚拟DOM，响应式，单页面路由，数据与视图分开
- 缺点：单页面不利于SEO，不支持IE8以下，首屏加载时间长

# 渐进式的理解
想用啥就用啥，不用也不强求。例如component，Vuex等

# Vue跟React的异同点
相同点：
  - 都使用了虚拟dom
  - 组件化开发
  - 都是单向数据流（父子组件之间，不建议子修改父传下来的数据）
  - 都支持服务端渲染
不同点：
  - React的JSX，Vue的template
  - 数据变化，React手动（setState），Vue自动（响应式）
  - React单向绑定，Vue双向绑定
  - React的Redux，Vue的Vuex

# MVVM和MVC
- MVC：Controller将Model的数据展示在View上
  - Model(模型)：负责从数据库中取数据
  - View(视图)：负责展示数据的地方
  - Controller(控制器)：用户交互的地方，例如点击事件等等
- MVVM：实现了View和Model的自动同步
  - VM：View-Model，双向绑定，将模型转化成视图，视图转化成模型

# 修饰符
- .lazy：改变输入框的值时value不会改变，当光标离开输入框时，v-model绑定的值才会改变
- .trim
- .number
- .stop：阻止冒泡
- .capture：事件默认从里往外冒泡，capture修饰符的作用相反，由外往内捕获
- .self：只有点击事件绑定的本身才会触发事件
- .once：事件只触发一次
- .prevent：阻止默认事件
- .navtive：加在自定义组件的事件上，保证事件能执行
- .left，.right，.middle：鼠标的左中右按键触发的事件
- .passive：当我们在监听元素滚动事件的时候，会一直触发onscroll事件，在pc端是没问题的，但是在移动端，我们的网页会变卡，这个修饰符相当于给整个onscroll事件加了一个.lazy修饰符
- .camel：确保绑定参数被识别为驼峰写法
- .sync：父子组件传值，子组件想更新这个值，使用此修饰符可简写

# Vue内部指令
- v-text
- v-html
- v-show
- v-if
- v-else
- v-else-if
- v-for
- v-on
- v-bind
- v-model
- v-slot
- v-once：元素和组件只渲染一次
- v-pre：跳过这个元素和它的子元素的编译过程，可以用来显示原始Mustache标签，跳过大量没有指令的节点会加快编译
- v-cloak：这个指令保持在元素上直到关联实例编译结束

# 组件之间传值方式
- 父传子：props接收
- 子传父：$emit
- 组件可以使用`$parent`和`$children`获取到父组件和子组件实例，从而获取数据
- 使用`$refs`获取组件实例，从而获取数据
- Vuex状态管理
- `eventBus`进行跨组件触发事件，进而传递数据
- 本地浏览器缓存，`localStorage`等

# v-if 和 v-for
Vue2中，v-for的优先级高于v-if，Vue3中相反

# vuex
- State：定义状态的数据结构，可以在这里设置默认的初始状态
- Getter：允许组件从Store中获取数据，mapGetters辅助函数可以将store中的getter映射到局部计算属性
```js
import { mapGetters } from 'vuex'
computed: mapGetters(['count', 'user'])
```
- Mutation：唯一更改store中状态的方法，且必须是同步函数
- Action：用于提交mutation，而不是直接变更状态，可以包含异步操作
- Module： 允许将单一的Store拆分为多个Store且同时保存在单一的状态树中
- 原理：利用了全局混入`Mixin`，将store对象，混入到每一个Vue实例中

# 不需要响应式的数据应该怎么处理
一些不需要改变的数据，就不要做响应式处理了，不然会消耗性能
```js
// 方法一：将数据定义在data之外
data() {
    this.list1 = {...}
    this.list2 = {...}
    this.list3 = {...}
    return {}
}
// 方法二：Object.freeze()
data() {
    return {
        list1: Object.freeze({...})
    }
}
```

# watch
监听引用数据类型时
```js
watch: {
    obj: {
        handler() {...}, //执行回调
        deep: true, //是否进行深度监听
        immediate: true // 是否初始执行handler函数
    }
}
```

# 父子组件生命周期顺序
父beforeCreate -> 父created -> 父beforeMount -> 子beforeCreate -> 子created -> 子beforeMount -> 子mounted -> 父mounted

# 对象更新或删除属性无法更新视图
Vue.$set(obj, key), Vue.$delete(obj, key)

# 相同的路由组件如何重新渲染
```js
<template>
    <router-view :key="$route.path"></router-view>
</template>
```

# data中某一数据的初始状态
```js
this.$options.data().xxx
```

# Vue-router原理
- hash模式  
  hash指的是url描点，当描点发生变化的时候，浏览器只会修改访问历史记录，不会访问服务器重新获取页面。因此可以监听描点值的变化，根据描点值渲染指定dom
  实现原理：
  - 改变描点：通过`locaton.hash = '/hashpath'`的方式修改浏览器的hash值
  - 监听描点变化：
    ```js
    window.addEventListener('hashchange', () => {
      const hash = window.location.hash.substr(1)
    })
    ```
- history模式  
  实现原理：
  - 改变url：H5的history对象提供了pushState和replaceState两个方法，当调用这两个方法时，url发生变化，浏览器访问历史也会发生变化，但是浏览器不会向后台发送请求
  ```
  history.pushState({}, "", '/a')
  // 第一个参数：data对象，在监听变化的事件中能够获取到
  // 第二个参数：title标题
  // 第三个参数：跳转地址
  ```
  - 监听url变化：可以通过监听popstate事件监听history变化，也就是点击浏览器的前进或者后退功能时触发
  ```js
  window.addEventListener('popstate', () => {
    const path = window.location.pathname
  })
  ```