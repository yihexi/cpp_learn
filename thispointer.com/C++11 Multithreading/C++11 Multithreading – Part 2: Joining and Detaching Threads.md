原文链接：https://thispointer.com//c11-multithreading-part-2-joining-and-detaching-threads/



这篇文章讨论std::thread的join和detach方法。



### std::thread::join()

一个线程开始后，另外一个线程可以通过调用这个线程的std::thread的join()方法来等待这个线程执行完毕。

```c++
std::thread th(funcPtr);
// Some Code
th.join();
```

看一个实际例子，

假设主线程启动了10个工作线程，然后等待它们执行完毕，之后主线程再做其他事情。

```c++
#include <iostream>
#include <thread>
#include <algorithm>
class WorkerThread
{
public:
    void operator()()     
    {
        std::cout<<"Worker Thread "<<std::this_thread::get_id()<<" is Executing"<<std::endl;
    }
};
int main()  
{
    std::vector<std::thread> threadList;
    for(int i = 0; i < 10; i++)
    {
        threadList.push_back( std::thread( WorkerThread() ) );
    }
    // Now wait for all the worker thread to finish i.e.
    // Call join() function on each of the std::thread object
    std::cout<<"wait for all the worker thread to finish"<<std::endl;
    std::for_each(threadList.begin(),threadList.end(), std::mem_fn(&std::thread::join));
    std::cout<<"Exiting from Main Thread"<<std::endl;
    return 0;
}
```



###  std::thread::detach()

Detached threads也叫daemon / Background threads。调用std::thread的std::detach() 方法可以detach一个线程。

```c++
std::thread th(funcPtr);
th.detach();
```

调用了 detach(),之后std::thread对象就不再关联执行任务的线程了。



### 调用detach()和join()的注意事项



Case 1：不要在没有关联任何线程的std::thread对象上调用join()和detach()

```c++
 std::thread threadObj( (WorkerThread()) );
 threadObj.join();
 threadObj.join(); // It will cause Program to Terminate
```

当一个thread对象的join()返回0时，这个thread对象就已经不再关联任何线程了。所以再次调用join()函数会导致程序终止。

同样，调用thread对象的detach()方法也会让thread对象不再关联任何线程，再次调用detach()方法也会导致程序终止。

```c++
std::thread threadObj( (WorkerThread()) );
threadObj.detach();
threadObj.detach(); // It will cause Program to Terminate
```

因此，在调用join()或者detach()之前应该检查下thread对象是否有关联线程。使用joinable()方法。

```c++
std::thread threadObj( (WorkerThread()) );
if(threadObj.joinable())
{
  std::cout<<"Detaching Thread "<<std::endl;
  threadObj.detach();
}
if(threadObj.joinable())    
{
  std::cout<<"Detaching Thread "<<std::endl;
  threadObj.detach();
}

std::thread threadObj2( (WorkerThread()) );
if(threadObj2.joinable())
{
  std::cout<<"Joining Thread "<<std::endl;
  threadObj2.join();
}
if(threadObj2.joinable())    
{
  std::cout<<"Joining Thread "<<std::endl;
  threadObj2.join();
}
```

Case2:不要忘记调用调用thread对象的join()或者detach()方法

如果忘记了调用thread对象的join()或者detach()方法（关联了线程的），那么当thread对象析构的时候，程序将被终止。以为在thread对象的析构函数中，检查了对象是否是join-able的，如果是则终止程序。

```c++
#include <iostream>
#include <thread>
#include <algorithm>
class WorkerThread
{
public:
    void operator()()     
    {
        std::cout<<"Worker Thread "<<std::endl;
    }
};
int main()  
{
    std::thread threadObj( (WorkerThread()) );
    // Program will terminate as we have't called either join or detach with the std::thread object.
    // Hence std::thread's object destructor will terminate the program
    return 0;
}
```

需要注意的是，在处理异常的时候，不要忘记调用thread对象的 join()或者detach()。为了避免这种情况，我们需要用到RESOURCE ACQUISITION IS INITIALIZATION (RAII) 。

```c++
#include <iostream>
#include <thread>
class ThreadRAII
{
    std::thread & m_thread;
    public:
        ThreadRAII(std::thread  & threadObj) : m_thread(threadObj)
        {
            
        }
        ~ThreadRAII()
        {
            // Check if thread is joinable then detach the thread
            if(m_thread.joinable())
            {
                m_thread.detach();
            }
        }
};
void thread_function()
{
    for(int i = 0; i < 10000; i++);
        std::cout<<"thread_function Executing"<<std::endl;
}
 
int main()  
{
    std::thread threadObj(thread_function);
    
    // If we comment this Line, then program will crash
    ThreadRAII wrapperObj(threadObj);
    return 0;
}
```





