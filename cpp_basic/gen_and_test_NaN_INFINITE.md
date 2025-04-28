产生及判断NaN及INFINITE数值

发布日期: 2013-12-02 13:00:50
原文链接: https://blog.csdn.net/Tonny0832/article/details/17069537

---

本人工作中经常需要用到NaN及Inf浮点数，原来使用是IBM公司封装的CDecfloat数据类型，后来发现这个数据类型的许多缺点：

1\. 数据类型size太大，浪费内存：这个数据类型用来表示浮点数，在x86 32位机器上竟然需要100多个字节。 (浮点数只需要4字节啊，IBM的大神们)

2\. 操作效率低下，来回和string相倒换，有时候完全没有必要。

3\. rescale后，原有的数值精度将永久地失去，其仅仅方便于在GUI上显示而已，作为科学计算已经不再合适。

后来我只好用float/double来替换CDecfloat数据类型，发现碰到了NaN和Inf问题，而在CDecfloat中，NaN和Inf问题CDecfloat是自己记录了标志位实现的，即CDecfloat使用了额外的内存来表示NaN/inf属性。不自己封装类型的话，不可能给每个float/double来附加一个标志数据的。后来经过研究NaN及Inf的定义，及跟踪std对NaN/Inf的实现，总结出了如下的规律，在VS2008上验证通过。按照下列方法，我们可以简单的生成NaN/Inf及判断一个数据是不是NaN/Inf，不再需要平台提供的专有库函数。

  


NaN及INFINITE数值

通常INFINITE会简化表示为Inf。对于怎么判断一个浮点数是不是Nan或者inf，有现成的C++库函数可用：_isnan / _fpclass，但是对于怎么生成一个Nan浮点数，却没有现在的库函数可用，即使有也往往是第三方库封装生成的，或者有平台依赖的。现在探索如何生成一个Nan及Inf浮点数。   
  
1).  NaN及INFINITE定义及使用规则   
浮点数除了可以表示正常的数值外，还可以表示非数据及无穷大，无穷小等情况。   
  
  
1.1.  NaN及Inf数学运算   
这些数参与运算，有时会产生有意义的输出的：   
2 + Inf = Inf;  //任意数和无穷大相加结果为无穷大   
4/ Inf  = 0, //任意有限数除以无穷大结果为0   
atan(Inf) = &pi  //对无穷大数取反正切，结果为π。(园周率，3.1415...)   
任意数和NaN运算，结果都是NaN。   
  
  
1.2.  NaN及Inf逻辑运算   
NaN不大于，不小于，不等于任意数(包括其自身)。   
Inf在逻辑运算时，除了自身及NaN，正无穷大大于所有其它数，负无穷数小于所有其它数。   
  
  
1.3.  浮点数异常   
当产生任意NaN及INFINITE数值时，都会产生一个FP异常(FP Exceptions) 。IEEE 754定义了下列异常：   
  无效运算(Invalid Operation) ：如NaN及Inf参与的操作，负值求平方根，无效字符串转浮点数   
  除0运算(Division by Zero)   
  上溢出(Overflow) ： 数据太大超出浮点数表示范围(不是无穷大)   
  下溢出(Underflow) ： 数据太小超出浮点数表示精度(不是0，也不是负无穷大)   
  不精确(Inexact) ： 比较一个数表示对2取平方根的结果，或者表示1/3的结果，其小数位是无穷的，但是浮点数表示的小数位是有限的，从而导致不精确的异常。(注意：1.3333333 不等于 1/3)   
  
  
