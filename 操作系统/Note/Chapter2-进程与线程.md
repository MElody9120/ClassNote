# Chapter2 进程与线程

---

考点介绍
- 本章既是重点又是难点
- 对于单选题，主要集中在进程和线程的比较、进程的三个基本状态的转换、判断临界区算法的正确与否以及整型信号量和记录型信号量的定义
- 对于综合应用题，主要集中在信号量的应用、进程同步问题解决方法，如P、V操作

本章重点内容
- 信号量的含义
- 临界资源的界定
- 利用信号量解决进程同步问题的方法
  
---

## 知识结构

本章的知识结构图如下mermaid图所示

```mermaid
graph RL
a(知识点) --> b1(定义)
b1 --> c1(系统资源和调度单位)
b1 --> c2(程序在数据集上的一次执行)
a --> b2(特征)
b2 --> c3(并发性)
c3 --> d1(多个程序同时间段执行)
b2 --> c4(动态性)
c4 --> d2(有生命周期)
b2 --> c5(独立性)
c5 --> d3(独立操作单位)
b2 --> c6(异步性)
c6 --> d4(速度不确定结果确定)
a --> b3(状态)
b3 --> c7(就绪)
c7 --> d5(仅缺处理器资源)
b3 --> c8(运行)
c8 --> d6(拥有所有所需资源)
b3 --> c9(阻塞)
c9 --> d7(所需资源得不到满足)
a --> b4(描述)
b4 --> c10(进程控制块PCB)
a --> b5(同步)
b5 --> c11(临界资源)
c11 --> d8(只能互斥使用的资源)
b5 --> c12(信号量)
c12 --> d9(临界资源数量)
b5 --> c13(临界区)
c13 --> d10(访问临界资源的代码区)
a --> b6(同步问题)
b6 --> c14(生产者-消费者问题)
c14 --> d11(合作关系的抽象)
b6 --> c15(哲学家就餐问题)
c15 --> d12(竞争关系的抽象)
b6 --> c16(读者-写者问题)
c16 --> d13(共享关系的抽象)
a --> b7(线程)
b7 --> d14(降低并发时空开销)

```

---

## 进程 Process

进程是操作系统最核心的观念

现代操作系统的一切都是以进程而展开的

进程是**正在运行的程序的一个抽象（正解）**

- 进程执行模型
  - 顺序执行（Sequential Mode）
  - 并发执行（Parallel Mode）

- 伪并行
  - 并行与并发

### 进程的执行模型

对应到操作系统中，进程的执行模型同样有**顺序执行**和**并行执行**

顺序执行：一个进程执行完毕，再紧跟着执行另一个进程.

如下图2-1模型所示

![One Program Counter](../Img/Chapter2-1-One%20program%20counter.png)

缺点：执行效率太低

并行执行：四个进程同时执行，相互独立，互不干扰

如下图2-2模型所示

![Four Program Counter](../Img/Chapter2-2-Four%20program%20Counters.png)

缺点：对硬件要求高，并且对算法实现有一定的要求

#### 进程的并发执行模型

**并发（concurrence）执行是微观上的顺序执行，宏观上的并行执行**(并不是真并行，依赖于调度算法)

并发模型示意图2-3如下图所示

![进程并发模型示意图](../Img/Chapter2-3-进程并发执行模型示意图.png)

不同的执行顺序会产生不一样的结果

### 进程的状态

下图2-4为一般的宏观进程状态表示（两状态）

![两状态进程模式图](../Img/Chapter2-4-两状态进程模式图.png)

实际上进程可划分为三种基本状态  
- **就绪状态（Ready）**:当进程分配到除CPU以外的所有必要资源后，只要再获得CPU，便可以立即执行
- **执行状态（Running）**：进程已经获得足够资源，并获得CPU，程序此时正在执行、
- **阻塞状态(Blocked)**:正在执行的进程由于发生某时间而暂时无法继续执行时，便放弃处理机而处于暂停状态，这时候把这种暂停状态称为阻塞状态，有时候也称为等待状态。

#### 三态进程模式图,图2-5如下图所示

![三状态进程模式图](../Img/Chapter2-5-三态转换.png)

mermaid代码如下所示(需要打上一个mermaid插件),后续多态图以mermaid形式展示  

```mermaid
graph RL
a((就绪)) --进程调度--> b((运行))
b --时间片用完--> a
b --请求资源--> c((阻塞))
c --请求得到满足--> a((就绪态))
```

**单阻塞队列**

如下图2-6所示  

![单阻塞队列](../Img/Chapter2-6-单阻塞队列.png)

**多阻塞队列**

如下图2-7所示  

![多阻塞队列](../Img/Chapter2-7-多阻塞队列.png)

多阻塞队列可以分类请求，分别处理，只需要检测CPU请求的队列  

---
#### 五态进程模式图，在三态的情况下引进了新的两态

五状态图如下mermaid图所示

```mermaid
graph RL
a((就绪)) --进程调度--> b((运行))
b --时间片用完--> a
b --请求资源--> c((阻塞))
c --请求得到满足--> a((就绪))
d((新建态)) --创建进程--> a((就绪))
b((运行)) --终止进程--> e((终止))
```
新增
- **新建态**（New）：进程已经创建，但是未被OS接纳为可执行程序，并且程序还在辅存中，PCB在内存
- **退出态**（Exit）：因停止或取消，被OS从执行态释放

---
#### 七状态进程模式图，在五态的情况下引进了新的两态

七状态图如下mermaid图所示（觉得太乱可以下跳，有PNG截图版）

