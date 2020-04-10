1.Fence loop
简介：Farmer John有一片柱子，柱子间有栅栏连接。注意，一个柱子可以和多个栅栏连接。给出每根栅栏两边柱子的编号以及栅栏长度，问：栅栏围起来的封闭图形中，拥有最小周长的长多少？
![一个示例，数字是栅栏的编号](https://upload-images.jianshu.io/upload_images/14546900-0d2fb9a1417e8700.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
思路：
将问题转化为图：柱子为点，栅栏为边。但是由于只给出的是相邻栅栏的编号而不是柱子的编号，问题稍稍复杂了点。人为的为每个栅栏两边的柱子都分配编号，视为每个栅栏都有自己的两个柱子（将柱子拆开）。栅栏相连，就是对应的柱子间连上权为0的无向边。再加上栅栏自己的两个点连一条权为长度的无向边，问题就是图中最小的环。
求最小环：枚举每个栅栏，先删掉这根栅栏，再求栅栏一侧至另一侧的最短路。
建图的过程说起来简单，其实很要求编程底力。求最小环反而没有花太长时间。

2.Shuttle puzzle
一条线有2*N+1个空格，左边N个是白，右边N个是黑，中间一个空着。每次你可以把与空格相邻的棋子移到空格上，或让一个与空格距离为2的棋子“跳过”一个异色棋子跳到空格上，就像跳棋那样。问最少多少步让左边N个全是黑，右边N个全是白？要求打印每步移第几个棋子，而且答案方案的字典序最小。
思路：
本来毫无思路，看了样例后观察到了答案的模式：
**1.如果有棋子可以跳过异色棋子到空格上，跳它。**
**2.如果不能进行1，如果空格旁的棋子移到空格上，可以与自己“前方”第一个的同色棋子的距离从3变为2，即中间只有一个棋子，移动它，仿佛是为异色棋子创造“连跳”的机会。（什么是“前方”？对白色来说就是右边，黑色就是左边）**
**3.如果2也不能进行，随意移动一个棋子。**
注意，由于答案要求方案的字典序最小，就要尽量选“左边”的棋子移动。

3.Pollutant control
一家工厂会把产品运到不同的仓库里，再从不同仓库里运到商店。不同仓库与仓库（工厂与商店亦可视为仓库）之间有一名固定的司机来往运送，并且这个司机只运这一条线路。现在工厂发现产品有问题，要拦下一些司机来防止产品被运到商店，拦下不同司机有不同的花费，问最小花费是多少?要求拦下司机数量最小，并且拦下司机的编号组成的方案字典序最小。
思路：
抽象成图，仓库为点，工厂和商店可视为源点和终点，一个司机只运一条线，即在运的两点间连权为花费的边。删掉边使源点无法到终点，还要求费用最小，那么就是最小割问题了。**逆向的把拦下司机的花费看作点间的容量，那么拦下的总花费就是最大流，拦截方案就是最小割。**通常的最小割把边按容量降序排序，这里则不同。由于点之间可能不止一条边，所以将重边缩为一条，容量为总和，并标记这条新边实际上是几条边。对方案的限制产生了边的排序的要求。在一条边容量最大的条件下，司机更少，即这条边实际上的边数更小；字典序，则是要求边的实际的所有边的编号中最小的那个，更小。
```
//compare func to sort edges
bool cmp(int a,int b)
{
        /*bigger capacity*/
	if (edge[a].cap!=edge[b].cap)	return edge[a].cap>edge[b].cap;
        /*fewer drivers*/
	if (edge[a].num!=edge[b].num)	return edge[a].num<edge[b].num;
        /*smaller smallest index*/
	return edge[a].ind<edge[b].ind;
}
```
最后是三题的代码，写的比较杂。
Fence loop：
```
/*ID: lpjwork1
TASK: fence6
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
const int MAXN=100,MAXNO=200,INF=(1<<20);
struct Fence{
	int len;
	int no1,no2;
	bool nei1[MAXN+1],nei2[MAXN+1];
}fence[MAXN+1];
queue<int> q;
int N,tot=0,ans=INF;
int dist[MAXNO+1][MAXNO+1],g[MAXNO+1][MAXNO+1];
bool inq[MAXNO+1];
void input();
void spfa(int);
ofstream fout ("fence6.out");
ifstream fin ("fence6.in");
int main() {
	for (int i=0;i<=MAXNO;i++)
		for (int j=0;j<=MAXNO;j++)	g[i][j]=-1;
	input();
	for (int i=1;i<=N;i++)
	{
		int no1=fence[i].no1,no2=fence[i].no2;
		g[no1][no2]=g[no2][no1]=INF;
		spfa(no1);
		ans=min(ans,dist[no1][no2]+fence[i].len);
		g[no1][no2]=g[no2][no1]=fence[i].len;
	}
	fout<<ans<<endl;
    fout.close();fin.close();
    return 0;
}

void spfa(int sor)
{
	while (!q.empty())	q.pop();
	memset(inq,false,sizeof(inq));
	for (int i=0;i<=tot;i++)	dist[sor][i]=INF;
	dist[sor][sor]=0;inq[sor]=true;
	q.push(sor);
	while (!q.empty())
	{
		int now=q.front();q.pop();
		for (int i=1;i<=tot;i++)
			if (g[now][i]!=-1&&g[now][i]+dist[sor][now]<dist[sor][i]){
				dist[sor][i]=g[now][i]+dist[sor][now];
				if (!inq[i]){
					q.push(i);inq[i]=true;
				}
			}
		inq[now]=false;
	}
}

void input()
{
	fin>>N;
	for (int i=1;i<=N;i++)
	{
		int index;
		fin>>index;
		Fence& now=fence[index];
		memset(now.nei1,false,sizeof(now.nei1));
		memset(now.nei2,false,sizeof(now.nei2));
		int len;fin>>len;now.len=len;
		now.no1=++tot;now.no2=++tot;
		g[now.no1][now.no2]=g[now.no2][now.no1]=len;
		int no1size,no2size;fin>>no1size>>no2size;
		for (int j=0;j<no1size;j++){
			int temp;fin>>temp;now.nei1[temp]=true;
		}
		for (int j=0;j<no2size;j++){
			int temp;fin>>temp;now.nei2[temp]=true;
		}
	}
	for (int i=1;i<=N;i++)
	{
		for (int j=1;j<=N;j++)
		{
			if (i==j)	continue;
			Fence& fi=fence[i],&fj=fence[j];
			if (fi.nei1[j]&&fj.nei1[i])
				g[fi.no1][fj.no1]=g[fj.no1][fi.no1]=0;
			if (fi.nei2[j]&&fj.nei1[i])
				g[fi.no2][fj.no1]=g[fj.no1][fi.no2]=0;
			if (fi.nei1[j]&&fj.nei2[i])
				g[fi.no1][fj.no2]=g[fj.no2][fi.no1]=0;
			if (fi.nei2[j]&&fj.nei2[i])
				g[fi.no2][fj.no2]=g[fj.no2][fi.no2]=0;
		}
	}
}
```
Shuttle game：
```
/*ID: lpjwork1
TASK: shuttle
LANG: C++11
*/
#include<iostream>
#include<stdio.h>
#include<string>
#include<string.h>
#include<vector>
#include<fstream>
using namespace std;
const int MAXN=30;
enum color{nil,w,b};
color board[MAXN];
int len,ans=0,gap,N;
vector<int> op;
void shuttle(int);
void swap(int,int);
bool onboard(int);
bool final();
void show();
ofstream fout ("shuttle.out");
ifstream fin ("shuttle.in");
int main() {
	fin>>N;
	len=N*2+1;gap=N;
	board[N]=nil;
	for (int i=0;i<N;i++)
	{
		board[i]=w;board[i+N+1]=b;
	}
	shuttle(N);
    fout.close();fin.close();
    return 0;
}


void shuttle(int N)
{
	while (1)
	{
		//show();
		if (final())	break;
		ans++;
	/*check if can jump*/
		/*check w jump*/
		if (onboard(gap-2)&&board[gap-2]==w&&board[gap-1]==b){
			swap(gap,gap-2);continue;
		}
		/*check b jump*/
		if (onboard(gap+2)&&board[gap+2]==b&&board[gap+1]==w){
			swap(gap,gap+2);continue;
		}
	/*check if can make gap*/
		if (onboard(gap+2)&&onboard(gap-1)&&
			board[gap+2]==w&&board[gap-1]==w){
				swap(gap,gap-1);continue;
			}
			
		if (onboard(gap-2)&&onboard(gap+1)&&
			board[gap-2]==b&&board[gap+1]==b){
				swap(gap,gap+1);continue;
			}
	/*if nothing,move*/
		if (onboard(gap-1)&&board[gap-1]==w)	swap(gap,gap-1);
		else				swap(gap,gap+1);
	}
	for (int i=0;i<op.size();i++)
	{
		if ((i)%20==0){
			fout<<op[i];
		}
		else	fout<<' '<<op[i];
		if ((i+1)%20==0)	fout<<endl;
	}
	if ((op.size())%20!=0)	fout<<endl;
}

void swap(int g,int b)
{
	gap=b;op.push_back(b+1);
	board[g]=board[b];
	board[b]=nil;
}

bool onboard(int pos)
{
	return (pos>=0&&pos<len);
}

bool final()
{
	for (int i=0;i<N;i++)	if (board[i]!=b)	return false;
	for (int i=N+1;i<len;i++)	if (board[i]!=w)	return false;
	return true;
}

void show()
{
	for (int i=0;i<len;i++)
		if (board[i]==w)	cout<<'w';
		else if (board[i]==b)	cout<<'b';
		else if (board[i]==nil)	cout<<' ';
		else	cout<<'?';
	cout<<endl;
}
```
Pollutant control：
```
/*ID: lpjwork1
TASK: milk6
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
const int MAXN=32,MAXM=1000,INF=(1<<30);
struct Edge{
	int u,v,cap,ind;
	int num;
	int nxt;
} edge[MAXM*2+10];
vector<int> sortedge,ans;
int N,M,sor=1,tar,tot=1,flow=0;
int head[MAXN+10],fa[MAXN+10],dist[MAXN+10];;
Edge G[MAXM*2+10];
vector<vector<vector<int> > > data;
bool inq[MAXN+10];
queue<int> q;

void input();
void addedge(int,int,int,int,int);
void spfa();
int backtrack();
int maxflow();

bool cmp(int a,int b)
{
	if (edge[a].cap!=edge[b].cap)	return edge[a].cap>edge[b].cap;
	if (edge[a].num!=edge[b].num)	return edge[a].num<edge[b].num;
	return edge[a].ind<edge[b].ind;
}

ofstream fout ("milk6.out");
ifstream fin ("milk6.in");
int main() {
	fin>>N>>M;tar=N;
	data=vector<vector<vector<int> > >(N+1,vector<vector<int> >(N+1,vector<int>()));
	for (int i=0;i<=N;i++)	head[i]=0;
	input();
	sort(sortedge.begin(),sortedge.end(),cmp);
	for (int i=0;i<=tot;i++)	G[i]=edge[i];
	flow=maxflow();
	cout<<flow<<endl;
	fout<<flow<<' ';
	for (int i=0;i<M&&flow;i++)
	{
		int nowedge=sortedge[i],tempcap=edge[nowedge].cap,u=edge[nowedge].u,v=edge[nowedge].v;
		edge[nowedge].cap=0;
		int newflow=maxflow();
		if (newflow+tempcap==flow){
			for (int i=0;i<data[u][v].size();i++)
				ans.push_back(data[u][v][i]);
			flow=newflow;
		}
		else	edge[nowedge].cap=tempcap;
	}
	
	fout<<ans.size()<<endl;
	if (ans.size()>0){
		sort(ans.begin(),ans.end());
		for (int i=0;i<ans.size();i++)	fout<<ans[i]<<endl;
	}
    fout.close();fin.close();
    return 0;
}


/*maxflow*/


int maxflow()
{
	int flow=0;
	for (int i=0;i<=tot;i++)
	{
		G[i].cap=edge[i].cap;
		G[i].nxt=edge[i].nxt;
	}
		
	while (1)
	{
		spfa();
		if (dist[tar]==INF)	break;
		flow+=backtrack();
	}
	return flow;
}

int backtrack()
{
	int minflow=INF;
	for (int i=fa[tar];i;i=fa[G[i].u])
		minflow=min(minflow,G[i].cap);
	for (int i=fa[tar];i;i=fa[G[i].u])
	{
		G[i].cap-=minflow;
		G[i^1].cap+=minflow;
	}
	return minflow;
}

void spfa()
{
	while (!q.empty())	q.pop();
	for (int i=0;i<=N;i++)
	{
		dist[i]=INF;
		inq[i]=false;
	}
	fa[sor]=0;inq[sor]=true;dist[sor]=0;
	q.push(sor);
	int v,cap,now;
	while (!q.empty())
	{
		now=q.front();q.pop();
		
		for (int i=head[now];i;i=G[i].nxt)
		{
			v=G[i].v;cap=G[i].cap;
			if (now==tar)	return;
			if (dist[v]<=dist[now]+1||cap==0)	continue;
			dist[v]=dist[now]+1;
			fa[v]=i;
			if (!inq[v]){
				inq[v]=true;q.push(v);
			}
		}
		inq[now]=false;
	}
}

void input()
{
	int sum[N+1][N+1];
	for (int i=0;i<=N;i++)
		for (int j=0;j<=N;j++)	sum[i][j]=0;
	for (int i=1;i<=M;i++)
	{
		int u,v,cost;fin>>u>>v>>cost;
		sum[u][v]+=cost;data[u][v].push_back(i);
	}
	M=0;
	for (int i=1;i<=N;i++)
		for (int j=1;j<=N;j++)
			if (sum[i][j]!=0){
				addedge(i,j,sum[i][j],data[i][j][0],data[i][j].size());
				M++;
			}
}

void addedge(int u,int v,int cost,int ind,int num)
{
	edge[++tot]=Edge{u,v,cost,ind,num,head[u]};
	head[u]=tot;sortedge.push_back(tot);
	edge[++tot]=Edge{v,u,0,ind+1000,1,head[v]};
	head[v]=tot;
}
```

 -2018.12.3
