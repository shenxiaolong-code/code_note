左移/右移指定位数字符串 (算法http://blog.csdn.net/v_JULY_v/article/details/6322882 的实现)

发布日期: 2013-10-11 09:45:39
原文链接: https://blog.csdn.net/Tonny0832/article/details/12611143

---

以下内容来自于<http://blog.csdn.net/v_JULY_v/article/details/6322882>

[v_JULY_v](http://my.csdn.net/v_JULY_v)博主提供了非常精妙的算法原理，佩服。但是我个人觉得算法的实现稍显麻烦，看那些算法的实现看得很头疼。  


我依据博主的算法，写出了下面的实现。

实现原理：同博主所言，所谓的字符串左移/右移，就是交换字符串的两部分而已。

所以下面的算法实现中，除了最终的一次字符交换外，其它的都是一直在做递归交换字符串的两部分的工作。如果明白了博主的算法原理，下面的算法实现则简洁明了，不必看得头疼。

其实最麻烦的就是在交换不等长的字符串，但是在此题目中，有一个隐含的有利点就是：交换的不等长字符串都是相邻的。(不相邻的字符串都是等长的)

在这点上，把这个工作交给计算机递归去了，设计者不必搞得头疼，只要正确地传入递归的参数就可以了。

template<typename T>  
inline void _swap(T& p1,T& p2)  
{   
if (p1 != p2)  
{  
T tmp=p1;  
p1 = p2;  
p2 = tmp;  
}  
};  
  
  
void swapString(char* str1,char* str2,long iLen)  
{  
if (str1 && str2 && iLen)  
{   
while(iLen--)  
{  
_swap(*str1++,*str2++);  
}  
}   
}  
  
  
void swap2PartOfString(char* pH,char* pM,char* pT) //pHead,pMid,pTail  
{  
//Param check  
if (!pH || !pT || (pH>pM) || (pM>pT))  
{  
return;  
}  
  
int iLen = min(pM-pH,pT-pM+1);  
swapString(pH,pT-iLen+1,iLen);  
  
if (pM-pH != pT-pM+1) //not equal part  
{   
if (pM-pH < pT-pM)  
{  
swap2PartOfString(pH,pM,pT-iLen);  
}  
else  
{   
swap2PartOfString(pH+iLen,pM,pT);  
}  
}  
}  
  
  
void rightRotate(char* pChar,int m)  
{ 

int iLen=strlen(pChar);   
m = (m+iLen)%iLen;   
if (m > iLen/2)   
m = iLen-m; 

printf("Before rightRotate[%d] : %s\n",m,pChar);  
swap2PartOfString(pChar,pChar+strlen(pChar)-m,pChar+strlen(pChar)-1);  
printf("Before rightRotate[%d] : %s\n",m,pChar);  
}  


测试代码：

void main()   
{   
char pStr[] = "1234abcdefgh";  
rightRotate(pStr,4);  
}  


输出：

Before rightRotate[4] : 1234abcdefgh  
Before rightRotate[4] : efgh1234abcd  


在最坏的情况下(右移一位)，其交换字符的次数为strLen-1

在最好的情况下(对半交换)，其交换字符的次数为strLen/2
