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

### 单例模式(Singleton) ✔

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

### 简单工厂模式(Simple Factory) ✔

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

### 工厂方法模式(Factory Method) ✔

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

### 抽象工厂模式(Abstract Factory) ✘

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

### 建造者模式(Builder) ✔

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

### 原型模式(Prototype) ✘

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

- 结构型模式`Structural Pattern` 它主要关注如何将类或对象组合成更大的结构 以便在不改变原有类或对象的情况下 实现新的功能或优化系统结构
- 结构型模式的核心思想是通过组合`Composition`而不是继承`Inheritance`来实现代码的复用和扩展
- 它们帮助开发者设计出灵活、可扩展的系统结构 同时降低类与类之间的耦合度

### 外观模式(Facade) ✔

- 根据迪米特法则 如果两个类不必彼此直接通信 那么这两个类就不应当发生直接的相互作用
- Facade模式也叫外观模式 是由GoF提出的23种设计模式中的一种
- Facade模式为一组具有类似功能的类群 比如类库/子系统等等 提供一个一致的简单的界面
- 这个一致的简单的界面被称作facade

#### 适用场景

- 复杂系统需要简单入口使用
- 客户端程序与多个子系统之间存在很大的依赖性
- 在层次化结构中 可以使用外观模式定义系统中每一层的入口 层与层之间不直接产生联系 而通过外观类建立联系 降低层之间的耦合度

#### 优点

- 它对客户端屏蔽了子系统组件 减少了客户端所需处理的对象数目 并使得子系统使用起来更加容易
- 通过引入外观模式 客户端代码将变得很简单 与之关联的对象也很少
- 它实现了子系统与客户端之间的松耦合关系 这使得子系统的变化不会影响到调用它的客户端 只需要调整外观类即可
- 一个子系统的修改对其他子系统没有任何影响

#### 缺点

- 不能很好地限制客户端直接使用子系统类 如果对客户端访问子系统类做太多的限制则减少了可变性和灵活性
- 如果设计不当 增加新的子系统可能需要修改外观类的源代码 违背了开闭原则

#### 示例

```go
package main

import "fmt"

type SubSystemA struct {}

func (sa *SubSystemA) MethodA() {
    fmt.Println("子系统方法A")
}

type SubSystemB struct {}

func (sb *SubSystemB) MethodB() {
    fmt.Println("子系统方法B")
}

type SubSystemC struct {}

func (sc *SubSystemC) MethodC() {
    fmt.Println("子系统方法C")
}

type SubSystemD struct {}

func (sd *SubSystemD) MethodD() {
    fmt.Println("子系统方法D")
}

// 外观模式 提供了一个外观类 简化成一个简单的接口供使用
type Facade struct {
    a *SubSystemA
    b *SubSystemB
    c *SubSystemC
    d *SubSystemD
}

func (f *Facade) MethodOne() {
    f.a.MethodA()
    f.b.MethodB()
}


func (f *Facade) MethodTwo() {
    f.c.MethodC()
    f.d.MethodD()
}

func main() {
    // 如果不用外观模式实现MethodA() 和 MethodB()
    sa := new(SubSystemA)
    sa.MethodA()
    sb := new(SubSystemB)
    sb.MethodB()

    fmt.Println("-----------")
    // 使用外观模式
    f := Facade{
        a: new(SubSystemA),
        b: new(SubSystemB),
        c: new(SubSystemC),
        d: new(SubSystemD),
    }

    // 调用外观包裹方法
    f.MethodOne()
}

```

- 示例2

```go
package main

import "fmt"

// 电视机
type TV struct {}

func (t *TV) On() {
    fmt.Println("打开 电视机")
}

func (t *TV) Off() {
    fmt.Println("关闭 电视机")
}


// 音箱
type VoiceBox struct {}

func (v *VoiceBox) On() {
    fmt.Println("打开 音箱")
}

func (v *VoiceBox) Off() {
    fmt.Println("关闭 音箱")
}

// 灯光
type Light struct {}

func (l *Light) On() {
    fmt.Println("打开 灯光")
}

func (l *Light) Off() {
    fmt.Println("关闭 灯光")
}


// 游戏机
type Xbox struct {}

func (x *Xbox) On() {
    fmt.Println("打开 游戏机")
}

func (x *Xbox) Off() {
    fmt.Println("关闭 游戏机")
}


// 麦克风
type MicroPhone struct {}

func (m *MicroPhone) On() {
    fmt.Println("打开 麦克风")
}

func (m *MicroPhone) Off() {
    fmt.Println("关闭 麦克风")
}

// 投影仪
type Projector struct {}

func (p *Projector) On() {
    fmt.Println("打开 投影仪")
}

func (p *Projector) Off() {
    fmt.Println("关闭 投影仪")
}


// 家庭影院(外观)
type HomePlayerFacade struct {
    tv TV
    vb VoiceBox
    light Light
    xbox Xbox
    mp MicroPhone
    pro Projector
}


// KTV模式
func (hp *HomePlayerFacade) DoKTV() {
    fmt.Println("家庭影院进入KTV模式")
    hp.tv.On()
    hp.pro.On()
    hp.mp.On()
    hp.light.Off()
    hp.vb.On()
}

// 游戏模式
func (hp *HomePlayerFacade) DoGame() {
    fmt.Println("家庭影院进入Game模式")
    hp.tv.On()
    hp.light.On()
    hp.xbox.On()
}

func main() {
    homePlayer := new(HomePlayerFacade)

    homePlayer.DoKTV()

    fmt.Println("------------")

    homePlayer.DoGame()
}

```

### 适配器模式(Adapter) ✔

- 将一个类的接口转换成客户希望的另外一个接口
- 使得原本由于接口不兼容而不能一起工作的那些类可以一起工作

#### 优点

- 将目标类和适配者类解耦 通过引入一个适配器类来重用现有的适配者类 无须修改原有结构
- 增加了类的透明性和复用性 将具体的业务实现过程封装在适配者类中 对于客户端类而言是透明的 而且提高了适配者的复用性 同一个适配者类可以在多个不同的系统中复用
- 灵活性和扩展性都非常好 可以很方便地更换适配器 也可以在不修改原有代码的基础上增加新的适配器类 完全符合**开闭原则**

#### 缺点

- 适配器中置换适配者类的某些方法比较麻烦

#### 示例

