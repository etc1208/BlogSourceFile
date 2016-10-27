---
title: React Router
date: 2016-10-27 13:37:15
tags: [React-Router]
---

参考：[React Router](http://mp.weixin.qq.com/s?__biz=MzA4NjE3MDg4OQ==&mid=2650963771&idx=1&sn=f42da0e4f617b1b7b0a68e83e804476d&chksm=843a135db34d9a4bcf59759fef42ea5f1115f42a6610227758379513c032df1c75af0888aa24&mpshare=1&scene=1&srcid=1027rBLnvcpvOvrbAE010AcW#rd)


![React Router](http://static.open-open.com/lib/uploadImg/20160525/20160525205833_809.png)

<!--more-->

如果以前你曾经用过任何前端路由器，那么应该已经熟悉了很多概念。但是 React Router 与我以前曾经用过的任何其它路由器都不同，它用 JSX，这玩意开始看起来会有点奇怪。

作为入门，如下是如何渲染一个组件的示例代码：

	var Home = React.createClass({
	  render: function() {
	    return (<h1>Welcome to the Home Page</h1>);
	  }
	});

	ReactDOM.render((
	  <Home />
	), document.getElementById('root'));
如下是 Home 组件用 React Router 是如何渲染的：

	...
	
	ReactDOM.render((
	  <Router>
	    <Route path="/" component={Home} />
	  </Router>
	), document.getElementById('root'));
注意，这里 <Router> 和 <Route> 是两个不同的东西。从技术上讲，二者都是 React 组件，但是它们自己实际上都不会创建 DOM。看起来好像 <Router> 本身被渲染为 'root'，但是实际上我们只是定义应用程序如何工作的规则。继续下去的话，你会经常看到这个概念：组件有时候并非为自己创建为 DOM 而存在，而是协调创建 DOM 的其它组件。

在本例中，<Route> 定义了一个规则：访问主页（/）的地方，会渲染 Home 组件为 'root'。

### 多个 Route

前面的示例中，只有一个路由，这很简单。它并没有给我们更多的价值，因为我们不用路由器就可以渲染 Home 组件。React Router 的强大来自于：我们可以使用多个路由来定义根据当前活动的路径渲染哪个组件。

	ReactDOM.render((
	  <Router>
	    <Route path="/" component={Home} />
	    <Route path="/users" component={Users} />
	    <Route path="/widgets" component={Widgets} />
	  </Router>
	), document.getElementById('root'));
当 路径（path）匹配 URL 时，每个 <Route> 会渲染各自的组件。这三个组件中只有一个会在任何给定时间渲染到 'root' 中。使用这种策略，我们一次就把路由器挂载到 DOM 的 'root' 上，然后路由器就根据路径改变切换组件的进出。

还要指出的是，路由器不用向服务器发起请求就会切换路由，所以可以把每个组件假想为一个完整的新页面。

### 可重用的布局

我们现在看到的是单页应用程序最寒碜的开始。但是，它依然不能解决实际的问题。确实，你可以创建这三个组件来组成完整的 HTML 页面，但是要代码重用该怎么办？机会是，这三个组件共享相同的部件，比如 header 和 sidebar，所以我们如何防止每个组件中的 HTML 重复呢？

假设我们正在创建一个由如下界面原型组成的 Web 应用程序：

一个简单的网站原型

当你开始思考如何将这个原型分拆成可重用的部分时候，最后你可能会有如下的分拆：

将一个简单的 Web 原型分成多个部分

考虑在嵌套组件和布局方面会让我们创建可重用的部分。

突然，设计部门让你知道应用程序需要需要一个搜索部件页，该页由搜索用户页面组成。User List 和 Widget List 都需要搜索页面有相同的外观，那么现在将 Search Layout作为一个单独的组件就更有意义：

搜索组件取代搜索用户页，但是父界面部分不变

Search Layout 现在可以是所有搜索页面类型的父模板。并且在一些页面需要 Search Layout 的同时，其他的页面可以直接使用 Main Layout ，而不需要 Search Layout:

解耦了的布局

这是一种常见的策略，如果用过任何模板系统，你可能也做过很相似的事情。现在我们开始写 HTML。开始我们只写静态的 HTML，不用考虑 JavaScript：

	<div id="root">
	
	  <!-- Main Layout -->
	  <div class="app">
	    <header class="primary-header"><header>
	    <aside class="primary-aside"></aside>
	    <main>
	
	      <!-- Search Layout -->
	      <div class="search">
	        <header class="search-header"></header>
	        <div class="results">
	
	          <!-- User List -->
	          <ul class="user-list">
	            <li>Dan</li>
	            <li>Ryan</li>
	            <li>Michael</li>
	          </ul>
	
	        </div>
	        <div class="search-footer pagination"></div>
	      </div>
	
	    </main>
	  </div>
	</div>
记住，’root’元素总是存在的，因为它是 JavaScript 启动前初始 HTML Body 唯一的元素。这个 'root' 是恰当的，因为整个 React 应用程序都会挂载到它上面。但是没有恰当的名称或者惯例来称呼它，所以我选择用 'root'，而且会在整个示例中继续使用它。只是要注意：直接挂载到 <body> 元素是绝对不提倡的

创建完静态 HTML 之后，把它转换为 React 组件：

	var MainLayout = React.createClass({
	  render: function() {
	    // Note the `className` rather than `class`
	    // `class` is a reserved word in JavaScript, so JSX uses `className`
	    // Ultimately, it will render with a `class` in the DOM
	    return (
	      <div className="app">
	        <header className="primary-header"><header>
	        <aside className="primary-aside"></aside>
	        <main>
	          {this.props.children}
	        </main>
	      </div>
	    );
	  }
	});
	
	var SearchLayout = React.createClass({
	  render: function() {
	    return (
	      <div className="search">
	        <header className="search-header"></header>
	        <div className="results">
	          {this.props.children}
	        </div>
	        <div className="search-footer pagination"></div>
	      </div>
	    );
	  }
	});
	
	var UserList = React.createClass({
	  render: function() {
	    return (
	      <ul className="user-list">
	        <li>Dan</li>
	        <li>Ryan</li>
	        <li>Michael</li>
	      </ul>
	    );
	  }
	});
不要被我称为“布局”和“组件”这事上过于分心。这三个都是 React 组件。我称其中两个为“布局”，只是因为这是它们执行的职责。

最终我们会用嵌套的 route 将 UserList 放到 SearchLayout 中去，然后将 SearchLayout 放到 MainLayout 中去。但是首先，注意到当 UserList 被放到它的父组件 SearchLayout 中时，父组件会用 this.props.children 来判断 UserList 的位置。所有的组件都有 this.props.children 作为一个 prop，但是只有组件是嵌套的时，父组件才会被 React 自动填充这个 prop。对于没有父组件的组件，this.props.children 将是 null。

### 嵌套的 Route

那么，我们如何才能让这些组件嵌套呢？当我们嵌套 route 时，router 就为我们做了：

	ReactDOM.render((
	  <Router>
	    <Route component={MainLayout}>
	      <Route component={SearchLayout}>
	        <Route path="users" component={UserList} />
	      </Route> 
	    </Route>
	  </Router>
	), document.getElementById('root'));
组件将会与路由器嵌套它的 route 一样嵌套。当用户访问 /users 路由时，React Reater 会将 userList 组件放在 SearchLayout 里面，然后二者都放在 MainLayout 里面。访问 /users 的最终结果是三个嵌套的组件放在 ‘根‘ 里面。

注意，为简化起见，前面我们还没有为用户访问主页路径（/）或者想搜索部件时设置规则。现在我们可以把它们放进来：

	ReactDOM.render((
	  <Router>
	    <Route component={MainLayout}>
	      <Route path="/" component={Home} />
	      <Route component={SearchLayout}>
	        <Route path="users" component={UserList} />
	        <Route path="widgets" component={WidgetList} />
	      </Route> 
	    </Route>
	  </Router>
	), document.getElementById('root'));
你可能已经注意到了，JSX 在某种程度上是遵循 XML 规则的，Route 组件要么用 <Route /> 一个标记写，要么是用 <Route>...</Route> 两个标记写。所有的 JSX 都是这样的，包括自定义组件和普通的 DOM 节点。比如，<div /> 是有效的 JSX，并且在渲染时会被渲染为 <div></div>。

为简洁起见，假设 WidgetList 与 UserList 相似。

因为现在 <Route component={SearchLayout}> 有两个路径了，用户就可以访问 /users 或者 /widgets ，对应的 <Route> 会加载各自的组件到 SearchLayout组件。

同时，注意到，Home 组件将会被直接放到 MainLayout 里面，而没有包含 SearchLayout，这是因为 <Route> 被嵌套的方式。你可能会想到通过重新安排 route，可以重新安排布局和组件的嵌套。

### IndexRoutes

React Route 是很富有表现力的，并且经常有多种方法做相同的事情。例如，我们也可以像如下这样写上面的路由器：

	ReactDOM.render((
	  <Router>
	    <Route path="/" component={MainLayout}>
	      <IndexRoute component={Home} />
	      <Route component={SearchLayout}>
	        <Route path="users" component={UserList} />
	        <Route path="widgets" component={WidgetList} />
	      </Route> 
	    </Route>
	  </Router>
	), document.getElementById('root'));
尽管这跟前面的看起来不同，但是二者都是以相同的方式工作的。

### 可选的 Route 属性

有时，<Route> 没有 path 属性，但是有 component 属性，就像上面 SearchLayout 中的路径。有时，又需要 <Route> 有 path 属性，但是没有 component 属性。为什么会这样，我们来看一个示例：

	<Route path="product/settings" component={ProductSettings} /><Route path="product/inventory" component={ProductInventory} /><Route path="product/orders" component={ProductOrders} />
这里 path 的 /product 部分是重复的。我们可以将所有三个路径封装到一个新的 <Route> 中，从而去掉重复：

	<Route path="product">
	  <Route path="settings" component={ProductSettings} />
	  <Route path="inventory" component={ProductInventory} />
	  <Route path="orders" component={ProductOrders} /></Route>
这里，React Router 再次展示了它的表现力。小测验：你注意到这两种解决方案的问题了么？当用户访问 /product 路径时，没有定义规则。

为修正这个问题，我们可以添加一个 IndexRoute:

	<Route path="product">
	  <IndexRoute component={ProductProfile} />
	  <Route path="settings" component={ProductSettings} />
	  <Route path="inventory" component={ProductInventory} />
	  <Route path="orders" component={ProductOrders} /></Route>
	  
### 用 < Link > 而不要用 < a >

当为路径创建锚点时，必须用 < link to="" > 而不是 < a href="" >。但是不要担心，当使用 <link> 组件时，React Router 最终会在 DOM 中给一个普通的锚点。使用 <Link> 对于 React Router 发挥它的路由魔力来说是必须的。

下面我们给 MainLayout 添加点链接（锚点）：

	var MainLayout = React.createClass({
	  render: function() {
	    return (
	      <div className="app">
	        <header className="primary-header"></header>
	        <aside className="primary-aside">
	          <ul>
	            <li><Link to="/">Home</Link></li>
	            <li><Link to="/users">Users</Link></li>
	            <li><Link to="/widgets">Widgets</Link></li>
	          </ul>
	        </aside>
	        <main>
	          {this.props.children}
	        </main>
	      </div>
	    );
	  }
	});
<link> 组件上的属性会被传递给它们创建的锚点上。所以这段 JSX：

	`<Link to="/users" className="users">`
会变成 DOM 中的：

	`<a href="/users" class="users">`
如果需要为非路由器路径创建一个锚点，比如一个外部网站，那么就用普通的锚点标记好了

### 活动链接

<link> 组件的一个很酷的功能是能够知道什么时候它是活动的：

	`<Link to="/users" activeClassName="active">Users</Link>`
如果用户是在 /users 路径上，那么路由器就会查找 <link> 做的匹配的锚点，并且会切换它们的 active 类.

### 浏览器历史

为避免混淆，我把一些重要的细节留到现在。<Router> 需要知道要采用哪个 历史跟踪策略。React Router 文档 推荐的浏览器历史是按照如下的方法实现的：

	var browserHistory = ReactRouter.browserHistory;
	ReactDOM.render((
	  <Router history={browserHistory}>
	    ...
	  </Router>
	), document.getElementById('root'));
在前面版本的 React Router 中，history 属性不是必需的，默认是使用 hashHistory。如名字所建议的，它在 URL 中使用 # 哈希符号来管理前端 SPA 风格的路由，与在 Backbone.js 路由器中的类似。

使用 hashHistory，URL 看起来将会是这样的：

	example.com
	
	example.com/#/users?_k=ckuvup
	
	example.com/#/widgets?_k=ckuvup

但是这些丑陋的查询字符串到底是什么啊？

当 browserHistory 被实现时，这些路径看起来更有组织：

	example.com
	
	example.com/users
	
	example.com/widgets

但是当 browserHistory 被用在前端时，在服务器上有一个告诫：如果用户开始他们在 example.com 上的访问，然后导航到 /users 和 /widgets，React Router 会像期待的那种处理这种场景；但是，如果用户直接通过在浏览器中键入 example.com/widgets 或者在 example.com/widgets 上刷新来开始他们的访问，那么浏览器至少会发起一次为 /widgets 对服务器的请求。但是如果这不是一个服务器端的路由器，这就会得到一个 404 错误：

当心 URL。你可能会需要一个服务器端路由器。

要解决来自服务器的 404 问题，React Router 推荐在服务器端使用一个通配符路由器。使用这种策略的话，不管调用的是什么服务器端路由，服务器会总是提供相同的 HTML 文件。然后，如果用户直接从 example.com/widgets 开始，即使返回的是相同的 HTML 文件，React Router 也会足够聪明地加载正确的组件。

用户是不会注意到任何怪异的事情的，但是你也许会介意总是返回相同的 HTML 文件。在代码示例中，本系列教程会继续使用"通配符路由器"策略，但是这取决于你以你认为合适的方式来处理服务器端路由。

那么 React Router 能不能以一种同型（isomorphic） 的方式用在服务器端和客户端？它当然能，但是这超出来本教程的范围。

### 用 browserHistory 重定向

browserHistory 是一个单例对象，所以你可以将它包含在任何文件中。如果你需要在任何代码中手动重定向用户，你可以使用它的 push 方法来实现：

	browserHistory.push('/some/path');
### 路由匹配

React router 处理路由匹配的方法与其它路由器相似：

	<Route path="users/:userId" component={UserProfile} />
这个路由会匹配当用户访问任何以 users/ 开头，后面跟着任意值的路径。它会匹配 /users/、/users/143，甚至是 /users/abc （如果是这样你将需要自己校验）。

React Router 会将 :userId 的值作为 prop 传递给 UserProfile。这个属性可以通过UserProfile 内的 this.props.params.userId 访问。

### 路由器演示

如果点击示例中的一些路由，你会注意到浏览器的后退和前进按钮对路由器是起作用的。这也是这些 history 策略存在的一个主要原因。此外，记住对于你访问的每个路由，除了最开始要获取初始 HTML 外，就没有其它向服务器发起的请求。很酷是吧？

### ES6

在我们的 CodePen 示例中，React、ReactDOM 和 ReactRouter 都是来自 CDN 的全局变量。ReactRouter 对象内都是我们需要的各种东西，比如 Router 和 Route 组件。所以我们可以像这样使用 ReactRouter：

	ReactDOM.render((
	  <ReactRouter.Router>
	    <ReactRouter.Route ... />
	  </ReactRouter.Router>
	), document.getElementById('root'));
这里，我们不得不在路由器组件前面加上它们的父对象 ReactRouter 作为前缀。我们还可以像下面这样，用 ES6 新的解构法：

	var { Router, Route, IndexRoute, Link } = ReactRouter
这样子就把 ReactRouter 的各部分提取到普通变量中，这样我们就可以直接访问它们了。

### 用 Webpack 和 Babel 打包

webpack 将多个 JS 文件为浏览器打包到一个文件。
Babel 会将 ES6（ES2015）代码转换为 ES5，因为很多浏览器还不能理解 ES6。

### 小心已经被弃用的语法

网上很多有关 React Router 的文章都是 pre-1.0 版本的。现在很多 pre-1.0 的功能被弃用了。如下是一个简单的列表：

	<Route name="" /> 被弃用。用 <Route path="" /> 替代。
	<Route handler="" /> 被弃用。用 <Route component="" /> 替代。
	<NotFoundRoute /> 被弃用。看替代方案（ https://github.com/reactjs/react-router/blob/b60d6c0351ff91cf04bccdac8c4b6e976aec94ec/upgrade-guides/v1.0.0.md#notfound-route ）
	<RouteHandler /> 被弃用。
	willTransitionTo 被弃用。看 onEnter（ https://github.com/reactjs/react-router/blob/b60d6c0351ff91cf04bccdac8c4b6e976aec94ec/upgrade-guides/v1.0.0.md#willtransitionto-and-willtransitionfrom ）
	willTransitionFrom 被弃用。看 onLeave（ https://github.com/reactjs/react-router/blob/b60d6c0351ff91cf04bccdac8c4b6e976aec94ec/upgrade-guides/v1.0.0.md#willtransitionto-and-willtransitionfrom ）
	"Locations" 现在叫 "histories".
