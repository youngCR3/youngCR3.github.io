---
title: The Go Programming Language Note
date: 2023-02-11 00:58:52
tags: Golang
categories: Programming Language
---

# Chapter1 Tutorial

> Ex1.3要求测试不同版本echo的性能差异，学习time package和benchmark测试后补

1. Go的发展历史：受到C、Pascal、CSP等语言的影响

2. Go程序的结构总是：package声明、package import、程序主体

3. 命令行参数：从os.Args[1:]可以读取string slice格式存储的命令行参数

4. 读取标准输入：bufio.NewScanner(os.Stdin)返回Scanner对象，Scanner.Scan()读取标准输入的一行，返回值表示是否到达eof

5. 读取文件

   使用bufio：os.Open(filename)返回文件对象，bufio.NewScanner(f)返回对该文件的Scanner

   使用ioutil：ioutil.ReadFile(filename)返回byte slice，转换为string后可通过strings.Split(text, '\n')得到每行

6. 访问URL

   通过package net，可以收发网络信息，建立网络连接，设置服务器等

   net/http中的http.Get(url)可以通过HTTP获取信息，返回一个resp struct

   resp.Body：通过iouitil.ReadAll(resp.Body)可读取其Body字段（a readable stream）的信息，读取完resp.Body后需要关闭，resp.Body.Close()

   resp.Status：HTTP响应报文的状态码

7. 并发

   goroutine之间可通过channel传递特定类型的值（由channel类型决定）

8. switch语句

   只会进入其中一个分支，若都不匹配则进入default分支，default分支的位置可以任意

9. 指针

   允许&获取地址，*解指针，但指针无法算术运算

10. 文档

    通过go doc指令可直接查看go标准库中函数等的解释

    ```shell
    $ go doc http.Get
    ```

11. 注释

    在package声明前注释本程序文件，在每个function前注释函数

12. if、for、switch允许的额外语句

    short variable declaration、increment、assignment statement、function call，如

    ```go
    if err := r.ParseForm(); err != nil {
        // err handling
    }
    ```

# Chapter2 Promgram Structure

1. 作用域：在函数内声明的变量具有局部作用域，在函数外声明的变量在整个package可见，若该变量首字母大写则package外也可以访问该变量

2. 驼峰式命名

3. 零值机制：若声明时不显式地初始化，则变量初始化为其零值，数组的各个元素被初始化为零值，struct的各个字段被分别初始化为该类型的零值

4. 初始化变量的时间：package level的变量在main函数执行前初始化，local变量在函数执行到其声明的语句时初始化

5. `:=`声明方式

   左侧允许存在已声明变量（对其进行赋值），但应至少包含一个新变量

   上述已声明变量必须是在同一个最小作用域内声明的变量，若是外层作用域声明的变量将被忽略，因而在该作用域内声明了新的变量

   `:=`是Go语言的主要声明方式，var仅在以下情况使用

   - 变量初始值不重要，将在之后为变量赋值
   - 变量类型与其初始值类型不一，必须显式地声明其类型，如

   ```go
   var temperature float64 = 19
   ```

6. 指针

   函数允许返回指向局部变量的指针，且多次运行该函数所返回的指针值不同

   alias：当为指针赋值、复制引用类型变量（map、slice等）时，都会为该变量创建新的alias

   > 问：函数允许返回指向局部变量的指针，该局部变量何时被释放？

7. 变量生命周期

   package-level变量在整个程序执行期间存活

   local变量在其声明语句执行时创建，在它变为unreachable时销毁

   Go的**垃圾回收器（GC）**负责上述local变量销毁后的内存释放等工作，其基本工作原理为：每个package-level变量或local变量都可看作是经过多个pointer/reference最终指向目标变量的路径的起点，当目标变量的上述路径都不存在时，目标变量就被视为unreachable，因而可被释放

   逃逸：当从函数返回某个局部变量的指针时，可以说该局部变量逃逸了其所在的函数，逃逸会导致额外的内存开销

   内存管理：GC为我们进行内存管理，无需手动分配/释放空间，但了解变量生命周期、内存分配/释放时机，对编写高效率程序大有帮助

   不良实践：keeping unnecessary pointers to short-lived objects within long-lived objects will prevent the GC from reclaiming the short-lived objects，若在长生命周期对象中保存了指向短生命周期对象的pointer，将导致该短生命周期对象一直无法得到释放

