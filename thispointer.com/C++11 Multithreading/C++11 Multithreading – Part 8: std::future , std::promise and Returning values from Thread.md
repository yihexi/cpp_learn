原文链接：https://thispointer.com//c11-multithreading-part-8-stdfuture-stdpromise-and-returning-values-from-thread/



这篇文章主要介绍std::future和std::promise。很多时候，我们想在一个线程结束的时候返回执行结果。比如，应用开启一个线程压缩一个文件夹，在线程结束的时候返回压缩文件的名字和大小。

**原来的方案：通过共享变量。**

在主线程创建数据变量，然后将变量的指针传递给工作线程，然后等待工作线程结束。为了保证数据安全，需要一个条件变量，一个mutex和一个指针。

然而，如果我们想返回3个数据呢？如果想在不同的时间点返回而不是线程结束后呢？使用这个解决方案会让代码变得复杂。

**C++11的方案：使用std::future和std::promise**

std::future是一个类模板，保存了future value。那么，什么是future value？

std::future中保存了一个变量，这个变量会在将来被赋值，并且提供了获取这个变量值的方法。如果在这个变量被赋值之前访问这个变量，访问就会被阻塞，直到变量可用。

std::promise也是一个类模板，每一个std::promise都关联了一个std::future，并且会为std::future赋值。

std::promise和std::future是共享数据的。



下边看如何使用std::promise和std::future。
在Thread 1中创建一个std::promise：

```c++
std::promise<int> promiseObj;
```

现在std::promise对象没有关联任何数据，但是它承诺将来会关联一个int数据，一旦关联了数据，就可以通过std::future获取到这个int数据。

然后将std::promise对象传递给Thread2 。然后Thread1获取std::promise对应的std::future：

```c++
std::future<int> futureObj = promiseObj.get_future();
int val = futureObj.get();
```

在Thread2没有设置std::promise的值之前，Thread1会阻塞在futureObj.get()上。

Thread2设置std::promise

```c++
promiseObj.set_value(45);
```

然后Thread1就可以得到这个45了。



完整示例代码：

```c++
#include <iostream>
#include <thread>
#include <future>
void initiazer(std::promise<int> * promObj)
{
    std::cout<<"Inside Thread"<<std::endl;     
  	promObj->set_value(35);
}
int main()
{
    std::promise<int> promiseObj;
    std::future<int> futureObj = promiseObj.get_future();
    std::thread th(initiazer, &promiseObj);
    std::cout<<futureObj.get()<<std::endl;
    th.join();
    return 0;
}
```

如果std::promise在设置值之前被析构，调用std::future的get()会引发异常。



回到上边的问题，如果想要在不同时间返回多个值，就创建并传递多个std::promise到工作线程。然后通过多个std::future获取返回值。

> 然后必须等到所有数据都准备好了才能返回，没有select这样的机制





