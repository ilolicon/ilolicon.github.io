---
title: "GO DESIGN PATTERNS"
description: Go语言中设计模式的实现细节.
date: 2023-04-21 11:43:42+08:00
categories: 
- Golang
tags:
- Go编程模式
- Golang
- 设计模式
---

<!-- markdownlint-disable MD024 -->

- `设计模式`是软件设计中常见问题的典型解决方案 每个模式就像一张蓝图 你可以通过对其进行定制来解决代码中的特定设计问题
- 模式是针对软件设计中常见问题的解决方案工具箱 它们定义了一种让你的团队能更高效沟通的通用语言
- 不同设计模式在其复杂程度、细节层次以及应用范围等方面各不相同 此外 我们可以根据模式的目的来将它们划分为三个不同的组别
  - `创建型模式` 提供对象的创建机制 增加已有代码的灵活性和复用性
  - `结构型模式` 介绍如何将对象和类组装成较大的结构 并同时保持结构的灵活和高效
  - `行为型模式` 负责对象间的高效沟通和职责委派

## 创建型模式

- 在Go语言中 创建型模式(Creational Patterns)是一类用于处理对象创建的设计模式
- 它们的主要目标是提供一种灵活的方式来创建对象 同时隐藏对象创建的具体细节 从而降低代码的耦合度 并提高代码的可复用性和可维护性
- 比如标准库: `http.NewRequest()` `bytes.NewReader()` `md5.New()`  等等 ...
- 创建型模式的核心思想是将对象的创建与使用分离 使得系统不依赖于具体的对象创建方式 而是依赖于抽象

### 单例模式(Singleton)

- 单例模式(singleton) 保证一个类只有一个实例 并提供一个访问它的全局访问点
- 通常让一个全局变量成为一个对象被访问 但不能防止实例化多个对象
- 常见做法：让类自身负责保证它的唯一实例 这个类可以保证没有其他实例可以被创建 并且可以提供一个访问该实例的方法

#### 适用场景

- 当某个对象只能存在一个实例的情况下使用单例模式 比如：
  - 全局Config配置对象
  - 全局Logger日志对象
  - ...
- 优点：
  - 提供了对唯一实例的受控访问
  - 由于在系统内存中只存在一个对象 因此可以节约系统资源 对于一些需要频繁创建和销毁的对象 单例模式可以提高系统性能

#### 实现方式及解析

##### 饿汉模式

- 在程序加载时就创建对象实例
- Go语言中我们可以放到`init`函数中 保证线程安全
  - 会减慢程序启动速度(`init`启动时会先加载)
  - 如果实例不被调用 则会浪费一段内存空间 特别是该对象比较大的时候

```go
package singleton

var instance *singleton

func init() {
    instance = &singleton{}
}

type singleton struct{}

func GetInstance() *singleton {
    return instance
}

```

##### 懒汉模式

- 在获取对象实例时 如果实例为空则创建
- 避免饿汉模式的内存空间浪费
- 在Go中并发非常容易 容易被不正确使用 导致忽略并发安全

```go
package singleton

var instance *singleton

type singleton struct{}

func GetInstance() *singleton {
    if instance == nil {
        instance = &singleton{}
    }
    return instance
}

```

- 在上述情况下 多个goroutine都可以执行第一个检查
- 并且它们都将创建该singleton类型的实例并相互覆盖 无法保证它将在此处返回哪个实例 并且对该实例的其他进一步操作可能与开发人员的期望不一致
- 如果有代码保留了对该单例实例的引用 则可能存在具有不同状态的该类型的多个实例 从而产生潜在的不同代码行为 这也成为调试过程中的一个噩梦 并且很难发现该错误
- 因为在调试时 由于运行时暂停而没有出现任何错误 这使非并发安全执行的可能性降到了最低 并且很容易隐藏开发人员的问题

##### 双重检查模式(激进加锁)

- 解决懒汉模式的非线程安全 避免资源竞争导致数据不一致
- 潜在问题：
  - 如果实例存在 没有必要加锁
  - 每次请求都加锁(执行过多的锁定) 降低性能/增加瓶颈

