---
layout: post
title: Reading Acceleration
comments: True
---

一直不明白快读的原理，为校赛出题时研究了一下。

## cin/cout Acceleration

众所周知，以下三行代码可以加速 `cin/cout` 到与 `scanf/printf` 差不多、甚至略快一些的水平。

```cpp
ios::sync_with_stdio(false);
cin.tie(0);
cout.tie(0);
```

第一行顾名思义是设置标准 C++ 流与标准 C 流是否同步。所有标准 C++ 流默认与相应标准 C 流同步，自身是无缓冲的，每次 I/O 操作立即应用到 C 流缓冲区。这使得编程者可以混用两种 I/O。

关闭同步后，允许 C++ 流独立缓冲 I/O，因而更快。

注意，实际应用中，关闭同步可能会导致失去线程安全。

二三两行是管理联系流。联系流是输出流。默认情况下联系 `cin` 和 `cerr` 到 `cout`。为一个流绑定联系流，意味着在这个流的任何 I/O 操作发生前，在联系流上调用 `flush()`。

显然，在竞赛环境中，我们可能会希望这些无用开销尽可能少发生。

## Type Specified Acceleration

以 `cin` 输入一个 `int` 类型为例，程序实际上经历以下几步。

1. `>>` 构造 `sentry` 对象。
2. `sentry` 的作用是为有格式输入准备流。它的任务包括检查流是否成功、在联系流上调用 `flush()`、处理输入的空白符。
3. `>>` 调用 `num_get::get()`。
4. `num_get::get()` 调用最终导出类的成员函数（一个受保护的在子类实现的虚函数）`doget()`。
5. `doget()` 从输入迭代器中读取字符，考虑 I/O 格式化标志，处理字符释出，转换处理过的字符序列为数值。在最后一步中，`doget()` 还会处理由于 `boolalpha`、数位分组、`eof` 产生的流状态变化。

可以看到，反正原理都是读字符序列进来转换，如果我们只希望读入指定格式的指定类型的数据，完全可以自己实现一个类型限定的读入函数，或者，更弱的，实现一个可以读入任意类型，但是功能没有那么复杂的模板。

下面给出一个经典的类型限定读入挂。

```cpp
template <class T>
inline bool scan_d(T &ret) 
{
    char c; 
    int sgn;
    if (c = getchar(), c == EOF) 
    {
        return 0; //EOF 
    }
    while (c != '-' && (c < '0' || c > '9')) 
    {
        c = getchar(); 
    }
    sgn = (c == '-') ? -1 : 1;
    ret = (c == '-') ? 0 : (c - '0'); 
    while (c = getchar(), c >= '0' && c <= '9') 
    {
        ret = ret * 10 + (c - '0'); 
    }
    ret *= sgn;
    return 1;
}

template <class T>
inline void print_d(T x) 
{ 
    if (x > 9) 
    {
        print_d(x / 10); 
    }
    putchar(x % 10 + '0');
}
```

顺便一提，`cin` 的类型判断实际上是模板实现，之所以把上面的读入挂叫做类型限定读入挂，是针对 `scanf` 还要做字符串匹配而言。

## Syscall Acceleration

类型限定读入挂仍然存在一个局限性，那就是 `getchar` 调用系统调用 `read` 来实现。这意味着我们以 `getchar` 循环读入一个字符序列时，系统调用开销增加到线性级别。如果我们能够一次系统调用读入整个字符序列再处理，系统调用开销就会回到常数级别。

下面给出由 `fread` 实现的系统调用加速读入。

```cpp
const int MAXN = 10000000;
const int MAXS = 60*1024*1024;
int numbers[MAXN];
char buf[MAXS];
void analyse(char *buf,int len = MAXS)
{
    int i;
    numbers[i=0]=0;
    for (char *p=buf;*p && p-buf<len;p++)
        if (*p == ' ')
            numbers[++i]=0;
        else
            numbers[i] = numbers[i] * 10 + *p - '0';
}
void fread_analyse()
{
    freopen("data.txt","rb",stdin);
    int len = fread(buf,1,MAXS,stdin);
    buf[len] = '\0';
    analyse(buf,len);
}
```

在 Linux 上，将文件映射到内存的底层函数 `mmap` 也可以（并且更好地）完成这一工作。