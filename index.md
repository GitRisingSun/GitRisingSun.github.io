
## 一、概述

之前提到的几种模块化规范：CommonJS、AMD、CMD都是社区提出的。ES 2015在语言层面上实现了模块功能，且实现简单，可以替代CommonJS和AMD规范，成为在服务器和浏览器通用的解决方案

## 二、特性

#### 1、ES Module自动启用严格模式

```javascript
 <script type="module">
        console.log(this); //undefined
  </script>
```

#### 2、ES Module运行在单独的作用域中，与外界互不干扰

```javascript
    <script type="module">
        var foo = 100;
        console.log(foo);//100
    </script>
    <script type="module">
        console.log(foo);//Uncaught ReferenceError: foo is not defined
    </script>
```

#### 3、ES Module是通过CORS方式请求外部文件，需外部文件支持CORS请求

```html
 //改文件支持CORS请求，则可以请求成功
<script type="module" src="http://code.jquery.com/jquery-migrate-1.2.1.min.js"></script>

//报跨域错误
<script type="module" src="http://libs.baidu.com/jquery/2.0.0/jquery.min.js"></script> 
```

#### 4、ES Module引用的文件会延迟执行

```
//index.js
alert('hello');
```



```html
//index.html
<body>
	<script type="module" src="./index.js"></script>
    等价于
    <script defer src="./index.js"></script>
	<p>
       文本内容
    </p>
</body>
```

执行结果【js的加载并未影响到DOM的加载】

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fd7fda6755ae419c982a0dd4c95a0c68~tplv-k3u1fbpfcp-watermark.image)

## 三、export命令

由于在ES Module中，每个模块都是一个单独的作用域，如果想使模块内的变量暴露出去，使用**export**关键字，导入其他模块的关键字使用**import**。

#### 1、挨个导出变量

```javascript
//module.js
export var name = "张三";

export function add(a, b) {
    return a + b;
}

export var obj = {
    name: 'jack'
};
```

#### 2、批量导出变量

```javascript
//module.js
var name = "张三";

function add(a, b) {
    return a + b;
}

var obj = {
    name: 'jack'
};

export { name, add, obj }
```

优点：在脚本尾部，使用一个export统一导出，清晰简洁

#### 3、导出默认数据

```javascript
//module.js
export default 'es module'
或者
export default function(a,b){
	return a+b;
}
```

**注意：**说完import后再说注意事项




#### 4、导出时起别名，使用as关键字

```javascript
//module.js
var name = "张三";

function add(a, b) {
    return a + b;
}

var obj = {
    name: 'jack'
};

export { name as v1, add as v2, obj as v3}
```

## 四、import命令

要想接受其他模块通过export导出的成员时，使用import关键字

#### 1、import导入其他模块中的变量

```javascript
//module.js
var name = "张三";

function add(a, b) {
    return a + b;
} 

export { name ,add }
```

```javascript
//app.js
import { name, add as MathAdd} from "./module.js"
console.log(name);//张三
console.log(MathAdd(1, 1)) //2
```

注意：

##### 	1.1使用相对路径 后缀名和./不能省略

​	import {name} from "./module.js"

##### 	1.2使用绝对路径

​	import {name} from "/demo/module.js"

##### 	1.3使用全路径

​	import {name} from "http://localhost:8080/demo/module.js"

#####     1.4可以在导入模块时使用as 起别名

​     起别名后，as前的变量不可使用了

####  2、使用import只执行引入文件，不提取引入文件中的成员时

1. import {} from "./module.js";
2. import './module.js'【常用】

```javascript
//module.js
console.log('module.js中执行')

var name = "张三";

function add(a, b) {
    return a + b;
}

var obj = {
    name: 'jack'
};

export { name, add, obj }
```

```javascript
//app.js
import {} from "./module.js";
或者写成
import './module.js'
```

```html
//index.html
<body>
    <script type="module" src="./app.js"></script>
    //执行结果
    //module.js中执行
</body>
```

当多次执行同一个import语句时，只会执行一次

#### 3、模块的整体加载

使用import除了可以加载单个值，也可以一次性加载模块导出的所有值

```javascript
//module.js

var name = "张三";

function add(a, b) {
    return a + b;
}

var obj = {
    name: 'jack'
};

export { name, add, obj }
```

```javascript
//app.js
import * as all './module.js'
console.log('name:',all.name);
console.log('obj',all.obj);
console.log(add(1,1))
```

