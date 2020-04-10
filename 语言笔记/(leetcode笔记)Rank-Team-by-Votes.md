原题链接：[https://leetcode.com/problems/rank-teams-by-votes/description/](https://leetcode.com/problems/rank-teams-by-votes/description/)
leetcode的题目多半没什么算法难度，写笔记是为了记录优秀精致的代码以及不熟悉的语言特性。

```
class Solution:
    def rankTeams(self, votes: List[str]) -> str:
        cnt=[([0]*26 + [-i]) for i in range(26)]
        for vote in votes:
            for i,c in enumerate(vote):
                cnt[ord(c)-ord('A')][i]+=1
        return "".join(sorted(votes[0],key=lambda k:cnt[(ord(k)-ord('A'))],reverse=True))
```
1.list间的比较是字典序式的：可以把list的每项看作字符串的一个字符，这样两个list比大小相当于两字符串比大小。
2.用多个键对一种对象排序时，若不同键的正反序不一样，可以把反序的键值赋值为负。例如：给人按年龄增序、学号降序排序，可将学号设为负。
3.将需要排序的对象放进list，用lambda从其它数据结构获取对象的属性作为key。
4.用[obj]*n语法初始化list时，若obj是python中的对象而非像int一样的简单数据，生成的list中所有obj将是一个obj的引用。使用[] for range(n)创建list。
