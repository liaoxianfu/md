### vue-router配置

在vue-cli创建的项目中使用vue-router。官方文档地址https://router.vuejs.org/zh/installation.html



#### 自定义安装配置

在命令行中使用`npm install vue-router`安装，安装完成之后，创建在项目文件夹下创建router文件夹和index.js

![image-20210304164459977](https://cdn.jsdelivr.net/gh/liaoxianfu/blogimg/data/20210304164507.png)

index.js内容

```js
import Vue from 'vue';
import VueRouter from 'vue-router';

Vue.use(VueRouter);
const routes = [
]; // 路由配置

const router = new VueRouter({
  routes,
  mode: 'history', // 配置html5的history模式
});

export default router;

```

如果不配置	`mode:'history'`默认使用的hash模式，具体来说就是会产生一个'#' 使用history模式会去掉这个'#'详细的介绍在

https://router.vuejs.org/zh/guide/essentials/history-mode.html#%E5%90%8E%E7%AB%AF%E9%85%8D%E7%BD%AE%E4%BE%8B%E5%AD%90



#### 路由配置

这里以一个简单的例子进行模拟。模拟一个点击不同的标题栏跳转不同的组件上。

创建如下的vue组件

![image-20210304165508122](https://cdn.jsdelivr.net/gh/liaoxianfu/blogimg/data/20210304165510.png)

about.vue

```vue
<template>
  <div>这是关于页面</div>
</template>

<script>
export default {
  name: "About",
};
</script>

<style scoped>
</style>>
```

news.vue

```vue
<template>
  <div>这是新闻页面</div>
</template>

<script>
export default {
  name: "News",
};
</script>

<style>
</style>
```

index.vue

```vue
<template>
  <div class="main">
      <h1>这是一个标题</h1>
    <nav>
      <router-link to='/news'>新闻</router-link>&nbsp;&nbsp;&nbsp;
      <router-link to='/about'>关于</router-link>
    </nav>
    <router-view />
  </div>
</template>

<script>
export default {
  name: 'index',
};
</script>

<style scoped>
.main{
    margin-left: 20px;
}
</style>>
```

前面的about和news就是简单的vue组件，不再进行说明，这里的index.vue引入这两个组件，通过`<router-link>`进行跳转。相当于`<a>`标签，里面的`to=path`属性相当于`href`属性。其实，最终`<router-link>`默认情况下也是会渲染成`<a>`标签。那么，如何将

```vue
<router-link to='/news'>新闻</router-link>
```

绑定当对应的组件中呢?

这时候就要在`router/index.js`中进行定义了。

```js
import Vue from 'vue';
import VueRouter from 'vue-router';

Vue.use(VueRouter);

// 使用懒加载
const news = () => import('../components/index/news.vue');
const about = () => import('../components/index/about.vue');

const routes = [
  {
    path: '/news',
    component: news,
  },
  {
    path: '/about',
    component: about,
  },

];

const router = new VueRouter({
  routes,
  mode: 'history',
});

export default router;

```

通过routers集合定义对应的路由集合。为什么使用懒加载呢?

官方文件档解释：

> 当打包构建应用时，JavaScript 包会变得非常大，影响页面加载。如果我们能把不同路由对应的组件分割成不同的代码块，然后当路由被访问的时候才加载对应组件，这样就更加高效了。

具体见 <a href='https://router.vuejs.org/zh/guide/advanced/lazy-loading.html#%E6%8A%8A%E7%BB%84%E4%BB%B6%E6%8C%89%E7%BB%84%E5%88%86%E5%9D%97'>路由懒加载</a>



app.vue

```vue
<template>
  <div id="app">
    <index-vue />
  </div>
</template>
<script>
import indexVue from "./views/index.vue";

export default {
  name: "App",
  components: {
    indexVue,
  },
};
</script>
<style scoped>
</style>
```

![image-20210304172502427](https://cdn.jsdelivr.net/gh/liaoxianfu/blogimg/data/20210304173003.png)

#### 嵌套路由

我们知道，路由一般需要进行嵌套使用,创建如下文件

![image-20210304173127433](https://cdn.jsdelivr.net/gh/liaoxianfu/blogimg/data/20210304173129.png)

```vue
<template>
  <ol>
    <li>你好啊</li>
    <li>你好啊</li>
    <li>你好啊</li>
    <li>你好啊</li>
  </ol>
</template>
<script>
export default {
  name: 'newsList',
};
</script>
<style>
</style>

```

![image-20210304173326466](https://cdn.jsdelivr.net/gh/liaoxianfu/blogimg/data/20210304173328.png)



修改news.vue

```vue
<template>
  <div>
    这是新闻页面
    <router-link to="/news/list">加载列表</router-link>
    <router-view />
  </div>
</template>

<script>
export default {
  name: "News",
};
</script>

<style>
</style>
```

**注意： 嵌套的子路由前面不能添加'/'否者会直接变成一级路由**

#### 默认路由

有时候我们想要在路由在加载的时候就默认显示出来，例如我希望用户在访问的时候就加载news组件以及newsList组件。可以通过配置默认路由进行跳转实现。

```js
const routes = [
  {
    path: '',
    redirect: '/news', // 实现默认路由的跳转
  },
  {
    path: '/news',
    component: news,
    children: [
      {
        path: '',
        redirect: 'list', //实现默认路由的跳转
      },
      {
        path: 'list',
        component: newsList,
      },
    ],
  },
  {
    path: '/about',
    component: about,
  },

];
```

#### 编程式导航

可以通过使用`this.$router.push`进行路由跳转。具体可见<a href="https://router.vuejs.org/zh/guide/essentials/navigation.html">编程式导航</a>注意：在 Vue 实例内部，你可以通过 `$router` 访问路由实例。因此你可以调用 `this.$router.push`。

想要导航到不同的 URL，则使用 `router.push` 方法。这个方法会向 history 栈添加一个新的记录，所以，当用户点击浏览器后退按钮时，则回到之前的 URL。想要导航到不同的 URL，则使用 `router.push` 方法。这个方法会向 history 栈添加一个新的记录，所以，当用户点击浏览器后退按钮时，则回到之前的 URL。如果你不想进行返回的操作,也就是导航栏没有回退的标识.this.$router.replace('/');

#### 动态路由匹配

##### 1、进行路径参数匹配

请求url

```js
const path = `/userInfo/${this.userID}`;
this.$router.push(path)
    .catch((err) => {
     // eslint-disable-next-line no-console
     console.log(err);
     });
```

配置url

在router/index.js中进行配置

```js
const routes = [
 ...
    {
    path: '/userInfo/:userID',
    component: userID,
  },
  .....
];


```



使用`/userInfo/:userID`其中`:userID`用来进行通用的匹配,例如`/userInfo/111`,`/userInfo/222`都会匹配到同一个组件上。

**获取url上的参数**

```vue
<template>
  <div>
    {{ userID }}
  </div>
</template>

<script>
export default {
  name: 'userInfo',
  data() {
    return {
      userID: '',
    };
  },
  mounted() {
    this.userID = this.$route.params.userID;
  },
};
</script>

<style scoped>

</style>

```



通过`this.$route.params.userID`进行获取。其中`userID`是路由规则的参数。



