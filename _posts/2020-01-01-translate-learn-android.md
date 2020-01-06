---
layout:     post
title:      "(译)零基础新手如何入门Android开发"
subtitle:   "How to Learn Android Development as a Complete Beginner"
date:   2020-01-01 21:05:00 +0800
author:     "Nathan"
tags:
    - 翻译

---

> 我不私藏好文章，我只是好文章的搬砖工
>
> I am not a good article taker,  but a sharer!

在Face-book和其他社区，我经常看到很多人在询问该从哪开始学习Android开发。在可以获得大量资源的前提下，对于如何以“正确”的方式开始学习的争论也不足为奇。目前不仅有很多**书籍**，**视频**，**博客**以及还有不计其数的**文档资料**。还有免费的资料，收费的资料，以及一些价格不菲的资料。另外还有一些**简单的内容**，**很难懂的内容**，以及**完全看不明白的内容**，因此你很难评估这些内容会不会令你进步还是会浪费你的宝贵时间。在这篇文章里，我想要为零基础开始的同学提供一个入门Android开发的指导，这样的话，当任何人在任何地方寻求帮助时，我都能直接引用啦。

本文并不会教你任何实际的编程。相反，我会告诉你一些你需要(或者不需要)的入门**资源**，以及阐述当你入门后如何进一步自学。

按照惯例：本文仅供参考。对于一个比较好的方式来学习编程的观点是基于我个人意见。同时强调，我本人不是一个资深开发人员。我的粉丝经常问我该如何学习我视频教程中的内容，现在你可以知道答案啦。我是一个100%自学的，因此本文也是针对那些也想通过自学的朋友。本文不是一个直接告诉你一个具体课程的广告帖，而是我诚恳的意见和我一个人的学习方法。

那就开始吧。

### 学习原生Android开发是否还有意义？是不是使用React Native 或者是Flutter更好？关于Fuchsia呢？它会很快取代Android的，对吗？

Android和iOS是目前市场上最大的两个移动操作系统，很多公司都想把他们的apps同时运行在那两个平台上，以达到最大的用户数。出于这样的原因，他们必须考虑是否分别开发两个"**原生**"apps，还是使用一个跨平台框架来取代原生。开发一个原生app意味着你对于不同的操作系统---Android/iOS---需要使用特定语言以及特定开发工具，然后所写的代码**只会运行在一个平台上**。与此同时如何你想要让你的app也能运行在其他操作系统上的话，那么你不得不使用特定的语言来从头开始写。当然，这就非常烧钱而且耗时了，尤其是当它需要其他开发小组帮忙的时候。跨平台框架比如Ionic,Xamarin和React Native能够让你只写一次代码而可以运行**在两个操作系统上**。我不想在这里讨论它们是如何工作的细节，但是所有这些框架都会有**缺点**比如性能差，特性少，安全问题以及在开发流程上更容易出现压测和其他问题。正是由于这些局限，**原生app总是远超过**那些简单功能之上的框架。

**Flutter**是另一个跨平台框架，由谷歌亲自开发，很显然它能够开发出性能非常出色而且UI非常好看的apps，而且开发流程上相比其他框架也更快更简单，并且拥有质量非常好的文档。Flutter刚刚发布1.0稳定版(注:截止目前已经是1.12了),而且得到了谷歌的大力推广，但是Android社区似乎并不了解原因。有些人说Flutter就是未来，其他人说谷歌很可能在几年内砍掉这个项目，也有人说它不会比目前发挥太大作用的。同样，Flutter也不能使用原生app所拥有的所有特性和相关库。

简而言之：

如果你想要学习移动开发的主要原因是**业余爱好**或者你想要开发一个主要用于展示内容而不是复杂功能的app用于**初创企业**的话，你可以考虑马上学习其中一个**跨平台框架**(可以略过本文了)。如果你想要像程序员一样*工作*以及增加你**获取一个工作**的机会，你应该打个稳固的基础并且先要学习**原生**Android开发。如果你仅仅想要开发一个Android的app并且根本不关心iOS的话，你也应该选择后者。

你也必须记住截止到现在，找到清晰明了的教程，代码示例和回答你用Flutter写的程序问题的答案相比原生程序是非常难的。作为一个新手，这会使你**学习过程**毫无疑问更加耗时而且很有挫败感。

