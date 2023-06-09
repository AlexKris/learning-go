# Go学习

## Go优点

1. 简单，作为静态编程语言，入门门槛降低到和动态语言一个水平线
2. 生产力和性能的最佳时间
3. 挣得多

## Go设计哲学

1. 简单
2. 显式
3. 组合
4. 并发
5. 面向工程

### 简单

1. 没有传统的面向对象的类、构造函数与继承
2. 没有结构化的异常处理
3. 没有本属于函数编程范式的语法元素

#### 语法层面呈现

- 只有25个关键字
- 内置垃圾收集
- 首字母大写决定可见性
- 变量初始值为类型零值
- 内置数组边界检查
- 内置并发支持
- 原生提供完善的工具链

### 显式

- C语言隐式代码的行为特征
    ```C++
    # include <stdio.h>
    int main() {
        short int a = 5;
        int b = 8;
        long c = 0;
        c = a + b;
        printf("%ld\n", c)
    }
    ```
- 转化成Go代码
    ```go
    package main
    import "fmt"
    func main() {
        var a int16 = 5
        var b int = 8
        var c int64
        c = a + b
        fmt.Printf("%d\n", c)
    }
    ```
    - 该段代码将会得到错误`invalid operation: a + b (mismatched types int16 and int)`
    - 对变量 a 和 b 进行显式转型
    ```go
    c = int64(a) + int64(b)
    fmt.Printf("%d\n", c)
    ```

- Go语言采用了**显式的基于值比较的错误处理方案**，函数/方法中的错误都会通过return语句显式地返回，并且通常调用者不能忽略对返回的错误的处理

### 组合

- 在Go中找不到经典的面向对象语法元素、类型体系和集成机制
- Go提供了正交的语法元素
    - Go语言无类型层次体系，各类型之间事相互独立的，没有子类型的概念
    - 每个类型都可以有自己的方法集合，类型定义与方法实现是正交独立的
    - 实现某个接口时，无需像Java那样采用特定关键字修饰
    - 包之间事相对独立的，没有子包的概念
- Go语言使用**类型嵌入**（Type Embedding）实现组合，类似继承
    - **垂直组合**：通过类型嵌入，快速让一个新类型复用其他类型已经实现的能力，实现功能的垂直扩展
    ```go
    // 在 poolLocal 这个结构体类型中嵌入了类型 Mutex
    // 使 poolLocal 这个类型具有了互斥同步的能力
    // 可以通过 poolLocal 类型的变量直接调用 Mutex 类型的 Lock/Unlock 方法
    type poolLocal struct {
        private interface{}
        shared []interface{}
        Mutex
        pad [128]byte
    }

    // 标准库通过嵌入接口类型实现接口行为的聚合，组成大接口
    // $GOROOT/src/io/io.go
    type ReadWriter interface {
        Reader
        Writer
    }
    ```
    - **水平组合**：能力委托（Delegate），使用接口类型实现水平组合，Go 语言中的接口只是方法集合，与实现者之间的关系无需通过显式关键字修饰
    ```go
    // 通过接受接口类型参数的普通函数进行组合
    // ReadAll 通过 io.Reader 这个接口，将 io.Reader 的实现与 ReadAll 所在的包低耦合地水平组合在一起了
    // 目的：从任意实现 io.Reader 的数据源读取所有数据

    // $GOROOT/src/io/ioutil/ioutil.go
    func ReadAll(r io.Reader)([]byte, error)
    
    // $GOROOT/src/io/io.go
    func Copy(dst Writer, src Reader)(written int64, err error)
    ```

### 并发

- Go放弃了传统的基于操作系统线程的并发模型，采用了**用户层轻量级线程**，称之为**goroutine**
- Go运行时默认为每个goroutine分配的栈空间仅2KB，goroutine调度的切换不用陷入（trap）操作系统内核层完成，代价很低。
- 一个Go程序可以创建成千上万个并发的goroutine，所有Go代码都在goroutine中执行
- Go在语言层面内置了辅助并发设计的原语：channel和select，通过channel可以传递消息或实现同步，通过select实现多路channel的并发控制
- goroutines各自执行特定的工作，通过channel和select将goroutines组合连接起来

### 面向工程

- 解决问题
    - 程序构建慢
    - 依赖管理失控
    - 代码难于理解
    - 跨语言构建难
- gofmt，统一Go语言的代码风格
