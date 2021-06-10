接口也是 Go 语言中的一种类型，它能够出现在变量的定义、函数的入参和返回值中并对它们进行约束，不过 Go 语言中有两种略微不同的接口，

 
- runtime.iface 表示第一种接口,带有一组方法的接口，  
- runtime.eface 表示第二种不包含任何方法的接口   

```golang
type eface struct { // 16 字节
	_type *_type
	data  unsafe.Pointer
}

type _type struct {
	size       uintptr //类型大小
	ptrdata    uintptr //含有所有指针类型前缀大小
	hash       uint32  //类型hash值；避免在哈希表中计算
	tflag      tflag   //额外类型信息标志
	align      uint8   //该类型变量对齐方式
	fieldalign uint8   //该类型结构字段对齐方式   
	kind       uint8   //类型编号
	alg        *typeAlg //算法表 存储hash和equal两个操作。map key便使用key的_type.alg.hash(k)获取hash值
	// gcdata stores the GC type data for the garbage collector.
	// If the KindGCProg bit is set in kind, gcdata is a GC program.
	// Otherwise it is a ptrmask bitmap. See mbitmap.go for details.
	gcdata    *byte   //gc数据
	str       nameOff        // 类型名字的偏移
	ptrToThis typeOff
}
```


```golang
type iface struct { // 16 字节
	tab  *itab
	data unsafe.Pointer
}

// 非空接口的类型信息
type itab struct {
    //inter 和 _type 确定唯一的 _type类型
    inter  *interfacetype       // 接口自身定义的类型信息，用于定位到具体interface类型
    _type  *_type               // 接口实际指向值的类型信息-实际对象类型，用于定义具体interface类型
    hash int32                  //_type.hash的拷贝，用于快速查询和判断目标类型和接口中类型是一致
    _     [4]byte
    fun  [1]uintptr             //动态数组，接口方法实现列表(方法集)，即函数地址列表，按字典序排序
                                //如果数组中的内容为空表示 _type 没有实现 inter 接口
                    
}

// 非空接口类型，接口定义，包路径等。
type interfacetype struct {
   typ     _type
   pkgpath name
   mhdr    []imethod      // 接口方法声明列表，按字典序排序
}

// 接口的方法声明,一种函数声明的抽象
// 比如：func Print() error
type imethod struct {
   name nameOff          // 方法名
   ityp typeOff                // 描述方法参数返回值等细节
}

```