# Golang处理JSON（二）--- 解码



golang编码json还比较简单，而解析json则非常蛋疼。不像Python一句json.loads就能搞定。之前项目开发中，为了兼容不同客户端的需求，请求的content-type可以是json，也可以是www-x-urlencode。然后某天前端希望某个后端服务提供json的处理，而当时后端使用java实现了www-x-urlencode的请求，对于突然希望提供json处理产生了极大的情绪。当时不太理解，现在看来，对于静态语言解析未知的JSON确实是一项挑战。

### 定义结构

与编码json的Marshal类似，解析json也提供了Unmarshal方法。对于解析json，也大致分两步，首先定义结构，然后调用Unmarshal方法序列化。我们先从简单的例子开始吧

```go
type Account struct {
    Email    string  `json:"email"`
    Password string  `json:"password"`
    Money    float64 `json:"money"`
}

var jsonString string = `{
    "email":"rsj217@gmail.com",
    "password":"123",
    "money":100.5
}`

func main() {

    account := Account{}

    err := json.Unmarshal([]byte(jsonString), &account)
    if err != nil{
        log.Fatalln(err)
    }
    fmt.Printf("%+v\n", account)
}
```

Unmarshal接受一个byte数组和空接口指针的参数。和sql中读取数据类似，先定义一个数据实例，然后传其指针地址。

与编码类似，golang会将json的数据结构和go的数据结构进行匹配。匹配的原则就是寻找tag的相同的字段，然后查找字段。查询的时候是大小写不敏感的：

```go
type Account struct {
    Email    string  `json:"email"`
    PassWord string  
    Money    float64 `json:"money"`
}
```

输出`{Email:rsj217@gmail.com PassWord:123 Money:100.5}`，把 Password的tag去掉，再修改成PassWord，依然可以把json的password匹配到PassWord，但是如果结构的字段是私有的，即使tag符合，也不会被解析：

```go
type Account struct {
    Email    string  `json:"email"`
    password string  `json:"password"`
    Money    float64 `json:"money"`
}
```

输出`{Email:rsj217@gmail.com password: Money:100.5}`。上面的password并不会被解析赋值json的password，大小写不敏感只是针对公有字段而言。再寻找tag或字段的时候匹配不成功，则会抛弃这个json字段的值：

```cpp
type Account struct {
    Email    string  `json:"email"`
    Password string  `json:"password"`
}
```

输出 `{Email:rsj217@gmail.com Password:"123"}`， 并不会有money字段被赋值。

#### string tag

在编码的时候，我们使用tag string，可以把结构定义的数字类型以字串形式编码。同样在解码的时候，只有字串类型的数字，才能被正确解析，或者会报错：

```go
type Account struct {
    Email    string  `json:"email"`
    Password string  `json:"password"`
    Money    float64 `json:"money,string"`
}

var jsonString string = `{
    "email":"rsj217@gmail.com",
    "password":"123",
    "money":"100.5"
}`

func main() {

    account := Account{}

    err := json.Unmarshal([]byte(jsonString), &account)
    if err != nil{
        log.Fatalln(err)
    }
    fmt.Printf("%+v\n", account)
}
```

输出： `{Email:rsj217@gmail.com Password:123 Money:100.5}`， Money是float64类型。

如果json的money是`100.5`， 会得到下面的错误：

```csharp
2016/12/23 18:12:32 json: invalid use of ,string struct tag, trying to unmarshal unquoted value into float64
exit status 1
```

####  `-` tag

与编码一样，tag的`-`也不会被解析，但是会初始化其零值：

```go
type Account struct {
    Email    string  `json:"email"`
    Password string  `json:"password"`
    Money    float64 `json:"-"`
}
```

输出：`{Email:rsj217@gmail.com Password:123 Money:0}`。