如果你关注谷歌相关的新闻，你也许也会听说过**Fuchsia**。Fuchsia是谷歌正在开发的一个全新的操作系统，它有可能在将来*替代*Android，但是同样，没人能确认这个事实。但是即使真的会发生，那也会需要好多年才会成为现实。即使到那时，Android app也会跑在Fuchsia系统上(已经有迹象证明)，因为谷歌丢不起上百万的Android apps在那和疏远所有的Android开发者啊。

在这个节骨眼上，不要对这些新框架过于担心。原生Android开发不会马上过时的，而且一旦你有了经验，学习其他语言或者框架也会变得简单些。**因为你不小心选择了错误的入门语言或者平台而导致你以后不得不从零开始的情况是不可能的**。通过理解你自己写的或者其他人写的代码中内在逻辑，数据结构和设计模式，你正在学习的**构建软件工程师技能**超过了你正在工作的语言或者是框架本身。只要你是一个初学者，你就挑选一个方面然后坚持去做就行了。

### 好，我想学习原生Android开发！那我应该从Java还是Kotlin入手呢？

现在，我们已经决定走原生开发路线(至少我认为是这样的，因为你已经读到这里了啊)，我们仍然需要在2种语言中选其一，因为原生Android apps开发不仅能用Java编写。

你可能已经听说过**Kotlin**已经在谷歌2017年的I/O大会上作为官方"第一"语言"用于原生Android开发。Kotlin是一个现代编程语言并且相比Java拥有很多优势，比如更简洁的语法，空指针安全(意味着很少会down掉)以及其他一些使编码更容易的特性。基于以上，理论上你在开发Android apps时根本不需要学习任何Java相关内容。所以，作为一个新手，**你应该抛弃Java而拥抱Kotlin吗**?或者你应该从优秀的Java开始？又或许两个一起学？详细讨论，请点击**这里**，但是如果你不想跳转到那篇文章，如下是简要总结。然而，如果你在Java和Kotlin之间仍然很难选择的话，那就继续，把本文读完去寻找答案吧。

总结：**以Java起步**。对于Java有相当多的学习资源而且它仍然是非常广泛传播的语言。学习Kotlin的同时不学点Java的话不是很可行的，所以你不得不同时一起学，这样只会增加更多的困惑。排除现在关于Kotlin的炒作，打好基础，尤其是当你考虑将来可能从事**Android开发以外的编程工作**时。Kotlin，普及速度非常快，在Android开发中正扮演着非常重要的角色，但在其他领域就不是这样了。很多大公司的代码库有很大一部分是用Java而不是Kotlin编写的，因此空缺职位中的大部分是留给Java而不是Kotlin的。现如今大多数的Android相关工作都会要求你有Java技能，而Kotlin也仅仅是锦上添花。尽管谷歌大力推广Kotlin，但是他们不打算放弃Java的支持，这两种语言都是同样平等有效的一流语言，他们也可以做另外一种语言能做的事情(尽管Kotlin也许会做的更简单些)。

当你对Java感觉轻车熟路的时候你以后仍然可以继续学习Kotlin。这是完全没问题的。正如我以前提到的，学习编程并不是要记住一门特定的语言，因为语言只是一个工具。当你对Java足够了解时，学习Kotlin最多需要几周的时间，而且两种语言都可以并存于同一个项目中，所以过度应该很容易。

### 但是我不知道从哪里开始！我应该买书还是买课程？我应该按照说明文档一步一步走吗？

一般来说，学习编程的最佳方法是构建自己的项目，并研究在此过程中出现的每个步骤和问题。为此，你不需要任何书籍或课程，在我看来，它们甚至是有坑的。我将在本文后面更详细地讨论这个问题。

然而，如果你刚刚开始，你可能根本不知道从哪里开始。你甚至不知道如何安装像Android Studio这样的IDE(一个集成开发环境，可以编写代码的地方)。在这个阶段，试图靠自己用谷歌搜索弄明白所有的问题是非常困难和令人困惑的。出于这个原因，我建议你使用一本好的、结构良好的初学者书籍或课程来学习那些**绝对基础的知识**。

