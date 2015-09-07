
###一、jQuery deferred的诞生
场景：

我们需要依次：
1. ajax从服务器获取A
2. loop遍历B
3. ajax从服务器获取C
4. loop遍历D


####what we do ？
通常的做法是，为它们指定回调函数（callback）

    ajax(loop(ajax(loop)))


进化版：

    ajax(
        loop(
            ajax(
                loop
                )
            )
    )


###jQuery deferred的应需而生

deferred对象是jQuery的回调函数解决方案。

* defer的翻译是"延迟"，所以deferred对象的含义就是"延迟"到未来某个点再执行。

* 它解决了如何处理耗时操作的问题，对那些操作提供了更好的控制，以及统一的编程接口。


###jQuery deferred思想

jQuery规定，deferred对象有三种执行状态----未完成，已完成和已失败。

* 如果执行状态是"已完成"（resolved）,deferred对象立刻调用done()方法指定的回调函数；
* 如果执行状态是"已失败"，调用fail()方法指定的回调函数；
* 如果执行状态是"未完成"，则继续等待，或者调用progress()方法指定的回调函数（jQuery1.7版本添加）。


简易的Deferred

    var Deferred = function () {
        var callbacks = [];
        return {
            done: function (fn) {
                callbacks.push(fn);
            },
            resolve: function (arg) {
                $.each(callbacks,function(index,callback){
                    callback(arg)
                })
                
            }
        };
    };


实例：

    var fn1 = function(){	
			var dtd = $.Deferred(); // 新建一个deferred对象
			var tasks = function(){
			　　alert("执行完毕！");
			　　dtd.resolve(); // 改变deferred对象的执行状态
			};
			setTimeout(tasks,5000);
			return dtd;
	};
	
    $.ajax().done(fn);
    fn1.done();

ajax和普通方法都必须返回一个deferred对象 


分析
        
        var fn1 = function(){	
                var dtd = $.Deferred();
                var tasks = function(){
                　　dtd.resolve(); 
                };
                setTimeout(tasks,5000);
                return dtd;
        };
	
    fn1.done();
steps：
1. fn1 return dtd
2. dtd.done 注册done事件
3. 执行fn1
3. fn1完成后dtd.resolve 触发done事件


###jQuery deferred主要功能

1. 回调的的链式写法 
        $.ajax().done()
2. 指定同一操作的多个回调函数
        $.ajax().done().fail()
3. 为多个操作指定回调函数
        $.when($.ajax(),$.ajax).done()
4. 普通操作的回调函数接口
        $.when(wait()).done()


###一、ajax操作的链式写法
jQuery的ajax操作的传统写法：

	$.ajax({
	　　　　url: "test.html",
	　　　　success: function(){
	　　　　　　alert("哈哈，成功了！");
	　　　	},
	　　	error:function(){
	　　　　	alert("出错啦！");
	　　	}
	});	　　


现在，新的写法是这样的：

	$.ajax("test.html")
	　　.done(function(){ alert("哈哈，成功了！"); })
	　　.fail(function(){ alert("出错啦！"); });


###二、指定同一操作的多个回调函数
deferred对象的一大好处，就是它允许你自由添加多个回调函数。

	$.ajax("test.html")
	　　.done(function(){ alert("哈哈，成功了！");} )
	　　.fail(function(){ alert("出错啦！"); } )
	　　.done(function(){ alert("第二个回调函数！");} );


###三、为多个操作指定回调函数
deferred对象的另一大好处，就是它允许你为多个事件指定一个回调函数，这是传统写法做不到的。

	$.when($.ajax("test1.html"), $.ajax("test2.html"))
	　　.done(function(){ alert("哈哈，成功了！"); })
	　　.fail(function(){ alert("出错啦！"); });	　　


B:ajax是jQuery包装好的返回deferred对象的方法

Q:如何让普通的方法也能享受deferred的便易呢?


###四、普通操作的回调函数接口
deferred对象的最大优点，就是它把这一套回调函数接口，从ajax操作扩展到了所有操作


我们来看一个具体的例子。假定有一个很耗时的操作wait：

	var wait = function(){
	　　　　var tasks = function(){
	　　　　　　alert("执行完毕！");
	　　　　};
	　　　　setTimeout(tasks,5000);
	　　};　　
　　
我们为它指定回调函数，应该怎么做呢？


很自然的，可以使用$.when()：

	$.when(wait())
	　　.done(function(){ alert("哈哈，成功了！"); })
	　　.fail(function(){ alert("出错啦！"); });

done()方法立即执行，没有起到回调函数的作用。

why？


原因在deferred思想中提到过 方法必须返回deferred对象，所以必须对wait()进行改写：

	var dtd = $.Deferred(); // 新建一个deferred对象
	var wait = function(){			
			var tasks = function(){
			　　alert("执行完毕！");
			　　dtd.resolve(); // 改变deferred对象的执行状态
			};
			setTimeout(tasks,5000);
			return dtd.promise();
	};				

