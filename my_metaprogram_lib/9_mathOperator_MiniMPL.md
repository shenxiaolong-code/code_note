9) 泛型数学运算符函数mathOperator 

发布日期: 2013-10-07 14:34:20
原文链接: https://blog.csdn.net/Tonny0832/article/details/12383709

---

下面是一些算术操作符，其通常作为一些接口的比较参数传入。泛型的。

#ifndef mathOperator_h__  
#define mathOperator_h__  
  
  
#include "typeConvert.h"  
#include "mplmacro.h"  
/********************************************************************  
  
  
Description : general math operator .  
Author : Shen.Xiaolong (Shen Tony) (2010-2013)  
Mail : xlshen2002@hotmail.com, xlshen@126.com  
verified platform : VS2008  
copyright: : latest Version of The Code Project Open License (CPOL : http://www.codeproject.com/).  
*********************************************************************/  
  
  
namespace MiniMPL  
{  
/  
template <typename LeftT,typename RightT>  
bool equal(LeftT p1,RightT p2)  
{  
return p1==p2;  
}  
  
  
template <typename LeftT,typename RightT>  
bool lesser(LeftT p1,RightT p2)  
{  
return p1 < p2;  
}  
  
  
template<typename BinaryFunction_T,typename LeftT = typename BinaryFunction_T::TP1,typename RightT = typename BinaryFunction_T::TP2>  
class NotBinary   
{  
public:  
NotBinary(BinaryFunction_T& pCmp) : m_cmp(pCmp){};  
bool operator()(LeftT & p1,RightT & p2)  
{  
return !m_cmp(p1,p2);  
}  
  
  
protected:  
BinaryFunction_T& m_cmp;  
};  
  
  
template<typename UnaryFunction_T,typename TP=typename UnaryFunction_T::Param_T>  
class NotUnary   
{  
public:  
NotUnary(UnaryFunction_T& f) : m_f(f){};  
bool operator()(TP & p)  
{  
return !m_f(p);  
}  
  
  
protected:  
UnaryFunction_T& m_f;  
};  
  
  
template <typename LeftT,typename RightT>  
bool notEqual(LeftT p1,RightT p2 )  
{  
return !equal<LeftT,RightT>(p1,p2);  
}  
  
  
template <typename LeftT,typename RightT>  
bool lesserEqual(LeftT p1,RightT p2 )  
{  
return lesser<LeftT,RightT>(p1,p2) || equal<LeftT,RightT>(p1,p2);  
}  
  
  
template <typename LeftT,typename RightT>  
bool greater(LeftT p1,RightT p2 )  
{  
return lesser<RightT,LeftT>(p2,p1);  
}  
  
  
template <typename LeftT,typename RightT>  
bool greaterEqual(LeftT p1,RightT p2 )  
{  
return greater<LeftT,RightT>(p1,p2) || equal<LeftT,RightT>(p1,p2);  
}  
  
  
/  
template<typename DataType>  
void swap(DataType& left,DataType& right,SFINAE(logic::Not_T<IsStlContainer<DataType> >::value))  
{   
if (&left == &right) return; //same element  
  
  
DataType tmp = left;  
left = right;  
right = tmp;  
}  
  
  
template<typename DataType>  
void swap(DataType& left,DataType& right,SFINAE(IsStlContainer<DataType>::value))  
{   
if (&left == &right) return; //same element  
left.swap(right);  
}  
  
  
template<typename DataType,unsigned int LEN>  
void swap(DataType(&left)[LEN],DataType(&right)[LEN] )  
{   
if (&left == &right) return; //same element  
for (unsigned int i=0;i<LEN;i++)  
{  
swap(left[i],right[i]);  
}  
}  
  
  
template<typename DataType,typename Comparer_T>  
bool swapIf(DataType& left,DataType& right,Comparer_T& cmp)  
{  
if (cmp(left,right))  
{  
swap(left,right);  
return true;  
}   
return false;  
}  
  
  
template<typename DataType,typename Comparer_T>  
bool swapNotIf(DataType& left,DataType& right,Comparer_T& cmp)  
{  
if (!cmp(left,right))  
{  
swap(left,right);  
return true;  
}  
return false;  
}  
}  
  
  
  
  
#endif // mathOperator_h__  