```mermaid
graph RL
a((就绪)) --进程调度--> b((运行))
b --时间片用完--> a
b --请求资源--> c((阻塞))
c --请求得到满足--> a((就绪))
d((新建态)) --创建进程--> a((就绪))
b((运行)) --终止进程--> e((终止))
d --提交--> f((挂起就绪))
f --解除挂起--> a
a --挂起--> f
b --挂起--> f
g((挂起等待)) --等待事件结束--> f
g --解除挂起--> c
c --挂起--> g
```

挂起状态：使执行的进程暂停执行，静止下来，我们把这种静止状态称为挂起状态

以下表格将说明什么时候会出现挂起状态  表2-1  

| 事件 | 说明 |
| :----: | :---- |
| 交换 | 操作系统需要释放足够的内存空间，以调入并执行处于就绪状态的进程 |
| 交互式用户请求 | 用户可能希望挂起一个程序的执行，目的是为了调试或者与一个资源的使用和链接 |
| 定时 | 一个进程可能会周期性地执行（例如记账或者系统监视进程），而且可能等待下个时间间隔时会挂起 |
| 父进程请求 | 父进程可能会希望挂起后代进程的执行，以检验或修改挂起的进程，活着的协调不同后代进程之间的行为 |
| 其他OS原因 | 操作系统可能挂起后台进程或工具程序进程，或者被怀疑导致问题的进程|

因此引入了挂起状态，如下图 2-8 所示(由于七态图mermaid太乱也可观察下图)

![含有挂起态的进程状态图](../Img/Chapter2-8-含有挂起态的进程状态转换图.png)

进程之间的各状态转换说明

- 空白->新建：系统调用、用户登录、用户请求等.......
- 新建->就绪：系统尚有空余资源，接纳进程并放入就绪队列
- 就绪->运行：获得了除了CPU之外的所有资源
- 运行->完毕：进程正常执行完毕或者被 kill
- 运行->阻塞：等待请求完成
- 阻塞->就绪：请求已经完成
- 就绪(就绪挂起)->退出：无偿的被KILL
- 阻塞->阻塞挂起：没有阻塞进程，挂起就绪进程以释放空间
- 就绪挂起->就绪：没有就绪进程或者就绪进程优先级较低
- 阻塞挂起->阻塞：解除挂起
- 阻塞挂起->就绪挂起：等待事件结束


### 进程的控制

- 进程状态（若干状态之间的转换以及转换依据）

- 进程映像
  - 进程的程序、数据、堆、栈的集合

- 进程控制块 PCB（Process Control Block）
  - 用于控制进程属性的集合

- 进程图

PCB表的一般结构如下图 2-9 所示

![PCB的一般结构](../Img/Chapter2-9-操作系统控制表的一般结构.png)

Primary Process Table 中就是进程映像，Process Image中的进程也会和其他tables有交互

#### 进程控制块

PCB 存储不同信息的内容与功能介绍如下表2-2所示

| PCB 存储信息| 内容和功能 (P87) |
| :---:| :---:|
| 标识符 | PID 和 进程名称(名称会重复) win端使用 `tasklist`可以查看 |
| 状态 | |
| 优先级 | |
| 程序计数器|下一条执行指令的地址 |
| 内存指针 | 程序代码和进程相关数据指针、共享内存块指针等 |
| 上下文 | 处理器的寄存器数据信息 |
| IO状态 | 处理器的使用时间总和、时间限制等，设备状态 |
| 审计信息 | 执行时间，使用资源。提供操作系统参考和使用|

进程标识

- 标识符

处理器状态信息

- 用户可见寄存器、控制和状态寄存器，栈指针

进程控制信息

- 调度和状态信息，进程间通信，特权、存储管理、资源使用情况

进程的组成由下图 2-10 所示

![进程的组成](../Img/Chapter2-10-进程的组成.png)


- 操作系统维持着一个由PCB组成的链表，根据链表中的PCB来控制系统中的进程
- 进程控制块面临的难题-安全保护
  - 病毒回不回出现在PCB中，或修改其他进程

----
#### 进程图 Process Graph
- 进程图是用于描述一个进程的家族关系的有向树
- **子进程**可以继承**父进程**所拥有的资源
- 当子进程被撤销时，应该将其从父进程那里获得的资源归还给父进程
- 特别的：若是父进程先子进程死掉，子进程可能会进入孤儿进程状态（若是子进程退出了，父进程不清楚，则进入僵尸进程状态）

#### 进程的创建 Process Creation

会导致进程创建的事件:
- 系统初始化 
- 正在运行的进程调用了一个进程创建系统
- 用户请求
- 批处理作业的初始化


创建进程的步骤：
- 首先分配一个唯一进程标识符，分配相应的空间（包括进程映像所有元素）
- 然后调用进程**创建原语**按照下述步骤创建一个新进程：
    1. 申请空白PCB
	2. 为新进程分配资源，为新进程的程序和数据以及用户栈分配必要的内存空间
	3. 初始化进程控制快
		- 初始化标识信息
		- 初始化处理机状态信息。使得程序计数器指向程序的入口地址，使栈指针指向栈顶
		- 初始化处理机控制信息：进程的状态、优先级
	4. 将新进程插入就绪队列


#### 进程终止

进程终止的几种情况
- 普通终止（自愿退出）
- 错误终止（自愿退出）
- 致命性错误 （非自愿退出）
- 被其他进程杀死 （非自愿退出）

利用**终止原语**（原子操作，不可再分） 终止进程
- 检索被终止的进程PCB
- 终止该进程的执行 （若有子进程，一并终止）
- 回收资源
- 将该进程PCB从当前队列中移出

#### 进程的阻塞

调用**阻塞原语**自我阻塞

