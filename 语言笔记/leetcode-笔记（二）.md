### 1380. Lucky Numbers in a Matrix
有时想for遍历的同时判断，如果满足条件向list添加一个值，否则不添加任何东西。
```
[2 * x if x > 2 else None for x in some_list ]
```
类似上述的写法是不行的，None并不是真的空，且if后不能没有else。
解决方法很简单，把if判断放到for之后就不可以不加else。
```
class Solution:
    def luckyNumbers (self, matrix: List[List[int]]) -> List[int]:
        m=len(matrix)
        n=len(matrix[0])
        colMax=[max([matrix[row][col] for row in range(m)]) for col in range(n)]
        rowMin=[min([matrix[row][col] for col in range(n)]) for row in range(m)]
        return [colMax[col] for col in range(n) for row in range(m) if colMax[col]==rowMin[row]]
```

###1389. Create Target Array in the Given Order
```
class Solution:
    def createTargetArray(self, nums: List[int], index: List[int]) -> List[int]:
        ret=[]
        for ind,num in zip(index,nums):
            ret.insert(ind,num)
        return ret
```
zip将多个iterable并在一起遍历，不用写麻烦的```for i in range(n):a[i],b[i]```。

###int,round,floor,ceil
一张表说明四者的差别。
```
x       int floor   round   ceil
1.0     1   1.0     1.0     1.0
1.1     1   1.0     1.0     2.0
1.5     1   1.0     2.0     2.0
1.9     1   1.0     2.0     2.0
-1.1    -1  -2.0    -1.0    -1.0
-1.5    -1  -2.0    -2.0    -1.0
-1.9    -1  -2.0    -2.0    -1.0
```

###
