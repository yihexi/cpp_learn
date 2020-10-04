原文链接：https://thispointer.com//c11-multithreading-part-5-using-mutex-to-fix-race-conditions/



这篇文章讨论如何使用mutex在多线程环境下保护共享数据，避免竞态条件。在多线程环境下，在读取或者修改共享数据的时候，先使用mutex锁住，之后再释放锁就可以避免竞态条件。



### std::mutex

引入C++标准库中的<mutex>头文件就可以使用std:mutex类。

std:mutex有两个重要操作：

(1) lock

(2) unlock

我们仍然使用上文中的钱包的例子。在这篇文章中，将使用mutex修复上文中发生的静态条件。

因为我们在不同的线程中操作了同一个钱包对象，所以，在addMoney() 方法中，在增加成员函数mMoney之前应该先获取锁，增加之后再释放锁。

```c++
#include<iostream>
#include<thread>
#include<vector>
#include<mutex>
class Wallet
{
    int mMoney;
    std::mutex mutex;
public:
    Wallet() :mMoney(0){}
    int getMoney()   {     return mMoney; }
    void addMoney(int money)
    {
        mutex.lock();
        for(int i = 0; i < money; ++i)
        {
            mMoney++;
        }
        mutex.unlock();
    }
};
```

然后仍然使用5个线程并发的调用addMoney()。这次当所有线程执行完毕后，mMoney准确的是5000.

```c++
int testMultithreadedWallet()
{
    Wallet walletObject;
    std::vector<std::thread> threads;
    for(int i = 0; i < 5; ++i){
        threads.push_back(std::thread(&Wallet::addMoney, &walletObject, 1000));
    }
    for(int i = 0; i < threads.size() ; i++)
    {
        threads.at(i).join();
    }
    return walletObject.getMoney();
}
int main()
{
    int val = 0;
    for(int k = 0; k < 1000; k++)
    {
        if((val = testMultithreadedWallet()) != 5000)
        {
            std::cout << "Error at count = "<<k<<"  Money in Wallet = "<<val << std::endl;
            //break;
        }
    }
    return 0;
}
```

因为mutex锁保证了在同一时刻只有一个线程更改mMoney。

但是如果我们忘记了在函数结束前释放锁，那么其他线程就会一直等下去（尤其是发生了异常的时候，处理异常容易忘记释放锁）。为了避免忘记释放锁这种情况，应该使用std::lock_guard。



### std::lock_guard

std::lock_guard是一个类模板，实现了mutex的RAII。它内部包装了一个mutex，在构造函数中获取锁，在析构函数中释放锁。

```c++
class Wallet
{
    int mMoney;
    std::mutex mutex;
public:
    Wallet() :mMoney(0){}
    int getMoney()   {     return mMoney; }
    void addMoney(int money)
    {
        std::lock_guard<std::mutex> lockGuard(mutex);
        // In constructor it locks the mutex
        for(int i = 0; i < money; ++i)
        {
            // If some exception occurs at this
            // poin then destructor of lockGuard
            // will be called due to stack unwinding.
            //
            mMoney++;
        }
        // Once function exits, then destructor
        // of lockGuard Object will be called.
        // In destructor it unlocks the mutex.
    }
 };
```









