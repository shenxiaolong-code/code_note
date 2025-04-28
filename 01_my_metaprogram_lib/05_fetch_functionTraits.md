5) 函数特征 functionTraits 

发布日期: 2014-02-12 11:21:44
原文链接: https://blog.csdn.net/Tonny0832/article/details/19112297

---

源代码可以在我的资源中去下载(包括用法测试的源代码):   
http://download.csdn.net/detail/tonny0832/7426991   
只需要用VS建一个空的工程，然后引入我的测试头文件即可。然后测试函数会在main函数之前运行，在控制台窗口中可以看到输出。   
  
前面已经有了类型特征，用于判断一个类型的各种属性。   
函数也是程序设计中的一个重要概念，因此判断一个函数的各种属性也很重要。   
这些属性包括一个函数的：   
返回值，参数个数，每个参数的类型   
如果是成员函数，还应该包括对象的类型。   


这个文件支持了查询函数属性的功能，但是最多只支持5个参数的函数(成员函数的对象类型除外)

#ifndef functorTraits_h__  
#define functorTraits_h__  
/********************************************************************  
Description : functor is complex, sometime it is needed to detect functor properties .  
Limit : it is only suitable to the kind of function which has fixed parameter number.   
it is say : for style : R (*)(...) , it is not usable.   
Support functions style:  
R (*)(P1);  
R (*)(P1,P2);  
R (*)(P1,P2,P3);  
R (*)(P1,P2,P3,P4);  
R (*)(P1,P2,P3,P4,P5);  
R (O::*)(P1);  
R (O::*)(P1,P2);  
R (O::*)(P1,P2,P3);  
R (O::*)(P1,P2,P3,P4);  
R (O::*)(P1,P2,P3,P4,P5);  
note : macro FPP is usable to improve deliver parameter performance, it obey below rules:  
if function parameter (Px) is reference type, the FPP(F,x) is reference type also. (because it might be in/out parameter)  
if function parameter (Px) isn't reference type, the FPP(F,x) is :  
1\. non-reference type when Px is Atom type.(e.g. build-in type or enum type)  
1\. reference type when Px isn't Atom type. (e.g. struct/class type)  
  
  
Author : Shen.Xiaolong (Shen Tony) (2010-2013)  
Mail : xlshen2002@hotmail.com, xlshen@126.com  
verified platform : VS2008  
copyright: : latest Version of The Code Project Open License (CPOL : http://www.codeproject.com/)  
*********************************************************************/  
#include "typeConvert.h"  
  
  
#define FO(F) typename UtilExt::FunctionParams<F>::Object_T  
#define FR(F) typename UtilExt::FunctionParams<F>::Return_T  
#define FP(F,I) typename UtilExt::GetFunctionParam<F,I>::type  
#define FPP(F,I) TPP(FP(F,I)) // Function Parameter package  
  
  
namespace UtilExt  
{  
//FunctionParams Impl///  
template<typename P1=NullType,typename P2=NullType,typename P3=NullType,typename P4=NullType,typename P5=NullType>   
struct FunctionParamsArrayImpl  
{  
template<int N> struct apply;  
template<> struct apply<0> { };  
template<> struct apply<1> { typedef P1 TP1; };  
template<> struct apply<2> : public apply<1> { typedef P2 TP2; };  
template<> struct apply<3> : public apply<2> { typedef P3 TP3; };   
template<> struct apply<4> : public apply<3> { typedef P4 TP4; };  
template<> struct apply<5> : public apply<4> { typedef P5 TP5; };  
};  
  
  
template<typename R,typename Params,int N>  
struct FunctionParamsImpl : public Params::template apply<N>  
{  
typedef R Return_T;   
enum {ParamNum = N };  
};  
  
  
template<typename R,typename O,typename Params,int N>  
struct MemFunctionParamsImpl : public FunctionParamsImpl<R,Params,N>  
{  
typedef O Object_T;  
};  
  
  
template<typename R> struct FunctionParams; //3 style : function pointer / function reference / function   
  
  
template<typename R> struct FunctionParams<R (*)()> : public FunctionParamsImpl<R,FunctionParamsArrayImpl<>,0> { };  
template<typename R> struct FunctionParams<R (&)()> : public FunctionParamsImpl<R,FunctionParamsArrayImpl<>,0> { };  
template<typename R> struct FunctionParams<R ()> : public FunctionParamsImpl<R,FunctionParamsArrayImpl<>,0> { };  
  
  
template<typename R,typename P1> struct FunctionParams<R (*)(P1)> : public FunctionParamsImpl<R,FunctionParamsArrayImpl<P1>,1> { };  
template<typename R,typename P1> struct FunctionParams<R (&)(P1)> : public FunctionParamsImpl<R,FunctionParamsArrayImpl<P1>,1> { };  
template<typename R,typename P1> struct FunctionParams<R (P1)> : public FunctionParamsImpl<R,FunctionParamsArrayImpl<P1>,1> { };  
  
  
template<typename R,typename P1,typename P2> struct FunctionParams<R (*)(P1,P2)> : public FunctionParamsImpl<R,FunctionParamsArrayImpl<P1,P2>,2> { };  
template<typename R,typename P1,typename P2> struct FunctionParams<R (&)(P1,P2)> : public FunctionParamsImpl<R,FunctionParamsArrayImpl<P1,P2>,2> { };  
template<typename R,typename P1,typename P2> struct FunctionParams<R (P1,P2)> : public FunctionParamsImpl<R,FunctionParamsArrayImpl<P1,P2>,2> { };  
  
  
template<typename R,typename P1,typename P2,typename P3> struct FunctionParams<R (*)(P1,P2,P3)> : public FunctionParamsImpl<R,FunctionParamsArrayImpl<P1,P2,P3>,3> { };  
template<typename R,typename P1,typename P2,typename P3> struct FunctionParams<R (&)(P1,P2,P3)> : public FunctionParamsImpl<R,FunctionParamsArrayImpl<P1,P2,P3>,3> { };  
template<typename R,typename P1,typename P2,typename P3> struct FunctionParams<R (P1,P2,P3)> : public FunctionParamsImpl<R,FunctionParamsArrayImpl<P1,P2,P3>,3> { };  
  
  
template<typename R,typename P1,typename P2,typename P3,typename P4> struct FunctionParams<R (*)(P1,P2,P3,P4)> : public FunctionParamsImpl<R,FunctionParamsArrayImpl<P1,P2,P3,P4>,4> { };  
template<typename R,typename P1,typename P2,typename P3,typename P4> struct FunctionParams<R (&)(P1,P2,P3,P4)> : public FunctionParamsImpl<R,FunctionParamsArrayImpl<P1,P2,P3,P4>,4> { };  
template<typename R,typename P1,typename P2,typename P3,typename P4> struct FunctionParams<R (P1,P2,P3,P4)> : public FunctionParamsImpl<R,FunctionParamsArrayImpl<P1,P2,P3,P4>,4> { };  
  
  
template<typename R,typename P1,typename P2,typename P3,typename P4,typename P5> struct FunctionParams<R (*)(P1,P2,P3,P4,P5)> : public FunctionParamsImpl<R,FunctionParamsArrayImpl<P1,P2,P3,P4,P5>,5> { };  
template<typename R,typename P1,typename P2,typename P3,typename P4,typename P5> struct FunctionParams<R (&)(P1,P2,P3,P4,P5)> : public FunctionParamsImpl<R,FunctionParamsArrayImpl<P1,P2,P3,P4,P5>,5> { };  
template<typename R,typename P1,typename P2,typename P3,typename P4,typename P5> struct FunctionParams<R (P1,P2,P3,P4,P5)> : public FunctionParamsImpl<R,FunctionParamsArrayImpl<P1,P2,P3,P4,P5>,5> { };  
  
  
//NO reference-to-member-function in C++  
template<typename R,typename O> struct FunctionParams<R (O::*)()> : public MemFunctionParamsImpl<R,O,FunctionParamsArrayImpl<>,0> { };  
template<typename R,typename O,typename P1> struct FunctionParams<R (O::*)(P1)> : public MemFunctionParamsImpl<R,O,FunctionParamsArrayImpl<P1>,1> { };  
template<typename R,typename O,typename P1,typename P2> struct FunctionParams<R (O::*)(P1,P2)> : public MemFunctionParamsImpl<R,O,FunctionParamsArrayImpl<P1,P2>,2> { };  
template<typename R,typename O,typename P1,typename P2,typename P3> struct FunctionParams<R (O::*)(P1,P2,P3)> : public MemFunctionParamsImpl<R,O,FunctionParamsArrayImpl<P1,P2,P3>,3> { };  
template<typename R,typename O,typename P1,typename P2,typename P3,typename P4> struct FunctionParams<R (O::*)(P1,P2,P3,P4)> : public MemFunctionParamsImpl<R,O,FunctionParamsArrayImpl<P1,P2,P3,P4>,4> { };  
template<typename R,typename O,typename P1,typename P2,typename P3,typename P4,typename P5> struct FunctionParams<R (O::*)(P1,P2,P3,P4,P5)> : public MemFunctionParamsImpl<R,O,FunctionParamsArrayImpl<P1,P2,P3,P4,P5>,5> { };  
  
  
GetFunction Impl/  
template<typename F> struct GetFunction : Type2Type<F> {};  
template<typename F> struct GetFunction<FunctionParams<F> > : Type2Type<F> {};  
  
  
///GetFunctionParam Impl//  
//e.g. get No x param of func F : GetFunctionParam<F,x>::type  
  
  
template<class F,int N=FunctionParams<F>::ParamNum> struct GetFunctionParamImpl { };  
template<class F> struct GetFunctionParamImpl<F,0>   
{   
protected:  
template<bool bMemberFunctionPointer> struct apply0;  
template<> struct apply0<true>   
{  
template<typename FunctorTester> struct apply : public Type2Type<typename FunctionParams<FunctorTester>::Object_T> {};  
};  
template<> struct apply0<false>   
{  
template<typename FunctorTester> struct apply {};  
};  
  
  
public:  
template<int idx> struct apply ;  
template<> struct apply<0> : public apply0<IsMemberFunctionPointer<F>::value>::template apply<F>{};   
};  
  
  
template<class F> struct GetFunctionParamImpl<F,1> : public GetFunctionParamImpl<F,0> { template<> struct apply<1> : Type2Type<typename FunctionParams<F>::TP1> {}; };  
template<class F> struct GetFunctionParamImpl<F,2> : public GetFunctionParamImpl<F,1> { template<> struct apply<2> : Type2Type<typename FunctionParams<F>::TP2> {}; };  
template<class F> struct GetFunctionParamImpl<F,3> : public GetFunctionParamImpl<F,2> { template<> struct apply<3> : Type2Type<typename FunctionParams<F>::TP3> {}; };  
template<class F> struct GetFunctionParamImpl<F,4> : public GetFunctionParamImpl<F,3> { template<> struct apply<4> : Type2Type<typename FunctionParams<F>::TP4> {}; };  
template<class F> struct GetFunctionParamImpl<F,5> : public GetFunctionParamImpl<F,4> { template<> struct apply<5> : Type2Type<typename FunctionParams<F>::TP5> {}; };  
  
  
template<class F,int idx> struct GetFunctionParam : public Type2Type<typename GetFunctionParamImpl<F>::template apply<idx>::type> {};  
  
  
/  
template<typename TP1=NullType,typename TP2=NullType,typename TP3=NullType,typename TP4=NullType,typename TP5=NullType,typename P6=NullType,typename P7=NullType> struct ParmHolder;  
template<> struct ParmHolder<>  
{  
};  
  
  
template<typename TP1> struct ParmHolder<TP1>  
{  
ParmHolder(TP1 p1) : m_p1(p1){};  
TP1 m_p1;  
};  
  
  
template<typename TP1,typename TP2> struct ParmHolder<TP1,TP2> : public ParmHolder<TP1>  
{  
ParmHolder(TP1 p1,TP2 p2) : ParmHolder<TP1>(p1) , m_p2(p2){};  
TP2 m_p2;  
};  
  
  
template<typename TP1,typename TP2,typename TP3> struct ParmHolder<TP1,TP2,TP3> : public ParmHolder<TP1,TP2>  
{  
ParmHolder(TP1 p1,TP2 p2,TP3 p3) : ParmHolder<TP1,TP2>(p1,p2) , m_p3(p3){};  
TP3 m_p3;  
};  
  
  
template<typename TP1,typename TP2,typename TP3,typename TP4> struct ParmHolder<TP1,TP2,TP3,TP4> : public ParmHolder<TP1,TP2,TP3>  
{  
ParmHolder(TP1 p1,TP2 p2,TP3 p3,TP4 p4) : ParmHolder<TP1,TP2,TP3>(p1,p2,p3) , m_p4(p4){};  
TP4 m_p4;  
};  
  
  
template<typename TP1,typename TP2,typename TP3,typename TP4,typename TP5> struct ParmHolder<TP1,TP2,TP3,TP4,TP5> : public ParmHolder<TP1,TP2,TP3,TP4>  
{  
ParmHolder(TP1 p1,TP2 p2,TP3 p3,TP4 p4,TP5 p5) : ParmHolder<TP1,TP2,TP3,TP4>(p1,p2,p3,p4) , m_p5(p5){};  
TP5 m_p5;  
};  
  
  
template<typename TP1,typename TP2,typename TP3,typename TP4,typename TP5,typename TP6> struct ParmHolder<TP1,TP2,TP3,TP4,TP5,TP6> : public ParmHolder<TP1,TP2,TP3,TP4,TP5>  
{  
ParmHolder(TP1 p1,TP2 p2,TP3 p3,TP4 p4,TP5 p5,TP5 p6) : ParmHolder<TP1,TP2,TP3,TP4,TP5>(p1,p2,p3,p4,p5) , m_p6(p6){};  
TP6 m_p6;  
};  
}  
#endif // functorTraits_h__

  

