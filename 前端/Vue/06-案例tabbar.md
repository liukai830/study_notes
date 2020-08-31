## 零、TabBar案例效果

![image-20200405234313438](https://gitee.com/liukai830/picgo/raw/master/image-20200405234313438.png)

## 一、TabBar实现思路

【1】上述有一个单独的TabBar组件，如何封装

- 自定义TabBar组件，在App中使用
- 让TabBar处于底部，并设置相关的样式

【2】TabBar内容由外界决定

- 定义插槽
- flex布局平分TabBar

【3】自定义TabBarItem，可以传入图片、文字、跳转的URL

- 定义TabBarItem，并且定义两个插槽：图片、文字；设置props属性，存放跳转的URL
- 给两个插槽外层包装div，用于设置样式
- 填充插槽，实现底部TabBar的效果

