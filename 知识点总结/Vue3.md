# Vue3的6个优势
- 使用TS重构了项目，获得更好的类型支持
- 重构了响应式系统：
  Vue3使用`Proxy`替换`Object.defineProperty`，重构了Vue的响应式系统；解决了Vue2中存在的响应性问题，包括：
  - 可直接监听数组类型的数据变化
  - 坚挺的目标为对象本身，不需要像`Object.defineProperty`一样遍历每个属性，有一定的性能提升
  - 更强大的拦截功能，拦截器包括get,set,has,deleteProperty,OwnKeys,getOwnPropertyDescriptor,defineProperty,preventExtensions,getPropertyOf,isExtensible,setPropertypeOf,apply,construct 13种
- 引入了Composition API
  专门用于解决功能、数据和业务逻辑分散的问题，使项目更易于模块化开发以及后期维护。核心是setup函数
- 优化了Virtual DOM
  包括：
  - 模板编译时的优化，将一些静态节点编译成常量
  - slot优化，将slot编译为lazy函数，将slot的渲染决定权交给自组件
  - 模板中内联事件的提取并重用（原本每次渲染都声称内联函数）
- 更好的Tree shaking
  把无用的模块进行枝剪，很多没用到的API就不会被打包到最后的包里。实际上是webpack或rollup的功能。主要原理：依赖es6的模块化的语法，将无用的代码（dead-code）进行删除
- <script setup>