8. tuple assignment：常用于变量需要同时出现在赋值表达式左右两边的情况

   ```go
   a, b = b, a
   ```

   赋值表达式要求，该value能够赋予左侧变量的变量类型

9. 类型声明定义了一种新类型，它与已知的某种类型拥有相同的底层类型，能够防止不同类型的变量被错误地混用（尽管其底层类型相同）

   ```go
   type Celsius float64
   type Fahrenheit float64
   // 摄氏度和华氏度类型, 尽管其底层都是float64, 但两种类型的变量无法替换使用, 算术运算时需要显式地进行类型转换
   ```

10. 类型方法：可以为类型定义一些特定方法，如String，它将控制该类型变量被fmt打印时输出的内容

    ```go
    func (c Celsius) String() string {
        return fmt.Sprintf("%g°C", c)
    }
    ```

11. package注释：对于整个包的注释放在pacakge声明的前面，且一般整个package只有一个文件拥有该注释。额外的文档注释可以放在doc.go

12. package命名规范

    pcakge名字与其import path最后一段的名字一致

    import声明将一个名字与被引入的package绑定，默认为该package的名字，但import声明可以为其起一个别名以避免冲突（如不同路径下的两个package有相同的名字时，为其中一个package起一个别名）

13. import但不使用package会导致错误，使用goimport工具可自动管理package import，并像gofmt一样格式化代码
14. 包的初始化
    - 从初始化package-level变量开始，在解决依赖顺序后将按声明顺序初始化package-level变量
    - 如果package a引入了package b，则必须保证package b完全初始化后再初始化package a
    - package main总是最后初始化的
    - init函数：package的每个文件都可以定义一个或多个init函数，将在程序运行时按照其声明顺序首先执行init函数，init函数通常用于完成复杂的初始化工作

> Ex2.3-2.5要求比较不同PopCount算法的效率：每8位查表、每次右移1位，x&(x-1)，学习11.4节benchmark测试后补

15. 作用域

    - 语法块：全局语法块、package语法块（package-level变量）、file语法块（该文件import的package）、局部语法块（function、for、if、switch等语句、switch或select中的case标签）

    - 编译器从最内层作用域搜索变量声明，直至到达全局语法块，内层作用域会屏蔽外层作用域的同名变量声明

    - 隐式的语法块：for语句包含两个语法块，其condition以及循环体；if-else语句类似，且连续的if- else if语句的多个condition位于同一个语法块中，即

      ```go
      // 第二个if语句条件中的i在第一个if语句中的条件声明
      if i := gcd(a, b); i < 3 {
          // ...
      } else if i < 10 {
          // ...
      } else {
          // ...
      }
      ```

    - `:=`的“贪婪性”，使用`:=`声明多个变量时，即使外层作用域存在同名变量也将被屏蔽，而是在里层作用域重新声明新变量。因此如果想要使用在外层作用域声明的变量，则不要使用`:=`，应该先声明其他变量后使用tuple assignment

      ```go
      package main
      
      import "fmt"
      
      var x int = 1
      
      func f() {
      	x, y := 2, 3 // 此处的x具有局部作用域, 因此不会改变package-level变量x的值
      	fmt.Printf("%d %d\n", x, y)
      }
      
      func main() {
      	f()
      	fmt.Printf("%d\n", x) // 输出结果为1
      }
      ```

16. 关于init()的测试

    下列程序的初始化过程为，变量x、y先根据零值机制初始化为0、0，然后执行第一个init()令y变为0，再执行第二个init()令x变为1，最终在main()执行前x=1，y=0

    ```go
    package main
    
    import "fmt"
    
    var x, y int
    
    func init() {
    	y = x
    }
    
    func init() {
    	x = 1
    }
    ```


