---
layout: default
title: 程序语言的常见设计错误(2) - 试图容纳世界
---

之前的一篇文章里，我谈到了程序语言设计的一个常见错误倾向：[片面追求短小](http://www.yinwang.org/blog-cn/2013/03/15/language-design-mistake1)，它导致了一系列的历史性的设计错误。今天我来谈一下另外一种错误的倾向，这种倾向也导致了很多错误，并且继续在导致错误的产生。

今天我要说的错误倾向叫做“试图容纳世界”。这个错误导致了 Python，Ruby 和 JavaScript 等“动态语言”里面的一系列问题。Scheme 也算是“动态语言”，但是 Scheme 是一种非常严谨的语言，它直接源自计算机科学的源头 lambda calculus，比 C++，Java 都要严谨很多。所以我一般都避免把 Scheme 和 JavaScript 等语言相提并论，以免贬低 Scheme 的身份。实际上很多语言的设计者其实都是门外汉，根本没经过严格的训练，所以才会犯一些低级错误，却又自大得要命。比如有一阵子 Python 的创造者 Guido 在 blog 上扬言要把 lambda 从 Python 里去掉，说 lambda 比起有名字的函数根本没有什么优势，当年 Python 有 lambda 是因为一个 Lisp 程序员坚持，他为了照顾“公共关系”把它放进去了。其实他这样觉得的原因是他在 Python 里面实现的 lambda 根本就没有真正的 lambda 的能力。

不扯太远了，现在来说说 Python 和 JavaScript 常见的问题吧（Ruby 我不大熟，所以暂时不管它，你们自己用过之后可以对号入座）。我给 Python 写过一个静态分析器，所以我基本上实现了整个 Python 的语义，可以说是对 Python 了解的相当清楚了。如果你想知道这个代码的结构，可以在这里看到它的第一个版本的[源代码](http://hg.python.org/jython/file/11776cd9765b/src/org/python/indexer)。在设计这个静态分析的时候，我发现 Python 的设计让静态分析异常的困难，Python 的程序出了问题很难找到错误的所在，Python 程序的执行速度比大部分程序语言都要慢，这其实是源自 Python 本身的设计问题。这些设计问题，其实大部分出自同一个设计倾向，也就是“试图容纳世界”。

在 Python 里面，所有的“对象”都有一个“字典”（dictionary）。这个 dict 里面含有这个对象的 field 到它们的值之间的映射关系，其实就是一个哈希表。一般的语言都要求你事先定义这些名字，并且指定它们的类型。而 Python 不是这样，在 Python 里面你可以定义一个人，这个人的 field 包括“名字”，“头部”，“手”，“脚”，……

但是 Python 觉得，每个人都可以随时创建或者删除这些 field。所以，你可以给一个特定的人增加一个 field，比如叫做“第三只手”。你也可以删除它的某个 field，比如“头”。Python 认为，这更加符合这个世界的工作原理。有些人就是可以没有头，有些人又多长了一只手。

好吧，这真是太方便了。然后你就遇到这样的问题，你要给一个国家的人每人一顶帽子。当你写这段代码的时候，你意识中每个人都有头，所以你写了一个函数叫做 `putOnHat`，它的输入参数是任意一个人，然后它会给他（她）的头上戴上帽子。然后你想把这个函数 `map` 到一个国家的所有人的集合。

然而你没有想到的是，由于 Python 提供的这种“描述世界的能力”，其它写代码的人制造了各种你想都没想到的怪人。比如，无头人，或者有三只手，六只眼的人，…… 他们让这些人不声不响的偷渡到了你的国家。然后你就发现，无论你的 `putOnHat` 怎么写，总是会出意外。你惊讶的发现，这个国家居然有人没有头！ 最悲惨的事情是，当你费了几个月时间和相当多的能源，给好几亿人戴上了帽子之后，忽然遇到一个无头人，所以程序忽然当掉了。然而即使你知道程序有 bug，你却很难找出这些无头人是从哪里来的，因为他们来到这个国家的道路相当曲折，绕了好多道弯。为了重现这个 bug，你得等好几个月，它还不一定会出现…… 这就是所谓 [Higgs-Bugson](http://www.yinwang.org/blog-cn/2013/04/14/terminology) 吧。

怎么办呢？所以你想出了一个办法，把“正常人”单独放在一个列表里，其它的怪人另外处理。于是你就希望有一个办法，让别人无法把那些怪人放进这个列表里。你想要的其实就是 Java 里的“类型”，像这样：

    List<有一个头和两只手的正常人> normalPeople;

很可惜，Python 不提供给你这种机制，因为这种机制按照 Python 的“哲学”，不足以容纳这个世界的博大精深的万千变化。让程序员手工给参数和变量写上类型，被认为是“过多的劳动”。

所以你看到了，试图容纳世界的倾向，没带来很多好处，没有节省程序员很多精力，却使得代码完全没有规则可言。这就像生活在一个没有规则，没有制度，没有法律的国家，经常发生无法预料的事情。这个国家到处跑着没有头，三只手，六只眼的怪人。这是无穷无尽的烦恼和时间精力的浪费。

如果你想生活在这样的国家，那么就请使用 Python 和 JavaScript。