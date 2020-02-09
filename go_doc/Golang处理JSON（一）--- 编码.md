# Golang处理JSON（一）--- 编码



### JSON

http的交互的生命周期包含请求和响应。前面我们介绍了很多关于发起请求，处理请求的内容。现在该聊一聊返回响应内容了。对于web服务的响应，以前常见的响应是返回服务端渲染的模板。浏览器只要展示模板即可。随着Restful风格的api出现，已经前后端分离，更多的返回格式是json字串。本节我们将讨论在golang中如何编码和解码json。

JSON是一种数据格式描述语言。以key和value构成的哈系结构，类似javascript中的对象，python中的字典。通常json格式的key是字串，其值可以是任意类型，字串，数字，数组或者对象结构。更多关于json的可以访问[JSON](https://link.jianshu.com?t=http://json.org/)了解。

#### 数据结构map

json源于javascript的对象结构，golang中直接对应的数据结构，可是golang的map也是key-value结构，同时struct结构体也可以描述json。当然，对于json的数据类型，go也会有对象的结构所匹配。大致对应关系如下：

| 数据类型 | JSON   | Golang  |
| -------- | ------ | ------- |
| 字串     | string | string  |
| 整数     | number | int64   |
| 浮点数   | number | flaot64 |
| 数组     | arrary | slice   |
| 对象     | object | struct  |
| 布尔     | bool   | bool    |
| 空值     | null   | nil     |

#### 基本结构编码

golang提供了encoding/json的标准库用于编码json。大致需要两步：

1. 首先定义json结构体。
2. 使用 Marshal方法序列化。

定义结构体的时候，只有字段名是大写的，才会被编码到json当中。

```go
type Account struct {
    Email string
    password string
    Money float64
}

func main() {
    account := Account{
        Email: "rsj217@gmail.com",
        password: "123456",
        Money: 100.5,
    }

    rs, err := json.Marshal(account)
    if err != nil{
        log.Fatalln(err)
    }

    fmt.Println(rs)
    fmt.Println(string(rs))
}
```

可以看到输出如下，Marshal方法接受一个空接口的参数，返回一个[]byte结构。小写命名的password字段没有被编码到json当中，生成的json结构字段和Account结构一致。

```json
[123 34 69 109 97 105 108 34 58 34 114 115 106 50 49 55 64 103 109 97 105 108 46 99 111 109 34 44 34 77 111 110 101 121 34 58 49 48 48 46 53 125]
{"Email":"rsj217@gmail.com","Money":100.5}
```

#### 复合结构编码

相比字串，数字等基本数据结构，slice切片，map图则是复合结构。这些结构编码也类似。不过map的key必须是字串，而value必须是同一类型的数据。

```go
type User struct {
    Name    string
    Age     int
    Roles   []string
    Skill   map[string]float64
}

func main() {

    skill := make(map[string]float64)

    skill["python"] = 99.5
    skill["elixir"] = 90
    skill["ruby"] = 80.0

    user := User{
        Name:"rsj217",
        Age: 27,
        Roles: []string{"Owner", "Master"},
        Skill: skill,
    }

    rs, err := json.Marshal(user)
    if err != nil{
        log.Fatalln(err)
    }
    fmt.Println(string(rs))
}
```

输入：

```json
{
    "Name":"rsj217",
    "Age":27,
    "Roles":[
        "Owner",
        "Master"
    ],
    "Skill":{
        "elixir":90,
        "python":99.5,
        "ruby":80
    }
}
```

#### 嵌套编码

slice和map可以匹配json的数组和对象，当前提是对象的value是同类型的情况。更通用的做法，对象的key可以是string，但是其值可以是多种结构。golang可以通过定义结构体实现这种构造：

```go
type User struct {
    Name    string
    Age     int
    Roles   []string
    Skill   map[string]float64
    Account Account
}

func main(){
    
    ...

    user := User{
        Name:"rsj217",
        Age: 27,
        Roles: []string{"Owner", "Master"},
        Skill: skill,
        Account:account,
    }
    ...
}
```

输出：

```json
{
    "Name":"rsj217",
    "Age":27,
    "Roles":[
        "Owner",
        "Master"
    ],
    "Skill":{
        "elixir":90,
        "python":99.5,
        "ruby":80
    },
    "Account":{
        "Email":"rsj217@gmail.com",
        "Money":100.5
    }
}
```

通过定义嵌套的结构体Account，实现了key与value不一样的结构。golang的数组或切片，其类型也是一样的，如果遇到不同数据类型的数组，则需要借助空结构来实现：

```go
type User struct {
    ...

    Extra []interface{}
}

extra := []interface{}{123, "hello world"}

user := User{
    ...
    
    Extra:   extra,
}
```

输出：

```bash
{
    ...
    "Extra":[
        123,
        "hello world"
    ]
}
```

使用空接口，也可以定义像结构体实现那种不同value类型的字典结构。当空接口没有初始化其值的时候，零值是 nil。编码成json就是 null

```go
type User struct {
    Name    string
    Age     int
    Roles   []string
    Skill   map[string]float64
    Account Account

    Extra []interface{}

    Level map[string]interface{}
}

func main() {

    ...

    level := make(map[string]interface{})

    level["web"] = "Good"
    level["server"] = 90
    level["tool"] = nil

    user := User{
        Name:    "rsj217",
        Age:     27,
        Roles:   []string{"Owner", "Master"},
        Skill:   skill,
        Account: account,
        Level:   level,
    }

    ...
}
```

输出：

```csharp
{
    ...
    "Extra":null,
    "Level":{
        "server":90,
        "tool":null,
        "web":"Good"
    }
}
```

可以看到 Extra返回的并不是一个空的切片，而是null。同时Level字段实现了向字典的嵌套结构。

#### StructTag 字段重名

通过上面的例子，我们看到了`Level`字段中的key`server`等是小写字母，其他的都是大写字母。因为我们在定义结构的时候，只有使用大写字母开头的字段才会被导出。而通常json世界中，更盛行小写字母的方式。看起来就成了一个矛盾。其实不然，golang提供了`struct tag`的方式可以重命名结构字段的输出形式。

```go
type Account struct {
    Email    string  `json:"email"`
    Password string  `json:"pass_word"`
    Money    float64 `json:"money"`
}

func main() {
    account := Account{
        Email:    "rsj217@gmail.com",
        Password: "123456",
        Money:    100.5,
    }

    rs, err := json.Marshal(account)
    ...
}
```

我们使用struct tag，重新给Aaccount结构的字段进行了重命名。其中email小写了，并且password字段还使用了下划线，输出的结果如下：

```json
{"email":"rsj217@gmail.com","pass_word":"123456","money":100.5}
```

#####  `-`忽略字段

重命名的可以一个利器，这个利器还提供了更高级的选项。通常使用marshal的时候，会把结构体的所有除了私有字段都编码到json，而实际开发中，我们定义的结构可能更通用，我们需要某个字段可以导出，但是又不能编码到json中。

此时使用 struact tag的 `-`符号就能完美解决，我们已经知道`_`常用于忽略字段的占位，在tag中则使用短横线`-`。

```go
type Account struct {
    Email    string  `json:"email"`
    Password string  `json:"password,omitempty"`
    Money    float64 `json:"money"`
}
```

输出：

```json
{"email":"rsj217@gmail.com","money":100.5}
```

可见即使Password不是私有字段，因为`-`忽略了它，因此没有被编码到json输出。

#####  `omitempty`可选字段

对于另外一种字段，当其有值的时候就输出，而没有值(零值)的时候就不输出，则可以使用另外一种选项`omitempty`。

```go
type Account struct {
    Email    string  `json:"email"`
    Password string  `json:"password,omitempty"`
    Money    float64 `json:"money"`
}
func main() {
    account := Account{
        Email:    "rsj217@gmail.com",
        Password: "",
        Money:    100.5,
    }
    ...
}
```

此时password不会被编码到json输出中。

#####  `string`选项

gplang是静态类型语言，对于类型定义的是不能动态修改。在json处理当中，struct tag的string可以起到部分动态类型的效果。有时候输出的json希望是数字的字符串，而定义的字段是数字类型，那么就可以使用string选项。

```go
type Account struct {
    Email    string  `json:"email"`
    Password string  `json:"password,omitempty"`
    Money    float64 `json:"money,string"`
}
func main() {
    account := Account{
        Email:    "rsj217@gmail.com",
        Password: "123",
        Money:    100.50,
    }

    ...
}
```

可以看到输出为 `money: "100.5"`， money字段的值是字串。（其实能转换成`100.50`会比转换成`100.5`更好，可是我没有找到通过tag的方式实现 :(）。

### 总结

上面所介绍的大致覆盖了golang的json编码处理。总体原则分两步，首先定义需要编码的结构，然后调用encoding/json标准库的Marshal方法生成json byte数组，转换成string类型即可。

golang和json的大部分数据结构匹配，对于复合结构，go可以借助结构体和空接口实现json的数组和对象结构。通过struct tag可以灵活的修改json编码的字段名和输出控制。

既然有JSON的编码，当然就会有JSON的解码。相比编码JSON，解析JSON对于golang则需要更多的技巧。下一节我们聊一聊golang解析json相关的内容。