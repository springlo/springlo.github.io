---
title: 使用Guard的三个注意
---
相比一些以前的写法，这儿有更精简的方式。分享三个有用的guard技巧，使你的代码更简洁也更容易理解这种方式构建的功能。当然这不是一个关于一般的编码风格或编码指引，更多的关注guard如何简化代码。


## 1.绑定和条件判断

###  嵌套

代码很简单，一看便懂:
{% highlight swift linenos %}
//无意义的大量代码，显示代码结构如何嵌套看起来混乱
if let a = a() {
  let x = b(a)
  x.fn()
  if let u = x.nxt() {
    let ux = u.brm()
    if let uxt = ux.nxt() {
       perform(uxt)
    }
  }
}
{% endhighlight %}

更简洁的：
 {% highlight swift linenos %}

guard let a = a() else { return }
let x = b(a)
x.fn()
guard let u = x.nxt() else { return }
let ux = u.brm()
guard let uxt = ux.nxt() else { return }

perform(uxt)

{% endhighlight %}


###  模式绑定
以上代码如果你使用一个枚举 enum，效果更好。想想我们是如何处理以下代码:
{% highlight swift  linenos%}

protocol NotificationListener {
  func handleNotification(notification: Notification)
}
enum Notification {
  case UserLoggedIn(user: String, date: NSDate, domain: String)
  case FileUploaded(file: String, location: String, size: Int, user: String)
}

struct FileUploadHandler: NotificationListener {
  /**
    执行通知处理，把上传的文件移动到临时文件夹
  */
  func handleNotification(notification: Notification) {
    guard case .FileUploaded(let file, let location, _, let user) = notification
    else { return }

    if user == self.currentUser {
       self.moveFile(file, atLocation: location)
    }
  }
}

{% endhighlight %}

在使用guard的情况下实现了2个功能：

1.它确保处理通知仅适用于FileUploaded通知，而不是ISUserLoggedIn通知。

2.它结合了枚举的所有相关值到当前范围，使得我们很容易使用数据。


### 条件判断
不管怎样，使用guard 我们更能简化例子，像这样：
{% highlight swift  linenos%}

struct FileUploadHandler: NotificationListener {

  func handleNotification(notification: Notification) {
    guard case .FileUploaded(let file, let location, _, let user) = notification
    where user == self.currentUser
    else { return }
    self.moveFile(file, atLocation: location)
  }
}
{% endhighlight %}


并且还可以多个where 并用：

{% highlight swift  linenos%}

import Foundation

func confirmPath(pathObject: AnyObject) -> Bool {
  guard let url = pathObject as? NSURL,
  let components = url.pathComponents
    where components.count > 0,
  let first = components.dropFirst().first
    where first == "Applications",
  let last = components.last
    where last == "MyApp.app"
  else { return false }
  print("valid folder", last)
  return true
}
print(confirmPath(NSURL(fileURLWithPath: "/Applications/MyApp.app")))
// : valid folder MyApp.app
// : true
{% endhighlight %}


### 嵌套枚举

{% highlight swift  linenos%}
    enum SidebarEntry {
        case Headline(String)
        case Item(String)
        case Seperator
    }
    
{% endhighlight %}
定义一个数组:
{% highlight swift  linenos%}
    [.Headline("Global"),
        .Item("Dashboard"),
        .Item("Popular"),
        .Seperator,
        .Headline("Me"),
        .Item("Pictures"),
         .Seperator,
         .Headline("Folders"),
         .Item("Best Pics 2013"),
        .Item("Wedding")
    ]
    
{% endhighlight %}
每个 Item 都有一个不同事件：
{% highlight swift  linenos%}
enum Action {
  case .Popular
  case .Dashboard
  case .Pictures
  case .Folder(name: String)
}

enum SidebarEntry {
  case Headline(String)
  case Item(name: String, action: Action)
  case Seperator
}

[.Headline("Global"),
 .Item(name: "Dashboard", action: .Dashboard),
 .Item(name: "Popular", action: .Popular),
 .Item(name: "Wedding", action: .Folder("fo-wedding")]
{% endhighlight %}

现在我们用guard 试试发布一个文件夹：
{% highlight swift  linenos%}
func publishFolder(entry: SidebarEntry)  {
  guard case .Item(_, .Folder(let name)) = entry 
  else { return }
  Folders.sharedFolders().byName(name).publish()
}
{% endhighlight %}
这是一个层次结构比较复杂的模型，但仍然能够匹配，甚至还能匹配更复杂的嵌套类型。

### 单行返回
当你在else 标签里返回前，希望能够做一些其他的处理：
{% highlight swift  linenos%}
guard let a = b() else {
   print("wrong action")
   return
}
// or
guard let a = b() else {
   self.completion(items: nil, error: "Could not")
   return
}
{% endhighlight %}
当你的返回是Void，还可以这样处理：
{% highlight swift  linenos%}
guard let a = b() else {return print("wrong action")}
// or
guard let a = b() else {
   return self.completion(items: nil, error: "Could not")
}
{% endhighlight %}

### try? in guards
{% highlight swift  linenos%}
guard let item = item,
   result = try? item.perform()
else { return print("Could not perform") }
{% endhighlight %}

### 综合使用

最后，让 let ， case 在一个guard 中使用试试：

{% highlight swift  linenos%}
guard let messageids = overview.headers["message-id"],
    messageid = messageids.first,
    case .MessageId(_, let msgid) = messageid
    where msgid == self.originalMessageID
    else { return print("Unknown Message-ID:", overview) }
{% endhighlight %}