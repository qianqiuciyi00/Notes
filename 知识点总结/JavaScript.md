# DOMContentLoaded
- 当初始的HTML文档被完全加载和解析完成之后，DOMContentLoaded事件被触发，而无需等待样式表、图像和子框架的完全加载。如果文档中包含脚本，则脚本会阻塞文档的解析，而脚本需要等待CSSDOM构建完成才能执行。在任何情况下，DOMContentLoaded的触发不需要等待图片等其他资源加载完成。
- 使用
  ```js
    document.addEventListener('DOMContentLoaded', function(event) {
        ...
    })
  ```

# 异步脚本
- async 或defer
  当HTML文档被解析时如果遇见同步脚本，则停止解析，先去加载脚本，然后执行，执行结束后继续解析HTML文档。
- defer脚本：
  当HTML文档被解析时如果遇见defer脚本，则在后台加载脚本，文档解析过程不中断，而等文档解析结束之后，defer脚本执行。另外，defer脚本的执行顺序与定义时的位置有关系。
- async脚本：
  当HTML文档被解析时如果遇见async脚本，则在后台加载脚本，文档解析过程不中断。脚本加载完成后，文档停止解析，脚本执行，执行结束之后文档继续解析。

# 异步脚本与DOMContentLoaded
- defer：DOMContentLoaded 只有在defer脚本执行结束之后才会被触发
- async: HTML文档构建不受影响，解析完毕后，DOMContentLoaded触发，而不需要等待adync脚本执行、样式表加载等等。

# MutationObserver
- MutationObserver是一个内建对象，它观察DOM元素，并在检测到更改时触发回调