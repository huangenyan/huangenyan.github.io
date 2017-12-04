---
layout: post
title: 全栈开发时项目配置中的常见问题，目前解决方案，以及理想解决方案
category: Web 开发
tags: [ 'javascript', 'programming' ]
---

这两天写了一个非常小的全栈项目，大概的作用就是为公司将要进行的一些大项目提供一个数据输入平台。简单来说需要一个前端网站展示界面，接收输入，然后后端使用 Node.js 把输入的数据存到数据库。项目总共也就3个页面，写起来也就一天的事情，但本着使用最佳实践的原则，我仍旧尝试了使用目前最火的一些工具来构建项目，包括但不限于 Webpack，Vue.js，Babel，JSLint，Jest 等。但最后折腾了半天发现用的工具越多，项目反而变得越混乱，最后索性把这些东西全都扔掉，用最原始（但也稍微高级一些）的方法搞定：用 pug 去写 HTML，手写前端 ES5 的 JavaScript 和后端 Node.js ES6 的 JavaScript。发布时就在服务器上把整个 repo 拉下来，然后直接用 pm2 运行。这种做法当然有很多问题，但应对小项目还是足够的。从这个项目构建的过程中，我再一次思考了当前全栈开发过程中会遇到的各种问题，在此总结一下。

## 开发语言与运行语言不同

目前全栈开发，源代码通常由使用最新的 JS 写成，比如 ES2017。但是无论是浏览器（前端）还是 Node.js（后端），都无法直接运行源代码，所以要使用转译工具（Babel）将源代码翻译成运行语言再运行。除此之外，各种库甚至会创建自己的语言，比如 React 的 JSX，同样需要使用 Babel 进行转译。在这种情况下，JS 项目的开发流程更趋近于移动端的开发，只不过是把编译的过程变成了转译的过程，把编译器变成了 Babel。既然如此，那需要明确的一点就是需要把源代码和运行时的语言进行严格的区分，运行时的语言是开发的产物，不应当放入代码仓库，这一点相信大部分人都可以做的很好。然而，除了代码之外，在全栈开发过程中，HTML，CSS 以及资源的管理同样是非常重要的一部分。甚至很多项目连 HTML 和 CSS 都要使用另外一种语言进行书写（Pug 或者 SCSS）。可以说，这种开发语言的运行语言的不同正是导致目前全栈项目配置复杂的最主要原因，开发者虽然有很多选择，但因此导致了每一种选择都需要开发者自己去正确配置各种工具。

就目前的情况来看，虽然每一种语言都有各自的工具将开发语言转换为运行语言，但这些工具如何组合却没有一个公认的答案。究其原因，是因为在组合这些工具时有一系列问题要解决，而现在没有一个工具可以把所有的问题都解决好。

那，到底有哪些问题呢？

## JS，CSS，HTML 三者间的相互依赖

在移动端开发的过程中，视图和控制器之间如何关联是有固定的方法的，比如 iOS 中的 `IBOutlet` 以及 Android 中的 `findViewById`。编译器会把这种在代码中的关联转换为 App 中各个组件中的关联，开发者不必关心这个问题。但在前端开发中，这种代码中的关联和运行时的关联需要开发者自己解决。比如说，若我们使用 Pug 和 SCSS 来书写 HTML 和 CSS，我们并不能在 Pug 里写类似于 `link(href="main.scss")` 这种代码，因为这种在源代码中的依赖关系不会自动转化为生成的 HTML 和 CSS 的依赖关系。究其原因，是因为 Pug 并不知道我们是使用 SCSS 来写 CSS 的，而且这种转化本身也不应该是 Pug 的工作。理想情况下，这种源代码文件的依赖关系可以自动转换为生成文件的依赖关系，但目前还没有工具可以做到因为各个工具之间缺乏足够的沟通。

## 全 JS 化无法根本解决问题

对于上边提到的这个问题，现在有一种解决方案是“全 JS 化”，大意是整个 HTML 由 JS 生成，例如在 JS 里可以 `require('style.scss')`，然后生成的 HTML 就自动包含 `<link href="style.css">`，现在 Webpack 通过不同的 loader 已经基本可以做到这一点。这种方法的本意是通过统一语言来解决不同语言之间依赖困难的问题，但这个方案暂时还不能处理类似 `require('https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css')` 或者 `require('https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/js/bootstrap.min.js')` 这样的情况。此外，对于全栈开发，这个方案对于 SSR 和 CSR 的混合开发非常不友好，Webapck 只负责解决前端依赖，然对在 SSR 时如何解决依赖束手无策，仍然需要开发者自己关心。

### 再多说一点 Webpack

可以说 Webpack 给前端开发带来了很多改变，它要解决的问题正是从源代码依赖到运行语言的依赖的转换，这个问题无疑是当前前端开发中非常重要的。然而，Webpack 解决这个问题的手段是利用 JS，这一点我实在是无法完全认同。在前端开发中，HTML，CSS 和 JS 可以说是相互独立，同等重要，将三者统一为一种必然会带来很多不必要的麻烦。Webpack 中很多 loader 的使用 （style-loader，html-loader 等）让开发者深陷这种大一统的泥潭。在这里，我的建议是将 Webpack 专注于 JS，对于 HTML 和 CSS，应该寻求更好的解决方案。换句话说，Webpack 是构建 bundle 一部分，而不是全部。

## 理想的方案

首先要说，这个理想的方案目前并不存在，毕竟对于全栈项目，要考虑的事情有太多，想要做到面面俱到十分困难。根据自己的实际需要选择合适的方案正是当前全栈开发环境下对开发者的要求。然而，倘若有一种方案能解决所有问题，那大家一定会很开心的。

理想的方案应该支持以下几点：

1. JS，HTML，CSS 支持使用各自的方言。这一点自不必多说，大家都希望写 ES2017 胜过 ES5。

2. JS，HTML，CSS 维持相互独立。这一点主要是为了 SSR 和 CSR 混合开发考虑，在 SSR 和 CSR 时可以使用同一个 Pug 模板的话会方便很多。

3. 源代码中的依赖自动转化为运行环境中的依赖，包括 SSR 和 CSR。要解决这一点，实际上我们需要在后端也有一个类似于 Webpack 这样的工具，在后端代码中需要的资源可以自动打包到后端的运行时库中，或者自动复制到 public 目录下，然后运行时库中的链接会做出正确的变化。

4. 一键打包，生成的东西放到服务器上就能用。