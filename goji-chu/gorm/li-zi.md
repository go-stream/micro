    package main

    import (
    	"github.com/jinzhu/gorm"
    	_ "github.com/jinzhu/gorm/dialects/mysql"
    	"github.com/liudng/godump"
    	"time"
    )

    type User struct {
    	Id        uint
    	Name      string
    	CreatedAt time.Time  `gorm:"column:create_time"`
    	UpdatedAt time.Time  `gorm:"column:update_time"`
    	DeletedAt *time.Time `gorm:"column:delete_time"`
    }

    func main() {
    	db, err := gorm.Open("mysql", "wz:Password123!@tcp(192.168.74.50:3306)/wzcms?charset=utf8&parseTime=True&loc=Local")
    	if err != nil {
    		panic(err)
    	}
    	defer db.Close()
    	db.SingularTable(true) //全局限制不是复数

    	//创建
    	/*	user := User{Name: "1"}
    		db.Create(&user)
    		//判断是否创建成功，成功返回false
    		if db.NewRecord(user) {
    			//创建失败
    			return
    		}
    		godump.Dump(1)*/
    	//软删除
    	// num := db.Where("id = ?", 1).Delete(&User{}).RowsAffected
    	// godump.Dump(num)
    	//查询所有被删除的数据
    	// var users []User
    	// db.Unscoped().Where("delete_time > 0").Find(&users)
    	//永久删除被删除的数据
    	// db.First(&user, 1) // 查询id为1的product
    	// user := User{Id: 1}
    	// db.Unscoped().Delete(&user)

    	/*	//修改
    		var user User
    		db.First(&user, 2)
    		// 使用`struct`更新多个属性，只会更新这些更改的和非空白字段,如果没有指定会更新所有的记录
    		num := db.Model(&user).Updates(User{Name: "ww"}).RowsAffected
    		// db.Model(&user).Updates(map[string]interface{}{"name": "222"}).RowsAffected
    		godump.Dump(num)*/
    	// db.First(&product, "code = ?", "L1212") // 查询code为l1212的product

    	//查找
    	var users []User
    	db.Find(&users)
    	godump.Dump(len(users))
    	for k, v := range users {
    		godump.Dump(k)
    		godump.Dump(v)
    	}

    }