我开始读的那本书是一本纯Java的书，叫做《Head First Java》。它很好，但它并没有教你任何关于移动开发的具体内容。当我意识到我想要走Android路线时，我寻找了一些Android特有的资源，最后得到了关于**Udacity免费Android初学者课程**，我个人认为这非常有帮助而且做得很好。下面是本系列第一部分的[链接](https://www.udacity.com/course/android-basics-user-interface--ud834)。这些视频课程是由谷歌制作的，所有的部分都是免费的，你不需要任何Java经验来开始。**同时学习Java和Android是没有问题的**，因此，您不需要任何进一步的准备(也不需要购买Head First Java书籍)。这些课程将教你开始时所需要的一切，包括非常基础的东西，比如什么是类、变量和方法，以及如何构建布局。当然，如果您想更轻松一些的话，可以先学习一点简单的Java，但这不是强制的。同样的Udacity课程也有付费版本，但只有在你想获得纳米学位(这是Udacity的某种认证)时才需要这些课程。

还有其他不错的Android入门书籍和课程，但现在我真的不明白为什么你要花30美元来买你可以不花钱就能获取的东西。免费的Udacity课程已经足够了，而且它们的质量非常高。在写这篇文章的时候，他们已经2岁多了，但是因为他们涵盖了基本内容，这应该不是问题，因为基本的东西不会改变太多。当然，我不知道他们会保留多久，他们可能会在某个时候删除视频。**如果发生了这种情况，只要选一本评价好的书或课程开始学习就行了**。你选择哪一个并不重要，因为它们只是让你开始。真正的学习是靠你自己，稍后你就会看到。

另外，不要只是被动地看课程或读一本书，而是要通过把例子码出来并且要做相应的练习来训练自己。即使你认为*当*你看的时候都已经理解了所有的东西，但当你第一次尝试在没有任何帮助的情况下编写代码时，你会突然什么都不记得的(注：也即是不要眼高手低)。这很正常，我们都有过这样的经历。**你只能通过反复编写大量代码来学习编程**。

话虽如此，我还是建议你尽快完成课程。在我看来，这些冗长的形式只是一种太被动的学习方式。当有人手把手告诉你该怎么做时，你会感觉很轻松，但这也不是很有效。**你对一个问题或者概念研究越多，你就了解的越深入**。你不仅可以直接照着课程中的代码示例敲出来，还可以在没有任何帮助的情况下自己敲出来弄明白，然后在遇到问题时只查看解决方法(或直接下一步)，从而提高对内容的熟悉程度。更好的方法是，根据你课程中的示例，使用不同的类、方法和变量名，稍微更改下布局并编译查看结果的不同。这样的话，你就增加了难度级别，并且逼着自己思考的更多了。如果你喜欢这样，那就在整个课程中都这么做，慢慢来。然而，你很可能很快就会**感到无聊和沮丧**，而编程练习就会感觉像是在做数学作业一样。如果出现这种情况，而你又犯了拖延症，那就把剩下的课程大致浏览一遍，尽快完成，以便**对概念有一个总体的了解**。你可以在此基础上想办法转成令人兴奋的学习方式。当我读完"Head First Java"后，我都不能解释或者重新还原其中的场景，但是我知道有个东西叫做"接口"和"构造函数"，以后有机会就会更深入的去了解。我也没有完成所有的Udacity的课程，因为我感到太无聊了。

精品课程的另一个问题是，你遵循的是一个人或几个人的方法和观点，这让你在以后学习更高级的主题时，有一种狭隘的观点。通过博客文章、Youtube教程和在线讨论等更简短内容的方式，您可以看到不同开发人员使用的许多**不同方法**，正是因为您可以阅读和观看更多的内容，使你可以获得**更广泛的内容输入**。而且，书籍和课程很快就会过时，因为编程世界发展很快。正如前面提到的，课程越基础，就越不容易过时，因为基础的东西不会改变太多。当然，较短的教程也会过时，但是因为您可以轻松地观看其中的许多教程，所以它们可以让您更好地了解随时间变化的内容和保持不变的内容，而不是短暂的"快照"。

就我个人而言，我很少购买或观看免费的编程课程(或书籍)。我对花钱购买学习资源没有意见，但我总是发现谷歌搜索和更短的教程是一种更有效的学习方式。

### 我该如何自学？

移动开发的最伟大之处就是它非常容易编出一些"真实"的东西。不仅仅是控制台的输出，而是你可以实际使用并且能安装在你手机里的应用程序。我推荐的Udacity课程能够快速教会你如何在Android Studio的XML编辑器创建一个包含文本、图片和按钮的简单布局，然后向其中添加一些基本功能。这比花时间来解决无聊的书本练习，比如试图用if语句还创建圣诞树要有趣的多。一旦你掌握了基础知识(例如类、变量和方法是如何工作的，以及如何构建一个简单的布局)，你就不再需要别人手把手教你了。相反，你应该会很快开始创建属于你自己的小项目--真正的应用程序--并在这个过程中学习到所有你需要知道的知识。根据你学习初级课程的程度，从一些适合你当前技能水平的东西开始，比如像一个只有两个数字相加的计算器，或者是一个跳转到另一个页面的按钮，然后通过实现越来越复杂的功能来逐步增加难度。尽情享受编码过程，不要犹豫删除一个项目并重新开始。你的第二个或者第三个项目很可能是一个可以在数据库存储数据的待办列表app或者是一个功能齐全的计算器。理想情况下，你应该开发一些令你兴奋的应用程序，或许还能够解决你自己的问题，但不要在寻找正确的想法之中停滞不前。如果你想不出啥新的app，那就复制一个已经存在的好了。我建议你在开始时就跳过负责的网络特性，以及一些高级概念，比如软件架构和依赖注入，并以简单的离线功能开始，比如如何实践不同类型的菜单和控件，如何处理输入，或如何保存和加载手机上的数据，这样在关闭应用之后数据不会丢失。我也建议你在**需要的时候**再学习一些东西。你不必在这个理论还没有应用到的时候就坐下来开始逐个学习，只要在实践中学习就行了。例如，我不理解接口、泛型或lambdas表达式，尽管我试图从理论上理解它。只有我实际在代码中*使用*他们并*体验*它们是如何工作的时候才会开始去了解。

如果你想大致了解我学习不同主题的顺序，可以看看我在Youtube上的[视频](https://www.youtube.com/channel/UC_Fh8kvtkVPkeihBs42jGcA/videos)，请按时间**倒序排列**。因为当我开始制作视频时，我是一个完全的新手，这给你一个我学习路线的很好概述。从中获取一些灵感吧。

接下来是最重要的部分。这就是学习过程如何展开。

你看到了一个有趣的特性，你想要试试并加到你的app中。也许你想知道如何将下拉菜单添加到标题栏中或者是如何显示一个倒计时器。你的第一个目的地应该总是**谷歌搜索**。不要去Facebook群主或者其他社区中问"我该如何增加一个菜单到app标题栏上？"。这是一个又慢又懒的方法，最终并不会教你多少。相反，输入"android studio 如何在标题栏上增加菜单"到谷歌然后浏览相应的结果。官方的**developer.android**页面，很明显应该是你的首选，但是它们并不完美，所以务必也查看下其他结果。**随着时间的推移，你将了解哪些资源有用，哪些资源没有用**。

当你发现一个教程或者说明文档的时候，你要尝试自己按照做出来！不要只是读一下或者看一遍，因为你记不住多久。你的首要目标应该是让那个功能能够在模拟器或者真实的设备上运行起来。是否能马上理解所有内容并不重要，以后你仍然可以继续学习相应的细节。耐心点，别灰心。有时这个过程是非常简单的，但经常会充满各种坑并且令人困惑。当你遇到问题的时候，而且你不知道如何解决它，那么你不要马上去Facebook让别人帮你解决。相反，要尽可能的自己解决。看到错误信息了？**复制到谷歌里**！很有可能，其他很多人在你之前已经遇到了相同的错误信息，并且已经找到了解决办法。不知道某个方法或者属性是什么或者如何工作的？**拷贝到谷歌里**！我觉得你能看到一个专题在那。作为一个程序员(不仅仅是一个初学者)，谷歌搜索是你最好的朋友。如何整个教程看起来好像过时了或者不准确了，那就换另一个。这里是一篇[博客](https://codinginflow.com/google-programming-questions)，我在这里会更详细地解释我是如何用谷歌搜索到我想要的东西的。

我的YouTube订阅者(以及我生活中的朋友们)都会经常问我是如何学这么多东西并产出这些视频的。有些人似乎认为我找到了某种超级全面的指南，或者是我有非常好的老师。我没有老师，我也不买课程而且我也没学过计算机科学。我就是坐在家里，用谷歌学习这些东西。其他没别的了，而且它还完全免费。

如下是一个重要事情，你必须要记住：**有效的学习往往是不舒服和令人沮丧的**，而且本应该就是这个感觉啊。如果你试图理解一些超出你目前技能水平的东西，那通常会感到尴尬和困惑，你可能会考虑放弃Android或连软件开发也一起放弃。但是这正是你取得**最大进步**的时候，因为你已经走出了舒适区。这有点像最后一次在健身房的锻炼，虽然最痛苦，但也是最有效的。每当你试图理解一些有难度的东西时，你就会理解它更好一点(即使有时你会觉得你正在退步)。为了进步，你的大刀需要拿一些东西咀嚼消化，如果所有的事情都很容易的话，那仅仅意味着你没有足够的挑战自己。这不是心灵鸡汤，这就是事实。我建议你们阅读我关于[成长型思维模式](https://codinginflow.com/growth-mindset-programming)和[有效练习](https://codinginflow.com/deliberate-practice)的文章，在这些文章中我讨论了应对挑战的正确方法。

与使用谷歌同样重要的是学习**阅读源代码**。在Android Studio里，你可以通过同时按住Ctrl/Cmd和点击源码名字来跳转到源码中的类、方法或者变量。或者，你可以把光标放在上面然后按住Ctrl/Cmd + B。读别人的代码一开始会很艰巨的，但是随着实践，它就会越来越简单的。你会惊奇地发现，只要一步一步地遵循执行流程并阅读其他开发人员留下的注释，你就可以获得很多真知灼见和顿悟时刻。您不仅可以通过这种方式找到问题的解决方法，而且还可以开始**了解更深层次的代码是如何工作的**。

现在，当我说你应该先自己尝试解决的时候，我并不是说你不应该问*任何*问题。事实上，我认为你应该问**很多**问题。但是这些问题应该经过深思熟虑并缩小到一个特定的问题。理想情况下，你应该寻求**提示**，而不是寻求完整的解决方法，除非你处于紧急情况下(比如临近截止日期)。

一个好提问的例子：

> 大家好，我在app中增加了一个标题栏，但是现在发现文字和图标都是黑色而不是白色的。我试图使用titleTextColor属性，但是并没有改变浮动菜单的颜色。有没有什么属性可以改变所有标题栏中的所有视图的颜色？我已经在谷歌上搜索了一个小时了，但是还没有找到解决方法。如下是我的XML文件代码(已经格式化过，不是一个拍照模糊的截屏)和一个标题栏的截图，能帮帮我吗？

一个槽糕提问的例子：

> 嗨，我如何能改变标题栏的颜色？现在不起作用。

如上两个提问都会让你或早或晚的得到答案，但是在第一个例子中，你思考的更深入，你尝试自己解决过，你知道哪些方法行不通，你缩小了解决问题的范围，因此你问了一个具体的问题。在第二个例子中，你可能在问题已出现的时候就停止尝试解决，把问题抛给别人来替你解决。也许你随意尝试了几个解决方法，但你并没有表达出你正在思考的是什么。即使两个问题都能让你得到答案，但是明显你在第一个例子中学到的更多。如果你的问题很具体，很可能乐于帮助你这样的问题，因为他们不必解释整个话题并猜测你的问题到底在哪里。这也意味着你不应该在评论框中发布200行错误信息出来，然后希望有人能有帮你解释清楚。

在你得到答案后，你要重复之前的步骤：你先试着自己去解决，然后你去谷歌一下来找你遇到的新问题和不确定的问题，如果你卡住了，你再问另一个具体的问题。

你可以得到最高质量的编程问题答案在[StackOverflow](http://stackoverflow.com/)(最流行的编程主题问答平台)上，但在问题*质量*方面，他们也是最严格的。很多新手(包括我在内)在这里都伤透了脑筋，因为他们提出的问题都是非常基本的，而这些问题以前都已经回答过了。另一方面，在Facebook群里，你机会可以问任何问题，但是答案通常不是很可靠(如果你已经有答案的话)。如果你想要一个不像StackOverflow那么严格但是仍然有高质量的答案的论坛的话，看看reddit.com上的[androiddev的分论坛](https://reddit.com/r/androiddev/)吧。那里的网友都乐于助人，我自己也经常使用它。当然，我也欢迎你加入[Facebook的Coding in Flow讨论组](https://www.facebook.com/groups/1543003582456417/)，我们也在那里互相帮助。

除此之外，我不会在这里推荐任何特定的资源，比如特定的博客或者YouTube频道，因为我认为这些应该从你的研究过程(例如：谷歌搜索)**自然而然**的找到。如果我列出了20个你应该订阅的YouTube频道、网站和新闻的话，它会用大量信息淹没你。本文的目的是让你自己去**渔**。随着时间的推移(例如：你越用谷歌搜索)，你就会发现**哪些资源是有用的以及哪些是在浪费时间**。但是，在下一章中，我会推荐一些方法来了解**最新的**变化。

我非常强调这研究过程的原因是它对你的进步和职业生涯至关重要。对于一个程序员，最重要的技能不仅仅是在你更开始工作的时候，而且还在于你工作的后期，不是知道如何实现像XYZ这样的功能，而是**弄清楚**XYZ是如何实现的，然后**理解明白**解决方法(注：也就是中文中的知其然，也要知其所以然)。你学到如上这些是通过阅读教程，文档和源码，通过提出好的，具体的问题并从所有信息中总结得出自己的结论而来的。框架、库、正确的做事方法甚至是语言都会随着时间而改变，所以当你不知道如何做某件事时或者当你理解上有困难时，不要失去动力。把每一个新出现的概念都看作是**提高你的研究和学习技能的机会**和对软件工程有更多的了解的机会。这也是为什么你不应该过多担心是选择最好的框架或是语言还是记住你写代码的原因。但是也没有人能够手把手从头到尾的在你整个学习过程中帮你。没有一本人或者一门课程能够教你想知道的一切，然后你就可以轻松的坐下来，让代码在你的职业生涯的后面时间里毫不费力的从你的指尖流出(这样肯定也很无聊吧)。

无论你怎么学习，都要**坚持**下去。只要你是一个初学者，**坚持使用一种语言和一个平台**，不要试图尝试大量不同的东西，因为那样的话你就会总是忙于学习基础知识一遍又一遍。此外，不要在每次学习之间停留时间过长，**最好每天学一点**，除非遇到特别的情况。每个月总有几天不想写代码，但我仍然鼓励你最好写一点，或者至少读一些与编程相关的东西，这样可以给你的大脑一些信息去处理。**连续的小步终将会使你更快的进步**(不积跬步无以至千里)。

### 我该如何跟踪最新信息

正如我之前所说的，你关注的特定频道和信息源会随着自我引导，自主学习的方式而自然而然的改变。然而，我想给你一些方法来跟上Android 开发世界的最新事件(这有助于你选择更具体的信息源)。

如下是我正在用的：

Android开发者[官方博客](https://android-developers.googleblog.com/)是获取Android开发公告的很好的信息源。个人而言，我已经将它设置为我的浏览器主页了，因此当有新文章出现时，我能立即看到。

但是只有Android开发者博客是不足以让你实时了解每件事的最新情况的，因为每天都有小新闻和值得注意的变化发生。同样，我发现[androiddtv Subreddit版块](https://reddit.com/r/androiddev/)对于实时了解很有帮助。另外我还关注了Twitter上的[AndridDev bot](https://twitter.com/androiddevRTbot)板块，它会通过垃圾邮件过滤器转发所有带有#androiddev标签的帖子。Twitter有很多垃圾信息，但它也能让你更全面了解编程界的动态，因为每个人都正在那里发帖。记住，你可以设置账户设置**静音**模式其过滤很多无用的东西来减少整体的垃圾信息数量。

### 其他还有什么我能做的吗？

学习编程是很困难的，你需要找到长期保持动力的方法。我已经思考和阅读了很多关于激励和自律的东西，我也总是试图找到更好的方法来战胜拖延症，同时最大程度的享受这个学习过程。这些都是我在博客里写东西。我的YouTube视频里包含了技术内容和编程教程，而在我的博客中，我了很多更宽泛的主题比如激励和心态，以及作为程序员应该培养的次要技能和习惯。

如下是一些你可以马上阅读的文章。挑选一个开始吧：

* [如何战胜拖延症](https://codinginflow.com/how-to-beat-procrastination)
* [作为一名程序员，你应该培养的5种非技术技能和特点](https://codinginflow.com/non-technical-skills-programmer)
* [作为一名程序员如何获得一个工作](https://codinginflow.com/get-job-as-programmer)
* [作为一名程序员，你应该阅读的5本个人发展方面的书籍](https://codinginflow.com/personal-development-books-programmer)
* [作为一名程序员，如何保持健康](https://codinginflow.com/healthy-programmer)

最后祝大家好运连连，码途愉快！










