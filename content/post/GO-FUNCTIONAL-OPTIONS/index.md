---
title: "GO FUNCTIONAL OPTIONS"
description: Functional Options编程模式.
date: 2021-09-03 12:36:16+08:00
categories: 
- Golang
tags:
- Go编程模式
- Golang
- 函数选项模式
---

## 配置选项问题

在我们编程中 我们会经常对一个对象(或是业务实体)进行相关的配置 比如下面这个业务实体

```go
type Server struct {
    Addr     string
    Port     string
    Protocol string
    Timeout  time.Duration
    MaxConns int
    TLS      *tls.Config
}
```

在这个`Server`对象中 我们可以看到：

- 要有监听的地址`Addr`和端口`Port` 这两个选项是必须配置的(当然可以设置默认值 这里举例我们认为没有默认值且不能为空)
- 然后 还有协议`Protocol` `Timeout`和`MaxConns`字段 这几个字段是不能为空的 但是可以有默认值
  - 比如: 协议是`tcp`超时`30s`和最大连接数`1024`个
- 还有一个`TLS`安全链接 需要配置相关的证书和私钥 这个可以为空

针对上述这样的配置 我们需要有多种不同的创建配置`Server`的函数签名 如下：

```go
func NewDefaultServer(addr string, port int) (*Server, error) {
    return &Server{addr, port, "tcp", 30 * time.Second, 100, nil}, nil
}

func NewTLSServer(addr string, port int, tls *tls.Config) (*Server, error) {
    return &Server{addr, port, "tcp", 30 * time.Second, 100, tls}, nil
}

func NewServerWithTimeout(addr string, port int, timeout time.Duration) (*Server, error) {
    return &Server{addr, port, "tcp", timeout, 100, nil}, nil
}

func NewTLSServerWithMaxConnAndTimeout(addr string, port, maxconns int, timeout time.Duration, tls *tls.Config) (*Server, error) {
    return &Server{addr, port, "tcp", 30 * time.Second, maxconns, tls}, nil
}
```

因为Go语言不支持重载函数 所以 你得用不同得函数名来应对不同的配置选项
上面方案的缺点：

- 创建太多的NewServer函数
- 代码冗余
- 可扩展性差

## 配置对象方案(分离可选项)

要解决上面的问题 最常见的方式是使用一个配置对象 如下所示：

```go
type Config struct {
    Protocol string
    Timeout  time.Duration
    Maxconns int
    TLS      *tls.Config
}
```

我们把那些非必须的选项都分离到一个结构体里 于是新的`Server`对象如下：

```go
type Server struct {
    Addr string
    Port int
    Cfg *Config
}
```

于是 我们只需要一个`NewServer()`函数即可 在使用前需要构造`Config`对象

```go
func NewServer(addr string, port int, cfg *Config) (*Server, error) {
    //...
}

// Using the default configuratrion
s, _ := NewServer("localhost", 9000, nil)

cfg := ServerConfig{Protocol:"tcp", Timeout: 60*time.Duration}
s, _ := NewServer("locahost", 9000, &cfg)
```

上面这种`分离可选项`的方式 `Config`对象并不是必须的 所以 你需要判断其是否是`nil`或者是Empty`Config{}`

## Builder模式

如果你是一个Java程序员 熟悉设计模式的一定会很自然的使用上Builder模式
比如如下代码:

```java
User user = new User.Builder()
    .name("ilolicon")
    .email("97431110@qq.com")
    .role("admin")
    .build();
```

仿照上面这个模式 我们可以把上面的代码改写成如下代码(注：下面的代码没有考虑出错处理)

