# 异常控制流 #

1. 现代系统通过使控制流发生突变来对这些情况做出反应。一般而言，我们把这些突变称为**异常控制流**。异常控制流发生在计算机系统的各个层次。

## 异常 ##

1. 在处理器中，状态被编码为不同的位和信号。状态变化称为事件。事件可能和当前指令的执行直接相关。也可能和当前指令的执行没有关系。

2. 在任何情况下，当处理器检测到有事件发生时，它就会通过一张叫做**异常表**的跳转表，进行一个间接过程调用（异常），到一个专门设计用来处理这类事件的操作系统子程序（异常处理程序）。

3. 系统中可能的每种类型的异常都分配了一个唯一的非负整数的异常号。其中一些号码是由处理器的设计者分配的，其他号码是由操作系统内核的设计者分配的。在系统启动时，由操作系统分配和初始化的异常表表目k（异常号）包含异常k的处理程序的地址。

4. 异常区别于过程调用的地方：
    - 根据异常的类型，返回地址要么时当前指令，要么是下一条指令。
    - 处理器也把一些额外的处理器状态压到栈里。
    - 如果控制从用户程序转移到内核，所有这些项目都被压到内核栈中。
    - 异常处理程序运行在内核模式下，对所有的系统资源都有完全的访问权限。

5. 异常的分类：
    |类别|原因|异步/同步|返回行为|
    |---|---|---|---|
    |中断|来自I/O设备的信号|异步|总是返回到下一条指令|
    |陷阱|有意的异常|同步|总是返回到下一条指令|
    |故障|潜在可恢复的错误|同步|可能返回到当前指令|
    |终止|不可恢复的错误|同步|不会返回|

6. 中断（异步发生），硬件中断的异常处理程序常常称为**中断处理程序**。通过向处理器芯片上的一个引脚发信号，并将异常号放到系统总线上，来触发中断，这个异常号标识了引起中断的设备。

7. 陷阱和系统调用（同步发生），陷阱是有意的异常，最重要的用途是在用户程序和内核之间提供一个像过程一样的接口，叫做系统调用。普通的函数运行在用户模式中，限制了函数可以执行的指令的类型，而且只能访问与调用函数相同的栈。系统调用运行在内核模式中，允许系统调用执行特权指令，并访问定义在内核中的栈。

8. 故障（同步发生），根据故障是否能够被修复，故障处理程序要么重新执行引起故障的指令，要么终止。

9. 终止（同步发生），不可恢复的致命错误造成的结果，通常是一些硬件错误，终止处理程序从不将控制返回给应用程序。

## 进程 ##

1. 进程的经典定义就是一个执行中程序的实例。系统中的每个程序都运行在某个进程的上下文中。进程提供给应用程序的关键抽象：独立的逻辑控制流和私有的地址空间（都是假象）。

2. 一系列程序计数器（PC）的值，唯一地对应于包含在程序的可执行目标文件中的指令，或是包含在运行时动态链接到程序的共享对象中的指令。这个PC值的序列叫做**逻辑控制流**。

3. 一个逻辑流的执行在时间上与另一个流重叠，称为**并发流**（X在Y开始之后和Y结束之前开始）。多个流并发地执行的一般现象被称为**并发**。一个进程和其他进程轮流运行的概念称为**多任务**。一个进程执行它的控制流的一部分的每一时间段叫做**时间片**。如果两个流并发地运行在不同的处理器核或者计算机上，那么我们称它们为**并行流**。

4. 处理器通常是用某个控制寄存器中的一个模式位来提供这种功能的，该寄存器描述了进程当前享有的特权。当设置了模式位时，进程就运行在内核模式中。

5. 内核为每个进程维持一个上下文。上下文就是内核重新启动一个被抢占的进程所需的状态。它由一些对象的值组成，这些对象包括通用目的寄存器、浮点寄存器、程序计数器、用户栈、状态寄存器、内核栈和各种内核数据结构。

