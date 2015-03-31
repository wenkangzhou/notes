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

	e.currentTarget//绑定处的dom
	e.target//点击处的dom

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
    		if(xhr.readyState ==4){
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
	模块模式得到了广泛的使用，并且强烈建议使用这种方式组织你的代码，尤其是当代吗日渐增长的时候。


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
						args = slice.call(arguments,1)

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
							index = index + 2；
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
		装饰着模式
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

		frag = doucument.createDoucmentFragment();
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
```
