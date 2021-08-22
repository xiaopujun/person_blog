# nodejs简介

nodejs是一个运行环境，其可以让javascript运行在服务器中（之前基本都运行在浏览器中）

nodejs是单线程的，不会为每一个请求都创建一个线程，因此对于资源消耗来说，nodejs是轻量级的

# node模块化

在node中，每一个文件相当于一个单独的模块，每一个模块中写的代码会被包到一个方法中。因此在该模块中写的非全局变量或方法对外界是不可见的。可以通过关键字 **exports** 将变量、属性向外抛出，让其对外界可见。当要使用其他模块导出的变量时，可通过 **require** 关键字将对应的模块导入。

node中的每一个文件中的代码在实际执行时，都是被包入到一个方法中进行执行的，该方法传入了5个实参

```js
const demo01  = require('./demo01');
console.log(arguments.callee+'');
//对于模块中的如上两行代码，会被解析为下面的形式，node会自动将其包装到函数里

/**
 * 
 * @param  exports 通过该对象导出属性
 * @param  require 通过该对象导入属性
 * @param  module  指向该模块本身
 * @param  __filename 当前文件所在目录
 * @param  __dirname 当前文件夹所在目录
 */
function (exports, require, module, __filename, __dirname) {
    const demo01  = require('./demo01');
    console.log(arguments.callee+'');

}
```

node中存在一个全局变量，globle 其作用相当于浏览器中的window对象

# node-package

包其实就是对模块一种目标性的组织，一个包里会包含一个完整独立的项目

包结构中包含如下文件和文件夹

- package.json. ---  此文件为包的描述文件，包括这个包依赖什么，可执行的命令有哪些等等关键信息。必须包含该选项
- bin ----可执行二进制文件
- lib -----三方库
- doc ------  文档
- test  ----- 测试文件

# npm

npm是noe package manager，包管理器。其作用相当于软件管家一样。它将包统一管理起来，方便下载，发布

### npm命令

1. npm -v 查看版本
2. npm version 查看所有模块的版本
3. npm search 包名  搜索包
4. npm install 包名  安装包
5. npm remove 包名  删除包
6. npm install  下载当前项目所依赖的所有包
7. npm install 包名 -g    全局安装包

### npm全局安装cnpm

```bashba s
npm install -g cnpm --registry=https://registry.npm.taobao.org			
```

# node搜索包策略

使用node下载的包，引入时直接根据包名引用即可

node程序在运行程序时会搜索该项目以来的包，其搜索策略为

1. 首先搜索当前目录的mode_module
2. 若在当前目录中找不到对应的包，会去上一层目录的noe_module寻找
3. 以此类推，直到找到根目录，若依然无法找到，则报错