6. 调度，是内核中称为调度器的代码处理的。档内核调度了一个新的进程运行后，它就抢占当前进程，并使用一种称为上下文切换的机制来将控制转移到新的进程。上下文切换：
    - 保存当前进程的上下文。
    - 恢复某个先前被抢占的进程被保存的上下文。
    - 将控制传递给这个新恢复的进程。

## 系统调用错误处理 ##

## 进程控制 ##

1. 每个进程都有一个唯一的正数进程ID（PID）。getpid函数返回调用进程的PID。getppid函数返回它的父进程的PID（创建调用进程的进程）

2. 进程总是处于下面三种状态之一：
    - 运行。要么在CPU上执行，要么在等待被执行且最终会被内核调度。
    - 停止。进程执行被挂起，且不会被调度。当收到SIGSTOP、SIGTSTP、SIGTTIN或者SIGTTOU信号时，进程就停止，并且保持停止直到它收到一个SIGCONT信号。
    - 终止。进程永远地停止了，有三种原因：收到一个默认行为是终止进程的信号；从主程序返回；调用exit函数。

3. 父进程和新创建的子进程之间最大的区别在于它们有不同的PID。

4. fork的返回值提供一个明确的方法来分辨程序是在父进程还是在子进程中执行，在父进程中fork返回子进程的PID，在子进程中fork返回0。

5. 一个进程由于某种原因终止时，内核并不是立即把它从系统中清除。相反，进程被保持在一种已终止的状态中，直到被它的父进程回收。一个终止了但还未被回收的进程称为**僵死进程**。如果父进程没有回收它的僵死子进程就终止了，内核会安排init进程去回收它们。init进程的PID为1，是在系统启动时由内核创建的，它不会终止，是所有进程的祖先。

6. waitpid（简单版本wait）挂起调用进程的执行，直到它的等待集合中的一个子进程终止：
    - 参数pid>0，那么等待集合就是一个单独的子进程，它的进程ID等于pid。
    - 参数pid=-1，那么等待集合就是由父进程所有的子进程组成的。

7. 可以通过将options设置为常量WNOHANG、WUNTRACED和WCONTINUED的各种组合来修改默认行为：
    - WNOHANG，如果等待集合中的任何子进程都还没有终止，那么就立即返回（返回值0）
    - WUNTRACED，挂起调用进程的执行，直到等待集合中的一个进程变成已终止或者被停止。返回的PID为导致返回的已终止或被停止子进程的PID。
    - WCONTINUED，挂起调用进程的执行，直到等待集合中的一个正在运行的进程终止或等待集合中的一个被停止的进程收到SIGCONT信号重新开始执行。

8. 如果statusp参数非空，那么waitpid就会在status中放上关于导致返回的子进程的状态信息：
    - WIFEXITED，如果子进程通过调用exit或者一个return正常终止，就返回真。
    - WEXITSTATUS，返回一个正常终止的子进程的退出状态。只有在WIFEXITED为真时才定义这个状态。
    - WIFSIGNALED，如果子进程是因为一个未被捕获的信号终止的，那么就返回真。
    - WTERMSIG，返回导致子进程终止的信号的编号。只有在WIFSIGNALED为真时才定义这个状态。
    - WIFSTOPPED，如果引起返回的子进程当前时停止的，那么就返回真。
    - WSTOPSIG，返回引起子进程停止的信号的编号。只有在WIFSTOPPED为真时才定义这个状态。
    - WIFCONTINUED，如果子进程收到SIGCONT信号重新启动，则返回真。

9. 如果调用进程没有子进程，那么waitpid返回-1，并且设置errno为ECHILD。如果waitpid函数被一个信号中断，那么它返回-1，并设置errno为EINTR。

10. sleep函数将一个进程挂起一段指定的时间，如果请求的时间量已经到了，sleep返回0，否则返回还剩下的要休眠的秒数。pause函数让调用函数休眠，直到该进程收到一个信号。

11. execve函数加载并运行可执行目标文件，只有当出现错误时，execve才会返回到调用程序。所以，与fork一次调用返回两次不同，execve调用一次并从不返回。

