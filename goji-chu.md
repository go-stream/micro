52-82

1、程序结构

{

命名

{

所有命名规则：必须以unicode字符或下划线开头，后跟任意字母数字下划线，大写开头表示可导出（外部可以调用）

    关键字不可自定义，只能在特定语法结构 25个

        if else for switch case default fallthrough

        break goto return continue

        package import

        type func interface const var struct map chan

        defer go select range







    预定义名字可以修改 ，对应内建的常量、类型和函数 37个

    内建常量:

        true false iota nil

    内建类型:

        int int8 int16 int32 int64

        uint uint8 uint16 uint32 uint64 uintptr\(保存指针\)

        float32 float64 complex128 complex64

        bool byte rune\(int的别名\) string error

    内建函数:

        make len cap new append copy close delete

        complex real imag

        panic recover

	  函数内作用域：名字是在函数内部定义，只在函数内部有效。

    包内作用域：在函数外部定义，在当前包的所有文件中都可以访问。

    包外作用域：在函数外部定义的名字开头字母的大写，在包外可访问。

    包本身的名字总是用小写字母。



    名字的长度没有逻辑限制，风格是尽量使用短小的名字，如果一个名字的作用域比较大，生命周期也比较长，建议名字长一些有意义。

    推荐使用 驼峰式 命名，缩略词则避免使用大小写混合的写法：HTMLEscape或escapeHTML

	

}



声明

{

四种类型的声明语句：var、const、type和func，分别对应变量、常量、类型和函数实体对象的声明。

每个源文件以包的声明语句开始，说明该源文件是属于哪个包。包声明语句之后是import语句导入依赖的其它包，

然后是包一级的类型、变量、常量、函数的声明语句，包一级的各种类型的声明语句的顺序无关紧要



在包一级声明语句声明的名字可在整个包对应的所有源文件中访问。

一个函数的声明由一个函数名字、参数列表 、一个可选的返回值列表和包含函数定义的函数体组成。	

}



变量

{

var 变量名字 类型 = 表达式    类型”或“= 表达式”两个部分可以省略其中的一个

var str = "111"   根据初始化表达式来推导变量的类型信息

var str string    用零值初始化该变量

零值初始化机制让Go语言中不存在未初始化的变量

数值类型变量对应的零值是0，布尔类型变量对应的零值是false，字符串类型对应的零值是空字符串，

接口或引用类型（包括slice、map、chan和函数） 变量对应的零值是nil。

数组或结构体等聚合类型对应的零值是每个元素或字段都是对应该类型的零值。

var i, j, k int // int, int, int

var b, f, s = true, 2.3, "four" // bool, float64, string

包级别声明的变量会在main入口函数执行前完成初始化，局部变量将在声明语句被执行到的时候完成初始化。



在函数内部，有一种称为简短变量声明语句的形式可用于声明和初始化局部变量。



简短变量声明被广泛用于大部分的局部变量的声明和初始化（左侧有已经声明的变量时表示赋值）

var形式的声明语句往往是用于需要显式指定变量类型地方，或者因为变量稍后会被重新赋值而初始值无关紧要的地方。



变量：

	又称可寻址的值（不是所有的值有内存地址，但是变量肯定有）

指针：

	一个变量对应一个保存了变量对应类型的值的内存空间。变量声明时会绑定到一个变量名

	每次我们对一个变量取地址，或者复制指针，我们都是为原变量创建了新的别名。例如， \*p 就是是 变量v的别名

	var x int

	&x表达式会产生一个指向x的指针，指针类型是\*int。  var p \*int = &x    \*p是取x的值

	任何类型的指针的零值都是nil，只有当它们指向同一个变量或全部是nil时才相等

	不仅仅是指针会创建别名，很多其他引用类型也会创建别名，例如slice、map和chan，甚至结构体、数组和接口都会创建所引用变量的别名。

	指针是实现标准库中flag包的关键技术	

	



	

}



赋值

{

另一个创建变量的方法是调用用内建的new函数，new函数类似是一种语法糖，和普通变量声明一样，只是不需要临时变量名了

表达式new\(T\)将创建一个T类型的匿名变量，初始化为T类型的零值，然后返回变量地址，返回的指针类型为 \*T

p := new\(int\)

new函数使用常见相对比较少，因为对应结构体来说，可以直接用字面量语法创建新变量的方法会更灵活



对于在包一级声明的变量来说，它们的生命周期和整个程序的运行周期是一致的

在局部变量的声明周直到该变量不再被引用为止，变量的存储空间可能被回收。

函数的参数变量和返回值变量都是局部变量。它们在函数每次被调用的时候创建



数值变量也可以支持 ++ 递增和 -- 递减语句（语句不是表达式, x = i++ 之类的表达式是错误的）



元组赋值是另一种形式的赋值语句，它允许同时更新多个变量的值在赋值之前，

赋值语句右边的所有表达式将会先进行求值，然后再统一更新左边对应变量的值。



类型必须完全匹配，nil可以赋值给任何指针或引用类型的变量	

	

}



类型

{

type 类型名字 底层类型

一个类型声明语句创建了一个新的类型名称，和现有类型具有相同的底层结构。

新命名的类型提供了一个方法，用来分隔不同概念的类型，这样即使它们底层类型相同也是不兼容的。

类型声明语句一般出现在包一级，因此如果新创建的类型名字的首字符大写，则在外部包也可以使用。



类型转换操作不是函数调用。类型转换不会改变值本身，仅仅是改变值的类型,使它们的语义发生变化。



对于每一个类型T，都有一个对应的类型转换操作T\(x\)，用于将x转为T类型（如果T是指针类型，\(\*int\)\(0\) ） 。

只有当两个类型的底层基础类型相同时，才允许这种转型操作，或者是两者都是指向相同底层结构的指针类型，这些转换只改变类型而不会影响值本身。



数值类型之间的转型也是允许的，并且在字符串和一些特定类型的slice之间也是可以转换

这类转换可能改变值的表现:将一个浮点数转为整数将丢弃小数部分，将一个字符串转为 \[\]byte 类型的slice将拷贝一个字符串数据的副本。

在任何情况下，运行时不会发生转换失败的错误（错误只会发生在编译阶段） 。

底层数据类型决定了内部结构和表达方式，也决定是否可以像底层类型一样对内置运算符的支持。

这意味着，Celsius和Fahrenheit类型的算术运算行为和底层的float64类型是一样的，正如我们所期望的那样。



一个命名的类型可以提供书写方便

命名类型还可以为该类型的值定义新的行为,这些行为表示为一组关联到该类型的函数集合，我们称为类型的方法集	

}



包和文件

{

gopl.io/ch1/helloworld对应的目录路径是$GOPATH/src/gopl.io/ch1/helloworld

每个包都对应一个独立的名字空间



gopl.io/ch2/tempconv包的名字一般是tempconv

package tempconv





导入时：\(导入时不使用会编译报错\)

import  gopl.io/ch2/tempconv

tempconv.xxx



包初始化：

每个文件都可以包含多个init初始化函数，在程序开始执行时按照它们声明的顺序被自动调用

init初始化函数除了不能被调用或引用外，其他行为和普通函数类似

func init\(\) { }



每个包在解决依赖的前提下，以导入声明的顺序初始化，每个包只会被初始化一次。

如果一个p包导入了q包，那么在p包初始化的时候可以认为q包必然已经初始化过了。

初始化工作是自下而上进行的，main包最后被初始化。可以确保在main函数执行之前，所有依然的包都已经完成初始化工作了。	

}



作用域

{

一个声明语句将程序中的实体和一个名字关联，比如一个函数或一个变量。

声明语句的作用域是指源代码中可以有效使用这个名字的范围。

语法决定了内部声明的名字的作用域范围



对于内置的类型、函数和常量，比如int、len和true等是在全局作用域的。

对于导入的包，例如tempconv导入的fmt包，则是对应源文件级的作用域，因此只能在当前的文件中访问导入的

fmt包，当前包的其它源文件无法访问在当前源文件导入的包。



控制流标号，就是break、continue或goto语句后面跟着的那种标号，则是函数级的作用域。



当编译器遇到一个名字引用时，如果它看起来像一个声明，它首先从最内层的词法域向全局的作用域查找。

如果查找失败，则报告“未声明的名字”这样的错误。	

}	



代码逻辑

{

/\* 定义局部变量 \*/

var a int = 10

/\* 使用 if 语句判断布尔表达式 \*/

if a &lt; 20 {

	 /\* 如果条件为 true 则执行以下语句 \*/

	 fmt.Printf\("a 小于 20\n" \)

}



/\* 局部变量定义 \*/

var a int = 100;

/\* 判断布尔表达式 \*/

if a &lt; 20 {

	 /\* 如果条件为 true 则执行以下语句 \*/

	 fmt.Printf\("a 小于 20\n" \);

} else {

	 /\* 如果条件为 false 则执行以下语句 \*/

	 fmt.Printf\("a 不小于 20\n" \);

}



switch marks {

	case 90: grade = "A"

	case 80: grade = "B"

	case 50,60,70 : grade = "C"

	default: grade = "D"  

}



switch {

	case grade == "A" :

		 fmt.Printf\("优秀!\n" \)     

	case grade == "B", grade == "C" :

		 fmt.Printf\("良好\n" \)      

	case grade == "D" :

		 fmt.Printf\("及格\n" \)      

	case grade == "F":

		 fmt.Printf\("不及格\n" \)

	default:

		 fmt.Printf\("差\n" \);

}





var x interface{}

     

switch i := x.\(type\) {

	case nil:      

		 fmt.Printf\(" x 的类型 :%T",i\)                

	case int:      

		 fmt.Printf\("x 是 int 型"\)                       

	case float64:

		 fmt.Printf\("x 是 float64 型"\)           

	case func\(int\) float64:

		 fmt.Printf\("x 是 func\(int\) 型"\)                      

	case bool, string:

		 fmt.Printf\("x 是 bool 或 string 型" \)       

	default:

		 fmt.Printf\("未知型"\)     

}   



select {

	case i1 = &lt;-c1:

		 fmt.Printf\("received ", i1, " from c1\n"\)

	case c2 &lt;- i2:

		 fmt.Printf\("sent ", i2, " to c2\n"\)

	case i3, ok := \(&lt;-c3\):  // same as: i3, ok := &lt;-c3

		 if ok {

				fmt.Printf\("received ", i3, " from c3\n"\)

		 } else {

				fmt.Printf\("c3 is closed\n"\)

		 }

	default:

		 fmt.Printf\("no communication\n"\)

}



for true  {

		fmt.Printf\("这是无限循环。\n"\);

}



var b int = 15

var a int

numbers := \[6\]int{1, 2, 3, 5} 

/\* for 循环 \*/

for a := 0; a &lt; 10; a++ {

	fmt.Printf\("a 的值为: %d\n", a\)

}

for a &lt; b {

	a++

	fmt.Printf\("a 的值为: %d\n", a\)

}

for i,x:= range numbers {

	fmt.Printf\("第 %d 位 x 的值 = %d\n", i,x\)

}



/\* 定义局部变量 \*/

var a int = 10

/\* 循环 \*/

LOOP: for a &lt; 20 {

	if a == 15 {

		 /\* 跳过迭代 \*/

		 a = a + 1

		 goto LOOP

	}

	fmt.Printf\("a的值为 : %d\n", a\)

	a++     

}  

}



}



