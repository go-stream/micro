```
面向对象编程(OOP)-封装和组合
{
声明
{
	在函数声明时，在其名字之前放上一个变量，即是一个方法。
	这个附加的参数会将该函数附加到这种类型上，即相当于为这种类型定义了一个独占的方法。
	// same thing, but as a method of the Point type
	func (p Point) Distance(q Point) float64 {
		return math.Hypot(q.X-p.X, q.Y-p.Y)
	}
	附加的参数p，叫做方法的接收器(receiver)，（建议使用其类型的第一个字母）
	这种p.Distance的表达式叫做选择器，选择器也会被用来选择一个struct类型的字段。
	所有如果p的成员和方法重名会报错
	方法可以被声明到任意类型，只要不是一个指针或者一个interface。
	
}

基于指针对象的方法
{
func (p *Point) ScaleBy(factor float64) {
	p.X *= factor
	p.Y *= factor
}
如果一个类型名本身是一个指针的话，是不允许其出现在接收器中的，比如下面这个例子：
type P *int
func (P) f() { /* ... */ } // compile error: invalid receiver type

在现实的程序里，一般会约定如果Point这个类有一个指针作为接收器的方法，那么所有Point的方法都必须有一个指针接收器，即使是那些并不需要这个指针接收器的函数。


	
方法调用时go会隐式的吧指针和变量互换：
p := Point{1, 2}
(&p).ScaleBy(2)和p.ScaleBy(2)等价

pptr := &p
pptr.Distance(q)和(*pptr).Distance(q)等价

Point{1, 2}.Distance(q) // Point 可行
pptr.ScaleBy(2) // *Point
（但是临时变量的地址无法获取）
	Point{1, 2}.ScaleBy(2) // compile error: can't take address of Point literal
	
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
fmt.Println(cp.Point.X) // "1"
cp.Point.Y = 2
fmt.Println(cp.Y) // "2"

当编译器解析一个选择器到方法时，比如p.ScaleBy，它会首先去找直接定义在这个类型里的ScaleBy方法，
然后找被ColoredPoint的内嵌字段们引入的方法，
然后去找Point和RGBA的内嵌字段引入的方法，然后一直递归向下找。
如果选择器有二义性的话编译器会报错，比如你在同一级里有两个同名的方法。

}

方法值和方法表达式
{
r.Launch是方法值：p = r.Launch,p(a)
r.Launch()是方法表达式	:p=r.Launch()获取的是返回结果
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

```



