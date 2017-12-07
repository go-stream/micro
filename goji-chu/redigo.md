



```
https://www.cnblogs.com/steven520213/p/6726706.html
开启远程访问：在redis的配置文件redis.conf中，找到bind localhost注释掉。protected-mode no 

go get github.com/garyburd/redigo/redis

package main

import (
	"fmt"

	"github.com/garyburd/redigo/redis"
)

// This example implements ZPOP as described at
// http://redis.io/topics/transactions using WATCH/MULTI/EXEC and scripting.
func main() {
	c, err := redis.Dial("tcp", "192.168.74.50:6379")
	if err != nil {
		fmt.Println(err)
		return
	}
	defer c.Close()

	v, err := c.Do("SET", "name", "red")
	if err != nil {
		fmt.Println(err)
		return
	}
	fmt.Println(v)
	v, err = redis.String(c.Do("GET", "a"))
	if err != nil {
		fmt.Println(err)
		return
	}
	fmt.Println(v)

}

```





















