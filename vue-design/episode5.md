# 非原始值的响应式方案
## 代理和JavaScript对象
- 代理：对一个对象基本语义的代理，允许我们拦截并重新定义对一个对象的基本操作。
    - `[[GetPrototypeof]]`
    - `[[SetPrototypeof]]`
    - `[[IsExtensible]]`
    - `[[PreventExtensions]]`
    - `[[GetOwnProperty]]`
    - `[[DefineOwnProperty]]`
    - `[[HasProperty]]`
    - `[[Get]]`
    - `[[Set]]`
    - `[[Delete]]`
    - `[[OwnPropertyKeys]]`
    - 下面两个是额外的必要内部方法
    - `[[Call]]`（普通对象没有，函数对象才有）
    - `[[Construct]]`（构造函数）
> 基本操作对应的是复合操作，典型的是调用对象下的方法：`[[Get]] + [[Call]]`
- `Reflect`：全局对象，其下有许多方法可以实现原始对象的基本语义
    - 其对应方法能接收第三个参数`receiver`，确定函数调用中的`this`

- 常规对象：内部方法按照ECMA规范给出的定义实现
- 异质对象：所有非常规对象

## 代理Object
### 对象的读取
响应式系统中，“读取”是一个很宽泛的概念。包括：
- 访问属性：`obj.foo`
- 判断是否包含属性：`key in obj`
- 循环遍历：`for (const key in obj)`

在`Proxy`分别对应拦截函数
`get`、`has`、`ownkeys`
- `ownkeys`补充：
    - 循环遍历中用的不是`key`而是`const ITERATE_KEY = Symbol()`
    - 
