这是vs2008编译器的一个bug么？

发布日期: 2014-01-16 20:36:50
原文链接: https://blog.csdn.net/Tonny0832/article/details/18367837

---

在vs2008中测试如下代码：

struct tagS{};   
  
  
bool VSBug(tagS const& l, const tagS r)   
{   
return true;   
}   
  
  
template<typename T>   
void showType(T f)   
{   
cout << "\n" << typeid(T).name() << endl;   
}   
  
  
void main()   
{   


showType(VSBug);

｝

在showType设置一个断点，在调试窗口中观察变量f的类型及数据：

可以看到 T 的类型与其数据的类型不一致，

f 0x00c71fd0 VSBug(const tagS &, const tagS) bool (const tagS &, tagS)*  


即函数f的类型为 bool (const tagS &, tagS)*

而函数f的数据类型为VSBug(const tagS &, const tagS)

  


这个bug造成我的模板元代码运算时编译失败。

不知道这儿有MS编译器的专家没有？ 能否确认这是一个VS2008编译器的bug ?
