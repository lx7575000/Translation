#React-Router V4 简单实现
>听说V4终于发布了，本文是在阅读RRV4时做的一点小总结

在此对React-Router4.0版本中重要的几个组件做简单的实现。
##Match组件
`Match`组件的作用很简单，就是基于`url`地址来判断是否渲染对应组件。

###使用方式：
```JavaScript
<Match pattern='/test'component={Component} />
```
###基本原理实现：
```JavaScript
import React from 'react';

const Match = ({ pattern, component: Component }) => {
  const pathname = window.location.pathname;
  if (pathname.match(pattern)) {
    // 此处使用match方法对url地址进行解析是远远不够的！
    return (
      <Component />
    );
  } else {
    return null;
  }
}
```
`Match`可以提取出URL中所有动态参数，然后将他们以传入到对应的组件。

##Link组件
`Link`组件其实就是对`<a>`标签的**封装**。它会禁止掉浏览器的默认操作，通过`history`的API更新浏览器地址。
###使用方法

```JavaScript
<Link to="/test" > To Test</Link>
```

###基本实现原理
```JavaScript
import createHistory from 'history/createBroserHistory';
const history = createHistory();

// Link组件因为不需要维持自身数据，因此我们在此可以使用无状态(stateless)函数
const Link = ({ to, children }) => {
    <a
      onClick={(e) => 
        e.preventDefault();
        history.push(to)
      }
      href={to}
    >
      {children}
    </a>
  }
```
如上面代码所示，当我们点击`a`标签的时候。首先会禁止浏览器的默认行为，然后会调用`history.push`的API接口更新URL的地址。
>你可能会有疑问，既然通过点击就能更新URL地址了，那我们还设置`a`标签的`href`属性有何用呢？事实上的确没什么用处，但本着体验最优这一原则我们仍需要设定该属性，当用户将鼠标悬停在标签上时能够预览链接地址和右击复制链接。

React-Router的作用是在URL更改的时候，显示对应的URL的SPA组件。因此当我们点击`Link`组件更新URL时，需要同时更新到对应的组件。`history`为我们提供了相应的监听方法来监听URL变化。
```JavaScipt
class App extends React.Component {
  componentDidMount() {
    history.listen(() => this.forceUPdate())
  }
  render(){
    return (<div>App</div>);
  }
}
```
###`Link`晋级版本
我们希望能够根据给该组件传入更多参数。例如:
1. 当前URL为激活状态时，希望能够显示特别的样式
2. 如同普通`a`标签一样，居有`target`属性能够在另一个新页面来显示.
3. 传入特别的参数,给对应URL下组件使用，如`query`。

```JavaScript
function resolveToLocation(to, router) {
  return 
    typeof to === 'function' ? to(router.location) : to
}
function isEmptyObject(object) {
  for (const p in object)
    if (Object.prototype.hasOwnProperty.call(object, p))
      return false

  return true
}
class Link extends React.Component {
  //这次要根据React-Router写个相对简单，但功能更全的版本
  handleClick(e) {
    // 如果有自定义的点击处理，就用该方法
    if (this.props.onClick) {
      this.props.onClick();
    }
    // 如果设定了target属性，就使用浏览器默认方式来处理点击
    if (this.props.target) {
      return;
    }
    e.preventDefault();
    history.push(resolveToLocation(to));
  }
  render() {
    const { to, activeClassName, activeStyle, onlyActiveOnIndex, ...props } = this.props;
    
    const toLocation = resolveToLocation(to);
    props.href = toLocation;
    
    // 以下为React-router的代码，根据当前URL选择样式
     if (activeClassName || (activeStyle != null && !isEmptyObject(activeStyle))) {
        if (router.isActive(toLocation, onlyActiveOnIndex)) {
          if (activeClassName) {
            if (props.className) {
              props.className += ` ${activeClassName}`
            } else {
              props.className = activeClassName
            }
          }
          if (activeStyle)
            props.style = { ...props.style, ...activeStyle }
        }
      }
    return <a {...props} href={href}  onClick={this.handleClick} />
  }
}
Link.propTypes = {
  to: oneOfType([string, object, func]),
  query: object,
  state: object,
  activeStyle: object,
  activeClassName: string,
  onlyActiveOnIndex: bool.isRequired,
  onClick: func,
  target: string
}
Link.defaultProps = {
  onlyActiveOnIndex: false,
  style: {}
}
```

##Redirect
`Redirect`组件和`Link`组件类似。和它不同之处在于`Redirect`不需要点击操作，直接就会更新URL地址。

###使用方式
```JavaScript
<Redirect to="/test" />
```

### 基本实现原理
```JavaScript
class Redirect extends React.Component {
  static contextTypes = {
    history: React.PropTypes.object,
  };
  componentDidMount() {
    // 根本不需要DOM，只要你加载该组件，就立即进行跳转
    const history = this.context.history;
    const to = this.props.to;
    history.push(to);
  }
  render() {
    return null;
  }
}
```
以上只是简单的实现，V4版本的`Redirect`组件会判断代码是不是在服务端渲染，并根据结果不同来选择在组件生命周期的哪个阶段进行URL更新操作(isServerRender ? componentWillMount : componentDidMount)。
###晋级功能
```JavaScript
class Redirect extends React.Component {
  ...
  // 服务端渲染的话，连Redirect组件都不需要渲染出来就可以跳转。
  componentWillMount() {
    this.isServerRender = typeof window !== 'object';
    if (this.isServerRender) {
      this.perform();
    }
  }
  // 浏览器端渲染则需要Redirect组件渲染之后才能进行跳转。
  componentDidMount() {
    if (!this.isServerRender) {
      this.perform();
    }
  }
  perform() {
    const { history } = this.context;
    const { push, to } = this.props;
    // push是个bool值，用于选择使用history的哪个方法来跳转链接
    if (push) {
      history.push(to);
    } else {
      history.replace(to);
    }
  }
}
Redirect.defaultProps = {
  push: false,
}
```

