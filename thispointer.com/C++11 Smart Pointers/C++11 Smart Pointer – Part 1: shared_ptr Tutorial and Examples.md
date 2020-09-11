原文链接：https://thispointer.com/learning-shared_ptr-part-1-usage-details/



这篇文章中，我们讨论C++11中的智能指针shared_ptr的用法。



###什么是std::shared_ptr<> ?

shared_ptr是C++11提供的智能指针之一，它能在没有任何引用的时候自动释放持有的裸指针。因此能帮助开发人员避免内存泄露和悬空指针问题。



###shared_ptr和共享所有权
多个shared_ptr共享一个底层裸指针的所有权，是通过引用计数的方式实现的。



#### 每一个shared_ptr对象内部有两个指针

1）一个指向持有的对象；

2）另外一个指向引用计数数据；



#### 共享所有权是如何通过引用计数实现的？

* 当一个新的shared_ptr对象持有（关联）一个裸指的时候，这个shared_ptr的构造函数增加裸指针的引用计数+1。
* 当一个shared_ptr对象超出作用域，它的析构函数将裸指针的引用计数-1。如果引用计数变为0，说明没有任何shared_ptr对象持有该（关联）这个裸指针了，将会使用deleted释放裸指针指向的内存。



### 创建一个shared_ptr对象

将一个shared_ptr对象和裸指针关联：

```c++
std::shared_ptr<int> p1(new int());
```

上边这行代码会在堆上分配两块内存：

1）一块内存用于保存int

2）一块内存用于保存引用计数，初始值为1



#### 检查一个shared_ptr对象的引用计数

可以使用如下方法获取shared_prt对象的引用计数：

```c++
p1.use_count();
```



#### std::make_shared<T>

如何把一个裸指针和一个shared_ptr关联起来？

```c++
std::shared_ptr<int> p1 = new int(); // Compile error
```

因为shared_ptr的构造函数使用了Explicit形式的参数（不支持隐式的转换数据类型），上边的代码需要使用隐式数据类型转换的构造函数。所以会发生编译错误。

创建一个shared_ptr最好的方式是使用**std::make_shared**

```c++
std::shared_ptr<int> p1 = std::make_shared<int>();
```

std::make_shared函数为保存的对象和引用计数一起分配了一块内存，new运算符只被调用了一次。

> 不太懂，稍后需要理解下。





#### 解除shared_ptr和裸指针的关联

使用reset()方法可以解除一个shared_ptr对象对裸指针的关联。

reset()函数没有参数：

```c++
p1.reset();
```

调用reset()后，引用计数会-1，如果引用计数变为0，裸指针会被delete。

还有一个带参数的reset函数：

```c++
p1.reset(new int(34));
```

p1会解除对原来裸指针的关联，并且关联新的int指针。引用计数为1。

还可以使用nullptr解除shared_ptr对象对裸指针的关联：

```c++
p1 = nullptr;
```

此后p1就是空的。



#### shared_ptr是一个伪指针

在使用*或者->操作符的时候shared_ptr表现的就像是一个指针，并且我们也可以比较一个shared_ptr和另外一个shared_ptr（所指是否为统一对象）



完整的代码示例:

```c++
#include <iostream>
#include  <memory> // We need to include this for shared_ptr
int main()
{
    // Creating a shared_ptr through make_shared
    std::shared_ptr<int> p1 = std::make_shared<int>();
    *p1 = 78;
    std::cout << "p1 = " << *p1 << std::endl;
    // Shows the reference count
    std::cout << "p1 Reference count = " << p1.use_count() << std::endl;
    // Second shared_ptr object will also point to same pointer internally
    // It will make the reference count to 2.
    std::shared_ptr<int> p2(p1);
    // Shows the reference count
    std::cout << "p2 Reference count = " << p2.use_count() << std::endl;
    std::cout << "p1 Reference count = " << p1.use_count() << std::endl;
    // Comparing smart pointers
    if (p1 == p2)
    {
        std::cout << "p1 and p2 are pointing to same pointer\n";
    }
    std::cout<<"Reset p1 "<<std::endl;
    p1.reset();
    // Reset the shared_ptr, in this case it will not point to any Pointer internally
    // hence its reference count will become 0.
    std::cout << "p1 Reference Count = " << p1.use_count() << std::endl;
    // Reset the shared_ptr, in this case it will point to a new Pointer internally
    // hence its reference count will become 1.
    p1.reset(new int(11));
    std::cout << "p1  Reference Count = " << p1.use_count() << std::endl;
    // Assigning nullptr will de-attach the associated pointer and make it to point null
    p1 = nullptr;
    std::cout << "p1  Reference Count = " << p1.use_count() << std::endl;
    if (!p1)
    {
        std::cout << "p1 is NULL" << std::endl;
    }
    return 0;
}
```

输出：

```
p1 = 78
p1 Reference count = 1
p2 Reference count = 2
p1 Reference count = 2
p1 and p2 are pointing to same pointer
Reset p1 
p1 Reference Count = 0
p1  Reference Count = 1
p1  Reference Count = 0
p1 is NULL
```







