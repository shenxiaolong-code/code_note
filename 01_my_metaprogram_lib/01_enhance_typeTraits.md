1)  获取类型特征 typeTraits

发布日期: 2013-10-07 14:18:50
原文链接: https://blog.csdn.net/Tonny0832/article/details/12382911

---

源代码可以在我的资源中去下载(包括用法测试的源代码):  
http://download.csdn.net/detail/tonny0832/7426991  
只需要用VS建一个空的工程，然后引入我的测试头文件即可。然后测试函数会在main函数之前运行，在控制台窗口中可以看到输出。  


  


由于在我们的项目中，不能使用std及其它的主流库(boost等)，在需要判断一些类型特征及类型转换时，就只能自己写这些功能了。所以为了元代码的需要，我在我们的项目们自己写了这些type trait代码。

type trait是一切模板元编程的基础组件，所以它应该是模板元编程中非配置/宏定义功能集中的第一个文件，type trait(类型特征)如同普通编程中的if..else判断一样广泛使用，它通常决定了链接器在链接时应该链接哪个函数，在编译器识别正确的类型时，它也是必须的组件。

这方面功能的实现当然是boost实现得更好：如果在你的项目中允许使用boost这种重量级的库时，那么还是推荐你使用boost，如果不能，那么这些源代码可供替代使用)

这个文件定义了实现了一些基本的功能：判断一个类型是否是引用/指针/voliate/常量/函数指针，是否是STL容器，是否数组，是否内置类型，是否原子数据，比较两个带修饰符类型是否相同，是否不带修饰符类型相同，两个类型是否可以转换，是否两个类型的值可以相互转换(char==>int)，两个类型是否具有派生关系。

取得一个类型的原始类型(e.g. const int&==> int)，

取得一个指针/智能指针/迭代器所指示的(第一级)值类型(e.g. smartPtr<int> ==> int / int* ==> int / vector<int>::iterator ==> int )

取得一嵌套指针/智能指针/迭代器的最终值类型(非指针/智能指针/迭代器类型)，e.g ：int*==>int, int** ==> int , int*** == >int / vector<list<vector<int>::iterator>::iterator>::iterator ==> int / smartPtr<smartPtr<int> > ==> int

  


