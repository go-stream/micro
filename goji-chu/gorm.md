关联（包含一个，包含多个，属于，多对多，多种包含）

Callbacks（创建/保存/更新/删除/查找之前/之后）

预加载（急加载）

事务

复合主键

SQL Builder

自动迁移

日志

可扩展，编写基于GORM回调的插件



go get -u github.com/jinzhu/gorm



package main



import \(

    "github.com/jinzhu/gorm"

		\_ "github.com/jinzhu/gorm/dialects/mysql"

\)



type Product struct {

  gorm.Model

  Code string

  Price uint

}



func main\(\) {

  db, err := gorm.Open\("mysql", "user:password@/dbname?charset=utf8&parseTime=True&loc=Local"\)

  if err != nil {

    panic\("连接数据库失败"\)

  }

  defer db.Close\(\)



  // 自动迁移模式

  db.AutoMigrate\(&Product{}\)



  // 创建

  db.Create\(&Product{Code: "L1212", Price: 1000}\)

	//db.HasTable\(&User{}\)

  // 读取

  var product Product

  db.First\(&product, 1\) // 查询id为1的product

  db.First\(&product, "code = ?", "L1212"\) // 查询code为l1212的product



  // 更新 - 更新product的price为2000

  db.Model\(&product\).Update\("Price", 2000\)



  // 删除 - 删除product

  db.Delete\(&product\)

}





1、新增

user := User{Name: "Jinzhu", Age: 18, Birthday: time.Now\(\)}

if db.NewRecord\(user\){

	db.Create\(&user\)

}else{

	//已存在

}



2、查询

