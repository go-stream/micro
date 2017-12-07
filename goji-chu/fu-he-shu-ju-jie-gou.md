    //数组、slice、map和结构体字面值的写法都很相似。
    //crypto/sha256包的Sum256函数对一个任意的字节slice类型的数据生成一个对应的消息摘要
    {
    数组-聚合类型
    {
    var a[3]int
    q := [...]int{1, 2, 3} 根据初始化值的个数来计算
    r := [...]int{99: -1}  100个元素的数组r，最后一个初始化为-1，其它元素都是用0初始化。

    由同构的元素组成——每个数组元素都是完全相同的类
    数组是一个由固定长度的特定类型元素组成的序列
    内置的len函数将返回数组中元素的个数
    数组的长度是数组类型的一个组成部分,数组的长度需要在编译阶段确定,必须是常量表达式，
    [3]int和[4]int是两种不同的数组类型

    Go语言中很少直接使用数组
    和数组对应的类型是Slice（切片） ,它是可以增长和收缩动态序列

    }
    //slice和map则是动态的数据结构
    slice
    {
    一个slice类型一般写作[]T
    slice由三个部分构成：指针、长度和容量(长度不能大于容量)  内置的len和cap函数
    s[i:j] 引用s的从第i个元素开始到第j-1个元素的子序列
    i省略使用0代替，j省略用len(s)代替。

    字符串的切片操作和[]byte字节类型切片的切片操作是类似的

    slice之间不能比较	（bytes.Equal函数来判断两个字节型slice是否相等（[]byte））
    一个nil值的slice的长度和容量都是0，但是也有非nil值的slice的长度和容量也是0的
    len(s) == 0来判断slice是否为空

    make函数（make创建了一个匿名的数组变量，然后返回一个slice）
    make([]T, len)
    make([]T, len, cap)

    runes = append(runes, r)内置的append函数用于向slice追加元素，不知道是否内存重新分配

    一个slice可以用来模拟一个stack
    }

    map
    {
    map之间也不能进行相等比较	

    哈希表是一种巧妙并且实用的数据结构。它是一个无序的key/value对的集合
    在Go语言中，一个map就是一个哈希表的引用，map类型可以写为map[K]V
    map中的元素并不是一个变量，因此我们不能对map的元素进行取址操作

    ages := make(map[string]int) 

    ages := map[string]int{}
    ages := map[string]int{"alice": 31,"charlie": 34,}
    delete(ages, "alice") 

    遍历（随机的）
    for name, age := range ages {
    	fmt.Printf("%s\t%d\n", name, age)
    }

    map的下标语法将产生两个值；第二个是一个布尔值，用于报告元素是否存在
    if age, ok := ages["bob"]; !ok { /* ... */ }


    显式地对key进行排序，用slice
    import "sort"
    var names []string
    for name := range ages {
    	names = append(names, name)
    } 
    sort.Strings(names)
    for _, name := range names {
    	fmt.Printf("%s\t%d\n", name, ages[name])
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
    	Year int `json:"released"`
    	Color bool `json:"color,omitempty"`
    	Actors []string
    }
    一个构体成员Tag是和在编译阶段关联到该成员的元信息字符串
    结构体的成员Tag可以是任意的字符串面值，但是通常是一系列用空格分隔的key:"value"键值对序列；
    因为值中含义双引号字符，因此成员Tag一般用原生字符串面值的形式书写。
    json开头键名对应的值用于控制encoding/json包的编码和解码的行为，
    并且encoding/...下面其它的包也遵循这个约定。成员Tag中json对应值的第一部分用于指定JSON对象的名字，
    如TotalCount成员对应到JSON中的total_count对象。Color成员的Tag还带了一个额外的omitempty选项，表示结构体成员为空或零值时不生成JSON对象（这里false为零值）



    //Marshal函数返还一个编码后的字节slice，包含很长的字符串，并且没有空白缩进；
    data, err := json.Marshal(movies)
    if err != nil {
    	log.Fatalf("JSON marshaling failed: %s", err)
    } 
    fmt.Printf("%s\n", data)
    //json.MarshalIndent函数将产生整齐缩进的输出

    解密：json.Unmarshal函数完成
    }

    文本和html模板
    {
    text/template和html/template等模板包提供的，它们提供了一个将变量值填充到一个文本或HTML格式的模板的机制。

    }

    }




