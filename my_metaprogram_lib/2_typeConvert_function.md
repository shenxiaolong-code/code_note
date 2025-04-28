2) 转换类型 typeConvert 

发布日期: 2013-10-07 14:26:55
原文链接: https://blog.csdn.net/Tonny0832/article/details/12383167

---

这些基于typeTraits提供的一些类型转换功能，比如典型的if-else等等，判断两个类型是否相同，是否原始类型相同，是否可转换，是否值可转换，是否基类派生类，是否序列类，是否数组，是否是原子类型，是否是内置类型，是否是枚举类型。

在一些模板设计中，有时候有这些需求，要求输出参数的修饰符(const/ref/T*)和输入参数的修饰符是一样的，即输入参数如果是引用，那么要求输出也是引用，输入如果是const，要求输出也是const的，输入如果是指针，要求输出也是指针，典型的应用：一个结构的成员类型和这个结构体的const类型肯定是一样的。

下面的实现提供了下面功能：

> 使用统一的格式为一个类型增加(add)/消除(remove)修饰符(const/&/violate)，

> 把一个类型在指针类型/非指针类型之间切换，

> 让一个类型的修饰符(const/&/violate)和另外一个类型的修饰符一样，

> 引用适配器(如果一个类型是引用类型，保持不变；如果不是引用类型：如果是原子类型，保持不变/如果不是原子类型，给该类型加上引用)以解决参数传递时临时对象导致的效率/正确性问题，

> 取得一个量(指针/非指针)的指针，

> 取得一个量(指针/非指针)的值，

> 取得一个量(多级指针/迭代器/智能指针嵌套)的最终值。

下面的源代码提供了这些转换：

如果你的项目中允许使用boost这类重量级库的话，还是推荐你使用boost，下面的源代码提供了一个可供选择的替代方案：如果你不想引入boost这种巨无霸或者你的项目不允许使用boost时。

源代码可以在我的资源中去下载(包括用法测试的源代码),只需要用VS建一个空的工程，然后引入我的测试头文件即可。然后测试函数会在main函数之前运行，在控制台窗口中可以看到输出。

