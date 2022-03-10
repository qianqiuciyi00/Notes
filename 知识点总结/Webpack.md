# 常用loader
- style-loader将CSS添加到DOM的内联样式标签style里
- css-loader允许将css文件通过require的方式引入，并返回css代码
- less-loader
- sass-loader
- postcss-loader用postcss来处理css
- file-loader分发文件到uotput目录并返回相对路径
- url-loader 和file-loader类似，但是当文件小于设定的limit时可以返回一个data url
- html-minify-loader压缩html
- babel-loader 将ES6转换为ES5
- vue-loader 加载vue文件

# 常用Plugin
- HtmlWebpackPlugin：打包结束后，自动生成一个html文件，并把打包生成的js模块引入到该html中
- clean-webpack-plugin：清理（删除）构建目录
- mini-css-extract-plugin：提取css到一个单独的文件中
- DefinePlugin：允许在编译时创建配置的全局对象，是一个webpack内置的插件，不需要安装
- copy-webpack-plugin：复制文件或目录到执行区域，如vue的打包过程中，如果我们将一些文件放到public的目录下，那么这个目录会被复制到dist文件夹中

# plugin和loader的区别
loader是解析规则，因为webpack默认只能解析js，所以需要在loader里面配置一些规则  
plugin是插件，是用来扩展webpack的功能，比如压缩代码、提取公共代码

# 输出文件名的hash、chunkhash和contenthash有什么区别
- hash模式：只要修改一个文件，整个项目的文件命名都会改变，不能进行缓存
- chunkhash：根据入口文件命名
- contenthash：根据内容生成命名，进行缓存