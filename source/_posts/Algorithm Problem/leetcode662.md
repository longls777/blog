---
title: leetcode662
date: 2022-08-27 22:23:44
tags: DFS
categories: Algorithm Problem
math: true
---

#### [662. 二叉树最大宽度](https://leetcode.cn/problems/maximum-width-of-binary-tree/)

给你一棵二叉树的根节点 `root` ，返回树的 **最大宽度** 。

树的 **最大宽度** 是所有层中最大的 **宽度** 。

每一层的 **宽度** 被定义为该层最左和最右的非空节点（即，两个端点）之间的长度。将这个二叉树视作与满二叉树结构相同，两端点间会出现一些延伸到这一层的 `null` 节点，这些 `null` 节点也计入长度。

题目数据保证答案将会在 **32 位** 带符号整数范围内。

 

**示例 1：**

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/width1-tree.jpg)

```
输入：root = [1,3,2,5,3,null,9]
输出：4
解释：最大宽度出现在树的第 3 层，宽度为 4 (5,3,null,9) 。
```

**示例 2：**

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/maximum-width-of-binary-tree-v3.jpg)

```
输入：root = [1,3,2,5,null,null,9,6,null,7]
输出：7
解释：最大宽度出现在树的第 4 层，宽度为 7 (6,null,null,null,null,null,7) 。
```

**示例 3：**

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/width3-tree.jpg)

```
输入：root = [1,3,2,5]
输出：2
解释：最大宽度出现在树的第 2 层，宽度为 2 (3,2) 。
```

 

**提示：**

- 树中节点的数目范围是 `[1, 3000]`
- `-100 <= Node.val <= 100`

#### 题解

```java
class Solution {
    Map<Integer, Integer> map = new HashMap<>();
    int ans;
    public int widthOfBinaryTree(TreeNode root) {
        dfs(root, 1, 0);
        return ans;
    }
    void dfs(TreeNode root, int u, int depth) {
        if (root == null) return ;
        if (!map.containsKey(depth)) map.put(depth, u);
        ans = Math.max(ans, u - map.get(depth) + 1);
        u = u - map.get(depth) + 1;
        dfs(root.left, u << 1, depth + 1);
        dfs(root.right, u << 1 | 1, depth + 1);
    }
}
```

满二叉树的节点编号规则：若根节点编号为 `u`，则其左子节点编号为 `u << 1`，其右节点编号为 `u << 1 | 1`

![](http://longls777.oss-cn-beijing.aliyuncs.com/img/wKiom1cd666TzHXBAADE1wAN1l4793.png)

可以在 `DFS`过程中使用两个哈希表分别记录每层深度中的最小节点编号和最大节点编号，两者距离即是当前层的宽度，最终所有层数中的最大宽度即是答案。

而实现上，我们可以利用先 DFS 左节点，再 DFS 右节点的性质可知，每层的最左节点必然是最先被遍历到，因此我们只需要记录当前层最先被遍历到点编号（即当前层最小节点编号），并在 DFS 过程中计算宽度，更新答案即可。

为了防止树深度过大导致u溢出，使用`u = u - map.get(depth) + 1`将每一层重编号，这里不太好理解，照着上图自己计算一下就好了，除了root，每一层的编号都是从2开始的

> 作者：AC_OIer
> 链接：https://leetcode.cn/problems/maximum-width-of-binary-tree/solution/by-ac_oier-33er/
> 来源：力扣（LeetCode）