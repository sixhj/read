
Do not communicate by sharing memory;   
instead, share memory by communicating.  

不要以共享内存的方式来通信，相反，要通过通信来共享内存。




- M 代表着一个内核线程，也可以称为一个工作线程。goroutine就是跑在M之上的


- P 代表着(Processor)处理器 它的主要用途就是用来执行goroutine的，一个P代表执行一个Go代码片段的基础（可以理解为上下文环境），所以它也维护了一个可运行的goroutine队列，和自由的goroutine队列，里面存储了所有需要它来执行的goroutine。


- G 代表着goroutine 实际的数据结构(就是你封装的那个方法)，并维护者goroutine 需要的栈、程序计数器以及它所在的M等信息。


- Seched 代表着一个调度器 它维护有存储空闲的M队列和空闲的P队列，可运行的G队列，自由的G队列以及调度器的一些状态信息等


[1.14 抢占式调度](http://xiaorui.cc/archives/6535)


```golang
func initsig(preinit bool) {
	// 预初始化
	if !preinit { 
		signalsOK = true
	} 
	//遍历信号数组
	for i := uint32(0); i < _NSIG; i++ {
		t := &sigtable[i]
		//略过信号：SIGKILL、SIGSTOP、SIGTSTP、SIGCONT、SIGTTIN、SIGTTOU
		if t.flags == 0 || t.flags&_SigDefault != 0 {
			continue
		} 
		...  
		setsig(i, funcPC(sighandler))
	}
}
 


```