首先要说明的一个问题是：如何安全地将this指针返回给调用者。一般来说，我们不能直接将this指针返回。想象这样的情况，该函数将this指针返回到外部某个变量保存，然后这个对象自身已经析构了，但外部变量并不知道，此时如果外部变量使用这个指针，就会使得程序崩溃。

使用智能指针shared_ptr看起来是个不错的解决方法。但问题是如何去使用它呢？我们来看如下代码：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
#include <iostream>
#include <boost/shared_ptr.hpp>
class Test
{
public:
    //析构函数
    ~Test() { std::cout << "Test Destructor." << std::endl; }
    //获取指向当前对象的指针
    boost::shared_ptr<Test> GetObject()
    {
        boost::shared_ptr<Test> pTest(this);
        return pTest;
    }
};
int main(int argc, char *argv[])
{
    {
        boost::shared_ptr<Test> p( new Test( ));
        std::cout << "q.use_count(): " << q.use_count() << std::endl; 
        boost::shared_ptr<Test> q = p->GetObject();
    }
    return 0;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

运行后，程序输出：

　　`Test Destructor.`

　　q.use_count(): 1

```
　　``Test Destructor.
```

可以看到，对象只构造了一次，但却析构了两次。并且在增加一个指向的时候，shared_ptr的计数并没有增加。也就是说，这个时候，p和q都认为自己是Test指针的唯一拥有者，这两个shared_ptr在计数为0的时候，都会调用一次Test对象的析构函数，所以会出问题。

那么为什么会这样呢？给一个shared_ptr<Test>传递一个this指针难道不能引起shared_ptr<Test>的计数吗？

答案是：对的，shared_ptr<Test>根本认不得你传进来的指针变量是不是之前已经传过。

看这样的代码：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
int main()
{
    Test* test = new Test();
    shared_ptr<Test> p(test);
    shared_ptr<Test> q(test);
    std::cout << "p.use_count(): " << p.use_count() << std::endl;
    std::cout << "q.use_count(): " << q.use_count() << std::endl;
    return 0;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

运行后，程序输出：

p.use_count(): 1
q.use_count(): 1
Test Destructor.
Test Destructor.

也证明了刚刚的论述：shared_ptr<Test>根本认不得你传进来的指针变量是不是之前已经传过。

事实上，类对象是由外部函数通过某种机制分配的，而且一经分配立即交给 shared_ptr管理，而且以后凡是需要共享使用类对象的地方，必须使用这个 shared_ptr当作右值来构造产生或者拷贝产生（shared_ptr类中定义了赋值运算符函数和拷贝构造函数）另一个shared_ptr ，从而达到共享使用的目的。

 

解释了上述现象后，现在的问题就变为了：如何在类对象（Test）内部中获得一个指向当前对象的shared_ptr 对象？如果我们能够做到这一点，直接将这个shared_ptr对象返回，就不会造成新建的shared_ptr的问题了。

 

下面来看看enable_shared_from_this类的威力。

enable_shared_from_this 是一个以其派生类为模板类型参数的基类模板，继承它，派生类的this指针就能变成一个 shared_ptr。

有如下代码：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
#include <iostream>
#include <boost/enable_shared_from_this.hpp>
#include <boost/shared_ptr.hpp>
class Test : public boost::enable_shared_from_this<Test>        //改进1
{
public:
    //析构函数
    ~Test() { std::cout << "Test Destructor." << std::endl; }
    //获取指向当前对象的指针
    boost::shared_ptr<Test> GetObject()
    {
        return shared_from_this();      //改进2
    }
};
int main(int argc, char *argv[])
{
    {
        boost::shared_ptr<Test> p( new Test( ));
        std::cout << "q.use_count(): " << q.use_count() << std::endl;
        boost::shared_ptr<Test> q = p->GetObject();
    }
    return 0;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

运行后，程序输出：

　　Test Destructor.

　　q.use_count(): 2;

可以看到，问题解决了！

 

接着来看看enable_shared_from_this 是如何工作的，以下是它的源码：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
template<class T> class enable_shared_from_this
{
protected:

    BOOST_CONSTEXPR enable_shared_from_this() BOOST_SP_NOEXCEPT
    {
    }

    BOOST_CONSTEXPR enable_shared_from_this(enable_shared_from_this const &) BOOST_SP_NOEXCEPT
    {
    }

    enable_shared_from_this & operator=(enable_shared_from_this const &) BOOST_SP_NOEXCEPT
    {
        return *this;
    }

    ~enable_shared_from_this() BOOST_SP_NOEXCEPT // ~weak_ptr<T> newer throws, so this call also must not throw
    {
    }

public:

    shared_ptr<T> shared_from_this()
    {
        shared_ptr<T> p( weak_this_ );
        BOOST_ASSERT( p.get() == this );
        return p;
    }

    shared_ptr<T const> shared_from_this() const
    {
        shared_ptr<T const> p( weak_this_ );
        BOOST_ASSERT( p.get() == this );
        return p;
    }

    weak_ptr<T> weak_from_this() BOOST_SP_NOEXCEPT
    {
        return weak_this_;
    }

    weak_ptr<T const> weak_from_this() const BOOST_SP_NOEXCEPT
    {
        return weak_this_;
    }

public: // actually private, but avoids compiler template friendship issues

    // Note: invoked automatically by shared_ptr; do not call
    template<class X, class Y> void _internal_accept_owner( shared_ptr<X> const * ppx, Y * py ) const BOOST_SP_NOEXCEPT
    {
        if( weak_this_.expired() )
        {
            weak_this_ = shared_ptr<T>( *ppx, py );
        }
    }

private:

    mutable weak_ptr<T> weak_this_;
};

} // namespace boost

#endif  // #ifndef BOOST_SMART_PTR_ENABLE_SHARED_FROM_THIS_HPP_INCLUDED
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

标黄部分是shared_from_this()函数的实现。可以看到，这个函数使用一个weak_ptr对象(weak_this_)来构造一个 shared_ptr对象，然后将shared_ptr对象返回。

注意这个weak_ptr是实例对象的一个成员变量，所以对于一个对象来说，它一直是同一个，每次在调用shared_from_this()时，就会根据weak_ptr来构造一个临时shared_ptr对象。

也许看到这里会产生疑问，这里的shared_ptr也是一个临时对象，和前面有什么区别？还有，为什么enable_shared_from_this 不直接保存一个 shared_ptr 成员？

对于第一个问题，这里的每一个shared_ptr都是根据weak_ptr来构造的，而每次构造shared_ptr的时候，使用的参数是一样的，所以这里根据相同的weak_ptr来构造多个临时shared_ptr等价于用一个shared_ptr来做拷贝。（PS：在shared_ptr类中，是有使用weak_ptr对象来构造shared_ptr对象的构造函数的：

template<class Y>
    explicit shared_ptr( weak_ptr<Y> const & r ): pn( r.pn )

）

对于第二个问题，假设我在类里储存了一个指向自身的shared_ptr，那么这个 shared_ptr的计数最少都会是1，也就是说，这个对象将永远不能析构，所以这种做法是不可取的。

在enable_shared_from_this类中，没有看到给成员变量weak_this_初始化赋值的地方，那究竟是如何保证weak_this_拥有着Test类对象的指针呢？

首先我们生成类T时，会依次调用enable_shared_from_this类的构造函数（定义为protected），以及类Test的构造函数。在调用enable_shared_from_this的构造函数时，会初始化定义在enable_shared_from_this中的私有成员变量weak_this_（调用其默认构造函数），这时的weak_this_是无效的（或者说不指向任何对象）。

接着，当外部程序把指向类Test对象的指针作为初始化参数来初始化一个shared_ptr（boost::shared_ptr<Test> p( new Test( ));）。

现在来看看 shared_ptr是如何初始化的，shared_ptr 定义了如下构造函数：

```
template<class Y>
    explicit shared_ptr( Y * p ): px( p ), pn( p ) 
    {
        boost::detail::sp_enable_shared_from_this( this, p, p );
    }
```

里面调用了 **boost::detail::sp_enable_shared_from_this** ：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
template< class X, class Y, class T >
 inline void sp_enable_shared_from_this( boost::shared_ptr<X> const * ppx,
 Y const * py, boost::enable_shared_from_this< T > const * pe )
{
    if( pe != 0 )
    {
        pe->_internal_accept_owner( ppx, const_cast< Y* >( py ) );
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

里面又调用了**enable_shared_from_this** 的 **_internal_accept_owner** ：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
template<class X, class Y> void _internal_accept_owner( shared_ptr<X> const * ppx, Y * py ) const
    {
        if( weak_this_.expired() )
        {
            weak_this_ = shared_ptr<T>( *ppx, py );
        }
    }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

而在这里，对enable_shared_from_this 类的成员weak_this_进行拷贝赋值，使得weak_this_作为类对象 shared_ptr 的一个观察者。

这时，当类对象本身需要自身的shared_ptr时，就可以从这个weak_ptr来生成一个了：

```
shared_ptr<T> shared_from_this()
    {
        shared_ptr<T> p( weak_this_ );
        BOOST_ASSERT( p.get() == this );
        return p;
    }
```

 

以上。

 

从上面的说明来看，需要小心的是shared_from_this()仅在shared_ptr<T>的构造函数被调用之后才能使用，原因是enable_shared_from_this::weak_this_并不在构造函数中设置，而是在shared_ptr<T>的构造函数中设置。

所以，如下代码是错误的：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
class D:public boost::enable_shared_from_this<D>
{
public:
    D()
    {
        boost::shared_ptr<D> p=shared_from_this();
    }
};
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

原因是在D的构造函数中虽然可以保证enable_shared_from_this<D>的构造函数被调用，但weak_this_是无效的（还还没被接管）。

如下代码也是错误的：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
class D:public boost::enable_shared_from_this<D>
{
public:
    void func()
    {
        boost::shared_ptr<D> p=shared_from_this();
    }
};
void main()
{
    D d;
    d.func();
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

原因同上。

总结为：不要试图对一个没有被shared_ptr接管的类对象调用shared_from_this()，不然会产生未定义行为的错误