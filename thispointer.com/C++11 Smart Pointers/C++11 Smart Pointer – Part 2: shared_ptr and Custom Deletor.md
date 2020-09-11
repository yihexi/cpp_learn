原文链接：https://thispointer.com/shared_ptr-and-custom-deletor/



这篇文章讨论如何在std::shared_ptr中使用自对应的deleter。



当一个shared_ptr对象超出作用域以后，它的析构函数会将引用计数减1，如果引用计数变为0，将会delete关联的裸指针。

默认的，就是调用delete删除指针

```c++
delete Pointer;
```

但是，有时候我们不想用默认的delete函数删除指针，比如：

### 一个shared_ptr对象关联的裸指针是指向数组的而不是指向单一对象的

```c++
std::shared_ptr<int> p3(new int[12]);
```

shared_ptr在析构的时候，仍然会调用delete方法。然而，正确的释放指针方式是：

```c++
delete []
```



#### 使用自定义的deleter

可以在shared_ptr对象的构造函数中传入一个回调函数，然后在shared_ptr对象析构的时候， 如果需要释放对象，调用这个回调函数来回收内存。



#### 使用函数指针来定义deleter

```c++
// function that calls the delete [] on received pointer
void deleter(Sample * x)
{
    std::cout << "DELETER FUNCTION CALLED\n";
    delete[] x;
}
```

在shared_ptr对象构造函数中传入这个函数指针

```c++
// Creating a shared+ptr with custom deleter
std::shared_ptr<Sample> p3(new Sample[12], deleter);
```

整个例子代码如下：

```c++
#include <iostream>
#include <memory>
struct Sample
{
    Sample()
    {
        std::cout << "CONSTRUCTOR\n";
    }
    ~Sample()
    {
        std::cout << "DESTRUCTOR\n";
    }
};
// function that calls the delete [] on received pointer
void deleter(Sample * x)
{
    std::cout << "DELETER FUNCTION CALLED\n";
    delete[] x;
}
int main()
{
    // Creating a shared+ptr with custom deleter
    std::shared_ptr<Sample> p3(new Sample[12], deleter);
    return 0;
}
```

输出

```c++
CONSTRUCTOR
CONSTRUCTOR
CONSTRUCTOR
CONSTRUCTOR
CONSTRUCTOR
CONSTRUCTOR
CONSTRUCTOR
CONSTRUCTOR
CONSTRUCTOR
CONSTRUCTOR
CONSTRUCTOR
CONSTRUCTOR
DELETER FUNCTION CALLED
DESTRUCTOR
DESTRUCTOR
DESTRUCTOR
DESTRUCTOR
DESTRUCTOR
DESTRUCTOR
DESTRUCTOR
DESTRUCTOR
DESTRUCTOR
DESTRUCTOR
DESTRUCTOR
DESTRUCTOR
```



#### 使用lambda表达式或者函数对象定义deleter

shared_ptr构造函数中的deleter还可以传入lambda表达式或者函数对象。

```c++
class Deleter
{
    public:
    void operator() (Sample * x) {
        std::cout<<"DELETER FUNCTION CALLED\n";
        delete[] x;
    }
};
// Function Object as deleter
std::shared_ptr<Sample> p3(new Sample[12], Deleter());
// Lambda function as deleter
std::shared_ptr<Sample> p4(new Sample[12], [](Sample * x){
    std::cout<<"DELETER FUNCTION CALLED\n";
        delete[] x;
});
```



### 还有些情况，资源释放的方式不是delete

下边是一个内存池使用shared_ptr的例子：

```c++
#include <iostream>
#include <memory>
struct Sample
{
};
// Memory Pool Dummy Kind of Implementation
template<typename T>
class MemoryPool
{
public:
    T * AquireMemory()
    {
        std::cout << "AQUIRING MEMORY\n";
        return (new T());
    }
    void ReleaseMemory(T * ptr)
    {
        std::cout << "RELEASE MEMORY\n";
        delete ptr;
    }
};
int main()
{
    std::shared_ptr<MemoryPool<Sample> > memoryPoolPtr = std::make_shared<
            MemoryPool<Sample> >();
    std::shared_ptr<Sample> p3(memoryPoolPtr->AquireMemory(),
            std::bind(&MemoryPool<Sample>::ReleaseMemory, memoryPoolPtr,
                    std::placeholders::_1));
    return 0;
}
```

输出

```c++
AQUIRING MEMORY
RELEASE MEMORY
```



> std::bind函数是啥意思目前不清楚，稍后补充

