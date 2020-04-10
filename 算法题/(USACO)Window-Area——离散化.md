题目大意：有一个类似windows的窗口系统。窗口是矩形，且坐标是整数。因为顺序，上面的窗口会遮住下面的窗口。用户会创建窗口、销毁窗口、把窗口提到最上面及放到最下面，以及询问：问一个窗口没有被遮住的面积占其原面积的百分比**S**。
任意时刻窗口总数N小于64。窗口坐标大于0小于32768。询问的命令不超过500个。

这个问题的核心是：如何处理矩形互相覆盖？

###最朴素的想法：模拟
如果坐标范围很小的话，可以考虑最朴素的方法。以1*1的格子为单位，用二维数组模拟这个窗口系统。当要计算S时，先把要计算的窗口对应二维数组范围内的所有格子涂色；然后把其上方所有窗口对应的格子的颜色抹去。最后数出有颜色的格子的数量，也就是未覆盖的面积。

![红色即是未覆盖面积](https://upload-images.jianshu.io/upload_images/14546900-6589c380942a3e57.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


**这当然是个极差的做法。但是，从这最朴素的做法中我们可以发现优化的空间。**
下意识的，我们用了以1*1的格子为基本单位的二维数组来涂色。然而可以发现，很多格子其实是没有意义的。
矩形的四个点将整个平面划分开，产生一个一个的小矩形。实际上，是这些小矩形作为基本单位参与了计算。

![](https://upload-images.jianshu.io/upload_images/14546900-7e5b4013274f94b4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


如果我们能把以这些小矩形为基本单位，像操作1* 1的格子一样方便地在二维数组里涂色，问题就解决了。那么为什么对它们的操作会很艰难？
1* 1的格子可以规整的排列在二维数组里，而这些小矩形的大小“参差不齐”，无法排列。
但是，如果我们无视矩形不一的大小，将它们抽象成1*1的格子，把其坐标作为另一组信息保存并和格子建立对应，不就可以放进二维数组了吗？

![](https://upload-images.jianshu.io/upload_images/14546900-e4fa56069ff67053.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


####坐标系变换
请看下图中的点。如果我们打算把这些分布不一的点放进二维数组里，以点的顺序对应二维数组的下标，应该怎么对应?

![](https://upload-images.jianshu.io/upload_images/14546900-d4683459be71be62.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


设一个点在二维数组中的下标是[i][j]。显然，点的y坐标是$ 2*i $；而x坐标稍复杂，$2^j-1$。
实际上，在这个迷你的坐标变换系统中，我们用了两个函数：$y=2*i$和$x=2^j-1$将下标映射到坐标。
那么如何为像下图一样坐标分布几乎无规律，以至于找不到“一般的”函数以映射的点找一个变换函数呢？

![](https://upload-images.jianshu.io/upload_images/14546900-672a22b63ac79008.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


答案是：把坐标记下来。
设有数组xvector和yvector。按照x坐标从左向右，y坐标从上向下的顺序，**记录了所有的x坐标和y坐标**。定义这样的映射函数：
>$x=xvector[j]$
  $y=yvector[i]$

这样就成功地建立了下标到坐标的映射——虽然，这是一个看起来很一般的函数。

但这也是一个很普适的方法。这就是**离散化**。

####离散化
在我们的窗口系统中，每一个点都可能将整个平面划分出新的矩形来。所以我们对所有点进行离散化。
把所有点的x坐标和y坐标分别放进数组xvec和yvec，分别排序，去重，以xvec和yvec的大小作为布尔型二维数组的长和宽。通过xvec和yvec，我们就能用下标恢复到坐标。
已知一个窗口的坐标xy，分别在xvec和yvec中进行二分查找，就能知道该窗口在二维数组中下标覆盖的范围。
然后还是用最朴素的方法，涂色。只不过每个窗口都要二分查找找到下标范围并涂色。
最后，将所有上了色的格子对应矩形的面积相加，得到未覆盖面积。

这样，一次询问就完成了。最多64个窗口，每个矩形在x和y方向上各最多带来2个新的值，所以用于涂色的二维数组只要128*128，符合范围。

---
####懒惰计算的思考
在我给出的代码中，每逢一次询问都重新计算了一次S。最后一个测试数据花了0.14s。
但我曾考虑过一个懒惰计算的优化：对一个窗口来说，只有当它上方的窗口发生变化（增加了，减少了），它的S才会变化。所以可以给每个窗口增加一个double型变量表示S、一个布尔型变量need_change，表示是否需要重新计算。
当一个窗口变化时，需要维护其它窗口的need_change。例如，当一个窗口移到最底下时，其旧的位置和新的位置之间的所有窗口的need_change需设为true。其它变化以此类推。

![维护need_change](https://upload-images.jianshu.io/upload_images/14546900-6974f0ea90280d16.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


计算S时，若need_change为false则直接返回记录的S，否则才重新计算，更新其S，并把need_change设为false。


理论上这是个优化手段，我提交了运用此优化的版本。但发现这增加了不少代码复杂度。题库的最后一个测试数据中，250次询问只有40次应用到了此优化——也就是计算时need_change为false，以至于时间竟没有优化多少。但我认为这样的懒惰计算在实际应用中是有意义的，所以也在此记录。

代码如下。为了简洁，没有应用懒惰优化。

```
/*ID: lpjwork1
TASK: window
LANG: C++11
*/
#include<iostream>
#include<stdio.h>
#include<map>
#include<string>
#include<string.h>
#include<vector>
#include<queue>
#include<iomanip>
#include<algorithm>
#include<set>
#include<fstream>
using namespace std;
const int MAXN=124+10;

/*bool array to color*/
bool inhold[MAXN][MAXN];

struct rect{
	double x1,x2,y1,y2;
};

struct win{
	rect data;
	char id;
};

bool operator==(rect a,rect b)
{
	return a.x1==b.x1&&a.x2==b.x2&&
		   a.y1==b.y1&&a.y2==b.y2;
}

/*structure to save all window*/ 
vector<win> stack;
/*save coordinate,x and y coordinate*/
vector<double> xvec,yvec;

double area(rect&);
double area(win&);
double caculate(int);
int findbyid(char todo);
void swp(double&,double&);
void disolve(win& todo,bool color);

ofstream fout ("window.out");
ifstream fin ("window.in");
int main() {
	bool flag=true;
	while (flag)
	{
		char comm='\0';
		fin>>comm;fin.get();
		switch (comm)
		{
			/*create window*/
			case 'w':
			{	
				win temp;fin>>temp.id;fin.get();
				fin>>temp.data.x1;fin.get();
				fin>>temp.data.y1;fin.get();
				fin>>temp.data.x2;fin.get();
				fin>>temp.data.y2;fin.get();
				if (temp.data.x2<temp.data.x1)	swp(temp.data.x2,temp.data.x1);
				if (temp.data.y2<temp.data.y1)	swp(temp.data.y2,temp.data.y1);
				stack.push_back(temp);
				break;
			}
			/*bring to top*/
			case 't':
			{
				char todo;fin>>todo;fin.get();
				int temp=findbyid(todo);
				win twin=stack[temp];
				stack.erase(stack.begin()+temp);
				stack.push_back(twin);
				break;
			}
			/*put to bottom*/
			case 'b':
			{
				char todo;fin>>todo;fin.get();
				int temp=findbyid(todo);
				win twin=stack[temp];
				stack.erase(stack.begin()+temp);
				stack.insert(stack.begin(),twin);
				break;
			}
			/*destroy*/
			case 'd':
			{
				char todo;fin>>todo;fin.get();
				int temp=findbyid(todo);
				stack.erase(stack.begin()+temp);
				break;
			}
			/*show visble*/
			case 's':
			{
				char todo;fin>>todo;fin.get();
				int temp=findbyid(todo);
				caculate(temp);
				break;
			}
			/*over*/
			default:flag=false;break;
		}
	}
    fout.close();fin.close();
    return 0;
}

double area(rect& todo)
{
	return (todo.y2-todo.y1)*(todo.x2-todo.x1);
}

double area(win& todo)
{
	return area(todo.data);
}

int findbyid(char todo)
{
	int ret=0;
	while (stack[ret].id!=todo)	ret++;
	return ret;
}

double caculate(int todo)
{
	double raw_area=area(stack[todo]);
	
	xvec.clear();yvec.clear();	
	for (int i=0;i<=stack.size()*2;i++)
		for (int j=0;j<=stack.size()*2;j++)	inhold[i][j]=false;
	/*first,get all x,y in stack and discretization*/
	for (int i=stack.size()-1;i>=todo;i--)
	{
		rect& now=stack[i].data;
		double x1=now.x1,x2=now.x2,y1=now.y1,y2=now.y2;
		xvec.push_back(x1);xvec.push_back(x2);
		yvec.push_back(y1);yvec.push_back(y2);
	}
	sort(xvec.begin(),xvec.end());
	sort(yvec.begin(),yvec.end());
	xvec.resize(unique(xvec.begin(),xvec.end())-xvec.begin());
	yvec.resize(unique(yvec.begin(),yvec.end())-yvec.begin());
	
	disolve(stack[todo],true);
	/*disolve windows above and check overlap*/
	for (int i=stack.size()-1;i>todo;i--)
		disolve(stack[i],false);
	/*sum area*/
	double new_area=0;
	for (int i=0;i<=stack.size()*2;i++)
		for (int j=0;j<=stack.size()*2;j++)
			if (inhold[i][j]){
				rect temp=rect{xvec[i],xvec[i+1],yvec[j],yvec[j+1]};
				new_area+=area(temp);
			}
		
	fout<<setprecision(3)<<fixed<<new_area*100/raw_area<<endl;
}

void disolve(win& todo,bool color)
{
	int x1=todo.data.x1,x2=todo.data.x2,y1=todo.data.y1,y2=todo.data.y2;
	/*binary search,for index of high and low boundary of x and y*/
	int xl=lower_bound(xvec.begin(),xvec.end(),todo.data.x1)-xvec.begin(),
	xu=lower_bound(xvec.begin(),xvec.end(),todo.data.x2)-xvec.begin(),
	yu=lower_bound(yvec.begin(),yvec.end(),todo.data.y2)-yvec.begin(),
	yl=lower_bound(yvec.begin(),yvec.end(),todo.data.y1)-yvec.begin();
	for (int i=xl;i!=xu;i++)
		for (int j=yl;j!=yu;j++)
			inhold[i][j]=color;
}

void swp(double& a,double& b)
{
	double t=a;a=b;b=t;
}
```
2019.1.31
