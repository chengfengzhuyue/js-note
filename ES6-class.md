# ES6  class

<a name="gkhUt"></a>
### 一、简介
JavaScript 语言中，生成实例对象的传统方法是通过构造函数。例如：
```javascript
function Point (x, y) {
	this.x = x;
  this.y = y;
}
Point.prototype.toString = function () {
	return '(' + this.x + ',' + this.y + ')';
};
var p = new Ponit(1, 2);
p.toString();//"(1,2)"
```
这种写法与传统的面向对象（如 c++ 和 java）差异很大。

ES6 引入了 Class （类）这个概念，通过 class 关键字可以定义类。这样更接近传统语言的写法。上例通过 ES6 的类改写：
```javascript
class Point1 {
	constructor(x, y){
  	this.x = x;
    this.y = y;
  }
  toString(){
  	return '(' + this.x + ',' + this.y + ')';
  }
}
var p = new Point1(1, 2);
p.toString();//"(1,2)"
```
上例定义了 Point1 类，里面的 constructor 就是构造方法，相当于 ES5 的 Point 构造函数，this 关键字代表实例对象。Point1 类除了构造方法，还定义了 toString 方法。<br />**注意： 定义类的方法时，不需要在方法前加 function 关键字，方法之间也不需要  ','  逗号分隔，加了会报错。**

<a name="pS1Wy"></a>
#### **1. 类的本质**
```javascript
typeof Point;//"function"
Point === Point.prototype.constructor;//true
```
上例表明，类的数据类型是函数，类本身指向构造函数。类可以看成是构造函数的另一种写法。

<a name="GB03n"></a>
#### 2. 类的 prototype 属性
(1)构造函数的 prototype 属性，类也存在。<br />(2)类的所有方法都是定义在类的 prototype 属性上的。<br />(3)在类的实例上调用方法，其实就是调用原型的方法。
```javascript
class Point {
}
var p = new Point();
p.constructor === Point.prototype.constructor;//true
```

(4)Object.assign 方法可以一次向类添加多个方法。
```javascript
class Point{
	constructor(){}
}
Object.assign(Point.prototype, {
	toString(){},
  toValue(){}
});
```

(5)类的 prototype 属性上的 constructor 属性，指向类本身，与 ES5 一致。
```javascript
Point.prototype.constructor === Point;//true
```

(6)类内部定义的所有方法都不可枚举。（注： 这里与 ES5 的行为不一致。ES5 的构造函数的 prototype 属性上的方法可枚举）
```javascript
class Point {
  constructor(x, y) {}
  toString() {}
}

Object.keys(Point.prototype);// []
Object.getOwnPropertyNames(Point.prototype);// ["constructor","toString"]
```
<a name="DEV1y"></a>
#### 3. 类的 constructor 方法
(1)constructor 方法是类的默认方法，通过 new 生成实例对象时，自动调用该方法。<br />(2) 一个类必须有 constructor 方法，如果没有显示定义，一个空的 constructor 方法会被默认添加。<br />(3)constructor 方法默认返回实例对象（即 this），也可以指定返回另一个对象。
```javascript
class Point {
  constructor() {
    return Object.create(null)
  }
}
const p = new Point();
p instanceof Point;//false
```

(4)类必须使用 new 调用，否则会报错。
```javascript
Point();// Uncaught TypeError: Class constructor Point cannot be invoked without 'new'
```
<a name="K7WOM"></a>
#### 4. 类的实例
使用 new 调用类生成类的实例。<br />(1)与 ES5 一样，实例的属性除非显式定义在其本身（即 this 上），否则都是定义在原型上。
```javascript
class Point {
  constructor(x, y) {
    this.x = x;
		this.y = y;
  }
 toString(){
		return '('+this.x+','+this.y+')';
 }	
}
var p = new Point(1,2);
p.toString();//"(1,2)"
p.hasOwnProperty('x');//true
p.hasOwnProperty('y');//true
p.hasOwnProperty('toString');//false
p.__proto__.hasOwnProperty('toString');//true

//x,y即为实例的属性，toString 是原型的属性
```

