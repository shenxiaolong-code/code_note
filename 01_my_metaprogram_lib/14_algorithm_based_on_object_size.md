14) 常用的遍历/(条件)查找/收集数据/(拷贝2比较优先)排序算法algorithm

发布日期: 2013-10-07 14:13:17
原文链接: https://blog.csdn.net/Tonny0832/article/details/12381931

---

下面的一套算法，配合前面提供的泛型库的支持部件，能够提供下列基于概念的功能：

1.遍历处理

3.查找

5.收集 ： 收集的结果可以是任何形式的：数组/容器，T/T*，并且调用接口是完全相同的(减轻用户负担)，代码内部会自动进行类型识别。

7.排序

7.1) 用户可以指定执行方式是比较优先/拷贝优先/缺省自动 

7.2) 按参照顺序排序 ：即不是按从大到小，也不是按从小到大，而是按用户指定的参考顺序。比如有一个结构数据流是无顺序的，经过一个处理后，其按照结构中某个成员的大小进行了排序。但是在下一步处理中，用户要求查看原序－－没有排序过的原始数据。这时就可以使用算法提供的按参照排序：先记录原始数据流的每个unique值到一个数组/容器中，在按参考排序时传入这个参数数据。

  


除了排序外，其它的6种功能基于概念，都会经过一个关键路径throughoutImpl：基于finder的throughoutImpl和基于遍历的throughoutImpl. (finder找到后会返回，而遍历的会处理所有数据)。

并且每个基于概念的功能都提供一个实现体入口XXXImpl (类似于MS的原始API)，如果库提供封装的接口(类似于MS的MFC封装)不能满足用户的需求，用户可以直接调用XXXImpl接口并且传入用户按需求组装的参数--这些参数通常使用使用functionobject中提供的接口来生成。

代码：

  


