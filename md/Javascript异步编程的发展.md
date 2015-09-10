
###"异步模式"编程的发展
经常的我们用callback实现异步，你还知道多少异步方案呢？


1. 回调函数callback
2. 事件监听
3. publish/subscribe 发布/订阅模式
4. Promise对象


####一、回调函数callback  回调黑洞 回调金字塔

	doAsync1(function () {
	  doAsync2(function () {
	    doAsync3(function () {
	      doAsync4(function () {
	    })
	  })
	})
	
* 优点：简单、容易理解和部署

* 缺点：不利于代码阅读和维护，各部分高度耦合，流程混乱，且每个任务只能指定一个回调函数


###二、事件监听

	f1.on('done1', f2);
	f1.on('done2', f3);
	
	function f1(){
        // f1的任务代码
        f1.trigger('done1');
        f1.trigger('done2');
    }

执行晚f1的代码后触发f1的done事件，调用f2

* 优点：容易理解，可以绑定多个事件，每个事件可以指定多个回调函数，而且可以"去耦合"，有利于实现模块化。

* 缺点：整个程序都要变成事件驱动型，运行流程会变得很不清晰。


###三、publish/subscribe 发布/订阅模式

上一节的"事件"可以理解成"信号"。

1. 假定，存在一个"信号中心"
2. 某个任务执行完成，就向信号中心"发布"（publish）一个信号
3. 其他任务可以向信号中心"订阅"（subscribe）这个信号
4. 当"信号中心"接受一个"发布"信号时，通知相应“订阅者”开始执行任务

又称"观察者模式"。


1. 首先，f2向"信号中心"jQuery订阅"done"信号。
	
	    jQuery.subscribe("done", f2);

2. 然后，f1发布信号：

        function f1(){
                jQuery.publish("done");
        }

        f1执行完成后，向"信号中心"jQuery发布"done"信号，
        从而引发f2的执行。

3. 取消订阅。

        jQuery.unsubscribe("done", f2);

* 总结：这种方法的性质与"事件监听"类似，但是明显优于后者。因为我们可以通过查看"消息中心"，了解存在多少信号、每个信号有多少订阅者，从而监控程序的运行。


###四、Promise对象
Promise对象是CommonJS工作组提出的一种规范，目的是为异步编程提供统一接口。

* 思想：每一个异步任务返回一个Promise对象，该对象有一个then方法，允许指定回调函数。
* 示例：f1的回调函数f2：

	f1().then(f2);
	
	
	
	f1要进行如下改写：

    function f1(){
        var dfd = $.Deferred();
            setTimeout(function () {
                // f1的任务代码
                dfd.resolve();
        }, 0);
        return dfd.promise;
    }


* 优点：

    1. 回调函数变成了链式写法，程序的流程清楚，而且有整套的配套方法，可以实现许多强大的功能。

    2. 指定多个回调函数：

	f1().then(f2).then(f3);

    3. 多种类型的回调：

	f1().then(f2).fail(f3).always();
	
	4. 如果一个任务已经完成，再添加回调函数，该回调函数会立即执行。不用担心是否错过了某个事件或信号
	
* 缺点：不易理解和编写

