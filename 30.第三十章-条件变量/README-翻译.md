通过本作业，您可以探索一些使用锁和条件变量的实际代码，以实现本章中讨论的各种形式的生产者/消费者队列。 
您需要查看这些代码，以各种配置运行它，并使用它们来了解哪些方案有效，哪些无效，以及一些其他的问题。 阅读 README 文件了解详细信息。

不同版本的代码对应于不同的“解决”生产者/消费者问题的方式。大多数解决方式是不正确的；只有一个解决方案是正确的。 
阅读本章以了解有关生产者/消费者问题以及代相关代码的更多信息。

第一步是下载代码，输入 make 来构建所有方案的实现。您应该会看到四个文件：

- main-one-cv-while.c:  通过单个条件变量解决生产者/消费者问题。
- main-two-cvs-if.c:    使用两个条件变量，并使用 if 检查是否需要睡眠。
- main-two-cvs-while.c: 使用两个条件变量，并使用 while 检查是否需要睡眠（这是正确的解决方案）
- main-two-cvs-while-extra-unlock.c: 首先释放锁并在 fill 条件变量周围获取锁

查看 pc-header.h 很有用，它包含所有这些不同主程序的公共代码，以便使用 Makefile 正确地构建代码。

每个程序可以使用以下标志:
<pre>
  -l 每个生产者生产的数量
  -m 生产者/消费者共享的缓冲区大小
  -p 生产者数量
  -c 消费者数量
  -P <sleep string: how producer should sleep at various points>
  -C <sleep string: how consumer should sleep at various points>
  -v [verbose flag: 追踪发生了什么并打印]
  -t [timing flag: 打印执行总时间]
</pre>

前四个参数意思:

-l 指定每个生产者应该进行多少次循环(即每个生产者生产的数据量)

-m 控制共享缓冲区的大小(大于或等于 1),

-p -c 分别设置有多少生产者和消费者。

更有趣的是两个睡眠字符（sleep string），一个用于生产者，另一个用于消费者。 
这些标志使您可以使线程在执行过程中的某些点处于休眠状态，从而切换到其他线程。
这样一来，您就可以很方便的使用每种解决方案，甚至可以研究特定问题或研究生产者/消费者问题的其他方面。

字符串（string）参数的指定方式如下。例如，如果有三个生产者，睡眠字符串应该分别为每个生产者指定睡眠时间，
并使用冒号作为分隔符。这三个生产者的睡眠字符参数看起来像这样：

```shell script
sleep_string_for_p0:sleep_string_for_p1:sleep_string_for_p2 
```

每个睡眠字符串依次是一个逗号分隔的列表，用于决定代码中每个睡眠点的睡眠时间
为了更好地理解每个生产者或每个消费者的睡眠字符串，让我们看一下 main-two-cvs-while.c 中的代码。
特别是生产者代码。在此代码片段中，一个生产者线程循环一段时间，通过调用 do_fill() 将元素放入共享缓冲区中。
在 do_fill() 函数周围有一些等待和信号代码，以确保当生产者试图将元素添加到缓冲区时缓冲区没有被填满。更多细节见本章。

```c

void *producer(void *arg) {
    int id = (int) arg;
    int i;
    for (i = 0; i < loops; i++) {   p0;
        Mutex_lock(&m);             p1;
        while (num_full == max) {   p2;
            Cond_wait(&empty, &m);  p3;
        }
        do_fill(i);                 p4;
        Cond_signal(&fill);         p5;
        Mutex_unlock(&m);           p6;
    }
    return NULL;
}
```

从代码中可以看到，有许多标记为 p0、p1 的点。这些点是代码可以睡眠的地方。
消费者代码(这里没有显示)具有类似的睡眠点(c0 等)。

*译者注: p0,p1... 是宏，定义在 main-header.h 中*

为生产者指定睡眠字符串很简单：只需定义在 p0、p1、…… 、p6 睡眠点上生产者应该睡眠多长时间即可。
例如，字符串 1,0,0,0,0,0 指定生产者应该在睡眠点 p0 处休眠 1 秒(在获取锁之前)，然后在后面的休眠点都不休眠。

现在让我们展示运行一个程序的输出结果。首先，我们假设只有一个生产者和一个消费者。
让我们不添加任何睡眠行为(这是默认行为)。在本例中，缓冲区大小被设置为 2 (-m 2)。

首先，让我们构建代码

<pre>
prompt> make
gcc -o main-two-cvs-while main-two-cvs-while.c -Wall -pthread
gcc -o main-two-cvs-if main-two-cvs-if.c -Wall -pthread
gcc -o main-one-cv-while main-one-cv-while.c -Wall -pthread
gcc -o main-two-cvs-while-extra-unlock main-two-cvs-while-extra-unlock.c 
  -Wall -pthread
</pre>

现在我们可以运行:

