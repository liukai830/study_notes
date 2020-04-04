## 一、认识路由

​	单页面应用下，我们需要修改页面的url，但是不希望页面重新加载，有以下几种方式可以实现：

### 1.1、URL的hash

- 也就是锚点(#)，本质上是改变windows.location的href属性；
- 我们可以通过直接赋值location.hash来改变href，但是页面不发生刷新。

```javascript
【1】项目启动后在浏览器控制台查看当前地址
>	location.href
<."http://localhost:8080/#/"
【2】通过修改location的hash为'/'
> location.hash = '/'
<. "/"
【3】再次查看当前地址，没有变，因为'/'就是根路径
> location.href
<."http://localhost:8080/#/"

【4】重新修改hash为'/aaa'，发现页面变了，但是没有新的网络请求
> location.hash = '/aaa'
<."/aaa"
【5】再次查看地址，已经变为修改后的地址了
> location.href
<. "http://localhost:8080/#/aaa"
```



### 1.2 、pushState

- HTML5的history模式：pushState
- 使用方式：`history.pushState(data, title, ?url)`
  - 主要是最后一个参数url需要传

具体实例如下：

【1】首先当前页面地址为 http://localhost:8080/#/aaa

【2】在控制台输入`history.pushState({},'','/home')`

【3】观察到地址栏变成了 http://localhost:8080/home

【4】继续依次修改地址为 'about'、'me'

【5】此时历史记录顺序为 'aaa'、'about'、'me'

【6】可以通过`history.back()`返回历史，即'about'页面

注：这个操作和栈操作类似。



### 1.3、replaceState

【1】和pushState操作相似，`history.replaceState({},'','/home')`

【2】但是，不支持返回；这不是入栈操作



### 1.4、go

- go(1)   ==  history.forward()
- go(-1)  ==  history.back()



## 二、基本使用

### 2.1、安装和使用vue-router

【1】安装vue-router：`npm install vue-router --save`

【2】在模块化工程中使用它

​	【2.1】导入路由对象，并调用`Vue.use(VueRouter)`

​	【2.2】创建路由实例，并且传入路由映射配置

​	【2.3】在Vue实例中挂载创建的路由实例

第二步具体实现如下：

（1）创建router目录，在该目录中创建index.js来管理路由信息

```javascript
// 配置路由相关的信息
import Vue from 'vue'
import Router from 'vue-router'
// 1. 通过Vue.use(插件)，安装插件
Vue.use(Router)
// 2. 创建VueRouter对象
// 配置映射关系
const routes = []
const router = new Router({
	// 配置路由和组件之间的应用关系
	routes
})
// 3. 将router对象传入到Vue实例，在main.js
// 先到出，然后在main.js中导入，挂载
export default router
```

（2）在main.js的Vue实例中挂载路由

```javascript
import Vue from 'vue'
import App from './App'
import router from './router'
new Vue({
  el: '#app',
  router,
  render: h => h(App)
})
```



### 2.2、配置路由映射

【1】创建路由组件

【2】配置路由映射：组件和路径映射关系

【3】使用路由：通过<router-link></router-link>和<router-view/>

​	【3.1】<router-link to="/home">最终会被渲染成<a></a>标签

​	【3.2】<router-view/>是一个占位标签，对应的路由会在这个标签位置上显示

具体实现如下：

（1）创建路由组件：`Home.vue`和`About.vue`

```vue
<template>
	<div><h2>Home Page</h2></div>
</template>
<script>
	export default {
		name: 'Home'
	}
</script>
<style></style>
```

（2）在index.js中配置路由映射

```javascript
import Home from '../components/Home'
import About from '../components/About'
const routes = [
	{
		path: '/home',
		component: Home
	},
	{
		path: '/about',
		component: About
	},
]
```

（3）在App.vue中使用路由

```vue
<template>
  <div id="app">
		<router-link to="/home">首页</router-link>
		<router-link to="/about">关于</router-link>
		<router-view/>
  </div>
</template>

<script>
export default {
  name: 'App'
}
</script>

<style></style>
```



### 2.3、路由默认值和history模式

（1）默认值

```javascript
const routes = [
  // 默认显示Home，这种方式地址栏url还是localhost:8080/#/
	{
		path: '',
		component: Home
	}
  // 推荐使用重定向
  {
		path: '',
		redirect: '/home'
	}
]
```

（2）history模式

​	去除路径中的#即：

- localhost:8080/#/			 ->	localhost:8080/
- localhost:8080/#/home  ->     localhost:8080/home

在index.js进行修改：

```javascript
const router = new Router({
	routes,
	mode: 'history'	// 这里默认是hash模式
})
```



### 2.4、router-link属性

1. to：对应的路由地址

2. tag：渲染的样式，默认是<a>标签，tag="button"会渲染成按钮

3. replace：不会留下历史记录，即浏览器不能退回

4. active-class：当路由匹配成功时，会自动给当前元素设置一个router-link-active的class，设置active-class可以修改默认的名称
   - 在进行高亮显示的导航栏菜单，或者底部tabbar时，会用到该类
   - 但是通常不会修改类的属性，会直接使用router-link-active即可

```html
<router-link to="/home" tag="li" replace active-class="active">首页</router-link>
```



## 三、路由跳转

​	上面用了<router-link to="">方式进行路由跳转，还可以通过自定义事件，在代码中进行跳转

- `this.$router.push('/home')`
- `this.$router.replace('/home')`

```vue
<template>
  <div id="app">
<!-- 		<router-link to="/home">首页</router-link>
		<router-link to="/about">关于</router-link> -->
		<button @click="homeBtnClick">首页</button>
		<button @click="aboutBtnClick">关于</button>
		<router-view/>
  </div>
</template>

<script>
export default {
  name: 'App',
	methods: {
		homeBtnClick() {
			// this.$router.push('/home')
			this.$router.replace('/home')
		},
		aboutBtnClick() {
			// this.$router.push('/about')
			this.$router.replace('/about')
		}
	}
}
</script>

<style></style>

```



## 四、动态路由

（1）定义用户组件`User.vue`

```vue
<template>
	<div><h2>User Page</h2></div>
</template>
<script>
	export default {
		name: 'User',
	}
</script>
<style></style>
```

（2）在`index.js`配置动态路由

```javascript
const routes = [
	{
    // :userid就是动态传递的参数
		path: '/user/:userid',
		component: User
	},
]
```

（3）在`App.vue`中进行测试

​	定义了三个用户，通过<router-link>和代码形式实现动态路由

```vue
<template>
  <div id="app">
    <!-- 这里的to是需要数据动态绑定到user的，所以需要用:to -->
		<router-link v-for="user in users" :to="'/user/'+user"> 用户{{user}} </router-link>
		<button v-for="user in users" @click="userBtnClick(user)">用户：{{user}}</button>
		<router-view/>
  </div>
</template>

<script>
export default {
  name: 'App',
	data() {
		return {
			users: ['zhangsan', 'lisi', 'liuk']
		}
	},
	methods: {
		userBtnClick(userid) {
			this.$router.push('/user/' + userid)
      // this.$router.replace('/user/' + userid)
		}
	}
}
</script>

<style>
</style>
```

（4）在User页面获取动态参数

```vue
<template>
	<div>
		<h2>User Page</h2>
		<p>用户信息：{{userid}}</p>
	</div>
</template>

<script>
	export default {
		name: 'User',
		computed: {
			userid() {
        // 这里是route不是router
        // userid属性就是index.js动态路由中定义的：'/user/:userid'，需要匹配
				return this.$route.params.userid
			}
		}
	}
</script>

<style>
</style>
```



## 五、路由懒加载





## 六、嵌套路由





## 七、参数传递





## 八、router和route





## 九、导航守卫





## 十、keep-alive





