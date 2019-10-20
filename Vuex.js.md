# Vuex.js

## Vuex是什么

​	Vuex 是一个专为 Vue.js 应用程序开发的**状态管理模式**。它采用集中式存储管理应用的所有组件的状态，并以相应的规则保证状态以一种可预测的方式发生变化。

## 为什么需要使用状态管理

​	在我们构建一个中大型的单页面应用的时候，可能会遇到**多个组件**依赖于同**一个状态**，又或是**多个组件**的行为会变更**同一个状态**的情景，如果我们不适用vuex去进行统一的状态管理，那么这些状态和视图的变更会变得十分复杂。尤其是涉及到**非父子组件**之间的数据交互的时候，我们若是不使用vuex进行状态管理的话势必会变得相当无力。

​	在小型的项目中，一个简单的 [global event bus](http://vuejs.org/guide/components.html#Non-Parent-Child-Communication) 就能满足我们的需求。

## 概念

​	每一个 Vuex 应用的核心就是 store（仓库）。“store”基本上就是一个容器，它包含着你的应用中大部分的**状态 (state)**。Vuex 和单纯的全局对象有以下两点不同：

1. Vuex 的状态存储是响应式的。当 Vue 组件从 store 中读取状态的时候，若 store 中的状态发生变化，那么相应的组件也会相应地得到高效更新。
2. 你不能直接改变 store 中的状态。改变 store 中的状态的唯一途径就是显式地**提交 (commit) mutation**。这样使得我们可以方便地**跟踪每一个状态的变化**，从而让我们能够实现一些工具帮助我们更好地了解我们的应用。
   - 这样你在阅读代码的时候能更容易地解读应用内部的状态改变。
   - 有机会去实现一些能记录每次状态改变，保存状态快照的调试工具。

![](https://user-gold-cdn.xitu.io/2018/10/19/1668a5c2dd59154f?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



​	组件的状态和数据都放到一个统一的内存空间state去管理，state的数据映射到组件上，当组件的数据发生变化时，通过Dispatch Actions或者直接Commit Mutations的方法修改state。我们不能直接改变state中的状态。改变state中状态的唯一途径就是显式的提交（Commit）Mutations。这样使得我们可以方便的追踪每一个状态的变化，从而使我们能够实现一些功能帮助我们更好的了解我们的应用。state修改后反映到组件上，形成闭环。

## vue-cli使用vuex

1. `` yarn add vuex``

2. 编写store.js文件  基础模板

   ```vue
   import vue from 'vue'
   import Vuex from 'vuex'
   
   vue.use(Vuex)
   
   export default new Vuex.Store({
       state: {
         
       },
       mutations: {
   
       },
       actions: {
           
       }
   })
   ```

3. 在main.js中导入，并将store挂载到实例上。

4. 在组件中使用this.$store去访问

## state

​	Vuex 使用**单一状态树**——，用一个对象包含了全部的应用层级状态。作为一个唯一数据源存在。每个应用都只包含一个store实例，单一状态树的设计思路有利于定位状态片段---也就是类似于快照。但这其实与模块化的思路并不冲突。

​	我们通常在组件中使用计算属性去返回state里的属性。

```vue
computed: {
    count () {
    	return this.$store.state.count
    }
}
```

​	我们也可使用辅助函数mapState来帮助我们快速生成计算属性，但为了使用this去获取s局部状态时依然要使用常规的函数。

## Getter

​	getter属性用于对state里的属性派生出一些状态来，比如state中有一个起始值start,结束值end。我们在组件中需要用到start于end的差值，如果我们只是把这项计算过程写在compute中的话，一旦需要用到这项属性的组件变多，我们的代码也会变得冗余。因此我们可以将其写在getter中.

```js
getters: {
	startToEnd: state => {
    	return state.start - state.end
    }
},
```

​	利用getter返回函数来实现查询。

```js
getters: {
  // ...
  getTodoById: (state) => (id) => {
    return state.todos.find(todo => todo.id === id)
  }
}
```

## Mutation

​	提交mutation是唯一的更改state中数据的办法，每个 mutation 都有一个字符串的 **事件类型 (type)** 和 一个 **回调函数 (handler)**。这个回调函数就是我们实际进行状态更改的地方，并且它会接受 state 作为第一个参数：

```js
mutations: {
    increment (state) {
      // 变更状态
      state.count++
    }
}
//调用
this.$store.commit('increment')
```

 	使用payload(提交荷载)来传入额外的参数。大多数情况下建议把荷载写成对象的形式，这样能一次性包含多个字段，并且使mutation更加易读。

```js
mutations: {
  increment (state, payload) {
    state.count += payload.amount
  }
}
//调用
this.$store.commit('increment', {
  amount: 10
})
```

​	所有的mutation都是同步事务，异步的mutation会对状态追踪造成麻烦，如果需要使用异步函数，那么应该使用action来解决。

## Action

​	Action 函数接受一个与 store 实例具有相同方法和属性的 context 对象，因此你可以调用 `context.commit` 提交一个 mutation，或者通过 `context.state` 和 `context.getters` 来获取 state 和 getters。

- Action 提交的是 mutation，而不是直接变更状态。

- Action 可以包含任意异步操作。

- Action 通过 `store.dispatch` 方法触发`store.dispatch('methond')`

- Actions 支持同样的载荷方式和对象方式进行分发。

  ```js
  // 以载荷形式分发
  store.dispatch('incrementAsync', {
    amount: 10
  })
  
  // 以对象形式分发
  store.dispatch({
    type: 'incrementAsync',
    amount: 10
  })
  ```

​	`store.dispatch` 可以处理被触发的 action 的处理函数返回的 Promise，并且 `store.dispatch` 仍旧返回 Promise.既然返回的是promise，那么我们不仅可以用**.then**的方法去链式调用。也可以使用 **async / await**。

## Module

​	正如前面所说的，vuex使用单一状态树，每个应用只有一个store实例。如果我们的项目中需要管理的状态过多，那么这个store中的数据就很变得十分冗杂，不利于维护。因此vuex提供了模块，让我们可以分割我们的store，并且每个模块中都有自己的state、mutation、action、getter。模块中去嵌套模块也是可行的。

 ```js
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

store.state.a // -> moduleA 的状态
store.state.b // -> moduleB 的状态
 ```

​	对于模块内部的 action，局部状态通过 `context.state` 暴露出来，根节点状态则为 `context.rootState`：

```js
const moduleA = {
  // ...
  actions: {
    incrementIfOddOnRootSum ({ state, commit, rootState }) {
      if ((state.count + rootState.count) % 2 === 1) {
        commit('increment')
      }
    }
  }
}
```

### 命名空间

​	默认情况下，模块内部的 action、mutation 和 getter 是注册在**全局命名空间**。当然，如果为了使我们的模块具有更高的封装性和复用性，我们也可以使用命名空间的方法。也就是声明` namespaced: true `。声明namespace为true之后的模块中的getter，action，mutation会根据你模块的路径自动调整命名，我们调用时只需要根据相应的规则去修改即可。

```js
const store = new Vuex.Store({
  modules: {
    account: {
      namespaced: true,

      // 模块内容（module assets）
      state: { ... }, // 模块内的状态已经是嵌套的了，使用 `namespaced` 属性不会对其产生影响
      getters: {
        isAdmin () { ... } // -> getters['account/isAdmin'] account/isAdmin就是模块的路径
      },
      actions: {
        login () { ... } // -> dispatch('account/login')
      },
      mutations: {
        login () { ... } // -> commit('account/login')
      },

      // 嵌套模块
      modules: {
        // 继承父模块的命名空间
        myPage: {
          state: { ... },
          getters: {
            profile () { ... } // -> getters['account/profile']
          }
        },

        // 进一步嵌套命名空间
        posts: {
          namespaced: true,

          state: { ... },
          getters: {
            popular () { ... } // -> getters['account/posts/popular']
          }
        }
      }
    }
  }
})
```

### 注册

​	在 store 创建**之后**，可使用 `store.registerModule` 方法注册模块：

```js
// 注册模块 `myModule`
store.registerModule('myModule', {
  // ...
})
// 注册嵌套模块 `nested/myModule`
store.registerModule(['nested', 'myModule'], {
  // ...
})
```

​	然后通过 `store.state.myModule` 和 `store.state.nested.myModule` 访问模块的状态。