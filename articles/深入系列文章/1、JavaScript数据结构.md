![image](https://user-images.githubusercontent.com/70930284/100418999-99f7b780-30be-11eb-999c-dbe667ce92a6.png)

# JavaScript 中的数据类型

* 基本类型：String / Number / Boolean / Null /Undefined / Symbol(es6) / BingInt(es10)
* 引用类型：Object / Array / Function

## 堆和栈

* 程序运行的时候，需要内存空间存放数据。一般来说，系统会划分出两种不同的内存空间：一种叫做堆（heap），另一种叫做栈（stack）；
* 堆（heap）是没有结构的，数据可以任意存放，它是用来**存放复杂数据类型**（引用类型）的，例如数组对象、object对象等；
* 栈（stack）是有结构的，每个区块按照一定次序存放（先进后出，后进先出），**栈中主要存放的是基本类型的变量的值以及指向堆中的引用类型数据的地址**，存在栈中的数据大小与生存期必须是确定的。除此之外，还可以明确知道每个区块的大小，因此，stack 的寻址速度要快于 heap；
* 栈创建的时候，大小是确定的，如果超出了浏览器规定的栈限制，就会报 stack overflow 错误，而堆的大小是不确定的，需要的话可以不断增加；

### 基础数据类型

1. 值是不可变的；
2. 存放在栈区中；
3. 值得比较；
> `==`：只进行值的比较，会进行数据类型的转换
> `===`：不仅进行值的比较，还会进行数据类型的比较

### 引用类型数据

1. 值是可变的；
2. 指针是保存在栈内存，值是保存在堆内存；
![image](https://user-images.githubusercontent.com/70930284/100433784-86f0e180-30d6-11eb-8942-bff3066f4209.png)
3. 比较是引用的比较；

### `undefined` 和 `null` 的区别

相同：
* 都是基本类型；
* 都是虚值，`Boolean(value)` 或者 `!!value` 转换成布尔值时为false；

不同：
* `undefined` undefined表示"缺少值"，就是此处应该有一个值，但是还没有定义；
* `null` 表示"没有对象"，即该处不应该有值；
* 在比较 `undefined` 和 `null` 时，使用 `==` 得到 `true`，使用 `===` 得到 `false`；

``` js
console.log(null == undefined); // true
console.log(null === undefined); // false
```
