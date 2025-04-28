13) 参数适配 ParamterWrapper 

发布日期: 2013-10-07 13:52:58
原文链接: https://blog.csdn.net/Tonny0832/article/details/12380643

---

我们在写C++接口时，通常会有下列参数形式： 

T, T&以出于不同的需求，比如对于一些内置类型，由于其复制很高效，T及T&没有什么大的区别，我们就可以使用T形式； 但是对于一些比较大的对象，如果用T，则传递过程中会构造临时对象出来，浪费效率；或者对于一些不可复制对象，使用T根本编译不过，就需要使用T&形式；对于一些C程序员或者部分C++程序员来说，可能会使用T*形式，但是这样，就把校验参数的责任交给了接口实现者，有这甚至是不可能的，比如接口返回一个结构体数据，当指针参数为NULL时，返回什么？只能抛出异常。

2.有些时候，系统提供的隐匿转换甚至可以带来错误的逻辑，比如void DemoMP(int);void DemoMP(float&);如果调用 DemoMP(3.56f) 则会匹配 DemoMP(int), 而不是用户期望的 DemoMP(float&), 这完全是调用了错误的接口。

本文章中提供的ParamterWrapper 可以解决这些问题，也可以到我的资源中去下去源代码。

  


#ifndef objectWrapper_h__  
#define objectWrapper_h__  
#include "mplconfig.h"  
#include "mplmacro.h"  
#include <typeinfo>  
#include "parameterWrp.h"  
/********************************************************************  
Description : packaging any type to one unified class, those types have not any relation.  
e.g. those types have not shard base class , those are not convertible also.  
It has type traint   
It is different with force type cast of C style and it is not different with dynamic_cast/static_cast also.  
if you cast it to wrong type, it will return NULL.  
Author : Shen.Xiaolong (Shen Tony) (2010-2013)  
Mail : xlshen2002@hotmail.com, xlshen@126.com  
verified platform : VS2008  
copyright: : latest Version of The Code Project Open License (CPOL : http://www.codeproject.com/)  
*********************************************************************/  
  
  
//  
namespace MiniMPL  
{  
// Base Declaration/  
/*CAnyObject suits to hold different class which have not any relation and any similarity.*/  
class CAnyObject  
{  
public:  
template<typename operatorType> operator operatorType&() const;  
template<typename operatorType> operator operatorType*() const;  
template<typename NewType> NewType cast2Type() const; //readability interface , same as : T* pNewVal = (T*)(*this);  
virtual ~CAnyObject(){};  
  
  
virtual void dump() const = 0 ;   
virtual const type_info& getRealType() const=0;  
  
  
protected:  
CAnyObject(){};  
CAnyObject(CAnyObject const& other);  
CAnyObject& operator=(CAnyObject const& other);  
};  
  
  
Impl /  
  
  
template<typename objectType>   
class CAnyObjectImpl : public CAnyObject  
{  
public:  
explicit CAnyObjectImpl(objectType& Obj,bool bAutoDelete)  
:m_pObject(&Obj)  
,m_bAutoDelete(bAutoDelete)  
{}  
  
  
explicit CAnyObjectImpl()  
: m_bAutoDelete(true)  
{  
m_pObject = new objectType();  
}  
  
  
template<typename P1>  
explicit CAnyObjectImpl(P1& p1)  
: m_bAutoDelete(true)  
{  
m_pObject = new objectType(p1);  
}  
  
  
template<typename P1,typename P2>  
explicit CAnyObjectImpl(P1& p1,P2& p2)  
: m_bAutoDelete(true)  
{  
m_pObject = new objectType(p1,p2);  
}  
  
  
template<typename P1,typename P2,typename P3>  
explicit CAnyObjectImpl(P1& p1,P2& p2,P3& p3)  
:m_bAutoDelete(true)  
{  
m_pObject = new objectType(p1,p2,p3);  
}  
  
  
virtual ~CAnyObjectImpl()  
{   
if (m_bAutoDelete && m_pObject)  
{  
Static_Assert(0!=sizeof(objectType)); //check complete type  
delete m_pObject;  
m_pObject = NULL;  
}  
}  
  
  
inline operator objectType&()  
{  
return *getObject();  
}  
  
  
inline operator objectType*()  
{  
return getObject();  
}  
  
  
protected:  
//CAnyObjectImpl();  
virtual const type_info& getRealType() const  
{  
return typeid(objectType);  
}  
  
  
inline objectType* getObject()  
{  
return m_pObject;  
}  
  
  
virtual void dump() const  
{   
DMUPINFO(("Real Type : %s\n",typeid(objectType).name()));  
#if defined(_MSC_VER)  
__if_exists(objectType::dump)  
{  
m_pObject->dump();  
}  
#endif  
}  
  
  
private:  
objectType* m_pObject;   
bool m_bAutoDelete; //maintain objectType instance life cycle  
};  
  
  
template<typename operatorType>  
CAnyObject::operator operatorType&() const  
{  
operatorType* pObject = *this; //implement operator operatorType*   
AssertP(pObject);  
if (NULL == pObject)  
{  
throw 1;  
}  
  
  
return *pObject;  
}  
  
  
template<typename operatorType>  
CAnyObject::operator operatorType*() const  
{  
/*! Limition :   
the operator Type must be the same with actual type.  
operator Type can't be base class because CAnyObject isn't needed for some classes which have shared base class.  
In those cases they can use base class pointer directly, instead of CAnyObject.  
CAnyObject suits to hold different class which have not any relation and any similarity.  
*/  
if(getRealType()!=typeid(operatorType))  
{  
return NULL;  
}  
  
  
return (CAnyObjectImpl<operatorType>&)(*this); //implement CObjectWrapperImpl::operator objectType*  
}  
  
  
template<typename NewType>  
NewType CAnyObject::cast2Type() const  
{  
return *this;  
}  
  
  
/ maker //  
template<typename ObjectType>  
CAnyObject* bindAnyObject(ObjectType& object,bool bAutoDelete)  
{  
return new CAnyObjectImpl<ObjectType>(object,bAutoDelete);  
}  
  
  
template<typename ObjectType>  
CAnyObject* newAnyObject()  
{  
return new CAnyObjectImpl<ObjectType>(); //ObjectType()  
}  
  
  
template<typename ObjectType,typename P1>  
CAnyObject* newAnyObject(P1& p1)  
{  
return new CAnyObjectImpl<ObjectType>(p1); //ObjectType(p1)  
}  
  
  
template<typename ObjectType,typename P1>  
CAnyObject* newAnyObject(Paramter<P1>& p1)  
{  
return new CAnyObjectImpl<ObjectType>(p1); //ObjectType(p1)  
}  
  
  
template<typename ObjectType,typename P1,typename P2>  
CAnyObject* newAnyObject(P1& p1,P2& p2)  
{  
return new CAnyObjectImpl<ObjectType>(p1,p2); //ObjectType(p1,p2)  
}  
  
  
template<typename ObjectType,typename P1,typename P2>  
CAnyObject* newAnyObject(Paramter<P1>& p1,Paramter<P2>& p2)  
{  
return new CAnyObjectImpl<ObjectType>(p1,p2); //ObjectType(p1,p2)  
}  
  
  
template<typename ObjectType,typename P1,typename P2,typename P3>  
CAnyObject* newAnyObject(P1& p1,P2& p2,P3& p3)  
{  
return new CAnyObjectImpl<ObjectType>(p1,p2,p3); //ObjectType(p1,p2,p3)  
}  
  
  
template<typename ObjectType,typename P1,typename P2,typename P3>  
CAnyObject* newAnyObject(Paramter<P1>& p1,Paramter<P2>& p2,Paramter<P3>& p3)  
{  
return new CAnyObjectImpl<ObjectType>(p1,p2,p3); //ObjectType(p1,p2,p3)  
}  
}  
  
  
#endif // objectWrapper_h__  
  
  


[](http://www.rongshuxia.com/chapter/bookid-6125193-chapterid-254822.html)