```go
package main

import "fmt"

// 适配的目标
type V5 interface {
    Use5V()
}

// 业务类 依赖V5接口
type Phone struct {
    v V5
}

func NewPhone(v V5) *Phone {
    return &Phone{v}
}

func (p *Phone) Charge() {
    fmt.Println("Phone进行充电...")
    p.v.Use5V()
}

// 被适配的角色 适配者
type V220 struct{}

func (v *V220) Use220V() {
    fmt.Println("使用220V的电压")
}

// 电源适配器
type Adapter struct {
    v220 *V220
}

func (a *Adapter) Use5V() {
    fmt.Println("使用适配器进行充电")

    // 调用适配者的方法
    a.v220.Use220V()
}

func NewAdapter(v220 *V220) *Adapter {
    return &Adapter{v220}
}

// ------- 业务逻辑层 -------
func main() {
    iphone := NewPhone(NewAdapter(new(V220)))

    iphone.Charge()
}

```

### 代理模式(Proxy) ✔

- 它通过提供一个代理对象来控制对另一个对象的访问
- 代理模式的核心思想是在不改变原始对象的情况下 通过代理对象来增强或限制对原始对象的访问

#### 适用场景

- `延迟初始化` 当对象的创建成本较高时 可以通过代理延迟对象的初始化
- `访问控制` 通过代理对象限制对原始对象的访问权限
- `日志记录` 通过代理对象记录对原始对象的访问日志
- `缓存` 通过代理对象缓存原始对象的结果 避免重复计算

#### 示例

```go
package main

import (
    "fmt"
    "time"
)

// 主题接口
type Subject interface {
    Request()
}

// 真实对象
type RealSubject struct{}

func (r *RealSubject) Request() {
    fmt.Println("RealSubject: Handling Request")
}

// 代理对象
type Proxy struct {
    realSubject *RealSubject
}

func NewProxy() *Proxy {
    return &Proxy{}
}

func (p *Proxy) Request() {
    // 延迟初始化真实对象
    if p.realSubject == nil {
        fmt.Println("Proxy: Lazy Initialzing RealSubject")
        p.realSubject = &RealSubject{}
    }

    // 访问控制
    fmt.Println("Proxy: Checking Access")
    time.Sleep(time.Second) //  模拟访问控制逻辑

    // 调用真实对象的方法
    p.realSubject.Request()

    // 日志记录
    fmt.Println("Proxy: Logging Request")
}

func main() {
    proxy := NewProxy()

    // 通过代理对象访问真实对象
    proxy.Request()
}

```

### 组合模式(Composite) ✘

- 它允许你将对象组合成树形结构来表示`部分-整体`的层次关系
- 组合模式让客户端可以统一地处理单个对象和对象组合

#### 适用场景

- 文件系统：如目录和文件的管理 可以通过组合模式将文件夹视为组合节点 文件视为叶子节点
- 组织结构：如公司内部的部门和员工关系 可以通过组合模式将部门视为组合节点 员工视为叶子节点

#### 优点

- 简化客户端代码：客户端可以一致地对待单个对象和对象组合 而不需要关心它们的具体类型
- 增强灵活性：可以在不修改现有代码的情况下轻松添加新的组件或修改现有组件的结构
- 提高可扩展性：支持递归组合 使得复杂的层次结构易于构建和维护

#### 示例

```go
package main

import (
    "fmt"
)

// 组件接口：文件系统节点
type FileSystemNode interface {
    Display(indent string)
}

// 叶子节点：文件
type File struct {
    name string
}

func (f *File) Display(indent string) {
    fmt.Println(indent + f.name)
}

// 组合节点：文件夹
type Folder struct {
    name     string
    children []FileSystemNode
}

func (f *Folder) Display(indent string) {
    fmt.Println(indent + f.name)
    for _, child := range f.children {
        child.Display(indent + "  ")
    }
}

func (f *Folder) Add(child FileSystemNode) {
    f.children = append(f.children, child)
}

// 客户端代码
func main() {
    // 创建文件
    file1 := &File{name: "file1.txt"}
    file2 := &File{name: "file2.txt"}
    file3 := &File{name: "file3.txt"}

    // 创建文件夹
    folder1 := &Folder{name: "Folder1"}
    folder2 := &Folder{name: "Folder2"}

    // 将文件添加到文件夹
    folder1.Add(file1)
    folder1.Add(file2)
    folder2.Add(file3)

    // 将文件夹添加到另一个文件夹
    rootFolder := &Folder{name: "Root"}
    rootFolder.Add(folder1)
    rootFolder.Add(folder2)

    // 显示文件系统结构
    rootFolder.Display("")
}

/*
输出：
Root
  Folder1
    file1.txt
    file2.txt
  Folder2
    file3.txt
*/

```

### 享元模式(Flyweight) ✘

- 它通过共享对象来减少内存使用和提高性能
- 享元模式的核心思想是将对象的共享部分(内部状态)与不可共享部分(外部状态)分离 从而减少重复对象的创建
- 享元模式的核心思想
  - `共享对象` 享元模式通过共享相同的内在状态(内部状态)来减少内存使用
  - `分离状态` 将对象的状态分为内部状态(可共享)和外部状态(不可共享)
  - `工厂管理` 使用工厂模式来管理和复用享元对象

#### 示例

- 在数据库操作中 创建和销毁连接是非常耗时的操作
- 为了提高性能 通常会使用连接池来管理数据库连接 连接池的核心思想是：
  - 复用连接：从连接池中获取连接 使用完后将连接释放回连接池 而不是销毁
  - 减少开销：避免频繁创建和销毁连接 从而提高性能

```go
package main

import (
    "fmt"
    "sync"
)

// 享元接口：数据库连接
type Connection interface {
    Execute(query string)
}

// 具体享元：数据库连接对象
type ConcreteConnection struct {
    id int
}

func (c *ConcreteConnection) Execute(query string) {
    fmt.Printf("Connection %d executing query: %s\n", c.id, query)
}

// 享元工厂：连接池
type ConnectionPool struct {
    connections map[int]Connection
    mutex       sync.Mutex
    nextID      int
}

func NewConnectionPool() *ConnectionPool {
    return &ConnectionPool{
        connections: make(map[int]Connection),
        nextID:      1,
    }
}

// 获取连接对象
func (p *ConnectionPool) GetConnection() Connection {
    p.mutex.Lock()
    defer p.mutex.Unlock()

    if len(p.connections) > 0 {
        for id, conn := range p.connections {
            delete(p.connections, id)
            return conn
        }
    }

    // 创建新的连接对象
    conn := &ConcreteConnection{id: p.nextID}
    p.nextID++
    return conn
}

// 将连接对象释放回连接池 以便其他客户端可以复用这个连接对象
func (p *ConnectionPool) ReleaseConnection(conn Connection) {
    p.mutex.Lock()
    defer p.mutex.Unlock()

    if c, ok := conn.(*ConcreteConnection); ok {
        p.connections[c.id] = c
    }
}

// 客户端代码
func main() {
    pool := NewConnectionPool()

    conn1 := pool.GetConnection()
    conn1.Execute("SELECT * FROM users")
    pool.ReleaseConnection(conn1)

    conn2 := pool.GetConnection()
    conn2.Execute("SELECT * FROM orders")
    pool.ReleaseConnection(conn2)
}

/*
Connection 1 executing query: SELECT * FROM users
Connection 1 executing query: SELECT * FROM orders
*/

```

