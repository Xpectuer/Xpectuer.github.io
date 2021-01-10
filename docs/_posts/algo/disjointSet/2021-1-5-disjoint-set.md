---
layout: post
title: "聊聊并查集（Disjoin Set）"
tags: tree, special data structure
categories: algorithm
---
## 并查集（Disjoint Set 分立集合 Union Find）

并查集的概念比较跳跃，应当**独立记忆**

### 效用

并查集解决了独立集合之间的查找与合并问题

*并查集一般可以解决具有传递性关系的问题*

比如：

1. 朋友圈问题
2. 岛屿个数问题
3. 编译器变量多个引用问题

实现一个Union Find需要以下的API

```java
interface UF {
    /* 将 p 和 q 连接 */
    public void union(int p, int q);
    /* 判断 p 和 q 是否连通 */
    public boolean connected(int p, int q);
    /* 返回图中有多少个连通分量 */
    public int count();
}
```

### 初始化

```cpp
// iterative
for(int i: parent)
	parent[i] = i
```

此时每个元素都是集合的**领头元素**，即每个元素独立成一个集合。

### 查找领头元素（Find）

为了更好的合并，我们需要找到每个分立集合的**领头元素**

我们分别用递归和迭代的方式实现

- 迭代

  ```cpp
  int find(vector<int> p, int i) {
    int root = i;
    while(p[root] != root) {
      root = p[root];
    }
    // root found
    // route compression
    while(p[i]!=i) {
      int x=i;i=p[i];p[x]=root;
    }
  }
  ```



- 递归

  ```cpp
  // NO route compression
  int find(vector<int> p, int i) {
    return i==p[i]?i:find(p[i]);
  }
  ```



### 合并（Union）

![union](/assets/algo/disjointSet/union.png)


- 找到各个集合的**领头元素**
- 合并两个领头元素

```cpp
// 还可以优化
void union(vector<int> &p, int i, int j) {
  int rootI = find(p, i);
  int rootJ = find(p, j);
  if(rootI==rootJ) return;
  // p[rootJ] = rootI; // 等效
  p[rootI] = rootJ;
  count--;
}
```

#### 问题&优化

既然是合并集合树，那么必定会出现如：

1. **退化为链表**

2. 左右子树不平衡

这种降低**查询效率**与**路径压缩效率**的问题

- 优化：保持的UF的平衡性
  - 尽量不让小树的头作为领头元素

那么如何判断树的大小呢？

我们维护一个与``p[n] //parent``同样大小的``size[n]`` 数组

``size`` 内的元素维护每个元素的子节点个数

看代码:

```cpp
class UF {
private:
    int count;
    vector<int> parent;
    // 新增一个数组记录树的“重量”
		vector<int> size;
public:
  	UF(int n) {
        this.count = n;
        parent = vector<int>(n);
        // 最初每棵树只有一个节点
        // 重量应该初始化 1
        size = vector<int>(n, 1);
        for (int i = 0; i < n; i++) {
            parent[i] = i；
        }
    }
  	void union(vector<int> &p, int i, int j) {
      int rootI = find(i);
      int rootJ = find(j);
      if(size[rootI] > size[rootJ]) {
        p[rootJ] = rootI;
        size[rootI] += size[rootQ];
      } else {
        p[rootI] = rootJ;
        size[rootJ] += size[rootI];
      }
      count--;
    }


    /* 其他函数 */
}
```

### 例题

#### leetcode547. Province Numbers

这道题类似于岛屿数量，可以用DFS做

但是这里用并查集可以减少运算的复杂度

```cpp
class Solution {
private:
    vector<int> parent;
    vector<int> size;
    int count;
    int find(int p) {
        return p==parent[p]?p:find(parent[p]);
    }

    void _union(int p, int q) {
        int rootP = find(p);
        int rootQ = find(q);
        if (rootP==rootQ) return;
        //opt
        if(size[rootP] > size[rootQ]) {
            parent[rootQ] = rootP;
            size[rootP] += size[rootQ];
        } else {
            parent[rootP] = rootQ;
            size[rootQ] += size[rootP];
        }
        //cout<<this->count<<endl;
        this->count--;
    }

public:
    int findCircleNum(vector<vector<int>>& M) {
        int n = M.size();
        if(n==0) return 0;


        /*INITIALIZATION*/
        this->count = n;
        parent = vector<int>(n);
        for(int i=0;i<n;i++) parent[i] = i;
        size = vector<int>(n, 1);
        /*INITIALIZATION*/

        for(int i=0;i < n; i++) {
            for(int j=i+1;j < n; j++) {
                if(M[i][j]==1){
                    //cout<<count<<endl;
                    _union(i, j);
                }

            }
        }
        return count;
    }


};

```
### 进阶练习推荐

> 记住，并查集一般可以解决具有传递性关系的问题

[399. Evaluate Division](https://leetcode-cn.com/problems/evaluate-division/)\\
PAT 2020.12 第4题 化学反应方程式