稍微总结一下，解析json最好的方式就是定义与将要被解析json的结构。有人写了一个小工具[json-to-go](https://link.jianshu.com?t=https://mholt.github.io/json-to-go/)，自动将json格式化成golang的结构。

### 动态解析

通常更加json的格式预先定义golang的结构进行解析是最理想的情况。可是实际开发中，理想的情况往往都存在理想的愿望之中，很多json非但格式不确定，有的还可能是动态数据类型。

例如通常登录的时候，往往既可以使用手机号做用户名，也可以使用邮件做用户名，客户端传的json可以是字串，也可以是数字。此时服务端解析就需要技巧了。

#### Decode

前面我们使用了简单的方法Unmarshal直接解析json字串，下面我们使用更底层的方法NewDecode和Decode方法。

```go
type User struct {
    UserName string `json:"username"`
    Password string     `json:"password"`
}

var jsonString string = `{
    "username":"rsj217@gmail.com",
    "password":"123"
}`

func Decode(r io.Reader)(u *User, err error)  {
    u = new(User)
    err = json.NewDecoder(r).Decode(u)
    if err != nil{
        return
    }
    return
}

func main() {
    user, err := Decode(strings.NewReader(jsonString))
    if err !=nil{
        log.Fatalln(err)
    }
    fmt.Printf("%#v\n",user)
}
```

我们定义了一个Decode函数，在这个函数进行json字串的解析。然后调用json的NewDecoder方法构造一个Decode对象，最后使用这个对象的Decode方法赋值给定义好的结构对象。

> 对于字串，可是使用strings.NewReader方法，让字串变成一个Stream对象。

#### 接口

如果客户端传的username的值是一个数字类型的手机号，那么上面的解析方法将会失败。正如我们之前所介绍的动态类型行为一样，使用空接口可以hold住这样的情景。

```csharp
type User struct {
    UserName interface{} `json:"username"`
    Password string      `json:"password"`
}
```

然后再运行发送输出为 `&main.User{UserName:1.8512341234e+10, Password:"123"}`。怎么说，貌似成功了，可是返回的数字是科学计数法，有点奇怪。可以使用golang的断言，然后转换类型：

```go
func Decode(r io.Reader) (u *User, err error) {
    u = new(User)
    if err = json.NewDecoder(r).Decode(u); err != nil{
        return
    }
    switch t := u.UserName.(type) {
    case string:
        u.UserName = t
    case float64:
        u.UserName = int64(t)
    }
    return
}

func main() {
    user, err := Decode(strings.NewReader(jsonString))
    if err != nil {
        log.Fatalln(err)
    }
    fmt.Printf("%#v\n", user)
}
```

输出 `&main.User{UserName:18512341234, Password:"123"}`。看起来挺好，可是我们的UserName字段始终是一个空接口，使用他的时候，还是需要转换类型，这样情况看来，解析的时候就应该转换好类型，那么用的时候就省心了。

修改定义的结构如下：

```go
type User struct {
    UserName interface{} `json:"username"`
    Password string      `json:"password"`

    Email string
    Phone int64
}
```

这样就能通过 `fmt.Println(user.Email + " add me")`使用字段进行操作了。当然也有人认为Email和Phone纯粹多于，因为使用的时候，还是需要再判断当前结构实例是那种情况。

#### 延迟解析

因为UserName字段，实际上是在使用的时候，才会用到他的具体类型，因此我们可以延迟解析。使用json.RawMessage方式，将json的字串继续以byte数组方式存在。

```go
type User struct {
    UserName json.RawMessage `json:"username"`
    Password string      `json:"password"`

    Email string
    Phone int64
}

var jsonString string = `{
    "username":"18512341234@qq.com",
    "password":"123"
}`

func Decode(r io.Reader) (u *User, err error) {
    u = new(User)
    if err = json.NewDecoder(r).Decode(u); err != nil{
        return
    }
    var email string
    if err = json.Unmarshal(u.UserName, &email); err == nil{
        u.Email = email
        return
    }
    var phone int64
    if err = json.Unmarshal(u.UserName, &phone); err == nil{
        u.Phone = phone
    }
    return
}

func main() {
    user, err := Decode(strings.NewReader(jsonString))
    if err != nil {
        log.Fatalln(err)
    }
    fmt.Printf("%#v\n", user)
}
```

总体而言，延迟解析和使用空接口的方式类似。需要再次调用Unmarshal方法，对json.RawMessage进行解析。原理和解析到接口的形式类似。

### 不定字段解析

对于未知json结构的解析，不同的数据类型可以映射到接口或者使用延迟解析。有时候，会遇到json的数据字段都不一样的情况。例如需要解析下面一个json字串：

#### 接口配合断言

```csharp
var jsonString string = `{
        "things": [
            {
                "name": "Alice",
                "age": 37
            },
            {
                "city": "Ipoh",
                "country": "Malaysia"
            },
            {
                "name": "Bob",
                "age": 36
            },
            {
                "city": "Northampton",
                "country": "England"
            }
        ]
    }`
```

json字串的是一个对象，其中一个key things的值是一个数组，这个数组的每一个item都未必一样，大致是两种数据结构，可以抽象为person和place。即，定义下面的结构体：

```cpp
type Person struct {
    Name string `json:"name"`
    Age  int    `json:"age"`
}

type Place struct {
    City    string `json:"city"`
    Country string `json:"country"`
}
```

接下来我们Unmarshal json字串到一个map结构，然后迭代item并使用type断言的方式解析数据：

```go
func decode(jsonStr []byte) (persons []Person, places []Place) {
    var data map[string][]map[string]interface{}
    err := json.Unmarshal(jsonStr, &data)
    if err != nil {
        fmt.Println(err)
        return
    }

    for i := range data["things"] {
        item := data["things"][i]
        if item["name"] != nil {
            persons = addPerson(persons, item)
        } else {
            places = addPlace(places, item)
        }

    }
    return
}
```

迭代的时候会判断item是否是person还是place，然后调用对应的解析方法：

```go
func addPerson(persons []Person, item map[string]interface{}) []Person {
    name := item["name"].(string)
    age := item["age"].(float64)
    person := Person{name, int(age)}
    persons = append(persons, person)
    return persons
}

func addPlace(places []Place, item map[string]interface{})([]Place){
    city := item["city"].(string)
    country := item["country"].(string)
    place := Place{City:city, Country:country}
    places = append(places, place)
    return places
}
```

最后调用和输出如下：

```go
func main() {
    personsA, placesA := decode([]byte(jsonString))
    fmt.Printf("%+v\n", personsA)
    fmt.Printf("%+v\n", placesA)
}


/usr/local/go/bin/go run /Users/ghost/Rsj217/go/src/demo/main.go
[{Name:Alice Age:37} {Name:Bob Age:36}]
[{City:Ipoh Country:Malaysia} {City:Northampton Country:England}]
```

#### 混合结构

混合结构很好理解，如同我们前面解析username为 email和phone两种情况，就在结构中定义好这两种结构即可。

```cpp
type Mixed struct {
    Name    string `json:"name"`
    Age     int    `json:"age"`
    city    string `json:"city"`
    Country string `json:"country"`
}
```

混合结构的思路很简单，借助golang会初始化没有匹配的json和抛弃没有匹配的json，给特定的字段赋值。比如每一个item都具有四个字段，只不过有的会匹配person的json数据，有的则是匹配place。没有匹配的字段则是零值。接下来在根据item的具体情况，分别赋值到对于的Person或Place结构。

```go
func decode(jsonStr []byte) (persons []Person, places []Place) {
    var data map[string][]Mixed
    err := json.Unmarshal(jsonStr, &data)
    if err != nil {
        fmt.Println(err)
        return
    }
    fmt.Printf("%+v\n", data["things"])
    for i := range data["things"] {
        item := data["things"][i]
        if item.Name != "" {
            persons = append(persons, Person{Name: item.Name, Age: item.Age})
        } else {
            places = append(places, Place{City: item.city, Country: item.Country})
        }
    }
    return
}
```

混合结构的解析方式也很不错。思路还是借助了解析json中抛弃不要的字段，借助零值处理。

#### json.RawMessage

json.RawMessage非常有用，延迟解析也可以使用这个样例。我们已经介绍过类似的技巧，下面就贴代码了：

```go
func addPerson(item json.RawMessage, persons []Person)([]Person){
    person := Person{}
    if err := json.Unmarshal(item, &person); err != nil{
        fmt.Println(err)
    }else{
        if person != *new(Person){
            persons = append(persons, person)
        }
    }
    return persons
}

func addPlace(item json.RawMessage, places []Place)([]Place){
    place :=Place{}
    if err := json.Unmarshal(item, &place); err != nil{
        fmt.Println(err)
    }else{
        if place != *new(Place){
            places = append(places, place)
        }
    }
    return places
}


func decode(jsonStr []byte)(persons []Person, places []Place){
    var data map[string][]json.RawMessage
    err := json.Unmarshal(jsonStr, &data)
    if err != nil{
        fmt.Println(err)
        return
    }

    for _, item := range data["things"]{
        persons = addPerson(item, persons)
        places = addPlace(item, places)
    }
    return
}
```

把things的item数组解析成一个json.RawMessage，然后再定义其他结构逐步解析。上述这些例子其实在真实的开发环境下，应该尽量避免。像person或是place这样的数据，可以定义两个数组分别存储他们，这样就方便很多。不管怎么样，通过这个略傻的例子，我们也知道了如何解析json数据。

### 总结

关于golang解析json的介绍基本就这么多。想要解析越简单，就需要定义越明确的map结构。面对无法确定的数据结构或类型，再动态解析方面可以借助接口与断言的方式解析，也可以使用json.RawMessage延迟解析。具体使用情况，还得考虑实际的需求和应用场景。

总而言之，使用json作为现在api的数据通信方式已经很普遍了。我们从启动服务，构造请求，解析请求，渲染模板，持久化到现在的json解析，涉及一个request-response生命周期的介绍完整了。

接下来自然就是利用这些，构建一个简单的web应用。当然，我们可能根据需要，构建不同的应用。