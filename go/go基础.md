## 整型
整型数据分为两类，有符号和无符号两种类型

有符号: int, int8, int16, int32, int64

无符号: uint, uint8, uint16, uint32, uint64, byte

int和uint的大小和系统有关，32位系统表示int32和uint32，如果是64位系统则表示int64和uint64
fmt.Printf("%T", var_name)输出变量类型  //int uint 
unsafe.Sizeof(var_name)查看变量占用字节

## 浮点型
* 关于浮点数在机器中存放形式的简单说明，浮点数=符号位+指数位+尾数位
* 尾数部分可能丢失，造成精度损失。-123.0000901
var num1 float32 = -123.0000901 --->-123.00009
* golang的浮点型默认为float64类型
* 通常情况下，应该使用float64,因为它比float32更精确
* 0.123可以简写成.123,也支持科学计数法表示:5.1234e2 等价于512.34


## 字符
Golang中没有专门的字符类型，如果要存储单个字符(字母)，一般使用byte来保存。

字符只能被单引号包裹，不能用双引号，双引号包裹的是字符串
当我们直接输出type值时，就是输出了对应字符的ASCII码值
当我们希望输出对应字符，需要使用格式化输出
```go
   var c1 byte = 'a'
    var c2 byte = '0'

    //当我们直接输出type值时，就是输出了对应字符的ASCII码值
    //'a' ==> 97
    fmt.Println(c1, "--", c2) //97 __48
    //如果我们希望输出对应字符，需要使用格式化输出
    fmt.Printf("c2 = %c c2 = %c", c1, c2)
```

## 字符类型本质探讨
字符型存储到计算机中，需要将字符对应的码值(整数)找出来

存储:字符 --> 码值 --> 二进制 --> 存储
读取: 二进制 -->码值 --> 字符 --> 读取

## 布尔型
布尔类型也叫做bool类型，bool类型数据只允许取值true或false

bool类型占1个字节

## 字符串
字符串就是一串固定长度的字符连接起来的字符序列。Go的字符串是由单个字节连接起来的。Go语言的字符串的字节使用UTF-8编码标识Unicode文本
* 字符串一旦赋值了，就不能修改了:在Go中字符串是不可变的。
* 字符串的两种标识形式
> 双引号，会识别转义字符
> 反引号，以字符串的原生形式输出，包括换行和特殊字符，可以实现防止攻击、输出源代码等效果
* 字符串拼接方式"+"
> var str string = "hello " + "world"    str += "!"


## 指针
* 基本数据类型，变量存的就是值，也叫值类型
* 取变量的地址，用&，比如var num int,获取num的地址:&num
* 获取指针类型所指向的值，使用:*,比如,var ptr *int,使用*ptr获取ptr指向的值
* 指针类型，指针变量存的是一个地址，这个地址指向的空间存的才是值，比如:var ptr *int = &num

### 值类型包括:基本数据类型、数组和结构体struct

## 值类型与引用类型
值类型:变量直接存储值，内存通常在栈中分配
引用类型:变量存储的是一个地址，这个地址对应的空间才真正存储数据(值)，内存通常在堆上分配，当没有任何变量应用这个地址时，该地址对应的数据空间就成为一个垃圾，由GC来回收。

## Golang中值类型和引用类型的区分
值类型:基本数据类型(int系列、float系列、bool、string)、数组和结构体
引用类型:指针、slice切片、map、管道chan、interface等都是引用类型

## 基本数据类型默认值
整型	0
浮点型	0
字符串	""
布尔类型	false

## 基本数据类型相互转换
Golang在不同类型的变量之间赋值时需要显式转换。也就是Golang中数据类型不能自动转换。
基本语法:

表达式var_type(var_name) 将值v转换为类型var_type
var_type：就是数据类型，比如int32, int64, float32等等
var_name：就是需要转换的变量

### 注意事项
* go中，数据类型的转换可以是从表示范围小-->表示范围大，也可以 范围大—>范围小
* 转换的是变量存储的数据(即值),变量本身的数据类型并没有变化!
* 在转换中，比如将int64转成int8,编译时不会报错，只是转换的结果是按溢出处理，和我们希望的结果不一样。
* 数据的转换必须显式转换，不能自动转换
* 定义一个int8类型的整数(var num int8 = 0),如果一直自加1，这个变量的值会是(0...127 -128 -127... 0 ...127)循环往复下去，而不会超过类型最大值的范围

```go
package main
import "fmt"

func main() {
    
    var n1 int32 = 12
    var n2 int64
    var n3 int8
    
    n2 = n1 + 20 //int32  --> int64  错误
    n3 = n1 + 20 //int32  --> int8   错误
    
    n2 = int64(n1) + 20 //正确
    n3 = int8(n1) + 20  //正确
}
```