2).  如何产生NaN及INFINITE数   
如何产生一个NaN数呢？C库及C++库都没有提供。   
对于一个无效数值，通常可以用下列纯运算操作产生：   
对负1开方(sqrt (-1))  :  会产生无意义的数(Nan)   
除0 (1/0)  :  会产生正inf数.(正无穷大：Infinite)   
对0取对数(log(0)) :  会产生负inf数.(负无穷大：-Infinite)   
  
  
2.1.  纯数学运算产生   
产生NaN : 根据上面的定义，可以使用下列语句产生：   
float fNan = sqrt (-1.0f);   
double dNan = sqrt (-1.0);   
产生Inf : 产生一个正无穷大的数   
double div = 0;   
return 1.0/div;   
注意：如果直接使用1.0/0.0在MS的编译器上会导致除0的编译错误。   
(其它编译器或许可以，但不提倡使用这样编译器依赖的代码)   
产生一个负无穷大的数   
double div = 0;   
return -1.0/div;   
或：   
double dInf = log(0.0);   
  
  
2.2.  使用STD命名空间提供的封装   
std命令空间中为我们提供了大量的数值属性封装。   
如：std::numeric_limits<float>::has_quiet_NaN();   
std::numeric_limits<float>::quiet_NaN();   
std::numeric_limits<float>::infinity();   
可以用来生成这些Nan及Inf数值。   
但有些情况下，比如特殊平台，std::numeric_limits<float>可能不存在，那么将无法使用这些函数。但是跟踪win平台下这些函数的内部实现，发现它们都是使用的一个全局常量。   
extern _CRTIMP2_PURE /* const */ _Dconst _Denorm, _Hugeval, _Inf,_Nan, _Snan;   
_Dconst的类型定义：   
typedef union   
{  /* pun float types as integer array */   
unsigned short _Word[8];   
float _Float;   
double _Double;   
long double _Long_double;   
} _Dconst;   
  
  
如果如果要使用double的Nan值，可以：   
double  dNan = ::_Nan._Double   
如果如果要使用double的Inf值，可以：   
double  dInf = ::_Inf._Double   
注意：但是经过验证，要使用float的Nan值,使用::_Nan._Float所得结果是不正确的，其结果为0.0000000   
正确的获得float的Nan值的方法是使用_FNan._Float   
即在\VC\include\ymath.h中，定义有多组_Dconst常量，其分别对应不同类型的Nan值及inf值。   
对应double类型：extern _CRTIMP2_PURE /* const */ _Dconst _Denorm, _Hugeval, _Inf,_Nan, _Snan;   
对应double类型：extern _CRTIMP2_PURE /* const */ _Dconst _FDenorm, _FInf, _FNan, _FSnan;   
对应long double类型：extern _CRTIMP2_PURE /* const */ _Dconst _LDenorm, _LInf, _LNan, _LSnan;   
  
  
2.3.  [本质方法]填充特定位   
所谓有Nan及Inf数值，都是在IEEE中定义的特定位满足指定要求的数值。   
那么最原始的方法就是通过操纵移位，还构造满足要求的数值。   


有VS的源文件\VC\crt\src\xvalues.c里面有具体的实现代码。

  


3).  检测NaN及Inf   
3.1.  自身相等判断   
根据NaN及Inf逻辑运算中定义的规则，简单的办法(已在VS2008验证)：   
检查一个值是否为NaN : NaN ≠ NaN   
bool isNan(float fN)   
{   
return !(fN==fN);   
}   
如果dNan不是Nan，则dNan==dNan返回真，否则其返回假。   
  
检查一个值是否为Inf : inf + 1 = Inf   
bool isInf(float fN)   
{   
return fN == (fN+1.0);   
}   
  
  
3.2.  使用库函数   
double dNan;   
bool isNan = _isnan(dNan);   
或者判断一个数是不是inf，则要复杂一点，要使用库函数_fpclass。   
原型：int __cdecl _fpclass(_In_ double _X)   
注意：这个库函数在不同平台上，可能名称不一样。   
其返回值的意义：(不同平台上宏名称可能不一样，但意义大致相同)   
FP_NAN  //不是数值   
FP_INFINITE  //无穷数(无穷大或者无穷小)   
FP_ZERO  //此数是零. （+0 或者 -0 ,IEEE 754）   
FP_SUBNORMAL  //低于浮点数最小表示精度的非常小的数(但不是零)，接近于0的数   
FP_NORMAL  //除了以上几种情况的正常数据   
对这些返回值的组合会有下列的属性判断：   
isfinite <==> (fpclassify (x) != FP_NAN && fpclassify (x) != FP_INFINITE)   
isnormal <==> (fpclassify (x) == FP_NORMAL)   
isnan <==> (fpclassify (x) == FP_NAN)   
有些平台上还会提供一套对应的有符号属性，其一般用S表示。(比如MS平台，还指示是正无穷大/负无穷大)   
如NaN表示不带符号的NaN值，SNaN表示带符号的NaN值。 