因为是自我阻塞，所以阻塞是发生在运行过程中的。因此会插入到不同的阻塞队列中，此时会
- 暂停进程的执行，修改PCB运行状态
- 将PCB插入阻塞队列
- 调度新进程

#### 进程的唤醒

调用**唤醒原语**唤醒进程
- 检索阻塞队列，寻找想要换新进程的PCB
- 修改PCB的运行状态
- 插入PCB到就绪队列中

#### 进程的挂起与激活

利用**挂起原语**挂起进程  
利用**激活原语**激活进程

---
### 进程的切换

进程切换的主要步骤
1. 保存处理器状态信息
2. 更新当前进程的状态
3. 将当前进程PCB移动到相应的队列
4. 选择另一个进程PCB
5. 更新该进程的PCB
6. 更新内存管理的数据结构
7. 恢复当前进程被切换前的上下文信息


#### 模式切换

模式切换可以不改变正处于运行态的进程的状态
- 进程切换会导致模式切换，反之不一定有改变
- 保存上下文环境与恢复上下文环境只需要很小的开销

---
## 线程 Threads

在讨论线程前，回顾进程的概念 
- 每个进程有自己独立的地址空间
- 每个进程拥有自己资源的控制权
  - I/O、主存、文件......
- 进程是操作系统的最小调度单位
  - 进程切换

因此，为了减少操作系统的开销，现代操作系统引入线程的概念，因此有：
1. 线程是调度的最小单位  
2. 进程是资源拥有的最小单位  
3. 一个进程可以拥有多个线程
4. 组合方式有：
   1. 单进程单线程
   2. 单进程多线程
   3. 多进程单线程
   4. 多进程多线程

线程的特点：
1. 创建块（比进程大约快10倍，因为实在进程的资源基础上创建）
2. 终止快（线程终止不需要被回收）
3. 切换快（一般在进程内部切换，保存信息较少）
4. 通信快（进程间通信无需经过内核）

线程进程模型由下面的图2-11所示

![进程线程模型](../Img/Chapter2-11-线程进程模型.png)

因此综上所述，**在支持线程的操作系统中**，**线程**是**调度分派**的最小单位，**进程**是**资源拥有**的最小单位

线程与进程类似，同样具有几种基本状态
- 就绪、执行、阻塞

因为线程是基于进程的资源所建立，因此对进程的操作（如调度等），都会影响到进程中的所有线程

---
### 线程的基本操作

- 派生（Spwawn）：当产生一个新进程时，同时操作系统也为该进程派生了一个线程，随后，进程中的线程可以在同一个进程中派生另一个线程，新线程被放置在就绪队列中
- 阻塞（Block）：当线程需要等待一个时间时，它将阻塞，此时处理器转而执行另一个就绪线程
- 解除阻塞（Unblock）：当阻塞一个线程的事件发生时，该线程被转移到就绪队列中
- 结束（Finish）：当一个线程完成时，其寄存器的信息和栈都将被释放

线程的实现
 - 用户及线程
   - 现场的管理应该由应用程序完成
 - 内核级线程
   - 线程的管理应该由内核完成，应用程序通过API访问线程

用户及线程与内核级线程情况由下面的图2-12所示

![用户级线程和内核级线程](../Img/Chapter2-12-用户级线程和内核级线程.png)

用户级线程：操作系统感知不到线程存在，只知道进程存在。信息交换只在用户的空间内完成

内核级线程：调用API完成，操作系统可以感知到。用户空间不能完成信息交换，需要调用API在内核完成信息交换，需要模式切换有较大开销

**用户级线程**与*内核级线程*的比较
1. 用户级线程
     - 用户级线程不需要模式切换（由应用程序代为完成）
     - 用户级线程由应用程序实现调度管理
     - 用户级线程可以再任意操作系统中运行（C语言提供用户级线程库）

	**缺点**
		- 用户级线程等执行系统调用时，同一进程所有线程都会被阻塞
		- 不能利用多处理机技术
  
2. 内核级线程
    - 内核实现内核级线程的调度管理
    - 可以充分利用多处理技术

	**缺点**
	- 有模式切换的开销


---
### 进程的并发

并发是所有问题的基础，也是操作系统设计的基础

由并发带来的两个问题
- 对资源的相互制约：同步
- 对资源的相互共享：互斥

#### 相关的关键概念

**临界资源**：一次仅运行一个进程访问的资源为临界资源

**临界区**：
- 把在每个进程中访问临界资源的那段**代码**称为临界区
- 代码作为一个共享资源，一次只能允许一个进程访问

**死锁**：两个或两个以上的进程之间相互等待临界资源导致都不能执行的情况

**互斥**：当一个进程在临界区访问临界资源时，其他进程不能进入该临界区访问共享资源

**竞争**：多个进程读写一个共享数据时以来它们执行的相对时间
- 竞争条件发生在多个进程或县城读写数据时，其最终结果取决于多个进程的指令执行顺序

**饥饿**:一个进程始终得不到执行机会

**饿死**：饥饿的极端情况，进程因为不能按时完成任务而结束

---

进程的并发会导致程序执行结果不封闭（不封闭体现在执行顺序的偏差上）

- 全局资源  
  对全局资源的访问秩序非常重要

- 资源分配  
  不好的分配算法可能导致死锁

因此为了实现对临界资源的访问，每个进程都**互斥**地进入自己的临界区：**一次只有一个程序在临界区**

进程访问临界区的步骤：

1. 进入临界区之前，对预先访问的临界资源进行检查
2. 若该资源尚未被访问，则可以进入临界区；反之则不能
3. 设置正在访问标志
4. 进程使用资源完毕后，需要将资源恢复为未访问标志

