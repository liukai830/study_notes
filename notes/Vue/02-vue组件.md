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
	<head>
		<title></title>
	</head>
	<body>
		<div id="app">
			<my-cpn></my-cpn>
			<my-cpn></my-cpn>
			<cpn-liuk></cpn-liuk>
			<cpn-test></cpn-test>
		</div>
	</body>
	<template id="templatecpn">
		<div><h2>我是组件模板抽离标题</h2></div>
	</template>
	<script src="../js/vue.js"></script>
	<script>
		Vue.component('my-cpn',{
			template: '#templatecpn'
		})
		const app = new Vue({
			el: '#app',
			components: {
				cpnLiuk: { template: '#templatecpn' },
				'cpn-test': { template: '#templatecpn' }
			}
		})
	</script>
</html>
```



## 五、为什么组件data必须是函数

- 组件是一个单独功能模块的封装
  - 这个模块有属于自己的HTML模板，也应该有<font color="red">属于自己的数据data</font>
  - 这个<font color="red">data属性必须是一个函数</font>
  - 而且这个函数返回一个<font color="red">对象</font>，对象内部保存着数据

具体代码如下：

```html
<html>
	<body>
		<div id="app">
			<my-cpn></my-cpn>
      <my-cpn></my-cpn>
		</div>
	</body>
	<template id="cpn">
		<div>
      <h2>当前计数：{{counter}}</h2>
      <button @click="increment">+</button>
      <button @click="decrement">-</button>
    </div>
	</template>
	<script src="../js/vue.js"></script>
	<script>
		Vue.component('my-cpn',{
			template: '#cpn',
      // 这里是个函数，返回对象 
			data() {
				return {
					counter: 0
				}
			},
      methods: {
        increment() {
          this.counter++;
        },
        decrement() {
          this.counter--;
        }
      }
		})
		const app = new Vue({
			el: '#app'
		})
	</script>
</html>
```

- 为什么要用函数返回对象？

  有多个组件时，数据应该是不共享的。所以每次需要return不同的对象，开辟不同的堆内存。



## 六、组件通信

- 父传子：props
- 子传父：$emit Events

### 6.1、父传子

#### （1）如何使用

【1】 定义：在子组件中通过props定义数据

【2】绑定：在组件标签<my-cpn>中通过v-bind将props中的数据绑定到父组件的data中

【3】使用：在子组件模板里，用props中定义的名称即可获取到父组件的数据

具体实现代码如下：

```html
<html>
	<body>
		<div id="app">
			<!-- props属性绑定到父组件的data -->
			<my-cpn :cpnusers="users" :cpnmessage="message"></my-cpn>
		</div>
	</body>
	<template id="cpn">
		<div>
			<h2>{{cpnmessage}}</h2>
			<ul>
				<li v-for="user in cpnusers">{{user}}</li>
			</ul>
    </div>
	</template>
	<script src="../js/vue.js"></script>
	<script>
		Vue.component('my-cpn',{
			template: '#cpn',
			// 1. 数组方式
			// props:['cpnusers','cpnmessage']
			// 2. 对象方式
			props: {
				cpnusers: Array,
				cpnmessage: String
			}
		})
		const app = new Vue({
			el: '#app',
			data: {
				message: 'all Users',
				users: ['admin','cyinfo','saas_admin','user_rub']
			}
		})
	</script>
</html>
```

#### （2）props数据验证

​	在需要对props进行类型等验证时，需要用对象写法，支持String、Number、Boolean、Array、Object、Date、Function、Symbol类型验证，或者自定义构造函数验证。

​	验证实现代码如下：

```javascript
Vue.component('my-cpn', {
  template: '#cpn',
  // 1. 类型限制
  propsA: {
    cpnmessage: String,
		cpnusers: Array,
    // 多个类型
		cpnmessage: [String, Number],
	}
	// 2. 默认值和必填
	propsB: {
		cpnmessage: {
			type: String,
			default: 'asdd',
			required: true
		}
	}
	// 3. 默认值和必填（对象和数组）
	propsC: {
		cpnmessage: {
			type: String,
				default() {
					return ['1','2','3']
				},
			required: true
		}
	}
	// 4. 自定义验证函数
	propsD: {
		validator(value) {
			// 必须匹配下列字符串中的一个
			return ['success','error','warning'].indexOf(value) !== -1
		}
	}
	// 5.自定义类型
	function Person(firstName,lastName) {
 		this.firstName = firstName
  	this.lastName = lastName
	}
	propsE: {
		user: Person
	}
})
```

#### （3）props驼峰标识

​	v-bind不支持驼峰，所以需要通过“-”转换，即：cpnUsers -> v-bind:cpn-users

```html
<html>
	<body>
		<div id="app">
			<!-- props属性绑定到父组件的data -->
			<my-cpn :cpn-message="message"></my-cpn>
		</div>
	</body>
	<template id="cpn">
		<div>
			<h2>{{cpnMessage}}</h2>
    </div>
	</template>
	<script>
		Vue.component('my-cpn',{
			template: '#cpn',
			props: {
				cpnMessage:String
			}
		})
		const app = new Vue({
			el: '#app',
			data: {
				message: 'this is message'
			}
		})
	</script>
</html>
```



### 6.2、子传父





## 七、插槽













