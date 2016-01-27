##AngularJS Form validation
####2015-03-21

####各种属性介绍
* required：	
	用于判断是否为空		
* ng-maxlength:		
	验证内容最大长度
* ng-minlength：	
	验证内容的最小长度
* 表单名.$valid:		
	该属性用于获取表单的状态，只有所有的表单内容满足需求时才为true，否则为false
* ng-disabled：
	判断按钮属性是否禁用，值为true时，该按钮禁用。
* type属性	
	有'email','number'...等等属性，与html5基本相同。
* novalidate属性：	
    与用于失效HTML5所自带的属性，由我们重新设定规则。
    
标签中的输入格式

```
<input
       ng-model="{ string }"
       name="{ string }"
       required
       ng-required="{ boolean }"
       ng-minlength="{ number }"
       ng-maxlength="{ number }"
       ng-pattern="{ string }"
       ng-change="{ string }">
    </input>
```


<table summary = "表格简介文本">
  			<tbody>  		
  			  	<caption>AngularJS Form 属性</caption>	    
  			<tr>
  				<th>Property</th>
  				<th>Class</th>
  				<th>Description</th>
  			</tr>
  			<tr>
  				<td>$valid</td>
  				<td>ng-valid</td>
  				<td>Boolean类型， 用于说明当前是否满足我们所设定的规则</td>
  			</tr>
  			<tr>
  				<td>$invalid</td>
  				<td>ng-invalid</td>
  				<td>同上</td>
  			</tr>
  			<tr>
  				<td>$pristine</td>
  				<td>ng-pristine</td>
  				<td>当表单或者输入框尚未被使用过时为true</td>
  			</tr>
  			<tr>
  				<td>$dirty</td>
  				<td>ng-dirty</td>
  				<td>当表单或者输入框已经被使用过时为true</td>
  			</tr>
  			</tbody>
</table>

下述属性的样式CSS可以由我们来设定，不一一说明反正看得懂：	

```
	.ng-valid       {  }
    .ng-invalid     {  }
    .ng-pristine    {  }
    .ng-dirty       {  }

    /* really specific css rules applied by angular */
    .ng-invalid-required        {  }
    .ng-invalid-minlength       {  }
    .ng-valid-max-length        {  }
```

同样，可以通过`ng-class`表达式的形式来根据不同条件状态来设定不同的CSS样式：	
`ng-class="{ <class-you-want> : <expression to be evaluated > }"`

#####如何使用Form的属性
* To access the form: `<form name>.<angular property>`
* To access an input: `<form name>.<input name>.<angular property>`

####ng-submit
不同于`ng-click`，`ng-submit`是针对用于form中的一个命令。

```
<form ng-submit='clickButton()' novalidate>
	<button type='submit'>ClickButton</button>
</form>

```

**如上栗子：**		
`ng-submit`存在于form标签当中，当**form标签**内部的**button标签**（类型为‘submit’）即按钮被点击的时候。		
**'submit'**消息被传递给**form**。**form**接收到后，会调用`ng-submit`中的函数。然后就**balabala....**函数方法随你弄。{{{(>_<)}}}

####ng-form
`ng-form`的作用是独立每一个`ng-repeat`生成的单独作用域。