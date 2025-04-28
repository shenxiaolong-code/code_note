编译期assert函数

发布日期: 2013-10-10 10:57:47
原文链接: https://blog.csdn.net/Tonny0832/article/details/12560145

---

编译期assert函数的目的在于当条件不满足时，阻止编译，从而防止错误的逻辑通过编辑。

而运行期assert的目的在于运行时发现条件不满足时，产生一个Debug事件(DebugBreak)，从而让调试器停下来方便用户检查原因。

  


需求描述

有些比较关系，我们期望在编译期就能确保正确，

需求情形:比如A,B，我们要求编译期就能保证A>B，否则编译不能通过。

很明显，如果使用普通的方式，比如

if(A>B)

#error A MUST be bigger than B

则是行不通的，因为if只能在运行期间求值，而编译期不行。

解决办法：

编译期assert的原理就是利用编译期条件判断，最好执行一段不合法或者未定义的代码。

其实实现编译期assert的难点不在于构造不合法的代码或者未定义的代码，而在于实现编译期条件判断，即编译期对表达式求值。

Ø **不合法的代码方法**

当条件达到时，代码流转向不合法的代码。

int static_assert_impl[ sizeof(TypeA)==sizeof(TypeB)?1:-1];

当不满足sizeof(TypeA)==sizeof(TypeB)条件时，int static_assert_impl [-1];是不合法的，因为长度不能为负。从而达到了阻止编译的要求。

Ø **未定义的代码方法**

当条件达到时，代码流转向未定义的模板代码:利用模板来实现编译期assert函数。使用模板类来解决：模板是在编译期初始化的，利用这个特性，可以达到编译期运行某些实例。

当一个模板类实例化时，没有相关的实例化定义，则编译器就会报错。

那么只定义允许的实例化定义，不定义不允许的实例化定义。

具体到实践上即：只定义求值结果为true的实例化定义，不定义求值结果为false的实例化定义。

注：为了提高效率，我们使用struct来代替class，因为编译器不为struct产生缺省的构造、赋值等函数，而这些我们都不需要。

不定义条件为false的实现化定义：static_assert_impl<true>

再定义一个宏来生成模板参数为true的static_assert_impl：

#define static_assert(X) static_assert_impl<(bool)(X)> tem_obj;

这样，当在使用static_assert(A>B)时，如果A<B，则因为static_assert_impl<false>没有定义，而产生编译错误，从而达到了我们的目的。

衍生的两个问题

a) 重复定义问题

这些宏定义有个缺陷：在一个作用域范围内，只能使用一个static_assert(X)；这是不能容忍的。如果使用{}来对作用域降级的话，则static_assert(X)只能在函数内使用，不能满足我们的全局作用域内assert.

所以，不能用生成临时对象的方法来实现。

即如果满足了相等的条件，则定义了intstatic_assert_impl [1];这是合法的。但是如果多个地方都满足了条件，那么程序中就会定义多个intstatic_assert[1];很明显，这是重复定义的错误，同时还会导致消耗不必要的内存。(尽管只有一个字节)。

解决办法：C++为我们提供了一个关键字typedef，似乎这个关键字天生就是为我们测试/校验语法而生的。typedef不产生对象实例，只定义类型，其可以多次重复使用，如果多次定义则是后来覆盖前面的定义。其附带一个好处时，其仅仅产生一个语法上的定义，而不消耗任何实际的空间，其也就不产生任何实例，所以就不存在重复定义的说法。OK，完美的定义：

#define static_assert(X) typedefstatic_assert_impl[(X)?1:-1] static_assert_def;

#define static_assert(X) typedefstatic_assert_impl<(bool)(X)> static_assert_def;

这个定义有一个问题就是：typdef并不产生实例，所以其并不对X求值。如果需要对X求值，必须static_assert_defobj，即定义static_assert_def对象，这又回到老路上去了。

b) 编译期求值问题

即必须找到编译期求值的操作。在C/C++中，能够编译期求值的办法：常量定义，sizeof运算符，模板实例化。

sizeof还有一个好处是，其不仅仅可以对对象求值，还可以对类型求值－－这就是我们需要的:sizeof(static_assert_impl<(bool)(X)>)，而且sizeof还必须初始化static_assert_impl<(bool)(X)>。OK，达到了初始化static_assert_impl<(bool)(X)>的目的了。如果在全局作用域区，单独地写上一个整数：sizeof(static_assert_impl<(bool)(X)>)，同样编译错误。即，如果在cpp文件的前面写上一个4，明显不合C,C++语法。我们必须要用这个整数来做点什么，才合语法要求。

能用整数做什么而又不产生临时对象呢－－实例化C++模板类/函数不会产生临时对象。OK。定义一个参数为int的类模板即可。

template<int>struct static_assert_tester{}; 

则宏转换为：

#define static_assert(X) typedefstatic_assert_tester<sizeof(static_assert_impl<(bool)(X)>)>static_assert_def;

完整的代码定义：

应用测试：

static_assert(3>1); //OK,编译通过。

static_assert(3>4); //fail，编译错误：

编译错误提示：

error C2027:use of undefined type 'static_assert_impl<x>'

error C2371: assert_def': redefinition; different basic types 

在VC许多源文件中，都有现成的编译期assert可用：_STATIC_ASSERT。

Ø **static_assert之衍生应用：测试#if条件编译**

除了上面static_assert的原本作用外，利于static_assert我们还可以测试某些额外的功能。比如测试一个表达式是否在当前编译器下为编译期常量。

一个例子：#ifa==b

上述#if后，通常要求a==b是个常量表达式，如果a,b都是模板参数，从语法上讲，它们都应该是常量性质的，示例代码：

上面的，doSensorType是模板的特化参数，此时已经是编译期常量的，eDOSensorTypeid_DOSensorType_Optical是一个常量宏，#ifdoSensorType==eDOSensorTypeid_DOSensorType_Optical从语法上分析应该是一个编译期常量表达式的，应该符合#if的逻辑意图的。

但是，在MS编译器下，#if doSensorType==eDOSensorTypeid_DOSensorType_Optical的逻辑结果就是#if 1，即条件编译里的语句始终执行。这是错误的，但是MS编译器能够通过编译。

这应该是编译器的一个bug，但是我们应该避免，适当的时候，应该自己做些测试来避免编译器的bug。测试就是利用static_assert。

当编译器没有bug时，上述代码应该总是编译成功的。

但是事实上，当doSensorType不等于eDOSensorTypeid_DOSensorType_Optical时，其将触发编译错误，即宏#ifdoSensorType==eDOSensorTypeid_DOSensorType_Optical条件编译竟然总是为1，从而可以帮助我们发现这个编译器bug，而改用其它的解决方案实现意图。

  