12. getenv函数在环境数组中搜索字符串"name=value"。unsetenv会删除搜索到的字符串。setenv会用newvalue代替oldvalue，但是只有在overwrite非零时才会这样。如果name不存在，那么setenv就把"name=newvalue"添加到数组中。

## 信号 ##

1. 一个Linux信号就是一条小消息，它允许进程和内核中断其他进程，它通知进程系统中发生了一个某种类型的事件。每种信号类型都对应于某种系统事件。低层的硬件异常是由内核异常处理程序处理的，正常情况下，对用户进程而言是不可见的。信号提供了一种机制，通知用户进程发生了这些异常。

2. 传送一个信号到目的进程是由两个不同步骤组成的：
    - 发送信号，内核通过更新目的进程上下文中的某个状态，发送一个信号给目的进程。（原因：内核检测到一个系统事件；一个进程调用了kill函数，显式地要求内核发送一个信号给目的进程（也可以发给自己））
    - 接收信号，当目的进程被内核强迫以某种方式对信号的发送做出反应时，它就接收了信号。进程可以忽略这个信号，终止或者通过执行一个称为信号处理程序的用户层函数捕获这个信号。

3. 一个发出而没有被接收的信号叫做**待处理信号**。任何时刻，一种类型至多只会有一个待处理信号（pending位向量）。一个进程可以有选择性地阻塞接收某种信号（blocked位向量）。当一种信号被阻塞时，它仍可以被发送，但是产生的待处理信号不会被接收，直到进程取消对这种信号的阻塞。

4. 每个进程都只属于一个进程组，是由一个正整数进程组ID来标识的。getpgrp返回当前进程的进程组ID。setpgid改变自己或其他进程的进程组。kill发送信号给其他进程（包括自己）。alarm安排内核在secs秒后向它自己发送SIGALRM信号。

5. signal函数可以通过下列三种方法之一来改变和信号signum相关联的行为：
    - 如果handler是SIG_IGN，忽略类型为signum的信号。
    - 如果handler是SIG_DFL，那么类型为signum的信号恢复为默认行为。
    - 否则handler就是用户定义的函数的地址，这个函数被称为信号处理程序，只要进程接收到一个类型为signum的信号，就会调用这个程序。通过把处理程序的地址传递到signal函数从而改变默认行为，这叫做设置信号处理程序。调用信号处理程序被称为捕获信号。执行信号处理程序被称为处理信号。

6. sigprocmask函数改变当前阻塞的信号集合，具体行为依赖于how值：
    - SIG_BLOCK，把set中的信号添加到blocked中。
    - SIG_UNBLOCK，从blocked中删除set中的信号。
    - SIG_SETMASK，block=set。
    如果oldset非空，那么blocked位向量之前的值保存在oldset中。

7. 使用下述函数对set信号集合进行操作：sigemptyset初始化set为空的集合。sigfillset函数把每个信号都添加到set中。sigaddset函数把signum添加到set，sigdelset从set中删除signum，如果signum是set的成员，那么sigismember返回1，否则返回0。

8. 编写信号处理程序的原则：
    - 处理程序要尽可能的简单。
    - 在处理程序中只调用异步信号安全的函数。
    - 保存和恢复errno，在进入处理程序时把errno保存在一个局部变量中，在处理程序返回前恢复它。
    - 阻塞所有信号，保护对共享全局数据结构的访问。
    - 用valatile声明全局变量。
    - 用sig_atomic_t声明标志。sig_atomic_t提供一种整型数据类型，对它的读和写保证会是原子的。

9. 未处理的信号是不排队的，如果存在一个未处理的信号就表明至少有一个信号到达了。因此，不可以用信号来对其他进程中发生的事件计数。

## 非本地跳转 ##

1. 非本地跳转，将控制直接从一个函数转移到另一个当前正在执行的函数，而不需要经过正常的调用-返回序列。非本地跳转是通过setjmp和longjmp函数来提供的。

2. setjmp调用一次返回多次，longjmp调用一次从不返回。

3. sigsetjmp和siglongjmp函数是setjmp和longjmp的可以被信号处理程序使用的版本。

## 操作进程的工具 ##

## 家庭作业未完成 ##

- 22、24、25、26
