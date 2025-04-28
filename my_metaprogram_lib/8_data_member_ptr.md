8) 多级结构数据成员指针 memberPtr 

发布日期: 2014-02-12 13:40:05
原文链接: https://blog.csdn.net/Tonny0832/article/details/19115703

---

源代码及用法测试代码可以在我的资源中去下载.

在一些需求中，对结构体比较/计算/求和/测试的用例中，其往往是对结构/中的某一个数据成员/数据成员的某个子成员/数据成员的...某个子成员进行比较/计算/求和/测试，这就对某类算法产生了一个需求：给定一个结构体，产生这个结构体的指定需求子成员(包括多级子成员)。比如：

struct S1   
{   
int m_s1;   
};   
  
  
struct S2   
{   
S1  m_ss1;   
int m_s2;   
};   
  
  
struct S3   
{   
S2  m_ss2;   
int m_s3;   
};   
struct S4   
{   
  
  
S3  m_ss3;   
int m_s4;   
char m_s4c;   
};   
  
  
要求对结构体数组S4[N]进行排序，排序的依据分别是S4.m_s4/S4.m_s4c/S4.m_ss3.m_s3/.../S4.m_ss3.m_ss2.m_ss1.m_s1，   
很明显，这些排序的算法都是一样的，只是每次排序所使用的比较数据的类型/来源不一样。如果传入一个结构体，自动传出满足指定需求的结果，则就满足了排序算法的要求。   
注意：一个类型如果重载了比较操作符(<,=等等)，而也是可以直接比较的。   


如果对C语言开发者来说，第一个想到的就是结构体成员偏移量，没错，结构体成员偏移可以直接取得需求的成员地址，但是丧失了该成员类型信息，这些类型信息有可能也参与比较－－例如char只使用1字节计算,int使用sizeof(int)字节参与运算，结构型成员甚至有自己独特定义的运算操作符，C语言开发者即使引入第三个参数来指示这些信息，面对复杂的用户case也不能满足需求，而且出错率大大提高，本文件就是为了解决这类需求提供的。

本功能的优点在于：

1.没有任何强制类型转换，一切类型的使用都是安全的。

2.在需要明确地使用类型信息时，源代码中自带的编译期函数能够推导出所需要的参数类型及其修饰符。

3.一切都是可靠的，如果由于用户不小心使用了错误的参数，将导致编译失败，从而阻止错误的代码通过编译，即将部分逻辑的正确性交给编译器来保证－－这在普通代码中是不可能的事。

4.使用及其简单，比如一级子成员：MMP(&S::M) ，二级子成员MMP2(&S1::M1,&S2::M2)，三级子成员：MMP3(&S1::M1,&S2::M2,&S3::M3)或者MMP2(MMP2(&S1::M1,&S2::M2),&S3::M3)，其它提取更多级的子成员数据以此类推。(S为结构体类型，M为数据成员名)

5.没有使用虚函数，效率比较好，多级子成员可用递归嵌套来实现。

  
功能：以统一的形式支持任意多级的结构子成员/...子成员提取，并且不失去类型安全性，不需要使用者传入任何不必要的辅助/指示/约定参数。例如：   
对于S4.m_s4 ： ＆S4::m_s4   
对于S4.m_s4c ： ＆S4::m_s4c   
对于/S4.m_ss3.m_s3  : ＆S4::m_ss3 , &S3::m_s3   
...   
其它的类推。由于使用的是递归的方式实现类型定义的，所以子成员级数也不受限制(只受限于编译器支持的级数，工程实践上可以不必考虑此限制)，类似于printf的可变参数支持的参数个数。   


#ifndef test_memberPtr_h__  
#define test_memberPtr_h__  
#include <utility/utilSmartPtr.h>  
  
  
/********************************************************************  
Description : smart pointer support. it can be replaced by customized smart pointer  
Author : Shen.Xiaolong (Shen Tony)  
Date : 2012.10.26  
Mail : xlshen2002@hotmail.com, xlshen@126.com  
verified platform : VS2008  
copyright: : free to use / modify / sale in free and commercial software.  
Unique limit: MUST keep those copyright comments in all copies and in supporting documentation.  
usage demo : #define RUN_EXAMPLE_MEMBERPTR and include this header file to run demo  
*********************************************************************/  
//#define RUN_EXAMPLE_MEMBERPTR  
  
  
#ifdef COMPILE_EXAMPLE_ALL  
#define COMPILE_EXAMPLE_MEMBERPTR  
#endif  
  
  
#ifdef RUN_EXAMPLE_ALL  
#define RUN_EXAMPLE_MEMBERPTR  
#endif  
  
  
#if defined(RUN_EXAMPLE_MEMBERPTR) && !defined(COMPILE_EXAMPLE_MEMBERPTR)  
#define COMPILE_EXAMPLE_MEMBERPTR  
#endif  
  
  
usage demo code//  
#ifdef COMPILE_EXAMPLE_MEMBERPTR  
  
  
namespace DemoUsage  
{   
void showUsageMemberPtr()  
{  
PrintUsage();  
using namespace UtilExt;  
  
  
S4 obj;  
CGetType::Pointer_to_Member::apply<S4,S3,S2,S1,int>::type rPtrRaw4=MMP4(&S4::m_ss3,&S3::m_ss2,&S2::m_ss1,&S1::m_s1);  
MemberPtr<S4,int>& rPtr4=rPtrRaw4;  
//Assertb(rPtr4(obj)==obj.m_ss3.m_ss2.m_ss1.m_s1);  
Assertb(rPtrRaw4(obj)==obj.m_ss3.m_ss2.m_ss1.m_s1);  
  
  
CGetType::Pointer_to_Member::apply<S4,S3,S2,int>::type rPtrRaw3=MMP3(&S4::m_ss3,&S3::m_ss2,&S2::m_s2);  
MemberPtr<S4,int>& rPtr3=rPtrRaw3;  
//Assertb(rPtr3(obj)==obj.m_ss3.m_ss2.m_s2);  
Assertb(rPtrRaw3(obj)==obj.m_ss3.m_ss2.m_s2);  
  
  
CGetType::Pointer_to_Member::apply<S4,S3,int>::type rPtrRaw2=MMP2(&S4::m_ss3,&S3::m_s3);  
MemberPtr<S4,int>& rPtr2=rPtrRaw2;  
Assertb(rPtrRaw2(obj)==obj.m_ss3.m_s3);  
  
  
CGetType::Pointer_to_Member::apply<S4,int>::type rPtrRaw1=MMP(&S4::m_s4);  
MemberPtr<S4,int>& rPtr1=rPtrRaw1;  
Assertb(rPtrRaw1(obj)==obj.m_s4);  
}  
  
  
#ifdef RUN_EXAMPLE_MEMBERPTR  
InitRunFunc(showUsageMemberPtr);   
#endif //RUN_EXAMPLE_MEMBERPTR  
  
  
}  
  
  
#endif // COMPILE_EXAMPLE_MEMBERPTR  
  
  
#endif // test_memberPtr_h__  

