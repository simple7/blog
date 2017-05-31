---
title: vuex入坑
date: 2017-03-18 11:34:56
tags: [webpack]
---

webpack是目前一个很热门的前端打包工具，官网说得很清楚，webpack的出现就是要把requirejs干掉。同时它还提供了十分便利的本地开发的环境。网上并不容易找到一个讲解得比较详细完整的教程，本文结合实践经验，总结一套可用的开发和上线的配置和流程。

首先， [Require JS](http://requirejs.org/) 有什么问题

## **RequireJs存在的问题**

博主先是使用了RequireJs，后来又转了webpack，综合比较，requirejs确实存在一些缺点：

### **1.写法比较笨拙**
 
需要把所有的依赖模块写在require函数里面，当模块很多的时候，看起来逼格就不高了，感受如下：

![](http://static.open-open.com/lib/uploadImg/20160907/20160907140325_981.png)

而webpack既兼容requirejs的写法，也兼容commonjs的写法，也就是说，使用webpack你既可以继续像上面那样写，也可以像node那样写，感受如下：

```
var modules = {
signHandler: require("module/sign-log"),
chatHandler: require("module/chat-win"),
mapHandler: require("lib/map"),
util: require("lib/util")
};
```

可以在需要的时候再去require，而不是搞个大括号把全部的模块一下子写到一起。（模块的导出用module.exports = ....）

当然这两种写法不仅是感光上的区别，逻辑上也有区别。用中括号加载的模块通常webpack是动态去加载，而没有中括号是和主文件打包在一起的。

### **2\. 没有通用模块的概念**

例如有一个弹框模块，用在登陆注册，并且所有页面都有登陆注册，所以这是一个所有页面的通用模块。如果页面的其它模块都没调到通用模块里面的东西的话 ，用RequireJs没什么问题。但是实际情况上不是这样的，例如util模块既会被登陆注册的模块调用，也会被很多其它模块调用。这个时候合并压缩就有问题了：合并后的通用模块如common-app.js会带上util的代码、另外一个页面的例如detail.js也会带上util的代码，以后一改util.js里面的东西，就会一并改动其它所有用到util的页面js，就得重新打所有js的版本号。这样无论对布署上线，还是对于用户的缓存来说都是不利的。

webpack可以把几个文件的通用模块抽出来单独作为一个模块common-chunk.js，引用的时候每个页面先引一个common-chunk.js，再引一个该页面自己的js文件如detail.js，原detail.js里面和其它js文件共用的模块已经被提取到common-chunk.js里面。

### **3\. 没办法直接动态合并压缩一个需要异步加载的模块**

这个问题是这样的，假设我的聊天模块文件有500Kb这么大，并不希望一刷页面就加载，而是用户点了聊天再去加载。这个聊天模块有一个入口文件和其它几 个模块文件，我合并压缩了入口文件，需要有一个输出文件，而入口文件define的模块名和压缩优化后的输出文件的路径肯定是要不一样的，但是压缩之后他并不会自动去改变输 出文件的模块名。这样就导致你要手动去改一下压缩文件的模块名，不然会require不到。我之前找了一下，没有找到解决方案，所以采取了一个压缩两次的比较笨拙的方法。

而webpack有一个文件束chunkFile的概念，它会自动去把需要异步加载的文件变成一个chunkFile，然后触发加载的时候再去加载chunkFile。

### **4\. 需要借助gulp等管理工具进行开发**

webpack本身有一些插件和第三方的插件，可以在本地开一个webpack-dev-server，文件一保存的时候就会自动打包编译js/css/less/sass等。

使用RequireJs虽然看起来缺点比较多，但是使用RequireJs也有webpack不具备的优点，那就是RequireJs开发的时候在浏览器里面，每个模块都是单独一个文件，跟本地文件保持一致，而webpack是把主文件和该文件都用到的模块都打包成了一个文件，这样在调试的时候就需要你去搜索找到要调试的位置，而使用requireJs直接根据第几行就可以了。不过，考虑到使用webpack可以搭建一个很方便的本地开发环境，所以这个缺点也不是很明显。

## **使用webpack**

用一句概括就是：写一个配置文件，然后执行下webpack，就可以把生成的文件输出，可压缩带版本号，同时生成一个source-map文件，这个文件包含了每个模块的js和css的实际(带版本号)路径，根据这个路径就可以把html里面的js/css等换成真实的路径。

webpack是一个打包的工具，它有一个重要的概念，就是把js/css/image/coffee都当成地位相等的资源，你可以在js里面require一个css，也可以require一个image。但是这种模式比较适用于React等框架，都是用js控制。

webpack的其它几个重要概念：

### **1\. loader加载器**

上面说到，各种各样的资源都可以在webpack里面加载，而这些资源都需要相应的加载器，webpack才能识别，然后解析成正常的浏览器认识的资源。

换句话说， 你可以给webpack加载各种各样的资源：css/less/sass/png/babel等，然后在代码里面进行管理。

例如要加一个sass的loader，需要先安装：

```
npm install sass-loader node-sass
```

然后在配置文件添加一个loader:

```
{
test: /\.sass$/,
loaders: ["style", "css", "sass"]
},
```

这样当你require(“hello.sass”)的时候，webpack就能处理这种.sass结尾的文件。这样子有两个好处，一个是webpack能够自动编译sass为css，另一个是require进来的style，webpack会把它解析成一个object，这个object的key就是类名，就可以在js使用样式的类名，这种比较适合类似于react的开发模式。

### **2. 文件束chunk**

上面提到的，会把动态加载的文件生成一个个的chunk， 在配置文件的output里面加一行：

```
chunkFilename: "bundle-[id].js"
```

就会根据id区分不同动态加载的chunk文件，而这些chunk文件名对于我们来说是无关紧要，因为这个是webpack管理的，开发者无需关心叫什么又是怎么加载的。

### **3. webpack-dev-server**

这是webpack的一个插件，可以在本地开一个静态服务，用来作为本地开发的重要工具。具体步骤就是html里面引用的资 源用一个假的域名，如develop.com：

```
<script src="//develop.com/site/app-init.js"></script>
```

然后再把develop.com绑到本地回路:

127.0.0.1 fedren.com

这样请求就打到了本地的80端口。同时在本地开一个nginx监听在80端口，nginx收到80端口的请求后，再把请求转发到webpack的服务(默认是8080端口)。这样就能够实现本地开发，下文会具体介绍。

下面一步步介绍怎么配置和使用webpack

## **webpack的基本配置**

首先，npm init创建一个node的配置文件package.json，然后安装webpack：

```
npm install webpack
sudo npm install webpack -g //安装一个全局的命令
```

再创建一个webpack.config.js文件，加入最基本的配置：

```
module.exports = {
// The standard entry point and output config
//每个页面的js文件
entry: {
home: "js/home",
detail: "js/detail"
},
output: {
path: "assets", //打包输出目录
publicPath: "/static/build/", //webpack-dev-server访问的路径
filename: "[name].js", //输出文件名
chunkFilename: "bundle-[id].js" //输出chunk文件名
}
};
```

工程的js都放到js目录下，一个叫home.js，另一个叫detail.js，输出到assets目录，publicPath是为webpack-dev-server所使用

然后在当前目录执行webpack，发现webpack报错了：

```
ERROR in Entry module not found: Error: Cannot resolve module 'js/home' in /Users/yincheng/code/blog-webpack
```

找不到js/home的模块，只要在配置里面加一句resolve：

```
resolve: {
modulesDirectories: ['.']
}
```

告诉webpack所有模块的启始目录由当前目录开始，再执行下webpack就可以正常输出了：

![](http://static.open-open.com/lib/uploadImg/20160907/20160907140326_686.png)

到目前为此，当前工程的目录结构就是这样的了：

![](http://static.open-open.com/lib/uploadImg/20160907/20160907140326_976.png)

接下来，创建html：home.html，里面引入js文件，"static/build"即为上面定义的publicPath：

```

<body>
<p>home.html</p>
<script src="//develop.com/static/build/home.js"></script>
</body>
```

注意我们用了一个develop.com的域名，把这个域名绑到本地回路：

```
127.0.0.1 develop.com
```

然后配置nginx，打开nginx.conf，加多一个server:

```
server {
listen 80;
server_name payment-admin.com;
charset utf-8;
#工程路径
root /Users/yincheng/code/demo;
autoindex on;
autoindex_exact_size on;

location ~* /.+\.[a-z]+$ {
proxy_set_header x-request-filename $request_filename;
# webpack的服务
proxy_pass http://127.0.0.1:8080;
}
}
```

启动nginx或者重启下nginx

然后再装一个webpack-dev-server：

```
npm install webpack-dev-server --save-dev
sudo npm install webpack-dev-server -g
```

然后启动webpack-dev-server，执行：

```
webpack-dev-sever --port=8080 //不加port参数，默认就为8080端口
```

然后就可以访问：http://develop.com/html/home.html

![](http://static.open-open.com/lib/uploadImg/20160907/20160907140326_346.png)

这个时候，只要一改变home.js的内容，webpack-dev-server就会自动打包新的文件 ，一刷新页面，就是最新的修改了。这样就实现了最基本的本地开发，不管你用的jsp/php，都不需要把js/css往服务器上传。 注意webpack-dev-server是在内存生成的文件，你在本地是找不到static/build目录的，只有执行了webpack打包才会输出文件到assets目录。一个为上面配置里的publicPath，另一个为path。

引入样式文件——首先创建css/home.css：

```
body{
color: #f00;
}
```

然后在js里面引入这个css文件：

```
require("css/home.css");
```

一保存之后，会发现webpack-dev-server报错了：

```
ERROR in ./css/home.css
Module parse failed: /Users/yincheng/code/blog-webpack/css/home.css Unexpected token (1:4)
You may need an appropriate loader to handle this file type.
```

根据提示，我们需要加装一个css loader，让webpack能够处理css文件，更改webpack.config.js，加入一个loader：

```
module.exports = {
entry: ...,
output: ...,
resolve: ...,
module: {
loaders: [
{
test: /\.css$/,
loader: "style-loader!css-loader"
},
]
}
};
```

当然要先安装一下：npm install style-loader css-loader --save-dev，然后再重启下webpack-dev-server，就可以加载样式了，我们发现webpack是把样式动态插到了head标签的style里面，但是一般并不希望直接写到head里面，而是独立的一个css文件，这个时候借助一个分离css的插件就可以了：

```
npm install extract-text-webpack-plugin --save-dev
```

同时把配置文件的loader改一下：

```
var ExtractTextPlugin = require("extract-text-webpack-plugin");
module.exports = {
module: {
loaders: [
// Extract css files
{
test: /\.css$/,
loader: ExtractTextPlugin.extract("style-loader", "css-loader")
},
]
},

plugins: [
new ExtractTextPlugin("[name].css")
]
};
```

就会生成和js相同路径和名字的css文件，在home.html里面引入css文件：

```

<link rel="stylesheet" href="//develop.com/static/build/home.css"></link>
```

你也可以加载各种各样的loader，如加载一个sass/less loader，require一个sass/less文件后就可以写sass/less了，webpack会把它编译成和上面一样普通的css文件，读者可以自己试试，还可以再装一个png/jpg的loader，指定一个小于多少个k的图片的参数，webpack就会把小于指定尺寸的图片转成base64的格式。各种loader的安装查一查就有了。

到这里一个最基本的本地开发环境就已经搭起来了。接下来讨论自动刷新

## **自动刷新**

上面一保存js/css的时候，webpack server就会自动打包，刷新页面的时候就是最新的修改。这个刷新只要使用webpack的hot模式就可以自动实现，即一保存就自动打包刷新。将上面运行webpack-dev-server的命令再加多两个参数，按照官方文档的方式：

```
webpack-dev-server --port=8383 --hot --inline
```

如果没有意外，在你的电脑上将会报错：

```
ERROR in multi home
Module not found: Error: Cannot resolve module 'webpack/hot/dev-server' in /Users/yincheng/code/blog-webpack
@ multi home
```

这个问题困惑了笔者好久，因为在node_modules里面是有这个"webpack/hot/dev-server"的，其实只要认真看下上面的提示，就会发现它并不是说在node_modules里面，而是在当前工程目录里，所以把node_modules里的webpack文件夹拷一份到外面就可以正常运行了。（如果你又配了个context的参数的话，那就根据提示拷到context指定的目录）

使用hot模式，只要一保存js/css就可以自动刷新了，这个功能确实很方便。如果不写参数，也可以把它写在配置文件里面：

```
var hotModuleReplacementPlugin = require("webpack/lib/HotModuleReplacementPlugin");
module.exports = {
plugins: [
new ExtractTextPlugin("[name].css"),
new hotModuleReplacementPlugin()
],
devServer: {
historyApiFallback: true,
hot: true,
inline: true,
progress: true
}
};
```

然后运行server就不用带上后面那两个参数了。

## **Common chunk**

如上文提到，webpack可以将几个js的公共模块提取成一个chunk，需要借助一个commonChunkPlugin，在上面的plugins再添加一个：

```
var CommonsChunkPlugin = require("webpack/lib/optimize/CommonsChunkPlugin");
plugins: [
new CommonsChunkPlugin({
//minChunks: 3,
name: "common-app.chunk",
chunks: ["home", "detail", "list"]
})
]
```

这样就可以把home、detail、list三个js和css用到的公共模块提取到common-app.chunk.js和common-app.chunk.css这两个文件了。注意页面要先引入这两个文件，然后再引入具体页面的js，webpack在common chunk里面定义了它的require函数。如上面的home.html：

```

<script src="//develop.com/static/build/common-app.chunk.js"></script>
<script src="//develop.com/static/build/home.js"></script>
```

可以指定一个minChunk的参数，指定模块至少被require几次才能提取出来，默认是3

还可以定义两个commonChunk，例如在详情页、列表页和首页都有搜索的模块，而其它页面没有搜索的模块，也就是说除了所有页面都有的公共模块如登陆注册外，还有一个搜索的公共模块有三个页面要用到。如果都用一个common chunk，会把搜索的也放进来，但其它很多页面并不需要用到。这个时候需要加多一个common chunk：

```
plugins: [
new CommonsChunkPlugin({
name: "search-app.chunk",
chunks: ["search-app-init", "home", "detail", "list"]
}),
new CommonsChunkPlugin({
name: "common-app.chunk",
chunks: ["home", "detail", "search-map", "search-app.chunk", "sell", "about", "blog"]
})
]
```

注意要把search-app.chunk也写到下面那个所有页面的chunk里面，否则webpack会定义两个一样的require函数，页面的模块也会跟着混乱，一刷页面就报错。页面引用js的顺序就变成了：

```

<script src="//develop.com/static/build/common-app.chunk.js"></script>
<script src="//develop.com/static/build/search-app.chunk.js"></script>
<script src="//develop.com/static/build/home.js"></script>
```

## **压缩和版本号**

压缩只需要要在plugins里面再添加一个用来压缩的插件：

```
var webpack = require("webpack");
plugins: [
new webpack.optimize.UglifyJsPlugin()
]
```

这样执行webpack输出的js/css就是压缩的

版本号就是在输出带上hash的替换符，如下：

```
module.exports = {
output: {
path: "assets",
publicPath: "/static/build/",
filename: "[name]-[chunkhash].js",
chunkFilename: "bundle-[chunkhash].js"
},

plugins: [
new ExtractTextPlugin("[name]-[contenthash].css")
],
}
```

其中js用的是webpack的chunkhash，而css用的是contenthash，contenthash是根据内容生成的hash。如果不用contenthash，那么一改js，css的版本号也会跟着改变，这个就有问题了。webpack还有另外一个自带的叫做"[hash]"，这个hash是所有文件都用的同一个哈希，也就是说某个文件改了，所有文件的版本号都会跟着改，所以一般不用这个。

运行webpack，如果报了下面这个错误：

```
ERROR in chunk detail [entry]
[name]-[chunkhash].js
Cannot use [chunkhash] for chunk in '[name]-[chunkhash].js' (use [hash] instead)
```

那你就把plugins里面的热替换插件注释掉就好了，上线的config不需要热替换：

```
plugins: [
//new hotModuleReplacementPlugin(),
],
```

成功执行后，就会在设定的output目录下面输出加上版本号的文件：

```
.
├── detail-d19e4614a1c4f3c1581b.js
├── home-11198f8526424e8c58ce10a2799793e3.css
└── home-5ec13a52eea2a6faf96a.js
```

有了版本号之后，下一步是要把html里面的js/css换成带版本号的路径

## **替换Html里js/css路径**

之前在html里的路径是test.com，现在要把它换成cdn且带版本号的路径，也就是说，目标是要把下面的引入：

```
<script src="//develop.com/static/build/home.js"></script>
```

替换成下面的引入，并把新生成的html输出到built目录

```

<script src="//cdn.mycdn.com/test/home-5ec13a52eea2a6faf96a.js"></script>
```

目测没有现成符合格式的插件可以用，可以自已用node写一个，不费事。

首先要知道所有文件的对应的版本号，可以用 AssetsPlugin，生成source-map：

```
var AssetsPlugin = require('assets-webpack-plugin');
output: {
publicPath: "//cdn.mycdn.com/static/build/"
},
plugins: [
new AssetsPlugin({filename: './source-map.json', prettyPrint: true}),
]
```

执行webpack之后，就会生成source-map.json，打开这个文件：

```
{
"detail": {
"js": "//cdn.mycdn.com/static/build/detail-c8a2c82ebe2e48e06564.js"
},
"home": {
"js": "//cdn.mycdn.com/static/build/home-380af86bfeb6fcb477a4.js",
"css": "//cdn.mycdn.com/static/build/home-11198f8526424e8c58ce10a2799793e3.css"
}
}
```

根据develop.com开头的以及最后面的home.js/home.css，就可以在上面找到对应的路径名。笔者写了个脚本，可以实现这个功能，详见： [version-control-replace-html](https://github.com/liyincheng/version-control-replace-html)

到这里，整个流程就基本完成了。还有一些优化的步骤

## **优化**

### **1\. 优化模块id**

webpack对于每个模块都是用id标志，而不是用模块的名字，只是为了节省空间。还可以再节省，就是用它自带的 [occurrence-order](https://webpack.github.io/docs/optimization.html) 插件将最常用的模块靠前，这样可以再节省一点点空间，因为id是从0开始排的，从一位数到n位数。

```
new webpack.optimize.OccurenceOrderPlugin()
```

### **2\. 移出版本号**

在上面用了common-chunk的插件，抽离公共模块，在这个common-chunk.js里，webpack会定义每个模块加载的src，以便于加载那些需要动态加载的chunk，如下：

```
script.src = __webpack_require__.p + "" + chunkId + "-" +
{"0":"0cb48ff1ab1d1156015d","5":"e9e7f761f306c648ccef","6":"cbbdf8e3ad1aba34ced0"}[chunkId] + ".js";
```

从上面可以看出它会把版本号也写在里面，这样就导致一个问题，每改一个js文件，它的版本号就会变化，就会导致common chunk里面的内容发生变化，所以它的版本号也得跟着变，也就是说改了一个文件，影响了两个文件。所以需要把它抽出来，有个插件已经做了这样的事情，叫做 [ChunkManifestPlugin](https://github.com/diurnalist/chunk-manifest-webpack-plugin) ：

```
var ChunkManifestPlugin = require('chunk-manifest-webpack-plugin');
plugins: [
new ChunkManifestPlugin({
filename: "chunk-manifest.json",
manifestVariable: "webpackManifest"
})
]
```

传两个参数，一个是输出文件名，另一个是变量名，用于上面的script.src，执行webpack后，它会把上面script.src的那一坨东西放到chunk-manifest.json，然后在页面写一个内联的script，定义一个全局变量window.webpackManifest，值为manifest.json里面的内容。笔者已在上面的替换版本号的脚本做处理，只需在页面合适的地方写上一行：

```
<!--%webpack manifest%-->
```

就会把这行替换成一个script标签。

### **3\. 多个common-chunk的优化**

在上面写了两个common chunk，在生成的两个chunk文件里面，你会发现大量的的重复代码，已经失去了公共模块的作用，这个问题可以用一个 [MoveToParentMergingPlugin](https://github.com/soundcloud/move-to-parent-merging-webpack-plugin) 解决，它会把search-app用到的common-app的模块全部移到了common-app，search-app就不会重复common-app的内容了。

## **html保存自动刷新**

上面提到，只要一保存css/js，webpack-dev-server就会自动保存和刷新，但是html/jsp没办法（如果你用react开发，可以用react-hot-loader），其实可以手动解决这个问题。打开 node_modules/webpack-dev-server/client/index.js这个文件，可以发现webpack是用的sockjs实现自动刷新的。浏览器使用sockjs创建socket客户端，连接到webpack的服务，保存更改的时候，服务就向浏览器的socket发送消息，接收到这个消息后客户端就调window.location.reload刷新页面。所以可以模仿这个过程，在本地另开一个服务，监听html的修改，然后向浏览器端发送刷新页面的消息。

具体来说，首先在上面的node_modules/webpack-dev-server/client/index.js这个文件最后面再添加一个socket连接：

```
/*自定义reload window*/
var reload = new SockJS("http://localhost:9999/reload");
reload.onopen = function(){
console.log("customer reload start.......");
}

reload.onclose = function(){
console.log("customer reload close.......");
}

reload.onmessage = function(_msg){
var msg = JSON.parse(_msg.data);
if(msg.type === "reload"){
console.log("customer reload window now");
window.location.reload();
}
}
```

这个9999端口的server就是下面要在本地监听的一个socket服务。在开这个socket服务之前，需要先在本地开一个监听文件修改的服务，然后再向这个socket服务发送消息。监听的服务比较好写，有现成的node包可以用： [chokidar](https://www.npmjs.com/package/chokidar) ，使用也非常简单。监听到修改之后就可以执行上传服务器的命令，然后（使用进程间的通信）再向socket服务发送一个需要刷新的消息，再传递给浏览器的scoket，如上面的代码，一收到消息就刷新页面。具体代码查看 [github](https://github.com/liyincheng/webpack-tool/tree/master/file-watch)

除了优化，在使用中会遇到的一些问题：

## **解决问题**

### **1\. umd的require模式**

有时候会引入外部的库，这些库可能会用umd的require模式，判断是要用requirejs还是commonjs或是写个全局的函数：

```
/* CommonJS */ if (typeof require === 'function' && typeof module === 'object' && module && typeof exports ===
'object' && exports)
module['exports'] = init(require("ByteBuffer"));
/* AMD */ else if (typeof define === 'function' && define["amd"])
define("lib/chat/ProtoBuf", ["./ByteBuffer"], init);
/* Global */ else(global["dcodeIO"] = global["dcodeIO"] || {})["ProtoBuf"] = init(global["dcodeIO"]["ByteBuffer"]);
```

这个的问题就在于，只要页面上有require出现，webpack就会去打包，不管你是写if里面还click事件里面。因为像上面说的，webpack会把异步加载的文件打包成一个boundle文件，同时也会把非异步的打包到一起。像上面那样写，它会重复打包，生成好多个bundle。只要加多一个 [umdREquirePlugin](https://www.npmjs.com/package/umd-require-webpack-plugin) ，webpack就能正常打包了。

### **2\. 如何加载外部资源**

webpack是一个打包的工具，它并不是像requireJs那样可以支持直接require一个外部资源。

例如我要require谷歌地图：https://maps.googleapis.com/maps/api/js，打包的时候webpack会给出一个warning，说加载不到这个外部资源，运行代码的时候会报错，提示没有这个模块。

另外一个问题是，我需要if else判断，如果是中国的环境就加载中国域名的谷歌地图：http://ditu.google.cn/maps/api/js 否则就加载上面的，使用webpack是没办法做到的， 使用requireJs就可以很简单地直接require一下就行。

但其实这个问题很好解决只要自己写一个动态加载script的函数就好了，一个兼容性很好的版本：

```
function loadScript(url, callback){
var script = document.createElement("script")
script.type = "text/javascript";
if (script.readyState){ //IE
script.onreadystatechange = function(){
if (script.readyState == "loaded" || script.readyState == "complete"){
script.onreadystatechange = null;
callback();
}
};
} else { //Others
script.onload = function(){ callback(); };
}
script.src = url;
document.getElementsByTagName("head")[0].appendChild(script);
}
```

详见： [The best way to load external JavaScript](https://www.nczonline.net/blog/2009/07/28/the-best-way-to-load-external-javascript/)

webpack虽然是一个利器，但是坑也不少，目前遇到过的不太好解决的问题：

## **遇到的困难**

### **1\. chunkhash**

使用chunkhash有两个问题，一个是css改变之后，js的版本号也会跟着改变，即使js没有修改，但是比较这两个js文件的时候，你会发现这两个版本号不一样的文件内容是完全一模一样的。因为chunkhash不是根据文件内容算的hash值。第二个问题是，相同的代码在不同人的机器上打的包的版本号不一样。如果使用一些根据文件内容打版本号的插件，如 [webpack-md5-hash](https://www.npmjs.com/package/webpack-md5-hash) ，这个插件是用文件内容作一个md5的计算得出一个版本号，这样可以解决上面的两个问题，但是又引发了新的问题，这个md5的时不时就会出现打的版本号不唯一的情况，文件内容不同、版本号相同，而且这个概率还不小。所以最后还是放弃了使用这个插件，然后又尝试了另外一个使用sha算法计算，但是这个改了一个文件会使几个文件的版本号也发生变化。现在还是使用chunkhash

### **2\. 模块id发生变化**

上文提到，webpack的模块是用id标志的，每个模块对就一个id，例如util对应2，但是这个id不是固定不变的，在n次修改和打包之后，util的id可能会变成了3，这个就比较坑了，给增量上线造成了阻力，即单独上一个html有风险。因为在common-chunk里面，util的id是上次打包的时候定的，但是你这次打包util的id变了，而你只想上home.html，在home.html里面引的home.js里面使用到的util的id对不上common-chunk里面的，导致不能在home里面正常地加载util这个模块。一个临时的解决办法是，home.js不要使用common-chunk，所有的模块都打包到home.js里面就不会有这个问题。

