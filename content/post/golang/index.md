---
title: "Golang"
description: An open-source programming language supported by Google.
date: 2021-06-29 17:03:46+08:00
categories: 
- Golang
tags:
- Cloud Native
- Golang
---

<!-- markdownlint-disable-file MD036 -->
<!-- markdownlint-disable-file MD024 -->

## 环境准备

### 安装

[官网下载安装](https://golang.google.cn/)

- Windwos下载go.msi安装
- Linux下载go.tar.gz

```bash
解压: tar -xzvf go.tar.gz -C /usr/local/
配置环境变量(windows安装会自动配置)
  go $PATH
  GOROOT => go安装路径
  GOPATH => $HOME/go

```

### 配置

[Go模块代理](https://goproxy.cn/)

```bash
GO111MODULE=on    # go模块管理
GOPROXY=https://goproxy.cn,direct  # 下载第三方包网络有问题的时候设置代理

# Windows
1. 直接配置到环境变量(优先级高) => 系统 - 高级设置 - 新增环境变量

2. 配置到go env
   go env -w GO111MODULE=on  # 写入go环境变量
   go env -w GOPROXY=https://goproxy.cn,direct
           
# Linux 
# 配置脚本 /etc/profile.d/go.sh
if [[ "x" == "x${GOROOT}" ]]; then
    export GOROOT=/usr/local/go
    export GOPATH=${HOME}/go
    export PATH=${PATH}:${GOROOT}/bin:${GOPATH}/bin
fi

```

### 编辑器

```bash
vscode + 插件即可

ctrl + shift + p -> go install/update tools -> 全选安装插件

```

### 运行程序

```go
// go工具链
go build hello.go // 编译
go run hello.go   // 编译+运行

常用参数：
-x    // 打印执行命令
-n    // 只打印编译过程 并不执行
-o    // 执行程序输出名称
-work // 保留工作目录

```

## 基本语法

### 基本结构

```go
/* 
声明包 go要求每个源码都有属于的包
格式：package 包名
包名：目前阶段定义为main
声明包必须在源码的第一行 包声明前面可以包含注释
*/
package main  

// 导入包：使用标准包或第三方包需要先导入
import (
    "fmt"
)

// 目前阶段认为 main函数为程序入口
func main() {
    fmt.Println("Hello, Golang")
}

```

### 简单示例

```go
package main

import (
    "fmt"
)

func main() {
    // var 定义变量r 类型为float64 值为10
    var r float64 = 10
    var confirm string

    // const定义常量
    const pi float64 = 3.1415926

    for {
        // 从控制台上读取数据
        fmt.Print("请输入半径：") // 打印内容之后不会加换行
        fmt.Scan(&r)

        // 计算圆形面积
        fmt.Println(pi * r * r) // 打印内容之后自动换行

        // 从控制台接收是否继续的数据
        fmt.Print("是否继续(y/n)?")
        fmt.Scan(&confirm)
        if confirm != "y" {
            break
        }
    }
}

```

### 基本组成元素

```bash
1. 标识符
2. 关键字
3. 字面量
4. 操作符
5. 分隔符

```

## 变量常量声明

### 变量声明

```go
// 变量声明: var
// var 标识符 类型

package main

import (
    "fmt"
)

// 定义全局变量
var version string = "1.1.1" // 显示赋值

func main() {
    // go中要求局部变量定义后必须使用
    // 定义局部变量
    // 未对局部变量进行显示复制 go会对变量设置默认值(零值)
    // string 零值： 空字符串 ""
    var funcVersion string

    // 省略类型 帮你徐显示设置默认值 go根据字面量或赋值的值类型进行推导
    var name = "minho" // => var name string = "minho"

    // 批量定义 没设置默认值需要指定类型 设置默认值可以不指定类型
    var (
        a1      int
        a2      string
        a3      int    = 5
        a4      string = "aaa"
        a5             = 5
        a6             = "666"
        a9, a10 int
    )

    var a7, a8 string // 类型一样
    var a11, a12 string = "a11", "a12"

    fmt.Println(funcVersion, version, name)
    fmt.Println(a1, a2, a3, a4, a5, a6, a7, a8, a9, a10, a11, a12)
}

```

### 常量声明

```go
// 常量声明: const

package main

import (
    "fmt"
)

const version string = "2.2.2"

func main() {
    // 常量必须显示设置默认值(因为常量定义之后就不能修改)
    const funcVersion string = ""

    const pi = 3.141592653

    const (
        a1     string = "222222"
        a2     int    = 2
        a3, a4 string = "1", "2"
    )
    const a5, a6 = "111", 1

    const (
        e1 = "aaa"
        e2 // const里面 只定义标识符 使用上一行的常量标识符的值来进行赋值
        e3
        e4
        e5 = 1
        e6
        e7
    )

    fmt.Println(version, funcVersion, pi)
    fmt.Println(a1, a2, a3, a4, a5, a6)
    fmt.Println(e1, e2, e3, e4, e5, e6, e7)
}

```

### const实现枚举

```go
// const + iota实现枚举

package main

import "fmt"

func main() {
    const (
        e1 = iota // 0  iota 在小括号内初始化为0 每调用一次+1
        e2        // 1
        e3        // 2
    )

    // enum 枚举 e.g: 星期几 1、2、3、4、5、6、7
    // iota 结合const使用
    const (
        a1 = iota // 0  在每个小括号内都会初始化
        a2
        a3
    )

    fmt.Println(e1, e2, e3)
    fmt.Println(a1, a2, a3)
}

```

### 计算赋值

```go
package main

import "fmt"

func main() {
    // 变量赋值：可以通过任意表达式计算结果赋值
    var a1 = 10
    var a2 = a1 + 10
    var a3 = a1 + a2 // 变量/常量赋值 除了字面量 还可以是表达式

    // 常量赋值：只能通过字面量/字面量计算结果/常量计算结果/常量经过某些函数计算的结果 赋值
    const e1 = 10
    const e2 = e1 + 10
    const e3 = e1 * e2

    var a4 = a3 + e3
    // const a4 = a3 + e3  // 报错 a3是变量的计算结果

    fmt.Println(a3)
    fmt.Println(e3)
    fmt.Println(a4)
}

```

### 简短声明

```go
package main

import "fmt"

func main() {
    // var xx int

    // 简短声明：通过值来推导标识符类型
    // 问题：
    // 简短声明只能用在局部
    // 多种数据类型使用同一个字面量标识 只能推导为默认类型
    // 比如：整数 byte int int8 int16 int32 int 64 unit
    name := "minho"
    id := 1 // 只能默认推导为int 如果想定义id为int64 不能使用简短类型
    fmt.Println(name, id)
}

```

## 数据类型

### 基础数据类型

#### 布尔类型

```go
含义：        表示真假
类型名(标识符)：bool
初始化零值：   false
内存占用：     1个字节
值：          true(真)/false(假)

逻辑运算操作：
  与 && -> expr1 && expr2
  或 || -> expr1 || expr2
  非 ！ -> !expr
  
关系运算：
  等于    -> ==
  不等于  -> !=

```

#### 整数类型

```go
整数: 
    1字节：0000 0000
    有符号：可正/可负 -> 首位： 1负 0正
          000 0000 -2^(n-1) ~ 2^(n-1) - 1

    无符号：只表示>0
          0000 0000
          0 ~ 2^n - 1
          0 ~ 255
 
    int/uint     // 32位机器=>4字节 64位机器=>8字节
    byte         // 1字节  字节类型  ascii字符 -> 整数
    rune         // 32字节 码点     unicode编码表 字符 -> 整数
    int8/uint8   // 1字节 u开头的 无符号
    int16/uint16 // 2字节
    int32/uint32 // 4字节
    int64/uint64 // 8字节
    uintptr      // 32位机器=>4字节 64位机器=>8字节

零值：0
字面量: 各种进制表示
操作：
  - 算术运算 + - * / % ++ --
  - 关系运算 > < <= >=
  - 赋值运算 = += -+ *= /= %=

int + int64 -> 强制类型转换：int(xx1) + xx2
               小范围 -> 大范围 肯定无问题
               大范围 -> 小范围 可能被截断
```

#### 浮点类型

```go
// 小数
类型名：
  float32 -> 32位(4字节)
  float64 -> 64位(8字节)
零值：0.0 -> 0
字面量：十进制的小数
操作：
  - 算术运算 + - * / ++ -- 
  - 关系运算 > < <= >= 
  - 浮点数：等于/不等于 |a-b|<精度
  - 赋值运算 = += -+ *= /=

// %f 修饰
// %n.mf => n占位 m小数点保留位数(精度)

```

#### 复数类型

```go
// 略
```

#### 字符串类型

```go
string
零值："" 空字符串
字面量：
  - 可解析字符串： "双引号" 
  - 原生字符串： `反引号`

特殊字符：
    \t tab
    \r 回车
    \n 换行
    ...

操作：
  字符串连接： +
  关系运算：
    > >= < <= == !=
    从左边第一个元素开始比较(byte) 不能确定继续比较下一个操作
  赋值运算：
    =
    += 
  索引操作 => byte
  切片操作 => byte

```

```go
package main

import "fmt"

func main() {
    var desc string = "a\nb"
    var desc2 string = "a\\nb"
    var raw string = `a\nb` // 原生字符串可以包含多行
    var raw2 string = `
    a\nb
    fffds
    adadw
    weqejnfjsag
    `

    fmt.Println(desc)
    fmt.Println(desc2)
    fmt.Println(raw)
    fmt.Println(raw2)

    s := "我叫" + "minho"
    // 索引： 0，1，2：我 3，4，5：叫 6：m ...
    // 索引操作和切片操作 一定要注意字符串的编码
    // 切片的时候 如果是非ascii码的话 可能会产生乱码

    fmt.Println(s)
    fmt.Println(s[1])
    fmt.Printf("%T, %c\n", s[6], s[6])
    fmt.Println(s[0:5]) // 我�� 乱码
    fmt.Printf("%s\n", raw)
}

$ go run string.go 
>>> a
>>> b
>>> a\nb
>>> a\nb
>>>
        a\nb       
        fffds      
        adadw      
        weqejnfjsag

>>> 我叫minho
>>> 136
>>> uint8, m
>>> 我�� 
>>> a\nb

```

```go
// 重要: 索引和切片的时候 自己保证编码内容为ascii
package main

import (
    "fmt"
    "unicode/utf8"
)

func main() {
    txt := "我爱中华人名共和国!"

    // 获取字符串长度 len => 字节数量
    fmt.Println(len(txt))                    // 27
    fmt.Println(utf8.RuneCountInString(txt)) // 10 获取字符长度

    // 索引 切片
    fmt.Printf("%c\n", txt[27])
    // txt[27] = "%" // 字符串不可变 不能赋值

    // 遍历 for range
    for key, value := range txt {
        fmt.Printf("%d: %T, %c\n", key, value, value)
    }

    // 切片 byte []byte
    // 切片 rune []rune
    bs := []byte(txt) // 类型转换 类似 int('32') 字节数量
    rs := []rune(txt) // 字符的数量 可以转换为rune切片 再计算length
    fmt.Println(bs)
    fmt.Println(rs)

    // rune/byte 切片转字符串类型
    fmt.Println(string(bs))
    fmt.Println(string(rs))
}

```

#### 类型转换

```go
// 类型转换
// "123" -> int
// int -> string
// "12.2" -> float64
// float -> string

包：strconv

```

#### 指针类型

```go
package main

import "fmt"

func main() {
    var num int = 65

    num1 := num

    // 修改
    num1 = 165

    // 对num有无影响
    fmt.Println(num, num1) // 65 165

}

/*
函数/方法

调用函数 参数传递会拷贝(赋值过程) 在函数内部对拷贝后的对象修改 不影响外部
但是：真的想在函数内部修改传递的参数 如何解决？

解决: 定义一个变量 存储对应变量的内存地址
pointerNum = 0x0005  // 存储num的内存地址
此时 通过pointerNum直接去修改内存中的数据

1. pointerNum是什么类型？ -> 统称为指针类型
    int -> *int
    string -> *string
    type -> *type

2. 如何拿到num的地址？ -> 取引用
    &varname

3. 如果通过pointerNum去操作num对应内存中的数据？ -> 解引用 赋值
    *varname = xxx
*/

```

```go
package main

import "fmt"

func main() {
    var num int = 65

    num1 := num

    // 修改
    num1 = 165

    // 对num有无影响
    fmt.Println(num, num1) // 65 165

    var pointNum *int                          // 定义
    fmt.Printf("%T, %v\n", pointNum, pointNum) // *int, <nil>

    pointNum = &num                            // 取引用（取内存地址赋值）
    fmt.Printf("%T, %v\n", pointNum, pointNum) // *int, 0xc00001a098

    *pointNum = 116  // pointNum是指针类型 可以直接修改值
    fmt.Println(num) // 116

    // 简短声明
    pointNum2 := &num // 两个point指针存储的都是num的内存地址
    fmt.Printf("%T, %v\n", pointNum2, pointNum2) // *int, 0xc00001a098
    
    *pointNum2 = 5757 // pointNum2指针指向num的内存地址 修改的是num的值
    fmt.Printf("%T, %v\n", num, num) // *int, 5757
    
    fmt.Println(*pointNum, *pointNum2) // 5757, 5757 都指向num的值
    
    // 先定义变量 -> 取引用(取地址)
    // new(T) 不用定义变量
    pointer := new(int)
    fmt.Printf("%T, %v, %v\n", pointer, pointer, *pointer) // *int, 0xc00001a110, 0
    fmt.Printf("%p\n", pointer) // 占位
}

$ go run pointer.go 
>>> 65 165
>>> *int, <nil>       
>>> *int, 0xc00001a098
>>> 116
>>> *int, 0xc00001a098

// 了解：usafe包

```

### 复合数据类型

#### 数组(array)

- 序列
  由相同类型元素组成的一个有序的集

```go
0123456
"abcdefg"
...

```

- 数组
  由相同类型(任意类型)元素组成的一个**固定长度**的有序的集(一组固定长度的序列)

```go
声明： [length]T
      [3]int  // 和下面的数组[5]int类型不同 只是元素类型相同都是int
      [5]int
零值：由N个T类型的零值组成的数组(初始化)

```

```go
package main

import (
    "fmt"
    "unsafe"
)

func main() {
    var nums [10]int
    fmt.Printf("%T, %v\n", nums, nums) // [10]int, [0 0 0 0 0 0 0 0 0 0]

    var names [10]string
    fmt.Printf("%T, %q\n", names, names) // [10]string, ["" "" "" "" "" "" "" "" "" ""]

    // 字面量 初始化值
    nums = [10]int{1, 2, 3}
    fmt.Println(nums) // [1 2 3 0 0 0 0 0 0 0]

    // 通过索引指定值
    nums = [10]int{1: 1, 3: 2, 5: 3}
    fmt.Println(nums)

    // ... 语法糖 => go编译过程中进行转换
    // 简写 根据{}中的元素 自动推导length
    nums = [...]int{1, 2, 9: 3}
    fmt.Printf("%T, %v\n", nums, nums) // [10]int, [0 0 0 0 0 0 0 0 0 0]

    // 内存: int字节数量 * 元素的数量
    // 64位 8字节 * 10个元素
    fmt.Println(unsafe.Sizeof(nums)) // 80

    // 操作
    // 算数运算 无
    // 关系运算: ==  !=
    nums2 := [10]int{1, 2}
    fmt.Println(nums == nums2) // false 每个元素都相等才相等 长度不一样则类型不同 不能比较
    nums3 := [10]int{1, 2}
    fmt.Println(nums3 == nums2) // true

    // 相关函数
    // 长度
    fmt.Println(len(nums3)) // 10
    // 索引 通过索引获取每个元素的值
    // 左->右：0, 1, 2, ..., len(array) - 1; 无反向索引(负索引)
    fmt.Println(nums[1], nums[9])

    // 遍历数组元素
    for i := 0; i < len(nums); i++ {
        fmt.Printf("key：%v value: %v\n", i, nums[i])
    }

    // for range遍历
    for index, value := range nums {
        fmt.Println(index, value)
    }
    // 如果不使用index 用空白标识符 否则变量不使用 编译报错
    // var value int
    for _, value := range nums {
        fmt.Println(value)
    }
    
    // 修改元素
    fmt.Println(nums)
    nums[2] = 100 // 直接通过索引赋值
    fmt.Println(nums)
}

```

##### 多维数组

```go
package main

import "fmt"

func main() {
    // [length]T
    // T 可以是任意类型
    // T -> [5]int
    // 多维数组：数组本身的元素也是一个数组类型
    var multi [3][5]int // T = [5]int => 二维数组
    fmt.Printf("%T, %v\n", multi, multi)

    multi = [...][5]int{ // 也可以省略长度
        {1, 3, 5, 6, 7}, // 1: [5]int{1, 2, 3, 4, 5} 大括号里面定义二维 可以省略类型
        {2, 3, 4},
        {1: 2, 4: 7},
    }

    // 修改
    fmt.Println(multi)
    fmt.Println(multi[1])
    fmt.Println(multi[0][3])

    multi[1] = [5]int{1, 1, 1, 1, 1}
    fmt.Println(multi)
    multi[0][3] = 1000
    fmt.Println(multi)

    // 遍历
    for i := 0; i < len(multi); i++ {
        for j := 0; j < len(multi[i]); j++ {
            fmt.Println(i, j, multi[i][j])
        }
    }

    // range遍历
    for index, value := range multi {
        for index2, value2 := range value {
            fmt.Println(index, index2, value2)
        }
    }
}

```

#### 切片(slice)

切片：由相同类型元素组成的**长度可变**的有续集
声明：[]T  // 数组去掉长度

```go
package main

import "fmt"

func main() {
    var nums []int // var nums []int = nil

    // nil切片 赋值为nil
    fmt.Printf("%T, %v, %v\n", nums, nums, nums == nil) // []int, [], true

    // 内存 暂时不关注

    // 字面量
    nums = []int{}                                      // 空切片 不等于nil
    fmt.Printf("%T, %v, %v\n", nums, nums, nums == nil) // []int, [], false

    nums = []int{1, 2, 3, 4}
    fmt.Println(nums)
    nums = []int{1, 2, 3, 4, 5, 6, 7} // 长度可变
    fmt.Println(nums)

    nums = []int{1: 1, 10: 11} // [0 1 0 0 0 0 0 0 0 0 11]
    fmt.Println(nums)

    // make([]int, length)
    // length: 切片元素的数量
    // 赋值有5个元素的int类型零值组成的切片
    nums = make([]int, 5)
    fmt.Println(nums)

    // make([]int, length, cap)
    // length: 元素数量
    // cap: 容量 => 底层数组的长度
    // slice底层 => 数组存储 数组长度固定 当length == cap 再添加本素 重新申请内存 拷贝 释放旧数组内存
    // 切片长度增长(元素数量) 增长 => 底层数组长度不够怎么办 => 重新生成一个新的可以容纳更多元素的数组(拷贝原来数组中的元素)

    // 底层数组原长度：10 增加一个元素 增长：10 底层数组现长度：20
    // 选择的元素：11 剩9个没有使用

    nums = make([]int, 3, 10)
    fmt.Println(nums)

    // 切片操作 => 数组 字符串 切片
    numArray := [5]int{1, 3, 5, 7, 9}
    fmt.Printf("%T, %v\n", numArray[1:3], numArray[1:3])
    // [start:end] start <= end <= length

    // 数组切片赋值给切片
    nums = numArray[1:3]
    fmt.Println(nums)

    // 容量 cap()
    nums = numArray[0:4]
    fmt.Printf("%T, %v, %v, %v\n", nums, nums, len(nums), cap(nums)) // []int, [1 3 5 7], 4, 5
    nums = numArray[1:]
    fmt.Printf("%T, %v, %v, %v\n", nums, nums, len(nums), cap(nums)) // []int, [3 5 7 9], 4, 4

    // 切片的切片操作
    fmt.Printf("%T, %v\n", nums[1:3], nums[1:3])

    // 切片的赋值方式
    // 1. 零值 nil切片
    // 2. 字面量
    // 3. make函数(指定容量 不指定容量)
    // 4. 切片操作(数组切片 切片切片)

    // 操作：不能进行== 和 !=运算
    // 函数： len cap append copy

    // 元素的访问和修改： 通过索引 slice[i];slice[i] = value
    // 切片操作：
    // slice[start:end:cap_end] start <= end <= cap_end <= cap(slice)
    // array[start:end:cap_end] start <= end <= cap_end <= len(array)

    // 切片底层：
    //   数组地址
    //     容量
    //     长度

    // 长度：len() 容量：cap()
    nums = []int{}
    fmt.Println(len(nums), cap(nums))
    nums = []int{1, 2, 3}
    fmt.Println(len(nums), cap(nums))
    nums = make([]int, 10)
    fmt.Println(len(nums), cap(nums)) // 10, 10
    nums = make([]int, 0, 10)
    fmt.Println(len(nums), cap(nums)) // 0, 10

    nums = numArray[1:3]  // cap = length - start
    fmt.Println(numArray) // [1 3 5 7 9]
    fmt.Println(len(nums), cap(nums))

    // 元素的访问和修改
    fmt.Println(nums)
    nums[0] = 100 // 索引的范围：0,length-1 or 0,cap-1?
    // fmt.Println(nums[1])
    // fmt.Println(nums[2]) // 运行时报错 索引范围是0，length-1

    // 遍历
    for i := 0; i < len(nums); i++ {
        fmt.Printf("%v: %v\n", i, nums[i])
    }
    for index, value := range nums {
        fmt.Printf("%v: %v\n", index, value)
    }

    // 添加元素
    nums = append(nums, 1) // 末尾追加
    fmt.Println(nums)
    nums = append(nums, 100, 2, 3) // 追加多个元素
    fmt.Println(nums)

    // 删除元素
    // go中没有直接删除元素的方法 需要用到： 切片 + 解包

    // 删除索引为0的(首)
    nums = nums[1:len(nums)]
    fmt.Println(nums)

    // 删除索引为length-1的(尾)
    nums = nums[:len(nums)-1]
    fmt.Println(nums)

    // 删除中间的(索引为i)
    // [0:i] + [i+1:len(nums)]

    // 1. 循环append
    // prefix := nums[0:1]
    // suffix := nums[2:len(nums)]
    // fmt.Println(prefix, suffix)
    // for _, value := range suffix {
    //     prefix = append(prefix, value)
    // }
    // fmt.Println(prefix)

    // 2. 解包方式 go把切片元素展开 语法糖
    fmt.Println(append(nums[0:1], nums[2:len(nums)]...))

    // start:end start = 0  => [:end] 省略
    // start:end end = length => [start:]

    // 容量
    // 增长规则： <= 1024  n => 2*n
    //          > 1024 n => n*(1+~0.25)
    nums = []int{}
    fmt.Println(len(nums), cap(nums)) // 0 0
    nums = append(nums, 1)
    fmt.Println(len(nums), cap(nums)) // 1 1
    nums = append(nums, 2)
    fmt.Println(len(nums), cap(nums)) // 2 2
    nums = append(nums, 3)
    fmt.Println(len(nums), cap(nums)) // 3 4
    nums = append(nums, 4)
    fmt.Println(len(nums), cap(nums)) // 4 4
    nums = append(nums, 5)
    fmt.Println(len(nums), cap(nums)) // 5 8
    // 如果可以预期切片的最大长度 可以通过make函数指定cap容量 减少申请内存的损耗

    // start:end:cap_end
    numArray = [5]int{1, 3, 5, 7, 8}
    nums = numArray[1:3] // len = end - start; cap = len(底层数组) - start

    nums = append(nums, 1000) // 切片会共享底层数组
    fmt.Println(numArray)     // [1 3 5 1000 8]
    fmt.Println(nums)         // [3 5 1000]

    fmt.Println("#########")
    nums = numArray[3:5] // [1000 8]
    nums = append(nums, 3)
    fmt.Println(numArray) // [1 3 5 1000 8]
    fmt.Println(nums)     // [100 8 3]

    // [start:end:cap_end] 切片可以指定cap的位置(限制cap)
    // end <= cap_end <= 底层数组的长度
    nums = numArray[1:3:4]            // cap = cap_end - start
    fmt.Println(len(nums), cap(nums)) // 2 2
    nums = append(nums, 999)
    fmt.Println(nums)
    fmt.Println(numArray)
    // len和cap一样 append会触发扩容 扩容不会影响原来的底层数组 生成新的底层数组
    // 否则 会影响底层数组(改变底层数组的值)

    // 总结：如果使用数组来生成切片 那么操作切片(append扩容)根据容量的情况 会影响底层数组 或是生成新的底层数组

    // 切片的切片操作
    // [start:end] start <= end <= cap(nums); end小于等于容量cap
    fmt.Println("###########")
    nums = make([]int, 3, 10)
    nums[0] = 1
    nums[1] = 2
    nums[2] = 3
    fmt.Println(nums)

    numsSlice := nums[1:5] // [2 3 0 0]
    fmt.Println(numsSlice)
    fmt.Println(len(numsSlice), cap(numsSlice)) // 1, 9 仍然共享底层数组
    numsSlice = append(numsSlice, 100)
    fmt.Println(nums) // numsSlice扩容了一个元素100 nums值改变 [1 2 3] => [1 2 100] 原因：共享了底层数组

    // 限定cap
    numsSlice = nums[1:2:2]
    fmt.Println(numsSlice)
    fmt.Println(len(numsSlice), cap(numsSlice))

    // copy函数(dst, src)  后面的切片拷贝到前面的切片
    // copy：只赋值索引相同的元素 复制值不够：保留  复制只太多：剔除 并不扩容
    dst := make([]int, 3)
    src := []int{2, 3, 4}

    // len(src) == len(dst)
    fmt.Println(src, dst)
    copy(dst, src)
    fmt.Println(src, dst) // [2 3 4] [2 3 4]

    // len(dst) > len(src) 保留未被覆盖部分
    src = []int{200, 300}
    copy(dst, src)
    fmt.Println(src, dst) // [200 300] [200 300 4]

    // len(dst) < len(src)
    src = []int{1000, 2000, 3000, 4000}
    copy(dst, src)
    fmt.Println(src, dst) // [1000 2000 3000 4000] [1000 2000 3000] 长度并不会在拷贝过程中进行调整

    // 通过copy来删除索引为i的元素
    nums = []int{1, 2, 3, 4, 5}
    // 删除索引为2的
    fmt.Println(nums[2:]) // [3 4 5]
    fmt.Println(nums[3:]) // [4 5]
    copy(nums[2:], nums[3:])
    fmt.Println(nums[:len(nums)-1]) // [1 2 4 5 5]
}

```

##### 切片实现队列

```go
// 队列： 先进先出(消息队列)
    quene := []int{}
    1 2 3 => 1 2 3
    进切片(添加到后面)
        append(1)
        append(2)
        append(3)
    出切片
        获取索引为0的元素 删除索引为0的元素 quene[1:]

package main

import "fmt"

func main() {
    quene := []string{}

    var txt string

    for {
        fmt.Println("请输入任务(exit退出,do执行):")
        fmt.Scan(&txt)
        if txt == "exit" {
            break
        } else if txt == "do" {
            if len(quene) == 0 {
                fmt.Println("无任务")
            } else {
                fmt.Println("执行：", quene[0])
                quene = quene[1:]
            }
        } else {
            quene = append(quene, txt)
        }
    }
}

```

##### 切片实现堆栈

```go
// 堆栈： 先进后出 (有优先级的场景)
    stack := []int{}
    1 2 3 => 3 2 1
    进切片：
        append(1)
        append(2)
        append(3)
    出切片：
        获取索引为len()-1的元素 删除len()-1的元素 stack[:len(stack)-1]

package main

import "fmt"

func main() {
    stack := []string{}

    var txt string

    for {
        fmt.Println("请输入任务(exit退出,do执行):")
        fmt.Scan(&txt)
        if txt == "exit" {
            break
        } else if txt == "do" {
            if len(stack) == 0 {
                fmt.Println("无任务")
            } else {
                fmt.Println("执行：", stack[len(stack)-1])
                stack = stack[:len(stack)-1]
            }
        } else {
            stack = append(stack, txt)
        }
    }
}

// 切片 []T
// T => [3]int  数组
[][3]int
// T => []int   切片
[][]int

```

##### 循环删除切片多个元素

```go
package main

import "fmt"

func main() {
    sli := []int{1, 2, 3, 4, 5, 6, 7, 8, 9, 10}

    // 使用for循环遍历切片 并记录需要删除的元素的索引
    indexesToDelete := []int{}
    for i, v := range sli { 
        if v%2 == 0 {
            indexesToDelete = append(indexesToDelete, i)
        }
    }

    // 倒序遍历需要删除的元素的索引 因为删除元素后 切片的长度会发生变化
    // 如果正序遍历 则可能产生索引越界
    for i := len(indexesToDelete) - 1; i >= 0; i-- {
        sli = append(sli[:indexesToDelete[i]], sli[indexesToDelete[i]+1:]...)
    }

    fmt.Println(sli) // [1 3 5 7 9]
}

```

#### 映射(map)

```go
// 切片 或者 数组 访问元素是直接通过索引访问
// 如果我要找一个值 不知道索引的情况下 那就得把所有元素都遍历一遍
slice[i]
array[i]

映射：// 定义key和value对的集
    // (可能是有序的/可能是无序的 Go中是无序的：放入key的顺序与在内存中的顺序无关 与遍历无关)
    含义： ID => 数据
    映射实现方式：hashtable(go的实现方式) treemap

key:
    通过key对value进行增删查改
    key是唯一的
    key类型的要求：可以进行 == !=判断 (不能为切片)
    value类型任意

定义：map[keyType]valueType
赋值：
    make:
        make(map[keyType]valueType)
    字面量：
        map[keyType]valueType{}    // 空map
        map[keyType]valueType{k1:v, k2:v, k3:v}
    零值：nil
操作：
    元素数量：len()
    元素操作：
        获取：mapName[key]
            key存在：mapName[key] => 对应value值
            key不存在：不会报错 返回对应valueType类型的零值 因此 不能通过零值来判断key否存在
            如何判断key是否存在：
                value, ok := mapName[key]  // 可以返回两个值 ok表示key是否存在 true/false
                value := mapName[key]
        增：
            mapName[key] = value // 存在修改 不存在增加
        改：
            mapName[key] = value 
        删：
            delete(mapName, key)
```

```go
package main

import "fmt"

func main() {
    // name => count 记录每个学生上课次数
    // key: string  value: int
    var stats map[string]int
    fmt.Printf("%T, %v, %v\n", stats, stats, stats == nil)

    // 赋值
    stats = map[string]int{} // 空map 不等于nil
    fmt.Printf("%T, %v, %v\n", stats, stats, stats == nil)

    stats = map[string]int{"minho": 3, "kk": 10}
    fmt.Printf("%T, %v, %v\n", stats, stats, stats == nil)
    fmt.Println(len(stats))

    // stats = make(map[string]int)
    // fmt.Printf("%T, %v, %v\n", stats, stats, stats == nil)

    // 获取元素
    fmt.Println(stats["minho"])
    fmt.Println(stats["minho11"]) // key不存在 不报错

    value := stats["minho"]
    fmt.Println(value)
    value = stats["minho11"]
    fmt.Println(value)

    value, ok := stats["minho"] // 返回两个值 ok表示是否存在 true/false
    fmt.Println(value, ok)
    value, ok = stats["minho11"]
    fmt.Println(value, ok)

    // 增加元素
    stats["karubin"] = 11
    fmt.Println(stats)

    // 修改元素
    stats["karubin"] = 9
    fmt.Println(stats)

    // 删除元素
    stats["xxx"] = 7
    fmt.Println(stats)
    delete(stats, "xxx")
    delete(stats, "xxxxxx") // 删除的key如果不存在 不报错
    fmt.Println(stats)

    // 遍历 for range
    for key := range stats {
        fmt.Println(key, stats[key])
    }

    for key, value := range stats {
        fmt.Println(key, value)
    }
}

```

#### 显示初始化值

```go
package main

import "fmt"

func main() {
    var nilSlice []int = make([]int, 0)
    var nilMap map[string]int = make(map[string]int)

    // 显式为切片和map进行初始化值
    fmt.Println(append(nilSlice, 1)) // [1]
    fmt.Println(nilMap["kk"])
    nilMap["kk"] = 1 // 没有初始化值 报错 nil map
}

```

## 函数

为代码片段定义一个名称

### 功能

```go
1. 代码功能明确
2. 代码复用

```

### 定义/调用

```go
func 函数名称(函数参数, ...) 返回值 {
    // 函数体
}

```

#### 函数名称

```go
// 需要满足标识符规范
```

#### 函数参数(形参)

```go
// 可以是任意多个
// 参数：参数名称(变量名) 参数类型
// 定义参数 就是定义一个变量
// 形参作用域：只能在该函数体内使用
// 多个参数使用逗号分割
// go里面没有参数默认值
// 多个连续参数类型一致 只保留连续参数中最后一个参数类型 前面n个参数类型省略

// 可变参数

package main

import "fmt"

// 无参
func sayHello() {
    fmt.Println("Hello, Go")
}

// 有参
func sayHi(name string) {
    fmt.Println("Hi", name)
}

func add(num1 int, num2 int) (int, string) {
    fmt.Println(num1 + num2)
    return num1 + num2, "add"
}

// 连续同类型参数省略类型
func merge(a1 bool, a2, a3, a4 string) {
    fmt.Println(a1, a2, a3, a4)
}

// 可变参数
// 变量名 ...T
// 可变参数在一个函数内只能定义一次
func anyArgs(args ...string) { // 可以接收任意多个string类型的变量
    // args是切片类型
}

// 可变参数可以和固定参数结合使用 但只能放在最末尾
func addN(left, right int, args ...int) {
    sum := left + right
    for _, value := range args {
        sum += value
    }
    fmt.Println(sum)
}

func main() {
    addN(1, 2, 3, 4, 5, 6, 7)
}
```

#### 返回值

```go
// 可以是任意多个 
// 无返回值不定义
// 有一个返回值： 需定义返回值的类型
// 多个返回值：需定义所有返回值的类型 用逗号分割 ()包含 连续相同返回类型 同样可以省略
// 返回值可以定义名称(命名返回值) 命名返回值 建议在函数简单的时候使用

// 有返回值一定要用return关键字返回

package main

import "fmt"

// 无返回值
func sayHello() {
    fmt.Println("Hello, Go")
}

// 单个返回值
func add() int {
    return 2 + 3
}

// 多个返回值 同类型
func calc() (int, int, int, int) {
    return 2 + 3, 2 - 3, 2 * 3, 2 / 3
}

// 返回多个值 不同类型
func returnType() (int, bool, string, []int) {
    return 1, true, "string_return", nil
}

// 命名返回值
// 将函数执行完成时的name返回
func returnName() (name string) {
    // return // 命名返回值 可以直接只写return 返回""
    name = "returnName"
    return
}

func returnNameArgs() (name string, isBoby bool) {
    name = name + "Minho"
    return
}

func main() {
    sayHello()

    r := add()
    fmt.Println(r)

    num1, num2, _, num4 := calc()
    fmt.Println(num1, num2, num4)

    res1, res2, res3, res4 := returnType()
    fmt.Println(res1, res2, res3, res4)

    res := returnName()
    fmt.Printf("%q\n", res)

    name, isBody := returnNameArgs()
    fmt.Printf("%q, %v\n", name, isBody) // Minho false
}
```

#### 函数调用

```go
// 函数名称(实参参数) 实参的数量 == 形参的数量 
// 实参和形参：数量 需要一致 / 类型按顺序一致 / 数据按照顺序传递



// 函数调用 在栈上 一般是申请的连续的地址
// 闭包使用到的变量 go会存到堆中 该类变量生命周期边长
go tool compile -m -l xxx.go

var xxx does not escape
var xxx escapes to head // 逃逸到堆中

```

### 递归

```go
// 函数间接/直接调用自己
// 分治问题：将一个大问题拆分成N个相同解决方法的小问题


// 阶乘
package main

import "fmt"

// 递归实现阶乘
func fact(n int64) int64 {
    fmt.Println("call", n)
    if n == 0 || n == 1 {
        return 1 // 终止条件 没有的话会无限调用 调用深度(看语言层面是否有限制)
    }
    rt := n * fact(n-1)
    fmt.Println("result", n, rt)
    return rt
}

func main() {
    fmt.Println(fact(5)) // 120
}

$ go run fact.go // 先进去的后得到结果 类似栈
>>> call 5
>>> call 4
>>> call 3
>>> call 2
>>> call 1
>>> result 2 2
>>> result 3 6
>>> result 4 24
>>> result 5 120

```

```go
// 汉诺塔
package main

import "fmt"

// 将layer个盘子从t1移动到t3 借助t2
func tower(t1, t2, t3 string, layer int) {
    if layer == 1 {
        fmt.Printf("%s -> %s\n", t1, t3)
        return
    }

    // 先将layer-1个从t1 移动到t2 借助t3
    tower(t1, t3, t2, layer-1)
    // t1 -> t3
    fmt.Printf("%s -> %s\n", t1, t3)

    // 将layer-1个t2 移动到t3 借助t1
    tower(t2, t1, t3, layer-1)
}

func main() {
    tower("A", "B", "C", 3)
}

```

### 函数类型

```go
// 定义函数类型的变量 赋值 函数, 调用 => 其他语言的高阶函数

package main

import "fmt"

func add(l, r int) int {
    return l + r
}

func mapFunc(list []int, tl func(int) int) []int {
    rt := []int{}
    for _, value := range list {
        rt = append(rt, tl(value))
    }
    return rt
}

func tlAdd(n int) int {
    return n + 5
}

func main() {
    fmt.Printf("%T\n", add) // func(int, int) int

    // 定义函数类型的变量
    var f func(int, int) int // 零值 nil
    f = add                  // 引用函数add
    fmt.Printf("%T %v\n", f, f)
    fmt.Println(f(2, 3)) // 调用函数 5

    // map filter reduce
    // 对切片进行操作
    // map: 对切片中每个元素通过某种转换得到结果组成新的切片
    // filter: 对切片的元素进行过滤
    // reduce: 初始化元素 将1个元素与初始化元素 => result + 2 => result + 3

    nums := []int{1, 2, 3, 4, 5}
    // map => 新的切片 []int
    fmt.Println(mapFunc(nums, tlAdd))
}

```

### 匿名函数

```go
package main

import "fmt"

func mapFunc(list []int, tl func(int) int) []int {
    rt := []int{}
    for _, value := range list {
        rt = append(rt, tl(value))
    }
    return rt
}

func main() {
    // 匿名函数：没有名字的函数 定义后直接赋值给某个变量 直接调用
    // 往往做一些辅助性的功能(临时一个地方使用) 缩短作用域

    // 匿名结构体
    // 匿名接口

    add := func(left, right int) int {
        return left + right
    }

    fmt.Printf("%T, %v\n", add, add)
    fmt.Println(add(5, 5))

    // tlAdd5 := func(n int) int {
    //     return n + 5
    // }
    nums := []int{1, 2, 3, 4, 5}
    fmt.Println(mapFunc(nums, func(n int) int { return n + 10 }))

    x := func(left, right int) int {
        return left + right
    }(5, 5)
    fmt.Println(x)

    func(left, right int) int { // 不要返回值
        fmt.Println(left, right)
        return left + right
    }(11, 11)
}

```

### 闭包

```go
// 在函数内部 会定义匿名函数 该匿名函数会引用外层函数的变量
package main

import "fmt"

// 生成一个新的函数 每次调用生成一个ID => 从start开始 id的间隔是step
// 面向切面编程 装饰器

// generateId执行完成后 start step不能被销毁 闭包(内层匿名函数使用了外层函数的局部变量)使他们的生命周期变长
func generateId(start, step int) func() int {
    return func() int {
        fmt.Printf("start: %v\n", start)
        id := start
        start += step
        return id
    }
}

func main() {
    g1 := generateId(0, 1)
    fmt.Println(g1())
    fmt.Println(g1())
    fmt.Println(g1())
    fmt.Println(g1())
    fmt.Println(g1())

    g100 := generateId(100, 100)
    fmt.Println(g100())
    fmt.Println(g100())
    fmt.Println(g100())
    fmt.Println(g100())
}

// 闭包：改变了变量的生命周期和作用域
// 变量的声明周期：从变量创建到销毁的过程
// 变量的作用域：变量可以使用的范围

```

### 类型

```go
var t T
tmp := t

// 对tmp进行修改 可能影响到t => 引用类型
// 反之 任何操作都不能影响t => 值类型
```

#### 值类型

```go
int
string
bool
[...]int // 数组 也是值类型
结构体
指针 pointer // 有争议 为了降低函数参数拷贝带来的内存翻倍 会使用指针进行传递

package main

import "fmt"

func main() {
    nums := [...]int{1, 2, 3, 4, 5}
    nums2 := nums

    fmt.Println(nums, nums2) // [1 2 3 4 5] [1 2 3 4 5]
    nums2[0] = 1000
    fmt.Println(nums, nums2) // [1 2 3 4 5] [1000 2 3 4 5]
}

```

#### 引用类型

```go
// 切片
// 映射

// 使用了指针 或 内存地址
```

### 值传递

```go
形参 = 实参  
// 把实参 赋值给了实参 => 把实参的内存地址中的数据复制一份 赋值给形参
// 形参是实参的副本
// 在函数内对形参修改 不会对外面的实参有影响
// go中 不管值类型 还是引用类型 都会把实参对应内存地址的值复制一份 复制给形参

// 注意：当传递的是 引用类型的时候 比如切片 因为共享底层数组的关系 在函数内修改索引的值 会影响底层数组的值 从而影响实参的值
```

### 错误处理

```go
// go标准建议显式的将错误信息通过返回值返回给调用者 由调用者自行处理
func() (xxx1, xxx2, error)

package main

import (
    "errors"
    "fmt"
    "strconv"
)

/*
错误：
    语法错误 => 编译失败
    运行时 =>
        可恢复错误/不可恢复错误

        网络问题失败
        读取文件失败
        写文件磁盘空间不足
        ...
*/

func fact(n int64) (int64, error) {
    if n < 0 {
        return 0, errors.New("计算错误")
    }
    if n == 1 || n == 0 {
        return 1, nil
    }
    r, err := fact(n - 1)
    if err == nil {
        return n * r, nil
    }
    return 0, fmt.Errorf("计算错误")
}

func main() {
    var i int
    var err error
    // i, err := strconv.Atoi("1.1.1.1")
    i, err = strconv.Atoi("1.1.1.1")
    fmt.Println(i, err)
    fmt.Printf("%T %v\n", err, err)
}
```

#### panic抛出错误

```go
package main

func fact(n int64) int64 {
    if n < 0 {
        panic("n不能小于0")
    }
    if n == 1 || n == 0 {
        return 1
    }
    rt := n * fact(n-1)
    return rt
}

func main() {
    fact(-1)
}

```

#### 捕获错误

```go
// recover关键字 => 需要放置在延迟执行内部
package main

import "fmt"

func fact(n int64) int64 {
    if n < 0 {
        panic("n不能小于0")
    }
    if n == 1 || n == 0 {
        return 1
    }
    rt := n * fact(n-1)
    return rt
}

func main() {
    // defer 函数调用
    defer func() {
        if errMsg := recover(); errMsg != nil {
            fmt.Println(errMsg) // 相当于捕获错误
        }
    }()
    fmt.Println(fact(1))
}

```

```go
// 函数将错误返回
package main

import (
    "fmt"
)

func fact(n int64) int64 {
    if n < 0 {
        panic("n不能小于0")
    }
    if n == 1 || n == 0 {
        return 1
    }
    rt := n * fact(n-1)
    return rt
}

func testError() (err error) {
    // defer 函数调用
    defer func() {
        if errMsg := recover(); errMsg != nil {
            err = fmt.Errorf("%v", errMsg)
        }
    }()
    fmt.Println(fact(1))
    return
}

func main() {
    fmt.Println(testError())
}

```

#### 延迟执行

```go
// 执行函数 不管内部逻辑是否发生错误 都想要执行的代码
// 关键字：defer 后面必须跟函数调用
package main

import "fmt"

func fact(n int64) int64 {
    if n < 0 {
        panic("n不能小于0")
    }
    if n == 1 || n == 0 {
        return 1
    }
    rt := n * fact(n-1)
    return rt
}

func main() {
    // defer 函数调用
    defer fmt.Println("main") // 不管是否有错误 都在所在的函数退出的时候执行
    defer func() {
        fmt.Println("defer 1")
    }()
    defer func() {
        fmt.Println("defer 2")
    }()
    defer func() {
        fmt.Println("defer 3")
    }()
    fmt.Println(fact(1))
}

// $ go run panic.go // 同一个函数内多个defer 先定义的后执行
// 1
// defer 3
// defer 2
// defer 1
// main

```

#### defer注意事项

```go
// defer内修改值
package main

import "fmt"

func test() (i int) {
    defer func() {
        // defer 函数退出前执行 这个时候返回2
        // 除了error场景 不要在defer中使用修改返回值
        i = 2
    }()

    i = 1
    return i
}

func main() {
    fmt.Println(test())
}

```

```go
// 闭包陷阱
package main

import "fmt"

func main() {
    fmt.Println("start")
    for i := 0; i < 3; i++ { // 函数退出的时候 i的值为3 闭包陷阱
        defer func() {
            fmt.Println(i)
        }()
    }
    fmt.Println("end")
}

// $ go run defer2.go
// start
// end
// 3
// 3
// 3

// 注意：defer不要写在循环之内

```

## 作用域

- 作用域内定义变量只能被声明一次且变量必须使用 否则编译错误
- 在不同作用域可定义相同的变量 此时局部将覆盖全局

```go
package main

import "fmt"

var version string = "1.1.1"

func main() { // main作用域

    // 作用域： 变量的可使用范围
    // {}显示的声明一个作用域范围 - 嵌套{}可以想象为一个树状图
    // 1. 在某个作用域范围内定义的变量 可以在其作用域每使用；子孙定义的 父看不见
    // 2. 若标识符在某个作用域中有定义 若子孙某层作用域内重新定义 则被覆盖
    // 对内穿透 就近原则 对外不可见(类比Python作用域记忆)

    fmt.Println("version1", version) // 1.1.1
    var funcVersion string = "2.2.2"

    var version string = "1.2.3"
    fmt.Println("version2", version) // 1.2.3

    //  funcVersion string = "2.3.4"  // 报错 同一块作用域 不能重新定义
    { // A{}块
        fmt.Println("version3", version) // 1.2.3
        version = "AAAAA"                // 这里直接修改的 main.version 重新赋值变量
        fmt.Println("version4", version) // AAAAA
    }

    fmt.Println("version5", version) // AAAAA

    { // B{}块
        fmt.Println("version6", version) // AAAA  main.version
        var version string = "BBBBB"     // 在B块作用域内重新定义了一个version 没有改变main.version
        fmt.Println("version7", version) // BBBBB
    }

    fmt.Println("version8", version) // AAAAA

    fmt.Println(version, funcVersion)
}

$ go run scope.go 
>>> version1 1.1.1
>>> version2 1.2.3
>>> version3 1.2.3
>>> version4 AAAAA
>>> version5 AAAAA
>>> version6 AAAAA
>>> version7 BBBBB
>>> version8 AAAAA
>>> AAAAA 2.2.2

```

### 显式作用域

```go
显式声明作用域：
  {} 大括号 

```

## 隐式作用域

```go
隐式声明作用域：
  1. 全局作用域
  2. 包作用域
  3. 文件作用域
  4. if switch for select case语句块

```

## 流程控制

### if

```go
if expr1 {
    cmd1
} else if expr2 {
    cmd2
    ...
} else if exprn {
    cmdn
} else {
    default_cmd
}

// if必须有 
// else if任意多个
// esle 0个或1个

```

### switch

```go
package main

import "fmt"

func main() {
    var value string
    fmt.Scan(&value) // 从console获取值

    switch value {
    case "sg":
        fmt.Println("gre2")
    case "us":
        fmt.Println("gre3")
    case "tw":
        fmt.Println("gre4")
    default:
        fmt.Println("gre1")
    }
}

```

### for

```go
package main

import "fmt"

func main() {
    // var s string = "My name is Minho"

    // for init; expr; end {}
    // 1 + 2 + ... + 100
    var total int = 0
    for i := 1; i <= 100; i++ {
        total += i
    }
    fmt.Println(total)

    total = 0
    var i int = 1
    for i <= 100 { // 类while写法
        total += i
        i++
    }
    fmt.Println(total)

    // var nums int = 0
    // for { // 死循环 类似于python： while True: cmd
    //     nums++
    //     fmt.Println(nums)
    // }

    // break
    for i := 1; i < 5; i++ {
        if i == 3 {
            break // 结束循环
        }
        fmt.Println(i)
    }
    // continue
    for i := 1; i < 5; i++ {
        if i == 3 {
            continue // 跳过本次循环 下次继续
        }
        fmt.Println(i)
    }
}

// continue break只能跳出或结束当前循环
// 配合label使用 可以跳出到指定label
BEXIT:
for i := 0; i < 3; i++ {
    for j := 0; j < 3; j++ {
        if j == 1 {
            break BEXIT // 直接跳出for循环
        }
    }
}

CEXIT:
for i := 0; i < 3; i++ {
    for j := 0; j < 3; j++ {
        if j == 1 {
            continue CEXIT // 直接跳出到for循环
        }
    }
}
```

### goto

```go
package main

import "fmt"

func main() {
    total, incr := 0, 0

START: // label表示  标识符:
    total += incr
    incr++

    if incr <= 100 {
        goto START
    }
    goto END
    fmt.Println(total) // 此时 永远无法执行这个print语句

END:
    fmt.Println("total:", total)
}

```

## 单步调试工具 dlv

```bash
$ dlv debug xxx.go  # help查看完整帮助

# vscode有图形界面可以调试 debug模式

```

## 包管理/模块

```go
// 包 模块
代码main => 分函数 => 分文件 => 分文件夹 // 代码整理(项目的组织方式)

第三方别人开发的包 -> github.com
  -> 下载
  -> GOPATH src

go工具：
  go build
  go install 第三方包地址
  go get 第三方包地址 github.com/ilolicon/xxx  // 下载第三方包
     git/svn

第三方包查找地址：
  https://godoc.org/
  https://gowalker.org/

变量可见性：
  首字母大小写的问题
    首字母小写的变量只对包内可见
    首字母大写才对外部的其他包可见(可以调用)

// 包
包名：在同一个文件夹内 包名必须一致
main包：可编译为可执行的结果(二进制程序) package 定义为 main
功能代码：若功能不需要编译为可执行程序 -> 命名为非main -> 可以任意定义 -> 标识符规范(一般使用文件夹名称)
若不在同一个包下面调用不同文件中定义的函数
  导入：
    - package包名和目录一致：导入目录路径(GOPATH src后面的目录路径) import "cmdb/user"
    - package包名和目录不一致：import "cmdb/idc" 注意 导入仍是路径 但是这个调用的包名则不一致
  调用：通过包名.函数名称

// GO111MODULE
GO111MODULE=on/off/auto
on   -> 启动module
off  -> 关闭module -> gopath + vendor
auto -> 自动判断
        依据：代码不再GOPATH目录 目录存在go.mod文件 -> 都满足为module 否则为gopath+vendor方式
```

### GOPATH + vendor

```go
方案1：1.11 之前 gopath + vendor方式/其他开源方案
gopath:
  -> GOPATH(配置一个或者多个目录) // GOPATH=C:\Users\97431\go
     // 代码必须放在GOPATH目录下
     src -> 源码
     bin -> 放二进制文件
     pkg -> 第三方包(经过编译过的.a文件) // go install xx的时候会生成相关依赖的.a文件 -> 编译快

vendor: 主要解决多版本问题 优先调用vendor目录下面的版本(代码)
vendor：可以放在任何目录 查找的时候：
        当前目录查找vendor(无vendor/有vendor无包) -> 上一级目录 -> ... -> GOPATH vendor -> GOPATH -> 标准库 -> 报错

// 代码放在GOPATH目录下 可以在任意目录build
go build . || go build user(GOPATH src下面的目录名)

// cp $WORK\b001\exe\a.out.exe C:\GoWorkspace\go\bin\user.exe 
// 将程序编译并拷贝到GOPATH的bin目录下
go install -x user(GOPATH src下面的目录名) 


// 导入包
/*
GOPATH/
  cmdb/
    user/
      main.go  // 都在user包里面 调用user.go里面的函数 直接使用函数名称调用即可
      user.go  // 都在user包里面 
*/
package main

func main() {
    // 在同一个包下面调用不同文件中定义的函数 -> 直接调用函数名称
    add()
    modify()
    delete()
    find()
}

------------------------------

/*
GOPATH/
  cmdb/
    user/
      user.go  // 功能代码
    idc/
      idc.go  // 功能代码
    main.go
*/
package main

import (
    "cmdb/user" // GOPATH src后面的目录路径
)

func main() {
    // 在不同包下面调用不同文件中定义的函数 -> 先导入包 -> 通道包名.函数名称
    // 但是这里会报错 => 变量可见性问题 标识符首字符小写 只能包内使用 不能导出
    user.add()
    user.modify()
    user.delete()
    user.find()
}

```

### MODULE

```go
方案2：1.11及之后 GOMODULE(重点) // GO 1.11 MODULE

优势：
  - 不用设置GOPATH 代码可以任意放置
  - 自动下载依赖管理
  - 版本控制
  - 不允许使用相对导入
  - replace机制(网络问题goproxy 和 包重命名问题 现在很少使用)

GO111MODULE
  // 初始化模块
  go mod init 模块名称 // go mod init cmdb
  go mod tidy // 下载依赖的第三方模块
  执行：go run .
       go build .
 
  // 模块名称命名
  1. 远程仓库的命名称 cmdb -> github.com/ilolicon/cmdb // 简单 不能作为第三方包使用
  2. 带上仓库地址：git mod init github.com/ilolicon/cmdb // github地址 + 用户名 + 账号名称
     指定版本：依赖tag git/svn -> tag v0.x.x v1.x.x
     下载指定版本：go get github.com/ilolicon/math@v1.0.0 或者手动修改go.mod文件 重新go mod tidy

// GO111MODULE 导入包 同模块
package main

import (
    // 同模块 -> modulename(go mod init初始化时给出的模块名称)/目录路径
    // go.mod文本文件 依赖 模块名称 可以直接修改
    "cmdbxx/user"
    "fmt"
)

func main() {
    fmt.Println("Hello, World")
    user.Add()
}

-----------------------

package main

import (
    "cmdb/user" // 同模块 -> modulename/目录路径
    "fmt"

    "github.com/beego/beego/v2/server/web" // 使用第三方模块 go mod tidy
)

func main() {
    fmt.Println("Hello, World")
    user.Add()
    web.Run()
}


// 导入远程第三方包
package main

import (
    "fmt"

       // go mod tidy 会自动去下载
    "github.com/ilolicon/math"
)

func main() {
    fmt.Println(math.PI)
    fmt.Println(math.Add(5, 7))
}

// math包设置 模块名以 github.com + 用户名 + 模块名 设置
// go mod init github.com/ilolicon/math
package math

const PI float32 = 3.141592653

func Add(l, r int) int {
    return l + r
}

// 包名冲突 使用别名解决
// 别名导入
import (
    testmod/aaa/math"  // 绝对路径导入 -> 还有相对路径导入 gomod禁用
    bmath "testmod/bbb/math" // 别名直接在导入包前面设置即可
)

func main() {
    math.Add(1, 1)
    bmath.Add(1, 1)
}

// 点导入
import (
    . "testmod/aaa/math" // 点导入 调用包内函数的变量 不用加包名 尽量不用 减少变量污染
)

func main() {
    Add(1, 1)
}

// 下划线导入
// 包导入不使用 会报错 可以使用下划线导入
// 不使用还导入 有什么作用？ -> 包初始化 执行包内初始化函数 func init() {}
import (
    _ "testmod/aaa/math"  // 别名导入的特殊形式 注册机制会用到
)

```

## 导入总结

```go
/* 
导入包
 绝对路径导入
   GOPATH src下的目录
   gomod 同模块 模块名称/路径
         第三方模块 第三方模块名称/路径
 别名导入
   导入多个相同名称的包
   
 下划线导入
   包初始化 init 不要依赖他的顺序
   
 相对路径导入
   gomod中禁用
   
 点导入
   尽量不用 容易变量冲突 污染
*/

```

### init函数

```go
// 导入包 会自动调用该包内的init函数
// 一个包内可以有任意多个 可以定义无数次 但建议只写1个

func init() {
    cmd // go定义的初始化函数 没有参数 没有返回值 包被导入的时候 自动执行
}

```

#### 可见性

```go
// 只能在包内可用：包内可见 变量名称小写
// 可以在包外使用：包外可见 变量名称大写

```

## 标准库及第三方库

[标准库官网文档](https://golang.google.cn/pkg/)

[Golang标准库中文文档](https://studygolang.com/pkgdoc)

### fmt

```go
fmt.Printf() 

/*
格式化的字符串 占位 后面的数据进行填充
  占位符
    %T %v %#V 与数据类型无关 打印变量类型 %v %#V 以可识别的格式打印
    %s 字符串
    %d 整数
    %t 布尔值true|false
*/

package main

import "fmt"

func main() {
    var name = "minho"
    var id = 1
    fmt.Printf("name: %T, id: %T\n", name, id)
    fmt.Printf("name: %v, id: %v\n", name, id)
    fmt.Printf("name: %#v, id: %#v\n", name, id)
    fmt.Printf("name: %s, id: %d\n", name, id)
}

$ go run debug.go 
>>> name: string, id: int
>>> name: minho, id: 1   
>>> name: "minho", id: 1
>>> name: minho, id: 1

```

- 输入输出

```go
// 输出
fmt.Println()
fmt.Print()
fmt.Printf -> 占位符

// 输入(从控制台获取输入)
fmt.Scan  

// 先定义比变量
// fmt.Scan(&变量名)

```

### strconv

```go
// import "strconv"
// strconv包实现了基本数据类型和其字符串表示的相互转换
package main

import (
    "fmt"
    "strconv"
)

func main() {
    // i, err := strconv.Atoi("10")
    // if err != nil {
    //     panic(err)
    // }
    // fmt.Printf("%T, %v\n", i, i)

    // 上面部分可以写在一行 if放在最前面 这样变量i err就成了if块的局部变量
    if i, err := strconv.Atoi("10"); err != nil { // string => int
        panic(err)
    } else {
        fmt.Printf("%T, %v\n", i, i)
    }

    // int => string
    fmt.Printf("%T, %v\n", strconv.Itoa(10), strconv.Itoa(10))

    // string => float64
    f, err := strconv.ParseFloat("12.2", 64)
    if err != nil {
        panic(err)
    }
    fmt.Printf("%T, %v\n", f, f)

    // float64 => string
    fmt.Println(strconv.FormatFloat(12.2, 'E', -1, 64))
    fmt.Println(strconv.FormatFloat(12.2, 'f', -1, 64))

    // string => int8 int16 int32 int64 -> ParseInt ParseUint
    // int8 int16 int32 int64 => string -> FormatInt FormatUint
}

```

### strings

[用于操作字符的简单函数](https://golang.google.cn/pkg/strings/)

```go
package main

import (
    "fmt"
    "strings"
)

func main() {
    fmt.Println(strings.Compare("abc", "ade"))        // -1
    fmt.Println(strings.Contains("abc", "ab"))        // true
    fmt.Println(strings.Count("abcabcabc", "ab"))     // 3
    fmt.Println(strings.EqualFold("ab", "AB"))        // true 比较 忽略大小写
    fmt.Println(strings.Fields("a b\tc\r\nd\fe"))     // [a b c d e]
    fmt.Println(strings.HasPrefix("test.go", "test")) // true
    fmt.Println(strings.HasSuffix("test.go", ".exe")) // false
    fmt.Println(strings.Index("1abcabc", "ab"))       // 1 字符在字符串中出现的位置(索引) 没找到返回-1
    fmt.Println(strings.LastIndex("1abcabc", "ab"))   // 4

    // 连接与分割
    fmt.Println(strings.Join([]string{"a", "b", "c"}, "-")) // a-b-c
    fmt.Println(strings.Split("a-b-c", "-"))                // 分割为切片
    fmt.Println(strings.SplitN("a-b-c", "-", 2))            // [a b-c]

    fmt.Println(strings.Repeat("*", 20))
    fmt.Println(strings.Replace("abc,abc,abcd, abd", "abc", "xyz", -1)) // -1 == ReplaceAll

    fmt.Println(strings.Trim("bcaAbcdeacb", "abc")) // Abcde 左右都移除
    // TrimLeft/TrimRight 和 TrimSuffix/TrimPreffix 区别
    // TrimLeft/TrimRight 把给出的字符串中的每一个字符 都去目标字符串中移除
    // TrimSuffix/TrimPreffix 把给出的字符串当作一个整体 去目标字符串中移除
    fmt.Println(strings.TrimSpace(" abc abc\r\n\f")) // abc abc 去除前后空白字符
}


// 类比 bytes包 一个对字符串操作 一个对字节切片操作

```

### sync

#### sync.atomic

```go
// 原子操作
原子操作是指过程中不能中断的操作s Go语言sync/atomic包中提供了五类原子操作函数 其操作对象为整数型或整数指针

- Add* 增加/减少s
- Load* 载入
- Store* 存储
- Swap* 更新
- CompareAndSwap* 比较第一个参数引用值是否与第二个参数值相同 若相同则将第一个参数值更新为第三个参数

```

```go
// 对共享内存操作 - 加锁
package main

import (
    "fmt"
    "sync"
)

func main() {
    var counter int
    var wg sync.WaitGroup
    var locker sync.Mutex

    wg.Add(10)
    for i := 0; i < 5; i++ {
        go func() {
            for x := 0; x < 10000; x++ {
                locker.Lock()
                counter += 1
                locker.Unlock()
            }
            wg.Done()
        }()
        go func() {
            for x := 0; x < 10000; x++ {
                locker.Lock()
                counter -= 1
                locker.Unlock()
            }
            wg.Done()
        }()
    }

    wg.Wait()
    fmt.Println(counter)
    // 不加go关键字使用协程 结果为0
    // 使用go启动协程之后 对同一个内存(&counter)进行操作 会有冲突 需要加锁 或 原子操作
}

```

```go
// 原子操作 操作类型有限 使用较少
package main

import (
    "fmt"
    "sync"
    "sync/atomic"
)

func main() {
    var counter int64
    var wg sync.WaitGroup

    wg.Add(10)
    for i := 0; i < 5; i++ {
        go func() {
            for x := 0; x < 10000; x++ {
                atomic.AddInt64(&counter, 1)
            }
            wg.Done()
        }()
        go func() {
            for x := 0; x < 10000; x++ {
                atomic.AddInt64(&counter, -1)
            }
            wg.Done()
        }()
    }

    wg.Wait()
    fmt.Println(counter)
}

```

#### sync.Mutex/RWMutex

```go
// sync.Mutex 互斥锁
// sync.RWMutex 读写锁
// obj => 写write: g1 write; g2 write 需要加锁 不能同时写 互斥 => 互斥锁
// obj => 读read: g1 read; g2 write 正在写不让读 正在读不让写 => 互斥锁
// obj => 读read： g1 read; g2 read 正在读被其他读(可以允许 如果使用Mutex 有锁不会让其他协程读) => 读写锁

```

#### sync.Map

```go
// Go的内置类型 map 不是线程安全的
// sync.Map是线程安全的 但操作的类型都是接口类型 需要进行断言(类型转换)

```

#### sync.Once

```go
// 控制程序只执行一次 包括并发的情况下
package main

import (
    "fmt"
    "sync"
)

func conf() {
    fmt.Println("conf")
}

func main() {
    var once sync.Once
    // once.Do(conf)
    // once.Do(conf)
    // once.Do(conf)

    var wg sync.WaitGroup
    for i := 0; i < 10; i++ {
        wg.Add(1)
        go func() {
            once.Do(conf)
            // conf()
            wg.Done()
        }()
    }
    wg.Wait()
}

```

#### sync.Pool

```go
// sync.Pool
// 需要资源 从池子里面获取 当池子中没有会进行创建并返回 有则直接复用 用完后归还 资源的重复利用
// 对象创建时耗费资源

// 网络连接池
// 数据库连接池
package main

import (
    "fmt"
    "sync"
)

func main() {
    objPool := sync.Pool{
        New: func() interface{} {
            fmt.Println("new")
            return "123456789"
        },
    }

    obj := objPool.Get()
    fmt.Println(obj)
    obj2 := objPool.Get()
    fmt.Println(obj2)
    objPool.Put(obj)

    obj3 := objPool.Get()
    fmt.Println(obj3) // 直接打印123..9 不会调用打印new
}

// 自定义连接池结构体
type TCPPool struct {
    addr    string
    pool    *sync.Pool
    max        int // 池子中保持的 最大连接数 超过报错
    idle    int // 池子中闲置的连接数量
    min     int // 池子中最小的连接数
    poolCnt int // 池子的连接数量
    count     int // 总的连接数量
    locker     *sync.Mutex
}
```

### runtime

```go
// go doc runtime go运行时的一些配置信息

func GOMAXPROCS(n int) int // 配置GMP模型中 P的数量
func GOROOT() string // 获取Go的安装目录
func Gosched() // 让出时间片
func NumCPU() int // 获取CPU数量
func NumGoroutine() int // 获取协程数量

```

### logr

#### logrus

```go
package logger

import (
    "fmt"
    "path"
    "runtime"
    "strconv"

    log "github.com/sirupsen/logrus"
    "gopkg.in/natefinch/lumberjack.v2"
)

func RotateLogger(logName string) *lumberjack.Logger {
    cfg := global.CFG.Log
    return &lumberjack.Logger{
        Filename:   logName,
        MaxSize:    cfg.MaxSize,
        MaxBackups: cfg.MaxBackups,
        LocalTime:  cfg.LocalTime,
        Compress:   cfg.Compress,
    }
}

func Logger(cfg *config.Config) *log.Logger {
    if cfg.ServerLog == "" || cfg.MaxSize == 0 {
        log.Fatal("invalid configurations, please check")
    }

    l := RotateLogger(cfg.ServerLog)
    logger := log.New()
    logger.SetOutput(l)
    level, err := log.ParseLevel(cfg.LogLevel)
    if err != nil {
        log.Fatalf("invalid log level: %s", cfg.LogLevel)
    }
    logger.SetLevel(level)
    logger.SetReportCaller(cfg.ReportCaller)
    logger.SetFormatter(&log.TextFormatter{
        DisableColors:   true,
        TimestampFormat: "2006-01-02 15:04:05",
        CallerPrettyfier: func(f *runtime.Frame) (function string, file string) {
            return "", fmt.Sprintf("%s:%s", path.Base(f.File), strconv.Itoa(f.Line))
        },
    })

    if err := l.Close(); err != nil {
        log.Fatal(err)
    }
    return logger
}

```

#### zap

```go
package logger

import (
    "os"
    "time"

    "go.uber.org/zap"
    "go.uber.org/zap/zapcore"
    "gopkg.in/natefinch/lumberjack.v2"
)

func Logger(cfg *config.Config) *zap.Logger {
    var (
        customLevelEncoder = func(level zapcore.Level, encoder zapcore.PrimitiveArrayEncoder) {
            encoder.AppendString(level.String())
        }
        customTimeEncoder = func(t time.Time, encoder zapcore.PrimitiveArrayEncoder) {
            encoder.AppendString(t.Format("2006-01-02 15:04:05.000"))
        }
    )

    encoderConfig := zapcore.EncoderConfig{
        MessageKey:     "msg",
        LevelKey:       "level",
        TimeKey:        "time",
        NameKey:        "logger",
        CallerKey:      "caller",
        StacktraceKey:  "stacktrace",
        LineEnding:     zapcore.DefaultLineEnding,
        EncodeLevel:    customLevelEncoder,
        EncodeTime:     customTimeEncoder,
        EncodeDuration: zapcore.SecondsDurationEncoder,
        EncodeCaller:   zapcore.ShortCallerEncoder,
    }
    var encoder zapcore.Encoder
    if cfg.Log.JsonFormat {
        encoder = zapcore.NewJSONEncoder(encoderConfig)
    } else {
        encoder = zapcore.NewConsoleEncoder(encoderConfig)
    }

    // 日志轮询
    lumberjackLogger := &lumberjack.Logger{
        Filename:   cfg.Log.Filename,
        MaxSize:    cfg.Log.MaxSize,
        MaxAge:     cfg.Log.MaxAge,
        MaxBackups: cfg.Log.MaxBackups,
        Compress:   cfg.Log.Compress,
    }
    var syncer zapcore.WriteSyncer
    if cfg.Log.LogInConsole {
        syncer = zapcore.NewMultiWriteSyncer(zapcore.AddSync(os.Stdout), zapcore.AddSync(lumberjackLogger))
    } else {
        syncer = zapcore.AddSync(lumberjackLogger)
    }

    core := zapcore.NewCore(encoder, syncer, cfg.Log.UnmarshalLevel())
    logger := zap.New(core)
    if cfg.Log.ShowLine {
        logger = logger.WithOptions(zap.AddCaller())
    }
    return logger
}

```

## 面向对象

### 自定义类型

```go
- 想要自己定义一个由不同类型元素(属性 字段)组成一个复合类型 - 结构体

别名&重新定义
  - 别名 与原有类型相同 无区别 不能定义新的方法
  - 重新定义：将现有的类型进行重新定义 对新的类型提供一些方法 与原有类型的类型无关 具有原有类型特性 但是可以与原有类型进行相互转换 如：计数功能 int -> 计数类型; http包里的Header type Header map[string][]string

面向对象：
  代码复用 => 继承 组合
  Go: 嵌入 => 组合模式

```

### 定义

```go
// 定义的关键字
var   // 变量定义
const // 常量定义
func  // 函数定义
type  // 自定义类型定义

type TypeName Formatter
重定义：type Counter int
结构体：type User struct {
    属性名称1 属性类型1
    属性名称2 属性类型2
    属性名称3 属性类型3
    ...
}
别名：type TypeName = Formatter
  type byte = uint8
  type rune = int32

```

### 别名与重定义

```go
package main

import "fmt"

type Counter = int
type Counter2 int

func main() {
    var count Counter
    count = 10
    fmt.Printf("%T\n", count) // int

    var count2 Counter2
    fmt.Printf("%T %v\n", count2, count2) // main.Counter2 0

    count2 += Counter2(count) // 可以与原类型进行类型转换
    fmt.Println(count2)
}

```

### 结构体

```go
package main

import (
    "fmt"
    "time"
)

// 定义结构体 => 类 => 封装
/*
type StructName struct {
    Field1 FieldType1
    Field2 FieldType2
    ...
}

type User struct {
    id         int
    name       string
    addr       string
    tel        string
    birthday   string/uinxtime/time.Time 首选time
    created_at string/uinxtime/time.Time
    updated_at time.Time
    flag       bool // 标识逻辑删除
    status     int // 1删除 0未删除
    deleted_at *time.Time // 已删除: 删除时间 非nil 未删除：nil 指针零值
}
*/

type User struct {
    id        int
    name      string
    addr      string
    tel       string
    brithday  time.Time
    createdAt time.Time
    updatedAt time.Time
    flag      bool
    status    int
    deletedAt *time.Time
}

// 类 构造器 => 创建对应类的实例(对象/变量) // Go中无构造器的说法
// 函数 N(n)ew 结构体名称 构建结构体的(值/指针)对象

func NewUser(id int, name, addr, tel string, birtday time.Time) User {
    return User{
        id:        id,
        name:      name,
        addr:      addr,
        tel:       tel,
        brithday:  birtday,
        createdAt: time.Now(),
        updatedAt: time.Now(),
        flag:      false,
        status:    0,
        deletedAt: nil,
    }
}

// 返回指针对象
func NewUserPoint(id int, name, addr, tel string, birtday time.Time) *User {
    return &User{
        id:        id,
        name:      name,
        addr:      addr,
        tel:       tel,
        brithday:  birtday,
        createdAt: time.Now(),
        updatedAt: time.Now(),
        flag:      false,
        status:    0,
        deletedAt: nil,
    }
}

func main() {
    // 定义User类型的变量
    var u User
    fmt.Printf("%T\n", u) // main.User

    // 零值 => 所有属性的零值组成的一个User类型的变量值
    fmt.Printf("%v\n", u) // 非nil
    fmt.Printf("%#v\n", u)

    // 赋值 字面量
    // 设置值 按照结构体中属性定义额顺序设置 -> 需要给所有属性都进行赋值
    u = User{1, "minho", "成都", "13424244434", time.Now(), time.Now(), time.Now(), false, 0, nil}
    fmt.Printf("%#v\n", u)

    // 指定名称赋值 未指定的属性名称 对应类型的零值进行初始化
    u = User{id: 2, name: "KATHR1NE"}
    fmt.Printf("%#v\n", u)

    // 属性进行访问和修改
    fmt.Println(u.id, u.name)
    u.addr = "北京"
    fmt.Printf("%#v\n", u)

    // 结构体 值类型 赋值修改不影响原来的变量

    // 指针类型的对象
    var u3 *User
    fmt.Printf("%T, %v\n", u3, u3)
    // 指针赋值 => 取引用 &
    // new 申请空间并对元素使用0值进行初始化 取地址 赋值
    u3 = &u
    fmt.Printf("%#v\n", u3)

    var u4 *User = new(User)
    fmt.Printf("%#v\n", u4)

    // 字面量取引用
    var u5 *User = &User{}  // User{} User的零值
    fmt.Printf("%#v\n", u5) // 与 new(User)一样

    fmt.Println("################")
    u6 := NewUser(10, "lemon", "", "", time.Now())
    u7 := NewUserPoint(11, "kathr1ne", "", "", time.Now())
    fmt.Printf("%T %v\n", u6, u6)
    fmt.Printf("%T %v\n", u7, u7)
}

```

### 匿名结构体

```go
// 匿名结构体 => 不定义名称 => 不能再定义对应的变量 => 只需要定义该种结构的一个变量

package main

func main() {
    var user struct {
        id int
        name string
    }
    fmt.Printf("%T\n", user)  // struct {id: int name: string} 
    fmt.Printf("%#v\n", user) // 对应类型的零值
}

// 使用：
// 程序的配置
// template => 数据(匿名结构体) => 传递给模板

```

### new 和 make的区别

```go
// new make
make => 创建对象
    slice,map,chan
new  => 返回指针 创建结构体指针对象
    new 基本数据类型
    var i int = 1
    p := &i

    p := new(int)

    new(Struct) == &Struct{} => New

```

### 结构体嵌套(嵌入/组合)

```go
package main

import "fmt"

// 地址结构体
type Address struct {
    Province string
    City     string
    Street   string
}

// 定义用户信息
type User struct {
    Id    int
    Name  string
    Tel   string
    Email string
    Addr  Address // 嵌入Address结构体 即定义Address结构体的属性 - 命名嵌入
    PAddr *Address
}

func main() {
    // 1. 定义
    var u User
    var addr Address = Address{Province: "四川", City: "成都", Street: "高新"}
    
    // 2. 赋值
    u = User{Id: 1, Name: "Minho", Tel: "13308047517", Email: "97431110@qq.com", Addr: addr, PAddr: &addr}
    fmt.Printf("%#v\n", u)
    
    // 3. 属性的访问和修改
    fmt.Println(u.Id)
    fmt.Printf("%#v\n", u.Addr)
    fmt.Printf("%T, %#v\n", u.PAddr, u.PAddr)
    
    u.Id = 100
    u.Addr = Address{"北京", "北京", "海定区"}
    u.PAddr = &Address{"北京", "北京", "昌平区"}
    fmt.Println(u.Addr)
    fmt.Printf("%T, %#v\n", u.PAddr, u.PAddr)
    
    u.Addr.Street = "安静区"
    fmt.Println(u.Addr)
    
    u.PAddr.Street = "PP区"
    fmt.Println(u.PAddr.Street)
}

```

```go
// 匿名嵌入 不写属性名称 直接写类型
package main

type Address struct {
    Provice string
    City    string
    Street    string 
}

type User struct {
    Id        int
    Name    string
    Tel        string
    Address // 匿名嵌入 自定义属性名Address(与类型名一致) 只能匿名嵌入一次
    Street    string // 属性名与匿名嵌入对象属性名相同
}

type Company struct {
    Name        string
    Province    string
}

// 匿名嵌入两个对象 两个对象中有相同的属性名
type User2 struct {
    Id        int
    Name    string
    Tel        string
    Street     string
    Address
    Company
}

func main() {
    // 定义赋值访问 和命名嵌入基本一致
    // 不一样的地方
    // 当访问匿名嵌入对象的属性时可以省略嵌入对象的属性名
    var u User
    fmt.Println(u.Street) // 优先查找当前对象 若无再查找匿名对象中的属性名
    
    var u2 User2
    fmt.Println(u2.Provice) // 多个匿名嵌入对象具有相同属性名 Go直接报错 语言层面 不知道选择哪个(二义性) Go限制开发者使用全路径    
}

// 建议
// 1. 不使用匿名嵌入
// 2. 使用匿名嵌入 访问属性的时候 使用全路径

// 可以匿名嵌入指针对象

```

### 方法

#### 定义

```go
// 方法：给特定类型定义的函数 只能由特定的类型进行调用

// 函数定义
func funcName(args) returns {}

// 方法比函数多出 => 接收者
// 接收者：指定函数属于哪个类型 只能由对应的类型进行调用
// 除了接收者 方法的参数/返回值 与go中的函数完全一致

// 方法定义
func (receiver) methodName(args) returns {}

// 接收者也是一个参数： t T 参数类型 方法所属的类型 该方法只能由T类型的对象进行调用
// T 自定义类型
// t 变量名 在对象调用方法时用来传递调用的对象

1. 值接收者：t T
2. 指针接收者：t *T
3. 方法内不需要访问/修改t对象的属性 可以把t省略 保留类型T即可

```

```go
package main

import "fmt"

type User struct {
    id    int
    name  string
    tel   string
    email string
}

// 定义User的方法 返回用户名
func (u User) GetName() string {
    // u: 调用者的值拷贝
    return "GetName:" + u.name
}

// 修改名称的方法
func (u User) SetName(name string) {
    u.name = name
}

// 指针接收者
func (u *User) PsetName(name string) {
    u.name = name
}

func (User) No() {
    fmt.Println("No")
}

func main() {
    u := User{name: "minho"}
    fmt.Println(u.GetName()) // 调用方法 对象.方法名称(参数)
    u.SetName("karuin")
    fmt.Println(u.GetName()) // GetName:minho

    // 调用PsetName
    // 语法糖：如果接收者时指针接收者 调用可以使用值对象 => Go自动进行取引用
    u.PsetName("lemon")      // => (&u).PsetName("lemon")
    fmt.Println(u.GetName()) // GetName:lemon

    p := &User{name: "MM"}
    // 语法糖 接收者是值接收者调用指针对象 => 自动进行解引用
    fmt.Println(p.GetName()) // fmt.Println((*p).GetName())
    p.PsetName("QQ")
    fmt.Println(p.GetName())

    p.No()
}

type Counter = int // 别名不能定义方法

// 重定义可以定义方法
type Counter int

func (c *Counter) Incr() {
    (*c)++
}
```

#### 可见性

```go
// 与属性相同
// 首字母大写 => 包外可见
// 首字母小写 => 包内可见
```

#### 方法表达式

```go
package main

type User struct {
    Name string
}

func (u User) GetName() string {
    return u.Name
}

func (u User) SetName(name string) {
    u.Name = name
}

func (u *User) PsetName(name string) {
    u.Name = name
}

func main() {
    // 定义 => T类型
    // 调用 => T(值/指针)变量/对象/实例
    // 访问方法
      // T.方法名
      // 定义对象 对象.方法名
    
    // 方法表达式
    // 值接收者类型
    f := User.GetName
    fmt.Printf("%T\n", f) // func(main.User) string 是一个函数类型 可以直接调用 参数为自定义User类型
    fmt.Println(f(User{Name: "minho"})) // minho
    fmr.Println(User.GetName(User{Name: "XXXX"}))
    
    f2 := User.SetName // 多个参数
    fmr.Printf("%T\n", f2) // func(main.User, string)
    
    // 指针类型接收者
    f3 := (*User).PsetName // 方法为指针接收者 调用者必须为对应类型的指针类型 User.PsetName报错
    fmt.Printf("%T\n", f3)
    
    f4 := (*User).GetName
    fmt.Printf("%T\n", f4) // func(*main.User) string
}

// 如果定义了值接收者方法 Go会自动生成一个对应的指针接收者方法

```

#### 方法值

```go
// 代码接上

u = User{Name: "Minho"}
m1 = u.GetName

fmt.Printf("%T\n", m1) // func() strirng
fmt.Println(m1()) // 加括号执行 Minho

```

## 文件操作

```go
// 相关包(常用)
// os：文件操作
// filepath：文件路径
// bufio：带缓冲区IO
// fs: 抽象包 1.15提供的
// ioutil: 提供工具函数
```

### 文件路径

```go
// 相对路径
// 绝对路径
```

### 操作

```go
// 文件内容：010101 []byte <=编码/解码=> string(utf8)

读文件
  os.Open() // 文件必须存在
    参数：路径
    返回值：*File, error
  读文件
    Read([]byte) n, err
    ReadAt([]byte, int64) n, err
  关闭文件
    Close()
```

```go
package main

import (
    "fmt"
    "io"
    "os"
)

func main() {
    path := "readfile.txt"

    file, err := os.Open(path)
    if err != nil {
        fmt.Println("error:", err)
    } else {
        defer file.Close()
        fmt.Println("success")
        // 读文件
        data := make([]byte, 3)
        for {
            n, err := file.Read(data)
            if err != nil {
                // 出错
                if err != io.EOF {
                    fmt.Println(err)
                }
                break
            } else {
                fmt.Println(data[:n], string(data[:n]))
            }
        }
        // 读取文件内容到字节切片data中
        // n:读取字节的数量 error: EOF 结束符
        // n, err := file.Read(data)

        // fmt.Println(data, n, err)
        // fmt.Println(data[:n])
        // fmt.Printf("%q, %q", string(data), string(data[:n]))

        // file.Close()
    }
}

写文件
  os.Crete() // 创建文件 go doc os.Create 
  // 文件不存在 创建; 文件存在：清空
  写：
    Write
    WriteString
    WriteAt
  关闭文件：
    Close

// copyFile
package main

import (
    "io"
    "log"
    "os"
)

func main() {
    // copyfile src dst
    if len(os.Args) != 3 {
        log.Fatal("usage: copyfile srcfile dstfile")
    }

    srcFile, dstFile := os.Args[1], os.Args[2]
    // 读取srcFile
    src, err := os.Open(srcFile)
    if err != nil {
        log.Fatal(err)
    }
    defer src.Close()

    // 写dstFile
    dst, err := os.Create(dstFile)
    if err != nil {
        log.Fatal(err)
    }
    defer dst.Close()

    bufferSize := 10
    data := make([]byte, bufferSize)
    for {
        n, err := src.Read(data)
        if err != nil {
            if err != io.EOF {
                log.Fatal(err)
            }
            break
        }
        n, err = dst.Write(data[:n])
        if err != nil {
            log.Fatal(err)
        }
    }
}


追加文件
  OpenFile
  os.OpenFile(path, os.O_CREATE|os.OWRONLY|os.O_APPEND, os.ModePerm)

文件指针
  Seek(offset, whence)
  whence
    0 文件开始位置 
    1 当前指针位置 
    2 文件末尾
  offset 偏移

  获取位置Seek(1, 0)


// 将日志和文件结合起来
package main

import (
    "log"
    "os"
)

func main() {
    file, err := os.OpenFile("web.log", os.O_CREATE|os.O_WRONLY|OS.O_APPEND, os.ModePerm)
    if err != nil {
        log.Fatal(err)
    }
    log.SetOutput(file)
    log.Print("第一条日志")
}


--
fmt.Print/Println/Printf    // 输出内容
fmt.Sprint/Sprintln/Sprintf // 将字符串作为值返回
fmt.Fprint/Fprintln/Fprintf(io.Writer, 变量/模板, 变量) // 将内容写入文件

func main() {
    file, _ := os.Create("file")
    fmt.Fprint(file, 1, 2, 3)  // 将1 2 3写入文件
    fmt.Fprintln(file, 4, 5, 6)
    fmt.Fprintf(file, "My name is %s\n", "Minho")
}

--
fmt.Scan  // 从控制台扫描(读取)变量
fmt.Fscan // 从文件扫描
fmt.Sscan // 从字符串扫描

--
// 读取目录下的子文件(有可能是一个文件 有可能是一个目录)
fmt.Println(file.Readdir(10))
fmt.Println(file.Readdirnames(10))
fmt.Println(file.Readdirnames(10)) // 10 读取10个文件或目录
fmt.Println(file.Readdirnames(-1)) // -1 读取所有文件或目录

package main

import (
    "fmt"
    "log"
    "os"
)

func main() {
    file, err := os.Open(".")
    if err != nil {
        if os.IsNotExist(err) {
            fmt.Println("文件不存在")
            return
        }
        log.Fatal(err)
    }

    defer file.Close()
    files, err := file.Readdir(-1)
    for _, f := range files {
        // 文件信息
        fmt.Println(f.IsDir(), f.Name(), f.Size(), f.ModTime(), f.Mode())
    }
}

--
file, err := os.Stat(file/dir) // reutrn FileInfo
// 判断文件是否存在
// os.Lstat => 链接文件的信息 os.Stat => 源文件的信息
os.IsExist(error)  // 参数为：error类型 返回 bool
os.IsNotExist(error)

os.Remove(file)                   // 删除文件
os.Remove(dir)                    // 删除目录
os.Removeall(dir)                 // rm -rf 目录不为空也可以删除
os.Rename("aa", "bb")             // 文件重命名
os.Chmod("aa", 0600)              // 设置权限
os.Mkdir("xxx", os.ModePerm)      // 创建目录
os.MkdirAll("x/y/z", os.ModePerm) // mkdir -p

--
io.Copy  // 复制文件
io.MultiWriter()  // 输出流(可以写数据的)
io.MultiReader()  // 输入流(可以读取数据的) 按照顺序读取所有文件中的内容

f1, _ := os.Create("f1.txt")
f2, _ := os.Create("f2.txt")
m := io.MultiWriter(f1, f2)
m.Write([]byte("abc")) // 写到多个流
```

### 目录操作

```go
// filepath
// go doc filepath

filepath.Abs(".")              // 获取绝对路径
filepath.Dir("/root/cfg.ini")  // 获取文件目录 /root
filepath.Base("/root/cfg.ini") // 获取文件名 cfg.ini
filepath.Ext("/root/cfg.ini")  // 获取文件后缀 .ini
filepath.Join("/root/mm", "xx", "ll.ini") // 拼接路径
filepath.Split("/root/cfg.ini") // 分隔path /root/ cfg.ini

filepath.Glob("./*.go") // 匹配文件 类似Unix通配符
filepath.Glob("./*/*.txt")

// filepath.Walk 遍历目录
filepath.Walk(".", func(path string, fileInfo fs.FileInfo, err error) error {return nil})

```

### 文件操作有关工具类

```go
// go doc ioutil
package main

import (
    "fmt"
    "io/ioutil"
    "os"
)

func main() {
    // data, _ := ioutil.ReadFile("alias.go")
    // fmt.Println(string(data))

    files, _ := ioutil.ReadDir(".")
    for _, info := range files {
        fmt.Println(info.Name())
    }

    ioutil.WriteFile("ioutil.txt", []byte("xxxx"), os.ModePerm)
    ioutil.WriteFile("ioutil.txt", []byte{57, 58, 59}, os.ModePerm)

    dir, _ := ioutil.TempDir(".", "temp_")
    fmt.Println(dir)

    file, _ := ioutil.TempFile(".", "tempfile_")
    defer file.Close()
    file.WriteString("xxx")
}
```

### 带缓冲区IO

```go
// 为提高性能
// 程序中定义(类似)切片 内部定义切片(指定切片的长度) 读取内容到切片 函数直接从切片中读取数据 当切片中数据读取完成后 再从硬盘读取
// 写数据类似 先写到切片 写满刷新到磁盘 可以主动刷新

bufio // go doc bufio
  Scanner
  Reader
  Writer

// Scanner
package main

import (
    "bufio"
    "fmt"
    "os"
)

func main() {
    // var txt string
    // fmt.Scan(&txt) // fmt.Scan 不带缓冲区IO 不接收空白和空格
    // fmt.Println("scan:", txt)

    // 带缓冲区IO
    // 读取字符串数据 strconv
    scanner := bufio.NewScanner(os.Stdin)
    for scanner.Scan() {
        fmt.Println("输入内容：", scanner.Text())
        fmt.Println("输入内容：", scanner.Bytes())
    }
    fmt.Println(scanner.Err()) // 查看错误信息
}

// Reader
package main

import (
    "bufio"
    "fmt"
    "os"
)

func main() {
    file, _ := os.Open("bufio.txt")

    reader := bufio.NewReader(file) // defaultBufSize = 4096 4kb
    // bufio.NewReaderSize()
    data := make([]byte, 3)
    n, _ := reader.Read(data)
    fmt.Println(data[:n]) // linux strace 查看系统调用

    n, _ = file.Read(data)
    fmt.Println(data[:n])

    // isPrefix是True的话 还需要继续读取 表示一行还没有读取完整
    // 在缓冲区中数据处理完成后若无换行就会返回isPrefix为True需要继续读取
    ctx, isPrefix, err := reader.ReadLine()
    fmt.Println(string(ctx), isPrefix, err)
}

// Writer
package main

import (
    "bufio"
    "fmt"
    "os"
)

func main() {
    file, _ := os.Create("bufio.txt")
    defer file.Close()

    writer := bufio.NewWriter(file)
    fmt.Println(writer.WriteString("abcxyz"))
    writer.Flush() // 将缓冲区数据刷新到磁盘 否则文件没用内容
}

--
// go实现tailf命令
package main

import (
    "bufio"
    "fmt"
    "io"
    "log"
    "os"
    "time"
)

func main() {
    if len(os.Args) != 2 {
        log.Fatal("usage: tailf xxx.log")
    }

    file, err := os.Open(os.Args[1])
    if err != nil {
        log.Fatal(err)
    }

    reader := bufio.NewReaderSize(file, 16)
    line := make([]byte, 0, 4096)
    for {
        data, isPrefix, err := reader.ReadLine()
        if err != nil {
            if err != io.EOF {
                log.Print(err)
                break
            }
            time.Sleep(time.Second)
            // break
        } else if isPrefix {
            line = append(line, data...)
        } else {
            if len(line) > 0 {
                fmt.Println(string(append(line, data...)))
                line = make([]byte, 0, 4096)
            } else {
                fmt.Println(string(data))
            }
        }
    }
}

// 缓存算法：淘汰时间最早的 LRU缓存
```

### 几个结构体

```go
// go doc strings
type Builder struct{ ... } // 字符串变成流对象操作
type Reader struct{ ... }  // ...
    func NewReader(s string) *Reader

// go doc bytes.Reader

// 在内存中对字节切片和字符串处理
// 拼接字符串：string.Builder byte.Buffer
// 下载文件在内存中处理：bytes.Buffer  // 不落盘
```

### 文件格式

```go
// 需要将数据落盘(磁盘存储) 网络传输
// 内存 =序列化=> []byte(string) => 反序列化=> 内存
// 编码方式：gob csv json protocolbuffer 序列化和反序列化的方案

// 序列化(编码)
// 反序列化(解码)

库：
  编码：Marshal
       Encoder
  解码：Unmarshal
       Decoder

// 用哪种类型的数据编码 就解码为对应的数据类型
gob => Go语言提供的编码 二进制编码方式
encoding/gob

```

#### gob

```go
package main

import (
    "encoding/gob"
    "fmt"
    "os"
)

// 属性名称都大写
// 序列化和反序列化需要访问属性
type User struct {
    Id   int
    Name string
    Addr string
}

func main() {
    // 非go内置的基本类型和复合数据类型 需要注册
    gob.Register(&User{})
    
    users := []*User{
        {1, "Minho", "SC"},
        {2, "lemon", "SH"},
        {3, "karubin", "GD"},
    }

    file, _ := os.Create("data.gob")

    // 编码
    encoder := gob.NewEncoder(file)
    err := encoder.Encode(users)
    fmt.Println(err)
    file.Close()

    // 解码
    file, _ = os.Open("data.gob")
    defer file.Close()

    decoder := gob.NewDecoder(file)
    user2 := make([]*User, 0)
    err = decoder.Decode(&user2)
    fmt.Println(err, user2)
    for _, u := range user2 {
        fmt.Printf("%#v\n", u)
    }
}
```

#### csv

```go
package main

import (
    "encoding/csv"
    "fmt"
    "os"
)

func main() {
    file, _ := os.Open("table.csv")
    defer file.Close()

    reader := csv.NewReader(file)
    line, err := reader.Read()
    fmt.Println(err, line)

    line, err = reader.Read()
    fmt.Println(err, line)

    line, err = reader.Read()
    fmt.Println(err, line)

    line, err = reader.Read()
    fmt.Println(err, line)

    file, _ = os.Create("table.csv")
    writer := csv.NewWriter(file)
    writer.Write([]string{"A1", "B1", "C1"})
    writer.Write([]string{"A2", "B2", "C2"})
    writer.Write([]string{"A3", "B3", "C3"})
    writer.Write([]string{"A4", "B4", "C4"})
    writer.Flush()
}
```

## 接口

```go
数据 + 行为 => 结构体属性 方法

// 都有类似的方法
os.File: read    write
strings.Builder: write
strings.Reader:  read
bytes.Buffer:    write read
bytes.Reader:    read
bufio.Writer:    write
bufio.Reader:    read
bufio.Scanner:   read

接口：对行为的抽象
行为抽象的对象 => 实现行为的对象

// 定义
  接口：定义的一些函数(0 1 n个)
  签名：函数名称 参数列表 返回值列表

// 关键字：interface
type TypeName Formatter

type InterfaceName interface {
    方法签名
}

// 断言 => 接口
// 对应类型的变量, ok := 接口类型变量.(类型)

// switch 接口类型变量.(type) { // switch p := 接口类型变量.(type)
// case GobPersistent:
// case CsvPersistent:
// case *JsonPersistent:
// case *ProtobufPersistent:
// default:
// }

// 如何实现一个可以接收任意类型变量的函数
```

### 定义/赋值

```go
package main

import "fmt"

type User struct {
    Id   int
    Name string
}

// 定义一个持久化数据接口
// 行为：Save保存User切片 Load: 加载User切片
type Persistent interface {
    Save([]User, string) error
    Load(string) ([]User, error)
}

type GobPersistent struct{}

// 定义了Persistent接口所有(一个也不能少 但是可以多)的方法 就叫Gobpersistent实现了Persistent接口

func (g GobPersistent) Save(users []User, path string) error {
    fmt.Println("gob persistent Save")
    return nil
}
func (g GobPersistent) Load(path string) ([]User, error) {
    fmt.Println("gob persistent Load")
    return nil, nil
}

type CsvPersistent struct{}

func (c CsvPersistent) Save(users []User, path string) error {
    fmt.Println("csv persistent Save")
    return nil
}
func (c CsvPersistent) Load(path string) ([]User, error) {
    fmt.Println("csv persistent Load")
    return nil, nil
}

func call(persistent Persistent) {
    fmt.Println("call:")
    persistent.Save(nil, "")
    persistent.Load("")
}

type JsonPersistent struct{}

func (j *JsonPersistent) Save(users []User, path string) error {
    fmt.Println("json persistent Save")
    return nil
}

func (j *JsonPersistent) Load(path string) ([]User, error) {
    fmt.Println("json persistent Load")
    return nil, nil
}

type ProtobufPersistent struct{}

func (p *ProtobufPersistent) Save(users []User, path string) error {
    fmt.Println("protobuf persistent Save")
    return nil
}

func (p ProtobufPersistent) Load(path string) ([]User, error) {
    fmt.Println("protobuf persistent Load")
    return nil, nil
}

func main() {
    var persistent Persistent
    // persistent := GobPersistent{}
    fmt.Printf("%T, %#v\n", persistent, persistent) // nil nil

    // 赋值/初始化
    // 接口 不能直接通过接口类型初始化对象
    // 需要使用实现行为(实现接口的所有方法 某个对象的类型定义了接口中所有的方法)的对象

    persistent = CsvPersistent{} // 如果有接口后 只需要需改实现

    fmt.Printf("%T, %#v\n", persistent, persistent) // Gobpersistent, Gobpersistent{}
    persistent.Save(nil, "")                        // => Gobpersistent->Save
    persistent.Load("")                             // Gobpersistent->Load

    call(persistent)

    // 多态 => 某个对象赋值为不同对象 体现出不同的行为
    fmt.Println("多态")
    call(GobPersistent{})
    call(CsvPersistent{})

    // 定义类型 与 接口 是否实现 无语法上直接关联
    // 鸭子类型

    // persistent为什么不直接定义为Gobpersistent？
    // 改动代码非常麻烦 特别涉及到各种Gobpersistent参数传递的时候

    // 指针对象
    persistent = new(CsvPersistent)
    fmt.Printf("%T, %#v\n", persistent, persistent)
    persistent.Load("")
    persistent.Save(nil, "")

    call(&CsvPersistent{})
    call(&GobPersistent{})

    // 疑问：为什么指针对象也有Save和Load方法
    // 方法是值接收者 调用对象是一个指针 可以直接调用值接收者方法 -> 语法糖 指针对象会自动进行解引用
    // 但是接口这一块并不是语法糖

    // 指针类型接收者方法 如何赋值
    fmt.Println("指针接收者：")
    persistent = new(JsonPersistent)
    persistent.Load("")
    persistent.Save(nil, "")

    // persistent = JsonPersistent{}
    // persistent.Load("")
    // persistent.Save(nil, "")
    // sonPersistent does not implement Persistent (Load method has pointer receiver)

    // 又有指针 又有值接收者方法
    // ProtobufPersistent{} => 未实现Save值接收者方法
    // new(ProtobufPersistent) => 可以
    persistent = &ProtobufPersistent{}
    persistent.Load("")
    persistent.Save(nil, "")

    // 接口方法是值接收者 会生成对应的指针接收者方法
    // 接口方式是指针接收者 不会生成对应的值接收者方法
}
```

### 结构体赋值接口类型

```go
package main

import "fmt"

type User struct{}

type Persistent interface {
    Save([]User, string) error
    Load(string) ([]User, error)
}

type GobPersistent struct {
    Version string
}

func (g GobPersistent) Save(u []User, path string) error {
    fmt.Println("Save")
    return nil
}

func (g GobPersistent) Load(path string) ([]User, error) {
    fmt.Println("Load")
    return nil, nil
}

func (g GobPersistent) Test() {
    fmt.Println("Test")
}

func main() {
    // 接口赋值不能省略类型 省略之后类型不一样 一个是接口类型 一个是结构体
    var persistent Persistent = GobPersistent{}
    // var persistent = GobPersistent{}
    fmt.Printf("%#v\n", persistent)

    persistent.Save(nil, "") // 正常调用
    persistent.Load("")      // 正常调用
    // persistent.Test()               // type Persistent has no field or method Test. 不能调用Test方法 Persistent接口无Test方法 => 行为丢失
    // fmt.Println(persistent.Version) // 不能获取Version属性
}
```

### 接口赋值接口

```go
package main

import (
    "fmt"
)

type User struct{}

type Persistent interface {
    Save([]User, string) error
    Load(string) ([]User, error)
}

type GobPersistent struct {
    Version string
}

func (g GobPersistent) Save(u []User, path string) error {
    fmt.Println("Save")
    return nil
}

func (g GobPersistent) Load(path string) ([]User, error) {
    fmt.Println("Load")
    return nil, nil
}

func (g GobPersistent) Test() {
    fmt.Println("Test")
}

type Saver interface {
    Save([]User, string) error
}

type Loader interface {
    Load(string) ([]User, error)
}

type Dumper interface {
    Save([]User, string) error
    Dump()
}

func main() {
    var persistent Persistent = GobPersistent{}
    fmt.Printf("%#v\n", persistent)

    var saver Saver
    fmt.Printf("%T, %#v\n", saver, saver)

    saver = GobPersistent{}
    fmt.Printf("%T, %#v\n", saver, saver)
    saver.Save(nil, "")

    saver = persistent // 接口对象赋值给另一个接口对象
    fmt.Printf("%T, %#v\n", saver, saver)

    var loader Loader = persistent
    fmt.Printf("%T, %#v\n", loader, loader)

    // 没有实现Dump方法 无法赋值
    // var dumper Dumper = persistent

    // 接口赋值可以使用自定义类型创建的对象赋值 或者 通过接口类型的变量进行赋值(需要实现对应的方法)

    // 接口赋值的过程中 会丢失一些方法
}
```

### 类型断言

```go
package main

import (
    "fmt"
)

type User struct{}

type Persistent interface {
    Save([]User, string) error
    Load(string) ([]User, error)
}

type GobPersistent struct {
    Version string
}

func (g GobPersistent) Save(u []User, path string) error {
    fmt.Println("Save")
    return nil
}

func (g GobPersistent) Load(path string) ([]User, error) {
    fmt.Println("Load")
    return nil, nil
}

func (g GobPersistent) Test() {
    fmt.Println("gob Test")
}

type CsvPersistent struct {
    Version string
}

func (c CsvPersistent) Save(u []User, path string) error {
    fmt.Println("Save")
    return nil
}

func (c CsvPersistent) Load(path string) ([]User, error) {
    fmt.Println("Load")
    return nil, nil
}

func (c CsvPersistent) Test() {
    fmt.Println("csv Test")
}

type Saver interface {
    Save([]User, string) error
}

func main() {
    var persistent Persistent = GobPersistent{"1.1.1"}

    // 类型转换
    // 断言：只能对接口类型做操作 转换为一个更具体的类型
    // 对应类型的变量, ok := 接口类型变量.(类型)

    // persistent.Test()

    gobPersistent, ok := persistent.(GobPersistent)
    fmt.Printf("%T, %#v, %T, %#v\n", ok, ok, gobPersistent, gobPersistent)
    gobPersistent.Test()
    fmt.Println(gobPersistent.Version)

    // user没有实现persistent接口 无法断言
    // user, ok := persistent.(User)
    // fmt.Println(user, ok)

    csvPersistent, ok := persistent.(CsvPersistent)
    fmt.Printf("%T, %#v, %T, %#v\n", ok, ok, csvPersistent, csvPersistent) // ok == false
    csvPersistent.Test()

    var saver Saver = persistent

    p, ok := saver.(Persistent)
    fmt.Printf("%T, %#v, %T, %#v\n", ok, ok, p, p)
    p.Load("")

    g, ok := saver.(GobPersistent)
    fmt.Printf("%T, %#v, %T, %#v\n", ok, ok, g, g)
    fmt.Println(g.Version)

    c, ok := saver.(CsvPersistent)
    fmt.Printf("%T, %#v, %T, %#v\n", ok, ok, c, c)
    c.Test()
}
```

### 多个类型断言

```go
// 多个类型断言 if...else / switch
package main

import "fmt"

func main() {
    var persistent Persistent = &ProtobufPersistent{}

    // if g, ok := persistent.(GobPersistent); ok {
    //     fmt.Println("g", g, ok)
    // } else if c, ok := persistent.(CsvPersistent); ok {
    //     fmt.Println("c", c, ok)
    // } else if j, ok := persistent.(*JsonPersistent); ok {
    //     fmt.Println("j", j, ok)
    // } else if p, ok := persistent.(*ProtobufPersistent); ok {
    //     fmt.Println("p", p, ok)
    // } else {
    //     fmt.Println("error")
    // }

    /*
        switch 接口类型变量.(type) {
        case GobPersistent:
        case CsvPersistent:
        case *JsonPersistent:
        case *ProtobufPersistent:
        default:
        }

        switch p := 接口类型变量.(type) {
        case GobPersistent:
        case CsvPersistent:
        case *JsonPersistent:
        case *ProtobufPersistent:
        default:
        }
    */
    switch p := persistent.(type) {
    case GobPersistent:
        fmt.Println("g", p)
    case CsvPersistent:
        fmt.Println("c", p)
    case *JsonPersistent:
        fmt.Println("j", p)
    case *ProtobufPersistent:
        fmt.Println("p", p)
    default:
        fmt.Println("error")
    }
}
```

### 接口嵌入

```go
package main

import (
    "fmt"
)

type User struct{}

type GobPersistent struct {
    Version string
}

func (g GobPersistent) Save(u []User, path string) error {
    fmt.Println("Save")
    return nil
}

func (g GobPersistent) Load(path string) ([]User, error) {
    fmt.Println("Load")
    return nil, nil
}

type Saver interface {
    Save([]User, string) error
}

type Loader interface {
    Load(string) ([]User, error)
}

type PersistentV2 interface {
    Saver
    Loader
}

type PersistentV3 interface {
    Saver
    Loader
    Dump() // V3这个接口 还需要实现Dump方法
}

func main() {
    // 接口只有匿名嵌入
    var persistentV2 PersistentV2 = GobPersistent{}

    fmt.Printf("%T, %#v\n", persistentV2, persistentV2)
    persistentV2.Load("")
    persistentV2.Save(nil, "")
}

// 结构体当前使用接口嵌入
package main

import (
    "fmt"
)

type User struct{}

type Persistent interface {
    Save([]User, string) error
    Load(string) ([]User, error)
}

type GobPersistent struct {
    Version string
}

func (g GobPersistent) Save(u []User, path string) error {
    fmt.Println("Save")
    return nil
}

func (g GobPersistent) Load(path string) ([]User, error) {
    fmt.Println("Load")
    return nil, nil
}

type Storer struct {
    Persistent Persistent
}

type StorerV2 struct {
    Persistent // 匿名嵌入
}

func main() {
    storer := Storer{GobPersistent{}}
    storer.Persistent.Load("")
    storer.Persistent.Save(nil, "")

    storerV2 := StorerV2{GobPersistent{}}
    storerV2.Persistent.Load("")
    storerV2.Persistent.Save(nil, "")

    storerV2.Load("")
    storerV2.Save(nil, "")
}
```

### 匿名接口

```go
package main

import "fmt"

func main() {
    var test = func() {
        fmt.Println("匿名函数")
    }

    var user struct {
        ID   string
        Name string
    } // 匿名结构体

    var saver interface {
        Save(string) error
    } // 匿名接口

    fmt.Printf("%T, %#v\n", test, test)
    fmt.Printf("%T, %#v\n", user, user)
    fmt.Printf("%T, %#v\n", saver, saver)
}
```

### 空接口及其使用

```go
package main

import "fmt"

type EmptyStruct struct{}

type EmptyInterface interface{}

func main() {
    emptyStr := ""
    fmt.Println(emptyStr)

    emptySlice := make([]int, 0)
    fmt.Println(emptySlice)

    emptyMap := make(map[string]string)
    fmt.Println(emptyMap)

    emptyStruct := EmptyStruct{}
    fmt.Println(emptyStruct)

    // 空接口
    var emptyInterface EmptyInterface
    // 空接口如何赋值?
    // 空接口没有定义任何的签名 任意对象都可以赋值给空接口
    emptyInterface = 1
    fmt.Printf("%T, %#v\n", emptyInterface, emptyInterface)

    emptyInterface = "123"
    fmt.Printf("%T, %#v\n", emptyInterface, emptyInterface)

    emptyInterface = true
    fmt.Printf("%T, %#v\n", emptyInterface, emptyInterface)

    emptyInterface = emptyStruct
    fmt.Printf("%T, %#v\n", emptyInterface, emptyInterface)

    emptyInterface = emptySlice
    fmt.Printf("%T, %#v\n", emptyInterface, emptyInterface)

    // 空接口 + 匿名
    var empty interface{} // empty空接口的变量
    fmt.Printf("%T, %#v\n", empty, empty)
    empty = 1
    fmt.Printf("%T, %#v\n", empty, empty)
    empty = true
    fmt.Printf("%T, %#v\n", empty, empty)

    fmt.Println("") // fmt.Println 可以打印任意类型的变量

    // 如何实现一个可以接收任意类型变量的函数 => 使用空接口
    fmt.Println(t(1))
    fmt.Println(t(""))
    fmt.Println(t(true))
    fmt.Println(t([]string{"1", "@"}))
    fmt.Println(t(map[string]int{"1": 1, "@": 2}))

    // 如何实现一个可以接收任意数量 任意类型的函数
    args(1, 2, 3)
    args("abc", 1, []string{})
}

func t(args interface{}) string {
    return fmt.Sprintf("%T", args)
}

func args(args ...interface{}) {
    for i, arg := range args {
        fmt.Println(i, t(arg))
    }
}
```

## Json序列化/反序列化

```go
1. Type, Value
2. 使用反射
3. 反射应用(内部实现使用反射)： json \ xml
```

### encoding/json

```go
go doc encoding/json
```

### 基本数据类型/json 转换

```go
package main

/* json: 格式化的字符串
GO                  =>          Json
数值：number(int/float)   => 1.1 10 -10
字符串：string              => ""
布尔：bool                => true, false
数组：[]Type/[Length]Type => [1, 2, 3, "", ""]
字典(Object):map[KTepy]Value     => {"key": value, "key2": value2}

Go数据类型 与 Json字符串转换
Go数据类型 <=> Gob二进制字符串
Go数据类型 <=> xml字符串
Go数据类型 <=> Protobuffer

持久化/网络传输
=> 序列化： Go数据类型 => 字符串(文本/二进制)
=> 反序列化：字符串(文本/二进制) => Go数据类型

编码规则 => 库(包)
*/

import (
    "encoding/json"
    "fmt"
)

func main() {

    var (
        number float64        = 1.1
        name   string         = "minho"
        isBody bool           = true
        users  []string       = []string{"1", "2", "3"}
        scores map[string]int = map[string]int{"1": 90, "2": 80, "3": 100}
    )

    // 序列化为json字符串
    b, err := json.Marshal(number)
    fmt.Printf("%#v, %T, %#v\n", err, b, string(b))

    b, err = json.Marshal(name)
    fmt.Printf("%#v, %T, %#v\n", err, b, string(b))

    b, err = json.Marshal(isBody)
    fmt.Printf("%#v, %T, %#v\n", err, b, string(b))

    b, err = json.Marshal(users)
    fmt.Printf("%#v, %T, %#v\n", err, b, string(b))

    b, err = json.Marshal(scores)
    fmt.Printf("%#v, %T, %#v\n", err, b, string(b))

    // Unmarshal 反序列化
    jsonB := `
    {
        "name": "minho",
        "age": 30,
        "isBody": true,
        "scores": [1, 2, 3, 4]
    }
    `
    var rs map[string]interface{}

    err = json.Unmarshal([]byte(jsonB), &rs)
    fmt.Printf("%#v, %#v\n", err, rs)

    value, _ := rs["name"]
    fmt.Printf("%T", value) // value是一个空接口
    v, ok := value.(string)
    fmt.Printf("%T, %T, %#v, %#v\n", v, ok, v, ok)
}
```

### 自定义数据类型/json转换

```go
package main

import (
    "encoding/json"
    "fmt"
)

// 需要设置属性与Json字符串中key的对应关系
// Id => json.pk => 通过结构体(属性)标签设置
// key1:"value1" key2:"value2"
// json:"name"

// 网络传输时 不需要将某个key进行序列化 `json:"-"` 结构体标签设置为- 序列化/反序列化不会携带该属性
// json:"-" 序列化和反序列化时忽略属性
// json:"name,type(string),omitempty"
// omitempty 属性对应的值为零值在json字符串中不包含(忽略0值)
type User struct {
    Id       int               `json:"pk,string"`
    Name     string            `json:"name"`
    Password string            `json:"-"`
    IsBoy    bool              `json:"isBoy"`
    Scores   []float64         `json:"scores"`
    Phone    map[string]string `json:"phone"`
}

func main() {
    user := User{1, "minho", "123#@", true, []float64{1, 3, 4}, map[string]string{"tel": "13308047517"}}

    b, err := json.Marshal(user)             // 自定义属性是小写 json包无法访问属性
    fmt.Printf("%#v, %#v\n", err, string(b)) // 转换字典Object key是属性名称 value是值

    jsonB := `
    {
        "pk": "10",
        "name": "minho",
        "Password": "123@abc",
        "isBoy": false,
        "phone": {"mobile": "12314341"}
    }
    `
    var u User
    err = json.Unmarshal([]byte(jsonB), &u)
    fmt.Printf("%#v, %#v\n", err, u)

    users := []User{
        {1, "minho", "123#@", true, []float64{1, 3, 4}, map[string]string{"tel": "13308047517"}},
        {2, "minho2", "123#@", true, []float64{1, 3, 4}, map[string]string{"tel": "13308047517"}},
        {3, "minho3", "123#@", true, []float64{1, 3, 4}, map[string]string{"tel": "13308047517"}},
    }
    // 调试可以打印格式输出 网络传输不用 浪费字节
    b, err = json.MarshalIndent(users, "", "\t")
    fmt.Printf("%#v, %s\n", err, string(b))

    jsonB = `
    [
        {
                "pk": "11",
                "name": "minho",
                "isBoy": true,
                "scores": [
                        1,
                        3,
                        4
                ],
                "phone": {
                        "tel": "13308047517"
                }
        },
        {
                "pk": "22",
                "name": "minho2",
                "isBoy": true,
                "scores": [
                        1,
                        3,
                        4
                ],
                "phone": {
                        "tel": "13308047517"
                }
        },
        {
                "pk": "33",
                "name": "minho3",
                "isBoy": true,
                "scores": [
                        1,
                        3,
                        4
                ],
                "phone": {
                        "tel": "13308047517"
                }
        }
]
`
    var us []User
    err = json.Unmarshal([]byte(jsonB), &us)
    fmt.Println(err, us)
}
```

### 自定义嵌入结构体 转换

```go
package main

import (
    "encoding/json"
    "fmt"
)

type Addr struct {
    Street string `json:"street"`
    No     string `json:"no"`
}

type User struct {
    Id   int    `json:"id"`
    Name string `json:"name"`
    Addr Addr   `json:"addr"`
}

func main() {
    u := User{
        1,
        "minho",
        Addr{"成都", "640000"},
    }
    b, _ := json.MarshalIndent(u, "", "\t")
    fmt.Println(string(b))

    // 等于嵌入Object
    jsonB := `
    {
        "id": 1,
        "name": "minho",
        "addr": {
                "street": "成都",
                "no": "640000"
        }
    }
    `
    var rs User
    json.Unmarshal([]byte(jsonB), &rs)
    fmt.Println(rs)
}
```

### json.Valid

```go
// 验证一个字符串是否是正确的json格式
```

### 对流的操作

```go
// 序列化到文件/从文件反序列化
package main

import (
    "encoding/json"
    "fmt"
    "os"
)

type Addr struct {
    Street string `json:"street"`
    No     string `json:"no"`
}

type User struct {
    Id   int    `json:"id"`
    Name string `json:"name"`
    Addr Addr   `json:"addr"`
}

func main() {
    u := User{1, "MInho", Addr{"成都", "10001"}}

    file, _ := os.Create("user.json")
    defer file.Close()
    encoder := json.NewEncoder(file) // Strout输出到控制台
    err := encoder.Encode(u)
    fmt.Println(err)

    file, _ = os.Open("user.json")
    decoder := json.NewDecoder(file)
    var user User
    err = decoder.Decode(&user)
    fmt.Println(err, user)

}
```

### json.RawMessage

```go
package main

import (
    "encoding/json"
    "fmt"
)

type User struct {
    Id   int
    Name string
    Addr json.RawMessage
}

func main() {
    txt := `{
        "id": 100,
        "name": "Minho", 
        "addr": {"street": "成都", "no": "10002"}
        }`
    var u User
    err := json.Unmarshal([]byte(txt), &u)
    fmt.Println(err, u)
}
```

### MarshalJSON/UnmarshalJSON

```go
package main

import (
    "encoding/json"
    "fmt"
    "time"
)

type Date time.Time

func (d Date) MarshalJSON() ([]byte, error) {
    return json.Marshal(time.Time(d).Format("2006-01-02"))
}

// UnmarshalJSON 一般要赋值 定义成指针类型接收者
func (d *Date) UnmarshalJSON(b []byte) error {
    var s string
    if err := json.Unmarshal(b, &s); err != nil {
        return err
    }
    if date, err := time.Parse("2006-01-02", s); err != nil {
        return err
    } else {
        *d = Date(date)
    }
    return nil
}

type User struct {
    Id        int
    Name      string
    Birthday  time.Time
    Birthday2 Date
}

func main() {
    u := User{1, "", time.Now(), Date(time.Now())}
    // 无法序列化Birthday2
    b, err := json.MarshalIndent(u, "", "\t")
    fmt.Println(string(b), err)

    txt := `
    {
        "Id": 2,
        "Name": "Minho",
        "Birthday": "2021-08-20T17:34:58.3897013+08:00",
        "Birthday2": "2021-08-31"
    }
    `
    var us User
    err = json.Unmarshal([]byte(txt), &us)
    fmt.Println(err, us)
}

// MarshalText / UnmarshalText
// 与JSON区别是不需要再使用json.Marshal()函数转换

type Date2 time.Time

func (d Date2) MarshalText() ([]byte, error) {
    return []byte(time.Time(d).Format("2006-01-02")), nil
}

func (d *Date2) UnmarshalText(b []byte) error {
    s = := string(b)
    if date, err := time.Parse("2006-01-02", s); err != nil {
        return err
    } else {
        *d = Date2(date)
    }
    return nil
}
```

## 反射

```go
// 反射是指在运行时动态的访问和修改任意类型对象的结构和成员
// 在go语言中提供reflect包提供反射的功能
// 每一个变量都有两个属性：类型(Type) 和值(Value)

go doc reflect
```

### 通过反射实现简单的ORM

```go
// 生成结构体对应的建表SQL语句
package main

import (
    "fmt"
    "log"
    "reflect"
    "strings"
)

// sql:"type(text);size(32);pk;null;default()"
type User struct {
    Id   int
    Name string `sql:"type(text)"`
    Addr string `sql:"type(varchar);size(1024);null;default(成都)"`
    Desc string
}

func (u User) TableName() string {
    return "account"
}

type Department struct {
    Id   int
    Name string
    Addr string
}

func snake(name string) string {
    return strings.ToLower(name)
}

func tableName(value reflect.Value) string {
    tableName := value.Elem().Type().Name()
    methodValue := value.MethodByName("TableName")
    if !methodValue.IsValid {
        return tableName
    }
    methodType := methodValue.Type()
    if 0 != methodType.NumIn() || 1 != methodType.Numout() {
        return tableName
    }
    if outType := methodType.out(0); outType.Kind() != reflect.String {
        return tableName
    }
    
    params := make([]reflect.Value, 0)
    rts := methodValue.Call(params)
    if res[0].String() != "" {
        return rts[0].string()
    }
    return tableName
}

func typeMapper(field reflect.StructField) string {
    kind := field.Type.Kind()
    tag := field.Tag.Get("sql")

    typeValue := ""
    sizeValue := ""
    defaultValue := ""

    switch kind {
    case reflect.Int:
        typeValue = "int"
    case reflect.String:
        typeValue = "varchar"
        sizeValue = "32"
    default:
        return "text"
    }
    pkValue := false
    nullValue := false

    for _, v := range strings.Split(tag, ";") {
        if strings.HasPrefix(v, "type(") {
            typeValue = v[5 : len(v)-1]
        } else if strings.HasPrefix(v, "size(") {
            sizeValue = v[5 : len(v)-1]
        } else if strings.HasPrefix(v, "default(") {
            defaultValue = v[8 : len(v)-1]
        } else if v == "pk" {
            pkValue = true
        } else if v == "null" {
            nullValue = true
        }
    }

    var builder strings.Builder
    builder.WriteString(typeValue)
    if typeValue == "varchar" {
        builder.WriteString(fmt.Sprintf("(%s)", sizeValue))
    }
    // builder.WriteString(" ")

    // varchar(32) primary key not null default "";

    if pkValue {
        builder.WriteString(" primary key")
    }
    if !nullValue {
        builder.WriteString(" not null")
    }
    if defaultValue != "" {
        builder.WriteString(" default ")
        if typeValue == "varchar" {
            builder.WriteString(fmt.Sprintf(`"%s"`, defaultValue))
        } else {
            builder.WriteString(fmt.Sprintf(`%s`, defaultValue))
        }
    }
    return builder.String()
}

func sqlDump(models ...interface{}) string {
    var builder strings.Builder
    for _, model := range models {
        t := reflect.TypeOf(model)
        v := reflect.ValueOf(model)

        if t.Kind() == reflect.Ptr && t.Elem().Kind() == reflect.Struct {
            st := t.Elem()
            builder.WriteString(fmt.Sprintf("CREATE TABLE `%s` (\n", snake(tableName(v))))

            filedNum := st.NumField()
            if filedNum == 0 {
                log.Panicf("%s field num is zero", st.Name())
            }

            for i := 0; i < filedNum; i++ {
                field := st.Field(i)
                // fmt.Println(field.Name, field.Type.Kind())
                builder.WriteString(fmt.Sprintf("\t `%s` %s", snake(field.Name), typeMapper(field)))
                if i != filedNum-1 {
                    builder.WriteString(",")
                }
                builder.WriteString("\n")
            }

            builder.WriteString(fmt.Sprintf(") ENGINE=INNODB DEFAULT CHARSET utf8mb4;\n\n"))
        } else {
            panic("error")
        }
    }

    return builder.String()
}

func main() {
    sql := sqlDump(new(User), new(Department))
    fmt.Println(sql)
}
```

## Go并发

```go
// OS: 进程/线程
进程：资源(内存/文件IO) 以进程为单位 进程是资源分配的最小单位
线程：执行的逻辑(主线程) 执行调用的是线程 线程是CPU调度的最小单元
协程：应用程序层面 用户态的调度(消耗小 性能很高)
     - 其他语言 第三方库 python2.7 => gevent greentleet
     - Go 在语言层面就提供 goroutine

// GMP模型
  G => 例程 goroutine
  M => 执行代码的线程(CPU)
  P => Processer 调度 上下文的切换 内存申请...

  P是连接G和M的(Go中的调度器) // runtime包

// 例程
go 函数调用 // 关键字 go
goroutine执行切换：
  IO等待(IO阻塞)
  到达执行时间(最大执行时间)
  内核切换
  主动让出(sleep)

// main函数由GO调用器创建例程进行执行 => 主例程 
// 自己用go关键字加函数调用创建的例程 => 工作例程
// 当主例程执行完成 自动结束所有工作例程

// 在某个例程里面 代码执行顺序依然按照逻辑关系按顺序执行

// 并发
  goruntine => 多个例程时 如何通信
  并发通信
    内存：变量
```

### 主例程

```go
package main

import (
    "fmt"
    "time"
)

// 打印A-Z
func printChats(prefix string) {
    for c := 'A'; c <= 'Z'; c++ {
        fmt.Printf("%s: %c\n", prefix, c)
        time.Sleep(time.Millisecond)
    }
}

func main() {
    // 至少有三个
    // c1 c2交叉执行 c1 c2同时在执行
    go printChats("c1:")
    go printChats("c2:")
    time.Sleep(3 * time.Millisecond) // 不等待 没有结果打印 因为主例程直接退出

    // 问题： 能否等待工作例程结束后 再让主例程退出(主例程要等待某几个/或等待所有的工作例程结束)
}
```

### 计数信号量/WaitGroup

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

// 打印A-Z
func printChats(wg *sync.WaitGroup, prefix string) {
    defer wg.Done() // stage B
    for c := 'A'; c <= 'Z'; c++ {
        fmt.Printf("%s: %c\n", prefix, c)
        time.Sleep(time.Millisecond)
    }
    // stage C

    wg.Add(1)
    go childChars(wg, prefix)
}

func childChars(wg *sync.WaitGroup, prefix string) {
    defer wg.Done()

    for c := 'A'; c <= 'Z'; c++ {
        fmt.Printf("child.%s: %c\n", prefix, c)
        time.Sleep(time.Millisecond)
    }
}

func main() {
    var wg sync.WaitGroup // 定义计数信号量
    /*
        stage A
        go func() {
            stage B
            ...
            有可能会报错
            ...
            stage C
        }
        stage D
    */
    // wg.Add(N) => 启动例程之前执行 stage A阶段；在计数信号量中+N
    // wg.Done() => 例程执行结束后调用(例程函数退出时) stage C； 如何保证wg.Done()一定执行 stage B + defer; 当函数执行结束对计数信号量减1
    // wg.Wait() => 启动所有例程后调用 等待计数为0之后执行(不是0的时候一直等待)

    wg.Add(1) // stage A
    go printChats(&wg, "c1")
    wg.Add(1)                // stage A
    go printChats(&wg, "c2") // wg需要时指针 否则时两个wg 函数里面的wg Done减少计数的时候 不影响外面的wg

    fmt.Println("Wait")
    wg.Wait() // stage D
    fmt.Println("Over")

    // WaitGroup.Add数量 <= 例程数量
    // 某些例程不需要等待结束 即可结束主例程(程序)
}
```

### 闭包陷阱

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

func main() {
    wg := sync.WaitGroup{}

    wg.Add(2)
    for i := 0; i < 2; i++ {
        go func(i int) {
            prefix := fmt.Sprintf("c%d", i+1)
            for c := 'A'; c <= 'A'; c++ {
                fmt.Printf("%s: %c\n", prefix, c)
                time.Sleep(time.Millisecond)
            }
            wg.Done()
        }(i)
    }

    fmt.Println("Wait")
    wg.Wait()
    fmt.Println("Over")
}

// 使用的是同一个wg变量 不用再传递指针
// go func匿名函数里面使用的i变量 需要参数传递进来 如果直接使用外部i变量 结果不确定(联想延迟执行) for循环执行完i=2 执行例程 输出结果中 prefix值为3

```

### 并发通信/加锁

```go
// 多个例程对同一个内存资源进行修改 未对资源进行同步限制 导致修改数据混乱
package main

import (
    "fmt"
    "sync"
    "time"
)

// 从一个账户借钱/还钱
// 账户初始化500*10000

// 解决方法：借钱和还钱过程中不要中断
// 加锁

var money int = 500 * 10000

var locker sync.Mutex

// 借钱
func borrow(m int) int {
    locker.Lock()
    defer locker.Unlock()

    if money < m {
        // 退出一定要释放锁
        // locker.Unlock()
        return 0
    }
    money -= m

    // locker.Unlock()
    return m
}

// 还钱
func payback(m int) {
    locker.Lock()
    defer locker.Unlock()
    money += m
}

func main() {
    // 2个人借钱 A/B

    wg := sync.WaitGroup{}

    for person := 'A'; person <= 'Z'; person++ {
        wg.Add(1)
        go func(person rune) {
            defer wg.Done()
            // 每次2000
            m := borrow(2000)
            time.Sleep(2 * time.Microsecond)

            payback(m)

        }(person)
    }

    wg.Wait()
    fmt.Println("money:", money)
}
```

### 并发通信/管道

```go
package main

import (
    "fmt"
    "time"
)

// 管道
func main() {
    // CSP原理(顺序通信进程)
    // 队列 buffer区(线程/协程安全) -> 入队列 -> 出队列
    // 类型 chan T T:Go不限制类型 但是常常放一些值类型
    // 如果放一个切片(引用类型 修改会直接修改内存上面的值)到管道 其他例程从管道中读取切片后 对切片元素修改 会对放入端切片有影响

    // 放置int类型元素的管道
    var channel chan int
    fmt.Printf("%T, %#v\n", channel, channel)

    // 赋值 make函数
    // 对应有对管道不通的读和写操作的例程
    // make(chan T) 不带缓冲区的管道
    // make(chan T, size) 带缓冲区的管道 长度size
    // 5 size => 1 1 1 1 1 => 1 缓冲区满 需要等待(阻塞) 如果Go检查无goroutine可以读 发生死锁
    // 读 读空channel 如果Go检查无goroutine可以写 产生死锁
    channel = make(chan int)
    // 如何写
    go func() {
        time.Sleep(5 * time.Second)
        channel <- 4
    }()

    // 如何读
    e := <-channel
    fmt.Println(e)
}
```

#### channel实现WaitGroup

```go
// channel实现sync.WaitGroup 
// 需要提前知道你的程序需要起多少个例程
package main

import "fmt"

func main() {
    var channel chan int = make(chan int) // 初始化
    
    for i := 0; i < 2; i++ {
        go func(i int){
            for c := 'A'; c <= 'Z'; c++ {
                fmt.Printf("%d, %c\n", i, c)
            } 
            
            // 执行完例程之后写入数据
            channel <- 0
        }(i)
    }
    
    fmt.Println("wait")
    for i := 0; i < 2; i++ {
        <-channel
    }
    fmt.Println("over")
}
```

#### 带缓冲区channel

```go
package main

import (
    "fmt"
    "time"
)

// 带缓冲区channel
// channel跟队列类似 先进先出
func main() {
    // var channel chan string = make(chan string, 3)
    // var channel = make(chan string, 3)
    channel := make(chan string, 3)
    fmt.Println(len(channel)) // 0
    channel <- "a"
    fmt.Println(len(channel)) // 1
    channel <- "b"
    channel <- "c"
    fmt.Println(len(channel)) // 3
    fmt.Println(<-channel)    // a
    channel <- "d"
    fmt.Println(<-channel) // b
    fmt.Println(<-channel) // c
    fmt.Println(<-channel) // d

    go func() {
        time.Sleep(5 * time.Second)
        channel <- "x"
    }()

    fmt.Println(<-channel) // 死锁 管道为空 没有程序继续些数据
}
```

#### 关闭管道

```go
管道关闭之后 无法再写入数据 报错：panic: send on closed channel

管道关闭之后 可以读取数据 v, ok := <-channel
第二个值判断是否读取完数据 false无法再读取且此时v==0

```

```go
// 不带缓冲区管道
package main

import (
    "fmt"
    "time"
)

func main() {
    channel := make(chan int)
    // bufferChannel := make(chan string, 3)

    go func() {
        time.Sleep(3 * time.Second)
        v, ok := <-channel // ok第二个参数 判断管道是否关闭
        fmt.Println(v, ok) // 1

        v, ok = <-channel
        fmt.Println(v, ok) // 0
    }()

    // 关闭管道之后 还能读写吗
    // 不能针对关闭的管道写数据 panic: send on closed channel
    // 针对已经关闭的管道可以进行读取
    channel <- 1
    // 关闭管道
    close(channel)

    time.Sleep(5 * time.Second)
}

// 带缓冲区管道
package main

import "fmt"

func main() {
    bufferChannel := make(chan int, 3)

    bufferChannel <- 1
    close(bufferChannel)
    // bufferChannel <- 2 // 关闭之后不能写 panic: send on closed channel

    v, ok := <-bufferChannel
    fmt.Println(v, ok) // 1 true

    v, ok = <-bufferChannel
    fmt.Println(v, ok) // 0 false
}
```

#### 管道循环

```go
package main

import "fmt"

func main() {
    channel := make(chan int, 3)

    channel <- 1
    channel <- 2
    channel <- 3
    close(channel) // 不关闭 range循环管道会报错 死锁

    for v := range channel {
        fmt.Println(v) // 自动判断是否还能取到值 false
    }
    // 手动自己判断第二个值
    // for {
    //     if v, ok := <-channel; ok {
    //         fmt.Println(v)
    //     } else {
    //         break
    //     }
    // }
}
```

#### 并发下载图片示例

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

/*
要获取网站图片 1.jpg, 2.jpg, ... , n,jpg
下载 m.jpg - n.jpg
*/

// 模拟下载函数
func download(wg *sync.WaitGroup, name, path string) {
    defer wg.Done()
    time.Sleep(10 * time.Millisecond)
    fmt.Printf("download %s to %s\n", name, path)
}

func main() {
    start := time.Now()
    wg := sync.WaitGroup{}
    m, n := 1, 100
    for i := m; i <= n; i++ {
        wg.Add(1)
        go download(&wg, fmt.Sprintf("%d.jpg", i), fmt.Sprintf("download/%d.jpg", i))
    }

    wg.Wait()
    fmt.Printf("spend time: %s\n", time.Now().Sub(start))
}

// 问题：需要下载数里很多怎么办？ 不能创建任意数量例程
// 固定N个例程区去请求 比如 每次10个(Pool)
// 利用管道：生产者消费者模型 生产下载任务 -> 每次10worker消费

package main

import (
    "fmt"
    "sync"
    "time"
)

type Fiel struct {
    name string
    path string
}

func download(name, path string) {
    time.Sleep(10 * time.Millisecond)
    fmt.Printf("download %s to %s\n", name, path)
}

func main() {
    start := time.Now()
    N := 10
    m, n := 1, 100
    channel := make(chan Fiel, N)

    wg := sync.WaitGroup{}

    // 生产者
    // 生成下载图片的例程
    wg.Add(1)
    go func(channel chan<- Fiel) {
        for i := m; i <= n; i++ {
            channel <- Fiel{fmt.Sprintf("%d.jpg", i), fmt.Sprintf("download/%d.jpg", i)}
        }
        close(channel)
        wg.Done()
    }()

    // 消费者
    for i := 0; i <= N; i++ {
        // N个下载图片例程
        wg.Add(1)
        go func(channel <-chan Fiel) {
            for f := range channel {
                download(f.name, f.path)
            }
            wg.Done()
        }()
    }
    wg.Wait()

    fmt.Println(time.Now().Sub(start))
}
```

#### 管道类型/读写管道

```go
package main

import "fmt"

func main() {
    // 在某个函数中只需要读 或者只需要写的时候
    // 为防止在只读函数中误写 在只写函数中误读
    // 可以将管道声明为 只读 或 只写管道
    // 更多用在函数 形参定义

    // 赋值
    channel := make(chan int, 10)
    var readChannel <-chan int  // 只读管道
    var writeChannel chan<- int // 只写管道

    readChannel = channel
    writeChannel = channel

    writeChannel <- 1
    writeChannel <- 2

    fmt.Println(<-readChannel)
    fmt.Println(<-readChannel)

    // var ch chan<- int = make(chan int, 10)
    // 一般不这样声明 没有意义
}
```

#### selectcase

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    // 多个管道进行读 某一个管道读取成功就执行对应逻辑
    // 多个管道进行写 某一个管道写成功就执行对应逻辑
    // 多个管道 某些进行读 某些进行写 某个管道 读/写成功 执行对应逻辑

    // select case
    /*
        select {
        case value, ok := <-channel:
        case channel <- value:
        default:
        }
    */
    channelV1 := make(chan int, 0)
    channelV2 := make(chan int, 0)

    go func() {
        time.Sleep(3 * time.Second)
        channelV1 <- 1
    }()

    fmt.Println("wait", time.Now())
    select {
    case v, ok := <-channelV1:
        fmt.Println("channelV1:", time.Now(), v, ok)
    case v, ok := <-channelV2:
        fmt.Println("channelV2:", time.Now(), v, ok)
    default: // 不会等待(case条件都没有读取数据 直接执行逻辑) 直接default 一般不会加
        fmt.Println("default")
    }
    fmt.Println("over")

}
```

#### 示例

```go
// 生成随机5个0 5个1组成的数
package main

import "fmt"

func main() {
    zero := make(chan int)
    one := make(chan int)

    go func() {
        for i := 0; i < 5; i++ {
            zero <- 0
        }
    }()

    go func() {
        for i := 0; i < 5; i++ {
            one <- 1
        }
    }()

    for i := 0; i < 10; i++ {
        select {
        case <-one:
            fmt.Print(1)
        case <-zero:
            fmt.Print(0)
        }

    }
}
```

#### context上下文

```go
package main

import (
    "context"
    "fmt"
    "time"
)

func main() {
    // go doc context
    // 事件广播
    // ctx, cannel := context.WithCancel(ctx)
    // cannel() => 广播消息

    channel := make(chan int, 1)

    go func() {
        i := 0
        for {
            channel <- i
            time.Sleep(2 * time.Second)
            i += 1
        }
    }()

    // 让例程执行10s之后就退出
    // 根事件 只能通过 context.Backgroud创建
    // ctx, cannel := context.WithCancel(context.Background())
    // go func() {
    //     time.Sleep(10 * time.Second)
    //     fmt.Println("cannel")
    //     cannel() // 调用cannel就相当于向管道写入一个数据
    // }()

    ctx, _ := context.WithTimeout(context.Background(), 10*time.Second) // 10s后中断

END:
    for {
        select {
        case <-ctx.Done(): // 从管道读取到数 说明结束
            fmt.Println("interrupt")
            break END
        case v, ok := <-channel:
            if !ok {
                break END
            } else {
                fmt.Println(v)
            }
        }
    }
}
```

### 简单的SocketServer

#### server.go

```go
package main

import (
    "fmt"
    "log"
    "net"
    "time"
)

func main() {
    // 监听IP和端口
    addr := ":9999" // "0.0.0.0:9999"
    listener, err := net.Listen("tcp", addr)
    if err != nil {
        log.Fatal(err)
    }

    for {
        // 获取客户端连接
        conn, err := listener.Accept()
        if err != nil {
            fmt.Println(err)
            continue
        }
        time.Sleep(10 * time.Second)
        log.Println("client：", conn.RemoteAddr())
        // 交互处理
        fmt.Fprintf(conn, time.Now().Format("2006-01-02 15:04:05"))
        // 关闭连接
        conn.Close()

    }

    // 关闭服务器
    listener.Close()
}

// 2021/08/27 11:41:26 client： 127.0.0.1:58934
// 2021/08/27 11:41:36 client： 127.0.0.1:58935
// 处理两个客户端连接 顺序执行没有并发 相差10s

// 启动协程 处理客户端连接 此时服务端并发处理客户端请求
go func() {
    time.Sleep(10 * time.Second)
    log.Println("client：", conn.RemoteAddr())
    // 交互处理
    fmt.Fprintf(conn, time.Now().Format("2006-01-02 15:04:05"))
    // 关闭连接
    conn.Close()
}()
```

#### client.go

```go
package main

import (
    "fmt"
    "log"
    "net"
    "time"
)

func main() {
    // 连接服务器
    addr := "127.0.0.1:9999"
    conn, err := net.Dial("tcp", addr)
    if err != nil {
        log.Fatal(err)
    }
    start := time.Now()
    // 交互
    bytes := make([]byte, 1024)
    n, err := conn.Read(bytes) // n 从byte里面读的数量
    if err != nil {
        fmt.Println(err)
    } else {
        fmt.Println(string(bytes[:n]))
    }
    fmt.Println(time.Now().Sub(start))
    // 关闭连接
    conn.Close()
}
```

## Reference Link

- Blogs
  - [使用Viper解码自定义格式](https://sagikazarmark.hu/blog/decoding-custom-formats-with-viper/)
  - [String padding in Go](https://gosamples.dev/string-padding/)
  - [Set indentaton on yaml.v3](https://hashnode.com/post/set-indentation-on-new-golang-yaml-v3-library-ckbrwl63001skn8s1lj3jmtln)
  - [Go slice tricks cheat sheet](https://ueokande.github.io/go-slice-tricks/)

- Github Issues
  - [YAML库关于等同json.RawMessage的Issue](https://github.com/go-yaml/yaml/issues/13)