{

	// 获取第一条记录，按主键排序

db.First\(&user\)// SELECT \* FROM users ORDER BY id LIMIT 1;

// 获取最后一条记录，按主键排序

db.Last\(&user\)// SELECT \* FROM users ORDER BY id DESC LIMIT 1;

// 获取所有记录

db.Find\(&users\)// SELECT \* FROM users;

// 使用主键获取记录

db.First\(&user, 10\)// SELECT \* FROM users WHERE id = 10;

// 获取所有匹配记录

db.Where\("name = ?", "jinzhu"\).Find\(&users\)// SELECT \* FROM users WHERE name = 'jinzhu';

db.Where\("name &lt;&gt; ?", "jinzhu"\).Find\(&users\)

// IN

db.Where\("name in \(?\)", \[\]string{"jinzhu", "jinzhu 2"}\).Find\(&users\)

// LIKE

db.Where\("name LIKE ?", "%jin%"\).Find\(&users\)

// AND

db.Where\("name = ? AND age &gt;= ?", "jinzhu", "22"\).Find\(&users\)

// Time

db.Where\("updated\_at &gt; ?", lastWeek\).Find\(&users\)

db.Where\("created\_at BETWEEN ? AND ?", lastWeek, today\).Find\(&users\)



// Struct

db.Where\(&User{Name: "jinzhu", Age: 20}\).First\(&user\)

//// SELECT \* FROM users WHERE name = "jinzhu" AND age = 20 LIMIT 1;

// Map

db.Where\(map\[string\]interface{}{"name": "jinzhu", "age": 20}\).Find\(&users\)

//// SELECT \* FROM users WHERE name = "jinzhu" AND age = 20;

// 主键的Slice

db.Where\(\[\]int64{20, 21, 22}\).Find\(&users\)

//// SELECT \* FROM users WHERE id IN \(20, 21, 22\);

db.Not\("name", "jinzhu"\).First\(&user\)

//// SELECT \* FROM users WHERE name &lt;&gt; "jinzhu" LIMIT 1;



// Not In

db.Not\("name", \[\]string{"jinzhu", "jinzhu 2"}\).Find\(&users\)

//// SELECT \* FROM users WHERE name NOT IN \("jinzhu", "jinzhu 2"\);

// Not In slice of primary keys

db.Not\(\[\]int64{1,2,3}\).First\(&user\)

//// SELECT \* FROM users WHERE id NOT IN \(1,2,3\);

db.Not\(\[\]int64{}\).First\(&user\)

//// SELECT \* FROM users;



// Plain SQL

db.Not\("name = ?", "jinzhu"\).First\(&user\)

//// SELECT \* FROM users WHERE NOT\(name = "jinzhu"\);

// Struct

db.Not\(User{Name: "jinzhu"}\).First\(&user\)

//// SELECT \* FROM users WHERE name &lt;&gt; "jinzhu";



注意：使用主键查询时，应仔细检查所传递的值是否为有效主键，以避免SQL注入

// 按主键获取

db.First\(&user, 23\)

//// SELECT \* FROM users WHERE id = 23 LIMIT 1;

// 简单SQL

db.Find\(&user, "name = ?", "jinzhu"\)

//// SELECT \* FROM users WHERE name = "jinzhu";

db.Find\(&users, "name &lt;&gt; ? AND age &gt; ?", "jinzhu", 20\)

//// SELECT \* FROM users WHERE name &lt;&gt; "jinzhu" AND age &gt; 20;

// Struct

db.Find\(&users, User{Age: 20}\)

//// SELECT \* FROM users WHERE age = 20;

// Map

db.Find\(&users, map\[string\]interface{}{"age": 20}\)

//// SELECT \* FROM users WHERE age = 20;

db.Where\("role = ?", "admin"\).Or\("role = ?", "super\_admin"\).Find\(&users\)

//// SELECT \* FROM users WHERE role = 'admin' OR role = 'super\_admin';

// Struct

db.Where\("name = 'jinzhu'"\).Or\(User{Name: "jinzhu 2"}\).Find\(&users\)

//// SELECT \* FROM users WHERE name = 'jinzhu' OR name = 'jinzhu 2';

// Map

db.Where\("name = 'jinzhu'"\).Or\(map\[string\]interface{}{"name": "jinzhu 2"}\).Find\(&users\)

db.Where\("name &lt;&gt; ?","jinzhu"\).Where\("age &gt;= ? and role &lt;&gt; ?",20,"admin"\).Find\(&users\)

//// SELECT \* FROM users WHERE name &lt;&gt; 'jinzhu' AND age &gt;= 20 AND role &lt;&gt; 'admin';



db.Where\("role = ?", "admin"\).Or\("role = ?", "super\_admin"\).Not\("name = ?", "jinzhu"\).Find\(&users\)



db.Select\("name, age"\).Find\(&users\)

//// SELECT name, age FROM users;

db.Select\(\[\]string{"name", "age"}\).Find\(&users\)

//// SELECT name, age FROM users;



db.Table\("users"\).Select\("COALESCE\(age,?\)", 42\).Rows\(\)



db.Order\("age desc, name"\).Find\(&users\)

//// SELECT \* FROM users ORDER BY age desc, name;

// Multiple orders

db.Order\("age desc"\).Order\("name"\).Find\(&users\)

//// SELECT \* FROM users ORDER BY age desc, name;

// ReOrder

db.Order\("age desc"\).Find\(&users1\).Order\("age", true\).Find\(&users2\)

//// SELECT \* FROM users ORDER BY age desc; \(users1\)

//// SELECT \* FROM users ORDER BY age; \(users2\)



db.Limit\(3\).Find\(&users\)

//// SELECT \* FROM users LIMIT 3;

// Cancel limit condition with -1

db.Limit\(10\).Find\(&users1\).Limit\(-1\).Find\(&users2\)

//// SELECT \* FROM users LIMIT 10; \(users1\)

//// SELECT \* FROM users; \(users2\)



db.Offset\(3\).Find\(&users\)

//// SELECT \* FROM users OFFSET 3;

// Cancel offset condition with -1

db.Offset\(10\).Find\(&users1\).Offset\(-1\).Find\(&users2\)

//// SELECT \* FROM users OFFSET 10; \(users1\)

//// SELECT \* FROM users; \(users2\)



db.Where\("name = ?", "jinzhu"\).Or\("name = ?", "jinzhu 2"\).Find\(&users\).Count\(&count\)

//// SELECT \* from USERS WHERE name = 'jinzhu' OR name = 'jinzhu 2'; \(users\)

//// SELECT count\(\*\) FROM users WHERE name = 'jinzhu' OR name = 'jinzhu 2'; \(count\)

db.Model\(&User{}\).Where\("name = ?", "jinzhu"\).Count\(&count\)

//// SELECT count\(\*\) FROM users WHERE name = 'jinzhu'; \(count\)

db.Table\("deleted\_users"\).Count\(&count\)

//// SELECT count\(\*\) FROM deleted\_users;	

}



3、更新