---
#### 互斥可能带来的问题
- 死锁：进程P1占有资源R1，等待资源R2；进程P2占有资源R2，等待资源R1
- 饥饿：无限期地被推迟访问

##### 同步机制应该遵循的准则 :star:

1. **空闲让进**(无进程进入临界区，则允许一个请求进去临界区的进程进入)
2. **忙则等待**（有进程位于临界区。其他预进入的进程需要等待）
3. **有限等待**（预进入临界区的进程，应该在**有限时间**内有机会进入临界区）
4. **让权等待**（进程无法进入自己临界区时，应该立即释放处理机）

##### 实现互斥访问

- 严格轮换  
  每个进程每次都从头执行到尾

- 屏蔽中断
	刚刚进入临界区时就屏蔽中断，刚要出临界区就打开中断

- 专用机器指令
	`test_and_set`、`test_and_clear`

- 软件方法
	- 使用信号量（Semaphore）

##### 软件方法解决互斥与同步

1. 能否保持互斥
2. 会不会出现互斥礼让
3. 会不会发生资源死锁

算法分析（Dekker`s Algorithm）  
C语言伪代码形式

```C
int turn = 0;
    process 0{
        do{
            nothing;
        }while(turn != 0)
        <critical section> //临界区
        turn = 1;
    };
    process 1{
        do{
            nothing;
        }while()
        <critical section> //临界区
        turn = 0;
    };
```

解析：保持了互斥，但仍然存在问题：进程“忙等”进入临界区，若标志修改失败，则其他进程会**永久堵塞**

---

另一份代码,优化版
```C
bool flag[2]; //共享的全局变量
    Process 0
    {
        while(flag[1]){
            nothing; //直接结束
        }
        flag[0] = 0;
        <critical section>;
        flag[0] = false;
    };
    Process 1
    {
        while(flag[0]){
            nothing; //直接结束
        }
        flag[1]=true;
        <critical section>;
        flag[1] = false;
    };
```

解析：若进程需求临界区资源完毕，恢复自己标志位为"FALSE"失败，则其他进程永久阻塞

---
在检查其他进程之前，若是希望得到临界区资源，则设置自己的标志位，flag = True。

当设置标志位为真后，如果其他进程在临界区，则本进程将阻塞，知道其他进程释放临界区资源为止

```C
bool flag[2] = 0;
    Process 0 {
        flag[0] = true;
        while (flag[1]){
            nothing // 直接退出
        }
        <critical section>;
        flag[0] = flase;
    };
    Process 1 {
        flag[1] = true;
        while(flag[0]) {
            nothing; //直接退出
        }
        <critical section>
        flag[1] = false;
    };
```

**缺点**:如果两个进程在执行`while`之前豆浆标志位`flag`设置成True，那么每个进程都会认为对方进入了临界区，从而使得自己被阻塞，因为互相阻塞，会导致**死锁**产生。

---
另一个算法将优化上面算法的问题

主要思想：将标志位重置，不会发生死锁

实现过程：
- 如果一个进程希望进入临界区，则需要将自己的标志位：flag = true
- 如果其他进程已经在临界区，则将本进程标志位置为flag=false，设置一个延时，稍后又置位为true，这一过程将重复到最终进程能够进入到临界区为止

实现算法如下所示
```C
bool flag[2] = 0;
    Process 0 {
        flag[0] = true; //希望进入临界区
        while(flag[1]) {
            flag[0] = false;
            <delay for a shot-time>
            flag[0] = true;
        }
        <critical section>;
        flag[0] = true;
    };
    Process 1 {
        flag[1] = true;
        while(flag[0]) {
            flag[1] = false;
            <delay for a shot time>;
            flag[1] = true;
        }
        <critical section>
        flag[1] = false;
    };
```

实质上，上述算法的中心思想就死，检查其他进程，然后重置，再检查，再重置,重复循环上述过程.

重置序列因为可以无限延伸，使得任何一个进程都不能进入自己的临界区。这种现象又称之为 **互斥礼让**

**互斥礼让**的另一种算法

```C
bool flag[2] ;
    bool turn = 0;
    Process1{
        flag[0] = true; //希望进入临界区
        turn = 1;
        while(flag[1] && turn == 1){
            nothing;
        }
        <critical section>
        flag[0] = false;
    };
    Process2{
        flag[1] = true;
        while (flag[0] && turn == 0){
            nothing;
        }
        <critical section>;
        flag[1] = false;
    };
```

---
### 信号量

#### 整型信号量

最初是由 Dijkstra 把整型信号量定义为一个整型量，除初始化外，仅能通过两个标准的**原子操作**（Atomic Operation）即，wait（s） P操作，signal（s）V操作，这两个操作来访问。

其中wait，与signal操作的算法表达式如下
```C
    p(s): while(s<=0) s=s-1 ; 
    v(s): s = s+1;
```

特别需要注意的：不能对信号量直接做加减操作，因为加减操作并不满足**原子操作**。需要调用P(S)与V（S）实现。

面临的问题
- 只要信号量$S\leq0$,他就会不断地进行测试。因此，该机制并未遵循“让权等待”的准则，可能出现盲等的情况。

#### 记录型信号量

Value:代表可用资源数

L:链接所有等待进程

代码算法表示如下
```C
typedef struct semaphore{
        int Value;
        List_of_Process L;
    };
    P(S):
        S.value = S.value -1 ;
        if (s.value<0){
            block(S,L);//切换至阻塞队列
        }
    V(S)：
        S.Value = S.value + 1;
        if (s.balue<0){
            wakeup(S,L);
        }
