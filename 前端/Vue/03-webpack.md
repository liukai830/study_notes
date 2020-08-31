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

- 全局安装vue

    `npm install vue --save`   

  - runtime-only：代码中不能有任何的template
  - runtime-compiler：可以有template，因为compiler可以编译template

  解决方案：在`webpack.config.js`中增加配置

  ```javascript
  resolve: {
      alias: {
          // 指定项目使用vue的版本
          'vue$': 'vue/dist/vue.esm.js'
      }
  }
  ```

- 在main.js中引入vue

    `import Vue from 'vue'`   

- 然后就能在main.js中写vue代码了

  - 原则上我们不在index.html中写代码，只保留<div id="app"></div>
  - template中 会替换上面这个标签
  
  ```javascript
  import Vue from "vue"
  
  new Vue({
  	el: '#app',
  	template: `
  	<div>
  	  <span>{{name}}</span>
  	  <button @click="btnClick">点我</button>
  	  <span>{{msg}}</span>
  	</div>
  	`,
  	data: {
  		msg: 'hello,liuk!',
  		name: 'liuk'
  	},
  	methods: {
  		btnClick() {
  			this.msg += ' click ';
  		}
  	}
  });
  ```



## 九、Vue的终极使用方案

-   安装vue-loader：

    `npm install vue-loader vue-templete-compiler --save-dev`   

- 配置  `webpack.config.js`   

  ```javascript
  {
      test: /\.vue$/,
      loader: 'vue-loader'
  }
  ```

    **重新build后发现报错**   

    <font color="red">`vue-loader was used without the corrsponding plguin. Makr sure to include VueLoaderPlguin in your webpack config`</font>   

    **解决方案：**   

  ​	（1）安装插件

  ​	（2）修改 `package.json`中vue-loader版本为14以下，并重新`npm install`   

- 定义app.vue文件

  ```html
  <template>
  	<div>
  	  <span class="title">{{name}}</span>
  	  <button @click="btnClick">点我</button>
  	  <span>{{msg}}</span>
  	  <cpn></cpn>
  	  <cpn></cpn>
  	</div>
  </template>
  
  <script>
  	import cpn from "./cpn.vue"
  	export default {
  		name: 'App',
  		components: {
  			cpn
  		},
  		data(){
  			return {
  				msg: 'hello,liuk',
  				name: 'liuk'
  			}
  		},
  		methods: {
  			btnClick() {
  				this.msg += ' click ';
  			}
  		}
  	}
  </script>
  
  <style>
  	.title {
  		color: green;
  	}
  </style>
  ```

- 在main.js引用此文件即可

  ```javascript
  import App from "./vue/app.vue"
  
  new Vue({
  	el: '#app',
  	template: '<App/>',
  	components: {
  		App
  	}
  });
  ```



## 十、webpack-plugin的使用

​		下面的配置全部在`webpack.config.js` 文件中修改。

### 10.1、添加版权

  ```javascript
  const webpack = require('webpack');
  
  plugins: [
      new webpack.BannerPlugin('最终版权归liuk所有')
  ]
  ```

  最后会在打包后的bundle.js第一行增加注释： `/*！ 最终版权归liuk所有*/`



### 10.2、打包html

- HtmlWebpackPlugin插件可以为我们做这些事情：

  - 自动生成一个index.html文件（可以指定模板来生成）
  - 将打包的js文件，自动通过script标签插入到body中

- 安装HtmlWebpackPlugin插件：

  `npm install html-webpack-plugin --save-dev`

- 修改`webpack.config.js` 中plugins部分内容

  - 这里的template表示根据什么模板来生成index.html
  - 另外需要删除之前在output中添加到public属性，否则插入的script标签的src可能会有问题

  ```javascript
  const HtmlWebpackPlugin = require('html-webpack-plugin');
  const uglifyJsPlugin = require('uglifyjs-webpack-plugin')
  module.exports = {
    ...
    plugins: [
    new HtmlWebpackPlugin({
    		template: 'index.html'
  		})
    ]
  }
  ```
  
  

###  10.3、js代码压缩

`npm install uglifyjs-webpack-plugin@1.1.1 --save-dev`   

```javascript
const uglifyJsPlugin = require('uglifyjs-webpack-plugin')
module.exports = {
  ...
  plugins: [
    new uglifyJsPlugin();
  ]
}
```



### 10.4、搭建本地服务器

作用：代码热更新。

【1】下载插件

    `npm install webpack-dev-server@2.9.1 --save-dev`   

【2】增加配置：

  ```javascript
  devServer: {
      contentBase: './dist', //监听的目录
      inline: true,   //热更新
      port: 8099  //端口，默认8080
  }
  ```

【3】启动项目即可，启动方式：

  （1）  `.\node_modules\.bin\webpack-dev-server`   

  （2）  在script中增加脚本 `--open`表示自动打开浏览器   

  ```javascript
  "dev": "webpack-dev-server --open"
  ```

  

## 十一、webpack-配置文件的分离

目的：区分开发时依赖和运行时依赖。

【1】创建build文件夹

【2】文件夹下创建`base.config.js`	->	 通用配置

【3】文件夹下创建`dev.config.js`	  ->	 开发时配置

【4】文件夹下创建`prod.config.js`    ->     运行时配置（如增加代码压缩plugin...）

【5】根据不同场景，需要合并`base+dev`或者`base+prod`，

​			所以需要安装另一个插件`npm install webpack-merge`

【6】在`dev`和`prod`配置文件中，通过merge进行导出

- 原来的dev配置：

```javascript
const uglifyJsPlugin = require('uglifyjs-webpack-plugin')
module.exports = {
  ...
  plugins: [
    new uglifyJsPlugin();
  ]
}
```

- 通过merge导出配置：

```javascript
const uglifyJsPlugin = require('uglifyjs-webpack-plugin')
const webpackMerge = require('webpack-merge')
const baseConfig = require('./base.config.js')

module.exports = webpackMerge(
	baseConfig,
  {
    ...
  	plugins: [
    	new uglifyJsPlugin();
  	]
  }
)
```

【7】修改`package.json`中的script指令

```json
"scripts": {
  "build": "webpack --config ./build/prod.config.js",
  "dev": "webpack-dev-server --open --config ./builf/dev.config.js"
}
```

【8】最后使用`npm run build`或者`npm run dev`即可区分正式环境和开发环境

