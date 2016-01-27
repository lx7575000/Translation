#AngularJS -- UI-Router
####2015-03-12

	
ui-router 的工作原理非常类似于 Angular 的路由控制器，**但它只关注状态。**	
**在应用程序的整个用户界面和导航中，一个状态对应于一个页面位置**	
通过定义controller、template和view等属性，来定义指定位置的用户界面和界面行为
通过嵌套的方式来解决页面中的一些重复出现的部位

####嵌套路由
	ui-router的目的是为了实现在一个页面实现曾经需要跳转的其他页面才能完成的操作。类似于使用 a 标签所做的。

**例：**	
`<a href = 'www.baidu.com' target='_blank> 在新标签页打开链接</a>`

<h6>不同于上面代码的那种方式，ui-router是希望在一个页面下能够嵌套其他页面的内容。可以理解为类似于AngularJS的`router`所做的那样</h6>

<div class='menu' align='center'>
<span style='width=100px'>  <a href='www.baidu.com'>标签页1</a></span>
<span style='width=100px'> 	 <a href='www.baidu.com'>标签页2</a></span>
<span style='width=100px'>  <a href='www.baidu.com'>标签页3</a></span>
</div>
<div>
	<p align='center'>根据点击不同的标签，根据状态的改变来判断应该显示哪个标签页内代码块的内容</p>
</div>

####上代码！！
<pre>
	var myApp = angular.module('myApp', ['ui.router']);
</pre>

先创建AngularJS初始化， 此处重点只有一个。 Notice the `ui.router`这个是重点，必须要加入依赖。同时还需要加载 **angular-ui-router.min.js**这个文件。（否则依赖个屁！）

<pre>
	myApp.config(function($stateProvider, $urlRouteProvider){
		
		$urlRouteProvider.when('', '/pageTab');
		
		//此处是为了创立默认显示的页面,
		//当路由引擎没有页面可以匹配的时候就会跳转到pageTab.html页面。
		//可以将其理解为switch case 中的 default
		
		$stateProvider
        .state("PageTab", {
            url: "/PageTab",
            templateUrl: "PageTab.html"
        })
        //这一行定义了会在main.html页面第一个显示出来的状态，
        //作为页面被加载好以后第一个被使用的路由
        .state("PageTab.Page1", {
            url:"/Page1",
            templateUrl: "Page-1.html"
        })
        //现在，就由这一行来定义页面, 
        //但是等一等，这里有点不同，我们之前为上面的状态名称加上了前缀，并且使用点“.“号进行了分隔. 
        //这里很关键，它会告诉路由引擎我们在这里定义的是子页面/嵌入页面/嵌入
        //(sub page / nested page / nested)状态的page/route.
	})
</pre>

**后面设置各个页面的代码块就好了。**		
在主页面添加如下代码，然后就可以能够得到基础效果了。
######下面为pageTab页的代码段
```
<pre>
<div>
     <div>
         `<span style="width:100px" ui-sref=".Page1"><a href="">Page-1</a></span>`
         `<span style="width:100px" ui-sref=".Page2"><a href="">Page-2</a></span>`
         `<span style="width:100px" ui-sref=".Page3"><a href="">Page-3</a></span>`
     </div>
     <div>
          `<div ui-view=""/>`
     </div>
</div>
</pre>
```

下面为主页面，main.html的代码


<html xmlns="http://www.w3.org/1999/xhtml">
<head>
    <title></title>
    <script src="Scripts/angular.js"></script>
    <script src="Scripts/angular-ui-router.js"></script>
    <script src="App.js"></script>
 
</head>
<body data-ng-app="myApp">
    <h1>AngularJS Home Page (Ui-router Demonstration)</h1>
    <div data-ui-view=""></div> //此处是重点
</body>
<html>
```

参考： <span> <a href='http://www.oschina.net/translate/angularjs-ui-router-nested-routes'>AngularJS ui-router 嵌套路由</a></span>