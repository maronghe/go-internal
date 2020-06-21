> Defer 1.12 vs 1.13 vs 1.14

## **1.12**

defer func(){}  —> 会被翻译成两个函数

1. deferproc() : 注册，defer func(){} 到g的defer链表中，头插法。defer需要进行堆分配，而且参数变量需要进行堆栈间拷贝。

2. runtime.deferreturn() : 执行，defer链表的中的每一项

    （先注册 -》 后执行）

   ```go
   // runtime/runtime2.go
   type g struct {
   	...
   	_panic       *_panic 
   	_defer       *_defer // defer 链表
   	...
   }
   ```

   

步骤解析：

`deferproc`

1. 从`defer pool`预分配不同规格的defer，不满足时再创建，用完再放回池中。
2. 进行堆分配`_defer`结构体，将参数拷贝到堆上
3. 连接到`g`的`defer`链表上

`runtime.deferreturn`

1. 执行`defer`注册的`funcval`，将堆上的参数拷贝到栈上，进行执行。

注意：

1. 若其中有发现闭包，堆上分配捕获变量的地址，执行时通过指针加偏移量找到被捕获的变量进行操作
2. 形如`defer A(B(a))` 
   1. 函数A需要依赖B的返回值进行堆分配内存大小
   2. 所以B函数在注册时候执行，传入变量啊

存在问题：（**慢!!!!!**）

1. defer结构体堆分配，参数需要进行堆栈间拷贝。 GC
2. 链表注册defer信息，执行比较慢。



## **1.13**

**性能提升：30%**

增加`runtime.deferprocStack()`函数：

1. 编译阶段：将defer的参数分配到栈帧的局部变量部分。
2. `deferprocStack`defer内存分配到栈上，并注册到defer链表中，**以减少defer的堆分配**  // TODO 
3. 但是对于[循环defer](https://tva1.sinaimg.cn/large/007S8ZIlly1gfxptmrehij308o04udg3.jpg)或[隐式循环注册](https://tva1.sinaimg.cn/large/007S8ZIlly1gfxpd6ikvfj30a008ogm3.jpg)还是需要进行堆分配
4. defer结构体中加入`heap`字段区分是否为堆分配

`runtime.deferreturn`

1. 若没分配到堆中的defer函数会进行栈的`局部变量`拷贝到`参数空间`
2. 分配到堆上的**还是进行参数的堆栈间拷贝**



## 1.14

**性能提升一个数量级**

改造1： 普通的 `defer A(a,b)` 会进行分配局部变量`a,b`，在函数`ret`之前进行函数调用，达到延迟执行的效果

改造2： 对于带有条件的`defer A2()` 函数，需要到执行期间才知道是否需要执行。

	1.	引入`df`变量进行标记df的每一个bit是0或1，来进行决定defer是否需要执行
 	2.	执行过了标记为0，避免重复执行。



目标就是通过编译是插入代码，将defer在函数内进行展开执行，就不用defer结构体测创建和注册defer链表了。

官方称之为 `open coded defer`

但同1.13 一样不适用于循环defer和隐式注册，所以go1.12还是进行保留。



付出代价：

1. `panic()`和`runtime.GoExit()` 退出需要执行defer链表，
2. 但是由于没注册链表，
3. 所以需要栈扫描的方式来找到未注册defer函数
4. 才能按照正常执行。



`整体性能优化  > panic发生的概率`

















































