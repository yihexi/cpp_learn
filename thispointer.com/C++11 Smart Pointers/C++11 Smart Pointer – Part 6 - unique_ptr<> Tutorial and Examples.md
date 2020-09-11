原文链接：https://thispointer.com/c11-unique_ptr-tutorial-and-examples/



这篇文章中，我们讨论C++11中的智能指针std::unique<>的用法。



###什么是std::unique_str?

std::unique<>是C++11为了避免内存泄露问题，实现的智能指针之一。一个unique_ptr对象包装了一个裸指针，并负责这个裸指针的生命周期。当unique_ptr对象析构的时候，它的析构函数会释放其中的裸指针的内存。

unique_ptr重载了->和*操作符，因此可以像使用裸指针一样使用unique_ptr。

下边是一个使用unique_ptr的例子：

```c++
#include <iostream>
#include <memory>

struct Task {
    int mId;

    Task(int id ) : mId(id) {
        std::cout<<"Task::Constructor"<<std::endl;
    }

    ~Task() {
        std::cout<<"Task::Destructor"<<std::endl;
    }
};


int main(){
    // Create a unique_ptr object through raw pointer
    std::unique_ptr<Task> taskPtr(new Task(23));

    //Access the element through unique_ptr
    int id = taskPtr->mId;

    std::cout<<id<<std::endl;
    return 0;
}
```

输出

```
Task::Constructor
23
Task::Destructor
```

**taskPtr**是一个**unique_ptr<Task>**类型的对象，接收一个**Task**类型的裸指针作为构造函数。当函数退出的时候，**taskPtr**超出了生命周期，析构函数被调用，并释放了裸指针指向的**Task**对象。

因此，即使函数非正常退出（包括抛出异常），taskPtr的析构函数都会被调用，裸指针指向的**Task**始终都会被释放，从而避免了内存泄露。



### unique pointer的独占性 

一个unique_ptr对象始终独占其中的裸指针。不能复制一个unique_ptr对象，只能移动裸指针的所有权。因为独占性，unique_ptr对象总是可以安全的删除其中的裸指针，不用担心引起崩溃，也不用使用引用计数。



#### 创建一个空的unique_ptr对象

一个创建空**unique_ptr<int>**对象的例子：

```c++
// Empty unique_ptr object
std::unique_ptr<int> ptr1;
```

ptr1中没有持有（关联）任何裸指针，所以是空的。



#### 检查一个unique_ptr对象是否为空

有两种方法检查一个unique_ptr对象是否为空：

方法一：

```c++
// Check if unique pointer object is empty
if(!ptr1)
    std::cout<<"ptr1 is empty"<<std::endl;
```

方法二：

```c++
// Check if unique pointer object is empty
if(ptr1 == nullptr)
    std::cout<<"ptr1 is empty"<<std::endl;
```



#### 使用裸指针创建一个unique_ptr对象

将一个对应类型的裸指针传入unique_ptr对象的构造函数即可：

 ```c++
// Create a unique_ptr object through raw pointer
std::unique_ptr<Task> taskPtr(new Task(23));
 ```

注意，我们不能通过**=**的方式创建一个unique_ptr对象，会引发编译错误。

```c++
// std::unique_ptr<Task> taskPtr2 = new Task(); // Compile Error
```



#### 重置一个unique_ptr对象

通过调用unique_ptr对象的reset()函数可以重置一个unique_ptr对象。重置对象会将内部的裸指针删除，然后unique_ptr对象变成空的。

```c++
// Reseting the unique_ptr will delete the associated
// raw pointer and make unique_ptr object empty
taskPtr.reset();
```



#### unique_ptr对象是不能复制的

unique_ptr对象是不能复制的，只能移动所有权。因此，不能创建一个unique_ptr对象的副本，无论是使用复制构造函数还是使用赋值运算符=。

```c++
// Create a unique_ptr object through raw pointer
std::unique_ptr<Task> taskPtr2(new Task(55));

// Compile Error : unique_ptr object is Not copyable
std::unique_ptr<Task> taskPtr3 = taskPtr2; // Compile error

// Compile Error : unique_ptr object is Not copyable
taskPtr = taskPtr2; //compile error
```

unique_ptr<> 类型禁用了复制构造函数和赋值运算符。



####移动unique_ptr对象的所有权 

不能复制一个unique_ptr对象，但是可以移动它对裸指针的所有权。意思就是一个unique_ptr对象可以把持有的裸指针的所有权交给另外一个unique_ptr对象。我们可以通过例子来理解：

创建一个unique_ptr对象：

```c++
// Create a unique_ptr object through raw pointer
std::unique_ptr<Task> taskPtr2(new Task(55));
```

现在，taskPtr2不是空的。

然后把taskPtr2对裸指针的所有权移动给另外一个unique_ptr对象。