{

Save将包括执行更新SQL时的所有字段，即使它没有更改

db.First\(&user\)

user.Name = "jinzhu 2"

user.Age = 100

db.Save\(&user\)



// 更新单个属性（如果更改）

db.Model\(&user\).Update\("name", "hello"\)

//// UPDATE users SET name='hello', updated\_at='2013-11-17 21:34:10' WHERE id=111;

// 使用组合条件更新单个属性

db.Model\(&user\).Where\("active = ?", true\).Update\("name", "hello"\)

//// UPDATE users SET name='hello', updated\_at='2013-11-17 21:34:10' WHERE id=111 AND active=true;

// 使用\`map\`更新多个属性，只会更新这些更改的字段

db.Model\(&user\).Updates\(map\[string\]interface{}{"name": "hello", "age": 18, "actived": false}\)

//// UPDATE users SET name='hello', age=18, actived=false, updated\_at='2013-11-17 21:34:10' WHERE id=111;

// 使用\`struct\`更新多个属性，只会更新这些更改的和非空白字段

db.Model\(&user\).Updates\(User{Name: "hello", Age: 18}\)

//// UPDATE users SET name='hello', age=18, updated\_at = '2013-11-17 21:34:10' WHERE id = 111;

// 警告:当使用struct更新时，FORM将仅更新具有非空值的字段

// 对于下面的更新，什么都不会更新为""，0，false是其类型的空白值

db.Model\(&user\).Updates\(User{Name: "", Age: 0, Actived: false}\)







如果您只想在更新时更新或忽略某些字段，可以使用Select, Omit



db.Model\(&user\).Select\("name"\).Updates\(map\[string\]interface{}{"name": "hello", "age": 18, "actived": false}\)

//// UPDATE users SET name='hello', updated\_at='2013-11-17 21:34:10' WHERE id=111;

db.Model\(&user\).Omit\("name"\).Updates\(map\[string\]interface{}{"name": "hello", "age": 18, "actived": false}\)

//// UPDATE users SET age=18, actived=false, updated\_at='2013-11-17 21:34:10' WHERE id=111;









// 更新单个属性，类似于\`Update\`

db.Model\(&user\).UpdateColumn\("name", "hello"\)

//// UPDATE users SET name='hello' WHERE id = 111;

// 更新多个属性，与“更新”类似

db.Model\(&user\).UpdateColumns\(User{Name: "hello", Age: 18}\)

//// UPDATE users SET name='hello', age=18 WHERE id = 111;





Callbacks在批量更新时不会运行

db.Table\("users"\).Where\("id IN \(?\)", \[\]int{10, 11}\).Updates\(map\[string\]interface{}{"name": "hello", "age": 18}\)

//// UPDATE users SET name='hello', age=18 WHERE id IN \(10, 11\);

// 使用struct更新仅适用于非零值，或使用map\[string\]interface{}

db.Model\(User{}\).Updates\(User{Name: "hello", Age: 18}\)

//// UPDATE users SET name='hello', age=18;

// 使用\`RowsAffected\`获取更新记录计数

db.Model\(User{}\).Updates\(User{Name: "hello", Age: 18}\).RowsAffected		

}



4、删除

{

GORM将使用主键删除记录，如果主要字段为空，GORM将删除模型的所有记录

// 删除存在的记录

db.Delete\(&email\)

//// DELETE from emails where id=10;

删除所有匹配记录



db.Where\("email LIKE ?", "%jinzhu%"\).Delete\(Email{}\)

db.Delete\(Email{}, "email LIKE ?", "%jinzhu%"\)

//// DELETE from emails where email LIKE "%jinhu%";





如果模型有DeletedAt字段，它将自动获得软删除功能！ 那么在调用Delete时不会从数据库中永久删除，而是只将字段DeletedAt的值设置为当前时间。

db.Delete\(&user\)

//// UPDATE users SET deleted\_at="2013-10-29 10:23" WHERE id = 111;

// 批量删除

db.Where\("age = ?", 20\).Delete\(&User{}\)

//// UPDATE users SET deleted\_at="2013-10-29 10:23" WHERE age = 20;

// 软删除的记录将在查询时被忽略

db.Where\("age = 20"\).Find\(&user\)

//// SELECT \* FROM users WHERE age = 20 AND deleted\_at IS NULL;



// 使用Unscoped查找软删除的记录

db.Unscoped\(\).Where\("age = 20"\).Find\(&users\)

//// SELECT \* FROM users WHERE age = 20;

// 使用Unscoped永久删除记录

db.Unscoped\(\).Delete\(&order\)

//// DELETE FROM orders WHERE id=10;	

}





基本模型定义gorm.Model，包括字段ID，CreatedAt，UpdatedAt，DeletedAt，你可以将它嵌入你的模型，或者只写你想要的字段



db.SingularTable\(true\) //全局限制不是复数



// 使用tag\`primary\_key\`用来设置主键

type Animal struct {

	// ID   uint  // 字段\`ID\`为默认主键

  AnimalId int64 \`gorm:"primary\_key"\` // 设置AnimalId为主键

	AnimalId2    int64     \`gorm:"column:beast\_id"\`         // 设置列名为\`beast\_id\`

  Name     string

  Age      int64

	CreatedAt time.Time // 列名为 \`created\_at\`

	Name string \`gorm:"index:idx\_name\_code"\` // 创建索引并命名，如果找到其他相同名称的索引则创建组合索引

	Code string \`gorm:"index:idx\_name\_code"\` // \`unique\_index\` also works

	Name string \`gorm:"default:'galeone'"\`  //默认值

}









func CreateAnimals\(db \*gorm.DB\) err {

  tx := db.Begin\(\)

  // 注意，一旦你在一个事务中，使用tx作为数据库句柄



  if err := tx.Create\(&Animal{Name: "Giraffe"}\).Error; err != nil {

     tx.Rollback\(\)

     return err

  }



  if err := tx.Create\(&Animal{Name: "Lion"}\).Error; err != nil {

     tx.Rollback\(\)

     return err

  }



  tx.Commit\(\)

  return nil

}









