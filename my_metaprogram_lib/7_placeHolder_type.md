7) 占位符类型 placeHolder 

发布日期: 2014-02-12 11:09:10
原文链接: https://blog.csdn.net/Tonny0832/article/details/19111765

---

要泛型设计中，某些类型或者值在编译阶段是无法确实的，但是又确实需要把这个位子给占住，在实际运行中，再用确切的类型/值来替换。

就好比一个人雇人排队买房：排队的是占位符，但是真正买房时，还是你自己去的。

下面实现了占位符功能，且最多支持5个占位符。

  


#ifndef placeHolder_h__  
#define placeHolder_h__  
#include <mplconfig.h>  
  
  
#if HAS_BOOST_SUPPORT==0  
  
  
namespace MiniMPL  
{  
template<int N> struct Args;  
  
  
template<> struct Args<-1> {};  
typedef Args<-1> _;  
  
  
template<> struct Args<0> {};  
typedef Args<0> _0;  
  
  
template<> struct Args<1>  
{  
template<typename A1,typename A2=NullType,typename A3=NullType,typename A4=NullType,typename A5=NullType>  
struct apply  
{  
typedef A1 type;  
};  
};  
  
  
typedef Args<1> _1;  
  
  
template<> struct Args<2>  
{  
template<typename A1,typename A2,typename A3=NullType,typename A4=NullType,typename A5=NullType>  
struct apply  
{  
typedef A2 type;  
};  
  
  
};  
  
  
typedef Args<2> _2;   
  
  
template<> struct Args<3>  
{  
template<typename A1,typename A2,typename A3,typename A4=NullType,typename A5=NullType>  
struct apply  
{  
typedef A3 type;  
};  
  
  
};  
  
  
typedef Args<3> _3;   
  
  
template<> struct Args<4>  
{  
template<typename A1,typename A2,typename A3,typename A4,typename A5=NullType>  
struct apply  
{  
typedef A4 type;  
};  
  
  
};  
  
  
typedef Args<4> _4;   
  
  
template<> struct Args<5>  
{  
template<typename A1,typename A2,typename A3,typename A4,typename A5>  
struct apply  
{  
typedef A5 type;  
};  
  
  
};  
  
  
typedef Args<5> _5;   
}  
  
  
#endif //end of HAS_BOOST_SUPPORT  
  
  
#endif // placeHolder_h__  

