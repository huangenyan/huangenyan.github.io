---
layout: post
title: Android 和 iOS MVC 框架的简单比较
category: Android 开发
tags: [ 'android', 'programming' ]
---

最近在开发 Native Android App，在这之前我可以说是对 Android 开发毫无经验，临危受命只能自己网站搜罗各种资料从零学起。凭借着之前 Web 和 iOS 的经验，上手速度还算可以。从写下第一行 Android 代码到现在已经过去了一个月的时间，有了一些或深或浅的心得体会，简单整理一下吧。

在 MVC 架构下，iOS 和 Android 代码组织的整体思路是差不多的。Android 里有两大种 class 扮演着 View Controller 的角色：Activity 和 Fragment。

Activity 可以说是 Android 里最重要的概念，它代表了一个可以“独立存在”的活动，这里之所以强调“独立存在”，是因为 Android 里每一个 Activity 不仅可以在自己的的 App 里运行，而且可以在其它 App 里运行。比如说当其它 App 需要拍照时，它实际上就是调用了系统自带的拍照 App 的一个 Activity。不仅如此，如果有人开发了一个拍照的 App，那其它 App 同样可以调用个 App 的拍照的 Activity，而不使用系统相机来拍照。这很像 iOS 里的 App Extension，只不过在 Android 里可以被别的 App 调用的 Activity 和不可以被别的 App 调用的 Activity 都是基于相同的父类。

尽管如此，在大多数情况下，我们并不需要自己开发的 Activity 能够被别的 App 调用，甚至我们对于一个 Activity 是由谁发起的在开发时都一清二楚。这样的 Activity 实际和 iOS 里的一个 View Controller 的区别就很小了。但在开发时，尽量的让一个 Activity 和 Calling Activity 解耦仍然是一种好的实践（当然 iOS 里也是如此），一来方便我们进行单元测试，二来便于拓展。具体来说我们应当尽量保证任何一个 Activity 都可以作为 App 的第一个 Activity 启动而不会崩溃。

另外一种和 iOS 里 View Controller 很相似的东西是 Fragment，从名字也可以猜到它就是 View Controller 里的 View Controller。合理的使用 Fragment 可以减小 Activity 的代码量并且让 Activity 中的某些功能能够复用。相信这其中的道理也不难理解，也就不赘述了。

讲完了 View Controller，下面说一说 View。在这方面 iOS 和 Android 的区别就比较大了。首先，Android 里没有 Story Board （当然没有……），视图的定义是通过 XML 完成的，这个有点类似于 iOS 里的 nib。但不同之处在于 Android 里的视图文件是不和 Activity 绑定的。每一个视图文件实际上就是一种资源，Activity 在创建的时候可以决定自己把哪一个视图文件作为自己的 content view。在 Android 里有一个 inflate 的概念，指的就是将静态的 XML 视图文件转化为一个 View 实例的过程。视图文件作为一种资源，其本身可以是一整个页面，也可以是整个页面的某一部分，如何使用和组合都是由控制器决定的。

在构建视图布局这个问题上，Android 比 iOS 提供了更多种选择，在 iOS 上现如今我们几乎只有 Autolayout 这一种方法，而 Android 里与之相近的 Constraint Layout 只是众多布局方法中的一种。此外比较常见的方法还有 Linear Layout，Relative Layout 等，在布局哲学上 Android 和 Web 是比较相近的，比如大量使用 margin，padding，定义长高时可以指定为 match parent 或者 wrap content 等。众多的布局方法让构建视图更加方便，对付不同尺寸也屏幕也更加得心应手。这一点可以算作是 Android 开发的一个绝对优势。

在数据绑定方面，Android 和 iOS 也有所不同，而区别主要体现在对于 List View（相当于 iOS 的 Table View）的数据绑定上。对于单一视图的绑定，比如 Text View 等，实际上二者的处理方法是差不多的，都是在控制器中声明一个相对于视图中元素的成员变量，然后改变对应的属性，具体方法也就不在赘述。而对于 List View 这种视图，Android 中使用的是 Adapter。Adapter 说白了就是一个可以将一堆数据转化为一堆视图的实例，而具体的转化方法实际上和 `TableViewDataSource` 也有几分相似之处，在 Adapter 中有类似于  `getView(int poisition)` 这样的方法，Android SDK 已经为我们提供了一些实现方法，我们也可以通过自己继承对应的类来实现自己的 Adapter。一般来说，SDK 自带的 Adapter 已经可以满足很多常见的 List View 的展示方法，而即便要自己实现也通常会把 Adapter 声明在一个单独的文件中，这和 iOS 里通常让控制器自己实现 `TableViewDataSource` 是不同的。此外，在 Android 里有另外一种展示动态数量数据的视图叫做 `RecyclerView`，它就有点类似于 iOS 里的 `UICollectionView`，让我们可以自己整个视图的布局结构，在 `RecyclerView` 里，我们需要使用到一种新的设计模式，叫做 View Holder，以此来增加视图绑定数据时的灵活性。具体的做法是将每一个 Cell View 封装金一个个 View Holder 中，并将 `getView(int position)` 分离成两个单独的方法，分别为 `onCreateViewHolder()` 和 `onBindViewHolder(int position)`，`RecyclerView` 自己会在适当的时候调用者两个方法，让我们自己不必再关心当前使用的 Cell View 是复用来的还是全新的。`ViewHolder` 中提供了 `getAdapterPosition()` 这样的方法，方便我们在回调中清楚当前这个 `ViewHolder` 在列表中所处的位置。

当然，除了 MVC 之外，二者在系统层面也会有很多相似之处和不同之处，但这方面不是三言两语能说清楚的（主要是我也没有精力把 Android 的所有服务都看个遍……）。所谓具体情况具体分析，我也就懒得说太多了。