## 五、export default

前面使用export导出模块内的成员时，需要指定具体的成员名称，同样加载时需要根据导出的具体名称进行加载。还有一种快捷的导出方式，使用export default 为模块指定默认导出，**每个模块只能使用一次export default**。

#### 1、导出匿名成员时，使用任意变量接受成员

```javascript
//module.js
export default function(a, b) {
    return a + b;
}
```

```javascript
 //app.js
 import add from "./module.js"; 
 console.log(add(1,2));
```

因为使用export导出成员时是匿名的，所以在导入时并不知道这个成员的名字是什么，这时候就可以随意写变量去接受这个匿名成员，案例中使用add接受的匿名函数，也可以使用其他名称接收。

**注意：export default也可以导出具名成员，但效果和导出匿名成员是一样的**

```javascript
//module.js
export default function add(a, b) {
    return a + b;
}
//或者
function add(a,b){
    return a + b;
}
export default add
```

```javascript
 //app.js
 import temp from "./module.js"; 
 console.log(temp(1,2));
```

**总结：使用export default导出成员，在外部加载时都视为匿名成员加载，可以随意起变量名接受**

#### 2、同时导入其他模块的默认成员和具名成员时

```javascript
//module.js
var name = 'jack';
var age = 19;
export { name, age } 

//导出默认成员
export default 'default export'
```

```javascript
//app.js
import str,{name,age} from "./module.js";
或者
import {name,age,default as str} from "./module.js"
console.log(str);//default export
console.log(name);//jack
console.log(age);//19
```

#### 3、另类的默认导出

```javascript
//module.js
var number = 1;
export { number as default }
等同于
export default number;
```

```javascript
//app.js
import { default as number} from "./module.js"
console.log(number); //1
```

**总结：可见export default的本质就是导出一个名字叫default的成员，当导出的成员叫default时，接受这个成员可以随意命名**

#### 4、使用export default来导出类

```javascript
//Person.js
export default class Person(){
    ....
}


//main.js
import Person from "./Person.js";
const person = new Person();
```

#### 5、比较一下默认输出和正常输出

```javascript
//1.默认输出
export default function add(a,b){
    return a + b;
}
//接受默认输出
import temp from './module.js'

//2.具名输出
function add (a,b){
    return a + b;
}
export {add}
//接受具名输出
import {add} from './module.js'
```

**总结：使用默认导出时，在外部接受成员，不需要使用大括号；使用具名输出时，在外部接受成员时，import后需要使用大括号**

## 六、迷惑性的点

#### 1、export批量导出变量时，导出的并不是对象字面量

```javascript
//module.js
var name = 'jack';
var age = 19;
export { name, age } 
//export {name,age} 并不是导出的一个对象，而是export批量导出的语法
```

#### 2、import加载多个变量时，并不是ES6的解构用法

```javascript
import {name,age} from "./module.js"
//impot后面的大括号并不是解构作用
```

#### 3、import导入的变量都是常量，不可修改，与CommonJS不同

## 七、export和import的复合用法

当我们在文件内import一个成员后，同时把它导出时

```javascript
//index.js
import {Button} from './button.js'
export { Button }

可以写成

export {Button} from "./button.js"//具名导出
export { default } from "./button.js"//匿名导出
```

## 八、import()函数

import关键字导入其他模块成员时，必须写在最顶层作用域，不可嵌套在其他逻辑处理中。如果说我们想在某段逻辑执行完成后去动态加载某些成员，这时候可以使用import()函数加载

```javascript
//module.js
var name = "张三";

function add(a, b) {
    return a + b;
}

var obj = {
    name: 'jack'
};

export { name, add, obj }
export default function() {
    return '匿名成员'
}
```

```javascript
//app.js
//使用setTimeout来模拟一个异步加载的过程
setTimeout(() => {
    //import返回的是个promise，返回的模块内容都在回调函数的参数中（moduleResult）
    import ("./module.js").then(moduleResult => {
        console.log(moduleResult)
        console.log(moduleResult.add)
        console.log(moduleResult.name)
        console.log(moduleResult.default)
        console.log(moduleResult.obj)
    })
    
    //也可以直接把成员结构出来
       import ("./module.js").then(({add,name,obj,default:defaultData}}) => { 
        console.log(moduleResult.add)
        console.log(moduleResult.name)
        console.log(moduleResult.defaultData)
        console.log(moduleResult.obj)
    })
}, 2000);
```



