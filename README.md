# Xcode How To Detect Memory Leaks In a macOS Console Application
如何用Xcode检测macOS上的命令行应用的内存泄漏

<br />

对于大部分开发者而言，Mac是个好东西，而对于许多C语言、汇编语言开发者而言，Xcode则是个非常棒的IDE！用Xcode编写、调试C语言以及汇编语言代码不仅爽、支持完整的GNU11语法，而且效率也高。不过Xcode+Apple LLVM目前唯一不足的就是快速查询内存泄漏。

对于macOS、iOS App开发者而言，平时我们都会通过Profile（Instruments）中的Leaks工具来探测内存泄漏情况，此工具确实也非常好用，网上也有不少相关文章介绍使用方法。不过，如果我们有时要测试一些算法，要检测其是否有内存泄漏的话，单独创建一个桌面应用进行检测的话就显得太过复杂了，我们是否有办法通过创建一个命令行应用进行内存泄漏检测呢？方法是有的。不过macOS上的这个技术在国内网站上介绍很少，所以我专门收集了一些资料，然后经过一些实验整理成本篇博文供大家参考。

顺便说一句，在Windows系统上，通过使用MSVC自带的crtdbg库可以很轻松地查看内存泄漏情况；而在Linux系统下，则可以通过GCC或Clang自带的LeakSanitizer工具既能很轻松地查看程序内存泄漏的情况（只需要添加`-fsanitize=address`编译选项）。而在macOS中，事情则没那么简单。Apple提供了两套工具可以让我们检测指定应用的内存泄漏情况，一个是**leaks**，还有一个是**malloc_history**，这两者需要结合起来用才能查看到内存泄漏的具体信息。下面我们将具体介绍。

<br />

首先，我们用Xcode创建一个基于C语言的控制台应用，我这里将项目名起为“MyCTest”，项目工程的编译选项用默认的即可，或者也可以根据自己的喜好进行设定，这个没啥影响。

然后，我们点击左上角的项目名Icon以设置活跃方案（Set the active scheme），如下图所示：

<br />

![1.jpg](https://github.com/zenny-chen/Xcode-detect-memory-leaks-in-a-console-application/blob/master/1.jpg)

<br />

进入到方案界面之后，我们点击最右边的选项卡“Diagnostics”，如果我们要监测内存泄漏，则必须勾选上“**Guard Malloc**”，以及“**Malloc Stack**”，其他可根据自己的需求进行勾选，如下图所示：

<br />

![2.jpg](https://github.com/zenny-chen/Xcode-detect-memory-leaks-in-a-console-application/blob/master/2.jpg)

<br />

完成后点击“Close”按钮退出，我们再进入代码编辑界面，输入以下所示的测试代码。然后在最后`return 0;`语句处设置断点，我们查看打印日志信息，可以看到GuardMalloc工具所打印出来的一些信息。其中，项目名后面的数字即表示当前所运行程序的进程ID。我们在下一步使用leaks工具时需要这个进程ID号，因此这里可以先复制一下。

<br />

![3.png](https://github.com/zenny-chen/Xcode-detect-memory-leaks-in-a-console-application/blob/master/3.png)

<br />

