简单地使用sym*函数族导出函数栈(Based on x86)

发布日期: 2013-10-08 11:05:50
原文链接: https://blog.csdn.net/Tonny0832/article/details/12422311

---

有时我们需要导出当前线程的函数栈调用(特别是异常发生时)，但是我们又不期望自己来实现这个功能，下面的源代码可以帮你解决。

导出函数栈其实就是sym*函数族的应用，该函数在dbghelp.dll中的，下面只是实现了简单的用法，使用的缺省的符号搜索路径，没有设置用户自定义的搜索路径：

这样有两方面的考虑： 

1\. 这不是一个GUI程序，不想让使用者觉得麻烦，这里更合适的说法是提供了一个函数供用户调用。

2.一般说来，在编译链接时，搜索路径已经被嵌到符号文件中去了，如果有符号文件，一切OK。如果没有符号文件，抱歉，你设置了搜索路径也没有什么用处。

用法：

StackExplorer se(GetCurrentProcess());

se.DumpStack();

OK，就这么简单，如果你想接收输出信息，你可以替换其中的缺省参数。

//StackExplorer.h

#ifndef StackExplorer_h__  
#define StackExplorer_h__  
/********************************************************************  
created: 22:8:2013 14:06  
author: Shen Xiaolong  
  
purpose: use dbghlp.dll to resolve symbol info.   
refer to (CodeProject --> StackWalker)  
*********************************************************************/  
  
  
#include <string>  
#include <vector>  
#include <wtypes.h>  
#include <winnt.h>  
#include "config.h"  
  
  
struct ModInfo  
{  
stlString m_name;  
PVOID m_baseAddr;  
PVOID m_entryPoint; //for exe : main/winMain for dll : DllMain  
stlString m_entryName;  
DWORD m_size;  
  
  
ModInfo()  
{  
m_baseAddr = 0;  
m_entryPoint= 0;  
m_size = 0;  
}  
};  
  
  
struct FuncInfo  
{  
stlString m_name;  
stlString m_unDecorateName;  
PVOID m_addr;  
DWORD m_AsmOffset;  
FuncInfo()  
{  
m_addr = 0;  
m_AsmOffset = 0;  
}  
};  
  
  
struct AddressInfo  
{  
PVOID m_addr;  
stlString m_fileName;  
DWORD m_lineNumber;  
ModInfo m_mod;  
FuncInfo m_func;  
AddressInfo()  
{   
m_addr = 0;  
m_lineNumber = 0;  
}  
};  
  
  
struct CallStack  
{  
stlString m_funcName;  
stlString m_srcFileName;  
int m_lineNumber;  
PVOID m_funcAddr;  
DWORD m_AsmOffset;  
CallStack()  
{  
m_lineNumber = 0;  
m_funcAddr = 0;  
m_AsmOffset = 0;  
}  
};  
  
  
struct CallStackS  
{  
DWORD m_processId;  
DWORD m_threadId;  
std::vector<CallStack> m_stacks;  
  
  
CallStackS()  
{  
m_processId = 0;  
m_threadId = 0;  
}  
};  
  
  
class StackExplorer  
{  
public:  
StackExplorer(_In_ HANDLE hProcess = GetCurrentProcess());  
~StackExplorer();  
  
  
BOOL resloveAddrInfo(_In_ PVOID addr, _Inout_ AddressInfo& addrInfo);  
BOOL resloveModuleInfo(_In_ PVOID addr, _Inout_ ModInfo& modInfo);  
BOOL resloveFuncInfo(_In_ PVOID addr, _Inout_ FuncInfo& funcInfo);  
BOOL resloveSourceInfo(_In_ PVOID addr, _Inout_ stlString& fileName, _Inout_ DWORD& lineNumber);  
  
  
BOOL DumpStack(_In_ HANDLE hThread = GetCurrentThread(),_In_ CONTEXT* pThrdContext=NULL,_In_ BOOL bPrint=TRUE,_Inout_ CallStackS* pOutputStks=NULL);  
void print(_In_ CallStackS const& cs);  
void print(_In_ CallStack const& sk);  
  
  
private:  
HANDLE m_hProcess;   
};  
  
  
void handleException(_In_ _EXCEPTION_POINTERS * pExp );  
  
  
/  
// The "ugly" assembler-implementation is needed for systems before XP  
// If you have a new PSDK and you only compile for XP and later, then you can use   
// the "RtlCaptureContext"  
// Currently there is no define which determines the PSDK-Version...   
// So we just use the compiler-version (and assumes that the PSDK is   
// the one which was installed by the VS-IDE)  
  
  
// INFO: If you want, you can use the RtlCaptureContext if you only target XP and later...  
// But I currently use it in x64/IA64 environments...  
//#if defined(_M_IX86) && (_WIN32_WINNT <= 0x0500) && (_MSC_VER < 1400)  
  
  
#if defined(_M_IX86)  
#ifdef CURRENT_THREAD_VIA_EXCEPTION  
// TODO: The following is not a "good" implementation,   
// because the callstack is only valid in the "__except" block...  
#define GET_CURRENT_CONTEXT(c, contextFlags) \  
do { \  
memset(&c, 0, sizeof(CONTEXT)); \  
EXCEPTION_POINTERS *pExp = NULL; \  
__try { \  
throw 0; \  
} __except( ( (pExp = GetExceptionInformation()) ? EXCEPTION_EXECUTE_HANDLER : EXCEPTION_EXECUTE_HANDLER)) {} \  
if (pExp != NULL) \  
memcpy(&c, pExp->ContextRecord, sizeof(CONTEXT)); \  
c.ContextFlags = contextFlags; \  
} while(0);  
#else  
// The following should be enough for walking the callstack...  
#define GET_CURRENT_CONTEXT(c, contextFlags) \  
do { \  
memset(&c, 0, sizeof(CONTEXT)); \  
c.ContextFlags = contextFlags; \  
__asm call x \  
__asm x: pop eax \  
__asm mov c.Eip, eax \  
__asm mov c.Ebp, ebp \  
__asm mov c.Esp, esp \  
} while(0);  
#endif  
  
  
#else  
  
  
// The following is defined for x86 (XP and higher), x64 and IA64:  
#define GET_CURRENT_CONTEXT(c, contextFlags) \  
do { \  
memset(&c, 0, sizeof(CONTEXT)); \  
c.ContextFlags = contextFlags; \  
RtlCaptureContext(&c); \  
} while(0);  
#endif  
  
  
#endif // StackExplorer_h__

