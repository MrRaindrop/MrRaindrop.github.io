---
layout: post
title:  "阿里无线前端校招面试总结"
date:   2014-08-04 22:30:02
categories: verbose
tags: [javascript, js, interview, 前端, 面试, 阿里]
author: "Mr.Raindrop"
header-img: "img/home-bg.jpg"
---

下午阿里无线事业部面试归来，趁着记忆还热乎，赶紧记录一笔。
我还想着无线事业部，来面我的会不会是寒冬winter呢？结果真的是winter大神，终于一睹芳容了。不过因为是在帝都嘛，和杭州视频连线，看得见摸不着啊（你敢摸面试官么，摸一下你试试）

废话不多说，winter大神上来直接进入正题。大神的面试风格是引导式的，你觉得你写过的最牛逼的东西是什么？你最拿手的技能是什么？然后就抓住你说的这个点往深了问。
我说了我最牛逼的是实验室项目里用了angular（牛逼在于是老板毫不知情…），其实应该说写了个移动端web框架的，然后winter就开始问我angular相关的问题：

* 我为什么要在这个项目里用angular？
* angular怎么获取一段时间以后返回的结果并显示在UI上？angular里有哪些方法可以实现？我说了$timeout, $q, promise, $broadcast, $on，不过说实话没太理解问题想要我回答的点，于是有点关系的点都扯上去了。
* 假如用worker进行一段时间的运算，怎么获取结果并显示到UI上？我回答的message事件响应中把结果传入当前scope中。然后问我怎么获取到当前的scope，我回答的是angular.element(xx).scope()这个，然后大神哦了一声，说原来你是这个思路（不符合大神预期啊）。然后我接着又补充用$apply方法将这个值放到scope里，不知道大神是否满意。

> 代码应该是这个样子的：

