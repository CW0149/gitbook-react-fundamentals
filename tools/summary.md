# 总结

现在来重头看一遍运行过的命令：

```
$ mkdir mySite-fe && cd mySite-fe
$ git init
$ vim .gitignore
$ npm init -y
$ mkdir src && cd src && touch index.html index.js index.css && cd ..
$ npm install react react-dom react-router-dom
$ npm install webpack webpack-cli webpack-dev-server html-webpack-plugin babel-loader @babel/core @babel/preset-env @babel/preset-react css-loader style-loader --save-dev
$ vim package.json
$ vim webpack.config.js
$ npm start / npm run build
```
你可以把上面这些命令按需求定制打包成脚本，以后新建项目可以直接运行脚本来创建自己的个性化项目。

package.json配置示例：

```
"babel": {
  "presets": [
    "@babel/preset-env",
    "@babel/preset-react"
  ]
},
"scripts": {
  "build": "webpack",
  "start": "webpack-dev-server --open"
},
```

webpack.config.js配置示例：

```
const path = require('path')
const HtmlWebpackPlugin = require('html-webpack-plugin')

module.exports = {
	entry: './app/index.js',
	output: {
		path: path.resolve(__dirname, 'dist'),
		filename: 'index_bundle.js'
	},
	module: {
		rules: [
			{ test: /\.(js)$/, use: 'babel-loader' },
			{ test: /\.(css)$/, use: ['style-loader', 'css-loader'] },
		]
	},
	devServer: {
	    contentBase: './dist'
  },
	mode: 'development',
	plugins: [
		new HtmlWebpackPlugin({
			template: 'app/index.html'
		})
	]
}
```