---
title: 在CSS中使用功能查询
date: 2016-09-08 10:04:47
tags: [css3, @supports]
---

在 CSS 里，有一个你可能没有听过的工具，但是它已经出现一段时间了，而且非常强大。也许，它会成为 CSS 中你最喜欢的东西之一。

![使用@supports](http://img.htmleaf.com/1412/201412092129.jpg)

<!--more-->

- 原文：Jen Simmons 
- 译文：Erichain_Zain
- 链接：segmentfault.com/a/1190000006734430


那么，是什么呢？就是 @support，也就是功能查询。

通过 @support，你可以在 CSS 中使用一小段的测试来查看浏览器是否支持一个特定的 CSS 功能(这个功能可以是 CSS 的某种属性或者某个属性的某个值)，然后，根据测试的结果来决定是否要应用某段样式。比如：

	@supports ( display: grid ) {
	    // 如果浏览器支持 Grid，这里面的代码才会执行
	}

如果浏览器能够理解 display: grid，那么，大括号里的代码都会被应用，否则，这些样式就会被跳过。

现在，对于功能查询是什么，你也许还有一点疑惑。这并不是通过某种额外的验证来分析浏览器是否已经确切的实现了某个 CSS 属性。如果你需要查看额外的验证，可以查看 Test the Web Forward。

功能查询让浏览器自己就能够表现出是否支持某个 CSS 属性或者 CSS 属性值。然后通过这个结果来判断是否要应用某段 CSS。如果一个浏览器没有正确的(或者完全的)实现一个 CSS 属性，那么，@supports 就没有什么用了。它并不是一个能够让浏览器的 bug 消失的魔杖。

但是，我已经发现 @supports 是那么难以置信的有帮助。比起以前没有这个属性的时候，@supports 能够让我一再的使用新的 CSS 功能。

多年以来，开发者们都在使用 Modernizr 来实现功能查询，但是 Modernizr 需要 JavaScript。虽然这部分 JavaScript 很小，但是，CSS 结构中添加了 Modernizr 的话，在 CSS 被应用之前，就需要下载 JavaScript 然后等待执行完成。比起使用 CSS，加入了 JavaScript 总是会更慢。而且，要是 JavaScript 执行失败了呢？另外，Modernizr 还需要一层额外复杂的、很多项目都无法理解的东西。相比之下，功能查询更快，功能更强大，使用起来更简单。

你也许注意到了，@supports 的写法和媒体查询很类似，我觉得他们可能就是堂兄弟的关系。

	@supports ( display: grid ) {
	    main {
	        display: grid;
	        grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
	    }
	}

大多数时候，你其实不需要这样的测试。比如，你可以直接这样写：
	
	aside {
	    border: 1px solid black;
	    border-radius: 1em;
	}

如果浏览器能够理解 border-radius，那么，在相应的容器上就会应用圆角样式。如果它不能理解这个属性，那么，就会直接跳过并继续执行下面的语句。容器的边缘也就保持直角了。完全没有必要使用功能查询或者测试，CSS 就是这样运作的。这是属于 CSS 中稳固设计，渐进增强的一个基本的原则。浏览器会直接跳过他们无法解析的语句，不会抛出任何错误。


大多数浏览器都会应用 border-radius: 1em;，然后展示出右边的效果。但是，在 IE6，7，8 上你却不能看到圆角，你看到的将是左边的效果。可以看看这个例子：A Rounded Corner Box。

对于这个例子，没有必须要使用功能查询。

那么，什么时候才需要使用 @supports 呢？功能查询是将 CSS 声明绑定在一起的一个工具，以便于这些 CSS 规则能够在某种条件下以一个组合的方式运行。当你需要混合使用 CSS 的新规则和旧规则的时候，并且，仅当 CSS 新功能被支持的时候，就可以使用功能查询了。

译者注：以下例子中的 initial-letter 属性现在在所有现代浏览器中都不受支持，所以，以下例子中的代码可能都是无效的代码。如果下文中有提到此属性在某某浏览器中受支持的话，请忽略。需要了解 initial-letter 详细的说明，可以参考initial-letter | MDN。

来看一个关于使用 initial-letter 的例子。这个属性告诉浏览器要将特指的那个元素变得更大，就像一个段首大字一样。在这里，我们要做的就是让段落的第一个字母的大小为四行文字那么大。同时，我们再对它进行加粗，在它的右边设置一些 margin，还给它设置一个高亮的橘色。OK，很不错了。

	p::first-letter {
	    margin-right: 0.5em;
	    color: #FE742F;
	    font-weight: bold;
	    -webkit-initial-letter: 4;
	    initial-letter: 4;
	}

好吧，简直没法接受。除了使用 initial-letter 来达到我们想要的效果之外，我们并不想要改变字体的 color，margin，和font-weight。所以，我们需要一个方法来测试浏览器是否支持 initial-letter 这个属性，然后在浏览器支持这个属性的时候再应用相关的样式。所以，使用功能查询：

	@supports (initial-letter: 4) or (-webkit-initial-letter: 4) {
	    p::first-letter {
	        -webkit-initial-letter: 4;
	        initial-letter: 4;
	        color: #FE742F;
	        font-weight: bold;
	        margin-right: 0.5em;
	    }
	}

##### 注意，测试的时候需要进行完全的测试，CSS 属性和值都得写上。一开始我也比较疑惑，为什么非得测试 initial-letter: 4呢？4 这个值很重要吗？如果我写成 17 呢？莫非是需要匹配我即将要应用的 CSS 中的样式吗？

原因是这样的：@supports 在测试的时候，需要提供属性和值，因为有时候测试的是属性，有时候测试的是值。对于 initial-letter ，你输入多少值并不重要。但是，如果是 @suports ( dislay: grid ) 就不一样了，所有浏览器都识别 display，但是，并不是所有浏览器都识别 display: grid。

回到我们的例子，当前的 initial-letter 只在 Safari 9 上受支持，并且需要加前缀。所以，我加了前缀，同时，保持着不加前缀的规则，并且，我还写了测试来测试另外的属性。没错，在功能查询中，还可以使用 and, or, not 声明。

下面是新的结果。理解 initial-letter 的浏览器会显示一个巨大加粗高亮的段首大字。其他浏览器的行为就像是这个段首大字不存在一样。当然，如果更多的浏览器支持了这个属性，那么，他们的行为也将会是有一个段首大字。

#### 组织你的代码

现在，也许你迫不及待的想要使用这个工具来将你的代码分为两个分支，使其变得干净一些。“Hey，浏览器，如果你识别 Viewport 单位，就执行这个，否则，执行另外的”。感觉很不错，有条理。

	@supports ( height: 100vh ) {
	    // 支持 viewport height 的样式
	}
	@supports not ( height: 100vh ) {
	    // 对于旧浏览器的替代样式
	}
	// 我们希望是好的，但这是一段烂代码

这段代码并不好，至少当前看来是这样的。发现问题了吗？

问题就是，并不是所有的浏览器都支持功能查询，不理解 @supports 的浏览器会直接跳过两段代码，这也许就太糟糕了。

意思就是，除非浏览器百分之百支持功能查询，否则我们就没法使用了吗？当然不是，我们完全可以使用功能查询，而且应该使用功能查询，只要不像上面那样写代码就行。

那么，怎么做才对呢？其实与使用媒体查询类似，我们在浏览器完全支持媒体查询之前就开始使用了不是吗？事实上，功能查询使用起来比媒体查询更简单，只要脑子放聪明一点就行了。

你想要让你的代码知道浏览器是否支持功能查询或者正在测试的某个功能，我来告诉你怎么做。

当然，在未来，浏览器 100% 支持功能查询的时候，我们可以大量使用 @supports not 来组织我们的代码。

#### 功能查询的支持情况

所以，@supports 现在支持度什么样了呢？

自从 2013 年中，@supports 就能够在 Firefox，Chrome 和 Opera 中使用了。在 Edge 的各个版本中也受支持。Safari 在 2015 年秋季才实现这个功能。

你可能会觉得 IE 不支持此功能会是一个大问题，但是，其实不是这样的。待会儿就会告诉你原因。我相信，最大的一个障碍是 Safari 8，我们需要留意在这个浏览器上发生的事情。

让我们来看另外一个例子。假设我们有些布局代码，为了正常运行，需要使用 object-fit: cover;。对于不支持这个属性的浏览器，我们想要使用不同的样式。

所以，我们可以这样写：

	@supports (initial-letter: 4) or (-webkit-initial-letter: 4) {
	    p::first-letter {
	        -webkit-initial-letter: 4;
	        initial-letter: 4;
	        color: #FE742F;
	        font-weight: bold;
	        margin-right: 0.5em;
	    }
	}
	 
	div {
	    width: 300px;
	    background: yellow;
	}
	@supports (object-fit: cover) {
	    img {
	        object-fit: cover;
	    }
	    div {
	        width: auto;
	        background: green;
	    }
	}


#### 最佳实践

所以，我们应该怎么写功能查询的代码呢？像下面这样：

	// fallback code for older browsers
	 
	@supports ( display: grid ) {
	    // code for newer browsers
	    // including overrides of the code above, if needed
	}
