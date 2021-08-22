# React-router

使用react-router时注意版本，不同版本的组件，使用方式会有差异。

在新版本中，Link组件放在了react-router-dom包下

# react-router精确匹配

使用方式：在router组件中添加exact关键字

```react
<Route path={"/index"} exact component={Index} />
```

精确匹配的效果：当开启精确匹配后，路由地址必须完全和path属性相同才会匹配上，反之无法匹配。对应的非精确匹配只需有path部分的样子即可

如上路由，在非精确匹配下，/index/aaaa也可匹配上

# React-router动态传值

使用roter动态传值，可以在对应的路由有加上 :key 

```react
<Route path={"/workflow/:id"} exact component={Index} />
```

这样即可在路由上动态的传入参数，参数的key值就是id，其传入到目标组件后，会将值放入props的match属性中

```js
isExact: true
params: {id: "33333"}
path: "/index/:id"
url: "/index/33333"
```

# react-router重定向

两种方式

1. 标签式：使用<Redirent>标签即可实现重定向功能
2. 函数式：使用 this.props.history.push('/newRouter')即可实现页面重新跳转