```go
// 使用一个builder类来做包装
type ServerBuilder struct {
    Server
}

func (sb *ServerBuilder) Create(addr string, port int) *ServerBuilder {
    sb.Server.Addr = addr
    sb.Server.Port = port
    // 其他代码设置其他成员的模式值
    return sb
}

func (sb *ServerBuilder) WithProtocol(protocol string) *ServerBuilder {
    sb.Server.Protocol = protocol
    return sb
}

func (sb *ServerBuilder) WithMaxConns(maxconns int) *ServerBuilder {
    sb.MaxConns = maxconns
    return sb
}

func (sb *ServerBuilder) WithTimeout(timeout time.Duration) *ServerBuilder {
    sb.Timeout = timeout
    return sb
}

func (sb *ServerBuilder) WithTLS(tls *tls.Config) *ServerBuilder {
    sb.Server.TLS = tls
    return sb
}

func (sb *ServerBuilder) Build() Server {
    return sb.Server
}
```

于是 我们可以这样去使用：

```go
sb := ServerBuilder{}
server := sb.Create("127.0.0.1", 8080).
    WithProtocol("udp").
    WithMaxConns(1024).
    WithTimeout(30 * time.Second).
    Build()
```

上面这样的方式也很清楚 不需要额外的`Config`对象 使用链式的函数调用的方式来构造一个对象 只需要多增加一个Builder类

但是这个Builder类似乎有点多余 我们似乎可以直接在`Server`上进行这样的Builder构造 事实也是如此 但是在处理错误的时候可能就有点麻烦(需要为`Server`结构增加一个error成员 破坏了`Server`结构体的"纯洁") 不如一个包装类更好一些

如果我们想省掉这个包装的结构体 那就就轮到`FUNCTIONAL OPTIONS` 函数式编程

## Functional Options

函数选项模式的实现 主要有两种

- 基于闭包的实现
- 基于接口的实现
  
### 基于闭包实现

1.定义一个函数类型Option

```go
type Option func(*Server)
```

2.闭包的方式定义如下一组函数

```go
func WithProtocol(proto string) Option {
    return func(s *Server) {
        s.Protocol = proto
    }
}

func WithTimeout(timeout time.Duration) Option {
    return func(s *Server) {
        s.Timeout = timeout
    }
}

func WithMaxConns(maxConns int) Option {
    return func(s *Server) {
        s.MaxConns = maxConns
    }
}

func WithTLS(tls *tls.Config) Option {
    return func(s *Server) {
        s.TLS = tls
    }
}
```

以`WithMaxConns()`函数为例 传入一个参数maxConns 返回一个函数 这个函数会将参数maxConns赋值给`Server`对应的MaxConns字段

- 当我们调用其中一个函数用`WithMaxConns(30)`时
- 其返回值时一个 `func(s *Server) {s.MaxConns = 30}`的函数

3.定义`NewServer()`函数

函数参数是必填字段和可选字段(可变参数类型)

```go
// NewServer先填上必须的字段 然后是可选字段
func NewServer(addr string, port int, opts ...Option) *Server {
    // 创建Server对象并填写可选项的默认值
    s := &Server{
        Addr:     addr,
        Port:     port,
        Protocol: "tcp",
        Timeout:  30 * time.Second,
        MaxConns: 1024,
        TLS:      nil,
    }
    // 对选项列表中的每项用for-loop来设置Server对象属性
    for _, option := range opts {
        option(s)
    }

    return s
}
```

- 首先 `NewServer()`函数当然opts是可变参数 可以传递上面的`WithXXX()`一个或多个函数
- 然后 创建`Server`对象 并填写可选项的默认值 如Timeout超时时间为30s
- 最后 for-loop循环opts可变参数 执行函数option去设置s对象中的各个属性

4.测试`NewServer()`函数

```go
func main() {
    s := NewServer("127.0.0.1", 8080)
    fmt.Printf("Default Server: %v\n", s)

    s = NewServer("127.0.0.1", 8080, WithProtocol("udp"))
    fmt.Printf("ServerWithProto: %v\n", s)

    s = NewServer("127.0.0.1", 8080, WithProtocol("udp"), WithMaxConns(5000), WithTimeout(5*time.Second))
    fmt.Printf("ServerWithAll: %v\n", s)
}
```

