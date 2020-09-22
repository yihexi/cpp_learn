原文链接：https://thispointer.com/c11-make_tuple-tutorial-example/



这篇文章讨论如何构造一个std::tuple



### 初始化一个std::tuple

可以通过构造函数来创建一个std::tuple对象：

```c++
// Creating and Initializing a tuple
std::tuple<int, double, std::string> result1 { 22, 19.28, "text" };
```

我们需要将所有值的类型作为模板参数传入，当tuple中元素个数很多时，这很痛苦。

可惜的是没有办法做类型推断，下边代码会报编译错误：

```c++
// Compile error, as no way to deduce the types of elements in tuple
auto result { 22, 19.28, "text" }; // Compile error
```

**Error**

```
error: unable to deduce ‘std::initializer_list<_Tp>’ from ‘{22, 1.9280000000000001e+1, "text"}’
auto result { 22, 19.28, "text" };
```

c++11提供了std::make_tuple函数解决了我们的痛苦。

### std::make_tuple

std::make_tuple可以通过对参数的类型推断来创建一个tuple对象。看例子

```c++
// Creating a tuple using std::make_tuple
auto result2 = std::make_tuple( 7, 9.8, "text" );
```

这里我们没有指定result2的类型。std::make_tuple做了以下事情：

接收了3个参数，然后推断出类型分别为int,double和string。然后创建一个 std::tuple<int, double, std::string>对象并使用参数初始化，之后返回。

本质上，std::make_tuple内部自动做了类型推断。

完整代码示例：

```c++
#include <iostream>
#include <tuple>
#include <string>
int main()
{
    // Creating and Initializing a tuple
    std::tuple<int, double, std::string> result1 { 22, 19.28, "text" };
    // Compile error, as no way to deduce the types of elements in tuple
    //auto result { 22, 19.28, "text" }; // Compile error
    // Creating a tuple using std::make_tuple
    auto result2 = std::make_tuple( 7, 9.8, "text" );
    // std::make_tuple automatically deduced the type and created tuple
    // Print values
    std::cout << "int value = " << std::get < 0 > (result2) << std::endl;
    std::cout << "double value = " << std::get < 1 > (result2) << std::endl;
    std::cout << "string value = " << std::get < 2 > (result2) << std::endl;
    return 0;
}
```

输出

```c++
int value = 7
double value = 9.8
string value = text
```







