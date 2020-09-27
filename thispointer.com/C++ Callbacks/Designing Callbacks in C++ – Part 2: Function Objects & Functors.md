原文链接：https://thispointer.com//designing-callbacks-in-c-part-2-function-objects-functors/



这篇文章讲什么是函数对象，为什么需要函数对象以及如何使用函数对象作为回调函数。



### 什么是函数对象

函数对象是有状态的回调函数。

用程序员的话来说，函数对象是一个重载了()运算符的类，举个例子

```c++
#include <iostream>
class MyFunctor
{
public:
    int operator()(int a , int b)
    {
        return a+b;
    }
};
```

创建一个函数对象并调用

```c++
MyFunctor funObj;
std::cout<<funObj(2,3)<<std::endl;
```

也可以这样调用

```c++
MyFunctor funObj;
funObj.operator ()(2,3);
```

看一个实际例子：

之前有个例子讲如何通过函数指针改变API的行为。（[Designing Callbacks with Function Pointers in C++](https://thispointer.com//designing-callbacks-in-c-part-1-function-pointers/)）

我们再看一下这个过程：

1）给原始数据添加头部和尾部
2）加密数据
3）返回消息

API知道如何添加头部和尾部，但是不知道如何加密数据，加密数据的方式是调用方决定的。所以让调用方通过一个函数指针传入加密算法。

```c++
std::string buildCompleteMessage(std::string rawData,
        std::string (*encrypterFunPtr)(std::string)) {
    // Add some header and footer to data to make it complete message
    rawData = "[HEADER]" + rawData + "[FooTER]";
    // Call the callBack provided i.e. function pointer to encrypt the
    rawData = encrypterFunPtr(rawData);
    return rawData;
}
```

假设应用程序有3个加密逻辑：

* 每个字符+1
* 每个字符+2
* 每个字符-1

使用函数指针的调用方式，我们需要创建3个函数，因为函数是没有状态的。

```c++
//This encrypt function increment all letters in string by 1.
std::string encryptDataByLetterInc1(std::string data) {
    for (int i = 0; i < data.size(); i++)
        if ((data[i] >= 'a' && data[i] <= 'z')
                || (data[i] >= 'A' && data[i] <= 'Z'))
            data[i]++;
    return data;
}
//This encrypt function increment all letters in string by 2.
std::string encryptDataByLetterInc2(std::string data) {
    for (int i = 0; i < data.size(); i++)
        if ((data[i] >= 'a' && data[i] <= 'z')
                || (data[i] >= 'A' && data[i] <= 'Z'))
            data[i] = data[i] + 2;
    return data;
}
//This encrypt function increment all letters in string by 1.
std::string encryptDataByLetterDec(std::string data) {
    for (int i = 0; i < data.size(); i++)
        if ((data[i] >= 'a' && data[i] <= 'z')
                || (data[i] >= 'A' && data[i] <= 'Z'))
            data[i] = data[i] - 1;
    return data;
}
int main() {
    std::string msg = buildCompleteMessage("SampleString",
            &encryptDataByLetterInc1);
    std::cout << msg << std::endl;
    msg = buildCompleteMessage("SampleString", &encryptDataByLetterInc2);
    std::cout << msg << std::endl;
    msg = buildCompleteMessage("SampleString", &encryptDataByLetterDec);
    std::cout << msg << std::endl;
    return 0;
}
```



然而，使用函数对象，我们就可以得到一个有状态的函数。下边创建一个加密的函数对象：

```c++
class Encryptor {
    bool m_isIncremental;
    int m_count;
public:
    Encryptor() {
        m_isIncremental = 0;
        m_count = 1;
    }
    Encryptor(bool isInc, int count) {
        m_isIncremental = isInc;
        m_count = count;
    }
    std::string operator()(std::string data) {
        for (int i = 0; i < data.size(); i++)
            if ((data[i] >= 'a' && data[i] <= 'z')
                    || (data[i] >= 'A' && data[i] <= 'Z'))
                if (m_isIncremental)
                    data[i] = data[i] + m_count;
                else
                    data[i] = data[i] - m_count;
        return data;
    }
};
```

API需要改一下:

```c++
std::string buildCompleteMessage(std::string rawData,
        Encryptor encyptorFuncObj) {
    // Add some header and footer to data to make it complete message
    rawData = "[HEADER]" + rawData + "[FooTER]";
    // Call the callBack provided i.e. function pointer to encrypt the
    rawData = encyptorFuncObj(rawData);
    return rawData;
}
```

这样调用的时候会非常简洁：

```c++
int main() {
    std::string msg = buildCompleteMessage("SampleString", Encryptor(true, 1));
    std::cout << msg << std::endl;
    msg = buildCompleteMessage("SampleString", Encryptor(true, 2));
    std::cout << msg << std::endl;
    msg = buildCompleteMessage("SampleString", Encryptor(false, 1));
    std::cout << msg << std::endl;
    return 0;
}
```



完整代码如下：

```c++
#include <iostream>

class Encryptor {
    bool m_isIncremental;
    int m_count;
public:
    Encryptor() {
        m_isIncremental = 0;
        m_count = 1;
    }
    Encryptor(bool isInc, int count) {
        m_isIncremental = isInc;
        m_count = count;
    }
    std::string operator()(std::string data) {
        for (int i = 0; i < data.size(); i++)
            if ((data[i] >= ‘a’ && data[i] <= ‘z’)
                    || (data[i] >= ‘A’ && data[i] <= ‘Z’))
                if (m_isIncremental)
                    data[i] = data[i] + m_count;
                else
                    data[i] = data[i] – m_count;
        return data;
    }

};

std::string buildCompleteMessage(std::string rawData,
        Encryptor encyptorFuncObj) {
    // Add some header and footer to data to make it complete message
    rawData = "[HEADER]" + rawData + "[FooTER]";

    // Call the callBack provided i.e. function pointer to encrypt the
    rawData = encyptorFuncObj(rawData);

    return rawData;
}

int main() {
    std::string msg = buildCompleteMessage("SampleString", Encryptor(true, 1));
    std::cout << msg << std::endl;

    msg = buildCompleteMessage("SampleString", Encryptor(true, 2));
    std::cout << msg << std::endl;

    msg = buildCompleteMessage("SampleString", Encryptor(false, 1));
    std::cout << msg << std::endl;

    return 0;
}
```