{% highlight javascript %}
    var app = angular.module('testApp', []);

    // in service.js
    app.factory('WorkerService', ['$q', function($q) {
        var worker = new Worker('calculateWorker.js'),
            defer;
        worker.addEventListener('message', function(e) {
            defer.resolve(e.data);
        }, false);
        return {
            calculate : function(data){
                defer = $q.defer();
                worker.postMessage(data);
                return defer.promise;
            }
        };
    });
 
    // in controller.js
    app.controller('WorkerControl', ['WorkerSerivice', function(WorkerSerivice) {
        var testData = 100,
            promise,
            calcResult;    // ng-model=calcResult to 
        promise = WorkerSerivice.calculate(testData);
        promise.then(function(res) {
            angular.element(document.getElementById('calculateResultCt'))
                .scope().$apply('calcResult = ' + res);
        }, function(err) { console.error(err) });
    }]);
{% endhighlight %}

* 我用过angular以后觉得angular和一般的MVC框架有什么不同？

继续问我最熟悉的技能是什么？我说我最熟悉的还是js。大神问我平时学习前端技能的途径？我说，看书，博客，社区等。问我看过哪些书，blabla。然后问我书里讲到的js哪部分我觉得最重要，印象最深刻？我回答原型继承。然后开始问我了：

* 讲讲prototype
* Function的*prototype*是什么？*_proto_*是什么？
* 怎么在原型链上查找一个对象的属性？
* delete一个对象的属性，是delete实例的还是原型prototype的？我回答实例的。问我如果继续delete会 是什么样？这样设计合理不合理？如果是你会怎么设计？我老实回答不知道，然后开始脑洞大开了：如果我设计的话delete会直接把所有的包括原型链上每一级的这个属性都一次性delete掉。大神显然被我（的脑洞）震住了…

> 实际上delete是分次的，每次delete掉查找链上最底层的那个。如果我是设计者，其实可以把delete设计为只delete掉实例的，如果要delete原型链上的属性，可以delete xxx.prototype.yyy这样，否则就抛异常

* 创建一个对象有哪些方式？
* 给一个元素绑定事件有哪几种方式？
* addEventListener的参数有哪些？
* 如果给一个元素绑定了4个事件处理（用addEventListener），其中两个是capture, 两个冒泡，那么触发顺序是什么？我回答了不是底层以及是底层target的情况。这个题比较自信，感觉大神还是比较满意的。

继续问我还有什么印象最深的？我说js闭包。大神开始问我闭包问题：

* 什么是闭包？讲讲。那我就讲吧，blabla，好像也没讲清楚。
* 比如返回一个函数，里面有一个a，返回以后可以拿到这个a，这就是闭包对吧？我当然反驳，这个a不是函数里的，是外面的。
* 怎么在返回的函数里查找属性a？解释了一通作用域的向上查找。
* 假如outer函数被调用了n多次，返回的inner函数的闭包里的一个变量a每次都是同一个还是n次都不一样？回答是同一个。
* 然后继续追问，我要让它们不一样怎么办？我说改为new调用，把a改为this.a，这样每次就是new一个新的对象，对象实例就不一样了。这里明显犯2了，因为接下来大神指出，但是return的不是对象啊。我只能“哦，对”附和。然后补充说，可以返回一个对象，把这个函数作为这个对象的一个属性。
* 大神接着追问，这样的话这个函数里面的this指向哪里？我明白大神是指出我这种实现里面的坑，不过既然往这个方向走，只能硬着头皮走到底了，我说赋值给谁就是指向谁了(忘指出是“迟绑定”了，不过大神应该明白我的意思了…)。既然有坑，为了挽救危局，我必须得把坑填平了：我说可以把这个函数包一层立即执行函数，然后把this作为参数传进去，在函数里面拿这个参数。大神又是“哦，这个思路啊”——明显大神不是要的这个结果，我赶紧补救：还有一种方式是Function.prototype.bind把函数绑定给this. 到此这条路已经走到黑。

> 那么大神#5这道题到底想要听的是哪种方案？我估计有以下几种可能：
> 1. 之前设计的innerFunc里面console.log(a+b); 现在每次在innerFunc里都改变a这个值，比如a = a + b; ?这样每次a都不同了，因为修改了嘛；不过感觉这是另一个层面的“不同”，实际上还是同一个a…
> 2. 改变整个闭包的结构，把a这个变量放到innerFunc里面来，做为innerFunc的局部变量，这样每次返回的innerFunc里面的a自然是不同的；
> 3. 为outerFunc传递一个参数，这个参数作为a的值，这样每次调用outerFunc(a)所得到的innerFunc里面的a自然是不同的，这其实就是一个函数工厂了；
> 4. 返回的闭包函数不保存给变量，而是直接调用，这样每次innerFunc一返回就执行结束了，比如outerFunc()()这样。这种情况下每次a都是不同的，严格来说都不能称之为闭包——因为每次返回的innerFunc并没有被变量引用，这个临时的“闭包”在innerFunc执行完成以后随之立即被垃圾回收了

大神继续问我觉得自己有什么技能可以拿出来炫技？html，css？我说html，css都写过。大神问我熟悉css哪部分，我说布局吧。然后：

* 你最常用哪种布局方式？我说position.
* position有哪些值？各是什么意思？(居然没有追问bfc, 包围盒等等，说好的“顺着一路扯到normal flow、containing block、bfc、margin collapse，base line，writing mode，bidi，这样一路问下去”呢…)
* 除了这些还有哪些布局方式？blabla

然后问我还有什么技能可以拿出来炫技的，或者还有什么想问的？现在回想起来我可能有点太实在了，这种时候就应该把你简历上罗列的那一堆前端技能一股脑子倒出来的，比如模块管理、用过的框架、设计模式、前端自动化、实习经历之类的统统亮出来。谁知道面试官面你之前有没有过目你的简历呢。面winter的整个面试过程中涉及到的知识点基本上都是基础知识（虽然就这些也抛了几次异常了），前后面了快一个钟头，然后我问了winter几个问题就结束了。