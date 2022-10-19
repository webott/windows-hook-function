# windows-hook-function
企业微信SDK-windows 钩子函数
Windows下的窗口应用程序是基于消息驱动的，但是在某些情况下需要捕获或者修改消息，从而完成一些特殊的功能。对于捕获消息而言，无法使用IAT或Inline Hook之类的方式去进行捕获，不过Windows提供了专门用于处理消息的钩子函数。7. 4. 1 钩子原理Windows下的应用程序大部分是基于消息模式机制的，一些CUI的程序不是基于消息的。Windows下的应用程序都有一个消息过程函数，根据不同的消息来完成不同的功能。Windows操作系统提供的钩子机制的作用是用来截获、监视系统中的消息。Windows 操作系统提供了很多不同种类的钩子，可以处理不同的消息。
Windows系统提供的钩子按照挂钩范围分为局部钩子和全局钩子。局部钩子是针对一个线程的，而全局钩子则是针对整个操作系统内基于消息机制的应用程序的。全局钩子需要使用DLL文件，DLL文件里存放了钩子函数的代码。在操作系统中安装全局钩子以后，只要进程接收到可以发出钩子的消息后，DLL 文件会被操作系统自动或强行地加载到该进程中。由此可见，设置消钩子也是一种可以进行DLL注入的方法。
上面已经简单地讲述了Windows钩子的基本原理，现在来介绍Windows下的钩子函数。主要有3个，分别是SetWindowsHookExO、CallNextHookExO)和 Unhook WindowsHookExo).下面介绍这些函数的使用方法。SetWindowsHookEx(函数的定义如下：HHOOK SetWindowsHookEx(int idHook,HOOKPROC 1pfn,HINSTANCE hMod,DWORD dwThreadId）；该函数的返回值为一个钩子句柄。这个函数有4个参数，下面分别进行介绍。lpfn:该参数指定为Hook函数的地址。如果dwThreadld参数被赋值为0,或者被设置为一个其他进程中的线程ID,那么lpfn则属于DLL中的函数过程。如果dwThreadld为当前进程中的线程ID,那么lpfn可以是指向当前进程中的函数过程，也可以是属于DLL中的函数过程。hMod:该参数指定钩子函数所在模块的模块句柄。该模块句柄就是lpfn所在的模块的句柄。如果dwThreadld为当前进程中的线程ID,而且lpfn所指向的函数在当前进程中，那么hMod将被设置为NULL.dw Threadld:该参数设置为需要被挂钩的线程的ID号。如果设置为0,表示在所有的线程中挂钩（这里的“所有的线程”表示基于消息机制的所有的线程）。如果指定为具体的线程的ID号，表示要在指定的线程中进行挂钩。该参数影响上面两个参数的取值。该参数的取值决定了该钩子属于全局钩子，还是局部钩子。idHook:该参数表示钩子的类型。由于钩子的类型非常多，因此放在所有的参数后面进行介绍。下面介绍几个常用到的钩子，也可能是读者比较关心的几个类型。1. WH_GETMESSAGE安装该钩子的作用是监视被投递到消息队列中的消息。也就是当调用 GetMessage()或钩子函数。
7. 4 Windows 钩子函数函数从程序的消息队列中获取一个消息后调用该钩子。WH_GETMESSAGE钩子函数的定义如下：LRESULT CALLBACKGetMsgProc(int code,／／ 钩子编码WPARAM wParam,／／ 移除选项LPARAM 1Param／／ 消息）：2. WH_MOUSE安装该钩子的作用是监视鼠标消息。该钩子函数的定义如下：LRESULTCALLBACK MouseProc(int nCode,／／ 钩子编码WPARAM wParam,／／ 消息标识符LPARAM 1Param／／鼠标的坐标）：3. WH_KEYBOARD安装该钩子的作用是监视键盘消息。该钩子函数的定义如下：LRESULT CALLBACK Keyboardproc(int code,／／ 钩子编码WPARAM wParam,／／ 虚拟键编码LPARAM 1Param1/ 按键信息1:4. WH_DEBUG安装该钩子的作用是调试其他钩子的钩子函数。该钩子函数的定义如下：LRESULT CALLBACK DebugProcint nCode,／／钩子编码WPARAM wParam,／／ 钩子类型LPARAM 1Param／／ 调试信息）：其他的钩子类型请读者参考MSDN.从上面的这些钩子函数定义可以看出，每个钩子函数的定义都是一样的。每种类型的钩子监视、截获的消息不同。虽然它们的定义都是相同的，但是函数参数的意义是不同的。由于篇幅所限，每个函数的具体意义请参考MSDN,其中有最为详细的介绍。接着介绍跟钩子有关的另外一个函数：UnhookWindowsHookEx0,其定义如下：BOOL UnhookWindowsHookEx(HHOOK hhk／／ 钩子函数的句柄这个函数是用来移除先前用SetWindowsHookEx(安装的钩子。该函数只有一个参数，是钩子句柄，也就是调用该函数通过指定的钩子句柄来移除与其相应的钩子。在操作系统中，可以多次反复地使用SerWindowsHookEx()函数来安装钩子，而且可以安装多个同样类型的钩子。这样，钩子就会形成一条钩子链，最后安装的钩子会首先截获到消息。当该钩子对消息处理完毕以后，会选择返回，或者选择把消息继续传递下去。在通常情况下，如果为了屏蔽消息，则直接在钩子函数中返回一个非零值。比如要在自己程序中屏蔽鼠标消息，则在安装的鼠标钩子函数中直接返回非零值即可。如果为了消息在经过钩子函数后可以继续传达到目标窗口，必须选择将消息继续传递。使消息能继续传递的函数的定义如下：LRESULTCallNextHookEx(HHOOK hhk,／／ 当前钩子的句柄1/ 传递给钓子函数的钓子编码Int nCode,／／ 传递给钓子函数的值WPARAM wParam,／／ 传递给钓子函数的值LPARAM 1Param）：PeekMessage0函数时。

个人微信
目前已经实现了很多有趣的功能，运行稳定，比如：发各种消息，
接收各种消息，群管，下载文件，加好友，检测僵尸粉，朋友圈等等功能，
可提供接口，方便各种语言二次开发，欢迎技术交流，Q：2645542961
，请勿用于商业用途。
![image](https://user-images.githubusercontent.com/73727649/196665532-8ff61217-f6f1-4259-ba1c-1731995fbf60.png)

企业微信：
目前已经实现了大部分功能，运行稳定，比如：发各种消息，
接收各种消息，外部群内部群管理，下载文件，加好友等等功能，
可提供接口，方便各种语言二次开发，欢迎技术交流。

