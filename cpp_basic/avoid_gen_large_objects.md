避免创建不必要的大对象(把大对象转换为小对象)

发布日期: 2013-10-24 16:58:51
原文链接: https://blog.csdn.net/Tonny0832/article/details/13001323

---

在模板函数中，很多情况下，我们仅仅需要一个对象的类型，而并不需要这个对象的实体，例如：

template<typenameT> void HandObject()

{

cout<< typeid(T).name() << endl;

} 

在调用时，用户必须明确地指定模板类型：

HandObject<bigObject>();

很多时候，用户并不喜欢这种调用方式，或许用户更喜欢下面的方式：

template<typenameT> void HandObject(T)

{ 

cout<< typeid(T).name() << endl;

} 

但是这样用户必须传入一个T对象，如果这个T对象很大，或者构造很复杂的话，则是得不偿失的：为了用户易用，而不得不构建一个很大或者很复杂的临时对象，并且这个临时对象完全是不必要创建的。如：

struct bigObject

{

int m_data[10240];

};

为了解决这个问题，我们可以使用一个Type2Type来把一个大对象转换为一个小得可以忽略不计的小对象。

template<typenameT> struct Type2Type

{

typedefT OriType;

};

template<typenameT> void HandObject(Type2Type<T>)

{ 

cout << typeid(T).name()<< endl;

}

这个Type2Type对象大小只为一个标记符尺寸：1字节。从而把一个大对象转化为一个小对象：把创建40K字节的bigObject对象改为创建1字节的Type2Type对象

则调用形式转换为：

HandObject(Type2Type<bigObject>());

HandObject还有另外一种形式：

template<typenameT> void HandObject(T,typename T::OriType* p= NULL)

{ 

cout<< typeid(T).name() << endl;

}

这两个HandObject形式可以并存，只是当模板匹配时，HandObject(Type2Type<T>)的匹配更优，从而HandObject(Type2Type<T>)被优先匹配，而HandObject(T,typename T::OriType* p= NULL)不被使用。