## 数值型转成string类型，或者将string类型转成数值型
* func Sprintf(format string, a ...interface{}) string
```go
    str = fmt.Sprintf("%d", num1)
    fmt.Printf("str类型为 %T str = %q\n",str, str)
```

*  使用strconv包的函数
```go
    var num1 int = 99;
    var num2 float64 = 23.456
    var isTrue bool = true 

    var str string

    str = strconv.FormatInt(int64(num1), 10)
    str = strconv.Itoa(num1)
    fmt.Printf("str类型为 %T str = %q\n",str, str)

    str = strconv.FormatFloat(num2, 'f', 10, 64)
    fmt.Printf("str类型为 %T str = %q\n",str, str)

    str = strconv.FormatBool(isTrue)
    fmt.Printf("str类型为 %T str = %q\n",str, str)
```


## string类型转其他基本类型
* 使用strconv包的函数
```go
    var str1 string = "123456"
    var str2 string = "123.456"

    var isTrue bool
    var num int64
    var num2 float64

    isTrue, _ = strconv.ParseBool(str)
    fmt.Printf("str类型为 %T str = %v\n",isTrue, isTrue)

    num, _ = strconv.ParseInt(str1, 10, 64)
    fmt.Printf("str类型为 %T str = %v\n",num, num)

    num2, _ = strconv.ParseFloat(str2, 64)
    fmt.Printf("str类型为 %T str = %v\n",num2, num2) 
```


## break
break 语句可以结束 for、switch 和 select 的代码块，
break只能跳出一层，假如需要跳出多层可以使用OuterLoop标签

## continue
Go语言中 continue 语句可以结束当前循环，开始下一次的循环迭代过程，仅限在 for 循环内使用。
默认跳出一层，假如需要跳出多层可以使用OuterLoop标签

## array

声明：
var a [3]int
如果在数组长度的位置出现“...”省略号，则表示数组的长度是根据初始化值的个数来计算，
q := [...]int{1, 2, 3}
var q [3]int = [3]int{1, 2, 3}
var r [3]int = [3]int{1, 2}  //r[2] --> 0

判等
数组可以 == !=

遍历数组
for k, v := range team



## slice
声明：
a := []int{1, 2, 3}
var strList []string
b := make([]int, 2, 10)

- append()
var a []int
a = append(a, 1) // 追加1个元素
a = append(a, 1, 2, 3) // 追加多个元素, 手写解包方式
a = append(a, []int{1,2,3}...) // 追加一个切片, 切片需要

- 切片复制
Go语言的内置函数 copy() 可以将一个数组切片复制到另一个数组切片中，如果加入的两个数组切片不一样大，就会按照其中较小的那个数组切片的元素个数进行复制。

copy( destSlice, srcSlice []T) int
slice1 := []int{1, 2, 3, 4, 5}
slice2 := []int{5, 4, 3}
copy(slice2, slice1) // 只会复制slice1的前3个元素到slice2中
copy(slice1, slice2) // 只会复制slice2的3个元素到slice1的前3个位置

- 切片中删除元素
删除开头的元素可以直接移动数据指针：
a = []int{1, 2, 3}
a = a[1:] // 删除开头1个元素
a = a[N:] // 删除开头N个元素

也可以不移动数据指针，但是将后面的数据向开头移动，可以用 append 原地完成
a = []int{1, 2, 3}
a = append(a[:0], a[1:]...) // 删除开头1个元素
a = append(a[:0], a[N:]...) // 删除开头N个元素

a = []int{1, 2, 3, ...}
a = append(a[:i], a[i+1:]...) // 删除中间1个元素
a = append(a[:i], a[i+N:]...) // 删除中间N个元素
a = a[:i+copy(a[i:], a[i+1:])] // 删除中间1个元素
a = a[:i+copy(a[i:], a[i+N:])] // 删除中间N个元素

## sync.map
初始化
方式1
 var mapLit map[string]int
 mapLit = map[string]int{"one": 1, "two": 2}
方式2
 mapCreated := make(map[string]float32)
 mapCreated["key1"] = 4.5

遍历数组
for k, v := range team

删除清空
delete(map, key)
Go语言中并没有为 map 提供任何清空所有元素的函数

sync.Map
操作
* 初始化  var scene sync.Map
* 存  scene.Store("greece", 97)
* 取  scene.Delete("london")



## nil
指针、切片、映射、通道、函数和接口的零值则是 nil

## list

- 初始化
l := list.New()

- 列表中插入元素
双链表支持从队列前方或后方插入元素，分别对应的方法是 PushFront 和 PushBack。
l= list.New()
l.PushBack("fist")
l.PushFront(67)









