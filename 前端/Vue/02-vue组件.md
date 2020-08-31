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

- 字传父通过**自定义事件**来完成
  - 在子组件中通过this.$emit('event_name', param_1, param_2...)来触发事件
  - 在父组件中通过v-on: event_name(param_1, param_2)="cpnClick"来监听事件
  - 最后在父组件的Vue实例methods中的cpnClick()接受参数

具体实现代码如下：

```html
<html>
	<body>
		<div id="app">
      <!-- 3. 在父组件里通过emit定义的事件名，监听该事件-->
			<cpn :childcatagories="catagories" @itemclick="cpnItemClick"></cpn>
		</div>
	</body>
	<script src="../js/vue.js"></script>
	<template id="categorycpn">
		<div>
      <!-- 1.子组件监听click事件-->
			<button v-for="item in childcatagories" 
              @click="itemClick(item)">
        {{item.name}}
      </button>
		</div>
	</template>

	<script>
		const cpn = {
			template: '#categorycpn',
			props: {
				childcatagories: Array
			},
			methods: {
				itemClick(item) {
          // 2. 在子组件的click事件中，通过$emit向父组件发射事件，并且可以带参数
					this.$emit('itemclick',item)
				}
			}
		}
		
		const app = new Vue({
			el: '#app',
			data: {
				catagories: [
					{id: 1, name: '热销产品'},
					{id: 2, name: '手机数码'},
					{id: 3, name: '家用电器'}
				]
			},
			components: {
				cpn
			},
			methods: {
        // 4. 最后在父组件的methods中可以处理事件和接受参数
				cpnItemClick(item) {
					console.log(item)
				}
			}
		})
	</script>
</html>
```



### 6.3、父子组件双向绑定案例

需求：

（1）父组件中的值在子组件中显示

（2）子组件中input改变这个值

（3）父组件的值也要改变

注意事项：

- 在子组件通过props用number获取父组件的num值，但是不能用v-modal绑定这个number;
- 要在子组件用data()或者计算属性用一个新的值dnumber去接收number，再实现v-modal绑定;
- 至此，实现的是子组件与子组件中的dnumber绑定，还需要绑定到父组件data中的num
- 通过子传父#emit即可，这里是用了watch监听子组件值的改变
- 完整路径如下：num->number->dnumber:->input->双向绑定dnumber-> (watch,$emit)num->number

实现代码如下：

```html
<html>
	<body>
		<div id="app">
      <h2>父组件num{{num}}</h2>
      <!-- 4. 在父组件中监听子组件值的改变 -->
			<cpn :number="num" @numberchange="numchange"></cpn>
		</div>
	</body>
	<template id='childtemplate'>
		<div>
			<h2>父组件传过来的值：{{number}}</h2>
			<h2>子组件重新绑定的值：{{dnumber}}</h2>
      <!-- 1. 原则上不能直接修改父组件的值，所以不能直接绑定number-->
			<input type="number" v-model.number="dnumber">
		</div>
	</template>
	<script src="../js/vue.js"></script>
	<script>
		const cpn = {
			template: '#childtemplate',
			props: {
				number: Number
			},
      // 2. 需要通过data或者计算属性引用父组件的值number，然后绑定新的值dnumber
			data() {
				return {
					dnumber: this.number
				}
			},
      // 3. 通过watch监听值的改变，用$emit向父组件返回改变后的值
			watch:{
				dnumber(newValue,oldValue) {
					this.$emit('numberchange',newValue);
				}
			}
		}
		const app = new Vue({
			el: '#app',
			data: {
				num: 1
			},
			components:{
				cpn
			},
			methods:{
        // 5. 重新改变父组件data值
				numchange(newValue) {
					this.num = newValue
				}
			}
		})
	</script>
</html>
```

### 6.4、父子组件的访问方式

- 有时候我们需要父组件直接访问子组件、子组件直接访问父组件、子组件直接访问根组件
  - 父组件访问子组件：使用$children或$refs
    - $children是一个数组类型，包含了所有子组件对象，使用较少
    - $refs获取较多
  - 子组件访问父组件：使用$parent
  - 访问根组件：$root

具体实现代码如下：

```html
<html>
	<body>
		<div id="app">
			<h2>父组件</h2>
			<button @click="showchildmsg">点击调用子组件showMsg</button>
			<cpn ref="aaa"></cpn>
			<cpn ref="bbb"></cpn>
		</div>
	</body>
	<template id="cpn">
		<div>
			<h2>这是子组件</h2>
			<button type="button" @click="showMsg">子组件按钮，点击打印</button>
		</div>
	</template>
	<script src="../js/vue.js"></script>
	<script>
		const cpn = {
			template: '#cpn',
			methods: {
				showMsg() {
					console.log("子组件...")
					// 访问父组件
					console.log(this.$parent)
          // 访问根组件
          console.log(this.$root)
				}
			}
		}
		const app = new Vue({
			el: '#app',
			components: {
				cpn
			},
			methods: {
				showchildmsg() {
					// 1. $children，实际开发中使用较少
					// this.$children[0].showMsg()
					// this.$children.forEach(child=>console.log(child.$attrs.id))
					// 2. $refs 在组件上加ref="aaa"属性，就可以通过this.$refs.aaa获取子组件
					console.log(this.$refs.aaa)
				}
			}
		})
	</script>
</html>
```



