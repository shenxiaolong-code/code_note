15). 回调模板函数/回调模板成员函数，回调参数个数和类型不固定，消除类似代码

发布日期: 2014-11-30 11:28:58
原文链接: https://blog.csdn.net/Tonny0832/article/details/41621173

---

完整的源代码及用法测试代码可以在我的资源中去下载.  


  


C++语法不支持模板函数/模板成员函数作为回调函数，同时把运行期代码向编译期代码转换也只有switch...case或者if..else能够实现。

如果case比较多的时候，代码非常臃肿，而且类似的大片代码中，往往只有一个参数的不同，其它都是相同的。这对于用户来说，都是一个大量的重复性的hard-code性的负担，而且也容易导致出错。

  


本库采用了封装，可以把运行性的switch...case/if..else分支选择转换为编译期的代码，支持模板函数的回调，并且支持最多6个可变参数（可以简易扩充参数个数），从而可以大幅度地减少代码量，(代码量可以减少到原代码的1/N，N为case数目)。并且由于是编译期代码，从而保证不会出错(出错将导致编译失败)。

  


并且附带提供了CEnumRange，可以检测一个枚举值是否在枚举范围内。

  


namespace UnitTest  
{  
template<SeqEunm TVal>  
bool globalEnumFunc(int a,char* pStr)  
{  
printf("globalEnumFunc [%d],param=%d,%s",TVal+1,a,pStr?pStr:"NULL");  
return true;  
}  
Declare_Template_Global_CallBack(globalEnumFunc,globalEnumFunc);  
  
  
struct CNonSeqEunmInterface_1  
{  
CNonSeqEunmInterface_1(int p) : m_bResult(false),m_param(p){};  
  
  
template<NonSeqEunm TVal>  
void execute()  
{  
dbgOutput(TXT("\n\n[NonSeqEunm]Enum value is %d."),TVal);  
dbgOutput(TXT("\n[CEnumInterface_1::execute] %d\n"),TVal);  
m_bResult = (TVal+m_param) > NS_Val_Max;  
}  
  
  
bool m_bResult;  
int m_param;  
};  
Declare_Template_Global_CallBack(CNonSeqEunmInterface_1,::UnitTest::CNonSeqEunmInterface_1::execute);  
  
  
struct CNonSeqEunmInterface_2  
{  
CNonSeqEunmInterface_2(VCHAR* pStr) : m_bResult(false),m_pStr(pStr){};  
  
  
template<NonSeqEunm TVal>  
void execute()  
{  
dbgOutput(TXT("\n\n[NonSeqEunm]Enum value is %d."),TVal);  
dbgOutput(TXT("\n[CEnumInterface_2::execute] %s %d\n"),m_pStr,TVal);  
m_bResult = (TVal+2) > NS_Val_Max;  
}  
  
  
bool m_bResult;  
VCHAR* m_pStr;  
};  
Declare_Template_Global_CallBack(CNonSeqEunmInterface_2,::UnitTest::CNonSeqEunmInterface_2::execute);  
  
  
struct CSeqEunmInterface_1  
{  
CSeqEunmInterface_1(int p) : m_bResult(false),m_param(p){};  
  
  
template<SeqEunm TVal>  
void execute()  
{  
dbgOutput(TXT("\n\n[SeqEunm]Enum value is %d."),TVal);  
dbgOutput(TXT("\n[CSeqEunmInterface_1::execute] %d\n"),TVal);  
m_bResult = (TVal+m_param) > NS_Val_Max;  
}  
  
  
bool m_bResult;  
int m_param;  
};  
Declare_Template_Global_CallBack(CSeqEunmInterface_1,::UnitTest::CSeqEunmInterface_1::execute);  
  
  
struct CSeqEunmInterface_2  
{  
CSeqEunmInterface_2(VCHAR* pStr) : m_bResult(false),m_pStr(pStr){};  
  
  
template<SeqEunm TVal>  
void execute()  
{  
dbgOutput(TXT("\n\n[SeqEunm]Enum value is %d."),TVal);  
dbgOutput(TXT("\n[CSeqEunmInterface_2::execute] %s %d\n"),m_pStr,TVal);  
m_bResult = (TVal+2) > NS_Val_Max;  
}  
  
  
bool m_bResult;  
VCHAR* m_pStr;  
};  
Declare_Template_Global_CallBack(CSeqEunmInterface_2,::UnitTest::CSeqEunmInterface_2::execute);  
  
  
class CTestEnumSuite  
{  
  
  
public:  
template<NonSeqEunm TVal>  
bool printEnum()  
{  
outputTxt((TXT("CTestEnumSuite::printEnum [%d]\n"),TVal));  
return true;  
}  
  
  
template<SeqEunm TVal>  
bool printNext()  
{  
outputTxt((TXT("CTestEnumSuite::printNext [%d]\n"),TVal+1));  
return true;  
}  
  
  
template<SeqEunm TVal>  
void inOutParam(char p1,int& outParam)  
{  
outputTxt((TXT("CTestEnumSuite::inOutParam [%d]\n"),TVal+1));  
outParam = p1+2;  
}  
  
  
template<SeqEunm TVal>  
int getReturnValue(char p1,int outParam)  
{  
outputTxt((TXT("CTestEnumSuite::inOutParam [%d]\n"),TVal+1));  
return outParam+p1+TVal+4;  
}  
  
  
template<SeqEunm TVal>  
bool print2Param(int a,char* pStr)  
{  
printf("CTestEnumSuite::print2Param [%d],param=%d,%s\n",TVal+1,a,pStr?pStr:"NULL");  
return true;  
}  
  
  
Declare_Template_Global_CallBack(printEnum,::UnitTest::CTestEnumSuite::printEnum );  
Declare_Template_Global_CallBack(printNext,::UnitTest::CTestEnumSuite::printNext );  
Declare_Template_Global_CallBack(print2Param,::UnitTest::CTestEnumSuite::print2Param );  
Declare_Template_Global_CallBack(inOutParam,::UnitTest::CTestEnumSuite::inOutParam );  
Declare_Template_Global_CallBack(getReturnValue,::UnitTest::CTestEnumSuite::getReturnValue );  
  
  
void testEntry()  
{  
int a = 5;  
char* pStr = "in testEntry";  
srand( (unsigned)time( NULL ) );   
  
  
typedef CTraverseTypeSet<SeqEnumRange,false> SeqEnumRange0;  
typedef CTraverseTypeSet<SeqEnumRange,true> SeqEnumRange1;  
typedef CTraverseTypeSet<NonSeqEnumRange,false> NonSeqEnumRange0;  
typedef CTraverseTypeSet<NonSeqEnumRange,true> NonSeqEnumRange1;  
  
  
  
  
//test member function  
typedef bool (CTestEnumSuite::*pNoParam)();  
typedef bool (CTestEnumSuite::*pMemParam2)(int,char*);  
  
  
ASSERT_AND_LOG((NonSeqEnumRange0::applyTester<>::execute<Callback_printEnum<pNoParam> >(NS_Val_5,makeParamPackage_Ref(*this))));  
ASSERT_AND_LOG((NonSeqEnumRange1::applyTester<>::execute<Callback_printEnum<pNoParam> >(NS_Val_5,makeParamPackage_Ref(*this))));  
ASSERT_AND_LOG((SeqEnumRange0::applyTester<>::execute<Callback_printNext<pNoParam> >(S_Val_3,makeParamPackage_Ref(*this))));  
  
  
ParmPackage<CTestEnumSuite&,int,char*> params_print2Param(*this,a,pStr);  
ASSERT_AND_LOG((SeqEnumRange1::applyTester<>::execute<Callback_print2Param<pMemParam2> >(S_Val_6,params_print2Param)));  
  
  
//test foreach  
SeqEnumRange1::forEach<Callback_print2Param<pMemParam2> >(params_print2Param);  
  
  
//test in-out parameter   
char p1 = 3;  
int outP = 6;  
typedef void (CTestEnumSuite::*pParam2ci)(char,int&);  
ParmPackage<CTestEnumSuite&,char,int&> params_inOutParam(*this,p1,outP);  
ASSERT_AND_LOG((SeqEnumRange0::applyTester<>::execute<Callback_inOutParam<pParam2ci> >(S_Val_5,params_inOutParam)));  
ASSERT_AND_LOG((p1+2)==outP);  
ASSERT_AND_LOG(SeqEnumRange1::applyTester<>::execute<Callback_inOutParam<pParam2ci> >(S_Val_5,makeParamPackage_Ref(*this,p1,outP)));  
ASSERT_AND_LOG((p1+2)==outP);  
  
  
typedef int (CTestEnumSuite::*pGetRetVal)(char,int);  
ASSERT_AND_LOG((p1+outP+S_Val_5+4)==SeqEnumRange1::applyTester<>::execute<Callback_getReturnValue<pGetRetVal> >(S_Val_5,params_inOutParam,-1));  
  
  
pGetRetVal pf=SeqEnumRange1::applyTester<>::getFuncAddr<Callback_getReturnValue<pGetRetVal> >(S_Val_5);  
ASSERT_AND_LOG(pf);  
pf=SeqEnumRange1::applyTester<>::getFuncAddr<Callback_getReturnValue<pGetRetVal> >(S_Val_4);  
ASSERT_AND_LOG(!pf); //S_Val_4 is skipped  
  
  
stlMap<NonSeqEunm,pNoParam> rMap10;  
ASSERT_AND_LOG((NonSeqEnumRange0::applyFuncIdGenerator<>::getAllFuncAddr<Callback_printEnum<pNoParam> >(rMap10)));  
ASSERT_AND_LOG(CGetEnumRangeSize<NonSeqEnumRange>::value==rMap10.size());  
  
  
stlMap<NonSeqEunm,pNoParam> rMap11;  
ASSERT_AND_LOG((NonSeqEnumRange1::applyFuncIdGenerator<>::getAllFuncAddr<Callback_printEnum<pNoParam> >(rMap11)));  
ASSERT_AND_LOG(rMap10 ==rMap11 );  
  
  
stlMap<SeqEunm,pParam2ci> rMap20;  
ASSERT_AND_LOG((SeqEnumRange0::applyFuncIdGenerator<>::getAllFuncAddr<Callback_inOutParam<pParam2ci> >(rMap20)));  
  
  
stlMap<SeqEunm,pParam2ci> rMap21;  
ASSERT_AND_LOG((SeqEnumRange1::applyFuncIdGenerator<>::getAllFuncAddr<Callback_inOutParam<pParam2ci> >(rMap21)));  
ASSERT_AND_LOG(rMap20 ==rMap21 );  
  
  
stlMap<SeqEunm,pParam2ci> rMap21New;  
ASSERT_AND_LOG((SeqEnumRange1::applyFuncIdGenerator<>::getAllFuncAddr<Callback_inOutParam<pParam2ci> >(rMap21New)));  
ASSERT_AND_LOG(rMap20 ==rMap21New );  
  
  
ASSERT_AND_LOG((NonSeqEnumRange0::applyTester<>::execute<Callback_printEnum<pNoParam> >(NS_Val_5,makeParamPackage_Ref(*this))));  
ASSERT_AND_LOG((NonSeqEnumRange1::applyTester<>::execute<Callback_printEnum<pNoParam> >(NS_Val_5,makeParamPackage_Ref(*this))));  
  
  
//test global function  
typedef bool (*pGlobalParam2)(int,char*);  
ParmPackage<int,char*> params_globalEnumFunc(a,pStr);  
ASSERT_AND_LOG((SeqEnumRange0::applyTester<>::execute<Callback_globalEnumFunc<pGlobalParam2> >(S_Val_Min,params_globalEnumFunc)));  
ASSERT_AND_LOG((SeqEnumRange1::applyTester<>::execute<Callback_globalEnumFunc<pGlobalParam2> >(S_Val_Max,makeParamPackage_Ref(a,pStr))));  
ASSERT_AND_LOG(!(SeqEnumRange0::applyTester<>::execute<Callback_globalEnumFunc<pGlobalParam2> >(SeqEunm(S_Val_Max+1),params_globalEnumFunc)));  
}  
};  
  
  
//  
inline void TestCase_traverseTypeSet()  
{  
PrintTestcase();  
  
  
CNonSeqEunmInterface_1 nsCallObj_1(rand()%NS_Val_Max);  
CNonSeqEunmInterface_2 nsCallObj_2(TXT("I am CEnumInterface_2 :"));  
CSeqEunmInterface_1 sCallObj_1(rand()%S_Val_Max);  
CSeqEunmInterface_2 sCallObj_2(TXT("I am CSeqEunmInterface_2 :"));  
  
  
CTestEnumSuite testObj;  
testObj.testEntry();  
  
  
typedef CTraverseTypeSet<SeqEnumRange,false> SeqEnumRange0;  
typedef CTraverseTypeSet<SeqEnumRange,true> SeqEnumRange1;  
typedef CTraverseTypeSet<NonSeqEnumRange,false> NonSeqEnumRange0;  
typedef CTraverseTypeSet<NonSeqEnumRange,true> NonSeqEnumRange1;  
  
  
outputTxt(( SeqEnumRange0::dump().c_str() ));  
outputTxt(( SeqEnumRange1::dump().c_str() ));  
outputTxt(( NonSeqEnumRange0::dump().c_str() ));  
outputTxt(( NonSeqEnumRange1::dump().c_str() ));  
outputTxt(( CTraverseTypeSet<SeqEnumSkipOneRange,true>::dump().c_str() ));  
outputTxt(( CTraverseTypeSet<SeqEnumSkipOneRange,false>::dump().c_str() ));  
  
  
//test MiniMPL.CTraverseEnum.execute  
typedef void (CNonSeqEunmInterface_1::*pNoParamN11)();  
typedef void (CSeqEunmInterface_1::*pNoParamS11)();  
  
  
pNoParamS11 pf = SeqEnumRange1::applyTester<>::getFuncAddr<Callback_CSeqEunmInterface_1<pNoParamS11> >(S_Val_5);  
ASSERT_AND_LOG( &CSeqEunmInterface_1::execute<S_Val_5> == SeqEnumRange1::applyTester<>::getFuncAddr<Callback_CSeqEunmInterface_1<pNoParamS11> >(S_Val_5));  
ASSERT_AND_LOG( &CSeqEunmInterface_1::execute<S_Val_3> != SeqEnumRange1::applyTester<>::getFuncAddr<Callback_CSeqEunmInterface_1<pNoParamS11> >(S_Val_5));  
  
  
ASSERT_AND_LOG((NonSeqEnumRange1::applyTester<>::execute<Callback_CNonSeqEunmInterface_1<pNoParamN11> >(NS_Val_5,makeParamPackage_Ref(nsCallObj_1)))); //test OK case  
ASSERT_AND_LOG((NonSeqEnumRange0::applyTester<>::execute<Callback_CNonSeqEunmInterface_1<pNoParamN11> >(NS_Val_5,makeParamPackage_Ref(nsCallObj_1)))); //test OK case  
ASSERT_AND_LOG((SeqEnumRange1::applyTester<>::execute<Callback_CSeqEunmInterface_1<pNoParamS11> >(S_Val_5,makeParamPackage_Ref(sCallObj_1)))); //test OK case  
ASSERT_AND_LOG((SeqEnumRange0::applyTester<>::execute<Callback_CSeqEunmInterface_1<pNoParamS11> >(S_Val_5,makeParamPackage_Ref(sCallObj_1)))); //test OK case  
  
  
//test MiniMPL.CTraverseEnum.executeFrom   
typedef void (CNonSeqEunmInterface_2::*pNoParamN21)();  
typedef void (CSeqEunmInterface_2::*pNoParamS21)();  
  
  
ASSERT_AND_LOG((NonSeqEnumRange1::applyTester<>::executeFrom<NonSeqEunm,NS_Val_1,Callback_CNonSeqEunmInterface_2<pNoParamN21> >(NS_Val_3,makeParamPackage_Ref(nsCallObj_2)))); //test OK case  
ASSERT_AND_LOG((NonSeqEnumRange0::applyTester<>::executeFrom<NonSeqEunm,NS_Val_5,Callback_CNonSeqEunmInterface_2<pNoParamN21> >(NS_Val_3,makeParamPackage_Ref(nsCallObj_2)))); //test OK case  
ASSERT_AND_LOG((SeqEnumRange1::applyTester<>::executeFrom<SeqEunm,S_Val_1,Callback_CSeqEunmInterface_2<pNoParamS21> >(S_Val_3,makeParamPackage_Ref(sCallObj_2)))); //test OK case  
ASSERT_AND_LOG((SeqEnumRange0::applyTester<>::executeFrom<SeqEunm,S_Val_5,Callback_CSeqEunmInterface_2<pNoParamS21> >(S_Val_3,makeParamPackage_Ref(sCallObj_2)))); //test OK case  
  
  
#if defined(_MSC_VER) && (WINVER >= 0x0500)  
#pragma push_macro("Assert_Trigger")  
#pragma push_macro("ChkExceptionThrow")  
#undef Assert_Trigger  
#define Assert_Trigger() 0  
#undef ChkExceptionThrow  
#define ChkExceptionThrow() 0  
#endif //end defined(_MSC_VER)  
  
  
ASSERT_AND_LOG((NonSeqEnumRange1::applyTester<>::execute<Callback_CNonSeqEunmInterface_1<pNoParamN11> >(NonSeqEunm(NS_Val_Max+2),makeParamPackage_Ref(nsCallObj_1)))); //test Failure case because DemoEunm(NS_Val_Max+2) is not in range  
ASSERT_AND_LOG((NonSeqEnumRange0::applyTester<>::execute<Callback_CNonSeqEunmInterface_1<pNoParamN11> >(NonSeqEunm(NS_Val_Min-2),makeParamPackage_Ref(nsCallObj_1)))); //test Failure case because DemoEunm(NS_Val_Min-2) is not in range  
ASSERT_AND_LOG((SeqEnumRange1::applyTester<>::execute<Callback_CSeqEunmInterface_1<pNoParamS11> >(SeqEunm(S_Val_Max+2),makeParamPackage_Ref(sCallObj_1)))); //test Failure case because DemoEunm(S_Val_Max+2) is not in range  
ASSERT_AND_LOG((SeqEnumRange0::applyTester<>::execute<Callback_CSeqEunmInterface_1<pNoParamS11> >(SeqEunm(S_Val_Min-2),makeParamPackage_Ref(sCallObj_1)))); //test Failure case because DemoEunm(S_Val_Min-2) is not in range  
  
  
ASSERT_AND_LOG((NonSeqEnumRange1::applyTester<>::executeFrom<NonSeqEunm,NS_Val_5,Callback_CNonSeqEunmInterface_2<pNoParamN21> >(NS_Val_3,makeParamPackage_Ref(nsCallObj_2)))); //test Failure case because traverse from NS_Val_5. (NS_Val_3 < NS_Val_5)  
ASSERT_AND_LOG((NonSeqEnumRange0::applyTester<>::executeFrom<NonSeqEunm,NS_Val_1,Callback_CNonSeqEunmInterface_2<pNoParamN21> >(NS_Val_3,makeParamPackage_Ref(nsCallObj_2)))); //test Failure case because reversed traverse from NS_Val_1. (NS_Val_1 < NS_Val_3)  
ASSERT_AND_LOG((SeqEnumRange1::applyTester<>::executeFrom<SeqEunm,S_Val_5,Callback_CSeqEunmInterface_2<pNoParamS21> >(S_Val_3,makeParamPackage_Ref(sCallObj_2)))); //test Failure case because traverse from S_Val_5. (S_Val_3 < S_Val_5)  
ASSERT_AND_LOG((SeqEnumRange0::applyTester<>::executeFrom<SeqEunm,S_Val_1,Callback_CSeqEunmInterface_2<pNoParamS21> >(S_Val_3,makeParamPackage_Ref(sCallObj_2)))); //test Failure case because reversed traverse from S_Val_1. (S_Val_1 < S_Val_3)  
  
  
#if defined(_MSC_VER) && (WINVER >= 0x0500)  
#pragma pop_macro("Assert_Trigger")  
#pragma pop_macro("ChkExceptionThrow")  
#endif //end defined(_MSC_VER)  
}  
  
  
#ifdef RUN_EXAMPLE_TRAVERSEENUM  
InitRunFunc(showUsage_traverseEnum);  
#endif //RUN_EXAMPLE_TRAVERSEENUM  
}  
  
  
#endif // COMPILE_EXAMPLE_TRAVERSEENUM  
  
  
#endif // __test_TRAVERSEENUM_h_  

