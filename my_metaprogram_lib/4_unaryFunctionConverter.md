4) 将多元函数转换为一元函数 unaryFunctionConverter

发布日期: 2014-02-12 10:55:11
原文链接: https://blog.csdn.net/Tonny0832/article/details/19110831

---

在既有代码中编程时，如果想对一个数据集(容器/数组)中的所有元素进行处理时，其处理函数一般只能接受一个参数－－数据集中元素的数据类型，而既有的接口可能需要多个参数－－数据集的数据类型只是这多个需求参数中的一个参数。

在通常情况下，如果想使用标准算法中的forEach算法，则需要就此功能另外再写一个函数/函数对象以把额外的参数封装在函数内部，或者直接自己遍历这个数据集，把既有函数中的代码Ctrl+C/Ctrl+V到遍历处，无论怎样，这都额外增加了工作量，而且有可能因为疏忽而出错。

下面的代码就是为了解决这一问题而实现的：把一个多元函数(最少一元，最多五元)转换为一元函数，以供遍历算法(如forEach类似的函数)使用。支持的转换函数形式为：

非成员函数：

R (*)(P1);  
R (*)(P1,P2);  
R (*)(P1,P2,P3);  
R (*)(P1,P2,P3,P4);  
R (*)(P1,P2,P3,P4,P5);

成员函数：  
R (O::*)();   
R (O::*)(P1);  
R (O::*)(P1,P2);  
R (O::*)(P1,P2,P3);  
R (O::*)(P1,P2,P3,P4);  
R (O::*)(P1,P2,P3,P4,P5);  


可以把既有函数列表中的任何一个参数作为新的函数接口的传入参数，这样就极大地增强了使用灵活性：所需要的新函数的参数，可能是旧函数的第0，1，2，3，4，5个参数中的任意一个。其中0表示是成员函数的对象类型。

如果执行了错误的转换，则编译时会失败，从而保证了能够通过的转换都是正确的，将代码的逻辑功能正确性交付给了编译器来验错－－这正是元代码的最让人兴奋的威力，我相信编译器出错的概率相对于编程者来说是可以忽略不计的。

源代码可以在我的资源中去下载(包括用法测试的源代码),只需要用VS建一个空的工程，然后引入我的测试头文件即可。然后测试函数会在main函数之前运行，在控制台窗口中可以看到输出。

