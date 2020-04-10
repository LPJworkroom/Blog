题目来源：USACO section 3.3
题目大意：一个王和他的骑士们要聚会。现有一张R*C的棋盘（R<=26，C<=30），王一步可以走到相邻八格任意一格，骑士走“日”字。这个棋盘的每一格可以放任意多棋子（也就是：棋子可以在同一格不影响）。棋盘上有一个王和N个骑士（0<=N，最多可以铺满棋盘）。你的任务是用最小的总步数让所有骑士和王聚在任意一个格子里。棋子们不用同时行动，你可以任意决定哪个棋子什么时候走，走几步。另外：当王和骑士在同一格时，你可以选择让一个骑士“带着”王走，两者视为一个骑士行动（当然，这样只算一步）。

![](http://upload-images.jianshu.io/upload_images/14546900-8803386e25d88c16?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![国王的步伐](http://upload-images.jianshu.io/upload_images/14546900-9a7fa0e5eec46243?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![骑士的步伐](http://upload-images.jianshu.io/upload_images/14546900-b8cb413f7c5fe0dd?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)/




**如果没有王……**         
那么可以BFS求出每个骑士到每个格子的最小距离，每格保留所有骑士到这格的最小距离之和。答案就是所有格子中和最小的。很轻松。

![](http://upload-images.jianshu.io/upload_images/14546900-42acfc924b28e906?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 如果没有骑士“接上”王的动作，聚集在一点的距离只比之前的方案多了一项王到该格的距离。
/
     现在考虑让一个骑士去接王。与其说要去“接"，不如说当骑士在某格时，让王走到这格，之后一起行动。之前每格的骑士步数和就需要加上骑士去接王的额外步数和王走到骑士所在格的步数。那么在哪格让王“上马”呢？又在哪格聚集呢？哪个骑士去接？都需要枚举。 很自然的，一个朴素的枚举方案出现了。我们枚举每一个聚集点，枚举每一个可能去接王的骑士，再枚举每一个可能的“上马”点，取最小距离。就像这样的伪代码：

    

```
for (every grid to gather)
    for(every knight to pick up king)
        for (every gird to wait for king)
        {
            /*no body pick up king*/
            ans=min(ans,sum of gather distance+king move to gather point);
            ans=min(ans,sum of gather distance + extra move to pick up king+king move to pick up point);
        }
```
 复杂度也很清晰。所有R*C格，只要所有骑士都能走到，都能当聚集点和“上马”点。加上最多R*C个骑士，O((R*C)^3)。最多780格，就是47455200次循环，超时到爽，在我的机器上跑了4000ms左右。
    
**优化：启发式搜索**

一个很简单的想法是：王跑的慢，骑士快，让王多跑一点。选择“上马”点时，尽量离王近一点。抽象一下就是：只有在离王一定距离内（常数C）才能作为pick up点，超出这个距离不考虑。这是在网上广为流传的启发方案。网上的结论常用5作为C。确实通过了，并且速度很快。

```
for (every grid to gather)
    for(every knight to pick up king)
        for (every gird to wait for king)
        {
         if (this knight is 5 more grids from king) continue;
            /*no body pick up king*/
            ans=min(ans,sum of gather distance+king move to gather point);
            ans=min(ans,sum of gather distance + extra move to pick up king+king move to pick up point);
        }
```
我则想出了另一个类似的想法：记录下已知的离王最近骑士的距离d。枚举新的骑士时，如果该骑士到王的距离超出d太多就不考虑。我将这个“太多”定义为>=2*d。
```
if (dist[k][kingpos[0]][kingpos[1]]/2>mindist)	continue;	
mindist=min(mindist,dist[k][kingpos[0]][kingpos[1]]);
```
确实也通过了，约600ms。

看上去很美好，启发式搜索顺利通过，并且速度不俗。然而，上述优化都

#                                     没有严格证明正确性

对于网上流行的方案，甚至已经有了反例：[http://www.yuxuandong.com/post/usaco37.html](http://www.yuxuandong.com/post/usaco37.html)。
由于我的想法和网上的基于同一个思维“让骑士多跑”，我认为也是错的（虽然我不知道怎么证明，遗憾）。也就是说，所谓的启发式，不过是侥幸罢了。

我们换一种思维。枚举每一个聚集点应当不变，但是不应枚举骑士与“上马”点。如果我们能计算出骑士接上王再走到聚集点的**额外**步数，对这个聚集点来说，答案就是：
        所有骑士走到聚集点的步数和，加上某个骑士在某格接上王之后走到聚集点 比他直接走到聚集点多花的步数。（哪格接上？能使多花步数最小的一格。哪个骑士？一样，能使多花步数最小的一个。）

相当难理解。但很显然，我们需要得到每个骑士在每一格**接上王**之后到走到每一格的最小步数。新方法应运而出。

**广搜变形**

我们不仅要得到每个骑士不接王到每格的最短距，也要得到每个骑士接到王之后再走到每格的最小步数。想象在进行广搜时，既有王“上马”的状态，也有未“上马”的状态，两者当然不能混在一起更新步数。很明显，每一格都需要“上马”和未“上马”两种状态，进行广搜时，也要定义当前王有没有“上马”。

```
struct bfsnode{
int x,y,dist;
int king;//    0: not yet pick up    1:picked up!
};
queue<bfsnode> q;
int dist[N][MAXR+1][MAXC+1][2]；//knight's min dist to every grid in 2 different status
```
king值，以及dist的最后一维，代表王是否“上马”。

进行广搜时，对这两种状态，自然要有不同的处理方式。如果当前王没有“上马”，我们可以随时让王“上马”：只要付出王走到骑士的步数作为代价。如果王已经“上马”了，这时广搜就可以更新“上马”之后到每格的步数。也就是说，对于king==0的状态，我们既可以不“上马”，继续保持king==0更新不接王的最小步数，也可以让王“上马”，之后更新“上马”后到每一格的最小步数。

```
/*king_dist[][] is king's min dist to grid(i,j)*/
/*dist[][][] keeps min cost for knight to different grid in different status*/
            if (king==0){            /*now king is not picked up*/
				if (dist[newx][newy][0]>dist+1){/*keep king alone*/
					dist[newx][newy][king]=dist+1;
					q.push(bfsnode{newx,newy,dist+1,0});
				}
				if (dist[newx][newy][1]>dist+1+kingdist[newx][newy]){/*pick up king now!*/
					dist[newx][newy][1]=dist+1+kingdist[newx][newy];
					q.push(bfsnode{newx,newy,dist+1+kingdist[newx][newy],1});
				}
			}
			else  /*king already picked up*/
				if (dist[newx][newy][1]>dist+1){
					dist[newx][newy][1]=dist+1;
					q.push(bfsnode{newx,newy,dist+1,1});
				}
```

得到不同状态下骑士到每格的最小步数还不行，要把它转化为某骑士到这格的最小额外步数，也就是接王再到这格的步数减去不接王到这格的步数。（当然，王自己走的步数已经包含在里面）。

我用exmove[][]保存。如果一个骑士接王到这格的最小额外步数更小，更新之。

```
for_range(i,0,row)
    for_range(j,0,col)
        exmove[i][j]=min(exmove[i][j],dist[k][i][j][1]-dist[k][i][j][0]);
```

那么答案就是每格exmove[][]+distsum[][]的最小值。别忘了加上王自己步行的情况。

```
for_range(i,0,row)
		for_range(j,0,col)
		{
			/*nobody carries king*/
			ans=min(ans,distsum[i][j]+kingdist[i][j]);
			/*one knight take the shortest route to (i,j)*/
			ans=min(ans,distsum[i][j]+exmove[i][j]);
		}
```
完整代码如下。最坏例子400ms。
```
/*ID: lpjwork1
TASK: camelot
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
#include<ctime>
#define for_range(i,x,y)	for (int i=x;i<y;i++)
using namespace std;
const int MAXC=30,MAXR=26,base='A'-1,MAXK=26*30,INF=(1<<20);
const int 	kstep[8][2]={{-2,1},{-1,2},{-2,-1},{-1,-2},{2,1},{1,2},{2,-1},{1,-2}},
			kingstep[8][2]={{-1,0},{-1,1},{0,1},{1,1},{1,0},{1,-1},{0,-1},{-1,-1}};
struct bfsnode{
	int x,y,dist;
	int king;
};
int dist[MAXK+1][MAXR+1][MAXC+1][2],kpos[MAXK+1][2],kingpos[2],
	kingdist[MAXR+1][MAXC+1],distsum[MAXR+1][MAXC+1],exmove[MAXR+1][MAXC+1];
int kn=0,row,col,ans=INF;
queue<bfsnode> q;
/**/
void init();
void input();
void knbfs(int,int,int[][MAXC+1][2],const int[][2]);
void bfs(int,int,int[][MAXC+1],const int[][2]);
bool inboard(int,int);
ofstream fout ("camelot.out");
ifstream fin ("camelot.in");
int main() {
	input();
	init();
	for_range(k,0,kn)
	{
		knbfs(kpos[k][0],kpos[k][1],dist[k],kstep);
		for_range(i,0,row)
			for_range(j,0,col)
				if (dist[k][i][j][0]!=INF)	distsum[i][j]+=dist[k][i][j][0];
				else						distsum[i][j]=INF;
		for_range(i,0,row)
			for_range(j,0,col)
			{
				exmove[i][j]=min(exmove[i][j],dist[k][i][j][1]-dist[k][i][j][0]);
			}
	}
	for_range(i,0,row)
		for_range(j,0,col)
		{
			/*nobody carries king*/
			ans=min(ans,distsum[i][j]+kingdist[i][j]);
			/*one knight take the shortest route to (i,j)*/
			ans=min(ans,distsum[i][j]+exmove[i][j]);
		}
	cout<<ans<<endl;
	fout<<ans<<endl;
    fout.close();fin.close();
    return 0;
} 

void bfs(int sorx,int sory,int to[][MAXC+1],const int step[][2])
{
	for_range(i,0,row)
			for_range(j,0,col)	to[i][j]=INF;
	while (!q.empty())	q.pop();
	to[sorx][sory]=0;
	q.push(bfsnode{sorx,sory,0});
	while (!q.empty())
	{
		int x=q.front().x,y=q.front().y,dist=q.front().dist;q.pop();
		for_range(i,0,8)
		{
			int newx=x+step[i][0],newy=y+step[i][1];
			if (!inboard(newx,newy)||to[newx][newy]!=INF)	continue;
			to[newx][newy]=dist+1;
			q.push(bfsnode{newx,newy,dist+1});
		}
	}
}

void input()
{
	fin>>col>>row;
	char r;fin>>r>>kingpos[1];r-=base;kingpos[0]=r;
	kingpos[0]--;kingpos[1]--;
	int c;
	while (1)
	{
		r='$';c=-1;
		fin>>r>>c;
		if (r=='$'||c==-1)	return;
		kpos[kn][0]=r-base;kpos[kn][1]=c;
		kpos[kn][0]--;kpos[kn][1]--;
		kn++;
	}
}

void init()
{
	for_range(i,0,row)
		for_range(j,0,col)	distsum[i][j]=0;
	for_range(i,0,row)
			for_range(j,0,col)	exmove[i][j]=INF;
	bfs(kingpos[0],kingpos[1],kingdist,kingstep);
}

bool inboard(int x,int y)
{
	return (x>=0&&x<row&&y>=0&&y<col);
}

void knbfs(int knx,int kny,int to[][MAXC+1][2],const int step[][2])
{
	while (!q.empty())	q.pop();
	for_range(i,0,row)
		for_range(j,0,col)
			for_range(k,0,2)	to[i][j][k]=INF;
	to[knx][kny][0]=0;
	q.push(bfsnode{knx,kny,0,0});
	q.push(bfsnode{knx,kny,kingdist[knx][kny],1});
	while (!q.empty())
	{
		bfsnode& now=q.front();
		int x=now.x,y=now.y,dist=now.dist;int king=now.king;
		q.pop();
		for_range(i,0,8)
		{
			int newx=x+step[i][0],newy=y+step[i][1];
			if (!inboard(newx,newy))	continue;
			/*without king*/
			if (king==0){
				if (to[newx][newy][0]>dist+1){
					to[newx][newy][king]=dist+1;
					q.push(bfsnode{newx,newy,dist+1,0});
				}
				if (to[newx][newy][1]>dist+1+kingdist[newx][newy]){
					to[newx][newy][1]=dist+1+kingdist[newx][newy];
					q.push(bfsnode{newx,newy,dist+1+kingdist[newx][newy],1});
				}
			}
			/*with king*/
			else
				if (to[newx][newy][1]>dist+1){
					to[newx][newy][1]=dist+1;
					q.push(bfsnode{newx,newy,dist+1,1});
				}
		}
	}
}
```
/                                                                                  2018.11.14


                                                                                                
