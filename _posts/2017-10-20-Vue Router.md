---
layout:     post
title:      Vue 路由实战
subtitle:   客户端路由
date:       2017-10-20
author:     jinxm
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Vue
    - 工作总结
---

# 客户端路由
各种路由的使用场景以及问题
1. 嵌套路由
2. 具名路由
3. 多个平级路由出口
4. 复杂匹配规则
5. 当前活跃连接
6. 重定向/别名
7. 跳转动画
8. 异步数据处理
9. 跳转规则限制
10. 滚动条（一般定位使用）
11. 懒加载
> 暂时这么多，开始一个解决

# router.js 的配置

#### mode:

默认为hash，但是用hash模式的话，页面地址会变成被加个#号比较难看了，

 http://localhost:8080/#/linkParams/xuxiao

所以一般我们会采用 history模式，它会使得我们的地址像平常一样。

http://localhost:8080/linkParams/xuxiao

#### base
应用的基路径。例如，如果整个单页应用服务在 /app/ 下，然后 base 就应该设为 "/app/"。
一般写成 __dirname，在webpack中有配置。

#### routes
routes就是我们的大核心了，里面包含我们所有的页面配置。
path 很简单，就是我们的访问这个页面的路径
name 给这个页面路径定义一个名字，当在页面进行跳转的时候也可以用名字跳转，要唯一哟
component 组件，就是咱们在最上面引入的 import ...了，当然这个组件的写法还有一种懒加载

懒加载的方式，我们就不需要再用import去引入组件了，直接如下即可。懒加载的好处是当你访问到这个页面的时候才会去加载相关资源，这样的话能提高页面的访问速度。
component: resolve => require(['./page/linkParamsQuestion.vue'], resolve)

# router 传参

#### 路由传参

```
首先在路由配置文件router.js中做好配置。标红出就是对 /linkParams/的路径做拦截，这种类型的链接后面的内容会被vue-router映射成name参数。
 {
            path: '/:gid(\\d+)/:roleType(\\d+)/news',
            name: 'groupNews',
            component: GroupNews,
        },
代码中获取name的方式如下：

let name = this.$route.params.name; // 链接里的name被封装进了 this.$route.params
```

#### Get 传参

链接后加上?进行传参。

`http://localhost:8080/linkParamsQuestion?age=18`

代码获取
`let age = this.$route.query.age; //问号后面参数会被封装进 this.$route.query;`


# vue-router前端路由的两种实现
> 先来看看路由相关原理，从简。

单页应用（spa）成为前端应用的主流形式。大型单页应用最显著的就是采用前端路由系统，通过改变URL，在不重新请求页面的情况下，更新视图
"更新视图但不重新请求页面"是前端路由原理核心之一，在浏览器实现主要有两种：
1. 利用URL中的hash（#）。
2. 利用History interface在HTML5中新增的方法

### 模式参数
在vue-router中通过mode这一参数控制路由的实现模式:
```
const router = new VueRouter({
    mode: 'history',
    router: [...]
})
```
创建VueRouter的实例对象时。mode以构造函数参数的形式传入,（具体可以去看源码，现在只总结）
1. 作为参数传入的字符串属性mode只是一个标记，用来指示起作用的对象属性history的实现类
` modehistory'history'HTML5History'hash'HashHistory'abstract'AbstractHistory`
2. 在初始化对应的history之前，对mode做校验：浏览器不支持h5 history mode就设为'hash'；若不在浏览器环境运行，则mode强制设为'abstract'
3. vueRouter类中的onReady（），Push（）等方法只是个代理，实际调用的具体的history对象的对应环境，在init()方法中初始化时，也是根据history对象具体的类执行不同的操作
4. 在浏览器环境下的两种方式，分别就是在HTML5History，HashHistory两个类中实现的

replace()方法与push()方法不同之处在于，它并不是将新路由添加到浏览器访问历史的栈顶，而是替换掉当前的路由

