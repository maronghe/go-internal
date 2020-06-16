> Leetcode 206 链表翻转

Smart clean code 

```go
/**
 * Definition for singly-linked list.
 * type ListNode struct {
 *     Val int
 *     Next *ListNode
 * }
 */
func reverseList(head *ListNode) *ListNode {
    var p *ListNode
    for head != nil {
        head.Next, head, p = p, head.Next, head // ??? 某些人可能困惑在这里
    }
    return p
}
```

`尼古拉斯·永强` leave a comment : 

​	I learn it from google that the assignment is from the left to the right. So the point I do not get is that when `head.Next=prev` at the initial time, it equals to `nil`, and then why using `head=head.Next` could make it works? For me, I think `head.Next` equals `nil` at that moment. Correct me if I was wrong.



我们使用 [dlv](https://github.com/go-delve/delve)，go的一个 `debug` 命令行工具来观看下

```shell
# logan at macmadeMacBook-Pro.local in ~/go/src/leetcode/tmp
→ dlv debug main.go                                                                                                               master ✖︎ [9:44:25]
Type 'help' for list of commands.
(dlv) b main.main
Breakpoint 1 set at 0x10c2398 for main.main() ./main.go:5
(dlv) c
> main.main() ./main.go:5 (hits goroutine(1):1 total:1) (PC: 0x10c2398)
     1:	package main
     2:	
     3:	import "fmt"
     4:	
=>   5:	func main() {
     6:		a, b, c, d := 1, 2, 3, 4
     7:		b, c, d, a = a, b, c, d
     8:		fmt.Println(a, b, c, d)
     9:	}
(dlv) disassemble
TEXT main.main(SB) /Users/logan/go/src/leetcode/tmp/main.go
	main.go:5		0x10c2380	65488b0c2530000000		mov rcx, qword ptr gs:[0x30]
	main.go:5		0x10c2389	488d442490			lea rax, ptr [rsp-0x70]
	main.go:5		0x10c238e	483b4110			cmp rax, qword ptr [rcx+0x10]
	main.go:5		0x10c2392	0f8620020000			jbe 0x10c25b8
=>	main.go:5		0x10c2398*	4881ecf0000000			sub rsp, 0xf0
	main.go:5		0x10c239f	4889ac24e8000000		mov qword ptr [rsp+0xe8], rbp
	main.go:5		0x10c23a7	488dac24e8000000		lea rbp, ptr [rsp+0xe8]
	main.go:6		0x10c23af	48c744244801000000		mov qword ptr [rsp+0x48], 0x1 // a := 1
	main.go:6		0x10c23b8	48c744244002000000		mov qword ptr [rsp+0x40], 0x2 // b := 2 
	main.go:6		0x10c23c1	48c744243803000000		mov qword ptr [rsp+0x38], 0x3 // c := 3 
	main.go:6		0x10c23ca	48c744243004000000		mov qword ptr [rsp+0x30], 0x4 // d := 4
	main.go:7		0x10c23d3	488b442440			mov rax, qword ptr [rsp+0x40] 
	main.go:7		0x10c23d8	4889442460			mov qword ptr [rsp+0x60], rax // 0x60 := b 相当于临时变量
	main.go:7		0x10c23dd	488b442438			mov rax, qword ptr [rsp+0x38]
	main.go:7		0x10c23e2	4889442458			mov qword ptr [rsp+0x58], rax // 0x58 := c 相当于临时变量
	main.go:7		0x10c23e7	488b442430			mov rax, qword ptr [rsp+0x30]
	main.go:7		0x10c23ec	4889442450			mov qword ptr [rsp+0x50], rax // 0x50 := d 相当于临时变量
	main.go:7		0x10c23f1	488b442448			mov rax, qword ptr [rsp+0x48]
	main.go:7		0x10c23f6	4889442440			mov qword ptr [rsp+0x40], rax // 0x40 := a -> b = a
	main.go:7		0x10c23fb	488b442460			mov rax, qword ptr [rsp+0x60]
	main.go:7		0x10c2400	4889442438			mov qword ptr [rsp+0x38], rax // 0x38 = 0x60 -> c = b
	main.go:7		0x10c2405	488b442458			mov rax, qword ptr [rsp+0x58] 
	main.go:7		0x10c240a	4889442430			mov qword ptr [rsp+0x30], rax // 0x30 = 0x58 -> d = c
	main.go:7		0x10c240f	488b442450			mov rax, qword ptr [rsp+0x50] 
	main.go:7		0x10c2414	4889442448			mov qword ptr [rsp+0x48], rax // 0x48 = 0x50 -> a = d
	main.go:8		0x10c2419	48890424			mov qword ptr [rsp], rax
	main.go:8		0x10c241d	e87e78f4ff			call $runtime.convT64
	main.go:8		0x10c2422	488b442408			mov rax, qword ptr [rsp+0x8]
	main.go:8		0x10c2427	4889442470			mov qword ptr [rsp+0x70], rax // 
	main.go:8		0x10c242c	488b442440			mov rax, qword ptr [rsp+0x40]
	main.go:8		0x10c2431	48890424			mov qword ptr [rsp], rax
	main.go:8		0x10c2435	e86678f4ff			call $runtime.convT64
	main.go:8		0x10c243a	488b442408			mov rax, qword ptr [rsp+0x8]
	main.go:8		0x10c243f	4889442468			mov qword ptr [rsp+0x68], rax
	main.go:8		0x10c2444	488b442438			mov rax, qword ptr [rsp+0x38]
	main.go:8		0x10c2449	48890424			mov qword ptr [rsp], rax
	main.go:8		0x10c244d	e84e78f4ff			call $runtime.convT64
	main.go:8		0x10c2452	488b442408			mov rax, qword ptr [rsp+0x8]
	main.go:8		0x10c2457	4889842488000000		mov qword ptr [rsp+0x88], rax
	main.go:8		0x10c245f	488b442430			mov rax, qword ptr [rsp+0x30]
	main.go:8		0x10c2464	48890424			mov qword ptr [rsp], rax
	main.go:8		0x10c2468	e83378f4ff			call $runtime.convT64
	main.go:8		0x10c246d	488b442408			mov rax, qword ptr [rsp+0x8]
	main.go:8		0x10c2472	4889842480000000		mov qword ptr [rsp+0x80], rax
	main.go:8		0x10c247a	0f57c0				xorps xmm0, xmm0
	main.go:8		0x10c247d	0f118424a8000000		movups xmmword ptr [rsp+0xa8], xmm0
	main.go:8		0x10c2485	0f57c0				xorps xmm0, xmm0
	main.go:8		0x10c2488	0f118424b8000000		movups xmmword ptr [rsp+0xb8], xmm0
	main.go:8		0x10c2490	0f57c0				xorps xmm0, xmm0
	main.go:8		0x10c2493	0f118424c8000000		movups xmmword ptr [rsp+0xc8], xmm0
	main.go:8		0x10c249b	0f57c0				xorps xmm0, xmm0
	main.go:8		0x10c249e	0f118424d8000000		movups xmmword ptr [rsp+0xd8], xmm0
	main.go:8		0x10c24a6	488d8424a8000000		lea rax, ptr [rsp+0xa8]
	main.go:8		0x10c24ae	4889442478			mov qword ptr [rsp+0x78], rax
	main.go:8		0x10c24b3	8400				test byte ptr [rax], al
	main.go:8		0x10c24b5	488b4c2470			mov rcx, qword ptr [rsp+0x70]
	main.go:8		0x10c24ba	488d15dfdd0000			lea rdx, ptr [rip+0xdddf]
	main.go:8		0x10c24c1	48899424a8000000		mov qword ptr [rsp+0xa8], rdx
	main.go:8		0x10c24c9	48898c24b0000000		mov qword ptr [rsp+0xb0], rcx
	main.go:8		0x10c24d1	8400				test byte ptr [rax], al
	main.go:8		0x10c24d3	488b442468			mov rax, qword ptr [rsp+0x68]
	main.go:8		0x10c24d8	488d0dc1dd0000			lea rcx, ptr [rip+0xddc1]
	main.go:8		0x10c24df	48898c24b8000000		mov qword ptr [rsp+0xb8], rcx
	main.go:8		0x10c24e7	48898424c0000000		mov qword ptr [rsp+0xc0], rax
	main.go:8		0x10c24ef	488b442478			mov rax, qword ptr [rsp+0x78]
	main.go:8		0x10c24f4	8400				test byte ptr [rax], al
	main.go:8		0x10c24f6	488b8c2488000000		mov rcx, qword ptr [rsp+0x88]
	main.go:8		0x10c24fe	488d159bdd0000			lea rdx, ptr [rip+0xdd9b]
	main.go:8		0x10c2505	48895020			mov qword ptr [rax+0x20], rdx
	main.go:8		0x10c2509	488d7828			lea rdi, ptr [rax+0x28]
	main.go:8		0x10c250d	833d6c37100000			cmp dword ptr [runtime.writeBarrier], 0x0
	main.go:8		0x10c2514	7405				jz 0x10c251b
	main.go:8		0x10c2516	e990000000			jmp 0x10c25ab
	main.go:8		0x10c251b	48894828			mov qword ptr [rax+0x28], rcx
	main.go:8		0x10c251f	eb00				jmp 0x10c2521
	main.go:8		0x10c2521	488b4c2478			mov rcx, qword ptr [rsp+0x78]
	main.go:8		0x10c2526	8401				test byte ptr [rcx], al
	main.go:8		0x10c2528	488b842480000000		mov rax, qword ptr [rsp+0x80]
	main.go:8		0x10c2530	488d1569dd0000			lea rdx, ptr [rip+0xdd69]
	main.go:8		0x10c2537	48895130			mov qword ptr [rcx+0x30], rdx
	main.go:8		0x10c253b	488d7938			lea rdi, ptr [rcx+0x38]
	main.go:8		0x10c253f	833d3a37100000			cmp dword ptr [runtime.writeBarrier], 0x0
	main.go:8		0x10c2546	7402				jz 0x10c254a
	main.go:8		0x10c2548	eb5a				jmp 0x10c25a4
	main.go:8		0x10c254a	48894138			mov qword ptr [rcx+0x38], rax
	main.go:8		0x10c254e	eb00				jmp 0x10c2550
	main.go:8		0x10c2550	488b442478			mov rax, qword ptr [rsp+0x78]
	main.go:8		0x10c2555	8400				test byte ptr [rax], al
	main.go:8		0x10c2557	eb00				jmp 0x10c2559
	main.go:8		0x10c2559	4889842490000000		mov qword ptr [rsp+0x90], rax
	main.go:8		0x10c2561	48c784249800000004000000	mov qword ptr [rsp+0x98], 0x4
	main.go:8		0x10c256d	48c78424a000000004000000	mov qword ptr [rsp+0xa0], 0x4
	main.go:8		0x10c2579	48890424			mov qword ptr [rsp], rax
	main.go:8		0x10c257d	48c744240804000000		mov qword ptr [rsp+0x8], 0x4
	main.go:8		0x10c2586	48c744241004000000		mov qword ptr [rsp+0x10], 0x4
	main.go:8		0x10c258f	e87c9fffff			call $fmt.Println
	main.go:9		0x10c2594	488bac24e8000000		mov rbp, qword ptr [rsp+0xe8]
	main.go:9		0x10c259c	4881c4f0000000			add rsp, 0xf0
	main.go:9		0x10c25a3	c3				ret
	main.go:8		0x10c25a4	e89714faff			call $runtime.gcWriteBarrier
	main.go:8		0x10c25a9	eba5				jmp 0x10c2550
	main.go:8		0x10c25ab	4889c8				mov rax, rcx
	main.go:8		0x10c25ae	e88d14faff			call $runtime.gcWriteBarrier
	main.go:8		0x10c25b3	e969ffffff			jmp 0x10c2521
	main.go:5		0x10c25b8	e863f6f9ff			call $runtime.morestack_noctxt
	<autogenerated>:1	0x10c25bd	e9befdffff			jmp $main.main
```



​	上述过程中：`main.go:7 ` 行显示，b c d tmps 三个变量创建了三个临时变量，之后，把a赋值给b，b tmp 赋值给c ， c tmp 赋值给 d， d tmp 赋值给 a。给我个人理解来说，在java中，变量交换使用了临时变量。只不过是显示处理（由程序员处理），而go中的变量交换。只不过是编译器给我们做了相关的处理。go语言本身就是以简单为目标。编译器能做的事，就做了。程序员就不用考虑创建临时变量来处理交换变量的值的问题了。

​	所以LeetCode的206题，上面的写法也相当于。LeetCode内存占用一致。

```go
/**
 * Definition for singly-linked list.
 * type ListNode struct {
 *     Val int
 *     Next *ListNode
 * }
 */
func reverseList(head *ListNode) *ListNode {
    var p *ListNode
    for head != nil {
      t := head.Next
      head.Next = p 
      p = head
      head = t
    }
    return p
}
```



![](https://tva1.sinaimg.cn/large/007S8ZIlly1gfu21duq5xj313q0bg0uk.jpg)



![](https://tva1.sinaimg.cn/large/007S8ZIlly1gfu2212i6nj31qm0nwtne.jpg)





## mov rax, qword ptr [rsp+0x40] 

​	mov : This is the [opcode](https://en.wikipedia.org/wiki/Opcode) part of the instruction. It describes the base operation the CPU is required to perform. `mov` is an opcode instructing a CPU to copy data from the second [operand](https://en.wikipedia.org/wiki/Operand) to the first operand. The first operand on the `mov` instruction is a target operand, and the second is the source.

​	也就是说mov是个cpu指令，让第二个参数src，拷贝到第一个参数dst上。

base : https://reverseengineering.stackexchange.com/questions/10746/what-does-mov-qword-ptr-dsrax18-r8-mean