```go
package singleton

import "sync"

var (
    instance *singleton
    mutex    sync.Mutex
)

type singleton struct{}

func GetInstance() *singleton {
    mutex.Lock() // 如果实例存在没有必要加锁
    defer mutex.Unlock()

    if instance == nil {
        instance = &singleton{}
    }
    return instance
}

```

- 优化：检查对应是否为空之后再加锁

```go
package singleton

import "sync"

var (
    instance *singleton
    mutex    sync.Mutex
)

type singleton struct{}

func GetInstance() *singleton {
    if instance == nil { // // 不太完美 因为这里不是完全原子的
        mutex.Lock()
        defer mutex.Unlock()

        if instance == nil {
            instance = &singleton{}
        }
    }
    return instance
}

```

- 上面的方式仍然不太完美 因为判断实例是否为空的过程仍然不是完全原子的
- 我们可以使用`sync/atomic`包 原子化加载并设置一个标志 该标志表明我们是否已初始化实例

```go
package singleton

import "sync"
import "sync/atomic"

var (
    initialized uint32
    instance *singleton
    mutex    sync.Mutex
)

type singleton struct{}

func GetInstance() *singleton {
    if atomic.LoadUInt32(&initialized) == 1 {  // 原子操作 
        return instance
    }

    mu.Lock()
    defer mu.Unlock()

    if initialized == 0 {
         instance = &singleton{}
         atomic.StoreUint32(&initialized, 1)
    }

    return instance
}

```

##### Sync.Once

- Go语言单例惯用实现解析
- 上面实现原子操作的写法稍显繁琐
- Go标准库`sync`中的`Once`类型 它能保证某个操作仅且只执行一次
- `sync.Once`源码

```go
// Copyright 2009 The Go Authors. All rights reserved.
// Use of this source code is governed by a BSD-style
// license that can be found in the LICENSE file.

package sync

import (
    "sync/atomic"
)
// Once is an object that will perform exactly one action.
//
// A Once must not be copied after first use.
type Once struct {
    // done indicates whether the action has been performed.
    // It is first in the struct because it is used in the hot path.
    // The hot path is inlined at every call site.
    // Placing done first allows more compact instructions on some architectures (amd64/386),
    // and fewer instructions (to calculate offset) on other architectures.
    done uint32
    m    Mutex
}

// Do calls the function f if and only if Do is being called for the
// first time for this instance of Once. In other words, given
//  var once Once
// if once.Do(f) is called multiple times, only the first call will invoke f,
// even if f has a different value in each invocation. A new instance of
// Once is required for each function to execute.
//
// Do is intended for initialization that must be run exactly once. Since f
// is niladic, it may be necessary to use a function literal to capture the
// arguments to a function to be invoked by Do:
//  config.once.Do(func() { config.init(filename) })
//
// Because no call to Do returns until the one call to f returns, if f causes
// Do to be called, it will deadlock.
//
// If f panics, Do considers it to have returned; future calls of Do return
// without calling f.
//
func (o *Once) Do(f func()) {
    // Note: Here is an incorrect implementation of Do:
    //
    // if atomic.CompareAndSwapUint32(&o.done, 0, 1) {
    //      f()
    //  }
    //
    // Do guarantees that when it returns, f has finished.
    // This implementation would not implement that guarantee:
    // given two simultaneous calls, the winner of the cas would
    // call f, and the second would return immediately, without
    // waiting for the first's call to f to complete.
    // This is why the slow path falls back to a mutex, and why
    // the atomic.StoreUint32 must be delayed until after f returns.

    if atomic.LoadUint32(&o.done) == 0 {
        // Outlined slow-path to allow inlining of the fast-path.
        o.doSlow(f)
    }
}

func (o *Once) doSlow(f func()) {
    o.m.Lock()
    defer o.m.Unlock()
    if o.done == 0 {
        defer atomic.StoreUint32(&o.done, 1)
        f()
    }
}

```

- 通过上面源码发现 我们可以借助这个实现只执行一次某个函数/方法
- once.Do()的用法如下

```go

var once sync.Once

once.Do(func() {
    // 在这里执行安全的初始化
})

```

##### 完整示例

