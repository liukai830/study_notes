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



### 1.2、pushState

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

- 当我们打包构建应用时，JavaScript包会变得非常大，影响页面加载
- 需要把不同路由对应的组件分割成不同的模块，当路由被访问时才加载对应的组件，这样就高效了

使用方式：

```javascript
// import Home from '../components/Home'
// import About from '../components/About'
// import User from '../components/User'

// 【2】建议用变量先保存下来，后面路由较多的时候，管理方便
const Home = () => import('../components/Home')
const About = () => import('../components/About')
const User = () => import('../components/User')

const routes = [
	{
		path: '/home',
		// component: Home,
    // 【1】方式1：直接在路由中写
		component: () => import('../components/Home')
	},
	{
		path: '/about',
		// component: About,
		component: () => import('../components/About')
	},
	{
		path: '/user/:userid',
		// component: User,
		component: () => import('../components/User')
	},
]
```



## 六、嵌套路由

现在需要在Home组件下再添加News和Messages组件，完整结构如下：

```
|— Home
|		|— News
|		|— Message
|— About
|— User
```

实现嵌套路由有两个步骤：

【1】创建对应的子组件，并在路由映射中配置对应的路由

创建子组件代码省略，配置路由代码如下：

```javascript
import Vue from 'vue'
import Router from 'vue-router'

const Home = () => import('../components/Home')
// home下的两个子组件
const HomeNews = () => import('../components/HomeNews')
const HomeMessages = () => import('../components/HomeMessages')

Vue.use(Router)

const routes = [
	{
		path: '',
		redirect: '/home'
	},
	{
		path: '/home',
		component: Home,
    // 在home路由中，增加children属性
		children: [
      // 默认路径
			{
				path: '',
				redirect: 'news'
			},
      // 子组件路径不能加/ -> path: '/news' -> 这种写法无法加载
			{
				path: 'news',
				component: HomeNews
			},
			{
				path: 'messages',
				component: HomeMessages
			},
		]
	},
]


```

【2】在组件内部使用<route-view>标签

在`Home.vue`组件中增加<route-view>标签用于显示News和Messages

```vue
<template>
	<div>
		<h2>Home Page</h2>
    <div>
      <h3>下面是子路由</h3>
      <!-- 这里的路径要写全 -->
			<router-link to="/home/news">新闻</router-link>
			<router-link to="/home/messages">消息</router-link>
			<router-view/>
  	</div>
	</div>
</template>
 
<script>
	export default {
		name: 'Home'
	}
</script>

<style>
</style>
```



## 七、参数传递

- 传递的方式主要有两种：params和query
- params的类型：
  - 配置动态路由，格式：<font color="red">/router/:id</font>
  - 传递方式：<font color="red">在path后面跟上对应的值</font>
  - 实例：<font color="red">/router/123, router/abc</font>
  - 取参：<font color="red">this.$route.params.id</font>

- query的类型：

  - 配置普通路由：<font color="red">/route</font>

  - 传递的方式：<font color="red">对象中使用query的key作为传递当时</font>

  - 实例：<font color="red">/route?id=123, /route?id=abc</font>

  - 代码演示：

    ```vue
    <template>
      <div id="app">
        <router-link :to="{path: '/profile',query: {name: 'liuk', sex: 'male'}}">
          档案
      	</router-link>
    		<button @click="profileBtnClick">档案</button>
    		<hr/>
    		<router-view/>
      </div>
    </template>
    
    <script>
    export default {
      name: 'App',
    	methods: {
    		profileBtnClick() {
    			this.$router.push({
    				path: 'profile',
    				query: { name: 'liuk', sex: 'male' }
    			})
    		}
    	}
    }
    </script>
    ```

  - 取参：<font color="red">this.$route.query.name, this.$route.query.sex</font>

- 代码跳转：如button按钮点击跳转等

  ```javascript
  // 【1】动态路由：页面刷新数据不会丢失
  this.$router.push('/user/' + userid)
  this.$router.push(`/user/{userid}`)
  // 获取参数
  this.$route.params.id
  
  // 【2】params：页面刷新数据会丢失
  // 通过路由属性中的name来确定匹配的路由，通过params来传递参数
  // 路由配置如下：
  {
    path: '/particulars',
    name: 'particulars',
    component: particulars
  }
  // 跳转
  this.$router.push({
    name: 'particulars',
    params: { id: id }
  })
  // 获取参数
  this.$route.params.id
  
  // 【3】query
  this.$router.push({
    path: '/profile',
    query: {
      name: 'aaaaa',
  		sex: 'female'
  	}
  })
  // 获取参数
  this.$route.query.id
  ```