### 装饰模式(Decorator) ✔

- 它允许你动态地为对象添加行为或职责 而不需要修改对象的原始类
- 通过引入装饰者类 可以在运行时灵活地组合不同的功能 而不需要创建大量的子类
- 装饰者模式的核心思想是将对象包装在一个或多个装饰者中 每个装饰者都可以在调用被装饰对象的方法之前或之后添加额外的行为

#### 优点

- 动态扩展功能
  - 你可以在运行时为请求处理器添加日志记录和性能监控功能 而无需修改核心请求处理器的代码
- 单一职责原则
  - 每个装饰器只负责一个特定的功能(如日志记录或性能监控) 符合单一职责原则
- 灵活性
  - 可以轻松地添加或移除装饰器 而不会影响其他部分的代码

#### 示例

- 假设你正在开发一个Web服务 其中有一个核心功能是处理用户请求
- 现在 你需要在不修改核心功能代码的情况下 为请求处理添加以下功能
  - 日志记录：记录每个请求的详细信息
  - 性能监控：记录每个请求的处理时间
- 使用装饰器模式 你可以轻松地实现这些功能 而无需修改原始请求处理逻辑

```go
package main

import (
    "fmt"
    "time"
)

// 组件接口：请求处理器
type RequestHandler interface {
    HandleRequest(url string) string
}

type CoreRequestHandler struct{}

func (handler *CoreRequestHandler) HandleRequest(url string) string {
    time.Sleep(100 * time.Millisecond)
    return fmt.Sprintf("Response from %s", url)
}

// 装饰器: 日志记录
type LoggingDecorator struct {
    handler RequestHandler
}

func (l *LoggingDecorator) HandleRequest(url string) string {
    fmt.Printf("LoggingDecorator: Request to %s\n", url)
    response := l.handler.HandleRequest(url)
    fmt.Printf("Logging: Response for %s: %s\n", url, response)
    return response
}

// 装饰器：性能监控
type PerformanceMonitorDecorator struct {
    handler RequestHandler
}

func (p *PerformanceMonitorDecorator) HandleRequest(url string) string {
    // 记录开始时间
    start := time.Now()

    // 调用原始处理器
    response := p.handler.HandleRequest(url)

    // 记录处理时间
    duration := time.Since(start)
    fmt.Printf("Performance: Request for %s took %v\n", url, duration)
    return response
}

func main() {
    // 创建请求处理器
    handler := &CoreRequestHandler{}

    // 添加日志记录
    loggingHandler := &LoggingDecorator{handler: handler}

    // 添加性能监控
    monitorHandler := &PerformanceMonitorDecorator{handler: loggingHandler}

    // 处理请求
    fmt.Println(monitorHandler.HandleRequest("http://www.baidu.com"))
}

```

### 桥模式(Bridge) ✘

- 它的核心思想是将抽象部分与实现部分分离 使它们可以独立变化
- 通过这种方式 桥接模式能够避免类的数量爆炸(即类的组合呈指数增长) 同时提高代码的可扩展性和可维护性

#### 示例

- 假设你有两台电脑：一台 Mac 和一台 Windows 还有两台打印机：爱普生和惠普
- 这两台电脑和打印机可能会任意组合使用
- 刚开始功能会这样写 windows组合爱普生和惠普 mac组合爱普生和惠普 打印就调用自己的print方法

```go
package main

import "fmt"

// Mac + Epson
type MacEpson struct{}

func (m *MacEpson) Print() {
    fmt.Println("Print request for mac")
    fmt.Println("Printing by a EPSON Printer")
}

// Mac + HP
type MacHp struct{}

func (m *MacHp) Print() {
    fmt.Println("Print request for mac")
    fmt.Println("Printing by a HP Printer")
}

// Windows + Epson
type WindowsEpson struct{}

func (w *WindowsEpson) Print() {
    fmt.Println("Print request for windows")
    fmt.Println("Printing by a EPSON Printer")
}

// Windows + HP
type WindowsHp struct{}

func (w *WindowsHp) Print() {
    fmt.Println("Print request for windows")
    fmt.Println("Printing by a HP Printer")
}

```

- 但是 如果引入新的打印机 代码量会成倍增长
- 所以 我们需要把计算机和打印机解耦 计算机的print的方法 就桥接到选择的打印机的print方法上

```go
package main

import "fmt"

// 定义Printer接口
type Printer interface {
    PrintFile()
}

// 定义Epson打印机
type Epson struct{}

func (e *Epson) PrintFile() {
    fmt.Println("Printing by Epson printer")
}

// 定义惠普打印机
type HP struct{}

func (h *HP) PrintFile() {
    fmt.Println("Printing by HP printer")
}

// 定义Computer接口
type Computer interface {
    Print()
    SetPrinter(Printer)
}

// 定义Mac计算机
type Mac struct {
    printer Printer
}

func (m *Mac) Print() {
    fmt.Println("Print request for Mac")
    m.printer.PrintFile()
}

func (m *Mac) SetPrinter(p Printer) {
    m.printer = p
}

// 定义windows计算机
type Windows struct {
    printer Printer
}

func (w *Windows) Print() {
    fmt.Println("Print request for Windows")
    w.printer.PrintFile()
}

func (w *Windows) SetPrinter(p Printer) {
    w.printer = p
}

func main() {
    // 创建打印机实例
    hpPrinter := &HP{}
    epsonPrinter := &Epson{}

    // 创建计算机实例
    macComputer := &Mac{}
    winComputer := &Windows{}

    // Mac 使用 HP 打印机
    macComputer.SetPrinter(hpPrinter)
    macComputer.Print()
    fmt.Println()

    // Mac 使用 Epson 打印机
    macComputer.SetPrinter(epsonPrinter)
    macComputer.Print()
    fmt.Println()

    // Windows 使用 HP 打印机
    winComputer.SetPrinter(hpPrinter)
    winComputer.Print()
    fmt.Println()

    // Windows 使用 Epson 打印机
    winComputer.SetPrinter(epsonPrinter)
    winComputer.Print()
    fmt.Println()
}

```

#### 优点

- 解耦
  - 抽象部分和实现部分可以独立变化 互不影响
  - 修改实现部分不会影响抽象部分的代码
- 灵活性
  - 可以动态地切换实现部分(例如 运行时更换实现)
