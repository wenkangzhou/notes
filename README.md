# 继承
### 类式继承
```javascript
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
```
* 默认模式
```javascript
    function inherit(C,P){
		C.prototype = new P();
	}
	var kid = new Child();
	kid.say();
```
缺点：同时继承两个对象的属性，即添加到this的属性以及原型属性，绝大多数时候，并不需要这些自身的属性，因为它们很可能是指向一个特定的实例而非复用。   
注意：对于构造函数的一般经验法则是：应该将可复用的成员添加到原型中。
* 借用构造函数
```javascript
    function Child(a,b,c,d){
		Parent.apply(this,arguments)
	}
```
多重继承
```javascript
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
```
缺点：无法从原型中继承任何东西
* 借用和设置原型
```javascript
    function Child(a,c,b,d){
    	Parent.apply(this,arguments)
    }
    Child.prototype = new Parent();
```
缺点：父构造函数被调用了两次
```javascript
	var kid = new Child("Patrick");
	kid.name;//输出"Patrick"
	kid.say();//输出"Patrick"
	delete kid.name;
	kid.say();//输出Adam
```
删除自身属性后，随后输出的是原型链的值
* 共享原型   
本模式的经验法则在于：可复用成员应该转移到原型中而不是放置在this中。因此，出于继承的目的，任何值得继承的东西都应该放置在原型中实现。所以，可以仅将子对象的原型与父对象的原型设置为相同的即可
```javascript
	function inherit(C,P){
		C.prototype = P.prototype;
	}
```
缺点：继承链下方的某处存在一个子对象或孙子对象修改了原型，它将影响到所有的父对象和祖先对象。
* 临时构造函数
```javascript
	function inherit(C,P){
		var F = function(){};
		F.prototype = P.prototype;
		C.prototype = new F();
	}
	//在上面模式的基础上，还可以添加一个指向原始对象的引用。
	function inherit(C,P){
		var F = function(){};
		F.prototype = P.prototype;
		C.prototype = new F();
		C.uber = P.prototype;
	}
	//最后，针对这个近乎完美的类式继承模式，还需要做的一件事就是重置该构造函数的指针，以免将来的某个时候还需要该构造函数。
function inherit(C,P){
	var F = function(){};
	F.prototype = P.prototype;
	C.prototype = new F();
	C.uber = P.prototype;
	C.prototype.constructor = C;
}
```
###Klass
```javascript
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
```
klass()实现中三个令人关注且独特的部分：
    1. 创建Child()构造函数。该函数将是最后返回的，并且该函数也用作类。在这个函数中，如果
    存在_construct方法，那么将会调用该方法。另外，在此之前，通过使用静态uber属性，其父
    类的_construct方法也会被自动调用（同样，如果存在改方法的话）。可能在有些情况下，当没
    有定义uber属性时，比如直接从Object类中继承时，这与Man类的定义中继承时相似的情况。
    2. 第二部分在一定程度上处理继承关系。它只是采用了本章前面章节中所讨论的类式继承的终极版。
    从代码上，只有一个新语句（Parent = Parent || Object），即如果没有传递需要被继承的类，那
    么就将Parent变为Object。
    3. 最后一节是遍历所有的实现方法（比如，本例中的_construct和getName），这些是该类的实际定义，
    并且也是将它们添加到Child的原型中的部分代码。       
    
###原型继承
```javascript
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

	//继承原型
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
```
通过复制属性实现继承
```javascript
	//浅复制：
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

	   //浅复制，由于通过引用传递，对子属性改变会影响父对象

		var dad = {
			counts:[1,2,3],
			reads:{paper:true}
		}
		var kid = extend(dad);
		kid.counts.push(4);
		dad.counts.toString();//1,2,3,4
		dad.reads === kid.reads;//true

	//深复制：
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
```
###混入
```javascript
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

	var cake = mix{
		{egg:2,large:true},
		{flour:"3 cups"}
	}
```
###借用方法
```javascript
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
```
