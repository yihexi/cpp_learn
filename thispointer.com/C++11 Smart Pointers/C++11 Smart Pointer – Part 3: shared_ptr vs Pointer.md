原文链接：https://thispointer.com/how-shared_ptr-object-is-different-from-a-raw-pointer/



这篇文章比较了C++11中的智能指针shared_ptr和普通指针。



### 没有++，--以及[]运算符

shared_ptr只提供以下两种运算符

1) ->, *

2) ==

没有：

1) 算数运算符如：+, -,++, --

2) 下标运算符[]

下边是例子：

```c++
#include<iostream>
#include<memory>
struct Sample
{
    void dummyFunction()
    {
        std::cout << "dummyFunction" << std::endl;
    }
};
int main()
{
    std::shared_ptr<Sample> ptr = std::make_shared<Sample>();
    (*ptr).dummyFunction(); // Will Work
    ptr->dummyFunction(); // Will Work
    // ptr[0]->dummyFunction(); // This line will not compile.
    // ptr++;  // This line will not compile.
    //ptr--;  // This line will not compile.
    std::shared_ptr<Sample> ptr2(ptr);
    if (ptr == ptr2) // Will work
        std::cout << "ptr and ptr2 are equal" << std::endl;
    return 0;
}
```

输出：

```c++
dummyFunction
dummyFunction
ptr and ptr2 are equal
```



### 检查NULL

如果创建一个shared_ptr，但是没有关联裸指针，它就是空的。

注意，如果裸指针是一个悬空指针，shared_ptr是没法判断出来的。

可以使用以下3种方式判断一个shared_ptr为空：

```c++
std::shared_ptr<Sample> ptr3;
if(!ptr3)
    std::cout<<"Yes, ptr3 is empty" << std::endl;
if(ptr3 == NULL)
    std::cout<<"ptr3 is empty" << std::endl;
if(ptr3 == nullptr)
    std::cout<<"ptr3 is empty" << std::endl;
```

从shared_ptr中获取裸指针

```c++
std::shared_ptr<Sample> ptr = std::make_shared<Sample>();
Sample * rawptr = ptr.get();
```

一般来说，不应该获取shared_ptr中的裸指针。因为可能会破坏shared_ptr中的引用计数，从而造成crash。即使不对指针做任何delete，当shared_ptr中引用计数为0的时候，将会删除裸指针指向的对象，那么通过get方法的到的指针就是一个悬空指针，也是很危险的。

