g++ 编译器模板解析近似缺陷的限制（C++标准议论）

发布日期: 2024-08-29 11:23:39
原文链接: https://blog.csdn.net/Tonny0832/article/details/141673171

---

最近想把我的C++模板库移植到linux，用g++编译，发现了大量编译错误，逐个仔细审查外，我更多地倾向于认为是linux g++编译器对复杂模板元编程的解析能力很弱，而不是代码问题。

#### 1\. vs编译器支持的模板库

这套模板库(<https://github.com/shenxiaolong-code/MiniMPL>)，从vs2008的C++03， 直到验证过的支持C++20的vs2019，(估计以后版本的vs也可以，没测试过)，每个vs版本都支持得很完美，但在g++上，各种应该支持的模板解析都编译报错。

#### 2\. g++对模板的语法要求

并且g++对模板的语法要求非常生硬，比如：

  - 对于子类型，一律要求使用typename来指示，
  - 对于模板成员函数，要求使用来指示，比如obj.template run<...>(...)，
  - 对于内嵌的模板类，要求使用::template来指示，比如T::template apply<...>;



而vs就宽松得多，在不出现多重定义的情况下，会自动探测到应该使用什么，并不需要用户显式指定，除非碰到有岐义的地方。

#### 3\. 模板特化的问题

最近碰到一个模板特化的问题，我的模板特化的终结类值是依赖于外层模板类的输入参数的，这就导致我的模板特化终结器必须定义在类内。  
MS编译器没有任何问题，各种MS编译器就完美支持(MSBuild , visual studio), 而 g++ 就报告说不支持类内特化。
[code] 
    ```
    error: explicit specialization in non-namespace scope ‘xxxxxxx’
    ```
    
[/code]

#### 4\. perplexity AI的结论

和 perplexity AI 反复讨论后得出的结论就是： g++ 编译器没法满足这种真实的需求。  
**perplexity 的总结：**

> 1\. 现状总结  
>  你的代码逻辑是合理的，且符合C++模板编程的常见模式。  
>  由于GCC不支持类内特化，而你又需要这种特化的功能，这确实是一个限制。  
>  可能的解决方案  
>  虽然GCC的限制可能看起来像是一个bug，但实际上这是编译器对C++标准的实现方式。我们可以尝试一些替代的设计模式，尽管这些可能不完全符合你的初衷，但可以实现类似的功能。
> 
> 2 根据你提供的代码和错误信息，以及搜索结果中的讨论，g++确实不支持在类内部进行显式特化。这是一个g++的限制，而不是你代码中的问题。  
>  所以你提供的两种方式，都属于在类内部进行特化，g++都会报错。这是由于g++的实现限制造成的。

即: perplexity 确认我的代码是合理的，并且符合C++模板编程. g++编译报错的原因是因为g++的限制。

####   
5\. g++编译器无法支持的模板代码

源文件链接： [enumTrait_detail.hpp#L23](https://github.com/shenxiaolong-code/MiniMPL/blob/master/sources/MiniMPL/include/MiniMPL/innerDetail/enumTrait_detail.hpp#L23 "enumTrait_detail.hpp#L23")
[code] 
    ```
    template<typename T>                        struct EnumRangeSizeImpl;
    template<typename F,F F_min,F F_max>        struct EnumRangeSizeImpl< CEnumRange<F,F_min,F_max> >
    {
        typedef CEnumRange<F, F_min, F_max>     TEnumRange;
    
        struct detail
        {
            template<typename TEnumValue>   struct CountHelper: public Int2Type<1+CountHelper<typename TEnumValue::template Next<true>::type >::value >{};
            template<>                      struct CountHelper<typename TEnumRange::Max_T >   : public Int2Type<1> {};
        };
    
        enum {value=detail::CountHelper<typename TEnumRange::Min_T >::value};
    };
    ```
    
[/code]

简单解释:  
这是一个计算枚举类型枚举值数量的模板函数(计算某枚举类型有几个枚举值)，防止枚举时越界。

由于 CountHelper 的终结器依赖于EnumRangeSizeImpl类的输入模板参数 TEnumRange，所以其只能在类EnumRangeSizeImpl的内部进行特化。 然后编译 CountHelper<typename TEnumRange::Max_T > 时，g++就编译出报错了：
[code] 
    ```
    error: explicit specialization in non-namespace scope ‘struct MiniMPL::EnumRangeSizeImpl<MiniMPL::CEnumRange<F, F_min, F_max> >’
    ```
    
[/code]

#### 6\. 个人浅见(C++标准议论)

这是模板编程的真实需求，并不是臆造的。

如果一个特化终结器依赖于一个类型，那么它必须在一个类内，只有类进行实例化编译时，这个类的输入参数类型才是确定的，从而这个终结器的值才是确定的，任何类外的终结器都是没法实现的(无法提供一个明确的值)

把类的特化限制在命名空间内，而强制不能在类内特化，这是C++03的标准规定，可能是初期是由于编译器厂商实现上的限制，但是我发现随后的C++标准到现在一直都没有更新

[C++03标准 14.7.3节](https://timsong-cpp.github.io/cppwp/n3337/temp.spec#3 "C++03标准 14.7.3节")提到:

> An explicit specialization shall be declared in the namespace of which the template is a member, or, for member templates, in the namespace of which the enclosing class or enclosing class template is a member. 

但是MS编译器并没有遵循这个要求，而是支持类内特化  
[Stack Overflow上的讨论](https://stackoverflow.com/questions/3052579/explicit-specialization-in-non-namespace-scope "Stack Overflow上的讨论"):

> VC++ is non-compliant in this case - explicit specializations have to be at namespace scope. C++03, §14.7.3/2:   
>  An explicit specialization shall be declared in the namespace of which the template is a member, or, for member templates, in the namespace of which the enclosing class or enclosing class template is a member. 

希望C++标准能取消这个限制，根据商业编译器(如MS编译器)的实践经验来更新标准，同时g++编译器也应该增强它的模板解析能力。
