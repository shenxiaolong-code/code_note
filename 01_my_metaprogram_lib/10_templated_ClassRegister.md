10) 泛型工厂方法创建实例ClassRegister 

发布日期: 2013-10-07 12:41:17
原文链接: https://blog.csdn.net/Tonny0832/article/details/12378845

---

源代码可以在我的资源中去下载(包括用法测试的源代码):

<https://github.com/ShenXiaolong1976/MiniMPL>

只需要用VS建一个空的工程，然后引入我的测试头文件即可。然后测试函数会在main函数之前运行，在控制台窗口中可以看到输出。

这个源文件的功能是实现像MFC的动态创建的效果。 但是和MFC相比，这个功能有下列优势：

1.MFC的动态创建创建出来的类的基类必须是CObject的，这个源文件没有任意限制，因为这个源文件的实现是模板化的，这创建出来的对象的基类可以是任意类型的－－取决于你给它的约定。 关键一点：这些创建活动是类型安全的，其中没有任何类型强制转换行为－－包括down-cast行为。

2.MFC的动态创建创建对象只能根据类型名字字符串，比如"CMyClass",而这些功能源文件不仅可以根据类型名称字符串来创建，还可以根据下列特征来创建：

2.1 关联类型 ：比如用户指定一个A类型，ClassRegister自动帮你创建一个B类型对象出来，因为在B中指定了A是B的关系类型。

2.2 关联数据 ：比如不同的设备类型有不同的设备描述符结构，每个设备描述符结构中都有多个特征值，ClassRegister可以根据用户指定的任意设备描述符的任意特征值来创建最终派生类设备。

2.3 MFC的动态创建不可使用构造参数，本ClassRegister提供的template<typename B> class CTypeObjectCreater经过用户重载后，可以使用构造参数进行创建，非常自由灵活。