dtd.promise()在原来的deferred对象上返回另一个deferred对象，
只开放与改变执行状态无关的方法（比如done()方法和fail()方法
屏蔽与改变执行状态有关的方法（比如resolve()方法和reject()方法），从而使得执行状态不能被外部改变。


###resolve()和reject()

* ajax操作，deferred对象会根据返回结果，自动改变自身的执行状态；

* 但是，在普通方法wait()函数中，这个执行状态必须由程序员手动指定。


* dtd.resolve()的意思是，将dtd对象的执行状态从"未完成"改为"已完成"，从而触发done()方法。

* deferred.reject()方法，作用是将dtd对象的执行状态从"未完成"改为"已失败"，从而触发fail()方法。


###小结：deferred对象的方法

1. $.Deferred() 
2. deferred.done() 
3. deferred.fail() 
4. deferred.promise() 
5. deferred.resolve() 
6. deferred.reject() 
7. $.when() 
8. deferred.then()，
9. deferred.always()


###jqeury deferred和promise关系
JavaScript原生异步编程的Promise模式有着自己的规范Promises/A+

* jQuery Deferred并没有按Promises/A+ 规范实现，但实现了类似的功能，
* 为了兼容规范 jquery实现了deferred.promise()


###promise规范有那些？


####promise规范
* 一个promise可能有三种状态：等待（pending）、已完成（fulfilled）、已拒绝（rejected）

* 一个promise的状态只可能从“等待”转到“完成”态或者“拒绝”态，不能逆向转换，同时“完成”态和“拒绝”态不能相互转换

* promise必须实现then方法（可以说，then就是promise的核心）

* then方法接受两个参数


###思考
不管是deferer promise callbacks实质都是包装的回调的方法 只是更加友好
有没有跳出来的方法的


###扩展 其他方法


####ES6 generator ES7 async/await


基础

* generator 生成器
* 迭代器协议 iterator


generator 生成器 原理：吧yield比作多个事物的原料 当所有yield原料都准备好了后 具体什么时候生产产品（执行）由我们决定  （顺序：依次生产）

        function* anotherGenerator(i) {
            yield i + 1;//index 2
            yield i + 2;//index 3
            yield i + 3;//index 4
        }
    
        function* generator(i){
            yield i; //index 1
            yield* anotherGenerator(i);//index 2-4
            yield i + 10;//index 5
        }
    
        var gen = generator(10);//返回一个这个生成器函数的迭代器（iterator）对象。
        console.log("--------------");
        console.log(gen.next().value); // 10
        console.log(gen.next().value); // 11
        console.log(gen.next().value); // 12
        console.log(gen.next().value); // 13
        console.log(gen.next().value); // 20
        console.log("--------------");


迭代器协议 iterator

ES6里的迭代器是一种协议 (protocol). 所有遵循了这个协议的对象都可以称之为迭代器对象.


核心：

next 的方法, 调用该方法后会返回一个拥有两个属性的对象：

* value 可以是任意值
* done  布尔值, 表示该迭代器是否已经被迭代完毕
     
        function iterator(){
            var index = 0;
    
            return {
                next: function(){
                    return {value: index++, done: false};
                }
            }
        }

        var it = iterator();
    
        console.log(it.next().value); // '0'
        console.log(it.next().value); // '1'
        console.log(it.next().value); // '2'


###新一代异步嵌套方案
1. Generator 函数就是一个异步操作的容器,按照迭代器协议收集了所有待生产的原料
2.  ？ 如何控制生产流程


###控制生产流程
 
 1. 回调函数。将异步操作包装成 Thunk 函数，在回调函数里面交回执行权。
 2. Promise 对象。将异步操作包装成 Promise 对象，用 then 方法交回执行权。


现有方法：

1. 使用co函数库 
>co 函数库其实就是将两种自动执行器（Thunk 函数和 Promise 对象），包装成一个库。使用 co 的前提条件是，Generator 函数的 yield 命令后面，只能是 Thunk 函数或 Promise 对象。

2. ES7 async/await


###async/await

		var asyncReadFile = async function (){
			  var f1 = await readFile('/etc/fstab');
			  var f2 = await readFile('/etc/shells');
			  console.log(f1.toString());
			  console.log(f2.toString());
			};	
	
	

不同：async 函数就是将 Generator 函数的星号（*）替换成 async，将 yield 替换成 await


优点：
1. 内置执行器。 Generator 函数的执行必须靠执行器，所以才有了 co 函数库，而 async 函数自带执行器

2. 更好的语义。 async 和 await，比起星号和 yield，语义更清楚了

3. 更广的适用性。 
    * co 函数库约定，yield 命令后面只能是 Thunk 函数或 Promise 对象，
    * async 函数的 await 命令后面，可以跟 Promise 对象和原始类型的值（数值、字符串和布尔值，这时等同于同步操作）

		