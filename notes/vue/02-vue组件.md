## 一、组件化的基本使用

- 组件的使用分成三个步骤：
  - 创建组件构造器  ->  调用Vue.extend()方法<font color="red">创建组件构造器</font >
  - 注册组件              ->  调用Vue.compent()方法<font color="red">注册组件</font>
  - 使用组件              ->  在Vue实例的作用范围内<font color="red">使用组件</font>

具体实现代码如下（这种写法在Vue2.x的文档中几乎看不到了，会用下面讲到的语法糖实现）：

```html
<html>
	<body>
		<div id="app">
			<my-cpn></my-cpn>
			<my-cpn></my-cpn>
		</div>
	</body>
	<script src="../js/vue.js"></script>
	<script>
		// 1.创建组件构造器对象
		const cpnC = Vue.extend({
			template: `
				<div>
					<h2>我是组件标题</h2>
					<p>我是内容001</p>
					<p>我是内容002</p>
				</div>
			`
		})
		// 2.注册组件（全局组件）
		Vue.component('my-cpn',cpnC)
		const app = new Vue({
			el: '#app'
		})
	</script>
</html>
```



## 二、全局组件和局部组件

- 全局组件：可以在多个Vue的实例下面使用

- 局部组件：在Vue实例下注册，就是这个Vue实例的局部组件，只能在这个Vue实例下使用

```html
<body>
		<div id="app">
			<my-cpn></my-cpn>
			<my-cpn></my-cpn>
			<cpn-liuk></cpn-liuk>
		</div>
</body>
<script>
  const cpnC = Vue.extend({
  	template: `<div><h2>我是组件标题</h2><p>我是内容001</p></div>`
	})
  // 这样子注册的是全局组件
	//Vue.component('my-cpn',cpnC)

	const app = new Vue({
		el: '#app',
  	components: {
    	// 局部组件，只能在app的Vue实例中使用
   	 cpnLiuk: cpnC, // key就是组件的标签名，value就是上面定义的的组件
     cpn-liuk: cpnC // 不能用-分隔符，要用驼峰形式，然后HTML在用-分隔符
  	}
	})
</script>
```

**<font color="red">注：Vue实例下局部组件中不支持”-“，只能用驼峰；使用时在html标签中再通过”-“转换，因为HTML标签不区分大小写</font>**



## 三、父组件和子组件的区分

父子组件实现代码如下：

```html
<html>
	<body>
		<div id="app">
			<cpn-parent></cpn-parent>
		</div>
	</body>
	<script src="../js/vue.js"></script>
	<script>
		const cpnChild = Vue.extend({
			template: `<div>我是子组件</div>`
		})
		
		const cpnParent = Vue.extend({
			template: `<div><div>我是父组件</div><cpn-child></cpn-child></div>`,
			components: {
				cpnChild: cpnChild
			}
		})
		
		const app = new Vue({
			el: '#app',
			components: {
				cpnParent: cpnParent
			}
		})
	</script>
</html>
```



## 四、组件注册语法糖与模板抽离

全局组件和局部组件的注册语法糖实现如下：

```html
<script>
	  // 语法糖注册全局组件
		Vue.component('my-cpn',{
			template: `<div><h2>我是全局组件标题</h2></div>`
		})
		
		const app = new Vue({
			el: '#app',
			components: {
				// 局部组件注册的语法糖
				cpnLiuk: { template: `<div><h2>我是局部组件标题</h2></div>` }
			}
		})
</script>
```



上述代码代码层次很多，看起来比较乱，所以我们需要把template进行抽离，使得代码看起来更清晰。实现代码如下：

```html
<html>
	<body>
		<div id="app">
			<my-cpn></my-cpn>
			<my-cpn></my-cpn>
			<cpn-liuk></cpn-liuk>
		</div>
	</body>
  
	<template id="templatecpn">
		<div><h2>我是组件模板抽离标题</h2></div>
	</template>
  
	<script src="../js/vue.js"></script>
	<script>
		Vue.component('my-cpn',{
			template: '#templatecpn'	// 模板分离，通过模板id获取
		})
		const app = new Vue({
			el: '#app',
			components: {
				cpnLiuk: { template: '#templatecpn' }
			}
		})
	</script>
</html>
```



## 五、为什么组件data必须是函数



## 六、父子组件通信





## 七、插槽