2.4 可以统计满足任意条件的类型数量，并且分别创建出它们的实例来。

  
#ifndef __test_Class_Register_H__  
#define __test_Class_Register_H__  
#include <utility/classregister.h>  
  
  
/********************************************************************  
Description : This macro is used to create class object by class's proper value, such as class name, ID,characters value,etc.  
Author : Shen.Xiaolong (Shen Tony)  
Date : 2010.5.8  
Mail : xlshen2002@hotmail.com, xlshen@126.com  
verified platform : gcc version 4.1.0 (SUSE Linux) , VS2008  
copyright: : free to use / modify / sale in free and commercial software .  
Unique limit: MUST keep those copyright comments in all copies and in supporting documentation.  
usage demo : #define RUN_EXAMPLE_CLASSREGISTER and include this header file to run demo  
*********************************************************************/  
//#define RUN_EXAMPLE_CLASSREGISTER  
  
  
#ifdef COMPILE_EXAMPLE_ALL  
#define COMPILE_EXAMPLE_CLASSREGISTER  
#endif  
  
  
#ifdef RUN_EXAMPLE_ALL  
#define RUN_EXAMPLE_CLASSREGISTER  
#endif  
  
  
#if defined(RUN_EXAMPLE_CLASSREGISTER) && !defined(COMPILE_EXAMPLE_CLASSREGISTER)  
#define COMPILE_EXAMPLE_CLASSREGISTER  
#endif   
  
  
/  
//demo example implement begin //  
#ifdef COMPILE_EXAMPLE_CLASSREGISTER  
#include <string>  
#include <vector>  
#include "test_macroDef.h"  
  
  
namespace DemoUsage  
{  
typedef struct _tagDevice  
{  
stlString m_name;  
int m_id;  
double m_price;  
  
  
_tagDevice(stlString name,int id ,double price)  
{   
m_name = name;  
m_id = id;  
m_price = price;  
}  
  
  
bool operator==(_tagDevice const& other) const  
{  
return m_id == other.m_id;  
}  
  
  
stlString dumpStr()  
{  
#if 1==USE_UNICODE_STRING  
wchar_t buf[128] = {0};  
swprintf_s(buf,TXT("Name : %s\tid:%d\tprice:%f"),m_name.c_str(),m_id,m_price);  
#else  
char buf[128] = {0};  
sprintf_s(buf,TXT("Name : %s\tid:%d\tprice:%f"),m_name.c_str(),m_id,m_price);  
#endif  
  
  
return getStlString(buf);  
}  
  
  
}Device,*LPDevice;  
  
  
bool IsThisDevice(const Device& Dev, const int& p)  
{  
if (Dev.m_id == p)  
return true;  
  
  
return false;  
}  
  
  
class CDemoCast   
{  
public:  
virtual ~CDemoCast(){};  
virtual const type_info& getTypeInfo() const = 0;  
};  
  
  
class CGenBase : public CDemoCast  
{  
public:  
CGenBase(){;};  
virtual ~CGenBase(){;};  
  
  
};  
  
  
class CDri_Type1 :public CDemoCast  
{  
DECLARE_CLASS(CDri_Type1,CDemoCast);  
public:  
CDri_Type1(void){};  
virtual ~CDri_Type1(void){};  
  
  
virtual const type_info& getTypeInfo() const  
{  
return typeid(CDri_Type1);  
}  
  
  
int m_CDri_Type1;  
};  
  
  
struct DemoTraintType {}; //define type DemoTraintType for creating object  
  
  
//IMPLEMENT_CLASS(CDri_Type1,CDemoCast);  
//IMPLEMENT_CLASS_BIND_DAT(CDri_Type1,CDemoCast,2);  
IMPLEMENT_CLASS_BIND_TYPE(CDri_Type1,CDemoCast,DemoTraintType); //Test with pNew7, pNew7 will be not CDri_Type1 because predefined base is different  
  
  
class CDri_Type2 :public CGenBase  
{  
DECLARE_CLASS(CDri_Type2,CGenBase);  
public:  
CDri_Type2(void){};  
virtual ~CDri_Type2(void){};  
  
  
virtual const type_info& getTypeInfo() const  
{  
return typeid(CDri_Type2);  
}  
  
  
int m_CDri_Type2;  
};   
  
  
//IMPLEMENT_CLASS(CDri_Type2,CGenBase);  
//IMPLEMENT_CLASS_BIND_DAT(CDri_Type2,CGenBase,2);  
IMPLEMENT_CLASS_BIND_DAT(CDri_Type2,CGenBase,DemoTraintType()); //Test with pNew7 by createObjectByTraitType  
  
  
class CDri_Value1 : public CGenBase  
{  
DECLARE_CLASS(CDri_Value1,CGenBase);   
public:  
  
  
CDri_Value1(void){};  
virtual ~CDri_Value1(void){};  
  
  
virtual const type_info& getTypeInfo() const  
{  
return typeid(CDri_Value1);  
}  
  
  
int m_CDri_1;  
};  
  
  
//IMPLEMENT_CLASS(CDri_Value1,CGenBase); //Test with pNew1, pCls  
IMPLEMENT_CLASS_BIND_DAT(CDri_Value1,CGenBase,1); //Test with pNew2,pNew3  
//IMPLEMENT_CLASS_BIND_DAT(CDri_Value1,CGenBase,Device("CDri_Value1",1,1.11)); //Test with pNew4  
  
  
class CDri_Value2 :public CGenBase  
{  
DECLARE_CLASS(CDri_Value2,CGenBase);  
public:  
CDri_Value2(void){};  
virtual ~CDri_Value2(void){};  
  
virtual const type_info& getTypeInfo() const  
{  
return typeid(CDri_Value2);  
}  
  
  
int m_CDri_2;  
};  
  
  
//IMPLEMENT_CLASS(CDri_Value2,CGenBase);  
//IMPLEMENT_CLASS_BIND_DAT(CDri_Value2,CGenBase,2);  
IMPLEMENT_CLASS_BIND_DAT(CDri_Value2,CGenBase,Device(TXT("Device_name_1"),2,2.34)); //Test with pNew4  
  
  
class CDri_Value3 :public CGenBase  
{  
DECLARE_CLASS(CDri_Value3,CGenBase);  
public:  
CDri_Value3(void){};  
virtual ~CDri_Value3(void){};  
  
  
virtual const type_info& getTypeInfo() const  
{  
return typeid(CDri_Value3);  
}  
  
  
int m_CDri_3;  
};  
  
  
//IMPLEMENT_CLASS(CDri_Value3,CGenBase);  
//IMPLEMENT_CLASS_BIND_DAT(CDri_Value3,CGenBase,2);  
IMPLEMENT_CLASS_BIND_DAT(CDri_Value3,CGenBase,Device(TXT("Device_name_2"),3,3.45)); //Test with pNew5  
  
  
template<int V>  
struct ValToRef  
{  
static const int value=V;  
};  
  
  
bool showUsageClassRegister()  
{  
using namespace UtilExt;  
  
  
PrintUsage();  
CGenBase* pNew1 = UtilExt::ClassRegisterEx<CGenBase>::createObjectByName(TXT("CDri_Value2")); //create CDri_Value2  
Assertb(typeid(CDri_Value2)==pNew1->getTypeInfo());  
CGenBase* pNew2 = UtilExt::ClassRegisterEx<CGenBase>::createObject(1); //create CDri_Value1  
Assertb(typeid(CDri_Value1)==pNew2->getTypeInfo());  
CGenBase* pNew3 = UtilExt::ClassRegisterEx<CGenBase>::createObject(2); //Create NULL , No type whose trait type is int and value is 2  
Assertb(NULL == pNew3);  
CGenBase* pNew4 = UtilExt::ClassRegisterEx<CGenBase>::createObject(2,IsThisDevice); //create CDri_Value2  
Assertb(typeid(CDri_Value2)==pNew4->getTypeInfo());  
CGenBase* pNew5 = UtilExt::ClassRegisterEx<CGenBase>::createObject(3,MMP(&Device::m_id)); //create CDri_Value3  
Assertb(typeid(CDri_Value3)==pNew5->getTypeInfo());  
UtilExt::ClassRegister* pCls = UtilExt::ClassRegisterEx<CGenBase>::getClassRegisterByName(TXT("CDri_Value1"));  
Assertb((NULL == pCls->getTypedRegister<CDemoCast>()));  
Assertb((NULL != pCls->getObjectCreater<CGenBase>()));  
CGenBase* pClsObj = pCls->tryCreateObject<CGenBase>();  
AssertP(pClsObj);  
  
  
UtilExt::ClassRegisterEx<CGenBase>* pExRegister = pCls->getTypedRegister<CGenBase>();   
CGenBase* pNew6 = pExRegister->createObject(); //create CDri_Value1  
Assertb(typeid(CDri_Value1)==pNew6->getTypeInfo());   
  
  
CGenBase* pNew7 = UtilExt::ClassRegisterEx<CGenBase>::createObjectByTraitType<DemoTraintType>(); //Create CDri_Type2  
Assertb(typeid(CDri_Type2)==pNew7->getTypeInfo());  
CDemoCast* pNew8 = UtilExt::ClassRegisterEx<CDemoCast>::createObjectByTraitType<DemoTraintType>(); //Create CDri_Type1  
Assertb(typeid(CDri_Type1)==pNew8->getTypeInfo());  
  
  
//display how many classes use Device as its trait type  
unsigned int iSize = UtilExt::ClassRegisterEx<CGenBase>::getClassRegisterNumberByTraitType<Device>();  
outputTxt((TXT("\nThere are %d type whose trait type is Device.\n"),iSize));  
  
for(unsigned int i=0;i<iSize;i++)  
{  
UtilExt::ClassRegisterEx<CGenBase>* pCur = UtilExt::ClassRegisterEx<CGenBase>::getClassRegisterByTraitType<Device>(i);  
outputTxt((TXT("No. %d class register with Device trait : \nName:%s\tDevice trait value : %s\n"),i,pCur->getName().c_str(),pCur->getTraitValue<Device>()->dumpStr().c_str()));  
}  
  
  
//display how many classes use int as its trait type  
iSize = UtilExt::ClassRegisterEx<CGenBase>::getClassRegisterNumberByTraitType<int>();  
outputTxt((TXT("\nThere are %d type whose trait type is int.\n"),iSize));  
  
  
for(unsigned int i=0;i<iSize;i++)  
{  
UtilExt::ClassRegisterEx<CGenBase>* pCur = UtilExt::ClassRegisterEx<CGenBase>::getClassRegisterByTraitType<int>(i);  
outputTxt((TXT("No. %d class register with int trait : \nName:%s\tint trait value : %d\n"),i,pCur->getName().c_str(),*pCur->getTraitValue<int>()));  
}  
  
  
outputTxt((TXT("\ndump ClassRegisterBase:\n")));  
UtilExt::ClassRegister::dump();  
  
  
outputTxt((TXT("dump ClassRegisterEx<CGenBase>:\n")));  
UtilExt::ClassRegisterEx<CGenBase>::dump();  
  
  
outputTxt((TXT("dump ClassRegisterEx<CDemoCast>:\n")));  
UtilExt::ClassRegisterEx<CDemoCast>::dump();  
  
  
delete pNew1;  
delete pNew2;  
delete pNew3;  
delete pNew4;  
delete pNew5;  
delete pNew6;  
delete pNew7;  
delete pNew8;  
delete pClsObj;  
  
  
return true;  
}  
  
  
#ifdef RUN_EXAMPLE_CLASSREGISTER  
InitRunFunc(showUsageClassRegister);  
InitRunFunc(showUsageClassRegister);  
#endif //RUN_EXAMPLE_CLASSREGISTER  
  
  
}  
  
  
INIT_REGISTER_IN_BASE_CPP(DemoUsage::CDemoCast);  
INIT_REGISTER_IN_BASE_CPP(DemoUsage::CGenBase);   
  
  
#endif // COMPILE_EXAMPLE_CLASSREGISTER  
  
  
#endif // __test_Class_Register_H__
