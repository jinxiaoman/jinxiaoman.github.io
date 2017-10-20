---
layout:     post
title:      Vue 路由实战
subtitle:   客户端路由
date:       2017- -
author:     jinxm
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - vue
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

# 两种模式的比较
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

# router-link和router-view
    router-link

    router-link是一个组件。它默认会被渲染成一个带有链接的a标签，通过to属性指定链接地址
> 被选中的router-link将自动日安家一个class属性值。.router-link-active

router-link属性配置

to

这是必须设置的属性，否则路由无法生效，它表示路由的链接，可以是一个字符串，或者一个描述目标位置的对象

在编译之后，<router-link> 会被渲染为 <a> 标签， to 会被渲染为 href，当 <router-link> 被点击的时候，url 会发生相应的改变

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

## 嵌套路由
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