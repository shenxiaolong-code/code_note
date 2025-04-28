内部链接(internal linkage)和外部链接(external linkage)

发布日期: 2013-10-10 10:23:12
原文链接: https://blog.csdn.net/Tonny0832/article/details/12558493

---

### 内部链接(internallinkage)/外部链接 (externallinkage)是和编译单元(translation unit)相关的一个术语，其主要影响函数或者对象的作用域及存储方式—是全局只存储一个，还是全局有许多量的副本。

具有externallinkage的变量可以被其它源文件使用，整个程序内有效，并且全局只有一个。

具有internallinkage的变量只能被本translation unit所引用，如果有多个translation unit，则会有多个副本—每个cpp文件中都会有一个。

#### 1\. 编译单元(translation unit)

一个.cpp文件或者.c文件才称为一个translation unit，头文件(.h)不能称为一个translation unit，因为其最终会被加(plus)入到include它们的.cpp/.c文件中去，编译器编译的是.cpp/.c文件，而不是.h文件。

#### 2\. 缺省的链接方式

在缺省情况下，常量(const)具有static属性(internal linkage)，而非常量(non-const)具有extern属性(external linkage)。

常量(const)因为具有static的internal linkage属性，其会导致每个使用它的cpp文件中都会包含一个const副本，这个副本是local的，可能会导致程序中有大量的常量副本。

变量具有extern的external linkage属性，如果在头文件中定义变量并且该头文件被多个cpp文件引用，将导致重复定义的编译错误；对于变量正确的做法应该是在cpp文件中定义，然后在使用它的cpp文件中使用extern来声明它，即不要将变量直接暴露在.h文件中，如果一定要在头文件中暴露变量，那么应使用extern来修饰它。

**非extern实例声明即是定义**

对于实例，除了类/结构的静态成员外，其声明处就是定义处，即声明等同于定义，其不存在函数有函数声明和函数定义两个部分。测试证明：

在一个类的.h文件中 ： int g_testDef;

在该类相应的.cpp文件中： int g_testDef=0;

编译.cpp文件，则报告下列错误：

error C2086: 'int g_testDef' : redefinition

即.h中已经定义了一个叫做g_testDef的常量，在.cpp文件中又定义了它，即cpp文件中对g_testDef的使用不是初始化它，而是重新定义一个叫做g_testDef的变量并立刻初始化之为0。在.h文件中是只定义了g_testDef而没有显式地初始化它—它会被编译器初始化为随机值或者特定的值(该特定值往往在debug版本中被认定为未初始化的内存)。编译器并不强制要求用户在定义处就初始化变量，因为该变量可以由用户在其它处重新赋值。

如果使用const方式：

在一个类的.h文件中 ： int const  g_testDef;

在该类相应的.cpp文件中： int const  g_testDef=0;

编译.cpp文件，则报告下列错误：

error C2734: 'g_testDef' : const object must be initializedif not extern

error C2086: 'const int g_testDef' : redefinition

即首先一个错误：非extern常量必须定义处初始化(在.h文件中)。

其次的重定义错误和非const一样。

**extern实例的定义与非定义**

对于extern实例，如果没有extern修饰，则声明处就是定义处；而对于用extern修饰的extern实例，可以有多处;但是赋值处只能有一处，并且只有赋值处才被视为定义处

int g_testDef; //definition，①

int g_testDef=2; //definition, ②

extern int g_testDef=2; //definition，③

extern int g_testDef;  //non-definition，④

在上面①②③④混用的情况下，④可以在一个文件或者多个文件中多次出现，①②③等只允许其中一种形式出现一次并且必须出现一次，如果一次都没有出现，则链接时会报告"errorLNK2001: unresolved external symbol"，如果出现超过一次，则链接时会报告"errorLNK2005: "int g_testDef" (?g_testDef@@3HA) already defined ininit_1.obj"。

即对于extern量，定义必须存在并且只允许定义一次，而非定义则可以多次。

对于const量，由于没有第①种形式(常量定义处必须初始化)，其它三种形式的使用和non-const一样。

例如：

//1.cpp : 注意是.cpp文件

const int g_internalLinkage=2;

//2.cpp : 注意是.cpp文件

externconst int g_internalLinkage;

voidtestInternalLinkage()

{

printf("%d",g_internalLinkage);

}

上面代码可以顺利通过编译，但是在link时则报告下列错误：

error C2065: 'g_internalLinkage' : undeclared identifier

