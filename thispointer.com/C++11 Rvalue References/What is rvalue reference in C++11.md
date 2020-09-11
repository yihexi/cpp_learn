原文链接：https://thispointer.com/what-is-a-rvalue-reference/



这篇文章讲述什么是右值引用，以及右值引用和左值引用的区别。



### 左值引用

C++ 11以前，我们只有引用的概念。一个变量的引用是其别名，引用只能指向一个存在的变量。

```c++
int x = 7;
int & refX = x; // refX is a reference
```

在C++11中，这就是左值引用，也只能引用一个左值。

```c++
int & lvalueRef = x; // lvalueRef is a lvalue reference
```

左值引用不能指向一个右值。

```c++
int & lvalueRef2 = (x+1); // COMPILE Error - lvalue Reference Can't point to rvalue
```



### 右值引用

右值引用是C++11引入的，可以引用一个右值。

**声明一个右值引用**，需要使用&&运算符。

```c++
int && rvalueRef = (x+1); // rvalueRef is rvalue reference
```

这样就创建了一个指向(x+1)的引用。



另外一个例子：

```c++
int getData()
{
    return 9;
}
```

 getData()是一个右值，如果使用左值引用会报错：

```c++
int & lvalueRef3 = getData(); // Compile error - lvalue Reference Can't point to rvalue
```

可以使用一个常量左值引用来引用getData()的临时返回值。

```c++
const int & lvalueRef3 = getData(); // OK but its const
```

 但是我们可能不想要一个常量。右值引用就可以做到这一点：

```c++
int && rvalueRef2 = getData();
```



那么右值引用有什么用途呢？下一篇介绍。