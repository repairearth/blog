---
title: <i class='iconfont icon-zhuan' style='vertical-align:super;'></i> NPM版本管理
date: 2016-06-21 11:30:07
categories: Mark
tags: [NPM,Mark]
---

人老了，一是怀旧，二是健忘，所以记录一下。

## Semantic Versioning 2.0.0
NPM 中依赖模块的版本都需要遵循 [semver 2.0][1] 的语义化版本规则。

版本格式：主版本号.次版本号.修订号
版本号递增规则如下：

- 主版本号：当你做了不兼容的API 修改
- 次版本号：当你做了向下兼容的功能性新增 
- 修订号：当你做了向下兼容的问题修正

先行版本号及版本编译信息可以加到“主版本号.次版本号.修订号”的后面，作为延伸。

然后基于语义化的版本，我们在选择版本的时候就可以对依赖进行版本的控制：
```
dependencies: {
  "express": "3.x",
  "debug": "*",
  "express-session": "~1.0.0",
  "connect-redis": "1.2.3"
}
```
<!-- more -->
从例子中可以看到，有许多种选择版本范围的风格，可以在 [semver in npm][2] 上看到每一个不同风格的作用。 而在 `npm` 的依赖管理中，最常用到的几种是：

- `*`: 任意版本
- `1.1.0`: 指定版本
- `~1.1.0`: >=1.1.0 && < 1.2.0
- `^1.1.0`: >=1.1.0 && < 2.0.0

其中 `~` 和 `^` 两个前缀让人比较迷惑，简单的来说：
`~` 前缀表示，安装大于指定的这个版本，并且匹配到 `x.y.z` 中 `z` 最新的版本。
`^` 前缀在 `^0.y.z` 时的表现和 `~0.y.z` 是一样的，然而 `^1.y.z` 的时候，就会 匹配到 `y` 和 `z` 都是最新的版本。
特殊的是，当版本号为 `^0.0.z` 或者 `~0.0.z` 的时候，考虑到 `0.0.z` 是一个不稳定版本， 所以它们都相当于 `=0.0.z`。
## semver 的弊端
按照 semver 的规范：
所有的 breaking change 都需要升级主版本号。
向后兼容的新增功能可以只升级次版本号。
但是最终在实践过程中，许多包没有遵循该原则，或者是没注意破坏了该原则，导致：

- 新增功能带来了 breaking change
- 修复现有 bug 引入未知的 breaking change

由此引出了下一个话题：如何正确的选择依赖模块依赖范围。

## 如何选择依赖范围
### node 模块
在编写 node 的模块的时候，模块自身可能会依赖一些 dependencies, 然而我们并不想每一次依赖的 模块有任何小的 bug fix 的时候就要重新更新一次依赖，因此会推荐对大部分的依赖使用 `~` 前缀， 这样可以保证可以享受到这些依赖模块的 bug fix，并且不需要更新自身的版本。同时，也不会引入 一些 API 变更导致的无法预料的错误。当然，如果对一些完全信任的模块也可以考虑使用 `^` 前缀。
see: [koa][3]
### node 项目
然而在编写 node 的项目的时候，我们更加希望能够追踪到所有依赖的任何变动，因为项目也不会有 关于自身版本更新的烦恼，因此更加倾向于写死依赖的版本，这样如果有任何由于更加容易追踪 升级依赖导致的 bug。
see: [koa-todo][4]
## 如何选择发布模块的版本
看完编写过程中怎么样选择依赖的范围，回过头来看看，如果我们需要编写一个发布到 `npm` 的模块， 需要如何选择发布的模块的版本。
当刚开始开发这个模块，它的功能和 API 都会有较大的调整的时候，可以选择发布 `0.0.z` 版本。
而当一个模块的 `API` 基本已经成型，不过可能会有一些新的 API 增加的时候，可以选择发布 `0.y.z` 版本。
当一个模块的功能基本已经完全定下来的时候，可以选择发布 `x.y.z(x >= 1)` 版本。当模块发布到 `npm` 之后，就意味着已经正式准备好了，所以尽量把发布的第一个版本定为 `1.0.0`。
当修复了一些小的 bug 的时候，请升级 `z` 的版本号，这样可以让用户通过 `~` 和 `^` 前缀最快速的享受你的 bug fix。
当增加了新的功能的时候，可以选择升级 `y` 的版本号。
当变更会影响原有接口的情况下，起码要升级 `y` 的版本号，这样不会影响大部分依赖这个库的用户。
当重构了模块，从设计上就有很大不同的情况下，需要升级 `x` 版本号。
遵循这几条规则，这样用户在引入这个模块的时候就可以轻松的通过 `semver` 提供的验证机制来更轻松的使用你的模块。

原谅地址：[node.js 中的版本管理][5]

  [1]: http://semver.org/lang/zh-CN
  [2]: https://docs.npmjs.com/misc/semver
  [3]: https://github.com/koajs/koa/blob/master/package.json
  [4]: https://github.com/koajs/todo/blob/master/package.json
  [5]: http://deadhorse.me/nodejs/2014/04/27/semver-in-nodejs.html