##### HashHistory
hash(#) 符号本来作用就是在URL中指示网页中的位置

#符号本身以及他后面的字符称之为hash，可以通过window.location.hash属性读取。
>1. hash虽然在URl中，但不会被包括在HTTP请求中。它是用来指导浏览器动作的，对服务端完全无用，因此，改变hash不会重新加载页面
2. 可以为hash的改变添加监听事件：
` window.addEventListener("hashchange", funcRef, false)`
3. 每次改变hash 都会在浏览器的访问历史中增加一个记录
4. 这些特点，就可以用来实现前端路由"更新视图但不重新请求页面"的功能


##### HTML5History
History interface是浏览器历史记录栈提供的接口，可以通过back（），forward（），go（）等方法，我们可以读取浏览器历史记录栈的信息，进行各种跳转操作。

从HTMl5开始，History interface提供两个新的方法：pushState(),replaceState()使得我们可以对浏览器历史记录栈进行修改
```
window.history.pushState(stateObject, title, URL)
window.history.replaceState(stateObject, title, URL)
```
>1. stateObject: 当浏览器跳转到新的状态时，将触发popState事件，该事件将携带这个sateObject参数的副本
2. title: 所添加记录的标题
3. URL: 所添加记录的URL
4. 这两个方法有个共同的特点：当调用他们修改浏览器历史记录栈后，虽然当前URL改变了，但浏览器不会立即发送请求该URL（the browser won't attempt to load this URL after a call to pushState()），这就为单页应用前端路由“更新视图但不重新请求页面”提供了基础。

vue-Router代码结构以及更新视图的逻辑与hash模式基本类似，只不过将对window.location.hash直接进行赋值window.location.replace()改为了调用history.pushState()和history.replaceState()方法。
在HTML5History中添加对修改浏览器地址栏URL的监听是直接在构造函数中执行的：
```
constructor (router: Router, base: ?string) {

  window.addEventListener('popstate', e => {
    const current = this.current
    this.transitionTo(getLocation(this.base), route => {
      if (expectScroll) {
        handleScroll(router, route, current, true)
      }
    })
  })
}
```
当然了HTML5History用到了HTML5的新特特性，是需要特定浏览器版本的支持的，前文已经知道，浏览器是否支持是通过变量supportsPushState来检查的

以上就是hash模式与history模式源码的导读，这两种模式都是通过浏览器接口实现的，除此之外vue-router还为非浏览器环境准备了一个abstract模式，其原理为用一个数组stack模拟出浏览器历史记录栈的功能。当然，以上只是一些核心逻辑，为保证系统的鲁棒性源码中还有大量的辅助逻辑，也很值得学习。此外在vue-router中还有路由匹配、router-view视图组件等重要部分，关于整体源码的阅读推荐滴滴前端的这篇文章

#### 两种模式的比较
在一般的需求场景中，hash模式与history模式是差不多的，但几乎所有的文章都推荐使用history模式，理由竟然是："#" 符号太丑...0_0 "

根据MDN的介绍，调用history.pushState()相比于直接修改hash主要有以下优势：

1. pushState设置的新URL可以是与当前URL同源的任意URL；而hash只可修改#后面的部分，故只可设置与当前同文档的URL

2. pushState设置的新URL可以与当前URL一模一样，这样也会把记录添加到栈中；而hash设置的新值必须与原来不一样才会触发记录添加到栈中

3. pushState通过stateObject可以添加任意类型的数据到记录中；而hash只可添加短字符串

4. pushState可额外设置title属性供后续使用

history模式的一个问题

我们知道对于单页应用来讲，理想的使用场景是仅在进入应用时加载index.html，后续在的网络操作通过Ajax完成，不会根据URL重新请求页面，但是难免遇到特殊情况，比如用户直接在地址栏中输入并回车，浏览器重启重新加载应用等。

hash模式仅改变hash部分的内容，而hash部分是不会包含在HTTP请求中的：

`http://oursite.com/#/user/id   // 如重新请求只会发送http://oursite.com/`

故在hash模式下遇到根据URL请求页面的情况不会有问题。

而history模式则会将URL修改得就和正常请求后端的URL一样

`http://oursite.com/user/id`
在此情况下重新向后端发送请求，如后端没有配置对应/user/id的路由处理，则会返回404错误。官方推荐的解决办法是在服务端增加一个覆盖所有情况的候选资源：如果 URL 匹配不到任何静态资源，则应该返回同一个 index.html 页面，这个页面就是你 app 依赖的页面。同时这么做以后，服务器就不再返回 404 错误页面，因为对于所有路径都会返回 index.html 文件。为了避免这种情况，在 Vue 应用里面覆盖所有的路由情况，然后在给出一个 404 页面。或者，如果是用 Node.js 作后台，可以使用服务端的路由来匹配 URL，当没有匹配到路由的时候返回 404，从而实现 fallback。

# 实战篇
引入路由
` import goods from 'components/goods/goods'; `
 定义路由
```
const routes = [
  { path: '/goods', component: goods },
  { path: '/ratings', component: ratings },
  { path: '/seller', component: seller }
];
```
 创建路由实例
```
const router = new VueRouter({
  // ES6缩写语法，相当于routes:routes
  routes
});
> 在创建的 router 对象中，如果不配置 mode，就会使用默认的 hash 模式，该模式下会将路径格式化为 #! 开头。

  添加 mode: 'history' 之后将使用 HTML5 history 模式，该模式下没有 # 前缀，而且可以使用 pushState 和 replaceState 来管理记录。

  关于 HTML5 history 模式的更多内容，可以参考官方文档：https://router.vuejs.org/zh-cn/essentials/history-mode.html
```
创建vue实例并挂载
```
new Vue({
    el: '#app',
    router: new VueRouter(routerConfig),
    render: h => h(App)
});
```

#### router-link和router-view
    router-link

    router-link是一个组件。它默认会被渲染成一个带有链接的a标签，通过to属性指定链接地址
> 被选中的router-link将自动日安家一个class属性值。.router-link-active

<b>router-link映射路由</b>

to

这是必须设置的属性，否则路由无法生效，它表示路由的链接，可以是一个字符串，或者一个描述目标位置的对象

在编译之后，<router-link> 会被渲染为 < a> 标签， to会被渲染为 href，当 <router-link> 被点击的时候，url 会发生相应的改变

如果使用 v-bind 指令，还可以在 to 后面接变量，配合 v-for 指令可以渲染导航菜单
```
<router-link to="goods"></router-link>
<router-link v-else :to="'/' + group.gid" tag="div"></router-link>
<router-link :to="'/' + article.gid + '/' + article.aid + '/' + group.roleType" tag="div" class="group_li"></router>
```

如果对于所有 ID 各不相同的用户，都要使用 home 组件来渲染，可以在 routers.js 中添加动态参数：
```
{
    path: '/home/:id',
    component: Home
}
```
这样 "/home/user01"、"/home/user02"、"/home/user03" 等路由，都会映射到 Home 组件

然后还可以使用 $route.params.id 来获取到对应的 id
`   <router-link class="bg-f" :to="'/' + $route.params.gid + '/set'" tag="header">`
`    this.$router.push({path: '/coach/' + this.$route.params.id, query: queryData});`

#### 嵌套路由
为了加深项目的层级。app.vue只是作为一个存放组件的容器
```
<template>
    <div>
        <keep-alive>
            <router-view></router-view>
        </keep-alive>
    </div>
</template>
```
其中<router-view> 是用来渲染通过路由映射过来的组件，路径改变时，<router-view> 中的内容也会发生改变

home.vue是真正的父组件，first.vue login.vue group.vue等子组件都会渲染到home.vue中的<router-view>
```
<template>
    <div id='app'>
        <router-view><router-view>
        <footerDiv></footerDiv>
    <div>
<template>

import Home from './components/home.vue'
import First from './components/children/first.vue'
import Login from './components/children/login.vue'

const routers = [
  {
    path: '/',
    component: Home,
　　 children: [
　　　{
　　　　path: '/',
 　　　 component: Login
　　  }
　　]
  },
  {
    path: '/home',
    name: 'home',
    component: Home,
    children: [
      {
        path: '/',
        name: 'login',
        component: Login
      },
      {
        path: 'first',
        name: 'first',
        component: First
      }
    ]
  }
]

```

在配置的路由后面，添加children， 并在children中添加二级路由，就能实现路由的嵌套

<b>配置path的时候，以'/'开头的嵌套路径会被当做根路径，所以子路由的path 不需要添加'/'</b>

#### 编程式导航
实际情况下，在执行跳转前，还会执行一系列方法，这是可以使用 this.$router.push(location) 来修改url，完成跳转

```
<div>
    <button class='sign' @click='goFirst'>登录</button>
</div>

methods: {
    goFirst: function () {
        this.$router.push({ path: '/home/first' })
}
push后面可以是对象，也可以是字符串
// 字符串
this.$router.push('/home/first')

// 对象
this.$router.push({ path: '/home/first' })

// 命名的路由
this.$router.push({ name: 'home', params: { userId: wise }})
// 带查询参数，变成/backend/order?selected=2
this.$router.push({path: '/backend/order', query: {selected: "2"}});

// 设置查询参数
this.$http.post('v1/user/select-stage', {stage: stage})
      .then(({data: {code, content}}) => {
            if (code === 0) {
                // 对象
                this.$router.push({path: '/home'});
            }else if(code === 10){
                // 带查询参数，变成/login?stage=stage
                this.$router.push({path: '/login', query:{stage: stage}});
           }
});

// 设计查询参数对象
let queryData = {};
if (this.$route.query.stage) {
    queryData.stage = this.$route.query.stage;
}
if (this.$route.query.url) {
    queryData.url = this.$route.query.url;
}
this.$router.push({path: '/my/profile', query: queryData});


```

replace

设置 replace 属性的话，当点击时，会调用 router.replace() 而不是 router.push()，于是导航后不会留下 history 记录。即使点击返回按钮也不会回到这个页面。
> 加上replace: true后，它不会向 history 添加新记录，而是跟它的方法名一样 —— 替换掉当前的 history 记录。
```
this.$router.push({path: '/home', replace: true})
//如果是声明式就是像下面这样写：
<router-link :to="..." replace></router-link>
// 编程式:
router.replace(...)
        this.$router.replace({ name: 'Page05', params: { abc: 'hello', txt: this.world }, query: { name: this.id, type: this.ob }})
```
#### 路由监听方法

```
 watch: {
      //路由监听方法
      '$route' (to, from) {
        console.log(this.$route)
      },
    },
```

#### 重定向
```
const routes = [
    { path: '/', redirect: '/index'},     // 这样进/ 就会跳转到/index
    { path: '/index', component: index }
]
```

#### 懒加载
```
const routes = [
    { path: '/index', component: resolve => require(['./index.vue'], resolve) },
    { path: '/hello', component: resolve => require(['./hello.vue'], resolve) },
]
通过懒加载就不会一次性把所有组件都加载进来，而是当你访问到那个组件的时候才会加载那一个。对于组件比较多的应用会提高首次加载速度。
```

# 路由信息对象
1. $route.path
字符串，对应当前路由的路径，总是解析为绝对路径，如 "/foo/bar"。
2. $route.params
一个 key/value 对象，包含了 动态片段 和 全匹配片段，如果没有路由参数，就是一个空对象。
3. $route.query
一个 key/value 对象，表示 URL 查询参数。例如，对于路径 /foo?user=1，则有 $route.query.user == 1，如果没有查询参数，则是个空对象。
4. $route.hash
当前路由的 hash 值 (不带 #) ，如果没有 hash 值，则为空字符串。
5. $route.fullPath
完成解析后的 URL，包含查询参数和 hash 的完整路径。
6. $route.matched
一个数组，包含当前路由的所有嵌套路径片段的 路由记录 。路由记录就是 routes 配置数组中的对象副本（还有在 children 数组）

# 路由钩子
> 导航钩子函数，主要是在导航跳转的时候做一些操作，比如可以做登录的拦截，而钩子函数根据其生效的范围可以分为 全局钩子函数、路由独享钩子函数和组件内钩子函数。
1. 全局钩子      直接在配置文件router.js里编写代码逻辑。可以做一些全局性的路由拦截
2. 某个路由独享的钩子    做一些单个路由的跳转拦截
3. 组件内钩子       更细粒度的路由拦截，只针对一个人进入某一个组件的拦截

#### 全局钩子

```
const router = new VueRouter({
    mode: 'history',
    base: __dirname,
    routes: routerConfig
})

router.beforeEach((to, from, next) => {
    document.title = to.meta.title || 'demo'
    if (!to.query.url && from.query.url) {
        to.query.url = from.query.url
    }
    next()
})

router.afterEach(route => {

})
```

#### 某个路由独享钩子
给某个路由单独使用的，本质上和后面的组件内钩子是一样的。都是特指的某个路由。不同的是，这里的一般定义在router当中，而不是在组件内。如下

```
const router = new VueRouter({
  routes: [
    {
      path: '/foo',
      component: Foo,
      beforeEnter: (to, from, next) => {
        // ...
      },
      beforeLeave: (to, from, next) => {
        // ...
      }
    }
  ]
})
```

#### 组件内的钩子

> beforeRouteEnter
  beforeRouteUpdate (2.2 新增)
  beforeRouteLeave
  路由组件！== 组件
  路由组件：直接定义在router中的component处的组件

  使用 和data，method平级
  ```
  beforeRouteLeave(to, from, next) {
      // ....
      // 在渲染该组件的对应路由被 confirm 前调用
          // 不！能！获取组件实例 `this`
          // 因为当钩子执行前，组件实例还没被创建
      next()
  },
  beforeRouteEnter(to, from, next) {
      // ....
      // 在当前路由改变，但是改组件被复用时调用
          // 举例来说，对于一个带有动态参数的路径 /foo/:id，在 /foo/1 和 /foo/2 之间跳转的时候，
          // 由于会渲染同样的 Foo 组件，因此组件实例会被复用。而这个钩子就会在这个情况下被调用。
          // 可以访问组件实例 `this`


      next()
  },
  beforeRouteUpdate(to, from, next) {
      // ....
       // 导航离开该组件的对应路由时调用
          // 可以访问组件实例 `this`
      next()
  },
  computed: {},
  method: {}
  ```

#### 路由钩子参数
```
to: Route: 即将要进入的目标 路由对象     to对象下面的属性： path   params  query   hash   fullPath    matched   name    meta（在matched下，但是本例可以直接用）
from: Route: 当前导航正要离开的路由
next: Function: 一定要调用该方法来 resolve 这个钩子。执行效果依赖 next 方法的调用参数。
next(): 如果没有参数 进行管道中的下一个钩子。如果全部钩子执行完了，则导航的状态就是 confirmed （确认的）。
next(false): 中断当前的导航。如果浏览器的 URL 改变了（可能是用户手动或者浏览器后退按钮），那么 URL 地址会重置到 from 路由对应的地址。
next(‘/’) 或者 next({ path: ‘/’ }): 跳转到一个不同的地址。当前的导航被中断，然后进行一个新的导航。
```

# 滚动行为
> 在利用vue-router去做跳转的时候，到了新页面如果对页面的滚动条位置有要求的话..麻烦了

```
const router = new VueRouter({
  routes: [...],
  scrollBehavior (to, from, savedPosition) {
    // return 期望滚动到哪个的位置
  }
})

scrollBehavior 方法接收 to 和 from 路由对象。
第三个参数 savedPosition 当且仅当 popstate 导航 (mode为 history 通过浏览器的 前进/后退 按钮触发) 时才可用。
//所有路由新页面滚动到顶部：
scrollBehavior (to, from, savedPosition) {
  return { x: 0, y: 0 }
}
 gorkor
scrollBehavior(to, from, savedPosition) {
        if (from.name === 'group') {
            from.meta.savedPosition = document.body.scrollTop;
        }
        if (savedPosition) {
            return savedPosition
        } else {
            return {
                x: 0,
                y: to.meta.savedPosition || 0
            }
        }
    }

//如果有锚点
scrollBehavior (to, from, savedPosition) {
  if (to.hash) {
    return {
      selector: to.hash
    }
  }
}
```

# 路由元信息
这个概念非常简单，就是在路由配置里有个属性叫 meta，它的数据结构是一个对象。你可以放一些key-value进去，方便在钩子函数执行的时候用。
举个例子，你要配置哪几个页面需要登录的时候，你可以在meta中加入一个 requiresAuth标志位。

```
const router = new VueRouter({
  routes: [
    {
      path: '/foo',
      component: Foo,
      meta: { requiresAuth: true }
    }
  ]
})

然后在 全局钩子函数 beforeEach中去校验目标页面是否需要登录。

router.beforeEach((to, from, next) => {
  if (to.matched.some(record => record.meta.requiresAuth)) {
    //校验这个目标页面是否需要登录
    if (!auth.loggedIn()) {
      next({
        path: '/login',
        query: { redirect: to.fullPath }
      })
    } else {
      next()
    }
  } else {
    next() // 确保一定要调用 next()
  }
})
下版本会需要   <br>
这个auth.loggedIn 方法是外部引入的，你可以先写好一个校验是否登录的方法，再import进 router.js中去判断。

```




# 附上生命周期
beforeCreate（创建前）, el 和 data 并未初始化    .可以在这加个loading事件

created（创建后）,完成了 data 数据的初始化，el没有      .在这结束loading，还做一些初始化，实现函数自执行

beforeMount(载入前),完成了 el 和 data 初始化

mounted（载入后）,完成挂载     . 在这发起后端请求，拿回数据，配合路由钩子做一些事情

beforeUpdate（更新前）,

updated（更新后）,data里的值被修改后，将会触发update的操作

beforeDestroy（销毁前）,   .你确认删除XX吗？

destroyed（销毁后）,     .当前组件已被删除，清空相关内容
