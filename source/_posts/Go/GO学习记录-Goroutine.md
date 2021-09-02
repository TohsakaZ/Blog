# GO协程-Goroutine理解

**套接字使用的注意事项**

在这里先注意下`io.Reader`的接口定义的使用，这个在很多文件输入流、以及连接套接字都实现了这个接口，接口定义如下所示：

```go
// Implementations must not retain p.
type Reader interface {
	Read(p []byte) (n int, err error)
}
```

这里输入参数是一个字节动态数组，由于是传址，因此是直接修改这个参数指向的内容。正因为如此，`Read`函数内部的实现一定不会是对`p`变量进行更改，否则指向的新地址外部调用者根本无法获知，因此Read内部实现一定不会使用Append的方式对字节数组进行添加（扩容会改址）。 所以传入这个数组要事先分配好数组空间。以连接套接字的使用为例。

```go
for {
	//if _, err := io.Copy(os.Stdout, ConSock); err != nil {
	//	log.Fatal(err)
	//}
	buffer := make([]byte, 1024) // reserve size before!!!
	n, err := ConSock.Read(buffer)
	if err != nil {
		fmt.Println(err)
		os.Exit(1)
	}
	fmt.Println(n)
	fmt.Println(string(buffer))
}
```

## Context 的使用

Context通常

