---
layout: post
title:  "抽象类接口"
date:   2013-12-06 18:25:38 +0800
---
<img src="/images/fulls/03.jpg" class="fit image">


抽象类：

	1. 无法实例化，类前加abstract，此类就成了抽象类，无法实例化。
	2. 抽象方法是不能有方法体的。
	3. 有抽象方法，此类必须是抽象类，但是抽象类内未必都是或者都有抽象方法。但是即便全是具体方法，只要类是抽象的，依旧不能实例化。

	abstract class Person{
		public abstract function showMan();
		public abstract function showWoman();
		Public function showchild(){
			Echo “一群小孩子”;
		}
	}



 接口概念：
 使用接口（interface），你可以指定某个类必须实现哪些方法，但不需要定义这些方法的具体内容。

我们可以通过interface来定义一个接口，就像定义一个标准的类一样，但其中定义所有的方法都是空的。接口中定义的所有方法都必须是public，这是接口的特性。 

注意：

	1. 接口的方法默认是抽象的，之前不必在加abstract，而且方法必须是			public公开的。
	2. 接口是一堆方法的说明，不能有属性，但是常量可以用。
	3. 要实现一个接口，可以使用implements操作符。类中必须实现接口中定义		的所有方法，否则会	  报一个fatal错误。 
	4. 实现多个接口，可以用逗号来分隔多个接口的名称，接口中的方法不能有		重名。
	5. 接口也可以继承，使用extends操作符。如果有接口继承了父接口，那么		父接口中的方法也必须全部实现。
	6. 接口可以继承多个接口，这个和类不同。
	7. 接口就像零件，而用多种零件可以组合出一种新的东西。

代码实例：

	interface animal{
		public function eat();
	}
	interface bird{
		public function fly();
	}
	interface monkey extends animal{
		public function run();
	}
	class Human implements monkey,bird{
		public function eat(){
			echo "eat";
		}
		public function run(){
			echo "run";
		}
		public function fly(){
			echo "fly";
		}
	}
	$lisi = new Human();
	$lisi->eat();


4. 使用场景：
比如一社交网站，关于用户的处理是核心应用。应用如下：
登录
退出
写信
招呼
更新心情
吃饭
捣乱
示爱

这么多方法，都是用户的方法，自然可以写一个user类，全包装起来，但是，分析用户一次性使用不了这么多方法，会造成浪费，而且不能达到灵活使用的目的：

用户信息：（登录，退出，写信，看信，招呼，更新心情）
用户娱乐：（骂人，捣乱，示爱）

开发网站的时候，分析出这么多类，这么多方法，我们都晕了。而且作为调用 	者，我不需要了解你的用户信息类，用户娱乐类，我就可以知道如何调用者两	个类。
因为：这两个类都要实现上述的接口，通过这个接口就可以规范开发。


	interface UserBase{
		public function login($u,$p);
		public function logout();
	}
	interface UserMsg{
		public function writeMsg();
		public function readMsg();
	}
	interface UserFun{
		public function spit($to);
		public function showLove($to);
	}