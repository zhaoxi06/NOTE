[React Router 4 简易入门](https://segmentfault.com/a/1190000010174260)<br>
[React-router4简约教程](http://react-china.org/t/react-router4/15843)<br>

# React Router
包含三个包`react-router`、`react-router-dom`、`react-router-native`.<br>

* `react-router`提供核心API，但没有提供DOM操作进行跳转的API
* `react-router-dom`可以通过DOM事件控制路由。在开发过程中，更常使用这个。

## 1.路由器(Router)
>路由器(Router)就是一个React组件。它本身是个容器，真正的路由要通过`Route`组件定义。

路由器(Router)有两种类型：<br>

* `<BrowserRouter>`需要服务器管理动态请求时用
* `<HashRouter>`用于静态网站

## 2.路由(Route)
* 定义了URL路径与组件的对应关系。
* Route中常用的属性有3个：exact、path和component.
* exact：控制匹配到path所在路径后不会再继续向下匹配
* path：标识路由的路径
* component：表示路径对应的组件
* 通常会使用`<Switch>`组件来包裹一组路由。`<Switch>`会遍历自身的子元素，并对第一个匹配当前路径的元素进行渲染。
* 路由可以嵌套
```
<Switch>
    <Route exact path='/' component={Home}/>
    <Route path='/roster' component={Roster}/>
    <Route path='/schedule' component={Schedule}/>
</Switch>
```

## 3.`<Route>`渲染的方式有三种
* component
* render
* children
```
<Route path='/page' component={Page} />

const extraProps = { color: 'red' }
<Route path='/page' render={(props) => (
  <Page {...props} data={extraProps}/>
)}/>

<Route path='/page' children={(props) => (
  props.match ? <Page {...props}/> : <EmptyPage {...props}/>
)}/>
```
* component和render更经常使用

## 4.match
* path路径中还可以添加参数
* 通过props.match.params对象来获取参数
```
<Route path='/page/:id' component={Page} />

//使用
<Link to={`/page/12`}>{props.match.params.id}</Link>
```

## 5.Link
>实现页面间切换，并且能够避免页面被重新加载。

```
<nav>
    <ul>
        <li><Link to='/'>Home</Link></li>
        <li><Link to='/roster'>Roster</Link></li>
        <li><Link to='/schedule'>Schedule</Link></li>
    </ul>
</nav>

//to属性的值是location对象
<Link to={{ pathname: '/roster/7' }}>Player #7</Link>
```

* 通过to属性描述需要定位的页面。
* to属性的值可以是location对象（包含pathname，search，hash与state属性），也可以是字符串。如果是字符串，将会被转换成location对象。

## 6.NavLink
>与Link类似，NavLink可以为当前选择的路由设置类名、样式以及回调函数等。