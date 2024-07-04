## undefined 和 null 到底有什么区别？

JavaScript的设计者(Brendan Eich)的回答：null 表示无对象，undefined 表示无值。两者都表示无。但是null无的更具体，undefined无的不具体。
如果定义 let a = undefined; 从语义上看，a将来可以是数字、字符串也可以是对象。但是 let user =
null。从语义上看表示现在user为空，但是将来他应该指向一个对象。

但是这有一个漏洞，null和object共享了一个类型标记。typeof null === "object"

所以在使用上，如果不确定变量未来的值是什么类型的，应该使用undefined。反之如果确定未来应该是对象，则使用null。

从JavaScript的发展历史上看，1995年JavaScript诞生时，最初像Java一样，只设置了null作为表示"无"的值。
根据C语言的传统，null被设计成可以自动转为0。首先，null像在Java里一样，被当成一个对象。但是，JavaScript的数据类型分成原始类型（primitive）和合成类型（complex）两大类，Brendan
Eich觉得表示"无"的值最好不是对象。
其次，JavaScript的最初版本没有包括错误处理机制，发生数据类型不匹配时，往往是自动转换类型或者默默地失败。Brendan
Eich觉得，如果null自动转为0，很不容易发现错误。
因此，Brendan Eich又设计了一个undefined。

所以从设计的角度上看，undefined是对null的补充，实际上他们奔着相同的目的去的。

## 什么是函数的闭包？

它指的是一个函数能够记住并访问其创建时的环境，即使这个函数在其原始作用域之外被执行。换句话说，闭包允许一个函数访问创建时的作用域中的变量，即使这个函数被赋值给另一个变量或者在另一个作用域中被调用。

闭包的工作原理：

- 创建阶段：当一个函数在某个作用域内被定义时，它捕获了那个作用域的变量。
- 调用阶段：即使这个函数离开了原始作用域，它仍然可以访问那些变量。

如果学习过java或者c可能就知道，他的功能差不多是在完成面向对象的封装，比如java中的通过类创建对象，c的结构体实例化。他形成了一个私有化的环境，而不是像普通方法一样，每次运行都是一个全新的状态;
即使外部函数已经执行完毕，只要还有闭包引用这些外部变量，这些变量就不会被垃圾回收机制清理掉。这意味着闭包可以“记住”它被创建时的环境。

那么就有一个疑问，我基本认为闭包有一点过度持有环境变量的苗头。如果不能及时明确的释放闭包，那么闭包就会一直存在，一直不垃圾回收，占用内存。
所以，闭包执行完毕后，应该及时清理掉闭包，避免内存泄漏。

```javascript
function createClosure() {
    var closureVar = 'I am a closure variable';
    return function () {
        console.log(closureVar);
        closureVar = null;  // 手动解除引用
    };
}

var myClosure = createClosure();
myClosure();  // 执行闭包函数
myClosure = null;  // 解除闭包函数的引用

```

这个事情并不绝对，垃圾回收机制还是很优秀的，很多时候，闭包中的环境变量不再被任何其他变量引用，或者整个闭包函数都不再被使用，那么垃圾回收机制会自动清理掉这些变量。上面的例子，只是一个明确的信号，我认为这是一个好习惯

### 闭包的使用场景和方式

#### 隐藏封装数据

闭包可以创建私有变量和方法，防止外部访问和修改。如，可以使用闭包实现模块模式，封装私有数据和函数，只有通过特定的方法才能访问这些数据。你差不多可以理解为和创建一个对象一样；设计思想和目的感觉还是为了实现面向对象。

```javascript
function createCounter() {
    let count = 0;
    return {
        increment: function () {
            count++;
            return count;
        },
        decrement: function () {
            count--;
            return count;
        },
        getCount: function () {
            return count;
        }
    };
}

const counter = createCounter();
console.log(counter.increment()); // 1
console.log(counter.getCount());  // 1
console.log(counter.decrement()); // 0

```

#### 函数工厂（用函数创建函数）

闭包可以用于创建带有特定参数的函数。例如，可以创建一个通用的函数生成器，根据不同的参数生成不同的函数。

```javascript
function makeAdder(x) {
    return function (y) {
        return x + y;
    };
}

const add5 = makeAdder(5);
const add10 = makeAdder(10);

console.log(add5(2));  // 7
console.log(add10(2)); // 12

```

#### 保持状态（记住变量）

闭包可以让函数记住其执行环境中的变量，从而保持状态。例如，可以用闭包实现迭代器，记录当前迭代的位置。

```javascript
function createIterator(arr) {
    let index = 0;
    return function () {
        if (index < arr.length) {
            return arr[index++];
        } else {
            return null;
        }
    };
}

const iterator = createIterator([1, 2, 3]);
console.log(iterator()); // 1
console.log(iterator()); // 2
console.log(iterator()); // 3
console.log(iterator()); // null

```

#### 回调函数和异步编程

闭包在异步编程中非常有用，尤其是处理回调函数时。闭包可以确保异步操作完成后依然能够访问其创建时的环境

```javascript
function fetchData(url) {
    return function (callback) {
        setTimeout(() => {
            // Simulating an async operation
            const data = `Data from ${url}`;
            callback(data);
        }, 1000);
    };
}

const fetchFromAPI = fetchData('https://api.example.com');
fetchFromAPI((data) => {
    console.log(data); // Data from https://api.example.com
});

```

## 什么是Promise、async/await？