#ifndef test_typeConvert_h__  
#define test_typeConvert_h__  
#include "testconfig.h"  
#include <utility/typeConvert.h>  
  
  
//#define RUN_EXAMPLE_SAMEQUALIFIER  
  
  
#ifdef COMPILE_EXAMPLE_ALL  
#define COMPILE_EXAMPLE_SAMEQUALIFIER  
#endif  
  
  
#ifdef RUN_EXAMPLE_ALL  
#define RUN_EXAMPLE_SAMEQUALIFIER  
#endif  
  
  
#if defined(RUN_EXAMPLE_SAMEQUALIFIER) && !defined(COMPILE_EXAMPLE_SAMEQUALIFIER)  
#define COMPILE_EXAMPLE_SAMEQUALIFIER  
#endif   
  
  
usage demo code//  
#ifdef COMPILE_EXAMPLE_SAMEQUALIFIER  
#include <vector>  
#include "test_macroDef.h"  
#include "test_data_def.h"  
  
  
using namespace std;  
  
  
namespace DemoUsage  
{  
struct ConstStruct  
{  
int m_a;  
std::vector<char> m_Chars;  
};  
  
  
template<typename DataType>  
struct CTraitType;  
  
  
template<>  
struct CTraitType<ConstStruct>  
{  
typedef char type;  
};  
  
  
template<typename DataType>  
typename UtilExt::SameAllQualifier<typename CTraitType<typename UtilExt::GetRawType<DataType>::type>::type,DataType>::type& RunTest(DataType& obj,unsigned int idx)  
{  
outputTxt((TXT("\n[RunTest]\ttype name [%s]\n\t\tconst qualifiers ==>%s\n"),PRINTPARAM(typeid(DataType).name()),  
UtilExt::IsConst<DataType>::value ? TXT("Const") : TXT("non-Const")));  
  
  
if (idx<obj.m_Chars.size())  
{  
return obj.m_Chars.at(idx);  
}  
else  
{  
return obj.m_Chars.back();  
}  
}  
  
  
template<typename DataType>  
void process(DataType& obj)  
{  
outputTxt((TXT("\n[process]\tType name [%s]\n\t\tconst qualifiers ==> %s\n\t\tvalue [%d]\n"),PRINTPARAM(typeid(DataType).name()),  
UtilExt::IsConst<DataType>::value ? TXT("Const") : TXT("non-Const"),obj));  
}  
  
  
template<typename T,template<class> class IsXXX>  
void testTwiceAddRemove()  
{  
typedef typename AddQualifier<IsXXX> AddHelper;  
typedef typename AddHelper::apply<T>::type Qualifier_T;  
Static_Assert((IsXXX<Qualifier_T>::value));  
  
  
typedef typename RemoveQualifier<IsXXX> RemoveHelper;  
typedef typename RemoveHelper::apply<T>::type NotQualifier_T;  
Static_Assert((!IsXXX<NotQualifier_T>::value));  
  
  
typedef typename SelectQualifier<TrueType,T,IsXXX>::type SelectType_True;  
Static_Assert((IsXXX<SelectType_True>::value));  
  
  
typedef typename SelectQualifier<FalseType,T,IsXXX>::type SelectType_False;  
Static_Assert(!(IsXXX<SelectType_False>::value));  
}  
  
  
  
  
template<typename T>  
void testAddRemove(T a)  
{  
//test Add const  
testTwiceAddRemove<T,IsConst>();  
typedef typename AddRemoveHelper<AddConst<_1> >::apply<T>::type Const_T;  
Static_Assert((IsConst<Const_T>::value));  
  
  
typedef typename AddRemoveHelper<RemoveConst<_1> >::apply<T>::type NotConst_T;  
Static_Assert((!IsConst<NotConst_T>::value));  
  
  
//test Add pointer  
testTwiceAddRemove<T,IsPointer>();  
typedef typename AddRemoveHelper<AddPointer<_1> >::apply<T>::type Pointer_T;  
Static_Assert((IsPointer<Pointer_T>::value));  
//test Remove pointer  
typedef typename AddRemoveHelper<RemovePointer<_1> >::apply<T>::type NotPointer_T;  
Static_Assert((!IsPointer<NotPointer_T>::value));  
  
  
//test Add reference  
testTwiceAddRemove<T,IsRef>();  
typedef typename AddRemoveHelper<AddRef<_1> >::apply<T>::type Ref_T;  
Static_Assert((IsRef<Ref_T>::value));  
//test Remove reference  
typedef typename AddRemoveHelper<RemoveRef<_1> >::apply<T>::type NotRef_T;  
Static_Assert((!IsRef<NotRef_T>::value));  
  
  
//test Add Volatile  
testTwiceAddRemove<T,IsVolatile>();  
typedef typename AddRemoveHelper<AddVolatile<_1> >::apply<T>::type Volatile_T;  
Static_Assert((IsVolatile<Volatile_T>::value));  
//test Remove Volatile  
typedef typename AddRemoveHelper<RemoveVolatile<_1> >::apply<T>::type NotVolatile_T;  
Static_Assert((!IsVolatile<NotVolatile_T>::value));  
  
  
//test SelectQualifier  
typedef typename SelectQualifier<TrueType,T,IsConst>::type ConstSelectTrue_T;  
Static_Assert((IsConst<ConstSelectTrue_T>::value));  
  
  
typedef typename SelectQualifier<FalseType,T,IsConst>::type ConstSelectFalse_T;  
Static_Assert(!(IsConst<ConstSelectFalse_T>::value));  
  
  
typedef typename SelectQualifier<TrueType,T,IsPointer>::type PointerSelectTrue_T;  
Static_Assert((IsPointer<PointerSelectTrue_T>::value));  
  
  
typedef typename SelectQualifier<FalseType,T,IsPointer>::type PointerSelectFalse_T;  
Static_Assert(!(IsPointer<PointerSelectFalse_T>::value));  
  
  
typedef typename SelectQualifier<TrueType,T,IsRef>::type RefSelectTrue_T;  
Static_Assert((IsRef<RefSelectTrue_T>::value));  
  
  
typedef typename SelectQualifier<FalseType,T,IsRef>::type RefSelectFalse_T;  
Static_Assert(!(IsRef<RefSelectFalse_T>::value));  
  
  
typedef typename SelectQualifier<TrueType,T,IsVolatile>::type VolatileSelectTrue_T;  
Static_Assert((IsVolatile<VolatileSelectTrue_T>::value));  
  
  
typedef typename SelectQualifier<FalseType,T,IsVolatile>::type VolatileSelectFalse_T;  
Static_Assert(!(IsVolatile<VolatileSelectFalse_T>::value));   
}  
  
  
bool showUsageTypeConvert()  
{  
PrintUsage();  
  
  
DemoUsage::S3 v3;  
testAddRemove(v3);  
  
  
const DemoUsage::S4 v4;  
testAddRemove(v4);  
  
  
const DemoUsage::S4 *p=NULL;  
testAddRemove(p);  
  
  
stlSmartptr<int> stlPtr;  
testAddRemove(stlPtr);  
  
  
ConstStruct obj;  
obj.m_a = 112;  
for (int i=0;i<10;i++)  
{  
obj.m_Chars.push_back('1'+i);  
}  
  
  
ConstStruct const& demo=obj;  
  
  
char ch = RunTest(obj,2);  
char const& cCh = RunTest(demo,3);  
  
  
process(RunTest(obj,5));  
process(RunTest(demo,6));  
  
  
//test IsSameType && IsSameRawType  
Static_Assert((UtilExt::IsSameType<int,int>::value));  
Static_Assert((UtilExt::IsSameRawType<int,int>::value));  
Static_Assert((!UtilExt::IsSameType<int,char>::value));  
Static_Assert((!UtilExt::IsSameRawType<int,char>::value));  
Static_Assert((!UtilExt::IsSameType<float,double>::value));  
Static_Assert((!UtilExt::IsSameRawType<float,double>::value));  
Static_Assert((!UtilExt::IsSameType<int,const int>::value));  
Static_Assert((UtilExt::IsSameRawType<int,const int>::value));  
Static_Assert((!UtilExt::IsSameType<int,int*>::value));  
Static_Assert((!UtilExt::IsSameRawType<int,int*>::value));  
Static_Assert((!UtilExt::IsSameType<int,volatile int>::value));  
Static_Assert((!UtilExt::IsSameRawType<volatile int,int*>::value));  
Static_Assert((!UtilExt::IsSameType<int&,int>::value));  
Static_Assert((UtilExt::IsSameRawType<int,int&>::value));  
  
  
//demo If_T and lazy_If_T  
Static_Assert((UtilExt::IsSameType<UtilExt::If_T<true,int,char>::type,int>::value));  
Static_Assert((!UtilExt::IsSameType<UtilExt::If_T<false,int,char>::type,int>::value));  
Static_Assert((UtilExt::IsSameType<UtilExt::If_T<false,int,char>::type,char>::value));  
Static_Assert((!UtilExt::IsSameType<UtilExt::If_T<true,int,char>::type,char>::value));  
  
  
//demo SameAllQualifier  
Static_Assert((UtilExt::IsSameType<typename UtilExt::SameAllQualifier<int, float const*>::type,int const*>::value));  
Static_Assert((!UtilExt::IsSameType<typename UtilExt::SameAllQualifier<int,const float>::type,const float>::value));  
Static_Assert((UtilExt::IsSameType<typename UtilExt::SameAllQualifier<const int,volatile float>::type,volatile int>::value));  
Static_Assert((!UtilExt::IsSameType<typename UtilExt::SameAllQualifier<const int,volatile char>::type,char>::value));  
  
  
//demo RefAdapter  
Static_Assert((!UtilExt::IsRef<typename UtilExt::RefAdapter<int>::type>::value)); //if it is build-in type,keep original type : ref type / non-ref type  
Static_Assert((UtilExt::IsRef<typename UtilExt::RefAdapter<int&>::type>::value));  
Static_Assert((UtilExt::IsSameType<typename UtilExt::RefAdapter<int&>::type,int&>::value));   
Static_Assert((UtilExt::IsSameType<typename UtilExt::RefAdapter<int>::type,int>::value));   
Static_Assert(!(UtilExt::IsSameType<typename UtilExt::RefAdapter<int>::type,int&>::value));   
Static_Assert(!(UtilExt::IsSameType<typename UtilExt::RefAdapter<int&>::type,int>::value));   
Static_Assert(!(UtilExt::IsRef<typename UtilExt::RefAdapter<const int>::type>::value));   
Static_Assert((UtilExt::IsSameType<typename UtilExt::RefAdapter<int&>::type,int&>::value));  
Static_Assert((UtilExt::IsSameType<typename UtilExt::RefAdapter<int>::type,int>::value));  
Static_Assert(!(UtilExt::IsSameType<typename UtilExt::RefAdapter<int>::type,int&>::value));  
Static_Assert(!(UtilExt::IsSameType<typename UtilExt::RefAdapter<int&>::type,bool>::value));  
Static_Assert(!(UtilExt::IsSameType<typename UtilExt::RefAdapter<int&>::type,char>::value));   
Static_Assert(!(UtilExt::IsSameType<typename UtilExt::RefAdapter<int>::type,bool>::value));  
Static_Assert(!(UtilExt::IsSameType<typename UtilExt::RefAdapter<int>::type,char>::value));  
Static_Assert(!(UtilExt::IsSameType<typename UtilExt::RefAdapter<S3&>::type,S3>::value)); //if it is not build-in type, it is always ref-type  
Static_Assert(!(UtilExt::IsSameType<typename UtilExt::RefAdapter<S3>::type,S3>::value));  
Static_Assert((UtilExt::IsSameType<typename UtilExt::RefAdapter<S3&>::type,S3&>::value));  
Static_Assert((UtilExt::IsSameType<typename UtilExt::RefAdapter<S3>::type,S3&>::value));  
Static_Assert((UtilExt::IsSameType<typename UtilExt::RefAdapter<S3*>::type,S3*>::value));  
Static_Assert((UtilExt::IsSameType<typename UtilExt::RefAdapter<stlVector<S3>::iterator>::type,stlVector<S3>::iterator>::value));  
Static_Assert((UtilExt::IsSameType<typename UtilExt::RefAdapter<stlSmartptr<S3> >::type,stlSmartptr<S3>>::value));  
  
  
//test RefAdapter  
Static_Assert((UtilExt::IsRef<typename UtilExt::RefAdapter<ConstStruct>::type>::value));  
Static_Assert((UtilExt::IsRef<typename UtilExt::RefAdapter<int[20]>::type>::value));  
Static_Assert((UtilExt::IsRef<typename UtilExt::RefAdapter<ConstStruct[20]>::type>::value)); //need reference because original type is not reference type  
Static_Assert((UtilExt::IsRef<typename UtilExt::RefAdapter<int(&)[20]>::type>::value)); //RefAdapter will give reference type for reference type  
Static_Assert((UtilExt::IsRef<typename UtilExt::RefAdapter<ConstStruct(&)[20]>::type>::value)); //RefAdapter will give reference type for reference type  
  
  
//test same const  
//demo SameConst  
Static_Assert((UtilExt::IsSameType<typename UtilExt::SameConst<int,const float>::type,const int>::value));  
Static_Assert((!UtilExt::IsSameType<typename UtilExt::SameConst<int,const float>::type,const float>::value));  
Static_Assert((UtilExt::IsSameType<typename UtilExt::SameConst<const int,float>::type,int>::value));  
Static_Assert((!UtilExt::IsSameType<typename UtilExt::SameConst<const int,char>::type,char>::value));  
Static_Assert((UtilExt::IsConst<UtilExt::SameConst<int,const char>::type>::value));  
Static_Assert((UtilExt::IsConst<UtilExt::SameConst<const int,const char>::type>::value));  
Static_Assert((!UtilExt::IsConst<UtilExt::SameConst<const int,char>::type>::value));  
Static_Assert((!UtilExt::IsConst<UtilExt::SameConst<int,char>::type>::value));  
  
  
//test Same SameRef  
Static_Assert((UtilExt::IsRef<UtilExt::SameRef<int&,const char&>::type>::value));  
Static_Assert((UtilExt::IsRef<UtilExt::SameRef<int,const char&>::type>::value));  
Static_Assert((!UtilExt::IsRef<UtilExt::SameRef<int&,char>::type>::value));  
Static_Assert((!UtilExt::IsRef<UtilExt::SameRef<int,char>::type>::value));  
  
  
//test Same SamePointer  
Static_Assert((UtilExt::IsPointer<UtilExt::SamePointer<int&,const char*>::type>::value));  
Static_Assert((UtilExt::IsPointer<UtilExt::SamePointer<int,const char*>::type>::value));  
Static_Assert((!UtilExt::IsPointer<UtilExt::SamePointer<int&,char>::type>::value));  
Static_Assert((!UtilExt::IsPointer<UtilExt::SamePointer<int,char>::type>::value));  
  
  
//test Same SameVolatile  
Static_Assert((UtilExt::IsVolatile<UtilExt::SameVolatile<int,volatile char>::type>::value));  
Static_Assert((UtilExt::IsVolatile<UtilExt::SameVolatile<int,volatile char>::type>::value));  
Static_Assert((!UtilExt::IsVolatile<UtilExt::SameVolatile<int,char>::type>::value));  
Static_Assert((!UtilExt::IsVolatile<UtilExt::SameVolatile<int,char>::type>::value));  
  
  
//demo Get::pointer and Get::value  
ConstStruct rObj;  
ConstStruct* pObj11 = UtilExt::Get::pointer(rObj);  
ConstStruct* pObj12 = UtilExt::Get::pointer(pObj11);  
  
  
ConstStruct& rVal11 = UtilExt::Get::value(rObj);  
ConstStruct& rVal12 = UtilExt::Get::value(pObj12);   
  
  
stlSmartptr<ConstStruct> pStlPtr(new ConstStruct()); //test smart pointer  
ConstStruct* pObj21 = UtilExt::Get::pointer(pStlPtr);  
ConstStruct& rVal21 = UtilExt::Get::value(pObj21);  
ConstStruct& rVal22 = UtilExt::Get::value(pStlPtr);  
ConstStruct& rVal23 = UtilExt::Get::value(pObj21);  
  
  
stlVector<ConstStruct> arr;  
arr.push_back(rObj); //test iterator  
stlVector<ConstStruct>::iterator pIt = arr.begin();   
ConstStruct* pObj31 = UtilExt::Get::pointer(pIt);   
ASSERT_AND_LOG((rObj.m_a == pObj31->m_a));  
ConstStruct& rVal32 = UtilExt::Get::value(pObj31);  
ConstStruct& rVal33 = UtilExt::Get::value(pIt);  
  
  
//demo Get::finalValue  
S4 objs[ContainerSize10];  
std::vector<S4*> rArr;  
std::list<std::vector<S4*>::iterator> rList;  
Static_Assert((IsSameType<GetFinalValueType<std::list<std::vector<S4*>::iterator>::iterator>::type,S4>::value));   
  
  
cout<<"\n[Get::finalValue] forEach S4::printNoParam before S4::m_s4=i*i:\n";  
forEach(makeDataSet(objs,ContainerSize10),makeUnaryFunction(&S4::printNoParam,_$));   
cout << endl;  
for(int i=0;i<10;i++)  
{   
rArr.push_back(&objs[i]);  
rList.push_back(--rArr.end());  
Get::finalValue(--rList.end()).m_s4 = i*i;  
}  
cout<<"\n[Get::finalValue] forEach S4::printNoParam after S4::m_s4=i*i:\n";  
forEach(makeDataSet(objs,10),makeUnaryFunction(&S4::printNoParam,_$));  
cout << endl;  
  
  
return true;  
}  
  
  
Static_Assert(5);  
Static_Assert(5!=0);  
  
  
#ifdef RUN_EXAMPLE_SAMEQUALIFIER  
InitRunFunc(showUsageTypeConvert);  
#endif //RUN_EXAMPLE_SAMEQUALIFIER  
}  
#endif  
  
  
#endif // test_typeConvert_h__  


  


  

