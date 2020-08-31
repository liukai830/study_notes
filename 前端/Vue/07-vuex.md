	Vuex是一个专门为Vue.js应用程序开发的状态管理模式，有的状态需要组件间共享，如：

- 用户的登录状态、用户名、头像、地理位置等
- 商品的收藏、购物车中的物品等
- 这些状态信息，我们都可以放在统一的地方，对它们进行保存和管理，而且还是响应式的



> Vuex状态图例：

![image-20200406163646404](https://gitee.com/liukai830/picgo/raw/master/image-20200406163646404.png)



## 一、基本使用

【1】安装 `npm install vuex --save`

【2】创建stroe/index.js，内容如下

```javascript
import Vue from 'vue'
import Vuex from 'vuex'

// 1. 安装插件
Vue.use(Vuex)

// 2. 创建对象
const store = new Vuex.Store({
  state: {
    counter: 100
  },
  mutations: {

  },
  actions: {

  },
  getters: {

  },
  modules: {

  }
})

// 导出store对象
export default store
```

【3】在main.js中挂载Vuex

```javascript
import Vue from 'vue'
import App from './App'
import router from './router'
import store from './store/index.js'

Vue.config.productionTip = false

/* eslint-disable no-new */
new Vue({
  el: '#app',
  router,
  store,
  render: h => h(App)
})
```

【4】使用

`<h2>{{$store.state.counter}}</h2>`



## 二、核心概念

### 2.1、State

#### 2.1.1 State单一状态树

​	把所有数据都放在同一个store中，而不是创建多个store进行管理。



### 2.2、Getter

​	类似于计算属性。

#### 2.2.1 通过属性访问

```javascript
const store = new Vuex.Store({
  state: {
    todos: [
      { id: 1, text: '...', done: true },
      { id: 2, text: '...', done: false }
    ]
  },
  getters: {
    filterTodos(state){
      return state.todos.filter(todo => todo.done)
    }
  }
})
// 访问
this.$store.getters.filterTodos
```

#### 2.2.2 接受其他 getter 作为参数

```javascript
const store = new Vuex.Store({
  state: {
    todos: [
      { id: 1, text: '...', done: true },
      { id: 2, text: '...', done: false }
    ]
  },
  getters: {
    // 1. 过滤
    filterTodos(state){
      return state.todos.filter(todo => todo.done)
    },
    // 2. 过滤后的数量
    filterTodosCount(state,getters){
      return getters.filterTodos.length
    }
  }
})
```

#### 2.2.3 通过方法访问：传入参数

```javascript
const store = new Vuex.Store({
  state: {
    todos: [
      { id: 1, text: '...', done: true },
      { id: 2, text: '...', done: false }
    ]
  },
  getters: {
    // 1. 过滤
    filterTodos(state){
      return state.todos.filter(todo => todo.done)
    },
    // 2. 过滤后的数量
    filterTodosCount(state,getters){
      return getters.filterTodos.length
    },
    // 3. 根据传来的参数过滤
    filterTodosByDoneStatus(state) {
      return function(doneStatus) {
        return state.todos.filter(todo => todo.done === doneStatus)
      }
    }
  }
})

// 调用方式，只显示状态为true的
this.$store.getters.filterTodosByDoneStatus(true)
```

上面的这个可以改为箭头函数写法：

```javascript
// 1. 普通写法
filterTodosByDoneStatus(state) {
  return function(doneStatus) {
    return state.todos.filter(todo => todo.done === doneStatus)
  }
}
// 2. 箭头函数修改1
filterTodosByDoneStatus(state) {
  return doneStatus => state.todos.filter(todo => todo.done === doneStatus)
}

//3. 箭头函数再修改2
filterTodosByDoneStatus： state => doneStatus => state.todos.filter(todo => todo.done === doneStatus)
```



### 2.3、Mutation

​	更改 Vuex 的 store 中的状态的唯一方法是提交 mutation。mutations中的方法第一个参数默认都是state。

#### 2.3.1 没有参数

```javascript
// 定义方法
const store = new Vuex.Store({
  state: {
    counter: 100
  },
  mutations: {
    counterAdd(state) {
      state.counter++;
    },
    counterSub(state) {
      state.counter--;
    }
  }
})
// 调用
this.$store.commit('counterAdd')
this.$store.commit('counterSub')
```

#### 2.3.2 提交载荷（Payload）

​	一般会是一个对象，这样可以包含多个字段并且记录的 mutation 会更易读

```javascript
// 定义方法
const store = new Vuex.Store({
  state: {
    msg: 'hello World'
  },
  mutations: {
    msgChange(state,payload) {
      state.msg = payload.newValue;
    }
  }
})
// 调用
this.$store.commit('msgChange', {newValue: '改变后的字符串'})
```

#### 2.3.3 以对象风格提交

```javascript
const store = new Vuex.Store({
  state: {
    count: 1
  },
  mutations: {
    increment (state, payload) {
    	state.count += payload.amount
  	}
  }
})
// 提交
store.commit({
  type: 'increment',
  amount: 10
})
```

#### 2.3.4 <font color="red">Mutation 必须是同步函数</font>

​	假设mutation不是同步函数，下面的例子：

```javascript
mutations: {
  someMutation (state) {
    api.callAsyncMethod(() => {
      state.count++
    })
  }
}
```

​	现在想象，我们正在debug一个app并且观察devtools中的mutation日志。每一条mutation都被记录，devtools都需要捕捉到前一状态和后一状态的快照。然后在上面这个例子中mutation中的异步函数中的回调让这不可能完成：因为当mutation触发的时候，回调函数还没有被调用，<font color="red">devtools不知道什么时候回调函数实际上被调用</font>——实质上任何在回调函数中进行的状态的改变都是不可追踪的。

#### 2.3.5 类型常量

（1）定义常量文件，store/mutations-types.js

```javascript
const types = {
  "COUNTER_ADD": "counterAdd"
}

export default types
```

（2）使用

```javascript
import Vue from 'vue'
import Vuex from 'vuex'
import Names from './mutations-types.js'

// 1. 安装插件
Vue.use(Vuex)

// 2. 创建对象
const store = new Vuex.Store({
  state: {
    counter: 100,
    msg: 'hello World'
  },
  mutations: {
    [Names.COUNTER_ADD](state) {
      state.counter++;
    },
  }
})

```

```vue
<template>
  <div id="app">
    <h2>{{$store.state.msg}}</h2>
    <button @click="changeMsg">改变字符串</button>
  </div>
</template>
<script>
import HelloVuex from './components/HelloVuex'
import Names from './store/mutations-types.js'
export default {
  name: 'App',
  methods: {
    add() {
      this.$store.commit(Names.COUNTER_ADD)
    }
  }
}
</script>

<style>
</style>
```



### 2.4、Action

Action类似于mutation，不同在于：

- Action提交的是mutation，而不是直接变更状态
- Action可以包含任意异步操作

#### 2.4.1 简单案例

```javascript
const store = new Vuex.Store({
  state: {
    count: 0
  },
  mutations: {
    increment(state) {
      state.count++
    }
  },
  actions: {
    incrementAsync(context) {
      // 在actions中提交mutations，让mutations去操作state
      context.commit('increment')
    },
    incrementPayloadAsync(context,payload) {
      // 在actions中提交mutations，让mutations去操作state
      console.log(payload)
      context.commit('increment')
    }
  }
})
```

```javascript
$store.dispatch('incrementAsync')
$store.dispatch('incrementPayloadAsync',{param:'ssss'})
```

#### 2.4.2 回调

```javascript
const store = new Vuex.Store({
  state: {
    count: 0
  },
  mutations: {
    increment(state) {
      state.count++
    }
  },
  actions: {
    incrementAsync(context) {
      return new Promise((resolve,reject) => {
        context.commit('increment')
        relove()
      })
    },
  }
})
```

```javascript
$store.dispatch('incrementAsync').then(() => {})
```



### 2.5、Module

​	将 store 分割成**模块（module）**。每个模块拥有自己的 state、mutation、action、getter、甚至是嵌套子模块——从上至下进行同样方式的分割：

​	其实可以把每个module抽离成不同的js，然后再统一导入

```javascript
const moduleA = {
  state: { ... },
  mutations: { ... },
  actions: { ... },
  getters: { ... }
}

const moduleB = {
  state: { ... },
  mutations: { ... },
  actions: { ... }
}

const store = new Vuex.Store({
  modules: {
    a: moduleA,
    b: moduleB
  }
})


```

#### 2.5.1 State调用

```javascript
$store.state.a // -> moduleA 的状态
$store.state.b // -> moduleB 的状态
```

#### 2.5.2 Mutation调用

​	和正常一样调用，而且建议各个模块中mutations方法名别重复

```javascript
$store.commit('xxx')
```

#### 2.5.3 Getter调用

​	和正常一样调用，而且建议各个模块中getters方法名别重复

```javascript
this.$store.getters.xxxx
```

​	但是如果需要引用大模块的内容，在Getter函数中有第三个参数rootState

```javascript
const moduleA = {
  // ...
  getters: {
    sumWithRootCount (state, getters, rootState) {
      return state.count + rootState.count
    }
  }
}
```

#### 2.5.4 Action调用

​	action中的commit只会调用自己模块内的mutations中的函数