#ifndef typeTrait_h__  
#define typeTrait_h__  
#include "libconfig.h"  
#include "typeTraits.h"  
#include "placeHolder.h"  
#include "typeLogic.h"  
  
  
/********************************************************************  
Description : capture type's property, and it is independent on OS and compiler.  
e.g. isConst , IsVolatile , isReference , isPointer,isBuildInType  
getRawType of QualifiedType(e.g. raw type of "const int*" is int )  
isRawType , isSameType , IsBaseDerive, IsConvertable , IsValueConvertable  
Author : Shen.Xiaolong (Shen Tony) (2010-2013)  
Mail : xlshen2002@hotmail.com, xlshen@126.com  
verified platform : VS2008  
copyright: : latest Version of The Code Project Open License (CPOL : http://www.codeproject.com/)  
*********************************************************************/  
  
  
#ifdef _MSC_VER  
#pragma warning(disable:4244)  
#endif  
  
  
#define SA(X,Y) typename SameAllQualifier<X,Y>::type //Set X by Y (Same All Qualifier)  
#define SC(X,Y) typename SameConst<X,Y>::type //Set X by Y (Same Const)  
#define DC(X) typename Evl_If_T<IsConst<X>,RemoveConst<X>,AddConst<X> >::type::type  
  
  
#define RT(X) typename AddRef<X>::type //always get reference type  
#define NR(X) typename RemoveRef<X>::type //always get non-reference type  
#define VT(X) typename GetValueTypeFromPointer<X>::type //get value type from pointer type  
  
  
#define RAWTYPE(X) typename UtilExt::GetRawType<X>::type  
  
  
namespace UtilExt  
{   
/  
struct NullType;  
typedef char Yes_Type;  
typedef struct { char dummy[2]; } No_Type;  
  
  
template<bool x> struct BoolType { enum { value=(x) }; };  
struct TrueType : BoolType<true> { };  
struct FalseType : BoolType<false> { };  
/  
  
template<typename T> struct Type2Type { typedef T type; };  
  
  
//Int2Type::value is compile-period constant expression, it can apply for compile period calculation, e.g Static_Assert  
//and the T can only integer or enum type  
template<int Val> struct Int2Type { enum { value=Val }; };  
  
  
//Value2Type::value is not compile-period constant expression, it can't apply for compile period calculation, e.g Static_Assert  
//but it works well in runtime period, and it can hold non-integer type, e.g. function pointer, member pointer etc.  
template<typename T,T Val> struct Value2Type : public Type2Type<T> { static const T value; };  
template<typename T,T Val> T const Value2Type<T,Val>::value=Val;  
  
  
template<typename T,T Val,template<class T,T Val> class Impl_T>   
typename Impl_T<T,Val>::type getTypeValue(Impl_T<T,Val> const&) { return Val; };  
  
  
//forward declaration and helper///  
template<typename T> struct IsStlIterator;  
template<typename T> struct sfinae_helper : Type2Type<NullType> {};   
  
//if T has not nested 'type', then nested 'type' is T itself , else nested 'type' is 'T::type'  
template<typename T,typename P=NullType> struct GetNestedType : public Type2Type<T> {};  
template<typename T> struct GetNestedType<T,typename sfinae_helper<typename T::type>::type> : public Type2Type<typename T::type> {};  
  
  
//if T has not nested 'value', then nested 'value' is T itself , else nested 'value' is 'T::value'  
template<typename T,bool b=(0!=T::value)> struct GetBoolValue : BoolType<b> {}; //No test code  
template<bool b> struct GetBoolValue<bool,b> : BoolType<b> {}; //No test code  
  
  
if..else... select interfaces /  
//default is true  
template<bool,typename True_T,typename False_T> struct If_T : Type2Type<True_T> {} ;  
template<typename True_T,typename False_T> struct If_T<false,True_T,False_T> : Type2Type<False_T> {} ;  
template<typename B,typename True_T,typename False_T> struct Evl_If_T : public If_T<GetBoolValue<B>::value,True_T,False_T> {};  
  
  
template<template<class> class T> struct AddRemoveTrait;  
template<template<class> class AddQualifier_T, template<class> class RemoveQualifier_T>  
struct AddRemoveDef  
{  
typedef AddQualifier_T<_1> Add_T;  
typedef RemoveQualifier_T<_1> Remove_T;  
};   
  
  
Is interfaces /  
template<typename T> struct IsRef : FalseType {};   
template<typename T> struct IsRef<T&> : TrueType {};  
template<> struct IsRef<void> : FalseType {};  
  
  
template<typename T> struct AddRef : public Type2Type<T&> {};  
template<typename T> struct AddRef<T&> : public Type2Type<T&> {};  
template<> struct AddRef<void> : public Type2Type<void>{};  
template<typename T,unsigned int LEN> struct AddRef<T(&)[LEN]> : public Type2Type<T(&)[LEN]> {};  
template<typename T,unsigned int LEN> struct AddRef<T[LEN]> : public Type2Type<T(&)[LEN]> {};  
  
  
  
  
template<typename T> struct RemoveRef : public Type2Type<T> {};  
template<typename T> struct RemoveRef<T&> : public Type2Type<T> {};  
template<> struct RemoveRef<void> : public Type2Type<void>{};  
template<typename T,unsigned int LEN> struct RemoveRef<T(&)[LEN]> : public Type2Type<T[LEN]> {};  
template<typename T,unsigned int LEN> struct RemoveRef<T[LEN]> : public Type2Type<T[LEN]> {};  
  
  
template<> struct AddRemoveTrait<IsRef> : public AddRemoveDef<AddRef,RemoveRef> {};  
/  
template<typename T> struct IsSmartPointer : FalseType {};  
template<typename T> struct IsSmartPointer<stlSmartptr<T> > : TrueType {};  
  
  
template<typename T> struct IsMemberPointer : FalseType {};  
template<typename S,typename M> struct IsMemberPointer<M S::*> : TrueType {};  
  
  
template<typename T> struct IsPointer : logic::Or_T< IsSmartPointer<T> , IsStlIterator<T> > {};  
template<typename T> struct IsPointer<T*> : TrueType {};   
template<typename T> struct IsPointer<T*&> : TrueType {};   
  
  
template<typename T,typename S=NullType> struct AddPointerImpl : Type2Type<typename RemoveRef<T>::type*> {}; //pointer to reference is illegal  
template<typename T> struct AddPointerImpl<T,typename sfinae_helper<typename T::iterator_category>::type> : Type2Type<T> {};  
  
template<typename T> struct AddPointer : AddPointerImpl<T> {};   
template<typename T> struct AddPointer<T*> : Type2Type<T*> {};  
template<typename T> struct AddPointer<T&> : Type2Type<T*&> {};  
template<typename T> struct AddPointer<T*&> : Type2Type<T*&> {};  
template<typename T> struct AddPointer<stlSmartptr<T> > : Type2Type<stlSmartptr<T> > {}; //TODO : Not tested, No test code  
  
  
template<typename T,typename S=NullType> struct RemovePointerImpl : Type2Type<T> {};   
template<typename T> struct RemovePointerImpl<T,typename sfinae_helper<typename T::iterator_category>::type> : Type2Type<typename T::value_type> {};  
  
template<typename T> struct RemovePointer : RemovePointerImpl<T> {};  
template<typename T> struct RemovePointer<T*> : Type2Type<T> {};  
template<typename T> struct RemovePointer<T*&> : Type2Type<T&> {};  
template<typename T> struct RemovePointer<stlSmartptr<T> >:Type2Type<T> {}; //TODO : Not tested, No test code  
  
  
template<> struct AddRemoveTrait<IsPointer> : public AddRemoveDef<AddPointer,RemovePointer> {};  
  
  
/  
template<typename T> struct IsConst : FalseType {};   
template<typename T> struct IsConst<const T> : TrueType {};  
template<typename T> struct IsConst<const T&> : TrueType {};  
template<typename T> struct IsConst<T const*> : TrueType {}; //pointer point to constant ,pointer can be moved.  
template<typename T> struct IsConst<T const* const> : TrueType {}; //pointer point to constant ,pointer can be moved.  
template<typename T> struct IsConst<T* const> : FalseType {}; //const pointer point to variable, pointer can't be moved  
template<typename T,unsigned int LEN> struct IsConst<T[LEN]> : FalseType {}; //array style  
template<typename T,unsigned int LEN> struct IsConst<const T[LEN]> : TrueType {}; //array style  
template<typename T,unsigned int LEN> struct IsConst<T(&)[LEN]> : FalseType {}; //array ref style  
template<typename T,unsigned int LEN> struct IsConst<const T(&)[LEN]> : TrueType {}; //array ref style  
  
  
template<typename T> struct AddConst : Type2Type<typename Evl_If_T<IsPointer<T>,typename RemovePointer<T>::type const*,T const>::type >{};  
template<typename T> struct AddConst<T&> : Type2Type<T const&> {};   
template<typename T> struct AddConst<T const&> : Type2Type<T const&> {};   
template<typename T> struct AddConst<T const> : Type2Type<T const> {};  
template<typename T> struct AddConst<T const*> : Type2Type<T const*> {}; //pointer point to constant ,pointer can be moved.  
template<typename T> struct AddConst<T const* const> : Type2Type<T const* const> {}; //pointer point to constant ,pointer can be moved.  
template<typename T> struct AddConst<T* const> : Type2Type<T const*> {}; //const pointer point to variable, pointer can't be moved  
template<typename T,unsigned int LEN> struct AddConst<T[LEN]> : Type2Type<const T[LEN]> {}; //array style  
template<typename T,unsigned int LEN> struct AddConst<const T[LEN]> : Type2Type<const T[LEN]> {}; //array style  
template<typename T,unsigned int LEN> struct AddConst<T(&)[LEN]> : Type2Type<const T(&)[LEN]>{}; //array ref style  
template<typename T,unsigned int LEN> struct AddConst<const T(&)[LEN]>: Type2Type<const T(&)[LEN]>{}; //array ref style  
  
  
template<typename T> struct RemoveConst : Type2Type<T> {};  
template<typename T> struct RemoveConst<T&> : Type2Type<T&> {};   
template<typename T> struct RemoveConst<T const&> : Type2Type<T&> {};   
template<typename T> struct RemoveConst<T const> : Type2Type<T> {};  
template<typename T> struct RemoveConst<T const*> : Type2Type<T*> {}; //pointer point to constant ,pointer can be moved.  
template<typename T> struct RemoveConst<T const* const> : Type2Type<T*> {}; //pointer point to constant ,pointer can be moved.  
template<typename T> struct RemoveConst<T* const> : Type2Type<T*> {}; //const pointer point to variable, pointer can't be moved  
template<typename T,unsigned int LEN> struct RemoveConst<T[LEN]> : Type2Type<T[LEN]> {}; //array style  
template<typename T,unsigned int LEN> struct RemoveConst<const T[LEN]> : Type2Type<T[LEN]> {}; //array style  
template<typename T,unsigned int LEN> struct RemoveConst<T(&)[LEN]> : Type2Type<T (&)[LEN]> {}; //array ref style  
template<typename T,unsigned int LEN> struct RemoveConst<const T(&)[LEN]> : Type2Type<T (&)[LEN]> {}; //array ref style  
  
  
template<> struct AddRemoveTrait<IsConst> : public AddRemoveDef<AddConst,RemoveConst>{};  
/  
template<typename T> struct IsVolatile : FalseType {};   
template<typename T> struct IsVolatile<volatile T> : TrueType{};  
  
  
template<typename T> struct AddVolatile : Type2Type<volatile T> {};  
template<typename T> struct AddVolatile<volatile T> : Type2Type<volatile T> {};  
  
  
template<typename T> struct RemoveVolatile : Type2Type<T> {};  
template<typename T> struct RemoveVolatile<volatile T> : Type2Type<T> {};   
  
  
template<> struct AddRemoveTrait<IsVolatile> : public AddRemoveDef<AddVolatile,RemoveVolatile>{};  
/  
#if HAS_BOOST_SUPPORT==0  
//only max support 5 parameter  
template<typename T> struct IsFunctionPointer : FalseType {};  
template<typename R> struct IsFunctionPointer<R (*)()> : TrueType {};  
template<typename R> struct IsFunctionPointer<R (*)(...)> : TrueType {};  
template<typename R,typename P1> struct IsFunctionPointer<R (*)(P1)> : TrueType {};  
template<typename R,typename P1> struct IsFunctionPointer<R (*)(P1,...)> : TrueType {};  
template<typename R,typename P1,typename P2> struct IsFunctionPointer<R (*)(P1,P2)> : TrueType {};  
template<typename R,typename P1,typename P2> struct IsFunctionPointer<R (*)(P1,P2,...)> : TrueType {};  
template<typename R,typename P1,typename P2,typename P3> struct IsFunctionPointer<R (*)(P1,P2,P3)> : TrueType {};  
template<typename R,typename P1,typename P2,typename P3> struct IsFunctionPointer<R (*)(P1,P2,P3,...)> : TrueType {};  
template<typename R,typename P1,typename P2,typename P3,typename P4> struct IsFunctionPointer<R (*)(P1,P2,P3,P4)> : TrueType {};  
template<typename R,typename P1,typename P2,typename P3,typename P4> struct IsFunctionPointer<R (*)(P1,P2,P3,P4,...)> : TrueType {};  
template<typename R,typename P1,typename P2,typename P3,typename P4,typename P5> struct IsFunctionPointer<R (*)(P1,P2,P3,P4,P5)> : TrueType {};  
template<typename R,typename P1,typename P2,typename P3,typename P4,typename P5> struct IsFunctionPointer<R (*)(P1,P2,P3,P4,P5,...)> : TrueType {};  
  
  
template<typename T> struct IsMemberFunctionPointer : FalseType {};  
template<typename R,typename O> struct IsMemberFunctionPointer<R (O::*)()> : TrueType {};  
template<typename R,typename O> struct IsMemberFunctionPointer<R (O::*)(...)> : TrueType {};  
template<typename R,typename O,typename P1> struct IsMemberFunctionPointer<R (O::*)(P1)> : TrueType {};  
template<typename R,typename O,typename P1> struct IsMemberFunctionPointer<R (O::*)(P1,...)> : TrueType {};  
template<typename R,typename O,typename P1,typename P2> struct IsMemberFunctionPointer<R (O::*)(P1,P2)> : TrueType {};  
template<typename R,typename O,typename P1,typename P2> struct IsMemberFunctionPointer<R (O::*)(P1,P2,...)> : TrueType {};  
template<typename R,typename O,typename P1,typename P2,typename P3> struct IsMemberFunctionPointer<R (O::*)(P1,P2,P3)> : TrueType {};  
template<typename R,typename O,typename P1,typename P2,typename P3> struct IsMemberFunctionPointer<R (O::*)(P1,P2,P3,...)> : TrueType {};  
template<typename R,typename O,typename P1,typename P2,typename P3,typename P4> struct IsMemberFunctionPointer<R (O::*)(P1,P2,P3,P4)> : TrueType {};  
template<typename R,typename O,typename P1,typename P2,typename P3,typename P4> struct IsMemberFunctionPointer<R (O::*)(P1,P2,P3,P4,...)> : TrueType {};  
template<typename R,typename O,typename P1,typename P2,typename P3,typename P4,typename P5> struct IsMemberFunctionPointer<R (O::*)(P1,P2,P3,P4,P5)> : TrueType {};  
template<typename R,typename O,typename P1,typename P2,typename P3,typename P4,typename P5> struct IsMemberFunctionPointer<R (O::*)(P1,P2,P3,P4,P5,...)> : TrueType {};  
#endif  
/   
template<typename T> struct GetRawType; //forward declaration, below interface will use GetRawType interface  
template<typename T1, typename T2> struct IsSameType : FalseType {};   
template<typename T> struct IsSameType<T,T> : TrueType {};  
template<typename T1, typename T2> struct IsSameRawType : IsSameType<typename GetRawType<T1>::type,typename GetRawType<T2>::type>{};  
  
template<typename T> struct IsRawType : IsSameType<T,typename GetRawType<T>::type> {};  
  
  
/  
template<typename T>  
struct IsStlIterator  
{ //borrow idea from boost lib  
template<typename U,typename S=NullType> struct IsIteratorImpl : FalseType {};  
template<typename U> struct IsIteratorImpl<U,typename sfinae_helper<typename U::iterator_category>::type> : TrueType {};  
  
  
enum{value=IsIteratorImpl<T>::value}; //PrivateHelper::TypeHelper  
};  
  
  
template<typename T>  
struct IsStlContainer  
{   
//borrow idea from boost lib  
template<typename U,typename S=NullType> struct CHasIterator : FalseType {};  
template<typename U> struct CHasIterator<U,typename sfinae_helper<typename U::iterator>::type> : TrueType {};  
  
  
enum{value= logic::And_T<logic::Not_T<IsDataSet<T> >,CHasIterator<RAWTYPE(T)> >::value};  
};  
  
  
template<typename T>  
struct IsIterator   
: logic::Or_T <logic::And_T<IsPointer<T>,logic::Not_T<IsSmartPointer<T> > ,logic::Not_T<IsFunctionPointer<T> > ,logic::Not_T<IsMemberFunctionPointer<T> > >,IsStlIterator<T> >{};  
/  
template<typename S> struct GetValueTypeFromPointer  
{  
public:  
template<bool bisIterator> struct CImpl;  
template<> struct CImpl<true>  
{  
template<typename T> struct apply  
{  
typedef typename T::value_type type;  
};  
};  
  
  
template<> struct CImpl<false>  
{  
template<typename T> struct apply  
{  
typedef T type;  
};  
};  
  
  
typedef typename CImpl<IsIterator<S>::value>::template apply<S> H;  
  
  
public:  
typedef typename H::type type;  
};  
  
  
template<typename T> struct GetValueTypeFromPointer<stlSmartptr<T> > : Type2Type<T> {};  
template<typename T> struct GetValueTypeFromPointer<T*> : Type2Type<T> {};  
  
  
template<typename T> struct GetFinalValueType : Evl_If_T<IsPointer<typename GetValueTypeFromPointer<T>::type >,GetFinalValueType<typename GetValueTypeFromPointer<T>::type>,GetValueTypeFromPointer<T> >::type {};  
  
  
/  
  
  
namespace PrivateHelper  
{   
template<typename FromT,typename ToT> struct ConvertHelper  
{   
static UtilExt::No_Type test(...);  
static UtilExt::Yes_Type test(ToT);  
static FromT makeFrom();  
  
  
enum {value = sizeof(UtilExt::Yes_Type) == sizeof(test(makeFrom())) };  
};  
}  
  
//Note : int and int* is not convertable , because they are different type : int type and pointer type  
//similar with char and char*, sizeof(char)=1,sizeof(char*)=4  
template<typename FromT, typename ToT> struct IsConvertable  
{//in order to independent on OS and compiler , don't use MS specific : __is_convertible_to   
enum {value = PrivateHelper::ConvertHelper<FromT,ToT>::value };  
};  
  
  
//Note : int and int* is not convertable , but they are value convertable.  
template<typename FromT, typename ToT> struct IsValueConvertable  
{  
private:   
typedef PrivateHelper::ConvertHelper<typename GetValueTypeFromPointer<FromT>::type,typename GetValueTypeFromPointer<ToT>::type> H;  
  
  
public:  
enum {value = H::value };  
};  
  
  
/  
  
  
template<typename BaseT, typename DeriveT> struct IsBaseDerive  
{ //in order to independent on OS and compiler , don't use MS specific : __is_base_of   
private:  
typedef PrivateHelper::ConvertHelper<typename GetRawType<typename AddPointer<DeriveT>::type>::type,typename GetRawType<typename AddPointer<BaseT>::type>::type> H;   
//filter int/char/float convertible  
  
  
public:  
enum { value = logic::And_T<H,logic::Not_T<IsSameRawType<BaseT,DeriveT> > >::value };  
};  
  
  
template<typename DeriveT> struct IsBaseDerive<void,DeriveT>  
{  
enum {value = false};  
};  
  
  
/  
  
  
template<typename BaseT, typename DeriveTOrBaseT> struct IsSerialClass : logic::Or_T<IsBaseDerive<BaseT,DeriveTOrBaseT>,IsSameRawType<BaseT,DeriveTOrBaseT> >{};  
  
  
/  
  
  
template<typename S> struct isArray : FalseType {}; //array pointer is not array  
template<typename S,unsigned int LEN> struct isArray<S[LEN]> : TrueType {};  
template<typename S,unsigned int LEN> struct isArray<S(&)[LEN]> : TrueType {};  
  
  
/  
  
  
template<typename S> struct ArraySize : Int2Type<0> {}; //for pass compiler graceful  
template<typename S,unsigned int LEN> struct ArraySize<S[LEN]> : Int2Type<LEN> {};  
template<typename S,unsigned int LEN> struct ArraySize<S(&)[LEN]> : Int2Type<LEN> {};  
  
  
/  
  
  
template<typename T> struct isAtomType  
{  
template<typename C> static UtilExt::Yes_Type test(...); //SFINAE : capture default type  
template<typename C> static UtilExt::No_Type test(bool C::*); //SFINAE : capture customized type  
  
  
enum { value = logic::And_T<logic::Not_T<isArray<T> > , BoolType<sizeof(test<T>(0)) == sizeof(UtilExt::Yes_Type)> >::value };  
};  
  
  
/  
  
  
template<typename T> struct isBuildInType   
{  
private:  
typedef typename GetRawType<T>::type rawT;  
template<typename U> struct tester: IsSameType<rawT,U> {};  
  
  
public:  
enum{value=logic::Or_T<  
logic::Or_T<IsPointer<T>, tester<__int64>, tester<unsigned __int64> >,  
logic::Or_T<tester<bool>, tester<char>, tester<unsigned char>, tester<signed char> >,  
logic::Or_T<tester<wchar_t>, tester<short>, tester<unsigned short>, tester<int>, tester<unsigned int> >,  
logic::Or_T<tester<float>, tester<long>, tester<unsigned long>, tester<double>, tester<long double> > >::value };  
/* below comes from C++ primer chapter 3, other is default  
bool //ANSI/ISO C++   
wchar_t //unicode , compiler dependency  
float //ANSI/ISO C++   
double //ANSI/ISO C++   
__int64 //C99  
unsigned __int64 //C99   
long double //ANSI/ISO C++  
IsPointer //default pointer  
*/  
};  
  
  
//because array doesn't support operator "=" , when it is copied and transfer , it will be degraded and correct transfer has performance consumption  
template<typename T,unsigned int LEN> struct isBuildInType<T[LEN]> : FalseType{};  
  
  
//because array doesn't support operator "=" , and it can be copy and transfer without performance consumption by reference  
template<typename T,unsigned int LEN> struct isBuildInType<T(&)[LEN]> : TrueType{};  
  
  
/  
template<typename T> struct isEnumType  
{  
enum {value = logic::And_T<BoolType<sizeof(enum EnumType)==sizeof(T)>,isAtomType<T>,logic::Not_T<isBuildInType<T> > >::value };  
};  
  
  
/   
template<typename T>  
struct isString  
{  
protected:  
template<typename U> struct StlStringHelper  
: public BoolType<logic::Or_T<IsSameRawType<U,stlStringA> ,IsSameRawType<U,stlStringW> ,IsSameRawType<U,stlString> >::value >{};  
  
  
template<typename T> struct isCharArray : public BoolType<false>{};  
template<typename T,unsigned int iLen> struct isCharArray<T[iLen]>   
: public BoolType<logic::Or_T<IsSameRawType<T,wchar_t>,IsSameRawType<T,char> >::value> {};  
  
  
  
  
public:  
enum {  
value = IsSameRawType<T,char*>::value ||   
IsSameRawType<T,wchar_t*>::value ||  
isCharArray<T>::value ||   
StlStringHelper<T>::value   
};  
};  
  
  
/  
  
  
template<typename T> struct GetRawType : public RemoveVolatile<typename RemoveConst<typename RemoveRef<T>::type>::type> {};  
}   
  
  
#ifdef _MSC_VER  
#pragma warning(default:4244)  
#endif  
#endif // typeTrait_h__  