83-118

//Go语言将数据类型分为四类：基础类型、复合类型、引用类型和接口类型。

2、基础数据结构

{



整型

{

int8、int16、int32和int64  -128~127 。。。

uint8、uint16、uint32和uint64 0~255 。。。

分别对应8、16、32、64bit大小的有无符号整形数

对应特定CPU平台机器字大小的有符号和无符号整数int和uint  32/64

Unicode字符rune类型是和int32等价的类型，通常用于表示一个Unicode码点。这两个名称可以互换使用。

同样byte也是uint8类型的等价类型，byte类型一般用于强调数值是一个原始的数据而不是一个小的整数。

无符号的整数类型uintptr，没有指定具体的bit大小但是足以容纳指针。（底层编程才用）

int、uint和uintptr是不同类型的兄弟类型



二元运算符有五种优先级。在同一个优先级，使用左优先结合规则：

\* / % &lt;&lt; &gt;&gt; & &^\(位清空 \(AND NOT\)\)

+ - \| ^

== != &lt; &lt;= &gt; &gt;=

&&

\|\|



//%仅用于整数

//对于整数，+x是0+x的简写，-x则是0-x的简写；对于浮点数和复数，+x就是x，-x则是x 的负数

//无符号数往往只有在位运算或其它特殊的运算场景才会使用

//任何大小的整数字面值都可以用以0开始的八进制格式书写，例如0666；或用以0x或0X开头的十六进制格式书写，例如0xdeadbeef。%d、%o或%x参数控制输出的进制格式

//字符使用 %c 参数打印，或者是用 %q 参数打印带单引号的字符

//常Printf格式化字符串包含多个%参数时将会包含对应相同数量的额外操作数，但是%之后的 \[1\] 副词告诉Printf函数再次使用第一个操作数。第二，%后的 \# 副词告诉Printf在用%o、%x或%X输出时生成0、0x或0X前缀。

fmt.Printf\("%d %\[1\]x %\#\[1\]x %\#\[1\]X\n", x\)

}



浮点型

{

float32和float64

%g参数打印浮点数

表格的数据，使用%e（带指数） 或%f的形式打印可能更合适	

fmt.Printf\("x = %d e^x = %8.3f\n", x, math.Exp\(float64\(x\)\)\)

NaN非数，一般用于表示无效的除法操作结果0/0或Sqrt\(-1\)

math.IsNaN用于测试一个数是否是非数NaN

}



复数

{

complex64和complex128，分别对应float32和float64两种浮点数精度	

}



布尔型

{

	一个布尔类型的值只有两种：true和false。

}



字符串

{

一个字符串是一个不可改变的字节序列。字符串的值是不可变的

文本字符串通常被解释为采用UTF8编码的Unicode码点（rune） 序列	

内置的len函数可以返回一个字符串中的字节数目（不是rune字符数目） ，

索引操作s\[i\]返回第i个字节的字节值，i必须满足0 ≤ i&lt; len\(s\)条件约束。

超出字符串索引范围的字节将会导致panic异常

第i个字节并不一定是字符串的第i个字符，因为对于非ASCII字符的UTF8编码会要两个或多个字节。

子字符串操作s\[i:j\]基于原始的s字符串的第i个字节开始到第j个字节（并不包含j本身） 

字符串可以用==和&lt;进行比较；比较通过逐个字节比较完成的，因此比较的结果是字符串自然编码的顺序



Go语言源文件总是用UTF8编码，并且Go语言的文本字符串也以UTF8编码的方式处理，

因此我们可以将Unicode码点也写到字符串面值中



原生的字符串用\`



现在utf8已经是Unicode的标准，UTF8编码使用1到4个字节来表示每个Unicode码点



标准库中有四个包对字符串处理尤为重要：bytes、strings、strconv和unicode包



strings包提供了许多如字符串的查询、替换、比较、截断、拆分和合并等功能。

bytes包提供了针对和字符串有着相同结构的\[\]byte类型。因为字符串是只读的，因此逐步构建字符串会导致很多分配和复制。在这种情况下，使用bytes.Buffer类型将会更有效。

strconv包提供了布尔型、整型数、浮点数和对应字符串的相互转换，还提供了双引号转义相关的转换。

unicode包提供了IsDigit、IsLetter、IsUpper和IsLower等类似功能，它们用于给字符分类。每

个函数有一个单一的rune类型的参数，然后返回一个布尔值。而像ToUpper和ToLower之类的

转换函数将用于rune字符的大小写转换。所有的这些函数都是遵循Unicode标准定义的字母、数字等分类规范。strings包也有类似的函数，它们是ToUpper和ToLower，将原始字符串的每个字符都做相应的转换，然后返回新的字符串。





}



常量	

{

常量表达式的值在编译期计算，而不是在运行期。

const \(

e = 2.71828182845904523536028747135266249775724709369995957496696763

pi = 3.14159265358979323846264338327950288419716939937510582097494459

\)



iota 常量生成器

const \(

	Sunday Weekday = iota

	Monday

	Tuesday

	Wednesday

	Thursday

	Friday

	Saturday

\)







	

}



}

//基本类型聚合为复合类型



3、复合数据结构

//数组、slice、map和结构体字面值的写法都很相似。

//crypto/sha256包的Sum256函数对一个任意的字节slice类型的数据生成一个对应的消息摘要

{

数组-聚合类型

{

var a\[3\]int

q := \[...\]int{1, 2, 3} 根据初始化值的个数来计算

r := \[...\]int{99: -1}  100个元素的数组r，最后一个初始化为-1，其它元素都是用0初始化。



由同构的元素组成——每个数组元素都是完全相同的类

数组是一个由固定长度的特定类型元素组成的序列

内置的len函数将返回数组中元素的个数

数组的长度是数组类型的一个组成部分,数组的长度需要在编译阶段确定,必须是常量表达式，

\[3\]int和\[4\]int是两种不同的数组类型



Go语言中很少直接使用数组

和数组对应的类型是Slice（切片） ,它是可以增长和收缩动态序列

	

}

//slice和map则是动态的数据结构

slice

{

一个slice类型一般写作\[\]T

slice由三个部分构成：指针、长度和容量\(长度不能大于容量\)  内置的len和cap函数

s\[i:j\] 引用s的从第i个元素开始到第j-1个元素的子序列

i省略使用0代替，j省略用len\(s\)代替。



字符串的切片操作和\[\]byte字节类型切片的切片操作是类似的



slice之间不能比较	（bytes.Equal函数来判断两个字节型slice是否相等（\[\]byte））

一个nil值的slice的长度和容量都是0，但是也有非nil值的slice的长度和容量也是0的

len\(s\) == 0来判断slice是否为空



make函数（make创建了一个匿名的数组变量，然后返回一个slice）

make\(\[\]T, len\)

make\(\[\]T, len, cap\)



runes = append\(runes, r\)内置的append函数用于向slice追加元素，不知道是否内存重新分配



一个slice可以用来模拟一个stack

}



map

{

map之间也不能进行相等比较	

	

哈希表是一种巧妙并且实用的数据结构。它是一个无序的key/value对的集合

在Go语言中，一个map就是一个哈希表的引用，map类型可以写为map\[K\]V

map中的元素并不是一个变量，因此我们不能对map的元素进行取址操作



ages := make\(map\[string\]int\) 



ages := map\[string\]int{}

ages := map\[string\]int{"alice": 31,"charlie": 34,}

delete\(ages, "alice"\) 



遍历（随机的）

for name, age := range ages {

	fmt.Printf\("%s\t%d\n", name, age\)

}



map的下标语法将产生两个值；第二个是一个布尔值，用于报告元素是否存在

if age, ok := ages\["bob"\]; !ok { /\* ... \*/ }





显式地对key进行排序，用slice

import "sort"

var names \[\]string

for name := range ages {

	names = append\(names, name\)

} 

sort.Strings\(names\)

for \_, name := range names {

	fmt.Printf\("%s\t%d\n", name, ages\[name\]\)

}

	

}



结构体-聚合类型

{

由异构的元素组成的。数组和结构体都是有固定内存大小的数据结构。	

所有的成员也同样是变量



type Employee struct {

	ID int

	Name string

	Address string

	DoB time.Time

	Position string

	Salary int

	ManagerID int

} 

var dilbert Employee



type Point struct{ X, Y int }

p := Point{1, 2}

但是常用下面的方式

p := Point{X:1,Y;2} //成员被忽略的话将默认用零值



结构体类型的零值是每个成员都对是零值。



如果结构体没有任何成员的话就是空结构体，写作struct{}。它的大小为0，但是有时候依然是有价值的。

有些Go语言程序员用map模拟set数据结构时，用它来代替map中布尔类型的value，只是强调key的重要性，但是因为节约的空间有限，而且语法比较复杂，所有我们通常避免这样的用法。



所有的函数参数都是值拷贝传入的,需要内部修改的必须传指针



匿名成员



}



-------------------

json格式

{

标准库中的encoding/json、encoding/xml、encoding/asn1等包提供支持	

Protocol Buffers的支持由github.com/golang/protobuf 包提供	



JavaScript对象表示法（JSON） 是一种用于发送和接收结构化信息的标准协议。



基本的JSON类型有数字（十进制或科学记数法） 、布尔值（true或false） 、字符串



基础类型可以通过JSON的数组和对象类型进行递归组合

JSON的对象类型可以用于编码Go语言的map类型	



Year和Color成员后面的字符串面值是结构体成员Tag

type Movie struct {

	Title string

	Year int \`json:"released"\`

	Color bool \`json:"color,omitempty"\`

	Actors \[\]string

}

一个构体成员Tag是和在编译阶段关联到该成员的元信息字符串

结构体的成员Tag可以是任意的字符串面值，但是通常是一系列用空格分隔的key:"value"键值对序列；

因为值中含义双引号字符，因此成员Tag一般用原生字符串面值的形式书写。

json开头键名对应的值用于控制encoding/json包的编码和解码的行为，

并且encoding/...下面其它的包也遵循这个约定。成员Tag中json对应值的第一部分用于指定JSON对象的名字，

如TotalCount成员对应到JSON中的total\_count对象。Color成员的Tag还带了一个额外的omitempty选项，表示结构体成员为空或零值时不生成JSON对象（这里false为零值）







//Marshal函数返还一个编码后的字节slice，包含很长的字符串，并且没有空白缩进；

data, err := json.Marshal\(movies\)

if err != nil {

	log.Fatalf\("JSON marshaling failed: %s", err\)

} 

fmt.Printf\("%s\n", data\)

//json.MarshalIndent函数将产生整齐缩进的输出



解密：json.Unmarshal函数完成

}



文本和html模板

{

text/template和html/template等模板包提供的，它们提供了一个将变量值填充到一个文本或HTML格式的模板的机制。

	

}

	

}



4、函数

{

函数声明

{

func name\(parameter-list\) \(result-list\) {

	body	

}

func add\(x int, y int\) int {return x + y}

func sub\(x, y int\) \(z int\) { z = x - y; return}

func first\(x int, \_ int\) int { return x }

func zero\(int, int\) int { return 0 }

fmt.Printf\("%T\n", add\) // "func\(int, int\) int"

fmt.Printf\("%T\n", sub\) // "func\(int, int\) int"

fmt.Printf\("%T\n", first\) // "func\(int, int\) int"

fmt.Printf\("%T\n", zero\) // "func\(int, int\) int"



函数的类型被称为函数的标识符。

没有默认参数值，所有形参和返回值的变量名对于函数调用者而言没有意义

实参通过值的方式传递，因此函数的形参是实参的拷贝。

实参包括引用类型，如指针，slice\(切片\)、map、function、channel等类型，实参可能会由于函数的间接引用被修改。



没有函数体的函数声明，这表示该函数不是以Go实现的。这样的声明定义了函数标识符。



}



递归

{

函数可以直接或间接的调用自身。	

非标准包 golang.org/x/net/html ，解析HTML

golang.org/x/... 目录下存储了一些由Go团队设计、维护，

对网络编程、国际化文件处理、移动平台、图像处理、加密解密、开发者工具提供支持的扩展包。



}



多返回值



错误

{

一个良好的程序永远不应该发生panic异常。

value, ok := cache.Lookup\(key\)

if !ok {

	// ...cache\[key\] does not exist…

}

内置的error是接口类型。

error类型可能是nil或者non-nil。nil意味着函数运行成功，non-nil表示失败。

resp, err := http.Get\(url\)

if err != nil{

	return nill, err

}

	

}



函数值

{

在Go中，函数被看作第一类值（first-class values） ：

	函数像其他值一样，拥有类型，可以被赋值给其他变量，传递给函数，从函数返回。

	对函数值（function value） 的调用类似函数调用。

	func square\(n int\) int { return n \* n }

	f := square

	fmt.Println\(f\(3\)\) // "9



函数类型的零值是nil,可以与nil比较。调用值为nil的函数值会引起panic错误

函数值之间是不可比较的，也不能用函数值作为map的key。

	

}



匿名函数

{

拥有函数名的函数只能在包级语法块中被声明，

通过函数字面量（function literal） ，我们可在任何表达式中表示一个函数值。

函数字面量的语法和函数声明相似，区别在于func关键字后没有函数名。

函数值字面量是一种表达式，它的值被称为匿名函数（anonymous function） 。	

// squares返回一个匿名函数。

// 该匿名函数每次被调用时都会返回下一个数的平方。

func squares\(\) func\(\) int {

	var x int

	return func\(\) int {

		x++

		return x \* x

	}

}

 func main\(\) {

	f := squares\(\)

	fmt.Println\(f\(\)\) // "1"

	fmt.Println\(f\(\)\) // "4"

	fmt.Println\(f\(\)\) // "9"

	fmt.Println\(f\(\)\) // "16"

}

函数squares返回另一个类型为 func\(\) int 的函数。

squares的例子证明，函数值不仅仅是一串代码，还记录了状态。

在squares中定义的匿名内部函数可以访问和更新squares中的局部变量,这意味着匿名函数和squares存在变量引用。这就是函数值属于引用类型和函数值不可比较的原因。

Go使用闭包（closures） 技术实现函数值，Go程序员也把函数值叫做闭包。



}



可变参数

{

在函数体中,vals被看作是类型为\[\] int的切片。

func sum\(vals...int\) int {

	total := 0

	for \_, val := range vals {

		total += val

	} 

	return total

}	

fmt.Println\(sum\(1, 2, 3, 4\)\) // "10"

原始参数已经是切片类型,只需最后一个参数后加上省略符。

values := \[\]int{1, 2, 3, 4}

fmt.Println\(sum\(values...\)\) // "10



...int 型参数的行为看起来很像切片类型，

但实际上，可变参数函数和以切片作为参数的函数是不同的。	

func f\(...int\) {}

func g\(\[\]int\) {}

fmt.Printf\("%T\n", f\) // "func\(...int\)"

fmt.Printf\("%T\n", g\) // "func\(\[\]int\)"



可变参数函数经常被用于格式化字符串。



}



deferred函数

{

你只需要在调用普通函数或方法前加上关键字defer，就完成了defer所需要的语法。

当defer语句被执行时，跟在defer后面的函数会被延迟执行。

直到包含该defer语句的函数执行完毕时，defer后的函数才会被执行，不论包含defer语句的函数是通过return正常结束，还是由于panic导致的异常结束。

你可以在一个函数中执行多条defer语句，它们的执行顺序与声明顺序相反。	



defer语句经常被用于处理成对的操作，如打开、关闭、连接、断开连接、加锁、释放锁。

调试复杂程序时，defer机制也常被用于记录何时进入和退出函数。

}



panic异常

{

Go的类型系统会在编译时捕获很多错误，

但有些错误只能在运行时检查，如数组访问越界、空指针引用等。这些运行时错误会引起painc异常

当panic异常发生时，程序会中断运行，并立即执行在该goroutine中被延迟的函数（defer 机制） 

switch s := suit\(drawCard\(\)\); s {

	case "Spades": // ...

	case "Hearts": // ...

	case "Diamonds": // ...

	case "Clubs": // ...

	default:

		panic\(fmt.Sprintf\("invalid suit %q", s\)\) // Joker?

}

我们应该使用Go提供的错误机制，而不是panic，尽量避免程序的崩溃。

	

}



recover捕获异常

{

如果在deferred函数中调用了内置函数recover，并且定义该defer语句的函数发生了panic异

常，recover会使程序从panic中恢复，并返回panic value。导致panic异常的函数不会继续运

行，但能正常返回。在未发生panic时调用recover，recover会返回nil。

	

}

	

}



5、方法

面向对象编程\(OOP\)-封装和组合

{

声明

{

	在函数声明时，在其名字之前放上一个变量，即是一个方法。

	这个附加的参数会将该函数附加到这种类型上，即相当于为这种类型定义了一个独占的方法。

	// same thing, but as a method of the Point type

	func \(p Point\) Distance\(q Point\) float64 {

		return math.Hypot\(q.X-p.X, q.Y-p.Y\)

	}

	附加的参数p，叫做方法的接收器\(receiver\)，（建议使用其类型的第一个字母）

	这种p.Distance的表达式叫做选择器，选择器也会被用来选择一个struct类型的字段。

	所有如果p的成员和方法重名会报错

	方法可以被声明到任意类型，只要不是一个指针或者一个interface。

	

}



基于指针对象的方法

{

func \(p \*Point\) ScaleBy\(factor float64\) {

	p.X \*= factor

	p.Y \*= factor

}

如果一个类型名本身是一个指针的话，是不允许其出现在接收器中的，比如下面这个例子：

type P \*int

func \(P\) f\(\) { /\* ... \*/ } // compile error: invalid receiver type



在现实的程序里，一般会约定如果Point这个类有一个指针作为接收器的方法，那么所有Point的方法都必须有一个指针接收器，即使是那些并不需要这个指针接收器的函数。





	

方法调用时go会隐式的吧指针和变量互换：

p := Point{1, 2}

\(&p\).ScaleBy\(2\)和p.ScaleBy\(2\)等价



pptr := &p

pptr.Distance\(q\)和\(\*pptr\).Distance\(q\)等价



Point{1, 2}.Distance\(q\) // Point 可行

pptr.ScaleBy\(2\) // \*Point

（但是临时变量的地址无法获取）

	Point{1, 2}.ScaleBy\(2\) // compile error: can't take address of Point literal

	

就像一些函数允许nil指针作为参数一样，方法理论上也可以用nil指针作为其接收器，

尤其当nil对于对象来说是合法的零值时，比如map或者slice。

	

}



通过嵌入结构体扩展类型

{

import "image/color"

type Point struct{ X, Y float64 }

type ColoredPoint struct {

	Point //Point类的方法也被引入了ColoredPoint，ColoredPoint也可以直接使用

	Color color.RGBA

}	

var cp ColoredPoint

cp.X = 1

fmt.Println\(cp.Point.X\) // "1"

cp.Point.Y = 2

fmt.Println\(cp.Y\) // "2"



当编译器解析一个选择器到方法时，比如p.ScaleBy，它会首先去找直接定义在这个类型里的ScaleBy方法，

然后找被ColoredPoint的内嵌字段们引入的方法，

然后去找Point和RGBA的内嵌字段引入的方法，然后一直递归向下找。

如果选择器有二义性的话编译器会报错，比如你在同一级里有两个同名的方法。



}



方法值和方法表达式

{

r.Launch是方法值：p = r.Launch,p\(a\)

r.Launch\(\)是方法表达式	:p=r.Launch\(\)获取的是返回结果

}

bit数组

封装	

{

一个对象的变量或者方法如果对调用方是不可见的话，一般就被定义为“封装”。（信息隐藏）

Go语言只有一种控制可见性的手段：

	大写首字母的标识符会从定义它们的包中被导出，小写字母的则不会。

	这种限制包内成员的方式同样适用于struct或者一个类型的方法。

要封装一个对象，我们必须将其定义为一个struct，类型名首字母大写，不需要被外部访问的属性首字母小写。

一个struct类型的字段对同一个包的所有代码都有可见性，无论你的代码是写在一个函数还是一个方法里。

因为对象内部变量只可以被同一个包内的函数修改，所以包的作者可以让这些函数确保对象内部的一些值的不变性。	

只用来访问或修改内部变量的函数被称为setter或者getter

}



}



6、接口

{

接口类型是对其它类型行为的抽象和概括,满足隐式实现，

接口类型具体描述了一系列方法的集合，一个实现了这些方法的具体类型是这个接口类型的实例。

一个类型如果拥有一个接口需要的所有方法，那么这个类型就实现了这个接口。



还记得在T类型的参数上调用一个T的方法是合法的，只要这个参数是一个变量；

编译器隐式的获取了它的地址。但这仅仅是一个语法糖：

T类型的值不拥有所有\*T指针的方法，那这样它就可能只实现更少的接口。



flag,value接口





接口值

{

由两个部分组成，一个具体的类型和那个类型的值。被称为接口的动态类型和动态值。

对于像Go语言这种静态类型的语言，类型是编译期的概念；因此一个类型不是一个值。

在我们的概念模型中，一些提供每个类型信息的值被称为类型描述符，比如类型的名称和方法。

在一个接口值中，类型部分代表与之相关类型的描述符



var w io.Writer  type nill,value nil

用w==nil或者w!=nil来判读接口值是否为空

将一个\*os.File类型的值赋给变量w:



w = os.Stdout 和io.Writer\(os.Stdout\)是等价的

这个接口值的动态类型被设为\*os.Stdout指针的类型描述符，它的动态值持有os.Stdout的拷贝；

这是一个代表处理标准输出的os.File类型变量的指针



w.Write\(\[\]byte\("hello"\)\) // "hello"

调用一个包含\*os.File类型指针的接口值的Write方法，使得\(\*os.File\).Write方法被调用



一个包含nil指针的接口不是nil接口



	

}



sort.interface接口

{

	

}



http.handler接口

{

package http

type Handler interface {

	ServeHTTP\(w ResponseWriter, r \*Request\)

} 

func ListenAndServe\(address string, h Handler\) error

istenAndServe函数需要一个例如“localhost:8000”的服务器地址，和一个所有请求都可以分派的Handler接口实例。

}



error接口

{

	

}



类型断言

{

	

}



类型分支	

{

	

}



}

50-288

-----------------------------

基础↑

-----------------------------

高级↓

-----------------------------



7、goroutines和channels

Go语言中的并发程序可以用两种手段来实现：goroutine和channel、多线程共享内存

{

goroutines

{

goroutine和channel，其支持“顺序通信进程”\(communicating sequential processes\)或被简称为CSP。

CSP是一种现代的并发编程模型，在这种编程模型中值会在不同的运行实例\(goroutine\)中传递，尽管大多数情况下仍然是被限制在单一实例中。



在Go语言中，每一个并发的执行单元叫作一个goroutine。

当一个程序启动时，其主函数即在一个单独的goroutine中运行，我们叫它main goroutine。新

的goroutine会用go语句来创建。go语句会使其语句中的函数在一个新创建的goroutine中运行。而go语句本身会迅速地完成。

	

}





channels

{

ch := make\(chan int\) //无缓冲 ch has type 'chan int'	

ch = make\(chan int, 3\)//带三个缓冲

ch &lt;- x // a send statement

x = &lt;-ch // a receive expression in an assignment statement

&lt;-ch // a receive statement; result is discarded

close\(ch\)



goroutine是Go语音程序的并发体的话，channels它们之间的通信机制

channel都有一个特殊的类型，也就是channels可发送数据的类型。



和map类似，channel也一个对应make创建的底层数据结构的引用。

两个相同类型的channel可以使用==运算符比较。

一个channel有发送和接受两个主要操作，都是通信行为。



Channel还支持close操作，用于关闭channel，

发送操作会panic异常，但是依然可以接受到之前已经成功发送的数据





无缓存Channels的发送操作将导致发送者goroutine阻塞，直到另一个goroutine在相同的Channels上执行接收操作；

当发送的值通过Channels成功传输之后，两个goroutine可以继续执行后面的语句。

反之，如果接收操作先发生，那么接收者goroutine也将阻塞，直到有另一个goroutine在相同的Channels上执行发送操作。

基于无缓存Channels的发送和接收操作将导致两个goroutine做一次同步操作。

因为这个原因，无缓存Channels有时候也被称为同步Channels。当通过一个无缓存Channels发送数据时，接收者收到数据发生在唤醒发送者goroutine之前（译注：happens before，这是Go语言

并发内存模型的一个关键术语！） 。





}

并发的循环

并发的web爬虫

基于select的多路复用

并发的字典遍历

并发的退出

聊天服务	

}



8、基于共享变量的并发

多goroutine之间的共享变量

{

竞争条件

sync.nutex互斥锁

	import "sync"

	var \(

		mu sync.Mutex // guards balance

		balance int

	\) 

	func Deposit\(amount int\) {

		mu.Lock\(\)

		balance = balance + amount

		mu.Unlock\(\)

	} 

	func Balance\(\) int {

		mu.Lock\(\)

		b := balance

		mu.Unlock\(\)

		return b

	}

sync.rwmutext读写锁

	var mu sync.RWMutex

	var balance int

	func Balance\(\) int {

		mu.RLock\(\) // readers lock

		defer mu.RUnlock\(\)

		return balance

	}

内存同步

sync.once初始化

竞争条件检测

并发的非阻塞缓存

goroutines和线程	

{

每一个OS线程都有一个固定大小的内存块\(一般会是2MB\)来做栈，存储当前正在被调用或挂起\(指在调用其它函数时\)的函数的内部变量。

，同时创建成百上千个gorutine是非常普遍的

一个goroutine会以一般2KB的栈开始其生命周期，，和操作系统线程一样，会保存其活跃或挂起的函数调用的本地变量，

栈的大小会根据需要动态地伸缩，栈的最大值有1GB，尽管一般情况下，大多goroutine都不需要这么大的栈。



OS线程会被操作系统内核调度。

Go的运行时包含了其自己的调度器，但是这个调度器只关注单独的Go程序中的goroutine

所以重新调度一个goroutine比调度一个线程代价要低得多。	

}





}



9、包和工具

{

导入路径

包声明

导入声明

包的匿名导入

包和命名

工具

}



10、测试



11、反射



12、底层编程



13、go原理分析



