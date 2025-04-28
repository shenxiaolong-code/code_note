编译器bug ?

发布日期: 2025-04-10 15:40:36
原文链接: https://blog.csdn.net/Tonny0832/article/details/147120348

---

## 问题描述

两个结构几乎相同的模板实现，一个能正常工作，另一个在 VS2019 和 GCC 中都会报错。

## 最小化测试代码
[code] 
    ```
    // bug_report.cpp
    #include <type_traits>
    #include <string>
    
    template<typename T> struct Type2Type { using type = T; };
    template<typename T1, typename T2> struct IsSameType : std::false_type {};
    template<typename T> struct IsSameType<T, T> : std::true_type {};
    
    // 工作正常的版本
    template<size_t idx, typename TList> struct FindNthTypeInTypeList;
    template<typename T, template <typename ...> class TList, typename ... Types>
    struct FindNthTypeInTypeList<0,TList<T, Types...>> : public Type2Type<T> {};
    template<size_t idx, template <typename ...> class TList, typename T, typename ... Types>
    struct FindNthTypeInTypeList<idx, TList<T, Types ...>> : public FindNthTypeInTypeList<idx-1, TList<Types ...>> {};
    template<size_t idx, typename TList> using FindNthTypeInTypeList_t = typename FindNthTypeInTypeList<idx, TList>::type;
    
    // 不能工作的版本
    template<size_t idx, typename TList> struct GetAllTypesFromIdx;
    template<template <typename ...> class TList, typename ... Types>
    struct GetAllTypesFromIdx<0, TList<Types...>> : public Type2Type<TList<Types...>> {};
    template<size_t idx, typename T, template <typename ...> class TList, typename ... Types>
    struct GetAllTypesFromIdx<idx, TList<T, Types...>> : public GetAllTypesFromIdx<idx-1, TList<Types...>> {};
    template<size_t idx, typename TList> using GetAllTypesFromIdx_t = typename GetAllTypesFromIdx<idx, TList>::type;
    
    // 测试类型列表
    template<typename...> struct TypeList {};
    using tc_typeList = TypeList<int, char, std::string, char*>;
    
    int main()
    {
        // FindNthTypeInTypeList 测试 - 这些都能工作
        static_assert(IsSameType<FindNthTypeInTypeList_t<0, tc_typeList>, int>::value);
        static_assert(IsSameType<FindNthTypeInTypeList_t<1, tc_typeList>, char>::value);
        static_assert(IsSameType<FindNthTypeInTypeList_t<2, tc_typeList>, std::string>::value);
        static_assert(IsSameType<FindNthTypeInTypeList_t<3, tc_typeList>, char*>::value);
        static_assert(!IsSameType<FindNthTypeInTypeList_t<0, tc_typeList>, char>::value);
    
        // GetAllTypesFromIdx 测试 - 这些会报错
        static_assert(IsSameType<GetAllTypesFromIdx_t<0, tc_typeList>, TypeList<int, char, std::string, char*>>::value);
        // static_assert(IsSameType<GetAllTypesFromIdx_t<1, tc_typeList>, TypeList<char, std::string, char*>>::value);
        // static_assert(IsSameType<GetAllTypesFromIdx_t<2, tc_typeList>, TypeList<std::string, char*>>::value);
        // static_assert(IsSameType<GetAllTypesFromIdx_t<3, tc_typeList>, TypeList<char*>>::value);
    }
    ```
    
[/code]

## 错误信息

在 VS2019 和 GCC 中都会报错：  

[code] 
    ```
    1>C:\...\MiniMPL\sources\UnitTest\UT_MiniMPL\src\tc_typeList_cpp11.cpp(135,9): error C2752: 'MiniMPL::GetAllTypesFromIdx<0,UnitTest::tc_typeList>': more than one partial specialization matches the template argument list
    1>C:\...\MiniMPL\sources\MiniMPL\include\MiniMPL\typeList_cpp11.hpp(56,105): message : could be 'MiniMPL::GetAllTypesFromIdx<0,TList<Types...>>'
    1>C:\...\MiniMPL\sources\MiniMPL\include\MiniMPL\typeList_cpp11.hpp(58,105): message : or       'MiniMPL::GetAllTypesFromIdx<idx,TList<T,Types...>>'
    1>C:\...\MiniMPL\sources\UnitTest\UT_MiniMPL\src\tc_typeList_cpp11.cpp(135): message : see reference to alias template instantiation 'MiniMPL::GetAllTypesFromIdx_t<0,UnitTest::tc_typeList>' being compiled
    1>C:\...\MiniMPL\sources\MiniMPL\include\MiniMPL\typeList_cpp11.hpp(59,176): error C2794: 'type': is not a member of any direct or indirect base class of 'MiniMPL::GetAllTypesFromIdx<0,UnitTest::tc_typeList>'
    1>C:\...\MiniMPL\sources\UnitTest\UT_MiniMPL\src\tc_typeList_cpp11.cpp(135,9): error C2938: 'MiniMPL::GetAllTypesFromIdx_t' : Failed to specialize alias template
    ```
    
[/code]

## 问题分析

1\. 两个模板实现 `FindNthTypeInTypeList` 和 `GetAllTypesFromIdx` 的结构几乎完全相同

2\. 它们都有：

\- 一个 `idx=0` 的特化

\- 一个通用的递归特化

\- 相同的类型列表处理方式

3\. `FindNthTypeInTypeList` 能正常工作，所有测试都通过

4\. `GetAllTypesFromIdx` 在 VS2019 和 GCC 中都会报错

## 预期行为

\- 两个模板都应该能正常工作

\- `FindNthTypeInTypeList` 应该返回指定索引的类型

\- `GetAllTypesFromIdx` 应该返回从指定索引开始的所有类型

## 实际行为

\- `FindNthTypeInTypeList` 按预期工作

\- `GetAllTypesFromIdx` 报错，认为两个特化是同等特化的

## 环境信息

\- 编译器：VS2019 和 GCC

\- 标准：C++11 或更高

\- 操作系统：Windows/Linux

## 可能的解决方案

1\. 使用 SFINAE 来限制第二个特化只在 `idx>0` 时匹配

2\. 重新设计模板特化，使其特化规则更明确

3\. 等待编译器更新修复这个 bug
