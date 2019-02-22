# zenbone（一个全自动的前端构建工具）

本版本是基于[zenbone](https://www.npmjs.com/package/zenbone)的升级版本，兼容以前版本的zenbone命令。

----

**注意：本工具的部分功能和公司网络强绑定，外部不可使用**

**建议：Node.js版本 > v6.0.0**

## 一，概述

**要点**

- 基于[webpack](https://webpack.github.io/)
- 模块化管理、打包工具
- 集成脚手架工具，适用于创建项目或组件
- HTML打包，CDN资源路径替换
- 基于模板自动配置webpack.config
- 强大的自定义模板功能
- 组件安装、卸载、批处理
- 集成Gitlab-API，命令查看模板组件列表
- 集成Jenkins任务，一条命令部署stage、product环境

**优势：**

- 代码向后兼容——完全遵守webpack编码规范，让我们更加方便使用ES6（新一代Javascript编程规范）
- 高性能——使用webpack编译工具，编译性能高，开发体验佳
- 生态圈繁荣——可快速使用[NPM](https://www.npmjs.com/)海量JS组件 & [webpack插件](http://webpack.github.io/docs/list-of-plugins.html)
- 化繁为简——使用脚手架工具，可帮助我们跳过繁琐的webpack配置，及其他相关学习成本，快速进入项目开发状态
- 构建发布为一体，减少操作时间

**功能预览：**
```
    Usage: zenbone <command> [options]
    
    Options:
      -v, --version              output the version number
      -p, --port [number]        选择一个端口启动调试服务器 (default: 80)
      -s, --stage                构建或部署当前代码为测试(stage)环境
      -P, --product              构建或部署当前代码为生产(product)环境
      -S, --split                将语言包拆成一个单独的文件而不是打包在项目的js中，有助于单独拉取文案
      -M, --multi                是否导出为多个语言文件
      -L, --splitmulti           多语言入口文件&根据语言拆成多个单独的包，--split和--multi命令的合并
      -I, --src [string]         源文件夹路径，默认"js"，生成多语言使用
      -O, --output [string]      目标文件夹路径，默认"js/lang"，生成多语言使用
      -h, --help                 output usage information
    
    Commands:
      init                       使用脚手架初始化一个基于模板的项目，默认normal模板
      start                      启动调试服务器(sudo webpack-dev-server)
      sprite                     生成精灵图
      build                      构建项目到stage或者product环境
      deploy [configFile]        调用Jenkins构建项目，支持stage,product环境
      lang <action>              生成多语言的key，或者拉取多语言的文案
      widget|component [action]  对组件进行操作，可选操作action（init, list, build）
      template                   列出服务器上的所有的模板
      pull [name]                在远程的服务器上下载一个或多个模板
      install [name]             【已过时】将远程服务器上的组件下载到本地并添加到项目的依赖中
      uninstall [name]           【已过时】从本地node_modules中删除小组件并从项目依赖中移除
```

## 二，安装维护
### 安装

    npm install zenbone -g

### 更新

    npm update zenbone -g

若更新失败，可指定最新版本号重新安装，`npm install zenbone@x.y.z -g`

## 三，项目开发

### 1，创建项目

    mkdir test
    cd ./test
    zenbone init [template] # template 不传，默认是normal

创建项目结构如下：

	test
 	  | -- css
 	  	 | -- reset.css		# reset
 	  	 | -- whatever.css	# 项目级css
 	  | -- js
 	  	 | -- app.js	# js入口
 	  | -- images
 	  | -- index.html	# html入口
 	  | -- package.json		# 项目配置
 	  | -- webpack.config.js	# webpack配置文件
 	  | -- .gitignore

zenbone对`css、js、images`文件夹没有要求，可以任意存放，JS模块无论引用CSS模块还是图片，均使用相对路径，只要保证路径正确即可。


### 2，启动本地环境

启动本地静态服务器

	zenbone start
	# 或者使用
	sudo webpack-dev-server

执行上述命令，做了以下几件事情：

- 检测本机是否安装`webpack`及`webpack-dev-server`，若未安装，则自动安装;
- 检测是否已安装项目依赖，若未安装，则自动安装;
- 启动静态服务器，macos用户和linux用户需要输入密码;

首次使用`zenbone start`启动环境，等待时间通常会比较长，其主要时间消耗安装工具依赖及项目依赖上，亦可逐次执行上述操作。

	sudo npm install webpack@3.5.5 webpack-dev-server@1.14.1 -g
	npm install
	zenbone start

如果依赖下载过慢，可以访问国内的[npm镜像服务器](https://registry.npm.taobao.org), 目前我们内部已经搭建的私有仓库，可以直接使用私有仓库, 设置方式如下

    npm config set registry https://registry.npm.taobao.org

在浏览器键入: `localhost`即可查看index.html相关内容。由于在webpack.config.js中配置了watch功能，当文件发生变化后，系统将自动刷新浏览器。

### 3，使用组件

组件通常分为`通用组件`和`业务组件`。

通用组件，指和业务逻辑无关的组件、模块，如react、dialog等，若其使用npm系统管理，则直接通过npm安装。

	npm install react --save

【已过时】~~业务组件，通常和业务逻辑结合紧密，并不具备更高通用性，通常发布到业务服务器，则需要通过zenbone命令安装。~~

    # 下面的命令已不建议使用
	zenbone install xxx
	# 或者一次安装多个组件，所有的组件用""包起来，中间使用空格隔开即可
	zenbone install "xxx bbb nnn"

业务组件开发、发布方式，zenbone提供定制方案，详见下文『组件开发』。在业务模块中使用安装的组件:

	import React, {Component} from 'react';

**注意：** 有部分nodejs版本使用组件会有些问题，最好先安装npm组件，再安装zenbone组件。或者每次安装完npm组件，需要重新执行`zenbone install`

### 4，多语言包管理

#### 一键提取多语言包keys
代码中的格式必须包含 `lang.template('xxxx'` 时，xxx会被提取。

	# 支持两个参数
	# src 可以不传，默认是 js
	# output 可以不传，默认是 js/lang
	zenbone lang key --src js --output js/lang

自动在传递的目录（不传就取默认值）创建`keys`文件，JS文件中语言包字段都会被收录，形如：

	偷星
	我的
	商城
	Up主播经纪人
	系统异常
	你确认要花<%=diamond%>购买<%=coin%>吗?
	<%=num%>U钻
	<%=num%>金币
	金币购买成功~
	连续登陆第<%=value%>天奖励

项目经过迭代，再次抽取语言包，自动将新增字段标识出来。

    ==============================新增key（2018-3-1 17:47:36）==============================
    你的GTO不足
    你的账户余额不足完成支付手续费
    ==============================新增key（2018-3-2 10:33:27）==============================
    密码重置成功，可以前往登录

    ==============================新增key（2018-3-2 16:15:41）==============================
    请上传清晰的护照个人信息页照片
    ==============================新增key（2018-3-5 12:03:57）==============================
    重新编辑
    ==============================新增key（2018-3-5 18:37:21）==============================
    请上传小于1.5MB的图片
    姓只能由字母空格组成
    名只能由字母空格组成
    请上传清晰的护照个人信息页照片，建议大小<1.5MB
    稍等，当Txhash成功生成后，你可在Etherscan上查看到此交易
    暂无数据

### 创建本地多语言包文件（Google Spreadsheet）

使用Google Spreadsheet维护多语言配置文件，使用该明了从Google Spreadsheet中导出配置内容，创建本地多语言包js文件。

    # 支持一个参数
	# output 可以不传，默认是 js/lang
	zenbone lang file --output js/lang

package.json配置：

	"googleSpreadsheetId": "", // Google Spreadsheet Id
  	"googleSpreadsheetIndex": 0 // Google Spreadsheet中sheet的索引。

执行	`zenbone lang file`命令创建本地多语言包文件时，会使用`package.json`中配置的

    `googleSpreadsheetId`，`googleSpreadsheetIndex`。

**字段说明：**

* `googleSpreadsheetId`：Google Spreadsheet表格的ID。例如：

		文档链接：https://docs.google.com/spreadsheets/d/1Aiemu_rRqQTmDwr146URYPfB1PDv8ADE9SxrOOA1GUs/edit
		googleSpreadsheetId：1Aiemu_rRqQTmDwr146URYPfB1PDv8ADE9SxrOOA1GUs

* `googleSpreadsheetIndex`：Google Spreadsheet表格中Sheet的索引值。默认0，第一张表格。


**Google Spreadsheet使用步骤：**

* 创建一个Google表格
* 点击菜单：`File`->`Publish to the web.`，发布文件。［必须］
* 选中Published content & settings配置：Automatically republish when changes are made。每次变更都会自动发布文件，更新文件内容。
* 将Google表格的ID配置到项目文件package.json。



### 5，打包

	zenbone build --product		# 生产环境
	zenbone build --stage		# 预发布环境

预发布和生产环境打包之间，仅html中引用资源(css、js)路径不同，其它处理完全一致。

**webpack配置：**

	// 为product环境打包时
	if (env == 'product') {
    	// 定制cdn路径
    	output.publicPath = 'http://cdn/' + projectName + '/' + projectVersion + '/assets/';
	}

	if (env == 'stage') {
     	// 定制cdn路径
    	output.publicPath = '/' + projectName + '/' + projectVersion + '/assets/';
	}

由此配置可看出，项目打包时会使用package.json中`name`、`version`字段作为CDN文件输出的关键路径。

影响范围：

- css\js对图片资源的引用
- html入口对js\css的应用

**打包后项目结构：**

	test
	  | -- build
	  	 | -- assets
	  	 	| -- 880110af781e078951d0d0fda16353f4.png  # 项目图片，打包后js、css将自动使用此名称
	  	 	| -- app.js		# js入口，被打包后html引用
	  	 	| -- app.js.map	# source-map 便于调试
	  	 	| -- app.css	# css，被打包后html引用
	  	 	| -- app.css.map
	  	 	| -- commmons.js	# 多入口公用部分，将打包后html引用
	  	 	| -- commons.js.map
	  	 | -- index.html	# 打包后html入口
 	  | -- css
 	  	 | -- reset.css		# reset
 	  	 | -- whatever.css	# 项目级css
 	  | -- js
 	  	 | -- app.js	# js入口
 	  | -- images
 	  | -- index.html	# html入口
 	  | -- package.json		# 项目配置
 	  | -- webpack.config.js	# webpack配置文件
 	  | -- .gitignore

### 6, 使用jenkins部署到服务器上

    zenbone deploy
    # 线上环境没有配置
    zenbone deploy --product

**注意：**
1. 目前使用的是我的账号进行登录的，可能某一天我的账号就过期了，需要联系维护者进行认证信息的配置。
2. 在构建之前必须要保证已经build并且已经上传，只用此功能必须要保证package.json中的name和git的name保持一致。

## 四，组件开发

### 1，概述

如你所知，zenbone开发以单个项目为单位，即在项目内无法依赖本地项目外的资源模块。

在实际项目中往往存在此种类型的模块，它被用到A、B、C多个项目，那么在zenbone（即webpack）开发模式中，如何管理这些模块就至关重要！

把公用模块发布到npm系统是个选择，但显然不合适。

- 模块含有业务逻辑代码，不够通用
- 不希望把业务代码开源

把所有公用模块存放在本地库，在项目中单独硬拷贝也可满足需求，但人肉操作实为繁琐，也会操作失误率。因此需要另辟蹊径，既能发布组件到业务服务器，又能很方便安装、使用。


### 2，实现原理

组件遵守npm开发规范，在开发完成后上传组件tar包到业务服务器。安装时从业务服务器下载tar包到项目node_modules目录，同时正确处理依赖关系，并下载相关依赖组件，为了避免和npm系统模块冲突，在组件名称前统一添加前缀'c-'，使用方式则可保持和npm系统一致。

### 3，创建组件

	mkdir test
	cd ./test

	zenbone init widget
    # 或者使用老版本的命令执行
    zenbone component init

创建项目结构如下：

	test
 	  | -- src
 	  	 | -- index.js		# 开发时, 组件入口
 	  | -- test.html	# 测试htlm
 	  | -- test.js		# 测试js入口
 	  | -- package.json		# 组件项目配置, 如依赖组件等
 	  | -- webpack.config.js	# webpack配置文件
 	  | -- .gitignore

### 4，启动环境

	zenbone start

### 5，开发组件

编写`src/js`内组件代码，打开`localhost/test.html`进行调试。若组件依赖于其他组件，则通过如下方式进行安装：

	zenbone install componentNameA
    # 安装多个组件
    zenbone install "componentNameA componentNameB componentNameC"

安装完成后，打开项目目录下`node_modules`，会发现增加了componentNameA目录，此时便可在组件内引用。

	import componentNameA from 'componentNameA';

同时，配置文件`package.json`中增加了`_depComponent`字段，形如：

	"_depComponent": [
    	"componentNameA"
  	]

此字段用来标注该组件依赖的组件列表，以保证组件被安装时同时安装相关依赖。假若componentNameA又依赖于componentNameB，则`node_modules`中将增加componentNameA、componentNameB目录，`_depComponent`也会增加对componentNameB的依赖。

	"_depComponent": [
    	"componentNameA",
    	"componentNameB"
  	]

当非原始开发者对组件进行维护时，在下载源代码（通常不包括node_modules）后，通过`zenbone install`便可一键安装所有组件依赖。

若要修改组件安装源，需要先修改package.json中`componentDomain`字段。路径中最后一个`/`不可省略，以保证zenbone可正确找到组件tar包。

	{
		"componentDomain": "http://h5widget.xingyunzhi.cn/"
	}

下面是一个完整的tar包地址，其中name是组件名称。

	// http://h5widget.xingyunzhi.cn/componentNameA/componentNameA.tar.gz
	var url = componentDomain + name + '/' + name + '.tar.gz';


### 6，打包组件

	zenbone widget build
    # 或者使用旧版的命令
    zenbone component build

组件打包后在根目录将增加目录`dist`、`test.tat.gz`。

	test
 	  | -- src
 	  	 | -- index.js		# 开发时, 组件入口
 	  | -- dist
 	  	 | -- index.js		# 组件被安装后, 实际被引用的模块
 	  | -- test.html	# 测试htlm
 	  | -- test.js		# 测试js入口
 	  | -- package.json		# 组件项目配置, 如依赖组件等
 	  | -- webpack.config.js	# webpack配置文件
 	  | -- test.tar.gz		# 安装时下载的tar包
 	  | -- .gitignore

确定打包成功后，把test.tar.gz发布到`_depComponent`对应的服务器即可。

至此，一个完整组件开发、构建、发布的所有工作全部完成！

### 7，模板使用

我们在初始化项目的时候调用了

    zenbone init [templateName]

会出现以下的检查，
1. 先尝试连接服务器，在服务器上拉取该模板，拉去成功后将会存储在$HOME的.zenbone-template，也会开始创建项目。
2. 线上拉取失败，在$HOME的.zenbone-template中查找对应的模板，如果没有会报出找不到模板错误。

那要是我回家了，连不上公司的网络，但是我本地有没有模板怎么办，我们可以使用下面的命令将模板拉取到本地。

    zenbone pull [templateName]
    # 安装多个模板
    zenbone pull "templateA templateB templateC"

查看是否拉取成功，可以到 `~/.zenbone-template`下查看。

## FAQ

`zenbone(webpack)`把JS作为所有模块的入口，和传统开发方式相比有很大不同，初次接触，难免会有一点点不适应。这里列举一些常见问题及解决办法以供参考，后续持续补充。

### 1，css如何引用？

在js模块通过require、import方式依赖引入。

	import '../css/app.css';
	// require('../css/app.css');

经过webpack编译后，会把所有引入的css合并到入口js中，在开发环境不太关注性能，不需要额外处理。
在生产环境，则需要把css单独拆分出来用link在html单独载入，有两方面原因：

- link引用css非阻塞，可使用http并发特性
- js体积减少，将提高页面加载渲染速率

为了导出额外的assets文件，webpack提供了extract-text-webpack-plugin插件。

	var ExtractTextPlugin = require("extract-text-webpack-plugin")
	// 声明cssloader
	var cssLoader = {
    	test: /\.css$/,
    	loader: 'style!css'
	};
	// 为product环境打包时
	if (env == 'product') {
    	cssLoader.loader = ExtractTextPlugin.extract("style-loader", "css-loader");
	}
	// 插件
	plugins: [
    	new ExtractTextPlugin("[name].css"),
	]

如上配置，在开发环境仍然会将css合并至js，但在执行webpack -p后，在对应的输出目录将会生成独立的css文件，且在打包后js文件中将不在包含相应css代码。

另外有关css模块依赖，不能在css中使用import导入模块，会影响cssmode（m-base）的编译，推荐把css放到对应js模块中引入，也更方便于多人协作开发。

### 2，正确使用图片

#### css环境

正常使用相对路径引用，url-loader根据图片大小，决定是否生成base64或者临时性的图片放到内存中暂存，同时js中图片引用路径将会被编译成临时图片地址，确保图片正常显示！

#### js环境

使用require(path)方式载入，编译原理同上，若直接使用相对路径写到img-src，在开发环境可正常输出，但在打包时无法应用output规则输出在打包文件夹下。

引用示例：

	render() {
    	let png = require('../images/touch.png');
    	return (
        	<div>
            	<img src={png} />
            	<h1>Hello</h1>
        	</div>
    	);
	}

编译后：

	880110af781e078951d0d0fda16353f4.png

#### html环境

避免直接在html中使用图片，在css中使用background、js中使用require替代（html中的图片不会被打包）！

### 3，自定义字符串替换

在实际项目中，处理基本的编译任务使用webpack相关loader基本可以满足需求，若要定制项目中特殊的编译规则，则需要额外的字符串替换操作。比如在使用m-base（自适应各种分辨率，css长度单位使用动态rem）组件时，需要把css文件中长度单位px转换成rem，与之匹配的数字也要按算法做转换。

string-replace-webpack-plugin插件则为此而生，简单做如下配置。

	var StringReplacePlugin = require('string-replace-webpack-plugin');
	loaders: [
        cssLoader,
        {
            test: /\.css|jsx?$/,
            loader: StringReplacePlugin.replace({
                replacements: [
                    {
                        pattern: /\d+?px['"; ]/ig,
                        replacement: function (res) {
                            var res = res.replace(/(\d+?)px([; ',"])/ig, function($1 , $2 , $3, index , source) {
                                return ($2 * 2) / (cssmode / 10) + 'rem' + $3;
                            });
                            return res;
                        }
                    }
                ]
            })
        }
	],
	plugins: [
     	new StringReplacePlugin()
	]

要特别注意，插件的调用务必要在cssLoader之后，否则将无法生效。

### 4, 如何使用babel-polyfill?

Babel默认只转换新的JavaScript句法（syntax），而不转换新的API，比如Iterator、Generator、Set、Maps、Proxy、Reflect、Symbol、Promise等全局对象，以及一些定义在全局对象上的方法（比如Object.assign）都不会转码。

举例来说，ES6在Array对象上新增了Array.from方法。Babel就不会转码这个方法。如果想让这个方法运行，必须使用babel-polyfill，为当前环境提供一个垫片。

webpack.config.js对babel-polyfill配置的部分：

	// 在入口数组中,babel-polyfill必须在入口文件字符串前面
	for (var prop in config.entry) {
	    config.entry[prop].unshift(
	        'babel-polyfill'
	    );
	}

最终config.entry的结构如下：

	{
		entry: {
			app: ['babel-polyfill', './js/app']
		}
	}

在入口文件app.js中，在第一行引入`babel-polyfill`。

	import 'babel-polyfill';
	// require('babel-polyfill');
	// 如果没有必要，其实之引入其中的一部分

### 待续...
