原文链接：https://thispointer.com/c11-move-contsructor-rvalue-references/



这篇文章讨论如何在C++11的移动语法中使用右值引用



### 临时对象的问题

引入移动的概念的目的是减少内存中的临时对象。每次从函数返回一个对象的时候，都会生成一个临时对象，然后被拷贝给一个变量。这样就创建了两个对象，然而我们只需要一个。看例子：

假设我们有一个Container类型，其中有一个int指针类型的成员变量

```c++
class Container {
    int * m_Data;
public:
    Container() {
        //Allocate an array of 20 int on heap
        m_Data = new int[20];
        std::cout << "Constructor: Allocation 20 int" << std::endl;
    }
    ~Container() {
        if (m_Data) {
            delete[] m_Data;
            m_Data = NULL;
        }
    }
    Container(const Container & obj) {
        //Allocate an array of 20 int on heap
        m_Data = new int[20];
        //Copy the data from passed object
        for (int i = 0; i < 20; i++)
            m_Data[i] = obj.m_Data[i];
        std::cout << "Copy Constructor: Allocation 20 int" << std::endl;
    }
};
```

当创建一个Container对象的时候，默认的构造函数会为它的成员变量m_Data分配20个int大小的内存。

同样的，Container的拷贝构造函数也会分配20个int大小的内存，然后将传入对象的m_Data指向的内存拷贝到自己的m_Data指向的内存。

通过一个工厂类来创建Container对象：

```c++
// Create am object of Container and return
Container getContainer() 
{
    Container obj;
    return obj;
}
```

在main函数中创建一个Container的vector，通过getContainer()函数创建Container对象，然后放入vector中。

```c++
int main() {
    // Create a vector of Container Type
    std::vector<Container> vecOfContainers;
    //Add object returned by function into the vector
    vecOfContainers.push_back(getContainer());
    return 0;
}
```

可以看到，vector中只有一个Container对象，但是我们实际上创建了2个：getContainer() 创建一个临时对象，然后复制了一份创建一个新对象传给vector的push_back函数，之后临时对象析构。

	* 第一次调用了Container的构造函数，在getContainer()内
	* 第二次调用了Container的拷贝构造函数，在作为实参传给push_back的时候

每一次都分配了20个int大小的内存，造成了浪费。



如何结局这个问题？能不能不创建临时对象呢？答案是使用C++的move语法。



### 通过右值引用和移动构造函数解决临时对象问题

 getContainer() 函数是一个右值，所以可以通过一个右值引用来引用它。同样，使用右值引用也可以重载函数。那么我们就用右值引用重载Container的构造函数，这就叫做移动构造函数。



#### 移动构造函数

移动构造函数接收一个右值引用作为参数，重载了拷贝构造函数，因为拷贝构造函数接收一个常量左值引用作为参数。在移动构造函数中，只需要将成员变量直接复制给新的对象对应的成员变量，而不需要重新分配内存。

看一下Container的移动构造函数：

```c++
Container(Container && obj)
{
    // Just copy the pointer
    m_Data = obj.m_Data;
    // Set the passed object's member to NULL
    obj.m_Data = NULL;
    std::cout<<"Move Constructor"<<std::endl;
}
```

在移动构造函数中，仅仅做了指针赋值，新对象的m_Data和旧对象的m_Data指向同一块内存。然后将旧对象的m_Data设置为NULL。这样，我们没有分配任何内存，只是将旧对象的内存移交给了新的对象。

重复上边的过程，创建一个Container的vector，然后将从getContainer() 返回的对象加入到vector中。此时 getContainer()是一个右值，所以匹配到了移动构造函数而不是拷贝构造函数，这样就不会额外分配内存了。



同样的，我们也可以有移动赋值运算符。

完整代码示例：

```c++
#include <iostream>
#include <vector>
class Container {
    int * m_Data;
public:
    Container() {
        //Allocate an array of 20 int on heap
        m_Data = new int[20];
        std::cout << "Constructor: Allocation 20 int" << std::endl;
    }
    ~Container() {
        if (m_Data) {
            delete[] m_Data;
            m_Data = NULL;
        }
    }
    //Copy Constructor
    Container(const Container & obj) {
        //Allocate an array of 20 int on heap
        m_Data = new int[20];
        //Copy the data from passed object
        for (int i = 0; i < 20; i++)
            m_Data[i] = obj.m_Data[i];
        std::cout << "Copy Constructor: Allocation 20 int" << std::endl;
    }
    //Assignment Operator
    Container & operator=(const Container & obj) {
        if(this != &obj)
        {
            //Allocate an array of 20 int on heap
            m_Data = new int[20];
            //Copy the data from passed object
            for (int i = 0; i < 20; i++)
                m_Data[i] = obj.m_Data[i];
            std::cout << "Assigment Operator: Allocation 20 int" << std::endl;
        }
    }
    // Move Constructor
    Container(Container && obj)
    {
        // Just copy the pointer
        m_Data = obj.m_Data;
        // Set the passed object's member to NULL
        obj.m_Data = NULL;
        std::cout<<"Move Constructor"<<std::endl;
    }
    // Move Assignment Operator
    Container& operator=(Container && obj)
    {
        if(this != &obj)
        {
            // Just copy the pointer
            m_Data = obj.m_Data;
            // Set the passed object's member to NULL
            obj.m_Data = NULL;
            std::cout<<"Move Assignment Operator"<<std::endl;
        }
    }
};
// Create am object of Container and return
Container getContainer()
{
    Container obj;
    return obj;
}
int main() {
    // Create a vector of Container Type
    std::vector<Container> vecOfContainers;
    //Add object returned by function into the vector
    vecOfContainers.push_back(getContainer());
    Container obj;
    obj = getContainer();
    return 0;
}
```

输出

```
Constructor: Allocation 20 int
Move Constructor
Constructor: Allocation 20 int
Constructor: Allocation 20 int
Move Assignment Operator
```





