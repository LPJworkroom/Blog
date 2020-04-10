题目大意：一个R行C列的大矩形，里面有P个被称为“坏点”的点。求大矩形中不包含任何“坏点”的矩形，面积最大是多少。
R<=3000,C<=3000,P<=30000。
![](https://upload-images.jianshu.io/upload_images/14546900-12336d9339ac4158.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![一个最大矩形的例子](https://upload-images.jianshu.io/upload_images/14546900-cc6dd13c0e8a6b54.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
##扫描线视角
只用正常的方法是无法解决本题的。我们需要扫描线的思想。
想象一根扫描线，从左向右扫描。以这样的方式看，任意一个矩形都可以看作由它最左边的边，也就是它的高，一直向右扩展形成。
那么一直扩展到什么时候呢？直到碰到坏点或大矩形的边界才无法继续扩展。
![](https://upload-images.jianshu.io/upload_images/14546900-b6c351a76f392e19.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/14546900-dfe55df5e1fb8175.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/14546900-61412fdaab6e131d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
可是，从左边开始扫描，我们无从知道这个矩形的高应该是多少，我们并不是一开始就能知道一个矩形的高是哪一段。

再仔细观察可以发现：任意矩形的高是从整个大矩形的高，也就是最大的高变化而来的。
用最大的高从左向右扩展，由于碰到了坏点，不能继续向右扩展。但如果把最大的高用坏点切成不包含坏点的上下两段，由于这两段没有碰到坏点，就可以继承已扩展的长度，继续向右扩展，直到又碰到坏点。
![](https://upload-images.jianshu.io/upload_images/14546900-eef7cd2ad4c19d54.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/14546900-2fb83f3e41b0b165.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/14546900-1972920085224aec.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/14546900-14dcbf5102a08b29.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
示例的矩形的高，就是最大的高被最左边两个坏点切割而成的。如果没有这两个坏点，矩形的高就可以以更长的姿态向右扩展。
由于所有的矩形都可以看作通过以上的方式形成，经过扫描后就可以得到最大面积。
##算法描述
我们从最左边的坏点开始扫描到最右边的坏点，期间维护能够向右扩展的线段的集合，以及这些线段已经扩展的距离。
由于一个线段只有在碰到坏点或碰到大矩形的边界时才达到最大，所以当扫描到坏点时，我们需要检查所有无法继续向右扩展的，也就是碰到坏点的线段。我们需要检查这些线段已扩展的面积：也就是线段的长度乘以它已扩展的距离。
检查过面积后，由于已经无法继续扩展，这个线段需要从集合中删除。但是这个线段被坏点切割的两段仍可以继续扩展，所以这两个小段需要加入集合，并继承原线段的距离。

比较特殊的一个段是最大的高。虽然它从最左边开始扩展，但每当碰到坏点后，它可以重新出现在坏点之后——只不过已扩展的距离要从零开始。
形象的看，在碰到坏点后，我们可以从坏点之后开始，重新用最大的高向右扩展。
也就是说，最大的高一定会碰到每一个坏点，但在碰到坏点后它不用被从集合中删除：只需要把它的已扩展距离改为零，就可以继续扫描。
##集合的数据结构
算法的重点在于存放所有线段的集合的结构。为了找到所有碰到坏点的线段，给定一个点，我们需要知道所有经过这个点的线段。由于同一个点可以同时被大量的线段，甚至集合中所有线段经过，我们需要快速的遍历所有这些线段，所以遍历速度较低的树形结构被排除。同时我们需要不停的插入删除线段，所以最终我确定集合的数据结构为链表。
光一个链表还不够。在插入新的段时，必须检查这个段是否已经出现过，否则直接插入链表会导致线段数过多。这一步查找交给STL的set，一种排序二叉树处理。
所以，我用了一个链表用来遍历所有线段，一个set查找已出现线段。

最后，由于输入数据量巨大，用fscanf会比fstream快。
USACO最后一个例子1.8 sec，压着2 sec的线通过。
附上代码。

```
/*ID: lpjwork1
TASK: rectbarn
LANG: C++11
*/
#include<iostream>
#include<stdio.h>
#include<map>
#include<string>
#include<vector>
#include<string.h>
#include<algorithm>
#include<list>
#include<fstream>
using namespace std;
const int MAXP=30000+10;
/*srad==thread*/
struct srad{
	int u,d;
};
struct point{
	int x,y;
}badp[MAXP];

bool operator<(srad a,srad b)	
{
	if (a.u!=b.u)	return a.u<b.u;
	return a.d<b.d;
}
bool operator<(point a,point b)	{
	if (a.x!=b.x)	return a.x<b.x;
	return a.y<b.y;
}
list<srad> threads;
map<srad,int> life;
vector<int> xvec;
int R,C,P;
int tot=0;

void tryinsert(srad tmpnew,int lenth);
void check_cut(list<srad>::iterator&,int&,point&);
bool srad_contain(srad&,int);

ofstream fout ("rectbarn.out");
int main() {
	FILE* fin=fopen("rectbarn.in","r");
	fscanf(fin,"%d%d%d",&R,&C,&P);
	for (int i=0;i<P;i++)
	{
		fscanf(fin,"%d%d",&badp[i].y,&badp[i].x);
		xvec.push_back(badp[i].x);
	}
	
	sort(badp,badp+P);
	sort(xvec.begin(),xvec.end());
	xvec.resize(unique(xvec.begin(),xvec.end())-xvec.begin());
	
	threads.push_back(srad{1,R});
	life[srad{1,R}]=0;
	
	int pbad=0,ans=0;
	for (int i=0;i<xvec.size();i++)
	{
		tot=xvec[i]-1;
		/*for all bad point at this x coord*/
		for (;pbad<P&&badp[pbad].x==xvec[i];pbad++)
		{
			point bad=badp[pbad];
			for (auto it=threads.begin();it!=threads.end();)
			{
				if (srad_contain(*it,bad.y))	check_cut(it,ans,bad);
				else	it=next(it);
			}
		}
		life[srad{1,R}]=-xvec[i];
		threads.push_front(srad{1,R});
	}
	tot=C;
	for (auto it=threads.begin();it!=threads.end();it++)
	{
		srad now=*it;int lenth=life[now];
		ans=max(ans,(now.d-now.u+1)*(lenth+tot));
	}
	fout<<ans<<endl;
    fout.close();fclose(fin);
    return 0;
}

bool srad_contain(srad& a,int b)
{
	return a.u<=b&&a.d>=b;
}

void check_cut(list<srad>::iterator& it,int& ans,point& bad)
{
	srad now=*it;int lenth=life[now];
	ans=max(ans,(now.d-now.u+1)*(lenth+tot));
	if (bad.y!=now.u)	tryinsert(srad{now.u,bad.y-1},lenth);
	if (bad.y!=now.d)	tryinsert(srad{bad.y+1,now.d},lenth);
	life.erase(now);
	it=threads.erase(it);
}

void tryinsert(srad tmpnew,int lenth)
{
	/*new thread*/
	if (life.find(tmpnew)==life.end()){
		threads.push_back(tmpnew);
		life[tmpnew]=lenth;
	}
	else	
		if (life[tmpnew]<lenth)	life[tmpnew]=lenth;
}
```
