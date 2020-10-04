原文链接：https://thispointer.com//c11-multithreading-part-3-carefully-pass-arguments-to-threads/



想要给std::thread关联的线程中执行的函数传参，只需要在std::thread的构造函数中传入参数即可。默认的，餐胡会被copy到新线程栈中（By default all arguments are copied into the internal storage of new thread）

看一个例子：

### 传递简单参数给线程

```c++
#include <iostream>
#include <string>
#include <thread>
void threadCallback(int x, std::string str)
{
    std::cout<<"Passed Number = "<<x<<std::endl;
    std::cout<<"Passed String = "<<str<<std::endl;
}
int main()  
{
    int x = 10;
    std::string str = "Sample String";
    std::thread threadObj(threadCallback, x, str);
    threadObj.join();
    return 0;
}
```



### 不要向线程中传递地址

不要向线程的回调函数中传递局部变量的地址，因为在第二个线程访问的时候，这个变量可能已经不存在了，这样会造成野指针访问，后果不可预料。

```c++
#include <iostream>
#include <thread>
void newThreadCallback(int * p)
{
    std::cout<<"Inside Thread :  "" : p = "<<p<<std::endl;
    std::chrono::milliseconds dura( 1000 );
    std::this_thread::sleep_for( dura );
    *p = 19;
}
void startNewThread()
{
    int i = 10;
    std::cout<<"Inside Main Thread :  "" : i = "<<i<<std::endl;
    std::thread t(newThreadCallback,&i);
    t.detach();
    std::cout<<"Inside Main Thread :  "" : i = "<<i<<std::endl;
}
int main()
{
    startNewThread();
    std::chrono::milliseconds dura( 2000 );
    std::this_thread::sleep_for( dura );
    return 0;
}
```

其实传递分配在堆上的地址也不可靠，因为这个内存很可能不知道什么时候就被某个线程释放了，另外一个线程再访问也是野指针。

```c++
#include <iostream>
#include <thread>
void newThreadCallback(int * p)
{
    std::cout<<"Inside Thread :  "" : p = "<<p<<std::endl;
    std::chrono::milliseconds dura( 1000 );
    std::this_thread::sleep_for( dura );
    *p = 19;
}
void startNewThread()
{
    int * p = new int();
    *p = 10;
    std::cout<<"Inside Main Thread :  "" : *p = "<<*p<<std::endl;
    std::thread t(newThreadCallback,p);
    t.detach();
    delete p;
    p = NULL;
}
int main()
{
    startNewThread();
    std::chrono::milliseconds dura( 2000 );
    std::this_thread::sleep_for( dura );
    return 0;
}
```

### 向std::thread中的回调传递引用

值得注意的是，如果像通常一样给线程的回调参数传引用， 可能达不到预期的效果，看例子：

```c++
#include <iostream>
#include <thread>
void threadCallback(int const & x)
{
    int & y = const_cast<int &>(x);
    y++;
    std::cout<<"Inside Thread x = "<<x<<std::endl;
}
int main()
{
    int x = 9;
    std::cout<<"In Main Thread : Before Thread Start x = "<<x<<std::endl;
    std::thread threadObj(threadCallback, x);
    threadObj.join();
    std::cout<<"In Main Thread : After Thread Joins x = "<<x<<std::endl;
    return 0;
}
```

输出

```c++
In Main Thread : Before Thread Start x = 9
Inside Thread x = 10
In Main Thread : After Thread Joins x = 9
```

即使给线程的callback传递的是x的引用，但是其改变仍未影响到外部，这是因为线程函数中引用的是一个在新的线程栈中copy的临时对象。

如何修复呢？

答案是使用std::ref()。

```c++
#include <iostream>
#include <thread>
void threadCallback(int const & x)
{
    int & y = const_cast<int &>(x);
    y++;
    std::cout<<"Inside Thread x = "<<x<<std::endl;
}
int main()
{
    int x = 9;
    std::cout<<"In Main Thread : Before Thread Start x = "<<x<<std::endl;
    std::thread threadObj(threadCallback,std::ref(x));
    threadObj.join();
    std::cout<<"In Main Thread : After Thread Joins x = "<<x<<std::endl;
    return 0;
}
```

输出是：

```c++
In Main Thread : Before Thread Start x = 9
Inside Thread x = 10
In Main Thread : After Thread Joins x = 10
```

> 这样做是否有意义？如果是简单变量，多线程访问一个变量需要加锁同步，效率可能还不如传值。如果是复杂对象，这么做会让代码耦合混乱，难以调试。

### 指定一个成员函数作为线程回调函数

将对象本身作为第一个参数传递过期即可：

```c++
#include <iostream>
#include <thread>
class DummyClass {
public:
    DummyClass()
    {}
    DummyClass(const DummyClass & obj)
    {}
    void sampleMemberFunction(int x)
    {
        std::cout<<"Inside sampleMemberFunction "<<x<<std::endl;
    }
};
int main() {
 
    DummyClass dummyObj;
    int x = 10;
    std::thread threadObj(&DummyClass::sampleMemberFunction,&dummyObj, x);
    threadObj.join();
    return 0;
}
```





