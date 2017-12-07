```
{
函数声明
{
func name(parameter-list) (result-list) {
	body	
}
func add(x int, y int) int {return x + y}
func sub(x, y int) (z int) { z = x - y; return}
func first(x int, _ int) int { return x }
func zero(int, int) int { return 0 }
fmt.Printf("%T\n", add) // "func(int, int) int"
fmt.Printf("%T\n", sub) // "func(int, int) int"
fmt.Printf("%T\n", first) // "func(int, int) int"
fmt.Printf("%T\n", zero) // "func(int, int) int"

函数的类型被称为函数的标识符。
没有默认参数值，所有形参和返回值的变量名对于函数调用者而言没有意义
实参通过值的方式传递，因此函数的形参是实参的拷贝。
实参包括引用类型，如指针，slice(切片)、map、function、channel等类型，实参可能会由于函数的间接引用被修改。

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
value, ok := cache.Lookup(key)
if !ok {
	// ...cache[key] does not exist…
}
内置的error是接口类型。
error类型可能是nil或者non-nil。nil意味着函数运行成功，non-nil表示失败。
resp, err := http.Get(url)
if err != nil{
	return nill, err
}
	
}

函数值
{
在Go中，函数被看作第一类值（first-class values） ：
	函数像其他值一样，拥有类型，可以被赋值给其他变量，传递给函数，从函数返回。
	对函数值（function value） 的调用类似函数调用。
	func square(n int) int { return n * n }
	f := square
	fmt.Println(f(3)) // "9

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
func squares() func() int {
	var x int
	return func() int {
		x++
		return x * x
	}
}
 func main() {
	f := squares()
	fmt.Println(f()) // "1"
	fmt.Println(f()) // "4"
	fmt.Println(f()) // "9"
	fmt.Println(f()) // "16"
}
函数squares返回另一个类型为 func() int 的函数。
squares的例子证明，函数值不仅仅是一串代码，还记录了状态。
在squares中定义的匿名内部函数可以访问和更新squares中的局部变量,这意味着匿名函数和squares存在变量引用。这就是函数值属于引用类型和函数值不可比较的原因。
Go使用闭包（closures） 技术实现函数值，Go程序员也把函数值叫做闭包。

}

可变参数
{
在函数体中,vals被看作是类型为[] int的切片。
func sum(vals...int) int {
	total := 0
	for _, val := range vals {
		total += val
	} 
	return total
}	
fmt.Println(sum(1, 2, 3, 4)) // "10"
原始参数已经是切片类型,只需最后一个参数后加上省略符。
values := []int{1, 2, 3, 4}
fmt.Println(sum(values...)) // "10

...int 型参数的行为看起来很像切片类型，
但实际上，可变参数函数和以切片作为参数的函数是不同的。	
func f(...int) {}
func g([]int) {}
fmt.Printf("%T\n", f) // "func(...int)"
fmt.Printf("%T\n", g) // "func([]int)"

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
switch s := suit(drawCard()); s {
	case "Spades": // ...
	case "Hearts": // ...
	case "Diamonds": // ...
	case "Clubs": // ...
	default:
		panic(fmt.Sprintf("invalid suit %q", s)) // Joker?
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

```