- 使用sync.Once包是安全地实现此目标的首选方式

```go
package singleton

import (
    "sync"
)

var (
    instance *singleton
    once     sync.Once
)

type singleton struct{}

func GetInstance() *singleton {
    once.Do(func() {
        instance = &singleton{}
    })
    return instance
}

```

### 简单工厂模式(Simple Factory)

- 简单工厂并不是一个正式的设计模式 而是一种编程习惯 它通过一个工厂类来封装对象的创建逻辑 客户端只需要传递参数给工厂类 由工厂类决定创建哪种对象

#### 特点

- 只有一个工厂类 负责创建所有产品
- 通道条件判断(如`switch`或`if-else`)来决定创建哪种产品

#### 适用场景

- 产品种类少 且创建逻辑简单的场景

#### 优点

- 简单易用 适合小型项目
  
#### 缺点

- 不符合开闭原则(OCP) 新增产品时需要修改工厂类
  
> [开闭原则](https://zh.wikipedia.org/wiki/%E5%BC%80%E9%97%AD%E5%8E%9F%E5%88%99): 当需求发生变化时 可以通过增加新的代码来扩展系统的功能 而不是修改现有的代码

#### 示例

```go
package main

import "fmt"

type Product interface {
    Use()
}

type ProductA struct{}

func (p *ProductA) Use() {
    fmt.Println("Using Product A")
}

type ProductB struct{}

func (p *ProductB) Use() {
    fmt.Println("Using Product B")
}

func CreateProduct(productType string) Product {
    switch productType {
    case "A":
        return &ProductA{}
    case "B":
        return &ProductB{}
    default:
        return nil
    }
}

func main() {
    productA := CreateProduct("A")
    productA.Use()

    productB := CreateProduct("B")
    productB.Use()
}

```

### 工厂方法模式(Factory Method)

- 上面的简单工厂模式 一个工厂就负责了多个产品的生产
- 工厂方法模式则是定义了一个创建对象的接口 但将具体的创建逻辑延迟到子类中 每个子类负责创建一种具体的产品
  
#### 特点

- 每个产品对应一个工厂类
- 符合`开闭原则` 新增产品时只需增加新的工厂类 无需修改现有代码
- 将对象的实例化推迟到用于创建实例的专用函数
  
#### 适用场景

- 产品种类多 且创建逻辑复杂的场景

#### 优点

- 符合开闭原则 扩展性强
- 每个工厂只负责一种产品的创建 职责单一

#### 缺点

- 类的数量会增加 系统复杂度提高

#### 示例

```go
package main

import "fmt"

type Database interface {
    Connect() string
}

type MySQL struct{}

func (m MySQL) Connect() string {
    return "Connected to MySQL"
}

type PostgreSQL struct{}

func (p PostgreSQL) Connect() string {
    return "Connected to PostgreSQL"
}

// DatabaseFactory工厂接口
type DatabaseFactory interface {
    CreateDatabase() Database
}

type MySQLFactory struct{}

func (m MySQLFactory) CreateDatabase() Database {
    return MySQL{}
}

type PostgreSQLFactory struct{}

func (p PostgreSQLFactory) CreateDatabase() Database {
    return PostgreSQL{}
}

func UseDatabaseFactory(factory DatabaseFactory) {
    db := factory.CreateDatabase()
    fmt.Println(db.Connect())
}

func main() {
    UseDatabaseFactory(&MySQLFactory{})
    UseDatabaseFactory(&PostgreSQLFactory{})
}

```

### 抽象工厂模式(Abstract Factory)

- 抽象工厂模式提供一个创建一系列相关或相互依赖对象的接口 而无需指定它们的具体类 它适用于需要创建一组相关产品的场景

#### 特点

- 每个工厂类可以创建多个相关产品
- 强调**产品族**的概念 例如GUI库中的不同风格组件(Windows风格、Mac风格)

#### 适用场景

- 需要创建一组相关对象的场景

#### 优点

- 可以创建一组相关对象 保证对象之间的兼容性
- 符合开闭原则 扩展性强

#### 缺点

- 类的数量会增加 系统复杂度提高
- 新增产品族或产品等级结构时 需要修改抽象工厂接口及其所有实现类

#### 示例

```go
package main

import "fmt"

// 抽象层：数据库连接接口
type DBConnention interface {
    Connect() string
}

// 抽象层：数据库命令接口
type DBCommand interface {
    Execute(query string) string
}

// 具体产品：MySQL连接
type MySQLConnection struct{}

func (m *MySQLConnection) Connect() string {
    return "Connected to MySQL"
}

// 具体产品：MySQL命令
type MySQLCommand struct{}

func (m *MySQLCommand) Execute(query string) string {
    return fmt.Sprintf("Executed MySQL query: %s", query)
}

// 具体产品：Postgres连接
type PostgresSQLConnection struct{}

func (p *PostgresSQLConnection) Connect() string {
    return "Connected to Postgres"
}

// 具体产品：PostgresSQL命令
type PostgresSQLCommand struct{}

func (p *PostgresSQLCommand) Execute(query string) string {
    return fmt.Sprintf("Executed PostgresSQL query: %s", query)
}

// 抽象工厂接口
type DBFactory interface {
    CreateConnection() DBConnention
    CreateCommand() DBCommand
}

// 具体的MySQL工厂
type MySQLFactory struct{}

func (m *MySQLFactory) CreateConnection() DBConnention {
    return &MySQLConnection{}
}

func (m *MySQLFactory) CreateCommand() DBCommand {
    return &MySQLCommand{}
}

// 具体的PostgresSQL工厂
type PostgresSQLFactory struct{}

func (p *PostgresSQLFactory) CreateConnection() DBConnention {
    return &PostgresSQLConnection{}
}

func (p *PostgresSQLFactory) CreateCommand() DBCommand {
    return &PostgresSQLCommand{}
}

func UseDatabase(factory DBFactory) {
    connection := factory.CreateConnection()
    command := factory.CreateCommand()

    fmt.Println(connection.Connect())
    fmt.Println(command.Execute("SELECT * FROM table"))
}

// 业务逻辑
func main() {
    UseDatabase(&MySQLFactory{})
    UseDatabase(&PostgresSQLFactory{})
}

```

### 建造者模式(Builder)

- 用于分步构建复杂对象
- 建造者模式的核心思想是将一个复杂对象的构建过程与其表示分离 使得同样的构建过程可以创建不同的表示
- 建造者模式特别适用于以下场景
  - 对象的构建过程非常复杂 包含多个步骤
  - 对象的构建过程需要支持不同的配置或表示

#### 示例

```go
package main

import "fmt"

type House struct {
    Walls   string
    Roof    string
    Windows string
    Doors   string
}

func (h *House) Show() {
    fmt.Printf("House with %s walls, %s roof, %s windows, and %s doors\n",
        h.Walls, h.Roof, h.Windows, h.Doors)
}

// 建造者接口
type HouseBuilder interface {
    BuildWalls()
    BuildRoof()
    BuildWindows()
    BuildDoors()
    GetHouse() *House
}

// 具体建造者
type ConcreateHouseBuilder struct {
    house *House
}

// 返回一个空House
func NewConcreateHouseBuilder() *ConcreateHouseBuilder {
    return &ConcreateHouseBuilder{house: &House{}}
}

func (b *ConcreateHouseBuilder) BuildWalls() {
    b.house.Walls = "concrete"
}

func (b *ConcreateHouseBuilder) BuildRoof() {
    b.house.Roof = "tile"
}

func (b *ConcreateHouseBuilder) BuildWindows() {
    b.house.Windows = "glass"
}

func (b *ConcreateHouseBuilder) BuildDoors() {
    b.house.Doors = "wooden"
}

func (b *ConcreateHouseBuilder) GetHouse() *House {
    return b.house
}

// 指挥者
type Director struct {
    builder HouseBuilder
}

func NewDirector(builder HouseBuilder) *Director {
    return &Director{builder: builder}
}

func (d *Director) Construct() {
    d.builder.BuildWalls()
    d.builder.BuildRoof()
    d.builder.BuildWindows()
    d.builder.BuildDoors()
}

// 客户端代码
func main() {
    builder := NewConcreateHouseBuilder() // 常见具体构建者
    director := NewDirector(builder)      // 创建指挥者
    director.Construct()                  // 指挥者构建产品

    house := builder.GetHouse() // 获取最终产品
    house.Show()
}

```

### 原型模式(Prototype)

- 它通过复制现有对象来创建新对象 而不是通过新建类的方式
- 原型模式的核心思想是利用对象的克隆能力 避免重复初始化 特别适用于**创建成本较高**的对象

#### 示例

```go
package main

import "fmt"

// 原型接口
type Prototype interface {
    Clone() Prototype
}

// 具体原型
type ConcretePrototype struct {
    Name string
    Age  int
}

func (p *ConcretePrototype) Clone() Prototype {
    // 创建一个新对象 并复制当前对象的属性
    return &ConcretePrototype{
        Name: p.Name,
        Age:  p.Age,
    }
}

func (p *ConcretePrototype) String() string {
    return fmt.Sprintf("Name: %s, Age: %d", p.Name, p.Age)
}

func main() {
    // 创建原型对象
    prototype := &ConcretePrototype{
        Name: "Tom",
        Age:  18,
    }

    // 克隆原型对象
    clone := prototype.Clone().(*ConcretePrototype)

    // 修改克隆对象属性
    clone.Name = "Jerry"
    clone.Age = 20

    // 输出原型对象和克隆对象
    fmt.Println("Prototype:", prototype)
    fmt.Println("Clone:", clone)
}

```

#### 注意事项

- 使用原型模式 如果有**引用类型** 则需要考虑深拷贝和浅拷贝的问题
- 浅拷贝只复制对象本身而不复制其引用的对象 深拷贝则会递归地复制整个对象图
- 这需要根据需求选择适当的拷贝方式

## 结构型模式

### 外观模式(Facade)

### 适配器模式(Adapter)

### 代理模式(Proxy)

### 组合模式(Composite)

### 享元模式(Flyweight)

### 装饰模式(Decorator)

### 桥模式(Bridge)

## 行为型模式

### 中介者模式(Mediator)

### 观察者模式(Observer)

### 命令模式(Command)

### 迭代器模式(Iterator)

### 模板方法模式(Template Method)

### 策略模式(Strategy)

### 状态模式(State)

### 备忘录模式(Memento)

### 解释器模式(Interpreter)

### 职责链模式(Chain of Responsibility)

### 访问者模式(Visitor)

## 其余常用模式

### 性能分析模式(Profiling Patterns)

#### Timing Functions

- 包装函数并记录执行
- 在优化代码时 通常需要快速简单的时间测量 以此来验证假设 而不是使用profiler工具/框架
- 可以使用`time`包和`defer`语句执行时间测量

#### 用法

```go
// profile.go
package profile

import (
    "log"
    "time"
)

// 计算函数执行的持续时间
func Duration(invocation time.Time, name string) {
    elapsed := time.Since(invocation)

    log.Printf("%s lasted %s", name, elapsed)
}

// main.go
package main

import (
    "time"

    "package/profile"
)

func Func1() {
    // Arguments to a defer statement is immediately evaluated and stored.
    // The deferred function receives the pre-evaluated values when its invoked.
    // time.Now() 会立即执行并存储 即传入的是该函数的开始执行时间
    defer profile.Duration(time.Now(), "Func1")

    time.Sleep(5 * time.Second)
}

func Func2() {
    defer profile.Duration(time.Now(), "Func2")

    time.Sleep(3 * time.Second)
}

func main() {
    Func1()
    Func2()
}

/*
输出示例:
2025/04/02 12:54:16 Func1 lasted 5.0131114s
2025/04/02 12:54:19 Func2 lasted 3.0020849s
*/

```

## 参考文档

[singleton-pattern-in-go](http://marcio.io/2015/07/singleton-pattern-in-go/)

[Go24种设计模式](https://www.fengfengzhidao.com/article/JI33Q5QB8lppN5cbFPQR)

[go-patterns](https://github.com/tmrts/go-patterns.git)

[golang-design-pattern](https://github.com/senghoo/golang-design-pattern.git)