(2)与 ES5 一样，类的所有实例共享一个原型对象
<a name="Dg71M"></a>
#### 5. 取值函数（getter）和存值函数（setter）
(1)与 ES5 一样，在类的内部可以使用 get 和 set 关键字，对某个属性设置存值取值函数，拦截该属性的存取值行为。
```javascript
class myClass{
	constructor(){}
	get prop(){
		return 'getter'
	}
	set prop(value){
		console.log('setter: '+value)
	}
}
let inst = new myClass();
inst.prop;///"getter"
inst.prop = 123;// setter: 123
```

(2)存取值函数是设置在属性的 Descriptor 对象上.
```javascript
var descriptor = Object.getOwnPropertyDescriptor(myClass.prototype, 'prop');
'get' in descriptor;//true
'set' in descriptor;//true
```
<a name="BrxL4"></a>
#### 6. 属性表达式
类的属性名，可以使用表达式。
```javascript
var methodName = 'getArea';
class Square{
	constructor(x, y){
		this.x = x;
		this.y = y;
	}
	[methodName](){
		return this.x * this.y
	}
}
var s = new Square(2,3);
s[methodName]();//6
```
<a name="v3gkW"></a>
#### 7. Class 表达式
与函数一样，类也可以使用表达式的形式定义。
```javascript
const MyClass = class Me{
	getClassName(){
		return Me.name;
	}
}
let inst = new MyClass();
inst.getClassName();//"Me"
Me.name;//Uncaught ReferenceError: Me is not defined
```
上例中类的名字是 Me，但是 Me 只能在类的内部使用，指代当前类。在类的外部，这个类只能使用 MyClass 引用。<br />如果类的内部没有用到 Me，可以省略，写成如下形式：
```javascript
const MyClass = class{}
```

使用 Class 表达式的形式，可以写出立即执行的 Class：
```javascript
let person = new class{
	constructor(name){
		this.name = name;
	}
	sayName(){
		console.log(this.name);
	}
}('张三');
person.sayName();// 张三
```
<a name="hXGvC"></a>
#### 8. 注意点
**(1)严格模式**<br />类和模块的内部，默认都是严格模式，不需要使用 "use strict" 运行

**(2)不存在提升**<br />类的内部不存在变量提升，使用类前，必须先定义。（这与继承有关，子类必须在父类之后定义）

**(3)name 属性**<br />由于本质上 类 只是 ES5 的构造函数的一层包装，所以许多函数的特性都被 Class 继承，包括 name 属性。<br />name 属性总是返回紧跟 class 关键字后面的类名。

**(4)Generator 方法**<br />如果在某个函数前面加上星号（*），就表示这是一个 Generator 函数。
```javascript
class Foo{
	constructor(...args){
		this.args = args
	}
	*[Symbol.iterator](){
		for(let arg of this.args){
			yield arg;
		}
	}
}
var foo = new Foo('x', 'y', 'z');
for(let key of foo){
	console.log(key);
}
// x
// y
// z
```

<a name="MKucE"></a>
### 二、静态方法
在类中定义的二分法都会被实例继承，但如果在一个方法前加上 static 关键字，就表示该方法不会被实例继承，而是直接通过类来调用，这就称为 “静态方法”。
```javascript
class Foo{
	static className(){
		return 'hello'
	}
	toString(){
		return 'hello world'
	}
}
var foo = new Foo();
foo.toString();//"hello world"
foo.className();// Uncaught TypeError: foo.className is not a function
Foo.className();//"hello"
```

(1)如果静态方法中包含 this 关键字，那这个 this 指向类，而不是实例。静态方法可以与非静态方法重名。
```javascript
class Foo{
			static bar(){
				this.baz();
			}
			static baz(){
				console.log('hello');
			}
			baz(){
				console.log('world');
			}
}
Foo.bar();// hello
```

(2)父类的静态方法，可以被子类继承。
```javascript
class Foo{
	static classMethod(){
			return 'hello'
	}
			
}
class Bar extends Foo{}
Bar.classMethod();//"hello"
```
<a name="z7NDi"></a>
### 三、实例属性的新写法
实例属性除了定义在 constructor 方法里面的 this 上，也可以定义在类的最顶层，不需要在属性前加 this。
```javascript
class Foo{
			x = 2;
			constructor(y){
				this.y = y;
			}
			get value(){
				return this.x + this.y
			}
}
var foo = new Foo(3);
foo.x;//2
foo.y;//3
foo.value;//5
```
<a name="54puf"></a>
### 