- 可扩展性
  - 新增抽象部分或实现部分时 不需要修改现有代码
- 避免类爆炸
  - 通过组合代替继承 避免了类的数量呈指数增长

## 行为型模式

- 行为型模式`Behavioral pattern` 它主要关注对象之间的职责分配和通信方式
- 行为型模式的核心思想是通过定义对象之间的交互方式 来更好地实现系统的功能 同时降低对象之间的耦合度

### 中介者模式(Mediator)

- 它通过引入一个中介者对象来封装一组对象之间的交互
- 中介者模式的核心思想是 将对象之间的复杂交互集中到一个中介者对象中 从而减少对象之间的直接耦合

#### 优点

- 减少耦合：对象之间不直接通信 而是通过中介者进行交互 减少了对象之间的耦合
- 集中控制：将对象之间的交互逻辑集中到中介者中 便于维护和扩展
- 简化对象职责：每个对象只需要关注自己的行为 而不需要知道其他对象的存在

#### 示例

- 假设我们有一个聊天室系统 用户(同事类)之间通过聊天室(中介者)发送消息
- 用户之间不直接通信 而是通过聊天室来转发消息

```go
package main

import "fmt"

// 定义中介者接口
type Mediator interface {
    SendMessage(message string, sender Colleague)
}

// 具体中介者
type ChatRoom struct {
    users []Colleague
}

func (c *ChatRoom) Register(user Colleague) {
    c.users = append(c.users, user)
}

func (c *ChatRoom) SendMessage(message string, sender Colleague) {
    for _, user := range c.users {
        if user != sender {
            user.Receive(message)
        }
    }
}

// 定义同事类接口
type Colleague interface {
    Send(message string)
    Receive(message string)
}

// 具体同事类
type User struct {
    name     string
    mediator Mediator
}

func (u *User) Send(message string) {
    fmt.Printf("%s sends: %s\n", u.name, message)
    u.mediator.SendMessage(message, u)
}

func (u *User) Receive(message string) {
    fmt.Printf("%s receives: %s\n", u.name, message)
}

func main() {
    // 创建中介者
    chatRoom := &ChatRoom{}

    // 创建用户
    alice := &User{name: "Alice", mediator: chatRoom}
    bob := &User{name: "Bob", mediator: chatRoom}
    charlie := &User{name: "Charlie", mediator: chatRoom}

    // 注册用户到聊天室
    chatRoom.Register(alice)
    chatRoom.Register(bob)
    chatRoom.Register(charlie)

    // 用户发送消息
    alice.Send("Hello, everyone!")
    bob.Send("Hi, Alice!")
    charlie.Send("Hey, Bob!")
}

```

### 观察者模式(Observer) ✔

- 它定义了对象之间的一对多依赖关系
- 当一个对象(被观察者)的状态发生改变时 所有依赖它的对象(观察者)都会收到通知并自动更新
- 观察者模式的核心思想是 解耦被观察者和观察者 使得它们可以独立变化

#### 优点

- 解耦：被观察者和观察者之间是松耦合的 它们可以独立变化
- 支持动态注册和移除观察者：可以在运行时动态地注册和移除观察者
- 符合开闭原则：新增观察者时 不需要修改被观察者的代码

#### 示例

- 假设我们有一个天气站(被观察者) 当天气数据更新时 通知多个显示设备(观察者)更新显示内容

```go
package main

import "fmt"

// 定义观察者接口
type Observer interface {
    Update(temperature float64, humidity float64, pressure float64)
}

// 定义被观察者接口
type Subject interface {
    RegisterObserver(o Observer)
    RemoveObserver(o Observer)
    NotifyObservers()
}

// 具体被观察者
type WeatherData struct {
    observers   []Observer
    temperature float64
    humidity    float64
    pressure    float64
}

func (w *WeatherData) RegisterObserver(o Observer) {
    w.observers = append(w.observers, o)
}

func (w *WeatherData) RemoveObserver(o Observer) {
    for i, observer := range w.observers {
        if observer == o {
            w.observers = append(w.observers[:i], w.observers[i+1:]...)
            break
        }
    }
}

func (w *WeatherData) NotifyObservers() {
    for _, observer := range w.observers {
        observer.Update(w.temperature, w.humidity, w.pressure)
    }
}

func (w *WeatherData) SetMeasurements(temperature float64, humidity float64, pressure float64) {
    w.temperature = temperature
    w.humidity = humidity
    w.pressure = pressure
    w.NotifyObservers()
}

// 具体观察者
type DisplayDevice struct {
    name string
}

func (d *DisplayDevice) Update(temperature float64, humidity float64, pressure float64) {
    fmt.Printf("%s: Temperature=%.2f, Humidity=%.2f, Pressure=%.2f\n", d.name, temperature, humidity, pressure)
}

func main() {
    // 创建被观察者
    weatherData := &WeatherData{}

    // 创建观察者
    display1 := &DisplayDevice{name: "Display 1"}
    display2 := &DisplayDevice{name: "Display 2"}

    // 注册观察者
    weatherData.RegisterObserver(display1)
    weatherData.RegisterObserver(display2)

    // 更新天气数据并通知观察者
    weatherData.SetMeasurements(25.0, 65.0, 1013.0)
    weatherData.SetMeasurements(26.5, 70.0, 1012.5)

    // 移除一个观察者
    weatherData.RemoveObserver(display1)

    // 再次更新天气数据
    weatherData.SetMeasurements(27.0, 68.0, 1011.0)
}

```

### 命令模式(Command) ✔

- 它将请求封装为一个对象 从而使你可以用不同的请求对客户进行参数化 并且支持请求的排队、记录日志、撤销操作等功能

#### 核心思想

- 将请求封装为对象：将每个请求(如方法调用)封装为一个独立的对象
- 解耦请求的发送者和接收者：发送者不需要知道接收者的具体实现 只需要通过命令对象来执行请求
- 支持扩展：可以轻松地添加新的命令 而不需要修改现有代码

#### 示例

