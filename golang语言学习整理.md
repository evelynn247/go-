## golang语言学习整理

### 切片

1、切片是底层数组的引用 改变底层数组的值 切片的值也会改变
2、切片的长度是真实长度，容量是从切的位置开始算到底层数组的最后
3、append追加切片 尽量用原来的切片接受 容易造成内存浪费

### make&&new的区别

1、都是用来申请内存的
2、new很少用，一般是给基本数据类型、结构体(结构体属于值类型)申请内存 返回的是该类型的指针
3、make是用来给slice、map、chan申请内存的，返回的是对应的类型，因为这三种类型是引用类型，没有必要返回指针

### defer return

1、go语言中的return不是原子操作 包括 返回值赋值和真正的返回
2、defer介于二者步骤之间
3、recover一定要和defer一起使用
看下面的例子

```go
func f1() int{
        x:=5
        defer func(){
            x++
        }()
        return x
        //return 分两步
        //1 将x的值赋给返回值 即返回值为5
        //defer将x值修改为6
        //2 返回 返回值 5
    }
    func f2()  (x int){
        defer func(){
            x++
        }()
        return 5
        //return 分两步  此函数的返回值就是x
        //1 将5赋给返回值x
        //defer将x值修改为6
        //2 返回x 为6
    }
```

4、panic 需要等defer 结束后才会向上传递。 出现panic恐慌时候，会先按照defer的后入先出的顺序执行，最后才会执行panic

### 函数&&方法

1、函数内部不能声明有名字的函数,因此需要匿名函数,匿名函数后加括号即立马调用
2、函数和方法的区别是：方法只能特定的类型调用，函数都可以调用

### 闭包

1、闭包是一个函数,这个函数包含了他外部作用域的变量
闭包原理：闭包=函数+外部变量引用
1、函数可以作为返回值
2、函数内部查找引用变量，找不到再往外部找
闭包的应用场景：函数A需要参数函数B类型,闭包的作用就是将别的函数封装成函数B的类型

### 结构体

1、结构体属于值类型，赋值过程属于copy，
2、返回值类型：结构体类型复杂（返回指针类型），结构体简单返回值类型

### 接口

1、实现了接口所有方法的变量可以成为接口类型
2、同一个结构体可以实现多个接口
3、接口可以嵌套
4、空接口相当于任何变量都实现了

### 类型断言：

断言，即 猜。目的是获取变量的类型
-----------包-------------
1、只有main包才能编译成可执行文件
2、go语言编译从main包找到main函数
3、全局声明--init()--main 执行顺序

### 文件操作

读操作:

```go
//方式① 
func openFile(s string){
        open, err := os.Open(s)
        defer open.Close()
        if err !=nil{
            return
        }
        var tmp [128] byte
        for   {
            read, err := open.Read(tmp[:])
            if err!=nil{
                return
            }
            fmt.Printf("读了%d个字节\n",read)
            fmt.Println(string(tmp[:read]))
            if read==0 {
                return
            }
        }
    }
//方式②
    func bufferFile(s string){
        open, err := os.Open(s)
        defer open.Close()
        if err !=nil{
            return
        }

        reader := bufio.NewReader(open)
       for  {
            readString, _ := reader.ReadString('\n')
           fmt.Println(readString)
        }
    }
//方式③
    func ioUtils(s string){
        file, _ := ioutil.ReadFile(s)
        fmt.Println(string(file))
    }
//写操作：
    func writeFile90(filename  string){
        file, _ := os.OpenFile(filename, os.O_APPEND|os.O_CREATE, 0644)
        defer file.Close()
       file.Write([]byte("hello world\n"))
        file.WriteString("完事")
    }
//  同理 已上读方法也同样适用写文件
```

### 并发

1、go +函数名 调用go routine
2、GMP 模型 G:goroutine  M：系统线程的映射  P:P(Processor)是一个抽象的概念，并不是真正的物理CPU。		所以当P有任务时需要创建或者唤醒一个系统线程来执行它队列里的任务。所以P/M需要进行绑定，构成一个执行单元。P决定了同时可以并发任务的数量，可通过GOMAXPROCS限制同时执行用户级任务的操作系统线程。可以通过runtime.GOMAXPROCS进行指定。在Go1.5之后GOMAXPROCS被默认设置可用的核数，而之前则默认为1。
3、channel go-routine之间的通讯 channel是队列类型  即先进先出
    通道操作：

```django
  	var ch1 chan int 声明一个放int的通道
    ch1 =make(chan int,13) 分配内存 带缓冲区的通道 13为通道的容量
    发送：ch1< - 1 将数值1放进通道ch1中
```

```go
// 若没有缓冲区 即 ch1 =make(chan int) 通道中的值必须有roroutine接收 否则程序阻塞
func channel1(){
       ch1 := make(chan int) //无缓冲区
       wg.Add(1)
       go func() {
          defer  wg.Done()
          x :=<- ch1 //取值 取出来就少一个
          fmt.Println(x)
       }()
       ch1<-1 //向通道中发送值
       fmt.Println(1)
       wg.Wait()
    }
    func bufchannel1(){
       ch1 := make(chan int,16)//有缓冲区通道
       ch1 <-10
       s:=<-ch1
       fmt.Println(s)
    }
```

单向通道：确保该通道只能执行某一项操作 例如：传进函数的通道设置成只能取值，那就无法往通道中放值 防止影响功能


-------------------互联网--------------------
物理层：以光纤 无线的形式将电脑连接起来
数据链
路层：从光纤传来的信号为010101，负责解析电信号。每家公司的电信号分组不同，为了统一出现了以太网
           ，以太网规定所有设备必须具有网卡接口 网卡的地址就是mac地址 以太网协议
网络层：mac地址发送数据具有局限性，一般用于局域网中。因此出现ip ip协议
传输层：tcp  ucp 具体传输到某个应用程序，因此出现了端口 tcp协议
应用层：http等协议


socket将物理层、数据链路层、网络层、传输层封装 使得用户不必关注以上四层的原理 只需关注socket即可

### go module

版本管理工具  代码不必放在gopath下

go mod init 生成go.mod文件 里面管理版本信息



