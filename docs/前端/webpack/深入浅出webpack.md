# 1 前言

电子书链接：[深入浅出webpack](http://webpack.wuhaolin.cn/1%E5%85%A5%E9%97%A8/1-3%E5%AE%89%E8%A3%85%E4%B8%8E%E4%BD%BF%E7%94%A8.html)

webpack是前端打包工具，其作用是把源代码转换成发布到线上的可执行 JavaScrip、CSS、HTML 代码，包括如下内容。

1. 代码转换：TypeScript 编译成 JavaScript、SCSS 编译成 CSS 等。
2. 文件优化：压缩 JavaScript、CSS、HTML 代码，压缩合并图片等。
3. 代码分割：提取多个页面的公共代码、提取首屏不需要执行部分的代码让其异步加载。
4. 模块合并：在采用模块化的项目里会有很多个模块和文件，需要构建功能把模块分类合并成一个文件。
5. 自动刷新：监听本地源代码的变化，自动重新构建、刷新浏览器。
6. 代码校验：在代码被提交到仓库前需要校验代码是否符合规范，以及单元测试是否通过。
7. 自动发布：更新完代码后，自动构建出线上发布代码并传输给发布系统。

# 2 WebPack入门使用

## 2.1 安装webpack

> webpack基于nodejs实现，因此安装前需要先安装nodejs。版本要求>5.0.0

### 2.1.1 安装到本项目

```shell
# npm i -D 是 npm install --save-dev 的简写，是指安装模块并保存到 package.json 的 devDependencies
# 安装最新稳定版
npm i -D webpack

# 安装指定版本
npm i -D webpack@<version>

# 安装最新体验版本
npm i -D webpack@beta
```

### 2.1.2 安装到全局

安装到全局后你可以在任何地方共用一个 Webpack 可执行文件，而不用各个项目重复安装，安装方式如下：

```shell
npm i -g webpack
```

虽然介绍了以上两种安装方式，但是我们推荐安装到本项目，原因是可防止不同项目依赖不同版本的 Webpack 而导致冲突。

## 2.3 使用webpack

安装完后你可以通过这些途径运行安装到本项目的 Webpack：

- 在项目根目录下对应的命令行里通过node_modules/.bin/webpack运行Webpack可执行文件。
- 在NpmScript里定义的任务会优先使用本项目下的Webpack，代码如下：

```json
{
  "scripts": {
    "start": "webpack --config webpack.config.js"
  }
}

```

## 2.4 使用Loader

Webpack不原生支持解析CSS文件。要支持非JavaScript类型的文件，需要使用Webpack的Loader机制

```js
const path = require('path');

module.exports = {
    // JavaScript 执行入口文件
    entry: './main.js',
    output: {
        // 把所有依赖的模块合并输出到一个 bundle.js 文件
        filename: 'bundle.js',
        // 输出文件都放到 dist 目录下
        path: path.resolve(__dirname, './dist'),
    },
    module: {
        rules: [
            {
                // 用正则去匹配要用该 loader 转换的 CSS 文件
                test: /\.css$/,
                use: ['style-loader', 'css-loader?minimize'],
            }
        ]
    }
};
```

Loader可以看作具有文件转换功能的翻译员，配置里的module.rules数组配置了一组规则，告诉Webpack在遇到哪些文件时使用哪些Loader去加载和转换。如上配置告诉Webpack在遇到以.css
结尾的文件时先使用css-loader读取CSS文件，再交给style-loader把CSS内容注入到JavaScript里。在配置Loader时需要注意的是：

1. use属性的值需要是一个由Loader名称组成的数组，Loader的执行顺序是由后到前的；
2. 每一个Loader都可以通过URLquerystring的方式传入参数，例如css-loader?minimize中的minimize告诉css-loader要开启CSS压缩。

想知道Loader具体支持哪些属性，则需要我们查阅文档，例如css-loader还有很多用法，我们可以在css-loader主页上查到。

重新执行构建时，你会发现bundle.js文件被更新了，里面注入了在main.css中写的CSS，而不是会额外生成一个CSS文件。但是重新刷新index.html网页时将会发现Hello,Webpack
居中了，样式生效了！也许你会对此感到奇怪，第一次看到CSS被写在了JavaScript里！这其实都是style-loader的功劳，它的工作原理大概是把CSS内容用JavaScript里的字符串存储起来，在网页执行
JavaScript时通过DOM操作动态地往HTMLhead标签里插入HTMLstyle标签。也许你认为这样做会导致JavaScript文件变大并导致加载网页时间变长，想让Webpack单独输出CSS
文件。这需要通过WebpackPlugin机制来实现。

除了在webpack.config.js配置文件中配置Loader外，还可以在源码中指定用什么Loader去处理文件。以加载CSS文件为例，修改上面例子中的main.js如下：

```js
require('style-loader!css-loader?minimize!./main.css'); 
```

这样就能指定对./main.css这个文件先采用css-loader再采用style-loader转换

## 2.5 使用plugin

通过 Plugin 把注入到 bundle.js 文件里的 CSS 提取到单独的文件中，配置修改如下：

```js

const path = require('path');
const ExtractTextPlugin = require('extract-text-webpack-plugin');

module.exports = {
    // JavaScript 执行入口文件
    entry: './main.js',
    output: {
        // 把所有依赖的模块合并输出到一个 bundle.js 文件
        filename: 'bundle.js',
        // 把输出文件都放到 dist 目录下
        path: path.resolve(__dirname, './dist'),
    },
    module: {
        rules: [
            {
                // 用正则去匹配要用该 loader 转换的 CSS 文件
                test: /\.css$/,
                use: ExtractTextPlugin.extract({
                    // 转换 .css 文件需要使用的 Loader
                    use: ['css-loader'],
                }),
            }
        ]
    },
    plugins: [
        new ExtractTextPlugin({
            // 从 .js 文件中提取出来的 .css 文件的名称
            filename: `[name]_[contenthash:8].css`,
        }),
    ]
};
```

## 2.6 使用DevServer

```shell
npm i -D webpack-dev-server
```

安装成功后执行 webpack-dev-server 命令， DevServer 就启动了

这意味着DevServer启动的HTTP服务器监听在http://localhost:8080/，DevServer启动后会一直驻留在后台保持运行，访问这个网址你就能获取项目根目录下的index.html。
用浏览器打开这个地址你会发现页面空白错误原因是./dist/bundle.js加载404了。同时你会发现并没有文件输出到dist目录，原因是DevServer会把Webpack
构建出的文件保存在内存中，在要访问输出的文件时，必须通过HTTP服务访问。由于DevServer不会理会webpack.config.js里配置的output.path属性，所以要获取bundle.js的正确URL
是http://localhost:8080/bundle.js

### 2.6.1 开启实时预览

```shell
webpack --watch
```

### 2.6.2 开启热替换

```shell
webpack-dev-server --hot
```

### 2.6.3 开启源代码映射

```shell
webpack-dev-server --devtool source-map
```

## 2.7 核心概念

虽然Webpack 功能强大且配置项多，但只要你理解了其中的几个核心概念，就能随心应手地使用它。 Webpack 有以下几个核心概念。

- Entry：入口，Webpack 执行构建的第一步将从 Entry 开始，可抽象成输入。
- Module：模块，在 Webpack 里一切皆模块，一个模块对应着一个文件。Webpack 会从配置的 Entry 开始递归找出所有依赖的模块。
- Chunk：代码块，一个 Chunk 由多个模块组合而成，用于代码合并与分割。
- Loader：模块转换器，用于把模块原内容按照需求转换成新内容。
- Plugin：扩展插件，在 Webpack 构建流程中的特定时机注入扩展逻辑来改变构建结果或做你想要的事情。
- Output：输出结果，在 Webpack 经过一系列处理并得出最终想要的代码后输出结果。

Webpack启动后会从Entry里配置的Module开始递归解析Entry依赖的所有Module。每找到一个Module，就会根据配置的Loader去找出对应的转换规则，对Module进行转换后，再解析出当前
Module依赖的Module。这些模块会以Entry为单位进行分组，一个Entry和其所有依赖的Module被分到一个组也就是一个Chunk。最后Webpack会把所有Chunk转换成文件输出。在整个流程中
Webpack会在恰当的时机执行Plugin里定义的逻辑。

# 3 webpack配置

配置 Webpack 的方式有两种：

- 通过一个JavaScript文件描述配置，例如使用webpack.config.js文件里的配置；
- 执行Webpack可执行文件时通过命令行参数传入，例如webpack--devtoolsource-map。

## 3.1 Entry

entry是配置模块的入口，可抽象成输入，Webpack执行构建的第一步将从入口开始搜寻及递归解析出所有入口依赖的模块。

entry配置是必填的，若不填则将导致Webpack报错退出

### 3.1.2 context

Webpack 在寻找相对路径的文件时会以 context 为根目录，context 默认为执行启动 Webpack 时所在的当前工作目录。 如果想改变 context 的默认配置，则可以在配置文件里这样设置它：

```js

module.exports = {
    context: path.resolve(__dirname, 'app')
}   
```

注意， context 必须是一个绝对路径的字符串。 除此之外，还可以通过在启动 Webpack 时带上参数 webpack --context 来设置 context。

之所以在这里先介绍 context，是因为 Entry 的路径和其依赖的模块的路径可能采用相对于 context 的路径来描述，context 会影响到这些相对路径所指向的真实文件。

### 3.1.3 Entry类型

Entry 类型可以是以下三种中的一种或者相互组合

|类型|例子|含义|
|---|---|---|
|string|'./app/entry'|入口模块的文件路径，可以是相对路径|
|array|['./app/entry1', './app/entry2']|入口模块的文件路径，可以是相对路径。|
|object|{ a: './app/entry-a', b: ['./app/entry-b1', './app/entry-b2']}|配置多个入口，每个入口生成一个 Chunk|

如果是 array 类型，则搭配 output.library 配置项使用时，只有数组里的最后一个入口文件的模块会被导出。

### 3.1.4 Chunk名称

Webpack 会为每个生成的 Chunk 取一个名称，Chunk 的名称和 Entry 的配置有关：

1. 如果 entry 是一个 string 或 array，就只会生成一个 Chunk，这时 Chunk 的名称是 main；
2. 如果 entry 是一个 object，就可能会出现多个 Chunk，这时 Chunk 的名称是 object 键值对里键的名称。

### 3.1.5 动态Entry

假如项目里有多个页面需要为每个页面的入口配置一个 Entry ，但这些页面的数量可能会不断增长，则这时 Entry 的配置会受到到其他因素的影响导致不能写成静态的值。其解决方法是把 Entry
设置成一个函数去动态返回上面所说的配置，代码如下：

```js
// 同步函数
entry: () => {
    return {
        a: './pages/a',
        b: './pages/b',
    }
};
// 异步函数
entry: () => {
    return new Promise((resolve) => {
        resolve({
            a: './pages/a',
            b: './pages/b',
        });
    });
};
```

## 3.2 OutPut

output 配置如何输出最终想要的代码。output 是一个 object，里面包含一系列配置项，下面分别介绍它们

### 3.2.1 filename

output.filename 配置输出文件的名称，为string 类型。 如果只有一个输出文件，则可以把它写成静态不变的：

```js
filename: 'bundle.js'
```

但是在有多个 Chunk 要输出时，就需要借助模版和变量了。前面说到 Webpack 会为每个 Chunk取一个名称，可以根据 Chunk 的名称来区分输出的文件名：

```js
filename: '[name].js'
```

代码里的 [name] 代表用内置的 name 变量去替换[name]，这时你可以把它看作一个字符串模块函数， 每个要输出的 Chunk 都会通过这个函数去拼接出输出的文件名称。

内置变量除了 name 还包括：

|变量名|含义|
|---|---|
|id|Chunk 的唯一标识，从0开始|
|name|Chunk 的名称|
|hash|Chunk 的唯一标识的 Hash 值|
|chunkhash|Chunk 内容的 Hash 值|

其中 hash 和 chunkhash 的长度是可指定的，[hash:8] 代表取8位 Hash 值，默认是20位。

### 3.2.2 chunkFilename

output.chunkFilename 配置无入口的 Chunk 在输出时的文件名称。 chunkFilename 和上面的 filename 非常类似，但 chunkFilename 只用于指定在运行过程中生成的 Chunk
在输出时的文件名称。 常见的会在运行时生成 Chunk 场景有在使用 CommonChunkPlugin、使用 import('path/to/module') 动态加载等时。 chunkFilename 支持和 filename
一致的内置变量。

### 3.2.3 path

output.path 配置输出文件存放在本地的目录，必须是 string 类型的绝对路径。通常通过 Node.js 的 path 模块去获取绝对路径：

```js
path: path.resolve(__dirname, 'dist_[hash]')
```

### 3.2.4 publicPath

在复杂的项目里可能会有一些构建出的资源需要异步加载，加载这些异步资源需要对应的 URL 地址。

output.publicPath 配置发布到线上资源的 URL 前缀，为string 类型。 默认值是空字符串 ''，即使用相对路径。

这样说可能有点抽象，举个例子，需要把构建出的资源文件上传到 CDN 服务上，以利于加快页面的打开速度。配置代码如下：

```js
filename:'[name]_[chunkhash:8].js'
publicPath: 'https://cdn.example.com/assets/'
```

这时发布到线上的 HTML 在引入 JavaScript 文件时就需要：

```js
<script src='https://cdn.example.com/assets/a_12345678.js'></script>
```

使用该配置项时要小心，稍有不慎将导致资源加载404错误。

output.path 和 output.publicPath 都支持字符串模版，内置变量只有一个：hash 代表一次编译操作的 Hash 值。

### 3.2.5 crossOriginLoading

Webpack 输出的部分代码块可能需要异步加载，而异步加载是通过 JSONP 方式实现的。 JSONP 的原理是动态地向 HTML 中插入一个 <script src="url"></script> 标签去加载异步资源。
output.crossOriginLoading 则是用于配置这个异步插入的标签的 crossorigin 值。

script 标签的 crossorigin 属性可以取以下值：

- anonymous(默认) 在加载此脚本资源时不会带上用户的 Cookies；
- use-credentials 在加载此脚本资源时会带上用户的 Cookies。

通常用设置 crossorigin 来获取异步加载的脚本执行时的详细错误信息。

### 3.2.6 libraryTarget 和 library

当用 Webpack 去构建一个可以被其他模块导入使用的库时需要用到它们。

- output.libraryTarget 配置以何种方式导出库。
- output.library 配置导出库的名称。

它们通常搭配在一起使用。

output.libraryTarget 是字符串的枚举类型，支持以下配置。

### 3.2.7 libraryExport

output.libraryExport 配置要导出的模块中哪些子模块需要被导出。 它只有在 output.libraryTarget 被设置成 commonjs 或者 commonjs2 时使用才有意义。

假如要导出的模块源代码是：

```js
export const a = 1;
export default b = 2;
```

现在你想让构建输出的代码只导出其中的 a，可以把 output.libraryExport 设置成 a，那么构建输出的代码和使用方法将变成如下：

```js
// Webpack 输出的代码
module.exports = lib_code['a'];

// 使用库的方法
require('library-name-in-npm') === 1;
```

## 3.3 Module

module 配置如何处理模块。

### 3.3.1 配置Loader

rules 配置模块的读取和解析规则，通常用来配置 Loader。其类型是一个数组，数组里每一项都描述了如何去处理部分文件。 配置一项 rules 时大致通过以下方式：

1. 条件匹配：通过 test 、 include 、 exclude 三个配置项来命中 Loader 要应用规则的文件。
2. 应用规则：对选中后的文件通过 use 配置项来应用 Loader，可以只应用一个 Loader 或者按照从后往前的顺序应用一组 Loader，同时还可以分别给 Loader 传入参数。
3. 重置顺序：一组 Loader 的执行顺序默认是从右到左执行，通过 enforce 选项可以让其中一个 Loader 的执行顺序放到最前或者最后。

下面来通过一个例子来说明具体使用方法：

```js
module: {
    rules: [
        {
            // 命中 JavaScript 文件
            test: /\.js$/,
            // 用 babel-loader 转换 JavaScript 文件
            // ?cacheDirectory 表示传给 babel-loader 的参数，用于缓存 babel 编译结果加快重新编译速度
            use: ['babel-loader?cacheDirectory'],
            // 只命中src目录里的js文件，加快 Webpack 搜索速度
            include: path.resolve(__dirname, 'src')
        },
        {
            // 命中 SCSS 文件
            test: /\.scss$/,
            // 使用一组 Loader 去处理 SCSS 文件。
            // 处理顺序为从后到前，即先交给 sass-loader 处理，再把结果交给 css-loader 最后再给 style-loader。
            use: ['style-loader', 'css-loader', 'sass-loader'],
            // 排除 node_modules 目录下的文件
            exclude: path.resolve(__dirname, 'node_modules'),
        },
        {
            // 对非文本文件采用 file-loader 加载
            test: /\.(gif|png|jpe?g|eot|woff|ttf|svg|pdf)$/,
            use: ['file-loader'],
        },
    ]
}
```

在 Loader 需要传入很多参数时，你还可以通过一个 Object 来描述，例如在上面的 babel-loader 配置中有如下代码：

```js
use: [
    {
        loader: 'babel-loader',
        options: {
            cacheDirectory: true,
        },
        // enforce:'post' 的含义是把该 Loader 的执行顺序放到最后
        // enforce 的值还可以是 pre，代表把 Loader 的执行顺序放到最前面
        enforce: 'post'
    },
    // 省略其它 Loader
]
```

上面的例子中 test include exclude 这三个命中文件的配置项只传入了一个字符串或正则，其实它们还都支持数组类型，使用如下：

```js
{
    test:[
        /\.jsx?$/,
        /\.tsx?$/
    ],
        include
:
    [
        path.resolve(__dirname, 'src'),
        path.resolve(__dirname, 'tests'),
    ],
        exclude
:
    [
        path.resolve(__dirname, 'node_modules'),
        path.resolve(__dirname, 'bower_modules'),
    ]
}
```

数组里的每项之间是或的关系，即文件路径符合数组中的任何一个条件就会被命中

### 3.3.2 noParse

noParse 配置项可以让 Webpack 忽略对部分没采用模块化的文件的递归解析和处理，这样做的好处是能提高构建性能。 原因是一些库例如 jQuery 、ChartJS 它们庞大又没有采用模块化标准，让 Webpack
去解析这些文件耗时又没有意义。

noParse 是可选配置项，类型需要是 RegExp、[RegExp]、function 其中一个。

例如想要忽略掉 jQuery 、ChartJS，可以使用如下代码：

```js
// 使用正则表达式
noParse: /jquery|chartjs/

// 使用函数，从 Webpack 3.0.0 开始支持
noParse: (content) => {
    // content 代表一个模块的文件路径
    // 返回 true or false
    return /jquery|chartjs/.test(content);
}
```

> 注意被忽略掉的文件里不应该包含 import 、 require 、 define 等模块化语句，不然会导致构建出的代码中包含无法在浏览器环境下执行的模块化语句。

### 3.3.3 Parse

因为 Webpack 是以模块化的 JavaScript 文件为入口，所以内置了对模块化 JavaScript 的解析功能，支持 AMD、CommonJS、SystemJS、ES6。 parser
属性可以更细粒度的配置哪些模块语法要解析哪些不解析，和 noParse 配置项的区别在于 parser 可以精确到语法层面， 而 noParse 只能控制哪些文件不被解析。 parser 使用如下：

```js
module: {
    rules: [
        {
            test: /\.js$/,
            use: ['babel-loader'],
            parser: {
                amd: false, // 禁用 AMD
                commonjs: false, // 禁用 CommonJS
                system: false, // 禁用 SystemJS
                harmony: false, // 禁用 ES6 import/export
                requireInclude: false, // 禁用 require.include
                requireEnsure: false, // 禁用 require.ensure
                requireContext: false, // 禁用 require.context
                browserify: false, // 禁用 browserify
                requireJs: false, // 禁用 requirejs
            }
        },
    ]
}

```