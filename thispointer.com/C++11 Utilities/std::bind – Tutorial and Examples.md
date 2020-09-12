原文链接：https://thispointer.com//stdbind-tutorial-and-usage-details/



这篇文章讨论如何使用std::bind以及何时使用。



### std::bind

**std::bind**是一个标准库中的函数对象，作为函数适配器，接受一个函数作为输入，返回一个另外一个函数。返回的函数功能和原理函数相同， 只是将其中一个或者多个参数绑定。

假设有一个这样的函数：

```c++
int add(int first, int second)
{
    return first + second;
}
```

std::bind接收这个函数作为第一个参数，函数的两个参数紧随其后。

```c++
auto add_func = std::bind(&add, _1, _2);
```

此时**add_func**是一个和add等价的函数对象。

当调用函数的时候：

 ```c++
add_func(4,5);
 ```

add_func的内部会调用add函数，并将第一个参数4传给add的第一个参数，第二个参数5传给add的第二个参数。因此，add_func和add是等价的。

在某些场景下，我们想把第一个参数固定成12，然后让用户传入第二个参数，希望是这样：

```c++
int x = new_add_func(5);
// Will return 17
```

这种需求就可以用bind实现：

```c++
auto new_add_func = std::bind(&add, 12, _1);
```

这样当调用new_add_func的时候，内部会调用add函数，并将12作为第一个参数，5作为第二个参数。

还可以改变参数的顺序：_1, _2分别表示add的第一个参数第二个参数。

```c++
auto mod_add_func = std::bind(&add, _2, _1);
```

将_2, _1顺序倒过来，当调用mod_add_func(12,15)的时候，相当于调用了add(15,12)。



### std::bind在STL算法中的应用

std::bind是一个算法适配器，在STL算法中非常有用。

比如，有一个数字列表，需要检查其中是5的倍数的数字个数。为了实现这个需求，我们写了一个函数：

```c++
bool divisible(int num , int den)
{
    if(num % den == 0)
        return true;
    return false;
}
```

最基本的解决方案就是使用迭代，检查符合条件的项：

```c++
int approach_1()
{
    int arr[10] = {1,20,13,4,5,6,10,28,19,15};
    int count = 0;
    for(int i = 0; i < sizeof(arr)/sizeof(int); i++)
    {
        if(divisible(arr[i], 5))
            count++;
    }
    return count;
}
```

实际上，上述方法STL中已经有实现，可以使用**std::count_if** 算法：

***count_if (InputIterator firstValue, InputIterator lastValue, UnaryPredicate predFunctionObject);***

std::count_if返回范围在 [firstValue,lastValue) 中满足条件的元素个数，条件是一元谓词predFunctionObject函数返回true。

为了使用 std::count_if，我们需要把divisible()函数改造成一个一元谓词函数。此时就可以使用std::bind

```c++
int approach_2()
{
    int arr[10] = {1,20,13,4,5,6,10,28,19,15};
    return std::count_if(arr, arr + sizeof(arr)/sizeof(int) , std::bind(&divisible, _1, 5));
}
```

std::bind将divisible的第二个参数固定为5，返回一个一元谓词函数。

> 应该这样讲更自然些，一开始用std::count_if实现统计5的倍数的需求，然后写了一个被5整除的判断函数，然后又有了被7整除的需求，为了避免以后更多同质的需求，直接写了个divisible函数，然后通过std::bind适配。
>
> 怎么感觉也没啥意思呢？可能太深邃，以后慢慢消化。



### std::bind返回的是什么？

std::bind返回的是一个函数对象，上边的例子我们都用auto变量来引用返回值。其实也可以用std::function：

```c++
std::function<int (int) > mod_add_funcObj = std::bind(&add, 20, _1);
```



完整代码例子：

```c++
#include <memory>
#include <functional>
#include <iostream>
#include <algorithm>
using namespace std::placeholders;
int add(int first, int second)
{
    return first + second;
}
bool divisible(int num , int den)
{
    if(num % den == 0)
        return true;
    return false;
}
int approach_1()
{
    int arr[10] = {1,20,13,4,5,6,10,28,19,15};
    int count = 0;
    for(int i = 0; i < sizeof(arr)/sizeof(int); i++)
    {
        if(divisible(arr[i], 5))
            count++;
    }
    return count;
}
int approach_2()
{
    int arr[10] = {1,20,13,4,5,6,10,28,19,15};
    return std::count_if(arr, arr + sizeof(arr)/sizeof(int) , std::bind(&divisible, _1, 5));
}
int main()
{
    int x = add(4,5);
    // Will return 9
    // What if we want to fix the first argument
    auto new_add_func = std::bind(&add, 12, _1);
    x = new_add_func(5);
    // Will return 17
    std::cout<<x<<std::endl;
    auto mod_add_func = std::bind(&add, _2, _1);
    x = mod_add_func(12, 15);
    // Will return 27
    std::cout<<x<<std::endl;
    std::function<int (int) > mod_add_funcObj = std::bind(&add, 20, _1);
    x = mod_add_funcObj(15);
    // Will return 35
    std::cout<<x<<std::endl;
    std::cout<<approach_1()<<std::endl;
    std::cout<<approach_2()<<std::endl;
    return 0;
}
```





