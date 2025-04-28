遍历所有时区的代码[windows]

发布日期: 2013-10-10 10:15:47
原文链接: https://blog.csdn.net/Tonny0832/article/details/12558081

---

windows没有提供遍历时区的代码，只提供了一些接口，让用户自己从注册表中遍历。比较麻烦，下面代码把这个功能封装了一下，让用户可以比较简单地处理所有时区。

其中字符串用的是core::cstringex，大家可以用STD里面的字符串来代替，core::datetime里的数据结构体和SYSTEMTIME一样，大家可以把SYSTEMTIME封装一下就可以替换它了。

//timezone.h

#ifndef timezone_h__  
#define timezone_h__  
/********************************************************************  
purpose: support for time zone properties.  
*********************************************************************/  
#include "core\container\cvector.h"  
#include <winbase.h>  
#include "core\string\cstringex.h"  
#include "core\time\datetime.h"  
  
  
namespace Timezone  
{   
typedef struct _tagTimezoneInfo  
{  
core::cstringexm_displayName;  
TIME_ZONE_INFORMATIONm_tzi;  
} TimezoneInfo,*LPTimezoneInfo;  
  
  
core::cvector<TimezoneInfo> const&getTimeZoneList();  
TimezoneInfo const*lookupTimeZone(core::cstringex const& tziUniqueString);  
  
  
TimezoneInfo const&getTimeZone(unsigned int idx);  
bool setTimeZone(TIME_ZONE_INFORMATION const& tzi, bool bDaylightSaving); //invoker pays attention to WIN_CE   
  
  
core::datetime getLocalDatetime( core::datetime utc0DT,TIME_ZONE_INFORMATION const& tzi );  
core::datetime getUTC0Datetime( core::datetime localDT,TIME_ZONE_INFORMATION const& tzi );  
  
  
bool isDaylightSavingAvailable(TIME_ZONE_INFORMATION const& tzi);  
bool isInDaylightSavingPeriod(TIME_ZONE_INFORMATION const& tzi,core::datetime const& dt);  
}  
#endif // timezone_h__  


  


//timezone.cpp

