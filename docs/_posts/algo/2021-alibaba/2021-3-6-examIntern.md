---
title: "阿里巴巴实习生招聘3月6日笔试题"
layout: post
categories: algorithm
author: noobi
date: 2021-03-06
tags: algorithm
---
[第一题](#1)

[第二题](#2)

<h2 id="1"> 第一题 回溯、递推与数学</h2>

有1，2，3三张卡片，现在输入一个N，构造一个N*3的矩阵，在矩阵中填入1，2，3

相邻的单元格数字不能重复。

如图所示:

![小明的卡片](/assets/algo/alibaba/card.png)

请问有几种填法？

注：$1 \le N \le 500$，结果要求对 $1000000007$ 取模

### 前言

阿里笔试限时**1小时**做两题，本身还是比较紧张的。

平时力扣刷的比较多，笔试却是**acm模式**（手动处理输入输出）

用Go刷的题，标准输入却不怎么接触，刚开始调试输入输出花了不少时间，引以为戒。

当然第一道难关是语文，看懂题搞清边界再着手解题，再是编码。

### Naive Thoughts

这道题第一想法是回溯剪枝，类似于[N皇后](https://leetcode-cn.com/problems/n-queens/)的情况。

N皇后的位运算，将每个格子的状态分成了``0,1``两种状态（放/不放），用二进制比特表示。

而这道题的是将状态变成了三种：``1，2，3``。需要用**三进制（[吹特](https://en.wikipedia.org/wiki/Ternary_numeral_system)）表示**。

位运算剪枝的路子就被堵住了。

只能老老实实回溯。

注意⚠️：以下的解法TLE（超时）

~~~go
package main

import "fmt"

var res = 0

func main() {
	var (
		T int
	)
	fmt.Scanln(&T)
	arr := make([]int, T)
	for t := T-1; t >= 0; t-- {
		fmt.Scanln(&arr[t])
	}

	for _,n:=range arr {
		solve(n)
		fmt.Println("res: ", res)
		res = 0
	 }
}

func solve(n int) {
	board := make([][]int, n)
	for i := 0; i < n; i++ {
		board[i] = make([]int, 3)
	}
	dfs(board, n)
}

func dfs(board [][]int, n int,count int) {
	if count == n*3 {
		fmt.Println(board)
		res++
	}
	// filled out
	for i := 0; i < n; i++ {
		for j := 0; j < 3; j++ {
			if board[i][j] == 0 {
				for k := 1; k <= 3; k++ {
					if valid(i, j, board, k) {
						board[i][j] = k
						dfs(board, n,count+1) 	
					} 	
				}
			}
		}
		return 
	}
}

func valid(i int, j int, board [][]int, k int) bool {
	if i < 0 && i >= len(board) && j < 0 && j >= len(board) {
		return false
	}
	up, down, left, right := i-1, i+1, j-1, j+1
	if up >= 0 && board[up][j] == k {
		return false
	}
	if left >= 0 && board[i][left] == k {
		return false
	}
	if right <= len(board[i])-1 && board[i][right] == k {
		return false
	}
	if down <= len(board)-1  && board[down][j] == k {
		return false 
	}
	return true
}
~~~

### 通解法

参考了力扣各路大神的解法

主要思想是要跳出回溯的框架（因为$1 \le N \le 500$）

如果仍然回溯，粗略计算时间复杂度可以达到$O(3^n)$，**一定是超时的**。



##### 如何降低复杂度？

 1. 消除**重复子问题**（overlapping subproblems）

    这里的做法就是筛选可能的行列，即把上下相同左右相同的情况筛掉。

 2. **降维打击**，**状态表示方法**（state representation）很重要。

    前面讲过，每个格子的状态可以用 1吹特（trit），即一个三进制数表示。

    举个例子，[1,2,0]有三种状态，我们需要$O(n)$遍历数组确定某一行的填法。

    如果表示成$(120)_3$三进制形式，$O(1)$时间即可得到这一行的填法。

    于是有公式：
    $$
    row_i = x \times3^2 + y \times3^1 + z \times 3^0
    $$
    来唯一确定一行的状态。

    **换句话说，我们只要一个一维数组就能完全表示整个矩阵。**

    比如，

    从``[[1,2,1],[2,1,2],[3,2,1]]``到``[121,212,321]``

    是不小的开销节省。

3. **递推、穷举**

    我们对于 **每一行的一种表示、每一行** 穷举出所有可能的情况，并填入表格中。

    

### 编码

已经将说明写入注释

~~~go
package alibaba

import "fmt"

// 阿里巴巴3.6笔试题1
//有1，2，3三张卡片，现在输入一个N，构造一个N*3的矩阵，在矩阵中填入1，2，3

// 相邻的单元格数字不能重复。

// 请问有几种填法？
func main() {
	var (
		T int
	)
	fmt.Scanln(&T)
	arr := make([]int, T)
	for t := T-1; t >= 0; t-- {
		fmt.Scanln(&arr[t])
	}

	for _,n:=range arr {
		fmt.Println(solve(n))
	 }
}

func solve(n int) int {

	var mod int64 = 1000000007
	// 1. 压缩状态 1,2,3 -> (123)

	types := make([]int, 0, n)
	// 2. 预处理左右相同的情况
	for i := 0; i < 3; i++ {
		for j := 0; j < 3; j++ {
			for k := 0; k < 3; k++ {
				if i != j && j != k {
					types = append(types, i*9+j*3+k)
				}
			}
		}
	}

	typesCnt := len(types)
	// 3. 预处理上下相同的情况
	related := make([][]bool, typesCnt)
	for i := range related {
		related[i] = make([]bool, typesCnt)
	}

	// 两重循环暴力匹配
	for i := range types {
		x, y, z := types[i]/9, types[i]/3%3, types[i]%3
		for j := 0; j < typesCnt; j++ {
			x1, y1, z1 := types[j]/9, types[j]/3%3, types[j]%3
			if x != x1 && y != y1 && z != z1 {
				related[i][j] = true
			}
		}
	}
	// 4. 算法主体：递推dp
	// 构造n行，typesCnt 列矩阵，穷举所有可能的情况
	f := make([][]int64, n)
	for i := range f {
		f[i] = make([]int64, typesCnt)
	}
	// 从下标1开始，方便表示
	// 边界情况：第一行随便填
	for i := 0; i < typesCnt; i++ {
		f[0][i] = 1
	}
	// 第一行略过，已经填完了
	for i := 1; i < n; i++ {
		// 暴力穷举可能的情况
		for j := 0; j < typesCnt; j++ {
			// f[i][j] = ∑ f[i-1][k]
			for k := 0; k < typesCnt; k++ {
				// 随便取出两种情况比一比
				if related[k][j] {
					f[i][j] += f[i-1][k]
					f[i][j] %= mod
				}
			}
		}
	}
	fmt.Println(f)
	// 取答案，最后一行就是答案
	var ans int64
	for i := 0; i < typesCnt; i++ {
		ans += f[n-1][i]
		ans %= mod
	}
	return int(ans)
}

~~~



### 数学竞赛思路

这种思路可谓是超脱了穷举的思路，直接在例子中找规律。

引用力扣评论的一句话：

> 很多人困在以格子为单位进行推导，进入了死胡同，高手就是能及时抽身，想到以行为单位进行推导。
>
> ​																																	---[abstract](https://leetcode-cn.com/u/abstract-5/) 2020-11-23

#### 分析

借鉴**数学归纳法**的思想

当 $n=1$ 时，我们容易穷举出所有可能性：

```
*[1 2 1] *[2 1 2]  [3 1 2]

 [1 2 3]  [2 1 3] *[3 1 3]

*[1 3 1]  [2 3 1]  [3 2 1]

 [1 3 2] *[2 3 2] *[3 2 3]
```

###### 穷举代码见**附录**。

观察以上代码，发现存在ABA 与 ABC两种模式（ABA模式打了星号*）各占一半

拿``[1 2 1] , [1 2 3]``举例，尝试增加一行数字，可以填入如下的数字

~~~
121:		 123:
*2 1 2		*2 1 2
*2 3 2     2 3 1
 2 1 3		 3 1 2          
*3 1 3		*2 3 2 
 3 1 2
~~~

将ABA类型打星号*，发现如下规律：

```
ABA ---> 3 ABA + 2 ABC
		下层
		
ABC ---> 2 ABA + 2 ABC
		下层
```

即

设n层 ABA 填法数量为 $a_n$, ABC 填法数量为$ b_n$，有递推公式
$$
a_n = 3a_{n-1} + 2 b_{n-1}\\
b_n = 2a_{n-1} + 2b_{n-1}
$$
这就不难写出代码：

~~~go
func solveMath(n int) int {
	var mod int64 = 1000000007
	// 穷举得到n=1的情况
	var (
		aba int64 = 6
		abc int64 = 6
	)
	for i := 1; i < n; i++ {
		newAba := (aba*3)%mod + (abc*2)%mod
		newAbc := (aba*2)%mod + (abc*2)%mod
		aba = newAba
		abc = newAbc
	}
	return int((abc + aba) % mod)
}
~~~

至此，这样一道Hard题就迎刃而解。





<h2 id="2">第二题 最短路径与环</h2>

有N条**环形地铁线路**，每条之间可能**存在中转站**，比如，

~~~go
[1,2,3,4,5]
[2,6]
~~~

表示存在一条

~~~
1->2->3->4
	 +->6
~~~

的线路。

输入说明：

- 先输入T，代表一共有T组数，然后输出n,s,t 其中s和t代表**起始位置和终点位置。m**

- **代表一共有n个地铁线路。**

- **接下来再输入n\*2行，第一行是一个数字k，代表有k个停靠站点，第二行才是所有停靠站点。**

```shell
1 #T组数据
2 1 7 # 线路数量 起始点 终止点
3 # 三个站
1 2 3 # 站台
4 #四个站
2 4 7
# EOF
```



输出说明：

- 输出每组数据最少需要搭乘的线路数量
```shell
2 # 需要搭乘2趟列车
```



### 读题

这道题由于是图论，输入输出较为麻烦，我们先解决输入输出

```go
func main() {
	var (
		routes [][]int
		T      int
		n      int
		s      int
		t      int
		N      int
		res    []int
	)
	fmt.Scanln(&T)
	for t1 := T; t1 > 0; t1-- {
		fmt.Scanln(&n, &s, &t)
		// n 条线路
		for i := 0; i < n; i++ {
			// N 个站台
			fmt.Scanln(&N)
			tmp := make([]int, N)
			// 输入线路
			for j := 0; j < N; j++ {
				fmt.Scan(&tmp[i])
			}
			routes = append(routes, tmp)
		}
		res = append(res, solve(routes, s, t))
		routes = [][]int{}
	}

	for i := range res {
		_ = i
		fmt.Println(res)
	}
}
```



### 正式解题

此题的关键在于：如何定位换乘点，也就是地铁路线环的**交点**？

1. 

> 在计算两条公交路线是否有交集时，可以用的方法有很多种。
>
> 例如将公交路线放在集合中，检查两个集合的交集是否为空；
>
> 将公交路线中的车站进行递增排序，并使用双指针的方法检查是否有相同的车站。

~~~go
	
func main() {
	var (
		routes [][]int
		T      int
		n      int
		s      int
		t      int
		N      int
		res    []int
	)
	fmt.Scanln(&T)
	for t1 := T; t1 > 0; t1-- {
		fmt.Scanln(&n, &s, &t)
		// n 条线路
		for i := 0; i < n; i++ {
			// N 个站台
			fmt.Scanln(&N)
			//fmt.Println(routes)
			tmp := make([]int, N)
			var M = 0
			// 输入线路
			for j := 0; j < N; j++ {
				fmt.Scan(&M)
				tmp[j] = M
			}
			routes = append(routes, tmp)
			fmt.Println(routes)
		}
		res = append(res, solve(routes, s, t))

		routes = [][]int{}
	}

	for i := range res {
		_ = i
		fmt.Println(res)
	}
}

~~~



2. 我们也可以记录每个站台对应的路线集合，如果**一个站台存在两个及以上的路线，就是换乘点。**

~~~go
  // 站点 -> 路线的集合
	toRoute := map[int][]int{}
	for i:= range routes {
    for j := range routes[i] {
      toRoutes[j] = append(toRoutes[j], i)
    }
	}
~~~

接下来就是进行常规的BFS了

~~~go
func solve(routes [][]int, s int, t int) int {
	// 站点 -> 路线的集合
	toRoute := map[int][]int{}
	for i := range routes {
		for _, j := range routes[i] {
			toRoute[j] = append(toRoute[j], i)
		}
	}
	queue := make([][]int, 0)
	queue = append(queue, []int{s, 0})
	visited := map[int]int{s: 1}
	visitedRoute := make([]bool, len(routes))
  // bfs
	for len(queue) != 0 {
		top := queue[0]
		stop, bus := top[0], top[1]
		queue = queue[1:]
		if stop == t {
			return bus
		}
    // 遍历所有经过该点的线路
		for _, i := range toRoute[stop] {
      // 遍历线路
			for _, j := range routes[i] {
        // 检查是否访问过
				if _, ok := visited[j]; ok {
					continue
				}
				if visitedRoute[i] {
					continue
				}
        // 站台计数+1
				queue = append(queue, []int{j, bus + 1})
			}
			visitedRoute[i] = true
		}

	}
	return -1
}

~~~



### 反思

1. 刷题量不够大。

2. 力扣不需要处理输入输出，笔试前也没有提前适应。
3. 一小时的做题感受到了紧张，直面自己与大厂的差距。
4. 由于只实现了拉链法，Go语言的map不适合刷题，map的效率远不如java的HashMap与C++的unordered_map；第二题如果TLE了可以尝试双向BFS提交。
5. 网上都在说两道Hard题不讲武德，不行就是不行，不是题的问题。
6. 仅以此文，与君共勉。

## 附录

穷举代码

~~~go
func dfs(i int, array []int) {
	if i == len(array) {
		fmt.Println(array)
		return
	}
	for k := 1; k <= 3; k++ {
		if valid1(i, array, k) {
			array[i] = k
			dfs(i+1, array)
			array[i] = 0
		}
	}
	res++
}

func valid1(i int, array []int, k int) bool {
	left, right := i-1, i+1
	if left >= 0 && k == array[left] {
		return false
	}
	if right < len(array) && k == array[right] {
		return false
	}
	return true
}
~~~



### REF

- [官方通解](https://leetcode-cn.com/problems/number-of-ways-to-paint-n-3-grid/solution/gei-n-x-3-wang-ge-tu-tu-se-de-fang-an-shu-by-leetc/)

- [数学方法](https://leetcode-cn.com/problems/number-of-ways-to-paint-n-3-grid/solution/shu-xue-jie-jue-fei-chang-kuai-le-by-lindsaywong/)

- [BFS](https://leetcode.com/problems/bus-routes/discuss/122771/C%2B%2BJavaPython-BFS-Solution)
- [视频题解](https://www.youtube.com/watch?v=R58Q0J52qzI)

