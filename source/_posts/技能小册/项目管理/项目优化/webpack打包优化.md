---
title: webpack打包优化?
abbrlink: 7e0eb1ea
date: 2023-11-08 09:05:01
series: 项目优化
categories:
  - 技能小册
tags:
  - 工程化
---

# 优化?

前端的更新迭代是很快的，从最开始的三件套到现在的响应式开发，在到现在的项目管理化；

于是前端又得学会工程化以及对项目优化的知识，最开始的前端是没有工程这一技能的， 就是把简单的资源区分目录，再从不同的目录调取不同的资源文件;

但是暴露出有很多问题，就好比资源怎么重复利用，每次打开页面资源又要重新加载，而不是读取缓存。这样的情况会导致浏览器的卡顿，影响用户体验。

再者就是代码风格，起初哪有风格就是各写各的，对于团队式开发一直是个头疼的问题；

之后就出现了各式各样的工程化，比如：代码格式化的风格，检验代码的格式，代码压缩与混淆等等；

这样的一个好处就是做到统一，之后的开发全都是以这个为标准；关于如何规范化搭建项目可以参考之前的文章[Vue 项目搭建](/brochure/project/product/init_vue)

通过`webpack5`搭建的`vue3`的项目如何优化呢？ 这里具体说明打包优化; 一些优化的方案可参考[从哪些方面进行性能优化](/brochure/project/direction/direction)

## 简单了解 Webpack

`Webpack` 是一个前端资源打包工具，可以按照模块依赖关系打包项目，将不同模块的依赖关系打包在一起，最终生成一个或多个静态资源，如：js、css、图片等。

`Webpack`就是区分不同的模块，例如：项目打包的入口文件是什么，将打包出来的文件放在哪里，本地服务的配置以及对代码的拆分；其实`webpack`没有什么特殊的意义，可以理解的就是项目底层的建筑，只有吧项目底层搭建完善之后，才能更好的去开发项目。

项目中有一些特定的插件，比如混合式开发`h5`需要查看`networker`，那就需要安装对应版本的`vconsole`；这样的话不管您是本地开发还是其他环境测试都是一个不错的选择；

`Webpack` 的核心功能是：

1. 模块化：将项目拆分成小的模块，每个模块只包含一个功能，方便管理。
2. 模块化加载：通过模块化加载，可以减少请求数量，提高页面加载速度。
3. 模块化编译：通过模块化编译，可以减少代码体积，提高运行速度。

优化范围：

1. 缩小打包范围：缩小打包范围，可以减少打包体积。
2. 优化打包速度：优化打包速度，可以减少打包时间。
3. 优化打包体积：优化打包体积，可以减少打包体积。

最初的项目就是这样的目录:

```js
js/*
css/*
html/*
...
```

但是现在的项目是比较复杂的，引入的资源也是很多的，所以需要将这些资源进行分类，然后进行打包。最后通过打包工具打包出来的问题就类似于最初的项目;

```js
static / css;
static / js;
static / vue;
index.html;
```

## 初始化项目

我们拿`vue`来讲, 先要在全局安装`@vue/cli`, 后面我们会说到为什么要安装全局;

```sh
npm i -g @vue/cli
```

安装成功之后，我们创建一个简单的项目:

```sh
vue create 项目名称
```

我们选择`vue3`进行项目测试打包; 安装成功之后`install` 完成之后使用`vue ui`可以通过可视化面板导入项目，这样的化就可以查看自己项目安装的依赖以及对依赖进行更新的操作;

效果如下：

