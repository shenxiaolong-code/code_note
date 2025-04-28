# Ⅰ 文章目录简介
本目录收录了作者在 C++ 泛型编程、基础语法、编译器问题、日常记录及常用特性等方面的原创笔记，涵盖了我本人学习到的各种技术细节和实用经验。
- 其文章原始发表在中国大陆境内的 [CSDN](https://blog.csdn.net/Tonny0832?type=blog) 论坛，现在把它移植到 github上。
- 关于我的工作经历，可以参见 [linkIn](https://www.linkedin.com/in/shenxiaolong/)。
- 另外我会慢慢把我日常积累记录下来的100多万字的C++编程文档(本地存储的个人记录的笔记，未公开)逐渐整理公开在这儿(TODO)。
- 由于使用爬虫脚本进行的自动转换，转换内容难免会一些问题(内容丢失或者不一致)，欢迎指出。（我会逐渐阅览并修正）。

# Ⅱ 各子目录及代表性内容简介如下：
## 1). my_metaprogram_lib ([github 源代码](https://github.com/shenxiaolong-code/MiniMPL))
主要介绍作者自研的 C++ 泛型元编程库 MiniMPL，包括类型特征、类型转换、函数对象、成员指针、统一容器接口、类型安全封装等，也是本人最擅长的C++编程技能之一。  
每篇文章聚焦一个元编程组件，配有详细用法和设计思路，适合深入理解 C++ 模板与泛型编程。

- [00_easy_use_stl_algorithm.md](01_my_metaprogram_lib/00_easy_use_stl_algorithm.md)：MiniMPL 库总览与设计初衷
- [01_enhance_typeTraits.md](01_my_metaprogram_lib/01_enhance_typeTraits.md)：类型特征 typeTraits 的实现与应用
- [02_typeConvert_function.md](01_my_metaprogram_lib/02_typeConvert_function.md)：类型转换 typeConvert 的通用方案
- [03_pack_unary_binary_function.md](01_my_metaprogram_lib/03_pack_unary_binary_function.md)：一元/二元函数抽象
- [04_unaryFunctionConverter.md](01_my_metaprogram_lib/04_unaryFunctionConverter.md)：多元函数转一元函数
- [05_fetch_functionTraits.md](01_my_metaprogram_lib/05_fetch_functionTraits.md)：函数特征 functionTraits
- [06_create_function_object.md](01_my_metaprogram_lib/06_create_function_object.md)：一元/二元函数对象创建
- [07_placeHolder_type.md](01_my_metaprogram_lib/07_placeHolder_type.md)：占位符类型 placeHolder
- [08_data_member_ptr.md](01_my_metaprogram_lib/08_data_member_ptr.md)：多级结构数据成员指针
- [09_functionobject_MiniMPL.md](01_my_metaprogram_lib/09_functionobject_MiniMPL.md)：常用函数对象的泛型实现
- [09_mathOperator_MiniMPL.md](01_my_metaprogram_lib/09_mathOperator_MiniMPL.md)：泛型数学运算符函数
- [10_templated_ClassRegister.md](01_my_metaprogram_lib/10_templated_ClassRegister.md)：泛型工厂方法创建实例
- [11_unified_container_interface.md](01_my_metaprogram_lib/11_unified_container_interface.md)：统一容器接口 DataSet
- [12_CAnyObject.md](01_my_metaprogram_lib/12_CAnyObject.md)：类型安全的任意类封装
- [13_delivery_parameter.md](01_my_metaprogram_lib/13_delivery_parameter.md)：参数适配 ParamterWrapper
- [14_algorithm_based_on_object_size.md](01_my_metaprogram_lib/14_algorithm_based_on_object_size.md)：基于对象大小的算法
- [15_callback_template_function.md](01_my_metaprogram_lib/15_callback_template_function.md)：回调模板函数/成员函数

## 2). cpp_basic ([link](https://github.com/shenxiaolong-code/code_note/tree/main/02_cpp_basic))
C++ 基础语法知识点的总结与实战分析。

- [xxx.md](02_cpp_basic/xxx.md)

## 3). discuss ([link](https://github.com/shenxiaolong-code/code_note/tree/main/03_discuss))
编译器相关问题、标准实现差异、实际开发中遇到的 bug 及其分析。

- [xxx.md](03_discuss/xxx.md)

## 4). general_feature ([link](https://github.com/shenxiaolong-code/code_note/tree/main/04_general_feature))
日常编程中，可能会用到的一些常用功能。如果你需要这些功能，可以直接从这儿复制，避免再自己手工造轮子。

- [xxx.md](04_general_feature/xxx.md)

## 5). daily_record ([link](https://github.com/shenxiaolong-code/code_note/tree/main/05_daily_record))
日常流水帐，作不得数，怡笑方家。

- [xxx.md](05_daily_record/xxx.md)

# Ⅲ 移植记录
## port_record.md
本文记录移植时所有文章在原始[CSDN](https://blog.csdn.net/Tonny0832?type=blog) 上对应的链接，便于快速查找和导航。

---

如需详细内容，请查阅各子目录下的具体文章。 