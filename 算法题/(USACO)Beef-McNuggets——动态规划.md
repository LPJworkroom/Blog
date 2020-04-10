题目来源：USACO section4.1
题目大意：给出少于10个数字，这些数字最大不超过256。每个数字可以用任意次并求和，问不能组成的**最大**的数是多少。如果没有不能组成的数，要输出0。题目保证如果存在不能组成的数，这个数最大不超过2,000,000,000。

如果不考虑最大20亿的限制，问题就是简单经典的DP题。从1到20亿遍历，如果当前的数可以组成，就遍历给出的10个数，加上它，它们的和也是可以组成的。
就像这样的伪代码。

```
MAXN=2000000000;
/*whether a number can be made up*/
bool CanMakeUp[MAXN];
let all CanMakeUp=false;
/*boundary of dynamic programming*/
CanMakeUp[0]=true;
for (num=0->MAXN)
{
	if (CanMakeUp[num])
		/*new number is given number.
		 *add it with num,so you can 
		 *make up a new number.
		 */
		for (every new number to add up)
			CanMakeUp[num+new number]=true;
}
return the max number you can't make;
```

但20亿的空间无法承受。
为了解决空间限制，一种方案是用滚动数组。由于每个数字最大不超过256，所以DP时一个数字的状态最多会由前256个数字的状态决定。所以，我们可以只用一个256*2大小的数组。用一样的DP方法，前256个数字决定了后256个数字是否能组成。遍历完前256个数，就用后256个数覆盖掉前256个，同时记录最大的不能组成的数，进入下一个循环。

```  
MAXN=256;
/*whether a number can be made up*/
bool CanMakeUp[MAXN*2];
let all CanMakeUp=false;
/*boundary of dynamic programming*/
CanMakeUp[0]=true;
/*answer to return*/
ans=0;
for (turn:from 0 to 2000000000/MAXN)
{
	for (num:every number in first half of CanMakeUp)
		if (CanMakeUp[num])
			for (every new number to add up)
				CanMakeUp[num+new number]=true;
	ans=max number you can not make in CanMakeUp+turn*256;			
	move the second half to first half;
}
return ans;
```

空间问题的确解决了，现在的问题就是时间。循环20亿次是肯定不行的。
为了解决这个问题，我重新思考自己的方法。
每次循环，我保留了一个**可以组成的数的集合**，也就是CanMakeUp。我把对这个集合的操作定义为：循环中用集合里的数尝试更新新的数。每一次操作后，得到下一次循环的集合。
终于我发现了一个重要的规律：

**如果一次操作之后，集合里的数没有变少，那么不存在不能组成的最大的数。**

>每一次操作都一样。如果第一次操作后集合里的数不变少，第二次操作时，不仅能保留第一次操作之前的数，还会利用第一次操作新形成的数再尝试组成新的数，这个尝试只可能让集合里的数更多。所以，集合里的数永远不会变少。

因此，每一次操作后集合里的数不会变少，这个过程可以一直循环到无穷，也就不存在不能组成的最大的数了。

由此我们可以推导出更重要的规律：

**如果一次操作之后数会变少，那么最多256次操作后集合里就没有任何数。**

>如果操作后数字不变少，已经证明没有答案。否则，数字必须变少。如果第一次操作后集合里的数字变少，第二次操作时，原本可以用少掉的数组成的新数，现在也不能组成了。所以，集合里的数字只会更少。每次都变少，那么至少要减少1个数。所以最多256次操作就会让集合为空——也就是找到答案。

由此证明，最多循环256*256次就可以得到答案。20亿的范围，完全是吓唬人。

所以，在滚动数组的基础上，记录每次操作后集合内数的个数。如果没变少，直接输出答案0；否则一直循环下去——我们已经证明了最多迭代256次就能找到答案。

```
MAXN=256;
/*whether a number can be made up*/
bool CanMakeUp[MAXN*2];
let all CanMakeUp=false;
/*boundary of dynamic programming*/
CanMakeUp[0]=true;
/*answer to return*/
ans=0;
/*number in CanMakeUp*/
count=default number in CanMakeUp;
/*at most MAXN turns,and can get answer*/
for (turn:from 0 to MAXN)
{
	for (num:every number in first half of CanMakeUp)
		if (CanMakeUp[num])
			for (every new number to add up)
				CanMakeUp[num+new number]=true;
	ans=max number you can not make in CanMakeUp+turn*256;
	move the second half to first half;
	update count of numbers in CanMakeUp;
		if (count doesn't decrease)
			/*no answer*/
			return 0;
		/*else count decreased*/ 
}
return ans;
```
附上代码。
```
/*ID: lpjwork1
TASK: nuggets
LANG: C++11
*/
#include<iostream>
#include<stdio.h>
#include<map>
#include<string>
#include<string.h>
#include<vector>
#include<queue>
#include<algorithm>
#include<set>
#include<fstream>
using namespace std;
/*base equals turn*256*/
int base=0,N,ans;
/*bulast is bool of first half of CanMakeUp,back is second half*/ 
bool bulast[260],back[260];
queue<int> last;
/*pack is given numbers*/
vector<int> pack;
ofstream fout ("nuggets.out");
ifstream fin ("nuggets.in");
int main() {
	fin>>N;
	for (int i=0;i<260;i++)	bulast[i]=true;
	for (int i=0;i<N;i++)
	{
		int temp;fin>>temp;
		pack.push_back(temp);
	}
	sort(pack.begin(),pack.end());
	for (int i=pack.size()-1;i>=1;i--)
		for (int j=i-1;j>=0;j--)	
			if (pack[i]%pack[j]==0){
				pack.erase(pack.begin()+i);break;
			}
	int maxnum=pack[pack.size()-1],minnum=pack[0];
	bulast[0]=false;
	for (int i=0;i<pack.size();i++)
		for (int j=0;j<maxnum;j++)
			if (bulast[j]==false&&j+pack[i]<=maxnum)	bulast[j+pack[i]]=false;
	for (int i=1;i<maxnum;i++)	
		if (bulast[i]){
			last.push(i);
			ans=i;
		}
	int befsize=last.size();
	for (int i=0;i<260;i++)	back[i]=bulast[i];
	if (minnum!=1&&pack.size()!=1)
	while (1)
	{
		for (int k=0,m=last.size();k<m;k++)
		{
			int now=last.front();last.pop();
			for (int i=0;i<pack.size();i++)
			{
				if (now-pack[i]>0&&back[now-pack[i]]==false){
					back[now]=false;break;
				}
				if (now-pack[i]<=0&&bulast[now-pack[i]+maxnum]==false){
					back[now]=false;break;
				}
			}
			if (back[now]==true){
				ans=now;
				last.push(now);	
			}
		}
		if (last.size()==0)	break;
		if (last.size()==befsize){
			ans=base=0;break;
		}
		befsize=last.size();
		base+=maxnum;
		for (int i=0;i<260;i++)	bulast[i]=back[i];
	}
	else	ans=0;
    fout.close();fin.close();
    return 0;
}
```

2018.12.20