#include <utility/timezone.h>  
#include "utility/macroDef.h"  
#include "core/string/cstringex.h"  
#include "utility/registerKey.h"  
#include "utility/algorithm.h"  
#include "utility/globalFunctions.h"  
  
  
namespace Timezone  
{  
core::cstringexconst TZI_DisplayName = L"Display";  
core::cstringexconst TZI_StandardName = L"Std";  
core::cstringexconst TZI_DaylightName = L"Dlt";  
core::cstringexconst TZI_TZI = L"TZI";  
  
  
#ifdef UNDER_CE  
core::cstringex const TZI_Key = L"Time Zones";  
#else  
core::cstringex const TZI_Key = L"SOFTWARE\\\Microsoft\\\Windows NT\\\CurrentVersion\\\Time Zones";  
#endif // UNDER_CE  
  
  
/  
  
  
bool InitTimeZoneList(core::cvector<TimezoneInfo>& arrTZI);  
core::cvector<TimezoneInfo> gs_TimeZoneList;  
InitRunFunc1(InitTimeZoneList,gs_TimeZoneList);  
  
  
bool InitTimeZoneList(core::cvector<TimezoneInfo>& arrTZI)   
{  
struct tagTZREG {  
LONG Bias;  
LONG StandardBias;  
LONG DaylightBias;  
SYSTEMTIME StandardDate;  
SYSTEMTIME DaylightDate;  
} regTZI={0};   
Static_Assert(sizeof(tagTZREG)==44);   
  
Util::CRegistry reg(HKEY_LOCAL_MACHINE);  
core::cvector<core::cstringex> tzNames;  
reg.listSubKeys(TZI_Key,tzNames);  
if (tzNames.size()==0)  
{  
return false;  
}  
  
  
core::cstringex parentKeyPath = TZI_Key + L"\\\";  
  
  
TimezoneInfo tzi;  
ULONG nBytes = sizeof(tagTZREG);  
core::cstringexdisplayName;  
core::cstringexstandardName;  
core::cstringexdaylightName;  
for (core::cvector<core::cstringex>::const_iterator it=tzNames.begin();it != tzNames.end();it++)  
{  
if (reg.open(parentKeyPath + *it))  
{  
bool bRet = reg.readStringValue(TZI_DisplayName,tzi.m_displayName);  
bRet &=reg.readStringValue(TZI_StandardName,standardName);  
bRet &=reg.readStringValue(TZI_DaylightName,daylightName);  
bRet &=reg.readBinaryValue(TZI_TZI,(unsigned char *)(&regTZI),nBytes);  
  
  
ASSERT_AND_LOG(bRet);  
if (bRet)  
{  
tzi.m_tzi.Bias= regTZI.Bias;  
tzi.m_tzi.DaylightBias= regTZI.DaylightBias;  
tzi.m_tzi.DaylightDate= regTZI.DaylightDate;   
tzi.m_tzi.StandardBias= regTZI.StandardBias;  
tzi.m_tzi.StandardDate= regTZI.StandardDate;  
  
  
wsprintf(tzi.m_tzi.StandardName,L"%s",standardName);  
wsprintf(tzi.m_tzi.DaylightName,L"%s",daylightName);  
  
  
arrTZI.push_back(tzi);  
}  
}  
}  
UtilExt::sort(arrTZI,&TimezoneInfo::m_tzi,&TIME_ZONE_INFORMATION::Bias,true,&UtilExt::lesser<LONG,LONG>);  
return arrTZI.size() != 0;  
}  
  
  
  
  
TimezoneInfo const& getTimeZone( unsigned int idx )  
{  
ASSERT_AND_LOG(gs_TimeZoneList.size()>0);  
ASSERT_AND_LOG(idx<gs_TimeZoneList.size());  
return gs_TimeZoneList[idx];  
}  
  
  
core::cvector<TimezoneInfo> const& getTimeZoneList()  
{   
ASSERT_AND_LOG(gs_TimeZoneList.size()>0);  
  
  
return gs_TimeZoneList;  
}  
  
  
bool isDaylightSavingAvailable( TIME_ZONE_INFORMATION const& tzi)  
{  
return 0!=memcmp(&tzi.DaylightDate,&tzi.StandardDate,sizeof(SYSTEMTIME));  
}  
  
  
bool isInDaylightSavingPeriod( TIME_ZONE_INFORMATION const& tzi,core::datetime const& dt )  
{  
ASSERT_AND_LOG(isDaylightSavingAvailable(tzi));  
SYSTEMTIME stdDt=tzi.StandardDate;  
SYSTEMTIME dltDt=tzi.DaylightDate;  
SYSTEMTIME curDt = dt.getSystemTime();  
stdDt.wYear = curDt.wYear;  
dltDt.wYear = curDt.wYear;  
  
  
__time64_t std_t = core::datetime(stdDt).getTime_t();  
__time64_t dlt_t = core::datetime(dltDt).getTime_t();  
__time64_t cur_t = core::datetime(curDt).getTime_t();  
  
  
if (dlt_t>std_t)  
{  
return !(cur_t>= std_t && cur_t <dlt_t);  
}  
else  
{  
return cur_t >= dlt_t && cur_t <= std_t;  
}  
}  
  
  
bool setTimeZone( TIME_ZONE_INFORMATION const& tzi, bool bDaylightSaving )  
{   
SetTimeZoneInformation(&tzi);  
#ifdef UNDER_CE  
SetDaylightTime(bDaylightSaving?1:0);  
#endif // UNDER_CE  
  
  
RegFlushKey(HKEY_LOCAL_MACHINE);  
return true;  
}  
  
  
core::datetime getLocalDatetime( core::datetime utc0DT,TIME_ZONE_INFORMATION const& tzi )  
{   
//WinCE doesn't support SystemTimeToTzSpecificLocalTime, we have to do it by myself  
//UTC = local time + bias  
long bias = tzi.Bias;  
core::datetime localDatetime(utc0DT);  
if (bias<0)  
{  
localDatetime.addSeconds(-bias*60);  
}  
else  
{  
localDatetime.substractSeconds(bias*60);  
}  
return localDatetime;  
}  
  
  
core::datetime getUTC0Datetime( core::datetime localDT,TIME_ZONE_INFORMATION const& tzi )  
{  
//WinCE doesn't support TzSpecificLocalTimeToSystemTime, we have to do it by myself  
long bias = tzi.Bias;  
core::datetime utc0DT(localDT);  
if (bias<0)  
{  
utc0DT.substractSeconds(-bias*60);  
}  
else  
{  
utc0DT.addSeconds(bias*60);  
}  
return utc0DT;  
}  
  
  
bool strHasStr(core::cstringex& src,core::cstringex& dst)  
{   
return src.find(dst)!=-1;  
}  
  
  
TimezoneInfo const* lookupTimeZone( core::cstringex const& tziUniqueString )  
{  
ASSERT_AND_LOG(gs_TimeZoneList.size()!=0);  
using namespace UtilExt;  
  
  
return findImpl(makeDataSet(gs_TimeZoneList),makeFinder(core::cstringex(tziUniqueString),MMP(&TimezoneInfo::m_displayName),strHasStr));  
}  
}  


  

