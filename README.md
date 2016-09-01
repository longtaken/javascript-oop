# javascript-oop
JavaScript oop编程
##创建对象
工厂模式
函数内容创建对象，对象上绑定属性跟方法，返回这个对象。解决了创建多个对象的问题。
缺点：无法知道对象的类型
<code>
function createPerson(n,a){
	var o=new Object();
	o.n=n;
	o.a=a;
	o.export=function(){
		console.log(this.n);
	}
	return o;
}
var person=createPerson("n1","a1");
</code>

构造函数模式
构造函数里绑定对象跟方法，使用new实例化对象。跟普通函数不同的地方在于调用方式，使用new调用就是构造函数。
缺点：公用方法只能在全局绑定
function Person(n,a){
	this.n=n;
	this.a=a;
	this.export=function(){
		console.log(this.n);
	}
}
var person=new Person("n1","a1");


原型模式
在构造函数上添加prototype
缺点：实例在所有情况下都取相同的属性值，引用类型的修改会引起其他实例的改变
function Person(){}
Person.prototype.n="n1";
Person.prototype.a="a1";
Person.prototype.export=function(){console.log(this.n);};
var person=new Person();


组合构造函数模式和原型模式
构造函数用于定义实例属性，原型上定义方法及公用属性
function Person(n,a){
	this.n=n;
	this.a=a;
	this.single=[1,2];
}
Person.prototype={
	constructor:Person,
	export:function(){console.log(this.n);}
}
var per1=new Person('n1','a1');
var per2=new Person('n2','a2');
per1.single.push(3);
console.log(per1.single);//1,2,3
console.log(per2.single);//1,2


动态原型模式
把所有信息封装在构造函数中，在构造函数中定义原型。判断某个存在的方法是否存在来决定是否初始化原型
function Person(n,a){
	this.n=n;
	this.a=a;
	if(typeof this.export != "function"){
		Person.prototype.export=function(){
			console.log(this.n);
		};
	}
}
var person=new Person('n1','a1');
person.export();


寄生构造函数模式
寄生构造函数模式和工厂模式是一模一样的，只不过创建对象的时候使用了new 关键字。（用于为原生构造函数添加特殊的属性、方法）
function SpecialArray(){
	var arrs=new Array();
	arrs.push.apply(arrs,arguments);//按数组的形式，args是伪数组
	//arrs.push(arguments);
	arrs.toPipedString=function(){
		return this.join("|");
	};
	return arrs;
}
var colors=new SpecialArray("red","blue","green");
var colors2=new SpecialArray("orange","pink","black");
console.log(colors.toPipedString());


稳妥构造函数模式
稳妥对象最合适在一些安全的环境中（禁止使用this和new），或者在防止数据被其他应用程序改动时使用。
稳妥构造函数遵循与寄生构造函数类似的模式，但有两点不同：
1，新创建的对象的实例方法不引用this
2，不适用new操作符调用构造函数
function Person(n,a){
	var o=new Object();
	o.export=function(){
		console.log(n);//没有绑定到o上
	};
	return o;
}
除了使用export之外没有其他办法访问n的值（工厂模式通过o.n）


##继承
OO语言一般支持两种继承方式：接口继承和实现继承
ECMAscript只支持实现继承（主要依靠原型链）
原型链基本模式
function a(){
	this.aaproperty=true;
}
a.prototype.getaValue=function(){
	return this.aaproperty;
}
function b(){
	this.bb=false;
}
b.prototype=new a();//{aaproperty:true}
b.prototype.getbValue=function(){
	return this.bb;
}
var instance=new b();
console.log(instance.getaValue);

确定原型和实例的关系
instanceof操作符来测试实例与原型链中出现过的构造函数，只要出现过就返回true
instance instanceof Object//true
instance instanceof a//true
instance instanceof b//true

isPrototypeOf方法,只要是原型链中出现过的原型，都可以说是该原型链派生的实例的原型。
Object.prototype.isPrototypeOf(instance)//true
a.prototype.isPrototypeOf(instance)//true
b.prototype.isPrototypeOf(instance)//true

