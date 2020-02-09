###  					golang中struct成员变量的标签（Tag）说明和获取方式 				

 				 					 					老汉-憨憨 					 				 					 · 2018-01-16 18:33:05 · 2016 次点击 · 					预计阅读时间 2 分钟 · 					大约2小时之前 开始浏览   					 				

这是一个创建于 2018-01-16 18:33:05 的文章，其中的信息可能已经有所发展或是发生改变。

在处理json格式字符串的时候，经常会看到声明struct结构的时候，属性的右侧还有小米点括起来的内容。形如：

| 1 2 3 4 | `type User struct {` `  ``UserId  int  `json:``"user_id"` `bson:``"user_id"```` `  ``UserName string `json:``"user_name"` `bson:``"user_name"```` `}` |
| ------- | ------------------------------------------------------------ |
|         |                                                              |

这个小米点里的内容是用来干什么的呢？



## struct成员变量标签（Tag）说明

要比较详细的了解这个，要先了解一下golang的基础，在golang中，命名都是推荐都是用驼峰方式，并且在首字母大小写有特殊的语法含义：包外无法引用。但是由经常需要和其它的系统进行数据交互，例如转成json格式，存储到mongodb啊等等。这个时候如果用属性名来作为键值可能不一定会符合项目要求。

所以呢就多了小米点的内容，在golang中叫标签（Tag），在转换成其它数据格式的时候，会使用其中特定的字段作为键值。例如上例在转成json格式：

| 1 2 3 4 | `u := &User{UserId: ``1``, UserName: ``"tony"``}` `j, _ := json.Marshal(u)` `fmt.Println(string(j))` `// 输出内容：{"user_id":1,"user_name":"tony"}` |
| ------- | ------------------------------------------------------------ |
|         |                                                              |

如果在属性中不增加标签说明，则输出：

```
{"UserId":1,"UserName":"tony"}
```

可以看到直接用struct的属性名做键值。

其中还有一个bson的声明，这个是用在将数据存储到mongodb使用的。



## struct成员变量标签（Tag）获取

那么当我们需要自己封装一些操作，需要用到Tag中的内容时，咋样去获取呢？这边可以使用反射包（reflect）中的方法来获取：

| 1 2 3 4 | `t := reflect.TypeOf(u)` `field := t.Elem().Field(``0``)` `fmt.Println(field.Tag.Get(``"json"``))` `fmt.Println(field.Tag.Get(``"bson"``))` |
| ------- | ------------------------------------------------------------ |
|         |                                                              |

```go
package main
 
import (
    "encoding/json"
    "fmt"
    "reflect"
)
 
func main() {
    type User struct {
        UserId   int    `json:"user_id" bson:"user_id"`
        UserName string `json:"user_name" bson:"user_name"`
    }
    // 输出json格式
    u := &User{UserId: 1, UserName: "tony"}
    j, _ := json.Marshal(u)
    fmt.Println(string(j))
    // 输出内容：{"user_id":1,"user_name":"tony"}
 
    // 获取tag中的内容
    t := reflect.TypeOf(u)
    field := t.Elem().Field(0)
    fmt.Println(field.Tag.Get("json"))
    // 输出：user_id
    fmt.Println(field.Tag.Get("bson"))
    // 输出：user_id
}
```