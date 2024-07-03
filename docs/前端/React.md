# jsx语法规则

1. 定义虚拟dom时，无需加引号
2. jsx标签中混入js表达式时，要用{}包裹。（每一个表达式会产生一个具体的值，比如变量、逻辑运算、函数调式调用）
3. 组件样式的类名指定不能使用class，与es6语法冲突，应该使用className
4. 内联样式使用style={{}}的方式进行定义
5. jsx中，只能有一个根标签
6. jsx中每一个标签必须闭合，没有内容时，可以自闭合

# React组件实例的三大属性

1. props：该属性是只读的，不能直接修改他的值。用于获取组件外部的值
2. refs：常常用于事件处理
3. state：用于处理组件自身内部的数据

# React类组件方法的this指向问题

```react
export default class Demo01 extends React.Component {

    constructor(props) {
        super(props);
        this.doSthing = this.doSthing.bind(this);//将doSthing方法中的this绑定要该类的具体实例上，也既是让方法中的this指向类的实例对象
    }

    doSthing() {
        console.log("I will do sthing");
        console.log(this);//在没有进行构造器中this指向的变更时，此处的this指向的是undefine
    }

    render() {
        return (
            // 此处onClick的操作，只是将doSthing方法拿到，赋值给了点击事件，实际在调用的时候，函数属于普通调用。而非实例调用
            <h1 onClick={this.doSthing}>this is a test</h1>
        )
    }
}

ReactDOM.render(
    <div>
        <Demo01 />
    </div>,
    document.getElementById('root')
);
```

# react组件的初始化与箭头函数的使用

在类组件中，组件的初始化可以通过直接在类中使用变量的赋值语句实现

自定义函数可以直接使用肩头函数的方式进行this的自动绑定，使用箭头函数时，函数体内的this指向为空，但具体运行时它会找到函数外层的this对象，并将该对象作为自身的this对象

```react
export default class Demo01 extends React.Component {
	
  	//此种方式可以直接在类的实例对象上添加对应的属性，并赋上对应的数值
    state = {
        name: 'test',
        age: '23'
    }

		//使用箭头函数时，其内部的this指向会直接寻找其外层的this
    doSthing = () => {
        console.log("I will do sthing");
        console.log(this);//在没有进行构造器中this指向的变更时，此处的this指向的是undefine
    }

    render() {
        return (
            // 此处onClick的操作，只是将doSthing方法拿到，赋值给了点击事件，实际在调用的时候，函数属于普通调用。而非实例调用
            <h1 onClick={this.doSthing}>this is a test</h1>
        )
    }
}
```

# React对props属性进行限制

使用prop-types库进行限制

# react类组件中构造器中的props

1. 类组件中，构造器可写可不写
2. 如果写了构造器，必须调用super，并传入props，如果不传，则在构造器中通过实例对象操作props时，会丢失props

# react函数式组件使用props

直接将props作为函数参数传入即可

# react的第三大属性refs

- react中的refs用于唯一标识真实的dom节点，其作用与原生的i类似
- 需要注意的是，在使用标识时，属性key未ref。在通过组件实例对象获取其内容时，属性为refs

### 字符串类型的refs（官方已不推荐使用）

- 使用时，只需通过ref-“refname”指定具体的名称即可。react会自动将对应的dom元素收集并挂载到组件实例对象的refs属性内

```react
class Demo01 extends Component {

    showMessage01 = () => {
        const {showMsg01} = this.refs;
        alert(showMsg01.value);
    }
    showMessage02 = () => {
        const {showMsg02} = this.refs;
        alert(showMsg02.value);
    };

    render() {
        return (
            <div>
                <input ref={"showMsg01"} placeholder={"click the button show message"}/>
                <button onClick={this.showMessage01}>click me</button>
                <input ref={"showMsg02"} onBlur={this.showMessage02} placeholder={"lose the force show message"}/>
            </div>
        );
    }
}
```

### 回调函数式的refs

