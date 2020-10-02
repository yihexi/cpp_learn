原文链接：https://thispointer.com//c-11-multithreading-part-1-three-different-ways-to-create-threads/



这篇文章讨论了如何使用std::thread创建线程。



###  C++11线程库介绍

原来的C++标准库只支持单线程，新的C++11标准引入了一个新的线程库。

需要环境：

**Linux:** gcc 4.8.1 (Complete Concurrency support)
**Windows:** Visual Studio 2012 and MingW

### C++11中创建线程

每一个C++程序都有一个默认的主线程，在C++11中我们可以使用std::thread对象创建额外的进程。每一个 std::thread对象都可以关联一个线程。

需要引入头文件

```c++
#include <thread>
```



####如何构造一个std::thread对象？

可以给std::thread对象关联一个callback，这样当线程启动的时候就会调用这个callback，callback可以是

1）函数指针
2）函数对象
3）lambda表达式

就像这样创建std::thread：

```c++
std::thread thObj(<CALLBACK>);
```

创建对象之后线程立即开始，然后执行传入的callback，新的线程和调用线程是并发的。

任何一个线程可以通过 join() 函数等待其他线程对象的结束。

我们看一个例子：主线程创建了一个线程，然后在控制台打印一些数据，之后等待新创建的线程执行完毕。

使用函数指针：

```c++
#include <iostream>
#include <thread>
 
void thread_function()
{
    for(int i = 0; i < 10000; i++);
        std::cout<<"thread function Executing"<<std::endl;
}
 
int main()  
{
    
    std::thread threadObj(thread_function);
    for(int i = 0; i < 10000; i++);
        std::cout<<"Display From MainThread"<<std::endl;
    threadObj.join();    
    std::cout<<"Exit of Main function"<<std::endl;
    return 0;
}
```

使用函数对象：

```c++
#include <iostream>
#include <thread>
class DisplayThread
{
public:
    void operator()()     
    {
        for(int i = 0; i < 10000; i++)
            std::cout<<"Display Thread Executing"<<std::endl;
    }
};
 
int main()  
{
    std::thread threadObj( (DisplayThread()) );
    for(int i = 0; i < 10000; i++)
        std::cout<<"Display From Main Thread "<<std::endl;
    std::cout<<"Waiting For Thread to complete"<<std::endl;
    threadObj.join();
    std::cout<<"Exiting from Main Thread"<<std::endl;
    return 0;
}
```

使用lambda表达式：

```c++
#include <iostream>
#include <thread>
int main()  
{
    int x = 9;
    std::thread threadObj([]{
            for(int i = 0; i < 10000; i++)
                std::cout<<"Display Thread Executing"<<std::endl;
            });
            
    for(int i = 0; i < 10000; i++)
        std::cout<<"Display From Main Thread"<<std::endl;
        
    threadObj.join();
    std::cout<<"Exiting from Main Thread"<<std::endl;
    return 0;
}
```



### 线程的不同

每一个 std::thread有一个线程ID，可以通过成员函数get_id()获取：

```c++
std::thread::get_id()
```

获取当前的线程ID可以使用：

```c++
std::this_thread::get_id()
```

如果std::thread对象没有关联任何线程，get_id()将返回一个默认的 std::thread::id object对象。std::thread::id对象可以比较和打印，看个例子：

```c++
#include <iostream>
#include <thread>
void thread_function()
{
    std::cout<<"Inside Thread :: ID  = "<<std::this_thread::get_id()<<std::endl;    
}
int main()  
{
    std::thread threadObj1(thread_function);
    std::thread threadObj2(thread_function);
 
    if(threadObj1.get_id() != threadObj2.get_id())
        std::cout<<"Both Threads have different IDs"<<std::endl;
 
        std::cout<<"From Main Thread :: ID of Thread 1 = "<<threadObj1.get_id()<<std::endl;    
    std::cout<<"From Main Thread :: ID of Thread 2 = "<<threadObj2.get_id()<<std::endl;    
 
    threadObj1.join();    
    threadObj2.join();    
    return 0;
}
```