```go
package main

import (
    "fmt"
    "time"
)

// 定义命令接口
type Command interface {
    Execute()
}

// 具体命令 表示打印消息
type PrintCommand struct {
    message string
}

func (c *PrintCommand) Execute() {
    fmt.Println("Executing PrintCommand:", c.message)
}

// 具体命令 表示发送邮件
type EmailCommand struct {
    to      string
    subject string
    body    string
}

func (c *EmailCommand) Execute() {
    fmt.Printf("Executing EmailCommand: Sending email to %s\n", c.to)
    fmt.Printf("Subject: %s\n", c.subject)
    fmt.Printf("Body: %s\n", c.body)
}

// 任务队列 用于存储和执行命令
type TaskQueue struct {
    commands []Command
}

func (tq *TaskQueue) AddCommand(cmd Command) {
    tq.commands = append(tq.commands, cmd)
}

func (tq *TaskQueue) ProcessCommands() {
    for _, cmd := range tq.commands {
        cmd.Execute()
        time.Sleep(1 * time.Second) // 模拟任务执行的时间
    }
    tq.commands = nil // 清空任务队列
}

func main() {
    // 创建任务队列
    taskQueue := &TaskQueue{}

    // 添加任务到队列
    taskQueue.AddCommand(&PrintCommand{message: "Hello, World!"})
    taskQueue.AddCommand(&EmailCommand{
        to:      "user@example.com",
        subject: "Welcome",
        body:    "Thank you for signing up!",
    })
    taskQueue.AddCommand(&PrintCommand{message: "Task queue is running..."})

    // 处理任务队列
    fmt.Println("Processing task queue:")
    taskQueue.ProcessCommands()

    // 添加更多任务
    taskQueue.AddCommand(&PrintCommand{message: "Another task"})
    taskQueue.AddCommand(&EmailCommand{
        to:      "admin@example.com",
        subject: "Report",
        body:    "Here is the daily report.",
    })

    // 再次处理任务队列
    fmt.Println("\nProcessing task queue again:")
    taskQueue.ProcessCommands()
}

```

### 迭代器模式(Iterator)

- 它提供了一种方法顺序访问一个聚合对象中的各个元素 而又不需要暴露该对象的内部表示
- 迭代器模式的核心思想是将遍历逻辑从聚合对象中分离出来 使得聚合对象和遍历逻辑可以独立变化

#### 适用场景

- 集合类库：如Go的 slice、map 等集合类型 可以通过迭代器模式提供统一的遍历接口
- 数据库查询结果：数据库查询结果可以封装为一个聚合对象 并提供迭代器遍历查询结果
- 文件系统遍历：文件系统中的目录和文件可以封装为一个聚合对象 并提供迭代器遍历文件系统
- 树形结构遍历：树形结构(如二叉树、多叉树)可以提供多种遍历方式(如深度优先、广度优先)

#### 示例

- 假设我们有一个集合类BookCollection 它包含多本书
- 我们需要实现一个迭代器 用于遍历这个集合

```go
package main

import "fmt"

// 定义书籍结构体
type Book struct {
    Title  string
    Author string
}

// 定义迭代器接口
type Iterator interface {
    HasNext() bool
    Next() *Book
}

// 具体迭代器
type BookIterator struct {
    books    []*Book
    position int
}

func (b *BookIterator) HasNext() bool {
    return b.position < len(b.books)
}

func (b *BookIterator) Next() *Book {
    if b.HasNext() {
        book := b.books[b.position]
        b.position++
        return book
    }
    return nil
}

// 定义聚合接口
type Aggregate interface {
    CreateIterator() Iterator
}

// 具体聚合
type BookCollection struct {
    books []*Book
}

func (b *BookCollection) CreateIterator() Iterator {
    return &BookIterator{books: b.books}
}

func main() {
    // 创建书籍集合
    collection := &BookCollection{
        books: []*Book{
            {Title: "The Go Programming Language", Author: "Alan A. A. Donovan"},
            {Title: "Design Patterns", Author: "Erich Gamma"},
            {Title: "Clean Code", Author: "Robert C. Martin"},
        },
    }

    // 创建迭代器
    iterator := collection.CreateIterator()

    // 遍历集合
    for iterator.HasNext() {
        book := iterator.Next()
        fmt.Printf("Title: %s, Author: %s\n", book.Title, book.Author)
    }
}

```

### 模板方法模式(Template Method) ✔

- 它定义了一个算法的骨架 并将某些步骤延迟到子类中实现
- 模板方法模式的核心思想是 将算法的通用部分放在父类中 而将可变部分交给子类实现
- 通过模板方法模式 可以避免代码重复 并确保算法的结构不变

#### 适用场景

- `框架设计` 如Web框架中的请求处理流程(初始化、处理请求、返回响应等)
- `工作流引擎` 如订单处理流程(创建订单、支付订单、发货订单等)
- `测试框架` 如测试用例的执行流程(初始化、执行测试、清理资源等)
- `游戏开发` 如游戏角色的行为流程(移动、攻击、防御等)

#### 优点

- `代码复用` 将算法的通用部分放在父类中 避免了代码重复
- `扩展性` 新增子类时 只需要实现抽象方法 而不需要修改父类的代码
- `控制算法结构` 父类控制算法的结构 确保算法的步骤不会被子类改变

#### 示例

- 假设我们有一个饮料制作系统 支持制作咖啡和茶
- 咖啡和茶的制作过程有一些共同的步骤(如烧水、倒入杯子) 也有一些不同的步骤(如冲泡咖啡、泡茶)

```go
package main

import "fmt"

// 抽象类，定义饮料制作的模板方法
type Beverage interface {
    BoilWater()           // 烧水
    Brew()                // 冲泡
    PourInCup()           // 倒入杯子
    AddCondiments()       // 添加调料
    WantCondiments() bool // 是否添加调料（钩子方法）
}

// 模板方法
func MakeBeverage(b Beverage) {
    b.BoilWater()
    b.Brew()
    b.PourInCup()
    if b.WantCondiments() {
        b.AddCondiments()
    }
}

// 具体类：咖啡
type Coffee struct{}

func (c *Coffee) BoilWater() {
    fmt.Println("Boiling water")
}

func (c *Coffee) Brew() {
    fmt.Println("Brewing coffee")
}

func (c *Coffee) PourInCup() {
    fmt.Println("Pouring coffee into cup")
}

func (c *Coffee) AddCondiments() {
    fmt.Println("Adding sugar and milk")
}

func (c *Coffee) WantCondiments() bool {
    return true
}

// 具体类：茶
type Tea struct{}

func (t *Tea) BoilWater() {
    fmt.Println("Boiling water")
}

func (t *Tea) Brew() {
    fmt.Println("Steeping tea")
}

func (t *Tea) PourInCup() {
    fmt.Println("Pouring tea into cup")
}

func (t *Tea) AddCondiments() {
    fmt.Println("Adding lemon")
}

func (t *Tea) WantCondiments() bool {
    return false
}

func main() {
    // 制作咖啡
    fmt.Println("Making coffee:")
    coffee := &Coffee{}
    MakeBeverage(coffee)

    // 制作茶
    fmt.Println("\nMaking tea:")
    tea := &Tea{}
    MakeBeverage(tea)
}

```

### 策略模式(Strategy) ✔

