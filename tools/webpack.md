# Webpack、Babel

你已经使用Git管理网站版本，使用npm管理网站依赖。你现在要开始写代码，想使用es6语法，你还想部署到生产环境的文件是打包好的，然后觉得用less语法写css也不错...。那么，如何方便的做到这些呢？Webpack就是帮助你方便实现这些的工具。现在先在根目录下新建src文件夹，然后在里面新建index.html、index.js、index.css。

### Webpack
Webpack是什么？

> webpack 是一个现代 JavaScript 应用程序的静态模块打包器(static module bundler)。在 webpack 处理应用程序时，它会在内部创建一个依赖图(dependency graph)，用于映射到项目需要的每个模块，然后将所有这些依赖生成到一个或多个bundle。<br>换言之，webpack就像是一个装配工厂，能够按照装配图将各零件处理、组合、打包成成品。不仅如此，它还提供了如热更新等功能方便你开发调试。

如何使用Webpack？

> 你需要引入webpack依赖包：`npm install webpack webpack-cli --save-dev`，然后给package.json文件的scripts添加如 "build": "webpack"，在命令行输入 `npm run build`就可以运行webpack命令。不过，直接运行会报错，你需要先在根目录下新建一个webapck.config.js文件，这是webpack默认配置文件，这个文件配置示例如下：<br>
```
var path = require('path');
var HtmlWebpackPlugin = require('html-webpack-plugin');
module.exports = {
	entry: './src/index.js',
	output: {
		path: path.resolve(__dirname, 'dist'),
		filename: 'index_bundle.js'
	},
	module: {
		rules: [
			{ test: /\.(js)$/, use: 'babel-loader' },
			{ test: /\.(css)$/, use: ['style-loader/url', 'file-loader'] }
		]
	},
	mode: 'development',
	plugins: [
		new HtmlWebpackPlugin({
			template: './src/index.html'
		})
	]
}
```

