chrono是一个time library, 源于boost，现在已经是C++标准。话说今年似乎又要出新标准了，好期待啊！ 

　　要使用chrono库，需要#include<chrono>，其所有实现均在std::chrono  namespace下。注意标准库里面的每个命名空间代表了一个独立的概念。所以下文中的概念均以命名空间的名字表示！ chrono是一个模版库，使用简单，功能强大，只需要理解三个概念：duration、time_point、clock

 

1.Durations

std::chrono::duration 表示一段时间，比如两个小时，12.88秒，半个时辰，一炷香的时间等等，只要能换算成秒即可。

```
1 template <class Rep, class Period = ratio<1> > class duration;
```

 

其中

Rep表示一种数值类型，用来表示Period的数量，比如int float double

Period是ratio类型，用来表示【用秒表示的时间单位】比如second milisecond

常用的duration<Rep,Period>已经定义好了，在std::chrono::duration下：

ratio<3600, 1>        hours

ratio<60, 1>          minutes

ratio<1, 1>           seconds

ratio<1, 1000>        microseconds

ratio<1, 1000000>     microseconds

ratio<1, 1000000000>  nanosecons

 

这里需要说明一下ratio这个类模版的原型：

```
1 template <intmax_t N, intmax_t D = 1> class ratio;
```

 

N代表分子，D代表分母，所以ratio表示一个分数值。

注意，我们自己可以定义Period，比如ratio<1, -2>表示单位时间是-0.5秒。

 

由于各种duration表示不同，chrono库提供了duration_cast类型转换函数。

```
1 template <class ToDuration, class Rep, class Period>
2   constexpr ToDuration duration_cast (const duration<Rep,Period>& dtn);
```

 

典型的用法是表示一段时间：

 

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1 // duration constructor
 2 #include <iostream>
 3 #include <ratio>
 4 #include <chrono>
 5  
 6 int main ()
 7 {
 8   typedef std::chrono::duration<int> seconds_type;
 9   typedef std::chrono::duration<int,std::milli> milliseconds_type;
10   typedef std::chrono::duration<int,std::ratio<60*60>> hours_type;
11  
12   hours_type h_oneday (24);                  // 24h
13   seconds_type s_oneday (60*60*24);          // 86400s
14   milliseconds_type ms_oneday (s_oneday);    // 86400000ms
15  
16   seconds_type s_onehour (60*60);            // 3600s
17 //hours_type h_onehour (s_onehour);          // NOT VALID (type truncates), use:
18   hours_type h_onehour (std::chrono::duration_cast<hours_type>(s_onehour));
19   milliseconds_type ms_onehour (s_onehour);  // 3600000ms (ok, no type truncation)
20  
21   std::cout << ms_onehour.count() << "ms in 1h" << std::endl;
22  
23   return 0;
24 }
25  
26 duration还有一个成员函数count()返回Rep类型的Period数量，看代码：
27 
28 // duration::count
29 #include <iostream>     // std::cout
30 #include <chrono>       // std::chrono::seconds, std::chrono::milliseconds
31                         // std::chrono::duration_cast
32  
33 int main ()
34 {
35   using namespace std::chrono;
36   // std::chrono::milliseconds is an instatiation of std::chrono::duration:
37   milliseconds foo (1000); // 1 second
38   foo*=60;
39  
40   std::cout << "duration (in periods): ";
41   std::cout << foo.count() << " milliseconds.\n";
42  
43   std::cout << "duration (in seconds): ";
44   std::cout << foo.count() * milliseconds::period::num / milliseconds::period::den;
45   std::cout << " seconds.\n";
46  
47   return 0;
48 }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

 

***2.Time points\***

std::chrono::time_point  表示一个具体时间，如上个世纪80年代、你的生日、今天下午、火车出发时间等，只要它能用计算机时钟表示。鉴于我们使用时间的情景不同，这个time  point具体到什么程度，由选用的单位决定。一个time point必须有一个clock计时。参见clock的说明。

 

```
1 template <class Clock, class Duration = typename Clock::duration>  class time_point;
```

 

 

