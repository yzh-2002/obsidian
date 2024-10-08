> 主要用于构建服务端渲染（SSR）和静态站点（SSG）生成的应用。

为什么要使用nuxt？
1. seo：搜索引擎优化
	1. 多页面应用
	2. Web网站的title，描述，关键词信息设置
		1. SPA可通过`vue-meta-info`插件进行设置
	3. 网站内容渲染机制
		1. CSR（客户端渲染）：页面内容由JS生成，搜索引擎的爬虫无法获取页面内容
		2. 预渲染：SPA可以通过`prerender-spa-plugin`实现
			1. 所谓预渲染，就是打包时对于指定的页面，**先渲染出页面内容，再生成对应html文件**
			2. 存在的问题：**动态路由页面无法生成对应html文件**
			3. 适用场景：某网站仅几个页面需要做SEO优化
		3. SSR（服务端渲染）：
2. 貌似其他优势都不明显....

SSR原理：
![image.png](https://raw.githubusercontent.com/yzh-2002/img-hosting/main/notes/202409202106245.png)

SSR的nodejs服务层和平时所说的BFF（Backend for FrontEnd）区别是什么呢？
1. BFF：为每种前端提供一个特定的后端服务，作为前端与后端之间的中间层，专门处理前端特定需求的数据聚合和API调用，主要目的：**减少前端与多个微服务之间的直接交互**
2. SSR：不再赘述
3. 两者可以Nodejs环境下共存，并常常一起工作
### Nuxt基础

1. 新建`nuxt`项目：`yarn create nuxt-app xxx`
	1. 模板引擎（HTML与PUG）
	2. Nuxt.js Modules
		1. Axios
		2. PWA（Progressive Web App）
			1. `Service Workers`：pwa的核心技术之一，是JavaScript脚本，运行在浏览器后台，独立于网页，可拦截网络请求，缓存资源，从而使得Web应用具备**离线功能，后台同步，推送通知等功能**
			2. `Manifest`文件：定义应用的图标，启动URL，显示模式等
		3. Content（Git-based headless CMS）
	3. 渲染模式（Universal（SSR/SSG）或SPA）
	4. 部署目标：Server（nodejs hosting）和static（static/jamstack hosting）
2. Nuxt各文件夹含义：[官方文档](https://nuxt.com/docs/guide/directory-structure/app)
	1. `middleware`：该文件夹下定义的是路由中间件，需要与`Server`下的middleware区分开
		1. 该文件夹下通过`defineNuxtRouteMiddleware`定义中间件，全局中间件名称为`*.global.js`
		2. 页面组件中通过`definePageMeta`使用中间件
	2. `server`：
	3. `plugins`：该文件夹下的代码会在**应用启动时加载和初始化**，使其在整个应用中可用
	4. `modules`：**项目运行依赖的代码**，区分plugins，先于plugins运行。
3. Nuxt 生命周期：[官方文档](https://nuxt.com/docs/api/advanced/hooks)
	1. 需要注意的是：由于SSR应用涉及两个端，故生命周期也分两个端（app hooks和server hooks）
4. Nuxt特性
	1. 自动导入，即Nuxt默认生成文件夹中定义的函数等在其他文件夹下使用无需import，自动导入
	2. nuxt自带状态管理，接口请求等很多工具，无需额外引入
5. `nuxt.config.ts`
	1. `runtimeConfig`：运行时的全局变量，在组件中可使用`useRuntimeConfig`获取
		1. 定义在`public`属性的变量可以在服务端和客户端获取，其余变量只能在服务端获取
	2. 

Nuxt内置了`nitro服务器`，启动nuxt项目时，一般默认开启该服务器，浏览器访问的页面均是经nitro服务器渲染好的特面，我们平时写的`.vue`文件中的JS代码会在`Nitro`服务器运行一遍（填充模板），最后也会被携带进渲染好的html中，在浏览器也执行一遍（**执行两次**）。

存在的问题：如果需要执行的JS代码依赖浏览器的对象，例如DOM，BOM怎么办？可以通过判断当前环境是否在浏览器中，来限制代码在何处执行。

Nuxt路由：
1. `NuxtPage`（等同于`RouterView`），`Nuxt-link`
2. Nuxt的路由无需配置，和pages下的文件名称与结构有关
	1. 命名路由，可选路由，全局路由，嵌套路由
3. 编程式路由：`router.push`只能在客户端渲染时使用，`navigateTo`：两端都能使用
4. 路由导航守卫：基于路由中间件实现，在文件夹`middleware`实现

Nuxt提供了`$fetch`用于网络请求，但直接使用其会在浏览器和服务器均执行一次，多请求一次，为此，Nuxt提供了`useAsyncData`函数，需要传给他一个key和一个callback（异步请求），**服务端执行它之后会把callback的结果存储到key对应位置，客户端便可以通过key拿到结果，避免多余请求。**

除此之外，还提供了`useFetch`，相当于`useAsyncData`和`$fetch`的语法糖，参数直接是一个url

问题：`axios`可以定义拦截器实现一些必要操作，`useFetch`要怎么实现这些功能呢？二次封装
```javascript
export const apiCore = (url,options)=>{
	const config =useRuntimeConfig()
	return useFetch(url,{
		baseURL:config.public.baseURL,
		onRequest({options}){
			//请求拦截器的内容
		},
		onResponse(){
			//响应拦截器内容
		},
		onResponseError(){
			//
		}
	})
}
```

Nuxt登录验证逻辑处理：
1. 由于登录之后，客户端会存储一个token到`localStorage`作为登录标识，但是服务端无法获取该标识
2. 所以路由守卫在服务端侧中不能根据是否携带token阻止登录，而应该把这部分逻辑全部放在客户端执行。
	1. 无论是否携带token，服务端直接放行，去渲染跳转后的页面
		1. 客户端会再执行路由守卫及页面跳转过程，如果携带token，跳转成功
		2. 如果不携带token，跳转到登录页
	2. 同时需要注意，如果涉及到token，页面的初始化请求要放在`onMounted`hook内执行

**但如果将标识存储到`cookie`中，则两端都能获取**，也就不存在上述过程

Nuxt自带了一个状态管理，state
```javascript
// res是在客户端与服务端的全局对象
let res =useState('key',()=>{
	return xxx
})
```

1. 服务端设置state，服务端和客户端都能获取
	1. 在客户端未主动获取之前，浏览器上状态管理插件看不到对应状态
2. 客户端设置的state，服务端获取不到
	1. 因为请求服务端需要刷新页面，刷新之后客户端设置的state就丢失了
	2. 不过就算客户端持久化存储了，服务端仍然访问不到（也正常）

Nuxt错误处理：
1. `error.vue`：即为nuxt应用错误兜底页面
2. `defineProps({error:Object})`可获取错误信息

Nuxt打包发布：
1. nuxt build：会生成nodejs端的程序
2. nuxt generate