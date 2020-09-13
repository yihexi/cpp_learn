原文链接：https://thispointer.com//c11-auto-tutorial-and-examples/



这篇文章讲述了如何在C++11中使用auto变量。

auto关键字是C++11引入的，可以声明一个auto变量而不必指定类型，编译器会根据初始化的实际类型推断出其真实类型。

```c++
// Storing a int inside a auto variable
auto var_1 = 5;
// Storing a character inside a auto variable
auto var_2 = 'C';
std::cout<<var_1<<std::endl;
std::cout<<var_2<<std::endl;
```

var_1的类型是int，var_2的类型是char。

可以在auto中保存任何类型，比如函数，迭代器，lambda表达式；

```c++
// Storing Lambda function inside a auto variable
auto fun_sum = [](int a , int b){
                    return a+b;
                };
std::cout<<fun_sum(4,5)<<std::endl;
```

auto的主要作用是简化很长的变量类型。比如我们有一个std:map类型，其中的键值都是std:string类型：

```c++
std::map<std::string, std::string> mapOfStrs;
// Insert data in Map
mapOfStrs.insert(std::pair<std::string, std::string>("first", "1") );
mapOfStrs.insert(std::pair<std::string, std::string>("sec", "2") );
mapOfStrs.insert(std::pair<std::string, std::string>("thirs", "3") );
```

对这个map的迭代写成这样：

```c++
// Iterate over the map and display all data;
// Create an iterator
std::map<std::string, std::string>::iterator it = mapOfStrs.begin();
while(it != mapOfStrs.end())
{
    std::cout<<it->first<<"::"<<it->second<<std::endl;
    it++;
}
```

使用auto，可以简化上述代码：

```c++
// Iterate using auto
auto itr = mapOfStrs.begin();
while(itr != mapOfStrs.end())
{
    std::cout<<itr->first<<"::"<<itr->second<<std::endl;
    itr++;
}
```



### 使用auto的注意事项

1）一个auto变量初始化以后，其类型不能再变化。

```c++
auto x = 1;
// Cannot change the type of already initialized auto variable
// Error will occur at compile time
// x = "dummy";
```

2）auto变量必须初始化

```c++
// Can not declare auto variable without initialization
// because its type is based on initiazing value.
//auto a;
```



### 函数返回一个auto变量

返回auto变量的函数使用如下定义：

```c++
auto sum(int x, int y) -> int
{
    return x + y;
}
```

 使用函数：

```c++
auto value = sum(3, 5);
```

> 使用auto value = sum(3, 5);需要在函数定义的时候就指出，编译器不能自动推断出函数的返回值类型。感觉应该是编译器兼容问题所致。历史包袱太重。

完整代码示例：

```c++
#include <iostream>
#include <typeinfo>
#include <map>
#include <string>
#include <iterator>
auto sum(int x, int y) -> int
{
    return x + y;
}
int main()
{
    // Storing a int inside a auto variable
    auto var_1 = 5;
    // Storing a character inside a auto variable
    auto var_2 = 'C';
    std::cout << var_1 << std::endl;
    std::cout << var_2 << std::endl;
    // Storing Lambda function inside a auto variable
    auto fun_sum = [](int a , int b)
    {
        return a+b;
    };
    std::cout << fun_sum(4, 5) << std::endl;
    std::map<std::string, std::string> mapOfStrs;
    // Insert data in Map
    mapOfStrs.insert(std::pair<std::string, std::string>("first", "1"));
    mapOfStrs.insert(std::pair<std::string, std::string>("sec", "2"));
    mapOfStrs.insert(std::pair<std::string, std::string>("thirs", "3"));
    // Iterate over the map and display all data;
    // Create an iterator
    std::map<std::string, std::string>::iterator it = mapOfStrs.begin();
    while (it != mapOfStrs.end())
    {
        std::cout << it->first << "::" << it->second << std::endl;
        it++;
    }
    // Iterate using auto
    auto itr = mapOfStrs.begin();
    while (itr != mapOfStrs.end())
    {
        std::cout << itr->first << "::" << itr->second << std::endl;
        itr++;
    }
    auto x = 1;
    // Cannot change the type of already initialized auto variable
    // Error will occur at compile time
    // x = "dummy";
    // Can not declare auto variable without initialization
    // because its type is based on initiazing value.
    //auto a;
    auto value = sum(3, 5);
    std::cout << value << std::endl;
    return 0;
}
```

