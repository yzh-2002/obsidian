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
		1. 类似于路由守卫
	2. `server`：
	3. ...
3. Nuxt 生命周期：[官方文档](https://nuxt.com/docs/api/advanced/hooks)
	1. 需要注意的是：由于SSR应用涉及两个端，故生命周期也分两个端（app hooks和server hooks）
4. Nuxt特性
	1. 自动导入，即Nuxt默认生成文件夹中定义的函数等在其他文件夹下使用无需import，自动导入
	2. nuxt自带状态管理，接口请求等很多工具，无需额外引入