# Chapter3 Basic Data Type

1. Go的类型分为四种：基础类型、复合类型（数组、struct）、引用类型（slice、map）、接口类型

2. int8、int16、int32、int64，int根据编译器可能是32位或64位的，但它与int32、int64属于不同类型，不能一起算术运算

3. 无符号数只应该在位运算等特殊场合使用，即使一个变量的意义保证其为非负数，一般也用有符号数保存（如slice长度等）

4. %结果与被取模数符号一致

5. 左移`<<`会在低位补0，右移`>>`会在最高位补0或1（对于有符号数，与符号位一致）

6. 使用fmt打印整数

   - `%d`十进制、`%o`八进制、`%x`十六进制
   - `[1]`表示重复使用第一个操作数
   - `#`表示打印八/十六进制数时带上前缀

   ```go
   v := 16
   fmt.Printf("%d %[1]o %#[1]X", v)	// 16 20 0x10
   ```

7. float32和float64，一般使用后者，前者有效bit位仅有23位，常造成精度不足问题

8. math package提供了+Inf、-Inf和NaN值，并有IsNaN方法判断值是否非数字。但是最好不要直接返回NaN代表计算错误，而是返回一个额外的boolean值，因为NaN之间的比较总是返回false

   ```go
   nan := Math.NaN()
   fmt.Println(nan == nan, nan < nan, nan > nan) // false false false
   ```

> 3.2、3.3节的练习题跳过

9. Go的complex、real、imag函数用于处理复数

   ```go
   x := complex(1, 2)
   x := 1 + 2i			// 两个声明等价
   fmt.Printf("%d %d\n", real(x), imag(x)) // 1 2
   ```

   复数类型有complex64和complex128两种，分别使用了float32、float64作为实部、虚部的数据类型

10. boolean值不会隐式转换为0或1

11. 字符串
    - len(s)返回字符串s的字节数（而不是字符数）
    - 通过s[i]可以获取字符串s的第i个byte，但是不一定是其第i个字符，因为非ASCII字符的UTF8编码需要使用两个或多个字节

12. 使用s[i]访问字符串的字节时，若i超出了字符串的范围会导致panic

    使用s[i: j]获取子字符串时，若i或j超出范围，或者i大于j会导致panic

13. 复制字符串、获取子字符串操作都不会导致额外的空间分配，该字符串与其**副本或子字符串**变量**共享**同一份存储中的数据

14. 原生字符串字面值：字符串字面值中以\开头的转义字符具有特殊含义，原生字符串字面值（raw string literal）通过反引号相括，不进行任何转义

15. Unicode：Unicode的诞生是因为ASCII只包含128个字符，无法表示世界上许多国家的字符，后制定的Unicode code point几乎能囊括世界上国家的字符。Unicode code point使用int32（Go中也称为rune）进行存储

16. UTF-8：一种将字符串编码为字节序列的变长编码方式

    - 原理：ASCII码编码为1个字节，以0开头；对于非ASCII字符，编码为2-4个字节，第一个字节必定以1开头，且当编码为2字节时以110开头，当编码为3字节时以1110开头，当编码为4字节时以11110开头，其后的字节以10开头

    - Go的range循环应用于字符串时，会隐式地执行UTF-8解码，或者调用unicode/utf8 package的DecodeRuneInString执行解码
    - 当UTF-8解码遇到错误输入时，将返回特殊字符\uFFFD表示错误，打印结果为黑色六边形中含有白色问号�
    - Go的源文件统一采用UTF-8编码

17. Go的四个基础包用于进行字符串操作

    strings提供字符串搜索、替换、对比、剪切、分割、聚合等操作，其方法有

    ``` go
    func Contains(s, substr string) bool
    func Count(s, sep string) int
    func Fields(s string) []string
    func HasPrefix(s, prefix string) bool
    func Index(s, sep string) int
    func Join(a []string, sep string) string
    ```

    bytes与strings类似，但是针对[]byte类型

    strconv实现基本类型与其字符串的相互转换

    - 数字转字符串：fmt.Sprintf或strconv.Itoa
    - 字符串转数字：strconv.Atoi或strconv.ParseInt

    unicode常用于判断rune是否字母、数字、大小写，以及大小写转换等

