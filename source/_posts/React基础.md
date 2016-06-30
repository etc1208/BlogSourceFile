---
title: React基础
date: 2016-06-30 08:35:19
tags: [React, 前端]
---

####用于构建用户界面的JAVASCRIPT库
参考：*[http://reactjs.cn/react/docs/](http://reactjs.cn/react/docs/ "React官方文档")*

React 中全是`模块化、可组装的组件`!全是`模块化、可组装的组件`!全是`模块化、可组装的组件`(重要的事情说三遍......``)
![React](http://easyread.ph.126.net/IpX1Cv-rPea28EuvwgZ6PQ==/8796093022314275545.jpg)

<!--more-->

----------

## 用React构建应用的大体步骤： ##



- 第一步：拆分用户界面为一个组件树

- 第二步： 利用 React ，创建应用的一个静态版本

- 第三步：识别出最小的（但是完整的）代表 UI 的 state

**简单地对每一项数据提出三个问题，来指出哪一个是 state ：**

	1. 是否是从父级通过 props 传入的？如果是，可能不是 state 。
	2. 是否会随着时间改变？如果不是，可能不是 state 。
	3. 能根据组件中其它 state 数据或者 props 计算出来吗？如果是，就不是 state 。



- 第四步：确认 state 的生命周期

**对于应用中的每一个 state 数据：**

	1. 找出每一个基于那个 state 渲染界面的组件。
	2. 找出共同的祖先组件（某个单个的组件，在组件树中位于需要这个 state 的所有组件的上面）。
	3. 要么是共同的祖先组件，要么是另外一个在组件树中位于更高层级的组件应该拥有这个 state 。
	4. 如果找不出拥有这个 state 数据模型的合适的组件，创建一个新的组件来维护这个 state ，然后添加到组件树中，层级位于所有共同拥有者组件的上面。


- 第五步：添加反向数据流


----------
## 数据呈现 ---JSX##

用户界面能做的最基础的事就是呈现一些数据。React 让显示数据变得简单，当数据变化时，用户界面会`自动同步更新`。

    React 是不会去操作 DOM 的，除非不得不操作 DOM 。它用一种更快的内置仿造的 DOM 来操作差异，为你计算出效率最高的 DOM 改变。

	组件的输入被称为 props 。它们通过 JSX 语法进行参数传递。在组件里这些属性是不可直接改变的，也就是说 this.props 是只读的。

**注意:**

**一个限制: React 组件只能渲染单个根节点。如果你想要返回多个节点，它们必须被包含在同一个节点里。**

React 的 JSX 里约定分别使用`首字母大、小写`来区分本地组件的类和 HTML 标签。

**注意:
由于 JSX 就是 JavaScript，一些标识符像 class 和 for 不建议作为 XML 属性名。作为替代，React DOM 使用 className 和 htmlFor 来做对应的属性。**


**JavaScript 表达式**

属性表达式

	要使用 JavaScript 表达式作为属性值，只需把这个表达式用一对大括号 ({}) 包起来，不要用引号 ("")。

**延展属性（Spread Attributes）**
现在你可以使用 JSX 的新特性 - 延展属性：

	  var props = {};
	  props.foo = x;
	  props.bar = y;
	  var component = <Component {...props} />;
传入对象的属性会被复制到组件内。

它能被多次使用，也可以和其它属性一起用。注意顺序很重要，后面的会覆盖掉前面的。

	  var props = { foo: 'default' };
	  var component = <Component {...props} foo={'override'} />;
	  console.log(component.props.foo); // 'override'

----------

## 富交互性的动态用户界面 ##

###事件处理与合成事件
React 里只需把事件处理器以骆峰命名形式当作组件的 props 传入即可。React 内部创建一套合成事件系统来使所有事件在 IE8 和以上浏览器表现一致。

如果需要在手机或平板等触摸设备上使用 React，需要调用

	React.initializeTouchEvents(true); 启用触摸事件处理。

**State 工作原理**

常用的通知 React 数据变化的方法是调用 `setState(data, callback)`。

这个方法会合并data 到 this.state，并重新渲染组件。渲染完成后，调用可选的 callback 回调。大部分情况下不需要提供 callback，因为 React 会负责把界面更新到最新状态。

**哪些组件应该有State？**

大部分组件的工作应该是从` props` 里取数据并渲染出来。但是，有时需要对用户输入、服务器请求或者时间变化等作出响应，这时才需要使用 State。

    常用的模式是创建多个只负责渲染数据的无状态组件，在它们的上层创建一个有状态组件并把它的状态通过 props 传给子级。这个有状态的组件封装了所有用户的交互逻辑，而这些无状态组件则负责声明式地渲染数据。

**哪些应该作为State？**

State 应该包括那些可能被组件的事件处理器改变并触发用户界面更新的数据。 真实的应用中这种数据一般都很小且能被 JSON 序列化。当创建一个状态化的组件时，想象一下表示它的状态最少需要哪些数据，并只把这些数据存入 `this.state`。在 render() 里再根据 state 来计算你需要的其它数据。你会发现以这种方式思考和开发程序最终往往是正确的，因为如果在 state 里添加冗余数据或计算所得数据，需要你经常手动保持数据同步，不能让 React 来帮你处理。

**哪些不应该作为State？**

`this.state `应该仅包括能表示用户界面状态所需的最少数据。

因此，它不应该包括：

- 计算所得数据： 
- React 组件：
- 基于 props 的重复数据： 


----------
## 复合组件 ##

**动机：关注分离**

通过复用那些接口定义良好的组件来开发新的模块化组件，能够通过开发简单的组件把程序的不同关注面分离。

**子级**

实例化 React 组件时，你可以在开始标签和结束标签之间引用React 组件或者Javascript 表达式：

	<Parent><Child /></Parent>

***Parent 能通过专门的 this.props.children读取子级。***

**动态子级**
如果子组件位置会改变或者有新组件添加到列表开头，情况会变得更加复杂。如果子级要在多个渲染阶段保持自己的特征和状态，在这种情况下，你可以通过给子级设置惟一标识的 `key `来区分。

	  render: function() {
	    var results = this.props.results;
	    return (
	      <ol>
	        {results.map(function(result) {
	          return <li key={result.id}>{result.text}</li>;
	        })}
	      </ol>
	    );
	  }
当 React 校正带有 key 的子级时，它会确保它们被重新排序或者删除。务必把 key 添加到子级数组里组件本身上，而不是每个子级内部最外层 HTML 上

**数据流**

React 里，数据通过上面介绍过的 props `从拥有者流向归属者`。这就是高效的`单向数据`绑定：拥有者通过它的 props 或 state 计算出一些值，并把这些值绑定到它们拥有的组件的 props 上。因为这个过程会递归地调用，所以数据变化会自动在所有被使用的地方自动反映出来。


**Prop 验证**

随着应用不断变大，保证组件被正确使用变得非常有用。为此我们引入 `propTypes`。`React.PropTypes` 提供很多验证器 来验证传入数据的有效性。当向 props 传入无效数据时，JavaScript 控制台会抛出警告。

	React.createClass({
	  propTypes: {
	    // 可以声明 prop 为指定的 JS 基本类型。默认
	    // 情况下，这些 prop 都是可传可不传的。
	    optionalArray: React.PropTypes.array,
	    optionalBool: React.PropTypes.bool,
	    optionalFunc: React.PropTypes.func,
	    optionalNumber: React.PropTypes.number,
	    optionalObject: React.PropTypes.object,
	    optionalString: React.PropTypes.string,    
	  } 
	});

**默认 Prop 值**

React 支持以声明式的方式来定义 props 的默认值。

	var ComponentWithDefaultProps = React.createClass({
	  getDefaultProps: function() {
	    return {
	      value: 'default value'
	    };
	  }
	});
当父级没有传入 props 时，getDefaultProps() 可以保证 this.props.value 有默认值，注意 getDefaultProps 的结果会被 缓存。

----------
## 表单组件 ##

**交互属性**

表单组件支持几个受用户交互影响的属性：

    - value，用于 <input>、<textarea> 组件。
    - checked，用于类型为 checkbox 或者 radio 的 <input> 组件。
    - selected，用于 <option> 组件。

在 HTML 中，`<textarea> `的值通过子节点设置；在 React 中则应该使用 `value` 代替。

表单组件可以通过` onChange `回调函数来监听组件变化。当用户做出以下交互时，onChange 执行并通过浏览器做出响应：

    <input> 或 <textarea> 的 value 发生变化时。
    <input> 的 checked 状态改变时。
    <option> 的 selected 状态改变时。
和所有 DOM 事件一样，所有的 HTML 原生组件都支持 onChange 属性，而且可以用来监听冒泡的 change 事件。

**受限组件**

设置了` value` 的` <input> `是一个受限组件。 对于受限的 `<input>`，渲染出来的 HTML 元素始终保持 value 属性的值。例如：

	  render: function() {
	    return <input type="text" value="Hello!" />;
	  }

上面的代码将渲染出一个值为 Hello! 的 input 元素。用户在渲染出来的元素里输入任何值都不起作用，因为 React 已经赋值为 Hello!。如果想响应更新用户输入的值，就得使用 onChange 事件：

	  getInitialState: function() {
	    return {value: 'Hello!'};
	  },
	  handleChange: function(event) {
	    this.setState({value: event.target.value});
	  },
	  render: function() {
	    var value = this.state.value;
	    return <input type="text" value={value} onChange={this.handleChange} />;
	  }

**不受限组件**
没有设置 value(或者设为 null) 的 `<input>` 组件是一个不受限组件。对于不受限的 `<input> `组件，渲染出来的元素直接反应用户输入。例如：

	  render: function() {
	    return <input type="text" />;
	  }
上面的代码将渲染出一个空值的输入框，用户输入将立即反应到元素上。和受限元素一样，使用 onChange 事件可以监听值的变化。

如果想给组件设置一个非空的初始值，可以使用 `defaultValue `属性。例如：

	  render: function() {
	    return <input type="text" defaultValue="Hello!" />;
	  }

HTML 中` <select>` 通常使用 `<option> `的 `selected `属性设置选中状态；React 为了更方便的控制组件，采用以下方式代替：

	  <select value="B">
	    <option value="A">Apple</option>
	    <option value="B">Banana</option>
	    <option value="C">Cranberry</option>
	  </select>
如果是不受限组件，则使用 `defaultValue`。

**注意：**

给 value 属性传递一个数组，可以选中多个选项：

	<select multiple={true} value={['B', 'C']}>


----------
## Refs和findDOMNode() ##

为了和浏览器交互，你将需要对DOM节点的引用。React提供了`React.findDOMNode(component)`函数，你可以调用这个函数来获取该组件的DOM结点。

**注意：**

**findDOMNode()仅在挂载的组件上有效**（也就是说，组件已经被放进了DOM中）。如果你尝试在一个未被挂载的组件上调用这个函数（例如在创建组件的render()函数中调用findDOMNode()），将会抛出异常。

为了获取一个到React组件的引用，你可以使用this来得到当前的React组件，或者你可以使用refs来指向一个你拥有的组件。它们像这样工作：

	var MyComponent = React.createClass({
	  handleClick: function() {
	    React.findDOMNode(this.refs.myTextInput).focus();
	  },
	  render: function() {
	    return (
	      <div>
	        <input type="text" ref="myTextInput" />
	        <input
	          type="button"
	          value="Focus the text input"
	          onClick={this.handleClick}
	        />
	      </div>
	    );
	  }
	});
	
## 组件生命周期 ##

- 挂载： 组件被插入到DOM中。
- 更新： 组件被重新渲染，查明DOM是否应该刷新。
- 移除： 组件从DOM中移除。


**挂载**

- getInitialState()
- componentWillMount()
- componentDidMount()

**更新**

- componentWillUpdate(object nextProps, object nextState)
- componentDidUpdate(object prevProps, object prevState)

**移除**

- componentWillUnmount()






