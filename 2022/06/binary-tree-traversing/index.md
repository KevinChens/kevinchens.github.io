# algorithm-pattern | 二叉树


## binary-tree

本系列是[algorithm-pattern](https://greyireland.gitbook.io/algorithm-pattern/)的学习笔记。主要目的是为了合理刷题，积累算法模板，有套路的进行leetcode刷题。安利一下该[仓库](https://github.com/greyireland/algorithm-pattern)，作者总结归纳的不错，适合作为leetcode刷题的参考资料。  

模板能够帮助我们，以不变应万变；但模板不是教条主义，要具体问题具体分析，能够根据模板举一反三。  

本篇blog是**二叉树篇**，主要内容是二叉树的遍历模板的积累，4种遍历方式，2种遍历思想。  
4种遍历方式的snippet放在了[我的gist主页](https://gist.github.com/KevinChens/83a0e148d65acd52e01aeed0cb7b47bf)，易于大家自取积累snippet。  

### classification

二叉树的遍历主要有两种，DFS和BFS。  

对于DFS而言，能够**以root结点的访问顺序**细分为前序遍历、中序遍历和后序遍历。DFS的一个遍历特点是**左子树优先右子树访问**。  

以一个简单的二叉树为例，展示三种遍历的结果。  
{{< mermaid >}}
graph TB
A((root))
B((left))
C((right))
A-->B
A-->C
{{< /mermaid >}}

前序遍历-preOrder：  
{{< mermaid >}}
graph LR
A((root))
B((left))
C((right))
A-->B
B-->C
style A fill: #76BA99
{{< /mermaid >}}

中序遍历-inOrder:   
{{< mermaid >}}
graph LR
A((root))
B((left))
C((right))
B-->A
A-->C
style A fill: #76BA99
{{</mermaid >}}

后序遍历-postOrder:   
{{< mermaid >}}
graph LR
A((root))
B((left))
C((right))
B-->C
C-->A
style A fill: #76BA99
{{< /mermaid >}}

对于三种遍历而言，从逻辑上又能够分为递归遍历和非递归遍历。  

而BFS遍历本质是层序遍历，就像排队一样，一层结点遍历完再遍历下一层结点。以上图二叉树为例，BFS遍历结果如下。  
层序遍历-levelOrder：  
{{< mermaid >}}
graph LR
A((root))
B((left))
C((right))
A-.-B
B-->C
{{< /mermaid >}}

下面就分别总结一下4种遍历方式的模板，以一颗简单的二叉树结构为例。  
```go
type TreeNode struct {
    Val int
    Left *TreeNode
    Right *TreeNode
}
```

### preOrder
递归遍历：
```go
func preOrder(root *TreeNode) {
    ans := make([]int, 0)
    ans = append(ans, root.Val)
    preOrder(root.Left)
    preOrder(root.Right)
}
```

非递归遍历：利用stack存储已访问结点，用于原路返回
```go
func preOrder(root *TreeNode) []int {
    if root == nil {
        return nil
    }
    ans := make([]int, 0)
    stack := make([]*TreeNode, 0)
    
    for root != nil || len(stack) > 0 {
        for root != nil {
            // root->left->right
            ans = append(ans, root.Val)
            // push root
            stack = append(stack, root)
            root = root.Left
        }
        // pop top of stack
        node := stack[len(stack)-1]
        stack = stack[:len(stack)-1]
        root = node.Right
    }
    return ans
}
```

### inOrder
递归遍历:
```go
func inOrder(root *TreeNode) {
    ans := make([]int, 0)
    inOrder(root.Left)
    ans = append(ans, root.Val)
    inOrder(root.Right)
}
```

非递归遍历：利用stack存储已访问结点，用于原路返回
```go
func inOrder(root *TreeNode) []int {
    if root == nil {
        return nil
    }
    ans := make([]int, 0)
    stack := make([]*TreeNode, 0)
    
    for root != nil || len(stack) > 0 {
        for root != nil {
            // push root
            stack = append(stack, root)
            root = root.Left
        }
        // pop top of stack
        node := stack[len(stack)-1]
        stack = stack[:len(stack)-1]
        // left->root->right
        ans = append(ans, node.Val)
        root = node.Right
    }
    return ans
}

```

### postOrder
递归遍历:
```go
func postOrder(root *TreeNode) {
    ans := make([]int, 0)
    postOrder(root.Left)
    postOrder(root.Right)
    ans = append(ans, root.Val)
}
```

非递归遍历：同样是利用stack存储已访问结点，但需要一个标识lastVisit，确认右子结点是否已经弹出，保证**root结点在right结点之后弹出**
```go
func postOrder(root *TreeNode) []int {
    if root == nil {
        return nil
    }
    ans := make([]int, 0)
    stack := make([]*TreeNode, 0)
    var lastVisit *TreeNode
    
    for root != nil || len(stack) > 0 {
        for root != nil {
            // push root
            stack = append(stack, root)
            root = root.Left
        }
        // read top of stack
        node := stack[len(stack)-1]
        if node.Right != nil || node.Right != lastVisit {
            root = node.Right
        } else {
            // pop top of stack
            stack = stack[:len(stack)-1]
            // left->right->root
            ans = append(ans, node.Val)
            // marking current node pop
            lastVisit = node
        }
    }
    return ans
}
```

### top-down and bottom-up
同样的对于DFS有三种不同的遍历方式，但也有两种遍历思想，自顶向下和自底向上。
- top-down：将最终结果通过指针参数传递  
- bottom-up：先递归返回结果，最后将其合并，也就是分而治之  

以前序遍历示例两种不同的遍历思想。  
top-down:
```go
func preOrder(root *TreeNode) []int {
    ans := make([]int, 0)
    dfs(root, &ans)
    return ans
}
func dfs(root *TreeNode, ans *[]int) {
    if root == nil {
        return
    }
    // root->left->right
    *ans = append(*ans, root.Val)
    dfs(root.Left, ans)
    dfs(root.Right, ans)
}
```
其实本质是上面所展示的前序遍历的递归方式，结果是通过一个指针进行传参存储的。

bottom-up:
```go
func preOrder(root *TreeNode) []int {
    ans := dfs(root)
    return ans
}
func dfs(root *TreeNode) []int {
    ans := make([]int, 0)
    if root == nil {
        return ans
    }
    // divid
    left := dfs(root.Left)
    right := dfs(root.Right)
    // conquer
    ans = append(ans, root.Val)
    ans = append(ans, left...)
    ans = append(ans, right...)
    return ans
}
```
自底向上，分而治之，核心是每个部分都能够独立解决，最后将每个部分的结果进行归并汇总，就是整个问题的结果。  

分而治之的思想应用是十分广泛的，比如快速排序，归并排序等；这里只是简单引入一下该思想，后面同样会对于分治法进行模板总结。  

### levelOrder
层序遍历：利用queue存储每层结点，保证结点访问的层序性
```go
func levelOrder(root *TreeNode) [][]int {
    if root == nil {
        return nil
    }
    ans := make([][]int, 0)
    queue := make([]*TreeNode, 0)
    // push root
    queue = append(queue, root)
    // level by level
    for len(queue) > 0 {
        list := make([]int, 0)
        // length of current level
        l := len(queue)
        for i := 0; i < l; i++ {
            // pop head of queue
            node := queue[0]
            queue = queue[1:]
            
            list = append(lsit, node.Val)
            // push children of node
            if node.Left != nil {
                queue = append(queue, ndoe.Left)
            }
            if node.Right != nil {
                queue = append(queue, node.RIght)
            }
        }
        // add current level to ans
        ans = append(ans, list)
    }
    return ans
}
```

### summary

本篇blog主要是对二叉树的4种遍历方式进行了总结，DFS包括了前序遍历、中序遍历和后序遍历，而BFS本质就是层序遍历。  
而DFS的实现又能够分为递归实现和非递归实现。同时也引入了自顶向下和自底向上两种思想。  
不断进行模板总结，知识类比细化，对于个人而言，是一个不错的学习算法的方式。在下篇blog将对4种遍历方式进行实践，通过leetcode感受模板的快乐。  
欢迎大家评论转载，多多发表自己的意见或建议。  

### reference

1. [algorithm-pattern仓库](https://github.com/greyireland/algorithm-pattern)