```bash
$ go run main.go
Default Server : &{127.0.0.1 8080 tcp 30s 1024 <nil>}
ServerWithProto: &{127.0.0.1 8080 udp 30s 1024 <nil>}
ServerWithAll  : &{127.0.0.1 8080 udp 5s 5000 <nil>}
```

### 基于接口实现

1.定义接口

```go
// Server结构体
type Server struct {
    Addr     string        // 必填dd
    Port     int           // 必填
    Protocol string        // 非必填
    Timeout  time.Duration // 非必填
    MaxConns int           // 非必填
    TLS      *tls.Config   // 非必填 可以为空
}

// Option接口 定义需要实现的apply方法
type Option interface {
    apply(*Server)
}
```

Option接口定义`apply(server *Server)`方法 然后可选项需要实现接口

2.可选项实现接口

每个可选项都需要实现接口

```go
// ProtoOption 实现Option接口
type ProtoOption string

func (p ProtoOption) apply(s *Server) {
    s.Protocol = string(p)
}

func WithProtocol(proto string) Option {
    return ProtoOption(proto)
}

// TimeoutOption 实现Option接口
type TimeoutOption time.Duration

func (t TimeoutOption) apply(s *Server) {
    s.Timeout = time.Duration(t)
}

func WithTimeout(timeout time.Duration) Option {
    return TimeoutOption(timeout)
}

// MaxConnsOption 实现Option接口
type MaxConnsOption int

func (m MaxConnsOption) apply(s *Server) {
    s.MaxConns = int(m)
}

func WithMaxConns(maxConns int) Option {
    return MaxConnsOption(maxConns)
}
```

以可选项protocol的ProtoOption为例：
自定义类型ProtoOption实现Option接口的`apply()`方法

3.封装函数将可选项包装为接口类型

提供`WithProtocol()`函数将`string`类型的proto转换为`ProtoOption`类型
方便以Option接口的形式传递给函数使用

4.定义`NewServer()`函数

```go
func NewServer(addr string, port int, opts ...Option) *Server {
    // 创建Server 并填写可选项的默认值
    s := &Server{
        Addr:     addr,
        Port:     port,
        Protocol: "udp",
        Timeout:  30 * time.Second,
        MaxConns: 1024,
        TLS:      nil,
    }

    for _, opt := range opts {
        opt.apply(s)
    }

    return s
}
```

for-loop循环opts可变参数 执行接口中的apply方法去设置s对象中的各个属性
注意：s一定要传递**指针** 否则无法修改对象属性

```go
func main() {
    s := NewServer("127.0.0.1", 8080)
    fmt.Printf("Default Server: %v\n", s)

    s = NewServer("127.0.0.1", 9999, WithMaxConns(10240), WithTimeout(20*time.Second), WithProtocol("tcp"))
    fmt.Printf("ALL Server: %v\n", s)
}
```

```go
Default Server: &{127.0.0.1 8080 udp 30s 1024 <nil>}
ALL Server    : &{127.0.0.1 9999 tcp 20s 10240 <nil>}
```

## 总结

三种解决方法：

- 配置对象(分离可选项)
- Builder模式
- 函数选项模式(Functional Options) **使用较多**

函数选项模式 优点：

- 可读性强 将配置都转化为对应的函数项Option
- 扩展性好 新增参数只需要多增加一个方法

## 参考文档

[Self-referential functions and the design of options](https://commandcenter.blogspot.com/2014/01/self-referential-functions-and-design.html)

[GO 编程模式：FUNCTIONAL OPTIONS](https://coolshell.cn/articles/21146.html)

[深入浅出 Golang函数选项编程模式](https://zhuanlan.zhihu.com/p/472113355?utm_source=wechat_session&utm_medium=social&utm_oi=27697422008320&utm_campaign=shareopn&s_r=0)