//StackExplorer.cpp

#include "StackExplorer.h"  
#include <Psapi.h>  
#include <imagehlp.h>  
#include <Aclapi.h>  
#include <iostream>  
#include <iomanip>  
#include "exptxt.h"  
#include "algorithm.h"  
#include "dataSet.h"  
#include "functionObject.h"  
  
  
/  
#pragma comment(lib,"Psapi")  
#pragma comment(lib,"imagehlp")  
  
  
//Init symbols //  
BOOL g_bSymInited = FALSE;  
HANDLE gs_hProcess = NULL;  
  
  
void __cdecl symClean()  
{  
SymCleanup(gs_hProcess);  
}  
  
  
void __cdecl symInit()  
{  
SymSetOptions(SymGetOptions() | SYMOPT_FAIL_CRITICAL_ERRORS | SYMOPT_LOAD_LINES );  
gs_hProcess = GetCurrentProcess();  
g_bSymInited = SymInitialize(gs_hProcess, NULL, TRUE);  
  
  
atexit(symClean);  
}  
  
  
#define SECNAME ".CRT$XCA"  
#pragma section(SECNAME,long, read)   
typedef void (__cdecl *_PVFV)();  
__declspec(allocate(SECNAME)) _PVFV dummy[] = { symInit };  
  
  
/  
  
  
StackExplorer::StackExplorer(HANDLE hProcess)  
: m_hProcess(hProcess)  
{  
if (!g_bSymInited)  
{  
symInit();  
}  
}  
  
  
StackExplorer::~StackExplorer()  
{   
}  
  
  
BOOL StackExplorer::resloveAddrInfo( _In_ PVOID addr, _Inout_ AddressInfo& addrInfo )  
{  
if (!g_bSymInited)  
{  
return FALSE;  
}  
  
  
memset(&addrInfo,0,sizeof(AddressInfo));  
addrInfo.m_addr = addr;  
resloveModuleInfo(addr,addrInfo.m_mod);   
resloveFuncInfo(addr,addrInfo.m_func);   
resloveSourceInfo(addr,addrInfo.m_fileName,addrInfo.m_lineNumber);  
return TRUE;  
}  
  
  
BOOL StackExplorer::resloveModuleInfo( _In_ PVOID addr, _Inout_ ModInfo& modInfo )  
{  
if (!g_bSymInited)  
{  
return FALSE;  
}  
  
  
BOOL bRet = FALSE;  
  
  
MEMORY_BASIC_INFORMATION mbi;  
if (VirtualQuery(addr,&mbi,sizeof(mbi)))  
{  
modInfo.m_baseAddr = mbi.AllocationBase;  
  
  
MODULEINFO mi;  
if (GetModuleInformation(m_hProcess,HMODULE(mbi.AllocationBase),&mi,sizeof(MODULEINFO)))  
{ //or SymGetModuleInfo   
modInfo.m_size = mi.SizeOfImage;  
modInfo.m_entryPoint = mi.EntryPoint;  
if (modInfo.m_entryPoint != addr) //avoid fall in loop  
{  
FuncInfo fi;  
resloveFuncInfo(modInfo.m_entryPoint,fi);  
modInfo.m_entryName = fi.m_name;  
}  
}  
  
  
char moduleName[256] = {0};  
if(GetModuleBaseNameA(m_hProcess,HMODULE(mbi.AllocationBase),moduleName,256))  
{  
modInfo.m_name = moduleName;  
}  
  
  
return TRUE;  
}  
  
  
return FALSE;  
}  
  
  
BOOL StackExplorer::resloveFuncInfo( _In_ PVOID addr, _Inout_ FuncInfo& funcInfo )  
{  
if (!g_bSymInited)  
{  
return FALSE;  
}  
  
  
DWORD64 asmDisplacement;  
ULONG64 buffer[(sizeof(SYMBOL_INFO)+MAX_SYM_NAME*sizeof(TCHAR)+sizeof(ULONG64)-1)/sizeof(ULONG64)];  
PSYMBOL_INFO pSymbol = (PSYMBOL_INFO)buffer;  
pSymbol->SizeOfStruct = sizeof(SYMBOL_INFO);  
pSymbol->MaxNameLen = MAX_SYM_NAME;   
  
  
if (SymFromAddr(m_hProcess,DWORD64(addr),&asmDisplacement,pSymbol))  
{  
funcInfo.m_addr = PVOID(pSymbol->Address);  
funcInfo.m_name = pSymbol->Name; //UnDecorateSymbolName  
funcInfo.m_AsmOffset= DWORD(asmDisplacement);  
  
  
char undName[256]={0};  
UnDecorateSymbolName(pSymbol->Name,undName,sizeof(undName),UNDNAME_NAME_ONLY);  
funcInfo.m_unDecorateName = undName;  
return TRUE;  
}  
  
  
return FALSE;  
}  
  
  
BOOL StackExplorer::resloveSourceInfo(_In_ PVOID addr, _Inout_ std::string& fileName, _Inout_ DWORD& lineNumber)  
{  
fileName = "";  
lineNumber = -1;  
if (!g_bSymInited)  
{  
return FALSE;  
}  
  
  
IMAGEHLP_LINE line;  
line.SizeOfStruct = sizeof(IMAGEHLP_LINE);  
DWORD lineDisplacement;  
if(SymGetLineFromAddr(m_hProcess,DWORD64(addr),&lineDisplacement,&line))  
{  
fileName = line.FileName;  
lineNumber = line.LineNumber;  
return TRUE;  
}  
  
  
return FALSE;  
}  
  
  
BOOL StackExplorer::DumpStack( _In_ HANDLE hThread ,_In_ CONTEXT* pThrdContext,_In_ BOOL bPrint,_Inout_ CallStackS* pOutputStks )  
{  
if (!g_bSymInited)  
{  
return FALSE;  
}  
  
  
CallStackS cs;  
cs.m_processId = GetProcessId(m_hProcess);  
cs.m_threadId = GetThreadId(hThread);   
  
  
bool bIsCurThread = hThread==GetCurrentThread();  
CONTEXT context;   
if (pThrdContext)  
{  
context = *pThrdContext;  
}  
else  
{  
if (bIsCurThread)  
{  
GET_CURRENT_CONTEXT(context,CONTEXT_FULL); //RtlCaptureContext(&context);  
}  
else  
{  
SuspendThread(hThread);  
GetThreadContext(hThread,&context);   
}  
}  
  
  
STACKFRAME stackFrame;  
memset(&stackFrame, 0, sizeof(stackFrame));  
stackFrame.AddrPC.Mode = AddrModeFlat;  
stackFrame.AddrFrame.Mode = AddrModeFlat;  
stackFrame.AddrStack.Mode = AddrModeFlat;  
stackFrame.AddrReturn.Mode = AddrModeFlat;  
stackFrame.AddrBStore.Mode = AddrModeFlat;  
  
  
DWORD dwMachType;  
#if defined(_M_IX86)  
dwMachType = IMAGE_FILE_MACHINE_I386;  
stackFrame.AddrPC.Offset = context.Eip; //program counter  
stackFrame.AddrStack.Offset = context.Esp; //stack pointer  
stackFrame.AddrFrame.Offset = context.Ebp; //frame pointer  
#else  
//refer to ATL::_AtlThreadContextInfo::DoDumpStack to impl non-x86 machine  
#error("Unknown Target Machine");  
#endif  
  
  
while (StackWalk(dwMachType, m_hProcess, hThread,&stackFrame, &context, NULL,  
SymFunctionTableAccess, SymGetModuleBase, NULL))  
{  
if (stackFrame.AddrPC.Offset != 0)  
{  
CallStack sk;   
FuncInfo fi;  
if (resloveFuncInfo(PVOID(stackFrame.AddrPC.Offset),fi))  
{  
sk.m_funcAddr = fi.m_addr;  
sk.m_funcName = fi.m_name;  
sk.m_AsmOffset = fi.m_AsmOffset;  
}  
  
  
Static_Assert(sizeof(sk.m_lineNumber)>=sizeof(DWORD));  
if (resloveSourceInfo(PVOID(stackFrame.AddrPC.Offset),sk.m_srcFileName,(DWORD&)sk.m_lineNumber))  
{  
char dbgOut[256] = {0};  
sprintf_s(dbgOut,"%s(%d) :%s\n",sk.m_srcFileName.c_str(),sk.m_lineNumber,sk.m_funcName.c_str());  
OutputDebugStringA(dbgOut);  
}  
  
  
if (0 != sk.m_funcAddr)  
{  
cs.m_stacks.push_back(sk);  
}  
}  
}  
  
  
if (!bIsCurThread) ResumeThread(hThread);  
  
  
if (bPrint)  
{  
print(cs);  
}  
  
  
if (pOutputStks)  
{  
*pOutputStks = cs;  
}  
  
  
return TRUE;  
}  
  
  
void StackExplorer::print( CallStackS const& cs )  
{  
using namespace std;  
cout << "********************************************************************************************\n" ;  
cout << " Dump current process[ " << cs.m_processId << " ] - thread [" << cs.m_threadId << "] stack : \n" ;   
cout << "********************************************************************************************\n" ;  
cout << "Func. Addr\tAddr. offset\tFunc. Name\n";  
for (std::vector<CallStack>::const_iterator it=cs.m_stacks.begin();it != cs.m_stacks.end() ; ++it )  
{  
print(*it);  
}  
}  
  
  
void StackExplorer::print( _In_ CallStack const& sk)  
{  
using namespace std;  
//cout << "\nFunc. Addr\tAddr. offset\tFunc. Name\n";  
if (sk.m_funcAddr)  
{  
cout << std::showbase << setiosflags(ios::uppercase) << hex   
<< sk.m_funcAddr << "\t" << setw(10) << sk.m_AsmOffset << "\t"  
<< sk.m_funcName << endl;   
}  
  
  
if (0 != sk.m_srcFileName.size())  
{   
cout << sk.m_srcFileName << "(" << dec << sk.m_lineNumber << ")" << endl;  
}  
}  
  
  
/  
void interpreteException(EXCEPTION_RECORD const& ExpRecord)  
{   
using namespace std;  
using namespace UtilExt;  
ExceptionItem const* pExpText = findImpl(makeDataSet(exps),makeFinder(ExpRecord.ExceptionCode,MMP(&ExceptionItem::m_expCode)));  
cout << std::showbase << setiosflags(ios::uppercase) << hex  
<< "Exception Code\tMeaning\n"  
<< ExpRecord.ExceptionCode << "\t"   
<< pExpText->m_expText << "\n"  
<< "Continuable ?" << (EXCEPTION_NONCONTINUABLE==ExpRecord.ExceptionFlags ? "\tNo\n" : "\tYes\n" );  
if (ExpRecord.NumberParameters)  
{  
cout << "Available param(s):\t";  
for(DWORD i=0;i<ExpRecord.NumberParameters;++i)  
{   
cout << ExpRecord.ExceptionInformation[i] << "\t";  
}  
  
  
switch(ExpRecord.ExceptionCode)  
{ //ms-help://MS.MSDNQTR.v90.en/debug/base/exception_record_str.htm  
case EXCEPTION_ACCESS_VIOLATION:  
case EXCEPTION_IN_PAGE_ERROR:  
cout << ( 0 == ExpRecord.ExceptionInformation[0] ? "\nRead " : (1==ExpRecord.ExceptionInformation[0] ? "\nWrite":"\ndata execution prevention (DEP)"))   
<< " fails, Addr=" << ExpRecord.ExceptionInformation[1] << "\n";  
break;  
default:  
break;  
}  
}  
  
  
if (ExpRecord.ExceptionRecord)  
{  
cout << "Additional info " ;  
interpreteException(*ExpRecord.ExceptionRecord);  
}  
}  
  
  
void handleException(_In_ _EXCEPTION_POINTERS * pExp )  
{  
interpreteException(*pExp->ExceptionRecord);  
  
  
using namespace std;  
StackExplorer se(GetCurrentProcess());  
AddressInfo ai;  
se.resloveAddrInfo(pExp->ExceptionRecord->ExceptionAddress,ai);  
char dbgOut[256] = {0};  
sprintf_s(dbgOut,"\nException code position:\n%s(%d) : %s\n",ai.m_fileName.c_str(),ai.m_lineNumber,ai.m_func.m_name.c_str());  
cout << dbgOut ;  
OutputDebugStringA(dbgOut);  
  
  
EXCEPTION_RECORD& rExp = *pExp->ExceptionRecord;  
cout << "Exception info \nExp. Code\tExp.Addr\tModule base addr\tModule name\n";  
cout.setf(ios_base::showbase);  
cout<< std::showbase << setiosflags(ios::uppercase) << hex  
<< setw(10) << rExp.ExceptionCode << "\t"   
<< (DWORD)rExp.ExceptionAddress << "\t"   
<< (DWORD)ai.m_mod.m_baseAddr << "\t\t"   
<< ai.m_mod.m_name << endl;  
  
  
cout << "\nASM function Info\nFunc. Addr\tExp. offset\tFunc. Name\n";   
cout << std::showbase << setiosflags(ios::uppercase) << hex   
<< (DWORD)ai.m_func.m_addr  
<< "\t" << setw(10) << ai.m_func.m_AsmOffset << "\t"   
<< ai.m_func.m_name << endl;  
  
  
se.DumpStack(GetCurrentThread(),pExp->ContextRecord);  
return ;  
}