```shell script
prompt> ./main-two-cvs-while -l 3 -m 2 -p 1 -c 1 -v
```

在本例中，如果您跟踪代码(使用 verbose 标志，-v)，您将在屏幕上得到以下输出(或类似的输出)

<pre>
 NF             P0 C0
  0 [*---  --- ] p0
  0 [*---  --- ]    c0
  0 [*---  --- ] p1
  1 [u  0 f--- ] p4
  1 [u  0 f--- ] p5
  1 [u  0 f--- ] p6
  1 [u  0 f--- ] p0
  1 [u  0 f--- ]    c1
  0 [ --- *--- ]    c4
  0 [ --- *--- ]    c5
  0 [ --- *--- ]    c6
  0 [ --- *--- ]    c0
  0 [ --- *--- ] p1
  1 [f--- u  1 ] p4
  1 [f--- u  1 ] p5
  1 [f--- u  1 ] p6
  1 [f--- u  1 ] p0
  1 [f--- u  1 ]    c1
  0 [*---  --- ]    c4
  0 [*---  --- ]    c5
  0 [*---  --- ]    c6
  0 [*---  --- ]    c0
  0 [*---  --- ] p1
  1 [u  2 f--- ] p4
  1 [u  2 f--- ] p5
  1 [u  2 f--- ] p6
  1 [u  2 f--- ]    c1
  0 [ --- *--- ]    c4
  0 [ --- *--- ]    c5
  0 [ --- *--- ]    c6
  1 [f--- uEOS ] [main: added end-of-stream marker]
  1 [f--- uEOS ]    c0
  1 [f--- uEOS ]    c1
  0 [*---  --- ]    c4
  0 [*---  --- ]    c5
  0 [*---  --- ]    c6

Consumer consumption:
  C0 -> 3
</pre>

在描述这个简单示例中发生的事情之前，让我们先理解输出中对共享缓冲区的描述，如上面结果的左侧所示。
首先它是空的(一个空槽表示为 --- ，两个空槽显示为 [*--- --- ]);
输出还显示缓冲区中值的元素个数（num_full），数值从 0 开始。在 P0 放入一个值之后，
它的状态更改为 [u 0 f--- ]（num_full 更改为 1）。您可能还会注意到这里的一些额外标记。
u 标记显示 use_ptr 在指向哪里（这是下一个消费者获得值的地方）
类似地，f 标记显示 fill_ptr 的位置（这是下一个生产者将生产值的位置）。当您看到 * 标记时，它只是表示 use_ptr 和 fill_ptr 指向同一个槽。

在输出结果的右侧，您可以看到每个生产者和消费者将要执行的步骤的追踪。 
例如，生产者获取锁（p1），然后，由于缓冲区有一个空插槽，因此生成一个值放入其中（p4）。然后继续进行直到释放锁（p6），然后尝试重新获取它。 
在这个示例中，消费者获取锁并使用值（c1，c4）。进一步研究跟踪以了解生产者/消费者解决方案如何与单个生产者和消费者一起工作。

现在，让我们添加一些暂停来更改跟踪的行为。在这种情况下，假设我们要使生产者进入休眠状态，以便消费者可以先运行。 我们可以按以下方式完成此操作：

```shell script
prompt> ./main-two-cvs-while -l 1 -m 2 -p 1 -c 1 -P 1,0,0,0,0,0,0 -C 0 -v
```

结果：
<pre>
 NF             P0 C0
  0 [*---  --- ] p0
  0 [*---  --- ]    c0
  0 [*---  --- ]    c1
  0 [*---  --- ]    c2
  0 [*---  --- ] p1
  1 [u  0 f--- ] p4
  1 [u  0 f--- ] p5
  1 [u  0 f--- ] p6
  1 [u  0 f--- ] p0
  1 [u  0 f--- ]    c3
  0 [ --- *--- ]    c4
  0 [ --- *--- ]    c5
  0 [ --- *--- ]    c6
  0 [ --- *--- ]    c0
  0 [ --- *--- ]    c1
  0 [ --- *--- ]    c2
 ...
</pre>

如您所见，生产者在代码中执行到了 p0 标记处，然后从其睡眠字符串中获取了第一个值，值为 1，因此每生产者都睡了 1 秒，
在睡眠之前线程尝试获取锁。因此消费者开始运行，获取锁，但发现队列为空，从而进入睡眠状态（释放锁）。
生产者随后运行（最终），并且所有过程均符合您的预期。

请注意：必须为每个生产者和消费者提供睡眠字符串。因此，如果您创建两个生产者和三个消费者（使用-p 2 -c 3，则必须为每个指定休眠字符串
（例如，-P 0:1 或 -C 0,1,2:0:3,3,3,1,1,1）。睡眠字符串可以短于代码中的睡眠点的个数；其余被省略的睡眠点初始化为零。

