Go sync.Pool 对象池

> sync.Pool between 1.12 and 1.13

​	sync提供强大的‘对象池’，可以重用对象为了减少GC的压力。在使用此包之前，对于你的程序有个性能测试是一个前提。并使用完pool之后，如果你不知道它内部的工作原理，可能会降低你程序的性能。

​	让我们从一个小例子开始。

```go
package pool_test

import (
	"sync"
	"testing"
)

type Small struct {
	a int
}

var pool = sync.Pool{
	New: func() interface{} { return new(Small) },
}

//go:noinline
func inc(s *Small) { s.a++ }

func BenchmarkWithoutPool(b *testing.B) {
	var s *Small
	for i := 0; i < b.N; i++ {
		for j := 0; j < 10000; j++ {
			s = &Small{a: 1}
			inc(s)
		}
	}
}

func BenchmarkWithPool(b *testing.B) {
	var s *Small
	for i := 0; i < b.N; i++ {
		for j := 0; j < 10000; j++ {
			s = pool.Get().(*Small)
			s.a = 1
			inc(s)
			pool.Put(s)
		}
	}
}

```

benchmark结果 **TODO ``**

```
name           time/op        alloc/op        allocs/op
WithoutPool-8  3.02ms ± 1%    160kB ± 0%      1.05kB ± 1%
WithPool-8     1.36ms ± 6%   1.05kB ± 0%        3.00 ± 0%
```

结果显示：没有使用`sync.Pool`的内存分配和使用了的对比是 `10k vs 3`，可见，效果还是挺明显的。

但是当你使用`sync.Pool`时，会有大量的对象分配到堆上，以便复用。但当内存上涨，将会触发垃圾回收GC。我们也可以使用`runtime.GC()`来假装触发了GC行为。如下

```
name           time/op        alloc/op        allocs/op
WithoutPool-8  993ms ± 1%    249kB ± 2%      10.9k ± 0%
WithPool-8     1.03s ± 4%    10.6MB ± 0%     31.0k ± 0%
```

我们可以看到，使用`sync.Pool`的性能更低。内存分配和内存使用数量飙高。让我们往更深的代码瞧瞧。一探究竟。

>sync.Pool 内部工作流程

在pool初始化init函数中注册了个cleanup函数钩子，并标明在GC之前进行清理pool中的对象。

```go
// sync/pool.go
func init() {
   runtime_registerPoolCleanup(poolCleanup)
}

// runtime/mgc.go
func gcStart(trigger gcTrigger) {
   [...]
   // clearpools before we start the GC
   clearpools()
   [...]
}
```

这就能解释的通为什么在触发GC时性能会特别差。并且官方文档已经给我们了警告。https://golang.org/pkg/sync/#Pool



> 任何存储在池子中的对象可能在没有任何通知的情况下被删除

下面是1.12 pool工作流

![](https://tva1.sinaimg.cn/large/007S8ZIlgy1gflul7rk6gj30u00w010b.jpg)

对于每个`sync.Pool`，go生成一个内部`poolLocal`附着在每个P上。这个内部的pool有两个属性：`private` 和 `shared`。

private : 仅为P自己使用，push or pop，所以不需要锁

shared : 任何一个P都可以使用并且需要并发安全（协程安全，加锁）。

可以确定的是，pool不是一个本地缓存，它有可能被任意一个P和M使用。

> >  在1.13版本中，go将优化访问共享变量，将引入解决GC和清理Pool的一个新缓存



> 新型Lock-free池和victim缓存，无锁池和牺牲者缓存

go 1.13 引入[双向链表](https://github.com/golang/go/commit/d5fd2dd6a17a816b7dfd99d4df70a85f1bf0de31#diff-491b0013c82345bf6cfa937bd78b690d)做为分享池并且删除了锁，并优化了共享访问。这是提升缓存效率的基础。下面是1.13 共享访问：

![](https://tva1.sinaimg.cn/large/007S8ZIlly1gfm3mvsy63j30om05ht91.jpg)

有了这个新链式池，每个P可以在其队列开头都有push 和 pop的功能，而共享访问将从尾部pop。队列的头部可以分配原来构体的两倍。并连接到队列的头部。初始大小为8，意味着扩容的大小为16，32 等等。

那么，锁就自然而然的移除掉了。剩余的代码可以依赖于原子操作。

考虑到新的缓存方式，新的策略就十分简单了。现在有两个缓存池：一个活跃的和一个归档的。当一个GC运行时，go将保持一每个池子的引用到池内部的一个新属性，在清理当前池中数据时，要拷贝当前持池到归档池中。

```go

func poolCleanup() {
	// This function is called with the world stopped, at the beginning of a garbage collection.
	// It must not allocate and probably should not call any runtime functions.

	// Because the world is stopped, no pool user can be in a
	// pinned section (in effect, this has all Ps pinned).

	// Drop victim caches from all pools.
	for _, p := range oldPools {
		p.victim = nil
		p.victimSize = 0
	}

	// Move primary cache to victim cache.
	for _, p := range allPools {
		p.victim = p.local
		p.victimSize = p.localSize
		p.local = nil
		p.localSize = 0
	}

	// The pools with non-empty primary caches now have non-empty
	// victim caches and no pools have primary caches.
	oldPools, allPools = allPools, nil
}
```

有了这个策略，应用将有不仅一次的GC生命周期去创建/收集新的item作为backup。在这个工作流程下，请求共享池之后 还要请求 牺牲者 缓存 。



// TODO 
// empty