[Webpack几个核心概念](https://webpack.docschina.org/concepts)：

>作为一个工具webpack需要输入，也会有输出。在entry字段中定义输入，输出则定义在output字段中。输入可能有很多类型，可在webpack中引入loader将这些类型转换为浏览器识别的语言类型。实际项目中可能还需要进行优化等操作，webpack的plugin字段能够让我们进行如打包优化、资源管理和注入环境变量等操作。<br>
* 入口(entry)
	* entry指示了构建(内部依赖图)的起点文件，默认值是./src/index.js。
* 输出(output)
	*  output定义了webpack 在哪里输出它所创建的 bundles，以及如何命名这些文件，主输出文件默认为 ./dist/main.js，其他生成文件的默认输出目录是 ./dist。
* loader
	* 定义在module的rules字段里面。webpack 自身只支持 JavaScript。而 loader 能够让 webpack 处理那些非 JavaScript 文件，并且先将它们转换为有效 模块，然后添加到依赖图中。
	* [loader列表](https://webpack.docschina.org/loaders/)
* 插件(plugins)
	* loader 被用于转换某些类型的模块，而插件则可以用于执行范围更广的任务，插件的范围包括：打包优化、资源管理和注入环境变量。
* 模式(mode)
	* 通过将 mode 参数设置为 development, production 或 none，可以启用对应环境下 webpack 内置的优化。默认值为 production。

### Babel

要转译es6语法，你需要Babel。Babel能将你写的最新es6语法转译成浏览器支持的版本。当然Babel功能很强大，不止可以转译es6/7语法，还能转译react中使用的jsx语法。结合webpack使用，则需要安装babel-loader。下面看下命令行输入：

```
npm install @babel/core @babel/preset-env @babel/preset-react --save-dev  // @babel/core是babel核心代码，@babel/preset-env能转译最新js语法，@babel/preset-react能转译jsx语法。
```
安装转译所需Babel依赖后，需要配置Babel。如上webpack配置示例，在webpack.config.js中，添加loader `{ test: /\.(js)$/, use: 'babel-loader' }`，表示将所有.js结尾的文件使用[babel-loader](https://webpack.docschina.org/loaders/babel-loader/)转译，具体使用babel中哪种转译，可以在loader的options字段中定义，如下：

```
module: {
  rules: [
    {
      test: /\.js$/,
      exclude: /(node_modules|bower_components)/,
      use: {
        loader: 'babel-loader',
        options: {
          presets: ['@babel/preset-env', '@babel/preset-react'],
          plugins: [require('@babel/plugin-transform-object-rest-spread')]
        }
      }
    }
  ]
}
```
除此之外，也可以在package.json中添加babel字段：

```
"babel": {
	"presets": [
		"@babel/preset-env",
		"@babel/preset-react"
	]
}
```

### style-loader、css-loader

[style-loader](https://webpack.docschina.org/loaders/style-loader/)与[css-loader](https://webpack.docschina.org/loaders/css-loader/)是webpack中常用的两个loader。不像babel-loader需要配置使用哪种转译方法，style-loader和css-loader只需要`npm install style-loader css-loader --save-dev`，然后在webpack.config.js的rules中添加如`{ test: /\.(css)$/, use: ['style-loader', 'css-loader'] }`就行。

style-loader能够将js中引入的css文件添加到DOM的style标签中。选择style-loader/url，配合file-loader使用，也可以添加一个URL`<link href="path/to/file.css" rel="stylesheet">`而非内联css标签。

css-loader则将 @import和 url() 解释为import/require() 并解析它们。示例如下：

```
@import 'style.css' => require('./style.css')
@import url(style.css) => require('./style.css')
@import url('style.css') => require('./style.css')
@import './style.css' => require('./style.css')
@import url(./style.css) => require('./style.css')
@import url('./style.css') => require('./style.css')
@import url('http://dontwritehorriblecode.com/style.css') => @import url('http://dontwritehorriblecode.com/style.css') in runtime
```

### HtmlWebpackPlugin

运行`npm run build`，我们使用webpack配合babel-loader、style-loader、css-loader将index.js转译打包到了index_bundle.js，index_bundle.js就是网站的主js文件。但是好像还缺少点什么-index.html。在src中的index.html是网站入口页面。但是此时我们的index.html文件还在src文件夹中，js文件却在dist目录下，index.html引入的js文件也还是index.js，难道每次webpack后都要将index.html拷贝到dist并修改文件中js引入路径？

再看webpack的[plugin](https://webpack.docschina.org/api/plugins)功能：插件可以用于执行范围更广的任务，能够 钩入(hook) 到在每个编译(compilation)中触发的所有关键事件。html-webpack-plugin便是能够帮助我们解决上述问题的webpack插件。

安装[html-webpack-plugin](https://github.com/jantimon/html-webpack-plugin)：`npm install html-webpack-plugin --save-dev`。在webpack.config.js的plugins中添加如下:

```
plugins: [
	new HtmlWebpackPlugin({
		template: './src/index.html'
	})
]
```
参数中template字段定义了模版html，这个插件能够读取webpack配置将template文件拷贝到输出文件夹并更改里面资源的引入路径。

### webpack-dev-server

webpack-cli提供了在命令行中使用webpack的方法，你可以在package.json中添加scripts命令如：

```
"scripts": {
	"build": "webpack",
	"start": "webpack -w"
}
```
执行webpack命令会一次性将项目代码打包到配置的dist目录下，如果我们想修改文件后自动打包到dist要怎么做？webpack命令接受接收一个 -w/--watch 参数，在执行`webpack -w`后，webpack不会结束而会一直监听依赖图中的所有文件变化，如果其中一个文件被更新，代码将被重新编译更新到dist。

不过，本地开发毕竟不是生产环境，每次都重新打包到dist在项目比较大的情况下速度会很慢，而且每次重新打包后还要手动刷新页面好麻烦。有没有适合生产环境高效开发的方法？[webpack-dev-server](https://webpack.docschina.org/guides/development)就是解决这个问题的工具。

`npm install webpack-dev-server --save-dev`后，编辑package.json的scripts：

```
"scripts": {
	"start": "webpack-dev-server --open"
}
```

`npm start`后会自动开启一个web服务，打开浏览器，监听文件的变化。由于webpack-dev-server实时编译的文件是放在内存中，所以速度更快，并且更新代码后浏览器会自动刷新页面，这就是所谓的热更新。与此同时，webpack-dev-server还带有许多其他[可配置项](https://webpack.docschina.org/configuration/dev-server)。


