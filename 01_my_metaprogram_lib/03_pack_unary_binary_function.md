3) 一元/二元函数抽象function 

发布日期: 2014-02-12 10:36:48
原文链接: https://blog.csdn.net/Tonny0832/article/details/19110443

---

源代码可以在我的资源中去下载(包括用法测试的源代码):  
http://download.csdn.net/detail/tonny0832/7426991  
只需要用VS建一个空的工程，然后引入我的测试头文件即可。然后测试函数会在main函数之前运行，在控制台窗口中可以看到输出。  


  


function/function object是运行时调用的基础，在模板中，最常用的函数是一元函数及二元函数。

通常一元函数R (*)(TP)是遍历的组件，如果R是bool类型或者进行二次封装（把R由非bool转换为bool），则一元函数R (*)(P)还可以成为测试组件，用于if的条件。

通常二元函数是R (*)(TP1,TP2)是比较/交换的组件，用于比较两个值，其用于if的条件，或者用于交换两个量swap/swapIf。

此实现现实了一元函数/二元函数的抽象，并且其中的一元函数依赖unaryFunctionConverter.h，该组件实现了把一个非一元函数(1个参数，最多5个参数)的多元函数转换为一元函数的功能，可以参见本博客文章中的unaryFunctionConverter(4)了解其细节。

  


本源代码不提供将多元函数转换为二元函数的功能，设计者若使用此抽象，传入参数必须是二元函数。

  


#ifndef function_h__  
#define function_h__  
/********************************************************************  
created: 2014-1-16 14:27  
author: Shen Tony  
  
purpose: define unary function and binary function  
*********************************************************************/  
/********************************************************************  
Description : define unary function and binary function  
unary function is very usable and you can use unary function creator to   
convert one multiple parameter function into unary function.  
  
  
Author : Shen.Xiaolong (Shen Tony) (2010-2013)  
Mail : xlshen2002@hotmail.com, xlshen@126.com  
verified platform : VS2008  
copyright: : latest Version of The Code Project Open License (CPOL : http://www.codeproject.com/)  
*********************************************************************/  
#include "unaryFunctionConverter.h"  
  
  
namespace UtilExt  
{  
namespace PrivateHelper  
{  
template<typename F,typename R=NullType>   
struct FunctionReturnImpl : public Type2Type<typename FunctionParams<F>::Return_T> {};  
template<typename F>   
struct FunctionReturnImpl<F,typename sfinae_helper<typename F::Return_T>::type> : public Type2Type<typename F::Return_T> {};  
}  
  
  
template<typename F> struct GetFunctionReturnType: public PrivateHelper::FunctionReturnImpl<F> {};  
/unary function and unary function converter///  
  
  
template<typename P, typename R>  
struct UnaryFunctionBase  
{  
typedef P Param_T;  
typedef R Return_T;  
};  
  
  
template<typename F,int Idx> struct UnaryFunction : public UnaryFunctionBase<FPP(F,Idx),FR(F)>   
{  
typedef typename InitParams<F,Idx>::type InitParams_T;  
typedef CreateUnaryFunctionImpl<F> Invoker_T;  
  
  
UnaryFunction(F f,InitParams_T& params)  
: m_f(f)  
, m_Ps(params)  
{ }  
  
  
Return_T operator()(Param_T obj)  
{   
return Invoker_T::call(*this,obj);  
}  
  
  
Return_T operator()(typename RemoveRef<Param_T>::type* pObj)  
{   
return Invoker_T::call(*this,*pObj);  
}  
  
InitParams_T m_Ps;  
F m_f;  
};  
  
  
namespace PrivateHelper  
{  
template<typename F,typename P=NullType>   
struct FunctionParamImpl : public Type2Type<typename UnaryFunction<F,Evl_If_T<IsMemberFunctionPointer<F>,Int2Type<0>,Int2Type<1> >::type::value>::Param_T> {};  
template<typename F>   
struct FunctionParamImpl<F,typename sfinae_helper<typename F::Param_T>::type> : public Type2Type<typename F::Param_T> {};  
}  
  
  
template<typename F> struct GetUnaryFunctionParam: public PrivateHelper::FunctionParamImpl<F> {};  
  
  
binary function/  
template<typename P1,typename P2, typename R>  
struct BinaryFunctionBase  
{  
typedef P1 TP1;  
typedef P2 TP2;  
typedef R Return_T;  
};  
  
  
template<typename F, bool bMemberFunctionPointer=IsMemberFunctionPointer<F>::value>   
struct BinaryFunction;  
  
  
template<typename F>  
struct BinaryFunction<F,false> : public BinaryFunctionBase<FPP(F,1),FPP(F,2),FR(F)>  
{   
BinaryFunction(F f)  
: m_f(f)  
{  
Static_Assert(2==FunctionParams<F>::ParamNum);  
}  
  
  
Return_T operator()(TP1 p1,TP2 p2)  
{   
return m_f(p1,p2);  
}  
  
  
F m_f;  
};  
  
  
template<typename F>  
struct BinaryFunction<F,true> : public BinaryFunctionBase<FPP(F,1),FPP(F,2),FR(F)>  
{   
typedef TPP(FO(F)) Object_T;  
  
  
BinaryFunction(Object_T obj,F f)  
: m_obj(obj)  
, m_f(f)  
{  
Static_Assert(2==FunctionParams<F>::ParamNum);  
}  
  
  
Return_T operator()(TP1 p1,TP2 p2)  
{   
return (m_obj.*m_f)(p1,p2);  
}  
  
  
Object_T m_obj;  
F m_f;  
};  
  
  
namespace PrivateHelper  
{  
template<typename F,typename P=NullType>   
struct GBFP_helper : public Type2Type<BinaryFunction<F> > {};  
template<typename F>   
struct GBFP_helper<F,typename sfinae_helper<typename F::TP1>::type> : public Type2Type<F> {};  
}  
  
  
template<typename F,unsigned Idx> struct GetBinaryFunctionParam;  
template<typename F> struct GetBinaryFunctionParam<F,1> : public Type2Type<typename PrivateHelper::GBFP_helper<F>::type::TP1> {};  
template<typename F> struct GetBinaryFunctionParam<F,2> : public Type2Type<typename PrivateHelper::GBFP_helper<F>::type::TP2> {};  
}  
  
  
#endif // function_h__  