#ifndef test_unaryFunctorCreater_h__  
#define test_unaryFunctorCreater_h__  
#include <utility/functionCreater.h>  
  
  
/********************************************************************  
  
  
Description : pack multiple parameters (1...5 parameters) function as unary function, which is useful in STL algorithm(e.g. forEach) .  
Author : Shen.Xiaolong (Shen Tony)  
Date : 2012.12.03  
Mail : xlshen2002@hotmail.com, xlshen@126.com  
verified platform : VS2008  
copyright: : free to use / modify / sale in free and commercial software .  
Unique limit: MUST keep those copyright comments in all copies and in supporting documentation.  
usage demo : #define RUN_EXAMPLE_FUNCTORWRAPPER and include this header file to run demo  
*********************************************************************/  
//#define RUN_EXAMPLE_FUNCTORWRAPPER  
  
  
#ifdef COMPILE_EXAMPLE_ALL  
#define COMPILE_EXAMPLE_FUNCTORWRAPPER  
#endif  
  
  
#ifdef RUN_EXAMPLE_ALL  
#define RUN_EXAMPLE_FUNCTORWRAPPER  
#endif  
  
  
#if defined(RUN_EXAMPLE_FUNCTORWRAPPER) && !defined(COMPILE_EXAMPLE_FUNCTORWRAPPER)  
#define COMPILE_EXAMPLE_FUNCTORWRAPPER  
#endif  
  
  
/  
//demo example implement begin //  
#ifdef COMPILE_EXAMPLE_FUNCTORWRAPPER  
#include <vector>  
#include "test_data_def.h"  
  
  
namespace DemoUsage  
{  
struct DemoFunction3  
{   
DemoFunction3()  
{  
m_dat = rand()%100;  
}  
  
  
void memFuncP0()  
{  
}  
  
  
void memFuncP1(int a)  
{  
}  
  
  
void memFuncP2(int a,char& b)  
{  
}  
  
  
void memFuncP3(int a,char& b,DemoFunction3 c)  
{  
}  
  
  
void memFuncP4(int a,char& b,DemoFunction3 c,char*)  
{  
}  
  
  
void memFuncP5(int a,char& b,DemoFunction3 c,char*,int)  
{  
}  
  
  
int m_dat;  
};  
  
  
void funcP1(int a)  
{  
}  
  
  
void funcP2(int a,char& b)  
{  
}  
  
  
void funcP3(int a,char& b,DemoFunction3 c)  
{  
}  
  
  
void funcP4(int a,char& b,DemoFunction3 c,char*)  
{  
}  
  
  
void funcP5(int a,char& b,DemoFunction3 c,char*,int)  
{  
}  
  
  
void showUsageFunctorWrapper()  
{  
using namespace UtilExt;  
using namespace std;  
  
  
/  
DemoFunction3 a;  
int b = 4;  
char c = 5;  
DemoFunction3 d;  
char* e = NULL;  
int f = 7;  
  
  
DemoFunction3 objs[5];   
int ints[5];  
char chars[5];  
char* pchars[5]={NULL};  
  
  
typedef void (DemoFunction3::*MemF_T0)();  
typedef void (DemoFunction3::*MemF_T1)(int a);  
typedef void (DemoFunction3::*MemF_T2)(int,char&);  
typedef void (DemoFunction3::*MemF_T3)(int,char&,DemoFunction3);  
typedef void (DemoFunction3::*MemF_T4)(int a,char& b,DemoFunction3 c,char* d);  
typedef void (DemoFunction3::*MemF_T5)(int a,char& b,DemoFunction3 c,char* d,int e);  
  
  
UnaryFunction<MemF_T0,0> rever00=UnaryFunctionCreater<MemF_T0,0>::create(&DemoFunction3::memFuncP0,_$);  
UnaryFunction<MemF_T1,0> rever10=UnaryFunctionCreater<MemF_T1,0>::create(&DemoFunction3::memFuncP1,_$,b);  
UnaryFunction<MemF_T2,0> rever20=UnaryFunctionCreater<MemF_T2,0>::create(&DemoFunction3::memFuncP2,_$,b,c);  
UnaryFunction<MemF_T3,0> rever30=UnaryFunctionCreater<MemF_T3,0>::create(&DemoFunction3::memFuncP3,_$,b,c,d);  
UnaryFunction<MemF_T4,0> rever40=UnaryFunctionCreater<MemF_T4,0>::create(&DemoFunction3::memFuncP4,_$,b,c,d,e);  
UnaryFunction<MemF_T5,0> rever50=UnaryFunctionCreater<MemF_T5,0>::create(&DemoFunction3::memFuncP5,_$,b,c,d,e,b);  
  
  
UnaryFunction<MemF_T1,1> rever11=UnaryFunctionCreater<MemF_T1,1>::create(&DemoFunction3::memFuncP1,a,_$);  
UnaryFunction<MemF_T2,1> rever21=UnaryFunctionCreater<MemF_T2,1>::create(&DemoFunction3::memFuncP2,a,_$,c);  
UnaryFunction<MemF_T3,1> rever31=UnaryFunctionCreater<MemF_T3,1>::create(&DemoFunction3::memFuncP3,a,_$,c,d);  
UnaryFunction<MemF_T4,1> rever41=UnaryFunctionCreater<MemF_T4,1>::create(&DemoFunction3::memFuncP4,a,_$,c,d,e);  
UnaryFunction<MemF_T5,1> rever51=UnaryFunctionCreater<MemF_T5,1>::create(&DemoFunction3::memFuncP5,a,_$,c,d,e,b);  
  
  
UnaryFunction<MemF_T2,2> rever22=UnaryFunctionCreater<MemF_T2,2>::create(&DemoFunction3::memFuncP2,a,b,_$);  
UnaryFunction<MemF_T3,2> rever32=UnaryFunctionCreater<MemF_T3,2>::create(&DemoFunction3::memFuncP3,a,b,_$,d);  
UnaryFunction<MemF_T4,2> rever42=UnaryFunctionCreater<MemF_T4,2>::create(&DemoFunction3::memFuncP4,a,b,_$,d,e);  
UnaryFunction<MemF_T5,2> rever52=UnaryFunctionCreater<MemF_T5,2>::create(&DemoFunction3::memFuncP5,a,b,_$,d,e,b);  
  
  
UnaryFunction<MemF_T3,3> rever33=UnaryFunctionCreater<MemF_T3,3>::create(&DemoFunction3::memFuncP3,a,b,c,_$);  
UnaryFunction<MemF_T4,3> rever43=UnaryFunctionCreater<MemF_T4,3>::create(&DemoFunction3::memFuncP4,a,b,c,_$,e);  
UnaryFunction<MemF_T5,3> rever53=UnaryFunctionCreater<MemF_T5,3>::create(&DemoFunction3::memFuncP5,a,b,c,_$,e,b);  
  
  
UnaryFunction<MemF_T4,4> rever44=UnaryFunctionCreater<MemF_T4,4>::create(&DemoFunction3::memFuncP4,a,b,c,d,_$);  
UnaryFunction<MemF_T5,4> rever54=UnaryFunctionCreater<MemF_T5,4>::create(&DemoFunction3::memFuncP5,a,b,c,d,_$,f);  
  
  
UnaryFunction<MemF_T5,5> rever55=UnaryFunctionCreater<MemF_T5,5>::create(&DemoFunction3::memFuncP5,a,b,c,d,e,_$);  
/  
  
  
//*/  
UnaryFunction<MemF_T0,0> g_rever00=makeUnaryFunction(&DemoFunction3::memFuncP0,_$);  
UnaryFunction<MemF_T1,0> g_rever10=makeUnaryFunction(&DemoFunction3::memFuncP1,_$,b);  
UnaryFunction<MemF_T2,0> g_rever20=makeUnaryFunction(&DemoFunction3::memFuncP2,_$,b,c);  
UnaryFunction<MemF_T3,0> g_rever30=makeUnaryFunction(&DemoFunction3::memFuncP3,_$,b,c,d);  
UnaryFunction<MemF_T4,0> g_rever40=makeUnaryFunction(&DemoFunction3::memFuncP4,_$,b,c,d,e);  
UnaryFunction<MemF_T5,0> g_rever50=makeUnaryFunction(&DemoFunction3::memFuncP5,_$,b,c,d,e,b);  
  
  
UnaryFunction<MemF_T1,1> g_rever11=makeUnaryFunction(&DemoFunction3::memFuncP1,a,_$);  
UnaryFunction<MemF_T2,1> g_rever21=makeUnaryFunction(&DemoFunction3::memFuncP2,a,_$,c);  
UnaryFunction<MemF_T3,1> g_rever31=makeUnaryFunction(&DemoFunction3::memFuncP3,a,_$,c,d);  
UnaryFunction<MemF_T4,1> g_rever41=makeUnaryFunction(&DemoFunction3::memFuncP4,a,_$,c,d,e);  
UnaryFunction<MemF_T5,1> g_rever51=makeUnaryFunction(&DemoFunction3::memFuncP5,a,_$,c,d,e,b);  
  
  
UnaryFunction<MemF_T2,2> g_rever22=makeUnaryFunction(&DemoFunction3::memFuncP2,a,b,_$);  
UnaryFunction<MemF_T3,2> g_rever32=makeUnaryFunction(&DemoFunction3::memFuncP3,a,b,_$,d);  
UnaryFunction<MemF_T4,2> g_rever42=makeUnaryFunction(&DemoFunction3::memFuncP4,a,b,_$,d,e);  
UnaryFunction<MemF_T5,2> g_rever52=makeUnaryFunction(&DemoFunction3::memFuncP5,a,b,_$,d,e,b);  
  
  
UnaryFunction<MemF_T3,3> g_rever33=makeUnaryFunction(&DemoFunction3::memFuncP3,a,b,c,_$);  
UnaryFunction<MemF_T4,3> g_rever43=makeUnaryFunction(&DemoFunction3::memFuncP4,a,b,c,_$,e);  
UnaryFunction<MemF_T5,3> g_rever53=makeUnaryFunction(&DemoFunction3::memFuncP5,a,b,c,_$,e,b);  
  
  
UnaryFunction<MemF_T4,4> g_rever44=makeUnaryFunction(&DemoFunction3::memFuncP4,a,b,c,d,_$);  
UnaryFunction<MemF_T5,4> g_rever54=makeUnaryFunction(&DemoFunction3::memFuncP5,a,b,c,d,_$,f);  
  
  
UnaryFunction<MemF_T5,5> g_rever55=makeUnaryFunction(&DemoFunction3::memFuncP5,a,b,c,d,e,_$);  
  
  
for(int i=0;i<5;i++)  
{  
rever00(objs[i]);  
rever10(objs[i]);  
rever20(objs[i]);  
rever30(objs[i]);  
rever40(objs[i]);  
rever50(objs[i]);  
  
  
rever11(ints[i]);  
rever21(ints[i]);  
rever31(ints[i]);  
rever41(ints[i]);  
rever51(ints[i]);  
  
  
rever22(chars[i]);  
rever32(chars[i]);  
rever42(chars[i]);  
rever52(chars[i]);  
  
  
rever33(objs[i]);  
rever43(objs[i]);  
rever53(objs[i]);  
  
  
rever44(pchars[i]);  
rever54(pchars[i]);  
  
  
rever55(ints[i]);   
/  
g_rever00(objs[i]);  
g_rever10(objs[i]);  
g_rever20(objs[i]);  
g_rever30(objs[i]);  
g_rever40(objs[i]);  
g_rever50(objs[i]);  
  
  
g_rever11(ints[i]);  
g_rever21(ints[i]);  
g_rever31(ints[i]);  
g_rever41(ints[i]);  
g_rever51(ints[i]);  
  
  
g_rever22(chars[i]);  
g_rever32(chars[i]);  
g_rever42(chars[i]);  
g_rever52(chars[i]);  
  
  
g_rever33(objs[i]);  
g_rever43(objs[i]);  
g_rever53(objs[i]);  
  
  
g_rever44(pchars[i]);  
g_rever54(pchars[i]);  
  
  
g_rever55(ints[i]);   
}  
  
  
typedef void (*F_T1)(int a);  
typedef void (*F_T2)(int a,char& b);  
typedef void (*F_T3)(int a,char& b,DemoFunction3 c);  
typedef void (*F_T4)(int a,char& b,DemoFunction3 c,char*);  
typedef void (*F_T5)(int a,char& b,DemoFunction3 c,char*,int);  
  
  
UnaryFunction<F_T1,1> reverg11=UnaryFunctionCreater<F_T1,1>::create(&funcP1,_$);  
UnaryFunction<F_T2,1> reverg21=UnaryFunctionCreater<F_T2,1>::create(&funcP2,_$,c);  
UnaryFunction<F_T3,1> reverg31=UnaryFunctionCreater<F_T3,1>::create(&funcP3,_$,c,d);   
UnaryFunction<F_T4,1> reverg41=UnaryFunctionCreater<F_T4,1>::create(&funcP4,_$,c,d,e);   
UnaryFunction<F_T5,1> reverg51=UnaryFunctionCreater<F_T5,1>::create(&funcP5,_$,c,d,e,f);  
  
  
UnaryFunction<F_T2,2> reverg22=UnaryFunctionCreater<F_T2,2>::create(&funcP2,b,_$);  
UnaryFunction<F_T3,2> reverg32=UnaryFunctionCreater<F_T3,2>::create(&funcP3,b,_$,d);   
UnaryFunction<F_T4,2> reverg42=UnaryFunctionCreater<F_T4,2>::create(&funcP4,b,_$,d,e);   
UnaryFunction<F_T5,2> reverg52=UnaryFunctionCreater<F_T5,2>::create(&funcP5,b,_$,d,e,f);  
  
  
UnaryFunction<F_T3,3> reverg33=UnaryFunctionCreater<F_T3,3>::create(&funcP3,b,c,_$);   
UnaryFunction<F_T4,3> reverg43=UnaryFunctionCreater<F_T4,3>::create(&funcP4,b,c,_$,e);   
UnaryFunction<F_T5,3> reverg53=UnaryFunctionCreater<F_T5,3>::create(&funcP5,b,c,_$,e,f);  
  
  
UnaryFunction<F_T4,4> reverg44=UnaryFunctionCreater<F_T4,4>::create(&funcP4,b,c,d,_$);   
UnaryFunction<F_T5,4> reverg54=UnaryFunctionCreater<F_T5,4>::create(&funcP5,b,c,d,_$,f);  
  
  
UnaryFunction<F_T5,5> reverg55=UnaryFunctionCreater<F_T5,5>::create(&funcP5,b,c,d,e,_$);   
  
  
/  
UnaryFunction<F_T1,1> g_reverg11=makeUnaryFunction(&funcP1,_$);  
UnaryFunction<F_T2,1> g_reverg21=makeUnaryFunction(&funcP2,_$,c);  
UnaryFunction<F_T3,1> g_reverg31=makeUnaryFunction(&funcP3,_$,c,d);   
UnaryFunction<F_T4,1> g_reverg41=makeUnaryFunction(&funcP4,_$,c,d,e);   
UnaryFunction<F_T5,1> g_reverg51=makeUnaryFunction(&funcP5,_$,c,d,e,f);  
  
  
UnaryFunction<F_T2,2> g_reverg22=makeUnaryFunction(&funcP2,b,_$);  
UnaryFunction<F_T3,2> g_reverg32=makeUnaryFunction(&funcP3,b,_$,d);   
UnaryFunction<F_T4,2> g_reverg42=makeUnaryFunction(&funcP4,b,_$,d,e);   
UnaryFunction<F_T5,2> g_reverg52=makeUnaryFunction(&funcP5,b,_$,d,e,f);  
  
  
UnaryFunction<F_T3,3> g_reverg33=makeUnaryFunction(&funcP3,b,c,_$);   
UnaryFunction<F_T4,3> g_reverg43=makeUnaryFunction(&funcP4,b,c,_$,e);   
UnaryFunction<F_T5,3> g_reverg53=makeUnaryFunction(&funcP5,b,c,_$,e,f);  
  
  
UnaryFunction<F_T4,4> g_reverg44=makeUnaryFunction(&funcP4,b,c,d,_$);   
UnaryFunction<F_T5,4> g_reverg54=makeUnaryFunction(&funcP5,b,c,d,_$,f);  
  
  
UnaryFunction<F_T5,5> g_reverg55=makeUnaryFunction(&funcP5,b,c,d,e,_$);   
  
  
for(int i=0;i<5;i++)  
{  
reverg11(ints[i]);  
reverg21(ints[i]);  
reverg31(ints[i]);  
reverg41(ints[i]);  
reverg51(ints[i]);  
  
  
reverg22(chars[i]);  
reverg32(chars[i]);  
reverg42(chars[i]);  
reverg52(chars[i]);  
  
  
reverg33(objs[i]);  
reverg43(objs[i]);  
reverg53(objs[i]);  
  
  
reverg44(pchars[i]);  
reverg54(pchars[i]);  
  
  
reverg55(ints[i]);  
/  
g_reverg11(ints[i]);  
g_reverg21(ints[i]);  
g_reverg31(ints[i]);  
g_reverg41(ints[i]);  
g_reverg51(ints[i]);  
  
  
g_reverg22(chars[i]);  
g_reverg32(chars[i]);  
g_reverg42(chars[i]);  
g_reverg52(chars[i]);  
  
  
g_reverg33(objs[i]);  
g_reverg43(objs[i]);  
g_reverg53(objs[i]);  
  
  
g_reverg44(pchars[i]);  
g_reverg54(pchars[i]);  
  
  
g_reverg55(ints[i]);  
}  
}  
  
  
#ifdef RUN_EXAMPLE_FUNCTORWRAPPER  
InitRunFunc(showUsageFunctorWrapper);  
#endif //RUN_EXAMPLE_FUNCTORWRAPPER  
}  
#endif // SHOW_USAGE_FUNCTORWRAPPER  
  
  
#endif // test_unaryFunctorCreater_h__  

