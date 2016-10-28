---
title: React容器组件
date: 2016-10-28 09:53:56
tags: [容器组件]
---

转自：[React晋级：容器组件](http://mp.weixin.qq.com/s?__biz=MzA4NjE3MDg4OQ==&mid=2650963773&idx=1&sn=035a92a392a80581e66fd5ec99a4eea5&chksm=843a135bb34d9a4de8cde76599d97f4fd374c30a8c766f3283b383757e9b171c98fc50bc7020&mpshare=1&scene=1&srcid=1028KpI9COXO7ncRm5F8VyRl#rd)


![容器组件](http://upload.art.ifeng.com/2015/1102/1446435435655.jpg)

<!--more-->

### 用 AJAX 获取数据

作为一个不佳实践的示例，我们扩展一下前一篇教程中的 UserList 组件，让它可以处理它自己的数据抓取：

// 这个示例是一个不推荐的做法，它将视图与数据耦合得太紧。

	var UserList = React.createClass({
	  getInitialState: function() {
	    return {
	      users: []
	    }
	  },
	
	  componentDidMount: function() {
	    var _this = this;
	    $.get('/path/to/user-api').then(function(response) {
	      _this.setState({users: response})
	    });
	  },
	
	  render: function() {
	    return (
	      <ul className="user-list">
	        {this.state.users.map(function(user) {
	          return (
	            <li key={user.id}>
	              <Link to="{'/users/' + user.id}">{user.name}</Link>
	            </li>
	          );
	        })}
	      </ul>
	    );
	  }
	});

为什么说这是一个不太理想的示例？首先，我们破坏了不要把“行为”与“如何渲染视图"混在一起的规则 - 这两件事情应该分离才对。

要明白，用 getInitialState 来初始化组件的状态并没有什么错，从 componentDidMount 管理一个 AJAX 请求也没有什么错（虽然我们可能更应该将实际调用抽出来放到其他函数中）。问题是，我们是在存储视图的同一个组件中同时做了这两件事情。这种紧耦合让应用程序变得更死板、更啰嗦（wet）。如果你还需要在其他地方获取用户列表该怎么办？上面的示例中，获取用户的行为与这个视图绑在了一起，所以它不能被复用。

第二个问题是，我们使用了 jQuery 来发起 AJAX 调用。诚然，jQuery 有很多好用的功能，但是它的大部分功能都是用来处理 DOM 渲染，而 React 已经具备了自己的处理方法。至于 jQuery 的非 DOM 功能，比如 AJAX，我们可以找到很多只关注这一个功能的可选方案。

其中之一就是 Axios，一个基于 promise 的 AJAX 工具。它与 jQuery 基于 Promise 的 AJAX 功能在 API 上很相似。如何相似呢？

	// jQuery
	$.get('/path/to/user-api').then(function(response) { ... });
	
	// Axios
	axios.get('/path/to/user-api').then(function(response) { ... });
余下的示例中，我们会一直用 Axios。其它类似工具有 got、fetch 和 SuperAgent。

### Props 和 State

在学习容器组件和展示性组件之前，我们需要搞清楚有关 props 和 state 的一些知识。二者都可以从父组件向下传递到子组件。但是，父组件的 props 和 state 只会成为子组件的 props。

比如，假如 ComponentA 将它的一些 props 和 state 传递给它的子组件 ComponentB。ComponentA 的 render 方法看起来可能这样的：

	// ComponentA
	render: function() {
	  return <ComponentB foo={this.state.foo} bar={this.props.bar} />
	}
即使 foo 是父组件上的 “state”，它也会成为子组件 ComponentB 上的一个 “prop”。bar属性也会成为子组件上的 prop，因为所有从父组件传递到子组件的数据都会成为子组件中的 props。下面的示例展示 ComponentB 的方法是如何把 foo 和 bar 当作 props 访问的：
	
	// ComponentB
	componentDidMount: function() {
	  console.log(this.props.foo);
	  console.log(this.props.bar);
	}
在用 AJAX 获取数据的示例中，从 AJAX 接收的数据被设置为组件的 state。那个示例没有子组件，但是你可以料想一下，如果它有子组件的话，那么 state 会作为 props 从父组件向下“流”到子组件。

### 是时候分拆了

在用 AJAX 获取数据示例中，我们制造了一个问题。UserList 组件没啥毛病，但是它试图做太多事情了。为了解决这个问题，我们来把 UserList 分拆成两个组件，每个组件充当不同的角色。这两个组件的类型，在概念上将会称为 容器组件 和 展示性组件，又称“智能（smart)”组件和“木偶（dumb）”组件。

简而言之，容器组件获取数据，并处理状态。然后状态会被传递给展示性组件作为 props，然后被渲染到视图。
### 展示性组件

你也许不知道展示性组件，但是在本系列教程的前面你已经看到过它。想像一下 UserList 组件在管理自己的状态之前的样子：
	
	var UserList = React.createClass({
	  render: function() {
	    return (
	      <ul className="user-list">
	        {this.props.users.map(function(user) {
	          return (
	            <li key={user.id}>
	              <Link to="{'/users/' + user.id}">{user.name}</Link>
	            </li>
	          );
	        })}
	      </ul>
	    );
	  }
	});
这个组件与之前的不太一样，它是一个展示性组件。它和之前的组件最大的区别为，它靠遍历用户数据来创建列表条目，并且通过 props 接收用户数据。

展示性组件是“木偶”，就是说它们不知道它们接收的 props 是如何形成的，也不知道 state。

展示性组件永远不会自己修改 prop 数据。实际上，任何接收 props 的组件应该认为该数据是不可变的，是属于父组件的。不过，虽然展示性组件不能修改 prop 中数据的意义，但是它能为视图格式化数据（比如将 Unix timestap 转换为人类可读的东西）。

在 React 中，事件是直接通过像 onClick 这样的属性绑定到视图。但是，有人会问，既然展示性组件被认为不能修改 props，那么事件是如何工作的呢？为此，我们下面会有整整一个小节来解释事件。

### 迭代

当在循环中创建 DOM 节点时，key 属性是必需的，且是唯一的 （相对于相邻兄弟）。注意，这只是对最高级别的 DOM 节点 - 本例中的<li>。

此外，如果嵌套的 return 对你来说看起来有点古怪的话，可以考虑将列表条目的创建分离到它自己的函数中：

	var UserList = React.createClass({
	  render: function() {
	    return (
	      <ul className="user-list">
	        {this.props.users.map(this.createListItem)}
	      </ul>
	    );
	  },
	
	  createListItem: function(user) {
	    return (
	      <li key={user.id}>
	        <Link to="{'/users/' + user.id}">{user.name}</Link>
	      </li>
	    );
	  }
	});
	
### 容器组件

容器组件几乎总是展示性组件的父组件。在某种程度上，它充当展示性组件和应用程序其它部分之间的一个中介。它们也称为智能（smart)组件，因为它们知道整体应用程序。

