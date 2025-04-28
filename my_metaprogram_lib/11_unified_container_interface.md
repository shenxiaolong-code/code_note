11) 为STL容器和数组提供统一的接口DataSet

发布日期: 2013-10-07 13:34:49
原文链接: https://blog.csdn.net/Tonny0832/article/details/12380427

---

源代码可以在我的资源中去下载(包括用法测试的源代码):  
http://download.csdn.net/detail/tonny0832/7426991  
只需要用VS建一个空的工程，然后引入我的测试头文件即可。然后测试函数会在main函数之前运行，在控制台窗口中可以看到输出。

  


在一些产品上，对于数据块，有些人喜欢用数组，比较典型的C程序员，有些人喜欢用STL容器，对于框架操作，我们通常希望存取这些对象能够使用一套统一的接口去存取它，因为从概念上讲，它们无非就是一组数据的集合，我不关心存取它们方式的不同，只需要给我一个，存入一个数等。为了满足这些需求，下列源文件提供把C数组和STL容器做了统一的封装，以提供一样的接口。(没有提供查入/删除接口)

  


#ifndef __dataSet_h__  
#define __dataSet_h__  
#include "typeconvert.h"  
#include "macroDef.h"  
#include "typeTraits.h"  
#include "libConfig.h"  
/********************************************************************  
Description : provide same package for classical array and STL container to same format .  
Author : Shen.Xiaolong (Shen Tony) (2010-2013)  
Mail : xlshen2002@hotmail.com, xlshen@126.com  
verified platform : VS2008  
copyright: : latest Version of The Code Project Open License (CPOL : http://www.codeproject.com/)  
*********************************************************************/  
  
  
/// data set wrapper   
//below macro is used to reduce string length in function interface  
#define ET_(X) UtilExt::GetValueTypeFromContainer<X>::type  
#define ET(X) typename ET_(X) //ET : element type of STL container or C Array  
#define ETP_(X) UtilExt::GetCPointer<X>::type   
#define ETP(X) typename ETP_(X) //ET : element type of STL container or C Array  
  
  
namespace UtilExt  
{  
template<typename C,bool bStl=IsStlContainer<C>::value,bool bArr=isArray<C>::value> struct DataSet;  
template<typename Set_T,typename val_T,typename Pointer_T,typename size_T>   
struct DataSetBase  
{  
typedef Set_T PackageType;  
typedef typename AddRef<PackageType>::type RefPackage_T;  
typedef size_T size_type;  
  
  
typedef val_T value_type;  
typedef Pointer_T iterator;  
};  
  
namespace CGetType  
{  
namespace DataSet  
{  
template<typename T,bool bStl=IsStlContainer<T>::value,bool bArr=isArray<T>::value> struct apply;  
template<typename T> struct apply<T,false,true> : public Type2Type<UtilExt::DataSet<typename AddRef<T>::type> > {};  
template<typename T> struct apply<T,true,false> : public Type2Type<UtilExt::DataSet<typename RemoveRef<T>::type> > {};  
}  
}  
  
  
template<typename T> struct IsDataSet  
{  
private:  
template<typename U , typename P=NullType> struct CHasPackageType : public FalseType{};  
template<typename U> struct CHasPackageType<U,typename sfinae_helper<typename U::PackageType>::type>: public TrueType {};  
  
  
public:  
enum {value = CHasPackageType<T>::value};  
};  
  
  
template<typename T, bool bIsDataSet=IsDataSet<T>::value> struct GetCPointer;  
template<typename T> struct GetCPointer<T,false> : public GetCPointer<typename CGetType::DataSet::apply<T>::type> {};  
template<typename T> struct GetCPointer<T,true> : public Type2Type<typename T::value_type*> {};  
  
  
template<typename T, bool bIsDataSet=IsDataSet<T>::value> struct GetValueTypeFromContainer;  
template<typename T> struct GetValueTypeFromContainer<T,false> : public GetValueTypeFromContainer<typename CGetType::DataSet::apply<T>::type> {};  
template<typename T> struct GetValueTypeFromContainer<T,true> : public Type2Type<typename T::value_type> {};  
  
  
stl container ///  
template<typename Stl_T> struct DataSet<Stl_T,true,false>   
: public DataSetBase<Stl_T,  
typename Evl_If_T<IsConst<Stl_T>,AddConst<typename Stl_T::value_type>,Type2Type<typename Stl_T::value_type> >::type::type,  
typename Evl_If_T<IsConst<Stl_T>,typename Stl_T::const_iterator,typename Stl_T::iterator>::type,  
typename Stl_T::size_type>   
{  
explicit DataSet(RefPackage_T obj)  
: m_obj(obj)  
{ }  
  
  
iterator begin()  
{  
return m_obj.begin();  
}  
  
  
iterator end()  
{  
return m_obj.end();  
}  
  
  
iterator head()  
{  
return size()>0 ? begin() : end();  
}  
  
  
iterator tail()  
{  
iterator pTmp = end();  
return size()>0 ? --pTmp : pTmp;  
//return size()>0 ? --end() : end(); //some special stl container doesn't support this usage  
}  
  
  
value_type& front()  
{  
ChkExceptionX(size()>0);  
return m_obj.front();  
}  
  
  
value_type& back()  
{  
ChkExceptionX(size()>0);  
return m_obj.back();  
}  
  
  
template<typename T>  
static value_type& At(T& obj, size_type i)  
{  
Assertb(i < obj.size());  
iterator it=obj.begin();  
for (size_type cnt=0; cnt<i; cnt++ , ++it);  
Assertb(it!=obj.end());  
return *(it);  
}  
  
  
static value_type& At(stlVector<value_type>& obj, size_type i)  
{   
Assertb(i < obj.size());  
return obj[i];  
}  
  
  
value_type& operator[](size_type i)  
{   
return At(m_obj,i);  
}  
  
  
size_type size() const  
{  
return m_obj.size();  
}  
  
  
void clear()  
{  
m_obj.clear();  
}  
  
  
void push_back(value_type const& val)  
{  
m_obj.push_back(val);  
}  
  
operator PackageType&()  
{  
return m_obj;  
}   
  
  
RefPackage_T m_obj;  
};  
  
  
template<typename Stl_T>  
DataSet<Stl_T> makeDataSet(Stl_T& stlArr,SFINAE(IsStlContainer<Stl_T>::value))  
{  
return DataSet<Stl_T>(stlArr);  
}  
  
  
template<typename Stl_T>  
DataSet<Stl_T> makeResult(Stl_T& rResult,SFINAE(IsStlContainer<Stl_T>::value))  
{  
return makeDataSet(rResult);  
}  
  
  
class array ///  
template<typename T> struct ArrValTypeHelper;  
template<typename S,unsigned int LEN> struct ArrValTypeHelper<S[LEN]> : public Type2Type<S> {};  
template<typename S,unsigned int LEN> struct ArrValTypeHelper<S(&)[LEN]> : public Type2Type<S> {};  
  
  
template<typename Arr_T> struct DataSet<Arr_T,false,true>   
: public DataSetBase<Arr_T,  
typename ArrValTypeHelper<Arr_T>::type,  
typename ArrValTypeHelper<Arr_T>::type*,  
unsigned int>   
{  
enum{array_len=ArraySize<Arr_T>::value};  
  
  
explicit DataSet(RefPackage_T obj,size_type iSize=0)  
: m_obj(obj)  
, m_iSize(iSize)  
{  
ChkExceptionX(m_iSize<=array_len);  
}  
  
  
iterator begin()  
{  
return m_iSize>0 ? &m_obj[0] : NULL;  
}  
  
  
iterator end()  
{  
return m_iSize>0 ? &m_obj[m_iSize] : NULL ;  
}  
  
  
iterator head()  
{  
return begin();  
}  
  
  
iterator tail()  
{  
return m_iSize>0 ? &m_obj[m_iSize-1] : NULL ;  
}  
  
  
value_type& front()  
{  
ChkExceptionX(size()>0);  
return *head();  
}  
  
  
value_type& back()  
{  
ChkExceptionX(size()>0);  
return *tail();  
}  
  
  
void push_back(value_type const& val)  
{  
if (size()<array_len)  
{  
new (&m_obj[m_iSize]) value_type(val);  
m_iSize++;  
}  
else  
{  
ChkExceptionX(size()<array_len);  
}   
}  
  
  
value_type& operator[](size_type i)  
{   
Assertb(i<array_len);  
return m_obj[i];  
}  
  
  
size_type size() const  
{  
return m_iSize;  
}   
  
  
void clear()  
{  
m_iSize = 0;  
}  
  
  
/*   
below operator cause ambiguity, e.g. for rArr[2] , it could be  
S &UtilExt::DataSet<S(&)[LEN]>::operator [](UtilExt::DataSet<S(&)[LEN]>::size_type)  
or built-in C++ operator[](S [LEN], int)  
can un-comment-out below operation and compile to see compile error  
it is say : only one is available in value_type& operator[](size_type i) / operator PackageType&()  
because we need to assert index, so we have to discard operator PackageType&() and keep value_type& operator[](size_type i)  
*/  
/*/  
operator PackageType&()  
{  
return m_obj;  
}  
//*/  
  
size_type m_iSize;  
RefPackage_T m_obj;  
};  
  
  
template<typename S,unsigned int LEN>  
DataSet<S (&)[LEN]> makeDataSet(S (&obj)[LEN],unsigned int iSize=LEN)  
{   
return DataSet<S (&)[LEN]>(obj,iSize);  
};  
  
  
template<typename S,unsigned int LEN>  
DataSet<S(&)[LEN]> makeResult(S(&rResult)[LEN],unsigned int iSize=0)  
{  
return makeDataSet(rResult,iSize);  
}  
}  
  
  
#endif // dataSet_h__  

