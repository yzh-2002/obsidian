> 用于构建跨平台的移动端应用和小程序

1. 创建项目：`npx degit dcloudio/uni-preset-vue#vite-ts <name>`


## applet vs Web





## wx applet

1. 微信服务市场
2. 小程序插件等等机制....

### 内嵌H5
> [基于WebView实现](https://developers.weixin.qq.com/miniprogram/dev/component/web-view.html)，不支持个人类型的小程序

内嵌网页如果需要使用微信的相关能力，需要配置JSSDK，[JS-SDK说明文档](https://developers.weixin.qq.com/doc/offiaccount/OA_Web_Apps/JS-SDK.html)



### 迁移

现有的Vue H5项目如何迁移到uni-app中？

1. 路由配置：uni-app的页面路由需要通过`pages.json`中配置，[pages配置参数](https://zh.uniapp.dcloud.io/collocation/pages.html)
2. 