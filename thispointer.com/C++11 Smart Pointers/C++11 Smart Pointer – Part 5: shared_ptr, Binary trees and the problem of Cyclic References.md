原文链接：https://thispointer.com/shared_ptr-binary-trees-and-the-problem-of-cyclic-references/



shared_ptr的最大好处是在没有人使用指针的时候，能自动释放内存。

但是如果不小心使用，优点可能变成缺点，我们看例子，

假设我们需要设计一个二叉树，节点中有两个指针，一个指向左子节点，一个指向右子节点。

<img src="./img/binary tree.png" style="zoom:50%;" />

```c++
#include <iostream>
#include <memory>
class Node
{
    int value;
    public:
    std::shared_ptr<Node> leftPtr;
    std::shared_ptr<Node> rightPtr;
    Node(int val) : value(val) {
         std::cout<<"Contructor"<<std::endl;
    }
    ~Node() {
         std::cout<<"Destructor"<<std::endl;
    }
};
int main()
{
    std::shared_ptr<Node> ptr = std::make_shared<Node>(4);
    ptr->leftPtr = std::make_shared<Node>(2);
    ptr->rightPtr = std::make_shared<Node>(5);
    return 0;
}
```

在上边的例子中，一切都很正常。构造函数被调用了3次，析构函数也被调用了3次。没有内存泄露。

但是如果增加一个小需求，每个node添加一个指向父节点的指针，shared_ptr就会出问题了。

代码示例：

```c++
#include <iostream>
#include <memory>
class Node
{
    int value;
    public:
    std::shared_ptr<Node> leftPtr;
    std::shared_ptr<Node> rightPtr;
    std::shared_ptr<Node> parentPtr;
    Node(int val) : value(val)     {
         std::cout<<"Contructor"<<std::endl;
    }
    ~Node()     {
         std::cout<<"Destructor"<<std::endl;
    }
};
int main()
{
    std::shared_ptr<Node> ptr = std::make_shared<Node>(4);
    ptr->leftPtr = std::make_shared<Node>(2);
    ptr->leftPtr->parentPtr = ptr;
    ptr->rightPtr = std::make_shared<Node>(5);
    ptr->rightPtr->parentPtr = ptr;
    std::cout<<"ptr reference count = "<<ptr.use_count()<<std::endl;
    std::cout<<"ptr->leftPtr reference count = "<<ptr->leftPtr.use_count()<<std::endl;
    std::cout<<"ptr->rightPtr reference count = "<<ptr->rightPtr.use_count()<<std::endl;
    return 0;
}
```

构造函数仍然被调用了3次，然而析构函数一次也没有被调用。

造成这种现象的原因是循环引用。如果两个shared_ptr互相引用了对方，当它们超出作用域的时候，都不会释放其中的内存。

因为在shared_ptr的析构函数中，首先将引用计数-1，然后检查引用计数是否为0，如果为0将会释放内存，否则说明还有其他的shared_ptr在使用其中的对象。在上边的场景中，这些shared_ptrs互相引用，所以引用计数始终不为0.



让我们重放一下这个过程：

ptr超出作用域的时候，析构函数被调用；

在ptr的析构函数中，Node(4)的引用计数-1；

检查Node(4)的引用计数，发现是2，因为leftPtr中的Node(2)的parent和rightPtr中的Node(5) 的parent在使用，所以Node(4)不会被删除；

Node(4)不被删除，那么Node(4)中使用的leftPtr和rightPtr就不会超出生命周期，所以也不会被析构；

于是，没有任何析构函数被调用；



**如何解决这个问题？**

答案是使用**weak_ptr**

weak_ptr允许共享但是不持有对象。对象来自shared_ptr。

```c++
std::shared_ptr<int> ptr = std::make_shared<int>(4);
std::weak_ptr<int> weakPtr(ptr); 
```

weak_ptr对象不能直接使用*和->运算符来操作内部对象。必须先从weak_ptr中使用lock()方法创建一个shared_ptr对象出来。

代码示例：

```c++
#include <iostream>
#include <memory>
int main()
{
    std::shared_ptr<int> ptr = std::make_shared<int>(4);
    std::weak_ptr<int> weakPtr(ptr);
    std::shared_ptr<int> ptr_2 =  weakPtr.lock();
    if(ptr_2)
        std::cout<<(*ptr_2)<<std::endl;
    std::cout<<"Reference Count :: "<<ptr_2.use_count()<<std::endl;   
    if(weakPtr.expired() == false)
        std::cout<<"Not expired yet"<<std::endl;   
    return 0;
}
```

注意：如果shared_ptr中的对象已经被释放掉，lock()方法会返回一个空的shared_ptr；



使用weak_ptr改进上边的二叉树的例子：

将循环引用的其中一方改成weak_ptr引用就可以打破循环。

```c++
#include <iostream>
#include <memory>
class Node
{
    int value;
    public:
    std::shared_ptr<Node> leftPtr;
    std::shared_ptr<Node> rightPtr;
    // Just Changed the shared_ptr to weak_ptr
    std::weak_ptr<Node> parentPtr;
    Node(int val) : value(val)     {
         std::cout<<"Contructor"<<std::endl;
    }
    ~Node()     {
         std::cout<<"Destructor"<<std::endl;
    }
};
int main()
{
    std::shared_ptr<Node> ptr = std::make_shared<Node>(4);
    ptr->leftPtr = std::make_shared<Node>(2);
    ptr->leftPtr->parentPtr = ptr;
    ptr->rightPtr = std::make_shared<Node>(5);
    ptr->rightPtr->parentPtr = ptr;
    std::cout<<"ptr reference count = "<<ptr.use_count()<<std::endl;
    std::cout<<"ptr->leftPtr reference count = "<<ptr->leftPtr.use_count()<<std::endl;
    std::cout<<"ptr->rightPtr reference count = "<<ptr->rightPtr.use_count()<<std::endl;
    std::cout<<"ptr->rightPtr->parentPtr reference count = "<<ptr->rightPtr->parentPtr.lock().use_count()<<std::endl;
    std::cout<<"ptr->leftPtr->parentPtr reference count = "<<ptr->leftPtr->parentPtr.lock().use_count()<<std::endl;
    return 0;
}
```

上边的代码解决了内存泄露的问题。









