```
7、goroutines和channels
Go语言中的并发程序可以用两种手段来实现：goroutine和channel、多线程共享内存
{
goroutines
{
goroutine和channel，其支持“顺序通信进程”(communicating sequential processes)或被简称为CSP。
CSP是一种现代的并发编程模型，在这种编程模型中值会在不同的运行实例(goroutine)中传递，尽管大多数情况下仍然是被限制在单一实例中。

在Go语言中，每一个并发的执行单元叫作一个goroutine。
当一个程序启动时，其主函数即在一个单独的goroutine中运行，我们叫它main goroutine。新
的goroutine会用go语句来创建。go语句会使其语句中的函数在一个新创建的goroutine中运行。而go语句本身会迅速地完成。
	
}


channels
{
ch := make(chan int) //无缓冲 ch has type 'chan int'	
ch = make(chan int, 3)//带三个缓冲
ch <- x // a send statement
x = <-ch // a receive expression in an assignment statement
<-ch // a receive statement; result is discarded
close(ch)

goroutine是Go语音程序的并发体的话，channels它们之间的通信机制
channel都有一个特殊的类型，也就是channels可发送数据的类型。

和map类似，channel也一个对应make创建的底层数据结构的引用。
两个相同类型的channel可以使用==运算符比较。
一个channel有发送和接受两个主要操作，都是通信行为。

Channel还支持close操作，用于关闭channel，
发送操作会panic异常，但是依然可以接受到之前已经成功发送的数据


无缓存Channels的发送操作将导致发送者goroutine阻塞，直到另一个goroutine在相同的Channels上执行接收操作；
当发送的值通过Channels成功传输之后，两个goroutine可以继续执行后面的语句。
反之，如果接收操作先发生，那么接收者goroutine也将阻塞，直到有另一个goroutine在相同的Channels上执行发送操作。
基于无缓存Channels的发送和接收操作将导致两个goroutine做一次同步操作。
因为这个原因，无缓存Channels有时候也被称为同步Channels。当通过一个无缓存Channels发送数据时，接收者收到数据发生在唤醒发送者goroutine之前（译注：happens before，这是Go语言
并发内存模型的一个关键术语！） 。


}
并发的循环
并发的web爬虫
基于select的多路复用
并发的字典遍历
并发的退出
聊天服务	
}

8、基于共享变量的并发
多goroutine之间的共享变量
{
竞争条件
sync.nutex互斥锁
	import "sync"
	var (
		mu sync.Mutex // guards balance
		balance int
	) 
	func Deposit(amount int) {
		mu.Lock()
		balance = balance + amount
		mu.Unlock()
	} 
	func Balance() int {
		mu.Lock()
		b := balance
		mu.Unlock()
		return b
	}
sync.rwmutext读写锁
	var mu sync.RWMutex
	var balance int
	func Balance() int {
		mu.RLock() // readers lock
		defer mu.RUnlock()
		return balance
	}
内存同步
sync.once初始化
竞争条件检测
并发的非阻塞缓存
goroutines和线程	
{
每一个OS线程都有一个固定大小的内存块(一般会是2MB)来做栈，存储当前正在被调用或挂起(指在调用其它函数时)的函数的内部变量。
，同时创建成百上千个gorutine是非常普遍的
一个goroutine会以一般2KB的栈开始其生命周期，，和操作系统线程一样，会保存其活跃或挂起的函数调用的本地变量，
栈的大小会根据需要动态地伸缩，栈的最大值有1GB，尽管一般情况下，大多goroutine都不需要这么大的栈。

OS线程会被操作系统内核调度。
Go的运行时包含了其自己的调度器，但是这个调度器只关注单独的Go程序中的goroutine
所以重新调度一个goroutine比调度一个线程代价要低得多。	
}


}

9、包和工具
{
导入路径
包声明
导入声明
包的匿名导入
包和命名
工具
}

10、测试

11、反射

12、底层编程

13、go原理分析
```



