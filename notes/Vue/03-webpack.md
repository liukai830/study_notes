## 一、webpack安装

- 首先安装node.js，node.js自带了包管理软件npm
- 全局安装webpack `npm install webpack@3.6.0 -g`
- 局部安装`npm install webpack@3.6.0 --save=dev`



## 二、webpack的基本使用过程

- 项目的目录结构如下：

  dist

  ​    |— bundle.js

  src

  ​    |— main.js

  ​	|—mathUtil.js

  index.html

​     **index.html是文件的主入口，src是开发时文件，dist是打包后代码编译的文件**

- mathUtils.js代码如下：

```javascript
function add(num1,num2) {
  return num1 + num2;
};
function sub(num1,num2) {
  return num1 - num2;
};
export {add,sub}
```

- main.js代码如下

```javascript
import {add,sub} from "./mathUtils.js" 
console.log(add(10,20)); 
console.log(sub(1,2));
```

在index.html引用main.js（引用的是最终编译后的文件）

<script src="dist/bundle.js" type="text/javascript" charset="utf-8"></script>

最近打包到dist中，由于main.js中引用了mathUtils.js，打包的时候会把mathUtils.js也打包进去

`webpack ./src/main.js ./dist/bundle.js`



## 三、webpack.config.js和package.json

### 3.1 webpack.config.js

​	手动创建`webpack.config.js`文件，与`index.html`同级

```javascript
const path = require('path')	// 为了动态获取路径
module.exports = { 
  // 文件的入口 
  entry: './src/main.js', 
  output: { 
    // 打包生成文件路径 
    path: path.resolve(__dirname, 'dist'), 
  	// 打包进的文件名 
    filename: 'bundle.js' 
  } 
}
```

### 3.2 package.json

​	通过`npm init`生成`package.json`文件

```java
{
  "name": "meetwebpack",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "build": "webpack"
  },
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "webpack": "^3.6.0"
  }
}
```

### 3.3 打包

- 两个文件创建完成之后，就可以通过`webpack`指令进行打包了
- 或者在`package.json`文件中增加scripts命令"build": "webpack"，这样就可以通过`npm run build`打包



## 四、webpack-css文件的配置

- 安装css-loader和style-loader：

  `npm install --save-dev css-loader` （解析css文件后，使用import加载，并返回css代码）

  `npm install --save-dev style-loader`（将模块的到出作为样式添加到DOM中）

  也就是说css-loader负责加载，style-loader负责渲染

- `webpack.config.js`增加配置

  ```javascript
  module: {
    rules: [
      {
        test: /\.css$/ ,
  			// npm解析是从右往左
  			use: ['style-loader', 'css-loader']
  		}
    ]
  }
  ```

  

## 五、webpack-less文件的配置

- 安装less-loader：

  `npm install --save-dev less-loader less`

- webpack.config.js中module->rules增加配置：

  ```javascript
  {
    test: /\.less$/ ,
    use: [{
     loader: 'style-loader'
    },{
     loader: 'css-loader'
    },{
     loader: 'less-loader'
    }]
  }
  ```



## 六、webpack-图片文件

- 安装url-loader：

  `npm install --save-dev url-loader`

- webpack.config.js中module->rules增加配置：

  ```javascript
  {
      test: /\.(png|jpg|gif)$/,
      use: [
      {
          loader: 'url-loader',
          options: {
              // 当加载的图片,小于limit时,会将图片转为base64加载
              // 大于limit时,需要使用file-loader(不需要其他配置),当成文件处理
              limit: 8192, // 默认为8kb
              // 统一图片的名字
              name: 'img/[name].[hash:8].[ext]'
          }
      }]
  }
  ```

- 如果图片大小超过limit时，需要用file-loader加载，不需要其他配置

  `npm install --save-dev file-loader`



## 七、webpack-ES6转ES5

- 安装babel-loader

  `npm install -save-dev babel-loader@7 babel-core babel-preset-es2015`

- webpack.config.js中module->rules增加配置：

  ```javascript
  {
      test: /\.js$/,
      exclude: /(node_modules|bower_components)/,
      use: {
          loader: 'babel-loader',
          options: {
              presets: ['es2015']
          }
      }
  }
  ```



## 八、webpack-vue配置



## 九、Vue的终极使用方案



## 十、webpack-plugin的使用