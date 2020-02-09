## golang 中 json 转 struct

　　<1. 使用 json.Unmarshal 时，结构体的每一项必须是导出项(import field)。也就是说结构体的 key 对应的首字母必须为大写。请看下面的例子：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
package commontest

import (
    "testing"
    "encoding/json"
)

type Person struct {
    name string
    age int
}

func TestStruct2Json(t *testing.T) {
    jsonStr := `
    {
        "name":"liangyongxing",
        "age":12
    }
    `
    var person Person
    json.Unmarshal([]byte(jsonStr), &person)
    t.Log(person)
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

输出的结果如下：

```
`{ 0}`
```

 　从结果可以看出，json 数据并没有写入 Person 结构体中。结构体 key 首字母大写的话就可以，修改后：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
package commontest

import (
    "testing"
    "encoding/json"
)

type Person struct {
    Name string
    Age int
}

func TestStruct2Json(t *testing.T) {
    jsonStr := `
    {
        "name":"liangyongxing",
        "age":12
    }
    `
    var person Person
    json.Unmarshal([]byte(jsonStr), &person)
    t.Log(person)
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

打印结果如下：

```
`{liangyongxing 12}`
```

 　从以上结果我们可以发现一个很重要的信息，json 里面的 key 和 struct 里面的 key  一个是小写一个是大写，即两者大小写并没有对上。从这里我们就可以得出一个结论，要想能够附上值需要结构体中的变量名首字母大写，而在转换的 json  串中大小写都可以，即在 json 传中字段名称大小写不敏感。那么经过验证发现，在 json 中如果写成如下方式：

```
jsonStr := `
    {
        "NaMe":"liangyongxing",
        "agE":12
    }
    `
```

　　最终结果仍然是有值的，那么就验证了我们上面的结论，json 串中对字段名大小写不敏感(不一定是首字母，这点需要注意)

　　<2. 在结构体中是可以引入 tag  标签的，这样在匹配的时候 json 串对应的字段名需要与 tag 标签中定义的字段名匹配，当然在 tag  中定义的名称就不需要首字母大写了，且对应的 json 串中字段名称仍然大小写不敏感，和上面的结论一致。(**注意：**此时结构体中对应的字段名可以不用和匹配的一致，但是也必须首字母大写，只有大写的才是可对外提供访问的)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
package commontest

import (
    "testing"
    "encoding/json"
)

//这里对应的 N 和 A 不能为小写，首字母必须为大写，这样才可对外提供访问，具体 json 匹配是通过后面的 tag 标签进行匹配的，与 N 和 A 没有关系
//tag 标签中 json 后面跟着的是字段名称，都是字符串类型，要求必须加上双引号，否则 golang 是无法识别它的类型
type Person struct {
    N string     `json:"name"`
    A int        `json:"age"`
}

func TestStruct2Json(t *testing.T) {
    jsonStr := `
    {
        "name":"liangyongxing",
        "age":12
    }
    `
    var person Person
    json.Unmarshal([]byte(jsonStr), &person)
    t.Log(person)
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

这样输出的结果如下：

```
`{liangyongxing 12}`
```

 　当然，你也可以再做一个实验，验证 tag 标签中对应的字段名称大小写不敏感，这里就不做冗余介绍了。

### 2. golang 中 struct 转 json 串

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 package commontest
 2 
 3 import (
 4     "testing"
 5     "encoding/json"
 6 )
 7 
 8 func TestStruct2Json(t *testing.T) {
 9     p := Person{
10         Name: "liangyongxing",
11         Age: 29,
12     }
13 
14     t.Logf("Person 结构体打印结果:%v", p)
15 
16     //Person 结构体转换为对应的 Json
17     jsonBytes, err := json.Marshal(p)
18     if err != nil {
19         t.Fatal(err)
20     }
21     t.Logf("转换为 json 串打印结果:%s", string(jsonBytes))
22 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

打印结果如下所示：

```
`/usr/local/``go``/bin/``go` `test -v commontest -run ^TestStruct2Json$``  ``struct2json_test.``go``:14: Person 结构体打印结果:{liangyongxing 29}``  ``struct2json_test.``go``:21: 转换为 json 串打印结果:{``"name"``:``"liangyongxing"``,``"age"``:29}``ok   commontest 0.006s`
```

###  3. golang 中 json 转 map

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
package commontest

import (
    "testing"
    "encoding/json"
)

func TestJson2Map(t *testing.T) {
    jsonStr := `
    {
        "name":"liangyongxing",
        "age":12
    }
    `
    var mapResult map[string]interface{}
    //使用 json.Unmarshal(data []byte, v interface{})进行转换,返回 error 信息
    if err := json.Unmarshal([]byte(jsonStr), &mapResult); err != nil {
        t.Fatal(err)
    }
    t.Log(mapResult)
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

打印结果信息如下：

```
`/usr/local/``go``/bin/``go` `test -v commontest -run ^TestJson2Map$``  ``json2map_test.``go``:19: ``map``[name:liangyongxing age:12]``ok   commontest 0.007s`
```

### 4. golang 中 map 转 json 串

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
package commontest

import (
    "testing"
    "encoding/json"
)

func TestMap2Json(t *testing.T) {
    mapInstance := make(map[string]interface{})
    mapInstance["Name"] = "liang637210"
    mapInstance["Age"] = 28
    mapInstance["Address"] = "北京昌平区"

    jsonStr, err := json.Marshal(mapInstance)

    if err != nil {
        t.Fatal(err)
    }

    t.Logf("Map2Json 得到 json 字符串内容:%s", jsonStr)
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

打印结果如下：

```
`/usr/local/``go``/bin/``go` `test -v commontest -run ^TestMap2Json$``  ``map2json_test.``go``:20: Map2Json 得到 json 字符串内容:{``"Address"``:``"北京昌平区"``,``"Age"``:28,``"Name"``:``"liang637210"``}``ok   commontest 0.008s`
```

### 5. golang 中 map 转 struct

　　这个转换网上有比较完整的项目，已经上传到对应的 github 上了，需要下载之后使用：

```
`$ go get github.com``/goinggo/mapstructure`
```

 　之后我们就可以直接使用它提供的方法将 map 转换为 struct，让我们直接上代码吧

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
package commontest

import (
    "testing"
    "github.com/goinggo/mapstructure"
)

func TestMap2Struct(t *testing.T) {
    mapInstance := make(map[string]interface{})
    mapInstance["Name"] = "liang637210"
    mapInstance["Age"] = 28

    var person Person
    //将 map 转换为指定的结构体
    if err := mapstructure.Decode(mapInstance, &person); err != nil {
        t.Fatal(err)
    }
    t.Logf("map2struct后得到的 struct 内容为:%v", person)
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

打印结果如下:

```
`/usr/local/go/bin/go` `test` `-``v` `commontest -run ^TestMap2Struct$``  ``map2struct_test.go:18: map2struct后得到的 struct 内容为:{liang637210 28}``ok   commontest 0.009s`
```

### 6. golang 中 struct 转 map 

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 package commontest
 2 
 3 import (
 4     "testing"
 5     "reflect"
 6 )
 7 
 8 type User struct {
 9     Id        int    `json:"id"`
10     Username    string    `json:"username"`
11     Password    string    `json:"password"`
12 }
13 
14 func Struct2Map(obj interface{}) map[string]interface{} {
15     t := reflect.TypeOf(obj)
16     v := reflect.ValueOf(obj)
17 
18     var data = make(map[string]interface{})
19     for i := 0; i < t.NumField(); i++ {
20         data[t.Field(i).Name] = v.Field(i).Interface()
21     }
22     return data
23 }
24 
25 func TestStruct2Map(t *testing.T) {
26     user := User{5, "zhangsan", "password"}
27     data := Struct2Map(user)
28     t.Logf("struct2map得到的map内容为:%v", data)
29 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

打印结果如下：

```
`/usr/local/``go``/bin/``go` `test -v commontest -run ^TestStruct2Map$``  ``struct2map_test.``go``:28: struct2map得到的``map``内容为:``map``[Id:5 Username:zhangsan Password:password]``ok   commontest 0.007s`
```

 