##Router
`react-router`中的`Router`组件本质就是个高阶组件，用于包裹所有的页面组件(其实在react-router v4中，`BrowserRouter`才是最外层组件)，并为所有`react-router`提供全局参数`history`。因此`Router`很简单，也很容易实现。
```JavaScript
class Router extends React.Component {
  static propTypes = {
    history: ProTypes.object.isRequired,
    children: PropTypes.node
  };
  
  static childrenContextTypes = {
    history: PropTypes.object.isRequired
  };
  
  // 通过context给组件内所有元素提供共享数据: history
  getChildrenContext(){
    // 传入给所有react-router组件使用的history参数
    return { history: this.props.history };
  }
  
  render() {
    const { children } = this.props;
    return children ? React.Children.only(children) : null;
  };
}
```

#React-Router V4
###SeverRouter (服务端渲染时使用)
`createLocation`方法很简单，就是拆解URL地址，并将其分解为`pathname`,`search`,`hash`三个部分。并以数组的形式返回。
```JavaScript
// 例子url : www.webei.com/#/admin/dashboard/setting?username=liuxin&token=noface;
const createLocation = (url) => {
  let pathname = url || '/'
  let search = ''
  let hash = ''

  const hashIndex = pathname.indexOf('#')
    if (hashIndex !== -1) {
      hash = pathname.substr(hashIndex)
      pathname = pathname.substr(0, hashIndex)
    }
  
    const searchIndex = pathname.indexOf('?')
    if (searchIndex !== -1) {
      search = pathname.substr(searchIndex)
      pathname = pathname.substr(0, searchIndex)
    }
  
    return {
      pathname,
      search: search === '?' ? '' : search,
      hash: hash === '#' ? '' : hash
    }
    /* 最后返回 {
        pathname: 'www.webei.com/',
        search: '',
        hash: '#/admin/dashboard/setting?username=liuxin&token=noface',
      
      }
    
    */
  }
```

###NavLink 
`NavLink`组件是对`Route`组件的封装。目的是根据根据URL来**Active**对应的`NavLink`组件，为其添加**activeClassName**和**activeStyle**。

###Route
在React-router V4版本中，Route组件存在于**./Core.js**文件中。它是对`createRouteElement`方法的封装。

```JavaScript
const createRouteElement = ({ component, render, children, ...props} => (
  // createRouteElement 的执行优先级是
  component ? 
    (props.match ? React.createElement(component, props) : null)
     : render ? (
      props.match ? 
        render(props) : null 
      ) : children ? (
        typeof children === 'function' ? 
          children(props) : React.Children.only(children)
        ) : ( null )
));

// 上面的版本是不是很眼晕？下面这个版本的你大概能理解的更清楚。
const createRouteElement = ({ component, render, children, ...props} => {
  /* 
    参数优先级 component > render > children
    要是同时有多个参数，则优先执行权重更高的方法。
  */
    // component 要求是方法
    if (component && props.match) {
      return React.createElement(component, props);
    } else {
      return null;
    }
    // render 要求是方法
    if (render && props.match) {
      return render(props);
    } else {
      return null;
    }
    // children 可以使方法也可以是DOM元素
    if (children && typeof children === 'function') {
      return children(props);
    } else {
      return React.Children.only(children)
    }
    return null;
}

const Route = ({ match, history, path, exact, ...props}) => (
  createRouteElement({
    ...props,
    match: match || matchPath(history.location.pathname, path, exact),
    history
  })
);

Route.propTypes = {
  match: PropTypes.object, // private, from <Switch>
  history: PropTypes.object.isRequired, // 必备属性
  path: PropTypes.string,
  exact: PropTypes.bool,
  component: PropTypes.func,
  // TODO: Warn when used with other render props
  render: PropTypes.func, 
  // TODO: Warn when used with other render props
  children: PropTypes.oneOfType([
  // TODO: Warn when used with other render props
    PropTypes.func,
    PropTypes.node
  ])
}
```

##Router
根据history参数的设定选择使用不同封装的`Router`组件

###MemoryRouter
> 源代码注释
The public API for a <Router> that stores location in memory.

###BrowserRouter
> 源代码注释
The public API for a <Router> that uses HTML5 history.

###HashRouter
> 源代码注释
The public API for a <Router> that uses window.location.hash.

##withHistory
相当重要的组件，没有它就没有根据URL变化而产生的重新渲染了。
```JavaScript
/**
 * A higher-order component that starts listening for location
 * changes (calls `history.listen`) and re-renders the component
 * each time it does. Also, passes `context.history` as a prop.
 */
const withHistory = (component) => {
  return class extends React.Component {
    static displayName = `withHistory(${component.displayName || component.name})`

    static contextTypes = {
      history: PropTypes.shape({
        listen: PropTypes.func.isRequired
      }).isRequired
    }

    componentWillMount() {
      // Do this here so we can catch actions in componentDidMount (e.g. <Redirect>).
      this.unlisten = this.context.history.listen(() => this.forceUpdate())
    }

    componentWillUnmount() {
      this.unlisten()
    }

    render() {
      return React.createElement(component, {
        ...this.props,
        history: this.context.history
      })
    }
  }
}

```



