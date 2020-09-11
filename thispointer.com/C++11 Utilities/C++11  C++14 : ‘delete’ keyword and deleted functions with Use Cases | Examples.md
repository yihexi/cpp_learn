原文链接：https://thispointer.com/c11-c14-delete-keyword-and-deleted-functions-with-use-cases-examples/



这篇文章讨论delete关键字的作用。



### 使用detete关键字删除函数

delete是C++11引入的新特性。可以指定一个函数为delete，从而禁止调用这个函数。

```c++
void someFunction() = delete ;
```

主要用于以下场景：

* 删除编译器自动生成的拷贝构造函数，赋值运算符，移动构造函数，移动赋值运算符以及默认构造函数。
* 删除成员函数
* 删除new操作符，禁止在堆上创建对象。
* 禁止指定类型的模板实例化

下边一个个看



### 删除拷贝构造函数和复制运算符

```c++
class User
{
    int id;
    std::string name;
public:
    User(int userId, std::string userName) : id(userId), name(userName)
    {}
    // Copy Constructor is deleted
    User(const User & obj) = delete;
    // Assignment operator is deleted
    User & operator = (const User & obj) = delete;
    void display()
    {
        std::cout<<id << " ::: "<<name<<std::endl;
    }
};
```



在User类中，拷贝构造函数和复制运算符被delete掉了，任何这两个函数的调用都会报一个编译错误。

```c++
User userObj(3, "John");
User obj = userObj;
```

上边的代码会产生如下编译错误：

```
delete_example.cpp: In function ‘int main()’:
delete_example.cpp:136:13: error: use of deleted function ‘User::User(const User&)’
  User obj = userObj;
             ^
delete_example.cpp:111:2: note: declared here
  User(const User & obj) = delete;
```

除了编译器生成的函数，也可以将delete用在其他类型的函数中。



### 删除成员函数，避免无意义的数据类型转换

有时候，因为数据类型自动转换的特性，我们可以使用错误的参数类型调用函数。例如：

```c++
User(int userId, std::string userName) : id(userId), name(userName)
{}
```

尽管userId是int类型，我们还是可以传入double或者char类型

```c++
User obj4(5.5, "Riti");
User obj5('a', "Riti");
```

这些代码是合法的但是是无意义，可用通过delete禁止上述代码：

```c++
// Deleting a constructor that accepts a double as ID to prevent narrowing conversion
User(double userId, std::string userName) = delete ;
// Deleting a constructor that accepts a double as ID to prevent invalid type conversion
User(char userId, std::string userName) = delete ;
```

这样，如果再使用double或者char调用，会发生如下编译错误：

```c++
error: use of deleted function ‘User::User(double, std::__cxx11::string)’
  User obj4(5.5, "Riti");
                      ^
note: declared here
  User(double userId, std::string userName) = delete ;
error: use of deleted function ‘User::User(char, std::__cxx11::string)’
  User obj5('a', "Riti");
                       ^
note: declared here
  User(char userId, std::string userName) = delete ;
```



### 删除new操作符，强制对象创建在堆上

删除类的new操作符：

```c++
class User
{
    int id;
    std::string name;
public:
    User(int userId, std::string userName) : id(userId), name(userName)
    {}
    // Delete the new function to prevent object creation on heap
    void * operator new (size_t) = delete;
};
```

现在，如果有人试图使用new创建对象，会得到如下编译错误：

```c++
User * ptr = new User(1, "Riti");
```

Compile Error:

```
error: use of deleted function ‘static void* User::operator new(size_t)’
  User * ptr = new User(1, "Riti");
                                 ^
note: declared here
  void * operator new (size_t) = delete;
```



完整代码示例：

```c++
#include <iostream>
#include <string>
class User
{
    int id;
    std::string name;
public:
    User(int userId, std::string userName) : id(userId), name(userName)
    {}
    // Copy Constructor is deleted
    User(const User & obj) = delete;
    // Assignment operator is deleted
    User & operator = (const User & obj) = delete;
    void display()
    {
        std::cout<<id << " ::: "<<name<<std::endl;
    }
    // Deleting a constructor that accepts a double as ID to prevent narrowing conversion
    User(double userId, std::string userName) = delete ;
    // Deleting a constructor that accepts a double as ID to prevent invalid type conversion
    User(char userId, std::string userName) = delete ;
    // Delete the new function to prevent object creation on heap
    void * operator new (size_t) = delete;
};
int main()
{
    User userObj(3, "John");
    // Can not copy User object as copy constructor is deleted
    //User obj = userObj;
    /*
     * Creating User objects with double or char as ID will cause compile time error
     *
    User obj4(5.5, "Riti");
    obj4.display();
    User obj5('a', "Riti");
    obj5.display();
     */
    // Can not create object on heap as new operater is deleted
    //User * ptr = new User(1, "Riti");
    return 0;
}
```



### 删除指定类型的模板实例化

我们可以使用delete来禁止指定类型的模板实例化，类或者函数都行。看例子：

假设我们有如下一个复数模板

```c++
template <typename T>
class ComplexNumber
{
    T x;
    T y;
public:
    ComplexNumber(T a, T b) : x(a) , y(b)
    {}
    void display()
    {
        std::cout<<x << " + i"<<y<<std::endl;
    }
};
```

可以创建不同的类：

```c++
ComplexNumber<int> obj1(1,2);
ComplexNumber<double> obj2(1.0,2.0);
ComplexNumber<char> obj3('1' , '2');
```

下边使用delete关键字来禁止ComplexNumber模板对double和char的实例化：

```c++
ComplexNumber(char a, char b) = delete;
ComplexNumber(double a, double b) = delete;
```

现在，再将double或者char作为模板参数，就会得到编译错误：

完整的模板代码：

```c++
template <typename T>
class ComplexNumber
{
    T x;
    T y;
public:
    ComplexNumber(T a, T b) : x(a) , y(b)
    {}
    void display()
    {
        std::cout<<x << " + i"<<y<<std::endl;
    }
        // Deleted template specialisation 
    ComplexNumber(char a, char b) = delete;
        // Deleted template specialisation  
    ComplexNumber(double a, double b) = delete;
};
```



### delete和private的不同



* private的成员函数可以被其他成员函数调用，delete则禁止一切调用

* 被delete的函数在查找的时候不可见，如果编译器发现一个函数是deleted的，将停止查找其他匹配的函数，因此能避免无意义的类型转换。

    > 原文：
    >
    > - Private member functions can be called from other member functions, whereas, deleted functions can not be called even from other member functions.
    > - Deleted functions exists in name lookup , if compiler finds a function is deleted then it will not lookup for other matching functions based on type lookups, hence prevents unneccesary data loss and bugs.

