# 如何优化网站速度

我用create-react-app脚手架工具搭建了[博客网站](https://notebook.huolong.tk)。项目安装了某些依赖包后，在租用的1核1G的服务器`npm run build`通常会失败，使用`top`发现在build过程中，它占用了100%的cpu导致build中断，而且网站访问速度也变慢了。我想，原因可能是引入了大的依赖导致build负荷过大，以及打包后的文件过大导致页面访问变慢。于是，将问题拆分为两个：1、如何处理打包文件；2、如何优化网站访问速度。

### 如何处理打包文件？

```
反馈就像是灯塔，能指引你不断调整方向。
```
反馈能让我们知道自己的每一步优化操作是否产生作用。

在处理前，问自己一个问题：是否有个工具能让我们实时看到打包文件的大小呢？所幸，有一个叫[webpack-bundle-analyzer](https://github.com/webpack-contrib/webpack-bundle-analyzer)的webpack插件就是做这个的。

`npm install webpack-bundle-analyzer --save-dev` 后，在webpack配置的plugin中添加这个插件，之后每次`npm start`、`npm run build`它便会在默认8888端口开启一个服务，浏览器127.0.0.1:8888访问就能在页面上看到每个打包文件的组成和大小。

问题来了，create-react-app默认将webpack封装在node_modules的react-scripts里面，如果你不想使用eject命令将webpack配置文件暴露出来(想随时跟随react-scripts的更新)又如何修改webpack配置呢？

#### reate-react-app如何不使用eject命令修改默认webpack配置？<br>
```
凡是合理的，你所需要的通常都有人做了，你所想的通常都有人想过。
就像作为工具的螺丝钉是普遍的存在，你所需要的工具粒度越小，它越可能存在着。
```
有一个工具[react-app-rewired](https://github.com/timarney/react-app-rewired/tree/master/packages/react-app-rewired)就是为解决这个问题而存在的。下面是这个工具的使用方法：<br>
```
第一步：安装依赖
npm install react-app-rewired --save-dev
第二步：在根目录下创建config-overrides.js文件
module.exports = function override(config, env) {
  //do stuff with the webpack config...
  return config;
}
第三步：将package.json里面的scripts里面的react-scripts命令替换成react-app-rewired
"scripts": {
-   "start": "react-scripts start",
+   "start": "react-app-rewired start",
-   "build": "react-scripts build",
+   "build": "react-app-rewired build",
-   "test": "react-scripts test --env=jsdom",
+   "test": "react-app-rewired test --env=jsdom"
}
```
运行react-app-rewired命令会先require react-script中对应webpack配置，这样配置便缓存在require的cache中，然后再用新的配置对象替代require中的cache（会将require了的对象和NODE_ENV传入config.overrides.js对应的函数中并返回新配置对象），接着根据传给react-app-rewired的参数运行react-scripts中对应的start/build/..命令。<br>
#### 如何使用react-app-rewired给webpack添加webpack-bundle-analyzer插件？<br>
相关步骤如下：<br>
```
第一步：下载相关依赖
npm install react-app-rewired webpack-bundle-analyzer --save-dev
第二步：根目录下创建config-overrides.js
const { paths } = require('react-app-rewired')
const path = require('path')
const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin
module.exports = function override(config, env) {
	config.plugins = [
		...config.plugins,
		new BundleAnalyzerPlugin()
	]
  return config
}
```
重新`npm start`后边浏览器就会自动打开一个地址为127.0.0.1:8888的页面显示打包信息。

#### 现在我们已经能够实时查看打包文件信息，接下来要如何对打包做优化呢？

create-react-app已经对build后的文件做了压缩，没必要再进行压缩处理（create-react-app也已经在webpack对moment这样很常用的依赖进行优化处理）。实际上打包文件大是因为默认webpack将我们引入的公共依赖都打包到了同一个js文件，而实际上在加载某个页面的时候可能并不需要用到某些很大的依赖，因此我们需要将包打碎细分成更小的块，打开某页的时候按需加载-懒加载。<br>
#### [如何实现代码分离？](https://webpack.docschina.org/guides/code-splitting/)
webpack官方文档提出了解决方法，在最新的create-react-app也都得到支持(这就是让react-scripts自己维护webpack的好处)，如防止依赖引用重复的splitChunks、动态导入功能。webpack会自动将动态导入的模块分离到一个单独的文件，这种方式使用起来就很灵活了动态引入大的依赖，也可以按需引入路由-路由懒加载。

**如何实现路由懒加载呢？**

你可以实现一个路由懒加载的组件，如下：
```
import React from 'react'
class Bundle extends React.PureComponent {
    componentWillMount() {
        this.load(this.props)
    }
    componentWillReceiveProps(nextProps) {
        if (nextProps.load !== this.props.load) {
            this.load(nextProps)
        }
    }
    load = (props) => {
        this.setState({
            mod: null
        })
        props.load().then((mod) => {
            this.setState({
                mod: mod.default || mod
            })
        })
    }
    render() {
        return this.props.children(this.state.mod)
    }
}
const lazyLoad = loadComponent => props => (
    <Bundle load={loadComponent}>
        { Comp => (Comp ? <Comp {...props} /> : <div>LOADING</div>) }
    </Bundle>
)
export default lazyLoad
```
然后在页面中如：
```
<Route
	path="/about"
	render={
		() => lazyLoad(() => import(/* webpackChunkName: "about" */ '/about'))
	}
</Route>
```
刷新127.0.0.1:8888就可以看到一个前缀带有about的包了，里面包含/about模块及其子模块。

### 如何优化网站访问速度？

现在我们刷新网页只会加载当前页的代码，大大加快了相应速度。现在还能做怎样的优化呢？

我们还可以对nginx服务器进行设置，使其使用gzip对代码进行压缩、以及启用http/2。

使用http/1协议，做网站优化需要尽量将需要的js、图片、css等文件打包成一个，这样能够减少占用时间长的网络传输时间。但是使用http2在网络传输上做了很大优化，将不常更新的文件分离出来反而能利用缓存增加传输效率。不仅如此http/2会压缩请求标头、解决了队头阻塞问题而且被广泛支持、向后兼容。


#### nginx服务器如何启用gzip?

在/etc/nginx的nginx.conf文件中设置如下：

```
# 开启gzip
gzip on;
# 启用gzip压缩的最小文件，小于设置值的文件将不会压缩
gzip_min_length 1k;
# gzip 压缩级别，1-10，数字越大压缩的越好，也越占用CPU时间，后面会有详细说明
gzip_comp_level 2;
# 进行压缩的文件类型。javascript有多种形式。其中的值可以在 mime.types 文件中找到。
gzip_types text/plain application/javascript application/x-javascript text/css application/xml text/javascript application/x-httpd-php image/jpeg image/gif image/png font/ttf font/otf image/svg+xml;
# 是否在http header中添加Vary: Accept-Encoding，建议开启
gzip_vary on;
# 禁用IE 6 gzip
gzip_disable "MSIE [1-6]\.";
```
设置成功后访问你的网站，打开浏览器控制台，就能在network的response headers里面看到content-encoding: gzip。

#### 如何启用http2？

首先nginx版本是否支持https，安装说明进行操作，设置如下：
```
listen       443 ssl;
listen       [::]:443 ssl http2;
```

### 总结

为了优化网站访问速度，我使用了：
* react-app-rewired修改默认webpack配置
* webpack-bundle-analyzer查看打包文件情况
* 路由懒加载实现代码分离以及路由按需加载
* nginx设置传输gzip压缩
* nginx使用http2传输协议




