反转单向链表的实现

发布日期: 2013-10-14 09:36:14
原文链接: https://blog.csdn.net/Tonny0832/article/details/12704599

---

单向链接的反转往往会拿来考验人的基本逻辑，因为其中几个变量的初始化和更新稍微考虑疏漏一点儿就出错。

下面的算法提供了一个简洁的实现，不用再考虑那几个变量的初始化及更新了。

//反转单向链表的实现。

template<typename T>  
T* reverseSingleList(T* pHead, T* T::*pNext,T*& pNewHead)  
{   
if (!pHead || !(pHead->*pNext))  
{  
pNewHead = pHead;  
}  
else  
{  
reverseSingleList(pHead->*pNext,pNext,pNewHead)->*pNext = pHead; //reverse single list  
pHead->*pNext = NULL;  
}  
return pHead;  
}  


  


//测试代码：

struct SList  
{  
int m_data;  
SList* pNext;  
};  
  
  
void printList(SList* pHead)  
{  
while(pHead)  
{  
printf("[%d] ",pHead->m_data);  
pHead = pHead->pNext;  
}  
}  
  
  
void testreverseSingleList()   
{   
SList demoObj[10];  
demoObj[0].m_data = 0;  
for (int i=1;i<10;i++)  
{  
demoObj[i].m_data = i;  
demoObj[i-1].pNext = &demoObj[i];  
}  
demoObj[9].pNext = NULL;  
  
  
SList* pSList = demoObj;  
printList(pSList);  
printf("\n");  
reverseSingleList(pSList,&SList::pNext,pSList);  
printList(pSList);  
printf("\n");  
}  


输出：

[0] [1] [2] [3] [4] [5] [6] [7] [8] [9]  
[9] [8] [7] [6] [5] [4] [3] [2] [1] [0]  


  


  