- 它定义了一系列算法 并将每个算法封装起来 使它们可以互相替换
- 策略模式的核心思想是将算法的使用与算法的实现分离 从而使得算法可以独立于客户端而变化
- 通过策略模式 可以在运行时动态地选择算法 而不需要修改客户端代码

#### 适用场景

- 支付系统：如示例所示 支持多种支付方式
- 排序算法：支持多种排序算法(如快速排序、归并排序、冒泡排序等)
- 压缩算法：支持多种压缩算法(如ZIP、RAR、7z等)
- 路由算法：支持多种路由算法(如最短路径、最快路径、最少收费路径等)

#### 优点

- 解耦：将算法的使用与算法的实现分离 使得算法可以独立于客户端而变化
- 易于扩展：新增算法时 只需要添加新的策略类 而不需要修改现有代码
- 动态选择算法：可以在运行时动态地选择算法 而不需要修改客户端代码

#### 示例

- 假设我们有一个支付系统 支持多种支付方式(如信用卡支付、支付宝支付、微信支付等) 我们可以使用策略模式来实现支付方式的选择

```go
package main

import "fmt"

// 定义策略接口
type PaymentStrategy interface {
    Pay(amount float64)
}

// 具体策略：信用卡支付
type CreditCardPayment struct{}

func (c *CreditCardPayment) Pay(amount float64) {
    fmt.Printf("Paying %.2f using Credit Card\n", amount)
}

// 具体策略：支付宝支付
type AlipayPayment struct{}

func (a *AlipayPayment) Pay(amount float64) {
    fmt.Printf("Paying %.2f using Alipay\n", amount)
}

// 具体策略：微信支付
type WechatPayment struct{}

func (w *WechatPayment) Pay(amount float64) {
    fmt.Printf("Paying %.2f using Wechat Pay\n", amount)
}

// 上下文
type PaymentContext struct {
    strategy PaymentStrategy
}

func (p *PaymentContext) SetStrategy(strategy PaymentStrategy) {
    p.strategy = strategy
}

func (p *PaymentContext) ExecutePayment(amount float64) {
    p.strategy.Pay(amount)
}

func main() {
    // 创建上下文
    context := &PaymentContext{}

    // 使用信用卡支付
    context.SetStrategy(&CreditCardPayment{})
    context.ExecutePayment(100.0) // 输出: Paying 100.00 using Credit Card

    // 使用支付宝支付
    context.SetStrategy(&AlipayPayment{})
    context.ExecutePayment(200.0) // 输出: Paying 200.00 using Alipay

    // 使用微信支付
    context.SetStrategy(&WechatPayment{})
    context.ExecutePayment(300.0) // 输出: Paying 300.00 using Wechat Pay
}

```

### 状态模式(State)

- 它允许对象在其内部状态改变时改变其行为
- 状态模式的核心思想是将对象的状态封装成独立的类 并将对象的行为委托给当前状态对象
- 通过状态模式 可以将复杂的条件逻辑分散到多个状态类中 从而使得代码更加清晰和易于维护

#### 优点

- 清晰的状态转换逻辑：将状态转换逻辑分散到各个状态类中 避免了复杂的条件语句
- 符合开闭原则：新增状态时 只需要添加新的状态类 而不需要修改现有代码
- 易于扩展：可以轻松地添加新的状态和行为

#### 示例

```go
package main

import "fmt"

// 定义状态接口
type State interface {
    PressSwitch(context *Context)
}

// 上下文
type Context struct {
    state State
}

func (c *Context) SetState(state State) {
    c.state = state
}

func (c *Context) PressSwitch() {
    c.state.PressSwitch(c)
}

// 具体状态：开
type OnState struct{}

func (o *OnState) PressSwitch(context *Context) {
    fmt.Println("Turning the light off...")
    context.SetState(&OffState{})
}

// 具体状态：关
type OffState struct{}

func (o *OffState) PressSwitch(context *Context) {
    fmt.Println("Turning the light on...")
    context.SetState(&OnState{})
}

func main() {
    // 创建上下文并设置初始状态
    context := &Context{state: &OffState{}}

    // 按下开关
    context.PressSwitch() // 输出: Turning the light on...
    context.PressSwitch() // 输出: Turning the light off...
    context.PressSwitch() // 输出: Turning the light on...
}

```

### 备忘录模式(Memento)

- 它允许在不破坏封装性的前提下 捕获并外部化一个对象的内部状态 以便稍后可以将该对象恢复到之前的状态
- 备忘录模式的核心思想是 将对象的状态保存到一个备忘录对象中 并在需要时从备忘录中恢复状态

#### 优点

- 这种方式非常适合需要支持撤销操作或状态管理的场景
- 比如文本编辑器、游戏存档、事务回滚等
- 在Go语言中 备忘录模式可以通过接口和结构体的组合来实现 代码简洁且易于扩展

#### 示例

```go
package main

import "fmt"

// 定义备忘录接口
type Memento interface {
    GetState() string
}

// 具体备忘录
type TextMemento struct {
    state string
}

func (t *TextMemento) GetState() string {
    return t.state
}

// 原发器
type Originator struct {
    state string
}

func (o *Originator) SetState(state string) {
    o.state = state
}

func (o *Originator) GetState() string {
    return o.state
}

func (o *Originator) CreateMemento() Memento {
    return &TextMemento{state: o.state}
}

func (o *Originator) RestoreMemento(m Memento) {
    o.state = m.GetState()
}

// 管理者
type Caretaker struct {
    mementos []Memento
}

func (c *Caretaker) AddMemento(m Memento) {
    c.mementos = append(c.mementos, m)
}

func (c *Caretaker) GetMemento(index int) Memento {
    if index < len(c.mementos) {
        return c.mementos[index]
    }
    return nil
}

func main() {
    // 创建原发器
    editor := &Originator{}

    // 创建管理者
    caretaker := &Caretaker{}

    // 编辑文本并保存状态
    editor.SetState("State 1")
    caretaker.AddMemento(editor.CreateMemento()) // 创建一个备忘录对象，保存当前状态。

    editor.SetState("State 2")
    caretaker.AddMemento(editor.CreateMemento())

    editor.SetState("State 3")
    caretaker.AddMemento(editor.CreateMemento())

    fmt.Println(editor.GetState()) // 这个时候的状态已经是  State 3

    // 恢复到之前的状态
    editor.RestoreMemento(caretaker.GetMemento(1))
    fmt.Println("Restored State:", editor.GetState()) // 输出: Restored State: State 2

    editor.RestoreMemento(caretaker.GetMemento(0))
    fmt.Println("Restored State:", editor.GetState()) // 输出: Restored State: State 1
}

```

### 解释器模式(Interpreter)