18. bytes package提供了bytes.Buffer类型，用于构建[]bytes（字符串），常用方法有

    ```go
    func WriteString(s string)
    func WriteRune(c rune)	// 对于Unicode码点, 优先使用WriteRune
    func WriteByte(b byte)
    ```

19. 常量

    - 常量表达式在编译时计算其值，不可修改
    - 常量可以作为类型的 一部分，如具有常量大小的数组

    ```go
    const IPv4Len = 4
    var p [IPv4Len]byte
    ```

    - 使用()声明一组常量时，允许某个常量省略其等式右边，代表与上一个常量拥有相同的值和类型

    ```go
    const (
    	Alice int64 = 0
        Bob			// Bob is 0 int64 type, the same as Alice
    )
    ```

    - iota常量生成器：生成枚举类型、定义标志变量中不同bit的含义等

    ```go
    type Weekday int
    const (
    	Sunday Weekday = iota	// iota的值从0开始不断递增
    	Monday
    	Tuesday
    	Wednesday
    	Thursday
    	Friday
    	Saturday
    )
    ```

20. Untyped Constants常量面值

    - 诸如0、0.0、\000、0i、true、"hello"等常量面值分别是untyped integer、untyped floating-point、untyped rune、untyped complex、untyped boolean、untyped string
    - untyped constants被赋值给变量或出现在显式指明类型的变量声明中时，会转换为该类型
    - 如果untyped constants出现在未显式指明类型的变量声明中时，则该变量类型为：untyped integer对应int（底层可能为32或64位）、untyped floating-point对应float64、untyped complex对应complex128、untyped rune对应rune(int32)

# Chapter4 Composite Types

1. 四种常用的复合类型：array、slice、struct、map

2. 数组声明时需要指定数组大小和元素类型，数组具有固定大小，不可增加或删除元素，只可以修改元素的值

3. 数组列表初始化方式：允许使用{}括起的列表（数组字面值）初始化数组，且使用这种方式声明的数组可以将其类型中的大小信息替换为`...`，并由初始值列表大小决定数组大小

   ```go
   var x [10]int = [10]int{1, 2, 3}
   y := [...]int{1, 2, 3}
   ```

4. 数组传参问题

   Go中函数传参时会拷贝数组，效率较低

   通过传入数组的指针而不是数组本身，使得函数可以对数组元素进行修改

5. 数组or slice：但由于数组具有固定大小，无法动态增删元素，除了SHA256摘要算法等场景会使用外，一般选择slice而不是array

6. slice由指针、len、cap三部分组成，slice底层引用了某个数组

7. 切片操作：切片运算符`s[i:j]`中的s可以是数组、数组指针或者slice，并返回一个新的slice。切片超出cap(s)将导致panic，但只是超出len(s)意味着扩展了slice，不一定会panic（取决于s的底层数组）

8. slice的复制：slice的复制只是为该底层数组创建了一个新的alias，没有额外的内存分配和数据拷贝，新的slice与旧的slice的指针指向底层数组相同位置，len与cap相等

9. slice不可比较

   - slice是间接引用，slice甚至可以自包含
   - 对于chan、指针等引用类型的==操作会比较两者是否引用同一个对象，如果slice的比较这么定义则与array的比较显著不同（但slice与array又密切相关，容易造成混淆）
   - 唯一允许的slice比较是slice==nil，代表该slice没有指向任何底层数组。但是len=0的slice不一定是nil，即其指针指向某个数组元素，但是len和cap为0。一般地，Go函数应该平等地对待nil slice和零长度的非nil slice

10. make创建slice

    make创建了一个匿名的底层数组，并创建了slice引用该数组

    ```go
    make([]T, len)		// 不指定cap时, cap与len一致
    make([]T, len, cap)
    ```

