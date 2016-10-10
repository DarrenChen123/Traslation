###Swift:如何优雅地使用 print ()?
####Log是一种时尚，而你是时尚设计师。
当 Swift beta 版横空出世时，社区中最具有好奇心的开发者们立刻赶上了这班车，开始利用这门语言的特性进行试验，并写下和讨论他们从中获取的经验。
关于 Swift, 有太多提到 Tailor Swift 的 Twitter 了以至于我没办法列举出最好的， 但直到现在这也很有趣，没有人知道为什么😂。另一件人们讨论得很多的就是 Swift 代码中可以使用emoji表情。
```
func combinedWeatherConditions(lhs: Int, _ rhs: Int) -> Int {
    return lhs + rhs
}
let 🌨 = -10, 🌤 = 10, 💧 = 0
if combinedWeatherConditions(🌨, 🌤) == 💧 {
    print(“😔 — No 🏂 Today.”)
}
```

>我纯粹是因为表情功能而开始写Swift的。—— 没有人这样说

这是一个新颖的功能，但你也许永远不会需要在你的代码里使用 emoji 或者用在打印语句中。但是，我们讲到了打印语句，新的控制台 `print `语句相比古老的前身 `NSLog` 已经被优雅地简化了。
####NSLog
作为初学者调试的主力，`NSLog` 用起来相当简单，是一个在运行时检测变量的值和通知开发者事件的合适的工具。你可以随心所欲地打印消息或者对象。但在我看来，默认的 `NSLog` 消息包含了大量的垃圾：
```
2016–04–02 09:15:25.660 Blog_Print[13053:3567169] Hello, cruel world. Why don’t you love me? Whyyy!!!
```
不要误会，我肯定原来写` NSLog `的工程师投入了大量的想法来造就一个良好的日志信息。我们把它拆开来看：
```
[日期戳] [时间戳] [工程名] [进程 ID: 线程 ID] [消息]
```
但是，的确，这相比于我们需要的日志信息来说典型地信息过剩了。日期/时间戳、进程/线程ID组合占用了大量空间，尤其是当你的控制台窗口在你的Xcode窗口的底部中间偏右时，因为空间本来就小，再打印这么冗长的信息就显得太小了。

####print
没必要从 `NSLog `简化，`print`从原始信息中去掉了冗余的信息。
```
Hello, beautiful world! I love you.
```
但是也许，只是也许，有时它的确有点太简单…我知道你会怎样想，请包涵。
>『这家伙一会抱怨 NSLog 太多， 一会又抱怨 print 太少？真难伺候。』—— 你

就像衣服，你在不同的场合和用途穿不同的服装。你不会在雪地里穿短裤和背心，对不对？
`print`应该没什么不同，有时你需要打印日期，或者 API 调用，而有时你会希望你的 log 在其余多余信息中显得突出，相信我，你会希望你的 log 显得突出的。这就是我们所讨论的时尚。
如果你在项目中使用过第三方库，你可能会注意到你的控制台充满了垃圾，各种垃圾，就像让 Wall-e（译者注：《机器人总动员》中的机器人瓦力）郁闷的那些垃圾一样。没错，说的就是你，Urban Airship。
####多多益善的表情
就像我刚提到的，使用表情符号是一个新功能，但它实际上是一个令人惊讶的有用的功能。当你通过控制台的输出分析一切时，你的打印语句中的表情符号对减少认知的负荷的调试实际上是一个巨大的增强。

>Pro tip: 
在 OSX 系统中，在任意文本框中点击 Ctrl + Cmd + Space 将会打开一个 emoji 弹出框。

![](https://cdn-images-1.medium.com/max/800/1*32Y_9OrQhKOMU6FjnCpMLQ.jpeg)

#####Strings
```
let string = "Emojis are life"
print("🔹 " + string)
// 🔹 Emojis are life
```
#####NSDate
```
let date = NSDate()
print("🕒 " + String(date))
// 🕒 2016-04-02 00:14:18 +0000
```

#####NSURL
```
let url = NSURL(string: "http://www.andyyhope.com")
print("🌏 " + String(url))
// 🌏 http://www.andyyhope.com
```

#####NSError
```
let userInfo = [NSLocalizedDescriptionKey: "File not found"]
let error = NSError(domain: "Domain", code: 404, userInfo: userInfo)
print(“❗️ “ + “\(error.code): “ + error.localizedDescription)
// ❗️ 404: File not found
```

#####AnyObject
```
let anyObject = UIColor.redColor()
print("◽️ " + String(anyObject))
// ◽️ UIDeviceRGBColorSpace 1 0 0 1
```

#####和同事开个小玩笑
```
let joke = "What is this... A center for ANTS?!"
print("🏫🐜 " + joke)
// 🏫🐜 What is this... A center for ANTS?!
```

![](https://cdn-images-1.medium.com/max/800/1*jg0ZyJOF0qzttmjl24hLgw.jpeg)

由于 iOS 9.1 有184种表情可供选择，将来还会更多，你的大多数日志的需求都能用表情来满足。

####实现
不要误会，上面的实现挺烂的…实际上它烂得 Hoover （译者注：一种吸尘器品牌）想把它扫走。把所有不是字符串的东西都包进 String 的括号里，还要找到合适的表情是一件很痛苦的事，如果你正在寻找的并不经常用到，通常不太可能找到。
我认为系统其实知道你在寻找哪个表情，故意把它藏在了第五个维度来惹你，让你觉得你快疯了。

>『也许便便的表情不存在了？或者它从来就没有存在过？』 —— 你

>『哦，等等，它在这里。我发誓我看过那里的？什么鬼！』 —— 你（十分钟后）

然而，这第一篇文章只是为了演示通过在你的日志中使用表情符号，来区分不同形式的日志，并减少认知负荷的效果。

如果你看完第一部分还能忍受，这篇文章的下一部分和最后一部分将正式实现，告诉你如何使这种表情日志的方式更有用。

这篇文章的[示例代码](https://github.com/andyyhope/Blog_PrettyPrint)可以在 Github 上找到。

第二部分将包含在 log 中使用表情的代码实现。敬请期待！
