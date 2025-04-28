9) 常用函数对象 functionobject 

发布日期: 2013-10-07 13:25:50
原文链接: https://blog.csdn.net/Tonny0832/article/details/12379299

---

源代码可以在我的资源中去下载(包括用法测试的源代码):

<http://download.csdn.net/detail/tonny0832/7426991>

只需要用VS建一个空的工程，然后引入我的测试头文件即可。然后测试函数会在main函数之前运行，在控制台窗口中可以看到输出。  
  


在使用STL算法时，第三个参数通常是一些函数对象。这样，我们为了使用STL提供的算法函数，就不得不提供一些零碎的，小的函数对象，有可能这些函数对象仅仅这只这一处使用，我们就不得不写这样一个函数对象类。大多数时候，这个函数对象提供的功能可能非常简单，比如说就是根据结构体里的某个子结构体的某个子成员值相比较，或者满足某种大小的需求，这类需求功能从概念上讲是一样的，但是实现起来，就不得不一一书写这些琐碎的函数对象。本泛型代码基于概念，实现了泛型的这些函数对象，用户可以在大部分情况下避免写自己的函数对象。  


#ifndef functionObject_h__  
#define functionObject_h__  
/********************************************************************  
Description : pack function as object, which is useful in STL algorithm .  
Author : Shen.Xiaolong (Shen Tony) (2010-2013)  
Mail : xlshen2002@hotmail.com, xlshen@126.com  
verified platform : VS2008  
copyright: : latest Version of The Code Project Open License (CPOL : http://www.codeproject.com/)  
*********************************************************************/  
#include "libconfig.h"  
#include "macroDef.h"  
#include "mathOperator.h"  
#include "typeconvert.h"  
#include "dataSet.h"  
#include "typeTraits.h"  
#include "unaryFunctionConverter.h"  
#include "memberPtr.h"  
#include "functionTraits.h"  
#include "function.h"  
  
  
namespace UtilExt  
{  
///Comparer implement //  
//it is used to compare same type data,such as in one sequence / container / array  
template<typename L,typename R,typename R_T=bool>  
struct Comparer  
{  
typedef L TP1;  
typedef R TP2;  
typedef R_T Return_T;  
};  
  
  
template<typename Comparer_T>  
struct NotComparor : public Comparer<typename GetBinaryFunctionParam<Comparer_T,1>::type,typename GetBinaryFunctionParam<Comparer_T,2>::type>  
{  
NotComparor(Comparer_T& rCmp)  
: m_cmpImpl(rCmp)  
{}  
  
template<typename TL,typename TR>  
Return_T operator()(TL& p1,TR& p2)  
{  
return !m_cmpImpl(p1,p2);  
}  
  
  
Comparer_T& m_cmpImpl;  
};  
template<typename Comparer_T>  
NotComparor<Comparer_T> makeComparerNot(Comparer_T& rCpr)  
{  
return NotComparor<Comparer_T>(rCpr);  
}  
  
  
namespace CGetType  
{  
namespace Comparer  
{  
template<typename Comparer_T>  
struct applyByNot : public Type2Type<NotComparor<Comparer_T> > {};  
}  
}  
  
template<typename BF_T>  
struct ComparerImpl : public Comparer<typename GetBinaryFunctionParam<BF_T,1>::type,typename GetBinaryFunctionParam<BF_T,2>::type>  
{  
explicit ComparerImpl(BF_T& f)  
: m_f(f)   
{  
AssertXI(f.m_f,("Function pointer is NULL."));  
};  
  
  
template<typename TL,typename TR>  
Return_T operator()(TL& p1,TR& p2)  
{  
return m_f(p1,p2);  
}  
  
  
private:  
BF_T m_f;  
};  
  
  
template<typename BF_T>  
ComparerImpl<BF_T> makeComparer(BF_T& f)  
{  
TRACE_COMPILE();  
return ComparerImpl<BF_T>(f);  
}  
  
  
namespace CGetType  
{  
namespace Comparer  
{  
template<typename BF_T>  
struct applyByBinaryFunction : public Type2Type<ComparerImpl<BF_T> > {};  
}  
}  
  
  
template<typename MemPtr_T,typename BF_T>  
struct ComparerByMember : public Comparer<  
typename SameConst<typename GetInputParam<MemPtr_T>::type,typename GetBinaryFunctionParam<BF_T,1>::type>::type,  
typename SameConst<typename GetInputParam<MemPtr_T>::type,typename GetBinaryFunctionParam<BF_T,2>::type>::type  
>  
{  
explicit ComparerByMember(MemPtr_T& member,BF_T& f)  
: m_member(member) , m_f(f) //f MUST be function object, instead of function pointer, else it cause C2436 compile error  
{};  
  
  
template<typename TL,typename TR>  
Return_T operator()(TL& p1,TR& p2)  
{  
return m_f(m_member(p1),m_member(p2));  
}  
  
  
protected:  
BF_T m_f;  
MemPtr_T m_member;  
};  
  
  
template<typename MemberPtr_T,typename BF_T>  
ComparerByMember<MemberPtr_T,BF_T> makeComparer(MemberPtr_T& member,BF_T& f)  
{  
Static_Assert((IsSameRawType<typename GetMember<MemberPtr_T>::type,typename GetBinaryFunctionParam<BF_T,1>::type>::value));  
Static_Assert((IsSameRawType<typename GetMember<MemberPtr_T>::type,typename GetBinaryFunctionParam<BF_T,2>::type>::value));  
return ComparerByMember<MemberPtr_T,BF_T>(member,f);  
}   
  
  
namespace CGetType  
{  
namespace Comparer  
{  
template<typename MemberPtr_T,typename BF_T>  
struct applyByMemberCmp : public Type2Type<ComparerByMember<MemberPtr_T,BF_T> > {};  
}  
}  
  
  
/  
/  
template<typename D> //note : D can be const and non-const.(e.g. D can be int or const int)  
struct UnaryTester  
{  
typedef D Param_T;  
};  
  
  
template<typename UF_T,typename R_T=typename GetFunctionReturnType<UF_T>::type>  
struct UnaryTesterImpl : public UnaryTester< typename GetUnaryFunctionParam<UF_T>::type >  
{  
typedef R_T Return_T;  
  
  
UnaryTesterImpl(UF_T& f,Return_T retChk)  
: m_f(f)  
, m_ret(retChk)  
{ }  
  
  
template<typename TP> bool operator()(TP& obj)  
{  
return m_ret == m_f(obj);  
}  
  
  
UF_T m_f;  
Return_T m_ret;  
};  
  
  
template<typename UF_T>  
UnaryTesterImpl<UF_T> makeTester(UF_T& f,TPP(typename GetFunctionReturnType<UF_T>::type) ret)  
{  
return UnaryTesterImpl<UF_T>(f,ret);  
}  
  
  
template<typename T,typename R>  
UnaryTesterImpl<R (*)(T)> makeTester(R (*f)(T),TPP(R) ret)  
{  
typedef R (*F)(T);  
return UnaryTesterImpl<F>(f,ret);  
}   
  
  
namespace CGetType  
{  
namespace UnaryTester  
{  
template<typename UF_T>  
struct applyByUnaryFunction : public Type2Type<UnaryTesterImpl<UF_T> > {};  
  
  
template<typename T,typename R>   
struct applyByUnaryFunction<R (*)(T)> : public Type2Type<UnaryTesterImpl<R (*)(T)> > {};  
}  
}  
  
  
template<typename MemPtr_T,typename UF_T,typename R_T=typename GetFunctionReturnType<UF_T>::type>  
struct UnaryTesterByMember   
: public UnaryTester<typename SameConst<typename GetInputParam<MemPtr_T>::type,typename GetUnaryFunctionParam<UF_T>::type>::type>  
{  
typedef R_T Return_T;  
  
  
explicit UnaryTesterByMember(MemPtr_T& member,UF_T& f,Return_T ret)  
: m_member(member) , m_f(f) , m_ret(ret)  
{};  
  
  
template<typename TP> bool operator()(TP& obj)  
{  
return m_ret == m_f(m_member(obj));  
}  
  
  
protected:   
Return_T m_ret;  
MemPtr_T m_member;  
UF_T m_f;   
};  
  
  
template<typename MemPtr_T,typename UF_T>  
UnaryTesterByMember<MemPtr_T,UF_T> makeTester(MemPtr_T& member,UF_T& f,TPP(typename GetFunctionReturnType<UF_T>::type) retChk)  
{  
return UnaryTesterByMember<MemPtr_T,UF_T>(member,f,retChk);  
}  
  
  
template<typename T,typename R,typename MemPtr_T>  
UnaryTesterByMember<MemPtr_T,UnaryFunction<R (*)(T),1> > makeTester(MemPtr_T& member,R (*pF)(T),R retChk)  
{  
return makeTester(member,UnaryFunction<bool(*)(T),1>(pF),retChk);   
}  
  
  
namespace CGetType  
{  
namespace UnaryTester  
{  
template<typename MemPtr_T,typename UF_T>  
struct applyByMemberCompare : public Type2Type<UnaryTesterByMember<MemPtr_T,UF_T> > {};  
  
  
template<typename MemPtr_T,typename T,typename R>  
struct applyByMemberCompare<MemPtr_T,R (*)(T)>: public Type2Type<UnaryTesterByMember<MemPtr_T,UnaryFunction<R (*)(T),1> > > {};  
}  
}  
  
  
template<typename T1,typename T2,bool bSameRawType=IsSameRawType<T1,T2>::value,  
bool bSameConst = IsConst<T1>::value==IsConst<T2>::value, bool bSameRef = IsRef<T1>::value == IsRef<T2>::value>  
struct SelectCommonParam;   
template<typename T1,typename T2> struct SelectCommonParam<T1,T2,true,true,true> : Type2Type<T1> {};  
template<typename T1,typename T2> struct SelectCommonParam<T1,T2,true,false,false> : Type2Type<T1> {};  
template<typename T1,typename T2> struct SelectCommonParam<T1,T2,true,true,false> : Type2Type<typename AddRef<T1>::type> {};  
template<typename T1,typename T2> struct SelectCommonParam<T1,T2,true,false,true> : Type2Type<typename RemoveConst<T1>::type> {};  
  
  
template<typename Condition_T1,typename Condition_T2>  
struct UnaryTesterByMultiCondition :   
public UnaryTester<typename SelectCommonParam<  
typename GetUnaryFunctionParam<Condition_T1>::type,  
typename GetUnaryFunctionParam<Condition_T2>::type>::type>  
{  
UnaryTesterByMultiCondition(Condition_T1& condi1,Condition_T2& condi2)  
: m_condi1(condi1) , m_condi2(condi2) {};  
  
  
template<typename TP> bool operator()(TP& obj)  
{  
return m_condi1(obj) && m_condi2(obj);  
}  
  
  
Condition_T1 m_condi1;  
Condition_T2 m_condi2;  
};  
  
  
template<typename Condition_T1,typename Condition_T2>  
UnaryTesterByMultiCondition<Condition_T1,Condition_T2> makeTester(Condition_T1& rCondi1,Condition_T2& rCondi2)  
{  
return UnaryTesterByMultiCondition<Condition_T1,Condition_T2>(rCondi1,rCondi2);  
}  
  
  
namespace CGetType  
{  
namespace UnaryTester  
{  
template<typename Condition_T1,typename Condition_T2>  
struct applyByMultiCondition : public Type2Type<UnaryTesterByMultiCondition<Condition_T1,Condition_T2> > {};  
}  
}  
  
  
template<typename D1,typename D2, typename MemberPtr_T>  
UnaryTesterByMultiCondition<UnaryTesterByMember<MemberPtr_T,UnaryFunction<bool (*)(D1,D2),1> >,  
UnaryTesterByMember<MemberPtr_T,UnaryFunction<bool (*)(D1,D2),2> > >  
makeRangeTester(MemberPtr_T& rMember,TPP(D1) vMin,TPP(D2) vMax,bool (*pGreater)(D1,D2)/*=UtilExt::greater<D1,D2>*/)  
{  
return makeTester(makeTester(rMember,makeUnaryFunction(pGreater,_$,vMin),true),makeTester(rMember,makeUnaryFunction(pGreater,vMax,_$),true));  
}  
  
  
template<typename D1,typename D2>  
UnaryTesterByMultiCondition<UnaryTesterImpl<UnaryFunction<bool (*)(D1,D2),1> >,  
UnaryTesterImpl<UnaryFunction<bool (*)(D1,D2),2> > >  
makeRangeTester(TPP(D1) vMin,TPP(D2) vMax,bool (*pGreater)(D1,D2)/*=UtilExt::greater<D1,D2>*/)  
{  
return makeTester(makeTester(makeUnaryFunction(pGreater,_$,vMin),true),makeTester(makeUnaryFunction(pGreater,vMax,_$),true));  
}  
  
  
namespace CGetType  
{  
namespace UnaryTester  
{  
template<typename Condition_T1,typename Condition_T2> struct applyByRange ;  
  
  
template<typename D1,typename D2, typename MemberPtr_T>  
struct applyByRange<MemberPtr_T,bool (*)(D1,D2)> : public applyByMultiCondition<  
UnaryTesterByMember<MemberPtr_T,UnaryFunction<bool (*)(D1,D2),1> >,  
UnaryTesterByMember<MemberPtr_T,UnaryFunction<bool (*)(D1,D2),2> > > {};  
  
  
template<typename D1,typename D2>  
struct applyByRange<bool (*)(D1,D2),_> : public applyByMultiCondition<  
UnaryTesterImpl<UnaryFunction<bool (*)(D1,D2),1> >,  
UnaryTesterImpl<UnaryFunction<bool (*)(D1,D2),2> > > {};  
}  
}   
  
  
//  
template<typename IF_T, typename DO_T>  
struct IfDO  
{  
IfDO(IF_T& rIf,DO_T& rDo)  
: m_if(rIf)  
, m_do(rDo)  
{ };  
  
  
template<typename Param_T> bool operator()(Param_T& obj)  
{  
if (m_if(obj))  
{  
m_do(obj);  
return true;  
}  
  
  
return false;  
}  
  
  
IF_T& m_if;  
DO_T& m_do;  
};  
  
  
template<typename IF_T, typename DO_T>  
IfDO<IF_T,DO_T> makeIfDO(IF_T& rIf,DO_T& rDo)  
{  
return IfDO<IF_T,DO_T>(rIf,rDo);  
}  
  
  
///Collector///  
template<typename C,typename MemberPtr_T>  
struct Collector  
{  
typedef typename CGetType::DataSet::apply<C>::type DataSet_T;  
typedef typename DataSet_T::value_type value_type;  
Collector(DataSet_T& rResult,MemberPtr_T& memPtr)  
: m_rResult(rResult)  
, m_memPtr(memPtr)  
{  
}  
  
  
template<typename TP>  
void operator()(TP& obj)  
{  
saveResult(m_memPtr(obj));  
}  
  
  
DataSet_T& getResult()  
{  
return m_rResult;  
}  
  
  
protected:  
template <typename DataSet_T,typename Obj_T,  
bool bPtr1=IsPointer<typename DataSet_T::value_type>::value,  
bool bPtr2=IsPointer<Obj_T>::value>  
struct Saver  
{  
void apply(DataSet_T& rArr,Obj_T& p)  
{  
rArr.push_back(p);  
}  
};  
  
  
template <typename DataSet_T,typename Obj_T>  
struct Saver<DataSet_T,Obj_T,true,false>  
{  
void apply(DataSet_T& rArr,Obj_T& p)  
{  
rArr.push_back(&p);  
}  
};  
  
  
template <typename DataSet_T,typename Obj_T>  
struct Saver<DataSet_T,Obj_T,false,true>  
{  
void apply(DataSet_T& rArr,Obj_T& p)  
{  
rArr.push_back(*p);  
}  
};  
  
  
template<typename TP>  
void saveResult(TP& obj)  
{  
Saver<DataSet_T,TP> hlp;  
hlp.apply(m_rResult,obj);  
}  
  
  
MemberPtr_T m_memPtr;  
DataSet_T m_rResult;  
};  
  
  
template<typename C,typename MemberPtr_T>  
Collector<C,MemberPtr_T> makeCollector(C& rArr,MemberPtr_T& rMemPtr)  
{  
return Collector<C,MemberPtr_T>(makeResult(rArr),rMemPtr);  
}  
  
  
//  
/Apply object/  
FinderImpl implement /  
template<typename Tester_T>  
struct Finder  
{  
typedef typename GetUnaryFunctionParam<Tester_T>::type Param_T;  
typedef typename AddPointer<typename RemoveRef<Param_T>::type>::type Result_T;  
  
  
Finder(Tester_T& rTester)  
: m_curObj(NULL)  
, m_tester(rTester)  
{};  
  
  
template<typename TP>  
bool operator()(TP& obj)  
{  
if (m_tester(obj))  
{  
m_curObj = const_cast<Result_T>(&obj);  
return true;  
}  
return false;  
}  
  
  
Result_T getLastMatchedData()  
{  
return m_curObj;  
}  
  
  
mutable Result_T m_curObj;  
Tester_T m_tester;  
};  
  
  
template<typename Tester_T>  
Finder<Tester_T> makeFinder(Tester_T& rTester)  
{  
return Finder<Tester_T>(rTester);  
}  
  
  
namespace CGetType  
{  
namespace Finder  
{  
template<typename Tester_T> struct applyTester : public Type2Type<UtilExt::Finder<Tester_T> > {};  
}  
}  
  
  
MaxFinder/Min FinderOld/No.x finder implement /  
template<typename Comparer_T>  
struct MaxMinFinder : public UnaryFunctionBase<typename GetBinaryFunctionParam<Comparer_T,1>::type,void>  
{  
typedef typename AddPointer<typename RemoveRef<Param_T>::type>::type Result_T;  
public:  
MaxMinFinder(Comparer_T& pCmp)  
: m_comparor(pCmp)  
, m_maxMinElement(NULL)  
{  
}  
  
  
~MaxMinFinder()  
{  
}  
  
  
void operator()(Param_T obj)  
{  
if (NULL == m_maxMinElement)  
{  
m_maxMinElement = &obj;  
}  
else  
{  
if (m_comparor(obj,*m_maxMinElement))  
{  
m_maxMinElement = &obj;  
}  
}  
}  
  
  
Result_T getResult()  
{  
return m_maxMinElement;  
}  
  
  
protected:  
Comparer_T& m_comparor;  
mutable Result_T m_maxMinElement;   
};   
  
  
  
  
template<typename Comparer_T>  
MaxMinFinder<Comparer_T> makeMaxMinFinder(Comparer_T& rComparer)  
{  
return MaxMinFinder<Comparer_T>(rComparer);  
}  
  
  
}  
/*/  
VS2008 issue : it might be C++ issue, it needs perfect forward support. C++0x has resolved this issue.  
VS2008 compiler doesn't support C++0x : it can't recognize the const qualifier after first function parameter.  
e.g. R (*)(T1, const T2) ==> it recognize it as : R (*)(T1, T2)  
e.g. R (*)(T1, const T2, const T3) ==> it recognize it as : R (*)(T1, T2,T3)  
it only work well for the qualifier on first function parameter : R (*)(const T1,T2,T3)  
pls verify this bug : obverse the variable "f" in VS2008 debug window  
test code begin //  
struct tagS{};  
bool VSBug(tagS const& l, const tagS r){ return true;};   
template<typename T> void showType(T f) { };  
void main()  
{  
showType(VSBug);  
};  
test code end //  
  
  
debug window output:  
f 0x010a2020 VSBug(const tagS &, const tagS) bool (const tagS &, tagS)*  
Type is bool (const tagS &, tagS)* ==> it is wrong  
data type is VSBug(const tagS &, const tagS) ==> it is correct  
//*/  
#endif // functionObject_h__  