11. append

    append可能导致内存分配：append时若底层数组容量不足，会重新创建一个容量double的底层数组，将数据进行拷贝，然后让slice指向新的数组并返回

    append返回值：Go中append使用的策略可能更复杂，我们无法对append是否导致内存分配进行任何假设，因此每次append对应该将其返回值赋予原slice变量

    复杂度：append发生内存分配时容量double的原因是，保证append单个元素的均摊时间复杂度为O(1)

    append多个元素或一个slice：由于Go内置的append是一个variadic函数，即可以接受变长参数，因此可以append多个元素甚至append另一个slice。

    ```go
    func appendInt(x []int, y ...int) []int {
        var z []int
        zlen := len(x) + len(y)
        // expand z to at least zlen
        if zlen <= cap(x) {
            z = x[:zlen]
        } else {
            zcap := zlen
            if zlen <= 2 * len(x) {
                zcap = 2 * len(x)
            }
            z = make([]int, zlen, zcap)
            copy(z, x)
        }
        copy(z[len(x):], y)
        return z
    }
    ```

> Ex4.7，使用原地算法翻转UTF-8编码的字符串，参数类型为[]byte
>
> Sol：首先用utf8.DecodeRune解码每个rune，然后翻转该rune对应的1-4个byte，最后反转整个[]byte
>
> 总结：这个练习题设计巧妙，结合了UTF-8编码知识，以及slice原地修改的技巧。初看之下，由于UTF-8中每个rune占用的byte数量不一定，因此要设计原地算法颇有难度

12. map

    - 底层为哈希表：map是对哈希表的引用，nil map代表其没有引用任何哈希表

    - key可比较：map要求key类型必须能用`==`比较。如slice是不可比较的，不能直接作为map的key类型，但可以通过辅助函数将其转换为可比较类型（如string）再存入map，但必须保证k(s1)==k(s2)当且仅当s1==s2

    - 创建map

      ```go
      var f1 map[int]int	// nil map
      f2 := map[int]int{}	// map with no elements, but not nil map
      f3 := make(map[int]int) // not nil
      ```

    - 删除map中的元素

      ```go
      f := make(map[int]int)
      delete(f, 1)
      ```

      对nil map执行删除是安全的，实际不会做任何事

    - map元素不是变量：不能对map元素取地址，因为map元素增长时可能导致rehash，使得元素的地址改变

    - 遍历元素顺序随机：使用range遍历map中的元素的时候顺序是不确定的，每次遍历的顺序都是随机的

    - lookup操作：可以返回两个值，分别是value值和布尔值，布尔值代表该key是否存在于map中，若不存在则第一个返回值是该value类型的zero value

    - nil map：对nil map执行lookup、删除、len、range操作都是安全的，但是尝试**将元素存入nil map会导致panic**。对nil map执行lookup时，返回的时该map的value类型的零值

    - no set type：Go中没有set类型，map代替完成其工作

    - map的value也可以是复合类型

      ```go
      var f map[string]map[string]bool
      f["abc"]["def"]	
      // 此lookup操作是安全的, f本身是nil map, 因此第一个lookup会返回其value类型map[string]bool的zero value, 也是一个nil map, 第二个lookup则对返回的nil map操作得到其value类型bool的zero value, 因此最终结果为false
      ```

12. Structs

    点操作符：struct变量、struct指针可以通过点操作符获取其成员

    成员的顺序：struct的成员顺序影响struct类型

    ```go
    type Employee struct {
    	ID 		int
    	Name	string
    	Age 	int
    }
    
    type Employee1 struct {
    	ID 		int
    	Age 	int
    	Name 	string
    }
    ```
    
    不可自包含：类型为S的结构体不可以包含类型为S的成员，但可以包含*S指针类型的成员，常用于定义链表、树等结构
    
    空struct：不含任何成员的struct，写作struct{}
    
    零值：零值的struct的各个成员均为各自类型的零值
    
    struct面值：struct面值初始化struct有两种形式，分别是按顺序列出所有成员值；或列出任意成员的名字以及值（常用）
    
    ```go
    type Point struct {
        X, Y int
    }
    p1 := Point{X: 1, Y: 2}
    p2 := Point{1, 2}
    ```
    
    struct比较：如果每个成员都是可比较的，且每个成员的值都相等，则两个struct相等
    
