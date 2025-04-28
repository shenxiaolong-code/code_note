利用编译期函数构建又快又小的程序

发布日期: 2024-08-02 13:29:22
原文链接: https://blog.csdn.net/Tonny0832/article/details/140869875

---

**1\. 需求背景： **  
通常情况下，我们都喜欢又快又小的程序，谁都不喜欢动不动几个G的程序。  
而一个程序变成几个G，用户不喜欢，其实开发者也不喜欢，其往往都有不得已的原因:要么是功能数量太多，要么是没做尽可能多的大小优化。  
binary文件size的优化方法有非常多，我们这儿不一一列举，这儿只探讨通过编译期模板函数来优化binary文件size的方法。

**2\. 通常观点**  
一般说来，模板会导致代码膨胀，进而导致binary文件的size变大。  
比如：
[code] 
    ```
    template<typename T>
    struct SomeClassTemplate
    {
      void someApis();
    }
    
    template<typename T>
    void someTemplateFunc(T const& rObj)
    {
      //doSomething
    }
    ```
    
[/code]

当用户使用不同的类型参数传入，并且调用 someApis / someTemplateFunc 时，会生成一个新函数体，这种运行期模板函数确实会导致代码膨胀，从而生成文件变大。

**3\. 优化原理**  
并不是所有的模板代码就会导致代码膨胀，从而使生成文件变大的. 那就是纯编译期模板函数tag类。  
比如：
[code] 
    ```
    template <typename T>           struct Type2Type                          {  typedef T type;                 };
    template <typename T, T val>    struct Value2Type : public Type2Type<T>   {  static constexpr T value = val; };
    template <int Val>              using IntType = Value2Type<int , Val>{};
    
    template<NumericTypeID id> struct getBits;
    template<>                 struct getBits<NumericTypeID::kVoid> :IntType<2>{};
    
    template<NumericTypeID id> struct getRelatedType;
    template<>                 struct getRelatedType<NumericTypeID::kBool> : public Type2Type<bool>{};
    ```
    
[/code]

  
3.1 结论  
这儿先说结论，后面继续分析原因细节：  
a. 运行期模板类/模板函数一定会导致代码膨胀，从而导致生成程序变大。  
b. 纯编译期函数不会导致代码膨胀，其会使binary文件的size变小(是的，你没有看错，是变小而不是变大)，并且运行性能提高(鱼和熊掌兼得了，不用喂料的千里马)。  
注： 本贴中说的编译期函数其实并不是函数，而是模板类。 因为其作用类似于函数(输入=>输出)，并且其只在编译期生效，所以称之为编译期函数。比如: Value2Type

**4 技术细节分析**  
增加一个新的符号(模板类)，其相对于binary大小新增加的代码有：

  1. 这个新类本身占用的大小 ： sizeof(getBits). 这个大小是固定值。 （对于纯编译期函数，这个size最终会在优化时被擦除掉的）
  2. 调用这个新类的功能时，其对caller函数的代码字节数的影响。 这个影响是和调用次数成正比的， N个不同的函数调用，就会有N倍的影响。



