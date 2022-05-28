# 响应式系统的实现与作用
## 响应式数据与副作用函数
假如我们现在存有一个数据且希望它在页面上实时更新(数据刷新，页面就随之刷新)。那么，自然地我们要想办法监测该数据的变动，并在每次变动后调用页面生成对应元素时的渲染方法实现更新。对应地，该数据称为“响应式数据”，与之相关的渲染方法称为“副作用函数”。
## 响应式系统实现思路及完善的步骤
1. 标记监控响应式数据
    - Vue 2 使用`Object.defineProperty`
    - Vue 3 使用`Proxy`
2. 标记保存副作用函数(上面例子中渲染数据的函数)以便后续调用
3. 执行副作用函数
    - 执行(渲染)时自然需要取响应式数据，通过之前对响应式数据的监控，拦截`get`操作，并同时建立响应式数据和副作用函数之间的联系
    - 实际上可能同时有多个副作用函数依赖某一个响应式数据，所以需要用`Set`保存所有依赖，再用`Map`标记响应式数据和整个依赖集合
4. 如果后续响应式数据发生变化，自动拦截`set`操作，先将数据更新后，再触发之前存下的所有对该响应式数据有依赖的副作用函数
    - 读添加，写触发
5. 为了能监控更多对象，使用`WeakMap`标记对象和其所有属性的`depsMap`
    - `WeakMap`和`Map`的区别

6. `cleanUp`：之前的副作用函数和响应式数据之间的依赖关系可能发生变化，例如副作用函数根据某一数据的值决定是否对其他数据依赖。这就需要每次执行副作用函数时，清空之前的依赖关系，并根据即时实际情况重新建立依赖连接

7. debug：既读又写时，副作用函数会在之前的包裹层中无限调用自己。所以每次触发前，判断一下是否正在添加依赖，避免这种既读又写情况下的无限递归。

8. 副作用函数中可能存在嵌套，例如组件中渲染组件。加入副作用函数栈在子组件渲染函数注册完成后还原父组件渲染函数完成注册

9. 添加调度器。让用户能更灵活地执行副作用函数，而不是每次响应式数据更新后必须立即执行
    - 可以选择延时执行
    - 可以选择懒加载
    - 可以选择以微任务的形式在响应式数据的多次同步修改全部完成之后再执行，减少执行次数

10. `computed`：利用懒加载，只有在依赖响应式数据变化后才执行计算函数得到计算属性。平时获取直接返回缓存
    - 记得手动添加依赖和触发，保证底层响应式数据更新后，依赖计算属性的副作用函数也能立即被调用