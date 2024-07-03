## undefined 和 null 到底有什么区别？

js设计者的回答：null 表示无对象，undefined 表示无值。两者都表示无。但是null无的更具体，undefined无的不具体。
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