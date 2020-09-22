原文链接：https://thispointer.com/c11-stdtuple-tutorial-examples/



这篇文章讲述什么是std::tuple以及如何使用。



### 什么是std::tuple？为什么要用它？

std::tuple可以保存固定个数的不同类型的值。使用std::tuple需要在定义的时候将元素的类型作为模板参数。

#### 创建一个std::tuple对象

下边创建一个std::tuple,其中保存一组int,double,std:string类型的值

```c++
// Creating a tuple of int, double and string
std::tuple<int, double, std::string> result(7, 9.8, "text");
```

这样3个不同类型的对象被封装到一个对象中，这样就可以从一个函数返回多个值了。否则，我们就需要创建一个结构体来实现。

使用上述代码需要加头文件

```c++
#include <tuple> // Required for std::tuple
```

#### 获取std:tuple中的值

可以使用std::get函数获取指定下标的std:tuple中的值。

这样可以获取std::tuple中的一个元素：

```c++
// Get First int value from tuple
int iVal = std::get<0>(result);
```

同样的，可以获取第二个和第三个元素:

```c++
// Get second double value from tuple
double dVal = std::get<1>(result);
// Get third string value from tuple
std::string strVal = std::get<2>(result);
```

#### 获取越界下标元素

获取std::tuple中的元素时候，如果传入的下标越界，将会得到一个编译错误（这是非常幸运的事儿）。

例如，对上边的tuple对象使用下标4来获取元素，会得到一个编译错误，因为tuple对象中只有3个元素：

```c++
// Get 4th int value from tuple
// Will cause compile error because this tuple
// has only 3 elements
int iVal2 = std::get<4>(result); // Compile error
```

#### 使用了错误的类型

将从tuple中取出的元素赋值给一个错误的类型，同样会得到一个编译错误：

```c++
// Wrong cast will force compile time error
// Get first value from tuple in a string
std::string strVal2 = std::get<0>(result); // Compile error
```

#### 使用变量下标获取tuple中的元素

会得到一个编译错误

```c++
int x = 1;
// Get second double value from tuple
// Compile error because x is not compile time contant
double dVal2 = std::get<x>(result); // Compile error
```

tuple的下标必须是一个编译时常量，所以上述代码会报错，可以使用下边的代码：

```c++
const int i = 1;
// Get second double value from tuple
double dVal3 = std::get<i>(result);
```



完整代码示例：

```c++
#include <iostream>
#include <tuple> // Required for std::tuple
#include <string>
/*
 * Returning multiple values from a function by binding them in a single
 * tuple object.
 */
std::tuple<int, double, std::string> someFunction()
{
    // Creating a tuple of int, double and string
    std::tuple<int, double, std::string> result(7, 9.8, "text");
    // Returning tuple object
    return result;
}
int main()
{
    // Get tuple object from a function
    std::tuple<int, double, std::string> result = someFunction();
    // Get values from tuple
    // Get First int value from tuple
    int iVal = std::get < 0 > (result);
    // Get second double value from tuple
    double dVal = std::get < 1 > (result);
    // Get third string value from tuple
    std::string strVal = std::get < 2 > (result);
    // Print values
    std::cout << "int value = " << iVal << std::endl;
    std::cout << "double value = " << dVal << std::endl;
    std::cout << "string value = " << strVal << std::endl;
    // Get 4th int value from tuple
    // Will cause compile error because this tuple
    // has only 3 elements
    //int iVal2 = std::get<4>(result); // Compile error
    // Wrong cast will force compile time error
    // Get first value from tuple in a string
    //std::string strVal2 = std::get<0>(result); // Compile error
    int x = 1;
    // Get second double value from tuple
    // Compile error because x is not compile time contant
    //double dVal2 = std::get<x>(result);
    const int i = 1;
    // Get second double value from tuple
    double dVal3 = std::get < i > (result);
    return 0;
}
```

输出：

```c++
int value = 7
double value = 9.8
string value = text
```





## 