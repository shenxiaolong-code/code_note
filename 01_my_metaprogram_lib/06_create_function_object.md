6) 创建一元函数/二元函数 functionCreater 

发布日期: 2014-02-12 11:32:56
原文链接: https://blog.csdn.net/Tonny0832/article/details/19112755

---

源代码可以在我的资源中去下载(包括用法测试的源代码):  
http://download.csdn.net/detail/tonny0832/7426991  
只需要用VS建一个空的工程，然后引入我的测试头文件即可。然后测试函数会在main函数之前运行，在控制台窗口中可以看到输出。  


  


这个文件是和将多元函数转换为一元函数unaryFunctionConverter配合使用的，unaryFunctionConverter提供了转换的实现，但是使用起来不便，此文件functionCreater将这些转换封装成了makeUnaryFunction/makeBinaryFunction以方便使用，使用者只需要提供函数指针及必要的传入参数即可创建正确的一元/二元函数对象、

  


#ifndef functorCreater_h__  
#define functorCreater_h__  
/********************************************************************  
created: 2014-1-9 15:35  
author: Shen Tony  
  
purpose: helper to create binary function , avoid writing complex template struct/class  
create interface to convert (global /member) function into unary function ,this function supports 5 parameters dynamical at most. e.g.  
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
  
  
UnaryFunction<F,x> : No.x parameter of functor F is input parameter in every call.  
FunctorParams<F>::Return_T is the return type of every calling  
*********************************************************************/  
#include "function.h"  
  
  
namespace UtilExt  
{  
/// binary function creator /  
template<typename TP1 ,typename TP2, typename R>  
BinaryFunction<R (*)(TP1,TP2)> makeBinaryFunction( R (*f)(TP1,TP2) )  
{  
return BinaryFunction<R (*)(TP1,TP2)>(f);  
}  
  
  
template<typename O,typename TP1 ,typename TP2, typename R>  
BinaryFunction<R (O::*)(TP1,TP2)> makeBinaryFunction(TPP(O) obj, R (O::*f)(TP1,TP2) )  
{  
return BinaryFunction<R (*)(TP1,TP2)>(obj,f);  
}  
  
  
/// unary function creator //  
template<typename F,int Idx> struct UnaryFunctionCreater;  
template<typename F> struct UnaryFunctionCreater<F,0> : public CreateUnaryFunctionImpl<F> { };  
template<typename F> struct UnaryFunctionCreater<F,1> : public CreateUnaryFunctionImpl<F> { };  
template<typename F> struct UnaryFunctionCreater<F,2> : public CreateUnaryFunctionImpl<F> { };  
template<typename F> struct UnaryFunctionCreater<F,3> : public CreateUnaryFunctionImpl<F> { };  
template<typename F> struct UnaryFunctionCreater<F,4> : public CreateUnaryFunctionImpl<F> { };  
template<typename F> struct UnaryFunctionCreater<F,5> : public CreateUnaryFunctionImpl<F> { };  
  
  
//member function  
template<typename R,typename O>   
static UnaryFunction<R(O::*)(),0> makeUnaryFunction(R(O::*f)(),_) { return UnaryFunctionCreater<R(O::*)(),0>::create(f,_$); };  
template<typename R,typename O,typename TP1>   
static UnaryFunction<R(O::*)(TP1),0> makeUnaryFunction(R(O::*f)(TP1),_,TPP(TP1) p1) { return UnaryFunctionCreater<R(O::*)(TP1),0>::create(f,_$,p1); };  
template<typename R,typename O,typename TP1,typename TP2>   
static UnaryFunction<R(O::*)(TP1,TP2),0> makeUnaryFunction(R(O::*f)(TP1,TP2),_,TPP(TP1) p1,TPP(TP2) p2) { return UnaryFunctionCreater<R(O::*)(TP1,TP2),0>::create(f,_$,p1,p2); };  
template<typename R,typename O,typename TP1,typename TP2,typename TP3>   
static UnaryFunction<R(O::*)(TP1,TP2,TP3),0> makeUnaryFunction(R(O::*f)(TP1,TP2,TP3),_,TPP(TP1) p1,TPP(TP2) p2,TPP(TP3) p3) { return UnaryFunctionCreater<R(O::*)(TP1,TP2,TP3),0>::create(f,_$,p1,p2,p3); };  
template<typename R,typename O,typename TP1,typename TP2,typename TP3,typename TP4>   
static UnaryFunction<R(O::*)(TP1,TP2,TP3,TP4),0> makeUnaryFunction(R(O::*f)(TP1,TP2,TP3,TP4),_,TPP(TP1) p1,TPP(TP2) p2,TPP(TP3) p3,TPP(TP4) p4) { return UnaryFunctionCreater<R(O::*)(TP1,TP2,TP3,TP4),0>::create(f,_$,p1,p2,p3,p4); };  
template<typename R,typename O,typename TP1,typename TP2,typename TP3,typename TP4,typename TP5>   
static UnaryFunction<R(O::*)(TP1,TP2,TP3,TP4,TP5),0> makeUnaryFunction(R(O::*f)(TP1,TP2,TP3,TP4,TP5),_,TPP(TP1) p1,TPP(TP2) p2,TPP(TP3) p3,TPP(TP4) p4,TPP(TP5) p5) { return UnaryFunctionCreater<R(O::*)(TP1,TP2,TP3,TP4,TP5),0>::create(f,_$,p1,p2,p3,p4,p5); };  
  
  
template<typename R,typename O,typename TP1>   
static UnaryFunction<R(O::*)(TP1),1> makeUnaryFunction(R(O::*f)(TP1),TPP(O) p0,_) { return UnaryFunctionCreater<R (O::*)(TP1),1>::create(f,p0,_$); };  
template<typename R,typename O,typename TP1,typename TP2>   
static UnaryFunction<R(O::*)(TP1,TP2),1> makeUnaryFunction(R(O::*f)(TP1,TP2),TPP(O) p0,_,TPP(TP2) p2) { return UnaryFunctionCreater<R(O::*)(TP1,TP2),1>::create(f,p0,_$,p2); };  
template<typename R,typename O,typename TP1,typename TP2,typename TP3>   
static UnaryFunction<R(O::*)(TP1,TP2,TP3),1> makeUnaryFunction(R(O::*f)(TP1,TP2,TP3),TPP(O) p0,_,TPP(TP2) p2,TPP(TP3) p3) { return UnaryFunctionCreater<R(O::*)(TP1,TP2,TP3),1>::create(f,p0,_$,p2,p3); };  
template<typename R,typename O,typename TP1,typename TP2,typename TP3,typename TP4>   
static UnaryFunction<R(O::*)(TP1,TP2,TP3,TP4),1> makeUnaryFunction(R(O::*f)(TP1,TP2,TP3,TP4),TPP(O) p0,_,TPP(TP2) p2,TPP(TP3) p3,TPP(TP4) p4) { return UnaryFunctionCreater<R(O::*)(TP1,TP2,TP3,TP4),1>::create(f,p0,_$,p2,p3,p4); };   
template<typename R,typename O,typename TP1,typename TP2,typename TP3,typename TP4,typename TP5>   
static UnaryFunction<R(O::*)(TP1,TP2,TP3,TP4,TP5),1> makeUnaryFunction(R(O::*f)(TP1,TP2,TP3,TP4,TP5),TPP(O) p0,_,TPP(TP2) p2,TPP(TP3) p3,TPP(TP4) p4,TPP(TP5) p5) { return UnaryFunctionCreater<R(O::*)(TP1,TP2,TP3,TP4,TP5),1>::create(f,p0,_$,p2,p3,p4,p5); };  
  
  
template<typename R,typename O,typename TP1,typename TP2>   
static UnaryFunction<R(O::*)(TP1,TP2),2> makeUnaryFunction(R(O::*f)(TP1,TP2),TPP(O) p0,TPP(TP1) p1,_) { return UnaryFunctionCreater<R(O::*)(TP1,TP2),2>::create(f,p0,p1,_$); };  
template<typename R,typename O,typename TP1,typename TP2,typename TP3>   
static UnaryFunction<R(O::*)(TP1,TP2,TP3),2> makeUnaryFunction(R(O::*f)(TP1,TP2,TP3),TPP(O) p0,TPP(TP1) p1,_,TPP(TP3) p3) { return UnaryFunctionCreater<R(O::*)(TP1,TP2,TP3),2>::create(f,p0,p1,_$,p3); };  
template<typename R,typename O,typename TP1,typename TP2,typename TP3,typename TP4>   
static UnaryFunction<R(O::*)(TP1,TP2,TP3,TP4),2> makeUnaryFunction(R(O::*f)(TP1,TP2,TP3,TP4),TPP(O) p0,TPP(TP1) p1,_,TPP(TP3) p3,TPP(TP4) p4) { return UnaryFunctionCreater<R(O::*)(TP1,TP2,TP3,TP4),2>::create(f,p0,p1,_$,p3,p4); };   
template<typename R,typename O,typename TP1,typename TP2,typename TP3,typename TP4,typename TP5>   
static UnaryFunction<R(O::*)(TP1,TP2,TP3,TP4,TP5),2> makeUnaryFunction(R(O::*f)(TP1,TP2,TP3,TP4,TP5),TPP(O) p0,TPP(TP1) p1,_,TPP(TP3) p3,TPP(TP4) p4,TPP(TP5) p5) { return UnaryFunctionCreater<R(O::*)(TP1,TP2,TP3,TP4,TP5),2>::create(f,p0,p1,_$,p3,p4,p5); };  
  
  
template<typename R,typename O,typename TP1,typename TP2,typename TP3>   
static UnaryFunction<R(O::*)(TP1,TP2,TP3),3> makeUnaryFunction(R(O::*f)(TP1,TP2,TP3),TPP(O) p0,TPP(TP1) p1,TPP(TP2) p2,_) { return UnaryFunctionCreater<R(O::*)(TP1,TP2,TP3),3>::create(f,p0,p1,p2,_$); };  
template<typename R,typename O,typename TP1,typename TP2,typename TP3,typename TP4>   
static UnaryFunction<R(O::*)(TP1,TP2,TP3,TP4),3> makeUnaryFunction(R(O::*f)(TP1,TP2,TP3,TP4),TPP(O) p0,TPP(TP1) p1,TPP(TP2) p2,_,TPP(TP4) p4) { return UnaryFunctionCreater<R(O::*)(TP1,TP2,TP3,TP4),3>::create(f,p0,p1,p2,_$,p4); };   
template<typename R,typename O,typename TP1,typename TP2,typename TP3,typename TP4,typename TP5>   
static UnaryFunction<R(O::*)(TP1,TP2,TP3,TP4,TP5),3> makeUnaryFunction(R(O::*f)(TP1,TP2,TP3,TP4,TP5),TPP(O) p0,TPP(TP1) p1,TPP(TP2) p2,_,TPP(TP4) p4,TPP(TP5) p5) { return UnaryFunctionCreater<R(O::*)(TP1,TP2,TP3,TP4,TP5),3>::create(f,p0,p1,p2,_$,p4,p5); };  
  
  
template<typename R,typename O,typename TP1,typename TP2,typename TP3,typename TP4>   
static UnaryFunction<R(O::*)(TP1,TP2,TP3,TP4),4> makeUnaryFunction(R(O::*f)(TP1,TP2,TP3,TP4),TPP(O) p0,TPP(TP1) p1,TPP(TP2) p2,TPP(TP3) p3,_) { return UnaryFunctionCreater<R(O::*)(TP1,TP2,TP3,TP4),4>::create(f,p0,p1,p2,p3,_$); };   
template<typename R,typename O,typename TP1,typename TP2,typename TP3,typename TP4,typename TP5>   
static UnaryFunction<R(O::*)(TP1,TP2,TP3,TP4,TP5),4> makeUnaryFunction(R(O::*f)(TP1,TP2,TP3,TP4,TP5),TPP(O) p0,TPP(TP1) p1,TPP(TP2) p2,TPP(TP3) p3,_,TPP(TP5) p5) { return UnaryFunctionCreater<R(O::*)(TP1,TP2,TP3,TP4,TP5),4>::create(f,p0,p1,p2,p3,_$,p5); };  
  
  
template<typename R,typename O,typename TP1,typename TP2,typename TP3,typename TP4,typename TP5>   
static UnaryFunction<R(O::*)(TP1,TP2,TP3,TP4,TP5),5> makeUnaryFunction(R(O::*f)(TP1,TP2,TP3,TP4,TP5),TPP(O) p0,TPP(TP1) p1,TPP(TP2) p2,TPP(TP3) p3,TPP(TP4) p4,_) { return UnaryFunctionCreater<R(O::*)(TP1,TP2,TP3,TP4,TP5),5>::create(f,p0,p1,p2,p3,p4,_$); };  
  
  
//non-member function  
template<typename R,typename TP1>   
static UnaryFunction<R(*)(TP1),1> makeUnaryFunction(R(*f)(TP1),_) { return UnaryFunctionCreater<R(*)(TP1),1>::create(f,_$); };  
template<typename R,typename TP1,typename TP2>   
static UnaryFunction<R(*)(TP1,TP2),1> makeUnaryFunction(R(*f)(TP1,TP2),_,TPP(TP2) p2) { return UnaryFunctionCreater<R(*)(TP1,TP2),1>::create(f,_$,p2); };  
template<typename R,typename TP1,typename TP2,typename TP3>   
static UnaryFunction<R(*)(TP1,TP2,TP3),1> makeUnaryFunction(R(*f)(TP1,TP2,TP3),_,TPP(TP2) p2,TPP(TP3) p3) { return UnaryFunctionCreater<R(*)(TP1,TP2,TP3),1>::create(f,_$,p2,p3); };  
template<typename R,typename TP1,typename TP2,typename TP3,typename TP4>   
static UnaryFunction<R(*)(TP1,TP2,TP3,TP4),1> makeUnaryFunction(R(*f)(TP1,TP2,TP3,TP4),_,TPP(TP2) p2,TPP(TP3) p3,TPP(TP4) p4) { return UnaryFunctionCreater<R(*)(TP1,TP2,TP3,TP4),1>::create(f,_$,p2,p3,p4); };   
template<typename R,typename TP1,typename TP2,typename TP3,typename TP4,typename TP5>   
static UnaryFunction<R(*)(TP1,TP2,TP3,TP4,TP5),1> makeUnaryFunction(R(*f)(TP1,TP2,TP3,TP4,TP5),_,TPP(TP2) p2,TPP(TP3) p3,TPP(TP4) p4,TPP(TP5) p5) { return UnaryFunctionCreater<R(*)(TP1,TP2,TP3,TP4,TP5),1>::create(f,_$,p2,p3,p4,p5); };  
  
  
template<typename R,typename TP1,typename TP2>   
static UnaryFunction<R(*)(TP1,TP2),2> makeUnaryFunction(R(*f)(TP1,TP2),TPP(TP1) p1,_) { return UnaryFunctionCreater<R(*)(TP1,TP2),2>::create(f,p1,_$); };  
template<typename R,typename TP1,typename TP2,typename TP3>   
static UnaryFunction<R(*)(TP1,TP2,TP3),2> makeUnaryFunction(R(*f)(TP1,TP2,TP3),TPP(TP1) p1,_,TPP(TP3) p3) { return UnaryFunctionCreater<R(*)(TP1,TP2,TP3),2>::create(f,p1,_$,p3); };  
template<typename R,typename TP1,typename TP2,typename TP3,typename TP4>   
static UnaryFunction<R(*)(TP1,TP2,TP3,TP4),2> makeUnaryFunction(R(*f)(TP1,TP2,TP3,TP4),TPP(TP1) p1,_,TPP(TP3) p3,TPP(TP4) p4) { return UnaryFunctionCreater<R(*)(TP1,TP2,TP3,TP4),2>::create(f,p1,_$,p3,p4); };   
template<typename R,typename TP1,typename TP2,typename TP3,typename TP4,typename TP5>   
static UnaryFunction<R(*)(TP1,TP2,TP3,TP4,TP5),2> makeUnaryFunction(R(*f)(TP1,TP2,TP3,TP4,TP5),TPP(TP1) p1,_,TPP(TP3) p3,TPP(TP4) p4,TPP(TP5) p5) { return UnaryFunctionCreater<R(*)(TP1,TP2,TP3,TP4,TP5),2>::create(f,p1,_$,p3,p4,p5); };  
  
  
template<typename R,typename TP1,typename TP2,typename TP3>   
static UnaryFunction<R(*)(TP1,TP2,TP3),3> makeUnaryFunction(R(*f)(TP1,TP2,TP3),TPP(TP1) p1,TPP(TP2) p2,_) { return UnaryFunctionCreater<R(*)(TP1,TP2,TP3),3>::create(f,p1,p2,_$); };  
template<typename R,typename TP1,typename TP2,typename TP3,typename TP4>   
static UnaryFunction<R(*)(TP1,TP2,TP3,TP4),3> makeUnaryFunction(R(*f)(TP1,TP2,TP3,TP4),TPP(TP1) p1,TPP(TP2) p2,_,TPP(TP4) p4) { return UnaryFunctionCreater<R(*)(TP1,TP2,TP3,TP4),3>::create(f,p1,p2,_$,p4); };   
template<typename R,typename TP1,typename TP2,typename TP3,typename TP4,typename TP5>   
static UnaryFunction<R(*)(TP1,TP2,TP3,TP4,TP5),3> makeUnaryFunction(R(*f)(TP1,TP2,TP3,TP4,TP5),TPP(TP1) p1,TPP(TP2) p2,_,TPP(TP4) p4,TPP(TP5) p5) { return UnaryFunctionCreater<R(*)(TP1,TP2,TP3,TP4,TP5),3>::create(f,p1,p2,_$,p4,p5); };  
  
  
template<typename R,typename TP1,typename TP2,typename TP3,typename TP4>   
static UnaryFunction<R(*)(TP1,TP2,TP3,TP4),4> makeUnaryFunction(R(*f)(TP1,TP2,TP3,TP4),TPP(TP1) p1,TPP(TP2) p2,TPP(TP3) p3,_) { return UnaryFunctionCreater<R(*)(TP1,TP2,TP3,TP4),4>::create(f,p1,p2,p3,_$); };   
template<typename R,typename TP1,typename TP2,typename TP3,typename TP4,typename TP5>   
static UnaryFunction<R(*)(TP1,TP2,TP3,TP4,TP5),4> makeUnaryFunction(R(*f)(TP1,TP2,TP3,TP4,TP5),TPP(TP1) p1,TPP(TP2) p2,TPP(TP3) p3,_,TPP(TP5) p5) { return UnaryFunctionCreater<R(*)(TP1,TP2,TP3,TP4,TP5),4>::create(f,p1,p2,p3,_$,p5); };  
  
  
template<typename R,typename TP1,typename TP2,typename TP3,typename TP4,typename TP5>   
static UnaryFunction<R(*)(TP1,TP2,TP3,TP4,TP5),5> makeUnaryFunction(R(*f)(TP1,TP2,TP3,TP4,TP5),TPP(TP1) p1,TPP(TP2) p2,TPP(TP3) p3,TPP(TP4) p4,_) { return UnaryFunctionCreater<R(*)(TP1,TP2,TP3,TP4,TP5),5>::create(f,p1,p2,p3,p4,_$); };  
  
  
}  
  
  
#endif // functorCreater_h__  

