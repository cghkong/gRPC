# RPC
'''
RPC是指远程过程调用，一个程序在不同的地址空间执行(往往是通过共享网络跨越不同计算机),其编码就像本地过程调用一样。简单的来说，RPC使得程序就像调用本地函数一样调用远程函数。
'''

## 简单用法
1. 注册RPC
```
type Master struct {
	mu          sync.Mutex
	mapTasks    []Task
	reduceTasks []Task
	nMap        int
	nReduce     int
}

func (m *Master) server() {
	rpc.Register(m)
	rpc.HandleHTTP()
  
	listener, err := net.Listen("tcp", ":1234")
  /**
  进程间通信
	sockname := masterSock()
	os.Remove(sockname)
	listener, err := net.Listen("unix", sockname)
  */
	if err != nil {
		log.Fatal("listen error:", err)
	}
	go http.Serve(listener, nil)
}
```
2. 客户端调用
```
func call(rpcname string, args interface{}, reply interface{}) bool {
	client, err := rpc.DialHTTP("tcp", "127.0.0.1"+":1234")
  /**
  Unix socket
	sockname := masterSock()
	c, err := rpc.DialHTTP("unix", sockname)
	*/
  if err != nil {
		log.Fatal("dialing:", err)
	}
	defer client.Close()

	err = client.Call(rpcname, args, reply)
	if err != nil {
		fmt.Println(err)
    return false
	}
  
	return true
}
```

**注意**
1. 注册的对象的导出方法是将可以远程访问的

2. 服务器可以注册多个不同类型的对象（服务），但不能注册多个相同类型的对象

可远程访问的方法满足以下条件
1. 方法的类型是可导出的
2. 方法是可导出的
3. 方法有两个参数并且是可导出的，第二个参数是指针
4. 方法放回错误类型
   