下面是构造使用time_point的例子：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1 // time_point constructors
 2 #include <iostream>
 3 #include <chrono>
 4 #include <ctime>
 5  
 6 int main ()
 7 {
 8   using namespace std::chrono;
 9  
10   system_clock::time_point tp_epoch;    // epoch value
11  
12   time_point <system_clock,duration<int>> tp_seconds (duration<int>(1));
13  
14   system_clock::time_point tp (tp_seconds);
15  
16   std::cout << "1 second since system_clock epoch = ";
17   std::cout << tp.time_since_epoch().count();
18   std::cout << " system_clock periods." << std::endl;
19  
20   // display time_point:
21   std::time_t tt = system_clock::to_time_t(tp);
22   std::cout << "time_point tp is: " << ctime(&tt);
23  
24   return 0;
25 }
26  
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

time_point有一个函数time_from_eproch()用来获得1970年1月1日到time_point时间经过的duration。

举个例子，如果timepoint以天为单位，函数返回的duration就以天为单位。

 

由于各种time_point表示方式不同，chrono也提供了相应的转换函数 time_point_cast。

```
1 template <class ToDuration, class Clock, class Duration>
2   time_point<Clock,ToDuration> time_point_cast (const time_point<Clock,Duration>& tp);
```

 

比如计算

/ 

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1 / time_point_cast
 2 #include <iostream>
 3 #include <ratio>
 4 #include <chrono>
 5  
 6 int main ()
 7 {
 8   using namespace std::chrono;
 9  
10   typedef duration<int,std::ratio<60*60*24>> days_type;
11  
12   time_point<system_clock,days_type> today = time_point_cast<days_type>(system_clock::now());
13  
14   std::cout << today.time_since_epoch().count() << " days since epoch" << std::endl;
15  
16   return 0;
17 }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

3.Clocks

 

std::chrono::system_clock 它表示当前的系统时钟，系统中运行的所有进程使用now()得到的时间是一致的。

每一个clock类中都有确定的time_point, duration, Rep, Period类型。

操作有：

now() 当前时间time_point

to_time_t() time_point转换成time_t秒

from_time_t() 从time_t转换成time_point

典型的应用是计算时间日期：



[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1 // system_clock example
 2 #include <iostream>
 3 #include <ctime>
 4 #include <ratio>
 5 #include <chrono>
 6  
 7 int main ()
 8 {
 9   using std::chrono::system_clock;
10  
11   std::chrono::duration<int,std::ratio<60*60*24> > one_day (1);
12  
13   system_clock::time_point today = system_clock::now();
14   system_clock::time_point tomorrow = today + one_day;
15  
16   std::time_t tt;
17  
18   tt = system_clock::to_time_t ( today );
19   std::cout << "today is: " << ctime(&tt);
20  
21   tt = system_clock::to_time_t ( tomorrow );
22   std::cout << "tomorrow will be: " << ctime(&tt);
23  
24   return 0;
25 }
26  
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

std::chrono::steady_clock 为了表示稳定的时间间隔，后一次调用now()得到的时间总是比前一次的值大（这句话的意思其实是，如果中途修改了系统时间，也不影响now()的结果），每次tick都保证过了稳定的时间间隔。

操作有：

now() 获取当前时钟

典型的应用是给算法计时：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1 // steady_clock example
 2 #include <iostream>
 3 #include <ctime>
 4 #include <ratio>
 5 #include <chrono>
 6  
 7 int main ()
 8 {
 9   using namespace std::chrono;
10  
11   steady_clock::time_point t1 = steady_clock::now();
12  
13   std::cout << "printing out 1000 stars...\n";
14   for (int i=0; i<1000; ++i) std::cout << "*";
15   std::cout << std::endl;
16  
17   steady_clock::time_point t2 = steady_clock::now();
18  
19   duration<double> time_span = duration_cast<duration<double>>(t2 - t1);
20  
21   std::cout << "It took me " << time_span.count() << " seconds.";
22   std::cout << std::endl;
23  
24   return 0;
25 }
26  
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

最后一个时钟，std::chrono::high_resolution_clock 顾名思义，这是系统可用的最高精度的时钟。实际上high_resolution_clock只不过是system_clock或者steady_clock的typedef。

操作有：

now() 获取当前时钟。