![ui](https://wangxiaoze-view.github.io/picx-images-hosting/images/Snipaste_2024-04-30_16-48-15.png)

点开`任务`：

![任务](https://wangxiaoze-view.github.io/picx-images-hosting/images/Snipaste_2024-04-30_16-49-34.png)

这里我们就能看到项目的一些资源大小，然后通过`webpack`配置针对于这些依赖项目进行优化;

## 优化

首先要先知道`public/index.html`文件中的`<title><%= htmlWebpackPlugin.options.title %></title>`是怎么来的；

对最开始的版本是需要安装`htmlWebpackPlugin`对应的依赖，然后在`webpack.config.js`中配置`htmlWebpackPlugin`，然后就可以在`public/index.html`中通过`<%= htmlWebpackPlugin.options.title %>`来获取`title`的值了；[html-webpack-plugin](https://www.npmjs.com/package/html-webpack-plugin)

```js
const HtmlWebpackPlugin = require("html-webpack-plugin");
{
	plugins: [
		new HtmlWebpackPlugin({
			title: "My App", // 标题
			filename: "assets/admin.html", // 文件
		}),
	];
}
```

## 如何设置 title

但是对于高版本就不需要特别安装插件了；想要设置`title`有俩种方案: 在`vue.config.js`中配置

### 方案一

```js
module.exports = defineConfig({
 transpileDependencies: true,
 pages: {
  index: {
   entry: "src/main.js",
   title: "测试项目", // 这里就是最终的title
  },
 },
}
```

### 方案二

```js
{
	chainWebpack: config => {
		config.plugin("html").tap(args => {
			args[0].title = "测试项目";
			return args;
		});
	};
}
```

这样您就可以定制化标题了;

## 设置打包目录以及静态资源目录

对于配置, 打包目录默认的是`dist`, 静态目录默认的是`static`; 当然您可以自定义

```js
{
  // 输出目录
 outputDir: "dist",
 // 静态资源目录
 assetsDir: "static",
}
```

## 简单区分 chainWebpack 和 configureWebpack

1. `configureWebpack`: 通过操作对象的形式，来修改默认的 webpack 配置，该对象将会被 webpack-merge 合并入最终的 webpack 配置
2. `chainWebpack` 通过链式编程的形式，来修改默认的 webpack 配置

```js
// configureWebpack 形式
{
  configureWebpack:{
    resolve: {
      // 别名配置
      alias: {
        'assets': '@/assets',
        'common': '@/common',
        'components': '@/components',
        'network': '@/network',
        'configs': '@/configs',
        'views': '@/views',
        'plugins': '@/plugins',
      }
    }
  },
}
```

```js
// chainWebpack形式
{
	chainWebpack: config => {
		config.resolve.alias
			.set("@", resolve("src"))
			.set("@v", resolve("src/views"))
			.set("@c", resolve("src/components"))
			.set("@u", resolve("src/util"))
			.set("@h", resolve("src/hooks"));
	};
}
```

这里我们通过对象的形式对别名进行配置，当然也可以使用数组的形式，但是数组的形式需要我们自己进行配置；也是通过对象的形式去分包

## 分包优化

我们的项目用的有`ui组件, lodash一些特定的工具，还有axios, vuex或者pinia`等等；但是对于一些插件是很大的，包括自己再写页面的时候没有注意优化的思维那么后期维护起来绝对是很痛苦的；

简单的说分包就是 将一些`install`的依赖进行分包，比如`axios`，`vuex`，`pinia`等等；打包出来的文件如：`chunk-axios.js`，`chunk-vuex.js`，`chunk-pinia.js`等等；这样的话我们就知道这些文件都是什么文件了, 而不是`common.js, vendor.js`；

```js
{
  configureWebpack: {
    optimization: {
      moduleIds: "deterministic",
      runtimeChunk: "single",
      minimize: true,
      splitChunks: {
        // 分割所有类型的chunk（包括异步和同步）
        chunks: "all",
        // 最小提取文件大小（默认值）
        minSize: 20000,
        // maxSize: 0,
        // 需在两个模块中共享才进行拆分
        minChunks: 2,
        // 最大异步请求并发数（默认值）
        maxAsyncRequests: 5,
        // 最大初始化请求并发数（默认值）
        maxInitialRequests: 3,
        // 缓存组
        cacheGroups: {
        // vendor组 存放node_modules下的chunk
        vendor: {
          // 匹配node_modules下所有的chunk
          test: /[\\/]node_modules[\\/]/,
          name(module) {
          let packageName = "vendors";
          const reg = /[\\/]node_modules[\\/](.*?)([\\/]|$)/;
          if (reg.test(module.context)) {
            packageName = module.context.match(reg)[1];
          }
          // 最后以 chunk-lodash 命名
          return `chunk-${packageName.replace("@", "")}`;
          },
          // 优先级10 优先将node_modules下的chunk拆分到vendor组
          priority: 10,
          // 重用模块，而不是重新生成
          reuseExistingChunk: true,
          // 强制拆分
          enforce: true,
        },
        // 默认组 非node_modules下的文件块 将执行default缓存组规则
        default: {
          // 重用模块，而不是重新生成
          reuseExistingChunk: true,
          // 优先级 -10
          priority: -10,
          // 强制拆分
          enforce: true,
        },
        },
      },
    }
  }
}
```

这样的话一些基础的公共模块就会被抽离出来， 这样在打包的时候， 就不会生成多个公共模块了。 具体的参数配置可参考文档 [webpack](https://webpack.docschina.org/)

## 插件

但是做到这里还是不够的，就比如打包出来的文件进行`gz`压缩，组件按需加载等等;

```js
const AutoImport = require("unplugin-auto-import/webpack");
const Components = require("unplugin-vue-components/webpack");
const { ElementPlusResolver } = require("unplugin-vue-components/resolvers");
const BundleAnalyzerPlugin = require("webpack-bundle-analyzer").BundleAnalyzerPlugin;
const productionGzipExtensions = ["js", "css"];
{
  configureWebpack: {
    // 插件
    plugins: [
    // 具体查看element-plus文档
    AutoImport({
      resolvers: [ElementPlusResolver()],
    }),
    Components({
      resolvers: [ElementPlusResolver()],
    }),
    // 压缩
    new CompressionWebpackPlugin({
      algorithm: "gzip",
      test: new RegExp("\\.(" + productionGzipExtensions.join("|") + ")$"),
      threshold: 10240,
      deleteOriginalAssets: false, // 不删除源文件
      minRatio: 0.8,
    }),
    // 构建预览， 打包分析
    new BundleAnalyzerPlugin(),
    ],
  }
}
```

## 忽略打包的依赖

```js
const cdn = {
 css: ["https://cdn.bootcdn.net/ajax/libs/element-plus/2.7.2/index.min.css"],
 js: [
  // vue
  "https://cdn.bootcdn.net/ajax/libs/vue/3.2.13/vue.global.min.js",
  // ele
  "https://cdn.bootcdn.net/ajax/libs/element-plus-icons-vue/2.3.1/global.iife.min.js",
  "https://cdn.bootcdn.net/ajax/libs/element-plus/2.7.2/index.full.min.js",
  // lodash
  "https://cdn.bootcdn.net/ajax/libs/lodash.js/4.17.21/lodash.min.js",
  // router
  "https://cdn.bootcdn.net/ajax/libs/vue-router/4.3.2/vue-router.global.min.js",
 ],
};

{
  configureWebpack: {
    externals: {
      // 将一些体积比较大的包拆出来，以cdn的链接引入， 这样减少打包体积
      vue: "Vue",
      "element-plus": "ElementPlus",
      "@element-plus/icons-vue": "ElementPlusIconsVue",
      "lodash-es": "_",
      "vue-router": "VueRouter",
    }
  }
}
```

接着在`public/index.html`配置 cdn

```js
<% for (var i in htmlWebpackPlugin.options.cdn && htmlWebpackPlugin.options.cdn.css) { %>
  <link href="<%= htmlWebpackPlugin.options.cdn.css[i] %>" rel="stylesheet" />
<% } %>


<% for (var i in htmlWebpackPlugin.options.cdn && htmlWebpackPlugin.options.cdn.js) { %>
  <script src="<%= htmlWebpackPlugin.options.cdn.js[i] %>" ></script>
<% } %>
```

## 配置 devserver

```js
{
  devServer: {
    open: true,
    host: "localhost",
    port: 8080,
    https: false,
    client: {
      // 允许在浏览器中设置日志级别，默认是普通的提示
      logging: "info",
      // 当出现编译错误或警告时，在浏览器中显示全屏覆盖
      overlay: true,
      // 在浏览器中以百分比显示编译进度； 打开控制台就可以看见
      progress: true,
      // 限次尝试重新连接
      reconnect: true,
    },
  },
}
```

到这里一个简单的`webpack`打包优化就完成了；我们通过`vue ui`打包看一下;

![build](https://wangxiaoze-view.github.io/picx-images-hosting/images/Snipaste_2024-04-30_17-18-15.png)

这里的我们就可以看出压缩前与压缩后的文件大小对比，部署项目也可以轻松的实现`CDN`加速， 优化打包速度， 减少服务器压力， 提升用户体验。那这样的是不是可以做到瞬间打开页面呢？

![success](https://wangxiaoze-view.github.io/picx-images-hosting/images/Snipaste_2024-04-30_17-20-15.png)

![success](https://wangxiaoze-view.github.io/picx-images-hosting/images/Snipaste_2024-04-30_17-21-36.png)

然后我们可以看出一个项目最后所用到的资源大小；

## 最终测试代码

```js
const { defineConfig } = require("@vue/cli-service");
const CompressionWebpackPlugin = require("compression-webpack-plugin");
const AutoImport = require("unplugin-auto-import/webpack");
const Components = require("unplugin-vue-components/webpack");
const { ElementPlusResolver } = require("unplugin-vue-components/resolvers");
const BundleAnalyzerPlugin =
	require("webpack-bundle-analyzer").BundleAnalyzerPlugin;

const path = require("path");

// const isProduction = process.env.NODE_ENV === "production";

const cdn = {
	css: ["https://cdn.bootcdn.net/ajax/libs/element-plus/2.7.2/index.min.css"],
	js: [
		// vue
		"https://cdn.bootcdn.net/ajax/libs/vue/3.2.13/vue.global.min.js",
		// ele
		"https://cdn.bootcdn.net/ajax/libs/element-plus-icons-vue/2.3.1/global.iife.min.js",
		"https://cdn.bootcdn.net/ajax/libs/element-plus/2.7.2/index.full.min.js",
		// lodash
		"https://cdn.bootcdn.net/ajax/libs/lodash.js/4.17.21/lodash.min.js",
		// router
		"https://cdn.bootcdn.net/ajax/libs/vue-router/4.3.2/vue-router.global.min.js",
	],
};

const externals = {
	vue: "Vue",
	"element-plus": "ElementPlus",
	"@element-plus/icons-vue": "ElementPlusIconsVue",
	"lodash-es": "_",
	"vue-router": "VueRouter",
};

const productionGzipExtensions = ["js", "css"];

module.exports = defineConfig({
	transpileDependencies: true,
	pages: {
		index: {
			entry: "src/main.js",
			title: "测试项目",
			// cdn: isProduction ? cdn : {},
			cdn,
		},
	},
	// 输出目录
	outputDir: "dist",
	// 静态资源目录
	assetsDir: "static",
	// 是否开启eslint保存检测
	lintOnSave: true,
	// 是否在构建生产包时生成 sourceMap 文件，false将提高构建速度
	productionSourceMap: false,

	// 链式编程 修改默认的webpack配置
	chainWebpack: () => {
		// config.plugin("html").tap(args => {
		//  args[0].title = "测试项目";
		//  return args
		// });
	},

	// 对象的形式操作 webpack配置
	configureWebpack: {
		// 别名
		resolve: {
			alias: {
				"@": path.resolve(__dirname, "src"),
			},
		},
		devtool: "inline-source-map",

		// 分包优化
		optimization: {
			moduleIds: "deterministic",
			runtimeChunk: "single",
			minimize: true,
			splitChunks: {
				chunks: "all", // 分割所有类型的chunk（包括异步和同步）
				minSize: 20000, // 最小提取文件大小（默认值）
				// maxSize: 0,
				minChunks: 2, // 需在两个模块中共享才进行拆分
				maxAsyncRequests: 5, // 最大异步请求并发数（默认值）
				maxInitialRequests: 3, // 最大初始化请求并发数（默认值）
				cacheGroups: {
					// vendor组 存放node_modules下的chunk
					vendor: {
						test: /[\\/]node_modules[\\/]/, // 匹配node_modules下所有的chunk
						name(module) {
							let packageName = "vendors";
							const reg = /[\\/]node_modules[\\/](.*?)([\\/]|$)/;
							if (reg.test(module.context)) {
								packageName = module.context.match(reg)[1];
							}

							console.log(module.context, "==========");
							// 最后以 chunk-lodash 命名
							return `chunk-${packageName.replace("@", "")}`;
						},
						// name: "chunk-vendor",
						priority: 10, // 优先级10 优先将node_modules下的chunk拆分到vendor组
						reuseExistingChunk: true, // 重用模块，而不是重新生成
						enforce: true, // 强制拆分
					},
					// 默认组 非node_modules下的文件块 将执行default缓存组规则
					default: {
						reuseExistingChunk: true, // 重用模块，而不是重新生成
						priority: -10, // 优先级 -10
						enforce: true, // 强制拆分
					},
				},
			},
		},
		// 插件
		plugins: [
			AutoImport({
				resolvers: [ElementPlusResolver()],
			}),
			Components({
				resolvers: [ElementPlusResolver()],
			}),
			// 压缩
			new CompressionWebpackPlugin({
				algorithm: "gzip",
				test: new RegExp("\\.(" + productionGzipExtensions.join("|") + ")$"),
				threshold: 10240,
				deleteOriginalAssets: false, // 不删除源文件
				minRatio: 0.8,
			}),
			// 构建预览
			new BundleAnalyzerPlugin(),
		],

		// 忽略的依赖
		// externals: isProduction ? externals : {},
		externals,
	},

	// 服务
	devServer: {
		open: true,
		host: "localhost",
		port: 8080,
		https: false,
		client: {
			// 允许在浏览器中设置日志级别，默认是普通的提示
			logging: "info",
			// 当出现编译错误或警告时，在浏览器中显示全屏覆盖
			overlay: true,
			// 在浏览器中以百分比显示编译进度
			progress: true,
			// 限次尝试重新连接
			reconnect: true,
		},
	},
});
```

```html
<!DOCTYPE html>
<html lang="">
	<head>
		<meta charset="utf-8" />
		<meta http-equiv="X-UA-Compatible" content="IE=edge" />
		<meta name="viewport" content="width=device-width,initial-scale=1.0" />
		<link rel="icon" href="<%= BASE_URL %>favicon.ico" />
		<title><%= htmlWebpackPlugin.options.title %></title>

		<% for (var i in htmlWebpackPlugin.options.cdn &&
		htmlWebpackPlugin.options.cdn.css) { %>
		<link href="<%= htmlWebpackPlugin.options.cdn.css[i] %>" rel="stylesheet" />
		<% } %> <% for (var i in htmlWebpackPlugin.options.cdn &&
		htmlWebpackPlugin.options.cdn.js) { %>
		<script src="<%= htmlWebpackPlugin.options.cdn.js[i] %>"></script>
		<% } %>
	</head>
	<body>
		<noscript>
			<strong
				>We're sorry but <%= htmlWebpackPlugin.options.title %> doesn't work
				properly without JavaScript enabled. Please enable it to
				continue.</strong
			>
		</noscript>
		<div id="app"></div>
		<!-- built files will be auto injected -->
	</body>
</html>
```

## 效果图

![result](https://wangxiaoze-view.github.io/picx-images-hosting/images/Snipaste_2024-04-30_17-24-45.png)
