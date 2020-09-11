原文链接：https://thispointer.com/create-shared_ptr-objects-carefully/



我们在创建shared_ptr对象的时候要非常小心。



特别是以下两种情况：

1）不要试图使用一个裸指针创建两个shared_ptr对象。因为不同的shared_ptr对象互相不知道对方的存在，分别独立的引用计数。

下边我们看下这样做会引起什么问题？

假设两个shared_ptr对象都是用一个裸指针创建的：

```c++
int * rawPtr = new int();
std::shared_ptr<int> ptr_1(rawPtr);
std::shared_ptr<int> ptr_2(rawPtr);
```

然后ptr_2超出了作用域，讲裸指针指向的内存释放了。此时ptr_1中的裸指针就是一个悬空指针。

然后当ptr_2超出作用域后，再次释放内存，就会引起Crash。

代码示例：

```c++
#include<iostream>
#include<memory>
typedef struct Sample {
Sample() {
    internalValue = 0;
    std::cout<<"Constructor"<<std::endl;
}
~Sample() {
    std::cout<<"Destructor"<<std::endl;
}
}Sample;
int main()
{
    {
    Sample * rawPtr = new Sample();
    std::shared_ptr<Sample> ptr_1(rawPtr);
        {
        std::shared_ptr<Sample> ptr_2(rawPtr);
        }
// As ptr_2 dont know that the same raw pointer is used by another shared_ptr i.e. ptr_1, therefore
// here when ptr_2 goes out of scope and it deletes the raw pointer associated with it.
// Now ptr_1 is internally containing a dangling pointer. Therefore when we it
// will go out of scope it will again try to delete the already deleted raw pointer and application
// will crash.
    }
return 0;
}
```



2) 不要使用一个指向栈上对象的指针创建shared_ptr

代码示例：

```c++
#include<iostream>
#include<memory>
int main()
{
	int x = 12;
	std::shared_ptr<int> ptr(&x);
	return 0;
}
```

shared_ptr的实现假定其中的裸指针是指向堆中的对象的。如果将一个栈上的对象指针传入，在超出作用的时候，仍然会造成重复释放指针的问题，引发Crash。



鉴于以上两个原因，建议不要通过裸指针直接创建shared_ptr对象。而是使用make_shared<>方法

```c++
std::shared_ptr<int> ptr_1 = make_shared<int>();
std::shared_ptr<int> ptr_2 (ptr_1);
```



> 译者：编码规范：先创建shared_ptr，再赋值。直接避免裸指针的定义出现。



