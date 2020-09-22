原文链接：https://thispointer.com/is-rvalue-immutable-in-c/



这篇文章讨论C++中的右值能否被更改。



如果你不了解左值和右值的区别，请参看之前的文章：

译文链接：[Difference between lvalue and rvalue in C++](./Difference between lvalue and rvalue in C++.md)

原文链接：https://thispointer.com/difference-between-lvalue-and-rvalue-in-c/



不能获取右值的地址，右值在一个表达式结束后不会保存。但是我们能改变右值么？答案看右值的类型是什么，有不同的答案。



### 内置类型的右值不能被修改

不能修改内置类型的右值

```c++
(x+7) = 7; // Compile error - Can not Modify rvalue
```



```C++
int getData();
```

getData()的返回值是一个内存类型的右值，如果试图修改它，也会造成一个编译错误。

```c++
getData() = 9; // Compile Error - Can not modify rvalue
```





### 用户自定义的右值可以被更改

用户自定义的右值可以被更改。但是只能在同一个表达式中使用自己的成员函数更改。看例子，

创建一个用户自定义类型：

```c++
class Person {
    int mAge;
public:
    Person() {
        mAge = 10;
    }
    void incrementAge()
    {
        mAge = mAge + 1;
    }
};
```

它有一个成员函数，incrementAge()，能修改mAge成员变量。然后，我们创建一个函数，返回一个Person对象。

```c++
Person getPerson()
{
    return Person();
}
```

getPerson()是一个右值，我们不能获取其地址。

```c++
Person * personPtr = &getPerson(); // COMPILE ERROR
```

但是我们可以通过成员函数来修改它的内部状态：

```c++
getPerson().incrementAge();
```

这就是上边说的"在同一个表达式中，通过自己的成员函数来修改"。（因为右值的生命周期只在同一个表达式中）



完整代码例子：

```c++
#include <iostream>
class Person {
    int mAge;
public:
    Person() {
        mAge = 10;
    }
    void incrementAge()
    {
        mAge = mAge + 1;
    }
};
Person getPerson()
{
    return Person();
}
int main() {
//    Person * personPtr = &getPerson();
    getPerson().incrementAge();
    return 0;
}
```





















