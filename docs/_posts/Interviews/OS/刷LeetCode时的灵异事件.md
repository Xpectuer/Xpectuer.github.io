# 刷LeetCode时的灵异事件

https://leetcode.com/problems/path-sum-ii/



```go
func pathSum(root *TreeNode, targetSum int) [][]int {
    var res [][]int = [][]int{}
    if root == nil {
        return res
    }
    path := []int{}
    dfs(root, targetSum, path,&res) 
    return res
}

func dfs(root *TreeNode, targetSum int, path []int,res *[][]int) {
    
    if root == nil {
        return
    }
    if root.Left == nil && root.Right == nil {
      
        if targetSum - root.Val == 0 {
            // 灵异事件
            fmt.Println(path, root.Val)
            (*res) = append((*res), append(path, root.Val))
        }
        return
    }
    path = append(path, root.Val)
    dfs(root.Left, targetSum - root.Val, path,res)    
    dfs(root.Right, targetSum - root.Val, path,res)     
}
```