#ifndef globalFunc_h__  
#define globalFunc_h__  
#include "mplconfig.h"  
#include "mplMacro.h"  
#include "typeTraits.h"  
#include "functionObject.h"  
#include "dataSet.h"  
#include "functionTraits.h"  
/********************************************************************  
Description : provide algorithm for no STL algorithm supported library .  
features : Finder and Non-Finder.   
Finer is depended on Tester function object  
Non_Finder is depended on UnaryFunction function object.  
All function object can be got in "functionobject.h"   
Author : Shen.Xiaolong (Shen Tony) (2010-2013)  
Mail : xlshen2002@hotmail.com, xlshen@126.com  
verified platform : VS2008  
copyright: : latest Version of The Code Project Open License (CPOL : http://www.codeproject.com/)  
*********************************************************************/  
  
  
declartion//  
namespace MiniMPL  
{  
Implement//  
template<typename T,typename S=typename T::Param_T> struct IsFinder : public IsSerialClass<Finder<S>,T> {};  
  
template<typename iteratorT,typename Finder_T>  
Finder_T& throughoutImpl(iteratorT pHead,iteratorT pGuard,Finder_T& rFinder,SFINAE((IsFinder<Finder_T>::value)))  
{  
for (;pHead != pGuard;pHead++)  
{  
if(rFinder(*pHead))  
{  
return rFinder;  
}  
}  
return rFinder;  
}  
  
  
template<typename iteratorT,typename Processer_T>  
Processer_T& throughoutImpl(iteratorT pHead,iteratorT pGuard,Processer_T& rProcesser,SFINAE((!IsFinder<Processer_T>::value)))  
{  
for (;pHead != pGuard;pHead++)  
{  
rProcesser(*pHead);  
}  
return rProcesser;  
}  
  
  
template< typename C , typename HandlerType>  
HandlerType& throughout(DataSet<C>& arr,HandlerType& rHandler)  
{  
return throughoutImpl(arr.head(),arr.end(),rHandler);  
}  
  
  
// [Processor]call specified function to process every object ///  
template<typename C,typename UnaryFunction_T>  
typename DataSet<C>::size_type forEach(DataSet<C>& rDataSet,UnaryFunction_T& rProcesser)  
{  
throughout(rDataSet,rProcesser);  
return rDataSet.size();  
}  
  
  
///search function///  
template<typename C,typename Tester_T>  
SC(ET(C)*,C) find(DataSet<C>& arr,Tester_T& tester)  
{  
Static_Assert((IsSameRawType<ET(C),typename Tester_T::Param_T>::value==true));  
return removeConst(throughout(arr,makeFinder(tester)).getLastMatchedData());  
}   
  
  
///min/max function///  
template<typename C,typename Comparer_T>  
SC(ET(C)*,C) findMaxminElement(DataSet<C>& arr,MaxMinFinder<Comparer_T>& finder)  
{  
return throughout(arr,finder).getResult();  
}  
  
  
/collect function///  
template<typename C,typename Tester_T,typename ResultCollector_T>  
unsigned int collect(DataSet<C>& rDataSet,Collector<Tester_T,ResultCollector_T>& rCollector)  
{  
return throughout(rDataSet,rCollector).size();  
}  
  
  
/sort function///  
//this algorithm more compare and less copy, it is suitable for big object instance  
template<typename PointerType,typename Comparer_T>  
bool sortImplPriorCmp(PointerType pHead, PointerType pTail,Comparer_T& cmp)  
{   
if (pHead == pTail)  
{  
return true;  
}  
  
  
PointerType pNewHead = pHead;  
pNewHead++;  
PointerType pGuard = pTail;  
pGuard++;  
swapIf(*pHead,*throughoutImpl(pNewHead,pGuard,MaxMinFinder<VT(PointerType)>(NotComparor<VT(PointerType)>(cmp))).getResult(),cmp);  
return sortImplPriorCmp(++pHead,pTail,cmp);  
} ;  
  
  
//this algorithm less compare and more copy, it is suitable for small object instance  
template<typename PointerType,typename Comparer_T>  
bool sortImplPriorCopy(PointerType head, PointerType tail,Comparer_T& cmp)  
{  
if (head == tail)  
{  
return true;  
}  
  
  
PointerType pNewHead = head;  
pNewHead++;  
  
  
if (pNewHead == tail)  
{  
swapIf(*head,*pNewHead,cmp);  
return true;  
}  
  
  
sortImplPriorCopy(pNewHead,tail,cmp);  
  
  
if (cmp(*head,*pNewHead))  
{  
GetValueType<PointerType>::type TmpObj(*head);  
  
  
if (cmp(*head,*tail))  
{ //case : *pNewHead < *tail < *head  
while(head != tail)   
{  
*head++ = *pNewHead++ ;  
};  
*head = TmpObj;  
}  
else  
{ //case : *pNewHead < *head < *pTail   
*head++ = *pNewHead++;  
while(cmp(TmpObj,*pNewHead))   
{  
*head++ = *pNewHead++;  
};  
*head = TmpObj ;  
}  
}  
  
  
return true;  
}  
  
  
///  
  
  
template<typename C,typename Compare_T>  
bool sortImpl(DataSet<C>& arr,Compare_T& rCmp,bool bCmpPrior)  
{  
return bCmpPrior ? sortImplPriorCmp(arr.head(),arr.tail(),rCmp) : sortImplPriorCopy(arr.head(),arr.tail(),rCmp);   
}  
  
  
template<typename C1,typename M1,typename C2,typename M2>  
bool sortByRefOrder(DataSet<C1>& rArr,MemberPtr<ET(C1),M1>& rMemberPtr1,DataSet<C2>& rArrRef,MemberPtr<ET(C2),M2>& rMemberPtr2)  
{  
typedef ET(C1) Struct_T1;  
typedef typename DataSet<C1>::size_type SizeType;  
  
  
Assertb(rArrRef.size() == rArr.size());  
SizeType iSize = rArr.size();  
for (SizeType i=0;i<iSize;i++)  
{   
Struct_T1* ptr = find(rArr,makeTester(rMemberPtr2(rArrRef[i]),rMemberPtr1,equal<M1,M2>));  
Assertb(NULL != ptr);  
MiniMPL::swap(rArr[i],*ptr);  
}  
  
  
return true;  
}  
}  
  
  
#endif // globalFunc_h__  