对于1,我们的测量手段是 sizeof。 对于一个标签类(tag class，空类)， 其sizeof依赖于编译器，但通常是1或者0.   
对于2,我们的测量手段是 调用函数的汇编代码。 ( 使用工具 <https://godbolt.org/> )

**5\. 测试分析的源代码**  
这儿提供的源代码，仅仅用于论证编译期函数可以优化binary文件的size，请各位不必执意于一些逻辑的正确性。（比如NumericAttributes的第一个构造函数应该根据type_id来swich...case设置不同的其余成员值）  
各位可以把下面代码粘贴到 <https://godbolt.org/> 上面，查看生成的汇编代码
[code] 
    ```
    #include <iostream>
    enum class NumericTypeID {
      kVoid = 0,  
      kBool,
      kF64,
      kComplex,
      kCF64,      
    };
    
    struct NumericAttributes {
    
      NumericTypeID       type_id_;
      NumericTypeID       real_;
      NumericTypeID       complex_;  
    
      int32_t             bits_;
      bool                is_real_;
      bool                is_signed_;
      int32_t             exponent_bits_;
      int32_t             mantissa_bits_;
      char const*         cpp_;
    
      NumericAttributes(NumericTypeID  type_id){
      // it is not real value, just for quick example. ignore switch ... case (type_id) procedure
      *this = {type_id, NumericTypeID::kVoid, NumericTypeID::kVoid, 3, false, false, 3, 2, "void"};
      }
    
    // initialize list ctor
      NumericAttributes(NumericTypeID type_id, NumericTypeID real, NumericTypeID complex, int32_t bits, 
      bool is_real, bool is_signed, int32_t exponent_bits, int32_t mantissa_bits, const char* cpp)
      : type_id_(type_id), real_(real), complex_(complex), bits_(bits), is_real_(is_real), 
        is_signed_(is_signed), exponent_bits_(exponent_bits), mantissa_bits_(mantissa_bits), cpp_(cpp) 
      {}
    
      int32_t bits() const { return bits_; }
    };
    
    template<typename T>         struct type2type                       {  typedef T type;                 };
    template<typename T, T val > struct typeValue : public type2type<T> {  static constexpr T value = val; };
    
    // the value is for example , not real value and not all enum values are used
    // index   type_id_     real_       complex_     bits_  is_real_  is_signed_  exponent_bits_  mantissa_bits_   cpp_ 
    #define declare_NumericAttributes(_)                                                                            \
      _( 0 , kVoid, kVoid, kVoid, 0,   false,  false,  0,  0,  "void"                     )                         \
      _( 1 , kBool, kBool, kVoid, 1,   true,   false,  0,  1,  "bool"                     )                         \
      _( 2 , kCF64, kF64,  kCF64, 128, false,  true,  11,  54, "cutlass::complex<double>" )                         \
    
    template <NumericTypeID id> struct getBits;
    #define declare_NumericAttributes_Bits(index, type_id, real, complex, bits, is_real, is_signed, exponent_bits, mantissa_bits, cpp )  template<> struct getBits<NumericTypeID::type_id> : public typeValue< int32_t, bits > {};
    declare_NumericAttributes(declare_NumericAttributes_Bits)
    
    // *************************************************  test begin  *************************************************
    template <NumericTypeID id> void print_bits_use_compile_value(){
      printf("compile_value : %d\r\n", getBits<id>::value);  
    }
    
    template <NumericTypeID id> void print_bits_use_runtime_value(){
      printf("runtime_value : %d\r\n", NumericAttributes(id).bits());
    }
    
    void test_bits(){
      print_bits_use_compile_value<NumericTypeID::kBool>();
      print_bits_use_runtime_value<NumericTypeID::kBool>();
    }
    
    int main() {
        test_bits();
        return 0;
    }
    ```
    
[/code]

**6\. 测试函数**  
由于新增加的符号 getBits 的sizeof(getBits)是固定的，所以我们这儿只需要分析下面两个测试函数的汇编代码即可。  
为了进一步简化论证过程，这儿我们只着眼着根据 NumericTypeID id 来取得 bits 值的过程， printf 并不是必要的操作。  

[code] 
    ```
    template <NumericTypeID id> void print_bits_use_compile_value(){
      printf("compile_value : %d\r\n", getBits<id>::value);  
    }
    
    template <NumericTypeID id> void print_bits_use_runtime_value(){
      printf("runtime_value : %d\r\n", NumericAttributes(id).bits());
    }
    ```
    
[/code]

**7\. 这两个函数的汇编代码：**
[code] 
    ```
    .LC1:
            .string "compile_value : %d\r\n"
    void print_bits_use_compile_value<(NumericTypeID)1>():
            push    rbp
            mov     rbp, rsp
            mov     esi, 1
            mov     edi, OFFSET FLAT:.LC1
            mov     eax, 0
            call    printf
            nop
            pop     rbp
            ret
    .LC2:
            .string "runtime_value : %d\r\n"
    void print_bits_use_runtime_value<(NumericTypeID)1>():
            push    rbp
            mov     rbp, rsp
            sub     rsp, 48
            lea     rax, [rbp-48]
            mov     esi, 1
            mov     rdi, rax
            call    NumericAttributes::NumericAttributes(NumericTypeID) [complete object constructor]
            lea     rax, [rbp-48]
            mov     rdi, rax
            call    NumericAttributes::bits() const
            mov     esi, eax
            mov     edi, OFFSET FLAT:.LC2
            mov     eax, 0
            call    printf
            nop
            leave
            ret
    ```
    
[/code]

**8\. 汇编代码的分析**  
截止于 call printf ，我们可以看到 print_bits_use_runtime_value 函数的大小比 print_bits_use_compile_value 大不少，并且 print_bits_use_compile_value 函数中只有一个 printf 符号，不再有 getBits<id> 的符号了。 (这其实是编译器的 constant folding 优化技术).

  - print_bits_use_compile_value : 二进制文件中汇编指令简单， 指令数量少， 用的函数栈空间也小
  - print_bits_use_runtime_value : 二进制文件中汇编指令复杂， 指令数量多， 用的函数栈空间也大，其中还有其它的运行时符号(NumericAttributes::NumericAttributes , NumericAttributes::bits)



进一步的分析

  - 在 print_bits_use_compile_value 函数中， getBits<id> 一旦编译完， getBits<id> 就失去了价值，其永远不会在程序运行时起作用。 所以一旦编译完，getBits<id>这个符号就会在优化时被擦除掉，在链接时，其就属于未被任何函数链接使用的符号了，那么其就没有存在的必要了(只在调试时有用),编译器/链接器优化擦除它就是理所当然的事了。 这即意味着 其仅仅占用的 sizeof（getBits<id>） 模块空间也被释放了。 真正地做到了 "事了拂衣去，不留功与名"。
  - 新符号本身大小对binary大小的影响过程：  
由于新符号的引入导致binary尺寸大小临时增大 => 使用新符号 => 新符号使用完后擦除新符号 => 由于符号擦除导致binary尺寸恢复原样 => 结论 ： 新符号的引入并没占用额外的binary size。
  - 编译期函数对binary size的优化主要在于其对caller 函数大小的贡献：其通常会减小caller函数的大小，从而减小了整个binary size的大小。当N个不同函数调用编译期函数时，其优化效果也是N倍的。
  - 在一些主流编译器上，虽然能够尽量优化得 print_bits_use_runtime_value 在函数大小上接近 print_bits_use_compile_value ，但是性能的差别，却是不可避免的。
  - 结论 ： print_bits_use_compile_value 比 print_bits_use_runtime_value 占用栈空间少，也占用binary空间小，还运行速度快，质量上全面碾压对方。



**9\. 结语**  
用 print_bits_use_compile_value 和 print_bits_use_runtime_value 的对比，演示了怎么用编译期函数来减少栈/模块大小优化的原理及用法。  
模板并不是总是导致代码膨胀的，在适合的场景用合适的模板，也可以导致模块缩小并且性能更高。 模板的功效如何，最终取决于你怎么用它。
