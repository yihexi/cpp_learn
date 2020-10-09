原文链接：https://thispointer.com/c11-multithreading-part-9-stdasync-tutorial-example/



这篇文章讨论如何在C++11中使用std::async执行异步任务。

### 什么是std::async()

std::async是一个函数模板，有一个callback参数，它会异步的执行这个callback。

```c++
template <class Fn, class... Args>
future<typename result_of<Fn(Args...)>::type> async (launch policy, Fn&& fn, Args&&... args);
```

std::async返回了一个std::future<T>, 保存了异步执行的结果。同样，也可以给callback传递额外的参数。

可以使用3中策略启动一个std::async。

* std::launch::async 传入的函数会另外启动一个线程执行。

* std::launch::deferred 非异步执行，函数将在get()被调用的时候执行

* std::launch::async | std::launch::deferred 默认行为，根据系统负载决定是异步执行，还是在get的时候执行。我们不能控制执行方式。

​    我们可以传递任何形式的callback

* 函数指针
* 函数对象
* Lambda表达式



### 为什么需要std::async()

假设我们要从数据库中取一批数据，同时还要从文件系统加载一些数据。单线程的做法是这样的：

```c++
#include <iostream>
#include <string>
#include <chrono>
#include <thread>
using namespace std::chrono;
std::string fetchDataFromDB(std::string recvdData)
{
    // Make sure that function takes 5 seconds to complete
    std::this_thread::sleep_for(seconds(5));
    //Do stuff like creating DB Connection and fetching Data
    return "DB_" + recvdData;
}
std::string fetchDataFromFile(std::string recvdData)
{
    // Make sure that function takes 5 seconds to complete
    std::this_thread::sleep_for(seconds(5));
    //Do stuff like fetching Data File
    return "File_" + recvdData;
}
int main()
{
    // Get Start Time
    system_clock::time_point start = system_clock::now();
    //Fetch Data from DB
    std::string dbData = fetchDataFromDB("Data");
    //Fetch Data from File
    std::string fileData = fetchDataFromFile("Data");
    // Get End Time
    auto end = system_clock::now();
    auto diff = duration_cast < std::chrono::seconds > (end - start).count();
    std::cout << "Total Time Taken = " << diff << " Seconds" << std::endl;
    //Combine The Data
    std::string data = dbData + " :: " + fileData;
    //Printing the combined Data
    std::cout << "Data = " << data << std::endl;
    return 0;
}
```

输出

```
Total Time Taken = 10 Seconds
Data = DB_Data :: File_Data
```

fetchDataFromDB和fetchDataFromFile各自需要5秒钟执行，因此总时间是10秒钟。但是因为从数据库中取数据和从文件系统取数据没有依赖关系，我们希望这两个任务并行。于是我们使用上一章的方案，创建一个线程，然后传入一个promise可以解决。

还有一个更简单的方案，使用std::async

### 使用函数指针作为std::async的回调

下边让我们使用std::async来执行fetchDataFromDB：

```c++
std::future<std::string> resultFromDB = std::async(std::launch::async, fetchDataFromDB, "Data");
// Do Some Stuff 
//Fetch Data from DB
// Will block till data is available in future<std::string> object.
std::string dbData = resultFromDB.get();
```

std::async做了以下事情：

* 自动创建一个线程（或者从线程池中拿一个线程）和一个promise对象。
* 将promise对象传递给新线程，然后将future对象返回
* 当回调执行完成后，返回值会被设置到promise对象中，future就可以取到返回值。



完整示例代码：

```c++
#include <iostream>
#include <string>
#include <chrono>
#include <thread>
#include <future>
using namespace std::chrono;
std::string fetchDataFromDB(std::string recvdData)
{
    // Make sure that function takes 5 seconds to complete
    std::this_thread::sleep_for(seconds(5));
    //Do stuff like creating DB Connection and fetching Data
    return "DB_" + recvdData;
}
std::string fetchDataFromFile(std::string recvdData)
{
    // Make sure that function takes 5 seconds to complete
    std::this_thread::sleep_for(seconds(5));
    //Do stuff like fetching Data File
    return "File_" + recvdData;
}
int main()
{
    // Get Start Time
    system_clock::time_point start = system_clock::now();
    std::future<std::string> resultFromDB = std::async(std::launch::async, fetchDataFromDB, "Data");
    //Fetch Data from File
    std::string fileData = fetchDataFromFile("Data");
    //Fetch Data from DB
    // Will block till data is available in future<std::string> object.
    std::string dbData = resultFromDB.get();
    // Get End Time
    auto end = system_clock::now();
    auto diff = duration_cast < std::chrono::seconds > (end - start).count();
    std::cout << "Total Time Taken = " << diff << " Seconds" << std::endl;
    //Combine The Data
    std::string data = dbData + " :: " + fileData;
    //Printing the combined Data
    std::cout << "Data = " << data << std::endl;
    return 0;
}
```

现在，整个过程只需要5秒钟了。

输出

```
Total Time Taken = 5 Seconds
Data = DB_Data :: File_Data
```



### 使用函数对象的例子

```c++
/*
 * Function Object
 */
struct DataFetcher
{
    std::string operator()(std::string recvdData)
    {
        // Make sure that function takes 5 seconds to complete
        std::this_thread::sleep_for (seconds(5));
        //Do stuff like fetching Data File
        return "File_" + recvdData;
    }
};
//Calling std::async with function object
std::future<std::string> fileResult = std::async(DataFetcher(), "Data");
```



### 使用Lambda表达式的例子

```c++
//Calling std::async with lambda function
std::future<std::string> resultFromDB = std::async([](std::string recvdData){
                        std::this_thread::sleep_for (seconds(5));
                        //Do stuff like creating DB Connection and fetching Data
                        return "DB_" + recvdData;
                    }, "Data");
```



