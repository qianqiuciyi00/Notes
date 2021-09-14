# position: sticky

元素根据正常文档流进行定位，相对它的最近滚动祖先元素和containing block(最近块级祖先)，包括table-related元素，基于top, right, bottom, left的值进行偏移，偏移值不会影响其他任何元素的位置。

该值总会创建一个新的层叠上下文，一个sticky元素会固定在离他最近的一个拥有‘滚动机制’的祖先上(该祖先的overflow是hidden，scroll, auto, 或overlay)，即便这个祖先不是最近的真实可滚动祖先。