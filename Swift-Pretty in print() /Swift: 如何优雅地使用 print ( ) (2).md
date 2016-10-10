### Swift:如何优雅地使用 print( ) （2）
#### 如果说 Log 是一种时尚，那你就是时尚设计师。
在[上一篇文章](http://swift.gg/2016/08/03/swift-prettify-your-print-statements-pt-1/)中，我聊到了如何通过在打印的 log 中使用 emoji 表情来帮助你从冗杂的log信息丛林中减少认知负荷。然而，我给的糟糕的实现并不会让你对在自己的代码中使用表情产生强烈的意愿。
这篇文章我将会实现承诺，告诉你如何轻量地实现带表情的 log。

####概述
在本文的剩余部分，你将会看到我们打破 Swift 的命名约定，但我们这样做并不是毫无理由的。为了降低替代 print 的成本，我们需要降低 log 的体量，包括使用大写字母和标题的必要性也需要讨论。但是，在本文结束时，如果你的认知负荷还是存在的话，那就换成你喜欢的命名吧。

####介绍 log
```
enum log { }
```
我们使用 enum, 而不是 class 或者 struct 是有原因的。其中一个原因是我们永远不需要实例化一个 log。我们选择 case 而不是函数因为我们要确保我们实现的 log 是安全的。你很快就会知道我为什么这么说。

####case 的相关值
```
enum log {
    case ln(_ line: String)
    case url(_ url: String)
    case obj(_ any: AnyObject)
}
```

有些朋友可能不知道吧， `ln` (line) 曾经在 Swift 中出现过， 在 Swift 2.0 `print ()` 成为主要的 log 方式前， `println()` 是它的前身。我也写了一些其他的例子来证明 log 的扩展性。根据不同情况，我们包含了不同的相关值，因为当我们需要打 log 时，是需要有相应的输入的，对吧？同时你会注意到，我们省略了声明中的外部参数名，因为我们使用 case 名来描述该声明。

看看我们的进展如何吧：
```
print(log.ln(“Hello World”))
// ln("Hello World")
print("Hello World")
// "Hello World"
```

呃，的确是有用的，但是它绝不是一个合适的补充或替代 log 的选择。原因如下：
* 它依然很重；
* 在原始信息外面有多余的东西；
* 看起来讨厌；
* 甚至没有用到 emoji 表情；
* 简直弱爆了！

所以我们现在需要一个方法来修复这五件事，来让你乘上到达我所允诺的 log 之目的地的列车。 

####自定义操作符
```
postfix operator / { }
```
我会假定你们中的大部分人都没有过实现自定义操作符的需求，这很正常，我也是最近才开始用的，但其实这并不难。
我们自定义的操作符将会是*后缀*，因为我们希望它在 log 的代码后面，在操作符的左边，我们只需要一个输入。
我选择了『/』操作符，因为它是最接近注释语法而又不会实际创建注释的，还因为它是为数不多的不需要按 shift 键进行输入的操作符。
...我真的开始感觉我就像一个不断削减预算的政客。

####实现
```
postfix func / (target: log) { 
    switch target {
    case ln(let line):
        log("✏️", line)
    case url(let url):
        log("🌏", url)
    case obj(let object):
        log("🔹", object)
}
```
这个实现很像是声明，但是我们提供了一个函数体，增加了要求传入的参数为 `log` 枚举的限制， 这就是我说的写『更安全的代码』。另一个实现『更安全的代码』的部分是我们把 `log`声明为枚举而不是类或者结构体，因为枚举的 switch 语句一定很详细。每当我们添加一个新的 emoji 日志类型，我们必须同时在操作符的 switch 语句中包含它。
```
private func log<T>(emoji: String, _ object: T) {
    print(emoji + “ “ + String(object))
}
```
最后，我们实现了 log 函数，这简单得难以置信。它是私有的，因为我们不希望它在我们正在写的 Swift 文件外被访问。 它的第二个参数是一个泛型， 因为我们可能会传任何类型进去。

如你所见，它只是一个简单地把 emoji 表情和对象用一个空格连接起来的  `print` 语句。

####行动起来
```
log.ln(“Pretty”)/
✏️ Pretty
log.url(url)/
🌏 http://www.andyyhope.com
log.obj(date)/
🔹 2016–04–02 23:23:05 +0000
Maybe i should use a screenshot here instead?
```
这样就生效了，只需要两个额外的键程，我们成功地使特定的类型区分于 log 中的其余部分和塞满控制台的其他第三方库中的 log。但事情还没做完…

####性能提升
很多开发者都忽略了的一个事实是调用 print 实际上会降低你的应用的性能。在调试过程中代码中遍布大量的 print 完全没问题，但是在上架 AppStore 之前，你真的应该删掉它们。

> 『你的意思是我必须每次在提交前注释掉所有的 print , 然后再取消注释吗？』-- 你

#####预编译指令
Xcode 允许我们在每个工程中创建额外的配置。默认情况下 Xcode 为新工程提供了两种配置，Debug 和 Release。
在模拟器或通过 USB 连接的设备上运行你的 app 时，Debug  是默认配置，当你要将你的 app 上架，打包时使用的是 Release 配置。
我们将把我们的 print 代码用 Debug 预编译指令包起来，这样我们就不用每次打包时都注释/取消注释/添加/删除所有的 print 了。相反，我们将会告诉编译器『Yo! 请注意，只在我们不在 release 模式下时运行这段代码！』

####Build Settings
![](https://cdn-images-1.medium.com/max/800/1*wExNt9uLhE8ewadbCzTQCQ.png)
1. 点击 Project 导航图标；
2. 点击 Project 名；
3. 点击 Build Settings;
4. 搜索 "Compiler Flag"；
5. 展开 "Other C Flags"行；
6. 点击 "+" 按键；
7. 输入 "-D DEBUG"
最后，我们将把我们实际的 print 函数打包进我们刚才设置的预编译指令。

```
private func log<T>(emoji: String, _ object: T) {
    #if DEBUG
        print(emoji + “ “ + String(object))
    #endif
}
```

瞧！现在你的 print 语句只会在调试时运行。你可以通过[改变你的 build configuration scheme ](https://developer.apple.com/library/mac/recipes/xcode_help-scheme_editor/Articles/SchemeDialog.html)为Release 再运行你的 app, 但是不要忘了把它改回 Debug !

###Framework,  Carthage 和 Cocoapods 的支持
也许你会喜欢你在这里看到的内容，并会想：『如果Andyy再提供...就更好了。』，但是实际上这对 log 的实用性来说没有好处。原因是，如果我提供这三者之一，每次你想打 log 的时候，在你使用前，你都需要将框架导入你的 Swift 文件，这样做很傻，因为你在每次使用这个愚蠢的 log 把戏前都需要做一些额外的管理工作。这也是为什么那么多  NSLog 的替代品在 Objective-C下面工作地并不好。

```
import Log // This looks like 💩
```
###探索与使用
我为你们提供了一个  playground 用来测试你们刚才看的内容，同时还提供了一个 *log.swift *用来放进你自己的项目中。示例代码中有一些额外的你可以用在你的日常开发中的 log 示例。请慢用！

[示例代码](https://github.com/andyyhope/Blog_PrettyPrint)已上传 Github.

像往常一样，如果你喜欢你今天看到的内容，或者已经实现了它，请 [tweet](https://twitter.com/AndyyHope) 我。我喜欢读者的反馈，这会让我很高兴！



