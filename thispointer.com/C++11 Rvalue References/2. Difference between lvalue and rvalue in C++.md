原文链接：https://thispointer.com/difference-between-lvalue-and-rvalue-in-c/



这篇文章讨论了了C++中左值和右值的区别。



在C语言中，确定一个变量是左值还是右值很简单，=号左边的就是左值，右边的就是右值。但是在C++中事情变得复杂了。C++中每一个表达式或者是左值或者是右值。下边一个个讨论。



### 什么是左值？

任何可以通过&操作符获取内存地址的变量表达式等都是左值。



看例子：

例1:

```c++
int x = 1;
```

x是一个左值，因为可以获取其内存地址。

```c++
int * ptr = &x;
```



例2：

```c++
int z = x + 1;
```

可以获取z的内存地址：

```c++
int * ptr2 = &z;
```

因此，z是左值，但是不能获取x+1的内存地址

```c++
int * ptr3 = &(x+1); // Compile Error
```

上边这行代码编译不过，因为x+1在表达式执行后的结果是临时值，所以x+1没有内存地址，就不能用&获取其地址。所以**x+1不是左值**。**不是左值的就是右值，因此x+1是右值**。

> 此段译者有更改，原文如下：
>
> Above line will not compile because we are trying to address of (x+1) and (x+1) doesn’t persist after single expression. As, we cannot take the address of (x+1). So, **(x+1) is not an lvalue**. So, what the heck is this (x+1) ?
>
> As, lvalue is something whose address is accessible using & operator. But in this case, it’s not possible to access the address of (x+2). So, (x+2) is something that is not lvalue and that is exactly what rvalue is. **(x+2)** here is **rvalue**



### 什么是右值？

不是左值的就是右值。右值不能获取其内存地址，表达式结束后也不保存其结果。



看例子

例1

```c++
int x = 1;
```

不能通过&获取1的地址

```c++
int * ptr = &(1); // Compile Error
```

1是右值。



例4:

```c++
int a = x+1;
```

不能通过&获取x+1的地址

```c++
int * ptr3 = &(x+1); // Compile Error
```

x+1是右值。



更多例子：

```c++
int a = 7; // a is lvalue & 7 is rvalue
int b = (a + 2); // b is lvalue & (a+2) is rvalue
int c = (a + b) ; // c is lvalue & (a+b) is rvalue
int * ptr = &a; // Possible to take address of lvalue
//int * ptr3 = &(a + 1);  // Compile Error. Can not take address of rvalue
```



现在，我们写一个函数，返回一个整数

```c++
int getData()
{
    int data = 0;
    return data;
}
```

函数getData()的返回值是临时的，当表达式结束后不再存在，只有将返回值赋值给一个左值变量才能得到返回结果。

函数的返回值是右值。

```c++
int * ptr = &getData(); // Compile error - Cannot take address of rvalue
```



> 感觉左值右值水很深呀，稍后再理解理解。