## 八、router和route





## 九、导航守卫

### 9.1、全局守卫

作用：监听路由的变化

vue-router全局有三个守卫

- router.beforeEach：全局前置守卫，进入路由之前
  - `router.beforeEach( (to, from, next) => {} )`
- router.beforeResolve：全局解析守卫，在beforeRouteEnter调用之后调用
  - `router.beforeResolve( (to,from.next) => {} )`
- router.afterEach：全局后置钩子，进入路由之后
  - `router.afterEach( (to,form) => {} )`

- 补充说明：
  - next()函数有以下几个取值：
    - next()：进行管道中的下一个钩子，如果钩子全部都执行完了，那么导航的状态就是confirmed
    - next(false)：中断当前导航。如果URL改变了，实际上会重置到URL改变前的from导航那里。
    - next("/")或者next({path:"/"})：跳转到其他的地址。当前导航被中断，然后跳转到新导航。
    - next(error)：如果传入的是一个error实例，则导航会被终止，且该错误会被传递给router.onError()注册过的回调。
- 注意事项：
  - <font color="red">一定要确保调用next()方法。</font>



#### 9.1.1、全局前置守卫

**案例一：**

假设有需求：跳转到不同的界面title需要改变。

解决方案【1】：

​	可以在每个Vue组件生命周期函授created(){}中是同document.title='xxx'，但是组件太多，工作量大，不符合实际；

解决方案【2】：

​	使用导航守卫：

​	（1）在每个路由上创建meta元数据属性，里面定义这个路由所对应组件的title

```javascript
// 这里只定义了第一层的
const routes = [
	{
		path: '/home',
		component: Home,
		meta: {
			title: '首页'
		},
		children: [
			{
				path: 'news',
				component: HomeNews
			},
		]
	},
	{
		path: '/profile',
		meta: {
			title: '档案'
		},
		component: Profile,
	},
]
```

​	（2）vue-router全局有一个beforeEach：前置守卫，进入路由之前调用

```javascript
const router = new Router({
	// 配置路由和组件之间的应用关系
	routes,
	mode: 'history'
})

// 导航守卫
router.beforeEach((to, from, next) => {
  // 正常可以直接这样子使用
	// document.title = to.meta.title;
  // 但是嵌套路由会undefined，所以可以通过matched获取第一个
	document.title = to.matched[0].meta.title;
	console.log(to)
  // 最后一定要执行next(),否则不会进行下一步
	next();
})
```



**案例二：**

登录案例，验证是否登录。

```javascript
import Vue from 'vue'
import Router from 'vue-router'
import route from './router'

Vue.use(Router)

const router = new Router({
  routes: route // 路由列表
})

// 模拟用户登录与否
const HAS_LOGIN = true;

// 全局前置守卫
// 在Router实例上进行守卫
router.beforeEach((to, from, next) => {
  // to和from都是路由实例
  // to：即将跳转到的路由
  // from：现在的要离开的路由
  // next：函数
  // 如果未登录，就跳到登录页，如果登录了，选择哪个页面跳到哪个页面；如果登录了还去了login页面，就跳到首页。
  if (to.name !== 'login') {
    if (HAS_LOGIN) next()
    else next({ name: 'login' })
  } else {
    if (HAS_LOGIN) next({ name: 'home' })
    else next()
  }
})

// 全局解析守卫
router.beforeResolve((to,from.next) => {
})

// 全局后置钩子
router.afterEach((to,form) => {
})

export default router
```



### 9.2、路由独享的守卫

> 如果不想在全局配置路由的话，可以为某些路由单独配置守卫

比如：给home页面单独配置守卫

```javascript
{
    path: '/',
    name: "home",
    component: Home,
    // 路由独享守卫
    beforeEnter: (to, from, next) => {
      if(from.name === 'about'){
        alert("这是从about来的")
      }else{
        alert("这不是从about来的")
      }
      next();  // 必须调用来进行下一步操作。否则是不会跳转的
    }
  }
```



