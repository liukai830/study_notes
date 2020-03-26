

### 一、插值操作

#### 1、v-once

​	组件和元素只会渲染一次。


```html
<div id="app" v-once> {{msg}}</div>
```

#### 2、v-html

​	把数据按照HTML格式进行解析，并且显示对应的内容

```html
<body>
	<div id="app">
		<h2 v-html="url"></h2>
	</div>
</body>
<script>
const app = new Vue({
	el: '#app',
	data: {
		url: '<a href="http://www.baidu.com">百度一下</a>'
	}
});
</script>
```

#### 3、v-text

​	效果和mustache语法相似：都是用于将数据显示在界面中。
```html
<div id="app" v-text="msg"></div>
<div id="app">{{msg}}</div>
```

#### 4、v-pre

​	不解析{{}}，如{{message}}，渲染后就会直接显示"{{message}}"
```html
<div id="app" v-pre> {{msg}}</div>
```

#### 5、v-cloak

​	在vue解析之前，div中有v-cloak，先不会显示；解析完成后，会删除v-cloak，然后显示对应信息

```html
<div id="app" v-cloak> {{msg}}</div>
```



### 二、绑定操作