原型链的问题
引用类型值的原型属性会被所有实例共享
创建子类型的实例是，不能向超类型的构造函数中传递参数（会影响对象实例）
function A(){
	this.c=[1,2,3];
}
function B(){}
B.prototype=new A();//引用类型
var in1=new B();
in1.c.push(4);
var in2=new B();
console.log(in1.c,in2.c);//[1,2,3,4]

借用构造函数
function A(){
	this.c=[1,2,3];
}
function B(){
	A.call(this);
}
var in1=new B();
in1.c.push(4);
var in2=new B();
console.log(in1,in2);//[1,2,3,4],[1,2,3]

传递参数
借用构造函数可以在子类型构造函数中向超类型构造函数传递参数
function A(n){
	this.n=n;
}
function B(){
	A.call(this,"n1");
	this.a='a1';
}
var ins1=new B();
console.log(ins1.n,ins1.a);
缺点：方法都在构造函数内，无法复用。在超类型的原型中定义的方法，对子类型而言是不可见的。


组合继承（伪经典继承）
原型链和借用构造函数技术组合。使用原型链实现对原型属性和方法的继承，通过构造函数来实现对实例属性的继承。
function A(n){
	this.n=n;
	this.c=[1,2,3];
}
A.prototype.exportA=function(){
	console.log(this.n);
};
function B(n,a){
	A.call(this,n);
	this.a=a;
}
B.prototype=new A();
B.prototype.exportB=function(){
	console.log(this.a);
};
var ins1=new B('bbb11','bbb22')
ins1.c.push(4);
console.log(ins1.c,"ins1");
ins1.exportA();
ins1.exportB();
var ins2=new B('bbb33','bbb44');
console.log(ins2.c,"ins2");
ins2.exportA();
ins2.exportB();


原型式继承
function createobj(o){
	function F(){}
	F.prototype=o;
	return new F();
}
var person={
	n:'nn',
	f:[1,2,3]
};
var a2=createobj(person);

ES5 Object.create()
接受两个参数，一个用作新对象原型的对象和一个为新对象定义额外属性的对象，在传入一个参数的情况下，Object.create()和createobj相同
var p={n:'n1',f:[1,2,3]};
var an=Object.create(p,{n:{value:'n2'}});
console.log(an);


寄生式继承
创建一个仅用于封装继承过程的函数，该函数在内部已某种方式来增强对象，最后返回函数
function createAn(ori){
	var clone=createobj(ori);
	clone.export=function(){
		console.log("parasitic");
	}
	return clone;
}
createobj函数不是必须的，任何能返回新对象的函数都使用于此模式
使用寄生式继承来为对象添加函数，会由于不能做到函数复用而降低效率


寄生组合式继承
通过借用构造函数来继承属性，通过原型链的混成形式来继承方法。
组合继承最大问题是会调用两次超类型构造函数:一次是创建子类型原型的时候，另一次是在子类型构造函数内部
function A(n){
	this.n=n;
	this.c=[1,2,3];
}
A.prototype.export=function(){
	console.log(this.n);
}
function B(n,a){
	A.call(this,n);//第二次调用A
	this.a=a;
}
B.prototype=new A();//第一次调用A
B.prototype.constructor=B;
B.prototype.saya=function(){
	console.log(this.a);
};

寄生组合式继承最简单形式
function inheritpro(A,B){
	var pro=createobj(B.prototype);//创建对象
	pro.constructor=A;//增强对象
	A.prototype=pro;//制定对象
}
这个函数接受两个参数：子类型构造函数和超类型构造函数。第一步是创建超类型原型的一个副本。第二部是为创建的副本添加constructor属性。第三步，将新创建的对象复制给子类型的原型
function super(n){
	this.n=n;
	this.c=[1,2,3];
}
super.prototype.sayn=function(){
	console.log(this.n);
};
fucntion sub(n,a){
	super.call(this,n);
	this.a=a;
}
inheritpro(sub,super);
sub.prototype.saya=function(){
	console.log(this.a);
};
这个例子的高效率体现在它只调用一次super构造函数，并避免在sub.prototype上创建不必要的属性。原型链也能保持不变。能正常使用instanceof和isPrototypeOf



