
```golang 

type hchan struct {
	qcount   uint  //Channel 中的元素个数；
	dataqsiz uint //Channel 中的循环队列的长度
	buf      unsafe.Pointer //Channel 的缓冲区数据指针
	elemsize uint16
	closed   uint32
	elemtype *_type
	sendx    uint // Channel 的发送操作处理到的位置
	recvx    uint // Channel 的接收操作处理到的位置
	recvq    waitq
	sendq    waitq

	lock mutex
}

type waitq struct {
	first *sudog  // 表示一个在等待列表中的 Goroutine，
	last  *sudog
}


```