- 它定义了一种语言的语法表示 并提供了一个解释器来解释这种语法
- 解释器模式通常用于处理类似编程语言、查询语言、规则引擎等场景
- 核心思想
  - 定义语法规则：将语言的语法规则表示为一个抽象语法树(AST)
  - 解释执行：通过解释器遍历抽象语法树 执行相应的操作

#### 抽象语法树

- 抽象语法树(Abstract Syntax Tree-AST)是编译器和解释器中的一个核心概念
- 它是一种树形数据结构 用于表示源代码的语法结构
- AST 之所以被称为`抽象语法树` 是因为它抽象掉了源代码中的一些具体细节 只保留了程序的逻辑结构
- 为什么叫 抽象语法树
  - 抽象(Abstract)
    - AST 并不包含源代码中的所有细节 比如括号、分号、空格等 它只关注程序的逻辑结构
    - 例如 表达式 2 * (3 + 4) 的`AST`会抽象掉括号 只保留运算符和操作数的层次关系
  - 语法(Syntax)
    - AST 表示的是源代码的语法结构 而不是语义(即程序的含义)
    - 例如 AST 可以表示一个`if`语句的语法结构 但不会解释`if`语句的具体行为
  - 树(Tree)
    - AST是一种树形数据结构 每个节点表示一个语法结构(如表达式、语句、函数等)
    - 树的层次结构反映了源代码的嵌套关系
- 比如：

```go
if x > 0 {
    print(Positive)
} else {
    print(Non-positive)
}

```

- 对应的AST如下:
  
```bash
IfStatement
├── Condition: BinaryExpression
│   ├── Left: Identifier (x)
│   ├── Operator: >
│   └── Right: Number (0)
├── ThenBlock: BlockStatement
│   └── Expression: CallExpression
│       ├── Function: Identifier (print)
│       └── Arguments: StringLiteral ("Positive")
└── ElseBlock: BlockStatement
    └── Expression: CallExpression
        ├── Function: Identifier (print)
        └── Arguments: StringLiteral ("Non-positive")

```

#### 适用场景

- 数据库查询引擎：如 MySQL、PostgreSQL 等数据库的查询解析和执行
- 规则引擎：如业务规则引擎 用于解析和执行规则
- 模板引擎：如Go的`text/template`用于解析和执行模板
- 数学表达式计算：如计算器应用程序 用于解析和计算数学表达式

#### 示例

- 以Go语言里面的模板字符串为例

```go
package main

import (
    "os"
    "text/template"
)

func main() {
    // 定义模板字符串
    const tmpl = `Hello, {{.Name}}! You are {{.Age}} years old.`

    // 解析模板
    t, err := template.New("example").Parse(tmpl)
    if err != nil {
        panic(err)
    }

    // 定义数据
    data := struct {
        Name string
        Age  int
    }{
        Name: "Alice", Age: 25,
    }

    // 执行模板并输出结果
    err = t.Execute(os.Stdout, data)
    if err != nil {
        panic(err)
    }
}

```

- 这个是解释器模式的应用 我们需要了解内部的原理 然后自己手搓一个这样的模板字符串解析器出来

```go
package main

import (
    "fmt"
    "strings"
)

// 定义上下文 存储模板数据
type Context struct {
    Data map[string]interface{}
}

// 定义抽象表达式接口
type Expression interface {
    Interpret(ctx *Context) string
}

// 是文本表达式 表示普通文本
type TextExpression struct {
    text string
}

func (t *TextExpression) Interpret(ctx *Context) string {
    return t.text
}

// 是变量表达式 表示变量替换
type VariableExpression struct {
    key string
}

func (v *VariableExpression) Interpret(ctx *Context) string {
    value, ok := ctx.Data[v.key]
    if !ok {
        return ""
    }
    return fmt.Sprintf("%v", value)
}

// 是模板 包含多个表达式
type Template struct {
    expressions []Expression
}

func (t *Template) Interpret(ctx *Context) string {
    var result strings.Builder
    for _, expr := range t.expressions {
        result.WriteString(expr.Interpret(ctx))
    }
    return result.String()
}

// 解析模板字符串并构建抽象语法树
func ParseTemplate(tmpl string) *Template {
    template := &Template{}
    start := 0
    for {
        // 查找变量表达式的起始位置
        startVar := strings.Index(tmpl[start:], "{{")
        if startVar == -1 {
            // 如果没有变量表达式，剩下的都是普通文本
            template.expressions = append(template.expressions, &TextExpression{text: tmpl[start:]})
            break
        }
        // 添加普通文本
        template.expressions = append(template.expressions, &TextExpression{text: tmpl[start : start+startVar]})
        // 查找变量表达式的结束位置
        endVar := strings.Index(tmpl[start+startVar:], "}}")
        if endVar == -1 {
            // 如果没有结束符，直接退出
            break
        }
        // 解析变量表达式
        key := strings.TrimSpace(tmpl[start+startVar+2 : start+startVar+endVar])
        template.expressions = append(template.expressions, &VariableExpression{key: key})
        // 更新起始位置
        start = start + startVar + endVar + 2
    }
    return template
}

func main() {
    // 定义模板字符串
    tmpl := "Hello, {{Name}}! You are {{Age}} years old."

    // 解析模板并构建抽象语法树
    template := ParseTemplate(tmpl)

    // 定义上下文数据
    ctx := &Context{
        Data: map[string]interface{}{
            "Name": "Alice",
            "Age":  25,
        },
    }

    // 解释执行模板并输出结果
    result := template.Interpret(ctx)
    fmt.Println(result) // 输出: Hello, Alice! You are 25 years old.
}

```

### 职责链模式(Chain of Responsibility)

- 是一种行为设计模式 它允许多个对象有机会处理请求 从而避免请求的发送者与接收者之间的耦合
- 责任链模式将这些对象连成一条链 并沿着这条链传递请求 直到有对象处理它为止

#### 核心思想

- 解耦请求发送者和接收者：请求的发送者不需要知道具体由哪个对象处理请求 只需要将请求发送到链上即可
- 动态组合处理逻辑：可以动态地组合处理者 灵活地调整处理顺序或增加新的处理者
- 每个处理者只关心自己的职责：每个处理者只处理自己能处理的请求 不能处理的请求会传递给下一个处理者

#### 适用场景

- 当一个请求需要经过多个对象处理 但具体由哪个对象处理不确定时
- 当需要动态指定处理请求的对象时
- 当希望避免请求发送者与接收者之间的紧密耦合时
- 实际应用场景
  - 中间件
  - 审批流程

#### 示例

- 实现一下gin的中间件功能 中间件自己可以处理请求 可以拦截 可以放行

