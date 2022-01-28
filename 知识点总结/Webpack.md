# 常见的loader
- style-loader将CSS添加到DOM的内联样式标签style里
- css-loader允许将css文件通过require的方式引入，并返回css代码
- less-loader
- sass-loader
- postcss-loader用postcss来处理css
- file-loader分发文件到uotput目录并返回相对路径
- url-loader 和file-loader类似，但是当文件小于设定的limit时可以返回一个data url
- html-minify-loader压缩html
- babel-loader 将ES6转换为ES5