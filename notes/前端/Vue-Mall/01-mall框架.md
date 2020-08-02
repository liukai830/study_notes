## 一、划分目录结构

```css

```



## 二、CSS文件的引入



## 三、vue.config和editorconfig

### 3.1、配置相关文件夹别名

创建`vue.config.js`配置别名

```javascript
module.exports = {
  configureWebpack: {
    resolve: {
      alias: {
        'components': '@/components',
        'content': 'components/content',
        'common': 'components/common',
        'assets': '@/assets',
        'network': '@/network',
        'views': '@/views',
      }
    }
  }
}
```

### 3.2、editorconfig

```javascript
root = true

[*]
charset = utf-8
indent_style = space
indent_size = 2
end_of_line = lf
insert_final_newline = true
trim_trailing_whitespace = true
```



## 四、TabBar引入和项目模块划分