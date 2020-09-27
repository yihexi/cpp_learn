原文链接：https://thispointer.com//designing-callbacks-in-c-part-1-function-pointers/



这边文章介绍什么是callback，callback有哪些类型以及在C++中如何使用函数指针设计callback。



### 什么是callback

callback是一个函数，当我们调用一个API的时候，将其作为参数传入。被调用的API会在某一时刻调用我们传入的函数。



### C++中的callback有3种类型

1) 函数指针
2)函数对象
3)Lambda表达式

我们调用的API的结果由我们传入的callback决定，不同的callback会返回不同的结果。（将API中不确定的部分交由调用方决定）



#### 使用函数指针作为callback

有一个framework，其中有一个API，能将原始数据构建成完整的消息。详情如下：

1）给原始数据添加头部和尾部
2）给整个数据加密
3）返回消息

这个API知道如何添加头部和尾部，但是不知道如何加密数据。加密数据的逻辑由应用程序决定，因此，API提供了一个函数指针参数，由应用程序传入一个加密函数，当需要加密的时候，调用传入的函数指针实现加密逻辑。

API 的实现：

```c++
std::string buildCompleteMessage(std::string rawData, std::string (* encrypterFunPtr)(std::string) )
{
    // Add some header and footer to data to make it complete message
    rawData = "[HEADER]" + rawData + "[FooTER]";
    // Call the callBack provided i.e. function pointer to encrypt the
    rawData = encrypterFunPtr(rawData);
    return rawData;
}
```

Callback函数的实现：

```c++
//This encrypt function increment all letters in string by 1.
std::string encryptDataByLetterInc(std::string data)
{
    for(int i = 0; i < data.size(); i++)
    {
        if( (data[i] >= 'a' && data[i] <= 'z' ) || (data[i] >= 'A' && data[i] <= 'Z' ) )
            data[i]++;
    }
    return data;
}
```

这个回调函数将字符串中的每一个字母加了1.

API的调用：

```c++
std::string msg = buildCompleteMessage("SampleString", &encryptDataByLetterInc);
std::cout<<msg<<std::endl;
```

输出

```c++
[IFBEFS]TbnqmfTusjoh[GppUFS]
```

下边设计一个新的加密函数

```c++
//This encrypt function decrement all letters in string by 1.
std::string encryptDataByLetterDec(std::string data)
{
    for(int i = 0; i < data.size(); i++)
    {
        if( (data[i] >= 'a' && data[i] <= 'z' ) || (data[i] >= 'A' && data[i] <= 'Z' ) )
            data[i]--;
    }
    return data;
}
```

这个加密函数将每个字母减1.

API的调用：

```c++
std::string msg = buildCompleteMessage("SampleString", &encryptDataByLetterDec);
std::cout<<msg<<std::endl;
```

输出：

```c++
[GD@CDQ]R`lokdRsqhmf[EnnSDQ]
```

这两次输入的数据是一样的，但是结果是不一致的，API函数却不用改，这样就通过不同的callback改变了API函数的行为。

以上就是如何使用函数指针作为回调函数。下篇文章会介绍什么是函数对象，为什么需要函数对象。