```

算法解释：
- `S.value`的初值表示系统中某类资源的数目，因而又称为资源信号量，对它的每次wait操作，意味着进程请求一个单位的该类资源，因此描述为`S.value=S.value-1`；

- 当`S.value＜0`时，表示该类资源已分配完毕，因此进程应调用`block`原语，进行自我阻塞，放弃处理机，并插入到信号量链表`S.L`中。可见，该机制遵循了“**让权等待**”准则。 此时`S.value`的绝对值表示在该信号量链表中已阻塞进程的数目。

- 对信号量的每次`signal`操作，表示执行进程释放一个单位资源，故`S.value=S.value+1`操作表示资源数目加1。若加1后仍是S.value≤0，则表示在该信号量链表中，仍有等待该资源的进程被阻塞，故还应调用`wakeup`原语，将S.L链表中的第一个等待进程唤醒。

- 如果`S.value`的初值为1，表示只允许一个进程访问临界资源，此时的信号量转化为互斥信号量。

- 因此有，强信号量，按排队顺序唤醒。弱信号量，唤醒资源靠**竞争**

信号量的机制如下图2-13所示

![信号量](../Img/Chapter2-13-Semaphore_Mechanism.png)

#### AND信号量

AND信号量同步机制的基本思想

**将进程在整个运行过程中需要的所有资源，一次性全部地分配给进程，待进程使用完后再一起释放。只要尚有一个资源未能分配给进程，其它所有可能为之分配的资源，也不分配给他.** 即，对若干个临界资源的分配，采取原子操作方式：要么全部分配到进程，要么一个也不分配。

#### 信号量集

思考：记录型信号量有何不便之处？
- 当一次需要多个资源时，需要进行多次`P`操作
- 同理，释放占用资源时，也要进行多次释放`v`操作

如何改进，可以尝试将 t作为下限值，d作为需求值

```c
Swait(S1, t1, d1, …, Sn, tn, dn)
    if ( S1≥t1 and … and Sn≥tn )
       for  ( i=1;i<=n; i++)
             Si =Si-di;
    else
        Place the executing process in the waiting queue of the first Si with Si＜ti and set its program counter to the beginning of the Swait Operation. ;
signal(S1, d1, …, Sn, dn)
   for  ( i=1;i<=n; i++){
       Si =Si+di;
       Remove all the process waiting in the queue associated with Si into the ready queue;
   } 
```

一般“信号量集”的几种特殊情况：

1. `Swait(S, d, d)`。 此时在信号量集中只有一个信号量`S`， 但允许它每次申请`d`个资源，当现有资源数少于`d`时，不予分配。
2. `Swait(S, 1, 1)`。 此时的信号量集已蜕化为一般的记录型信号量(`S＞1`时)或互斥信号量(`S=1`时)。
3. `Swait(S, 1, 0)`。这是一种很特殊且很有用的信号量操作。当`s>=1 `时，允许多个进程进入某特定区；当`S = 0`后，将阻止任何进程进入特定区。换言之，它相当于一个可控开关。

### 信号量的应用

#### 利用信号量实现进程互斥

利用信号量实现互斥代码算法实现情况如下图所示

```C
Process1:
        Semaphore mutex = 1;
        while (1){
            p(multex);
            <critical section>;
            v(multex);
        }
Process2:
        Semaphore mutex =1;
        while (1){
            p(mutex);
            <critical section>
            v(mutex);
        }
```
---

#### 利用信号量实现前趋关系

前驱关系的mermaid示意图例如下所示

```mermaid
graph LR
a(S1) --a--> b(s2)
a --b--> c(s3)
b --c--> d(s4)
b --d--> e(s5)
d --f--> f(s6)
e --g--> f(s6)
c --e--> f(s6)
```

根据上图关系可知，需要保证在前趋结束后，才能开始执行后面的进程

```C
    Semaphore a=0,b=0,c=0,e=0,f=0;
    Semaphore g = 0;
    //使用信号量必须先进行初始化
    P(a);S2;V(c);V(d);
	P(b);S3;V(e);
	P(c);S4;V(f);
	P(d);S5;V(g);
	P(e);P(f);P(g);S6;
```

同一层级的执行顺序依靠算法进行调度

#### 生产者消费者问题 :star:

模型组成原理:
1. 生产者生产产品，提供给消费者去消费。  
2. 为使得生产者进程和消费者进程能**并发执行**（执行中可能会被打断），在两者之间设置了具有`*n*`个缓冲区的缓冲池。  
3. 生产者进程将他生产的产品放入缓冲池（**一次一个**），消费者进程从缓冲池中拿走产品（**一次一个**）。  
4. 缓冲池**已满时**，生产者不能**再放**，缓冲池**已空时**，此时消费者**不能再拿**。

生产者与消费者之间的联系(模型特点)
- 互斥
  - **共享缓冲区**（缓冲区作为一种临界资源）
- 同步
  - **相互等待**（有产品才能消费、有消费才能不断生产）


使用循环缓存，只需要单一方向生成/消费即可

假定在生产者和消费者之间的公用缓冲池中，具有n个缓冲区，这时可利用**互斥信号量Mutex**实现诸进程对缓冲池的互斥使用

利用**信号量Empty和Full**分别表示缓冲池中空缓冲区和满缓冲区的数量。

生产者-消费者模型算法实现 (C伪代码)

```C
Semaphore Mutex=1; //定义互斥信号量，初始值必须为1
    Semaphore Full=0,Empty = n; //定义同步信号量
    //这样初始化是为了阻塞消费者，此时消费者可用资源为0，n是缓冲区大小，为生产者可生产空位
    product_Item Buffer[n]; //定义产品缓冲区
    int in = 0; //定义生产者初始化指针
    int out = 0; //定义消费者初始化指针
    
    Producer:// 生产者进程
    while (1){
        Create goods; // 生产者生产产品 Pro_Item
        p(Empty);
        p(Mutex); //上互斥锁
        Buffer[in] = Pro_Item;
        in = (in++)%n;
        v(Mutex);//解锁
        v(full);
    }
    
    Consumer:
    while (1){
        p(Full);
        P(Mutex);
        Item = Buffer[out];
        out = (out++) % n;
        v(Mutex);
        v(Empty);
        ...; //消费者消费过程
    }
