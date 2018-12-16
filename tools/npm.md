# NPM

你已经使用Git用于网站版本管理，这时候要开始准备编写代码了。在命令行的下一输入便是`npm init`，这个命令会引导你初始化一个叫package.json的文件，`npm init -y`会直接生成默认的package.json文件。在package.json文件中记录了网站所需要点依赖的列表及其版本以及其他配置信息。

npm是什么？
> npm是可以管理本地项目的所需模块并自动维护依赖情况的工具。如果一个项目中存在package.json文件，那么用户可以直接使用npm install命令自动安装和维护当前项目所需的所有模块。开发者还可以指定每个依赖版本范围。

为什么要使用npm？
> 可以先回答另一个问题：如果不使用依赖管理工具，我们怎样引入依赖？
>
> 按比较早的做法，我们要使用jQuery库会在页面中加入一个src为jQuery某个版本路径的script标签，我们的网站可能还会这样引入富文本编辑器、动画组件等等。当然，在依赖不多的情况下我们可以手动一个个查询最新依赖版本链接并用script插入，但是一旦依赖很多，再加上你所引用的依赖很可能会更新（比如添加新特性、修复bug），这时候去一个个去修改script中的src便是很麻烦的事。如果这时有个能帮我们管理依赖的工具，在配置文件中列出你想要的依赖然后install一下，就能获取最新依赖版本，网页引入时只需要依赖名而不涉及版本号，有新版本更新也直接install一下就好，岂不是很方便。npm就是这样的工具。

如何使用npm？
> 你可以使用npm添加、删除依赖，管理依赖版本。你还可以用它发布依赖。npm管理包常用命令：
>
```
npm install 包名：为当前正在开发的包新增一个依赖包；--save-dev 将包添加到devDependencies列表
npm init：初始化包；
npm install：安装package.json 文件里定义的所有依赖包；
npm publish：发布一个包到包管理器；
npm uninstall：从当前包里移除一个未使用的包。
```
> 有时候npm install会比较慢，这时可以` npm install -g cnpm --registry=https://registry.npm.taobao.org`安装淘宝镜像 cnpm，使用cnpm下载依赖会快很多，命令也与npm一致。

你还可以使用yarn作为npm的替代。yarn的常用命令：

```
yarn add：为当前正在开发的包新增一个依赖包；
yarn init：初始化包；
yarn install：安装package.json 文件里定义的所有依赖包；
yarn publish：发布一个包到包管理器；
yarn remove：从当前包里移除一个未使用的包。
```