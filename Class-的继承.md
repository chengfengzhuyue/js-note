# Class 的继承

<a name="9Jv6G"></a>
### 一、extends 
class 可以通过 extends 关键字实现继承，这比 ES5 通过修改原型链实现继承要清晰和方便。
```javascript
class Foo{
	constructor(){}
    static hello() {
    	console.log('hello world');
  	}
	toString(){
			return 'hello'
	}		
}
class Bar extends Foo{
	constructor(x){
    	super();//调用父类的 constructor()
        this.x = x;
    }
}
var bar = new Bar(1);
bar.toString();//"hello"
bar.x;//1
Bar.hello();// hello world   //子类会继承父类的静态方法
```

<a name="7Krl9"></a>
### 二、Object.getPrototypeOf()
Object.getProtitypeOf 方法用来从子类上获取父类。也可以用来判断一个类是否继承了另一个类。
```javascript
Object.getPrototypeOf(Bar) === Foo;//true
```

<a name="1Nkcx"></a>
### 三、super 关键字
super 关键字，既可以当做函数使用，也可以当做对象使用。
<a name="p0bt7"></a>
#### 1. super 作为函数使用
super 作为函数调用时，**代表父类的构造函数**。ES6 要求，**子类的构造函数必须执行一次 super 函数，**否则新建实例时会报错。
```javascript
class A {}
class B extends A {
  constructor() {
    super();
  }
}

class Baz extends Foo{
	constructor(){}
}
var baz = new Baz();// Uncaught ReferenceError: Must call super constructor in derived class before accessing 'this' or returning from derived constructor
```
这是因为 ES6 的继承中，子类自己的 this 对象必须先通过父类的构造函数完成塑造，得到与父类同样的实例属性和方法，然后再加上子类自己的实例属性和方法。不调用 super 方法，子类就得不到 this 对象。所以子类构造函数中，必须在调用 super() 之后，才能使用 this 关键字，否则会报错。

**注意： 作为函数时，super() 只能在子类的构造函数中使用，用在其他地方会报错。**
<a name="2Gp6Z"></a>
#### 2. super 作为对象使用
super 作为对象时，在普通方法中，指向父类的原型对象；在静态方法中指向父类。
```javascript
class A{
  static p(){
		return 1
	}
	p(){
		return 2
	}
}
class B extends A{
	static p(){
		console.log(super.p())
	}
	p(){
		console.log(super.p())
	}
}
B.p();// 1
var b = new B();
b.p();// 2
```

注意：<br />(1)在子类普通方法中通过 spuer 调用父类方法时，方法内部的 this 指向当前的子类实例。<br />(2)在子类的静态方法中通过 super 调用父类的方法时， 方法内部的 this 指向子类。
```javascript
class A{
	constructor(){
		this.x = 1;
	}
    static p() {
        console.log(this.x);
    }
	p(){
		console.log(this.x);
	}
}
class B extends A{
    constructor(){
      super();
      this.x = 2;
    }
    static m() {
      super.p();
    }
    m(){
      super.p();
    }
}
B.x = 3;
B.m();//3
var b = new B();
b.m();// 2

```




