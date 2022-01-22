# 前端模块化之CommonJS

##    一、CommonJS特点

​      经过前面讨论，已经知道无模块化时项目中存在的问题。CommonJS的特点就是解决这些问题即：

​       1.每个文件都是一个单独的模块，有自己的作用域，声明的变量不是全局变量(除非在模块内声明的变量挂载到global上)

​       2.每个文件中的成员都是私有的，对外不可见

​       3.A模块依赖B模块时，在A模块内部使用require函数引入B模块即可，模块之间依赖关系更加清晰

​       4.模块的加载有缓存机制，当加载完一次后，后续再加载就会读取缓存中的内容

​       5.模块的加载顺序是按照代码的书写顺序来加载

##    二、CommonJS的应用环境

​        应用在Node.js中。CommonJS的加载机制是同步的，在Node环境中模块文件是存在本地硬盘中，所以加载起来比较快，不用考虑异步模式。

##    三、CommonJS的用法

​        已经知到CommonJS规范下，每个文件就是一个模块，都有私有作用域，那如何才能让外部访问某个模块中的内容呢？

​       方式1：把模块中的成员挂载到global全局对象中   【非常不推荐】

​       方式2：使用模块的成员module.exports或者exports导出成员

​       最后在外部使用require函数引入所需模块

###   方式1：把变量挂载到global模块下【看完忘掉即可】

```javascript
//module-b.js
var a = 10;
global.a = a;
```

```javascript
//module-a.js
require("./module-b.js");
console.log(a); //10
```

### 方式2：使用module.exports导出成员

####  1、使用module.exports单个导出

```javascript
//module-b.js
const a = 10;
const add = function(a, b) {
    return a + b;
}
module.exports.a = a;
module.exports.add = add;
```

```javascript
//module-a.js
const moduleB = require("./module-b.js");

console.log(moduleB.a); //10
console.log(moduleB.add(1, 1)); //2
```

####   2、使用module.exports直接导出一个对象

```javascript
//module-b.js
const a = 10;
const add = function(a, b) {
    return a + b;
}
module.exports={a,add};
```

```javascript
//module-a.js
const moduleB = require("./module-b.js");

console.log(moduleB.a); //10
console.log(moduleB.add(1, 1)); //2
```

####   3、使用exports代替module.exports来导出成员

```javascript
//module-b.js
const a = 10;
const add = function(a, b) {
    return a + b;
}
exports.a=a;
exports.add=add;
//使用exports时 不可使用下面这种方式导出成员
exports={a,add}
```

```javascript
//module-a.js
const moduleB = require("./module-b.js");

console.log(moduleB.a); //10
console.log(moduleB.add(1, 1)); //2
```

由代码可见，在使用module.exports和exports导出成员时略有不同，具体是为什么呢？稍后作出解释

## 四、module.exports、exports和require为什么可以直接使用？

从我们平时写代码的经验来看，在一个文件中可以使用的成员由以下几种情况：

  1、全局成员

   2、在文件内部声明了该成员

但我们所了解的代码运行环境中的全局成员只有一个，像window和global这种，那大概率不是全局成员。而且我们在模块内部并未声明这三个变量，那为何能直接使用呢？

**其实在node运行环境中，每个模块都是运行在一个函数中，正是因为这个函数的存在，才让每个模块有了私有作用域**

```javascript
(function (exports, require, module, __filename, __dirname) {
  // HERE IS YOUR CODE
});
```

**通过代码来证明一下这个函数的存在**

既然我们写的代码都在函数内部，那我们应该通过**arguments**能获取到这个函数的参数

```JavaScript
//module-a.js
console.log('模块中的第一句代码');
console.log(arguments.length)

//运行结果
模块中的第一句代码
5
```

arguments.length的值是5，那八成就是这个样子了。但感觉说服力不强，继续看...

```javascript
//module-a.js
console.log('模块中的第一句代码');
console.log(arguments.callee.toString())

//运行结果
模块中的第一句代码
function (exports, require, module, __filename, __dirname) {
    console.log('模块中的第一句代码');
    console.log(arguments.callee.toString())
}
```

终于露出了庐山真面目，为什么可以直接用，应该一目了然了！（可以尝试打印一下这个五个参数中都是什么内容）

## 五、module.exports和exports

通过打印module成员，可以看到exports是module下的一个对象。module的exports属性表示当前模块对外输出的桥梁，module.exports指向的成员都会被暴露出去

以上示例中的写法都是把模块内的成员挂载到module.exports中暴露出去的

```javascript
const a = 10;
const add = function(a, b) {
    return a + b;
}
module.exports.a = a;
module.exports.add = add;
或者使用
module.exports={a,add}
```

exports又如何导出的呢？

```javascript
//module-a.js
console.log(module.exports)
console.log(exports)
//输出结果
{}
{}
```

