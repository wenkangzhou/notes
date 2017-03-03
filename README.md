```javascript
/*************
     笔记
**************/
ECMAScript包含数据属性和访问属性
	1.数据属性：4个特性[[Configurable]]、[[Enumerable]]、[[Writable]]、[[Value]] 
	例子：
	var person = {};
	Object.defineProperty(person,"name",{
		writable: false,
		value:"Jimmy"
	})
	alert(person.name); //"Jimmy"
	person.name = "Jason";
	alert(person.name)//Jimmy

	一旦配置了Configurable，就无法修改，尝试修改会在严格模式报错	
	var person = {};
	Object.defineProperty(person,"name",{
		configurable: false,
		value:"Jimmy"
	})
	//抛出错误
	var person = {};
	Object.defineProperty(person,"name",{
		configurable: true,
		value:"Jimmy"
	})

	2.访问器属性：4个特性[[Configurable]]、[[Enumerable]]、[[Get]]、[[Set]] 
	var book = {
		_year: 2014,
		edition；1
	};

	Object.defineProperty(book,"year",{
		get:function(){
			return this._year;
		},
		set: function(newValue){
			if(newValue > 2004){
				this._year = newValue;
				this.edition += newValue -2004;
			}
		}
	});
	book.year =2005;
	alert(book.edition);//2

定义多属性：defineproperties

读取属性：getOwnPropertyDescriptor
_________

创建对象

	1.工厂模式
		function createPerson(name, age, job){
			var o = new Object();
			o.name = name;
			o.age = age;
			o.job = job;
			o.sayName = function(){
				alert(this.name);
			}
			return o;
		}
		var person1 = createPerson("Jimmy", 29, "Front-end Engineer");
		var person2 = createPerson("David", 27, "Front-end Engineer");
	2.构造函数模式
		function Person(name, age, job){
			this.name = name;
			this.age = age;
			this.job = job;
			this.sayName = function(){
				alert(this.name);
			}
		}
		var person1 = new Person("Jimmy", 29, "Front-end Engineer");
		var person2 = new Person("David", 27, "Front-end Engineer");

			//当作构造函数使用
			var person = new Person("Jimmy", 29, "Front-end Engineer");
			person.sayName();//"Jimmy"
			//作为普通函数调用
			Person("Jimmy", 29, "Front-end Engineer");//增加到window
			window.sayName();//"Jimmy"
			//在另一个作用域调用
			var o = new Object();
			Person.call(o,"Jimmy", 29, "Front-end Engineer");
			o.sayName();//"Jimmy"
		构造函数的问题：不同实例上的同名函数式不相等的
			alert(person1.sayName == person2.sayName)//false
		解决：把函数定义转移到构造函数外
			function Person(name, age, job){
				this.name = name;
				this.age = age;
				this.job = job;
				this.sayName = sayName;
			}
			function sayName(){
				alert(this.name)
			}
			var person1 = new Person("Jimmy", 29, "Front-end Engineer");
			var person2 = new Person("David", 27, "Front-end Engineer");
		不过这样要定义很多的全局函数，肯定是不妥的，so，还好有原型模式来解决。
	3.原型模式
		function Person(){

		}
		Person.prototype.name="Jimmy";
		Person.prototype.age=29;
		Person.prototype.name="Front-end Engineer";
		Person.prototype.sayName=function(){
			alert(this.name)
		}
		var person1 = new Person();
		person1.sayName();//"Jimmy"
				
		var person2 = new Person();
		person2.sayName();//"Jimmy"
		
		alert(person1.sayName == person2.sayName)//true

		在默认情况下，所有原型对象都会自动获得一个constructor(构造函数)属性，这个属性
		包含一个指向prototype属性所在函数的指针。

		也就是我们在调用person1.sayName()的时候，会先后执行两次搜索。
		首先，解析器会问：“实例person1”有sayName属性吗？答：“没有”。
		然后，它继续搜索，再问：“person1”的原型有sayName属性吗。答：“有”。
		于是它就读取在原型中的函数。当我们调用person2.sayName()，会重复上述步骤，
		这正是对象实例共享原型所保存的属性和方法的基本原理。


		先实例后原型
		function Person(){

		}
		Person.prototype.name="Jimmy";
		Person.prototype.age=29;
		Person.prototype.name="Front-end Engineer";
		Person.prototype.sayName=function(){
			alert(this.name)
		}
		var person1 = new Person();
		var person2 = new Person();
		person1.name = "Jason";
		alert(person1.name);//Jason-来自实例
		alert(person2.name)//Jimmy-来自原型

		delete person1.name;
		alert(person1.name);//Jimmy-来自原型

		hasOwnProperty()方法检测一个属性是否存在于实例中

		in操作符会在通过对象能够访问给定属性时返回true

		alert(person1.hasOwnProperty("name"))//false
		alert("name" in person1)//true

		Object.keys()和Object.getOwnPropertyNames()都可以替代for-in循环
		区别是后者的结果中包含了constructor属性
		function keys(obj){
			var a = [];
			for(a[a.length] in obj);
			return a;
		}
		更简单的原型语法
		function Person(){

		}
		Person.prototype = {
			constructor : Person,//需要特意设一下||重写时会切断与最初原型的指针
			name : "Jimmy",
			age : 29,
			job :"Front-end Engineer",
			sayName : function(){
				alert(this.name)
			}
		}
		Object.defineProperty(Person.prototype,"constructor",{
			enumerable: false,//enumerable被设为true，要手动设回来
			value:Person
		})
		原生对象的原型
		String.prototype.startsWith =function(text){
			return this,indexOf(text) == 0;
		}
		var msg = "Hello World";
		alert(msg.startsWith("Hello"));//true

		原型对象的问题：所有属性被很多实例所共享，对于函数当然非常合适，但对于引用类型，问题就比较突出了。

		function Person(){

		}
		Person.prototype = {
			constructor : Person,
			name : "Jimmy",
			age : 29,
			job :"Front-end Engineer",
			friends : ["cate","lucy"]
			sayName : function(){
				alert(this.name)
			}
		}
		var person1 = new Person();
		var person2 = new Person();

		person1.friends.push("lily");

		alert(person1.friends);//"cate,lucy,lily"
		alert(person2.friends)//"cate,lucy,lily"
		alert(person1.friends === person2.friends)//true

————————————————————到这里的总结——————————————————————————
		构造函数适合定义实例属性，原型适合定义方法和共享的属性
————————————————————————结束——————————————————————————————
	4.组合使用构造函数模式和原型模式

	function Person(name, age, job){
		this.name = name;
		this.age = age;
		this.job = job;
		this.friends = ["cate","lucy"];
	}

	Person.prototype = {
		constructor : Person,
		sayName : function(){
			alert(this.name)
		}
	}
	var person1 = new Person("Jimmy", 29, "Front-end Engineer");
	var person2 = new Person("David", 27, "Front-end Engineer");

	person1.friends.push("lily");
	alert(person1.friends);//"cate,lucy,lily"
	alert(person2.friends)//"cate,lucy"
	alert(person1.friends === person2.friends)//false
	alert(person1.sayName === person2.sayName)//false

	5.动态原型模式(best)

	把原型的创建放在构造函数中完成。

	function Person(name, age, job){
		//属性
		this.name = name;
		this.age = age;
		this.job = job;
		//方法
		if(typeof this.sayName != "function"){

			Person.prototype.sayName = function(){
				alert(this.name)
			}

		}
	}

	var friend = new Person("Jimmy", 29, "Front-end Engineer");

	friend.sayName();

	6.寄生构造函数模式

		function createPerson(name, age, job){
			var o = new Object();
			o.name = name;
			o.age = age;
			o.job = job;
			o.sayName = function(){
				alert(this.name);
			}
			return o;
		}
		var friend = new Person("Jimmy", 29, "Front-end Engineer");
		friend.sayName();//"Jimmy"

		特殊情况才用到此模式


		function SpecialArrary(){

			var values = new Array();

			values.push.apply(values,arguments);

			values.toPipedString = function(){
				return this.join("|");
			}

			return value;
		}

		var colors = new SpecialArrary("red","blue","green");
		alert(colors.toPipedString())//"red|blue|green"
	7.稳妥构造函数模式
		与寄生构造函数两点不同：1.新创建对象的实例方法不引用this;2.不使用new操作符调用构造函数

		function createPerson(name, age, job){
			var o = new Object();
			//这里可以定义私有变量和函数


			o.sayName = function(){
				alert(name);
			}
			return o;
		}
		var friend = Person("Jimmy", 29, "Front-end Engineer");
		friend.sayName();//"Jimmy"

继承
	1.原型链
		function SuperType(){
			this.property = true;
		}
		SuperType.prototype.getSuperValue = function(){
			return this.property;
		}
		function SubType(){
			this.subproperty = false;
		}
		//继承SuperType
		SubType.prototype = new SuperType();
		SubType.prototype.getSubValue = function(){
			return this.subproperty;
		}

		var instance = new SubType();
		alert(instance.getSuperValue());//true

		所有函数的默认原型都是Object的实例

		子类要重写超类的方法，代码一定要放在替换原型的语句之后

		function SuperType(){
			this.property = true;
		}
		function SubType(){
			this.subproperty = false;
		}
		SuperType.prototype.getSuperValue = function(){
			return this.property;
		}
		function SubType(){
			this.subproperty = false;
		}
		//继承SuperType
		SubType.prototype = new SuperType();
		//添加新方法
		SubType.prototype.getSuperValue = function(){
			return this.subproperty;
		}
		//重写超类型中的方法
		SubType.prototype.getSuperValue = function(){
			return false;
		}

		var instance = new SubType();
		alert(instance.getSuperValue());//false

		在通过原型链实现继承时，不能使用对象字面量创建原型方法，因为这样做会重写原型链

		原型链的继承存在的问题：引用类型值的原型属性会被所有实例共享（例如：数组、push）

	2.借用构造函数

		function SuperType(){
			this.colors = ["red","black","blue"];
		}
		function SubType(){
			//继承SuperType
			SuperType.call(this);
		}

		var instance1 = new SubType();
		instance1.colors.push("green");
		alert(instance1.colors)//"red,black,blue,green"

		var instance2 = new SubType();
		alert(instance1.colors)//"red,black,blue"

		传递参数

		function SuperType(name){
			this.name = name;
		}

		function SubType(){
			//继承SuperType,同时还传递了参数
			SuperType.call(this,"Jimmy");

			//实例属性
			this.age="27";
		}

		var instance = new Subtype();
		alert(instance.name);//"Jimmy"
		alert(instance.age);//27
	3.组合继承
		function SuperType(name){
			this.name = name;
			this.colors = ["red","black","blue"];
		}

		SuperType.prototype.sayName = function(){
			alert(this.name)
		}

		function SubType(name,age){
			//继承属性
			SuperType.call(this,name);

			this.age = age;
		}

		//继承方法
		SubType.prototype = new SuperType();
		SubType.prototype.constructor = SubType;
		SubType.prototype.sayAge = function(){
			alert(this.age)
		}

		var instance1 = new SubType("Jimmy"，29)；
		instance1.colors.push("green");
		alert(instance1.colors);//"red,black,blue,green"
		instance1.sayName();//"Jimmy"
		instance1.sayAge()//29

		var instance2 = new SubType("Jason"，29)；
		alert(instance2.colors);//"red,black,blue"
		instance2.sayName();//"Jason"
		instance2.sayAge()//29

	4.原型式继承

		function object(o){
			function F(){};
			F.prototype = o;
			return new F();
		}

		var person = {
			name : "Jimmy",
			friend : ["Shelby","Sam","Van"]
		}

		var anotherPerson = object(person);
		anotherPerson.name = "Gerg";
		anotherPerson.friends.push("Rob");

		var yetPerson = object(person);
		yetPerson.name = "Linda";
		yetPerson.friends.push("Barbie");

		alert(person.friends);//"Shelby,Sam,Van,Rob,Barbie"

		Object.create()
		var person = {
			name : "Jimmy",
			friend : ["Shelby","Sam","Van"]
		}

		var anotherPerson = Object.create(person,{
			name:{
				value:"Greg"
			}
		})

		alert(anotherPerson.name);//"Greg"


	5.寄生式继承

		function createAnother(original){
			var clone = object(original);
			clone.sayHi = function(){
				alert("hi")
			}
			return clone;
		}

		var person = {
			name : "Jimmy",
			friend : ["Shelby","Sam","Van"]
		}

		var anotherPerson = createAnother(person);
		anotherPerson.sayHi();//"hi"

	6.寄生组合式继承(best)
		function inheritPrototype(subType,superType){
			var prototype = object(superType.prototype);//创建对象
			prototype.constructor = subType;//增强对象
			subType.prototype = prototype;//指定对象
		}

		function SuperType(name){
			this.name = name;
			this.colors = ["red","black","blue"];
		}

		SuperType.prototype.sayName = function(){
			alert(this.name)
		}

		function SubType(name,age){
			//继承属性
			SuperType.call(this,name);

			this.age = age;
		}

		inheritPrototype(SubType,SuperType);

		SubType.prototype.sayAge = function(){
			alert(this.age)
		}

闭包
	//一篇不错的解读：http://bbs.html5cn.org/thread-86078-1-1.html

	下面用一些代码来解释这个定义。
	function foo() {
	var a = 2;
	function bar() {
	console.log( a ); // 2
	}
	bar();
	}
	foo();
	这段代码看起来和嵌套作用域中的示例代码很相似。基于词法作用域的查找规则，函数bar()可以
	访问外部作用域中的变量a（这个例子中的是一个RHS引用查询）。
	这是闭包吗？
	技术上来讲，也许是。但根据前面的定义，确切地说并不是。我认为最准确地用来解释bar()对a的
	引用的方法是词法作用域的查找规则，而这些规则只是闭包的一部分。（但却是非常重要的一部
	分！）
	从纯学术的角度说，在上面的代码片段中，函数bar()具有一个涵盖foo()作用域的闭包（事实上，涵
	盖了它能访问的所有作用域，比如全局作用域）。也可以认为bar()被封闭在了foo()的作用域中。为
	什么呢？原因简单明了，因为bar()嵌套在foo()内部。
	但是通过这种方式定义的闭包并不能直接进行观察，也无法明白在这个代码片段中闭包是如何工
	作的。我们可以很容易地理解词法作用域，而闭包则隐藏在代码之后的神秘阴影里，并不那么容易
	理解。
	下面我们来看一段代码，清晰地展示了闭包：
	function foo() {
	var a = 2;
	function bar() {
	console.log( a );
	}
	return bar;
	}
	var baz = foo();
	baz(); // 2 ———— 朋友，这就是闭包的效果。
	函数bar()的词法作用域能够访问foo()的内部作用域。然后我们将bar()函数本身当作一个值类型
	进行传递。在这个例子中，我们将bar所引用的函数对象本身当作返回值。
	在foo()执行后，其返回值（也就是内部的bar()函数）赋值给变量baz并调用baz()，实际上只是通过
	不同的标识符引用调用了内部的函数bar()。
	bar()显然可以被正常执行。但是在这个例子中，它在自己定义的词法作用域以外的地方执行。
	在foo()执行后，通常会期待foo()的整个内部作用域都被销毁，因为我们知道引擎有垃圾回收器用
	来释放不再使用的内存空间。由于看上去foo()的内容不会再被使用，所以很自然地会考虑对其进
	行回收。
	而闭包的“神奇”之处正是可以阻止这件事情的发生。事实上内部作用域依然存在，因此没有被回
	收。谁在使用这个内部作用域？原来是bar()本身在使用。
	拜bar()所声明的位置所赐，它拥有涵盖foo()内部作用域的闭包，使得该作用域能够一直存活，以
	供bar()在之后任何时间进行引用。
	bar()依然持有对该作用域的引用，而这个引用就叫作闭包。
	因此，在几微秒之后变量baz被实际调用（调用内部函数bar），不出意料它可以访问定义时的词法
	作用域，因此它也可以如预期般访问变量a。
	这个函数在定义时的词法作用域以外的地方被调用。闭包使得函数可以继续访问定义时的词法作
	用域。
	当然，无论使用何种方式对函数类型的值进行传递，当函数在别处被调用时都可以观察到闭包。
	function foo() {
	var a = 2;
	function baz() {
	console.log( a ); // 2
	}
	bar( baz );
	}
	function bar(fn) {
	fn(); // 妈妈快看呀，这就是闭包！
	}
	把内部函数baz传递给bar，当调用这个内部函数时（现在叫作fn），它涵盖的foo()内部作用域的闭
	包就可以观察到了，因为它能够访问a。
	传递函数当然也可以是间接的。
	var fn;
	function foo() {
	var a = 2;
	function baz() {
	console.log( a );
	}
	fn = baz; // 将baz分配给全局变量
	}
	function bar() {
	fn(); // 妈妈快看呀，这就是闭包！
	}
	foo();
	bar(); // 2
	无论通过何种手段将内部函数传递到所在的词法作用域以外，它都会持有对原始定义作用域的引
	用，无论在何处执行这个函数都会使用闭包。


	闭包是指有权访问另一个函数作用域中的变量的函数。创建闭包的常见方式，就是在一个函数内部创建另一个函数

	function createComparisonFunction(propertyName){
		return function(object1,object2){
			var value1 = object1[propertyName];
			var value2 = object2[propertyName];

			if(value1<value2){
				return -1
			} else{
				return 0;
			}
		}
	}
	匿名函数的作用链包含createComparisonFunction函数的活动对象和全局变量对象
	所以当函数返回后，执行环境的作用链会被销毁，但它的活动对象仍然留在内存中
	直至匿名函数被销毁

	//创建函数
	var compareNames = createComparisonFunction("name");
	//调用函数
	var result = compareNames({name:"Nicholas"},{name:"Greg"});
	//解除对匿名函数的引用（以便释放内存）
	compareNames = null;

	闭包只能取得包含函数中任何变量的最后一个值

	匿名函数的执行环境具有全局性，因此其this对象通常指向window

	var name = "Window";

	var object = {
		name : "My Object",
		getName : function(){
			return function(){
				return this.name;
			}
		}
	}
	alert(object.getName()())//window


	var name = "Window";

	var object = {
		name : "My Object",
		getName : function(){
			var that = this;//把当前的this做一份保存
			return function(){
				return that.name;
			}
		}
	}
	alert(object.getName()())//My Object

	var name = "Window";

	var object = {
		name : "My Object",
		getName : function(){
			return this.name
		}
	}
	alert(object.getName())//My Object

	object.getName();//My Object
	(object.getName)();//My Object
	(object.getName = object.getName) ();//window

	内存泄露

	IE：如果闭包作用域链中保存着一个HTML元素，那么意味着该元素将无法销毁
	function assignhandler(){
		var element = document.getElementById("someElement");
		element.onclick = function(){
			alert(element.id)
		}
	}
	解决办法：

	function assignhandler(){
		var element = document.getElementById("someElement");
		var id = element.id;
		element.onclick = function(){
			alert(id)
		}
		element =null;
	}

	有权访问私有变量和私有函数的公有方法称为特权方法

	function person(name){
		this.getname = function(){
			return name;
		}
		this.setname = function(value){
			name = value;
		}
	}

	(function (){
		var name = "";
		Person = function(value){
			name = value
		}
		Person.prototype.getname = function(){
			return name;
		}
		Person.prototype.setname = function(value){
			name = value;
		}
	})()

	var person1 = new Person("Jimmy");
	alert(person1.getname());//"Jimmy"
	person1.setname("Sam");
	alert(person1.getname());//"Sam"

	var person2 = new Person("David");
	alert(person1.getname());//"David"
	alert(person2.getname());//"David"


	var application = function(){
		//私有变量和函数
		var components = new Array();

		//初始化
		components.push(new BaseComponent());

		//公共
		return{
			getComponentCount : function(){
				return components.length;
			},
			registerCompoent : function(component){
				if(typeof component == "object"){
					components.push(component)
				}
			}
		}

	}

	一个基于jQuery简单的订阅/发布
	(function($){
		var o = $({});
		$.subscribe = function(){
			o.bind.apply(o,arguments)
		}
		$.unsubscribe = function(){
			o.unbind.apply(o,arguments)
		}
		$.publish = function(){
			o.trigger.apply(o,arguments)
		}
	})(Jquery)

	$.subscribe("/some/topic",function(event,a,b,c){
		console.log(event.type,a+b+c)
	})

	$.publish("/some/topic","a","b","c")

	define(function () {
	    "use strict"
	    var $root = $(document);
	    var EventManager = {
	        trigger: function (type) {
	            var data = Array.prototype.slice.call(arguments, 1);//第一个是function，后面是data，所以从第二个开始取
	            $root.trigger(type, data);
	        },

	        on: function (type, callback) {
	            $root.off(type);
	            $root.on(type, function () {
	                var data = Array.prototype.slice.call(arguments, 1);
	                callback && callback.apply(this, data);
	            })
	        }

	    };

	    return EventManager;
	});
	Array.prototype.slice.call(arguments, 0)
	//将arguments对象转换成真正的数据，arguments无slice方法
	/*
		伪组数元素
		call 方法以另一个对象替换当前对象
		thisObj   可选项。将被用作当前对象的对象。
		﻿这就是说：Array.prototype.slice.call(arguments,0) 这句里，就是把 arguments 当做当前对象
		﻿也就是说 要调用的是 arguments 的slice 方法，后面的 参数 0 也就成了 slice 的第一个参数slice(0)就是获取所有
		为什么要这么调用 arguments 的slice 方法呢？就是因为 arguments 不是真的组数，typeof arguments==="Object" 而不是 "Array"

		它没有slice这个方法，通过这么Array.prototype.slice.call调用，JS的内部机制应该是 把arguments对象转化为Array

		因为Array.prototype.slice.call调用后，返回的是一个组数
	*/
	Array.prototype.slice.call(arguments, 1)
	//去除传参的第一个参数，类似click（click,1,2,3）

	Object.create(this.prototype)//传入一个对象，返回一个继承这个对象的新对象

	jQuery.extend(a,b) //a添加b属性

	//生成GUID(全局统一标识)
	Math.guid = function(){
		return "xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx".replace(/[xy]/g,function(c){
			var r = Math.random()*16|0,v = c =='x' ? r :(r&0x3|0x8);
			return v.toString(16);
		}).toUpperCase();
	}
	//跨域
	CORS
	HTTP协议头加
	Access-Control-Allow-Origin:example.com //允许任意访问的用*
	Access-Control-Request-Method:GET,POST

	var object = {some:"object"};
	//序列化并保存一个对象
	localStorage.setItem("seriData",JSON.stringify(object));
	//读取并将JSON
	var result = JSON.parse(localStorage.getItem("seriData"))

	e.currentTarget//e
	绑定处的dom.target//点击处的dom

	_.template(template,tData, { "variable": "Intro" });//第三个参数可以自定义变量名称

	/*
    * stopImmediatePropagation
    * 除了阻止元素上其它的事件处理函数的执行，这个方法还会通过在内部调用 event.stopPropagation() 来停止事件冒泡。
    * 如果仅仅想要停止事件冒泡到祖辈元素上，而让这个元素上的其它事件处理函数继续执行，我们可以使用 event.stopPropagation() 来代替。
    * 相当于e.stopPropagation() and return fasle
    */
    e.stopImmediatePropagation();

    控制台
    dir()输出了对象中的所有属性
    dir({one:1})

    inspcect()的参数可以是元素、数据库或存储区域，并会自动跳转到调试工具的对应面板以显示相关的信息
    inspect($("uesr"))

    keys()返回由对象中所有的属性的名字组成的数组
    //返回["two"]
    keys({two:12})

    values()返回由对象属性值组成的数据，用法和keys()类似
    //返回[12]
    values({two:12})

    Profile指调试工具提供的一个功能，用来统计代码中函数执行的次数和时间，并给出报表。
    只需要在你想要统计的代码两端加上console.profile()和console.profileEnd()即可
    console.profile()；
    //...
    console.profileEnd()
    同样console.time可以统计代码间的时间
    console.time(name)和console.logEnd(name)之间
    console.time(name)
    //...
    console.logEnd(name)

    数据排序sort

    var values = [0,1,5,10,15];
    values.sort();
    alert(values);//0,1,10,15,5

    sort可传入函数进行排序
    //如果第一个参数应该位于第二个之前则返回一个负数，正数则在之后
    function compare(va1,va2){
    	if(va1 < va2){
    		return -1;
    	} else if(va1 > va2){
    		return 1;
    	} else{
    		return 0;
    	}
    }

    var values = [0,1,5,10,15];
    values.sort(compare);
    alert(values);//0,1,5,10,15

    arguments有一个callee属性，可以指向拥有arguments对象的函数
    function fa(num){
    	if(num <= 1){
    		return 1;
    	} else {
    		return num * fa(num-1)
    	}
    }

    等同于

    function fa(num){
    	if(num <= 1){
    		return 1;
    	} else {
    		return num * arguments.callee(num-1)
    	}
    }

    function outer(){
    	inner()
    }

    function inner(){
    	alert(arguments.callee.caller);
    }

    outer();

    //function outer(){inner()}
    
    alert(String.fromCharCode(104,101,108,108,111))//"hello"

    encodeURI() //不对冒号 正斜杠 问好 井号编码
    encodeURIComponent()

    //获取查询字符串参数
    function getQueryStringArgs() {
    	var qs = (location.search.length > 0 ? location.search.substring(1) : ""),
    		args = {},
    		items =qs.length ? qs.split("&") : [],
    		item = null,
    		name = null,
    		value = null,
    		i = 0,
    		len = items.length;

    	for (i = 0; i < len; i++){
    		item = items[i].split("=");
    		name = decodeURIComponent(item[0]);
    		value = decodeURIComponent(item[1]);
    		if(name.length){
    			args[name] = value;
    		}
    	}
    	return args;
    }
    //模拟事件
    var btn = document.getElementById("myBtn");

    var event  = document.createEvent("MouseEvents");

    event.initMouseEvent("click",true,true,document,defaultView,0,0,0,0,0,false,false,false,false,0,null);

    btn.dispatchEvent(event)

    提升：零散变量的问题

    myname = "global";
    function fuc(){
    	alert(myname);//undefined
    	var myname = "local";
    	alert(myname);//local
    }
    无所在哪声明，都等同于在函数顶部声明


    检查是否是数组
    ES5+ ： Array.isArray()

    之前版本做兼容：
    	if(typeof Array.isArray === "undefined"){
    		Array.isArray = function (arg){
    			return Object.prototype.toString.call(arg) === "[object Array]";
    		}
    	}

    XHR2的FormData
    	var data = new FormData()
    	data.append("name","jimmy")

    	var xhr = createXHR();
    	xhr.onreadystatechange = function(){
    		if(xhr.readyState == 4){
    			if((xhr.status >=200 && xhr.status <300)|| xhr.status ==304){
    				alert(xhr.responseText)
    			}else{
    				alert("Request was unsuccessful"+xhr.status)
    			}
    		}
    	}
    	xhr.open("post","demo.php",true)
    	xhr.send(data)
    CORS(Cross-Origin Resource Sharing,跨源資源共享)
    	背后的基本思想，就是使用自定义的HTTP头部让浏览器与服务器进行沟通，从而决定请求或响应式应该成功还是失败
    	检测xhr是否支持CORS，就是检查是否存在withCredentials属性，再结合检测XDomainRequest对象是否存在，就可以兼顾所以浏览器。
    	"withCredentials" in xhr
    	typeof XDomainRequest != "undefined"
    其他跨域技术
    	图片Ping（广告跟踪浏览的主要方式）
    		图片Ping是于服务器进行简单、单向的跨域通信的一种方式。
    		var img = new Image();
    		img.onload = img.onerror = function(){
    			alert("Done!");
    		}
    		img.src = "http://xx.com/t?name=xx"
    		图片Ping有两个主要缺点，一是只能发送GET请求，而是无法访问服务器的响应文本。
    	JSONP(JSON with padding)
    		JSONP由两部分组成：回调函数和数据。回调函数式当响应到来时应该在页面中调用的函数。回调函数的名字一般在请求中指定的。而
    		数据就是传入回调函数中的JSON数据。下面是一个典型的JSONP
    		"http://xx.com/t/?callback=handleResponse"
    		JSONP是通过动态<script>元素来使用的，<script>和<img>都有能力不受限制的从其它域加载资源。
    		function handleResponse(){
    			alert(data)
    		}
    		var script = document.createElement("script");
    		script.src = "http://xx.com/t/?callback=handleResponse"
    		document.body.insertBefore(script,document.body.firstChild)
    		JSONP两个缺点，1是从其它域加载代码，其它域的安全要保证2是要确定JSONP请求是否失败并不容易
    Comet
   		Comet是一种服务器向页面推送数据的技术
   		两种实现Comet的方式：长轮询和流。
   		SSE
   			是围绕只读Comet交互推送API或者模式
   		事件流
   	Web Sockets
   		Web Sockets的目标是在一个单独的持久连接上提供全双工、双向通信。
   		var socket = new WebSockets("ws://xx.com/x.php")
   		socket.opening(0):正在建立连接
   		socket.open(1):已经建立连接
   		socket.closing(2):正在关闭连接
   		socket.close(3):已经关闭连接
   		socket.send("xx"):发送数据
   		socket.onmessage = function(event){
   			var data = event.data;
   			//处理数据
   		}
   	cookie操作
   		var CookieUtil = {
   			get:function(name){
   				var cookieName = encodeURIComponent(name) + "=",
   					cookieStart = document.cookie.indexOf(cookieName),
   					cookieValue = null;
   				if(cookieStart > -1){
   					var cookieEnd = document.cookie.indexOf(";",cookieStart);
   					if(cookieEnd == -1){
   						cookieEnd = document.cookie.length;
   					}
   					cookieValue = decodeURIComponent(doucument.cookie.substring(cookieStart+cookieName.length,cookieEnd))
   				}
   				return cookieValue;
   			},
   			set:function(name,value,expires,path,domain,secure){
   				var cookieText = encodeURIComponent(name) + "=" + encodeURIComponent(value);
   				if(expires instanceof Date){
   					cookieText += "; expires=" +expires.toGMTString();
   				}
   				if(path){
   					cookieText += "; path=" + path;
   				}
   				if(domain){
   					cookieText += "; domain=" + domain;
   				}
   				if(secure){
   					cookieText += "; secure=" + secure;
   				}
   				document.cookie = cookieText;
   			},
   			unset:function(name,path,domain,secure){
   				this.set(name,"",new Date(0),path,domain,secure)
   			}
   		}
   		//用法
   		CookieUtil.set("name","Nicholas")
   		CookieUtil.get("name")
   		CookieUtil.unset("name")
   	性能
   		1.注意作用域
   			1.1 避免全局查找
   			1.2 避免with语句
   		2.选择正确方法
   			2.1 避免不必要的属性查找
   				O(1)     常数 不管多少值，执行事件都是恒定的。一般表示简单值和存储在变量的中的值
   				O(log n) 对数 总的执行时间和值的数量相关，但是要完成算法并一定要获取每个值。例如：二分查找
   				O(n)     线性 总执行时间和值的数量直接相关。例如：遍历某个数组中的所有元素
   				O(n*n)   平方 总执行时间和值的数量相关，每个值至少获取n次。例如：插入排序

   				O(1):
   					var value = 6;
   					var sum = 10 - value;
   					alert(sum)
   				O(n):
   					注意单个值的多重查找，看下面的代码：
   						var query = window.location.href.substring(window.location.href.indexOf("?"));
   						上面的代码有6次属性查找，window.location.href.substring有3次，window.location.href.indexOf又3次
   						一旦多次用到对象属性，应该将其存储在局部变量中，一次访问会是O(n)，后续都会是O(1)，就会节省很多，重写代码：
   						var url = window.location.href;
   						var query = url.substring(url.indexOf("?"))
   						上面的代码只要了四次属性查找，节省了33%
   			2.2 优化循环
   				减值迭代——在很多情况下，可能从最大值开始，在循环中不断减值的迭代器更加高效
   				简化终止条件——由于每次循环过程都会计算终止条件，所以必须保证它尽可能快。也就是说避免属性查找或其它O(n)操作 
   				简化循环体——循环体是执行最多的，所以要确保其被最大限度的优化。确保没有某些可以被很容易移出循环的密集计算
   				使用后测试循环——for和while是前测试循环，而do-while是后测试循环，可以避免最终终止条件的计算，因此运行更快

   				看下面的代码
   				for(var i = 0; i < values.length; i++){
   					process(values[i])
   				}
   				可优化为
   				for(var i = values.length-1; i >= 0; i--){
   					process(values[i])
   				}
   				把values.length的O(n)调用改为0的O(1)调用
   				再改成后测试循环
   				var i = values.length-1;
   				if(i > -1){
   					do{
   						process(values[i]);
   					}while(--i >= 0)
   				}
   			2.3 展开循环
   				如果循环中的迭代次数不能实现确定，可以考虑一种叫Duff的技术
   				代码如下：
   					var iterations = Math.ceil(values.length / 8);
   					var startAt = values.length % 8;
   					var i = 0;
   					do{
   						switch(start){
   							case 0: process(values[i++]);
   							case 7: process(values[i++]);
   							case 6: process(values[i++]);
   							case 5: process(values[i++]);
   							case 4: process(values[i++]);
   							case 3: process(values[i++]);
   							case 2: process(values[i++]);
   							case 1: process(values[i++]);
   						}
   						startAt = 0;
   					}while(--iterations > 0)
   			2.4 避免双重解释
   				当JavaScript代码想解析JavaScript的时候就会存在双重解释惩罚。
   				当使用eval()函数或Function构造函数以及使用setTimeout()传一个字符串参数时都会发生这种情况，下面是例子：
   				//某些代码求值——避免
   				eval("alert('hello world')")
   				//创建新函数——避免
   				var sayHi = new Function("alert('Hello World')")
   				//设置超时——避免
   				setTimeout("alert('Hello World')",500)
   				修正后：
   				alert('hello world')
   				var sayHi = function(){alert('Hello World')}
   				setTimeout(function(){alert('Hello World'),500)
   			2.5 性能的其他注意事项
   				原生方法较快
   				switch语句较快
   				位运算符较快
   		3.最小语句数
   			3.1 多个变量声明
   				var a = 1;
   				var b = 2;
   				var c = 3;
   				var d = 4;
   				重写
   				var a = 1,
   					b = 2,
   					c = 3,
   					d = 4;
   			3.2 插入迭代值
   				当使用迭代值时，尽可能合并语句
   				var name = values[i];
   				i++;
   				合并 
   				var name = values[i++];
   			3.3 使用数组和对象字面量
   				构造函数会带来更多的语句，尽量使用对象字面量
   				var values = new Array();
   				values[0] = 123;
   				values[1] = 456;
   				values[2] = 789;

   				var person = new Object();
   				person.name = "Nicholas";
   				person.age = 29;
   				person.sayName = function(){
   					alert(this.name)
   				}
   				重写后
   				var values = [123,456,789]
   				var person = {
   					name:"Nicholas",
   					age:29,
   					sayName:function(){
	   					alert(this.name)
	   				}
   				}
   		4.优化DOM交互
   			4.1 最小化现场更新
   				看个例子：
   				var list = doucument.getElementById("myList"),
   					item,
   					i;
   				for(i = 0; i < 10; i++){
   					item = document.createElement("li");
   					list.appendChild(item);
   					item.appendChild(document.createTextNode("Item"+i))
   				}
   				这段代码为列表增加10个项目，添加每个项目时，都有两个现场更新，优化后
   				var list = doucument.getElementById("myList"),
   					fragment = doucument.createDocumentFragment(),
   					item,
   					i;
   				for(i = 0; i < 10; i++){
   					item = document.createElement("li");
   					fragment.appendChild(item);
   					item.appendChild(document.createTextNode("Item"+i))
   				}
   				list.appendChild(fragment);
   				一旦需要更新DOM,请考虑使用文档片段构建DOM结构，然后再将其添加到现存的文档中
   			4.2 使用innerHTML效率
   				使用innerHTML效率高于createElement()、appendChild()之类的DOM方法
   				前面的代码可以改写为
   				var list = doucument.getElementById("myList"),
   					html = "",
   					i;
   				for(i = 0; i < 10; i++){
   					html += "<li>Item" + i + "<li>";
   				}
   				list.innerHTML = html;
   			4.3 使用事件代理
   				页面上的事件处理程序的数量和页面响应用户交互的速度之间有个负相关。
   				为了减轻这种惩罚，最好使用事件代理。
   			4.4 注意HTMLCollection
   				记住，任何时候要访问HTMLCollection，不管他是一个属性还是方法，都是在文档上进行查询，
   				这个查询开销和昂贵，最小化访问HTMLCollection的次数可以极大改进脚本性能。
   				看例子：
   				var images = document.getElementByTagName("img"),
   					i,
   					len;
   				for(i = 0; len = images.length; i < len; i++){
   					//处理
   				}
   				这里的关键在于长度length存入了len变量，而不是每次都去访问HTMLCollection的length
   				var images = document.getElementByTagName("img"),
   					image,
   					i,
   					len;
   				for(i = 0; len = images.length; i < len; i++){
   					image = images[i]
   				}
   				这段代码添加了image变量，保存了当前图像。这之后，在循环内就没理由再访问images的HTMLCollection
   				以下情况会返回HTMLCollection对象
   					进行了getElementByTagName()调用
   					获取了元素的childNodes属性
   					获取了元素的attributes属性
   					访问了特殊的集合，如document.forms,document.images等
   				下面的每个项目（以及它们指定的属性）都返回 HTMLCollection：
					Document (images, applets, links, forms, anchors)
					form (elements)
					map (areas)
					select (options)
					table (rows, tBodies)
					tableSection (rows)
					row (cells)
				了解了何时返回HTMLCollection对象，就能最小化对它进行访问
	新兴的API
		1.requestAnimationFrame
			function updateProgress(){
				var div = document.getElementById("status");
				diy.style.width = (parseInt(div.style.width,10)+5)+"%";
				if(diy.style.width != "100%"){
					requestAnimationFrame(updateProgress)
				}
			}
			requestAnimationFrame(updateProgress)
		2.Page Visibility API 
			判断用户是否与页面在交互
			2.1 document.hidden 表示页面是否隐藏的布尔值。页面隐藏包括页面在后台标签中或者浏览器最小化
			2.2 document.visibilityState 表示4个可能状态的值
				(1)	页面在后台标签中或浏览器最小化
				(2) 页面在嵌套标签页中
				(3) 实际的页面已经隐藏，但用户可以看到页面的预览
				(4) 页面在屏幕外执行预渲染处理
			2.3 visibilityChange事件 当文档从可见变为不可见或不可见变为可见，触发改事件
		3.Geolocation API
			navigator.geolocation
				1.getCurrentPosition 调用这个方法就会触发请求用户共享地理位置信息的对话框
					navigator.geolocation.getCurrentPosition(function(position){
						console.log(position.coords.latitude+"##"+position.coords.longtitude)
					})
				2.watchPosition 跟踪用户的位置
		4.File API
			4.1.FileReader类型
				(1)readAsText(file,encoding) 以纯文本形式读取文件，保存在result中
				(2)readAsDataURL(file) 读取文件将文件以URI的形式保存在result
				(3)readAsBinaryString(file) 读取文件并将一个字符串保存在result
				(4)readAsArrayBuffer(file) 读取文件并将一个包含文件内容的ArrayBuffer保存在result
			4.2 读取部分内容
				我们只想读取文件的一部分而不是全部内容，File对象还支持一个slice()方法
			4.3 对象URL
				对象URl也被称为blobURL，指的是引用保存在File和Blob中数据的URl
				要创建对象URL，可以使用window.URL.createObjectURL()
			4.4 读取拖放的文件
				event.dataTransfer.files
			4.5 使用XHR上传文件
				data = new FormData()
				data.append("file"+i,files[i])
				xhr.send(data)
		5.Web计时
			window.performance
				用于页面性能分析
				http://webtimingdemo.appspot.com/
		6.Web Workers
    curry化函数的示例：
    	// a curried add()
		// accepts partial list of arguments
		function add(x, y) {
		    var oldx = x, oldy = y;
		    if (typeof oldy === "undefined") { // partial
		        return function (newy) {
		            return oldx + newy;
		        };
		    }
		    // full application
		    return x + y;
		}
		// test
		typeof add(5); // "function"
		add(3)(4); // 7
		// create and store a new function
		var add2000 = add(2000);
		add2000(10); // 2010
		//示例2
		var adder = function(num) {
		    return function(y) {
		        return num+y;
		    };
		};
		var inc = adder(1);
		var dec = adder(-1);
		//inc, dec现在是两个新的函数,作用是将传入的参数值 (+/‐)1
		alert(inc(99));//100
		alert(dec(101));//100 
		alert(adder(100)(2));//102 
		alert(adder(2)(100));//102
		//示例3 阿里玉伯的seaJS源码片段
		/**
		 * util-lang.js - The minimal language enhancement
		 */
		function isType(type) {
		  return function(obj) {
		    return {}.toString.call(obj) == "[object " + type + "]"
		  }
		}
		var isObject = isType("Object");
		var isString = isType("String");
	抽象通用方法：
	    function schonfinkelize(fn) {
		    var slice = Array.prototype.slice,
		        stored_args = slice.call(arguments, 1);
		    return function () {
		        var new_args = slice.call(arguments),
		            args = stored_args.concat(new_args);
		        return fn.apply(null, args);
		    };
		}

		schonfinkelize(function(){console.log(arguments)},1,2)(3)//[1,2,3]
		schonfinkelize(function(){console.log(arguments)},1,2)()//[1,2]
	例子：
		// a normal function
		function add(x, y) {
		    return x + y;
		}
		// curry a function to get a new function
		var newadd = schonfinkelize(add, 5);
		newadd(4); // 9
		// another option -- call the new function directly
		schonfinkelize(add, 6)(7); // 13
	实现add(1)(2)(3)(4)
		function add(x) {
		    var sum = x;
		    var tmp = function (y) {
		        sum = sum + y;
		        return tmp;
		    };
		    tmp.toString = function () {
		        return sum;
		    };
		    return tmp;
		}
		console.log(add(1)(2)(3));  //6
		console.log(add(1)(2)(3)(4));   //10

	错误捕获——使用image对象来发送请求：
		function logError(sev,msg){]
			var img =  new Image();
			img.src = "log.php?sev="+encodeURIComponent(sev)+"&msg="+encodeURIComponent(msg);
		}
	这样做的好处：
		1.所有浏览器支持Image对象，包括不支持XMLHttpRequest对象的浏览器
		2.可以避免跨域的限制。
		3.在记录错误过程中出问题的概率比较低。Ajax不如image来的稳定，毕竟Ajax是包装函数处理的。

	通用命名空间函数:
	var MYAPP = MYAPP || {};

	MYAPP.namespace = function (ns_string) {
		var parts = ns_string.split('.'),
			parent = MYAPP,
			i;

		if(parts[0] === "MYAPP"){
			parts = parts.slice(1);
		}
		for(i = 0 ; i < parts.length; i += 1){
			if(typeof parent[parts[i]] === "undefined") {
				parent[parts[i]] = {};
			}
			parent = parent[parts[i]];
		}
		return parent;
	}

	MYAPP.namespace('MYAPP.modules.module2')

	私有成员
	function Gadget() {
		//私有成员
		var name = 'iPod';
		//公有函数   又叫特权方法(可访问私有成员的公共方法)
		this.getName  = function () {
			return name;
		}
	}

	var toy = new Gadget();
	console.log(toy.name);//undefined
	console.log(toy.getName);//ipod

	私有性失效
	function Gadget() {
		//私有成员
		var specs = {
			a : 1,
			b : 2,
			c : 3
		}
		//公有函数
		this.getSpecs  = function () {
			return specs;
		}
	}

	var toy = new Gadget();
	var specs = toy.getSpecs();
	specs.color = 'black';
	specs.price = 'free';

	console.log(toy.getSpecs());//返回了引用，所以外部修改，源也会修改

	模块模式

	var myobj = (function (){
		//私有成员
		var name = "Jimmy";
		//实现公有部分
		return {
			getName: function (){
				return name;
			}
		}
	}())

	myobj.getName();//输出“my,oh my”

	原型和私有性

	function Gadget() {
		var name = 'abc';
		this.getName = function () {
			return name;
		}
	}

	Gadget.prototype = (function (){
		var browser = 'webkit';
		return {
			getBrowser: function (){
				return browser;
			}
		}
	}())

	var toy = new Gadget();
	console.log(toy.getName())//特权“own“方法
	console.log(toy.getBrowser())//特权原型方法

	揭示模块模式

	var myarray;

	(function () {
		var astr = '[object Array]',
			toString = Object.prototype.toString;

		function isArray(a) {
			return toString.call(a) === astr;
		}

		function indexof(haystack,needle){
			var i = 0,
				max = haystack.length;

			for(; i < max; i+=1){
				if(haystack[i] === needle){
					return i;
				}
			}
			return -1;
		}

		myarray = {
			isArray: isArray,
			indexOf: indexOf,
			inArray: indexOf
		}
	}())

	myarray.isArray([1,2])

	模块模式

	MYAPP.utilties.array = (function () {
		//依赖
		var uobj = MYAPP.utilties.object,
			ulang = MYAPP.utilties,lang,

			//私有属性
			array_string = "[object Array]",
			ops = Object.prototype.toString;
			//私有方法
			//...
			//var变量定义结束
		//可选的一次性初始化过程
		//...

		//公有API
		return{
			inArray:function(needle,haystack){
				for(var i = 0, max = haystack.length; i< max; i+=1){
					if(haystack[i] === needle){
						return true
					}
				}
			},
			isArray: function(a){
				return ops.call(a) === array_string;
			}
			//更多方法和属性
		}
	}())
	模块模式得到了广泛的使用，并且强烈建议使用这种方式组织你的代码，尤其是当代码日渐增长的时候。


	揭示模块模式

	MYAPP.utilties.array = (function () {
		//依赖
		var uobj = MYAPP.utilties.object,
			ulang = MYAPP.utilties,lang,

			//私有属性
			array_string = "[object Array]",
			ops = Object.prototype.toString,
			//私有方法
			//...
			//var变量定义结束
		//可选的一次性初始化过程
		//...
		//私有方法
		inArray = function(needle,haystack){
			for(var i = 0, max = haystack.length; i< max; i+=1){
				if(haystack[i] === needle){
					return true
				}
			}
		},
		isArray = function(a){
			return ops.call(a) === array_string;
		};

		return{
			isArry: isArray,
			indexof: inArray
		}
	}())

	私有静态成员
	var Gadget = (function(
		//静态变量/属性
		var counter = 0,
			NewGadget;
		//这将成为新的构造函数
		NewGadget = function (){
			counter += 1;
		}
		//特权方法
		NewGadget.prototype.getlastId = function () {
			return counter;
		}
		//覆盖该构造函数
		return NewGadget;
	){}())//立即执行

	var iPhone = new Gadget();
	iPhone.getlastId();//1
	var iPod = new Gadget();
	iPod.getlastId();//2
	var iPad = new Gadget();
	iPad.getlastId();//3 

	//method
	if(typeof Function.prototype.method !== "function"){
		Function.prototype.method = function(name,implementation){
			this.prototype[name] = implementation;
			return this
		}
	}
	method('getName',function(){
		return this.name;
	})
	method('setName',function(name){
		this.name = name;
		return this;
	})
	继承
		类式继承
			//父构造函数
			function Parent(name){
				this.name = name || 'Adam';
			}
			//向该原型添加功能
			Parent.prototype.say = function (){
				return this.name;
			}
			//空白的子构造函数
			function Child (name) {}

			//继承的魔力在这里发生
			inherit(Child,Parent);

			#1——默认模式
				function inherit(C,P){
					C.prototype = new P();
				}

				var kid = new Child();
				kid.say();
				缺点：同时继承两个对象的属性，即添加到this的属性以及原型属性，绝大多数时候，并不需要这些自身的属性，因为它们很可能是指向一个特定的实例
				而非复用。
			注意：对于构造函数的一般经验法则是：应该将可复用的成员添加到原型中。

			#2——借用构造函数
				function Child(a,b,c,d){
					Parent.apply(this,arguments)
				}
				多重继承
				function Cat(){
					this.legs = 4;
					this.say = function (){
						return "meaowww";
					}
				}
				function Bird(){
					this.wings = 2;
					this.fly = true;
				}

				function CatWings(){
					Cat.apply(this);
					Bird.apply(this);
				} 
				var jane = new CatWings();
				缺点：无法从原型中继承任何东西
			#3——借用和设置原型
				function Child(a,c,b,d){
					Parent.apply(this,arguments)
				}
				Child.prototype = new Parent();
				缺点：父构造函数被调用了两次

				var kid = new Child("Patrick");
				kid.name;//输出"Patrick"
				kid.say();//输出"Patrick"
				delete kid.name;
				kid.say();//输出Adam
				删除自身属性后，随后输出的是原型链的值
			#4——共享原型
				本模式的经验法则在于：可复用成员应该转移到原型中而不是放置在this中。因此，出于继承的目的，任何值得继承的东西都应该
				放置在原型中实现。所以，可以仅将子对象的原型与父对象的原型设置为相同的即可
				function inherit(C,P){
					C.prototype = P.prototype;
				}
				缺点：继承链下方的某处存在一个子对象或孙子对象修改了原型，它将影响到所有的父对象和祖先对象。

			#5——临时构造函数
				function inherit(C,P){
					var F = function(){};
					F.prototype = P.prototype;
					C.prototype = new F();
				}
				在上面模式的基础上，还可以添加一个指向原始对象的引用。
				function inherit(C,P){
					var F = function(){};
					F.prototype = P.prototype;
					C.prototype = new F();
					C.uber = P.prototype;
				}
				最后，针对这个近乎完美的类式继承模式，还需要做的一件事就是重置该构造函数的指针，以免将来的某个时候还需要该构造函数。
				function inherit(C,P){
					var F = function(){};
					F.prototype = P.prototype;
					C.prototype = new F();
					C.uber = P.prototype;
					C.prototype.constructor = C;
				}
		Klass
			var Man = klass(null,{
				_construct:function(what){
					console.log("Man's construct");
					this.name = what;
				},
				getName:function(){
					return this.name;
				}
			})
			var first = new Man('Adam');
			first.getName();//"Adam"

			var SuperMan = klass(Man,{
				_construct:function(what){
					console.log("SuperMan's construct")
				},
				getName:function(){
					var name = SuperMan.uber.getName.call(this);
					return "I am"+ name;
				}
			})

			var clark = new SuperMan('Clark Kent');
			clark.getName();//结果为"I am Clark Kent"

			var klass = function(Parent,props){
				var Child,F,i;
				//1.
				//新构造函数
				Child = function(){
					if(Child.uber && Child.uber.hasOwnProperty("_construct")){
						Child.uber._construct.apply(this,arguments);
					}
					if(Child.prototype.hasOwnProperty("_construct")){
						Child.prototype._construct.apply(this,arguments)
					}
				}
				//2
				//继承
				Parent = Parent || Object;
				F = function(){};
				F.prototype = Parent.prototype;
				Child.prototype = new F();
				Child.uber = Parent.prototype;
				Child.prototype.construct =Child;  
				//3
				//添加实现方法
				for(i in props){
					if(props.hasOwnProperty(i)){
						Child.prototype[i] = props[i];
					}
				}
				return Child;
			}
			klass()实现中三个令人关注且独特的部分:
			1.创建Child()构造函数。该函数将是最后返回的，并且该函数也用作类。在这个函数中，如果
			  存在_construct方法，那么将会调用该方法。另外，在此之前，通过使用静态uber属性，其父
			  类的_construct方法也会被自动调用（同样，如果存在改方法的话）。可能在有些情况下，当没
			  有定义uber属性时，比如直接从Object类中继承时，这与Man类的定义中继承时相似的情况。

			2.第二部分在一定程度上处理继承关系。它只是采用了本章前面章节中所讨论的类式继承的终极版。
			从代码上，只有一个新语句（Parent = Parent || Object），即如果没有传递需要被继承的类，那
			么就将Parent变为Object。

			3.最后一节是遍历所有的实现方法（比如，本例中的_construct和getName），这些是该类的实际定义，
			并且也是将它们添加到Child的原型中的部分代码。

		原型继承
			var parent = {
				name:"papa"
			}
			//新对象
			var child = object(parent);

			//测试
			alert(child.name)//papa

			function object(o){
				function F(){};
				F.prototype = o;
				return new F();
			}


			function Person(){
				this.name = "Adam"
			}

			Person.prototype.getName = function(){
				return this.name;
			}

			var papa = new Person();

			var kid = object(papa);

			kid.getName();//"Adam"

			继承原型
			function Person(){
				this.name = "Adam"
			}

			Person.prototype.getName = function(){
				return this.name;
			}

			var kid = object(Person.prototype);

			typeof kid.getName;//结果为"function"类型因为它在原型中
			typeof kid.name;//结果为"undefined"类型，因为只有原型被继承

			ECMAScript5中Object.create()

			var child = Object.create(parent)

			var child = Object.create(parent,{
				age:{value:2}
			})
			child.hasOwnProperty("age")

			//通过复制属性实现继承

			浅复制：
				function extend(parent,child){
					var i;
					child = child || {};
					for (i in parent){
						if(parent.hasOwnProperty(i)){
							child[i] = parent[i];
						}
					}
					return child;
				}

				var dad = {name:"Adam"};
				var kid = extend(dad);
				kid.name;//结果为"Adam"

				浅复制，由于通过引用传递，对子属性改变会影响父对象

				var dad = {
					counts:[1,2,3],
					reads:{paper:true}
				}
				var kid = extend(dad);
				kid.counts.push(4);
				dad.counts.toString();//1,2,3,4
				dad.reads === kid.reads;//true

			深复制：
				function extendDeep(parent,child){
					var i,
						toStr = Object.prototype.toString,
						astr = "[object Array]";
					child = child || {}; 
					for(i in parent){
						if(parent.hasOwnProperty(i)){
							console.log(parent[i] +"##"+i+"###"+child[i])
							if(typeof parent[i] === "object"){
								console.log("@@@"+parent[i] +"##"+i+"###"+child[i])
								console.log(parent)
								console.log("###############")
								child[i] = (toStr.call(parent[i]) == astr)?[]:{};
								extendDeep(parent[i],child[i]);
							}else{
								console.log("!!!"+parent[i] +"##"+i+"###"+child[i])
								console.log(child)
								console.log("###############")
								console.log(parent)
								console.log("###############")
								child[i] = parent[i]
							}	
						}
					}
					return child;
				}

				var dad = {
					counts:[1,2,3],
					reads:{paper:true}
				}
				var kid = extendDeep(dad);
				kid.counts.push(4);
				kid.counts.toString();//1,2,3,4
				dad.counts.toString();//1,2,3

				dad.reads === kid.reads;//false
				kid.reads.paper = false;

				kid.reads.web =true;
				dad.reads.paper//结果为true
		混入
			function mix(){
				var arg,prop,child={};
				for(arg = 0; arg < arguments.length;arg +=1){
					for(prop in arguments[arg]){
						if(arguments[arg].hasOwnProperty(prop)){
							child[prop] = arguments[arg][prop];
						}
					}
				}
				return child;
			}

			var cake = mix(
				{egg:2,large:true},
				{flour:"3 cups"}
			)
		借用方法
			call、apply
			[].slice.call(arguments)

			Function.prototype.bind()

			if(typeof Function.prototype.bind === "undefined"){
				Function.prototype.bind = function(thisArg){
					var fn = this,
						slice = Array.prototype.slice,
						args = slice.call(arguments,1)//arguments是类数组 没有slice属性 Array.prototype.slice.call(arguments) === arguments.slice() 返回argument对应的数组

					return function(){
						//拼接参数列表，即那些传递给bind()的参数（除第一个外），以及那些传递给由bind()所返回的新函数的参数
						return fn.apply(thisArg,args.concat(slice.call(arguments)))
					}
				}
			}

			var twosay3 = one.say.bind(two,'enss');
			twosays();//结果为enss，another object

	设计模式
		单体模式
			思想在于保证一个特定仅有一个实例，意味着第二次使用同一类创建新对象的时候，应该与第一次所创建对象完全相同。
			function Universe(){
				//缓存实例
				var instance;
				//重写构造函数
				Universe = function Universe(){
					return instance;
				}
				//保留原型属性
				Universe.prototype = this;
				//实例
				instance = new Universe();
				//重置构造函数指针
				instance.constructor = Universe;
				//所有功能
				instance.start_time = 0;
				instance.bang = "Big";

				return instance;
			}

			var Universe;
			(function(){
				var instance;

				Universe = function Universe(){

					if(instance){
						return instance;
					}

					instance = this;

					//所有功能
					this.start_time = 0;
					this.bang = "Big";
				}
			}())

		工厂模式
			1.当创建相似对象时执行重复操作
			2.在编译时不知道具体类型（类）的情况下，为工厂客户提供一个创建对象的接口

			var corolla = CarMarker.factory('Compact');
			var solstice = CarMarker.factory('Convertible');
			var cherokee = CarMarker.factory('SUV');
			corolla.drive();//结果为"Vroom,i have 4 doors"
			solstice.drive();//结果为"Vroom,i have 2 doors"
			cherokee.drive();//结果为"Vroom,i have 17 doors"

			//父构造函数
			function CarMarker(){}

			//a method of the parent
			CarMarker.prototype.drive = function(){
				return "Vroom,i have"+ this.doors + "doors";
			}

			//静态工厂方法

			CarMarker.factory = function (type){
				var constr = type,
					newcar;

				//如果构造函数不存在，则发生错误
				if(typeof CarMarker[constr] !== "function"){
					throw{
						name:"Error",
						message:constr + "doern't exist"
					}
				}
				//在这里，构造函数是已知存在的
				//我们使得原型继承父类，但仅继承一次
				if(typeof CarMarker[constr].prototype.drive !== "function"){
					CarMarker[constr].prototype = new CarMarker();
				}
				//创建一个新的实例
				newcar = new CarMarker[constr]();

				//可选择性的调用方法然后返回
				return newcar
			}

			CarMarker.Compact = function (){
				this.door = 4;
			}
			CarMarker.Convertible = function (){
				this.door = 2;
			}
			CarMarker.SUV = function (){
				this.door = 17;
			}
			内置对象工厂
				Object()，最常见的工厂模式

			var o = new Object(),
				n = new Object(1),
				s = Object('1'),
				b = Object(true);

			//test
			o.constructor === Object;//true
			o.constructor === Number;//true
			o.constructor === String;//true
			o.constructor === Boolean;//true
		迭代器模式
			var element;
			while(element = agg.next()){
				//处理该元素
				console.log(element)
			}

			while(agg.hasNext()){
				//处理下一个元素
				console.log(agg.next())
			}

			var agg = (function()(
					var index = 0,
						data = [1,2,3,4,5],
						length = data.length;
					return{
						next:function(){
							var element;
							if(!this.hasNext()){
								return null
							}
							element = data[index];
							index = index + 2;
							return element;
						},
						hasNext:function(){
							return index < length;
						},
						rewind:function(){
							index = 0;
						},
						current: function(){
							return data[index]
						}					
					}
				){}())
		装饰者模式
			Sale.prototype.decorate = function (decorator) {
				var F = function(){},
					overrides = this.constructor.decorators[decorator],
					i,newobj;
				F.prototype = this;
				newobj = new F();
				newobj.uber = F.prototype;
				for(i in overrides){
					if(overrides.hasOwnProperty(i)){
						newobj[i] = overrides[i]
					}
				}
				return newobj;
			}

		策略模式
			var validator = {
				//所有可用的检查
				types:{},
				//在当前验证会话中的
				//错误信息
				messages:[],
				//当前验证配置
				//名称：验证类型
				config:{},
				//接口方法
				//data为键值对
				validate:function(data){
					var i,msg,type,checker,result_ok;
					//重置所有消息
					this.messages = [];

					for(i in data) {
						if(data.hasOwnProperty(i)){
							type = this.config[i];
							checker = this.type[type];

							if(!type){
								continue;
							}
							if(!checker){
								throw{
									name:"ValidationError",
									message:"No handler to validate type" + type;
								}
							}

							result_ok = checker.valadate(data[i]);
							if(!result_ok){
								msg="Invalid value for *"+ i + "," + checker.instuctions;
								this.message.push(msg);
							}
						}
					}
					return this.hasErrors();
				},
				//帮助程序
				hasErrors:function(){
					return this.messages.length !==0;
				}
			}
		外观模式
			创建一个方法以包装重复的方法调用

			var myevent = {
				stop:function(e){
					e.preventDefalut();
					e.stopPropagation();
				}
			}
		代理模式
			一个对象充当另一个对象的接口，介于对象的客户端和对象本身之间，并且对该对象的访问进行保护。

			var videos ={};
			var http = {};
			var proxy ={
				a:{
					//调videos里的对象
				},
				b:{
					//调http里的对象
				}
			};

			proxy负责http和videos之前的通信
			
		中介者模式

		观察者模式(订阅/发布模式)
			subscribers
				一个数组
			subscribe()
				将订阅者添加到subscribers数组
			unsubscribe()
				从订阅者数组subscribers中删除订阅者
			publish()
				循环遍历subscribers中的每个元素，并且调用他们注册时所提供的方法

		var publisher = {
			sunscribers:{
				any:[]
			},
			subscribe:function(fn,type){
				type = type || 'any';
				if(typeof this.subscribers[type] === "undefined"){
					this.subscribers[type] = [];
				}
				this.subscribers[type].push(fn)
			},
			unsubscribe:function(fn,type){
				this.visitSubscribers('unsubscribe',fn,type);
			},
			publish:function(publication,type){
				this.visitSubscribers('publish',publication,type)
			},
			visitSubscribers:function(action,arg,type){
				var pubtype = type || "any",
					subscribers = this.subscribers[pubtype],
					i,
					max = subscribers.length;
				for(i=0; i < max; i +=1){
					if(action === 'publish'){
						sunscribers[i](arg)
					} else {
						if(subscribers[i] === arg){
							subscribers.splice(i,1)
						}
					}
				}
			}
		}
	DOM和浏览器模式
		关注分离
			内容
				HTML文档
			表现
				指定文档外观的CSS样式
			行为
				处理用户交互和文档各种动态变化的JavaScript
		DOM访问的代价是昂贵的，应该减少到最低，这意味着：
			1.避免循环中使用DOM访问。
			2.将DOM引用分配给局部变量，并使用这些局部变量
			3.在可能的情况下使用selectorAPI
			4.当在HTML容器中重复使用时，缓存重复的次数
		创建文档碎片来升级节点信息
		
		var p,t,frag;

		frag = document.createDocumentFragment();
		p = doucument.createElement('p');
		t = document.createTextNode('first paragraph');
		p.appendChild(t);
		frag.appendChild(p);

		p = doucument.createElement('p');
		t = document.createTextNode('second paragraph');
		p.appendChild(t);
		frag.appendChild(p);

		document.body.appendChild(frag);

		事件
			var b = document.getElementById('clickme');
			if(document.addEventListener){//W3C
				b.addEventListener('click',myHandler,false);
			}else if(document.attachEvent){//IE
				b.attachEvent('onclick',myHandler)
			}else{//终极手段
				b.onclick = myHandler;
			}

			function myHandler(e){
				
				var src,parts;

				//获取事件和源元素
				e = e || window.event;
				src = e.target || e.srcElement;

				//实际工作：升级标签
				parts = src.innerHTML.split(": ");
				parts[1] = parseInt(parts[1],10)+1;
				src.innerHTML = parts[0] + ": " +parts[1];
				//无冒泡
				if(typeof e.stopPropagation === "function"){
					e.stopPropagation();
				}
				if(typeof e.cancelBubble !== "undefined"){//IE
					e.cancelBubble = true;
				}

				//阻止默认操作
				if(typeof e.preventDefalut === "function"){
					e.preventDefalut();
				}
				if(typeof e.returnValue !== "undefined"){
					e.returnValue = false;
				}
			}
	设计模式分类
		工厂方法：基于接口数据或事件生成几个派生类的一个实例
		抽象工厂：创建若干类系列的一个实例，无需详述具体的类
		生成器：从表示中分离对象构建；总是创建相同类型的对象
		原型：用于复制或克隆完全初始化的实例
		单例：一个类在全局访问点只要唯一一个实例
		结构型模式：基于构建对象块的想法
		适配器：匹配不同类的接口，因此类可以再不兼容接口的情况下共同工作
		桥接：将对象接口从其实现中分离，因此它们可以独立进行变化
		组合：简单和复合对象的结构，使对象的总和不只是它各部分的总和
		装饰：向对象动态添加备选的处理
		外观：隐藏整个子系统复杂性的唯一一个类
		享元：一个用于实现包含在别处信息的高效共享的细粒度实例
		代理：占位符对象代表真正的对象
		行为模式：基于对象在一起配合工作的方式
		解释器：将语言元素包含在应用程序中的方法，以匹配预期语言的语法
		模板方法：在方法中创建算法的shell，然后将确切的步骤推到子类
		职责链：在对象链之间传递请求的方法，以找到能够处理请求的对象
		命令：将命令执行从其调用程序中分离的方法
		迭代器：顺序访问一个集合中的元素，无需了解该集合的内部工作原理
		中介者：在类之间定义简化的通信，以防止一组类显式引用彼此
		备忘录：捕获对象的内部状态，以能够在以后恢复它
		观察者：向多个类通知改变的方式，以确保类之间的一致性
		状态：状态改变时，更改对象的行为
		策略：在一个类中封装算法，将选择与实现分离
		访问者：向类添加一个新的操作，无需改变类
	设计模式
		Constructor(构造器)模式
			对象创建
				var newObject = {}
				var newObject = new Object();
			对象赋值
				1.“点”语法
					newObject.someKey = "Hello"
				2.中括号语法
					newObject["someKey"] = "Hello"
				3.Object.defineProperty
				4.Object.defineProperties

		Module(模块)模式
			JavaScript中，有几种用于实现模块的方法，包够：
				1.对象字面量表示法
				2.Module模式
					Module模式使用闭包封装“私有”状态和组织。它提供一种包装混合公有/私有方法和变量的方式，防止其泄露至
					全局作用域，并与别的开发人员的接口发生冲突。通过该模式，只需返回一个公有API，而其他的一切都维持在
					私有闭包里。
					示例1：
						var myNamespace = (function(){
							//私有计算器变量
							var myPrivateVar = 0;
							//记录所有参数的私有函数
							var myPrivateMethod = function(foo){
								console.log(foo)
							}
							return {
								//公有变量
								myPublicVar:"foo",
								//调用私有变量和方法的公用函数
								myPublicFunction：function(bar){
									//增加私有计算器值
									myPrivateVar++;
									//传入bar调用私有方法
									myPrivateMethod(bar);
								}
							}
						})()
					示例2:
						var basketModule = (function(){
							//私有
							var basket = [];
							function doSomethingPrivate(){
								//.....
							}
							function doSomethingElse(){
								//...
							}
							//返回一个暴露的公有对象
							return{
								//添加item到购物车
								addItem:function(values){
									basket.push(values);
								},	
								//获取购物车里item数
								getItemCount:function(){
									return basket.length;
								},
								//私有函数的公有形式别名
								doSomething:doSomethingElse,
								//获取购物车所以item的介个总值
								getTotal:function(){
									var itemCount = this.getItemCount(),
									total = 0;
									while(itemCount--){
										total += basket[itemCount].price;
									}
									return total;
								}
							}
						})()
				3.AMD模块
				4.CommonJs模块
				5.ECMAScript Harmony 模块
			优点：简洁，代码公有部分能接触私有部分，外界无法接触私有部分
			缺点：私有部分的扩展性不好

		Revealing Module(揭示模块)模式
			能够在私有范围内简单定义所以函数和变量，并返回一个匿名对象，它拥有指向私有函数的指针，
			该函数是他希望展示为公有的方法。
			示例： 
				var myRevalingModule = function(){
					var myPrivateVar = "Ben Cherry",
						publicVar = "Hey there!";
					function privateFunction(){
						console.log("Name:"+ privateVar);
					}
					function publicSetName(strName){
						privateName = strName;
					}
					function publicGetName(){
						privateFunction()
					}
					//将暴露的公有指针指向私有函数和属性上
					return{
						setName:publicSetName,
						greeting:publicVar,
						getName:publicGetName
					}
				}();
				myRevalingModule.setName("lora")

			优点：
				该模式可以使脚本语法更加一致。在模块底部代码，它也会很容易指出哪些函数和变量可以被公开访问，从而改善可读性。
			缺点： 
				如果一个私有函数引用一个公有函数，在需要打补丁时，公有函数是不能被覆盖的。这是因为私有函数将继续引用私有实现，
				改模式并不适用于公有成员，只适合函数。

		Singleton(单例)模式
			类的实例化次数只能一次。
			示例：
				var mySingleton = (function(){
					//实例保持了Singleton的一个引用
					var instance;
					function init(){
						//Singleton
						//私有方法和变量
						function privateMethod(){
							console.log("I am private")
						}
						var privateVariable = "I am also private";
						var privateRandomNumber = Math.random();
						return {
							//公有方法和变量
							publicMethod:function(){
								console.log("The public can see me!")
							},
							publiceProperty:"I am also public",
							getRandomNumber: function(){
								return privateRandomNumber;
							}
						}
					};
					return {
						//获取Singleton的实例,如果存在就返回，不存在就创建新实例
						getInstance:function(){
							if(!instance){
								instance = init();
							}
							return instance;
						}
					}
				})
			Sington模式适用性的描述如下：
				1.当类只能有一个实例而且客户可以从一个众所周知的访问点访问它时。
				2.该唯一的实例应该是通过子类化可扩展的，并且客户应该无需更改代码就能使用一个扩展的实例时。
				关联到一个场景，可能需要如下代码：
					mySingleton.getInstance = function(){
						if(this._instance == null){
							if(isFoo()){
								this._instance = new FooSingleton();
							}else{
								this._instance = new BasicSingleton();
							}
						}
						return this._instance;
					}
			在实践中，当在系统中确实需要一个对象来协调其他对象时，Sington模式是很有用的。
				var SingtonTester = (function(){
					//options：包含singleton所配置信息的对象
					//e.g var options = {name:"test",pointX:5}
					function Singleton(options){
						options = options || {};
						this.name = "SingtonTester";
						this.pointX = options.pointX || 6;
						this.pointY = options.pointY || 10;
					}

					//实例持有者
					var instance;

					//静态变量和方法的模拟
					var _static = {
						name: "SingtonTester",
						//获取实例的方法，返回sington对象的sington实例
						getInstance: function(options){
							if(instance === undefined){
								instance = new Singleton(options);
							}
							return instance;
						}
					}
					return _static;
				})();

				var SingletonTest = SingletonTest.getInstance({
					pointX:5
				})
				//记录pointX的输出以便验证
				//输出5
				console.log(SingletonTest.pointX)

		Observer(观察者)模式
			一个对象（称为subject）维系一系列依赖于它（观察者）的对象，将有关状态的任何变更自动通知给它们。
			组件： 
				Subject(目标)
					维护一系列的观察者，方便添加或删除观察者
				Observer(观察者)
					为那些在目标状态发生改变时需获得通知的对象提供一个更新接口
				ConcreteSubject(具体目标)
					状态发送改变时，向Observer发出通知，储存ConcreteObesever的状态
				ConcreteObserver(具体观察者)
					存储一个指向ConcreteSubject的引用，实现Observer的更新接口，以使自身状态与目标的状态保持一致
			模拟一个目标可能拥有一系列依赖Observer:
				function ObserveList(){
					this.observerList = [];
				}
				ObserveList.prototype.Add = function(obj){
					return this.observerList.push(obj);
				}
				ObserveList.prototype.Empty = function(){
					this.observeList = [];
				}
				ObserveList.prototype.Count = function(){
					return this.observeList.length;
				}
				ObserverList.prototype.Get = function(index){
					if(index > -1 && index < this.observeList.length){
						return this.observeList[index];
					}
				}
				ObserveList.prototype.Insert = function(obj,index){
					var pointer = -1;
					if(index === 0){
						this.observeList.unshift(obj);//向数组的开头添加一个或更多元素，并返回新的长度
						pointer = index;
					} else if(index === this.observeList.length){
						this.observeList.push(obj);
						pointer = index;
					}
					return pointer;
				}
				ObserveList.prototype.IndexOf = function(obj,startIndex){
					var i = startIndex,pointer = -1;
					while(i < this.observeList[i].length){
						if(this.observeList[i] === obj){
							pointer = i;
						}
						i++;
					}
					return pointer;
				}
				ObserveList.prototype.RemoveIndexAt() = function(index){
					if(index === 0){
						this.observeList.shift();
					}else if(index = this.observeList.length -1){
						this.observeList.pop();
					}
				}
				//使用extension扩展对象
				function extend(obj,extension){
					for(var key in obj){
						extension[key] = obj[key];
					}
				}
				接下来，模拟目标（Subject）和观察者列表上添加、删除或通知观察者的能力
				function Subject(){
					this.observers = new ObserveList();
				}

				Subject.prototype.AddObserver = function(observer){
					this.observers.Add(observer)
				}

				Subject.prototype.RemoveObserver = function(observer){
					this.observers.RemoveIndexAt(this.observers.IndexOf(observer,0))
				}				
				Subject.prototype.Notify = function(context){
					var observerCount = this.observers.Count();
					for(var i = 0; i < observerCount; i++){
						this.observers.Get(i).Update(context);
					}
				}
				然后定义一个框架来创建新的Observer
				//The Observer
				function Observer(){
					this.Update = function(){
						//...
					}
				}
				Publish/Subscribe(发布/订阅)模式
				//非常简单的mail处理程序

				//接收到的消息数量
				var mailCounter = 0;

				//初始化订阅，名称是inbox/newMessage

				//呈现消息预览
				var subscriber1 = subscribe("index/newMessage",function(topic,data){
					//使用从目标subject传递过来的data，一般呈现消息预览
					$(".messageSender").html(data.sender);
					$(".messagePreview").html(data.body);
				})
				//另外一个订阅，使用同样data数据用于不同的任务
				//通过发布者更新所接收消息的数量
				var subscribe2 = subscribe("index/newMessage",function(topic,data){
					$(".newMessageCounter").html(mailCounter++);
				})

				publish("inbox/newMessage",[{
					sender:"hello@google.com",
					body:"Hey there！How are you doing today?"
				}])
				//之后可以用个unsubscribe取消订阅
				unsubscribe(subscriber1)
				unsubscribe(subscriber2)

				优点：解耦
				缺点：难跟踪

				基于jQuery的订阅发布
				//publish
				$(el).trigger("/login",[{username:"test",userData:"test"}])
				//subscribe
				$(el).on("/login",function(event){})
				//unsubscribe
				$(el).off("/login")
				纯JS实现
					var pubsub = {};
					(function(q){
						var topics = {},
							subUid = -1;
						//发布或广播事件，包含特定topic名称和参数（比如传递的数据）
						q.publish = function(topic,args){
							if(!topics[topic]){
								return false;
							}
							var subscribers = topics[topic],
								len = subscribers? subscribers.length:0;
							while(len--){
								subscribers[len].func(topic,args);
							}
							return this;
						}
						//通过特定的名称和回调函数订阅事件，topic/event触发时执行事件
						q.subscribe = function (topic,func){
							if(!topics[topic]){
								topics[topic] = [];
							}
							var token = (++subUid).toString();
							topics[topic].push({
								token:token,
								func:func
							})
							return token;
						}
						//基于订阅上的标记引用，通过特定topic取消订阅
						q.unsubscribe = function (token){
							for(var m in topics){
								if(topics[m]){
									for(var i =0,j = topics[m].length,i < j;i++){
										if(topics[m][i].token === token){
											topics[m].splice(i,1);//删除第i项
											return token;
										}
									}
								}
							}
							return this;
						}
					}(pubsub))
				使用上述实现
					var messageLogger = function (topics,data){
						console.log("Logging" + topics +": " + data);
					} 	
					//订阅者监听订阅的topic，一旦改topic广播一个通知，订阅者就调用回调函数
					var subscription = pubsub.subscribe("inbox/newMessage",messageLogger)
					//发布者负责发布程序感兴趣的topic或者通知，例如：
					pubsub.publish("inbox/newMessage","hello world!")
					//或者
					pubsub.publish("inbox/newMessage",["test","a","b","c"])
					//或者
					pubsub.publish("inbox/newMessage"),{
						sender:"hello@google.com",
						body:"Hey again!"
					})
					//如果订阅者不想被通知了，也可以取消订阅
					pubsub.unsubscribe("inbox/newMessage");

		Mediator(中介者)模式
			“中介者”是指“协助谈判和解决冲突的中立方”。它允许我们公开一个统一的接口，系统的不同部分可以通过该接口进行通信。
			基本实现：
				var mediator =(function(){
					//存储可被广播或监听的topic
					var topics = {};
					//订阅一个topic，提供一个回调函数，一旦topic被广播就执行该回调
					var subscribe = function(topic,fn){
						if(!topics[topic]){
							topics[topic] = []
						}
						topics[topic].push({context:this,callback:fn});
						return this;
					}
					//发布/广播事件到程序剩余部分
					var publish = function(topic){
						var args;

						if(!topics[topic]){
							return false;
						}
						args = Array.prototype.slice.call(arguments,1);
						for(var i =0,l = topics[topic].length; i<l; i++){
							var subscription = topics[topic][i];
							subscription.callback.apply(subscription.context,args);
						}
						return this;
					}

					return{
						Publish: publish,
						Subscribe:subscribe,
						installTo:function(obj){
							obj.subscribe = subscribe;
							obj.publish = publish;
						}
					}
				})()
			优点：它能够将系统中对象或组件之间所需的通信渠道从多对多减少到多对一。由于现在的解耦程度较高，添加新发布者和
				  订阅者相对也容易多了。
			缺点：它会引入单一故障点。

		Prototype（原型）模式
			基于原型继承的模式，可以再其中创建对象，作为其他对象的原型。
			Object.create(prototype,optionalDescriptorObjects)
			
			var myCar = {
				name:"Food Escort",
				drive:function(){
					console.log("Weeee.I'm driving")
				},
				panic:function(){
					console.log("Wait.How do you stop this thing?")
				}
			}

			//使用Object.create实例化一个新car
			var yourcar = Object.create(myCar);
			//现在可以看到一个对象是另外一个对象的原型
			console.log(yourCar.name)

			Object允许我们使用第二个提供的参数来初始化对象属性

			var vehicle = {
				getModel:function(){
					console.log("The model of this vehicle is" + this.model)
				}
			}

			var car = Object.create(vehicle,{
				"id":{
					value:"hah",
					enumerable:true
				},
				"model":{
					value:"Ford",
					enumerable:true
				}
			})
			另一种实现
			var beget = (function(){
				function F(){}
				return function(proto){
					F.prototype = proto;
					return new F()
				}
			})()

		Command(命令)模式
			Command模式旨在将方法调用。请求或操作封装到单一对象中，从而根据我们不同的请求对客户进行参数化
			和传递可供执行的方法调用。此外，这种模式将调用操作的对象与知道如何实现该操作的对象解耦，并在交
			换具体类（对象）方面提供更大的整体灵活性。
			示例：
				(function(){
					var CarManager = {
						//请求信息
						requestInfo:function(model,id){
							return "xxx"
						},
						//订购汽车
						buyVehicle:function(model,id){
							return "xxx"
						},
						//组织一个view
						arrangeViewing:function(model,id){
							return "xxx"
						}
					}
				})()
				CarManager.execute = function (name){
					return CarManager[name] && CarManager[name].apply(CarManager,[].slice.call(arguments,1))
				}

				最终调用如下：
					CarManager.execute("requestInfo","a",1)
					CarManager.execute("arrangeViewing","b",2)
		Facade(外观)模式
			为更大的代码体提供了一个方便的高层次接口，能够隐藏其底层的真实复杂性。
			例如最有名的jQuery的$(el)就是外观模式
			优点：易于使用和实现该模式时占用空间小
			几个例子：
			var addMyEvent = function (el,ev,fn){
				if(el.addEventListener){
					el.addEventListener(ev,fn,false)
				}else if(el.attachEvent){
					el.attachEvent("on"+ev,fn)
				}else{
					el["on" + ev] = fn;
				}
			}

			$(document).ready()的实现

			bindReady:function{
				//...
				if(doucument.addEventListener){
					document.addEventListener("DOMContentLoaded",DOMContentLoaded,false)
					window.addEventListener("load",jQuery.ready,false)
				}else if(doucument.attachEvent){
					document.attachEvent("onreadystatechange",DOMContentLoaded)
					window.attachEvent("onload",jQuery.ready)
				}
			}

			var module = (function(){
				var _private = {
					i:5,
					get:function(){
						console.log("current value:" + this.i)
					},
					set:function(val){
						this.i = val
					},
					run:function(){
						console.log("running")
					},
					jump:function(){
						cosole.log("junping")
					}
				}
				return{
					facade:function(args){
						_private.set(args.val);
						_private.get();
						if(args.run){
							_private.run()
						}
					}
				}
			})()
			//输出"running",10
			module.facade({run:true,val:10});
			缺点：注意性能问题

		Factory(工厂)模式
			是另一种创建型模式，涉及创建对象的概念。其分类不同于其他模式的地方在于它不显式地要求使用
			一个构造函数。而Factory可以提供一个通用的接口来创建对象，我们可以指定我们所希望创建的工厂对象的类型。
			示例：
				//Types.js - 构造函数存放文件
				//定义Car构造函数
				function Car(options){
					//默认值
					this.doors = options.doors || 4;
					this.state = options.state || "brand new";
					this.color = options.color || "silver";
				}

				//定义Truck构造函数
				function Truck(options){
					this.state = options.state || "used";
					this.wheelSize = options.wheelSize || "large";
					this.color = options.color || "blue";
				}
				//FactoryExample.js
				//定义vehicle工厂的大体代码
				function VehicleFactory(){}
				//定义改工厂factory的原型和试用工具，默认vehicleClass是car
				VehicleFactory.prototype.vehicleClass = Car;
				//创建新vehicle实例的工厂方法
				VehicleFactory.prototype.createVehicle = function(options){
					if(options.vehicleType === "car"){
						this.vehicleClass = Car;
					}else{
						this.vehicleClass = Truck;
					}
					return new this.vehicleClass(options)
				}
				//创建生成汽车的工厂实例
				var carFactory = new VehicleFactory();
				var car = carFactory.createVehicle({
					vehicleType:"car",
					color:"yellow",
					doors:6
				})
				//测试汽车是由vehicleClass的原型prototype里的Car创建的
				//输出true
				console.log(car instanceof Car)

				//输出：Car对象，color:"yellow",doors:6,state:"brand new"
				console.log(car)

				var movingTruck = carFactory.createVehicle({
					vehicleType:"truck",
					color:"red",
					state:"like new",
					wheelSize:"small"
				})

				console.log(movingTruck instanceof Truck);
				console.log(movingTruck)
			使用场景：
				1.当对象或组件设置涉及高复杂性时
				2.当需要根据所在不同环境轻松生成对象的不同实例时
				3.当处理很多共享相同属性的小型对象或组件时
				4.在编写只需要满足一个API契约（亦称鸭子类型）的其他对象的实例对象时。对于解耦很有用的。
			不应该使用：
				如果应用错误，这种模式会为应用程序带来大量不必要的复杂性。除非创建对象提供一个接口是我们
				正在编写的库或框架的设计目标，否则我建议坚持使用显式狗杂函数，以避免不必要的开销。

				由于对象创建的过程实际上是藏身接口之后抽象出来的，单元测试也可能带来问题，这取决于对象创建
				的过程有多复杂
			抽象工厂：
				用于封装一组具有共同目标的单个工厂。
				应当使用的场景是：一个系统必须独立于它所创建的对象的生成方式，或它需要与多种对象类型一起工作。
				示例：
					var AbstractVehicleFactory = (function(){
						//存储车辆类型
						var types = {};
						return {
							getVehicle:function(type,customizations){
								var Vehicle = types[type];
								return (Vehicle)?return new Vehicle(customizations):null;
							},
							registerVehicle:function(type,Vehicle){
								var proto = Vehicle.prototype;
								//只注册实现车辆契约的类
								if(proto.drive && proto.breakDown){
									types[type] = Vehicle;
								}
								return AbstractVehicleFactory;
							}
						}
					})()
					//用法 
					AbstractVehicleFactory.registerVehicle("car",Car);
					AbstractVehicleFactory.registerVehicle("truck",Truck);
					//基于抽象车辆类型实例化
					var car = AbstractVehicleFactory.getVehicle("car",{
						color:"green",
						state:"like now"
					})
					//同理truck
					var truck = AbstractVehicleFactory.getVehicle("truck",{
						wheelSize:"medium",
						color:"yellow"
					})
			Mixin模式
				Mixin是可以轻松被一个子类或一组子类继承功能的类，目的是函数复用。
					子类化
						子类化这个术语是指针对一个新对象，从一个基础或超类对象中继承相关的属性。
						在传统的面向对象编程中，类B是从另外一个类A扩展得来。这里我们认为A是一个
						超类，B是A的一个子类。因此，B的所有实例从A处继承了相关方法。但是B仍然能
						够定义自己的方法，包够那些A最初所定义方法的重写。

						A中的一个方法，在B里以及被重写了,那么B还需要调用A的这个方法吗，我们称此
						为方法链。B需要调用构造函数A（超类）吗，我们称此为构造函数链。

						示例：
							var Person = function(firstName,lastName){
								this.firstName = firstName;
								this.lastName = lastName;
								this.gender = "male";
							}

							//Person的新实例很容易像如下这样创建
							var clark = new Person("Clark","Kent");
							//为超人（Superhero）定义一个子类构造函数
							var Superhero = function(firstName,lastName,powers){
								//调用超类的构造函数，然后使用.call方法进行调用从而进行初始化
								Person.call(this,firstName,lastName)
								//最后，保存powers，在正常Person里找不到的特性数组
								this.powers = powers;
							}
							Superhero.prototype = Object.create(Person.prototype)
							var superman = new Superhero("Clark","Kent",["flight","heat-vision"])
							console.log(superman)
					Mixin 
						在JavaScript中，我们可以将继承Mixin看作为一种通过扩展收集功能的方式。
						示例：
							var myMixins = {
								moveUp:function(){
									console.log("move up");
								},
								moveDown:function(){
									console.log("move down");
								},
								stop:function(){
									console.log("stop！ in the name of love")
								}
							}

							//carAnimator构造函数的大体代码
							function carAnimator(){
								this.moveLeft = function(){
									console.log("move left");
								}
							}
							//personAnimator构造函数的大体代码
							function perAnimator(){
								this.moveReadomly = function(){}
							}
							//使用Mixin扩展2个构造函数
							_.extend(carAnimator.prototype,myMixins);
							_.extend(perAnimator.prototype,myMixins);
							//创建carAnimator的新实例
							var myAnimator = new carAnimator();
							myAnimator.moveLeft();
							myAnimator.moveDown();
							myAnimator.stop();
						示例2:
							//定义简单的Car构造函数
							var Car = function(settings){
								this.model = settings.model || "no model provided";
								this.color = settings.color || "no color provided";
							}
							//Mixin
							var Mixin = function(){};

							Mixin.prototype = {
								driveForward:function(){
									console.log("drive forward")
								},
								driveBackward:function(){
									console.log(drive backward)
								},
								driveSideways:function(){
									console.log("drive sideways")
								}
							}
							//通过一个方法将现有对象扩展到另外一个对象上
							function augment(receivingClass,givingClass){
								//只提供特定的方法
								if(arguments[2]){
									for(var i = 2,len = arguments.length;i <len;i++){
										receivingClass.prototype[arguments[i]] = givingClass.prototype[arguments[i]]
									}
								}else{//提供所有方法
									for(var methodName in givingClass.prototype){
										//确保接受类不包含所处理方法的同名方法
										if(!Object.hasOwnProperty(receivingClass.prototype,methodName)){
											receivingClass.prototype[methodName] = givingClass.prototype[givingClass]
										}
										//另一种方式
										// if(!receivingClass.prototype[methodName]){
										// 	receivingClass.prototype[methodName] = givingClass.prototype[givingClass]
										// }
									}
								}
							}
							//给Car构造函数增加"driveForward"和"driveBackward"两个方法
							augment(car,Mixin,"driveForward","driveBackward")
							//创建一个新Car
							var myCar = new Car({
								model:"Ford Escort",
								color:"blue"
							})
							//测试确保新增方法可用
							myCar.driveForward();
							myCar.driveBackward();
							//也可以通过不声明特定方法名的形式，将Mixin的所有方法都添加到Car里
							augment(Car,Mixin)

							var mySportsCar = new Car({
								model:"Porsche",
								color:"red"
							})
							mySportsCar.driveSideways();
						优点：有助于减少系统中的重复功能及增加函数复用
						缺点：它会导致原型污染和函数起源方面的不确定性

			Decorator（装饰者）模式
				一种结构型设计模式，旨在促进代码复用。
				示例1：
					//车辆vehicle构造函数
					function vehicle(vehicleType){
						//默认值
						this.vehicleType = vehicleType || "car";
						this.model = "default";
						this.license = "00000-000";
					}
					//测试基本的vehicle实例
					var testInstance = new vehicle("car");
					console.log(testInstance);
					//输出
					//vehicle:car,model:default,license:00000-000

					//创建一个vehicle实例进行装饰
					var truck = new vehicle("truck");
					//给truck装饰新的功能
					truck.setModel = function (modelName){
						this.model = modelName;
					}
					truck.setColor = function(color){
						this.color = color;
					}
					//测试赋值操作是否正常工作
					truck.setModel("CAT");
					truck.setColor("blue");
					console.log(truck);
					//输出
					//vehicle:truck,model:CAT,color:blue

					//下面的代码，展示vehicle依然是不被改变的
					var secondInstance = new vehicle("car");
					console.log(secondInstance)
					//输出
					//vehicle:car,model:default,license:00000-000
				示例2：
					//被装饰的对象构造函数
					function MacBook(){
						this.cost = function(){return 997};
						this.screenSize = function (){return 11.6};
					}
					//Decorator 1
					function Memory(macbook){
						var v = macbook.cost();
						macbook.cost = function(){
							return v + 75;
						}
					}
					//Decorator 2
					function Engraving(macbook){
						var v = macbook.cost();
						macbook.cost = function(){
							return v + 200;
						}
					}
					//Decorator 3
					function Insurance(macbook){
						var v = mac.cost();
						macbook.cost = function(){
							return v + 250
						}
					}

					var mb = new MacBook();
					Memory(mb);
					Engraving(mb);
					Insurance(mb);
					//输出：1522
					console.log(mb.cost())
					//输出：11.6
					console.log(mb.screenSize())
				伪经典Decorator(装饰者)
					接口
					抽象Decorator(抽象装饰者)
				使用jQuery的装饰者
					示例：
						var decoratorApp = decoratorApp || {};
						//定义要使用的对象
						decoratorApp = {
							defaults:{
								validate:false,
								limit:5,
								name:"foo",
								welcome:function(){
									console.log("welcome!")
								}
							},
							options:{
								validate:true,
								name:"bar",
								helloWorld:function(){
									console.log("hello world")
								}
							},
							settings:{},
							printObj:function(obj){
								var arr = [],
									next;
								$.each(obj,function(key,val){
									next = key + ":";
									next += $.isPlainObject(val)?printObj(val):val;
									arr.push(next);
								})
								return "{"+arr.join(",")+"}"
							}
						}
						//合并defaults和options，没有显示修改defaults
						decoratorApp.settings = $.extend({},decoratorApp.default,decoratorApp.options)
						//这里所做的就是装饰可以访问defaults属性和功能的方式（options也一样）,defaults本身未作改变
						$("#log").append(decoratorApp.printObj(decoratorApp.settings)+
							+decoratorApp.printObj(decoratorApp.options)+
							+decoratorApp.printObj(decoratorApp.defaults))
				优点：透明，灵活
				缺点：如果管理不当，它会极大地复杂化应用程序架构

			Flyweight(享元)模式 
				是一种经典的结构型解决方案，用于优化重复、缓慢及数据共享效率较低的代码。
				它旨在通过与相关的对象共享尽可能多的数据数据来减少应用程序中的内存使用。

				使用享元模式
					方式有两种：
						1.用于数据层，处理内存中保存的大量相似对象的共享数据
						2.用于DOM层，享元可以用作中央事件管理器，来避免将事件处理程序附加到
						  父容器中的每个字元素上，而将事件处理程序附加到这个父容器上
				享元和共享数据

				实现经典享元
					三个享元组件：
						1.享元：描述一个接口，通过这个接口享元可以接受并作用于外部状态
						2.具体享元：实现享元接口，并存储内部状态。对象必须是可共享的，并
						  能够控制外部状态
						3.享元工厂:创建并管理享元对象
					几个定义：
						1.CoffeeOrder：享元
						2.CoffeeFlavor：具体享元
						3.CoffeeOrderContext:辅助器
						4.CoffeeFlavorFactory：享元工厂
						5.testFlyweight：享元应用
				鸭子补丁“实现”
					//在JS里模拟纯虚拟继承 
					Function.prototype.implementsFor = function(parentClassOrObject){
						if(parentClassOrObject.constructor === Function){
							//正常继承
							this.prototype = new parentClassOrObject();
							this.prototype.constructor = this;
							this.prototype.parent = parentClassOrObject.prototype;
						}else{
							//纯虚拟继承
							this.prototype = parentClassOrObject;
							this.prototype.constructor = this;
							this.prototype.parent = parentClassOrObject;
						}
						return this;
					}
					//享元对象
					var CoffeeOrder = {
						//接口
						serveCoffee:function(context){},
						getFlavor:function(){}
					}
					//实现CoffeeOrder的具体享元对象
					function CoffeeFlavor(newFlavor){
						var flavor = newFlavor;
						//如果已经为某一个功能定义了接口，则实现该功能
						if(typeof this.getFlavor === "function"){
							this.serveCoffee = function(context){
								console.log(context.getTable())
							}
						}
					}
					//为CoffeeOrder实现接口
					CoffeeFlavor.implementsFor(CoffeeOrder);
					//处理coffee订单的table数
					function CoffeeOrderContext(tableNumber){
						return{
							getTable:function(){
								return tableNumber;
							}
						}
					}
					//享元工厂对象
					function CoffeeFlavorFactory(){
						var flavors = [],
						return{
							getCoffeeFlavor:function(flavorName){
								var flavor = flavors[flavorName];
								if(flavor === undefined){
									flavor = new CoffeeFlavor(flavorName);
									flavors.pushc[flavorName,flavor]
								}
								return flavor
							},
							getTotalCoffeeFlavorsMade:function(){
								return flavors.length;
							}
						}
					}
					//样例用法
					//testFlyweight()
					function testFlyweight(){
						//已订购的flavor
						var flavors = new CoffeeFlavor(),
						//订单table
						tables = new CoffeeOrderContext(),
						//订单数量
						ordersMade = 0,
						//TheCoffeeFlavorFactory实例
						flavorFactory;
						function takeOrders(flavorIn,table){
							flavors[ordersMade] = flavorFactory.getCoffeeFlavor(flavorIn);
							tables[ordersMade++] = new CoffeeOrderContext(table)
						}
						flavorFactory = new CoffeeFlavorFactory();

						takeOrders("a",1)
						takeOrders("b",2)
						takeOrders("c",3)
						takeOrders("d",4)
						takeOrders("e",5)
						takeOrders("f",6)
						takeOrders("g",7)

						for(var i = 0; i < ordersMade; ++i){
							flavors[i].serveCoffee(tables[i])
						}
						console.log("");
						console.log(flavorFactory.getTotalCoffeeFlavorsMade())
					}
			转换代码以使用享元模式
				示例:
					var Book = function(id,title,author,genre,pageCount,publusherID,ISBN,checkoutDate,checkoutMember,dueReturnDate,availability){
						this.id = id;
						this.title = title;
						this.author = author;
						this.genre = genre;
						this.pageCount = pageCount;
						this.publusherID = publusherID;
						this.ISBN = ISBN;
						this.checkoutDate = checkoutDate;
						this.checkoutMember = checkoutMember;
						this.dueReturnDate = dueReturnDate;
						this.availability = availability;
					}

					Book.prototype = {
						getTitle:function(){
							return this.title;
						},
						getAuthor:function(){
							return this.author;
						},
						getISBN:function(){
							return this.ISBN;
						},
						//...
						updateCheckoutStatus:function(bookID,newStatus,checkoutDate,checkoutMember,newReturnDate){
							this.id = bookID;
							this.availability = newStatus;
							this.checkoutDate = checkoutDate;
							this.checkoutMember = checkoutMember;
							this.dueReturnDate =  newReturnDate;
						},
						extendCheckoutPeriod:function(bookID,newReturnDate){
							this.id = bookID;
							this.dueReturnDate = newReturnDate;
						},
						isPastDue:function(bookID){
							var currentDate = new Date();
							return currentDate.getTime() > Date.parse(this.dueReturnDate)
						}
					}
			基本工厂
				示例：
					//书籍工厂单例
					var BookFactory = (function(){
						var existingBooks = {},existingBook;
						return{
							createBook:function(title,author,genre,pageCount,publusherID,ISBN){
								//如果书籍之前已经创建，则找出并返回它
								//！！强制返回布尔值
								existingBook = existingBooks[ISBN];
								if(!!existingBook){
									return existingBook;
								} else{
									//如果没找到，则创建一个该书的新实例，并保存
									var book = new Book(title,author,genre,pageCount,publusherID,ISBN);
									existingBooks[ISBN] = book;
									return book;
								}
							}
						}
					})
			管理外部状态
				示例：
					//书籍记录管理器单例
					var BookRecordManager = (function(){
						var bookRecordDatabase = {};
						return{
							//添加新书到图书馆系统
							addBookRecord:function(id,title,author,genre,pageCount,publusherID,ISBN,checkoutDate,checkoutMember,dueReturnDate,availability){
								var book = BookFactory.createBook(title,author,genre,pageCount,publusherID,ISBN);
								bookRecordDatabase[id] = {
									checkoutMember:checkoutMember,
									checkoutDate:checkoutDate,
									dueReturnDate:dueReturnDate,
									availability:availability,
									book:book
								}
							},
							updateCheckoutStatus:function(bookID,newStatus,checkoutDate,checkoutMember,newReturnDate){
								var record = bookRecordDatabase[bookID];
								record.availability = newStatus;
								record.checkoutDate = checkoutDate;
								record.checkoutMember = checkoutMember;
								record.dueReturnDate = newReturnDate;
							},
							extendCheckoutPeriod:function(bookID,newReturnDate){
								bookRecordDatabase[bookID].dueReturnDate = newReturnDate;
							},
							isPastDue:function(bookID){
								var currentDate = new Date();
								return currentDate.getTime() > Date.parse(bookRecordDatabase[bookID].dueReturnDate)
							}
						}
					})
			享元模式和DOM
				示例1
					var stateManager = {
						fly:function(){
							var self = this;
							$("#container").unbind().on("click",function(e){
								var target = $(e.originalTarget || e.srcElement){
									if(target.is("div.toggle")){
										self.handleClick(target)
									}
								}
							})
						},
						handleClick:function(elem){
							elem.find("span").toggle("slow");
						}
					}
				示例2
					jQuery.single = (function(o){
						var collection = jQuery([1]);
						return function (element){
							//将元素赋值给集合
							collection[0] = element;
							//返回集合
							return collection;
						}
					})

					$("div").on("click",function(){
						var html = jQuery.single(this).next.html();
						console.log(html)
					})

	MV*模式
		MVC（模型-视图-控制器）、MVP（模型-视图-表示器）、MVVM（模型-视图-视图模型）

		MVC 
			为我们提供了什么：
				1.整体维护更容易。数据中心是否改变，什么时候需要更新应用程序这点很清楚，
				  这意味着是Model(模型)或者也可能是Controller(控制器)的变化，或者仅仅
				  是视觉，这意味着View(视图)的改变。
				2.解耦Model(模型)和View(视图)，意味着它能够更直接地编写业务逻辑的单元测试。
				3.在整个应用程序中，底层Model(模型)和Controller(控制器)代码的重复被消除了。
				4.取决于应用程序的大小和角色的分离，这种模块性可以让负责核心逻辑的开发人员
				  和负责用户界面的开发人员同时工作。
		MVP
	MVVM 
		优点：
			1.MVVM使得UI和为UI提供驱动的行为模块的并行开发变得更容易
			2.MVVM使View抽象化，从而减少代码背后所需的业务逻辑量
			3.ViewModel在单元测试中的使用比在时间驱动代码中的使用更加容易
			4.不需要考虑UI自动化和交互就可以测试ViewModel
		缺点：
			1.对应简单的UI来说，使用MVVM有些大材小用
			2.数据绑定可以是声明性的，使用也很方便，但比命令式代码更难调试，在命令式代码中，我
			  们只需设置断点
			3.大型应用程序中的数据绑定可以产生大量的标记。我们不想看到的情况是：绑定比被绑定的
			  对象还要繁重
			4.在较大应用程序中，预先设计大量的ViewModel可能更加困难
	Backbone.js与KnockoutJS
		1.这两个库设有不同的目标，它通常不会像是选择MVC或MVVM那样简单
		2.如果数据绑定和双向沟通都是你的主要关注点，那么KnockoutJS一定是要选择的方向。实际上，存储
		  在DOM节点中的任何属性或值都可以使用这种方法被映射到JavaScript对象。
		3.Backbone的优势在于它易于与RESTful服务相集成，而KonckoutJS Model只是JavsScript对象，用于更
		  新Model的代码必须由开发人员编写
		4.KonckoutJS重点关注自动化UI绑定，如果尝试使用Backbone来完成这项工作，就需要更详细的自定义代
		  码。这对于Backbone本身不是问题，因为它有意尝试脱离UI。但Knockback试图协助解决这个问题
		5.通过使用KonckoutJS，我们可以将自己的函数绑定至ViewModel observables,任何时候observable发生
		  变化时，都会执行这些observables。这使我们能够实现与在Backbone中相同级别的灵活性。
		6.Backbone内置一个强大的路由解决方案，而KnockoutJS却没有提供可用的路由选择。但如果需要，我们
		  可以使用插件来替代这一行为
		总结：我个人发现KnockoutJS更适用于小型应用程序，而在创建大型应用程序时，Backbone的特性集确实发
		      挥了它的作用。

模块化的JavaScript设计模式
	AMD（异步模块定义）
		入门
			示例1：//了解AMD：define()
				//演示目的，定义module_id(myModule)
				define("myModule",
					["foo","bar"],
					//模块定义函数，依赖（foo和bar）作为参数映射到函数上
					function (foo,bar){
						//返回定义的模块输出（例如，要暴露的内容）
						//在这里创建你的模块
						var myModule = {
							doStuff:function(){
								console.log("Yay!Stuff")
							}
						}
					}
				)
				//另一种方式
				define("myModule",
					["math","graph"],
					function (math,graph){
						//注意和AMD不太一样，通过特定的语法以不同的形式定义模块
						return {
							plot:function(x,y){
								return graph.drawPie(math.randomGrid(x,y))
							}
						}

					}
				)
			示例2：//了解AMD：require()
				//本例中，foo和bar是两个外部模块，两个模块加载以后的输出座位回调函数的参数传入
				require(["foo","bar"],function(foo,bar){
					//剩余的代码
					foo.doSomething();
				})
			示例3：//动态加载依赖
				define(function(require){
					var isReady = false,foobar;
					//注意模块内部里的require定义
					require(["foo","bar"],function(foo,bar){
						isReady = true;
						foobar = foo() + bar();
					})
					//依然返回一个模块
					return{
						isReady:isReady,
						foobar:foobar
					}
				})
			示例4：//了解AMD插件
				//使用AMD可以加载任意格式的内容（包括文本文件和HTML）
				//这种方式可以用于模板依赖，以便在页面加载的时候进行做换肤方面的工作
				define(["./templates","text!./template.md","css!./template.css"],function(templates,template){
					console.log(templates);
					//利用模板继续处理
				})
			示例5：//使用RequireJS加载AMD模块
				require("app/myModule",function(myModule){
					//开始主模块，顺序加载其它模块
					var module = new myModule();
					module.doStuff();
				})
			示例6：//使用curl.js加载AMD模块
				curl(["app/myModule.js"],function(myModule){
					//开始主模块，顺序加载其它模块
					var module = new myModule();
					module.doStuff();
				})
		AMD好处：
			1.对于如何完成灵活模块定义提供了明确的建议
			2.比很多人依赖现有全局命名空间和<script>标签解决方案都更加简洁。
			  有一种简洁的方法可以声明肯呢过的独立模块和依赖。
			3.封装模块定义，帮助我们避免全局命名空间被污染。
			4.可以说它比一些替代解决方案运行的更好。它没有跨域、本地或调试的问题，
			  也不依赖服务端的工具。大多数AMD加载器支持浏览器中的加载模块，无需构
			  建流程。
			5.提供一个“transport”方法，将多个模块包括在单个文件中。CommonJS等其他
			  方法尚未在transport格式方面达成一致
			6.如果需要，可以延迟加载脚本
	CommonJS
		入门
			CommonJS模块基本上包含两个主要部分：自由变量exports,它包含一个模块希望其他模块能够使用
			的对象，以及require函数，模块可以使用该函数导入(import)其他模块的导出(exports)

			示例1：//了解CommonJS：require()和导出
				//package/lib是我们需要的一个依赖
				var lib = require("package/lib");

				//我们模块的行为
				function foo(){
					lib.log("hello world")
				}
				//导出（暴露）foo给其他模块
				exports.foo = foo;
			示例2：//第一个CommonJS示例的AMD等效代码
				define(function(require){
					var lib = require("package/lib");
					//我们模块的行为
					function foo(){
						lib.log("hello world")
					}
					//导出（暴露）foo给其他模块
					return {
						foobar:foo
					}
				})
			示例3：
				define(function (require, exports, module) {
					var lib = require("package/lib");
					//我们模块的行为
					function foo(){
						lib.log("hello world")
					}
					module.exports = foo;
				})
		使用多个依赖
			app.js
				var modA = require("./foo");
				var modB = require("./bar");

				exports.app = function(){
					console.log("I am an application")
				}
				exports.foo = function(){
					return modA.helloWorld();
				}
			bar.js
				exports.name = "bar";
			foo.js
				require("./bar");
				exports.helloWorld = function(){
					return "Hello World"
				}
	AMD和CommonJS：互相竞争，标准同效
		AMD和CommonJS都是有效的模块格式，有不同的最终目标
		
		AMD采用浏览器优先的开发方法，选择异步行为和简化的向后兼容性，但是它没有任何I/O的概念。
		它支持对象、函数、构造函数、字符串、JSON以及很多其他类型的模块，在浏览器中原生运行。其
		使用非常的灵活。

		另一方面，CommonJS采用服务器优先的方法，假定同步行为，没有全局概念这个精神包袱，并试图
		迎合未来技术（在服务器上）。我们这种做的意思是，由于CommonJS支持非包装模块，它可以更接
		近下一代ES Harmony规范，让我们摆脱AMD强制执行的define()保质期。但CommonJS模块仅将对象作
		为模块给予支持
	UMD:用于插件的AMD和CommonJS兼容模块
		define(function (require, exports, module) {
			var lib = require("package/lib");
			//我们模块的行为
			function foo(){
				lib.log("hello world")
			}
			module.exports = foo;
		})
	ES Harmony

jQuery中的设计模式
	Composite（组合）模式
		以统一的方式来处理单个对象和组合。
		例如：
			//single elements
			$("#singleItem").addClass("active")
			//Collections of elements
			$(".div").addClass("active")

			//addClass源码
			addClass:function(value){
				var classNames,i,l,elem,setClass,c,cl;
				if(jQuery.isFunction(value)){
					return this.each(function(j){
						jQuery(this).addClass(value.call(this,j,this.className))
					})
				}
				if(value && typeof value === "string"){
					classNames = value.split(rspace);
					for(i = 0,l = this.length; i < l; i++){
						elem = this[i];
						if(elem.nodeType === 1){
							if(!elem.className && classNames.length ===1){
								elem.className = value;
							}else{
								setClass = "" + elem.className + "";
								for(c = 0,cl = classNames.length;c < cl;c++){
									if(!~setClass.indexOf("" + classNames[c] + "")){
										setClass += classNames[c] = "";
									}
								}
								elem.className = jQuery.trim(setClass);
							}
						}
					}
				}
				return this;
			}
		//学到一招!~用于indexOf判断，逼格瞬间提升
	Adapter（适配器）模式
		将对象或类的接口转变为与特定的系统兼容的接口
		我们可能用过的适配器的示例是jQuery的jQuery.fn.css().它有助于标准化接口，以展示样式应用于多种浏览器
		//跨浏览器透明度
		//opacity:0.9;chrome、ios、android
		//filter:alpha(opacity=90);IE6-IE8
		//设置opacity
		$(".container").css({opacity:.5})
		//获取opacity
		var currentOpacity = $(".container").css("opacity")
		相应jQuery核心cssHook使上面代码得以实现
		get:function(elem,computed){
			//IE使用filter获取opacity
			return ropacity.test(computed && elem.currentStyle?
				elem.currentStyle.filter:elem.style.filter || "")?
			(parseFloat(RegExp.$1)/100)+"":computed?"1":""
		},
		set:function(elem,value){
			var style = elem.style,
				currentStyle = elem.currentStyle,
				opacity = jQuery.isNumeric(value)?"alpha(opacity="+value*100+")":"",
				filter = currentStyle && currentStyle.filter || style.filter || "";
			//如果没有layout,IE在opacity方面有问题
			//强制设置zoom level
			style.zoom = 1;
			//如果设置opacity为1,并且没有其他的filter存在，尝试删除filter属性
			if(value >= 1 && jQuery.trim(filter.replace(ralpha,"")) ===""){
				//设置filter属性为null，“”，“”时，该属性依然存在
				style.removeAttribute("filter")

				//如果没有filter样式属性了，也就完成了
				if(currentStyle && !currentStyle.filter){
					return;
				}
				//否则，设置新filter值
				style.filter = ralpha.test(filter)?
					filter.replace(ralpha,opacity):filter+""+opacity;
			}
		}
	Facade(外观)模式
		该模式为更大的（可能更复杂）的代码体提供了一个更简单的接口
		例如 jQuery.ajax()
			$.get(url,data,callback,dataType);
			$.post(url,data,callback,dataType);
			$.getJSON(url,data,callback);
			$.getScript(url,data,callback);
		这些是在后台进行转变
			//$.get()
			$.ajax({
				url:url,
				data:data,
				dataType:dataType
			}).done(callback)
			//$.post()
			$.ajax({
				type:"POST"
				url:url,
				data:data,
				dataType:dataType
			}).done(callback)
			//$.getJSON()
			$.ajax({
				url:url,
				data:data,
				dataType:"json"
			}).done(callback)
			//$.getScript()
			$.ajax({
				url:url
				dataType:"script"
			}).done(callback)
		用于规范XHR的jQuery核心中的代码
			//创建xhrs的函数
			function createStandardXHR(){
				try{
					return new window.XMLHttpRequest();
				}catch(e){

				}
			}

			function createActiveXHR(){
				try{
					return new window.ActiveObject("Microsoft.XMLHTTP");
				}catch(e){

				}
			}
			//创建request请求对象
			jQuery.ajaxSettings.xhr = window.ActiveObject?
				function(){
					return !this.isLocal && createStandardXHR() || createActiveXHR();
				} :
				createStandardXHR;
	Observer(观察者)模式
		对象可订阅其它对象的地方，并在感兴趣的事件发生时获得通知
		jQuery早期版本使用jQuery.bind()(subscribe)、jQuery.trigger()(publish)、jQuery.unbind()(unsubscribe)
		可以访问这些自定义事件，但在新版本，可以用jQuery.on()/jQuery.trigger()和jQuery.off()来实现
		示例1：
			//等价于subscribe(topicName,callback)
			$(document).on("topicName",function(){
				//..执行相应行为
			})
			//等价于publish(topicName)
			$(document).trigger("topicName");
			//等价于unsubscribe(topicName)
			$(document).off("topicName")
		示例2：
			(function($){
				var o = $({});
				$.subscribe = function(){
					o.on.apply(o,arguments)
				}
				$.unsubscribe = function(){
					o.off.apply(o,arguments)
				}
				$.publish = function(){
					o.trigger.apply(o,arguments)
				}
			}(jQuery))
		示例3：//多用途对象（jQuery.Callbacks）
			var topics = {};
			jQuery.Topic = function(id){
				var callbacks,
					topic = id && topics[id];
				if(!topic){
					callbacks = jQuery.Callbacks();
					topic = {
						publish:callbacks.fire,
						subscribe:callbacks.add,
						unsubscribe:callbacks.remove
					}
					if(id){
						topics[id] = topic;
					}
				}
				return topic;
			}
			//使用
			//订阅者
			$.Topic("mailArrived").subscribe(fn1);
			$.Topic("mailArrived").subscribe(fn2);
			$.Topic("mailSent").subscribe(fn1);
			//发布者
			$.Topic("mailArrived").publish("hello world");
			$.Topic("mailSent").publish("woo!mail!");
			//这里,当"mailArrived"通知发布时，"hello world!"推送到fn1和fn2上
			//"mailSent"通知发布时，"woo!mail!"也推送到fn1
			//输出：
			//hello world
			//fn2输出：hello world
			//woo!mail!
	Iterator(迭代器)模式
		迭代器（允许我们遍历集合的所有元素的对象）
		顺序访问聚合对象的元素，无需公开其基本形式
		jQuery.fn.each()
		看看源码
		each:function(object,callback,args){
			var name,i = 0,
				length = object.length,
				isObj = length === undefined || jQuery.isFunction(object);
			if(args){
				if(isObj){
					for(name in object){
						if(callback.apply(object[name],args) === false){
							break;
						}
					}
				}else{
					for(;i < length){
						if(callback.apply(object[i++],args) === false ){
							break;
						}
					}
				}
			}else{
				if(isObj){
					for(name in object){
						if(callback.apply(object[name],name,object[name]) === false){
							break;
						}
					}
				}else{
					for(;i < length){
						if(callback.apply(object[i],i,object[i++]) === false ){
							break;
						}
					}
				}
			}
			return object;
		}
	延迟初始化
		它能够延迟昂贵的过程，直到它的第一个实例需要时
		jQuery.ready()
		$(document).ready(function(){

		})
		jQuery.fn.ready()是由jQuery.bindReady()驱动
		bindReady:function(){
			if(readyList){
				return;
			}
			readyList = jQuery.Callbacks("once memory")
			//浏览器事件发生后，这里调用$(document).ready()
			if(document.readyState === "complete"){
				//异步处理允许脚本延迟执行
				return setTimeout(jQuery.ready,1)
			}
			//Moilla,Opera和webkit支持该事件
			if(document.addEventListener){
				//使用事件处理回调
				document.addEventListener("DOMContentLoaded",DOMContentLoaded,false);
				//window.onload后的事件，永远可用
				window.addEventListener("load",jQuery.ready,false)
			}else if(document.attachEvent){//IE
				//确保onload之前触发事件，可能晚点，但安全，对iframe也是
				doucument.attachEvent("onreadystatechange",DOMContentLoaded);
				//window.onload后的事件，永远可用
				window.attachEvent("onload",jQuery.ready)
				//如果是IE，并且不是frame，继续检查文档是否就绪
				var toplevel = false;
				try{
					toplevel = window.frameElement == null;
				}catch(e){

				}
				if(document.documentElement.doScroll && toplevel){
					doScrollCheck();
				} 
			}
		}
	Proxy(代理)模式
		控制对象后面的访问权限和上下文
		jQuery.proxy()
		$("button".on("click",function(){
			setTimeout($.proxy(function(){
				//这里的this即引用我们所想的元素上了
				$(this).addClass("active")
			},this),500)
		})
		proxy的实现
		//绑定函数到上下文（context）上，参数可选
		proxy:function(fn,context){
			if(typeof context === "string"){
				var tmp = fn[context];
				context = fn;
				fn = tmp;
			}
			//快速判断fn是否可以调用，如果不可以，返回undefined
			if(!jQuery.isFunction(fn)){
				return undefined;
			}
			//模拟绑定
			var args = slice.call(arguments,2),
				proxy = function(){
					return fn.apply(context,args.concat(slice.call(arguments)))
				}
			//为原始的处理程序设置一个唯一的guid
			proxy.guid = fn.guid = fn.guid || proxy.guid || jQuery.guid++;
			return proxy;
		}
	Builder(生成器)模式
		拥有用于创建复杂的DOM对象并独立于对象本身的机制能够提供这种灵活性，而这正是Builder模式所提供的。
		$("<div class = 'foo'>bar</div>")
		$("<P/>").text("hello")
		appendTo
		//处理$(html) -> $(array)
		if(match[1]){
			context = context instanceof jQuery ? context[0] : context;
			doc = (context?context.ownerDocument || context : document);
			//如果仅仅传入一个单独的字符串并且是单独的标签，则创建元素并且忽略剩余部分
			ret = rsingleTag.exec(selector)
			if(ret){
				if(jQuery.isPlainObject(context)){
					selector = [document.createElement(ret[1])];
					jQuery.fn.attr.call(selector,context,true)
				}else{
					selector = [doc.createElement(ret[1])]
				}
			} else{
				ret = jQuery.buildFragment([match[1]],[doc]);
				selector = (ret.cacheable ? jQuery.clone(ret.fragment):ret.fragment).childNodes
			}
			return jQuery.merge(this,selector)
		}
jQuery插件设计模式
	模式
		(function($){
			$.fn.myPluginName = function(){
				//插件逻辑
			}
		})
	Lightweight Start 模式
		1.常见最佳实践，例如，放在函数调用之前的分号
		2.window、document和undefined作为参数传入
		3.基本的默认对象
		4.简单的插件构造函数，用于与初始化创建相关的逻辑，以及用于所使用元素的赋值
		5.扩展有默认值的选项
		5.构造函数周围的lightweight包装器，帮助避免多实例等问题
		6.遵守jQuery核心格式指南，以实现最优可读性

		示例：
			/*!
			 * jQuery lightweight plugin bolerplate
			 * Original author : @ajpiano
			 * Further changes, comments: @addyosmani
			 * Licensed under the MIT license
			*/

			//函数调用之前的分号是为了安全目的，防止前面的其他插件没有正常关闭
			;(function($,window,document,undefined){
				//这里使用的undefined是ECMAScript 3里的全局变量undefined,是可以修改的。
				//undefined没有真正传进来，以便可以确保改值是真正的undefined。
				//ES5里，undefined是不可修改的
				//window和document传递进来座位局部变量存在，而非全局变量，因为这可以
				//加快解析流程以及影响最小化（尤其是同时引用一个插件的时候）

				//创建默认值
				var pluginName = "defaultPluginName",
					defaults = {
						propertyName:"value"
					};
				//真正的插件构造函数
				function Plugin(element,options){
					this.element = element;
					//jQuery有个extend方法用于将两人或多个对象合并在一起，在第一个对象里进行排序
					//第一个对象通常为空，因为我们不想为插件的新实例影响默认的opinion选项
					this.options = $.extend({},defaults,options);
					this._defaults = defaults;
					this._name = pluginName;
					this.init();
				}
				Plugin.prototype.init = function(){
					//这里处理初始化逻辑
					//已经可以访问DOM，并且通过实例访问options，例如this.element,this.options
				};
				//真正的插件包装，防止出现多实例
				$.fn[pluginName] = function(options){
					return this.each(function(){
						if(!$.data(this,"plugin_"+pluginName)){
							$.data(this,"plugin_"+pluginName,new Plugin(this,options))
						}
					})
				}
			})(jQuery,window,document);
			//用法
			$("#elem").defaultPluginName({
				propertyName:"a custom value"
			})
	完整的widget factory模式
		示例：
			/*!
			 * jQuery UI Widget-factory plugin bolerplate(for 1.8/9+)
			 * Author : @addyosmani
			 * Further changes: @peolanha
			 * Licensed under the MIT license
			*/
			;(function($,window,document,undefined){
				//在指定的命名空间下创建widget,命名空间可作为额外的参数传递进来
				//$.widget("namespace.widgetname",(optional))-已经存在的可被继承的widget原型，是一个对象字面量
				$.widget("namespace.widgetname",{
					//可以作为默认值的options
					options:{
						someValue:null
					},
					//创建widget(例如创建元素，应用主题，绑定时间等)
					_create: function(){
						//_create在第一次调用widget的时候创建，在此防止widget初始化代码
						//然后通过this.element可以访问调用该widget的元素
						//可以通过this.options this.element.addstuff访问上面定义的options
					},
					//销毁插件实例，清楚widget在DOM上的修改
					destory: function(){
						//this.element.removeStuff();
						//对UI 1.8，destory必须调用基类的widget
						$.Widget.prototype.destory.call(this)
						//对UI 1.9，定义_destory代替，不必担心调用基类的widget
					},
					methodB:function(event){
						//_trigger 派发插件用户可以订阅的回调callback
						//用法：_trigger("callbackName",[eventObject],[uiObject])
						//例如this._trigger("hover",e/*where e.type == "mouseenter"*/,{hoverd:$(e.target)})
						this._trigger("methodA",event,{
							key:value
						})
					},
					methodA:function(event){
						this._trigger("dataChanged",event,{
							key:value
						})
					},
					//响应用户通过option方法修改的任何值
					_setOption:function(key,value){
						switch(key){
							case "someValue":
								//this.options.someValue = doSomethingWith(value)
								break;
							default:
								//this.option[key] = value
								break;
						}
						//对UI 1.8,必须手工调用基类widget中的_setOption
						$.Widget.prototype._setOption.apply(this,arguments)
						//对UI 1.9,可以使用_super代替
						//this._super("_setOption",key,value)
					}
				})
			})(jQuery,window,document);
			//用法
			var collection = $("#elem").widgetName({
				foo:false
			})
			collection.widgetName("methodB")
	嵌套命名空间插件模式
		示例：
			/*!
			 * jQuery namespaced "Starter" plugin bolerplate(for 1.8/9+)
			 * Author : @dougneiner
			 * Further changes: @addyosmani
			 * Licensed under the MIT license
			*/
			;(function($){
				if(!$.myNamespace){
					$.myNamespace = {}
				}
				$.myNamespace.myPluginName = function(el,myFunctionParam,options){
					//避免作用域问题，使用base代替this来引用内部事件和函数中的this
					var base = this;
					//访问jQuery和元素的DOM版本
					base.$el = $(el);
					base.el = le;
					//为DOM对象添加一个反向引用
					base.$el.data("myNamespace.myPluginName",base);
					base.init = function (){
						base.myFunctionParam = myFunctionParam;
						base.options = $.extend({},$.myNamespace.myPluginName.defaultOptions,options);
						//放置初始化代码
					};
					//示例函数，取消注释即可使用
					//base.functionName =function (	parameters ){
					//
					//}
					//运行初始化函数
					base.init();
				}
				$.myNamespace.myPluginName.defaultOptions = {
					myDefalutValue:""
				}
				$.fn.mynamespace_myPluginName = function(myFunctionParam,options){
					return this.each(function({
						(new $.myNamespace.myPluginName(this,myFunctionParam,options))
					}))
				}
			})(jQuery)
			//用法
			$("#elem").mynamespace_myPluginName({
				myDefalutValue:"foobar"
			})
	自定义事件插件模式（使用Widget Factory）
		示例：
			/*!
			 * jQuery custom-events plugin bolerplate(for 1.8/9+)
			 * Author : @Devpatch
			 * Further changes: @addyosmani
			 * Licensed under the MIT license
			*/

			//本模式，使用jQuery自定义事件添加pub/sub能力到widget上
			//每个widget都能发布特定的事件并且可以订阅其它事件。这种方式显著解耦了widget，使得它们的函数独立
			;(function($,window,document,undefined){
				$.widget("ao.eventStatus",{
					options:{

					},
					_create:function(){
						var self = this;
						//self.element.addClass("my-widget");
						//订阅到"myEventStart"
						self.element.on("myEventStart",function(e){
							console.log("event start")
						})
						//订阅到"myEventEnd"
						self.element.on("myEventEnd",function(e){
							console.log("event end")	
						})
						//取消订阅"myEventStart"
						// self.element.off("myEventStart",function(e){
						// 	console.log("unsubscribe to this event")
						// })
					},
					destory: function(){
						$.Widget.prototype.destory.apply(this,arguments)
					}
				})	
			})(jQuery,window,document);
			//发布事件通知
			// $(".my-widget").trigger("myEventStart")
			// $(".my-widget").trigger("myEventEnd")
			//用法
			var el = $("#elem")
			el.eventStatus();
			el.eventStatus().trigger("myEventStart")
	使用DOM-to-Object Bridge模式的原型继承
		示例：
			/*!
			 * jQuery prototypal inheritance plugin bolerplate(for 1.8/9+)
			 * Author : Alex Sexton,Scott Gonzalez
			 * Further changes: @addyosmani
			 * Licensed under the MIT license
			*/
			//myObject 一个描述我们想模拟的概念,例如一辆汽车car
			var myObject = {
				init: function(options,elem){
					//将默认options与传递进来的options混入在一起
					this.options = $.extend({},this.options,options)//经典写法
					//保存元素引用，jQuery引用和正常引用都保存
					this.elem = elem;
					this.$elem = $(elem)
					//构建DOM的初始化
					this._bulid();
					//返回this,以便可以链式使用
					return this;
				},
				options:{
					name:"No name"
				},
				_build: function(){
					//this.$elem.html("<h1>"+this.options.name+"</h1>")
				},
				myMethod:function(msg){
					//可以直接访问相关的和缓存的jQuery元素
					//this.$elem.append("<p>"+msg+"</p>")
				}
			}
			//测试Object.create,为不支持的浏览器提供create支持
			if(typeof Object.create !== "function"){
				Object.create = function(o){
					function F(){};
					F.prototype = o;
					return new F();
				}
			}
			//基于定义的对象上创建插件
			$.plugin = function(name,object){
				$.fn[name] = function(options){
					return this.each(function(){
						if(!$.data(this,name)){
							$.data(this,name,Object.create(object).init(options,this))
						}
					})
				}
			}

			//用法
			$.plugin("myobj",myObject)
			$("#elem").myobj({name:"John"})
			var collection = $("#elem").data("myobj")
			collection.myMethod("I am a method")
	jQuery UI Widget Factory Bridge 模式
		$.widget.bridge
			1.像在经典OOP中一样，进行公有和私有方法的处理，与预期一致（即，暴露了公有方法，而隐藏了私有方法）
			2.对多个初始化的自动保护
			3.已传递对象实例的自动生成，及它们所选择元素内部$.data缓存中的存储
			4.初始化后可以更改选项
		示例：
			/*!
			 * jQuery UI Widget Factory Bridge plugin bolerplate
			 * Author : @erichynds
			 * Further changes,additional comments:@addyosmani
			 * Licensed under the MIT license
			*/

			//"widgetName"对象构造器 
			//required:必须接收两个参数
			//options:配置选择参数
			//element：在之上创建实例的DOM元素
			var widgetName = function(options,element){
				this.name = "myWidgetName";
				this.options = options;
				this.element = element;
				this._init();
			}
			//"widgetName"原型 
			widgetName.prototype = {
				//widget第一次调用的时候，_create会自动运行
				_create:function(){
					//创建代码
				},
				//required:插件的初始化逻辑代码放在_init中，第一次创建实例或初始化之后再次
				//尝试初始化widget（通过桥接）时，会触发_init
				_init:function(){
					//初始化代码
				},
				//required:使用桥接的对象必须包含一个option，深度初始化，改变options的逻辑放在这里
				option:function(key,value){
					//可选：如果不需要则可以忽略获取
					//用法：$("#foo").bar({cool:false})
					if($.isPlainObject(key)){
						this.options = $.extend(true,this.options,key)
					}else if(key && typeof value === "undefined"){//用法：$("#foo").option("cool"); --getter
						return this.options[key];
					}else{
						//用法：$("#foo").bar("option","baz",false)
						this.options[key] = value;
					}
					//required:option必须返回当前实例，在元素上重写初始化实例时，首先调用option，然后链式到_init方法
					return this;
				},
				//注意公有方法没有使用下划线
				publicFunction:function(){
					console.log("public function")
				}
				//私有方法使用了下划线
				_privateFunction:function(){
					console.log("private function")
				}
			}
			//用法
			//在foo命名空间下，连接widget对象到jQuery的API
			$.widget.bridge("foo",widgetName);
			//创建widget的实例
			var instance = $("#foo").foo({
				bar:true
			})
			//elenm的数据上已经存在widget实例
			//输出：#elem
			console.log(instance.data("foo").element)
			//桥接允许我们调用公有方法
			//输出："public method"
			instance.foo("publicFunction");
			//桥接阻止调用内部方法
			instance.foo("_privateFunction")
	使用Widget Factory的jQuery Mobile Widget

	RequireJS和jQuery UI Widget Factory

	全局选项和单次调用可重写选项（最佳选项模式）

	高可配和高可变的插件模式

	是什么使插件超越模式
		1.质量 
			遵守你所编写的JavaScript和jQuery的最佳实践
		2.代码风格
			插件是否遵守一贯的代码风格指南，例如jQuery核心风格指南
		3.兼容性
			用新版本来测试插件，保证预期工作
		4.可靠性
			单元测试
		5.性能 
			优化性能 
		6.文档 
			提供良好的文档记录
		7.维护的可能性
	命名空间模式

	命名空间基础
		单一全局变量
			var myApplication = (function(){

			}())
		前缀命名空间
			var myApplication_propertyA = {};
		对象字面量表示法
			var myApplication = {
				getInfo:function(){

				},
				models:{

				},
				views:{

				}
			}
			myApplication.foo = function（）{
				return "bar"
			}
			myApplication.utils = {
				toString:function(){

				}
			}
		嵌套命名空间插件模式
			var myApp = myApp || {};
			//定义嵌套子对象时，检测是否存在
			myApp.routers = myApp.routers || {}
			myApp.model = myApp.model || {}
			myApp.model.special = myApp.model.special || {}
		立即调用的函数表达式
			(function(){})()
			(function foobar(){})()
			function foobar(){foobar()}
		命名空间注入
			var myApp = myApp || {}
			myApp.utils = {}
			(function(){
				var val = 5;
				this.getValue = function(){
					return val
				}
				this.setValue = function(newVal){
					val = newVal;
				}
				//也定义一个新的子命名空间
				this.tools = {};
			}).apply(myApp.utils)
			//为utils命名空间注册新的行为
			//可以通过utilities模块定义
			(function(){
				this.diagnose = function(){
					return "diagnosis"
				}
			}).apply(myApp.utils.tools)
			//用法
			//输出命名空间
			console.log(myApp)
			//输出
			console.log(myApp.utils.getValue())
			myApp.utils.setValue(25)
			//测试其他层次功能
			console.log(myApp.utils.tools.diagnose())
	高级命名空间模式
		自动嵌套的命名空间
			var application = {
				utilties：{
					drawing:{
						canvas:{

						}
					}
				}
			}

			//通用函数，解析命名空间字符串，并自动生成嵌套的命名空间
			function extend(ns,ns_string){
				var parts = ns_string.split("."),
					parent = ns,
					pl;
				pl = parts.length;
				for(var i =0; i< pl;i++){
					//属性如果不存在，则创建它
					if(typeof parent[parts[i]] === "undefined"){
						parent[parts[i]] = {}
					}
					parent = parent[parts[i]];
				}
				return parent;
			}
			//用法
			var mod = extend(myApp,"module.module2")
			extend(myApp,"moduleA.moduleB.moduleC")
		依赖声明模式
			//访问嵌套命名统一方式
			maApp.utilties.math.fibonacci(25)
			//局部引用
			var utils = maApp.utilties,
			maths = utils.math;
			//简便访问
			maths.fibonacci(25)
		深度对象扩展
			//实现$.extend
			//extend.js
			//Andrew Dupont,编写，Addy Osmani优化
			function extend(destination,source){
				var toString = Object.prototype.toString,
					objTest = toString.call({});
				for(var property in source){
					if(source[property] && objTest === toString.call(source[property])){
						destination[property] = destination[property]  || {};
						extend(destination[property],source[property]);
					}else{
						destination[property] = source[property]
					}
				}
				return destination;
			}
			//用法
				//定义一个顶级命名空间
				var myNs = myNs || {};
				//1.使用utils对象扩展命名空间
				extend(myNs,{
					utils:{

					}
				})
				//2.使用多次路径进行扩展（namespace.hello.world.wave）
				extend(myNs,{
					hello:{
						world:{
							wave:{
									test:function(){

								}
							}
						}
					}
				})
				//3.如果添加的命名空间已存在，确保不被重写
				myNs.library = {
					foo:function(){}
				}
				extend(myNs,{
					library:{
						bar:function(){

						}
					}
				})
				//myNs也包含library.foo，library.bar

				//4.如果不想输入整个命名空间，而只想部分
				var shorterAccess1 = myNs.hello.world;
				shorterAccess1.test3 = "hello again";
				console.log("test 4",myNS);




css
	盒子模型
	继承和层叠
	BFC(http://www.cnblogs.com/lhb25/p/inside-block-formatting-ontext.html、http://www.cnblogs.com/dojo-lzz/p/3999013.html) 
		BFC 定义
　　        BFC布局规则：BFC(Block formatting context)直译为"块级格式化上下文"。
			它是一个独立的渲染区域，只有Block-level box参与， 它规定了内部的Block-level Box如何布局，并且与这个区域外部毫不相干。

		BFC布局规则：
			内部的Box会在垂直方向，一个接一个地放置。
			Box垂直方向的距离由margin决定。属于同一个BFC的两个相邻Box的margin会发生重叠
			每个元素的margin box的左边， 与包含块border box的左边相接触(对于从左往右的格式化，否则相反)。即使存在浮动也是如此。
			BFC的区域不会与float box重叠。
			BFC就是页面上的一个隔离的独立容器，容器里面的子元素不会影响到外面的元素。反之也如此。
			计算BFC的高度时，浮动元素也参与计算
		哪些元素会生成BFC?
			根元素
			float属性不为none
			position为absolute或fixed
			display为inline-block, table-cell, table-caption, flex, inline-flex
			overflow不为visible
	box-sizing
		box-sizing属性可以为三个值之一：content-box（default），border-box，padding-box。
		content-box，border和padding不计算入width之内
		padding-box，padding计算入width内
		border-box，border和padding计算入width之内，其实就是怪异模式了~
	box-flex
		例如：
			#p1
			{
				-moz-box-flex:1.0; /* Firefox */
				-webkit-box-flex:1.0; /* Safari 和 Chrome */
				box-flex:1.0;
				border:1px solid red;
			}

			#p2
			{
				-moz-box-flex:2.0; /* Firefox */
				-webkit-box-flex:2.0; /* Safari 和 Chrome */
				box-flex:2.0;
				border:1px solid blue;
			}
	转换
		transform
			translate()
			rotate()
			scale()
			skew()
			matrix()
		例如：
			div
			{
				transform: rotateX(120deg);
				-webkit-transform: rotateX(120deg);	/* Safari 和 Chrome */
				-moz-transform: rotateX(120deg);	/* Firefox */
			}
	过渡
		transition
		例如：
			div
			{
				transition: width 2s;
				-moz-transition: width 2s;	/* Firefox 4 */
				-webkit-transition: width 2s;	/* Safari 和 Chrome */
				-o-transition: width 2s;	/* Opera */
			}
	动画
		@keyframes
		animation
		例如：
			div
			{
				animation: myfirst 5s;
				-moz-animation: myfirst 5s;	/* Firefox */
				-webkit-animation: myfirst 5s;	/* Safari 和 Chrome */
				-o-animation: myfirst 5s;	/* Opera */
			}
			@keyframes myfirst
			{
				from {background: red;}
				to {background: yellow;}
			}

			@-moz-keyframes myfirst /* Firefox */
			{
				from {background: red;}
				to {background: yellow;}
			}

			@-webkit-keyframes myfirst /* Safari 和 Chrome */
			{
				from {background: red;}
				to {background: yellow;}
			}

			@-o-keyframes myfirst /* Opera */
			{
				from {background: red;}
				to {background: yellow;}
			}


理解HTTP协议
http://www.blogjava.net/zjusuyong/articles/304788.html


javascript包括 ECMAscript DOM BOM
defer和async区别就是async是异步，不按顺序
无文档声明就是混杂模式，加声明是标准（严格）模式，混杂模式不同浏览器差异很大，标准模式是按标准执行
变量、函数、操作符区分大小写
undefined Null Boolean Number String
typeof null === "object" typeof [] === "object" typeof undefined === "undefined" null == undefined
无论你叫它什么，在JavaScript中创建一个空对象最简单的方法都是Object.create(null)（详细介
绍请看第5章）。Object.create(null)和{}很像，但是并不会创建Object.prototype这个委托，所以它
比{}“更空”
parseInt("1234blue")//1234
0.1+0.2 != 0.3
toString() 转进制 var  a = 10; a.toString(16)//"a"
hasOwnProperty()给定的属性在当前对象实例中是否存在
基本类型按值访问 引用类型按引用访问
所有函数的参数都是按值传递的
typeof检测基本类型 instanceof检测是不是某种类型对象
person instanceof Object //person是Object的吗？
垃圾收集：1.标记清除 2.引用计数
检测数组
if(typeof Array.isArray === "undefined"){
	Array.isArray = function (arg){
		return Object.prototype.toString.call(arg) === "[object Array]";
	}
}
Array 对象方法
是否改变现有数组	方法	描述
否 	concat()	连接两个或更多的数组，并返回结果。  
否  join()	把数组的所有元素放入一个字符串。元素通过指定的分隔符进行分隔。
是  pop()	删除并返回数组的最后一个元素
是	push()	向数组的末尾添加一个或更多元素，并返回新的长度。
是	reverse()	颠倒数组中元素的顺序。
是	shift()	删除并返回数组的第一个元素
否	slice()	从某个已有的数组返回选定的元素
是	sort()	对数组的元素进行排序
是	splice()	删除元素，并向数组添加新元素。
	toSource()	返回该对象的源代码。
	toString()	把数组转换为字符串，并返回结果。
	toLocaleString()	把数组转换为本地数组，并返回结果。 
是	unshift()	向数组的开头添加一个或更多元素，并返回新的长度。
	valueOf()	返回数组对象的原始值

ES5迭代方法
every() 对数组中的每一项运行给定函数，如果该函数对每一项都返回true，则返回true
filter() 对数组中的每一项运行给定函数，返回该函数会返回true的项组成的数组
forEach() 对数组中的每一项运行给定函数。这个方法没有返回值
map() 对数组中的每一项运行给定函数，返回每次函数调用的结果组成的数组
some() 对数组中的每一项运行给定函数，如果该函数对任一项返回true，则返回true

var numbers = [1,2,3,4,5]
var result = numbers.every(function(item,index,array){
	return item > 2;
})
//false
var result = numbers.filter(function(item,index,array){
	return item > 2;
})
//[3,4,5]
numbers.forEach(function(item,index,array){
	//dosomething
})
var result = numbers.filter(function(item,index,array){
	return item * 2;
})
//[2,4,5,8,10]
var result = numbers.some(function(item,index,array){
	return item > 2;
})
//true

ES5数组归并方法 reduce 和 reduceRight
numbers.reduce(function(prev,cur,index,array){
	return prev + cur;
})
//15
//应用
有一个长度为100的数组，请以优雅的方式求出该数组的前10个元素之和
分析：其实，对于数组求和有很多种方法，也很简单。但是，这题有两个限制条件：优雅的方式、前10个元素。对于“前10个元素”这个限制条件可以使用Array.prototype.slice()方法来截取，对于"优雅的方式"，我的理解是应该尽可能使用数组自带的方法，最好可以使用高阶函数！所以我觉得应该是Array.prototype.reduce()方法。代码如下：

var a = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15],
sum = 0;

sum = a.slice(0, 10).reduce(function(pre, current) {
　　return pre + current;
});

console.log(sum); //55

实现对数组进行乱序
var a = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10],
    sign = 1; 
a.sort(function(a, b) {
    //因为Math.random产生的数在0-1之间
    //所以0.5两边的概率是相等的
    //大于0.5时为升序，小于0.5时为降序
    sign = (Math.random() > 0.5) ? 1 : -1;
    return (a - b) * sign;

});

Date
ES5 新增了Date.now()可以获取当前时间的毫秒数
常用方法 
getFullYear() 取得4位数年份
getMonth() 返回月份 从0开始
getDate() 返回日期月份中的天数（1到31）
getDay() 返回星期几（0是周日，6是周六）
getHours() 小时（0-23）
getMinutes() 分钟（0-59）
getSeconds() 秒（0-59）

正则
https://github.com/wenkangzhou/RegularExpression

函数声明提升
对的
alert(sum(1,1))
function sum (a,b){}{
	return a + b;
}
错的
alert(sum(1,1))
var sum = function (a,b){}{
	return a + b;
}
变量声明提升
a = 2;
var a;
console.log( a );
你认为console.log(..)声明会输出什么呢？
很多开发者会认为是undefined，因为var a声明在a = 2之后，他们自然而然地认为变量被重新赋
值了，因此会被赋予默认值undefined。但是，真正的输出结果是2。
考虑另外一段代码：
console.log( a );
var a = 2;
鉴于上一个代码片段所表现出来的某种非自上而下的行为特点，你可能会认为这个代码片段也会
有同样的行为而输出2。还有人可能会认为，由于变量a在使用前没有先进行声明，因此会抛
出ReferenceError异常。
不幸的是两种猜测都是不对的。输出来的会是undefined。
那么到底发生了什么？看起来我们面对的是一个先有鸡还是先有蛋的问题。到底是声明（蛋）在
前，还是赋值（鸡）在前？

为了搞明白这个问题，我们需要回顾一下第1章中关于编译器的内容。回忆一下，引擎会在解释
JavaScript代码之前首先对其进行编译。编译阶段中的一部分工作就是找到所有的声明，并用合适
的作用域将它们关联起来。第2章中展示了这个机制，也正是词法作用域的核心内容。
因此，正确的思考思路是，包括变量和函数在内的所有声明都会在任何代码被执行前首先被处理。
当你看到var a = 2;时，可能会认为这是一个声明。但JavaScript实际上会将其看成两个声
明：var a;和a = 2;。第一个定义声明是在编译阶段进行的。第二个赋值声明会被留在原地等待执
行阶段。
我们的第一个代码片段会以如下形式进行处理：
var a;
a = 2;
console.log( a );
其中第一部分是编译，而第二部分是执行。
类似地，我们的第二个代码片段实际是按照以下流程处理的：
var a;
console.log( a );
a = 2;
函数优先
函数声明和变量声明都会被提升。但是一个值得注意的细节（这个细节可以出现在有多个“重复”声
明的代码中）是函数会首先被提升，然后才是变量。
考虑以下代码：
foo(); // 1
var foo;
function foo() {
console.log( 1 );
}
foo = function() {
console.log( 2 );
};
会输出1而不是2！这个代码片段会被引擎理解为如下形式：
function foo() {
console.log( 1 );
}
foo(); // 1
foo = function() {
console.log( 2 );
};

call(this,a,b,c)
apply(this,arguments)

引用类型和基本包装类型的区别就是对象的生存期，引用类型一直保存在作用域，包装类型只存在一行代码的执行瞬间，立即销毁

String 对象方法
方法	描述
anchor()	创建 HTML 锚。
big()	用大号字体显示字符串。
blink()	显示闪动字符串。
bold()	使用粗体显示字符串。
charAt()	返回在指定位置的字符。
charCodeAt()	返回在指定的位置的字符的 Unicode 编码。
concat()	连接字符串。
fixed()	以打字机文本显示字符串。
fontcolor()	使用指定的颜色来显示字符串。
fontsize()	使用指定的尺寸来显示字符串。
fromCharCode()	从字符编码创建一个字符串。
indexOf()	检索字符串。
italics()	使用斜体显示字符串。
lastIndexOf()	从后向前搜索字符串。
link()	将字符串显示为链接。
localeCompare()	用本地特定的顺序来比较两个字符串。
match()	找到一个或多个正则表达式的匹配。
replace()	替换与正则表达式匹配的子串。
search()	检索与正则表达式相匹配的值。
slice()	提取字符串的片断，并在新的字符串中返回被提取的部分。
small()	使用小字号来显示字符串。
split()	把字符串分割为字符串数组。
strike()	使用删除线来显示字符串。
sub()	把字符串显示为下标。
substr()	从起始索引号提取字符串中指定数目的字符。
substring()	提取字符串中两个指定的索引号之间的字符。
sup()	把字符串显示为上标。
toLocaleLowerCase()	把字符串转换为小写。
toLocaleUpperCase()	把字符串转换为大写。
toLowerCase()	把字符串转换为小写。
toUpperCase()	把字符串转换为大写。
toSource()	代表对象的源代码。
toString()	返回字符串。
valueOf()	返回某个字符串对象的原始值。

var a = function (){};
a.prototype.name = 'b'
var c = new a();
c.__proto__ == a.prototype == Object.getPrototypeOf(c)//true
Object.keys(a.prototype)//name
Object.getOwnPropertyNames(a.prototype)//constructor,name

window location navigator screen history
location.assign("http://baudu.com") === location.href = "http://baudu.com" === window.location = "http://baudu.com"

.nodeType .nodeName
节点操作
appendChild()
insertBefore()
replaceChild()
removeChild()
cloneNode()//true、false深、浅复制
节点关系
childNodes
firstChild
lastChild
parentNode
nextSibling
previousSibling

Node类型(nodeType)
Node.ELEMENT_NODE 1
	HTML
		HTMLElement:
			id
			title
			lang 元素语言代码
			dir
			className
		var div = document.getElementById("div")
		alert(div.id)
		alert(div.className)
		....
	取得特性
		var div = document.getElementById("div")
		document.getAttribute("id")
		document.getAttribute("class")
		document.getAttribute("title")
	设置特性
		document.setAttribute("id",'111')
	删除特性
		document.removeAttribute("id")
	attributes
		Element类型是使用attributes属性唯一一个DOM节点类型
		attributes属性包含一个NamedNodeMap，与NodeList类似
		元素每个特性都由一个Attr节点表示
		每个节点保存在NamedNodeMap，NameNodeMap：
			getNamedItem(name) 返回nodeName属性等于name的节点
			removeNamedItem(name) 列表中移除nodeName属性等于name
			setNamedItem(node) 列表添加节点，节点的nodeName为索引
			item(pos) 返回数字pos处的节点
		var value = element.attributes.getNamedItem("id").nodeValue
		相当于 element.getAttribute("id")
		var name = element.attributes.getNamedItem("id").nodeName
	创建元素 
		document.createElement
Node.ATTRIBUTE_NODE 2
	Attr元素特性，对应三个属性
		name 特性名称
		value 特性值
		specified 区别特性是在代码中指定的
	var attr = document.createAttribute("align")
	attr.value = "left"
	element.setAttributeNode(attr)
	alert(elemnt.attributes["align"].value)//left
	alert(elemnt.getAttributeNode("align").value)//left
	alert(elemnt.getAttribute("align"))//left
Node.TEXT_NODE 3
	document.createTextNode
	normalize()//合并相邻文本节点
	splitText()//将一个文本节点拆分成两个文本节点
Node.CDATA_SECTION_NODE 4
Node.ENTITY_REFERENCE_NODE 5
Node.ENTITY_NODE 6
Node.PROCESSING_INSTRUCTION_NODE 7
Node.COMMENT_NODE 8
Node.DOCUMENT_NODE 9
	document是HTMLDocument（继承自Document类型）的实例
	document.documentElement//取得对<html>的引用
	document.body//取得对<body>的引用
	查找元素
		getElementById
		getElementsByTagName
			NodeList
			document.getElementByTagName(),返回HTMLCollection对象
			<img src="xx.jpg" name="myImg">
			var images = document.getElementByTagName("img")
			images[0].src images.item(0).src
			var myImg = images.namedItem("myImg")
			images["myImg"]
Node.DOCUMENT_TYPE_NODE 10
	<DOCTYPE　HTML>
Node.DOCUMENT_FRAGMENT_NODE 11
	document.createDocumentFragment
Node.NOTATION_NODE 12

NodeList转为数组
var arratNodes = Array.prototype.slice.call(someNode.childNodes,0)



element
getAttribute()
setAttribute()
attributes
	NamedNodeMap
		getNamedItem
		removeNamedItem
		setNamedItem
		item
var id = element.attributes.getNamedItem(id).nodeValue

document.createElement()
document.createTextNode()
normalize()//合并相邻文本节点
splitText()//将一个文本节点拆分成两个文本节点

Comment//用来表示注释

CDATASection

DocumentFragment
//动态加载外部JavaScript
function loadScript(url){
	var script = document.createElement("script");
	script.type = "text/javascript";
	script.src = url;
	document.body.appendChild(script);
}
//动态加载外部css
function loadStyles(url){
	var link = document.createElement("link");
	link.rel = "stylesheet";
	link.type = "text/css";
	link.href = url;
	var head = document.getElementByTagName("head")[0];
	head.appendChild(link)
}

NodeList NamedNodeMap HTMLCollection

querySelector()
querySelectorAll()

document.readyState //loading正在加载 complete加载完毕
document.compatMode

//<div id="xx" data-name="hi"></div>
document.getElementById("xx").dataset.name === $("#xx").data("name")

事件捕获 html->body->div
事件冒泡 div->body->html

addEventListener removeEventListener
attachEvent detachEvent

DOMContentLoaded
readystatechange
	uninitialized(未初始化)
	loading(正在加载)
	loaded(加载完毕)
	interactive(交互)
	complete(完成)
pageshow//用于往返缓存
pagehide//用于往返缓存
hashchange

设备事件
	orientationchange
	MozOrientation
	deviceorientation
	devicemotion

	touchstart
	touchmove
	touchend
	touchcancel

	gesturestart
	gesturechange
	gestureend
事件委托
	利用事件冒泡，只指定一个事件处理程序，就可以管理某一类型的所有事件。 
	例如，click事件会一直冒泡到document层次。也就是说，我们可以为整个页面指定
	一个onclick事件处理程序，而不必给每个可单击的元素分别添加事件处理程序

	“空事件处理程序” btn.onclick = null

$0.contentEditable = "true"

Canvas是使用JavaScript程序绘图(动态生成)，SVG是使用XML文档描述来绘图。
从这点来看：SVG更适合用来做动态交互，而且SVG绘图很容易编辑，只需要增加或移除相应的元素就可以了。
同时SVG是基于矢量的，所有它能够很好的处理图形大小的改变。
Canvas是基于位图的图像，它不能够改变大小，只能缩放显示；
所以说Canvas更适合用来实现类似于Flash能做的事情(当然现在Canvas与Flash相比还有一些不够完善的地方)。
WebGL是针对Canvas的3D上下文，可以为HTML5 Canvas提供硬件3D加速渲染
window.postMessage()//跨文档消息传送
dragstart drag dragend
audio 接口 P487 事件 P488
history.pushState
history.popState
history,replaceState
//解决setTimeout传参的问题
setTimeout(function(){self.countDown($dom,self);}, 1000);
window.onerror = function(message,url,line){
	alert(message);
	return false;
}
JSON.stringify();//转为JSON字符串
JSON.parse();//转为对象字面量
toJSON() 
xhr.status
	0：未初始化。尚未调用open()方法
	1：启动。已经调用open()方法，但尚未调用send()方法
	2：发送。已经调用send()方法，但尚未接收到响应
	3：接收。已经接收到部分响应数据
	4：完成。已经接收到全部响应数据，而且已经可以再客户端使用了
HTTP头部信息
	Accept：浏览器能够处理的内容类型
	Accept-Charset：浏览器能够显示的字符集
	Accept-Encoding：浏览器能够处理的压缩编码
	Accept-language：浏览器当前设置的语言
	Connection：浏览器与服务器之间连接的类型
	Cookie：当前页面设置的任何Cookie
	Host：发出请求的页面所在的域
	Referer：发出请求的页面的URI
	User-Agent：浏览器的用户代码字符串
时间进度
	loadstart
	progress
	error 
	abort
	load
	loadend
IE实现CORS XDR(XDomainRequest)

you donnot konw JavaScript 

var anotherObject = {
a:2
};
var myObject = Object.create( anotherObject );
anotherObject.a; // 2
myObject.a; // 2
anotherObject.hasOwnProperty( "a" ); // true
myObject.hasOwnProperty( "a" ); // false
myObject.a++; // 隐式屏蔽！
anotherObject.a; // 2
myObject.a; // 3
myObject.hasOwnProperty( "a" ); // true
尽管myObject.a++看起来应该（通过委托）查找并增加anotherObject.a属性，但是别忘了++操作相
当于myObject.a = myObject.a + 1。因此++操作首先会通过[[Prototype]]查找属性a并
从anotherObject.a获取当前属性值2，然后给这个值加1，接着用[[Put]]将值3赋给myObject中新建
的屏蔽属性a，天呐！
修改委托属性时一定要小心。如果想让anotherObject.a的值增加，唯一的办法
是anotherObject.a++。

委托控件对象
下面的例子使用对象关联风格委托来更简单地实现Widget/Button：
var Widget = {
init: function(width,height){
this.width = width || 50;
this.height = height || 50;
this.$elem = null;
},
insert: function($where){
if (this.$elem) {
this.$elem.css( {
width: this.width + "px",
height: this.height + "px"
} ).appendTo( $where );
}
}
};
var Button = Object.create( Widget );
Button.setup = function(width,height,label){
// 委托调用
this.init( width, height );
this.label = label || "Default";
this.$elem = $( "<button>" ).text( this.label );
};
Button.build = function($where) {
// 委托调用
this.insert( $where );
this.$elem.click( this.onClick.bind( this ) );
};
Button.onClick = function(evt) {
console.log( "Button '" + this.label + "' clicked!" );
};
$( document ).ready( function(){
var $body = $( document.body );
var btn1 = Object.create( Button );
btn1.setup( 125, 30, "Hello" );
var btn2 = Object.create( Button );
btn2.setup( 150, 40, "World" );
btn1.build( $body );
btn2.build( $body );
} );

why is typeof null “object”?

In the first implementation of JavaScript,
JavaScript values were represented as a type tag and a value, 
with the type tag for objects being 0, 
and null was represented as the NULL pointer (0x00 on most platforms). 
As a result, null had 0 as a type tag, hence the bogus typeof return value (reference).

This is generally regarded as a mistake. 

null有时会被当作一种对象类型，但是这其实只是语言本身的一个bug，
即对null执行typeof null时会返回字符串"object"。1实际上，null本身是基本类型。
1原理是这样的，不同的对象在底层都表示为二进制，在JavaScript中二进制前三位都为0的话会被
判断为object类型，null的二进制表示是全0，自然前三位也是0，所以执行typeof时会返
回“object”。——译者注




这个函数在定义时的词法作用域以外的地方被调用。
闭包使得函数可以继续访问定义时的词法作用域。
当函数可以记住并访问所在的词法作用域，即使函数是在当前词法作用域之外执行，
这时就产生了闭包。

为什么要用Array.prototype.slice.call

在javascript里有一种array-like的对象，就是和Array很像，比如拥有push，slice等方法，拥有length属性，而且你可以很容易的创建一个这样的对象:

var foo = {0:'hello',1:'world',length:2,slice:Array.prototype.slice}
转变成一个数组对象:

console.log(Array.prototype.slice.call(foo,0));
    //["hello", "world"]
    console.log(foo.slice())
    //["hello", "world"]
这里是因为这个像数组的对象拥有这个slice方法，如果没有呢？就不能直接使用，要使用原型的方法调用了。如下:

console.log(Array.prototype.splice.call(foo,0));
    //["hello", "world"]
    console.log(foo.splice())
    //TypeError: Object #<Object> has no method 'splice'
常用的arguments对象, document.links, document.forms等都是非常像数组（Array）的，可以使用Array.prototype.slice.call将其转换成数组操作。

function foo() {
console.log( this.a );
}
var a = 2;
var o = { a: 3, foo: foo };
var p = { a: 4 };
o.foo(); // 3
(p.foo = o.foo)(); // 2
赋值表达式p.foo = o.foo的返回值是目标函数的引用，因此调用位置是foo()而不是p.foo()或
者o.foo()。根据我们之前说过的，这里会应用默认绑定。


new到底发生了什么

还是我们上面的问题,当我们执行 var a = new A('testa') 到底发生了什么.MDN上是这么说的

对于 var o = new Foo();
//JavaScript 实际上执行的是：
var o = new Object();
o.[[Prototype]] = Foo.prototype;
Foo.call(o);



思考下面的代码：
function Foo() { /* .. */ }
Foo.prototype = { /* .. */ }; // 创建一个新原型对象
var a1 = new Foo();
a1.constructor === Foo; // false!
a1.constructor === Object; // true!
Object(..)并没有“构造”a1，对吧？看起来应该是Foo()“构造”了它。大部分开发者都认为是Foo()执
行了构造工作，但是问题在于，如果你认为“constructor”表示“由......构造”的话，a1.constructor应
该是Foo，但是它并不是Foo！
到底怎么回事？a1并没有.constructor属性，所以它会委托[[Prototype]]链上的Foo.prototype。但
是这个对象也没有.constructor属性（不过默认的Foo.prototype对象有这个属性！），所以它会继续
委托，这次会委托给委托链顶端的Object.prototype。这个对象有.constructor属性，指向内置
的Object(..)函数。


// 和你想要的机制不一样！
Bar.prototype = Foo.prototype;
// 基本上满足你的需求，但是可能会产生一些副作用 :(
Bar.prototype = new Foo();
Bar.prototype = Foo.prototype并不会创建一个关联到Bar.prototype的新对象，它只是
让Bar.prototype直接引用Foo.prototype对象。因此当你执行类似Bar.prototype.myLabel = ...的赋
值语句时会直接修改Foo.prototype对象本身。显然这不是你想要的结果，否则你根本不需要Bar对
象，直接使用Foo就可以了，这样代码也会更简单一些。
Bar.prototype = new Foo()的确会创建一个关联到Bar.prototype的新对象。但是它使用了Foo(..)
的“构造函数调用”，如果函数Foo有一些副作用（比如写日志、修改状态、注册到其他对象、给this添
加数据属性，等等）的话，就会影响到Bar()的“后代”，后果不堪设想。
因此，要创建一个合适的关联对象，我们必须使用Object.create(..)而不是使用具有副作用
的Foo(..)。这样做唯一的缺点就是需要创建一个新对象然后把旧对象抛弃掉，不能直接修改已有
的默认对象。

if (!Object.create) {
Object.create = function(o) {
function F(){}
F.prototype = o;
return new F();
}

jQuery deffered和promise对象方法

Promise对象其实就是deferred对象的特例，因为Promise对象不能更改异步状态，而deferred对象可以。这点在他们的方法设计上，有着明显的体现。
要知道，Promise对象是没有resolve,reject,notify等方法的，也就意味着，你无法针对Promise对象进行状态更改，只能在done或fail中进行回调配置。
//https://segmentfault.com/a/1190000000523676

if (!Function.prototype.bind) {
  Function.prototype.bind = function (oThis) {
    if (typeof this !== "function") {
      // closest thing possible to the ECMAScript 5
      // internal IsCallable function
      throw new TypeError("Function.prototype.bind - what is trying to be bound is not callable");
    }

    var aArgs = Array.prototype.slice.call(arguments, 1), 
        fToBind = this, 
        fNOP = function () {},
        fBound = function () {
          return fToBind.apply(this instanceof fNOP
                                 ? this
                                 : oThis || this,
                               aArgs.concat(Array.prototype.slice.call(arguments)));
        };

    fNOP.prototype = this.prototype;
    fBound.prototype = new fNOP();

    return fBound;
  };
}

function test(){
	var a = function (){

	};
	a.prototype.A = function(){
		console.log("A");
		var b = function(){

		}
		b.prototype.B = function(){
			console.log("B");
		}
		return new b;
	}
	return new a;
}

当你对一个空变量求值时，空值null在数值类型环境中会被当作０来对待，而布尔类型环境中会被当作false
。例如：

var n = null;
console.log(n * 32); //  0

对象属性名字可以是任意字符串，包括空串。如果对象属性名字不是合法的javascript标识符，它必须用""包裹。
属性的名字不合法，那么便不能用.访问属性值，而是通过类数组标记访问和赋值。

var unusualPropertyNames = {
  "": "An empty string",
  "!": "Bang!"
}
console.log(unusualPropertyNames."");   // SyntaxError: Unexpected string
console.log(unusualPropertyNames[""]);  // An empty string
console.log(unusualPropertyNames.!);    // SyntaxError: Unexpected token !
console.log(unusualPropertyNames["!"]); // Bang!

var str = "this string \
is broken \
across multiple\
lines."
console.log(str);   // this string is broken across multiplelines.

try...catch 语句有一个包含一条或者多条语句的try代码块,  0个或多个的catch代码块, catch代码块中的语句会在try代码块中抛出异常时执行。 
换句话说, 如果你在try代码块中的代码如果没有执行成功，那么你希望将执行流程转入catch代码块。
如果try代码块中的语句 (或者try 代码块中调用的方法) 一旦抛出了异常,那么执行流程会立即进入catch 代码块. 
如果try代码块没有抛出异常,catch代码块就会被跳过. finally 代码块总会紧跟在try和catch代码块之后执行，但会在try和catch代码块之后的其他代码之前执行.

function f() {
    try {
        alert(0);
        throw "bogus";
    } catch(e) {
        alert(1);
        return true; // this return statement is suspended until finally block has completed
        alert(2); // not reachable
    } finally {
        alert(3);
        return false; // overwrites the previous "return"
        alert(4); // not reachable
    }
    // "return false" is executed now
    
    alert(5); // not reachable
}
f(); // alerts 0, 1, 3; returns false

终止一个label

var x = 0;
var z = 0
labelCancelLoops: while (true) {
  console.log("外部循环: " + x);
  x += 1;
  z = 1;
  while (true) {
    console.log("内部循环: " + z);
    z += 1;
    if (z === 10 && x === 10) {
      break labelCancelLoops;
    } else if (z === 10) {
      break;
    }
  }
}

继续一个label
checkiandj:
  while (i < 4) {
    console.log(i);
    i += 1;
    checkj:
      while (j > 4) {
        console.log(j);
        j -= 1;
        if ((j % 2) == 0) {
          continue checkj;
        }
        console.log(j + " is odd.");
      }
      console.log("i = " + i);
      console.log("j = " + j);
  }


  一些 JavaScript 对象, 例如 document.getElementsByTagName() 返回的 NodeList 或者函数内部可用的 arguments 对象，
  他们表面上看起来，外观和行为像数组，但是不共享他们所有的方法。arguments 对象提供一个 length 属性，但是不实现 forEach() 方法

function printArguments() {
	Array.prototype.forEach.call(arguments, function(item) {
	  	console.log(item);
	});
}
NaN === NaN//false
Number.NaN(NaN)//true
Map可以是重复值
Set不可以 length属性的值为0


1.数组排序

[1,9,22,3,5,50,81].sort(function(a, b) {
    return a - b;//改成b - a则从大到小排列
});
4.数组去重

[1,3,3,4,6,5,4,7,8,8,8,5,2,1].filter(function(value, index, array) {
    return array.indexOf(value) == index;
});
5.数字千分位分割

var a = 123123123123123;
console.log(a.toLocaleString()); // '123,123,123,123,123'


判等
x	y	==	===	Object.is(x,y)
+0	-0	true	true	false
NaN	NaN	false	false	true

关闭输入法
< input style=”ime-mode:disabled”>

封装JS

个人给一点建议不要再搞什么命名空间了。已经进入 2016 年啦，JavaScript 现在的模块化机制已经相当成熟，学习一下如何用 ES2015 Module 来封装模块，实际用的时候可以考虑各种模块转换编译器／加载器，浏览器的兼容性不会是问题。比如说 babel／webpack／jspm 等等都可以……这样吧，我简单帮你梳理一下：

首先学会怎么用 ES2015 来编写／封装模块（node／npm，加上 babel 的入门知识）
记住，在编写封装模块时是不需要考虑兼容性的，后面有办法让你向后兼容（至少到 ES5）
学会发布它，比如说发布到 npm
学会如何引入模块到你的应用体系中去，如何加载／打包（用上 gulp／webpack／jspm 等等，取决于项目）
这是一个完整的生态系统，封装不只是要学习代码怎么写，更要知道如何维护，如何应用，否则封装的没有普适性就没有价值。

至于具体到新的模块语法怎么写，我这里有之前回答别人（不在 SF）的一部分内容供你参考——别想太复杂，可以很简单的：

我不喜欢用 Class，不管是过去的构造器模式还是现在的新语法。不是因为对任何编程范式有偏见，本来 JavaScript 就不是 Class Based OO 语言，硬生生的去模仿就是会觉得别扭罢了。和 Java、C# 等语言不同，class 不是必需品，这也就意味着你完全可以不用。然而奇怪的是使用 JavaScript 的人很多却是不用 class 不行，这是不是对这门语言存在很大的误解呢？

更重要的是在实践中我们会发现使用其他的模式——比如工厂函数（Factory Functions）要远比 class 简洁、灵活，用它来替代 class 几乎总能得到更好的结果。

简单工厂函数
// person.js
export default _ => {
  return {
    greet() {
      console.log('Hello world!')
    }
  }
}
用起来和一个 Class 几乎一模一样——除了不需要用 new，这不算坏处吧？

import Person from './person'

const nightire = Person()
nightire.greet()  // "Hello world!"
依赖注入
console.log 限制了 greet 方法的行为，为了不局限问候的方式，可以使用依赖注入——这是解耦的一种简便易行的方法。即使在现实中很多时候看不出需要依赖注入的迹象，我们也应该有意识的这么做。在定义一种“类型”的时候对外界知道的越少越好（于是就更容易复用、扩展、组合……）。

// person.js
export default ioStream => {
  return {
    greet() {
      ioStream.send('Hello world!')
    }
  }
}
比如说我们可以把 console 封装一下，让系统内所有的 ioStream 都具有统一的接口，然后就可以直接使用：

import Person from './person'
import ConsoleStream from 'console-stream'

const nightire = Person(ConsoleStream())
nightire.greet()  // "Hello world!"
不用 Mocking 的单元测试
这个是顺带一提的事情，因为我注意到不懂得处理依赖注入（或者说更高层次上的解耦概念）的人通常都会把单元测试写得无比蛋疼……实际上，对象字面量在很多时候胜过一切构造模式：

import test from 'ava'
import Person from './person'

test(`a person can send message to io stream`, t => {
  const ioStream = {
    send(anything) {
      t.same(anything, 'Hello world!');
    }
  }
  
  const anyone = Person(ioStream)
  anyone.greet()
})
封装
其实私有成员可以变得很自然很自然，闭包一样在用，只是不那么扎眼了：

// person.js
export default ioStream => {
  let _message = 'Hello world!'

  return {
    greet() {
      ioStream.send('Hello world!')
    },
    
    get message() {
      return _message
    },
    
    set message(message) {
      _message = message
    }
  }
}
用法就不写了，和之前没什么区别。getter/setter 也不是必须的，看接口设计需求了。

组合
这才是对 OO 来说最重要的（相较于怎么定义／创建对象来说），总的来说组合总是要优于继承，工厂模式搞起来尤其轻松。

比方说我们已经有了一个动作“类”：

// action.js
export default ioStream => {
  return {
    wave() {
      ioStream.send('(Waving Hands...)')
    }
  }
}
那么与 Person 的组合可以这样：

import Person from './person'
import Action from './action'
import ConsoleStream from 'console-stream'

const _console = ConsoleStream()
const nightire = Object.assign({}, Person(_console), Action(_console))

nightire.message = 'Farewell, my friend!'
nightire.wave()  // "(Waving Hands...)"
nightire.greet()  // "Farewell, my friend!"
事前绑定的方法引用
这是我觉得最好的一个优点。由于 this 在 JavaScript 中是在运行时动态绑定的，如果使用你代码的人不理解这一点，那么他们就会犯错误（而且会指责是你写的不对……）。有些人是因为不理解 this 而不敢用，有些人则是为了迁就前者而干脆不去用，架构师会比较容易体会这类情况。

这是典型的容易犯错的例子：

// stepper.js
export default class Stepper {
  constructor(offset) {
    this.offset = offset
  }
  
  add(amount) {
    return amount + this.offset
  }
}

// main.js
import Stepper from './stepper'

const stepper = new Stepper(1)
[1, 2, 3].map(stepper.add.bind(stepper))
容易犯错的地方就是最后一行，如果不加 .bind(stepper) 的话最终 this 的指向就是错误的。但往往使用者并不理解这一点，反正看到你的文档就知道这个能加上初始化传入的 offset 就是了，除非你不厌其烦的在文档里强调：“注意上下文的变化，如有必要请用 bind() 明确 this 的指向“……啊，说不定你还得培训一下让大家都知道如有“必要”的确切范围。

然而你也可以这样来重写一下：

// stepper.js
export default offset => {
  return {
    add(amount) {
      return amount + offset
    }
  }
}

// main.js
import Stepper from './stepper'

const stepper = Stepper(1)
[1, 2, 3].map(stepper.add)  // [2, 3, 4]
于是无论是具体实现还是接口定义都能保持简洁一致。


如何获得某个月的天数？
function getDays(year, month) {
  if (month === 1) return new Date(year, month, 29).getMonth() === 1 ? 29 : 28;
  return new Date(year, month, 31).getMonth() === month ? 31 : 30;
}
我们发现，new Date()的第三个参数是可以大于我们所知的每个月的最后一天的的，比如：

new Date(2016, 0, 200) //Mon Jul 18 2016 00:00:00 GMT+0800 (CST)

原生AJAX
(() => {
  'use strict'

  // 创建一个新的 XMLHttpRequest。这是在无框架情况下使用 AJAX 的方法
  const xhr = new XMLHttpRequest()
  // 声明 HTTP 请求方法和地址
  xhr.open('GET', 'https://randomuser.me/api/?results=3')
  // in a GET request what you send doesn't matter GET 请求
  // in a POST request this is the request body
  xhr.send(null)

  // 等待 'readystatechange' 状态改变去触发 xhr 对象
  xhr.onreadystatechange = function () {
    //等待 xhr 成功成功返回
    if (xhr.readyState !== 4 ) { return }
    // 非 200 状态时输出错误信息
    if (xhr.status !== 200) { return console.log('Error: ' + xhr.status) }

    // 一切正常！输出响应
    console.log(xhr.responseText)
  }
})()


随波逐流的组件写法
// Window root
var root = (window !== 'undefined' ? window : self);

(function (global, factory) {

    'use strict';

    /* Use AMD */
    if (typeof define === 'function' && define.amd) {
        define(function () {
            return new (factory(global, global.document))();
        });
    }
    /* Use CommonJS */
    else if (typeof module !== 'undefined' && module.exports) {
        module.exports = (factory(global, global.document))();
    }
    /* Use Browser */
    else {
        global.Push = new (factory(global, global.document))();
    }

})(root, function (w, d) {

相比 HTTP/1.x，HTTP/2 在底层传输做了很大的改动和优化：

1、HTTP/2 采用二进制格式传输数据，而非 HTTP/1.x 的文本格式。二进制格式在协议的解析和优化扩展上带来更多的优势和可能。
2、HTTP/2 对消息头采用 HPACK 进行压缩传输，能够节省消息头占用的网络的流量。而 HTTP/1.x 每次请求，都会携带大量冗余头信息，浪费了很多带宽资源。头压缩能够很好的解决该问题。
3、多路复用，直白的说就是所有的请求都是通过一个 TCP 连接并发完成。HTTP/1.x 虽然通过 pipeline 也能并发请求，但是多个请求之间的响应会被阻塞的，所以 pipeline 至今也没有被普及应用，而 HTTP/2 做到了真正的并发请求。同时，流还支持优先级和流量控制。
4、Server Push：服务端能够更快的把资源推送给客户端。例如服务端可以主动把 JS 和 CSS 文件推送给客户端，而不需要客户端解析 HTML 再发送这些请求。当客户端需要的时候，它已经在客户端了。

移动端常见问题
http://www.cnblogs.com/PeunZhang/p/3407453.html#question_23
类型转换
var myVar = "3.1415";
str = "" + myVar,//to stringify
int = ~~myVar,//to integer
float = 1*myVar,//to float
bool = !!myVar,//to Boolean
array = [myVar];//to Array

requestAnimationFrame的用法

// shim layer with setTimeout fallback
window.requestAnimFrame = (function(){
  return  window.requestAnimationFrame       ||
          window.webkitRequestAnimationFrame ||
          window.mozRequestAnimationFrame    ||
          function( callback ){
            window.setTimeout(callback, 1000 / 60);
          };
})();


// usage:
// instead of setInterval(render, 16) ....

(function animloop(){
  requestAnimFrame(animloop);
  render();
})();
// place the rAF *before* the render() to assure as close to
// 60fps with the setTimeout fallback.



React在ES6的实现中去掉了getInitialState这个hook函数，规定state在constructor中实现
let app = React.createClass({
	getInitialState: function(){
        // some thing
    }
})
ES6中可以写成
class app extends React.Component{
    getInitialState(){
         // some thing
    }
}
视频直播HLS
http://www.alloyteam.com/2016/05/h5-camera-literacy/
JavaScript 原型中的哲学思想
https://segmentfault.com/a/1190000005824449

chrome使用技巧

ctrl+p 项目中定位文件
ctrl+shif+o 定位成员函数
Chrome在souces页面提供snippets一栏，这里我们可以随时编写JS代码
JSON.stringify({name: 'lxjwlt'}, null, 4);
copy 格式化拷贝：
	请求项的右键菜单中选择Copy Response拷贝响应内容
	命令行中使用copy接口处理数据（copy(xxxx)）
	得到格式化的JSON数据(ctrl + v)
Chrome控制台提供debug接口，可以传入一个函数，当这个函数下次执行的时候，调试器会自动在该函数中进行断点调试。
开启Async模式后，异步函数之前的调用栈都会被记录下来，而且调用栈中代码执行状态也得到了保留。

H5图片上传
https://segmentfault.com/a/1190000006140042
canvas压缩图片
var canvas = document.createElement('canvas'),
    ctx = canvas.getContext('2d');
    
function compress(img) {
    canvas.width = img.width;
    canvas.height = img.height;
    
    //利用canvas进行绘图
    
    //将原来图片的质量压缩到原先的0.2倍。
    var data = canvas.toDataURL('image/jpeg', 0.2); //data url的形式
    
    return data;
}


逻辑判断
    判断为false的特殊值：false、undefined、null、0、NaN、""。
    简单boolean和对象Boolean类型：简单boolean类型的false和true与对象Boolean类型的false和true是有区别，两者是不相等的。如下面的例子：
		var b = new Boolean(false);
		if (b) // 返回true
		if (b == true) // 返回false

异常处理
  finally返回值：如果finaly添加了return 语句，则不管整个try.catch返回什么，返回值都是finally的return。如下所示：

	function f() {
	    try {
	        console.log(0);
	        throw "bogus";
	    } catch(e) {
	        console.log(1);
	        return true; // 返回语句被暂停，直到finally执行完成
	        console.log(2); // 不会执行的代码
	    } finally {
	        console.log(3);
	        return false; //覆盖try.catch的返回
	        console.log(4); //不会执行的代码
	    }
	    // "return false" is executed now 
	    console.log(5); // not reachable
	}
	f(); // 输出 0, 1, 3; 返回 false

	    finally吞并异常：如果finally有return并且catch中有throw异常。throw的异常不会被捕获，因为已经被finally的return覆盖了。如下代码所示：

	function f() {
	    try {
	        throw "bogus";
	    } catch(e) {
	        console.log('caught inner "bogus"');
	        throw e; // throw语句被暂停，直到finally执行完成
	    } finally {
	        return false; // 覆盖try.catch中的throw语句
	    }
	    // 已经执行了"return false"
	}

	try {
	    f();
	} catch(e) {
	    //这里不会被执行，因为catch中的throw已经被finally中的return语句覆盖了
	    console.log('caught outer "bogus"');
	}
	// 输出
	// caught inner "bogus"


	这里有个问题需要特别注意，如果字符串键值能够被强制类型转换为十进制数字的话，它
	就会被当作数字索引来处理。
	var a = [ ];
	a["13"] = 42;
	a.length; // 14

	var a = "foo"
	可惜我们无法“借用”数组的可变更成员函数，因为字符串是不可变的：
	Array.prototype.reverse.call( a );
	// 返回值仍然是字符串"foo"的一个封装对象（参见第3章）:(
	一个变通（破解）的办法是先将字符串转换为数组，待处理完后再将结果转换回字符串：
	var c = a
	 // 将a的值转换为字符数组
	 .split( "" )
	 // 将数组中的字符进行倒转
	 .reverse()
	 // 将数组中的字符拼接回字符串
	 .join( "" );
	c; // "oof"
	这种方法的确简单粗暴，但对简单的字符串却完全适用。

	2.toFix(3)//报错
	2..toFix(3)//2.000 
	因为 . 被视为常量 2. 的一部分

	void 0 === void 1 === void(0) === undefined

	NaN 是 JavaScript 中唯一一个不等于自身的值

	if (!Object.is) {
	 	Object.is = function(v1, v2) {
		 	// 判断是否是-0
		 	if (v1 === 0 && v2 === 0) {
		 		return 1 / v1 === 1 / v2;
		 	}
		 	// 判断是否是NaN
		 	if (v1 !== v1) {
		 		return v2 !== v2;
		 	}
	 		// 其他情况
	 		return v1 === v2;
	 	};
	}

	function foo(x) {
		x.push( 4 );
		x; // [1,2,3,4]
		// 然后
		x = [4,5,6];
		x.push( 7 );
		x; // [4,5,6,7]
	}
	var a = [1,2,3];
	foo( a );
	a; // 是[1,2,3,4]，不是[4,5,6,7]
	我们向函数传递 a 的时候，实际是将引用 a 的一个复本赋值给 x，而 a 仍然指向 [1,2,3]。
	在函数中我们可以通过引用 x 来更改数组的值（push(4) 之后变为 [1,2,3,4]）。但 x =
	[4,5,6] 并不影响 a 的指向，所以 a 仍然指向 [1,2,3,4]。

	所有 typeof 返回值为 "object" 的对象（如数组）都包含一个内部属性 [[Class]]（我们可
	以把它看作一个内部的分类，而非传统的面向对象意义上的类）。这个属性无法直接访问，
	一般通过 Object.prototype.toString(..) 来查看。例如：
	Object.prototype.toString.call( [1,2,3] );

	如果想要得到封装对象中的基本类型值，可以使用 valueOf() 函数：
	原生函数 ｜ 37
	var a = new String( "abc" );
	var b = new Number( 42 );
	var c = new Boolean( true );
	a.valueOf(); // "abc"
	b.valueOf(); // 42
	c.valueOf(); // true
	在需要用到封装对象中的基本类型值的地方会发生隐式拆封。具体过程（即强制类型转
	换）将在第 4 章详细介绍。
	var a = new String( "abc" );
	var b = a + ""; // b的值为"abc"
	typeof a; // "object"
	typeof b; // "string"

	try{
		something
	}catch(e){
		window.location.href = "http://stackoverflow.com/search?q=[js]+"+e.message;
	}

	~x 大致等同于 -(x+1)

	~~-49.6; // -49

	var a = { b: 42 };
	var b = { b: 43 };
	a < b; // false
	a == b; // false
	a > b; // false
	a <= b; // true
	a >= b; // true
	强制类型转换 ｜ 91
	为什么 a == b 的结果不是 true ？它们的字符串值相同（同为 "[object Object]"），按道
	理应该相等才对？实际上不是这样，你可以回忆一下前面讲过的对象的相等比较。
	但是如果 a < b 和 a == b 结果为 false，为什么 a <= b 和 a >= b 的结果会是 true 呢？
	因为根据规范 a <= b 被处理为 b < a，然后将结果反转。因为 b < a 的结果是 false，所
	以 a <= b 的结果是 true。
	这可能与我们设想的大相径庭，即 <= 应该是“小于或者等于”。实际上 JavaScript 中

	还有一个坑常被提到（涉及强制类型转换，参见第 4 章）：
	[] + {}; // "[object Object]"
	{} + []; // 0
	表面上看 + 运算符根据第一个操作数（[] 或 {}）的不同会产生不同的结果，实则不然。
	第一行代码中，{} 出现在 + 运算符表达式中，因此它被当作一个值（空对象）来处理。第
	4 章讲过 [] 会被强制类型转换为 ""，而 {} 会被强制类型转换为 "[object Object]"。
	但在第二行代码中，{} 被当作一个独立的空代码块（不执行任何操作）。代码块结尾不需
	要分号，所以这里不存在语法上的问题。最后 + [] 将 [] 显式强制类型转换（参见第 4 章）
	为 0。
	[] + [];//""
	{} + {};//NaN

	Promise
		http://www.jianshu.com/p/063f7e490e9a

		Promise API
			new Promise(..) 构造器
			Promise.resolve(..) 和 Promise.reject(..)
			then(..) 和 catch(..)
			Promise.all([ .. ]) 和 Promise.race([ .. ])
	在我们对 Web 平台 HTML5 的一个叫作 Web Worker 的新增特性的探索过程中，这些都是
	很好的问题。这是浏览器（即宿主环境）的功能，实际上和 JavaScript 语言本身几乎没什
	么关系。也就是说，JavaScript 当前并没有任何支持多线程执行的功能。
	但是，像你的浏览器这样的环境，很容易提供多个 JavaScript 引擎实例，各自运行在自己
	的线程上，这样你可以在每个线程上运行不同的程序。程序中每一个这样的独立的多线程
	部分被称为一个（Web）Worker。这种类型的并行化被称为任务并行，因为其重点在于把
	程序划分为多个块来并发运行。
	从 JavaScript 主程序（或另一个 Worker）中，可以这样实例化一个 Worker：
	var w1 = new Worker( "http://some.url.1/mycoolworker.js" );
	这个 URL 应该指向一个 JavaScript 文件的位置（而不是一个 HTML 页面！），这个文件将
	被加载到一个 Worker 中。然后浏览器启动一个独立的线程，让这个文件在这个线程中作
	为独立的程序运行。	
	结构化克隆算法（structured clone algorithm）（https://
developer.mozilla.org/en-US/docs/Web/Guide/API/DOM/The_structured_clone_algorithm		


浏览器 HTTP 协议缓存机制详解:
https://my.oschina.net/leejun2005/blog/369148
Ajax五个readystate：
http://www.jb51.net/article/16966.htm
js图片懒加载【可视区域加载】
https://www.talkingcoder.com/article/6370149516108046040
JavaScript函数节流和函数防抖之间的区别
http://www.jianshu.com/p/b73c2acad696?utm_campaign=hugo&utm_medium=reader_share&utm_content=note
微信、浏览器唤醒APP
	引用lizard的'cUtilCryptBase64'模块对url编码
	 
	var encodeUrl = UtilCryptBase64.Base64.encode(window.location.href);
	var awakeurl = "ctrip://wireless/h5?url=" + encodeUrl + "&type=2";
	 
	var si = setInterval(function () {
	if ($(".modal-tip").text() == "仅限携程App参加活动") {
	$(".modal-tip").text("正在跳转携程APP...");
	}
	if (navigator.userAgent.indexOf("MicroMessenger") > 0) {
	//微信
	if (window.Mkt && window.Mkt.Weixin && window.Mkt.Weixin.wxLanuch3rd) {
	clearInterval(si);
	window.Mkt.Weixin.wxLanuch3rd({
	"schema": awakeurl,
	isdown: true,
	mktwxpageid: $("#page_id").val(),
	callback: function () { }
	});
	}
	}
	else {//浏览器
	clearInterval(si);
	if (typeof window["__wakeup"] === "undefined") {
	window["__wakeup"] = [];
	}
	window["__wakeup"].push({
	'url': window.location.href,
	'isdown': true,
	'backurl': awakeurl,
	'waking': function () { }
	});
	}
	}, 100);
Js二分搜索
	function binarySearch(data, dest, start, end){  
	    var end = end || data.length - 1,  
	        start = start || 0,  
	        m = Math.floor((start + end) / 2);  
	    if(data[m] == dest){  
	        return m;  
	    }  
	    if(dest < data[m]){  
	        return binarySearch(data, dest, 0, m-1);  
	    }else{  
	        return binarySearch(data, dest, m+1, end);  
	    }  
	    
	    return false;  
	}  
	var arr = [-34, 1, 3, 4, 5, 8, 34, 45, 65, 87];  
	binarySearch(arr,4); 
堆排序:(https://github.com/hustcc/JS-Sorting-Algorithm/blob/master/7.heapSort.md)
	var len;    // 因为声明的多个函数都需要数据长度，所以把len设置成为全局变量

	function buildMaxHeap(arr) {   // 建立大顶堆
	    len = arr.length;
	    for (var i = Math.floor(len/2); i >= 0; i--) {
	        heapify(arr, i);
	    }
	}

	function heapify(arr, i) {     // 堆调整
	    var left = 2 * i + 1,
	        right = 2 * i + 2,
	        largest = i;

	    if (left < len && arr[left] > arr[largest]) {
	        largest = left;
	    }

	    if (right < len && arr[right] > arr[largest]) {
	        largest = right;
	    }

	    if (largest != i) {
	        swap(arr, i, largest);
	        heapify(arr, largest);
	    }
	}

	function swap(arr, i, j) {
	    var temp = arr[i];
	    arr[i] = arr[j];
	    arr[j] = temp;
	}

	function heapSort(arr) {
	    buildMaxHeap(arr);

	    for (var i = arr.length-1; i > 0; i--) {
	        swap(arr, 0, i);
	        len--;
	        heapify(arr, 0);
	    }
	    return arr;
	}
LazyMan
	function _LazyMan(name) {
	    this.tasks = [];   
	    var self = this;
	    var fn =(function(n){
	        var name = n;
	        return function(){
	            console.log("Hi! This is " + name + "!");
	            self.next();
	        }
	    })(name);
	    this.tasks.push(fn);
	    setTimeout(function(){
	        self.next();
	    }, 0); // 在下一个事件循环启动任务
	}
	/* 事件调度函数 */
	_LazyMan.prototype.next = function() { 
	    var fn = this.tasks.shift();
	    fn && fn();
	}
	_LazyMan.prototype.eat = function(name) {
	    var self = this;
	    var fn =(function(name){
	        return function(){
	            console.log("Eat " + name + "~");
	            self.next()
	        }
	    })(name);
	    this.tasks.push(fn);
	    return this; // 实现链式调用
	}
	_LazyMan.prototype.sleep = function(time) {
	    var self = this;
	    var fn = (function(time){
	        return function() {
	            setTimeout(function(){
	                console.log("Wake up after " + time + "s!");
	                self.next();
	            }, time * 1000);
	        }
	    })(time);
	    this.tasks.push(fn);
	   return this;
	}
	_LazyMan.prototype.sleepFirst = function(time) {
	    var self = this;
	    var fn = (function(time) {
	        return function() {
	            setTimeout(function() {
	                console.log("Wake up after " + time + "s!");
	                self.next();
	            }, time * 1000);
	        }
	    })(time);
	    this.tasks.unshift(fn);
	    return this;
	}
	/* 封装 */
	function LazyMan(name){
	    return new _LazyMan(name);
	}
XSS和CSRF的区别
	XSS是你访问正常网站的时候，你浏览器执行了攻击者的代码。有XSS的网站可以直接伪造请求，比CSRF强多了。
	CSRF是你点击了攻击者提供的链接的时候（正常网站或其它有攻击者代码的网站），你的浏览器向正常网站发送攻击者期望的请求。
	http://blog.csdn.net/liushulin183/article/details/52613658
前端优化不完全指南：
	https://aotu.io/notes/2016/03/16/optimization/
Underscore _.template 方法使用详解
	https://github.com/hanzichi/underscore-analysis/issues/26
		_.template 支持以下三种模板。

			1. <%  %> - to execute some code
			2. <%= %> - to print some value in template
			3. <%- %> - to print some values HTML escaped
		如果你不喜欢它默认的模板风格，也可以自己定义，注意 key 必须和源码中的 key 保持一致，才能覆盖。

		_.templateSettings = {
		  // 三种渲染模板
		  evaluate    : /<%([\s\S]+?)%>/g,
		  interpolate : /<%=([\s\S]+?)%>/g,
		  escape      : /<%-([\s\S]+?)%>/g
		};

坑：

0.1+0.2这个坑在支付上确实麻烦

Chrome 中文界面下默认会将小于 12px 的文本强制按照 12px 显示, 
  可通过加入 CSS 属性 -webkit-text-size-adjust: none; 解决.

好了，下面开始罗列下这次遇到的keng们：

开始之前先来描述下页面的结构：

<div class='header'>....</div>
<div class='main'>....</div>
<div class='footer'>
    <textarea></textarea>
</div>
其实就是最基本的上中下布局而已，问题是header和footer需要分别fixed到头部和底部

0.首页就是fixed定位的问题

遇到的都知道在ios的safari里面不支持position:fixed;(呵呵了),其实也不算是不支持，只是在软键盘弹出来的时候使用fixed的元素就开始各种抽风了。

解决方法：在键盘弹出来之前按照正常的定位，使用fixed，弹出来的时候将footer这部分position:static;然后这样footer就会跑向页面的最下面了，然后再将页面主动滚动到底部。当blur的时候再把footer设置回去

1.textarea的focus问题

这次有个需求是点击main的某个元素，需要textarea获取焦点并弹出软键盘。其实这些都不是关键。问题是在点击main的该元素的时候，还是需要跑个ajax判断下能否进行评论，这样一跑ajax就获取不了焦点了。

解决方法：其实为啥使用了oTextarea.focus()不管用，其实就是中间的这层ajax使用了异步了。这样浏览器以为这是两件事，处于安全性考虑它给禁止了，然后将ajax改成同步的就OK了。

同样在window.open()的时候也会遇到这个问题，解法相同。

2.在特定的android的机子(vivo…)里，在键盘收起来不会失焦

本来是在失焦的时候触发了一个blur事件的东西，这样就会出现问题。

解决方法：

(function() {
    var height = window.innerHeight;
    function loop_height() {
        setTimeout(function() {
            if(window.innerHeight > height + 200){
                //触发blur事件
            }else{
                height = window.height;
            }
        },1000);
    }
    loop_height();
})();
其实就是一直获取页面的可视高度，在键盘弹出来的时候键盘的部分不算innerHeight;这样在键盘收起来的时候就能知道了，如果感觉时间比较长可以把setTimeout的时间改小一点，默认在少于300ms的时候感觉不出页面的迟钝。

3.在微信内zepto的方法不触发(偶发)

这个还真是难为了测试妹子了，没想到还有这种

解决方法：这个大家就都知道了，改成原生的就成了呗

4.在微信内textarea focus的时候弹出键盘，会遮盖一部分textarea的元素(偶发,特定机型)

解决方法：在focus事件的最后

$body.scrollTop($body.get(0).scrollHeight + $("textarea").height());
就是让页面主动滚动下，反正textarea在最底部，那么就把页面滚动到最底下就成了

5.IOS上带-的日期，new Date时会出错

6.微信分享from参数

7.revert merge会导致下次merge无效
http://www.tuicool.com/articles/iIzeY3e



```