用法及测试代码//

//test_StackExplorer.h

#ifndef test_StackExplorer_h__  
#define test_StackExplorer_h__  
#include "StackExplorer\StackExplorer.h"  
  
  
//#define RUN_EXAMPLE_STACKEXPLORER  
  
  
#ifdef COMPILE_EXAMPLE_ALL  
#define COMPILE_EXAMPLE_STACKEXPLORER  
#endif  
  
  
#ifdef RUN_EXAMPLE_ALL  
#define RUN_EXAMPLE_STACKEXPLORER  
#endif  
  
  
#if defined(RUN_EXAMPLE_STACKEXPLORER) && !defined(COMPILE_EXAMPLE_STACKEXPLORER)  
#define COMPILE_EXAMPLE_STACKEXPLORER  
#endif  
  
  
/  
#ifdef COMPILE_EXAMPLE_STACKEXPLORER  
  
  
int g_int = 0;  
int* g_intP = NULL;  
  
  
int Func5()   
{  
// StackExplorer se(GetCurrentProcess());  
// se.DumpStack(GetCurrentThread(),NULL);  
  
  
//RaiseException(1,0,2,0);   
*g_intP = 0x5678;  
//return 5/g_int ;  
  
  
return 0;  
}  
void Func4() { Func5(); }  
void Func3()   
{   
g_intP = (int*)0x1234;  
Func4();   
}  
void Func2() { Func3(); }  
  
  
int testException()  
{  
__try  
{  
Func2();   
}  
__except(handleException(GetExceptionInformation()), EXCEPTION_EXECUTE_HANDLER)  
{  
}  
return 0;  
}  
  
  
#ifdef RUN_EXAMPLE_STACKEXPLORER  
InitRunFunc(testException);  
#endif  
  
  
#endif // COMPILE_EXAMPLE_STACKEXPLORER  
  
  
  
  
#endif // test_StackExplorer_h__