## 七、插槽

### 7.1 基本使用

- 插槽是为了让组件开发更具有扩展性，为父组件预留的空间
  （1）插槽的基本使用`<slot></slot>`

  （2）插槽设置默认值`<slot><p>这是插槽默认值</p></slot>`

  （3）同时有多个标签时，会同时把这些标签全部汰替换到插槽中


```html
<html>
	<body>
		<div id="app">
			<cpn><button>d</button></cpn>
			<cpn><input type="text"></cpn>
			<cpn></cpn>
			<!--同时有多个标签时，会同时把这些标签全部汰替换到插槽中-->
			<cpn>
				<h2>qqqqq</h2>
				<h3>22222</h3>
				<button>aaq</button>
			</cpn>
		</div>
	</body>
	<template id="cpn">
		<div>
			<h2>这是子组件</h2>
			<!-- 插槽，预留的空间 -->
			<!-- <slot></slot> -->
			<!-- 插槽设置默认值-->
			<slot><p>这是插槽默认值</p></slot>
			<hr/>
		</div>
	</template>
	<script src="../js/vue.js"></script>
	<script>
		const cpn = {
			template: '#cpn',
		}
		const app = new Vue({
			el: '#app',
			components: {
				cpn
			}
		})
	</script>
</html>
```



### 7.2 具名插槽

​	作用：用于区分多个插槽

- 在slot标签上，用name进行区分每个插槽

- 使用时候，增加标签属性slot="name"即可替换对应的插槽

具体代码如下：

```html
<html>
	<body>
		<div id="app">
      <!-- 2. 使用slot="xxx"替换对应的插槽 -->
			<cpn><span slot="left">www</span></cpn>
			<cpn><span slot="left">左左左</span><button slot="right">右右右</button></cpn>
		</div>
	</body>
	<template id="cpn">
		<div>
			<h2>这是子组件</h2>
      <!-- 1. 定义三个插槽，name不同 -->
			<slot name="left">左边</slot>
			<slot name="middle">中间</slot>
			<slot name="right">右边</slot>
			<hr/>
		</div>
	</template>
	<script src="../js/vue.js"></script>
	<script>
		const cpn = {
			template: '#cpn',
		}
		const app = new Vue({
			el: '#app',
			components: {
				cpn
			}
		})
	</script>
</html>
```



### 7.3 作用域插槽

- 目的：父组件替换插槽的标签，但是内容由子组件来提供。



- 假设有个需求
  - 子组件中包含一组数据，比如：languages: ['JAVA', 'Python', 'C#', 'Go', 'C++']
  - 需要在多个界面展示：
    - 某些界面是水平方向一一展示的；
    - 某些界面是以列表的方式展示的；
    - 某些界面是直接展示一个数组；
  - 内容在子组件，希望父组件告诉子组件如何展示就可以了
    
    - 利用**slot作用域插槽**就可以了
    




-  实现代码如下：

```html
<html>
	<body>
		<div id="app">
			<cpn></cpn>
			<cpn>
				<!-- 2. 要定义一个template模板，并且属性v-slot="xxx" -->
				<template v-slot="slot">
					<!-- 3. 这样就能用xxx.data 获取到子组件插槽的数据了-->
					<span v-for="item in slot.data"> {{item}}、 </span>
				</template>
			</cpn>
			<cpn></cpn>
		</div>
	</body>
	<template id="cpntemplate">
		<div>
			<!-- 1. 在slot使用:data="languages"获取数据，这里的:data可以随便取值 -->
			<slot :data="languages">
				<ul>
					<li v-for="item in languages">{{item}}</li>
				</ul>
			</slot>
		</div>
	</template>
	<script src="../js/vue.js"></script>
	<script>
		const cpn = {
			template: '#cpntemplate',
			data() {
				return {
					languages: ['JAVA', 'Python', 'C#', 'Go', 'C++']
				}
			}
		}
		const app = new Vue({
			el: '#app',
			components: {
				cpn
			}
		})
	</script>
</html>
```



- 效果如图所示：

  ![image-20200401221508129](https://gitee.com/liukai830/picgo/raw/master/image-20200401221508129.png)



## 八、动态组件

```html
<html>
	<body>
		<div id="app">
			<components :is='type'></components>
		</div>
	</body>
	<script src="../js/vue.js"></script>
	<script>
		const cpn1 = {
			template: `<div><h2>组件1</h2></div>`
		}
		const cpn2 = {
			template: `<div><h2>组件2</h2></div>`
		}
		const app = new Vue({
			el: '#app',
			components: {
				cpn1, cpn2
			},
			data: {
				type: 'cpn1'
			}
		})
	</script>
</html>
```

