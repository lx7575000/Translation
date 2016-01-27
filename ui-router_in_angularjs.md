#UI-router in AngularJS
###2015-03-20

	
**创建一个单页面应用时路由是必不可少，且非常重要的。AngularJS此时提供了个非常好的方法。我们希望导航栏可以像普通站点页面一样，且在跳转时候不需要刷新页面。其中一个方法是使用`ngRoute`方法，我这次学习的是另一种UI-Router。**

###AngularUI Router是什么？
	
`ui-router`是一个路由框架，用于建立AngularJS的<strong>AngularUI team</strong>.与ngRoute不同的是，它是通过此时该页面的状态变化来进行跳转，不同于ngRoute使用URL。

####URL VS States
使用ui-router，你的页面不需要与页面URL绑定。通过这种方法我们可以改变应用的部分模块，甚至不需要改变URL。

		对于ngRoute不好用地方我已经无法吐槽了。。反正就是不好用！！！

###Let's Begin! Sample Application.
	由于ui-router不属于AngularJS的核心体系里面的内容，需要另行下载添加，或者再进行链接加载。

**如下代码：**		
	当我们链接了ui-router，我们将会使用到ui-sref。曾经用ngRoute时添加的href，此时通过它来生成。而且你需要给他一个明确的状态，来应用在你的应用当中。这些东西会在下面的app.js来创建。

想必你也注意到代码中的`<div ui-view></div>`了，可以很容易理解，它就是`ng-view`的替代品。

**index.hmtl**
	
```

<body ng-app="routerApp">
    <nav class="navbar navbar-inverse" role='navigation'>
        <div class='navbar-header'>
            <a class="navbar-brand" ui-sref="#">AngularUI Router</a>
        </div>
        <ul class="nav navbar-nav">
            <li><a ui-sref="home">Home</a></li>
            <li><a ui-sref="about">About</a></li>
        </ul>
    </nav>
    <div class="container">
        <div ui-view></div>
    </div>
</body>

```

接下来我们开始写app.js 中的内容。
	
可以看到，我在模块中进行了依赖注入（DI）,因为ui-router不是核心组件，所以需要声明注入`ui.router`。

在下面的代码中我们创建了.state()并且在其中创建了模板链接。

```

var routerApp = angular.module('routerApp', ['ui.router']);

    routerApp.config(function ($stateProvider, $urlRouterProvider) {
        $urlRouterProvider.otherwise('/home');
        $stateProvider
            .state('home', {
                url: '/home',
                templateUrl: 'partial-home.html'
            })
            .state('about', {
                url: '/about',
                templateUrl: 'partial-about.html'
            });
    });
```			

下面的代码是home页面中的具体代码，我们可以看到。
如果我们想要连接**nest view**,我们应该使用点操作符。`ui-sref=".list"`和`ui-sref=".paragraph"`如果我们注册了这个，我们后面会注入到我们创建的`<div ui-view></div>`

<pre>
<div class="jumbotron text-center">
    <h1>The Homey Page</h1>
    <p>This page demonstrates <span class="text-danger">nested</span> views.</p>  
    <a ui-sref=".list" class="btn btn-primary">List</a>
    <a ui-sref=".paragraph" class="btn btn-danger">Paragraph</a>
</div>
<div ui-view></div>
</pre>

	想要创建，只需要将上面的app.js中的内容稍微更改一点。

**栗子：**		

```
.state('home.list', {		
	url:'/list',		
	templateUrl: 'partial-hone-list.html'	
});

```

**接下来我们进行about页面的编写，编写这个页面是为了说明在一个单页面中可以存在两个不同的`ui-view`,他们将整个页面平分为两个部分。代码如下：**

	<div class="jumbotron text-center">
    <h1>The About Page</h1>
    <p>This page demonstrates <span class="text-danger">multiple</span> and <span class="text-danger">named</span> views.</p>
	</div>
	
	<div class="row">

    <!-- COLUMN ONE NAMED VIEW -->
    <div class="col-sm-6">
        <div ui-view="columnOne"></div>
        	<!--Look At Here!!!!!!!!-->        
    </div>
    
    <!-- COLUMN TWO NAMED VIEW -->
    <div class="col-sm-6">
        	<div ui-view="columnTwo"></div>
        	<!--Look At Here!!!!!!!!-->
    	</div>
	</div>    

注意到上面的代码中，有两个`ui-view`，分别进行了命名为**columnOne**和**columnTwo**.这么做有其优点，可以讲一些相关且内容较少的元素结合在同一页面显示，即简洁明了又可以减少用户频繁操作的烦恼。
当然这么做一定还是要在app.js中进行定义的：		
**栗子来喽：**

```
.state('about', {
        url: '/about',
        views: {
            // the main template will be placed here (relatively named)
            '': { templateUrl: 'partial-about.html' },
            // the child views will be defined here (absolutely named)
            'columnOne@about': { template: 'Look I am a column!' },
            // for column two, we'll define a separate controller 
            'columnTwo@about': { 
                templateUrl: 'table-data.html',
                controller: 'scotchController'
            }
```
由上面的代码可以发现ui-router在此处的命名规则是**子界面名称@主界面**。此处练习项目里，明显该页面的主页面是about页面，columnOne和columnTwo是其下属页面。

**ui-router的优点：**	

	You can see how easy it is to change different parts of our application based on the state. We didn’t have to do any sort of work with ngInclude, ngShow, ngHide, or ngIf. This keeps our view files cleaner since all the work is in our app.js.
	
	
<a href='https://scotch.io/tutorials/angular-routing-using-ui-router'>原文学习</a>		
<a href='https://github.com/angular-ui/ui-router/wiki/Multiple-Named-Views#view-names---relative-vs-absolute-names'>有关多视图的命名介绍</a>