既然容器组件和展示性组件需要有不同的名字，所以为避免混淆，我们称之为 UserListContainer：
	
	var React = require('react');
	var axios = require('axios');
	var UserList = require('../views/list-user');
	
	var UserListContainer = React.createClass({
	  getInitialState: function() {
	    return {
	      users: []
	    }
	  },
	
	  componentDidMount: function() {
	    var _this = this;
	    axios.get('/path/to/user-api').then(function(response) {
	      _this.setState({users: response.data})
	    });
	  },
	
	  render: function() {
	    return (<UserList users={this.state.users} />);
	  }
	});
	
	module.exports = UserListContainer;
为简单起见，前面的例子都省略了 require() 和 module.exports 语句。但是在本例中，展示容器组件将它们各自的展示性组件作为直接依赖是很重要的。为完整性起见，本例展示了所有需要的 require。

容器组件可以像其它 React 组件一样创建。它们跟其它组件一样，也有一个 render 方法，它们只是不创建任何渲染它们自己的东西，而是返回展示性组件的结果。

ES6 箭头函数快速备注：你可能注意到上面的例子中用到了传统的 var _this = this技巧。相比之下， ES6 箭头函数除了语法更短外，还有其它的好处，借此可以减少使用这种技巧的需要。
### 事件

迄今为止，我们展示了状态是如何从容器组件传递到展示性组件，但是行为如何传递呢？事件也属于行为的范畴，它们通常需要修改数据。React 中的事件是在视图级别绑定的。对于分离关注点来说，如果我们在视图所在的地方创建事件函数，就会在展示性组件中出问题。

为详细阐述，我们首先直接给展示性组件添加一个事件（一个可以点击的 < button >），来演示问题：
	
	// 展示性组件
	
	var UserList = React.createClass({
	  render: function() {
	    return (
	      <ul className="user-list">
	        {this.props.users.map(function(user) {
	
	          return (
	            <li key={user.id}>
	              <Link to="{'/users/' + user.id}"\>{user.name}</Link>
	              <button onClick={this.toggleActive}>Toggle Active</button>
	            </li>
	          );
	
	        })}
	      </ul>
	    );
	  },
	
	
	  toggleActive: function() {
	    // 我们不能在展示性组件中修改状态 :(
	  }
	});
从技术讲，这段代码是可以运行的，但是并不是一个较好的实现。事件需要修改数据，能修改的数据应该被存为状态 ，也就是展示性组件不应该知道的东西。

在我们的示例中，状态改变的将是用户的“激活状态”，但是你可以把你想要的任何函数绑定到 onClick。

更好的解决方案是将功能作为 prop，从容器组件传递给展示性组件，像这样子：

	// 容器组件
	
	var UserListContainer = React.createClass({
	  ...
	  render: function() {
	    return (<UserList users={this.state.users} toggleActive={this.toggleActive} />);
	  },
	
	  toggleActive: function() {
	    // 我们应该在容器组件中修改状态 :)
	  }
	});
	
	// 展示性组件
	
	var UserList = React.createClass({
	  render: function() {
	    return (
	      <ul className="user-list">
	      {this.props.users.map(function(user) {
	
	        return (
	          <li key={user.id}>
	            <Link to="{'/users/' + user.id}">{user.name}</Link>
	            <button onClick={this.props.toggleActive}>Toggle Active</button>
	          </li>
	        );
	
	      })}
	      </ul>
	    );
	  }
	});
