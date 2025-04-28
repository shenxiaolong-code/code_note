把一个非延缓评估的元函数转换为一个延缓评估的元函数(Lazy evaluation)

发布日期: 2013-12-04 15:22:58
原文链接: https://blog.csdn.net/Tonny0832/article/details/17119017

---

在C++的模板元编程中，如果使用了T::type , T::value , T::apply之类的语法，其将导致编译器立马对该类型表达式进行求值，有些时候，这些求值工作是不必要的，将增加编译器的编译时间，我们可以将之延迟到最后一刻确实需要进行类型计算的时候再去求值，此称之为Lazy evaluation。

下面通过Lazy_If_T 为例演示了如何把一个非延缓评估的元函数转换为延迟评估的元函数----此模板源代码的应用示例可以到我的资源中的"[博客中的mini泛型库源代码](http://download.csdn.net/detail/tonny0832/6373127)"中去下载。

  


template<bool,typename True_T,typename False_T> struct If_T //default is true

{   
typedef True_T type;   
};   
template<typename True_T,typename False_T> struct If_T<false,True_T,False_T>   
{   
typedef False_T type;   
};   
  
  
//for lazy evaluation   
template<typename B,typename True_T,typename False_T> struct Lazy_If_T : public If_T<B::value,True_T,False_T>::type   


{};

//---------使用Lazy_If_T ，编译器将不会对Lazy_If_T 的type立即进行求值计算，而仅仅作一个类型标记，待到确实需要时，再进行计算，节省了编译时间。