两个成员都是对象，那会不会是同一个东西呢？

```
//module-a.js
console.log(module.exports)
console.log(exports)
console.log(module.exports===exports)
//输出结果
{}
{}
true
```

可见两个成员完全相等，则指向的堆内存的地址是同一个，所以使用exports导出模块内的成员也是理所应当的了。

所以模块最外部函数应该是有这么一句代码的

```javascript
(function (exports, require, module, __filename, __dirname) {
    exports=module.exports={}; //指向同一个内存地址
  // HERE IS YOUR CODE
});
```

既然exports和module.exports是指向的是同一个内存，按说用法是一样的，为什么上边使用exports导出成员时，特意说明不可以使用exports直接导出一个对象呢？不妨试一下：

```javascript
//module-b.js
const a = 10;
const add = function(a, b) {
    return a + b;
}

// module.exports = { a, add };
exports = { a, add};
```

```javascript
const moduleB = require("./module-b.js");

console.log(moduleB)//{}
console.log(moduleB.a)//undefined
```

实验得出，通过exports直接导出一个对象时在外部并拿不到导出的数据，为什么呢？

看一下module.exports和exports在内存中的情况

当加载该模块时，执行完exports=module.exports={}后的内存情况

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8505b0397703461c8ce272720b282e51~tplv-k3u1fbpfcp-watermark.image)

当执行完exports={a,add}时的内存情况

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/99b8aed09b8941779c7343c7f15c0177~tplv-k3u1fbpfcp-watermark.image)

当exports={a, add}时，exports在内存中和module.exports指向的就不是同一个内存地址了，说白了抱不了module.exports的大腿了，咱们上面说过，模块导出成员是通过module.exports导出的。exports和module.exports不是同一个内存时，exports自然无法导出成员了。

既然如此那就把导出的成员老老实实挂载到exports下吧。整洋气一些，module.exports和exports同时使用

```javascript
//module-b.js
const a = 10;
const add = function(a, b) {
    return a + b;
}

module.exports = add;
exports.a = a;
```

```javascript
//module-a.js
const moduleB = require("./module-b.js");

console.log(moduleB)//Function
console.log(moduleB.a)//undefined
```

纳尼？？？把导出的成员挂载到exports下了为何引用的时候还是undefined？？？

注意：在使用exports.a=a前 使用了module.exports=add了，这时候使用exports为什么导不出成员，大家应该都明白了【原因同上】

**为了避免在导出成员时，有这样或那样的问题，建议在模块中全部使用module.exports吧**

## 六、require()

通过上面一系列的代码案例可以看出，require的作用是加载所依赖的文件。说白了就是执行了所加载模块的最外层的函数。

```javascript
//module-b.js
const a = 10;
console.log('module-b中打印', a)
module.exports = { a };
```

```javascript
//module-a.js
const moduleB = require("./module-b.js");
console.log(moduleB);
```

```
执行module-a.js的结果：

module-b中打印 10
{ a: 10 }
```

module-b.js中的console.log执行了。可见require函数确实令模块最外部的函数执行了。

由require执行完后有个参数来接受返回值看出，模块最外部的函数执行完后是有返回值的，那么模块最外部的函数应该是这个样子：

```
(function (exports, require, module, __filename, __dirname) {
  exports = module.exports = {}; //指向同一个内存地址
  // HERE IS YOUR CODE
  return module.exports;//把module.exports返回出去，同时module.exports下挂载的成员也返回出去了
});
```

### 1、require加载文件的方式（后缀名默认是js）

​    （1）通过相对路径加载模块，以"./"或"../"开头。比如：require("./module-b")是加载同级目录下的module-b.js

​    （2）通过绝对路径加载模块，以"/"开头，这时候会去磁盘盘符根目录或者网站根目录去找此模块。比如：require("/module-b"),此时      			 会去根目录去找module-b.js

​    （3）直接通过模块名加载,比如：require("math")，这是加载提供node内置的核心模块或者node_modules目录中安装的模块。当加			 载的是node_modules中的模块时，查找规则如下   

```javascript
//文件所在目录  E:/Work/Module/CommonJs/module-a.js
const math = require("math");
查找规则：【假设每次在对应的目录下都没找到math模块】
1、先去CommonJS目录下的node_modules中去查找math模块
2、再去Module文件夹下的node_modules中去查找math模块
3、再去Work文件夹下node_modules中去查找math模块
4、再去E盘下的node_modules中去查找math模块
最顶级目录还找不到的话则报错...
```

### 2、模块的加载机制

CommonJS模块的加载机制是，引入的值是输出的值的拷贝，一旦模块内的值导出后，在外部如何改变都不会影响模块内部，同样模块内部对这个值如何改变也不会影响模块外部，犹如“嫁出去的女儿，泼出去的水”




