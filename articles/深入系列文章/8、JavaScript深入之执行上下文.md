## 前言

在[《JavaScript深入之执行上下文》](https://github.com/liu-fatty/library/blob/master/articles/%E6%B7%B1%E5%85%A5%E7%B3%BB%E5%88%97%E6%96%87%E7%AB%A0/4%E3%80%81JavaScript%E6%B7%B1%E5%85%A5%E4%B9%8B%E6%89%A7%E8%A1%8C%E4%B8%8A%E4%B8%8B%E6%96%87%E6%A0%88.md)中讲到，当JavaScript代码执行一段可执行代码(executable code)时，会创建对应的执行上下文(execution context)。

对于每个执行上下文，都有三个重要属性：

* 变量对象(Variable object，VO)
* 作用域链(Scope chain)
* this

然后分别在[《JavaScript深入之变量对象》](https://github.com/liu-fatty/library/blob/master/articles/%E6%B7%B1%E5%85%A5%E7%B3%BB%E5%88%97%E6%96%87%E7%AB%A0/5%E3%80%81JavaScript%E6%B7%B1%E5%85%A5%E4%B9%8B%E5%8F%98%E9%87%8F%E5%AF%B9%E8%B1%A1.md)、[《JavaScript深入之作用域链》](https://github.com/liu-fatty/library/blob/master/articles/%E6%B7%B1%E5%85%A5%E7%B3%BB%E5%88%97%E6%96%87%E7%AB%A0/6%E3%80%81JavaScript%E6%B7%B1%E5%85%A5%E4%B9%8B%E4%BD%9C%E7%94%A8%E5%9F%9F%E9%93%BE.md)、[《JavaScript深入之从ECMAScript规范解读this》](https://github.com/liu-fatty/library/blob/master/articles/%E6%B7%B1%E5%85%A5%E7%B3%BB%E5%88%97%E6%96%87%E7%AB%A0/7%E3%80%81JavaScript%E6%B7%B1%E5%85%A5%E4%B9%8B%E4%BB%8EECMAScript%E8%A7%84%E8%8C%83%E8%A7%A3%E8%AF%BBthis.md)中讲解了这三个属性。

阅读本文前，如果对以上的概念不是很清楚，希望先阅读这些文章。

因为，这一篇，我们会结合着所有内容，讲讲执行上下文的具体处理过程。

## 思考题

在[《JavaScript深入之词法作用域和动态作用域》](https://github.com/liu-fatty/library/blob/master/articles/%E6%B7%B1%E5%85%A5%E7%B3%BB%E5%88%97%E6%96%87%E7%AB%A0/3%E3%80%81JavaScript%E6%B7%B1%E5%85%A5%E4%B9%8B%E8%AF%8D%E6%B3%95%E4%BD%9C%E7%94%A8%E5%9F%9F%E5%92%8C%E5%8A%A8%E6%80%81%E4%BD%9C%E7%94%A8%E5%9F%9F.md)中，提出这样一道思考题：

``` js
var scope = "global scope";
function checkscope(){
  var scope = "local scope";
  function f(){
    return scope;
  }
  return f();
}
checkscope();
```

``` js
var scope = "global scope";
function checkscope(){
  var scope = "local scope";
  function f(){
    return scope;
  }
  return f;
}
checkscope()();
```

两段代码都会打印'local scope'。虽然两段代码执行的结果一样，但是两段代码究竟有哪些不同呢？

紧接着就在下一篇[《JavaScript深入之执行上下文栈》](https://github.com/liu-fatty/library/blob/master/articles/%E6%B7%B1%E5%85%A5%E7%B3%BB%E5%88%97%E6%96%87%E7%AB%A0/4%E3%80%81JavaScript%E6%B7%B1%E5%85%A5%E4%B9%8B%E6%89%A7%E8%A1%8C%E4%B8%8A%E4%B8%8B%E6%96%87%E6%A0%88.md)中，讲到了两者的区别在于执行上下文栈的变化不一样，然而，如果是这样笼统的回答，依然显得不够详细，本篇就会详细的解析执行上下文栈和执行上下文的具体变化过程。

## 具体分析过程

我们分析第一段代码：

``` js
var scope = "global scope";
function checkscope(){
  var scope = "local scope";
  function f(){
    return scope;
  }
  return f();
}
checkscope();
```

执行过程如下：

1. 执行全局代码，创建全局执行上下文，全局上下文被压入执行上下文栈

``` js
ECStack = [
  globalContext
];
```

2. 全局上下文初始化；同时 checkscope 函数被创建，保存作用域链到函数的内部属性[[scope]]

``` js
globalContext = {
  VO: [global],
  Scope: [globalContext.VO],
  this: globalContext.VO
}

checkscope.[[scope]] = [
  globalContext.VO
];
```

3. 执行 checkscope 函数，创建 checkscope 函数执行上下文，checkscope 函数执行上下文被压入执行上下文栈

``` js
ECStack = [
  checkscopeContext,
  globalContext
];
```

4. checkscope 函数执行上下文初始化：

  1. 赋值函数[[scope]]属性创建作用域链
  2. 用 arguments 创建活动对象
  3. 初始化活动对象，即加入形参、函数声明、变量声明
  4. 将活动对象压入 checkscope 作用域链顶端

同时 f 函数被创建，保存作用域链到 f 函数内部属性 [[scope]]

``` js
checkscopeContext = {
  AO: {
    arguments: {
      length: 0
    },
    scope: undefined,
    f: reference to function f(){}
  },
  Scope: [AO, globalContext.VO],
  this: undefined
}

f.[[scope]] = [
  checkscopeContext.AO,
  globalContext.VO
];
```

5. 执行 f 函数，创建 f 函数执行上下文，f 函数执行上下文被压入执行上下文栈

``` js
ECStack = [
  fContext,
  checkscopeContext,
  globalContext
];
```

6. f 函数执行上下文初始化, 以下跟第 4 步相同：

  1. 赋值函数[[scope]]属性创建作用域链
  2. 用 arguments 创建活动对象
  3. 初始化活动对象，即加入形参、函数声明、变量声明
  4. 将活动对象压入 f 作用域链顶端

``` js
fContext = {
  AO: {
   arguments: {
      length: 0
   }
  },
  Scope: [AO, checkscopeContext.AO, globalContext.VO],
  this: undefined
}
```

7. f 函数执行，沿着作用域链查找 scope 值，返回 scope 值

8. f 函数执行完毕，f 函数上下文从执行上下文栈中弹出

``` js
ECStack = [
  checkscopeContext,
  globalContext
];
```

9. checkscope 函数执行完毕，checkscope 执行上下文从执行上下文栈中弹出

``` js
ECStack = [
  globalContext
];
```

第二段代码就留给大家去尝试模拟它的执行过程。

``` js
var scope = "global scope";
function checkscope(){
  var scope = "local scope";
  function f(){
    return scope;
  }
  return f;
}
checkscope()();
```

不过，在下一篇《JavaScript深入之闭包》中也会提及这段代码的执行过程。

## 下一篇文章

[《JavaScript深入之闭包》](https://github.com/liu-fatty/library/blob/master/articles/%E6%B7%B1%E5%85%A5%E7%B3%BB%E5%88%97%E6%96%87%E7%AB%A0/9%E3%80%81JavaScript%E6%B7%B1%E5%85%A5%E4%B9%8B%E9%97%AD%E5%8C%85.md)