即在1.cpp中，g_internalLinkage因为是const的，缺省不是externallinkage的，其不会被暴露给全局。

在2.cpp中，虽然g_internalLinkage是external linkage的，但是它自己没有被初始化，所以被认定为引用全局extern的标签，在链接时，linker向全局寻找g_internalLinkage，但没有发现extern定义的g_internalLinkage，所以链接失败。

将定义的g_internalLinkage修饰为extern即可。

//1.cpp : 注意是.cpp文件

externconst intg_internalLinkage=2;

//2.cpp : 注意是.cpp文件

externconst int g_internalLinkage;

voidtestInternalLinkage()

{

printf("%d",g_internalLinkage);

}

编译链接OK。

**在头文件中定义变量将导致该头文件只能被一个cpp文件include**

如果在头文件中定义了一个变量(无extern修饰词)，然后在多个cpp文件引用这个.h文件，将导致每个include它的cpp文件中都有该变量(include就相当于把头文件中所有内容完全复制到cpp中一样，cpp将拥有.h中的所有内容)，编译后生成一个编译单元(translation unit)，由于变量缺省是external linkage的，则链接时该变量会扩散到全局以供其它编译单元也可以使用/修改它。这样问题就来了，如果多个编译单元都include了同一个包含有变量的头文件，那么这多个编译单元里都有这个全局变量。如果不链接，那么没有问题。一旦链接，则不同编译单元里的该全局变量就会导致重定义冲突。事实上是变量重名冲突，地址并不冲突的，因为各个全局变量都是都是在各自的translationunit里的。

在一个头文件(head.h)中定义一个变量：

int g_count=0;

然后有多个.cpp文件include这个头文件(init_1.cpp，init_2.cpp，init_3.cpp)，所以的源文件都能顺利通过编译，但当链接时，则报告下列错误：

error LNK2005: "int g_count"(?g_count@@3HA) already defined in init_1.obj

即其它cpp文件中也有g_count这个定义。奇怪？难道不是三个cpp文件共享head.h中定义的变量g_count么？事实上似乎是每个包含head.h的cpp文件中都有一个g_count变量的定义。事实上的确如此，下面可以进一步通过常量验证。

**在头文件中定义常量将导致该常量在程序中有大量副本**

在头文件中声明一个const变量g_testH_CPP，并使用初始化函数getH_CPP来初始化它。

在第一个.cpp文件(init_1.cpp)中使用该const量：

在第二个.cpp文件(init_2.cpp)中使用该const量：

运行测试程序：

输出：

从输出可以看到，在两个.cpp文件中引入同一个.h文件中定义的常量，该常量的初始化函数initH_CPP()函数被调用了4次，并且在这两个cpp中使用的常量的地址不相同，这证明了这两个名字相同的常量，实际并不是同一个。

**初始化处才是extern量的定义处**

无论是常量还是变量，如果是extern的，只有初始化处才被认为是定义处，其它extern被认为是引用的标签，标识链接时需要linker全局寻找该量的定义。

例子：

externconst intg_internalLinkage=2; //定义一个extern的常量，只允许一处

externconstintg_internalLinkage; //声明引用一个extern常量，可以允许多处

如果有多个同名的extern量在定义时都初始化了，编译都是成功，但链接时会失败，报告重复定义错误：

error LNK2005: "int g_testDef" (?g_testDef@@3HA)already defined in init_2.obj

fatal error LNK1169: one or more multiply defined symbolsfound

即linker发现了全局scope内有多个同名量的定义(初始化赋值)，这是必然的突然。

#### 3\. 明确地指定链接方式

除了缺省情况外，设计者还可以明确地指定一个量的链接方式，这个指定对变量和变量都有效。

Ø 用extern声明一个量，则其具有外部链接作用域，该量既可以是常量也可以是变量。

Ø 用static声明/定义一个变量，则其具有内部链接作用域，该量应该是变量，因为常量使用了const，而const缺省就是static的，可以省略static修饰词。(也可以是常量，但是static 和const修饰词同时使用的时候，static是多余的)

C++提倡static只使用在类的静态成员上，对于变量和函数，如果需要其具有internallinkage，推荐使用anonymous namespaces来实现。

附：作用域(scope)和生存期(lifetime)不是同一概念。作用域指某tanslation unit中定义的变量是否对其它tanslation unit可见(即其它tanslationunit中是否可以使用这个变量/函数)；生存期指一个变量的存活时间(从构造开始到析构结束)。

Auto和register属性的量有local生存期，而static及extern属性的量具有global生存期。
