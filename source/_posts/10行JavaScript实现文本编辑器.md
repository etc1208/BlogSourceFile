---
title: 小伙伴们惊呆了！10行JavaScript实现文本编辑器
date: 2016-09-09 15:21:44
tags: [js, 编辑器, contenteditable, execCommand]
---

转载自：[http://www.cnblogs.com/lhb25/p/html5-wysisyg-inline-editor.html](http://www.cnblogs.com/lhb25/p/html5-wysisyg-inline-editor.html)

今天下午在公司的技术分享会上听到关于编辑器开发的有关内容，期间提到了这篇博客，写得蛮不错的，搬过来看一看~

![编辑器](http://ico.ooopic.com/ajax/iconpng/?id=106179.png)

<!--more-->

用谷歌搜索找了很久，发现所有的插件都是功能太复杂，不是我想要的。所以，我决定我自己来实现需要的编辑功能。刚开始我觉得应该要花费很多的时间，因为我想象内容编辑功能应该是很复杂的。

　　但事实证明，它是如此简单，让我十分惊讶！借助 HTML5 特性，所有的工具都已经可用，所有你需要做的只是配合他们编写一些非常简单的 JavaScript 代码调用就可以了。

　　你需要两样东西，第一是：

	contenteditable
　　contenteditable 是 HTML 中的一个属性，设置 HTML标签的 contenteditable=“true” 时，即可开启该元素的编辑模式。contenteditable 的作用相当神奇，可以让 div 或整个网页，以及 span 等等元素设置为可编辑。

　　我们最常用的输入文本内容是 input 与 textarea，使用 contenteditable 属性后，可以在div、table、p、span、body等等很多元素中输入内容。如果想要整个网页可编辑，可以在 body 标签内设置 contenteditable。contenteditable 已在 HTML5 标准中得到有效的支持.

 　　第二样东西是一个功能特殊的函数：

	execCommand
　　execCommand 让我们能够对 contenteditable 区域的内容做一些相当复杂的操作。如果你想了解更为详细的内容可以看这里：The Mozilla Docs。 

　　从本质上讲， execCommand 有三个参数：

	　　Command Name – 命令名称；
	　　ShowDefaultUI – 未实现，所以最好设置为 false；
	　　ValueArgument – 命令的参数；

　　多数情况下，ValueArgument 可以设置为 null，但有一种情况：当你要设置一行文本的标签的时候（h1，h2，p 等），你需要使用 formatBlock 命令，并把标签放到 ValueArgument 中。

　　现在，事情就很简单了，我们把 exexCommand 要执行的命令放到 data-role 属性中，然后编写简单的 JavaScript 都可以很容易地完成编辑器功能了。核心代码其实就10行：


	$(function() {
	    $('#editControls a').click(function(e) {
	        switch($(this).data('role')) {
	            case 'h1':
	            case 'h2':
	            case 'p':
	                document.execCommand('formatBlock', false, '<' + $(this).data('role') + '>');
	                break;
	            default:
	                document.execCommand($(this).data('role'), false, null);
	                break;
	        }
	    })
	});
　　
完整代码如下：

	<!DOCTYPE html>
	<html>
	    <head>
	    <title>Editable WYSIWYG</title>
	        <link href="bootstrap.css" rel="stylesheet">
	        <link href="font-awesome.css" rel="stylesheet">
	        <style>
	            #editor {
	                resize:vertical;
	                overflow:auto;
	                border:1px solid silver;
	                border-radius:5px;
	                min-height:100px;
	                box-shadow: inset 0 0 10px silver;
	                padding:1em;
	            }
	        </style>
	    </head>
	    <body>
	        <div class="content">
	            <div class="container-fluid">
	                <div id='pad-wrapper'>
	                    <div id="editparent">
	                        <div id='editControls' class='span12' style='text-align:center; padding:5px;'>
	                            <div class='btn-group'>
	                                <a class='btn' data-role='undo' href='#'><i class='icon-undo'></i></a>
	                                <a class='btn' data-role='redo' href='#'><i class='icon-repeat'></i></a>
	                            </div>
	                            <div class='btn-group'>
	                                <a class='btn' data-role='bold' href='#'><b>Bold</b></a>
	                                <a class='btn' data-role='italic' href='#'><em>Italic</em></a>
	                                <a class='btn' data-role='underline' href='#'><u><b>U</b></u></a>
	                                <a class='btn' data-role='strikeThrough' href='#'><strike>abc</strike></a>
	                            </div>
	                            <div class='btn-group'>
	                                <a class='btn' data-role='justifyLeft' href='#'><i class='icon-align-left'></i></a>
	                                <a class='btn' data-role='justifyCenter' href='#'><i class='icon-align-center'></i></a>
	                                <a class='btn' data-role='justifyRight' href='#'><i class='icon-align-right'></i></a>
	                                <a class='btn' data-role='justifyFull' href='#'><i class='icon-align-justify'></i></a>
	                            </div>
	                            <div class='btn-group'>
	                                <a class='btn' data-role='indent' href='#'><i class='icon-indent-right'></i></a>
	                                <a class='btn' data-role='outdent' href='#'><i class='icon-indent-left'></i></a>
	                            </div>
	                            <div class='btn-group'>
	                                <a class='btn' data-role='insertUnorderedList' href='#'><i class='icon-list-ul'></i></a>
	                                <a class='btn' data-role='insertOrderedList' href='#'><i class='icon-list-ol'></i></a>
	                            </div>
	                            <div class='btn-group'>
	                                <a class='btn' data-role='h1' href='#'>h<sup>1</sup></a>
	                                <a class='btn' data-role='h2' href='#'>h<sup>2</sup></a>
	                                <a class='btn' data-role='p' href='#'>p</a>
	                            </div>
	                            <div class='btn-group'>
	                                <a class='btn' data-role='subscript' href='#'><i class='icon-subscript'></i></a>
	                                <a class='btn' data-role='superscript' href='#'><i class='icon-superscript'></i></a>
	                            </div>
	                        </div>
	                        <div id='editor' class='span12' style='' contenteditable>
	                            <h1>This is a title!</h1>
	                            <p>This is just some example text to start us off</p>
	                        </div>
	                    </div>
	                </div>
	            </div>
	        </div>
	        <script src="jquery.min.js"></script>
	        <script src="bootstrap.min.js"></script>
	        <script>
	 
	            $(function() {
	                $('#editControls a').click(function(e) {
	                    switch($(this).data('role')) {
	                        case 'h1':
	                        case 'h2':
	                        case 'p':
	                            document.execCommand('formatBlock', false, '<' + $(this).data('role') + '>');
	                            break;
	                        default:
	                            document.execCommand($(this).data('role'), false, null);
	                            break;
	                    }
	                     
	                })
	            });
	        </script>
	    </body>
	</html>