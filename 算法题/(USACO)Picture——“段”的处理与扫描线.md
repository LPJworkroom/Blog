题目大意：在一面墙上贴上N张大小不一的矩形海报。矩形间会互相重叠。求最后所有矩形组成的大图形的周长。矩形被覆盖的边，不算组成图形的周长。
N<5000,坐标范围[-10000,10000]。
![](https://upload-images.jianshu.io/upload_images/14546900-f785f96629b16e21.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/14546900-6e4fbf8d9c3b2292.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


与前一道题Window Area何其相似，只是从面积变成周长。当然的，我们可以用类似的方法对付，如离散化。

###墙面的存储
在Window Area中，由于不超过64个矩形，我们用一个128*128的二维数组保存状态。本题N可达5000，开不出这么大的二维数组，自然要换个方法表示墙面。

我们对y坐标离散化。把所有矩形各自的两个y坐标保存到数组yvec中，排序，去重，就得到计算时会用到的所有y坐标。
设想把整个墙面用一根根平行于x轴的线切割，这些线的y坐标正来自于yvec。由于yvec包含了矩形可能出现的所有y坐标，整个大图形必然被切割成一个个矩形“ **段** ”，高是yvec中相邻元素之差。
对每个y坐标，都用一个结构存储各自的“段”，就成功保存了墙面的状态。![](https://upload-images.jianshu.io/upload_images/14546900-dcc63cc2316e0400.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这样的方法是很简陋，但容易想到，简单。

对于所有矩形，我们在它所“跨过”的所有y坐标上插入被切割后的段。
如果段与段之间产生交叉，也要正确的处理。删除合并后的大段所覆盖的所有段，插入大段，这对计算周长来说是必要的。![](https://upload-images.jianshu.io/upload_images/14546900-b593472a1ac65f40.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如果能完成这样的切割，本题就完成了大半。

###“段”的存储
前面提到，在每个y坐标上都要有一个结构存储各自的段，暂且称之为 **ThreadSet**。由于段在各个y坐标上，所以段的两个y坐标是确定的。在ThreadSet中，只需要存储段的两个x坐标。

一种明显的存储方式是使用链表作为ThreadSet。用一个结构体存储一个段的两个x坐标。插入新的段要找到相应的正确位置。由于要频繁插入删除，链表是必须的。链表的形式也十分形象。

然而问题也正出在链表上。试想将很多极短且互不相交的的段，以x坐标第一大，第一小，第二大，第二小……以此类推的顺序插入链表，即每次插入的位置都正是整个链表的中央。为了找到插入的位置，即便是双向链表，每次都必须遍历链表长度的一半。这样速度无法接受。![](https://upload-images.jianshu.io/upload_images/14546900-9c6ec5d3e4b7909a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


ThreadSet对查询，插入，删除的速度都提出要求，这让我最终选择STL的set类，也就是一种二叉排序树。
我 **没有将段的两个x坐标作为一个整体** ，而是将int型的坐标和一个bool型变量IsLeft组成结构体作为set的结点，IsLeft正表示该结点是一个段的左边还是右边。也就是说，一个结点只表示一个坐标。
插入段时，找到这个段所覆盖范围。如果和其它段有交叉，经过计算后得到合并出的大的段的左右坐标，将应删除的段全部删除，并将合并的大段插入，一次插入就完成了。
记新插入的段左右x坐标为l和r，而合并的大段左右坐标为newl和newr。使用set成员函数upper_bound()找到ThreadSet中第一个大于l及第一个大于r的结点，由迭代器Lptr和Rptr指向。
这样，我们几乎得以使两迭代器覆盖所有应删除的段——但仅仅是几乎。
由于upper_bound()仅仅是大于号，还要讨论许多情况。
在讨论中用到了为结点做的标记IsLeft。
###插入情形的讨论
>**对于Lptr**
如果Lptr指向一个左结点（IsLeft为真的结点），需要检查Lptr的前驱是否等于l。如果不等于，Lptr不需要移动。否则，相当于l可以和前一个段连接起来，形成更大的段，而upper_bound()使Lptr跳过了前一个段，所以将Lptr向前移动两次——也就是指向前一个左结点。
如果Lptr指向一个右结点（IsLeft为假的结点），将Lptr向前移动一次，同样的，也就是指向前一个左结点。
**对于Rptr**
对Rptr的讨论简单许多。
如果Rptr指向一个左结点，Rptr向前移动一次，也就是指向前一个右结点。
否则，Rptr不需要移动。

至此，Lptr和Rptr指向了正确的位置。

还需要计算合并的大段的左右坐标。
>newl=min( *Lptr,l);
newr=max( *Rptr,r);

以上式子很容易推出。

接下来，使用set成员函数erase，erase(Lptr,next(Rptr))。使用成员函数insert()，将newl和newr插入。别忘了给IsLeft赋值。

至此，对段的维护完成。

看到这么繁琐的讨论，可能会有人问：
**既然upper_bound()不包含等于号，为什么不用lower_bound()？**
>lower_bound()确实包含等于号，但也包含大于号。所以用lower_bound()查询后必须先讨论：这个结点是大于查询的值还是等于？反而比upper_bound()更繁琐。

###扫描线思想
本题不能把所有矩形的所有段同时插入各自的ThreadSet。
N可达5000，最坏有O(N^2)个段，这至少超过USACO内存16MB的限制。
因此我们需要扫描线的思想。

想象一个扫描线从最低的高度，也就是最小的y坐标开始向上扫描，直到最大的y坐标，高度每次变成当前y坐标的后继。
当扫描到一个高度时，计算当前高度下的段的周长，并计算当前段和前一个扫描的段周长的重叠部分。![](https://upload-images.jianshu.io/upload_images/14546900-4551b10c7d374062.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

由于一个高度的段的重叠部分仅仅和它上下相邻两个高度的段有关，所以扫描不会漏算错算重叠的周长。![](https://upload-images.jianshu.io/upload_images/14546900-f19aa4cbd58ffed1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以看出，任意时刻只需要维护两个ThreadSet，分别表示前一个扫描的段和当前扫描的段。
当扫描下一个段，当前段变成前一个扫描的段，新的ThreadSet成为当前段。
这很像滚动数组。

这样，空间的问题得以解决。
###扫描线的实现
我们称前一个扫描的段为up，当前扫描的为down。
up和down初始为空。
以递增顺序，对每一个y坐标循环：
将down复制给up；将down清空；
遍历所有矩形：
如果矩形较小的y坐标等于当前扫描的y坐标，即扫描到了，用前面的插入方法，将矩形两x坐标作为新的段插入；然后，因为这个高度下的段已经计算过，将矩形较小的y坐标赋值为它的后继——这如同是把矩形最底部分“切下来”；
计算down所有段的周长；
计算down与up的重叠部分。
![“切下来”](https://upload-images.jianshu.io/upload_images/14546900-8444d9366e325aab.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

计算周长和重叠部分的方法很容易想到，不再赘述。
up和down初始为空，因此边界时的计算也正确。
至此，大功告成。

USACO最坏例子用时0.5s，可见此解法仍会被特定数据针对。
附上代码。
```
/*ID: lpjwork1
TASK: picture
LANG: C++11
*/
#include<iostream>
#include<stdio.h>
#include<map>
#include<list>
#include<string>
#include<string.h>
#include<vector>
#include<queue>
#include<algorithm>
#include<set>
#include<fstream>
using namespace std;
const int MAXN=5000+10;
/*save all rectangles*/
struct rect{
	int x1,y1,x2,y2;
}rectvec[MAXN];
/*node in Threadset*/
struct node{
	int val;
	bool isl;	//true if left,else right
};
/*STL set need operator<*/
bool operator<(const node a,const node b)
{
	return a.val<b.val;
}

int N;
/*scan line sets*/
set<node> up,down;
/*save all y coordinates*/
vector<int> yvec;
/*trans y coord to yvec index*/
map<int,int> yindex;

/*get overlap between up and down*/
int overlap(set<node>,set<node>);
/*insert thread to ThreadSet*/
void insert_cover(set<node>&,node&,node&);
/*get one ThreadSet perimetor*/
int perim(set<node>&,int);

ofstream fout ("picture.out");
ifstream fin ("picture.in");
int main() {
	fin>>N;
	for (int i=0;i<N;i++)
	{
		int x1,y1,x2,y2;fin>>x1>>y1>>x2>>y2;
		rectvec[i]=rect{x1,y1,x2,y2};
		yvec.push_back(y1);yvec.push_back(y2);
	}
	/*sort and unique*/
	sort(yvec.begin(),yvec.end());
	yvec.resize(unique(yvec.begin(),yvec.end())-yvec.begin());
	for (int i=0;i<yvec.size();i++)	yindex[yvec[i]]=i;
	
	int ans=0;
	
	for (int i=0;i<yvec.size();i++)
	{
		up=down;down.clear();
		for (int j=0;j<N;j++)
		{
			int x1=rectvec[j].x1,y1=rectvec[j].y1,
			x2=rectvec[j].x2,y2=rectvec[j].y2;
			/*not yet touch,or the rectangle all scanned*/
			if (y1!=yvec[i]||y1==y2)	continue;
			node tmpl{x1,true},tmpr{x2,false};
			insert_cover(down,tmpl,tmpr);
			/*cut thread down*/
			rectvec[j].y1=yvec[i+1];
		}
		ans+=perim(down,i);
		ans-=overlap(up,down);
	}
	
	fout<<ans<<endl;
    fout.close();fin.close();
    return 0;
}

/*insert one node,combine overlap threads*/
void insert_cover(set<node>& lst,node& l,node& r)
{
	if (lst.empty()){
		lst.insert(l);lst.insert(r);return;
	}
	
	set<node>::iterator lptr=lst.upper_bound(l),rptr=lst.upper_bound(r);
	/*if l.val equals last element int lst*/ 
	if (lptr==lst.end()&&prev(lptr)->val==l.val)	--lptr;
	
	if (lptr==lst.end()||rptr==lst.begin()){
		lst.insert(l);lst.insert(r);
		return;
	}
	
	int newl,newr;
	if (lptr->isl==true){
		/*connect with prev thread*/
		if (lptr!=lst.begin()&&lptr->val==prev(lptr)->val)	--lptr,--lptr,newl=lptr->val;
		else	newl=l.val;
	}
	else					--lptr,newl=lptr->val;
	
	if (rptr==lst.end())	newr=r.val;
	else{
		if (rptr->isl==true)	newr=r.val;
		else					newr=rptr->val,rptr=next(rptr);
	}
	lst.erase(lptr,rptr);
	lst.insert(node{newl,true});
	lst.insert(node{newr,false});
}

int overlap(set<node> up,set<node> down)
{
	if (up.size()==0||down.size()==0)	return 0;
	int ret=0;
	set<node>::iterator upl=up.begin(),downl=down.begin(),upr,downr;
	while (upl!=up.end()&&downl!=down.end())
	{
		upr=next(upl);downr=next(downl);
		int maxl=max(upl->val,downl->val),minr=min(upr->val,downr->val);
		if (maxl<minr){
			ret+=2*(minr-maxl);
			if (upl->val>downl->val&&upr->val>downr->val){
				node tmp{downr->val,true};
				up.insert(tmp);
				up.erase(upl);
				upl=up.find(tmp);
			}
			if (downl->val>upl->val&&downr->val>upr->val){
				node tmp{upr->val,true};
				down.insert(tmp);
				down.erase(downl);
				downl=down.find(tmp);
			}
		}
		if (upr->val<downr->val)	upl++,upl++;
		else						downl++,downl++;
	}
	return ret;
}

int perim(set<node>& todo,int yind)
{
	int ret=0;
	for (set<node>::iterator it=todo.begin();it!=todo.end();++it,++it)
	{
		ret+=(yvec[yind+1]-yvec[yind])*2;	//height of rect
		ret+=((next(it)->val)-(it->val))*2;
	}
	return ret;
}
```
2019.3.11





