###jQuery.Callback
1. jQuery.Callbacks()是在版本1.7中新加入的。
2. 是一个多用途的回调函数列表对象，提供了一种强大的方法来管理回调函数队列。
3. 如果需要控制一系列的函数顺序执行。那么一般就需要一个队列函数来处理这个问题


###核心思想（可触发式队列）
	var Callbacks = {
	      callbacks: [],
	      add: function(fn) {
	        this.callbacks.push(fn);
	      },
	      fire: function() {
	        this.callbacks.forEach(function(fn) {
	          fn();
	        })
	      }
	  }


###WHAT Jquery DO?

1. 添加懒人写法

2. 添加扩展方法


####jQuery.callbakcs可用参数

* once: 保证callback list只能被执行一次 
* memory: 保存最后fire传入的值，并在list增加callback时使用该值执行callback
* unique: 确保同一个callback只能被添加一次
* stopOnFalse:当一个回调返回false时中止队列.


####添加懒人写法
jQuery.Callbacks("once memory") 

使用createOptions参数调整并缓存 

jQuery.Callbacks({once:true,memory:true})

	// String to Object options format cache
	var optionsCache = {};  

	// Convert String-formatted options into Object-formatted ones and store in cache
	function createOptions( options ) {
		var object = optionsCache[ options ] = {};
		jQuery.each( options.match( rnotwhite ) || [], function( _, flag ) {
			object[ flag ] = true;
		});
		return object;
	}


####添加扩展方法

1. 基础方法
    * add   
    * fire 
    * remove
2. 追加：
    * has
    * empty
    * disable (Disable .fire and .add)
    * disabled
    * lock (Disable .fire)
    * locked
3. 实用：
    * fireWith

