原文链接：https://thispointer.com/c11-variadic-template-function-tutorial-examples/



可变参数模板是C++11引入的。可变参数模板能实现接受任意多参数以及任意类型的函数模板。

看一个例子，

假设我们想创建一个log()函数，接受不同类型的不同个数的参数，然后将这些参数打印。

```c++
log(1,4.3, "Hello");
log('a', "test", 78L, 5);
class Student;
Student obj;
log(3, obj);
etc..
```

我们需要做2件事：

1）函数接受任意类型的参数

2）函数接受任意个数的参数

1）意味着我们需要创建一个函数模板

```c++
template<typename T>
void log(T obj)
{
    std::cout<<obj;
}
```

但是上述函数只接受一个参数，要实现2)，就需要用到可变参数模板。



### 可变参数模板函数：创建函数接受任意个数任意类型的参数

这样定义一个可变参数函数模板：

```c++
template<typename T, typename ... Args>
void log(T first, Args ... args);
```

这个函数可以接受1个或者多个参数，Args就表示可变的模板参数。

声明一个可变参数函数模板容易，要实现就需要用些技巧。我们不能直接获得模板参数的个数，需要递归的使用c++的模板类型推断达到目的。看这个函数的定义：

```c++
/*
 * Variadic Template Function that accepts variable number
 * of arguments of any type.
 */
template<typename T, typename ... Args>
void log(T first, Args ... args) {
    // Print the First Element
    std::cout<<first<<" , ";
    // Forward the remaining arguments
    log(args ...);
}
```

看这个函数如何工作：

```c++
log(2, 3.4 , "aaa");
```

我们给log传入了3个参数，分别是int，double和const char *类型。由于模板类型推断机制，编译器将生成如下一个函数：

```c++
void log(int first, double b, const char * c)
{
  std::cout<<first<<" , ";
  log(b, c);
}
```

打印第一个参数，然后将剩下2个参数传入log，然后再用模板类型推断机制，生成函数：

```c++
void log(double first, const char * c)
{
  std::cout<<first<<" , ";
  log(c);
}
```

打印第一个参数，然后将剩下1个参数传入log，然后再用模板类型推断机制，生成一个参数的函数：

```c++
void log(const char * first)
{
  std::cout<<first<<" , ";
  log();
}
```

打印第一个参数。然后没有参数了，直接调用log()函数，因为模板推断不能再生成函数了，所以这个函数需要由开发人员来实现：

```c++
void log()
{
}
```

上述的可变函数模板函数的调用堆栈如下：

```c++
void log();
void log(const char * first);
void log(double first, const char * c);
void log(int first, double b, const char * c);
```



完整代码示例：

```c++
#include <iostream>
// Function that accepts no parameter
// It is to break the recursion chain of vardiac template function
void log()
{
}
/*
 * Variadic Template Function that accepts variable number
 * of arguments of any type.
 */
template<typename T, typename ... Args>
void log(T first, Args ... args) {
    // Print the First Element
    std::cout<<first<<" , ";
    // Forward the remaining arguments
    log(args ...);
}
int main() {
    log(2, 3.4, "aaa");
    return 0;
}
```

输出

```
2 , 3.4 , aaa ,
```