onclick 属性必须放在视图所在的地方，即展示性组件中。但是，它调用的函数已经移到父容器组件中了。这个方式更好，因为是由容器组件来处理状态。

如果父函数刚好改变了状态，那么状态改变就会导致重新渲染父函数，然后依次更新子组件。这在 React 中是自动发生的。

下面的示例，展示了容器组件上的事件如何修改状态，从而自动更新展示性组件：


	// 容器组件
	
	const UserListContainer = React.createClass({
	  getInitialState: function() {
	    return {
	      users: [
	        {id:1, name:'Ryan', active:true},
	        {id:2, name:'Michael', active:true},
	        {id:3, name:'Dan', active:true}
	      ]
	    }    
	  },
	
	  render: function() {
	    return (<UserList users={this.state.users} toggleActive={this.toggleActive} />);
	  },
	
	  toggleActive: function(userId) {
	
	    // 状态永远不应该被直接修改。我们应该总是创建一份状态的副本，然后用副本替换原始状态。
	    // 在本例中，我们是替换整个状态，如果有很多记录这样做可能性能不是最佳的，但是对于本例来说足够了。
	    // 我们会将在第三篇文章 Redux 中谈到更多有关可修改和不可修改状态。
	
	    var newState = Object.assign({}, this.state)
	    var user = _.find(newState.users, {id: userId});
	    user.active = !user.active
	    this.setState(newState)
	
	  }
	});
	
	// 展示性组件
	
	const UserList = React.createClass({
	  render: function() {
	    let _this = this;
	
	    return (
	      <ul className="user-list">
	        {this.props.users.map(function(user) {
	
	          // `.bind()` 方法确保当 `toggleActive()` 被调用时，第一个参数将是用户的 ID
	
	          return (
	            <li key={user.id}>
	              <a href="#">{user.name}</a>
	              <span>{user.active ? 'Active' : 'Not Active'}</span>
	              <button onClick={_this.props.toggleActive.bind(null, user.id)}>Toggle Active</button>
	            </li>
	            );
	        })}
	      </ul>
	      );
	  }
	});
	
	ReactDOM.render(<UserListContainer />, document.getElementById('root'))
请注意这个示例如何处理不可修改的数据，以及如何使用.bind()方法。

### 用路由器使用容器组件

路由器应该不在直接使用 UserList，而是直接使用 UserListContainer，而 UserListContainer 又会使用 UserList。最后，UserListContainer 返回 UserList 结果，所以路由器依然会接收到它所需的东西。

### 数据流和扩展运算符

在 React 中，props 被从父组件向下传递到子组件的概念被称为流。迄今为止的示例只展示了简单的父子关系，但是在真实的应用程序中可能有很多嵌套的组件。想像一下数据流通过状态和 props，从高层父组件向下流过很多子组件。这是一个 React 中必须记住的一个基础概念。

ES6 有一个新的扩展运算符很有用。React 已经把相似的语法用到了 JSX。这对 React 中数据通过 props 流动很有帮助。
### 无状态的函数式组件

从 React 0.14 开始，就有了一个更容易创建无状态（展示性）组件的新功能。这个新功能称为无状态函数式组件。

到现在你可能已经注意到，随着你分离容器组件和展示性组件，很多展示性组件只有一个 render 方法。在这些例子中，React 现在允许组件可以被写成一个函数：

	// 老的啰嗦的方式
	
	var Component = React.createClass({
	
	  render: function() {
	    return (
	      <div>{this.props.foo}</div>
	    );
	  }
	});
	
	// 较新的无状态函数式组件方式
	
	var Component = function(props) {
	  return (
	    <div>{props.foo}</div>
	  );
	};
你可以很清楚地看到，新的方式更简洁。但是记住，这种方式只能替代只需要一个 render 方法的组件。

用新的无状态函数式方式，函数的参数为 props。这意味着它不需要用 this 来访问 props。
### MVC

到现在你可能已经看到，React 跟传统的 MVC 不一样。React 更多时候被称为“单纯的视图层”。这个称呼会使得 React 初学者认为 React 可能属于他们熟悉的传统 MVC，就好像它命中注定要和第三方的传统控制器和模型一起用似的。

React 确实没有“传统的控制器”，但是它以自己特殊的方式提供了一种分离视图和行为的方法。我认为容器组件承担的基础用途达到了与传统 MVC 中的控制器一样的效果。

至于模型，我已经看到了有人在 React 中用 Backbone 模型，我也相信他们在这样做是否好方面有各种观点。但是我不认为传统模型是适合 React 的方式。React 流动数据的方式，使得它不适合传统模型的工作方式。Facebook 创建的 Flux 设计模式 适应了 React 对流动数据天生的处理能力。

