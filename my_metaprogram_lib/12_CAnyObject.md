12) 类型安全的任意类CAnyObject 

发布日期: 2013-10-07 12:06:03
原文链接: https://blog.csdn.net/Tonny0832/article/details/12378143

---

anyObject.h , 这是泛型库中13个源文件中的一个， 这个源文件可以封装任意类到一个共同的类中，不需要封装的类之间有任何联系，在具体应用时，down-cast时是类型安全的，并且不需要RTTI的支持。（有些开发环境为了提高运行速度，会把RTTI关掉的－－这样大约有10%的性能提高） 这个功能分两级实现的，第一级是一个普通类，用于用户使用非模板代码hold住，第二级是一个模板类，用户CAnyObject的实现。并且基类实现了模板化的两个操作符重载*及＆，在获取*及＆时，没有使用强制向下的类型转换(down-cast)，因为其是类型安全的。

//anyObject.h

#ifndef objectWrapper_h__  
#define objectWrapper_h__  
#include "config.h"  
#include "macrodef.h"  
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
namespace UtilExt  
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
  
test code/

//test_anyObject.h

#ifndef test_objectWrapper_h__  
#define test_objectWrapper_h__  
#include "..\anyObject.h"  
  
  
/********************************************************************  
Description : packaging any type to one unified class, those types have not any relation.  
e.g. those types have not shard base class , those are not convertible also.  
It has type traint   
It is different with force type cast of C style and it is not different with dynamic_cast/static_cast also.  
if you cast it to wrong type, it will return NULL.  
Author : Shen.Xiaolong (Shen Tony)  
Date : 2012.6.20  
Mail : xlshen2002@hotmail.com, xlshen@126.com  
verified platform : VS2008  
copyright: : free to use / modify / sale in free and commercial software   
Unique limit: MUST keep those copyright comments in all copies and in supporting documentation.  
usage demo : #define RUN_EXAMPLE_OBJECTWRAPPER and include this header file to run demo  
*********************************************************************/  
  
  
//#define RUN_EXAMPLE_OBJECTWRAPPER  
  
  
#ifdef COMPILE_EXAMPLE_ALL  
#define COMPILE_EXAMPLE_OBJECTWRAPPER  
#endif  
  
  
#ifdef RUN_EXAMPLE_ALL  
#define RUN_EXAMPLE_OBJECTWRAPPER  
#endif  
  
  
#if defined(RUN_EXAMPLE_OBJECTWRAPPER) && !defined(COMPILE_EXAMPLE_OBJECTWRAPPER)  
#define COMPILE_EXAMPLE_OBJECTWRAPPER  
#endif  
  
  
/  
//demo example implement begin //  
#ifdef COMPILE_EXAMPLE_OBJECTWRAPPER   
#include "test_data_def.h"  
#include "test_macrodef.h"  
  
  
namespace DemoUsage  
{  
template <typename paramType1> class CTypeTraints;  
class CObjectAdapter  
{  
public:  
template <typename objectType>   
CObjectAdapter(objectType& object,bool bAutoDelete=true)  
:m_pObject(UtilExt::bindAnyObject(object,bAutoDelete)) {}  
  
  
virtual ~CObjectAdapter()  
{  
delete m_pObject;  
}  
  
  
template<typename operatorType> operator operatorType&()  
{  
return (operatorType&)(*m_pObject);  
}  
  
  
template<typename operatorType> operator operatorType*()  
{  
return (operatorType*)(*m_pObject);  
}  
  
  
/*! known actual derived handler type, and invoke it explicitly*/  
template<typename objectType,typename RetT>  
RetT demoFuncCall0(RetT (objectType::*pFunc)())  
{  
return ((objectType&)*m_pObject.*pFunc)();  
};  
  
  
/*!  
* NOT known actual derived handler type and parameter type is different with handler type  
* using CTypeTraints binding to get actual derived handler type.  
*/  
template<typename objectType,typename paramType1,typename RetT>  
RetT demoFuncCall1(RetT (objectType::*pFunc)(paramType1&),paramType1& rParam1)  
{  
return (((typename CTypeTraints<paramType1>::Handler_Type&)*m_pObject).*pFunc)(rParam1);  
};  
  
  
template<typename objectType,typename paramType1,typename paramType2,typename RetT>  
RetT demoFuncCall2(RetT (objectType::*pFunc)(paramType1&,paramType2&),paramType1& rParam1,paramType2& rParam2)  
{  
return (((typename CTypeTraints<paramType1>::Handler_Type&)*m_pObject).*pFunc)(rParam1,rParam2);  
};  
  
  
void dump()  
{  
m_pObject->dump();  
}  
  
  
private:  
UtilExt::CAnyObject* m_pObject;  
};  
  
  
/*! example parameter type */  
typedef struct _tagParamType  
{  
int m_iData;  
}UserParam,*lpUserParam;  
  
  
/*! example handler type */  
class UserHandler  
{  
public:  
UserHandler():m_count(0)  
{  
TRACE_HERE();  
};  
UserHandler(UserHandler const& other)  
{  
TRACE_HERE();  
};  
  
  
virtual ~UserHandler()  
{  
TRACE_HERE();  
};  
  
  
int demoFuncTraintCall(UserParam& obj)   
{  
m_count++;  
TRACE_HERE();  
return m_count;  
};  
  
  
void dump()  
{  
printf("I am UserHandler!\n");  
}  
  
  
char demoFuncCall()   
{  
m_count++;  
TRACE_HERE();  
return 'Y';  
}  
  
  
private:  
int m_count;  
};  
  
  
/*! Bind parameter type [UserParam] and handler type [ UserHandler] */  
template<> class CTypeTraints<UserParam>  
{  
public:  
typedef UserHandler Handler_Type;  
};  
  
  
void otherUserFuncP(UserHandler* pObj)  
{  
AssertP(pObj);  
pObj->demoFuncCall();  
}  
  
  
void otherUserFunc(UserHandler& obj)  
{  
obj.demoFuncCall();  
}  
  
  
void demoObjectWrapperInterface(CObjectAdapter* pObj)  
{  
DMUPINFO(("dump MyObjectWrapper:\n"));  
pObj->dump();  
  
  
DMUPINFO(("No param , known handler type : %s\n",typeid(UserHandler).name()));  
char cInvokeRet0 = pObj->demoFuncCall0(&UserHandler::demoFuncCall);  
  
  
UserParam paramT;  
DMUPINFO(("param type : %s , handler type : %s\n",typeid(paramT).name(),"decided by traits"));   
int iInvokeRet1 = pObj->demoFuncCall1(&UserHandler::demoFuncTraintCall,paramT);  
  
  
/*!  
* otherUserFunc needs parameter "UserHandler", but we only has parameter "CObjectAdapter".  
* operator override will convert CObjectAdapter type to UserHandler automatically  
*/  
DMUPINFO(("required param type : %s , actual param type : %s\n",typeid(UserHandler).name(),typeid(CObjectAdapter).name()));  
otherUserFunc(*pObj);  
otherUserFuncP(*pObj);   
}  
  
  
struct VpAnyObject  
{  
VpAnyObject(){};  
VpAnyObject(int){};  
VpAnyObject(float&){};  
VpAnyObject(S3&){};  
VpAnyObject(char*p,int){};   
VpAnyObject(S3,CDemoStruct&){};  
  
  
VpAnyObject(char*p,int,float&){};  
~VpAnyObject()  
{  
TRACE_HERE();  
}  
};  
  
  
  
  
bool showUsageObjectWrapper()  
{  
using namespace UtilExt;  
  
  
//test operatorType* and cast2Type  
S3 obj;  
UtilExt::CAnyObject* pObj = UtilExt::bindAnyObject(obj,true);  
S3* pCastObj = pObj->cast2Type<S3*>();  
Assertb(pCastObj==&obj);  
pCastObj = *pObj;  
Assertb(pCastObj==&obj);  
//test operatorType&  
S3& rS3 = *pObj;  
Assertb(&rS3 == &obj);  
//delete pObj; //memeory leak becase of testing operatorType& , tolerate it  
  
  
//test default constructor  
pObj = newAnyObject<VpAnyObject>();  
delete pObj;  
  
  
//test constructor with 1 parameter  
//.1 hard-code input parameter : int or int&  
pObj = newAnyObject<VpAnyObject>(MP(4)); //newAnyObject<VpAnyObject>(4) cause compile error , for hard-code, only newAnyObject<T>(MP(p)) is OK  
delete pObj;  
//.2 variable input parameter : int or int&  
int iParam = 4;  
pObj = newAnyObject<VpAnyObject>(iParam);  
delete pObj;  
//.3 customized reference type : S3&  
pObj = newAnyObject<VpAnyObject>(MP(rS3));  
delete pObj;  
pObj = newAnyObject<VpAnyObject>(rS3); //for variable, both newAnyObject<T>(p) and newAnyObject<T>(MP(p)) are OK  
delete pObj;  
//.4 build in type reference type : float&  
pObj = newAnyObject<VpAnyObject>(MP(5.62f)); //for hard-code, only newAnyObject<T>(MP(p)) is OK  
delete pObj;  
  
  
//test constructor with 2 parameter  
pObj = newAnyObject<VpAnyObject>(MP((char*)NULL),MP(4)); //for hard-code, only newAnyObject<T>(MP(p)) is OK  
delete pObj;  
pObj = newAnyObject<VpAnyObject>(rS3,CDemoStruct()); //for variable, both newAnyObject<T>(p) and newAnyObject<T>(MP(p)) are OK  
delete pObj;  
pObj = newAnyObject<VpAnyObject>(MP(rS3),MP(CDemoStruct()));   
delete pObj;  
  
  
//test constructor with 3 parameter  
pObj = newAnyObject<VpAnyObject>(MP((char*)NULL),MP(4),MP(1.0f));  
delete pObj;  
  
  
PrintTitle1("Demo object wrapper usage");  
CObjectAdapter* pNew=new CObjectAdapter(*new UserHandler()); //wrapper encapsulates actual type : UserHandler  
demoObjectWrapperInterface(pNew);  
PrintEndLine();  
return true;  
}  
  
  
#ifdef RUN_EXAMPLE_OBJECTWRAPPER  
InitRunFunc(showUsageObjectWrapper);  
#endif //RUN_EXAMPLE_OBJECTWRAPPER  
}  
#endif // SHOW_USAGE_OBJECTWRAPPER  
  
  
#endif // test_objectWrapper_h__  
  
  


  


[](http://www.rongshuxia.com/chapter/bookid-6125191-chapterid-292106.html?pkey=015a8b23b33c800de823cd4944957604)