12. struct嵌入和匿名成员

    struct声明时可以声明最多一个**匿名成员**，即声明其类型但不指定名字，该类型必须是一个命名类型或指向命名类型的指针类型

    ```go
    type Point struct {
        X, Y int
    }
    type Circle struct {
        Point
        Radius int
    }
    // Point类型被嵌入到了Circle结构体
    ```
    
    对于匿名成员中的类型可以直接用点运算符访问
    
    ```go
    var c Circle
    c.X	 	// 还可以写作c.Point.X, 可以认为匿名成员并非真正匿名, 其名字就是它的类型的名字
    c.Y		// 还可以写作c.Point.Y
    ```

    对于包含匿名成员的结构体字面值

    ```go
    var c Circle
    c = Circle{Point{1, 1}, 5}
    c = Circle{
        Point: Point{X: 1, Y: 1},
        Radius: 5,
    }
    ```
    
    匿名成员的方法：匿名成员并非一定是结构体类型，任何命名类型都可以作为匿名成员。使用点运算符不仅能访问匿名成员嵌套的成员，**还可以访问匿名成员的方法**。通过这种方式实现的**组合**是Go语言OOP的核心
    
12. JSON是JavaScript对象表示法，常用作为传输结构化信息的格式。Go提供了encoding/json package用于编解码JSON

    JSON的基础类型是numbers、booleans和strings，此外还有JSON array和JSON object类型

    JSON object类型可以与Go struct类型相互转换，为此可以在声明Go struct时为成员加上field name，表示该成员与JSON中哪个名字的成员匹配。用于与JSON转换的Go struct中的成员应该是exported的

    ```go
type Movie struct {
        Title 	string
        Year 	int 	`json:"released"`
        Color 	bool 	`json:"color,omitempty"`
        Actors 	[]string
    }
// field name中的omitempty表示如果该成员是空或零值，则生成的JSON不包含该字段
    ```

12. Marshal与Unmarshal

    marshal表示将Go struct转换为JSON，encoding/json package中的json.Marshal和json.MarshalIndent提供了此功能

    ```go
    data, err := json.Marshal(movies)
    data, err := json.MarshalIndent(movies, "", "	")
    // Marhsal生成的JSON不含任何空白字符, 而MarshalIndent生成的JSON打印时具有每行前缀(第二个参数)和缩紧(第三个参数)
    ```
    
    unmarshal是将JSON转换为Go struct。通过定义Go struct可以选择性解码JSON中感兴趣的成员，即unmarshal只会将JSON中能与Go struct成员名字匹配的数据保存。且该名字匹配是忽略大小写的，因此如果Go struct成员名字和JSON名字只有大小写不同，无需额外定义feild tag，通常是在JSON名字包含下划线时才定义field tag
    
    ```go
    var titles []struct{Title string}
    if err := json.unmarshal(data, &titles); err != nil {
        // err handling
    }
    ```
    
17. 除了json.Marshal和json.Unmarshal外，Go还提供了基于流式的编解码器json.Decoder和json.Encoder，能够从一个输入流解码JSON数据，或者针对一个输出流编码JSON对象

# Chapter5 Functions

1. GC不做什么：Go的GC只会回收不再使用的内存，不会回收OS层面的资源，如打开的文件、网络连接等，程序必须显式地释放这些资源

2. 可以讲返回多个值的函数调用，作为调用另一个接受多个参数的函数的唯一参数，常用于debug时的log

   ```go
   log.Println(findLinks(url))
   // the same as
   links, err := findLinks(url)
   log.Println(links, err)
   ```

3. bare return：如果函数的每个返回值都命名了，则可以使用一个只含return的语句代替原return语句。bare return减少代码重复的同时，也可能使得代码更难以阅读，应谨慎使用