### 9.3、路由组件内的守卫

- beforeRouteEnter()：进入路由前
- beforeRouteUpdate()：路由复用同一个组件时
- beforeRouteLeave()：离开当前路由时

在Home.vue页面举个例子：

```vue
<script>
export default {
  name: 'Home',
  // 组件内守卫
  // 因为这个钩子调用的时候，组件实例还没有被创建出来，因此获取不到this
  beforeRouteEnter (to, from, next) {
    console.log(to.name);
    // 如果想获取到实例的话
    // next(vm=>{
    //   // 这里的vm是组件的实例（this）
    // });
    next();
  },
  // 路由即将要离开的时候调用此方法
  // 比如说，用户编辑了一个东西，但是还么有保存，这时候他要离开这个页面，就要提醒他一下，还没保存，是否要离开
  beforeRouteLeave (to, from, next) {
    const leave = confirm("确定要离开吗？");
    if(leave) next()    // 离开
    else next(false)    // 不离开
  },
}
</script>
```

对于beforeRouteUpdate的演示例子：

```vue
<script>
beforeRouteUpdate(to,from,next){
  console.log(to.name, from.name);
  next();
},
</script>
```

beforeRouteUpdate被触发的条件是：当前路由改变，但是该组件被复用的时候。

比如说：argu/fu1到argu/f2这个路由，都复用了arg.vue这个组件，这个时候beforeRouteUpdate就会被触发。可以获取到this实例



### 9.3、完整导航流程解析

```css
1、导航被触发。
2、在失活的组件（即将离开的页面组件）里调用离开守卫。 beforeRouteLeave
3、调用全局的 beforeEach 守卫。
4、在重用的组件里调用 beforeRouteUpdate 守卫 (2.2+)。
5、在路由配置里调用（路由独享的守卫） beforeEnter。
6、解析异步路由组件
7、在被激活的组件（即将进入的页面组件）里调用 beforeRouteEnter。
8、调用全局的 beforeResolve 守卫 (2.5+)。
9、导航被确认。
10、调用全局的 afterEach 钩子。所有的钩子都触发完了。
11、触发 DOM 更新。
12、用创建好的实例调用 beforeRouteEnter 守卫中传给 next 的回调函数。
```



## 十、keep-alive

### 10.1、基本使用

- keep-alive是Vue内置的一个组件，可以使被包含的组件保留状态，或避免重新渲染；
- <route-view>也是一个组件，如果直接被包在keep-alive里面，所有路径匹配到的视图组件都会被缓存。
- 当组件在keep-alive内被切换时组件的**activated、deactivated**这两个生命周期钩子函数会被执行

```vue
<template>
  <div id="app">
		<router-link to="/home">首页</router-link>
		<router-link to="/about">关于</router-link>
		<router-link v-for="user in users" :to="'/user/'+user"> 用户{{user}} </router-link>
		<router-link :to="{path: '/profile', query: {name: 'liuk', sex: 'male'}}">档案</router-link>
		<button @click="profileBtnClick">档案</button>
		<hr/>
    <!-- keep-alive缓存 -->
		<keep-alive>
			<router-view></router-view>
		</keep-alive>
  </div>
</template>
```



### 10.2、属性

- keep-alive有两个非常重要的属性
  - include - 字符串或正则表达式：只有匹配的组件会被缓存
  - exclude - 字符串或正则表达式：任何匹配的组件都不会被缓存

如希望Profile不被缓存，可以这样子写：

```vue
<!-- 单个 -->
<keep-alive exclude="Profile">
  <router-view/>
</keep-alive>
<!-- 多个，这里别随便加空格 -->
<keep-alive exclude="Profile,User">
  <router-view/>
</keep-alive>
```



【2】结合router，缓存部分页面

```javascript
export default[
 {
  path:'/home',
  components:Home,
  meta:{
    keepAlive:true //需要被缓存的组件
 },
 {
  path:'/user',
  components:Book,
  meta:{
     keepAlive:false //不需要被缓存的组件
 } 
]
```

```html
<keep-alive>
  <router-view v-if="this.$route.meta.keepAlive"></router-view>
  <!--这里是会被缓存的组件-->
</keep-alive>
<!--这里是不会被缓存的组件-->
<router-view v-if="!this.$router.meta.keepAlive"></router-view>
```

