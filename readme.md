# Ⅰ 文章目录简介
本目录收录了作者在 C++ 泛型编程、基础语法、编译器问题、日常记录及常用特性等方面的原创笔记，涵盖了我本人学习到的各种技术细节和实用经验。
- 其文章原始发表在中国大陆境内的 [CSDN](https://blog.csdn.net/Tonny0832?type=blog) 论坛，现在把它移植到 github上。
- 关于我的工作经历，可以参见 [linkIn](https://www.linkedin.com/in/shenxiaolong/)。
- 另外我会慢慢把我日常积累记录下来的100多万字的C++编程文档(本地存储的个人记录的笔记，未公开)逐渐整理公开在这儿(TODO)。
- 由于使用爬虫脚本进行的自动转换，转换内容难免会一些问题(内容丢失或者不一致)，欢迎指出。（我会逐渐阅览并修正）。

# Ⅱ 各子目录及代表性内容简介如下：
## 1). my_metaprogram_lib
主要介绍作者自研的 C++ 泛型元编程库 MiniMPL，包括类型特征、类型转换、函数对象、成员指针、统一容器接口、类型安全封装等。每篇文章聚焦一个元编程组件，配有详细用法和设计思路，适合深入理解 C++ 模板与泛型编程。

- [0_my_MiniMPL_lib.md](my_metaprogram_lib/0_my_MiniMPL_lib.md)：MiniMPL 库总览与设计初衷
- [1_enhance_typeTraits.md](my_metaprogram_lib/1_enhance_typeTraits.md)：类型特征 typeTraits 的实现与应用
- [2_typeConvert_function.md](my_metaprogram_lib/2_typeConvert_function.md)：类型转换 typeConvert 的通用方案
- [3_pack_unary_binary_function.md](my_metaprogram_lib/3_pack_unary_binary_function.md)：一元/二元函数抽象
- [4_unaryFunctionConverter.md](my_metaprogram_lib/4_unaryFunctionConverter.md)：多元函数转一元函数
- [5_fetch_functionTraits.md](my_metaprogram_lib/5_fetch_functionTraits.md)：函数特征 functionTraits
- [6_create_function_object.md](my_metaprogram_lib/6_create_function_object.md)：一元/二元函数对象创建
- [7_placeHolder_type.md](my_metaprogram_lib/7_placeHolder_type.md)：占位符类型 placeHolder
- [8_data_member_ptr.md](my_metaprogram_lib/8_data_member_ptr.md)：多级结构数据成员指针
- [9_functionobject_MiniMPL.md](my_metaprogram_lib/9_functionobject_MiniMPL.md)：常用函数对象的泛型实现
- [9_mathOperator_MiniMPL.md](my_metaprogram_lib/9_mathOperator_MiniMPL.md)：泛型数学运算符函数
- [10_templated_ClassRegister.md](my_metaprogram_lib/10_templated_ClassRegister.md)：泛型工厂方法创建实例
- [11_unified_container_interface.md](my_metaprogram_lib/11_unified_container_interface.md)：统一容器接口 DataSet
- [12_CAnyObject.md](my_metaprogram_lib/12_CAnyObject.md)：类型安全的任意类封装
- [13_delivery_parameter.md](my_metaprogram_lib/13_delivery_parameter.md)：参数适配 ParamterWrapper
- [14_algorithm_based_on_object_size.md](my_metaprogram_lib/14_algorithm_based_on_object_size.md)：基于对象大小的算法
- [15_callback_template_function.md](my_metaprogram_lib/15_callback_template_function.md)：回调模板函数/成员函数

## 2). cpp_basic
C++ 基础语法知识点的总结与实战分析。

- [struct_alignment_rule.md](cpp_basic/struct_alignment_rule.md)：结构体对齐规则详解
- [cpp_typeid_RTTI.md](cpp_basic/cpp_typeid_RTTI.md)：typeid 及 RTTI 机制
- [internal_external_linkage.md](cpp_basic/internal_external_linkage.md)：内部/外部链接详解
- [avoid_gen_large_objects.md](cpp_basic/avoid_gen_large_objects.md)：避免生成大对象的技巧
- [namespace_internal.md](cpp_basic/namespace_internal.md)：命名空间与 internal 机制
- [Lazy_unary_template_function.md](cpp_basic/Lazy_unary_template_function.md)：延缓求值的单参模板函数
- [built_small_quick_app.md](cpp_basic/built_small_quick_app.md)：快速构建小型应用
- [gen_and_test_NaN_INFINITE.md](cpp_basic/gen_and_test_NaN_INFINITE.md)：NaN/INFINITE 数值的生成与测试

## 3). discuss
编译器相关问题、标准实现差异、实际开发中遇到的 bug 及其分析。

- [gcc_compiler_limitation.md](discuss/gcc_compiler_limitation.md)：g++ 模板解析限制与标准讨论
- [compiler_template_bug.md](discuss/compiler_template_bug.md)：模板相关的编译器 bug
- [vs2008_compiler_bug.md](discuss/vs2008_compiler_bug.md)：VS2008 编译器的一个 bug 分析

## 4). general_feature
日常编程中，可能会用到的一些常用功能。如果你需要这些功能，可以直接从这儿复制，避免再自己手工造轮子。

- [reverse_single_linked_list.md](general_feature/reverse_single_linked_list.md)：单向链表反转算法
- [dump_callstack_by_sym_APIs.md](general_feature/dump_callstack_by_sym_APIs.md)：Windows 下符号 API 导出调用栈
- [compile_time_assert_function.md](general_feature/compile_time_assert_function.md)：编译期断言函数
- [windows_use_timezone.md](general_feature/windows_use_timezone.md)：Windows 时区遍历代码
- [left_right_shift_string.md](general_feature/left_right_shift_string.md)：字符串左移/右移算法

## 5). daily_record
日常流水帐，作不得数，怡笑方家。

- [coder_question.md](daily_record/coder_question.md)：C/C++/Java 程序员的追求
- [study_java.md](daily_record/study_java.md)：Java 学习笔记

# Ⅲ 移植记录
## port_record.md
本文记录移植时所有文章在原始[CSDN](https://blog.csdn.net/Tonny0832?type=blog) 上对应的链接，便于快速查找和导航。

---

如需详细内容，请查阅各子目录下的具体文章。 