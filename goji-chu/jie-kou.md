```
{
接口类型是对其它类型行为的抽象和概括,满足隐式实现，
接口类型具体描述了一系列方法的集合，一个实现了这些方法的具体类型是这个接口类型的实例。
一个类型如果拥有一个接口需要的所有方法，那么这个类型就实现了这个接口。

还记得在T类型的参数上调用一个T的方法是合法的，只要这个参数是一个变量；
编译器隐式的获取了它的地址。但这仅仅是一个语法糖：
T类型的值不拥有所有*T指针的方法，那这样它就可能只实现更少的接口。

flag,value接口


接口值
{
由两个部分组成，一个具体的类型和那个类型的值。被称为接口的动态类型和动态值。
对于像Go语言这种静态类型的语言，类型是编译期的概念；因此一个类型不是一个值。
在我们的概念模型中，一些提供每个类型信息的值被称为类型描述符，比如类型的名称和方法。
在一个接口值中，类型部分代表与之相关类型的描述符

var w io.Writer  type nill,value nil
用w==nil或者w!=nil来判读接口值是否为空
将一个*os.File类型的值赋给变量w:

w = os.Stdout 和io.Writer(os.Stdout)是等价的
这个接口值的动态类型被设为*os.Stdout指针的类型描述符，它的动态值持有os.Stdout的拷贝；
这是一个代表处理标准输出的os.File类型变量的指针

w.Write([]byte("hello")) // "hello"
调用一个包含*os.File类型指针的接口值的Write方法，使得(*os.File).Write方法被调用

一个包含nil指针的接口不是nil接口

	
}

sort.interface接口
{
	
}

http.handler接口
{
package http
type Handler interface {
	ServeHTTP(w ResponseWriter, r *Request)
} 
func ListenAndServe(address string, h Handler) error
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

```



