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

在`Proxy`分别对应拦截函数：`get`、`has`、`ownkeys`

**`ownkeys`补充：**
- 循环遍历中用的不是`key`而是`const ITERATE_KEY = Symbol()`
- 在`set`拦截的`trigger`里新增对`ITERATE_KEY`的触发，但要增加判断是已有属性修改还是新增属性
    - 已有属性修改不触发`ITERATE_KEY`
    - 新增属性触发`ITERATE_KEY`
- `deleteProperty`也触发`ITERATE_KEY`

## 合理地触发响应
### 更新了，但没有完全更新
`set`会自动触发`trigger`，但如果`set`的`newVal`和更新之前的`oldVal`相同，自然无需触发`trigger`。增加对前后值的判断，避免无效`trigger`，提升性能。
- 注意对特殊值`NaN`的处理。【`(NaN !== NaN) === true`】
### 代理对象间继承引发的二次`trigger`
```js
const child = reactive({});
const parent = reactive({foo: 1});
Object.setPrototypeof(child, parent);
effect(() => {
  console.log(child.bar);
})
child.bar = 2;
// 根据ECMA规范，如果设置的属性不存在在对象上，则会获取原型并调用其上的set
// 因此以上赋值不仅会触发child的set，还会触发parent.set

// TODO: 为啥会获取调用原型的set捏？明明从结果上观测只为子对象set，不影响父对象属性
```
解决方案：创建Reactive代理对象时，为对象添加`raw`属性记录原始对象，`set`拦截器中判断`receiver.raw === target`再`trigger`
## 浅响应与深响应
深响应的实现方案：代理对象时递归地为对象属性创建代理对象
## 浅只读与深只读
只读影响的拦截器包括：
- `get`时无需`track`
- 禁止`set`和`deleteProperty`

深只读同样递归包裹