```go
package main

import (
    "fmt"
    "net/http"
)

// 模拟Gin的上下文
type Context struct {
    Request  *http.Request
    Writer   http.ResponseWriter
    handlers []HandlerFunc // 责任链中的处理函数
    index    int           // 当前执行的处理函数索引
}

// 定义处理函数类型
type HandlerFunc func(c *Context)

// 调用责任链中的下一个处理函数
func (c *Context) Next() {
    c.index++
    if c.index < len(c.handlers) {
        c.handlers[c.index](c)
    }
}

// 终止责任链的执行
func (c *Context) Abort() {
    c.index = len(c.handlers) // 直接跳到末尾 不再执行后续处理函数
}

// 模拟Gin的引擎
type Engine struct {
    handlers []HandlerFunc // 全局中间件和责任链
}

// 添加中间件到责任链
func (e *Engine) Use(middleware HandlerFunc) {
    e.handlers = append(e.handlers, middleware)
}

// 实现http.Handler接口
func (e *Engine) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    c := &Context{
        Request:  r,
        Writer:   w,
        handlers: e.handlers,
        index:    -1,
    }
    c.Next() // 开始执行责任链
}

// 示例中间件 1：日志记录
func LoggingMiddleware(c *Context) {
    fmt.Printf("Request: %s %s\n", c.Request.Method, c.Request.URL.Path)
    c.Next() // 传递给下一个中间件
}

// 示例中间件 2：认证检查
func AuthMiddleware(c *Context) {
    token := c.Request.Header.Get("Authorization")
    if token != "valid-token" {
        c.Writer.WriteHeader(http.StatusUnauthorized)
        c.Writer.Write([]byte("Unauthorized"))
        c.Abort() // 终止责任链
        return
    }
    c.Next() // 传递给下一个中间件
}

// 示例中间件 3：请求处理
func HelloHandler(c *Context) {
    c.Writer.WriteHeader(http.StatusOK)
    c.Writer.Write([]byte("Hello, World!"))
}

func main() {
    // 创建引擎
    engine := &Engine{}

    // 添加中间件到责任链
    engine.Use(LoggingMiddleware)
    engine.Use(AuthMiddleware)
    engine.Use(HelloHandler)

    // 启动 HTTP 服务器
    fmt.Println("Server is running on :9090")
    fmt.Println(http.ListenAndServe(":9090", engine))
}

```

### 访问者模式(Visitor)

- 它允许你将算法与对象结构分离
- 访问者模式的核心思想是 将操作(算法)从对象结构中分离出来 使得可以在不修改对象结构的情况下定义新的操作
- 通过访问者模式 可以将对象结构的元素与作用于这些元素的操作解耦 从而使得操作可以独立变化

#### 优点

- 解耦：将操作与对象结构分离 使得操作可以独立变化
- 扩展性：新增操作时 只需要添加新的访问者 而不需要修改对象结构。
- 符合开闭原则：对象结构对扩展开放 对修改关闭

#### 示例

- 假设我们有一个文档对象模型(DOM) 包含多种元素(如文本、图片、表格等) 我们需要对这些元素进行不同的操作(如导出为PDF、导出为HTML等)

```go
package main

import "fmt"

// 定义访问者接口
type Visitor interface {
    VisitText(text *Text)
    VisitImage(image *Image)
    VisitTable(table *Table)
}

// 定义元素接口
type Element interface {
    Accept(visitor Visitor)
}

// 具体元素：文本
type Text struct {
    content string
}

func (t *Text) Accept(visitor Visitor) {
    visitor.VisitText(t)
}

// 具体元素：图片
type Image struct {
    src string
}

func (i *Image) Accept(visitor Visitor) {
    visitor.VisitImage(i)
}

// 具体元素：表格
type Table struct {
    rows int
    cols int
}

func (t *Table) Accept(visitor Visitor) {
    visitor.VisitTable(t)
}

// 具体访问者：导出为 PDF
type PDFExportVisitor struct{}

func (p *PDFExportVisitor) VisitText(text *Text) {
    fmt.Printf("Exporting text to PDF: %s\n", text.content)
}

func (p *PDFExportVisitor) VisitImage(image *Image) {
    fmt.Printf("Exporting image to PDF: %s\n", image.src)
}

func (p *PDFExportVisitor) VisitTable(table *Table) {
    fmt.Printf("Exporting table to PDF: %d rows, %d cols\n", table.rows, table.cols)
}

// 具体访问者：导出为 HTML
type HTMLExportVisitor struct{}

func (h *HTMLExportVisitor) VisitText(text *Text) {
    fmt.Printf("Exporting text to HTML: %s\n", text.content)
}

func (h *HTMLExportVisitor) VisitImage(image *Image) {
    fmt.Printf("Exporting image to HTML: %s\n", image.src)
}

func (h *HTMLExportVisitor) VisitTable(table *Table) {
    fmt.Printf("Exporting table to HTML: %d rows, %d cols\n", table.rows, table.cols)
}

// 是对象结构
type Document struct {
    elements []Element
}

func (d *Document) AddElement(element Element) {
    d.elements = append(d.elements, element)
}

func (d *Document) Accept(visitor Visitor) {
    for _, element := range d.elements {
        element.Accept(visitor)
    }
}

func main() {
    // 创建文档对象
    document := &Document{}

    // 添加元素
    document.AddElement(&Text{content: "Hello, World!"})
    document.AddElement(&Image{src: "image.png"})
    document.AddElement(&Table{rows: 3, cols: 2})

    // 创建访问者
    pdfVisitor := &PDFExportVisitor{}
    htmlVisitor := &HTMLExportVisitor{}

    // 导出为 PDF
    fmt.Println("Exporting to PDF:")
    document.Accept(pdfVisitor)

    // 导出为 HTML
    fmt.Println("\nExporting to HTML:")
    document.Accept(htmlVisitor)
}

```

## 其余常用模式

- 除开23种标准模式及简单工厂模式之外的常用设计模式

### 对象池模式(Object Pool Pattern) ✔

- 创建型模式
- 对象池创建设计模式用于根据需求预期准备和保留多个实例

#### 实现

```go
package pool

type Pool chan *Object

func New(total int) *Pool {
    p := make(Pool, total)

    for i :=0; i < total; i++ {
        p <- new(Object)
    }

    return &p
}

```

#### 用法

- 下面给出了一个关于对象池的简单生命周期示例

```go
p := pool.New(2)

select {
case obj := <-p:
    obj.Do( /*...*/ )

    p <- obj
default:
    // No more objects left — retry later or fail
    return
}

```

#### 经验法则

- 对象池模式在对象初始化比对象维护成本更高的情况下非常有用
- 如果需求出现峰值 而不是稳定需求 则维护开销可能会超过对象池的优势
- 由于对象是预先初始化的 因此它对性能有积极影响

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