```c++
{
    // Transfer the ownership
    std::unique_ptr<Task> taskPtr4 = std::move(taskPtr2);
    if(taskPtr2 == nullptr)
        std::cout<<"taskPtr2 is  empty"<<std::endl;
    // ownership of taskPtr2 is transfered to taskPtr4
    if(taskPtr4 != nullptr)
        std::cout<<"taskPtr4 is not empty"<<std::endl;
    std::cout<<taskPtr4->mId<<std::endl;
    //taskPtr4 goes out of scope and deletes the associated raw pointer
}
```

std::move() 会将taskPtr2变成一个右值引用，因此unique_ptr的移动构造函数被调用，并且将裸指针的所有权转移到taskPtr4中。

此后，taskPtr2为空。

> 不太理解变成右值引用这句，以后回头看看
>
> 原文：
>
> std::move() will convert the taskPtr2 to a RValue Reference. So that move constructor of unique_ptr is invoked and associated raw pointer can be transferred to taskPtr4.



#### 解除对裸指针的持有

通过调用unique_ptr对象的release()方法可以解除对裸指针的持有，方法返回裸指针本身。

```c++
// Create a unique_ptr object through raw pointer
std::unique_ptr<Task> taskPtr5(new Task(55));
if(taskPtr5 != nullptr)
    std::cout<<"taskPtr5 is not empty"<<std::endl;
// Release the ownership of object from raw pointer
Task * ptr = taskPtr5.release();
if(taskPtr5 == nullptr)
    std::cout<<"taskPtr5 is empty"<<std::endl;
```



以下是完整例子代码

```c++
#include <iostream>
#include <memory>
struct Task
{
    int mId;
    Task(int id ) :mId(id)
    {
        std::cout<<"Task::Constructor"<<std::endl;
    }
    ~Task()
    {
        std::cout<<"Task::Destructor"<<std::endl;
    }
};
int main()
{
    // Empty unique_ptr object
    std::unique_ptr<int> ptr1;
    // Check if unique pointer object is empty
    if(!ptr1)
        std::cout<<"ptr1 is empty"<<std::endl;
    // Check if unique pointer object is empty
    if(ptr1 == nullptr)
        std::cout<<"ptr1 is empty"<<std::endl;
    // can not create unique_ptr object by initializing through assignment
    // std::unique_ptr<Task> taskPtr2 = new Task(); // Compile Error
    // Create a unique_ptr object through raw pointer
    std::unique_ptr<Task> taskPtr(new Task(23));
    // Check if taskPtr is empty or it has an associated raw pointer
    if(taskPtr != nullptr)
        std::cout<<"taskPtr is  not empty"<<std::endl;
    //Access the element through unique_ptr
    std::cout<<taskPtr->mId<<std::endl;
    std::cout<<"Reset the taskPtr"<<std::endl;
    // Reseting the unique_ptr will delete the associated
    // raw pointer and make unique_ptr object empty
    taskPtr.reset();
    // Check if taskPtr is empty or it has an associated raw pointer
    if(taskPtr == nullptr)
        std::cout<<"taskPtr is  empty"<<std::endl;
    // Create a unique_ptr object through raw pointer
    std::unique_ptr<Task> taskPtr2(new Task(55));
    if(taskPtr2 != nullptr)
        std::cout<<"taskPtr2 is  not empty"<<std::endl;
    // unique_ptr object is Not copyable
    //taskPtr = taskPtr2; //compile error
    // unique_ptr object is Not copyable
    //std::unique_ptr<Task> taskPtr3 = taskPtr2;
    {
        // Transfer the ownership
        std::unique_ptr<Task> taskPtr4 = std::move(taskPtr2);
        if(taskPtr2 == nullptr)
            std::cout<<"taskPtr2 is  empty"<<std::endl;
        // ownership of taskPtr2 is transfered to taskPtr4
        if(taskPtr4 != nullptr)
            std::cout<<"taskPtr4 is not empty"<<std::endl;
        std::cout<<taskPtr4->mId<<std::endl;
        //taskPtr4 goes out of scope and deletes the assocaited raw pointer
    }
    // Create a unique_ptr object through raw pointer
    std::unique_ptr<Task> taskPtr5(new Task(55));
    if(taskPtr5 != nullptr)
        std::cout<<"taskPtr5 is not empty"<<std::endl;
    // Release the ownership of object from raw pointer
    Task * ptr = taskPtr5.release();
    if(taskPtr5 == nullptr)
        std::cout<<"taskPtr5 is empty"<<std::endl;
    std::cout<<ptr->mId<<std::endl;
    delete ptr;
    return 0;
}
```

输出:

```c++
ptr1 is empty
ptr1 is empty
Task::Constructor
taskPtr is  not empty
23
Reset the taskPtr
Task::Destructor
taskPtr is  empty
Task::Constructor
taskPtr2 is  not empty
taskPtr2 is  empty
taskPtr4 is not empty
55
Task::Destructor
Task::Constructor
taskPtr5 is not empty
taskPtr5 is empty
55
Task::Destructor
```