```

互斥信号量：PV同一进程中成对

同步信号量：PV不同进程中成对

如果 empty和Mutex 交换 会导致死锁，产生原因 生产者连续生产n次后，再次进行P([pro]mutex) P1([pro]empty) P2([con]mutex) 死锁

因此

1. P操作一定要对同步信号量操作，再对互斥信号量操作
2. PV成对出现

P不当会导致死锁，V操作不会导致阻塞

（P操作为请求资源，V操作为释放资源）

---
##### 利用AND 信号量解决 生产者-消费者问题

此时引入新的操作`Swait`,`Ssignal`，因为同步信号量机制要求规定，必须当一个进程所需的全部资源都满足时才进行分配，故此引入`Swait`，来实现同时分配进程所需的全部资源，`Ssiganl`来释放进程的所有资源。

算法实现，如下所示

```C
	Semaphore Mutex = 1;//定义互斥信号量，互斥信号量恒为1
    Semaphore Full = 0, Empty = n;//定义同步信号量
    //这样初始化是为了阻塞消费者，此时消费者可用资源为0，n是缓冲区大小为生产者可生产空位
    Product_Item Buffer[n]; //定义产品缓冲区
    int in = 0;
    int out = 0;

    Producer:
    while (1){
        ...; //生产者生产产品Product_Item
        Swait(Empty,Mutex); //and型信号量，需要同步操作
        buffer[in] = Product_Item; //填入产品
        in = (in ++ )%n;
        Ssignal(Mutex,Full); //and型信号量，需要同步操作
    }

    Consumer:
    while(1){
        Swait(Full,Mutex);
        Item = Buffer[out];
        out = (out++)%n;
        Ssignal(Mutex,Empty);
        ...;//消费者消费过程
```

---
##### 无限缓存问题

缓冲区无限大，导致生产者**不停的放**

满足：
- 互斥
- 消费者有限制

代码实现如下所示
```C
    binary_Semaphore s = 1,delay = 0; //创建二元信号量
    
    void producer(){
        while(1){
            produce();
            p(s);
            append(); //尾插入资源
            n++; //资源数增加
            if (n == 1) v(delay); //通知消费者有数据可取
            v(s);
        }
    }
    
    void consumer() {
        p(delay) //锁定资源
        take();
        n-- //资源数减少
        v(s);
        consume();
        if (n == 0) p(delay);//资源耗尽，发出提示
    }
void main(){
	int n = 0;
    perbegin(producer,consumer);
}
```

上述程序会发生错误，理由如下表 2-3 所示

|      | Producer                   | Consumer                                            | s    | n    | Delay |
| :----: | :--------------------------: | :--------------------------------------------------- | :----: | :----: | :-----: |
| 1    |                            |                                                     | 1    | 0    | 0     |
| 2    | semWaitB(s)                |                                                     | 0    | 0    | 0     |
| 3    | n++ //生产                 |                                                     | 0    | 1    | 0     |
| 4    | if(n==1) semSignalB(delay) |                                                     | 0    | 1    | 1     |
| 5    | semSignalB(s)              |                                                     | 1    | 1    | 1     |
| 6    |                            | semWaitB(delay)                                     | 1    | 1    | 0     |
| 7    |                            | semWaitB(s)                                         | 0    | 1    | 0     |
| 8    |                            | n-- //消费                                          | 0    | 0    | 0     |
| 9    |                            | semSignalB(s)                                       | 1    | 0    | 0     |
| 10   | semWaitB(s)                |                                                     | 0    | 0    | 0     |
| 11   | n++// 生产                 |                                                     | 0    | 1    | 0     |
| 12   | if(n==1) semSignalB(delay) |                                                     | 0    | 1    | 1     |
| 13   | semSignalB(s)              |                                                     | 1    | 1    | 1     |
| 14   |                            | if(n==0) :warning:semWaitB(delay) //这里delay不匹配 | 1    | 1    | 1     |
| 15   |                            | semWaitB(s)                                         | 0    | 1    | 1     |
| 16   |                            | n-- //这里消费空数据                                | 0    | 0    | 1     |
| 17   |                            | semSignalB(s)                                       | 1    | 0    | 1     |
| 18   |                            | if(n==0) :warning:semWaitB(delay)                   | 1    | 0    | 0     |
| 19   |                            | semWaitB(s)                                         | 0    | 0    | 0     |
| 20   |                            | n--:stop_sign:                                      | 0    | -1   |       |


主要问题发生在，生产者与消费者同时对n进行操作，让消费者在第一轮生产消费循环后，在第二轮中，消费者消费了一个并不存在的数据内容，导致出现死锁

改进版代码如下所示：

```C
    int n;
    binary_Semaphore s = 1,delay = 0; //创建二元信号量

    void producer(){
        while(1){
            produce();
            p(s);
            append(); //尾插入资源
            n++; //资源数增加
            if (n == 1) v(delay); //通知消费者有数据可取
            v(s);
        }
    }

    void consumer() {
        int m; // 引入额外本地变量，防止对n重复操作
        p(delay) //锁定资源
        while(1){
            p(s);
            take();
            n -- ;
            m = n;
            v(s);
            consume();
            if (m == 0) p(delay); //引入的m充当资源的作用
        }
    }
    n = 0;
    perbegin(producer,consumer);
```

---
#### 哲学家就餐问题

哲学家就餐问题的问题描述
- 哲学家需要就餐，需要同时使用左边与右边的筷子
- 一只筷子同时只能被一个人使用
- 使用完后放归原处

由上不难看出，哲学家就餐问题是**竞争关系**的很好体现

示例情况代码如下所示
```C
    Semaphore Chopstic[n]{1, 1, 1, 1, 1}; //定义n个共享资源
    while (1){
        p(Chopstic[i]);
        p(Chopstic[(i+1)%n]);
        Eating food ......;
        V(Chopstick[(i+1)%n]);
        v(Chopstick[i]);
        .....LOOP
    }
```

#### 盘子问题 （类似竞争问题）

问题描述：有一只空盘，只能每次往上放一个水果，父亲往盘子里放一个苹果或者一个橘子，儿子只拿橘子，女儿只拿苹果。

需要参考生产者-消费者模型，

互斥信号量：盘子（临界资源，每次只能放一个）

同步信号量：资源信号量（针对消费者，针对生产者）

算法实现如下所示

```C
    Semaphore Mutex = 1;
    Semaphore Empty = 1;
    Semaphore s_apple = 0,s_orange = 0;

    father:
        while(1){
            p(Empty);
            p(Mutex);
            put fruit;
            if(Orange) s(s_orange)
            else v(s_apple)
            v(Mutex)
        }

    son:
        while(1){
            p(s_orange);
            p(Mutex)
            Fetch orange;
            v(Empty);
            v(Mutex);
            Eating Orange;
        }
    
    daughter:
        while(1){
            p(s_apple);
            p(Mutex);
            Fetch Apple;
            v(Empty);
            V(Mutex);
            Eating Apple;
        }
```

通过以上例子可以说明：
1. 一定要先 P(资源信号量),再P（互斥信号量）
2. 一定不能忘记对生产者也有资源信号量

---
#### 读者-写者问题

核心问题**共享关系**的描述

问题描述：一个作家写，多个读者读，作家一次只能写一本小说，但是一本小说可以同时提供给多个读者读。

需求描述
- 不能同时写文件
- 不能同时读和写文件
- 可以同时读文件

为了满足问题需要 即在 `Reader` 与 `Write` 进程间在读或者写时的互斥而设置了一个**互斥信号量`Wmutex`**.

设置一个**整型变量`Readcount`表示正在读的进程数目**。由于只要有一个`Reader`进程在读，便不允许`Writer`进程去写。因此，仅当`Readcount=0`, 表示尚无`Reader`进程在读时，`Reader`进程才需要执行`Wait(Wmutex)`操作。若wait`(Wmutex)`操作成功，`Reader`进程便可去读，相应地，做`Readcount+1`操作。同理，仅当`Reader`进程在执行了`Readcount-1 = 0 `操作后，才须执行`signal(Wmutex)`操作，以便让`Writer`进程写。

`Readcount`是一个可被多个`Reader`进程访问的临界资源，因此，应该为它设置一个**互斥信号量rmutex**。 

算法代码表示如下
```C
    Semaphore Rmutex=1,Wmutex=1;
    int ReaderCount = 0;
    Reader:
    while(1){
        P(Rmutex); //对共享资源Rmutex加锁，Rmutex为读写互斥锁，Rmutex为读者数量互斥锁（只能被一次被一个Reader修改的Readcount）
        if(ReaderCount == 0) {
            p(Wmutex); //读之前对临界资源文件加互斥锁,Wmtex只有在Readercount计数为0时，才可锁上，允许写入
        }
        ReaderCounter++;
        V(Rmutex);
        //Reader 开始阅读
        p(Rmutex);
        ReaderCount = ReaderCOunt - 1;
        if(ReaderCount == 0){
            V(Wmutex); //若无读进程，对临界资源进行写操作
        }
        V(Rmutex); //对Rmutex解锁
    }

    Writer:
    while(1){
        P(Wmutex);
        //作家开始写入文件
        V(Wmutex)
    }
```

如果利用mermaid图进行展示，可如下图

```mermaid
stateDiagram
state 请求顺序{
R0 --> W0
W0 --> R1
R1 --> R2
}
请求顺序  --> 实际顺序:运行乱序

state 实际顺序{
R0 --> R1
R1 --> R2
R2 --> W0
}
```

从这里可以观察到，显然，只要有多位**读者**，就无法修改内容。所以需要进行优化，让写者优先，不然会处于一直无法更新内容状态，使得**写者**进程饥饿



优化后的代码如下所示｝

```C
    int readcount,writecount; //调用时，均初始化为0
    Semaphore x=1,y=1,z=1,wsem=1,rsem=1;
    // rsem:至少有一个进程准备访问数据区时，用于禁止所有的读进程
    // writecount：用于控制rsem
    // y:控制writecount 更新
    
    void reader(){
        while(1){
            semWait(z);
                semWait(rsem);
                semWait(x);
                readcount++;
                if(readcount == 1) semWait(wsem);
                semsignal(x);
            semSignal(rsem);
        semSignal(z);
        
        READUNIT();
        
        semWait(x);
            readcount--;
            if(readcount == 0) semSignal(wsem);
        semSignal(x);
        }
    }
    
    void  writer(){
        while(1){
            semWait(y);
                writeCount ++;
                if(writeCount == 1) semWait(rsem);
            semSignal(y);
            
            semWait(wsem);
            WRITEUNIT();
            semSignal(wsem);
            
            semWait(y);
                writecount -- ;
                if(writecount == 0) semSignal(rsem);
            semSignal(y);
        }
    }
```

---
#### 将三个经典例子牢记于心

- 生产者消费者问题：需要相互通知
- 哲学家就餐问题：需要
- 读写问题：间接知道（甚至不知道），团体问题

参考其他示例

- 理发师问题

解决P、V操作问题的关键 :star:

- 理解临界资源与临界区的概念！
- 准确理解问题的同步互斥过程与要求！
  - 同步：多个进程在执行次序上的协调，相互等待消息
  - 互斥：对临界资源的使用
- 建立信号量，准确定义信号量的意义和初始值！
- 信号量的定义根据同步和互斥要求来定


## 管程 Monitor

定义：管理进程同步与互斥的一种工具（管程将信号量与原子操作相结合，减少了操作量与程序复杂度）

Dr.Hansan 为管程所下的定义是：“一个管程定义了一个数据结构和能为并发进程所执行（在该数据结构上）的一族操作，这组操作能同步进程和改变管程中的数据”

![管程](../Img/Chapter2-14-Monitor.png)

### 利用管程来解决生产者与消费者问题

实现代码如下所示

```C
    Monitor proCon{  //管程
        int in = 0,out =0,count = 0;
        message buffer[n],inM,outM;
        condition notFull,notEmpty;
        Semaphore mutex = 1,empty = n,full = 0
    public:
        void put(message x){
            if(count >= 0) Cwait(notFull);
            buffer[in] = x;
            in = (in+1) % n;
            count ++ ;
            Csignal(notEmpty)
        };
        void get(message x){
            if(count<0) Cwait(notEmpty)
            x = buffer[out];
            out = (out+1) % n;
            count --;
            Csignal(notFull);
        }
    }PC;

    void procedure(){ //生产者
        message x;
        do{
            ....
            produce a message inM;
            Swait(empty,mutex)l
            PC.put(inM);
        }while(1)
    }
    
    void consumer(){  //消费者
        message outM;
        do{
            PC.get(outM);
            do{
                pc.get(outM);
                consume the message in outM;
                ....
            }while(1)
        }
        void main(){
            parbegin(procedure(),consumer());
        }
    }
```

函数分析与名词解释
- put(x)
  - 生产者利用该管程过程将生产的产品放入缓冲池
  - 整型变量count表示缓冲池中的产品数量,count>=n表示缓冲池已满
  - count++

- get(x)
  - 消费者利用该管程过程从缓冲池中取走一个产品。
  - 整型变量count表示缓冲池中的产品数量，count>=n表示缓冲池已满，count<=0则表示缓冲池已空
  - count --

condition 型变量
- notFull: 缓冲池不满
- notEmpty：缓冲池不空

Cwait(condition管程条件变量):管程被某进程占用时，其他进程调用Cwait(condition)进行阻塞，并插入condition的阻塞队列。

Csignal(condition)：唤醒被Cwait(condition)阻塞的进程
- condition 的阻塞队列为空：不用操作
- condition 的阻塞队列不为空：唤醒该队列的一个进程


**管程的主要特点**

- 局部数据变量只能被管程的过程访问，任何外部过程都不能访问。
- 一个进程通过调用管程的一个过程进入管程。
- 在任何时候，只能有一个进程在管程中执行，调用管程的任何其他进程都被挂起，以等待管程变成可用的。
- 为进行并发处理，管程必须包含同步工具
- 管程通过使用条件变量提供对同步的支持，这些条件变量包含在管程中，并且只有在管程中才能被访问。有两个函数可以操作条件变量：
  - Cwait(c)：调用进程的执行在条件c上挂起，管程现在可被另一个进程使用
  - Csignal(c)：恢复在cwait之后为某些条件而挂起的进程的执行。如果有多个这样的进程，选择其中一个；如果没有这样的进程，什么也不做

## 进程间通信 IPC

进程通信即进程之间的信息交换，分为低级通信和高级通信两类
- 低级通信：进程之间通信的数据量极少，如进程的互斥与同步
  - 效率低
  - 通信过程对用户不透明
- 高级通信
  - 用户直接利用操作系统提供的一组通信命令高效的传送大量数据的通信方式，有以下几种方式
    - 共享存储器系统
		1. 基于**共享存储区**的方式属于高级通信方式
		2. 基于**共享数据结构**的方式属于低级通信方式
    - 消息传递系统
        1. 直接消息传递系统：两个或多个进程利用系统提供的发送命令/原语，直接将消息发送给目标进程。
        2. 间接消息传递系统：又称信箱通信，即通过信箱通信原语实现通信
    - 管道通信
        - 管道通信中的“管道”指用于连接一个读进程和一个写进程以实现二者之间通信的共享文件，又称`pipe`文件。
          1. 向管道/共享文件提供输入的发送进程/写进程以字符流的形式将大量数据送入管道
          2. 接收管道输出的接收进程/读进程从管道中接收/读取数据
          3. 高效传送大量数据
          4. 首创于UNIX系统，广泛应用于其他操作系统
        - 管道机制必须提供以下协调能力
          - 互斥：当某进程正在对`pipe`执行读/写操作时，其他进程必须等待
          - 同步：当写/输入进程把一定数量的数据写入`pipe`后，就进入睡眠等待，知道读/输出进程取走数据后再将写/输入进程唤醒;`pipe`为空时，读进程睡眠等待，直至写进程将数据写入管道后才将其唤醒。
           - 确认对方是否存在，只有确定对方已经存在后才开始通信

  - 高级通信机制又具有以下特点：
    1. 使用方便/透明
    2. 高效传输大量数据