```react
class Demo02 extends Component {

    showMessage01 = () => {
        const {input01} = this;
        alert(input01.value);
    }
    showMessage02 = () => {
        const {input02} = this;
        alert(input02.value);
    };

    //渲染逻辑：react执行到渲染函数时，会直接调用ref的函数回调，传入当前的dom节点。在函数实现内使用箭头函数的方式将当前节点绑定到了组件实例对象的属性上
    render() {
        return (
            <div>
                <div>
                    <input ref={(currentNode) => {
                        this.input01 = currentNode;
                    }} placeholder={"click the button show message"}/>
                    <button onClick={this.showMessage01}>click me</button>
                    <input ref={(currentNode) => {
                        this.input02 = currentNode
                    }} onBlur={this.showMessage02} placeholder={"lose the force show message"}/>
                </div>
            </div>
        );
    }
}
```

#### 组件实例绑定式函数ref

```react
class Demo02 extends Component {

    showMessage01 = () => {
        const {input01} = this;
        alert(input01.value);
    }

    saveInput = (currentNode) => {
        this.input01 = currentNode;
    }

    //渲染逻辑：使用内联函数式的ref，在组件更新时，会两次调用回调，第一次传入null，清空之前的内容，第二次才会传入真实的dom。此种现象可以通过组焊实例对象绑定式函数解决
    render() {
        return (
            <div>
                <div>
                    <input ref={this.saveInput} placeholder={"place input"}/>
                    <button onClick={this.showMessage01}>click me</button>
                </div>
            </div>
        );
    }
}
```

### createRefs式的refs

- 此种方式为react官方最推荐的方式
- 此种方式通过reacteRef()方法创建存放ref的容器，但是该容器专人专用。因此不同的的ref都要单独调用该方法重新创建容器

```react
class Demo03 extends Component {

    ref1 = React.createRef();
    ref2 = React.createRef();

    showMessage01 = () => {
        console.log(this);
        alert(this.ref1.current.value);
    }

    showMessage02 = () => {
        alert(this.ref2.current.value);
    }

    render() {
        return (
            <div>
                <div>
                    <input ref={this.ref1} placeholder={"place input"}/>
                    <button onClick={this.showMessage01}>click me</button>
                    <input onBlur={this.showMessage02} ref={this.ref2} placeholder={"place input"}/>
                </div>
            </div>
        );
    }
}
```

## refs使用注意事项：

- 不要过度的使用refs
- 节点的事件操作节点本身的数据，此种情况可以省略ref，另行处理

# 受控组件与非受控组件

- 非受控组件：对于产生输入的网页控件，比如input框，对其值进行现取现用即为非受控组件
- 受控组件：对于产生输入的网页控件，比如input框，对其值先存入状态中，在需要时从状态中取，则为受控组件

# 高级函数

如果一个函数满足以下条件的任意一个即为高级函数

- 一个函数接受的参仍然是一个函数
- 一个函数调用的返回值仍然是一个函数

# 函数的颗粒化

通过函数调用，继续返回函数的方式，实现多次接受参数最后一次性处理的函数编码方式

# React生命周期

#### 生命周期图（旧）

1. forceUpdate：组件强制更新，在不更改任何状态数据的情况下想直接更新组件，可调用此回调
2. componentWillReceiveProps钩子在父子组件首次渲染时不调用

![截屏2021-08-08 下午3.25.25](/Users/yinzhenpu/Desktop/note/前端/image/截屏2021-08-08 下午3.25.25.png)

### 生命周期图（新）

![生命周期新](/Users/yinzhenpu/Desktop/note/前端/image/生命周期新.png)

## 新旧对比

废弃3个 旧的的will生命周期钩子，添加2个钩子

废弃钩子：

- componentWillUpdate()

- componentWillReceiveProps()

- componentWillMount()

新增的钩子：

- getSnapshotBeforeUpdate()：在更新之前获取快照，快照数据可自定义

- getDerivedStateFromProps()



# React diffing算法

react的比较算法依赖于标签的key值，故在给标签设置key值时，不要以index进行复制，可能会出现index值乱序导致页面渲染性能大幅下降。key值取标签的唯一标识即可

# react脚手架

核心基座：react+webpack+es6+eslint

# react如何确认数据放在那个组件的state中

- 某个组件自身使用，放在自身的state中即可
- 某些组件共同使用，放在他们共同的父组件内（状态提升）

# react父子组件的通信

- 父向子组件传递数据，直接使用props即可
- 子组件向父组件传递数据，通过props传入父组件的数据处理函数即可

注意：状态在哪里，操作状态的方法就在哪里
